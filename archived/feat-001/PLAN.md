# PLAN — feat-001：附近全天垃圾車站點

## 檔案清單

| 檔案 | 角色 |
|------|------|
| `data.js` | 垃圾車站點資料，`const TRUCKS = [...]` |
| `index.html` | 頁面結構 + CSS + 主邏輯 |

## data.js 產生方式

從 `garbage_truck.csv` 轉換，每筆資料只保留必要欄位以減少體積：

```javascript
const TRUCKS = [
  { route: "天母-1", arrive: 1630, depart: 1640, addr: "臺北市士林區天母西路48號", lat: 25.11836, lng: 121.525 },
  ...
];
```

欄位對應：
- `route` ← 路線
- `arrive` ← 抵達時間（整數，1630 = 16:30）
- `depart` ← 離開時間（整數）
- `addr` ← 地點
- `lat` ← 緯度
- `lng` ← 經度

省略欄位：行政區、里別、分隊、局編、車號、車次（查詢結果不需要）

## index.html 結構

```html
<header>   標題
<main>     #status（提示訊息）+ #results（站點清單）
<script>   載入 data.js 後執行主邏輯
```

## 主邏輯流程

```
DOMContentLoaded
  → checkDay()         // 週三/日 → 顯示休息訊息，停止
  → showStatus('定位中...')
  → navigator.geolocation.getCurrentPosition(
      onSuccess,
      onError,
      { timeout: 10000 }
    )

onError(err)
  → 依 err.code 顯示對應提示

onSuccess(pos)
  → userLat = pos.coords.latitude
  → userLng = pos.coords.longitude
  → nearby = TRUCKS
      .map(t => ({ ...t, dist: haversine(userLat, userLng, t.lat, t.lng) }))
      .filter(t => t.dist <= 200)
      .sort((a, b) => a.arrive - b.arrive)
  → nearby.length === 0 → 顯示「附近無垃圾車站點」
  → 否則 → renderList(nearby)
```

## 關鍵函式

### haversine(lat1, lng1, lat2, lng2) → 公尺
```
R = 6371000
φ1, φ2 = 緯度轉弧度
Δφ = φ2 - φ1
Δλ = lng2 - lng1（轉弧度）
a = sin²(Δφ/2) + cos(φ1)·cos(φ2)·sin²(Δλ/2)
return R · 2 · atan2(√a, √(1-a))
```

### formatTime(hhmm) → 字串
```
1800 → "18:00"
930  → "09:30"
```
將整數補零到 4 位後取前兩字元 + ":" + 後兩字元。

### renderList(stops)
產生 HTML 清單，每筆顯示距離、地址、抵達、離開、路線。

### checkDay()
`new Date().getDay()` 回傳 0（日）或 3（三）時顯示休息訊息並回傳 `true`。

## 休息日判斷

```javascript
const DAY_NAMES = ['日','一','二','三','四','五','六'];
const day = new Date().getDay(); // 0=日, 3=三
if (day === 0 || day === 3) { ... }
```

## CSS 方針

- Mobile-first，`max-width: 480px`，置中
- 每個站點用 card 樣式，清楚分隔
- 距離用較大字體突出顯示
- 系統字體，無外部字型依賴

## data.js 產生腳本

用 Node.js 或 Python 將 CSV 轉成 JS，執行一次即可：

```python
import csv, json, sys

rows = []
with open('garbage_truck.csv', encoding='utf-8-sig') as f:
    for r in csv.DictReader(f):
        rows.append({
            'route':  r['路線'],
            'arrive': int(r['抵達時間']),
            'depart': int(r['離開時間']),
            'addr':   r['地點'],
            'lat':    float(r['緯度']),
            'lng':    float(r['經度']),
        })

print('const TRUCKS = ' + json.dumps(rows, ensure_ascii=False) + ';')
```
