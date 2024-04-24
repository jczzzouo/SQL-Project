## 假設1：不同背景的學生，會有不同的投入程度
---
在第一步分析中，我們已經發現10-12月的同學在學習的時間以及投入上有明顯的下滑。為深入了解實際原因，我們採用學生的組成資料來進行分析，以找出影響學生學習的關鍵因素。

## **資料與分析**

### 客觀數據

- 性別：男性、女性以及不願透露
- 學科背景：文科+商科、理論+工程科學、資料科學以及其他
- 學習目標：Tourist(體驗撰寫程式)、Learner(進修撰寫程式能力)以及Career Switcher(轉職工程師)
- 學習過幾種程式：目前資料中，最少為學過 0 種、最多有學過 5 種。

### 分析方式

透過性別、學科背景、學習目標以及學習過幾種程式等維度，計算各組的總人數、高低投入人數與佔比，以辨別哪些維度是影響學生學習的關鍵因素。

### 分析結果

**性別**：從結果來看，高低投入佔比的男女比例分布，與整體分布並無顯著的差異，表示男女性在學習的投入上並未有顯著的差異。

| 性別 | 總人數 | 性別佔比 | 高投入人數 | 高投入佔比 | 低投入人數 | 低投入佔比 |
| --- | --- | --- | --- | --- | --- | --- |
| 男 | 1299 | 47% | 302 | 48% | 997 | 46% |
| 女 | 1380 | 49% | 311 | 49% | 1069 | 49% |
| 不提供 | 113 | 4% | 17 | 3% | 96 | 4% |

```sql
WITH invest AS (
    SELECT 
        student_id,
        SUM(study_time) / 60 AS study_time_hours,
        CASE 
            WHEN SUM(study_time) / 60 >= 70 THEN 'high'
            ELSE 'low'
        END AS invest_level
    FROM sessions
    GROUP BY student_id
),
new_demographic AS (
    SELECT 
        d.*,
        i.invest_level,
        CASE 
            WHEN d.background IN ('文', '商') THEN 'Group1(文+商)'
            WHEN d.background IN ('理論科學', '工程科學') THEN 'Group2(理論+工程科學)'
            WHEN d.background = '資訊科學（資工、資管等等）' THEN 'Group3(資訊科學)'
            ELSE '其他'
        END AS background_group
    FROM demographic d
    LEFT JOIN invest i ON i.student_id = d.student_id 
),
invest_summary AS (
    SELECT 
        gender,
        COUNT(*) AS total_students,
        SUM(CASE WHEN invest_level = 'high' THEN 1 ELSE 0 END) AS high_invest_count,
        SUM(CASE WHEN invest_level = 'low' THEN 1 ELSE 0 END) AS low_invest_count
    FROM new_demographic
    GROUP BY gender
),
total_counts AS (
    SELECT 
        SUM(high_invest_count) AS total_high_invest,
        SUM(low_invest_count) AS total_low_invest
    FROM invest_summary
)

SELECT 
    gender,
    total_students,
    high_invest_count AS 高投入人數,
    ROUND(high_invest_count / CAST((SELECT total_high_invest FROM total_counts) AS FLOAT), 2) AS 高投入佔比,
    low_invest_count AS 低投入人數,
    ROUND(low_invest_count / CAST((SELECT total_low_invest FROM total_counts) AS FLOAT), 2) AS 低投入佔比
FROM invest_summary
ORDER BY CASE gender
    WHEN '男' THEN 1
    WHEN '女' THEN 2
    ELSE 3
END;
```

學科背景：從結果來看，我們可以發現在高投入的學生分佈相較於整體的學生分佈有顯著的差異，其高投入的學生更多來自於理論+工程科學背景。而低投入佔比的學生分佈則與整體學分佈生沒有顯著的差異。

