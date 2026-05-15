---
name: practical-english
description: 實用英文學習系統。當使用者說「練英文」「開始練習」「英文練習」「下一個練習」「practical english」時啟動。使用者自選要包含的類型（📖閱讀/💬會話/🎧聽解/🎤口說），預設只有閱讀（零壓力入口），完成後詢問是否升級。生成互動式 HTML 練習頁，含 edge-tts 男女聲音檔。閱讀含文章+對話兩個子區塊，邊看邊聽。也處理練習中的補充請求（「PE-XXX 閱讀 第N句的___是什麼意思」）。即使使用者只是提到想練英文，也應主動觸發此 skill。
---

# Practical English — 實用英文學習 Skill

## 目標

即時聽懂並表達，應用在工作與生活實用情境。使用者自己決定每次練習包含哪些類型——預設只有「閱讀」（零壓力入口），完成後詢問是否加入更多挑戰。

---

## 首次執行

第一次執行時，依序處理以下步驟：

### 1. 環境檢查

```bash
python3 -c "import edge_tts" 2>/dev/null && echo "OK" || echo "NEED_INSTALL"
```

未安裝 → 提示使用者執行 `pip install edge-tts`。

### 2. 設定檔案路徑

詢問使用者要把練習檔放在哪裡（預設 `~/Documents/practical-english/`），寫進 `進度追蹤.md`。

### 3. 建立固定檔案

- `進度追蹤.md`：記錄路徑設定與輪替進度
- `複習與學習成果檢視.html`：從 `references/review-page.html` 複製
- 啟動本地伺服器的捷徑檔（路徑換成使用者設定的路徑）：
  - **Windows** → `啟動練習.bat`：
    ```bat
    @echo off
    echo 啟動 Practical English 本地伺服器...
    start http://localhost:9170
    python -m http.server 9170 --directory "使用者路徑"
    ```
  - **Mac** → `啟動練習.sh`（建立後執行 `chmod +x 啟動練習.sh`）：
    ```bash
    #!/bin/bash
    lsof -ti:9170 | xargs kill -9 2>/dev/null
    sleep 0.5
    open http://localhost:9170
    python3 -m http.server 9170 --directory "使用者路徑"
    ```
  **重要**：必須用 localhost 開啟，直接雙擊 html 以 `file://` 開啟時 Chrome 每次都會重問麥克風權限。
- `index.html`：練習首頁。**版本控制機制**：
  
  **當前版本：3**（v3：移除星星系統、純靜態首頁、無 JavaScript）
  
  **每次生成新練習前，先檢查 index.html 版本**：
  1. 讀 index.html 第二行，找 `<!-- pe-index-version: N -->`
  2. 如果**沒有版本號**或**版本 < 3**：整頁重建 + 舊課程升級
  3. 如果**版本 = 3**（最新）：只做增量更新

  **增量更新步驟**：
  1. 在練習列表區新增 `lesson-card`（含課程連結、類型 emoji）

  **整頁重建步驟**（版本過舊時）：
  1. 複製 `references/index-template.html` 作為基底
  2. 用 `ls` 掃描資料夾內所有 `PE-XXX/` 子目錄
  3. 讀每個 `PE-XXX/*.html` 的 tab-bar，判斷該課包含哪些類型
  4. 把掃描結果生成 lesson cards，插入模板的 `<!-- {{LESSON_CARDS}} -->` 位置
  5. 覆蓋舊的 index.html
  6. 同時複製 `references/review-template.html` 覆蓋 `review.html`
  7. **同時執行舊課程升級**（見下方）

  **舊課程自動升級**（版本重建時一併執行）：
  掃描所有 `PE-XXX/*.html`，對每個檔案檢查並修補：
  1. **速度選單**：如果 `<select>` 裡沒有 `0.25` 選項，在每個 speed-select 的 `<option value="0.7">` 前面插入 `<option value="0.25">0.25x</option>` 和 `<option value="0.5">0.5x</option>`
  2. **會話 Tab 速度控制**：如果會話 Tab 的播放按鈕旁沒有 speed-select，加上 conv-speed 選單 + `getConvSpeed()` / `updateConvSpeed()` 函式
  3. **口說 Tab 速度控制**：如果口說 Tab 沒有 speak-speed 選單，加上 + `getSpeakSpeed()` 函式
  4. **口說 Tab 對方台詞**：如果 speak-round 裡沒有 `.speak-transcript`，從 `speakPrompts` 陣列讀取對應文字，插入顯示區塊
  5. **口說進度持久化**：如果沒有 `saveSpeakState` 函式，加上 localStorage 存取邏輯
  6. **Tab 記憶**：如果 `switchTab` 函式裡沒有 `localStorage.setItem`，加上 Tab 切換記憶
  7. **GPT prompt**：如果 system prompt 裡沒有 `IMPORTANT: "text" must always be in English`，更新 prompt
  8. **啟動檔 port 統一**：檢查練習資料夾內的 `啟動練習.bat`（Windows）或 `啟動練習.sh`（Mac），如果裡面的 port 不是 `9170`，全部替換成 `9170`

