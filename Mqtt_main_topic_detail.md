# MQTT 主題參考文件

## 📡 主題架構概覽

### 主題分類
- **上行主題**（輸入至 Python 大腦）：`gpt.ib`, `rag.ib`, `sync.request`
- **下行主題**（Python 大腦輸出）：`ast.is`, `ast.os`
- **執行主題**（機器人動作）：`rdr.is`, `spk.is`, `rbo.is`

### 通信端口
- **MQTT 端口**：61613 (Java/Python 使用)
- **WebSocket 端口**：61623 (前端介面使用)

## 🔼 上行主題 (Input Topics)

### 1. `gpt.ib` - 對話處理請求
**用途**: 處理一般對話和任務請求

**訊息格式**:
```
{id}::{prompt}::{patient_info}::{disease}::{bed_number}::{gender}::{current_position}
```

**範例**:
```
sstc_001::(發話者在護理站)，請去101房間詢問病人用藥情況::病房:101,姓名:張三,疾病:高血壓::高血壓::101::男::護理站
```

**特殊標記**:
- `CONFIRM!!` - 確認模式
- `INTERACTIVE!!` - 互動模式

### 2. `rag.ib` - 知識檢索請求
**用途**: 醫療知識查詢、任務報告、衛生教育

**訊息格式**:
```
{id}::{operation_type}::{operation_state}::{disease}::{prompt}::{bed_number}
```

**操作類型**:
- `media` - 醫療知識查詢
- `report` - 任務報告
- `healthedu` - 衛生教育

**操作狀態** (for media):
- `PREO` - 治療前注意事項
- `POST` - 治療後注意事項
- `PREV` - 預防事項

**範例**:
```
# 醫療知識查詢
sstc_001::media::PREO::高血壓::治療前注意事項::101

# 任務報告
sstc_001::report::::已完成101房間巡視，病人狀況良好::101

# 衛生教育
sstc_001::healthedu::::糖尿病患者的飲食注意事項::202
```



## 🔽 下行主題 (Output Topics)

### 1. `ast.is` - 助手回應
**用途**: 返回對話處理結果

**訊息格式**:
```json
{
    "id": "unique_id",
    "text": ["action_commands"],
    "err": "error_message"
}
```

**動作命令格式**:
```json
{"act": "action_type", "ps": "parameters"}
```

**動作類型**:
- `chat` - 對話回應
- `move` - 移動指令
- `interactive` - 互動提示
- `finish_chat` - 完成對話

**範例**:
```json
{
    "id": "sstc_001",
    "text": [
        "{\"act\":\"chat\", \"ps\":\"我會去101房間詢問張三先生的用藥情況\"}",
        "{\"act\":\"move\", \"ps\":\"101\"}",
        "{\"act\":\"chat\", \"ps\":\"張三先生您好，請問您今天有按時服藥嗎？\"}"
    ],
    "err": ""
}
```

### 2. `ast.os` - 助手輸出狀態
**用途**: 返回知識檢索和報告結果

**訊息格式**:
```json
{
    "id": "unique_id",
    "text": ["status_commands"],
    "err": "error_message"
}
```

**狀態命令格式**:
```json
{"act": "status_type", "ps": "content"}
```

**狀態類型**:
- `mediaRAG` - 醫療知識回應
- `endreport` - 任務報告完成
- `endhealthedu` - 衛生教育完成

**範例**:
```json
{
    "id": "sstc_001",
    "text": ["{\"act\":\"mediaRAG\", \"ps\":\"高血壓患者在治療前應注意監測血壓值，避免劇烈運動...\"}"],
    "err": ""
}
```

## 🎯 執行主題 (Action Topics)

### 1. `rdr.is` - 前端介面控制
**用途**: 控制前端介面顯示和視訊通話
**通信端口**: 61623 (WebSocket)

**訊息格式**:
```json
{
    "command": "command_type",
    "module": "target_module",
    "data": "additional_data"
}
```

**命令類型**:
- `show` - 顯示指定模組
- `hide` - 隱藏指定模組
- `update` - 更新模組內容
- `video_call` - 啟動視訊通話
- `play_video` - 播放視訊
- `show_status` - 更新狀態顯示

**目標模組**:
- `face` - 機器人化身模組
- `jitsi` - 視訊通話模組
- `tasks` - 任務列表模組
- `video` - 視訊播放模組

**範例**:
```json
// 顯示機器人化身
{"command": "show", "module": "face", "status": "talking"}

// 啟動視訊通話
{"command": "video_call", "module": "jitsi", "room": "meet_123456"}

// 播放教育影片
{"command": "play_video", "module": "video", "url": "education_video.mp4"}

// 更新任務狀態
{"command": "update", "module": "tasks", "task_id": "task_001"}
```

### 2. `spk.is` - 語音合成 (TTS)
**用途**: 控制語音合成輸出

**訊息內容**:
- 語音合成文本
- 語音參數設定
- 確認提示音

### 3. `rbo.is` - 機器人移動/旋轉
**用途**: 控制機器人物理動作

**訊息內容**:
- 移動指令 (前進、後退)
- 旋轉指令 (左轉、右轉)
- 位置導航指令



### 常見問題
1. **ID 重複**: 確保同一次請求使用唯一 ID，不同ID會被當作不同次請求
2. **格式錯誤**: 檢查訊息格式是否符合規範
3. **主題訂閱失敗**: 確認 MQTT 連線狀態，確保61613 port 可對外使用
4. **前端 WebSocket 連線失敗**: 確認端口 61623 開放且 Broker 支援 WebSocket
5. **前端界面不更新**: 檢查 JavaScript 控制台錯誤和 MQTT 訊息格式
6. **視訊通話無法啟動**: 確認 Jitsi Meet 配置和網路權限，或是瀏覽器是否阻擋此功能


**注意**: 此文件應與系統更新同步維護，確保主題定義的準確性。 
