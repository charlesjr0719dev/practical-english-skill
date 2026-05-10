---
source_url: https://duolingo.com
site_name: Duolingo
extracted_at: 2026-05-10
theme: light
use_case: 實用英文學習
tags: [playful, gamification, rounded, bold, high-contrast, friendly, vibrant, flat-ui]
---

# Duolingo — 實用英文學習設計系統

## CSS 變數（直接複製到 HTML）

```css
@import url('https://fonts.googleapis.com/css2?family=Nunito+Sans:wght@400;600;700;800&display=swap');

:root {
  --bg: #ffffff;
  --surface: #ffffff;
  --surface-2: #f7f7f7;
  --border: #e5e5e5;
  --text: #3c3c3c;
  --text-dim: #777777;
  --accent: #1cb0f6;       /* Sky Blue — 播放鍵、Tab 切換、連結 */
  --accent-2: #ce82ff;     /* Purple — 補充區塊（單字/文法補充） */
  --success: #58cc02;      /* Duo Green — 主要按鈕、正確回饋 */
  --warn: #ff9600;         /* Orange — 關鍵句型 highlight */
  --danger: #ea2b2b;       /* Red — 錯誤回饋 */
}
```

## 色彩表

| Hex | Name | 用途 |
| --- | --- | --- |
| `#58cc02` | Duo Green | 主要 CTA、正確回饋、送出按鈕 |
| `#1cb0f6` | Sky Blue | 播放按鈕、Tab 選中狀態、連結 |
| `#d7ffb8` | Duo Green Light | 正確答案背景色 |
| `#ff9600` | Orange | 關鍵句型 highlight |
| `#ce82ff` | Purple | 補充區塊（單字/文法補充） |
| `#ffffff` | Snow White | 頁面背景、卡片背景 |
| `#f7f7f7` | Light Gray | 次要背景（情境描述、句型收穫區） |
| `#e5e5e5` | Cloud Gray | 邊框、Tab 分隔線 |
| `#777777` | Graphite | 次要文字、中文翻譯 |
| `#3c3c3c` | Almost Black | 主要文字 |
| `#ea2b2b` | Red | 錯誤回饋、答錯標示 |

## 設計規則

### Do
- 白底乾淨畫布，section 用 `border: 2px solid #e5e5e5; border-radius: 12px`
- 主要按鈕用 3D 觸感：`box-shadow: 0 4px 0 #46a302`，hover 時 `translateY(2px)`，active 時 `translateY(4px)` + shadow none
- 播放鍵用 Sky Blue bg + 白字 + `box-shadow: 0 3px 0 #1899d6`
- 字型：`'Nunito Sans'`（圓潤友善），fallback `-apple-system, "PingFang TC", sans-serif`
- 句型規則框：淺綠底 `#f0fff0` + 左邊線 `#58cc02`
- 資訊提示框（功能提示）：淺藍底 `#e8f4fd`
- 補充區塊：淺紫底 `#f3e8ff` + 左邊線 `#ce82ff`
- 藥丸標籤：`border-radius: 999px`，綠色外框
- Tab 切換：選中 = Sky Blue 底線，未選 = 灰字
- heading font-weight: 800

### Don't
- 不用尖角（所有 UI 元素至少 12px 圓角）
- 不用傳統 box-shadow 做卡片浮起效果（只用在按鈕 3D 效果）
- 不用深色背景
- highlight 文字不用黃色（白底上看不清），用 `#ff9600` 橘色

## 字型設定

```css
font-family: 'Nunito Sans', -apple-system, "PingFang TC", sans-serif;
font-size: 17px;
line-height: 1.7;
letter-spacing: 0.02em;
```

## 按鈕樣式範本

```css
/* 主要按鈕（Duo Green 3D） */
.btn-primary {
  background: #58cc02; color: white; border: none;
  padding: 14px 32px; border-radius: 12px;
  font-weight: 700; font-size: 16px; cursor: pointer;
  box-shadow: 0 4px 0 #46a302;
  transition: transform 0.1s, box-shadow 0.1s;
}
.btn-primary:hover { transform: translateY(2px); box-shadow: 0 2px 0 #46a302; }
.btn-primary:active { transform: translateY(4px); box-shadow: none; }

/* 播放按鈕（Sky Blue 3D） */
.play-btn {
  background: #1cb0f6; color: white; border: none;
  min-width: 36px; min-height: 36px; padding: 0 10px;
  border-radius: 12px; cursor: pointer; font-size: 14px;
  box-shadow: 0 3px 0 #1899d6;
  transition: all 0.15s;
}
.play-btn:hover { transform: translateY(1px); box-shadow: 0 2px 0 #1899d6; }
.play-btn:active { transform: translateY(3px); box-shadow: none; }

/* Tab 切換 */
.tab {
  padding: 12px 24px; border: none; background: transparent;
  font-weight: 700; font-size: 15px; color: #777777;
  cursor: pointer; border-bottom: 3px solid transparent;
  transition: all 0.2s;
}
.tab.active {
  color: #1cb0f6;
  border-bottom: 3px solid #1cb0f6;
}
```

---

Generated from Refero MCP (styles.refero.design) — Duolingo style
Adapted for 實用英文學習 by Charles, 2026-05-10
