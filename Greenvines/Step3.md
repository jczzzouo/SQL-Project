## 假設2: 首購品項與「通路」有關
---
在前面的分析中，我們確定了顧客的購買行為會顯著影響到新客的成長率與回購率，而為了繼續尋找其他可能的影響因素，我們將檢驗「銷售通路」是否會影響新客的首購行為，因顧客可能會根據方便性、價格、服務、用戶體驗等多種因素進而選擇不同的購買管道，故分析「銷售通路」可以進一步了解顧客回購的主要因素。

目前，綠藤生機的通路分成：線上官網電商、線下的實體門市。而本次分析將聚焦於兩點進行觀察：

1. 官網和門市在 2021 與 2022 年的新客回購數量與回購率：透過「銷售通路」先進行細分，並分別計算計算新客數、回購數、回購率以及其成長率，以檢驗不同通路的購買行為是否有顯著差異。
2. 首購購買行為在門市與官網的回購數量和回購率：延續上一點的觀察，再以首購購買行為進行細分，並分別計算新客數、回購數、回購率以及其成長率，以檢視首購購買行為在不同通路中是否有顯著差異。

   ## **驗證首購品項與「通路」有關**

## 問題描述

在前面的分析中，我們確定了顧客的購買行為會顯著影響到新客的成長率與回購率，而為了繼續尋找其他可能的影響因素，我們將檢驗「銷售通路」是否會影響新客的首購行為，因顧客可能會根據方便性、價格、服務、用戶體驗等多種因素進而選擇不同的購買管道，故分析「銷售通路」可以進一步了解顧客回購的主要因素。

目前，綠藤生機的通路分成：線上官網電商、線下的實體門市。而本次分析將聚焦於兩點進行觀察：

1. 官網和門市在 2021 與 2022 年的新客回購數量與回購率：透過「銷售通路」先進行細分，並分別計算計算新客數、回購數、回購率以及其成長率，以檢驗不同通路的購買行為是否有顯著差異。
2. 首購購買行為在門市與官網的回購數量和回購率：延續上一點的觀察，再以首購購買行為進行細分，並分別計算新客數、回購數、回購率以及其成長率，以檢視首購購買行為在不同通路中是否有顯著差異。

## 資料與分析

