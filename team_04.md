# 作業四：帕金森氏症早期語音風險評估系統

## 一、UML 類別圖（Class Diagram）

### 1.1 系統類別圖說明

本系統主要透過自由說話與母音發聲兩種評估方式，蒐集使用者的語音資料，經過音訊品質檢查及人工智慧模型分析後，產生語音風險指標，並提供歷史紀錄及趨勢查詢功能。

為了讓各項功能容易維護與擴充，本系統的類別設計分為四個部分：

1. **評估資料管理**：管理使用者、評估紀錄、錄音資料及評估結果。
2. **音訊處理**：負責錄音控制、音訊品質檢查及重新錄音。
3. **AI 語音分析**：分別處理自由說話及母音發聲的特徵擷取與 SVM 模型分析。
4. **歷史資料查詢**：負責歷史紀錄、日期篩選及風險指標趨勢圖。

本類別圖為依據系統需求規劃的設計模型，並非直接從既有程式碼反向產生。

### 1.2 核心資料類別圖

```mermaid
classDiagram
direction TB

class User {
    +String userId
    +String displayName
    +viewHistory()
    +startAssessment()
}

class Assessment {
    +String assessmentId
    +String assessmentType
    +DateTime assessmentTime
    +String status
    +start()
    +complete()
    +markFailed()
}

class AudioRecording {
    +String recordingId
    +String assessmentId
    +String vowelType
    +String filePath
    +double duration
    +String recordingStatus
    +saveRecording()
    +getDuration()
}

class QualityCheck {
    +String checkId
    +String recordingId
    +double noiseLevel
    +double volumeLevel
    +bool isComplete
    +bool isPassed
    +checkNoise()
    +checkVolume()
    +checkCompleteness()
    +getResult()
}

class AssessmentResult {
    +String resultId
    +String assessmentId
    +double riskScore
    +String riskLevel
    +String summary
    +String modelVersion
    +DateTime resultTime
    +getRiskLevel()
    +getSummary()
}

class HistoryService {
    +getAssessments()
    +filterByType()
    +filterByDate()
    +getAssessmentDetail()
}

class TrendService {
    +getTrendData()
    +comparePreviousResult()
    +groupByAssessmentType()
}

User "1" --> "0..*" Assessment : 擁有
Assessment "1" *-- "0..*" AudioRecording : 包含
AudioRecording "1" --> "0..*" QualityCheck : 接受檢查
Assessment "1" --> "0..1" AssessmentResult : 產生

HistoryService ..> Assessment : 查詢
HistoryService ..> AssessmentResult : 讀取
TrendService ..> AssessmentResult : 統計
```

### 1.3 核心資料類別說明

| 類別名稱 | 主要屬性 | 主要方法 | 功能說明 |
|:---|:---|:---|:---|
| User | userId、displayName | startAssessment、viewHistory | 代表使用者及其評估操作。 |
| Assessment | assessmentId、assessmentType、assessmentTime、status | start、complete、markFailed | 管理一次語音評估的基本資訊及狀態。 |
| AudioRecording | recordingId、assessmentId、vowelType、filePath、duration | saveRecording、getDuration | 保存自由說話或個別母音的錄音資訊。 |
| QualityCheck | checkId、noiseLevel、volumeLevel、isComplete、isPassed | checkNoise、checkVolume、checkCompleteness | 記錄音訊品質檢查的結果。 |
| AssessmentResult | resultId、riskScore、riskLevel、summary、modelVersion | getRiskLevel、getSummary | 保存一次評估產生的模型風險指標及相關資訊。 |
| HistoryService | 無固定資料屬性 | getAssessments、filterByType、filterByDate | 提供歷史紀錄查詢及篩選。 |
| TrendService | 無固定資料屬性 | getTrendData、comparePreviousResult | 提供歷史風險指標趨勢及前後次比較。 |

### 1.4 類別之間的關係

**1. User 與 Assessment**

一位使用者可以進行多次評估，因此 User 與 Assessment 為一對多關係。

**2. Assessment 與 AudioRecording**

一次自由說話評估通常包含一段有效錄音；一次母音發聲評估則需要 A、E、I、O、U 五段有效錄音。

