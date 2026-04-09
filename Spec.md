# 塔羅牌單檔應用（index.html）— 軟體開發規格

版本：1.0  
文件目的：定義「單檔（index.html）」塔羅抽牌 PoC 的功能、介面、資料、演算法與驗收標準。  
交付範圍：僅規格文件（Spec.md），實作檔 index.html 由本規格約束。

---

## 1. 產品目標（Goals）
建立一個可直接在瀏覽器開啟的塔羅抽牌頁面，提供：
- 基本 UI（控制區 + 結果呈現）
- 隨機抽牌（無放回、不重複）
- 正位/逆位判定
- 每張牌顯示對應的「簡單敘述」（一句話～數句話）

---

## 2. 交付物（Deliverables）
1. `index.html`（單檔）
   - 必須包含：HTML + CSS + JavaScript（可內嵌於同檔）
   - 必須可「離線」執行：不可依賴外部 CDN（例如外部 JS/CSS）
2. `Spec.md`（本文件）

---

## 3. 執行環境與限制（Constraints）
- 瀏覽器：Chrome / Edge / Firefox / Safari（近兩年版本）
- 不需後端、無 API 呼叫、無資料庫
- 不需登入，不儲存個資
- 牌庫必須內嵌於 `index.html`（PoC 建議至少 22 張大阿爾克那）

---

## 4. 功能需求（Functional Requirements）

### FR-1：基本 UI
頁面必須含下列區塊：
1. Header
   - 標題（例如「塔羅牌 PoC」）
   - 簡短說明文字（例如「基本 UI、隨機抽牌、簡單敘述」）
2. 控制區（Control Panel）
   - 抽牌張數選擇（至少支援 1 張、3 張）
   - 抽牌按鈕（「抽牌」）
   - 清除按鈕（「清除」）
   - （選配但建議）Seed 輸入與「使用 seed」勾選框，用於重現抽牌結果
3. 結果區（Result Board）
   - 顯示抽到的卡片列表
   - 每張卡片呈現：卡名、正位/逆位、位置（若為 3 張）、簡短牌義敘述

### FR-2：隨機抽牌（無放回）
- 抽牌需從牌庫中隨機挑選 N 張（N=1 或 3），同一次抽牌不可出現重複卡。
- 需使用標準洗牌/抽樣策略以避免偏差：
  - 推薦：Fisher–Yates shuffle 後取前 N 張。

### FR-3：正位 / 逆位
- 每張抽到的牌需決定 orientation：
  - `upright`（正位）
  - `reversed`（逆位）
- 判定規則：每張牌 50% 機率正位、50% 機率逆位（使用同一 RNG 來源）。

### FR-4：簡單敘述（牌義顯示）
- 每張牌必須至少內建兩種敘述：
  - upright 的簡短敘述（1～3 句）
  - reversed 的簡短敘述（1～3 句）
- 結果區顯示時，需依 orientation 顯示對應敘述。

### FR-5：（選配）Seed 可重現抽牌
若提供 seed 功能，需滿足：
- 當「使用 seed」勾選且 seed 字串相同，且抽牌張數相同時：
  - 抽到的牌（順序）與正/逆位結果必須一致（可重現）。
- 若未使用 seed，則每次抽牌結果應呈現非 deterministic（高機率不同）。

---

## 5. 資料規格（Data Specification）

### 5.1 Card（牌庫資料結構）
`CARDS` 為陣列，元素結構如下（欄位必填）：
- `id`: number（唯一，建議 0..21）
- `name`: string（例如「愚者 (The Fool)」）
- `meaning`: object
  - `upright`: string（正位敘述）
  - `reversed`: string（逆位敘述）

> PoC 最小牌庫：22 張（大阿爾克那）。允許後續擴充到 78 張。

### 5.2 CardInstance（抽牌結果結構）
抽牌結果為 `CardInstance[]`，元素結構：
- `card_id`: number
- `name`: string
- `position_index`: number（0-based）
- `position_name`: string
  - 若 N=1：固定為「單張」
  - 若 N=3：依序為「過去」「現在」「未來」
- `orientation`: 'upright' | 'reversed'
- `meaning`: string（依 orientation 選取的敘述）

---

## 6. 演算法規格（Algorithm Specification）

### 6.1 RNG（亂數來源）
- 非 seed 模式：
  - 使用 `Math.random()` 產生 [0,1) 的浮點亂數
- seed 模式（若實作）：
  1. 將 seed 字串 hash 成 32-bit integer（例如 cyrb128 或同等）。
  2. 將 hash 結果餵入 seedable PRNG（例如 mulberry32 / xorshift）。
  3. 由 PRNG 產生 [0,1) 浮點亂數序列。

### 6.2 洗牌與抽牌
- 使用 Fisher–Yates 進行洗牌：
  - 從尾端 i 到 1，取 j = floor(rng()*(i+1))，交換 a[i] 與 a[j]
- 抽牌：
  - 取洗牌後前 N 張（slice(0,N)）

### 6.3 Orientation 判定
- 對每張抽到的牌呼叫一次 rng()：
  - 若 < 0.5 → upright
  - 否則 → reversed

---

## 7. UI/UX 規格（UI/UX Specification）
- 行動裝置友善：結果卡片在窄螢幕需單欄或兩欄顯示
- 結果呈現需清楚可讀：
  - 卡名醒目（粗體/較大字）
  - 正位/逆位以 badge 標示
  - 敘述區塊行距適中
- （選配）動畫：
  - 抽牌後卡片淡入或翻牌效果
  - 動畫不應阻礙操作

---

## 8. 錯誤處理（Error Handling）
- 當張數 > 牌庫總數：
  - 顯示錯誤訊息（alert 或 UI 提示），且不進行抽牌。
- seed 欄位可空字串：
  - 若勾選使用 seed 且 seed 為空，允許（空字串視為固定 seed），或提示使用者輸入（兩者擇一，需在實作中一致）。

---

## 9. 無障礙（Accessibility）最低要求
- 主要控制元件需可鍵盤操作（Tab 走訪、Enter 觸發）
- 結果區使用 `aria-live="polite"` 或等效方式讓更新可被輔助工具感知
- 表單元件有 label 或 aria-label

---

## 10. 驗收標準（Acceptance Criteria）
1. 打開 `index.html`（離線）即可進入頁面，且無外部資源依賴。
2. 使用者可選 1 張或 3 張並抽牌，結果顯示正確。
3. 同一次抽牌的 N 張卡片不重複。
4. 每張卡片必顯示：
   - 卡名
   - 正位/逆位
   - 簡短敘述（依 orientation）
   - 若 N=3，顯示過去/現在/未來位置
5. 若實作 seed：
   - 相同 seed + 相同張數 → 結果可重現（牌與正逆位一致）

---

## 11. 測試案例（Test Cases）
- TC-01：抽 1 張（無 seed）連續抽 3 次，結果應高機率不同
- TC-02：抽 3 張（無 seed），確認無重複卡
- TC-03：抽 3 張（seed="abc" 並使用 seed），連續抽 3 次結果完全一致
- TC-04：三張牌位置名稱為「過去/現在/未來」
- TC-05：清除按鈕可清空結果區

---

## 12. 未來擴充（非本次範圍）
- 牌庫擴成 78 張、支援四花色與數字/宮廷牌
- 新增更多牌陣（塞爾特十字等）
- 加入 Reading 歷史（LocalStorage / 後端）
- 分享連結：URL 攜帶 seed + 牌陣參數
- AI 解讀（需後端 worker 與成本控管）
