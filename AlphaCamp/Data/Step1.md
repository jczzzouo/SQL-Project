## Step1：檢驗兩班學生的學習投入程度是否出現顯著差異

近期，Alpha Camp團隊觀察到第四季入學的學生在學習進度上似乎落後於第三季的學生，初步分析認為這可能與疫情對學生的線上學習習慣造成的影響有關。為深入了解情況，首先需驗證第三季與第四季學生在學習上的投入程度上是否存在顯著差異。

本次分析聚焦於兩個主要問題：

1. **相較於7-9月班級，10-12月班級學生的學習時間是否顯著減少**：透過比較第三季與第四季學生的總學習時間及平均學習時間，以評估學生時間的差異。
2. **在7-9月班級與10-12月班級中，高投入學生的比例是否有顯著差異**：分析這兩個季節入學的班級中，高投入學生（定義為每月學習時間超過70小時）的數量及其佔比，以評估學生投入程度的差異。

### **資料與分析**

- **相較於7-9月班級，10-12月班級學生的學習時間是否顯著減少？**
    - 分析方法：先以學生的入學時間進行分群，並分別計算各個班級的總學習時間(total_study_time)以及平均學習時間(average_study_time)後，以檢驗不同時期的學生在學習時間上是否有顯著的差異。
    - 分析結果：從結果來看，我們可以發現7-9月相較於10-12月的學生，其總學習時間(total_study_time)高出8000~10000小時，而平均學習時間(average_study_time)則高出25~30小時左右。
 
        | 班級 | 總學習時間 | 班級總人數 | 平均每人學習時間 |
        | --- | --- | --- | --- |
        | 7月班 | 26621 | 412 | 64 |
        | 8月班 | 28787 | 436 | 66 |
        | 9月班 | 32905 | 519 | 63 |
        | 10月班 | 17346 | 444 | 39 |
        | 11月班 | 20790 | 503 | 41 |
        | 12月班 | 19524 | 478 | 40 |
       
        ```sql
        SELECT 
        	class,
            SUM(study_time)/60 total_study_time, 
            COUNT(DISTINCT student_id) num_student,  
            SUM(study_time)/60 / COUNT(DISTINCT student_id) average_study_time
        FROM sessions s1
        LEFT JOIN students s2 ON s1.student_id = s2.id
        GROUP BY 1
        order by case class
        	WHEN '7月班' THEN 1
            WHEN '8月班' THEN 2
            WHEN '9月班' THEN 3
            WHEN '10月班' THEN 4
            WHEN '11月班' THEN 5
            WHEN '12月班' THEN 6
            END;
        ```
        
- **在7-9月班級與10-12月班級中，高投入學生的比例是否有顯著差異**
    - 分析方法：首先依照學生的入學時間先進行分群，並計算每個班級「每個月學習超過70小時」的高投入學生的人數(total_high_student)以及其佔整個班級的比例(high_ratio)後，以檢驗不同時期的學生的投入狀況是否有顯著的差異。
    - 分析結果：從結果來看，我們可以發現7-9月相較於10-12月的高投入學生的數量以及佔比皆超過10倍。
        
        | 班級 | 高頭入人數 | 班級總人數 | 高投入占比 |
        | --- | --- | --- | --- |
        | 7月班 | 187 | 412 | 45% |
        | 8月班 | 194 | 436 | 44% |
        | 9月班 | 214 | 519 | 41% |
        | 10月班 | 12 | 444 | 3% |
        | 11月班 | 12 | 503 | 2% |
        | 12月班 | 11 | 478 | 2% |
        
        ```sql
        WITH invest AS (
            SELECT 
                student_id,
                SUM(study_time) / 60 AS study_time,
                CASE 
                    WHEN SUM(study_time) / 60 >= 70 THEN 'high'
                    ELSE 'low'
                END AS invest
            FROM sessions
            GROUP BY student_id
        ), total_high_student AS (
            SELECT 
                class, 
                COUNT(s.id) AS total_high_student
            FROM students s
            LEFT JOIN invest i ON i.student_id = s.id
            WHERE invest = 'high'
            GROUP BY class
        ), total_student AS (
            SELECT 
                class, 
                COUNT(id) AS total_student
            FROM students
            GROUP BY class
        )
        
        SELECT
            t1.class, 
            total_high_student, 
            total_student,
            ROUND(CAST(total_high_student AS FLOAT) / total_student, 2) AS high_ratio
        FROM total_high_student t1
        LEFT JOIN total_student t2 ON t1.class = t2.class
        ORDER BY CASE t1.class
            WHEN '7月班' THEN 1
            WHEN '8月班' THEN 2
            WHEN '9月班' THEN 3
            WHEN '10月班' THEN 4
            WHEN '11月班' THEN 5
            WHEN '12月班' THEN 6
        END;
        
        ```
        

### 結論

根據分析的結果，我們可以發現不論是在學習投入的時間或班級高投入學生的比例，都可以看見7-9月的學生明顯高於10-12月的學生，因此「學生的學習時間有大幅下降」的假設是成立的。

### 延伸

考慮到學習投入度的變化可能與多種因素相關，建議接下來的分析可以包括更多的變量，如學生的背景信息、線上學習平台的互動數據、以及學習成效指標，以深入了解疫情期間線上學習行為的變化。
