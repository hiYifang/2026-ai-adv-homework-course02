
# 花漾生活 — EJS 頁面翻新重構計畫

## Context

**為什麼需要重構？**
現有 EJS 頁面採用「明亮玫瑰色」主題（奶油白背景 + #C4727F 玫瑰主色），風格偏向日系輕柔。
Pencil 設計稿（`docs/design/claude_code_pencil.pen`）已確立全新的「暗黑奢華花藝」風格：
- 全黑背景（`#0A0A0A`）搭配螢光萊姆綠（`$lime`）作為 accent
- Cormorant Garamond 斜體作為標題字型，增添藝廊質感
- 設計語言：editorial、luxury、dark magazine

**目標**：保留所有 EJS 邏輯 / Vue 互動 / API 串接，僅翻新 HTML 結構與 Tailwind class，使前台視覺符合 Pencil 設計稿。不修改後端、不修改 JS 互動邏輯。

---

## 一、現況 vs. 新設計對比

| 面向 | 現況 | 新設計 |
|------|------|------|
| 背景色 | `#FBF8F4` 奶油白 | `#0A0A0A` 純黑 |
| 主強調色 | `#C4727F` 玫瑰紅 | `#C8E600`（萊姆綠，待確認 hex） |
| 文字主色 | `#2C2A28` 深棕 | `#F5F0E8` 米白（chalk） |
| 文字次色 | `#6B6560` 棕灰 | `#888888`（stone） |
| 標題字型 | Noto Serif TC | **Cormorant Garamond** 斜體 |
| 內文字型 | Noto Sans TC | **Noto Sans TC**（保留） |
| 輔助字型 | — | **JetBrains Mono**（標籤、價格） |
| 卡片背景 | `#FFFFFF` 白色 | `#111111` 深灰 |
| 邊框 | `border-gray-100/200` | `#1E1E1E` / `#222222` |
| 圓角 | rounded-2xl / rounded-full | 較少圓角，多直角 / rounded-sm |
| Hero 佈局 | 漸層背景卡片（局部寬） | 全寬全出血影像 |
| Emoji | ✅ 大量使用 | ❌ 移除，改用文字標籤 |

---

## 二、CSS 主題重構（`public/css/input.css`）

替換 `@theme` 區塊的色彩變數為新設計系統，並新增字型變數：

```css
@theme {
  /* 背景 */
  --color-ink:          #0A0A0A;
  --color-ink-soft:     #0F0F0F;
  --color-surface-card: #111111;

  /* 強調色 */
  --color-lime:         #C8E600;

  /* 文字 */
  --color-chalk:        #F5F0E8;
  --color-stone:        #888888;

  /* 邊框 */
  --color-border:       #1E1E1E;
  --color-border-light: #222222;

  /* 保留舊玫瑰色（後台仍使用） */
  --color-rose-primary: #C4727F;
  --color-rose-dark:    #A85B67;
  --color-cream:        #FBF8F4;
}
```

新增 Google Fonts 引入（`partials/head.ejs`）：
```
Cormorant Garamond:ital,wght@1,400;1,600;1,700
JetBrains Mono:wght@400;500
```

---

## 三、重構策略

### 原則
- **Mobile First**：先寫 Mobile 樣式，再加 `md:` / `lg:` breakpoint
- **保留 EJS 邏輯**：所有 `<%= %>` / `<%- %>` / `v-for` / `v-if` 不動
- **保留 Vue 資料繫結**：`:src` / `@click` / `v-model` 不動
- **不新增後端路由或 API**
- **Tailwind CSS only**（不引入其他 CSS 框架）

### 共用 Partials 改動
| 檔案 | 改動重點 |
|------|---------|
| `partials/head.ejs` | 新增 Cormorant Garamond + JetBrains Mono |
| `partials/header.ejs` | 全黑背景、萊姆綠 Logo、導航字型改 mono、購物車徽章色 |
| `partials/footer.ejs` | 深色多欄 Footer（品牌 + 導航 + 社群），取代目前單行設計 |
| `layouts/front.ejs` | body 加 `bg-ink text-chalk`，移除 main 的 max-w-7xl（讓 Hero 全出血） |

---

## 四、元件拆分方案

新增以下 Partials（複用設計稿中重複出現的元件）：

| 新 Partial | 用途 | 套用頁面 |
|-----------|------|---------|
| `partials/section-label.ejs` | 萊姆綠 mono tag + Cormorant 斜體標題的組合（如 "FEATURED / 精選推薦"） | 首頁、訂單頁 |
| `partials/checkout-stepbar.ejs` | 三步驟進度條（購物車 → 收件資訊 → 付款） | 結帳頁 |

