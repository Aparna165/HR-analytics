# 📊 HR Analytics — Employee Attrition & Salary Gap Analysis

![SQL](https://img.shields.io/badge/SQL-PostgreSQL%20%7C%20MySQL-blue?logo=postgresql)
![Excel](https://img.shields.io/badge/Data-Excel%20%28.xlsx%29-217346?logo=microsoft-excel)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen)
![Records](https://img.shields.io/badge/Records-1471%20Employees-orange)

> A structured SQL-based analytics project exploring **employee attrition patterns** and **salary disparities** across departments, job roles, genders, and age groups using real-world HR data.

---

## 📁 Project Structure

```
HR-Analytics/
├── HR_Analytics_10_Columns.xlsx         # Raw dataset (1471 employee records)
├── Employee_Attrition_Salary_Gap.sql    # All 21 SQL queries
└── README.md
```

---

## 📋 Dataset Overview

**File:** `HR_Analytics_10_Columns.xlsx`  
**Sheet:** `HR_Data_10_Columns`  
**Total Records:** 999 employees

| Column | Type | Description |
|---|---|---|
| `Age` | INT | Employee age |
| `Attrition` | VARCHAR | Whether the employee left (`Yes` / `No`) |
| `Department` | VARCHAR | Department name |
| `Gender` | VARCHAR | Employee gender |
| `JobRole` | VARCHAR | Job role title |
| `MonthlyIncome` | INT | Monthly salary (USD) |
| `YearsAtCompany` | INT | Tenure in years |
| `JobSatisfaction` | INT | Satisfaction score (1–4) |
| `WorkLifeBalance` | INT | Work-life balance score (1–4) |
| `OverTime` | VARCHAR | Whether employee works overtime (`Yes` / `No`) |

### Departments
`Human Resources` · `Research & Development` · `Sales`

### Job Roles
`Healthcare Representative` · `Human Resources` · `Laboratory Technician` · `Manager` · `Manufacturing Director` · `Research Director` · `Research Scientist` · `Sales Executive` · `Sales Representative`

---

## 🗄️ Database Setup

```sql
CREATE TABLE employees (
    EmployeeNumber    INT PRIMARY KEY,
    Age               INT,
    Gender            VARCHAR(20),
    Department        VARCHAR(100),
    JobRole           VARCHAR(100),
    MonthlyIncome     INT,
    YearsAtCompany    INT,
    EducationField    VARCHAR(100),
    BusinessTravel    VARCHAR(50),
    Attrition         VARCHAR(10)
);
```

Import the Excel dataset into your SQL environment before running the queries.

---

## 🔍 SQL Queries — Full Reference

### 1. Attrition Rate by Department
Calculates total employees, attrition count, and attrition percentage per department, ordered by highest risk first.

```sql
SELECT Department,
       COUNT(*) AS total_employees,
       SUM(CASE WHEN Attrition = 'Yes' THEN 1 ELSE 0 END) AS attrition_count,
       ROUND(100.0 * SUM(CASE WHEN Attrition='Yes' THEN 1 ELSE 0 END) / COUNT(*), 2) AS attrition_rate
FROM employees
GROUP BY Department
ORDER BY attrition_rate DESC;
```

---

### 2. Job Roles with Highest Attrition Risk
Filters only roles where attrition rate exceeds 20%.

```sql
SELECT JobRole, COUNT(*) AS employees,
       SUM(CASE WHEN Attrition='Yes' THEN 1 ELSE 0 END) AS attritions,
       ROUND(100.0 * SUM(CASE WHEN Attrition='Yes' THEN 1 ELSE 0 END) / COUNT(*), 2) AS attrition_rate
FROM employees
GROUP BY JobRole
HAVING ROUND(100.0 * SUM(CASE WHEN Attrition='Yes' THEN 1 ELSE 0 END) / COUNT(*), 2) > 20;
```

---

### 3. Gender Salary Gap Analysis
Compares average monthly income across genders to identify pay disparities.

```sql
SELECT Gender, ROUND(AVG(MonthlyIncome), 2) AS avg_salary
FROM employees
GROUP BY Gender;
```

---

### 4. Department-Wise Salary Comparison
Ranks departments by average monthly salary (highest to lowest).

```sql
SELECT Department, ROUND(AVG(MonthlyIncome), 2) AS avg_salary
FROM employees
GROUP BY Department
ORDER BY avg_salary DESC;
```

---

### 5. Departments Paying Above Company Average
Uses a subquery to surface only departments with above-average pay.

```sql
SELECT Department, ROUND(AVG(MonthlyIncome), 2) AS dept_avg_salary
FROM employees
GROUP BY Department
HAVING AVG(MonthlyIncome) > (SELECT AVG(MonthlyIncome) FROM employees);
```

---

### 6. Attrition by Gender
Breaks down attrition count and rate by gender.

```sql
SELECT Gender,
       SUM(CASE WHEN Attrition='Yes' THEN 1 ELSE 0 END) AS attritions,
       COUNT(*) AS total_employees,
       ROUND(100.0 * SUM(CASE WHEN Attrition='Yes' THEN 1 ELSE 0 END) / COUNT(*), 2) AS attrition_rate
FROM employees
GROUP BY Gender;
```

---

### 7. Frequent Travelers at Risk
Examines how business travel frequency correlates with attrition.

```sql
SELECT BusinessTravel,
       COUNT(*) AS employees,
       SUM(CASE WHEN Attrition='Yes' THEN 1 ELSE 0 END) AS attritions
FROM employees
GROUP BY BusinessTravel;
```

---

### 8. Salary Distribution Bands
Segments employees into Low / Medium / High salary bands using `CASE`.

```sql
SELECT
  CASE
    WHEN MonthlyIncome < 5000 THEN 'Low Salary'
    WHEN MonthlyIncome BETWEEN 5000 AND 10000 THEN 'Medium Salary'
    ELSE 'High Salary'
  END AS salary_band,
  COUNT(*) AS employees
FROM employees
GROUP BY salary_band;
```

---

### 9. Attrition by Salary Band
Cross-references salary bands with attrition count to reveal income–retention patterns.

```sql
SELECT
  CASE
    WHEN MonthlyIncome < 5000 THEN 'Low Salary'
    WHEN MonthlyIncome BETWEEN 5000 AND 10000 THEN 'Medium Salary'
    ELSE 'High Salary'
  END AS salary_band,
  COUNT(*) AS employees,
  SUM(CASE WHEN Attrition='Yes' THEN 1 ELSE 0 END) AS attritions
FROM employees
GROUP BY salary_band;
```

---

### 10. Employees Earning Above Department Average
Uses a correlated subquery to find employees outearning their department peers.

```sql
SELECT * FROM employees e
WHERE MonthlyIncome > (
    SELECT AVG(MonthlyIncome) FROM employees d
    WHERE d.Department = e.Department
);
```

---

### 11. Long-Tenure Employees Who Left
Identifies experienced employees (5+ years) who still chose to leave.

```sql
SELECT * FROM employees
WHERE Attrition = 'Yes' AND YearsAtCompany > 5;
```

---

### 12. Average Tenure by Department
Shows how long employees typically stay in each department.

```sql
SELECT Department, ROUND(AVG(YearsAtCompany), 2) AS avg_tenure
FROM employees
GROUP BY Department;
```

---

### 13. Departments with More Than 50 Employees
Filters for departments large enough for statistically meaningful analysis.

```sql
SELECT Department, COUNT(*) AS employee_count
FROM employees
GROUP BY Department
HAVING COUNT(*) > 50;
```

---

### 14. Top 5 Highest Paid Employees
Quick lookup of the company's top earners.

```sql
SELECT EmployeeNumber, JobRole, MonthlyIncome
FROM employees
ORDER BY MonthlyIncome DESC
LIMIT 5;
```

---

### 15. Highest Paid Employee in Each Department
Uses a correlated subquery to identify the top earner per department.

```sql
SELECT * FROM employees e
WHERE MonthlyIncome = (
    SELECT MAX(MonthlyIncome) FROM employees d
    WHERE d.Department = e.Department
);
```

---

### 16. Attrition Rate by Education Field
Reveals which academic backgrounds have the highest turnover.

```sql
SELECT EducationField,
       COUNT(*) AS employees,
       SUM(CASE WHEN Attrition='Yes' THEN 1 ELSE 0 END) AS attritions,
       ROUND(100.0 * SUM(CASE WHEN Attrition='Yes' THEN 1 ELSE 0 END) / COUNT(*), 2) AS attrition_rate
FROM employees
GROUP BY EducationField
ORDER BY attrition_rate DESC;
```

---

### 17. Young Employees Leaving Frequently
Groups employees as Young / Middle Age / Senior and compares attrition counts.

```sql
SELECT
  CASE
    WHEN Age < 30 THEN 'Young'
    WHEN Age BETWEEN 30 AND 45 THEN 'Middle Age'
    ELSE 'Senior'
  END AS age_group,
  COUNT(*) AS employees,
  SUM(CASE WHEN Attrition='Yes' THEN 1 ELSE 0 END) AS attritions
FROM employees
GROUP BY age_group;
```

---

### 18. Departments with High Attrition AND Low Salary
Pinpoints departments that are both underpaying and losing people — the highest-risk combination.

```sql
SELECT Department,
       AVG(MonthlyIncome) AS avg_salary,
       SUM(CASE WHEN Attrition='Yes' THEN 1 ELSE 0 END) AS attritions
FROM employees
GROUP BY Department
HAVING AVG(MonthlyIncome) < (SELECT AVG(MonthlyIncome) FROM employees)
   AND SUM(CASE WHEN Attrition='Yes' THEN 1 ELSE 0 END) > 20;
```

---

### 19. Rank Employees by Salary Within Department
Uses the `RANK()` window function to produce within-department salary rankings.

```sql
SELECT EmployeeNumber, Department, MonthlyIncome,
       RANK() OVER (PARTITION BY Department ORDER BY MonthlyIncome DESC) AS salary_rank
FROM employees;
```

---

### 20. Employee Segments at Highest Attrition Risk
Two-dimensional segmentation: combines **age group × travel frequency** to find the most at-risk combinations.

```sql
SELECT
  CASE
    WHEN Age < 30 THEN 'Young'
    WHEN Age BETWEEN 30 AND 45 THEN 'Mid-Career'
    ELSE 'Senior'
  END AS age_group,
  BusinessTravel,
  COUNT(*) AS employees,
  SUM(CASE WHEN Attrition='Yes' THEN 1 ELSE 0 END) AS attritions,
  ROUND(100.0 * SUM(CASE WHEN Attrition='Yes' THEN 1 ELSE 0 END) / COUNT(*), 2) AS attrition_rate
FROM employees
GROUP BY age_group, BusinessTravel
ORDER BY attrition_rate DESC;
```

---

## 💡 Key Business Insights

| Area | Finding |
|---|---|
| **Salary & Attrition** | Low-salary employees show the highest attrition — compensation is a primary driver |
| **Gender Pay Gap** | Average salary varies between male and female employees across departments |
| **High-Risk Roles** | Roles with >20% attrition rate are flagged as critical retention risks |
| **Travel Impact** | Frequent business travelers exhibit noticeably higher attrition |
| **Young Employees** | Employees under 30 leave at higher rates than mid-career or senior staff |
| **Long Tenure Exits** | Experienced employees (5+ years) still leave, suggesting non-monetary factors |
| **Department Risk** | Departments with both below-average salary AND high attrition need immediate HR intervention |

---

## 🛠️ Tools & Technologies

- **Database:** PostgreSQL / MySQL (standard SQL syntax)
- **Dataset Format:** Microsoft Excel `.xlsx`
- **Concepts Used:** `GROUP BY`, `HAVING`, `CASE WHEN`, Correlated Subqueries, Window Functions (`RANK() OVER`), Aggregate Functions (`AVG`, `SUM`, `COUNT`, `MAX`)

---

## 🚀 How to Run

1. **Clone the repository**
   ```bash
   git clone https://github.com/your-username/hr-analytics-sql.git
   cd hr-analytics-sql
   ```

2. **Set up your database**  
   Create a new database in PostgreSQL  and run the `CREATE TABLE` statement from the `.sql` file.

3. **Import the dataset**  
   Import `HR_Analytics_10_Columns.xlsx` into the `employees` table using your preferred DB tool (pgAdmin, DBeaver, MySQL Workbench, etc.) or convert to CSV first:
   ```bash
   # Using psql (after converting to CSV)
   \copy employees FROM 'HR_Analytics_10_Columns.csv' DELIMITER ',' CSV HEADER;
   ```

4. **Run the queries**  
   Execute `Employee_Attrition_Salary_Gap.sql` in your SQL client — queries are numbered and self-contained.

---

## 👤 Author

**Upadrasta Aparna**  

---

## 📄 License

This project is open source and available under the [MIT License](LICENSE).