每段錄音都有對應的識別碼、時長及評估編號。

**3. AudioRecording 與 QualityCheck**

每段錄音都需要進行音訊品質檢查。

若錄音品質不合格，系統應提示使用者重新錄音；重新檢查時，可以產生新的檢查紀錄。

**4. Assessment 與 AssessmentResult**

一次成功完成的評估產生一筆最終結果。

尚未完成或分析失敗的評估可能沒有結果，因此使用零或一筆結果的關係。

**5. HistoryService 與 TrendService**

HistoryService 提供歷史評估紀錄，TrendService 則使用評估結果資料計算歷史趨勢。

兩者分別負責資料查詢及統計呈現。

---

### 1.5 AI 語音分析服務類別圖

自由說話與母音發聲使用不同的特徵及分析流程，因此以共同的分析介面加上兩種分析服務表示。

```mermaid
classDiagram
direction LR

class VoiceAnalysisService {
    <<interface>>
    +analyze()
    +validateFeatures()
}

class FreeSpeechAnalyzer {
    +String modelVersion
    +analyze()
    +transcribeAudio()
    +createEmbedding()
    +normalizeFeatures()
    +predictRisk()
}

class VowelAnalyzer {
    +String modelVersion
    +analyze()
    +extractAcousticFeatures()
    +normalizeFeatures()
    +predictRisk()
}

class WhisperService {
    +transcribe()
}

class EmbeddingService {
    +createEmbedding()
}

class LibrosaService {
    +extractFeatures()
}

class SVMModel {
    +String modelVersion
    +predict()
}

class AnalysisController {
    +analyzeAssessment()
    +getAnalysisStatus()
}

VoiceAnalysisService <|.. FreeSpeechAnalyzer
VoiceAnalysisService <|.. VowelAnalyzer

AnalysisController ..> VoiceAnalysisService : 呼叫

FreeSpeechAnalyzer --> WhisperService : 轉文字
FreeSpeechAnalyzer --> EmbeddingService : 文字特徵
FreeSpeechAnalyzer --> SVMModel : 自由說話模型

VowelAnalyzer --> LibrosaService : 聲學特徵
VowelAnalyzer --> SVMModel : 母音模型
```

### 1.6 AI 分析類別說明

| 類別名稱 | 功能說明 |
|:---|:---|
| VoiceAnalysisService | 定義不同評估方式共同使用的分析介面。 |
| FreeSpeechAnalyzer | 執行自由說話的語音轉文字、文字特徵處理及 SVM 分析。 |
| VowelAnalyzer | 執行五個母音的聲學特徵擷取及 SVM 分析。 |
| WhisperService | 將自由說話錄音轉換成文字。 |
| EmbeddingService | 將文字轉換成可供模型分析的特徵向量。 |
| LibrosaService | 擷取母音錄音的聲學特徵。 |
| SVMModel | 代表 SVM 預測功能；兩種評估方式使用各自訓練的模型。 |
| AnalysisController | 根據評估類型呼叫對應分析服務，並管理分析狀態。 |

### 1.7 AI 分析設計說明

#### 自由說話分析

自由說話採用以下流程：

1. 取得自由說話錄音。
2. 使用 Whisper 將語音轉換為文字。
3. 使用文字 Embedding 模型產生特徵向量。
4. 依照模型訓練時的規格進行標準化。
5. 使用自由說話專用 SVM 模型進行分析。
6. 將分析輸出轉換為系統定義的風險指標。
7. 保存分析結果。

#### 母音發聲分析

母音發聲採用以下流程：

1. 取得 A、E、I、O、U 五段合格錄音。
2. 使用 librosa 擷取各段錄音的聲學特徵。
3. 依照模型訓練時的規格進行特徵整理與標準化。
4. 將整理後的特徵輸入母音專用 SVM 模型。
5. 產生一次母音發聲評估的風險指標。
6. 保存分析結果。

#### 設計原則

- 自由說話與母音發聲使用不同的 SVM 模型。
- 五個母音共同產生一次母音發聲評估結果。
- 模型輸出必須依實際計分方式轉換為風險指標。
- 不可將風險指標直接解讀成罹患帕金森氏症的機率。
- 應保存模型版本，以利後續分析結果比較。

