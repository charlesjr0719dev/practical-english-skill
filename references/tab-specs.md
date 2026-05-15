# 實用英文學習 — 練習分類

---

## 四種類型（使用者自選）

| 類型 | 名稱 | 內容 | 互動方式 | 時間 |
|------|------|------|---------|------|
| 📖 | 閱讀 | 文章＋對話兩個子區塊，文字全開 | 邊看邊聽，零壓力沈浸 | 自由 |
| 💬 | 會話 | 跟閱讀同一段對話，但隱藏文字 | 先聽再答題，練即時理解 | 5 min |
| 🎧 | 聽解 | 跟閱讀同一篇文章，但隱藏文字 | 先聽再答題，練長段理解 | 5 min |
| 🎤 | 口說 | 獨立情境，自己組句子講出來 | 看中文情境，自己講，AI 回饋 | 5 min |

**預設只有 📖 閱讀**。使用者完成後可選擇加入更多類型。

---

## 對應需求

| 類型 | 對應能力 | 實際場景 |
|------|---------|---------|
| 📖 閱讀 | 建立信心 + 熟悉詞彙 | 邊看邊聽，不怕漏聽、不怕看不懂，先把內容吃進去 |
| 💬 會話 | 即時聽懂 + 即時回應 | 接待客戶、開會、電話、餐敘 |
| 🎧 聽解 | 長段理解 + 抓重點 | 直接吸收英文 podcast / YouTube / 產業報導 |
| 🎤 口說 | 從中文想法到英文表達 | 簡報、說明、社交閒聊 |

**學習路徑**：閱讀（先看懂）→ 會話/聽解（測試理解）→ 口說（自己表達）。使用者自己決定什麼時候升級。

---

## 句型與詞彙

不獨立成 tab，嵌在每個 tab 練完後：

- **閱讀 tab**：關鍵詞彙直接標色在文中，底部附完整句型收穫
- **會話 tab**：練完顯示該對話的關鍵句型 + 替換範例
- **聽解 tab**：練完顯示關鍵用語 + 長句拆解
- **口說 tab**：Web Speech API 語音辨識 → OpenAI API 判別回饋（情境是否恰當、語法、更自然的說法）→ 顯示參考表達

---

## 📖 閱讀 細部設計

### 設計理念

閱讀是**進入門檻最低**的學習方式——文字全開、音檔同步、不考試、不鎖定。適合：
- 覺得其他類型太難的學習者（先在閱讀熟悉內容，再挑戰聽解/會話）
- 想用「邊看邊聽」方式沈浸式學習的人
- 複習用（練完其他類型回來重讀）

### 兩個子區塊

閱讀 Tab 內含**兩個子區塊**，用分隔線隔開：

| 子區塊 | 內容來源 | 說明 |
|--------|---------|------|
| 📄 文章閱讀 | 與聽解共用同一篇文章 | 獨白/報導/演講，150-250 字，分 3-5 段 |
| 💬 對話閱讀 | 與會話共用同一段對話 | 兩人來回 6-8 句，含角色標示 |

兩個子區塊都是**文字全開、不考試、不鎖定**。

### 頁面結構

| 區塊 | 內容 | 初始狀態 |
|------|------|---------|
| 情境描述 | 說明場景主題 | 可見 |
| **📄 文章閱讀** | | |
| 小標題 | 「📄 Article」 | 可見 |
| 播放控制 | ▶ 播放整段 + 速度滑桿（0.7x / 1.0x / 1.2x） | 可見 |
| 文章區 | 英文全文，逐段排列，每段有 ▶ 播放鍵 + 關鍵詞彙標色 | ✅ 全部顯示 |
| 中文翻譯 | 每段英文下方附中文翻譯（淺灰色、較小字） | ✅ 全部顯示 |
| 句型收穫 | 4-6 個關鍵用語 + 替換範例 | ✅ 全部顯示 |
| **分隔線** | `<hr>` | |
| **💬 對話閱讀** | | |
| 小標題 | 「💬 Conversation」 | 可見 |
| 播放控制 | ▶ 播放整段對話 | 可見 |
| 對話區 | 每句一行：角色標示 + ▶ 播放鍵 + 英文文字 + 中文翻譯 | ✅ 全部顯示 |
| 句型收穫 | 4-6 個關鍵句型 + 替換範例 | ✅ 全部顯示 |
| **共用底部** | | |
| 功能提示 | 💡「有不懂的？告訴 Claude：PE-XXX 閱讀 文章/對話 第 N 段/句的 _____ 是什麼意思」 | 可見 |
| 補充單字 | Claude 事後補充寫入 | 初始為空 |
| 補充文法 | Claude 事後補充寫入 | 初始為空 |

