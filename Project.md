# Mental Health Survey Analysis

## Overview
This project involves the analysis of a mental health survey dataset exported from Kaggle using PostreSQL. The dataset contains information about respondents' demographics, employment status, and mental health treatment seeking behavior. The aim is to explore patterns and correlations within the data to gain insights into mental health treatment trends among survey participants. The dataset can be found [here](https://www.kaggle.com/datasets/osmi/mental-health-in-tech-survey/code?datasetId=311&searchQuery=sql).

### Note
This project is currently in progress. The analysis is ongoing, and queries will be added and edited as the project develops.

## Dataset
The dataset used for this analysis is stored in a PostgreSQL database. It includes the following columns:

- Timestamp
- Age
- Gender
- Country
- State
- Self-employed
- Family history
- Treatment
- Work interference
- No. of employees
- Remote work
- Tech company
- Benefits
- Care options
- Wellness program
- Seek help
- Anonymity
- Leave
- Mental health consequence
- Phys health consequence
- Coworkers
- Supervisor
- Mental health interview
- Phys health interview
- Mental vs physical
- Obs consequence
- Comments

## Queries

### Data preparation and cleaning

### Standardizing Gender Values

This query standardizes the values in the 'gender' column of the mental_health_survey table to ensure consistency across the dataset. It uses pattern recognition with flexible matching to categorize gender entries into 'Male', 'Female', or 'Other'. 

#### Query Explanation
1. The LOWER(gender) function converts all gender entries to lowercase to ensure case-insensitive matching.
2. The SIMILAR TO operator matches entries to patterns:
- Entries that match patterns like 'm' or 'male' are set to 'Male'.
- Entries that match patterns like 'f', 'female', or 'girl' are set to 'Female'.
- All other entries are set to 'Other'.

```sql
UPDATE mental_health_survey
SET gender = 
    CASE 
        WHEN LOWER(gender) SIMILAR TO '%(m|male)%' THEN 'Male'
        WHEN LOWER(gender) SIMILAR TO '%(f|female|girl)%' THEN 'Female'
        ELSE 'Other'
    END;

SELECT
	gender
FROM
 	mental_health_survey;
```
### The Output
| gender | 
|----------|
| Male     | 
| Other    |
| Female   |

### Analysis

### 1. Total Respondents with Treatment
This query counts the total number of respondents who have sought treatment for mental health issues.

#### Query Explanation

The query counts the total number of respondents who answered 'Yes' to seeking treatment for mental health issues.

```sql
SELECT COUNT(treatment) AS total_respondents_with_treatment
FROM mental_health_survey
WHERE treatment = 'Yes';
```
### The Output

| total_respondents_with_treatment | 
|----------------------------------|
|                              637 |

### 2. Overall Percentage of Treatment Seekers
The query aims to calculate the overall percentage of respondents who have sought treatment for mental health issues.

#### Query Explanation
This query uses a subquery within the SELECT statement to calculate the total number of respondents who have sought treatment for mental health issues. It then divides this count by the total number of respondents in the mental_health_survey table to determine the percentage of respondents who sought treatment.

```sql
SELECT 
    (SELECT COUNT(treatment) FROM mental_health_survey WHERE treatment= 'Yes') *100/ COUNT(*) percentage_sought_treatment
FROM mental_health_survey;
```
### The Output

| percentage_sought_treatment | 
|-----------------------------|
|                          50 |


### 3. Gender Breakdown of Treatment Seekers

This query examines the percentage of male and female respondents who have sought treatment, and checks for any significant differences between the two groups.

#### Query Explanation
1. Common Table Expression (CTE): Aggregates data by gender and calculates the total number of respondents (total_respondents) and the number of individuals who sought treatment (treated_respondents) for each gender group.
2. Final SELECT Statement: Calculates the percentage of treated respondents (percentage_treated) for each gender group.

```sql
WITH treatment_counts AS (
    SELECT 
        gender,
        COUNT(*) as total_respondents, 
        SUM(CASE WHEN treatment='Yes' THEN 1 ELSE 0 END) treated_respondents
    FROM 
        mental_health_survey
    GROUP BY 
        gender
)
SELECT 
    gender,
    total_respondents, 
    treated_respondents, 
    (treated_respondents*100)/ total_respondents as percentage_treated 
FROM treatment_counts;
```
### The Output