---

## 五、頁面重構順序（第一階段）

依視覺衝擊力與用戶旅程優先排序：

### Phase 1 — 基礎 + 最高曝光頁（立即進行）

| 順序 | 頁面 | 檔案 | 重點改動 |
|------|------|------|---------|
| 1 | CSS 主題 | `public/css/input.css` | 替換色彩變數、新增字型變數 |
| 2 | 字型引入 | `views/partials/head.ejs` | 新增 Cormorant Garamond / JetBrains Mono |
| 3 | Layout | `views/layouts/front.ejs` | `bg-ink text-chalk`，調整 main padding |
| 4 | Header | `views/partials/header.ejs` | 暗色 sticky header + lime accent + mono nav |
| 5 | Footer | `views/partials/footer.ejs` | 多欄暗色 footer，對齊設計稿 |
| 6 | **首頁** | `views/pages/index.ejs` | Hero 全出血 + FeaturedSec + AllProd grid + StatsBar |
| 7 | **登入頁** | `views/pages/login.ejs` | 全版背景圖 + 深色卡片（設計稿 05/05M） |

### Phase 2 — 核心購物流程
| 順序 | 頁面 | 檔案 |
|------|------|------|
| 8 | 商品詳情 | `views/pages/product-detail.ejs` |
| 9 | 購物車 | `views/pages/cart.ejs` |
| 10 | 結帳 | `views/pages/checkout.ejs`（含 stepbar partial） |

### Phase 3 — 訂單 + 邊緣頁
| 順序 | 頁面 | 檔案 |
|------|------|------|
| 11 | 我的訂單 | `views/pages/orders.ejs` |
| 12 | 訂單詳情 | `views/pages/order-detail.ejs` |
| 13 | 404 頁面 | `views/pages/404.ejs` |

---

## 六、關鍵設計規格（參照 Pencil 設計稿）

### 色彩對應（設計稿變數 → Tailwind class）
| 設計稿變數 | 用途 | Tailwind class |
|-----------|------|---------------|
| `$ink` | 頁面背景 | `bg-ink` |
| `$lime` | CTA 按鈕、強調標籤 | `bg-lime text-ink` |
| `$chalk` | 主文字 | `text-chalk` |
| `$stone` | 次要文字 | `text-stone` |
| `$surface-card` | 卡片背景 | `bg-surface-card` |
| `$border` | 分隔線 | `border-border` |

### 字型應用規則
```
Cormorant Garamond italic → 頁面大標題、產品名稱、品牌口號
  class: font-[Cormorant_Garamond] italic font-600

JetBrains Mono → 標籤文字（FEATURED / NEW）、價格、訂單編號
  class: font-mono tracking-widest uppercase text-xs

Noto Sans TC → 正文、表單標籤、按鈕文字
  class: （預設 body 字型，無需額外 class）
```

### 關鍵元件規格
**Header**：`h-[72px]` sticky、`bg-ink`、左側 Logo（Cormorant）、右側 nav（mono）+ 購物車徽章（lime）

**CTA 按鈕（主要）**：`bg-lime text-ink font-bold py-3 px-8 text-sm tracking-[2px]`（無 rounded-full，改直角或 rounded-none）

**CTA 按鈕（次要）**：`border border-border text-chalk py-3 px-8 text-sm` 

**卡片**：`bg-surface-card border border-border`（無白色，無陰影）

**標籤（Tag）**：`bg-lime text-ink font-mono text-[9px] tracking-widest uppercase px-2.5 py-1`

**進度條**：lime 圓形步驟符號 + mono 小字標籤（`購物車 → 收件資訊 → 付款`）

---

## 七、不改動的範圍

- `/src/` 下所有後端檔案
- `/public/js/` 下所有 JavaScript 邏輯
- `views/pages/admin/` 後台頁面（維持舊設計）
- `views/layouts/admin.ejs`
- `views/partials/admin-*.ejs`
- API 路由與資料格式

---

## 八、驗證方式

1. `npm run dev:css` 監視 CSS 編譯
2. `npm run dev:server` 啟動本地伺服器
3. 逐頁對照 Pencil 設計稿截圖（使用 `mcp__pencil__get_screenshot`）
4. 使用 Chrome DevTools 切換手機/桌面確認 Mobile First 響應式
5. 執行 `npm run test` 確保後端 API 邏輯未受影響