### 互動流程

1. 讀情境描述，知道主題
2. **文章閱讀**：直接看到完整英文 + 中文翻譯 → 按 ▶ 邊看邊聽（可調速）→ 逐段重聽
3. **對話閱讀**：直接看到完整對話 → 按 ▶ 聽整段或逐句聽 → 看角色標示理解對話脈絡
4. 關鍵詞彙已標色，看到就學
5. 喜歡的句型按 ⭐ 收藏
6. 有不懂的 → 告訴 Claude → Claude 補充寫入 HTML

### 播放中段落高亮

播放整段時，**當前播放的段落/對話句自動加底色高亮**（淺藍底 `#e8f4fd`），幫助追蹤進度。逐段/逐句播放時也高亮對應區塊。播放結束後移除高亮。

### 完成條件

閱讀 **不記錄完成狀態**（不計入星星地圖）。它是輔助工具，不是考試。不顯示完成狀態 badge 和重來按鈕。

### 文章閱讀 HTML 結構

```html
<h3>📄 Article</h3>
<div class="reading-controls">
  <button class="play-btn-large" onclick="playReadingFull()">▶ 播放整段</button>
  <select id="reading-speed" onchange="updateReadingSpeed()">
    <option value="0.7">0.7x</option>
    <option value="1.0" selected>1.0x</option>
    <option value="1.2">1.2x</option>
  </select>
</div>
<div class="reading-section">
  <div class="reading-para" id="read-p1" data-start="0" data-end="15">
    <div class="para-header">
      <button class="play-btn" onclick="playReadingPara(1)">▶</button>
      <span class="para-label">Paragraph 1</span>
    </div>
    <p class="reading-en">English text with <span class="highlight">key phrases</span> highlighted...</p>
    <p class="reading-zh">中文翻譯...</p>
  </div>
</div>
```

### 對話閱讀 HTML 結構

```html
<hr style="margin: 32px 0; border: none; border-top: 2px solid var(--border);">
<h3>💬 Conversation</h3>
<div class="reading-controls">
  <button class="play-btn-large" onclick="playConvFull()">▶ 播放整段對話</button>
</div>
<div class="reading-section">
  <div class="reading-conv-line" id="read-conv-1">
    <div class="para-header">
      <button class="play-btn" onclick="playConvLine(1)">▶</button>
      <span class="conv-role role-a">A</span>
    </div>
    <p class="reading-en">English dialogue line with <span class="highlight">key phrases</span>...</p>
    <p class="reading-zh">中文翻譯...</p>
  </div>
</div>
```

### 額外 CSS

```css
.reading-section { margin-top: 16px; }
.reading-para, .reading-conv-line { padding: 16px; border-radius: 12px; margin-bottom: 12px; border: 2px solid var(--border); transition: background 0.3s, border-color 0.3s; }
.reading-para.playing, .reading-conv-line.playing { background: #e8f4fd; border-color: var(--accent); }
.para-header { display: flex; align-items: center; gap: 10px; margin-bottom: 8px; }
.para-label { font-size: 13px; color: var(--text-dim); font-weight: 700; }
.reading-en { font-size: 17px; line-height: 1.8; margin-bottom: 6px; }
.reading-zh { font-size: 15px; color: var(--text-dim); line-height: 1.6; }
.reading-controls { display: flex; align-items: center; gap: 12px; margin-bottom: 16px; }
.role-a { background: var(--accent); color: white; padding: 2px 10px; border-radius: 999px; font-size: 13px; font-weight: 700; }
.role-b { background: var(--success); color: white; padding: 2px 10px; border-radius: 999px; font-size: 13px; font-weight: 700; }
```

---

## 會話 Tab 細部設計

### 頁面結構

