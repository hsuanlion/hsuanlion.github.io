# Work Project Highlights

## 1. Project Title  
**LLM-Powered Audience Persona**  
* 用大型語言模型產出受眾Persona、自動化受眾洞察報告。（其他LLM應用包括POI分類器、發票數據類別翻譯器、洞察翻譯器。）

### 3. Problem Statement  
- 場景1: App用戶偏好App清單不夠直觀（不是每個App大家都知道是做什麼的，業務常常會來問）
- 場景2: 每月有二十多個國家旅客到訪日本的動向分析BI報告，想要高效快速的替每份BI加上AI洞察摘要。

### 4. Solution Overview  
- 場景1:
- 核心做法（例如：結合使用者行為資料與 LLM 生成受眾描述）  
- 技術架構圖（如果有的話可以嵌圖或連到 image）  
- 高階流程步驟：  
  1. 資料收集（來源、清洗、ETL）  
  2. 特徵設計（例如：行為向量 + 聚類結果）  
  3. Prompt 設計與 LLM 呼叫（例如：如何把用戶行為轉成 prompt）  
  4. 生成 & 後處理（例如：從 LLM output 做標籤歸類 / 信心分數）  
  5. 呈現（dashboard / report / API）


### 5. Tech Stack  
- 語言／工具：Python, BigQuery, LangChain (如果用), GitHub Actions, Flask/React dashboard...  
- LLM／模型：OpenAI GPT-4、embedding service、自己的 prompt template engine  
- Infra：Cloud（GCP/AWS）、CI/CD、監控方式

### 6. Key Metrics & Impact  
- 原始 baseline vs 成果（例如：洞察生成時間從 3 天降到 5 分鐘；使用者採納率提升 X%；客戶決策速度加快 Y%）  
- 定量結果（KPI、轉換率提升、節省的人力時數、Revenue uplift、A/B 測試結果）  
- 定性反饋（使用者／stakeholder 的短評）

### 7. Challenges & How You Overcame Them  
- 例：LLM output 不穩定 → 加 prompt 層級驗證 + 自動 fallback  
- 資料稀疏 → 用 bootstrapping 與相似用戶補值  
- 效能瓶頸 → 批次化 / cache / rate limit handling

### 8. Lessons Learned / Takeaways  
- 你從這個專案學到什麼  
- 如果再做一次會怎麼改進  
- 可重用的模組、可擴展的設計

### 9. Demo / Screenshots  
- 嵌入圖片（相對路徑或外部 link）  
  ```markdown
  ![Audience Insight Dashboard](/assets/images/llm-insights-dashboard.png)