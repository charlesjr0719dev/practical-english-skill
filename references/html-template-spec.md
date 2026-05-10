# HTML 頁面生成規格

生成每個練習頁的完整技術規格。生成時同時參考 `design-system.md` 的 CSS 變數與按鈕樣式。

---

## 頁面結構

```html
<!DOCTYPE html>
<html lang="zh-TW">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>PE-XXX [類別] [主題] — Practical English</title>
  <!-- Nunito Sans 字型 -->
  <link href="https://fonts.googleapis.com/css2?family=Nunito+Sans:wght@400;600;700;800&display=swap" rel="stylesheet">
  <style>/* 見下方 CSS 規格 */</style>
</head>
<body>
  <h1>PE-XXX [類別] [主題]</h1>
  <p class="subtitle">Practical English — [場景分類]</p>
  
  <!-- Tab Bar -->
  <!-- Tab 1: 會話 -->
  <!-- Tab 2: 聽解 -->
  <!-- Tab 3: 口說 -->
  
  <!-- OpenAI API Key Modal -->
  
  <script>/* 見下方 JS 規格 */</script>
</body>
</html>
```

---

## CSS 必要元件

從 `design-system.md` 載入所有 CSS 變數（`--bg`, `--surface`, `--accent`, `--success` 等）。

額外必要樣式：

### Tab 系統
```css
.tab-bar { display: flex; border-bottom: 2px solid var(--border); margin-bottom: 24px; }
.tab { padding: 12px 24px; border: none; background: transparent; font-weight: 700; font-size: 15px; color: var(--text-dim); cursor: pointer; border-bottom: 3px solid transparent; margin-bottom: -2px; }
.tab.active { color: var(--accent); border-bottom-color: var(--accent); }
.tab-content { display: none; }
.tab-content.active { display: block; }
```

### 完成狀態
```css
.tab-status { display: flex; align-items: center; justify-content: space-between; margin-bottom: 16px; }
.status-badge { display: inline-flex; align-items: center; gap: 6px; padding: 6px 14px; border-radius: 999px; font-size: 13px; font-weight: 700; }
.status-badge.incomplete { background: var(--surface-2); color: var(--text-dim); }
.status-badge.complete { background: var(--success-light); color: var(--success-dark); }
.retry-btn { background: white; border: 2px solid var(--border); padding: 6px 14px; border-radius: 999px; font-size: 13px; font-weight: 700; color: var(--text-dim); cursor: pointer; display: none; }
.retry-btn.visible { display: inline-flex; }
```

### 錄音按鈕（開始 + 停止分開）
```css
.mic-controls { display: flex; align-items: center; justify-content: center; gap: 16px; margin: 16px auto; }
.mic-btn { background: var(--danger); width: 64px; height: 64px; border-radius: 50%; font-size: 28px; box-shadow: 0 4px 0 #c02020; }
.mic-btn:disabled { opacity: 0.4; cursor: not-allowed; }
.mic-btn.recording { animation: pulse 1s infinite; }
.stop-btn { background: #555; width: 64px; height: 64px; border-radius: 50%; font-size: 22px; box-shadow: 0 4px 0 #333; display: none; }
.stop-btn.visible { display: flex; }
```

### 其他必要元件
- `.scenario`：情境描述框（灰底圓角）
- `.play-btn` / `.play-btn-large`：播放按鈕（Sky Blue 3D）
- `.conv-line` / `.conv-role`：對話行
- `.question` / `.option`：選擇題
- `.btn-primary`：送出按鈕（Duo Green 3D）
- `.locked` / `.unlocked`：鎖定/解鎖區塊
- `.transcript-line`：原文行
- `.highlight`：關鍵句型標色（`color: var(--warn)`）
- `.pattern-box`：句型收穫框（淺綠底 + 左邊線）
- `.star-btn`：⭐ 收藏按鈕
- `.info-tip`：功能提示框（淺藍底）
- `.supplement`：補充區塊（淺紫底 + 紫左邊線）

---

## JavaScript 規格

### Tab 切換
```javascript
function switchTab(tab) {
  document.querySelectorAll('.tab-content').forEach(el => el.classList.remove('active'));
  document.querySelectorAll('.tab').forEach(el => el.classList.remove('active'));
  document.getElementById('tab-' + tab).classList.add('active');
  // 對應的 tab button 加 active
}
```

### 音訊播放
```javascript
let currentAudio = null;
function playAudio(name) {
  if (currentAudio) currentAudio.pause();
  currentAudio = new Audio('audio/' + name + '.mp3');
  currentAudio.play();
}
```

### 會話 Tab 提交
- 記錄選擇 → 對答案 → 標色（correct/wrong）→ 解鎖原文 → `saveProgress('conversation')`

### 聽解 Tab 提交
- 同會話，4 題 → 解鎖原文 → `saveProgress('listening')`
- 速度控制：`currentAudio.playbackRate = parseFloat(speed)`

### 語音辨識（口說 Tab）

**重要：SpeechRecognition 必須只建立一次實例（singleton），不可每次 startRecording 都 new 一個新的。**
在 `file://` 或 `localhost` 開啟時，每次 new 都會觸發 Chrome 重問麥克風權限。