| 區塊 | 內容 | 初始狀態 |
|------|------|---------|
| 情境描述 | 一句話說明場景（例：「你在 SEMICON 展會攤位，一位美國客戶走過來詢問你們的設備」） | 可見 |
| 對話區 | 每句一行，只有 ▶ 播放鍵 + 角色標示（A / B），沒有文字 | 只有播放鍵 |
| 整段播放 | ▶ 從頭到尾播放整段對話 | 可見 |
| 理解題 | 3 題選擇題 | 可見 |
| 送出按鈕 | 送出後顯示答對/答錯 + 解析 | 按鈕 |
| 對話原文 | 英文全文 + 中文翻譯 + 關鍵句型標色 | 🔒 送出後解鎖 |
| 句型收穫 | 4-6 個關鍵句型 + 替換範例 | 🔒 送出後解鎖 |
| 功能提示 | 💡「有不懂的單字或文法？告訴 Claude：PE-XXX 會話 第 N 句的 _____ 是什麼意思」 | 可見 |
| 補充單字 | Claude 事後補充寫入 | 初始為空 |
| 補充文法 | Claude 事後補充寫入 | 初始為空 |

### 互動流程

1. 讀情境描述，知道場景
2. 按 ▶ 聽整段對話（或逐句聽）
3. 作答 3 題理解題
4. 送出 → 解鎖原文 + 翻譯 + 句型
5. 有不懂的 → 告訴 Claude → Claude 補充寫入 HTML

### 對話規格

- 6-8 個來回，每句 1-2 句話
- 總長約 60-90 秒
- edge-tts 美式（男聲 `en-US-GuyNeural` + 女聲 `en-US-JennyNeural`）

---

## 聽解 Tab 細部設計

### 頁面結構

| 區塊 | 內容 | 初始狀態 |
|------|------|---------|
| 情境描述 | 一句話說明這段內容的類型與主題（例：「一段 Podcast 節目在討論 AI 晶片出口管制對台灣供應鏈的影響」） | 可見 |
| 播放控制 | ▶ 播放整段 + 速度滑桿 | 可見 |
| 理解題 | 4 題選擇題（每題 4 個選項） | 可見 |
| 送出按鈕 | 送出後顯示答對/答錯 + 解析 | 按鈕 |
| 原文 | 英文全文（關鍵用語標色）+ 逐段 ▶ 可重聽 | 🔒 送出後解鎖 |
| 中文翻譯 | 全文翻譯 | 🔒 送出後解鎖 |
| 句型收穫 | 4-6 個關鍵用語 + 長句拆解 + 替換範例 | 🔒 送出後解鎖 |
| 功能提示 | 💡「有不懂的單字或文法？告訴 Claude：PE-XXX 聽解 第 N 段的 _____ 是什麼意思」 | 🔒 送出後解鎖 |
| 補充單字 | Claude 事後補充寫入 | 初始為空 |
| 補充文法 | Claude 事後補充寫入 | 初始為空 |

### 互動流程

1. 讀情境描述，知道要聽什麼類型的內容
2. 按 ▶ 播放整段（可重聽、可調速）
3. 作答 4 題理解題
4. 送出 → 解鎖原文 + 翻譯 + 句型 + 功能提示
5. 可逐段重聽對照原文
6. 有不懂的 → 告訴 Claude → Claude 補充寫入 HTML

### 內容規格

- 單人獨白，模擬 podcast / YouTube 講解 / 新聞報導
- 長度約 150-250 字英文，約 90-120 秒
- edge-tts 美式（男聲 `en-US-GuyNeural` + 女聲 `en-US-JennyNeural`）
- 分 3-5 段，每段可獨立重聽

### 題型設計

- 主旨題：這段內容主要在講什麼
- 細節題：講者提到了哪個具體事實
- 推論題：根據內容可以推測什麼
- 觀點題：講者對某件事的態度是什麼

---

## 口說 Tab 細部設計

### 互動流程（4 輪對話）

| 步驟 | 畫面 | 使用者動作 |
|------|------|----------|
| 1 | 情境描述 + 對方第一句話 ▶ | 聽對方說的 |
| 2 | 🎤 錄音按鈕出現 | 按 🎤 說英文回應 |
| 3 | 顯示語音辨識結果（你說的文字） | 確認後送出 |
| 4 | OpenAI API 即時回饋（語法、自然度） | 閱讀回饋 |
| 5 | 對方第二句話出現 ▶ | 繼續下一輪 |
| ... | 重複 3-4 輪 | |

