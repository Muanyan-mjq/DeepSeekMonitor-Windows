# 开发日志 2026-06-08：详情页按日 Token 消耗堆叠柱状图

## 需求

详情页底部"按日 Token 消耗"柱状图需要展示缓存命中/未命中/输出 Token 的细分数据：

- 每根柱子从单色实心改为三色堆叠：缓存命中(青色)、缓存未命中(橙色)、输出 Token(紫色)
- 鼠标 hover 到柱子上时弹出半透明磨玻璃浮窗，显示该日详细数据
- 每次只显示一根柱子的浮窗
- 图表右上角显示颜色图例

---

## 数据流分析

### 改动前

```
平台 API (天级)
└── days[].data[].model_usage.usage[]
      ├── PROMPT_TOKEN
      ├── PROMPT_CACHE_HIT_TOKEN
      ├── PROMPT_CACHE_MISS_TOKEN
      ├── RESPONSE_TOKEN
      └── REQUEST
           ↓
后端 token_breakdown() → 只取 total (丢弃 hit/miss/response)
           ↓
UsageDaySummary { flash_tokens, pro_tokens, total_tokens }
           ↓
前端 points[] → value = day.flashTokens (单一数字)
           ↓
蓝色渐变单色柱
```

### 改动后

```
平台 API (天级)
└── days[].data[].model_usage.usage[]
      ├── PROMPT_CACHE_HIT_TOKEN  → 缓存命中
      ├── PROMPT_CACHE_MISS_TOKEN → 缓存未命中
      ├── RESPONSE_TOKEN          → 输出 Token
      └── ...
           ↓
后端 token_breakdown() → 返回 (total, request, hit, miss, response)
      ↓ 分别按 flash/pro 模型汇总
UsageDaySummary {
  flash_tokens, flash_cache_hit, flash_cache_miss, flash_response,
  pro_tokens,   pro_cache_hit,   pro_cache_miss,   pro_response,
  total_tokens, total_cost
}
           ↓
前端 points[] → { hit, miss, response, total }
           ↓
三色堆叠柱 + hover 磨玻璃浮窗 + 颜色图例
```

---

## 改动文件清单

### 1. `src-tauri/src/lib.rs` — 后端

#### a) 数据结构扩展 (line 656-670)

`UsageDaySummary` 新增 6 个字段：

| 新增字段 | Serde 序列化 | 含义 |
|---|---|---|
| `flash_cache_hit` | `flashCacheHit` | Flash 模型缓存命中 Token |
| `flash_cache_miss` | `flashCacheMiss` | Flash 模型缓存未命中 Token |
| `flash_response` | `flashResponse` | Flash 模型输出 Token |
| `pro_cache_hit` | `proCacheHit` | Pro 模型缓存命中 Token |
| `pro_cache_miss` | `proCacheMiss` | Pro 模型缓存未命中 Token |
| `pro_response` | `proResponse` | Pro 模型输出 Token |

```rust
#[derive(Debug, Serialize)]
#[serde(rename_all = "camelCase")]
struct UsageDaySummary {
    date: String,
    flash_tokens: u64,
    flash_cache_hit: u64,
    flash_cache_miss: u64,
    flash_response: u64,
    pro_tokens: u64,
    pro_cache_hit: u64,
    pro_cache_miss: u64,
    pro_response: u64,
    total_tokens: u64,
    total_cost: f64,
}
```

#### b) 每日聚合逻辑 (line 846-889)

原代码在每日循环中只取 `token_breakdown` 返回的 `total`：

```rust
let (tokens, _, _, _, _) = token_breakdown(&model_usage.usage);
```

改为完整接收所有返回值，并按模型分别累加：

```rust
let (tokens, _, hit, miss, response) = token_breakdown(&model_usage.usage);
// ...
match model_usage.model.as_str() {
    "deepseek-v4-flash" => {
        flash += tokens;
        flash_hit += hit;
        flash_miss += miss;
        flash_resp += response;
    }
    "deepseek-v4-pro" => {
        pro += tokens;
        pro_hit += hit;
        pro_miss += miss;
        pro_resp += response;
    }
    _ => {}
}
```

---

### 2. `src/main.tsx` — 前端

#### a) `UsageDay` 类型扩展 (line 52-64)

新增 6 个字段，与后端 `UsageDaySummary` 一一对应：

```typescript
type UsageDay = {
  date: string;
  flashTokens: number;
  flashCacheHit: number;
  flashCacheMiss: number;
  flashResponse: number;
  proTokens: number;
  proCacheHit: number;
  proCacheMiss: number;
  proResponse: number;
  totalTokens: number;
  totalCost: number;
};
```

#### b) `recentUsageDays` 默认值 (line 100-113)

补充新字段的 fallback 默认值为 `0`：

```typescript
source.get(date) ?? {
  date,
  flashTokens: 0,
  flashCacheHit: 0,
  flashCacheMiss: 0,
  flashResponse: 0,
  proTokens: 0,
  proCacheHit: 0,
  proCacheMiss: 0,
  proResponse: 0,
  totalTokens: 0,
  totalCost: 0,
}
```

#### c) `ModelDetailPanel` 柱状图重构 (line 924-1031)

**数据预处理**：每根柱子提取 hit/miss/response，用输入数据计算命中率：

