## 假設1：回購與「首購產品類型」有關
在上一步分析中，我們確認了2022/1~4月的新客回購呈現增長的狀況，為深入了解顧客回購增長的主要因素，我們將透過顧客回購的經驗(產品類型、購買通路、消費滿意度等)來進行分析。本次分析我們先以顧客回購的「**產品類型**」來了解顧客的購買狀況。而本次分析將聚焦於兩點進行觀察：

1. 2021～2022 年 1-4 月新客首購中，購買的產品的類別分佈以及其成長率
2. 4種購物行為的新客，後續的回購狀況

相關資料定義如下：

- 產品類型：
    - 核心產品：品牌主打的明星商品，公司最希望大家記住的產品。品牌的主力產品線通常獲得最多廣告資源，通常單價也較高。如：精華液、精華油。
    - 帶路產品：品牌吸引新客的商品，擁有較多的廣告曝光、宣傳，門市人員也會優先推廣，而為了降低顧客入手難度，因此單價較低。如：洗髮精、護唇油。
    - 其他：非上述兩類，其餘其他類型的產品，較少獲得廣告資源，通常為購買核心產品後的顧客會逐步嘗試其他產品。如：卸妝油。
- 購買行為：
    - 買核心產品但沒買帶路產品
    - 買帶路產品但沒買核心產品
    - 核心產品＆帶路產品都買
    - 只買其他產品
  ---
  ### 分析方式

本次分析我們將透過2021~2022年1-4月的訂單資料，將顧客首次購買的產品分為上述4種購買行為，並分別計算不同購買行為的新客戶數、新客回購數以及其年成長率，以檢視新客的購物狀況。

### 分析結果：

