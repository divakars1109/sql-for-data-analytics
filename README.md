# Introduction
📊 Dive into the data job market! Focusing on data analyst roles, this project explores 💰 top-paying jobs, 🔥 in-demand skills, and 📈 where high demand meets high salary in data analytics.

🔍 SQL queries? Check them out here: [project_sql folder](/project_sql/)
# Background
Driven by a quest to navigate the data analyst job market more effectively, this project was born from a desire to pinpoint top-paid and in-demand skills, streamlining others' work to find optimal jobs.

Data hails from my [SQL Course](https://www.lukebarousse.com/sql). It's packed with insights on job titles, salaries, locations, and essential skills.

### The questions I wanted to answer through my SQL queries were:

1. What are the top-paying data analyst jobs?
2. What skills are required for these top-paying jobs?
3. What skills are most in demand for data analysts?
4. Which skills are associated with higher salaries?
5. What are the most optimal skills to learn?
# Tools I Used
For my deep dive into the data analyst job market, I harnessed the power of several key tools:

- **SQL:** The backbone of my analysis, allowing me to query the database and unearth critical insights.
- **PostgreSQL:** The chosen database management system, ideal for handling the job posting data.
- **Visual Studio Code:** My go-to for database management and executing SQL queries.
- **Git & GitHub:** Essential for version control and sharing my SQL scripts and analysis, ensuring collaboration and project tracking.
# The Analysis

Each query for this project aimed at investigating specific aspects of the data analyst job market. Here’s how I approached each question:

### 1. Top Paying Data Analyst Jobs

To identify the highest-paying roles, I filtered data analyst positions by average yearly salary and location, focusing on remote jobs. This query highlights the high paying opportunities in the field.

```sql
select j.job_id ,
       j.job_title,
       j.job_location,
       j.job_schedule_type,
       j.salary_year_avg,
       j.job_posted_date,
       c.name AS company_name
FROM job_postings_fact AS j
LEFT JOIN company_dim AS c
ON j.company_id = c.company_id
WHERE job_title_short = 'Data Analyst' AND 
      job_location = 'Anywhere' AND
      salary_year_avg IS NOT NULL
ORDER BY salary_year_avg DESC
LIMIT 10;
```
Here's the breakdown of the top data analyst jobs in 2023:

- **Wide Salary Range:** Top 10 paying data analyst roles span from $184,000 to $650,000, indicating significant salary potential in the field.
- **Diverse Employers:** Companies like SmartAsset, Meta, and AT&T are among those offering high salaries, showing a broad interest across different industries.
- **Job Title Variety:** There's a high diversity in job titles, from Data Analyst to Director of Analytics, reflecting varied roles and specializations within data analytics.
### 2. Skills for Top-Paying Jobs

To understand what skills are required for the highest-paying roles, I joined the top-paying jobs with the skills data to identify the technical competencies valued most by high-paying employers.

```sql
with top_paying_jobs AS(
select j.job_id ,
       j.job_title,
       j.salary_year_avg,
       c.name AS company_name
FROM job_postings_fact AS j
LEFT JOIN company_dim AS c
ON j.company_id = c.company_id
WHERE job_title_short = 'Data Analyst' AND 
      job_location = 'Anywhere' AND
      salary_year_avg IS NOT NULL
ORDER BY salary_year_avg DESC
LIMIT 10
)

select top_paying_jobs.* ,
    skills
from top_paying_jobs
INNER JOIN skills_job_dim 
ON top_paying_jobs.job_id = skills_job_dim.job_id
INNER JOIN skills_dim
ON skills_job_dim.skill_id = skills_dim.skill_id
ORDER BY salary_year_avg DESC;
```
Most Demanded Skills for Top 10 Highest-Paying Data Analyst Roles (2023)

- **SQL** — Mentioned in **8** job postings  
- **Python** — Mentioned in **7** job postings  
- **Tableau** — Mentioned in **6** job postings  
- **Additional In-Demand Skills**: `R`, `Snowflake`, `Pandas`, `Excel`
### 3. In-Demand Skills for Data Analysts

Identifies the skills most frequently requested across all data analyst postings to highlight baseline market requirements.

```sql
SELECT 
    skills,
    COUNT(skills_job_dim.job_id) AS demand_count
FROM 
    job_postings_fact
INNER JOIN skills_job_dim 
    ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim 
    ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_title_short = 'Data Analyst'
GROUP BY
    skills
ORDER BY
    demand_count DESC
LIMIT 5;
```
#### Results

| Skill | Demand Count |
| :--- | :---: |
| **SQL** | 7,291 |
| **Excel** | 4,611 |
| **Python** | 4,330 |
| **Tableau** | 3,745 |
| **Power BI** | 2,609 |

#### Key Insights

* **Core Querying & Spreadsheets:** `SQL` remains the foundational industry requirement, with `Excel` holding strong as the secondary core tool.
* **Programming & BI:** `Python` leads general-purpose scripting, while visualization needs are split between `Tableau` and `Power BI`.
### 4. Skills Based on Salary

Explores the average salaries associated with different skills to reveal which technical competencies command the highest compensation in the data analyst job market.

```sql
select skills , Round(Avg(salary_year_avg),0) AS avg_salary
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE job_title_short = 'Data Analyst' AND salary_year_avg is NOT NULL
GROUP BY skills
ORDER BY avg_salary DESC 
limit 20;
```
#### Key Insights

* **Big Data & ML Dominance:** Top salaries are commanded by analysts skilled in big data frameworks (`PySpark`, `Couchbase`) and machine learning/Python data tools (`DataRobot`, `Jupyter`, `Pandas`), reflecting the industry's high valuation of scalable data processing and predictive modeling capabilities.
* **Engineering & DevOps Crossover:** Competency in deployment and version control tools like `GitLab`, `Bitbucket`, and workflow orchestration (`Airflow`, `Kubernetes`) indicates higher compensation for analysts capable of automating pipelines and managing code repositories.
* **Cloud Analytics:** Specialized tools and distributed systems (`Elasticsearch`, `Databricks`, `GCP`) significantly elevate earnings compared to standard querying alone, underscoring the shift toward cloud-native analytics environments.
### 5. Most Optimal Skills to Learn (High Demand & High Paying)

Combining insights from demand and salary data, this query pinpoints skills that offer both high market demand and competitive salaries for remote Data Analyst roles, serving as a strategic target for career development.

```sql
WITH skills_demand AS (
    SELECT 
        skills_dim.skill_id,
        skills_dim.skills,
        COUNT(skills_job_dim.job_id) AS count_jobs
    FROM job_postings_fact
    INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
    INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
    WHERE 
        job_title_short = 'Data Analyst' 
        AND salary_year_avg IS NOT NULL 
        AND job_work_from_home = True
    GROUP BY 
        skills_dim.skill_id,
        skills_dim.skills
), average_salary AS (
    SELECT
        skills_job_dim.skill_id,
        ROUND(AVG(salary_year_avg), 0) AS avg_salary
    FROM job_postings_fact
    INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
    INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
    WHERE 
        job_title_short = 'Data Analyst' 
        AND salary_year_avg IS NOT NULL 
        AND job_work_from_home = True
    GROUP BY 
        skills_job_dim.skill_id
)

SELECT 
    skills_demand.skill_id,
    skills_demand.skills,
    count_jobs,
    avg_salary
FROM
    skills_demand
INNER JOIN average_salary ON skills_demand.skill_id = average_salary.skill_id
Where count_jobs > 10
ORDER BY 
    count_jobs DESC, 
    avg_salary DESC
Limit 25;
```

#### Key Insights

* **High-Demand Programming Languages:** `Python` and `R` stand out with strong demand counts and average salaries around $101,397 and $100,499 respectively, confirming their role as market standards.
* **Cloud Tools & Warehousing:** `Snowflake`, `Azure`, `AWS`, and `BigQuery` show high demand alongside strong compensation premiums, highlighting the value of cloud infrastructure expertise.
* **BI Platforms:** `Tableau` and `Looker` maintain high demand and average salaries near $100k, underscoring the critical value of data visualization and executive reporting.
* **Database Systems:** Relational staples like `SQL Server` and `Oracle` retain steady market presence, reflecting persistent demand for enterprise database querying and maintenance.

# What I Learned

Throughout this adventure, I've turbocharged my SQL toolkit with some serious firepower:

- **🧩 Complex Query Crafting:** Mastered the art of advanced SQL, merging tables like a pro and wielding `WITH` clauses for ninja-level temp table maneuvers.
- **📊 Data Aggregation:** Got cozy with `GROUP BY` and turned aggregate functions like `COUNT()` and `AVG()` into my data-summarizing sidekicks.
- **💡 Analytical Wizardry:** Leveled up my real-world puzzle-solving skills, turning questions into actionable, insightful SQL queries.

# Conclusions
This project enhanced my SQL skills and provided valuable insights into the data analyst job market. The findings from the analysis serve as a guide to prioritizing skill development and job search efforts. Aspiring data analysts can better position themselves in a competitive job market by focusing on high-demand, high-salary skills. This exploration highlights the importance of continuous learning and adaptation to emerging trends in the field of data analytics.