| 學科背景 | 總人數 | 各學科佔比 | 高投入人數 | 高投入佔比 | 低投入人數 | 低投入佔比 |
| --- | --- | --- | --- | --- | --- | --- |
| Group1(文+商) | 1096 | 39% | 104 | 17% | 992 | 46% |
| Group2(理論+工程科學) | 1155 | 41% | 387 | 61% | 768 | 36% |
| Group3(資訊科學) | 338 | 12% | 97 | 15% | 241 | 11% |
| 其他 | 203 | 7% | 42 | 7% | 161 | 7% |

```sql
WITH invest AS (
  SELECT 
    student_id,
    SUM(study_time) / 60 AS study_time_hours,
    CASE 
      WHEN SUM(study_time) / 60 >= 70 THEN 'high'
      ELSE 'low'
    END AS invest_level
  FROM sessions
  GROUP BY student_id
),
new_demographic AS (
  SELECT *,
    CASE 
      WHEN background IN ('文', '商') THEN 'Group1(文+商)'
      WHEN background IN ('理論科學', '工程科學') THEN 'Group2(理論+工程科學)'
      WHEN background = '資訊科學（資工、資管等等）' THEN 'Group3(資訊科學)'
      ELSE '其他'
    END AS background_group
  FROM demographic d
  LEFT JOIN invest i on i.student_id = d.student_id 
), 
invest_summary AS (
    SELECT 
        background_group,
        COUNT(*) AS total_students,
        SUM(CASE WHEN invest_level = 'high' THEN 1 ELSE 0 END) AS high_invest_count,
        SUM(CASE WHEN invest_level = 'low' THEN 1 ELSE 0 END) AS low_invest_count
    FROM new_demographic
    GROUP BY background_group
), 
total_counts AS (
    SELECT
  		SUM(total_students) as total_student,
        SUM(high_invest_count) AS total_high_invest,
        SUM(low_invest_count) AS total_low_invest
    FROM invest_summary
)

SELECT 
    background_group as 學科背景,
    total_students as 總人數,
    ROUND(total_students/CAST((SELECT total_student FROM total_counts) AS FLOAT), 2) AS 各學科佔比,
    high_invest_count AS 高投入人數,
    ROUND(high_invest_count / CAST((SELECT total_high_invest FROM total_counts) AS FLOAT), 2) AS 高投入佔比,
    low_invest_count AS 低投入人數,
    ROUND(low_invest_count / CAST((SELECT total_low_invest FROM total_counts) AS FLOAT), 2) AS 低投入佔比
FROM invest_summary
```

學習目標：我們可以發現在高投入的學生分佈與整體學生分佈有顯著的差異，其主要集中於Career Swithcher，且人數是Learner與Tourist加總的3倍左右。而低投入佔比的學生分佈則較為均勻，與整體分佈無明顯的差異。

| 學習目標 | 總人數 | 各目標佔比 | 高投入人數 | 高投入佔比 | 低投入人數 | 低投入佔比 |
| --- | --- | --- | --- | --- | --- | --- |
| Career Switcher | 1091 | 39% | 472 | 75% | 619 | 29% |
| Learner | 676 | 24% | 96 | 15% | 580 | 27% |
| Tourist | 1025 | 37% | 62 | 10% | 963 | 45% |