1. **2021～2022 年 1-4 月新客首購中，購買的產品的類別分佈以及其成長率**
    
    分析的結果顯示，2022年的新顧客總數相較於2021年成長了16.3%，其中「買核心產品但沒買帶路產品」的新客人數呈現5.6%的成長率，但在人數中的佔比卻從46%降至42%。與此同時，「買帶路產品但沒買核心產品」的新客人數則呈現顯著的成長，達到31.2%的成長率，其在人數中的比例也從32%提高到36。而「核心產品、帶路產品都買」以及「只買其他產品」的新客人數分的成長率分別為13%以及20.4%，而人數占比的部分並無太大變動。這樣的結果表明了核心產品可能因單價較高、入手較難，導致在新客的成長上較為困難，而帶路產品則因單價較低在新客的成長上較為容易。
    
    ![image](https://github.com/jczzzouo/SQL-Project/assets/166914300/ebf0251b-922f-49b3-b71f-05d5ce7adeb3)

        
      ```sql
        WITH orders_customers AS (
          SELECT 
            od.OrderId,
            o.CustomerId,
            od.ProductID,
            od.ProductName,
            ProductType,
            TransactionDate,
            TransactionYear,
            FirstTransactionDate,
            FirstTransactionYear,
            Channel,
            FirstChannel
          FROM Orders o 
          LEFT JOIN Customers c ON c.CustomerId = o.CustomerId
          RIGHT join OrderDetails od on od.OrderId = o.OrderId
          LEFT JOIN Products p ON p.ProductId = od.ProductId
        ),
        new_customers AS (
          SELECT
            '總數' AS PurchaseGroup,
            COUNT(DISTINCT CASE WHEN FirstTransactionYear = 2021 THEN OrderId ELSE NULL END) AS `2021`,
            COUNT(DISTINCT CASE WHEN FirstTransactionYear = 2022 THEN OrderId ELSE NULL END) AS `2022` 
          FROM orders_customers
          WHERE TransactionDate = FirstTransactionDate
        ),
        purchase_group as (
        SELECT 
            CustomerId,
            FirstTransactionYear,
        	CASE
        		WHEN MAX(CASE WHEN ProductType = '核心產品' THEN 1 ELSE 0 END) = 1
                     AND MAX(CASE WHEN ProductType = '帶路產品' THEN 1 ELSE 0 END) = 0 THEN '只買核心產品'
                WHEN MAX(CASE WHEN ProductType = '核心產品' THEN 1 ELSE 0 END) = 0
                     AND MAX(CASE WHEN ProductType = '帶路產品' THEN 1 ELSE 0 END) = 1 THEN '只買帶路產品'
                WHEN MAX(CASE WHEN ProductType = '核心產品' THEN 1 ELSE 0 END) = 1
                     AND MAX(CASE WHEN ProductType = '帶路產品' THEN 1 ELSE 0 END) = 1 THEN '核心、帶路產品都買'
                ELSE '只買其他產品'
            END AS PurchaseGroup
        FROM orders_customers
        WHERE TransactionDate = FirstTransactionDate
        GROUP BY CustomerId),
        new_group as(
        SELECT * FROM new_customers
        UNION  
        SELECT
        	PurchaseGroup,
            SUM(CASE WHEN FirstTransactionYear = 2021 then 1 else 0 END) as `2021`,
            SUM(CASE WHEN FirstTransactionYear = 2022 then 1 else 0 end) as `2022`
        FROM purchase_group
        GROUP BY PurchaseGroup)
        
        SELECT * FROM new_group
        ORDER BY case PurchaseGroup 
        	WHEN '只買核心產品' then 1
            WHEN '只買帶路產品' then 2
            WHEN '核心、帶路產品都買' then 3
            WHEN '只買其他產品' then 4
            ELSE 5
            END
      ```
        
    
1. **4種購物行為的新客，後續的回購狀況**
    
    分析的結果顯示，「買核心產品但沒買帶路產品」的顧客回購率從21.3%上升至29.4%，其回購的成長率高達45.9%。而「核心產品和帶路產品都買」的顧客回購率也有所增長，從26.9%提高到了28.5%，回購成長率達到了19.6%。而「只購買其他產品」的顧客回購率從21.5%小幅下滑至18.2%，其回購成長率為2.3%。然而，僅有「買核心產品但沒買帶路產品」的顧客回購率卻從15.2%下滑至10.2%，其回購呈現12%的負成長。這樣的結果表明核心產品在建立顧客忠誠度和鼓勵回購方面的重要性。核心產品的高回購率增長顯示，一旦顧客對這些高單價但質量優異的產品形成購買習慣，他們通常不會轉向其他品牌。而路產品雖然對於吸引新客戶有顯著效果，但由於顧客傾向於嘗試不同品牌的類似產品，所以這些產品的回購率不如核心產品。
    
    
    ![image](https://github.com/jczzzouo/SQL-Project/assets/166914300/03da2e5e-59d2-4c99-bccc-7b49a66cfe44)

        
   ```sql
        WITH orders_customers AS (
          SELECT 
            od.OrderId,
            o.CustomerId,
            od.ProductID,
            od.ProductName,
            ProductType,
            TransactionDate,
            TransactionYear,
            FirstTransactionDate,
            FirstTransactionYear,
            Channel,
            FirstChannel
          FROM Orders o 
          LEFT JOIN Customers c ON c.CustomerId = o.CustomerId
          RIGHT JOIN OrderDetails od ON od.OrderId = o.OrderId
          LEFT JOIN Products p ON p.ProductId = od.ProductId
        ), 
        purchase_group AS (
          SELECT 
            CustomerId,
            FirstTransactionYear,
            CASE
              WHEN MAX(CASE WHEN ProductType = '核心產品' THEN 1 ELSE 0 END) = 1 AND MAX(CASE WHEN ProductType = '帶路產品' THEN 1 ELSE 0 END) = 0 THEN '只買核心產品'
              WHEN MAX(CASE WHEN ProductType = '核心產品' THEN 1 ELSE 0 END) = 0 AND MAX(CASE WHEN ProductType = '帶路產品' THEN 1 ELSE 0 END) = 1 THEN '只買帶路產品'
              WHEN MAX(CASE WHEN ProductType = '核心產品' THEN 1 ELSE 0 END) = 1 AND MAX(CASE WHEN ProductType = '帶路產品' THEN 1 ELSE 0 END) = 1 THEN '核心、帶路產品都買'
              ELSE '只買其他產品'
            END AS PurchaseGroup
          FROM orders_customers
          WHERE TransactionDate = FirstTransactionDate
          GROUP BY CustomerId, FirstTransactionYear
        ), 
        order_customers_group AS (
          SELECT 
            oc.*,
            pg.PurchaseGroup
          FROM orders_customers oc
          LEFT JOIN purchase_group pg USING (CustomerId)
        ), 
        new_customers AS (
          SELECT
            '總數' AS PurchaseGroup,
            COUNT(DISTINCT CASE WHEN FirstTransactionYear = 2021 THEN CustomerId ELSE NULL END) AS `2021`,
            COUNT(DISTINCT CASE WHEN FirstTransactionYear = 2022 THEN CustomerId ELSE NULL END) AS `2022`
          FROM orders_customers
          WHERE TransactionDate = FirstTransactionDate
        ), 
        rebuy_customers AS (
          SELECT
            '總數' AS Purchase,
            COUNT(DISTINCT CASE WHEN TransactionYear = 2021 THEN OrderId ELSE NULL END) AS `2021`,
            COUNT(DISTINCT CASE WHEN TransactionYear = 2022 THEN OrderId ELSE NULL END) AS `2022`
          FROM orders_customers
          WHERE TransactionDate <> FirstTransactionDate
            AND TransactionYear = FirstTransactionYear
        ), 
        new_group AS (
          SELECT * FROM new_customers
          UNION  
          SELECT
            PurchaseGroup,
            SUM(CASE WHEN FirstTransactionYear = 2021 THEN 1 ELSE 0 END) AS `2021`,
            SUM(CASE WHEN FirstTransactionYear = 2022 THEN 1 ELSE 0 END) AS `2022`
          FROM purchase_group
          GROUP BY PurchaseGroup
        ), 
        rebuy_group AS (
          SELECT
            PurchaseGroup,
            COUNT(DISTINCT CASE WHEN TransactionYear = 2021 THEN OrderID END) AS `2021`,
            COUNT(DISTINCT CASE WHEN TransactionYear = 2022 THEN OrderID END) AS `2022`
          FROM order_customers_group
          WHERE TransactionDate <> FirstTransactionDate
            AND TransactionYear = FirstTransactionYear
          GROUP BY PurchaseGroup
          UNION ALL
          SELECT * FROM rebuy_customers
        )
        
        SELECT 
          rg.PurchaseGroup AS 首購通路,
          ng.`2021` AS 總計_2021,
          rg.`2021` AS 已回購_2021,
          ng.`2022` AS 總計_2022,
          rg.`2022` AS 已回購_2022
        FROM rebuy_group rg
        LEFT JOIN new_group ng USING (PurchaseGroup)
        ORDER BY CASE PurchaseGroup 
          WHEN '只買核心產品' THEN 1
          WHEN '只買帶路產品' THEN 2
          WHEN '核心、帶路產品都買' THEN 3
          WHEN '只買其他產品' THEN 4
          ELSE 5
        END;
        
   ```
        

## 結論

綜合上述的分析，我們可以發現儘管新顧客的總數有所增加，但核心產品在新客成長的貢獻相對較小，這可能是由於這些產品的單價較高和購入門檻較高所導致。相反，帶路產品由於其更低的單價和更大的市場曝光，成功吸引了更多的新客戶。至於回購方面，購買核心產品的顧客展現出較高的回購成長率，這強調了核心產品在維持顧客忠誠度和促進回購方面的關鍵作用。這可能反應了顧客在評估產品價值和品質後做出的長期購買決定。因此，我們應該著重於進一步發展和推廣核心產品，同時尋找策略來提高帶路產品的回購率，以實現新客戶的持續成長與現有顧客的忠誠度提升。
