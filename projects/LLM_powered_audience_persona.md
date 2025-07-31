---
title: LLM Powered Audience Persona
layout: default
permalink: /projects/llm-powered-audience-persona/
---

# LLM-Powered Audience Persona
*用大型語言模型＋行為資料，自動生成具象化受眾 Persona 與族群標籤。*

## 1. Summary  
將原本難以解讀的 App 偏好清單，轉成可理解的「受眾 Persona」（包含基本屬性、個性、心理狀態、興趣與生活方式）與 hashtag 特色，以及年齡／性別估計分布，供業務與產品快速抓取群體洞察。

## 2. Role & Responsibilities  
- Lead Data Scientist / Owner of the proof-of-concept  
- 設計資料 pipeline（偏好清單收集與清洗）  
- Prompt engineering for LLM  
- 架構 LLM output 後處理與標籤化  
- 建立 Flask demo 供內部驗證與展示  

## 3. Problem Statement  
業務收到的 App 偏好清單只是「App 名單」，許多客戶或內部成員不熟悉這些 App 的意義，難以直觀理解某群體的特性。  
業務常詢問：這些用戶到底是什麼樣的人？他們的生活方式、心理狀態、興趣是什麼？  
![Audience Persona Example](images/llm_persona_01.png) <!-- alt: 轉換後的受眾 persona 範例 -->

## 4. Solution Overview  
- **核心做法：** 結合使用者行為（App 偏好清單）與大型語言模型，自動產出多維度 persona 與群體標籤。  
- **技術架構圖：**  
  ![System Architecture](images/llm_persona_04.png) <!-- alt: 技術架構圖，顯示資料流與 LLM 呼叫流程 -->

- **資料處理流程：**  
  1. 資料收集：蒐集每個 App 安裝用戶的偏好 App 清單（來源、清洗）。  
  2. Prompt 設計與 LLM 呼叫：將行為資料轉換成結構化 prompt，呼叫 OpenAI GPT 模型。  
  3. 生成與後處理：解析 LLM output，做標籤分類（基本屬性、個性、心理狀態、興趣、生活方式）。  
  4. 結果呈現：整合成 persona report 與 hashtag，並用 Flask 做 POC demo 展示。  


<div style="display:grid;grid-template-columns:repeat(2,1fr);gap:1rem;margin:1rem 0;">
  <div>
    <img src="/projects/images/llm_persona_05.png" alt="persona 各欄位解析" style="width:100%;border-radius:6px;">
    <div style="font-size:0.8rem;margin-top:4px;color:#555;">Persona Breakdown</div>
  </div>
  <div>
    <img src="images/llm_persona_06.png" alt="生活型態 hashtags 與人口統計預估" style="width:100%;border-radius:6px;">
    <div style="font-size:0.8rem;margin-top:4px;color:#555;">Hashtags & Demographics</div>
  </div>
  <div>
    <img src="images/llm_persona_07.png" alt="介面範例1" style="width:100%;border-radius:6px;">
    <div style="font-size:0.8rem;margin-top:4px;color:#555;">Interface Example 1</div>
  </div>
  <div>
    <img src="images/llm_persona_08.png" alt="介面範例2" style="width:100%;border-radius:6px;">
    <div style="font-size:0.8rem;margin-top:4px;color:#555;">Interface Example 2</div>
  </div>
</div>


| | |
|---|---|
| ![persona breakdown](/projects/images/llm_persona_05.png) | ![hashtags & demographics](/projects/images/llm_persona_06.png) |
| ![interface 1](/projects/images/llm_persona_07.png) | ![interface 2](/projects/images/llm_persona_08.png) |



## 5. Tech Stack  
- **Languages / Tools:** Python (OpenAI SDK), BigQuery, Flask  
- **LLM / Model:** OpenAI `gpt-3.5-turbo`  
- **Infra / Others:** 可擴展 prompt pipeline，可重用的 prompt template（用於其他應用如 POI 分類、中文分類標籤翻譯）

## 6. Impact  
- 將原本難以解讀的 App 偏好清單，轉換為具象化的 Persona 與族群標籤，提升業務與產品理解速度。  
- 自動生成的 hashtag 與人口統計（年齡/性別）估計，快速提供 segment 的輪廓（可補入：業務採納率提升 X%、洞察產出時間從 Y 降到 Z、節省人工分析 N 小時等具體數字）。  
- 範例視覺化結果：  
  ![Extracted Tags](images/llm_persona_02.png) <!-- alt: 從 persona 中提取的族群標籤與特徵 -->

## 7. Challenges & Solutions  
- **LLM output 不穩定／發散：**  
  解法：設計明確格式的 prompt template，提供範例回答（few-shot style）、固定結構（JSON keys）、調整參數（如 temperature=0 以降低隨機性）。  
- **資訊抽取結構化：**  
  將自由文字 output 進一步解析、驗證並映射到預定 taxonomy（基本屬性、個性、心理、興趣、生活方式）。  

## 8. Lessons Learned / Takeaways  
- **學到的事：** 明確且有範例的 prompt 能顯著提升輸出一致性與可用性。  
- **再做一次會改進：** 把每個偏好 App 的 score / weighting 一併納入 prompt input，提高 persona 精準度。  
- **可重用性：** 這套 pipeline 已套用到其他受眾輪廓分析、如 POI 分類器、中文標籤翻譯器等。  
- **模組化設計：** 建立 reusable text-prompt input pipeline，降低新 case 的上手成本。  
  ![Reusable Pipeline](images/llm_persona_12.png) <!-- alt: 可重用 prompt pipeline 示意 -->

## 9. Demo  
Flask-based proof-of-concept 展示 persona 生成結果與可互動的群體輪廓視覺化。  
![Audience Insight Dashboard](images/llm-insights-dashboard.png) <!-- alt: Flask demo dashboard -->

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

### Prompt Design (整理版)
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

