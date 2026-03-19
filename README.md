# Phishing Email Automation System | 釣魚郵件自動化系統

**An enterprise security awareness phishing simulation system built on n8n workflow engine and Large Language Models (LLM).**

**基於 n8n 工作流引擎與大型語言模型（LLM）的企業資安釣魚演練郵件自動化系統。**

---

## 1. Project Overview | 專案概述

### 1.1 Purpose | 目的
 
This project builds an **automated, diverse, and compliant** phishing simulation email system to raise employee security awareness and strengthen organizational cybersecurity. By simulating realistic phishing scenarios in a controlled environment, employees learn to identify and respond to social engineering attacks—without the repetitive templates and high overhead of manual design. The system reduces management cost while improving drill coverage and realism.

本專案旨在建立一套**自動化、多樣化且合規**的資安釣魚演練郵件系統，用於提升員工資安意識，強化企業整體防護力。透過模擬真實釣魚郵件情境，讓員工在受控環境中學習辨識與應對社交工程攻擊，同時避免傳統人工設計帶來的重複模板與高昂管理成本。系統在不增加管理負擔的前提下，提升演練覆蓋率與真實感。

### 1.2 Key Features | 核心特色


| Feature | Description |
|---------|-------------|
| **Automation** | End-to-end automation of email generation, verification, and delivery. Supports scheduled execution (e.g., monthly or weekly). |
| **Diversity** | LLM-powered content generation ensures different recipients and scenarios receive distinct emails, avoiding repetitive templates. |
| **Compliance & Safety** | Built-in validation prevents requests for passwords, OTP, or PII. Ensures drill emails pose no real security risk. |
| **Traceability** | Full audit trail with campaignId, runId, scenarioId for post-campaign analysis and reporting. |


| 特色 | 說明 |
|------|------|
| **自動化** | 郵件產生、驗證與寄送全程自動化，可排程執行（如每月一次、每週一次）。 |
| **多樣性** | 結合 LLM 生成技術，不同收件人、不同場景產生差異化內容，避免重複模板。 |
| **合規與安全** | 內建驗證機制，禁止要求密碼、OTP、個資，避免真實風險。 |
| **可追蹤性** | 建立完整演練紀錄（campaignId、runId、scenarioId），利於成效分析。 |

### 1.3 Background | 專案背景

Phishing remains one of the most common attack vectors for organizations. This system simulates realistic phishing emails to improve employee detection and reduce susceptibility to fraud, delivering scalable and realistic drills without proportional management overhead.

釣魚郵件仍為企業最常見的攻擊手法之一。本系統透過模擬真實釣魚郵件，提高員工辨識能力，避免落入詐騙陷阱；在不增加管理成本的前提下，提升演練覆蓋率與真實感。

---

## 2. System Architecture | 系統架構

### 2.1 Tech Stack | 技術棧



| Component | Technology |
|-----------|------------|
| Workflow Engine | n8n |
| Language Model | OpenAI GPT-4.1 |
| Email Delivery | SMTP (Email Send node) |
| Program Logic | JavaScript (Code node) |



| 組件 | 技術 |
|------|------|
| 工作流引擎 | n8n |
| 語言模型 | OpenAI GPT-4.1 |
| 郵件發送 | SMTP (Email Send node) |
| 程式邏輯 | JavaScript (Code node) |

### 2.2 Workflow Overview | 工作流程概覽

```
Schedule Trigger → Recipient → Scenario Picker → Basic LLM Chain (OpenAI) → 驗證 → Replace Me → Compose Email → Send email / Mail Sent
```

---

## 3. Node Descriptions | 節點說明

### 3.1 Schedule Trigger | 排程觸發器

 
The Schedule Trigger is the starting point of the workflow. It fires at configured intervals (e.g., monthly or weekly) to initiate the phishing campaign. This periodicity introduces unpredictability, preventing employees from relying on fixed patterns and mimicking the opportunistic nature of real phishing attacks.

 
排程觸發器是整個流程的起點，依預定時間觸發寄信流程。可設定固定時間（例如每月一次、每週一次），讓演練具備週期性與隨機性，避免同仁過度依賴經驗判斷，同時模擬真實駭客攻擊的「不預期」特性。

### 3.2 Recipient | 收件者

 
The Recipient node defines who receives the drill emails and generates campaign identifiers. It supports importing or configuring an employee roster (name, department, email, userId), auto-generates campaignId (e.g., 2025-03) and runId, uses variantSeed to produce different email versions per recipient, and optionally adds time jitter to stagger sending and avoid burst traffic.

 
收件者節點決定誰要收到演練郵件，並生成演練識別資訊。可匯入或設定員工名單（姓名、部門、Email、userId），自動產生 campaignId（如 2025-03）、runId，透過 variantSeed 使相同主題對不同員工產生不同版本，並可為每位收件人加入時間抖動，避免同時大量寄送。

