# 假設2: 「學習成效」是否遇到挑戰
在上一個分析中，我們發現了學習目標對於學生的投入程度有重要的影響，其中Career Switcher學生表現出較高的學習投入，而Tourist則相反。這很有可能是因為原有的教學設計無法同時有效幫助這兩個類型的學生，這種現象可能暗示目前的教學設計並不足以滿足這兩類學生的需求，進而影響了他們投入學習的意願。

為深入理解這一問題，本次分析將集中於以下三個關鍵問題：

1. 相比於 Career Switcher，Tourist 學生的作業繳交率是否真的比較低落？：透過比較Career Switcher與Tourist學生的作業繳交狀況，以評估作業繳交是否真的出現差異。
2. Tourist 學生在哪些類型的作業遇到了困難？：分析Tourist學生在「觀念型」與「實作型」作業的繳交狀況，以評估學生是在哪個類型的作業遇到困難。
3. Tourist 學生在那些難度的作業遇到了困難？：分析Tourist學生在「實務挑戰型」、「綜合應用型」以及「隨堂練習」作業的繳交狀況，以評估學生是在哪個難度的作業遇到困難。

透過這些問題的分析，我們期望能夠更深入地了解不同學習目標的學生在學習過程中的所遇到困難與挑戰，從而對教學、資源設計提供更具針對性的建議。

### **資料與分析**

1. 相比於 Career Switcher，Tourist 學生的作業繳交率是否真的比較低落？
    - 分析方法：先將學生以Career Switcher、Tourist進行分群，並分別計算兩種學生的應繳作業數、實際繳交數以及作業繳交率，以檢驗兩種學生的作業繳交狀況是否出現差異。
    - 分析結果：從分析的結果來看，學習目標對於學生的作業繳交表現有顯著的影響。以職業轉換為目標的學生（Career Switcher），他們的作業繳交率高達82%，是Tourist的2倍以上，這可能是由於Career Switcher相較於Tourist，他們對於達到一定的學習成果或職業目標等有更加明確的動機。
        
        
        | 學習目標 | 實際繳交量 | 應該繳交量 | 繳交率 |
        | --- | --- | --- | --- |
        | Career Switcher | 4932 | 6045 | 82% |
        | Tourist | 3726 | 9750 | 38% |
        
        ```sql
        WITH class_goal AS (
            SELECT 
                student_id, 
                CASE 
                    WHEN class IN ('7月班', '8月班', '9月班') THEN 'total_7_9'
                    ELSE 'total_10_12'
                END AS class_group,
                learning_goals
            FROM students s
            LEFT JOIN demographic d ON d.student_id = s.id
        ), tc_1012 AS (
            SELECT 
                * 
            FROM class_goal cg
            RIGHT JOIN submissions s ON cg.student_id = s.student_id
            WHERE learning_goals <> 'Learner' AND class_group = 'total_10_12'
        )
        
        SELECT 
            learning_goals AS 學習目標,
            COUNT(student_id) AS 實際繳交量,
            COUNT(DISTINCT student_id) * (SELECT COUNT(id) FROM assignments) AS 應該繳交量,
            ROUND(CAST(COUNT(student_id) AS FLOAT) / (COUNT(DISTINCT student_id) * (SELECT COUNT(id) FROM assignments)), 2) AS 作業繳交率
        FROM tc_1012
        GROUP BY learning_goals;
        
        ```
        
2. Tourist 學生在哪些類型的作業遇到了困難？
    - 分析方法：首先依作業的類型進行分群，並計算每個作業類型的應繳作業數、實際繳交數以及作業繳交率，以檢驗不同的作業類型繳交狀況是否有顯著差異。
    - 分析結果：從分析結果來看，實作類型的作業繳交率為34%，而觀念類型的繳交率較高，達到了45%。這可能表明了相較於觀念類型的作業，Tourist學生在實作類型的作業遇到較大挑戰的困難。
        
        
        | 作業類型 | 實際繳交數 | 應繳交份數 | 作業繳交率 |
        | --- | --- | --- | --- |
        | 實作 | 2879 | 8550 | 34% |
        | 觀念 | 5779 | 12825 | 45% |
        
        ```sql
        WITH class_goal AS (
            SELECT 
                student_id, 
                CASE 
                    WHEN class IN ('7月班', '8月班', '9月班') THEN 'total_7_9'
                    ELSE 'total_10_12'
                END AS class_group,
                learning_goals
            FROM students s
            LEFT JOIN demographic d ON d.student_id = s.id
        ), tc_1012 AS (
            SELECT 
                cg.*,
                s.student_id AS submission_student_id,
                a.type,
                a.id AS assignment_id
            FROM class_goal cg
            RIGHT JOIN submissions s ON cg.student_id = s.student_id
            LEFT JOIN assignments a ON a.id = s.assignment_id
            WHERE learning_goals <> 'Learner' AND class_group = 'total_10_12'
        ), type_count AS (
            SELECT
                type, 
                COUNT(id) AS total_assignment
            FROM assignments
            GROUP BY type
        ), type_should AS (
            SELECT
                tc.type,
                tc.total_assignment * (
                    SELECT COUNT(student_id) 
                    FROM class_goal 
                    WHERE class_group = 'total_10_12'
                ) AS should_submission
            FROM type_count tc
        )
        
        SELECT
            tc.type AS 作業類型,
            COUNT(tc.submission_student_id) AS 實際繳交數,
            ts.should_submission AS 應繳交份數,
            ROUND(COUNT(tc.submission_student_id) / CAST(ts.should_submission AS FLOAT), 2) AS 作業繳交率
        FROM tc_1012 tc
        LEFT JOIN type_should ts ON tc.type = ts.type
        GROUP BY tc.type, ts.should_submission;
        
        ```
        
    