---

## 二、循序圖（Sequence Diagram）

循序圖用來表示使用者與系統之間的互動順序，描述在特定使用情境中，各個元件如何傳遞訊息並完成任務。

本作業選擇四個使用案例進行設計：

- UC02：自由說話評估
- UC03：母音發聲評估
- UC09：查看當次評估結果
- UC10：查看歷史評估紀錄與趨勢

### 2.1 UC02 自由說話評估循序圖

```mermaid
sequenceDiagram
    autonumber

    actor U as 使用者
    participant APP as App 介面
    participant REC as 錄音服務
    participant QC as 音訊品質檢查
    participant AI as 自由說話分析
    participant DB as 評估資料儲存

    U->>APP: 選擇自由說話評估
    APP-->>U: 顯示注意事項與題目

    U->>APP: 點擊開始錄音
    APP->>REC: 開始錄音
    REC-->>APP: 回報錄音時間
    APP-->>U: 顯示錄音進度

    alt 錄音滿 10 秒後手動結束
        U->>APP: 點擊結束錄音
        APP->>REC: 停止錄音
    else 錄音達 25 秒
        REC->>REC: 自動停止錄音
    end

    REC-->>APP: 回傳音訊

    APP->>QC: 檢查錄音品質
    QC->>QC: 檢查噪音、音量、完整性

    alt 音訊品質未通過
        QC-->>APP: 回傳未通過原因
        APP-->>U: 顯示重新錄音提示
    else 音訊品質通過
        QC-->>APP: 回傳檢查通過
        APP->>AI: 提交自由說話音訊
        AI->>AI: Whisper 語音轉文字
        AI->>AI: 產生文字 Embedding
        AI->>AI: 特徵標準化
        AI->>AI: 自由說話 SVM 分析
        AI-->>APP: 回傳風險指標
        APP->>DB: 保存評估結果
        DB-->>APP: 保存完成
        APP-->>U: 顯示本次評估結果
    end
```

#### 流程說明

1. 使用者選擇自由說話評估。
2. 系統顯示錄音注意事項及題目。
3. 使用者開始錄音，系統持續顯示錄音進度。
4. 錄音滿 10 秒後可以手動結束，最長 25 秒。
5. 系統對音訊執行品質檢查。
6. 品質未通過時，系統提示重新錄音。
7. 品質通過時，系統執行 Whisper、Embedding、標準化及 SVM 分析。
8. 系統保存並顯示評估結果。

---

### 2.2 UC03 母音發聲評估循序圖

```mermaid
sequenceDiagram
    autonumber

    actor U as 使用者
    participant APP as App 介面
    participant DEMO as 示範音訊服務
    participant REC as 錄音服務
    participant QC as 音訊品質檢查
    participant AI as 母音分析服務
    participant DB as 評估資料儲存

    U->>APP: 選擇母音發聲評估
    APP-->>U: 顯示 A E I O U 進度

    loop 依序完成 A、E、I、O、U
        APP-->>U: 顯示目前母音

        loop 目前母音尚未通過品質檢查
            opt 使用者需要聆聽示範
                U->>APP: 點擊播放示範
                APP->>DEMO: 播放目前母音音訊
                DEMO-->>U: 播放發聲示範
            end

            U->>APP: 點擊開始錄音
            APP->>DEMO: 停止示範播放
            APP->>REC: 開始錄音
            REC-->>APP: 回報錄音時間
            APP-->>U: 顯示錄音進度

            U->>APP: 完成約 4 至 6 秒發聲
            APP->>REC: 停止錄音
            REC-->>APP: 回傳目前母音錄音

            APP->>QC: 檢查目前母音
            QC->>QC: 檢查時間與音訊品質

            alt 錄音不合格
                QC-->>APP: 回傳失敗原因
                APP-->>U: 提示重錄目前母音
            else 錄音合格
                QC-->>APP: 回傳檢查通過
                APP->>DB: 保存目前母音錄音
                DB-->>APP: 保存完成
                APP-->>U: 顯示完成狀態
            end
        end
    end

    APP->>DB: 讀取五段合格母音錄音
    DB-->>APP: 回傳五段音訊

    APP->>AI: 提交五個母音
    AI->>AI: librosa 擷取聲學特徵
    AI->>AI: 特徵整理與標準化
    AI->>AI: 母音 SVM 模型分析
    AI-->>APP: 回傳單次評估結果

    APP->>DB: 保存母音評估結果
    DB-->>APP: 保存完成
    APP-->>U: 顯示風險指標
```