### 3.3 Scenario Picker | 情境分配

 
The Scenario Picker is the core module that assigns a specific drill scenario to each recipient. Predefined scenario types include HR notices, card/account updates, password reset, account alerts, IT renewal, document sharing, security alerts, travel notifications, training notices, and more (24+ categories in the workflow). Output variables include scenarioId, subScenario, tone (neutral/formal/urgent_mild/friendly), style (concise/bullet/step_by_step/paragraph), and trainingLink.


情境分配節點為每位收件人指派演練情境，為情境分配核心模組。預設情境類型包含人資行政通知、卡片資訊更新、密碼重設、帳務異動通知、系統權限續期、文件共享、資安提醒、差旅系統通知、培訓與課程等（共 24+ 大類，詳見 workflow JSON）。輸出變數包括 scenarioId、subScenario、tone、style、trainingLink。

### 3.4 Basic LLM Chain (OpenAI Chat Model) | LLM 生成

 
The LLM Chain generates phishing email content based on scenario and recipient attributes. Generation rules: (1) Scenario-driven—use scenarioId + subScenario with varying tone/style for different versions. (2) No sensitive requests—do not ask for passwords, OTP, credit card numbers, IDs, etc. (3) Single link—use only the provided trainingLink; no other URLs. (4) Length—80–200 characters, natural and professional. (5) Disclaimer—must include “This is a company security drill email; clicking will direct to the training page.” (6) Output format—single JSON: `{ "subject": "...", "bodyText": "...", "bodyHtml": "..." }`.

 
LLM 生成節點依據情境與收件者屬性，自動生成模擬釣魚郵件內容。生成規則：(1) 情境驅動：以 scenarioId + subScenario 為核心，依 tone/style 產生不同版本。(2) 禁止敏感資訊：不得要求密碼、OTP、信用卡號、身分證等。(3) 唯一連結：僅可使用提供的 trainingLink，禁止新增其他 URL。(4) 長度控制：80–200 字，自然專業。(5) 固定尾註：必須包含「此為公司資安演練郵件，點擊後將前往訓練頁面」。(6) 輸出格式：單一 JSON `{ "subject": "...", "bodyText": "...", "bodyHtml": "..." }`。

### 3.5 Verification | 驗證

 
The Verification node validates and sanitizes LLM output before sending. It checks: (1) Required fields—subject, bodyText, bodyHtml must exist and be complete. (2) URL audit—blocks any link other than the authorized trainingLink. (3) Sensitive content filter—if action verbs (provide/reply/fill...) appear near sensitive terms (password/OTP/credit card/ID...) within 12 characters, it auto-replaces with a safe prompt. (4) Disclaimer—ensures the email ends with the mandatory security drill disclaimer.


驗證節點在寄出前檢查並淨化 LLM 輸出。檢查項目：(1) 必要欄位：subject、bodyText、bodyHtml 必須存在且完整。(2) 連結檢查：偵測未授權 URL，若出現非 trainingLink 的連結則阻擋。(3) 敏感資訊過濾：若動詞（提供/回覆/填寫…）與敏感詞（密碼/OTP/信用卡號…）距離過近，自動替換為安全提示。(4) 尾註補強：自動檢查並補上「此為公司資安演練郵件，點擊後將前往訓練頁面」。

### 3.6 Compose Email | 郵件組合

 
The Compose Email node assembles validated content into the final email format. It adds recipient salutation (Hi {name}), inserts the trainingLink as the only clickable link, and appends system labels and disclaimers.

 
郵件組合節點將驗證後的內容組合成最終郵件格式，加上收件者稱謂（Hi {name}），插入 trainingLink 作為唯一可點擊連結，並加上系統標籤與免責聲明。

### 3.7 Send email / Mail Sent | 發送與記錄

 
The Send email node delivers the message via SMTP, configuring sender, recipient, subject, and HTML body. The Mail Sent node logs sending results (subject, scenarioId, sentAt, etc.) for traceability and post-campaign analysis.

 
Send email 節點透過 SMTP 發送郵件，設定寄件人、收件人、主旨與 HTML 內文。Mail Sent 節點記錄寄件結果（subject、scenarioId、sentAt 等），便於後續追蹤與成效分析。

---

## 4. Scenario Examples | 情境範例



| Scenario ID | Scenario Name | Sub-Scenario Examples |
|-------------|---------------|------------------------|
| hr_notice | HR & Admin Notice | Annual performance review, leave balance reminder, payslip download |
| card_update | Card Info Update | Card limit alert, suspicious transaction, card expiry notice |
| pwd_reset | Password Reset | Password expiry,异地登入, new device login verification |
| acct_notice | Account & Billing Notice | Pending charges, invoice supplement, refund update |
| it_renewal | System Permission Renewal | Role expiry, VPN/certificate renewal, project permission adjustment |
| training_notice | Training & Course Notice | Mandatory security training, online quiz deadline |
| ... | (24+ scenario categories) | Multiple sub-scenarios per category |



