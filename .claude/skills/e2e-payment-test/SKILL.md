---
name: e2e-payment-test
description: 花漾生活電商 E2E 結帳流程測試（Playwright + 綠界 Sandbox）。登入 → 加入商品 → 填寫收件資訊 → 進入綠界金流（網路ATM + 台灣土地銀行）→ 完成付款 → 附上截圖。使用方式：/e2e-checkout 或 /e2e-checkout <收件人姓名>
---

# E2E 結帳流程測試

花漾生活電商網站完整結帳流程自動化測試，使用 Playwright MCP 操作瀏覽器，走過綠界科技 Sandbox 金流。

## 參數

- `args`（可選）：收件人姓名，例如 `/e2e-payment-test Eva`。未提供則預設 `測試用戶`。

## 帳號設定（測試環境固定值）

- Email：`admin@hexschool.com`
- 密碼：`12345678`
- 伺服器：`http://localhost:3001`

---

## 執行流程

執行此 skill 時，請於 Sandbox 環境下，依序完成以下 9 個步驟。**每步驟需確認成功後再進行下一步。**

### Step 1｜確認 Sandbox 環境與伺服器狀態

**1-A：驗證綠界 Sandbox 環境變數**

執行以下指令，確認 `ECPAY_ENV` 必須為 `staging`：

```bash
grep ECPAY_ENV .env
```

- **輸出應為** `ECPAY_ENV=staging`
- **若為 `production` 或不存在**：**立即停止**，修正 `.env` 後再繼續，避免觸及正式環境

同時確認商戶 ID 為 Sandbox 測試帳號（`3002607`）：

```bash
grep ECPAY_MERCHANT_ID .env
```

**1-B：確認伺服器狀態**

執行 Bash 指令檢查 port 3001：

```bash
lsof -i :3001 | head -3
```

- **若有輸出**：伺服器已運行，繼續。
- **若無輸出**：執行以下指令啟動，並等待 3 秒確認：
  ```bash
  npm run dev:server &> /tmp/server.log &
  sleep 3 && lsof -i :3001 | head -3
  ```

---

### Step 2｜登入

1. 導覽至 `http://localhost:3001/login`
2. 填入表單：
   - `[placeholder='your@email.com']` → `admin@hexschool.com`
   - `[placeholder='••••••••']` → `12345678`
3. 點擊 `button:has-text('登入')`
4. 確認頁面跳轉至 `http://localhost:3001/`（首頁）

---

### Step 3｜加入商品至購物車

點擊頁面中第一個 `Add` 按鈕：

```
getByRole('button', { name: 'Add' }).first()
```

---

### Step 4｜前往購物車並結帳

1. 導覽至 `http://localhost:3001/cart`
2. 確認購物車有商品
3. 點擊 `button:has-text('前往結帳')`
4. 確認頁面跳轉至 `/checkout`

---

### Step 5｜填寫收件資訊

從 `args` 取得收件人姓名（無則使用 `測試用戶`），填入：

| 欄位 | Selector | 值 |
|------|----------|----|
| 收件人姓名 | `[placeholder='請輸入收件人姓名']` | `{args 或 測試用戶}` |
| Email | `[placeholder='your@email.com']` | `admin@hexschool.com` |
| 收件地址 | `[placeholder='請輸入完整收件地址']` | `台北市信義區松仁路100號` |

填完後點擊 `button:has-text('前往付款')`。

---

### Step 6｜綠界金流 Sandbox 操作

確認已進入 `payment-stage.ecpay.com.tw`，依序執行：

1. 點擊「網路ATM」：
   ```
   getByRole('listitem', { name: 'WebATM' })
   ```

2. 選擇台灣土地銀行（使用精確的 ID selector 避免多選單衝突）：
   ```
   #selWebATMBank → 選擇「台灣土地銀行」
   ```

3. 點擊「前往付款」連結：
   ```
   getByRole('link', { name: '前往付款' })
   ```

4. 出現提示彈窗後點擊「關閉」：
   ```
   getByRole('button', { name: '關閉' })
   ```

---

### Step 7｜處理自動表單跳轉（偶發情況）

關閉彈窗後，可能出現兩種情況：

**情況 A（正常）：** 頁面直接跳轉至 `pay-stage.ecpay.com.tw/MockMPPost/LandWebAtm`，繼續 Step 8。

**情況 B（偶發）：** 頁面停留在 `DoAutoSubmitForm`，顯示「交易資料傳輸中」。

遇到情況 B 時，使用 `browser_evaluate` 手動觸發表單提交：

```javascript
() => { const form = document.querySelector('form'); if(form) { form.submit(); return 'submitted'; } return 'no form'; }
```

確認跳轉至 `LandWebAtm` 頁面後繼續。

---

### Step 8｜模擬銀行回傳

已在 `pay-stage.ecpay.com.tw/MockMPPost/LandWebAtm` 頁面：

- 確認 RC 欄位值為 `0`（交易成功）
- 點擊 `getByRole('button', { name: 'Save' })`
- 確認頁面標題變為「付款成功|綠界科技」

---

### Step 9｜截圖並回傳結果

1. 截圖付款成功頁，檔名：`e2e-payment-test-success.png`，並放在 `/.claude/skills/e2e-payment-test` 目錄下
2. 點擊「返回商店」：`getByRole('link', { name: '返回商店' })`
3. 截圖訂單詳情頁，檔名：`e2e-payment-test-order-detail.png`，並放在 `/.claude/skills/e2e-payment-test` 目錄下
4. 使用 `SendUserFile` 傳送兩張截圖給使用者
5. 回傳測試結果摘要：

| 步驟 | 結果 |
|------|------|
| 伺服器 port 3001 | ✅ |
| 登入 admin@hexschool.com | ✅ |
| 加入商品至購物車 | ✅ |
| 填寫收件資訊（{收件人姓名}） | ✅ |
| 綠界 Sandbox：選網路ATM | ✅ |
| 選擇台灣土地銀行 | ✅ |
| 模擬銀行回傳（RC=0） | ✅ |
| 訂單建立完成 | ✅ |

---

## 常見問題排查

| 問題 | 處理方式 |
|------|----------|
| 伺服器無法啟動 | 確認目前目錄為專案根目錄，執行 `npm run dev:server` |
| 登入失敗 | 確認測試帳號存在，或重新 seed 資料庫 |
| 綠界頁面無法載入 | 確認網路可連至 `payment-stage.ecpay.com.tw` |
| `DoAutoSubmitForm` 卡住 | 見 Step 7 情況 B，使用 `form.submit()` 手動觸發 |
| 銀行選單衝突 | 使用 `#selWebATMBank` 而非 `select` 選擇器（頁面有多個 select） |