#### 流程說明

1. 使用者進入母音發聲評估頁面。
2. 系統依序顯示 A、E、I、O、U 五個母音。
3. 使用者可聆聽目前母音的示範音訊。
4. 使用者在同一頁面開始錄音。
5. 系統停止示範音訊，避免影響正式錄音。
6. 使用者使用一口氣發聲約 4～6 秒。
7. 系統檢查目前母音的時間及錄音品質。
8. 若品質不合格，使用者重新錄製目前母音，直到通過品質檢查。
9. 若品質合格，系統保存錄音並進入下一個母音。
10. 五個母音完成後，系統使用 librosa 擷取聲學特徵。
11. 系統使用母音專用 SVM 模型進行分析。
12. 系統產生並保存一次母音發聲評估結果。

---

### 2.3 UC09 查看當次評估結果循序圖

```mermaid
sequenceDiagram
    autonumber

    actor U as 使用者
    participant APP as App 介面
    participant RESULT as 結果處理服務
    participant DB as 評估資料儲存

    Note over APP,DB: 語音分析已完成，評估結果已保存

    U->>APP: 進入當次評估結果頁面
    APP->>RESULT: 取得當次評估結果
    RESULT->>DB: 查詢已保存的評估結果
    DB-->>RESULT: 回傳風險指標、等級與摘要
    RESULT-->>APP: 回傳完整評估結果

    APP-->>U: 顯示風險指標與等級
    APP-->>U: 顯示日期、方式、摘要
    APP-->>U: 顯示非診斷提醒

    alt 使用者查看歷史紀錄
        U->>APP: 點擊查看評估紀錄
        APP->>DB: 查詢歷史紀錄
        DB-->>APP: 回傳歷史資料
        APP-->>U: 顯示評估紀錄頁
    else 使用者返回首頁
        U->>APP: 點擊首頁
        APP-->>U: 顯示系統首頁
    end
```

#### 流程說明

1. 自由說話或母音發聲評估完成後，系統已將分析結果保存。
2. 使用者進入當次評估結果頁面。
3. 結果處理服務查詢已保存的評估資料。
4. 系統取得風險指標、風險等級及分析摘要。
5. 系統顯示本次評估的日期、方式、風險指標及摘要。
6. 使用者可以進入歷史紀錄頁，或返回首頁。

本使用案例主要負責顯示當次評估結果，不會重複建立或保存評估資料。

#### 風險指標注意事項

目前原型介面使用低風險、中度風險與較高風險三種分類。

這些分類是介面展示規則，尚不代表通過臨床驗證的診斷門檻。

系統應明確說明評估結果僅供參考，不能取代醫療診斷。

---

### 2.4 UC10 查看歷史紀錄與趨勢循序圖

