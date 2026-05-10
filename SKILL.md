---
name: practical-english
description: 實用英文學習系統。當使用者說「練英文」「開始練習」「英文練習」「下一個練習」「practical english」時啟動。生成 15 分鐘互動式 HTML 練習頁（會話＋聽解＋口說三個 Tab），含 edge-tts 男女聲音檔。也處理練習中的補充請求（「PE-XXX 會話 第N句的___是什麼意思」）。即使使用者只是提到想練英文，也應主動觸發此 skill。
---

# Practical English — 實用英文學習 Skill

## 目標

即時聽懂並表達，應用在工作與生活實用情境。每次 15 分鐘，一個 HTML 頁面跑完三個 Tab = 完成一次學習。

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
    start http://localhost:7788
    python -m http.server 7788 --directory "使用者路徑"
    ```
  - **Mac** → `啟動練習.sh`（建立後執行 `chmod +x 啟動練習.sh`）：
    ```bash
    #!/bin/bash
    open http://localhost:7788
    python3 -m http.server 7788 --directory "使用者路徑"
    ```
  **重要**：必須用 localhost 開啟，直接雙擊 html 以 `file://` 開啟時 Chrome 每次都會重問麥克風權限。
- `index.html`：練習首頁，含折疊式輪次清單。每次生成新練習後必須更新此檔，在對應輪次的 `round-body` 內新增 `lesson-card`，並更新 `updateRoundProgress` 的 peIds 陣列。新輪次開始時新增對應的 `round-section` HTML 區塊（預設收合：header 不加 `open` class，body 不加 `open` class）。

### 4. 顯示使用說明

輸出以下教學引導（完整內容見 `references/onboarding.md`）：

1. **學習內容**：12 個情境主題循環輪替（W1-W6 工作 + P1-P4 實用 + T1-T2 時事）
2. **三個練習面向**：會話（短對話 5min）、聽解（長段理解 5min）、口說（自己開口 + AI 回饋 5min）
3. **遇到不懂的**：告訴 Claude「PE-001 會話 第 3 句的 _____ 是什麼意思」→ Claude 直接寫進 HTML
4. **星號收藏**：按 ⭐ 收藏句型 → 在「複習與學習成果檢視」頁統一查看
5. **注意事項**：口說 Tab 需要 OpenAI API key（第一次開網頁時輸入）+ 麥克風權限（macOS：系統設定 → 隱私權 → 麥克風 → 允許 Chrome；Windows：設定 → 隱私權 → 麥克風）

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

### 2. 設計內容

根據主題類別，設計三個 Tab 的內容（詳見 `references/tab-specs.md`）：

**會話 Tab**：
- 寫 6-8 句來回對話（約 60-90 秒）
- 設計 3 題理解選擇題
- 準備對話原文 + 中文翻譯 + 4-6 個關鍵句型

**聽解 Tab**：
- 寫 150-250 字英文獨白，分 3-5 段（約 90-120 秒）
- 設計 4 題選擇題（主旨 / 細節 / 推論 / 觀點）
- 準備原文 + 翻譯 + 4-6 個關鍵用語

**口說 Tab**：
- 設計 4 輪對話情境（對方說一句 → 使用者回應）
- 每輪準備中文情境提示

### 3. 生成音檔

用 edge-tts 生成所有 mp3 音檔。Python script 範例：

```python
import asyncio
import edge_tts

MALE = "en-US-GuyNeural"    # 男聲（You / 獨白）
FEMALE = "en-US-JennyNeural" # 女聲（Client / 對方）

async def generate(text, voice, path):
    comm = edge_tts.Communicate(text, voice)
    await comm.save(path)
```

音檔清單：
- `conv-1.mp3` ~ `conv-N.mp3`：對話逐句（男女交替）
- `listen-full.mp3`：聽解全段
- `listen-p1.mp3` ~ `listen-pN.mp3`：聽解分段
- `speak-prompt-1.mp3` ~ `speak-prompt-4.mp3`：口說提示句

`listen-full.mp3` 用 ffmpeg 串接分段：
```bash
ffmpeg -y -f concat -safe 0 -i concat.txt -c copy listen-full.mp3
```

### 4. 生成 HTML

根據 `references/html-template-spec.md` 的完整規格生成 HTML 頁面。

**關鍵規格**：
- 設計系統：讀取 `references/design-system.md` 的 CSS 變數與按鈕樣式
- 三個 Tab 切換
- 每個 Tab 頂部有完成狀態（☆ 未完成 / ⭐ 已完成）+ 重來按鈕
- 送出前鎖定原文，送出後解鎖
- Web Speech API 語音辨識（`lang = "en-US"`，`continuous = true`）
- OpenAI API (gpt-4o-mini) 口說回饋
- ⭐ 星號收藏存 localStorage `pe_stars`
- 進度存 localStorage `pe_progress`（每個 Tab 獨立記錄）
- 開始 🎤 和停止 ⏹ 是兩個獨立按鈕

### 5. 建立資料夾

```
PE-XXX/
├── PE-XXX [類別] [主題].html
└── audio/
    ├── conv-1.mp3 ... conv-N.mp3
    ├── listen-full.mp3
    ├── listen-p1.mp3 ... listen-pN.mp3
    └── speak-prompt-1.mp3 ... speak-prompt-4.mp3
```

### 6. 更新進度追蹤

在 `進度追蹤.md` 記錄新練習的編號、類別、主題、生成日期。

---

## 補充機制

使用者提問格式：
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

## 目前進度
- 下一個練習：PE-002
- 目前輪次：第 1 輪

## 練習記錄

| 編號 | 類別 | 主題 | 生成日期 |
|------|------|------|---------|
| PE-001 | W1 | 簡報開場 | 2026-05-10 |
```

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
| `references/tab-specs.md` | 三個 Tab 的詳細設計規格 | 生成內容時 |
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
