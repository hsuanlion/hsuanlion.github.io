---
title: LLM Powered Audience Persona
layout: default
permalink: /projects/llm-powered-audience-persona/
---

# LLM-Powered Audience Persona
*用大型語言模型＋行為資料(RAG-like)，自動生成具象化受眾 Persona 與族群標籤，並提出next actions。*

## 1. Summary  
將原本難以解讀的 App 偏好清單，轉成可理解的「受眾 Persona」（包含基本屬性、個性、心理狀態、興趣與生活方式）與 hashtag 特色，以及年齡／性別估計分布，和進階的合作和競業分析，供業務與產品快速抓取群體洞察。

## 2. Role & Responsibilities  
- 主導 LLM 應用設計與 proof-of-concept 建構
- 設計與實作資料 pipeline（App 偏好清單的擷取、正規化與清洗）
- 應用RAG-like context和prompt engineering 組出高質 persona prompt
- 建構 LLM output 的後處理與標籤化（persona 結構化、行銷建議、競品觀察）
- 自動化整個流程並建立 Flask demo site 供內部驗證與展示

## 3. Problem Statement  
業務拿到的只是某族群的 App 偏好清單（App 名 + 安裝偏好指標），但這些原始清單缺乏「這群人到底是什麼樣子的」語意化解讀，導致難以：
- 快速回答：「這群人是誰？他們的行為背後動機為何？」
- 提出合理的合作/行銷建議或與競品比較

![Audience Persona Example](images/llm_persona_01.png) <!-- alt: 轉換後的受眾 persona 範例 -->

## 4. Solution Overview  
- **核心做法：** 結合目標受眾行為資料（例如小紅書用戶的 top-10 App 偏好與對應的 confidence score）與大型語言模型，生成 grounded 的多維度 Persona 與族群標籤與建議。  
- **資料處理流程圖：**  
  ![System Architecture](images/llm_persona_04.png) <!-- alt: 技術架構圖，顯示資料流與 LLM 呼叫流程 -->

- **RAG-like 流程（檢索 + 生成）：**
  1. **資料檢索（Retrieve）：** 從自有資料庫擷取目標受眾的關鍵行為背景 —— 例如該族群的 top-10 App 偏好清單與對應的 confidence score，作為 grounding context。
  2. **Prompt 組裝與呼叫（Generate）：** 把這些行為背景整理放入 prompt 前段（說明這是該受眾的偏好與可信度），再搭配指令模板餵給 LLM 生成 persona（包含基本屬性、動機/痛點、行銷合作建議、競品觀察等）。
  3. **後處理：** 解析 LLM 回傳、結構化為 persona 欄位標籤、組合 hashtag、估計人口統計分布、生成 next action 建議。
  4. **展示：** 透過 Flask demo site 呈現 persona 與其 grounding source，使業務/PM 可以即時檢視與信任結果。
- **目前 scope 限定：** 僅實作「檢索 + 生成」階段（尚未包含 reranking / feedback loop），但已透過提供 grounding context 明顯提升 persona 的相關性與說服力。

- **POC成果展現：**  
<div style="display:grid;grid-template-columns:repeat(2,1fr);gap:1rem;margin:1rem 0;">
  <div>
    <img src="/projects/images/llm_persona_05.png" alt="Demo site的Input/Output" style="width:100%;border-radius:6px;">
    <div style="font-size:0.8rem;margin-top:4px;color:#555;">Demo site的Input和Output</div>
  </div>
  <div>
    <img src="/projects/images/llm_persona_06.png" alt="LLM output & persona framework" style="width:100%;border-radius:6px;">
    <div style="font-size:0.8rem;margin-top:4px;color:#555;">LLM output & persona framework</div>
  </div>
  <div>
    <img src="/projects/images/llm_persona_07.png" alt="合作建議" style="width:100%;border-radius:6px;">
    <div style="font-size:0.8rem;margin-top:4px;color:#555;">合作建議</div>
  </div>
  <div>
    <img src="/projects/images/llm_persona_08.png" alt="競爭者分析" style="width:100%;border-radius:6px;">
    <div style="font-size:0.8rem;margin-top:4px;color:#555;">競爭者分析</div>
  </div>
</div>

## 5. Tech Stack  
- **Languages / Tools:** Python (OpenAI), BigQuery, Flask  
- **LLM / Model:** OpenAI `gpt-3.5-turbo`  

## 6. Impact  
- 將難解的 App 偏好數據轉化成有故事、有行動建議的 persona 檔案，讓業務與產品可以快速理解目標族群並推動合作／行銷策略。
- 建立可重用的 prompt pipeline 與模板，能延伸至其他場景（如 POI 分類、中文標籤翻譯、報表摘要等）。
- 在團隊內分享 prompt engineering 的實作經驗（參數對比、格式設計、語言選擇），提升整體 AI 使用成熟度。