```mermaid
sequenceDiagram
    autonumber

    actor U as 使用者
    participant APP as App 介面
    participant HISTORY as 歷史紀錄服務
    participant TREND as 趨勢分析服務
    participant DB as 評估資料儲存

    U->>APP: 點擊紀錄
    APP->>HISTORY: 查詢歷史評估紀錄
    HISTORY->>DB: 讀取評估資料
    DB-->>HISTORY: 回傳紀錄
    HISTORY-->>APP: 回傳紀錄列表

    APP->>TREND: 取得趨勢資料
    TREND->>DB: 查詢風險指標
    DB-->>TREND: 回傳日期與指標
    TREND->>TREND: 依評估類型分組
    TREND-->>APP: 回傳圖表資料
    APP-->>U: 顯示紀錄與折線圖

    opt 使用者切換時間範圍
        U->>APP: 選擇近 1、3、6、12 個月
        APP->>HISTORY: 依期間查詢
        HISTORY->>DB: 讀取符合期間的資料
        DB-->>HISTORY: 回傳篩選結果
        HISTORY-->>APP: 回傳紀錄

        APP->>TREND: 取得所選期間的趨勢資料
        TREND->>DB: 查詢符合期間的風險指標
        DB-->>TREND: 回傳日期與指標
        TREND->>TREND: 依評估類型整理趨勢
        TREND-->>APP: 回傳更新圖表
        APP-->>U: 更新紀錄與趨勢圖
    end

    opt 使用者自訂日期
        U->>APP: 點擊日曆
        APP-->>U: 顯示日期篩選視窗
        U->>APP: 選擇開始與結束日期

        APP->>HISTORY: 查詢指定日期範圍
        HISTORY->>DB: 讀取指定日期的評估紀錄
        DB-->>HISTORY: 回傳符合條件的紀錄
        HISTORY-->>APP: 回傳紀錄列表

        APP->>TREND: 取得指定日期的趨勢資料
        TREND->>DB: 查詢指定期間的風險指標
        DB-->>TREND: 回傳日期與指標
        TREND->>TREND: 依評估類型整理趨勢
        TREND-->>APP: 回傳更新圖表
        APP-->>U: 更新列表與趨勢圖
    end

    opt 使用者查看單筆紀錄
        U->>APP: 點擊歷史紀錄
        APP->>HISTORY: 取得紀錄詳情
        HISTORY->>DB: 查詢指定評估
        DB-->>HISTORY: 回傳詳細資料
        HISTORY-->>APP: 回傳評估詳情
        APP-->>U: 顯示詳細結果
    end
```

#### 流程說明

1. 使用者進入評估紀錄頁面。
2. 系統讀取使用者有權查看的歷史紀錄。
3. 系統依評估方式整理歷史風險指標。
4. 系統顯示自由說話及母音發聲的不同資料系列。
5. 使用者可以切換近一個月、近三個月、近六個月及近一年。
6. 使用者可以透過日曆自訂日期範圍。
7. 系統根據所選日期重新查詢歷史紀錄及趨勢資料。
8. 系統更新符合篩選條件的紀錄與趨勢圖。
9. 使用者可以進入單筆紀錄詳細頁面。

---

## 三、活動圖（Activity Diagram）

活動圖用來描述特定使用案例中的操作步驟、判斷條件及不同情況下的處理方式。

以下活動圖與循序圖對應同樣的四個使用案例。

### 3.1 UC02 自由說話評估活動圖

```mermaid
flowchart TD
    START([開始]) --> A1["選擇自由說話評估"]
    A1 --> A2["顯示錄音注意事項"]
    A2 --> A3{"麥克風權限已開啟？"}

    A3 -- 否 --> E1["顯示權限提示"]
    E1 --> A3

    A3 -- 是 --> A4["顯示自由說話題目"]
    A4 --> A5["開始錄音"]
    A5 --> A6["持續錄音並更新時間"]
    A6 --> D1{"錄音達 25 秒？"}

    D1 -- 是 --> A7["自動停止錄音"]
    D1 -- 否 --> D2{"已滿 10 秒且使用者結束？"}

    D2 -- 否 --> A6
    D2 -- 是 --> A7

    A7 --> A8["執行音訊品質檢查"]
    A8 --> D3{"品質是否合格？"}

    D3 -- 否 --> E2["顯示失敗原因與重錄建議"]
    E2 --> A4

    D3 -- 是 --> A9["Whisper 語音轉文字"]
    A9 --> A10["產生文字 Embedding"]
    A10 --> A11["特徵標準化"]
    A11 --> A12["自由說話 SVM 分析"]
    A12 --> D4{"分析成功？"}

    D4 -- 否 --> E3["顯示分析失敗及重試提示"]
    E3 --> A12

    D4 -- 是 --> A13["保存評估結果"]
    A13 --> A14["顯示風險指標與摘要"]
    A14 --> END([結束])
```

#### 活動圖說明

本流程主要包含三項判斷：

1. 麥克風權限是否正常。
2. 錄音時間與音訊品質是否符合規格。
3. AI 模型分析是否成功。

若錄音品質不合格，使用者需重新錄音；若分析失敗，系統應提供重試方式，不應產生虛構的評估結果。

---

### 3.2 UC03 母音發聲評估活動圖