3. Tourist 學生在那些難度的作業遇到了困難？
    - 分析方法：首先作業的難度進行分群，並計算每個作業類型的應繳作業數，以檢驗不同的作業類型繳交狀況是否有顯著差異。
    - 分析結果：從分析結果來看，不同作業難度的繳交率顯示了學生面對不同挑戰時的行為差異。尤其是實務挑戰題的作業，其繳交率僅有8%，顯示了學生在面對高難度的實際應用問題時可能感到特別困難，導致繳交率低落。相比之下，綜合應用類作業的繳交率為37%，雖然比實務挑戰題高，但仍低於隨堂練習的60%，這表示學生在面對中等難度的作業時有較好的完成度，但仍有一定的挑戰。
        
        
        | 作業難度 | 實際繳交數 | 應繳交份數 | 作業繳交率 |
        | --- | --- | --- | --- |
        | 實務挑戰題 | 350 | 4275 | 8% |
        | 綜合應用 | 3146 | 8550 | 37% |
        | 隨堂練習 | 5162 | 8550 | 60% |
        
        ```sql
        WITH class_goal AS (
            SELECT 
                student_id, 
                CASE 
                    WHEN class IN ('7月班', '8月班', '9月班') THEN 'total_7_9'
                    ELSE 'total_10_12'
                END AS class_group,
                learning_goals
            FROM students s
            LEFT JOIN demographic d ON d.student_id = s.id
        ), tc_1012 AS (
            SELECT 
                cg.*,
                s.student_id AS submission_student_id,
                a.difficulty,
                a.id AS assignment_id
            FROM class_goal cg
            RIGHT JOIN submissions s ON cg.student_id = s.student_id
            LEFT JOIN assignments a ON a.id = s.assignment_id
            WHERE cg.learning_goals <> 'Learner' AND cg.class_group = 'total_10_12'
        ), difficulty_count AS (
            SELECT
                difficulty, 
                COUNT(id) AS total_assignment
            FROM assignments
            GROUP BY difficulty
        ), difficulty_should AS (
            SELECT
                dc.difficulty,
                dc.total_assignment * (
                    SELECT COUNT(student_id) 
                    FROM class_goal 
                    WHERE class_group = 'total_10_12'
                ) AS should_submission
            FROM difficulty_count dc
        )
        
        SELECT
            tc.difficulty AS 作業難度,
            COUNT(tc.submission_student_id) AS 實際繳交數,
            ds.should_submission AS 應繳交份數,
            ROUND(COUNT(tc.submission_student_id) / CAST(ds.should_submission AS FLOAT), 2) AS 作業繳交率
        FROM tc_1012 tc
        LEFT JOIN difficulty_should ds ON tc.difficulty = ds.difficulty
        GROUP BY tc.difficulty, ds.should_submission;
        
        ```
        
    

### 結論

透過對學生的學習目標、作業類型和作業難度的分析，我們發現學生的學習行為和作業繳交情況與其學習目標緊密相關。Career Switcher 學生展現出較高的作業繳交率（82%），顯示他們對於完成學習任務有較強的動機，這可能是由於他們對於達成轉職目標有明確的期望。相反，以Tourist學生表現出較低的作業繳交率（38%），這可能反映出他們對於課程內容的興趣或學習動機不如轉職者積極。

在作業類型方面，Tourist學生在實作類型的作業上遇到更大挑戰，繳交率僅為34%，而觀念類型的作業繳交率則相對較高，達到45%。這表明比起理論知識，這些學生在實際應用和技能的實踐上可能需要更多的支援和指導。

當考慮到作業的難度，所有學生在面對實務挑戰題時，繳交率普遍較低，特別是對於Tourist學生，他們在這類高難度作業的完成上顯得格外掙扎。綜合應用類作業的繳交率較實務挑戰題高，但仍低於隨堂練習，顯示當作業難度增加時，學生的繳交意願和能力均有所下降。

總體而言，學習目標的不同顯著影響了學生對學習的投入程度和作業完成情況。我們可能需要根據學生的學習目標和需求，以提供不同程度教材和資源，特別是需要更多實踐與指導的Tourist學生，以及在設計高難度作業時要更多加考慮學生的接受能力和學習需求。

### 延伸

除了已經分析的學習目標、作業類型和難度外，我們還可以進一步探討以下幾個方面，以獲得更全面的理解：

1. **上線學習次數**：探索學生參與在線學習平台的頻率，以及這是否與他們的學習成效有關。這可以幫助我們理解定期參與與學習成效之間的關係。
2. **平均學習時長**：分析學生在平台上的平均學習時長，尤其是與特定作業或測試相關的學習時長，這有助於我們了解學生投入學習的深度與廣度，以及這些因素如何影響他們的學習成果。

透過對這些額外維度的分析，我們可以進一步深化對學生學習行為和成效的理解，並根據這些見解來優化課程設計和學習支援服務。