### 4. 顯示使用說明

輸出以下教學引導（完整內容見 `references/onboarding.md`）：

1. **學習內容**：12 個情境主題循環輪替（W1-W6 工作 + P1-P4 實用 + T1-T2 時事）
2. **自選類型**：每次練習你選要包含哪些，預設只有「閱讀」
3. **四種類型**：
   - 📖 **閱讀**（預設）：文章＋對話，邊看邊聽，零壓力
   - 💬 **會話**：短對話 + 理解題，5min
   - 🎧 **聽解**：只聽不看 + 理解題，5min
   - 🎤 **口說**：自己開口 + AI 回饋，5min
4. **遇到不懂的**：告訴 Claude「PE-001 閱讀 第 3 句的 _____ 是什麼意思」→ Claude 直接寫進 HTML
5. **星號收藏**：按 ⭐ 收藏句型 → 在「複習與學習成果檢視」頁統一查看
6. **注意事項**：口說需要 OpenAI API key（第一次開網頁時輸入）+ 麥克風權限（macOS：系統設定 → 隱私權 → 麥克風 → 允許 Chrome；Windows：設定 → 隱私權 → 麥克風）

### 5. 開始

「要開始第一個練習嗎？」

---

## 每次生成練習的流程

### 1. 判斷下一個練習

讀取 `進度追蹤.md`，確認目前進度，決定下一個練習的編號與主題。

12 個類別輪替順序：
```
W1 簡報 → W2 會議 → W3 接待 → W4 電話 → T1 產業時事 →
W5 談判 → W6 展會 → P1 出差 → P2 社交 → T2 AI時事 →
P3 生活 → P4 資訊吸收 → (第二輪 W1，不同情境...)
```

每輪同類別但不同情境，永遠不重複。

### 2. 詢問使用者要包含哪些類型

讀取 `進度追蹤.md` 中的「偏好類型」設定。如果沒有記錄過偏好，預設只有「閱讀」。

顯示類型選單：
```
下一課：PE-XXX [類別] [主題]

這次要包含哪些類型？
  📖 閱讀（文章＋對話，邊看邊聽） ← 預設 ✅
  💬 會話（短對話 + 理解題）
  🎧 聽解（只聽不看 + 理解題）
  🎤 口說（自己開口 + AI 回饋）

直接按 Enter = 用上次的選擇（📖 閱讀）
或告訴我你想加入/移除哪些。
```

使用者可以說：
- 「就閱讀就好」→ 只生成閱讀 Tab
- 「加會話」→ 閱讀 + 會話
- 「全部」→ 四種都生成
- 「跟上次一樣」或直接說「開始」→ 沿用上次偏好
- 「只要聽解跟口說」→ 只生成聽解 + 口說（進階使用者可以跳過閱讀）

選擇結果寫入 `進度追蹤.md` 的「偏好類型」欄位。

### 3. 設計內容

根據使用者選擇的類型，**只設計被選中的類型**（詳見 `references/tab-specs.md`）：

**📖 閱讀（包含兩個子區塊）**：

- **文章閱讀**：150-250 字英文獨白（同聽解的內容源），分 3-5 段
  - 英文全文 + 中文翻譯 + 關鍵詞彙標色
  - 整段播放 / 逐段播放 / 速度控制
  - 播放中段落自動高亮
  - ⭐ 收藏句型

- **對話閱讀**：6-8 句來回對話（同會話的內容源）
  - 英文全文 + 中文翻譯 + 關鍵句型標色
  - 逐句播放 / 整段播放
  - 角色標示（A / B）+ 頭像顏色區分
  - ⭐ 收藏句型

