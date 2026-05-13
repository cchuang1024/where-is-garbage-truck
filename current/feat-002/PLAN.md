# PLAN — feat-002：接下來的垃圾車

## 修改範圍

只修改 `index.html`，`data.js` 不變。

## 新增 UI 元素

在 `<main>` 的定位結果上方加入固定操作列：

```html
<div id="toolbar" hidden>
  <button id="btnUpcoming">現在之後還有幾班？</button>
  <button id="btnAll" hidden>顯示全天</button>
</div>
```

- 定位成功後 `toolbar` 才顯示（`hidden` 移除）
- 兩個按鈕互斥切換：按「接下來」後顯示「全天」按鈕，反之亦然

## 狀態管理

新增模組層級變數：

```javascript
let allNearby = [];   // 定位後快取的 200m 內全天站點（feat-001 結果）
let viewMode  = 'all'; // 'all' | 'upcoming'
```

定位成功 → 計算 `allNearby` → 呼叫 `renderList(allNearby)` → 顯示 toolbar

## 新增函式

### nowHHMM() → 整數
```javascript
const d = new Date();
return d.getHours() * 100 + d.getMinutes(); // e.g. 18:05 → 1805
```

### minutesUntil(arriveHHMM) → 整數
```javascript
const now = nowHHMM();
const nowMin  = Math.floor(now / 100) * 60 + (now % 100);
const arrMin  = Math.floor(arriveHHMM / 100) * 60 + (arriveHHMM % 100);
return arrMin - nowMin; // 負數表示已抵達
```

### statusTag(stop) → 字串
```javascript
const mins = minutesUntil(stop.arrive);
if (mins > 0)  return `${mins} 分鐘後抵達`;
return '現正停靠';
```

### filterUpcoming(stops) → stops[]
```javascript
const now = nowHHMM();
return stops.filter(s => s.depart >= now);
```

## 修改 renderList

接受第二個參數 `showStatus = false`，若為 `true` 則每個 card 多顯示狀態標籤：

```html
<span class="tag status">現正停靠</span>
<!-- 或 -->
<span class="tag upcoming">12 分鐘後抵達</span>
```

## 按鈕事件

```javascript
btnUpcoming.addEventListener('click', () => {
  const upcoming = filterUpcoming(allNearby);
  if (upcoming.length === 0) {
    setStatus('今日附近班次已全數離站。');
  } else {
    renderList(upcoming, true); // showStatus = true
  }
  viewMode = 'upcoming';
  btnUpcoming.hidden = true;
  btnAll.hidden = false;
});

btnAll.addEventListener('click', () => {
  renderList(allNearby, false);
  viewMode = 'all';
  btnAll.hidden = true;
  btnUpcoming.hidden = false;
});
```

## CSS 新增

```css
.tag.status   { background: #fff3cd; color: #856404; }  /* 現正停靠：黃 */
.tag.upcoming { background: #d1ecf1; color: #0c5460; }  /* 即將抵達：藍 */

#toolbar { display: flex; gap: 8px; margin-bottom: 12px; }
#toolbar button {
  flex: 1;
  padding: 10px;
  border: none;
  border-radius: 8px;
  background: #2d7d46;
  color: #fff;
  font-size: 0.9rem;
  cursor: pointer;
}
#toolbar button:disabled { background: #ccc; cursor: default; }
```
