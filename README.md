# SQL skills practice challenge.

<p align="left"> <!-- LeetCode badge --> <a href="https://leetcode.com/"><img src="https://img.shields.io/badge/LeetCode-SQL%20Practice-orange?style=flat&logo=leetcode" alt="LeetCode Badge"></a> <!-- GitHub badge --> <a href="https://github.com/"><img src="https://img.shields.io/badge/GitHub-Active%20Repo-black?style=flat&logo=github" alt="GitHub Badge"></a> <!-- SQL Style Guide badge --> <a href="https://www.sqlstyle.guide/"><img src="https://img.shields.io/badge/SQL%20Style%20Guide-Consistent%20Formatting-blue?style=flat&logo=postgresql" alt="SQL Style Guide Badge"></a> </p>

This repository contains SQL exercises to practice common table expressions (CTEs), Window functions, joins, etc. 

[185. Department Top Three Salaries](https://leetcode.com/problems/department-top-three-salaries/description/)
### Intuition
We need to find the 3 most highest paid employees per each department. 
To achieve this, we generate a salary ranking per department using window functions. Also, according with the statment, there are no employees with the exact name, salary and department.

### Approach
1. Add the depertment name to the employee table by joinining the `Employee` and `Department` tables
2. Use the `DENSE_RANK()` function partitioned by `Department.name` and ordered by `Employee.salary` descending.
3. Select department, employee, and salary, filtering only ranks ≤ 3.

<details>
  <summary>Code</summary>

  ```sql
WITH ranking_salaries_by_department AS (
    SELECT 
        E.id, 
        E.name as Employee, 
        E.salary, 
        D.name as Department, 
        DENSE_RANK() OVER(PARTITION BY D.name ORDER BY E.salary DESC) as rnk
    FROM Employee E
    LEFT JOIN Department D ON E.departmentId = D.id
)
SELECT 
    Department, 
    Employee, 
    Salary
FROM ranking_salaries_by_department
WHERE rnk <=3
```
</details>

Note:
`DENSE_RANK()` ensures that tied salaries share the same rank and the next rank increments sequentially without gaps—ideal for “Top-N per group” queries.

[550. Game Play Analysis IV](https://leetcode.com/problems/game-play-analysis-iv/description/)

### Intuition
We need the fraction of players who logged in on the day immediately after their first-ever login date.

$\text{fraction}=
\frac{\text{players who logged in the day after their}}{\text{total players}}$
	​

To accomplish this, we can get the initial login date for each player. Then, get the total number of players who logged in the day after the initial logged in. Once we have the result, we can divide this value by the total number of players.

### Approach

1. Use a CTE to get the initital login date for every player.
2. Use another CTE to count players who logged in exactly one day after that date.
3. Divide the result by the total number of players, rounding to two decimals.



<details>
  <summary>Code</summary>

  ```sql
WITH initial_login_game AS (
    SELECT
        player_id,
        MIN(event_date) AS initial_login
        FROM Activity
    GROUP BY player_id
),
day_after_first_logging AS (
    SELECT
        count(ilg.player_id) AS count_logins
        FROM initial_login_game ilg
        JOIN Activity A ON ilg.player_id = A.player_id
    AND ilg.initial_login = DATE_SUB(A.event_date, INTERVAL 1 DAY)
)
SELECT ROUND(dafl.count_logins/(SELECT COUNT(player_id) FROM initial_login_game), 2) AS fraction 
FROM day_after_first_logging dafl
  ```
</details>


[178. Rank Scores](https://leetcode.com/problems/rank-scores/description/)

### Intuition
We must rank scores in descending order. The ranking must satisfy:

1. Equal scores share the same rank.
2. The next rank increments without gaps.

This exactly matches the behavior of `DENSE_RANK()` function.

### Approach
1. Select the scores and apply `DENSE_RANK()` over an window ordered by score in descending order.
2. Return the rank for each score.

<details>
  <summary>Code</summary>

  ```sql
 SELECT 
    score, DENSE_RANK() OVER(ORDER BY score DESC) AS `rank` 
FROM Scores

  ```
</details>


[584. Find Customer referee](https://leetcode.com/problems/find-customer-referee/description/)

### Intuition
Get the customer names that are referred by any customer with `id <> 2` or not referred by anyone. 
### Approach
1. Get the name of the customer and filter out those records with referees distinct from two or referees null.
<details>
  <summary>Code</summary>

  ```sql
 SELECT 
    name
FROM Customer
WHERE referee_id <> 2 OR referee_id IS NULL
  ```
</details>


[196. Delete Duplicate Emails](https://leetcode.com/problems/delete-duplicate-emails/description/)

### Intuition
We need to write a `DELETE` statement to delete the duplicates by email and leave the records with the **smallest** id. A window function can identify duplicates efficiently.

### Approach

1. Get the duplicates records in a CTE by using `ROW_NUMBER()` partitioned by email and order the result by id in an ascending way.

2. Write a `DELETE` statement and filter by id where they belong to the CTE calculated in step 1.

<details>
  <summary>Code</summary>

  ```sql
 WITH emails_numbered AS (
    SELECT 
        id,
        ROW_NUMBER() OVER(PARTITION BY email ORDER BY id ASC) as rw_num
    FROM Person 
)
DELETE FROM Person
WHERE id IN (SELECT id FROM emails_numbered WHERE rw_num>1)
  ```
</details>


[Population Density](https://platform.stratascratch.com/coding/10368-population-density?code_type=3)

### Intuition
We need to get the population density of the cities rounded to the nearest integer and get the cities with the minimum and maximum densities. We can leverage on window functions to get top density cities.
### Approach
1. Use a CTE to get the population density for each city and consider filtering those cities with null or zero area.
2. Get the minimum and maximum densities cities by using `RANK()` function to consider cities with same density.
3. Return the result

<details>
  <summary>Code</summary>

  ```sql
 
WITH density_pop AS (
    SELECT 
        city, 
        country, 
        ROUND(population/area) AS density 
    FROM cities_population
    WHERE population IS NOT NULL AND area IS NOT NULL AND area <> 0)
SELECT  city, country, density FROM (
    SELECT city, country, density, RANK() OVER(ORDER BY density DESC)  as rank_n FROM density_pop
) as t1
WHERE t1.rank_n = 1
UNION ALL
SELECT  city, country, density FROM (
    SELECT city, country, density, RANK() OVER(ORDER BY density ASC)  as rank_n FROM density_pop
) as t2
WHERE t2.rank_n = 1
  ```
</details>




[570. Managers with at Least 5 Direct Reports](https://leetcode.com/problems/managers-with-at-least-5-direct-reports/description/)

### Intuition
We need to return the manager names that have at least 5 direct reports. We can consider counting the number of employees that have the managers and filter those managers with more than or 5 employees.

### Approach
1. Use a CTE to get the counts of employees that have assigned every manager.
2. Filter employees with no managers.
3. Use `INNER JOIN` with the `Employee` table to get the manager name and return null if the `managerId` does not exist.

<details>
  <summary>Code</summary>

  ```sql
 
WITH manager_counts AS (
    SELECT 
        id, 
        managerId, 
        COUNT(*) AS count_manager
    FROM Employee
    WHERE managerId IS NOT NULL
    GROUP BY managerId
)
SELECT 
    e.name
FROM manager_counts mc
INNER JOIN Employee e ON mc.managerId = e.id
WHERE mc.count_manager>=5
  ```
</details>


[608. Tree Node](https://leetcode.com/problems/tree-node/description/)

### Intuition
We need to assign to each node in the `Tree` table its corresponding type (`Root, Inner, Leaf`).

### Approach
1. Root nodes:
A node is a root if its `p_id` is NULL.
Collect these in the first CTE.
2. Leaf nodes:
* Exclude any root nodes 
* Implement a `LEFT ANTI JOIN` to get the nodes with no children.
3. Inner nodes:
* Exclude any root nodes 
* Appear as a parent to some other node.
We can extract them by selecting distinct p_id values that are not roots.
4. Combine the previous results using `UNION ALL`.

<details>
  <summary>Code</summary>

  ```sql
 
WITH root_nodes AS (
    SELECT
        id,
        "Root" as type
    FROM Tree
    WHERE p_id is NULL
),
leaf_nodes AS (
    SELECT
        T1.id,
        "Leaf" as type
    FROM Tree T1
    LEFT JOIN Tree T2 ON T1.id = T2.p_id
    WHERE T2.p_id IS NULL AND T1.id NOT IN (SELECT id FROM root_nodes)
),
inner_nodes AS (
    SELECT DISTINCT
        p_id,
        "Inner" as type
    FROM Tree
    WHERE p_id NOT IN (SELECT id FROM root_nodes)
)
SELECT * FROM root_nodes 
UNION ALL 
SELECT * FROM leaf_nodes
UNION ALL
SELECT * FROM inner_nodes
  ```
</details>