```sql
WITH invest AS (
  SELECT 
    student_id,
    SUM(study_time) / 60 AS study_time_hours,
    CASE 
      WHEN SUM(study_time) / 60 >= 70 THEN 'high'
      ELSE 'low'
    END AS invest_level
  FROM sessions
  GROUP BY student_id
),
new_demographic AS (
  SELECT *,
    CASE 
      WHEN background IN ('文', '商') THEN 'Group1(文+商)'
      WHEN background IN ('理論科學', '工程科學') THEN 'Group2(理論+工程科學)'
      WHEN background = '資訊科學（資工、資管等等）' THEN 'Group3(資訊科學)'
      ELSE '其他'
    END AS background_group
  FROM demographic d
  LEFT JOIN invest i on i.student_id = d.student_id 
), 
invest_summary AS (
    SELECT 
        learning_goals,
        COUNT(*) AS total_students,
        SUM(CASE WHEN invest_level = 'high' THEN 1 ELSE 0 END) AS high_invest_count,
        SUM(CASE WHEN invest_level = 'low' THEN 1 ELSE 0 END) AS low_invest_count
    FROM new_demographic
    GROUP BY learning_goals
), 
total_counts AS (
    SELECT 
  		SUM(total_students) as total_student,
        SUM(high_invest_count) AS total_high_invest,
        SUM(low_invest_count) AS total_low_invest
    FROM invest_summary
)

SELECT 
    learning_goals as 學習目標,
    total_students as 總人數,
    ROUND(total_students/CAST((SELECT total_student FROM total_counts) AS FLOAT), 2) AS 各目標佔比,
    high_invest_count AS 高投入人數,
    ROUND(high_invest_count / CAST((SELECT total_high_invest FROM total_counts) AS FLOAT), 2) AS 高投入佔比,
    low_invest_count AS 低投入人數,
    ROUND(low_invest_count / CAST((SELECT total_low_invest FROM total_counts) AS FLOAT), 2) AS 低投入佔比
FROM invest_summary
```

學習過幾種程式語言：從結果來看，我們觀察到高投入佔比的學生分佈與整體學生分佈有顯著差異，其人數更加集中於學習過2~3種程式的人，而低投入學生的分佈則與整體學生並無顯著的差異，這一結果表明，有一定程式語言學習經驗的學生傾向於在學習上投入更多的時間與努力。

| 學過幾種程式 | 總人數 | 各數量佔比 | 高投入人數 | 高投入佔比 | 低投入人數 | 低投入佔比 |
| --- | --- | --- | --- | --- | --- | --- |
| 0 | 1129 | 40% | 10 | 2% | 1119 | 52% |
| 1 | 841 | 30% | 77 | 12% | 764 | 35% |
| 2 | 454 | 16% | 214 | 34% | 240 | 11% |
| 3 | 253 | 9% | 214 | 34% | 39 | 2% |
| 4 | 101 | 4% | 101 | 16% | 0 | 0% |
| 5 | 14 | 1% | 14 | 2% | 0 | 0% |

```sql
WITH invest AS (
  SELECT 
    student_id,
    SUM(study_time) / 60 AS study_time_hours,
    CASE 
      WHEN SUM(study_time) / 60 >= 70 THEN 'high'
      ELSE 'low'
    END AS invest_level
  FROM sessions
  GROUP BY student_id
),
new_demographic AS (
  SELECT *
  FROM demographic d
  LEFT JOIN invest i on i.student_id = d.student_id 
), 
invest_summary AS (
    SELECT 
        num_learned,
        COUNT(*) AS total_students,
        SUM(CASE WHEN invest_level = 'high' THEN 1 ELSE 0 END) AS high_invest_count,
        SUM(CASE WHEN invest_level = 'low' THEN 1 ELSE 0 END) AS low_invest_count
    FROM new_demographic
    GROUP BY num_learned
), 
total_counts AS (
    SELECT
  		SUM(total_students) as total_student,
        SUM(high_invest_count) AS total_high_invest,
        SUM(low_invest_count) AS total_low_invest
    FROM invest_summary
)

SELECT 
    num_learned 學過幾種程式,
    total_students as 總人數,
    ROUND(total_students/CAST((SELECT total_student FROM total_counts) AS FLOAT), 2) AS 各數量佔比,
    high_invest_count AS 高投入人數,
    ROUND(high_invest_count / CAST((SELECT total_high_invest FROM total_counts) AS FLOAT), 2) AS 高投入佔比,
    low_invest_count AS 低投入人數,
    ROUND(low_invest_count / CAST((SELECT total_low_invest FROM total_counts) AS FLOAT), 2) AS 低投入佔比
FROM invest_summary
```

