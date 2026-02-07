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

[626. Exchange Seats](https://leetcode.com/problems/exchange-seats/description/)

### Intuition

We need to swap the seat IDs for every **two consecutive students**. If the total number of students is odd, the last student keeps their original seat ID. 

To achieve this, we can create a new swapped seat ID for each row based on whether the current seat ID is even or odd, while handling the edge case where the last ID is odd.

### Approach
This is one approach to solving the problem:
1. Create three new columns to get the swap ids:
* id-1 &rarr; previous seat
* id+1 &rarr; next seat
* `MAX(id)` &rarr; used to consider the case when the last seat id is odd.
2. Use a CTE to get the swapped ID using `CASE` conditional operation :
* If the ID is even and less than `MAX(id)` column, then select the id-1 column. 
* If the ID is odd and is equal to the max_id then keep the same ID. 
* Otherwise select id+1.
3. Join with the Seat table to get the students name by the swapped ids.
4. Generate the final sequential seat IDs using `ROW_NUMBER()`.

<details>
  <summary>Code</summary>

  ```sql
 
WITH rank_student_pos AS(
    SELECT
        id,
        student,
        id-1 AS id1,
        id+1 AS id2,
        MAX(id) OVER() as max_id
    FROM Seat
),
swapped_ids AS(
SELECT 
    CASE 
        WHEN id % 2 = 0 AND id <= max_id THEN id1
        WHEN id % 2 <> 0 AND id = max_id THEN max_id
        ELSE id2
    END AS swapped_ids
 FROM rank_student_pos
)
SELECT 
    ROW_NUMBER() OVER () AS id,
    s.student
FROM swapped_ids si
LEFT JOIN Seat s ON si.swapped_ids = s.id
  ```
</details>

[1045. Customers who bought all products](https://leetcode.com/problems/customers-who-bought-all-products/description/)

### Intuition

We need to get the customers who bough all products from the `Product` table. We can count the distinct products by customer_id and compare it with the total distinct products from the `Product` table.

### Approach
This is one approach to solving the problem:
1. Group the `Customer` table by customer_id.
2. Apply `HAVING` clause to filter those customers where the count distinct of products equals the total number of products in the `Product` table

<details>
  <summary>Code</summary>

  ```sql
SELECT customer_id
FROM Customer
GROUP BY customer_id
HAVING COUNT(DISTINCT product_key) = (SELECT COUNT(*) FROM Product)

  ```
</details>


[585. Investments in 2016](https://leetcode.com/problems/investments-in-2016/description/)

### Intuition

We need to compute the total investments values in 2016 for the placeholders that meet two conditions:
1. Their `tiv_2015` value appears in more than one record
2. There are no more records with the same locations

### Approach
This is one approach to solving the problem:
1. Use a CTE from `Insurance` table to get all the repeated records by `tiv_2015` column.
2. Join the duplicated `tiv_2015` values to the original table to retain only qualifying policyholders.
3. Exclude policyholders whose (lat, lon) location appears more than once by using a `NOT EXISTS` subquery.
3. Sum and round to two digits the `tiv_2016` column.

<details>
  <summary>Code</summary>

  ```sql

WITH duplicated_tiv_2015 AS (
    SELECT tiv_2015
    FROM Insurance
    GROUP BY tiv_2015
    HAVING COUNT(*) > 1
)
SELECT
    ROUND(SUM(i.tiv_2016), 2) AS tiv_2016
FROM Insurance i
JOIN duplicated_tiv_2015 d
    ON i.tiv_2015 = d.tiv_2015
WHERE NOT EXISTS (
    SELECT 1
    FROM Insurance j
    WHERE
        j.lat = i.lat
        AND j.lon = i.lon
        AND j.pid <> i.pid
)
  ```
</details>


[602. Friend Requests II: Who Has the Most Friends](https://leetcode.com/problems/friend-requests-ii-who-has-the-most-friends/description/)

### Intuition

We need to get the user (only one) that has the most friends according to the `RequestAccepted` table. To get this we can calculate the count of accepters and requesters by ids, sum the total and get the id with the maximum sum.

### Approach
1. Use a CTE to get the `id`  and total requesters
2. Use another CTE to get the `id` and the total accepters
3. Union both results and sum the total by `id`
4. Get the id with the maximum sum

<details>
  <summary>Code</summary>

  ```sql

WITH total_requesters AS (
    SELECT
        requester_id as id,
        COUNT(*) as total
    FROM RequestAccepted
    GROUP BY requester_id
),
total_accepters AS (
    SELECT
        accepter_id as id,
        COUNT(*) as total
    FROM RequestAccepted
    GROUP BY accepter_id
),
total_friends AS(
    SELECT id, SUM(total) as total FROM(
        SELECT id, total FROM total_requesters
        UNION ALL
        SELECT id, total FROM total_accepters
    ) t1
    GROUP BY t1.id
)
SELECT id, total as num FROM total_friends
HAVING total = (SELECT MAX(total) FROM total_friends)

  ```
</details>

[1070. Product Sales Analysis III](https://leetcode.com/problems/product-sales-analysis-iii/description/)

### Intuition

We need to return for each product the earliest year in which the product appears in the `Sales` table and return all sales records for that product in that year. We can use a window function to get all the entries for each product the first year it was sold. 

### Approach
1. Use a CTE and apply the `RANk()` function partitioned by `product_id` and order by `year` in ascending order.
2. Filter the result to keep only rows where the rank equals 1.

Note:
`RANk()` Ensures that we return all the entries for a product in the given year.

<details>
  <summary>Code</summary>

  ```sql

WITH sales_rank AS (
    SELECT
        *,
        RANK() OVER(PARTITION BY product_id ORDER BY year ASC) AS rnk
    FROM Sales
)
SELECT
    product_id,
    year AS first_year,
    quantity,
    price
FROM sales_rank
WHERE rnk=1
  ```
</details>

[1164. Product Price at a Given Date](https://leetcode.com/problems/product-price-at-a-given-date/description/)

### Intuition

Given the `Products` table where each record represents a change in the product price, we need to return the price for all the products on `2019-08-16`. There is a case where a product has no change its price that day or before, in that case the initial price is 10. 
We can treat this problem into two parts and then perform an Union operation. The first part focuses on those products that have change its price on or before `2019-08-16`. In this case, we must retrieve the most recent price change prior to (or on) that date. 
The second part gets all the products that didn't belong to the previous result and assign the default price value to be 10.

### Approach
1. Use a CTE where `change_date <= 2019-08-16` and apply a `ROW_NUMBER()` partitioned by `product_id` and order by `change_date` to get the most recent change first.
2. Filter the first CTE to keep only rows where `ROW_NUMBER() = 1`.
This gives us the latest valid price per product before the target date.
3. Use another CTE to get the products that do not appear in the previous result and assign them the default price of 10.
4. Perform an `Union ALL` for both results.

<details>
  <summary>Code</summary>

  ```sql

WITH product_dates AS(
    SELECT
        product_id,
        new_price,
        change_date,
        ROW_NUMBER() OVER (PARTITION BY product_id ORDER BY change_date DESC) rw_num
    FROM Products
    WHERE change_date <= '2019-08-16'
),
filtered_product_dates AS (
    SELECT
        product_id,
        new_price AS price
    FROM product_dates
    WHERE rw_num = 1
),
remaining_products AS(
    SELECT DISTINCT
        p.product_id,
        10 AS price
    FROM Products p
    WHERE product_id NOT IN (SELECT product_id FROM filtered_product_dates)
)
SELECT * FROM filtered_product_dates
UNION ALL
SELECT * FROM remaining_products
  ```
</details>