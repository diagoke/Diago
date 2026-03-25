專案名稱：n8n 廣告生成機器人。

環境需求：需準備 n8n, LINE Developers 帳號, 以及 Gemini API。

快速開始：匯入 JSON 檔後，請至 Credentials 設定 API Key。

sequenceDiagram
    autonumber
    participant U as User (LINE App)
    participant L as LINE API Server
    participant N as n8n (Mac Server)
    participant G as Gemini AI (LLM)
    participant S as Google Sheets (DB)

    U->>L: 傳送訊息 (文字/圖片)
    L->>N: HTTPS POST (JSON Payload)
    
    Note over N: 1. Input Ingestion (提取 userId)
    
    rect rgb(240, 240, 240)
    Note over N: 2. Validation (If Node)
    alt Is Image?
        N->>L: 下載 Binary 檔案
        N->>N: Base64 Encoding
    else Is Text?
        N->>N: String Buffer 處理
    end
    end

    N->>S: Append Row: 寫入 Chat_History
    N->>S: Get Row(s): 查詢 userId 歷史筆數
    S-->>N: 回傳所有對話列 (SELECT *)

    Note over N: 3. Decision (JS Code: Sliding Window)
    
    alt Count >= 10 (第 11 輪觸發)
        N->>G: LLM Role 1: Extractive Summarization
        G-->>N: 回傳 100 字用戶畫像
        N->>S: Update State: 存入 Long_Term_Summary
        N->>S: Clear Buffer: 刪除舊對話紀錄
    end

    N->>G: LLM Role 2: AI Agent 生成回覆
    Note right of G: 結合「長記憶摘要」+「即時搜尋」
    G-->>N: 回傳廣告文案
    N->>L: HTTP Request (Push Message)
    L->>U: 顯示生成結果