```typescript
const points = days.map((day) => {
  const hit = isFlash ? day.flashCacheHit : day.proCacheHit;
  const miss = isFlash ? day.flashCacheMiss : day.proCacheMiss;
  const response = isFlash ? day.flashResponse : day.proResponse;
  return { date: day.date, hit, miss, response, total: hit + miss + response };
});
```

**hover 状态管理**：用 `React.useState<number | null>` 跟踪当前 hover 的柱子索引，`onMouseLeave` 清空。

**堆叠柱渲染**：按 hit → miss → response 顺序从底部到顶部堆叠（CSS flex-end 排列），每段高度按 `maxVal` 等比计算，保留 `MIN_SEG=6%` 最小可见高度：

```tsx
<div className="detail-bar-stacked">
  {point.hit > 0 && <i className="seg hit" style={{ height: `${Math.max(MIN_SEG, (point.hit / maxVal) * 100)}%` }} />}
  {point.miss > 0 && <i className="seg miss" style={{ height: `${Math.max(MIN_SEG, (point.miss / maxVal) * 100)}%` }} />}
  {point.response > 0 && <i className="seg response" style={{ height: `${Math.max(MIN_SEG, (point.response / maxVal) * 100)}%` }} />}
  {point.total === 0 && <i className="seg empty" style={{ height: `${MIN_SEG}%` }} />}
</div>
```

**磨玻璃浮窗**：hover 时在柱子上方绝对定位渲染：

```tsx
{hoveredIdx === idx && point.total > 0 && (
  <div className="bar-tooltip">
    <span className="bar-tooltip-date">{point.date}</span>
    <span>输入 <strong>{fmtTokensShort(point.hit + point.miss)}</strong></span>
    <span className="bar-tooltip-sub">缓存命中 {fmtTokensShort(point.hit)}</span>
    <span className="bar-tooltip-sub">缓存未命中 {fmtTokensShort(point.miss)}</span>
    {point.hit + point.miss > 0 && (
      <span>命中率 <strong>{((point.hit / (point.hit + point.miss)) * 100).toFixed(1)}%</strong></span>
    )}
    <span>输出 Token <strong>{fmtTokensShort(point.response)}</strong></span>
    <span className="bar-tooltip-total">合计 <strong>{fmtTokensShort(point.total)}</strong></span>
  </div>
)}
```

**颜色图例**：在 `detail-chart-head` 中新增 `chart-legend` 区域，窄屏时自动下沉到图表底部（`chart-legend-mobile`）。

---

### 3. `src/styles.css` — 样式

#### a) 堆叠柱 (line 837-858)

```css
.detail-bar-stacked {
  width: 18px;
  min-height: 6px;
  max-height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: flex-end;  /* 从底部向上排列 */
  border-radius: 8px;
  overflow: hidden;
}

.detail-bar-stacked .seg.hit      { background: #34d399; }   /* 青色 — 缓存命中 */
.detail-bar-stacked .seg.miss     { background: var(--orange); } /* 橙色 — 缓存未命中 */
.detail-bar-stacked .seg.response { background: #a78bfa; }   /* 紫色 — 输出 Token */
.detail-bar-stacked .seg.empty    { background: rgba(246, 239, 222, 0.12); } /* 灰色占位 */
```

关键设计：`flex-direction: column; justify-content: flex-end` 使 DOM 中先出现的元素在底部，后出现的在上方。渲染顺序 hit → miss → response 对应从下到上。

#### b) 磨玻璃浮窗 (line 860-916)

```css
.bar-tooltip {
  position: absolute;
  bottom: 100%;          /* 柱子上方 */
  left: 50%;
  transform: translateX(-50%);  /* 水平居中 */
  z-index: 10;
  background: rgba(30, 28, 16, 0.78);   /* 半透明深色底 */
  backdrop-filter: blur(14px) saturate(1.2); /* 磨玻璃效果 */
  border: 1px solid rgba(255, 255, 255, 0.18);
  border-radius: 10px;
  pointer-events: none;  /* 不影响 hover 冒泡 */
}
```

#### c) 颜色图例 (line 918-962)

```css
.chart-legend {
  display: flex;
  gap: 10px;
  align-items: center;
  flex-wrap: wrap;
}

.chart-legend-item .dot {
  display: inline-block;
  width: 8px;
  height: 8px;
  border-radius: 4px;
}

.chart-legend-item .dot.hit      { background: #34d399; }
.chart-legend-item .dot.miss     { background: var(--orange); }
.chart-legend-item .dot.response { background: #a78bfa; }

/* 窄屏下沉 */
@media (max-width: 280px) {
  .chart-legend { display: none; }
  .chart-legend-mobile { display: flex; }
}
```

---

## 交互效果示意

```
            ┌──────┐       颜色图例:
            │ ░░   │       ■ 缓存命中  ■ 缓存未命中  ■ 输出
            │ ▓▓   │
            │ ██   │
            └──────┘
             6/8
                ↑
        鼠标 hover →
        ┌──────────────────┐
        │ 2026-06-08       │
        │ 输入         12.5K│
        │  缓存命中    8.0K │
        │  缓存未命中  4.5K │
        │ 命中率   64.0%   │
        │ 输出 Token  6.8K │
        │──────────────────│
        │ 合计        19.3K │
        └──────────────────┘
```

---

## 验证

- TypeScript 编译：`tsc --noEmit` 通过，0 错误
- Vite 开发服务器：`http://127.0.0.1:5173/` 正常启动并响应
- 完整运行需 Rust 环境执行 `npm run tauri:dev`
