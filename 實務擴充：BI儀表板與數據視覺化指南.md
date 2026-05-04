# 實務擴充：BI 儀表板與數據視覺化指南

## 適用對象
系統已經開始累積實際數據，需要定期向管理層匯報「檢驗良率」、「異常趨勢」或「業務量分佈」，希望將冰冷的資料表轉化為高階決策看板的實踐者。

## 核心觀念：資料變現的最後一哩路
將資料安全地存進 PostgreSQL 資料庫，對工程師來說是終點，但對企業主管來說只是起點。**BI (Business Intelligence) 工具** 能直接連線至資料庫底層，透過「拖拉點選 (Drag & Drop)」的方式，零程式碼產出動態且即時更新的戰情儀表板，徹底告別手動匯出 Excel 再畫圖的繁瑣時代。

---

## 工具選型建議

1.  **Metabase (開源王者)**
    *   **優勢**：介面極度友善直覺，非常適合不懂 SQL 語法的行政主管或營運人員直接探索資料。支援極為美觀的圖表與自動化 Email/Slack 排程報表。
    *   **適用情境**：企業內部高階戰情室、團隊內部數據共享。
2.  **Looker Studio (前 Google Data Studio)**
    *   **優勢**：完全免費，能與 Google 體系 (Google Sheets, Google Analytics) 進行無縫的混合資料整合。
    *   **適用情境**：需要快速對外產出報告連結、或是已經大量依賴 Google 生態系的團隊。

---

## 實戰演練：建立第一個戰情儀表板 (以 Metabase + Supabase 為例)

### 步驟一：建立「唯讀」資料庫分身 (Security First)
**資安警告**：絕對不要使用最高權限（擁有刪除資料表能力）的預設密碼來連接 BI 工具！如果有人在 BI 介面不小心下達了刪除指令，將造成毀滅性災難。

1.  進入 Supabase 的 SQL Editor 介面。
2.  執行以下指令，在資料庫中建立一個專屬 BI 的「唯讀帳號」：
    ```sql
    -- 1. 建立一個只能讀取 (SELECT) 的角色
    CREATE ROLE bi_readonly WITH LOGIN PASSWORD '請替換成複雜的專屬密碼';
    -- 2. 允許連線至資料庫
    GRANT CONNECT ON DATABASE postgres TO bi_readonly;
    -- 3. 允許使用 public 結構
    GRANT USAGE ON SCHEMA public TO bi_readonly;
    -- 4. 賦予所有表格的「讀取權限」(禁止新增、修改、刪除)
    GRANT SELECT ON ALL TABLES IN SCHEMA public TO bi_readonly;
    ```

### 步驟二：串接資料庫
1.  註冊並登入 Metabase (或 Looker Studio)。
2.  選擇「新增資料庫連線 (Add Database)」，資料庫類型選擇 `PostgreSQL`。
3.  輸入從 Supabase Database 設定頁面取得的連線資訊：
    *   **Host**: `aws-0-ap-northeast-1.pooler.supabase.com` (依專案實際分配為主)
    *   **Port**: `5432` 或 `6543`
    *   **Database Name**: `postgres`
    *   **User**: `bi_readonly` (剛剛在步驟一建立的唯讀帳號)
    *   **Password**: 剛剛設定的唯讀密碼

### 步驟三：零程式碼建立趨勢圖
1.  連線成功後，點擊首頁的「向資料提問 (Ask a question)」。
2.  選擇 `records` (或檢驗報告) 資料表。
3.  **過濾 (Filter)**：篩選出「狀態 = 異常」的資料列。
4.  **總計 (Summarize)**：依據「建立時間 (Created At)」按「月 (Month)」進行群組計數。
5.  點擊右下角的「視覺化 (Visualize)」，系統會自動畫出一條精準的「每月異常案件趨勢線」。
6.  將圖表儲存，並將其釘選放置於團隊專屬的儀表板 (Dashboard) 上。

### 步驟四：自動化戰情匯報 (Subscriptions)
將儀表板設定好後，可使用 Metabase 的「Dashboard Subscriptions」功能：
*   讓系統在每週一早上 08:00，自動將這份「最新檢驗戰情看板」進行高畫質截圖，並轉成 PDF 檔案，定時寄送到主管的 Email 或發送至團隊的 Line/Slack 群組中。這才是發揮自動化系統最大商業價值的時刻。