### 結果頁（4 輪結束後自動生成）

每一輪顯示：

| 區塊 | 內容 |
|------|------|
| 對方說的 | 原句英文 + ▶ 播放 |
| 你的回答 | 語音辨識文字 |
| AI 回饋 | 語法/自然度評語 |
| 替代說法 | 3-4 種不同正式程度的說法，每句附 ▶ 播放 + 中文語意 |

### 替代說法格式範例

```
你說的：I think the delivery will be late.

✅ 語法正確，但偏口語。商務場合可以更正式：

1.【正式】I'm afraid there may be a delay in delivery. ▶
2.【委婉】We might need to adjust the timeline slightly. ▶
3.【直接】We're looking at a two-week delay on this order. ▶
4.【口語】Looks like it's gonna take a bit longer than expected. ▶
```

### 技術規格

- 語音辨識：Web Speech API `SpeechRecognition`，`lang = "en-US"`
- 回饋判別：OpenAI API（GPT），每輪一次 API call，同時回傳回饋 + 替代說法
- 替代說法音檔：Web Speech API SpeechSynthesis（動態生成，無需預錄），每句皆有 ▶ 播放鍵
- 功能提示：結果頁底部顯示 💡 提示（同補充機制）

### 頁面結構

| 區塊 | 初始狀態 |
|------|---------|
| 情境描述 | 可見 |
| 對話區（逐輪展開） | 逐輪出現 |
| 結果頁（全部輪次總覽） | 🔒 全部輪次完成後生成 |
| 功能提示 | 🔒 結果頁生成後顯示 |
| 補充單字 | 初始為空 |
| 補充文法 | 初始為空 |

---

## 複習與學習成果檢視頁

獨立的 `複習與學習成果檢視.html`，固定放在資料夾中，不需每次生成。

### 兩個 Tab

| Tab | 名稱 | 內容 |
|-----|------|------|
| 1 | 學習成果 | 星星地圖 |
| 2 | 收藏複習 | 所有 ⭐ 過的句型列表 |

### 星星地圖

- 橫軸：12 個主題（W1-W6 + P1-P4 + T1-T2）
- 縱軸：完成幾輪
- ⭐ = 完成、☆ = 未完成
- 每完成一個練習頁，對應位置自動亮星

```
         W1    W2    W3    W4    T1    W5    W6    P1    P2    T2    P3    P4
        簡報  會議  接待  電話  產業  談判  展會  出差  社交  AI   生活  資訊
第1輪    ⭐    ⭐    ⭐    ⭐    ⭐    ☆    ☆    ☆    ☆    ☆    ☆    ☆
第2輪    ☆    ☆    ☆    ☆    ☆    ☆    ☆    ☆    ☆    ☆    ☆    ☆
第3輪    ☆    ☆    ☆    ☆    ☆    ☆    ☆    ☆    ☆    ☆    ☆    ☆
```

### 收藏複習

- 篩選列：按 Tab 篩選（閱讀 / 會話 / 聽解 / 口說）、按來源篇數篩選
- 每筆：英文 + ▶ 播放 + 中文 + 來源標籤（PE-001 會話）
- 可取消 ⭐

### 資料儲存

全部使用 localStorage（同一個 file:// origin 共享）：
- 練習頁完成時自動寫入星星地圖資料
- ⭐ 按鈕點擊時寫入收藏資料
- 複習頁打開時從 localStorage 讀取並渲染

---

## 補充機制（跨 Tab 通用）

### 編號系統

每個 HTML 頁面有唯一編號：`PE-XXX`（Practical English）
檔名格式：`PE-001 W1 簡報開場.html`

### 提問格式

使用者對 Claude 說：
- 「PE-001 **會話** 第 3 句的 _____ 是什麼意思」
- 「PE-001 **聽解** 第 2 段的 _____ 不懂」
- 「PE-001 **口說** 第 1 題」

### 補充寫入

Claude 收到後，Edit 對應 HTML 頁面，在該 Tab 底部新增：

| 區塊 | 格式 |
|------|------|
| 補充單字 | 單字 + 音標 + 中文 + 例句 + ▶ |
| 補充文法 | 文法規則 + 正/反例句 + 使用情境 |

累積式新增，可作為複習內容。

### 功能提示

每個 Tab 底部固定顯示提示文字，引導使用者知道可以提問。