| 情境 ID | 情境名稱 | 子情境範例 |
|---------|----------|------------|
| hr_notice | 人資行政通知演練 | 年度績效評核結果查詢、年假餘額到期提醒、薪資單下載通知 |
| card_update | 卡片資訊更新演練 | 卡片額度異常待確認、近期可疑交易提醒、卡片即將到期換卡說明 |
| pwd_reset | 密碼重設演練 | 密碼即將到期提醒、偵測到異地登入、新裝置登入驗證 |
| acct_notice | 帳務異動通知演練 | 異常費用待釐清、發票資訊補件、退款處理異動 |
| it_renewal | 系統權限續期演練 | 角色即將到期、VPN/憑證續期、專案權限調整 |
| training_notice | 培訓與課程通知演練 | 強制資安訓練課程開放報名、線上安全測驗即將到期 |
| ... | （共 24+ 大類情境） | 每類含多個子情境 |

---

## 5. Compliance Rules | 合規規範



1. **Links:** Only use the provided trainingLink. No other URLs, shortened links, or rewrites are permitted.
2. **Sensitive data:** Do not request or encourage recipients to provide passwords, OTP, payment data, PII, ID/passport numbers, bank accounts, addresses, phone numbers, or birthdates.
3. **Brands:** Do not impersonate real external brands. Use generic internal system names (e.g., "internal accounting system," "CTCI EIP system").
4. **Disclaimer:** The email body must end with: "此為公司資安演練郵件，點擊後將前往訓練頁面" (This is a company security drill email; clicking will direct to the training page).



1. **連結：** 僅可使用 trainingLink 作為唯一連結，不得新增或改寫其他網址（含縮網址）。
2. **敏感資訊：** 不得要求或引導收件者提供密碼、OTP、支付資料、個資、身分證、銀行帳號、住址、電話、生日等。
3. **品牌：** 不得冒用真實對外品牌，可用泛稱（如「公司內部帳務系統」「中鼎EIP系統」）。
4. **尾註：** 內文尾段必須標註「此為公司資安演練郵件，點擊後將前往訓練頁面」。

---

## 6. Files | 檔案說明



| File | Description |
|------|-------------|
| `釣魚郵件.json` | n8n workflow export. Import into n8n to run the automation. |
| `釣魚信件.pptx.pdf` | Project presentation (architecture, node descriptions, conclusions, future plans). |



| 檔案 | 說明 |
|------|------|
| `釣魚郵件.json` | n8n workflow 匯出檔，可匯入 n8n 直接使用。 |
| `釣魚信件.pptx.pdf` | 專案說明簡報（含架構、節點說明、結論與後續發展）。 |

---

## 7. Setup and Usage | 安裝與使用

### 7.1 Prerequisites | 前置需求

  
You need n8n (self-hosted or n8n Cloud), an OpenAI API key for the LLM node, and SMTP credentials for the email sending node.

 
需要 n8n（可自架或使用 n8n Cloud）、OpenAI API Key（用於 LLM 生成節點）及 SMTP 憑證（用於郵件發送節點）。

### 7.2 Import Workflow | 匯入 Workflow

  
Open n8n, import `釣魚郵件.json`, configure OpenAI Chat Model credentials, configure Send email SMTP credentials, and adjust the Recipient roster (employee list) as needed.

 
開啟 n8n，匯入 `釣魚郵件.json`，設定 OpenAI Chat Model 的 API credentials，設定 Send email 的 SMTP credentials，並依需求調整 Recipient 中的員工名單（roster）。

### 7.3 Customization | 自訂設定


Set `companyName` in workflow variables (default: "本公司"). Optionally set `campaignSalt` for additional scenario randomization. Modify the `base` URL in Scenario Picker for the training link.


在 workflow 變數中設定 `companyName`（預設「本公司」）。可設定 `campaignSalt` 以增加情境隨機性。在 Scenario Picker 中修改 `base` 變數以設定 trainingLink 的基礎網址。

---

## 8. Future Development | 後續發展

 
Planned enhancements: integrate with a central database and visualization platform for real-time monitoring; tiered tracking of phishing susceptibility (open / click / form submission); auto-assignment of training content based on susceptibility level; dashboards for open rate, click rate, and related metrics.


後續發展方向：串接資料庫與視覺化平台，匯入中央資料庫提供即時監控；分級管理上鉤者行為（開信／點擊／輸入資料）；依上鉤嚴重程度自動分派對應資安訓練；建立開信率、點擊率等可視化看板。

