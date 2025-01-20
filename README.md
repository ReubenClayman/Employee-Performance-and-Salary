# Employee Performance and Salary Insights

**Project Overview**
This project analyses employee performance, department-level salary distributions, and organizational trends. The dataset consists of three tables:

`employees:` Information about employees (e.g., employee_id, name, department_id, hire_date, salary).

`departments:` Information about company departments (e.g., department_id, department_name).

`performance:` Employee performance reviews (e.g., employee_id, review_date, performance_score).


## Business Questions
- Which employees are top performers in their departments
- What are the average salary distributions across departments?
- How does salary relate to performance score?
- Who are the most recently hired employees in each department?
- What is the cumulative salary expenditure over time?

 ## 1. Join Employees and Departments
Purpose: To create a complete dataset showing employee details along with department information.

``sql
SELECT e.employee_id, e.employee_name, d.department_name, e.hire_date, e.salary
FROM employees e
JOIN departments d ON e.department_id = d.department_id;

``

## 2. Rank Employees Based on Performance Score
Purpose: Identify top performers in each department and ranking employees by performance within their departments.

```sql
WITH ranked_performance AS (
    SELECT e.employee_id, e.employee_name, d.department_name, p.performance_score,
           RANK() OVER (PARTITION BY e.department_id ORDER BY p.performance_score DESC) AS rank
    FROM employees e
    JOIN performance p ON e.employee_id = p.employee_id
    JOIN departments d ON e.department_id = d.department_id
)
SELECT * 
FROM ranked_performance
WHERE rank = 1;
```

## 3. Calculate Average Salary per Department
Purpose: Determine how salaries are distributed across departments. Helping identify disparities or patterns in pay.

``` sql
SELECT d.department_name, AVG(e.salary) AS avg_salary
FROM employees e
JOIN departments d ON e.department_id = d.department_id
GROUP BY d.department_name
ORDER BY avg_salary DESC;
```

## 4. Analyze Salary and Performance Correlation
Purpose: Compare salaries and performance scores across employees and assess whether higher salaries correlate with better performance.

```sql
SELECT e.employee_id, e.employee_name, e.salary, p.performance_score
FROM employees e
JOIN performance p ON e.employee_id = p.employee_id
ORDER BY e.salary DESC;
```

## 5. Recently Hired Employees in Each Department
Purpose: Identify the latest hires by department and track onboarding trends and new hires.

```sql
WITH latest_hires AS (
    SELECT e.employee_id, e.employee_name, d.department_name, e.hire_date,
           ROW_NUMBER() OVER (PARTITION BY e.department_id ORDER BY e.hire_date DESC) AS recent_rank
    FROM employees e
    JOIN departments d ON e.department_id = d.department_id
)
SELECT *
FROM latest_hires
WHERE recent_rank = 1;
```

## 6. Union Employees with and without Recent Performance Scores
Purpose: Combine employees with and without performance reviews in the last year and determine engagement levels or missed evaluations.

```sql
WITH recent_reviews AS (
    SELECT e.employee_id, e.employee_name, p.review_date, p.performance_score
    FROM employees e
    JOIN performance p ON e.employee_id = p.employee_id
    WHERE p.review_date >= NOW() - INTERVAL '1 year'
)
SELECT e.employee_id, e.employee_name, 'Reviewed' AS status
FROM recent_reviews
UNION
SELECT e.employee_id, e.employee_name, 'Not Reviewed' AS status
FROM employees e
WHERE e.employee_id NOT IN (SELECT employee_id FROM recent_reviews);
```

## 7. Offset and Fetch: Paginated Employee List
Purpose: Create a paginated view of the employee list for efficient browsing.

```sql
SELECT e.employee_id, e.employee_name, e.salary, d.department_name
FROM employees e
JOIN departments d ON e.department_id = d.department_id
ORDER BY e.employee_id
OFFSET 10 ROWS FETCH NEXT 10 ROWS ONLY;
```

## 8. Cumulative Salary Expenditure Over Time
Purpose: Compute cumulative salary spending by hire date to track organisational salary growth over time.

```sql
SELECT e.hire_date, e.salary,
       SUM(e.salary) OVER (ORDER BY e.hire_date) AS cumulative_salary
FROM employees e
ORDER BY e.hire_date;
```