```mermaid
flowchart TD
    START([開始]) --> A1["選擇母音發聲評估"]
    A1 --> A2["初始化 A、E、I、O、U 進度"]
    A2 --> A3["顯示目前母音"]
    A3 --> D1{"需要聆聽示範？"}

    D1 -- 是 --> A4["播放目前母音示範"]
    D1 -- 否 --> A5["準備錄音"]
    A4 --> A5

    A5 --> A6["停止示範音訊"]
    A6 --> A7["開始錄音"]
    A7 --> A8["一口氣持續發聲約 4～6 秒"]
    A8 --> A9["結束目前母音錄音"]
    A9 --> A10["執行音訊品質檢查"]
    A10 --> D2{"時間與品質合格？"}

    D2 -- 否 --> E1["提示重新錄製目前母音"]
    E1 --> A3

    D2 -- 是 --> A11["保存目前母音錄音"]
    A11 --> A12["更新已完成進度"]
    A12 --> D3{"五個母音全部完成？"}

    D3 -- 否 --> A13["切換下一個母音"]
    A13 --> A3

    D3 -- 是 --> A14["取得五段合格錄音"]
    A14 --> A15["librosa 擷取聲學特徵"]
    A15 --> A16["特徵整理與標準化"]
    A16 --> A17["母音 SVM 模型分析"]
    A17 --> D4{"分析成功？"}

    D4 -- 否 --> E2["顯示分析失敗"]
    E2 --> A17

    D4 -- 是 --> A18["產生一次母音評估結果"]
    A18 --> A19["保存結果並顯示"]
    A19 --> END([結束])
```

#### 活動圖說明

母音發聲評估採用循環式操作。

使用者完成一個母音後，系統檢查該母音的錄音品質。

若品質不合格，只需重新錄製目前母音。

若品質合格，系統保存音訊並更新完成進度，接著進入下一個母音。

五個母音全部完成後，系統才進行完整的母音聲學特徵分析，並產生一次評估結果。

---

### 3.3 UC09 查看當次評估結果活動圖

```mermaid
flowchart TD
    START([開始]) --> A1["進入當次評估結果頁面"]
    A1 --> A2["查詢已保存的評估結果"]
    A2 --> D1{"是否成功取得評估結果？"}

    D1 -- 否 --> E1["顯示無法取得結果提示"]
    E1 --> END1([結束：未顯示結果])

    D1 -- 是 --> A3["取得風險指標、等級與摘要"]
    A3 --> A4["顯示評估結果頁面"]
    A4 --> A5["顯示非醫療診斷提醒"]

    A5 --> D2{"使用者下一步操作？"}

    D2 -- 查看紀錄 --> A6["進入歷史評估紀錄"]
    D2 -- 返回首頁 --> A7["回到系統首頁"]

    A6 --> END2([結束])
    A7 --> END2
```

#### 活動圖說明

系統應先確認當次評估已完成，且有成功保存的評估結果。

使用者進入結果頁面時，系統讀取已儲存的資料，不需重新進行 AI 分析或保存結果。

結果頁面提供風險指標、風險等級、分析摘要及非診斷提醒。

若無法取得評估結果，系統應顯示提示訊息。

使用者可以選擇查看歷史紀錄或返回首頁。

---

### 3.4 UC10 歷史紀錄與趨勢查詢活動圖

```mermaid
flowchart TD
    START([開始]) --> A1["進入歷史評估紀錄"]
    A1 --> A2["讀取歷史評估資料"]
    A2 --> A3["顯示紀錄列表及趨勢圖"]
    A3 --> D1{"使用者選擇操作？"}

    D1 -- 評估類型篩選 --> A4["選擇全部、自由說話或母音"]
    A4 --> A5["依評估類型查詢"]
    A5 --> A10["更新紀錄與趨勢圖"]

    D1 -- 時間範圍切換 --> A6["選擇近 1、3、6 或 12 個月"]
    A6 --> A7["依所選期間查詢"]
    A7 --> A10

    D1 -- 自訂日期 --> A8["開啟日期篩選視窗"]
    A8 --> A9["輸入開始與結束日期"]
    A9 --> D2{"日期範圍有效？"}

    D2 -- 否 --> E1["提示日期範圍錯誤"]
    E1 --> A8

    D2 -- 是 --> A10

    A10 --> D3{"有符合條件的資料？"}

    D3 -- 否 --> E2["顯示此期間尚無評估紀錄"]
    D3 -- 是 --> A11["顯示篩選後資料"]

    E2 --> A3
    A11 --> A3

    D1 -- 查看單筆詳情 --> A12["選擇一筆評估紀錄"]
    A12 --> A13["顯示當次結果與前次比較"]
    A13 --> END([結束])

    D1 -- 離開紀錄頁 --> END
```