學習目標(7-9月 vs 10-12月)：在上面的分析中，我們認為學習目標對於學生的投入程度有著重要的影響，認為它可能是近期學習時間下降的主要原因。為了探究這一假設，我們比較了7-9月和10-12月兩個時期中，不同學習目標的學生的分佈狀況。分析結果顯示，7-9月班級的學生主要來自於有轉職目標的群體（Career Switcher），而10-12月班級則主要為對程式有興趣但目標較為體驗性質的旅行者（Tourist）。這一發現支持了我們的假設，即由於10-12月班級的學生大多來自於低投入的Tourist，進而導致班級整體的投入程度下滑。

| 學習目標 | 7-9月班 | 7-9月佔比 | 10-12月班 | 10-12月佔比 |
| --- | --- | --- | --- | --- |
| Career Switcher | 445 | 75% | 27 | 77% |
| Learner | 94 | 16% | 2 | 6% |
| Tourist | 56 | 9% | 6 | 17% |
| 總和 | 595 |  | 35 |  |

```sql
WITH class_goal AS (
    SELECT 
        id, 
        CASE 
            WHEN class IN ('7月班', '8月班', '9月班') THEN 'total_7_9'
            ELSE 'total_10_12'
        END AS class_group,
        learning_goals
    FROM students s
    LEFT JOIN demographic d ON d.student_id = s.id
), goals_each_class AS (
    SELECT 
        learning_goals,
        SUM(CASE WHEN class_group = 'total_7_9' THEN 1 ELSE 0 END) AS total_7_9,
        SUM(CASE WHEN class_group = 'total_10_12' THEN 1 ELSE 0 END) AS total_10_12  
    FROM class_goal
    GROUP BY learning_goals
), total_student AS (
    SELECT
        SUM(total_7_9) AS total_7_9,
        SUM(total_10_12) AS total_10_12
    FROM goals_each_class
)

SELECT
    learning_goals AS 學習目標,
    total_7_9 AS `7-9月班`,
    ROUND(CAST(total_7_9 AS FLOAT) / (SELECT total_7_9 FROM total_student), 2) AS `7-9月佔比`,
    total_10_12 AS `10-12月班`, 
    ROUND(CAST(total_10_12 AS FLOAT) / (SELECT total_10_12 FROM total_student), 2) AS `10-12月佔比`
FROM goals_each_class;

```

## 結論

透過對學生組成數據的詳細分析，我們觀察到學習目標對學生的學習投入具有重大影響。特別是目標為轉職（Career Switcher）的學生，相比於其他學習目標的學生，展現出更高的學習投入度。儘管學科背景和已學程式語言的數量也對學習投入程度有所影響，但班級的學生組成變化尤為顯著。從7-9月至10-12月班級，我們觀察到了班級主要群體從高投入的職業轉換者，改變為對程式撰寫抱有興趣但學習投入較低的旅行者（Tourist）。這一轉變更加強調了學習動機與學習投入度之間的密切關聯。

## 延伸

考慮到目前手邊的學生組成數據欄位，除了已經分析過的性別、學科背景、學習目標和已學程式數量之外，我們還可以探索以下新的假設和推演方向：

1. **年齡的影響**：學生的年齡可能影響他們的學習方式、時間分配、技術熟悉度以及對某些學習內容的興趣。我們可以分析不同年齡段學生在學習投入、進步速度和成果上是否存在顯著差異。
2. **地區差異**：學生的居住地區可能反映出不同的文化、教育資源等。這可能影響他們的學習體驗和成效。分析不同地區學生的表現，可以幫助我們理解地理位置對學習的潛在影響。
3. **語言學習經驗**：如果學生以前學過的語言（不限於任何程式語言），這可能反映他們的學習習慣和能力，尤其是在學習新的程式語言時。我們可以探索語言學習經歷是否對學習程式有幫助，以及這種經歷與學習成果之間的關係。

通過探索這些新的維度，我們可以更全面地了解影響學生學習效果的因素，從而為他們提供更適合的支持和資源，同時也為課程設計和教學方法提供更有針對性的建議。