| gender | total_respondents | treated_respondents | percentage_treated |
|--------|-------------------|---------------------|--------------------|
| Male   |              1192 |                 589 |                 49 |
| Female |                54 |                  36 |                 62 |
| Other  |                13 |                  12 |                 92 |

### 4. Remote Work and Mental Health Treatment
This query examines the correlation between remote work and mental health treatment seeking behavior among respondents. 

#### Query Explanation
This query groups the data by the 'remote_work' column and calculates the total number of respondents and the number of respondents who have sought treatment for mental health issues in each group. It then calculates the percentage of treated respondents within each group by dividing the count of treated respondents by the total count of respondents in that group.

```sql
SELECT 
    remote_work, 
    COUNT(*) AS total,
    SUM(CASE WHEN treatment = 'Yes' THEN 1 ELSE 0 END) AS treatment_count,
    (SUM(CASE WHEN treatment = 'Yes' THEN 1 ELSE 0 END) *100) / COUNT(*) AS treatment_percentage
FROM 
    mental_health_survey
GROUP BY 
    remote_work;
```
### The Output
| remote_work | total | treatment_count | treatment_percentage |
|-------------|-------|-----------------|----------------------|
| No          |   883 |             439 |                   49 |
| Yes         |   376 |             198 |                   52 |


### 5. Company Size and Treatment Seeking Behavior
This query investigates whether individuals from larger companies are more or less likely to seek treatment for mental health issues compared to those from smaller companies. 

#### Query Explanation
1. Common Table Expression (CTE): The query starts with a CTE named company_size_counts, which aggregates data by the number of employees (no_employees). It counts the total number of respondents (total_respondants) and the number of respondents who sought treatment (treated_respondants) for each group based on company size.
2. Final SELECT Statement: After grouping the data by company size, the final SELECT statement retrieves columns from the CTE, including no_employees, total_respondants, and treated_respondants. Additionally, it calculates the percentage of treated respondents (percentage) for each company size group.

```sql
WITH company_size_counts AS (
    SELECT
        no_employees,
            COUNT(*) AS total_respondants,
            SUM(CASE WHEN treatment = 'Yes' THEN 1 ELSE 0 END) AS treated_respondants
    FROM mental_health_survey
    GROUP BY no_employees
)
SELECT
    no_employees,
    total_respondants,
    treated_respondants,
    (treated_respondants *100)/total_respondants AS percentage
FROM company_size_counts
```
### The Output
| no_employees   | total_respondants | treated_respondants | percentage |
|----------------|-------------------|-----------------|----------------|
| More than 1000 |               282 |             146 |             51 |
| 45809          |               290 |             128 |             44 |
| 100-500        |               176 |              95 |             53 |
| 500-1000       |                60 |              27 |             45 |
| 45413          |               162 |              91 |             56 |
| 26-100         |               289 |             150 |             51 |

### 5. Analysis of Family History and Treatment Seeking Behavior for Mental Health

This SQL query examines the correlation between having a family history of mental health issues and the likelihood of seeking treatment. It calculates the percentage of individuals who have sought treatment among those with and without a family history of mental health issues.

### Query Explanation
The query has two parts:
1. Common Table Expression (CTE): Aggregates data by `family_history` and calculates the total number of responses (`total_count`) and the number of individuals who sought treatment (`treatment_count`) for each group.
2. Final SELECT Statement: Calculates the percentage of individuals who sought treatment (`percentage_treated`) for each group.

### SQL Query
```sql
WITH family_history_count AS (
    SELECT
        family_history,
        COUNT(*) AS total_count,
        SUM(CASE WHEN treatment = 'Yes' THEN 1 ELSE 0 END) AS treatment_count
    FROM mental_health_survey
    GROUP BY family_history
)
SELECT 
    family_history,
    total_count,
    treatment_count,
    (treatment_count * 100) / total_count AS percentage_treated
FROM family_history_count;
```

### The Output
| family_history | total_count | treatment_count | percentage_treated |
|----------------|-------------|-----------------|---------------------|
| Yes            | 767         | 272              | 35                  |
| No             | 492         | 365              | 74                  |


## Results
The analysis aims to provide insights into mental health treatment seeking behavior among survey respondents. Initial findings suggest variations in treatment seeking behavior across different demographics and employment characteristics.

## Next Steps
- Refine and expand the analysis to include additional variables and explore deeper insights.
- Document the analysis process and results comprehensively.