兩個子區塊都是**文字全開、不考試、不鎖定**。

**💬 會話**（需要閱讀的對話內容作為基礎）：
- 同一段對話，但隱藏文字、只有播放鍵
- 設計 3 題理解選擇題
- 送出後解鎖對話原文 + 翻譯 + 句型

**🎧 聽解**（需要閱讀的文章內容作為基礎）：
- 同一篇文章，但隱藏文字、只能聽
- 設計 4 題選擇題（主旨 / 細節 / 推論 / 觀點）
- 送出後解鎖原文 + 翻譯 + 句型

**🎤 口說**：
- 設計 4 輪對話情境（對方說一句 → 使用者回應）
- 每輪準備中文情境提示

**內容共用規則**：閱讀的文章 = 聽解的文章、閱讀的對話 = 會話的對話。如果使用者只選閱讀，仍然要生成文章和對話的完整內容與音檔，因為閱讀 Tab 兩個子區塊都需要。如果使用者加選會話或聽解，它們直接複用閱讀的內容，只是多了考試互動。

### 4. 生成音檔

用 edge-tts 生成**被選中類型需要的**音檔。Python script 範例：

```python
import asyncio
import edge_tts

MALE = "en-US-GuyNeural"    # 男聲（You / 獨白）
FEMALE = "en-US-JennyNeural" # 女聲（Client / 對方）

async def generate(text, voice, path):
    comm = edge_tts.Communicate(text, voice)
    await comm.save(path)
```

音檔清單（按類型）：

**📖 閱讀（永遠生成）**：
- `conv-1.mp3` ~ `conv-N.mp3`：對話逐句（男女交替）
- `conv-full.mp3`：對話整段（ffmpeg 串接）
- `listen-full.mp3`：文章全段
- `listen-p1.mp3` ~ `listen-pN.mp3`：文章分段

**💬 會話**：不需額外音檔（複用閱讀的 conv 音檔）
**🎧 聽解**：不需額外音檔（複用閱讀的 listen 音檔）
**🎤 口說**：`speak-prompt-1.mp3` ~ `speak-prompt-4.mp3`

`listen-full.mp3` 和 `conv-full.mp3` 用 ffmpeg 串接分段：
```bash
ffmpeg -y -f concat -safe 0 -i concat.txt -c copy listen-full.mp3
```

### 5. 生成 HTML

根據 `references/html-template-spec.md` 的完整規格生成 HTML 頁面。

**只生成使用者選擇的 Tab**。如果只選閱讀，HTML 中只有一個 Tab（不需要 Tab Bar）。

**關鍵規格**：
- 設計系統：讀取 `references/design-system.md` 的 CSS 變數與按鈕樣式
- 被選中的類型才生成對應 Tab
- 只有一個 Tab 時隱藏 Tab Bar（直接顯示內容）
- 閱讀 Tab 內用分隔線區分「文章閱讀」和「對話閱讀」兩個子區塊
- 會話/聽解 Tab：送出前鎖定原文，送出後解鎖
- 會話/聽解 Tab 頂部有完成狀態（☆ 未完成 / ⭐ 已完成）+ 重來按鈕
- 閱讀 Tab 不顯示完成狀態（零壓力）
- 口說 Tab：Web Speech API 語音辨識 + OpenAI API 回饋
- ⭐ 星號收藏存 localStorage `pe_stars`
- 進度存 localStorage `pe_progress`（每個 Tab 獨立記錄）
- 開始 🎤 和停止 ⏹ 是兩個獨立按鈕

### 6. 建立資料夾

```
PE-XXX/
├── PE-XXX [類別] [主題].html
└── audio/
    ├── conv-1.mp3 ... conv-N.mp3     ← 永遠生成
    ├── conv-full.mp3                  ← 永遠生成
    ├── listen-full.mp3                ← 永遠生成
    ├── listen-p1.mp3 ... listen-pN.mp3 ← 永遠生成
    └── speak-prompt-1.mp3 ... speak-prompt-4.mp3 ← 只在選口說時生成
```

### 7. 更新進度追蹤

在 `進度追蹤.md` 記錄新練習的編號、類別、主題、包含類型、生成日期。

### 8. 完成後詢問升級

練習頁生成完畢後，顯示：