## 7. Challenges & Solutions  
- **LLM 輸出不穩定 / 發散**
   
    解法：設計固定格式的 prompt template、提供 few-shot 範例、用 JSON-like 結構強制欄位、調低 temperature 以減少隨機性。
    
- **指令理解與語言差異**
    
    解法：比較中文與英文表現，採用表現更佳的語言並用 backticks 區隔資訊、拆解任務成步驟（step-by-step），確保 prompt 有一致 SOP。
    
- **有限背景會導致重複或泛化詞彙**
    
    解法：透過調整 top_p、presence/frequency penalty 控制生成多樣性與重複度，並在 prompt 裡給出具體限制與例子。

## 8. Lessons Learned / Takeaways  
- **學到的事：** 明確、有範例的 grounding prompt 能大幅提升 persona 的一致性與可用性；提供行為背景（RAG-like context）讓 LLM 生成更有根據、可信的描述。
<!-- - **再做一次會改進：**
    - 引入 reranking 或 consistency check 來簡單驗證生成 persona 與原始 grounding 是否匹配。
    - 建立 feedback loop：把高品質 persona 重新 chunk 進入 index，逐步強化 retrieval context。 -->
- **可重用性：** prompt pipeline 及輸出結構化模組可套用到其他受眾 / 報表分析。
- **模組化設計：** 每一步（檢索、prompt 組裝、LLM 呼叫、後處理、展示）分離封裝，降低新 case 上手門檻。


其他應用

<div style="display:grid;grid-template-columns:repeat(2,1fr);gap:1rem;margin:1rem 0;">
  <div>
    <img src="/projects/images/llm_persona_12.png" alt="其他分析模組的範例" style="width:100%;border-radius:6px;">
    <div style="font-size:0.8rem;margin-top:4px;color:#555;">Audience Insight Report的AI摘要示意圖</div>
  </div>
  <div>
    <img src="/projects/images/llm_persona_11.png" alt="POI 分類器" style="width:100%;border-radius:6px;">
    <div style="font-size:0.8rem;margin-top:4px;color:#555;">POI 分類器</div>
  </div>
</div>


## 9. Demo Site Snapshot
Flask-based Demo Site for proof-of-concept: 展示persona生成結果與可測試input和output的介面。  
![Audience Insight Dashboard](images/llm_persona_13.png) <!-- alt: Flask demo dashboard -->
![Audience Insight Dashboard](images/llm_persona_14.png) <!-- alt: Flask demo dashboard -->

---

## Appendix

### Code Snippet: Chat Completion Helper
```python
def get_completion(prompt, model="gpt-3.5-turbo"):
    messages = [{"role": "user", "content": prompt}]
    response = openai.ChatCompletion.create(
        model=model,
        messages=messages,
        temperature=0,  # 控制隨機性：0 表示最穩定
    )
    return response.choices[0].message["content"]
```

### Prompt Design （中譯整理版)
```text
Step 1:
根據下面由三個反引號包住的用戶 App 偏好清單，生成用戶輪廓，包括：用戶基本屬性、個性、心理狀態、個人興趣、生活方式。
- 不超過 5 句話。
- 使用中文，每句用 bullet point 分開。
- 回傳 JSON 格式，keys: 用戶基本屬性, 用戶個性, 用戶心理狀態, 用戶個人興趣, 用戶生活方式

Step 2:
為該用戶群擬出 2~3 個生活型態特色 hashtag，中文與英文各一組，用逗號分隔。
- 回傳 JSON 格式，keys: Chinese, English

Step 3:
根據同樣的 App 偏好清單，估計年齡層分布（加總為 100%）。
- 年齡群列表：{", ".join(age_group_list)}
- 回傳 JSON，keys 為各年齡群

Step 4:
根據 App 偏好清單，估計性別分布（加總為 100%）。
- 性別列表：{", ".join(gender_list)}
- 回傳 JSON，keys 為各性別

User App preference: {AppPreference}

最後，把以上結果整合成一個 JSON 物件，格式如下範例：

```json
{
  "App Preference": "...",
  "User Profile": {
    "用戶基本屬性": "...",
    "用戶個性": "...",
    "用戶心理狀態": "...",
    "用戶個人興趣": "...",
    "用戶生活方式": "..."
  },
  "Hashtags": {
    "Chinese": "...",
    "English": "..."
  },
  "Age Group": {
    "15-19": 30,
    "20-25": 40,
    ...
  },
  "Gender": {
    "male": 60,
    "female": 40
  }
}
```