```javascript
// ===== 語音辨識：singleton 模式 =====
let recognition = null;

function getRecognition() {
  if (recognition) return recognition;
  const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
  if (!SpeechRecognition) { alert('此瀏覽器不支援語音辨識，請使用 Chrome。'); return null; }
  recognition = new SpeechRecognition();
  recognition.lang = 'en-US';
  recognition.interimResults = true;
  recognition.continuous = true;
  return recognition;
}

// 開始錄音（每輪更新 callback，重用同一實例）
async function startRecording(round) {
  const rec = getRecognition();
  if (!rec) return;
  try { rec.stop(); } catch(e) {}  // 停上一輪，不 null 掉

  // 更新 UI
  document.getElementById('mic-' + round).style.display = 'none';
  document.getElementById('stop-' + round).classList.add('visible');
  const textEl = document.getElementById('recognized-' + round);
  textEl.textContent = '聆聽中...';
  textEl.classList.remove('has-text');

  // onresult 必須從 i=0 累積所有結果（不是從 e.resultIndex）
  rec.onresult = (e) => {
    let finalText = '', interimText = '';
    for (let i = 0; i < e.results.length; i++) {
      if (e.results[i].isFinal) finalText += e.results[i][0].transcript + ' ';
      else interimText += e.results[i][0].transcript;
    }
    const combined = (finalText + interimText).trim();
    textEl.textContent = combined || '聆聽中...';
    textEl.classList.toggle('has-text', !!combined);
    recognizedTexts[round] = finalText.trim() || combined;
  };

  rec.onerror = (e) => {
    if (e.error === 'aborted') return;
    textEl.textContent = `錯誤：${e.error}`;
    stopRecording(round);
  };

  setTimeout(() => { try { rec.start(); } catch(e) {} }, 100);
}

// 停止錄音（不 null recognition，保留實例）
function stopRecording(round) {
  try { recognition && recognition.stop(); } catch(e) {}
  document.getElementById('mic-' + round).style.display = '';
  document.getElementById('mic-' + round).classList.remove('recording');
  document.getElementById('stop-' + round).classList.remove('visible');
  if (recognizedTexts[round] && recognizedTexts[round].trim()) {
    document.getElementById('speak-submit-' + round).classList.add('visible');
  }
}
```

**retryTab('speaking') 時**：只呼叫 `recognition.stop()`，不要把 `recognition` 設為 null。

### OpenAI API 呼叫（口說回饋）
```javascript
async function submitSpeaking(round) {
  const apiKey = localStorage.getItem('openai_api_key');
  if (!apiKey) { /* 顯示 API key modal */ return; }
  
  const response = await fetch('https://api.openai.com/v1/chat/completions', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json', 'Authorization': 'Bearer ' + apiKey },
    body: JSON.stringify({
      model: 'gpt-4o-mini',
      messages: [
        { role: 'system', content: '你是英文口說教練...' },
        { role: 'user', content: `情境：... 對方說：... 學生回答：${userText}` }
      ],
      response_format: { type: 'json_object' }
    })
  });
  // 解析回傳 JSON：{ feedback, alternatives: [{ level, text, chinese }] }
}
```

回傳 JSON 格式：
```json
{
  "feedback": "語法正確，但偏口語。商務場合可以更正式。",
  "alternatives": [
    { "level": "正式", "text": "I'm afraid there may be a delay.", "chinese": "恐怕可能會有延遲。" },
    { "level": "委婉", "text": "We might need to adjust the timeline.", "chinese": "我們可能需要調整時程。" },
    { "level": "直接", "text": "We're looking at a two-week delay.", "chinese": "我們預計會延遲兩週。" },
    { "level": "口語", "text": "Looks like it's gonna take longer.", "chinese": "看起來要花更久。" }
  ]
}
```

### 進度儲存
```javascript
function saveProgress(tab) {
  const progress = JSON.parse(localStorage.getItem('pe_progress') || '{}');
  if (!progress['PE-XXX']) {
    progress['PE-XXX'] = { category: '...', title: '...', tabs: {} };
  }
  if (!progress['PE-XXX'].tabs[tab]) {
    progress['PE-XXX'].tabs[tab] = { completed_at: new Date().toISOString() };
    localStorage.setItem('pe_progress', JSON.stringify(progress));
  }
  updateTabStatus(tab, true);
}
```

### 星號收藏
```javascript
function toggleStar(btn, source, tab, english, chinese) {
  btn.classList.toggle('starred');
  const isStarred = btn.classList.contains('starred');
  btn.textContent = isStarred ? '⭐' : '☆';
  const stars = JSON.parse(localStorage.getItem('pe_stars') || '[]');
  if (isStarred) {
    stars.push({ source, tab, english, chinese, starred_at: new Date().toISOString().split('T')[0] });
  } else {
    const idx = stars.findIndex(s => s.english === english && s.source === source);
    if (idx >= 0) stars.splice(idx, 1);
  }
  localStorage.setItem('pe_stars', JSON.stringify(stars));
}
```

### 頁面載入初始化
```javascript
window.addEventListener('load', () => {
  // 1. 從 localStorage 恢復星號狀態
  // 2. 從 localStorage 恢復 Tab 完成狀態
});
```

### 重來按鈕
```javascript
function retryTab(tab) {
  // 重置該 tab 的 UI（清除選擇、鎖定原文、恢復按鈕）
  // 不清除 localStorage 的完成記錄
}
```

---

## 功能提示（每個 Tab 底部）

```html
<div class="info-tip">
  💡 有不懂的單字或文法？告訴 Claude：「PE-XXX [Tab名] 第 N 句的 _____ 是什麼意思」
</div>
```

---

## 補充區塊預留位置

每個 Tab 結尾留兩個空的 div，供 Claude 事後補充寫入：

```html
<div id="supplement-conversation-vocab"></div>
<div id="supplement-conversation-grammar"></div>
```