#### 活動圖說明

歷史紀錄頁面提供多種查詢方式：

1. 依照評估方式篩選。
2. 依照固定時間範圍切換。
3. 透過日曆自訂日期範圍。
4. 點選單筆紀錄查看詳細資料。

系統應檢查日期範圍是否正確，並依照查詢條件更新歷史紀錄與趨勢圖。

若沒有符合條件的資料，應顯示提示訊息，而非產生不存在的結果。

---

## 四、使用案例與圖表對照表

| 使用案例編號 | 使用案例名稱 | 循序圖 | 活動圖 |
|:---:|:---|:---:|:---:|
| UC02 | 自由說話評估 | 已繪製 | 已繪製 |
| UC03 | 母音發聲評估 | 已繪製 | 已繪製 |
| UC09 | 查看當次評估結果 | 已繪製 | 已繪製 |
| UC10 | 查看歷史評估紀錄與趨勢 | 已繪製 | 已繪製 |

本作業共繪製：

- 2 張 UML 類別圖。
- 4 張循序圖。
- 4 張活動圖。

其中，循序圖與活動圖均對應作業三中定義的使用案例。

---

## 五、系統設計重點

### 5.1 自由說話與母音發聲分開分析

本系統包含兩種不同的語音評估方式。

自由說話主要使用 Whisper、文字 Embedding 及 SVM；母音發聲主要使用 librosa 聲學特徵及另一個 SVM 模型。

兩條分析流程應各自管理，避免混用不相容的資料與模型。

### 5.2 五個母音共同產生一次評估

A、E、I、O、U 五個母音分別錄音，每個母音的目標發聲時間約為 4～6 秒。

五段錄音皆符合品質要求後，才進行完整母音分析，最後產生一次母音發聲評估結果。

若其中一個母音錄音不合格，只需重新錄製該母音，其他已完成的合格錄音可以保留。

### 5.3 錄音品質檢查

系統應於 AI 分析前檢查環境噪音、錄音音量及錄音完整性。

不符合要求的錄音不得直接視為有效評估資料，應提示使用者重新錄音。

### 5.4 歷史紀錄與風險指標

系統可將不同日期的評估結果整理成趨勢圖，協助使用者觀察長期變化。

自由說話與母音發聲應使用不同資料系列呈現。

若比較前後兩次結果，必須確認兩者屬於相同評估方式，且模型計分規格具有可比較性。

使用者切換時間範圍或自訂日期時，系統應同步更新歷史紀錄及趨勢圖，確保顯示的資料符合查詢條件。

### 5.5 系統定位

本系統定位為帕金森氏症早期語音風險評估的輔助工具。

模型產生的風險指標不代表疾病確診結果，也不能直接視為罹患疾病的機率。

實際醫療判斷仍需由專業醫療人員進行。

---

## 六、總結

本次作業以帕金森氏症早期語音風險評估系統為主題，依照作業三所建立的需求與使用案例，完成 UML 類別圖、循序圖及活動圖的設計。

類別圖主要呈現評估紀錄、錄音資料、音訊品質檢查、AI 分析及歷史趨勢等類別的屬性、方法與關係。

循序圖則透過使用者、App 介面、錄音服務、音訊品質檢查、AI 語音分析服務及資料儲存等參與者，描述各項功能的訊息傳遞順序。

活動圖進一步呈現評估過程中的判斷條件、正常流程、重新錄音、分析異常及歷史紀錄查詢等操作分支。

透過這些 UML 圖表，可以更加明確地了解系統各項功能之間的關係與執行流程，並作為後續 App 開發、AI 模型整合及功能測試的設計依據。