1. 官網和門市在 2021 與 2022 年的新客回購數量與回購率：
    
    分析的結果顯示，官網的新客數成長了17%，但回購數僅成長12.2%，其回購率微幅下滑0.7%，而門市的新客數成長14%，但回購數卻成長了43%，是官網的3.5倍左右，其回購率也從27.2%顯著的上升至34%。
    
    ![image](https://github.com/jczzzouo/SQL-Project/assets/166914300/e676f205-209e-4c17-8c3a-48883b1359d8)

      ```sql
      SWITH orders_customers AS (
        SELECT 
          od.OrderId,
          o.CustomerId,
          ProductType,
          TransactionDate,
          TransactionYear,
          FirstTransactionDate,
          FirstTransactionYear,
          CASE 
            WHEN Channel = '官網' THEN Channel 
            ELSE '門市' 
          END AS Channel,
          CASE 
            WHEN FirstChannel = '官網' THEN FirstChannel 
            ELSE '門市' 
          END AS FirstChannel
        FROM Orders o 
        LEFT JOIN Customers c ON c.CustomerId = o.CustomerId
        RIGHT JOIN OrderDetails od ON od.OrderId = o.OrderId
        LEFT JOIN Products p ON p.ProductId = od.ProductId
      ),
      new_customers AS (
        SELECT
          '總數' AS Channel,
          COUNT(DISTINCT CASE WHEN FirstTransactionYear = 2021 THEN CustomerId ELSE NULL END) AS `2021`,
          COUNT(DISTINCT CASE WHEN FirstTransactionYear = 2022 THEN CustomerId ELSE NULL END) AS `2022`
        FROM orders_customers
        WHERE TransactionDate = FirstTransactionDate
      ),
      channel_new_customers AS (
        SELECT
          Channel,
          COUNT(DISTINCT CASE WHEN FirstTransactionYear = 2021 THEN CustomerId ELSE NULL END) AS 新客數_2021,
          COUNT(DISTINCT CASE WHEN FirstTransactionYear = 2022 THEN CustomerId ELSE NULL END) AS 新客數_2022
        FROM orders_customers
        WHERE TransactionDate = FirstTransactionDate
        GROUP BY Channel
      ),
      channel_new_rebuy_customers AS (
        SELECT
          FirstChannel,
          COUNT(DISTINCT CASE WHEN TransactionYear = 2021 THEN OrderID END) AS 新客回購數_2021,
          COUNT(DISTINCT CASE WHEN TransactionYear = 2022 THEN OrderID END) AS 新客回購數_2022
        FROM orders_customers
        WHERE TransactionDate <> FirstTransactionDate
          AND TransactionYear = FirstTransactionYear
        GROUP BY FirstChannel
      )
      SELECT 
        cn.Channel,
        新客數_2021,
        新客回購數_2021,
        新客數_2022,
        新客回購數_2022
      FROM channel_new_rebuy_customers cnr
      LEFT JOIN channel_new_customers cn ON cnr.FirstChannel = cn.Channel;
      
      ```
    

1. 首購購買行為在門市與官網的回購數和回購率
    - 官網：根據結果顯示，官網的新客數整體成長17%，主要是受到「只買帶路產品」的新客所帶動，其成長率高達33.2%，而「核心、帶路產品都買」、「只買其他產品」也貢獻了一定的成長率。而回購數的部分，雖然「只購買核心產品」的成長率高達41.2%，但其「只買帶路產品」以及「只買其他產品」的回購數衰退(分別為-26.3%和-14%)對總體回購數的提升造成了拖累，使整體成長率僅達到12.2%。
    - 門市：根據結果顯示，門市的新客數整體成長14%，同樣是受到「只買帶路產品」的新客所帶動，其成長率達到24%，而其他的購買行為也貢獻了一定的成長率。而回購數的部分較官網不同，「只購買核心產品」的成長率高達54.5%，而「只買帶路產品」、「核心、帶路產品都買」以及「只買其他產品」的回購數呈現正面成長，使整體的成長率達到43%，為官網的3倍以上，同時也帶動回購率上升了6.9%。
    
      ![image](https://github.com/jczzzouo/SQL-Project/assets/166914300/c2357c79-2b09-4f9d-ba31-807c24c8ce78)
   
      ![image](https://github.com/jczzzouo/SQL-Project/assets/166914300/7917b22b-4bd3-4bfd-a5fe-8e175f5de639)

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
    new_customers AS (
      SELECT
        '總數' AS PurchaseGroup,
        COUNT(DISTINCT CASE WHEN FirstTransactionYear = 2021 THEN OrderId ELSE NULL END) AS `2021`,
        COUNT(DISTINCT CASE WHEN FirstTransactionYear = 2022 THEN OrderId ELSE NULL END) AS `2022` 
      FROM orders_customers
      WHERE TransactionDate = FirstTransactionDate
    ),
    purchase_group AS (
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
      GROUP BY CustomerId, FirstTransactionYear
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
    )
    SELECT * 
    FROM new_group
    ORDER BY CASE PurchaseGroup 
      WHEN '只買核心產品' THEN 1
      WHEN '只買帶路產品' THEN 2
      WHEN '核心、帶路產品都買' THEN 3
      WHEN '只買其他產品' THEN 4
      ELSE 5
    END;
    
    ```
    

## 結論

綜上所述，官網的新客雖然在「只購買核心產品」的類別中有很高的回購成長率，但「只購買帶路產品」和「只購買其他產品」的類別卻顯示出回購數的衰減，這可能意味著官網顧客的回購動機更多地與核心產品有關，而非對品牌的全面接受。

這種現象可能暗示了官網相對於實體店面，在顧客體驗方面存在差距。可能是因為在官網購物缺少了如實體店面般的即時互動與體驗，例如產品試用或即刻的顧客服務，這些都是促進顧客滿意度和回購意願的關鍵要素。

對比之下，門市的顧客不僅在「只購買核心產品」的類別中回購率高，即便是在其他類別的產品中也顯示出正面的成長。這可能反映了門市能夠提供一個更全面的品牌體驗，從而增強顧客對品牌的整體認同感，進而提升了回購率。
