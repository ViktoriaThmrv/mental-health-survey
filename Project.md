# Mental Health Survey Analysis

## Overview
This project involves the analysis of a mental health survey dataset exported from Kaggle using PostreSQL. The dataset contains information about respondents' demographics, employment status, and mental health treatment seeking behavior. The aim is to explore patterns and correlations within the data to gain insights into mental health treatment trends among survey participants. The dataset can be found [here](https://www.kaggle.com/datasets/osmi/mental-health-in-tech-survey/code?datasetId=311&searchQuery=sql).

### Note
This project is currently in progress. The analysis is ongoing, and additional queries will be added as the project develops.

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

### 1. Standardizing Gender Values
```sql
-- Update the gender column using pattern recognition with flexible matching to standardize the values for consistent analysis.
UPDATE mental_health_survey
SET gender = 
    CASE 
        WHEN LOWER(gender) SIMILAR TO '%(m|male)%' THEN 'Male'
        WHEN LOWER(gender) SIMILAR TO '%(f|female|girl)%' THEN 'Female'
        ELSE 'Other'
    END;
```
**Description:** This query updates the 'gender' column in the dataset to standardize the values to 'Male', 'Female', or 'Other' using pattern recognition with flexible matching.

### 2. Treatment Seeking Behavior
```sql
-- How many respondents have sought treatment for mental health issues?

SELECT COUNT(treatment) AS total_respondents_with_treatment
FROM mental_health_survey
WHERE treatment = 'Yes';

--What is the overall percentage of respondents who have sought treatment for mental health issues?
SELECT 
    (SELECT COUNT(treatment) FROM mental_health_survey WHERE treatment= 'Yes') *100/ COUNT(*) percentage_sought_treatment
FROM mental_health_survey;

--What percentage of male and female respondents have sought treatment, and is there a significant difference between the two?
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
**Description:** These queries analyze the treatment seeking behavior among survey respondents. The first query counts the total number of respondents who have sought treatment for mental health issues. The second query calculates the overall percentage of respondents who have sought treatment. The third query breaks down the treatment seeking behavior by gender and calculates the percentage of treated respondents for each gender category.

### 3. Remote Work and Mental Health Treatment
```sql
-- Explore the correlation between remote work and mental health treatment.
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
**Description:** This query examines the correlation between remote work and mental health treatment seeking behavior among respondents. It counts the total number of respondents and the number of respondents who have sought treatment for mental health issues, grouped by remote work status.

### 4. Company Size and Treatment Seeking Behavior
```sql
-- Are individuals from larger companies more or less likely to seek treatment compared to those from smaller companies?
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
**Description:** This query investigates whether individuals from larger companies are more or less likely to seek treatment for mental health issues compared to those from smaller companies. It calculates the percentage of treated respondents for each company size category.

## Results
The analysis aims to provide insights into mental health treatment seeking behavior among survey respondents. Initial findings suggest variations in treatment seeking behavior across different demographics and employment characteristics.

## Next Steps
- Refine and expand the analysis to include additional variables and explore deeper insights.
- Document the analysis process and results comprehensively.
