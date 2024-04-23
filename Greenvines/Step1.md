## 檢驗回購率是否增長 
在深入了解顧客回購的因素前，我們必須先檢驗顧客的回購狀況，因此首先的步驟是要先確認今年的客戶回購成長狀況，才會進一步探討可能的因素，以下是我的分析結果以及相關程式碼：


- 分析方法：透過2021/1-4月與2022/1-4月的客戶訂單數據，計算新客數、新客回購數、新客回購率以及其成長率，以檢視綠藤生機新客成長狀況。相關指標的定義如下：
    - 新客數：顧客在綠藤生機的首購時間發生在當年度
    - 新客回購數：首購發生在當年度的顧客，同年度再次進行購買
    - 新客回購率 = 新客回購數 / 新客數
- 分析結果：分析的結果顯示，新客數與新客回購數皆有顯著的成長，其年成率分別為16.3%與22.6%，而新客回購率分別為19.9%以及21%。整體而言，雖然新客回購率並未顯著成長，但新客回購的成長高於新客數的成長，證實了我們的初步假設，即新客戶中有更高比例的人選擇再次購買綠藤生機的產品。
    
    
    | 顧客數 | 2021 | 2022 | YoY |
    | --- | --- | --- | --- |
    | 新客數 | 8284 | 9636 | 16.3% |
    | 新客回購數 | 1649 | 2021 | 22.6% |
    | 新客回購率 | 19.9% | 21.0% | 5.4% |
    
    ```sql
    WITH orders_customers AS (
      SELECT 
        OrderId,
        o.CustomerId,
        TransactionDate,
        TransactionYear,
        FirstTransactionDate,
        FirstTransactionYear,
        Channel,
        FirstChannel
      FROM Orders o 
      LEFT JOIN Customers c ON c.CustomerId = o.CustomerId
    ),
    rebuy_orders as (
    SELECT * FROM orders_customers
    WHERE TransactionDate <> FirstTransactionDate
      AND TransactionYear = FirstTransactionYear
    ),
    new_customers AS (
      SELECT
        '新客數' AS 顧客數,
        COUNT(DISTINCT CASE WHEN FirstTransactionYear = 2021 THEN CustomerId ELSE NULL END) AS `2021`,
        COUNT(DISTINCT CASE WHEN FirstTransactionYear = 2022 THEN CustomerId ELSE NULL END) AS `2022` 
      FROM orders_customers
    ),
    rebuy_customers AS (
      SELECT
        '新客回購數' AS 顧客數,
        COUNT(CASE WHEN TransactionYear = 2021 THEN OrderId ELSE NULL END) AS `2021`,
        COUNT(CASE WHEN TransactionYear = 2022 THEN OrderId ELSE NULL END) AS `2022` 
      FROM rebuy_orders
    )
    
    SELECT * FROM new_customers
    UNION ALL 
    SELECT * FROM rebuy_customers;
    ```
    

### 結論

綜上所述，綠藤生機在新客成長與新客回購行為上顯示出正向發展。具體來看，新客數的年增長率為16.3%，而新客回購數的年增長率更是達到了22.6%。這表示不僅新客戶的增加速度保持健康的增長，更重要的是，這些新客戶中有更大比例選擇再次進行購買。雖然新客回購率僅從19.9%提升至21.0%，但這樣的提升仍然指向了上升的趨勢，表明綠藤生機在提升顧客忠誠度方面取得了一定的成效。