```
✅ PE-XXX 已生成！

這次包含：📖 閱讀
完成後如果覺得想要更多挑戰，可以告訴我：
  → 「加會話」：下次加入對話理解題
  → 「加聽解」：下次加入純聽力測驗
  → 「加口說」：下次加入口說練習
  → 「全部」：下次四種都來

也可以隨時說「只要閱讀」回到輕鬆模式。
```

這段提示根據使用者目前的選擇動態調整——只顯示還沒選的類型。如果已經是全部，就不顯示升級提示。

---

## 補充機制

使用者提問格式：
- 「PE-001 **閱讀 文章** 第 2 段的 _____ 是什麼意思」
- 「PE-001 **閱讀 對話** 第 3 句的 _____ 是什麼意思」
- 「PE-001 **會話** 第 3 句的 _____ 是什麼意思」
- 「PE-001 **聽解** 第 2 段的 _____ 不懂」
- 「PE-001 **口說** 第 1 題」

收到後，用 Edit 工具在對應 HTML 的該 Tab 底部新增：

**補充單字**：
```html
<div class="supplement" style="background:var(--purple-light);border-left:4px solid var(--accent-2);border-radius:12px;padding:16px;margin-top:12px;">
  <h3>補充單字</h3>
  <p><strong>walk you through</strong> /wɔːk juː θruː/</p>
  <p>帶你了解、逐步說明</p>
  <p style="color:var(--text-dim);font-size:14px;">例：Let me walk you through the new features.</p>
</div>
```

**補充文法**：
```html
<div class="supplement" style="background:var(--purple-light);border-left:4px solid var(--accent-2);border-radius:12px;padding:16px;margin-top:12px;">
  <h3>補充文法</h3>
  <p><strong>Feel free to + 原形動詞</strong></p>
  <p>表示「請隨意……」，正式但友善的邀請語氣</p>
  <p style="color:var(--text-dim);font-size:14px;">✅ Feel free to ask any questions.</p>
  <p style="color:var(--text-dim);font-size:14px;">❌ Feel free asking questions.（不可接 V-ing）</p>
</div>
```

累積式新增，不覆蓋之前的補充。

---

## 進度追蹤檔格式

```markdown
# 實用英文學習 — 進度追蹤

## 設定
- 檔案路徑：~/Documents/practical-english/
- 偏好類型：📖 閱讀

## 目前進度
- 下一個練習：PE-002
- 目前輪次：第 1 輪

## 練習記錄

| 編號 | 類別 | 主題 | 包含類型 | 生成日期 |
|------|------|------|---------|---------|
| PE-001 | W1 | 簡報開場 | 📖 | 2026-05-10 |
```

「偏好類型」記錄使用者上次的選擇，下次生成時作為預設值。格式範例：
- `📖 閱讀`（預設）
- `📖 閱讀 + 💬 會話`
- `📖 閱讀 + 💬 會話 + 🎧 聽解 + 🎤 口說`（全部）
- `🎧 聽解 + 🎤 口說`（進階使用者可以跳過閱讀）

---

## 時事包規則

每 5 包中 1 包是時事（T1 或 T2），用當週真實新聞編成對話。

時事範圍：
- T1：半導體 + PCB + AI 硬體供應鏈
- T2：AI 服務與生活應用

生成時事包時，先用 WebSearch 搜尋當週相關新聞，再編成練習內容。

---

## 參考文件

| 文件 | 內容 | 何時讀取 |
|------|------|---------|
| `references/design-system.md` | CSS 變數、按鈕樣式、色彩規範 | 生成 HTML 時 |
| `references/tab-specs.md` | 四種類型的詳細設計規格 | 生成內容時 |
| `references/html-template-spec.md` | HTML 完整結構與 JavaScript 規格 | 生成 HTML 時 |
| `references/onboarding.md` | 首次安裝引導文字 | 首次執行時 |
| `references/review-page.html` | 複習與學習成果檢視頁完整 HTML | 首次執行時複製 |

---

## 注意事項

- 對話中 Client 用女聲（JennyNeural）、You 用男聲（GuyNeural）
- 聽解獨白用男聲（GuyNeural）
- 口說提示句用女聲（JennyNeural，模擬對方）
- 口說替代說法的播放用 Web Speech API SpeechSynthesis（動態生成，不預錄）
- 所有 localStorage key：`pe_stars`（收藏）、`pe_progress`（進度）、`openai_api_key`（API key）
