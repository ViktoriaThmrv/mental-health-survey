# Mental Health Survey Analysis

## Overview
This project involves the analysis of a mental health survey dataset exported from Kaggle using PostreSQL. The dataset contains information about respondents' demographics, employment status, and mental health treatment seeking behavior. The aim is to explore patterns and correlations within the data to gain insights into mental health treatment trends among survey participants. The dataset can be found [here](https://www.kaggle.com/datasets/osmi/mental-health-in-tech-survey/code?datasetId=311&searchQuery=sql).

## Purpose
The primary purpose of this project is to understand the prevalence of mental health issues within the surveyed population and identify factors that may influence treatment-seeking behavior. By analyzing various demographic and workplace-related variables, the project aims to uncover patterns and correlations that can inform strategies for promoting mental health support and intervention programs in workplace settings.

### Note
This project is currently in progress. The analysis is ongoing, and queries will be added and edited as the project develops.

## Dataset
The dataset used for this analysis is stored in a PostgreSQL database. It includes the following columns:

- Timestamp - The date and time when the survey response was recorded;
- Age - The age of the respondent;
- Gender - The gender identity of the respondent, written as free text;
- Country - The country where the respondent resides;
- State -  The state (if applicable) where the respondent resides;
- Self-employed - Indicates whether the respondent is self-employed (Yes/No);
- Family history - Indicates whether the respondent has a family history of mental health issues (Yes/No);
- Treatment - Indicates whether the respondent has sought treatment for mental health issues (Yes/No);
- Work interference - Frequency of work interference due to mental health issues (e.g., Often, Rarely);
- No. of employees - The number of employees in the respondent's workplace;
- Remote work - Indicates whether the respondent has the option to work remotely (Yes/No);
- Tech company - Indicates whether the respondent works in a tech company (Yes/No);
- Benefits- Indicates whether the respondent's employer provides mental health benefits (Yes/No);
- Care options - Availability of mental health care options provided by the employer;
- Wellness program - Indicates whether the respondent's employer offers a wellness program (Yes/No);
- Seek help - Indicates whether the respondent feels comfortable seeking help for mental health issues (Yes/No);
- Anonymity - Indicates whether the respondent feels anonymity is preserved if they seek help (Yes/No/Don't know);
- Leave - Availability of leave options for mental health reasons provided by the employer;
- Mental health consequence - Perception of how mental health issues would affect work;
- Phys health consequence - Perception of how physical health issues would affect work;
- Coworkers - Comfort level discussing mental health issues with coworkers (e.g., Some of them, Yes);
- Supervisor - Comfort level discussing mental health issues with supervisors (e.g., Some of them, Yes);
- Mental health interview - Whether the employer discusses mental health as part of the interview process (Yes/No);
- Phys health interview - Whether the employer discusses physical health as part of the interview process (Yes/No);
- Mental vs physical - Perceived importance of mental health compared to physical health by the employer;
- Obs consequence - Perception of negative consequences for revealing mental health issues (Yes/No);
- Comments - Additional comments or remarks provided by the respondent.

## Queries

### 1. Data preparation and cleaning

### Standardizing Gender Values
This query standardizes the values in the `gender` column of the mental_health_survey table to ensure consistency across the dataset. It uses pattern recognition with flexible matching to categorize gender entries into `Male`, `Female`, or `Other`. 

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

### 2. Calculating the average age
This query calculates the average age of respondents in the mental_health_survey table and rounds the result to the nearest integer.

### Query Explanation
The `ROUND()` function rounds the result of the `AVG()` function to the nearest integer and the `AVG()` function calculates the average age of all respondents in the dataset.

### SQL Query
```sql
SELECT
	ROUND(AVG(age)) as avg_age
FROM
mental_health_survey;
```
### The Output
| avg_age | 
|---------|
|      41 |  

### 3. Counting self-employed respondents experiencing work interference often
This query counts the number of self-employed respondents who reported experiencing work interference often in the `survey_data` table.

### Query Explanation
The `COUNT(*)` function counts the number of rows that meet the specified conditions, with the WHERE clause filtering the rows where the respondents are self-employed `Yes` and report often experiencing work interference.

### SQL Query
```sql
SELECT COUNT(*) as self_employed_interference
FROM survey_data
WHERE self_employed = 'Yes' AND work_interfere = 'Often';
```
### The Output
| self_employed_interference | 
|----------------------------|
|                         28 |  

### 4. Total respondents with treatment
This query counts the total number of respondents who have sought treatment for mental health issues.

#### Query Explanation
The query counts the total number of respondents who answered `Yes` to seeking treatment for mental health issues.

```sql
SELECT COUNT(treatment) AS total_respondents_with_treatment
FROM mental_health_survey
WHERE treatment = 'Yes';
```
### The Output

| total_respondents_with_treatment | 
|----------------------------------|
|                              637 |

### 5. Overall percentage of treatment seekers
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

### 6. Analyzing treatment rates across different countries
The query aims to calculate the overall percentage of respondents who have sought treatment for mental health issues.

#### Query Explanation
Here, we analyze the treatment rates for mental health issues across different countries represented in the `mental_health_survey table`. This calculates the treatment rate for each country and ranks them based on their treatment rates.

```sql
SELECT 
    Country,
    treatment_rate,
    RANK() OVER (ORDER BY treatment_rate DESC) AS rank
FROM 
    (SELECT 
         Country,
         SUM(CASE WHEN treatment = 'Yes' THEN 1 ELSE 0 END) * 100 / COUNT(*) AS treatment_rate
     FROM 
         mental_health_survey
     GROUP BY 
         Country) AS treatment_rates;
```
### The Output

|    country   | treatment_rate | rank |
|--------------|----------------|------|
|     Slovenia |            100 |    1 | 
|      Denmark |            100 |    1 | 
|     Zimbabwe |            100 |    1 | 
|      Moldova |            100 |    1 | 
|      Croatia |            100 |    1 | 
|        Japan |            100 |    1 | 
|      Bahamas |            100 |    1 | 
| South Africa |             66 |    8 |
|  New Zealand |             62 |    9 | 
|    Australia |             61 |   10 | 
|       Poland |             57 |   11 | 
| United States|             54 |   12 | 
|       Canada |             51 |   13 | 
|     Bulgaria |             50 |   14 | 

### 7. Analyzing treatment-seeking behavior, based on mental health benefits
This query helps with categorizing respondents into two groups based on whether they have mental health benefits or not. It then calculates the percentage of respondents seeking treatment for mental health issues within each group.

#### Query Explanation
This SQL query uses a `CASE` statement to label respondents as 'With Benefits' or 'Without Benefits'. Within each group, it calculates the treatment percentage by averaging a conditional expression that evaluates to 1 if the respondent sought treatment "Yes" and 0 otherwise using the `AVG()` function. Multiplying the result by 100 gives the treatment percentage for each group. Finally, the `GROUP BY` clause is used to group the results by the benefit status.

```sql
SELECT 
    CASE 
        WHEN benefits = 'Yes' THEN 'With Benefits'
        ELSE 'Without Benefits'
    END AS benefit_status,
    AVG(CASE WHEN treatment = 'Yes' THEN 1 ELSE 0 END) * 100 AS treatment_percentage
FROM 
    mental_health_survey
GROUP BY 
    benefit_status;
```
### The Output

|  benefit_status  | treatment_percentage |
|------------------|----------------------|
| Without Benefits |                   42 | 
|    With Benefits |                   63 | 


### 8. Analysis of Family History and Treatment Seeking Behavior for Mental Health
This SQL query examines the correlation between having a family history of mental health issues and the likelihood of seeking treatment. It calculates the percentage of individuals who have sought treatment among those with and without a family history of mental health issues.

### Query Explanation
The query has two parts:
1. Common Table Expression (CTE) - Aggregates data by family_history and calculates the total number of responses total_count and the number of individuals who sought treatment treatment_count for each group.
2. Final SELECT Statement - Calculates the percentage of individuals who sought treatment percentage_treated for each group.

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
|----------------|-------------|-----------------|--------------------|
| Yes            | 767         | 272             | 35                 |
| No             | 492         | 365             | 74                 |


### 9. Analysis of anonymity and help-seeking behavior for mental health
This SQL query examines whether the availability of anonymity influences individuals' likelihood of seeking help for mental health issues. It calculates the percentage of individuals who sought help when anonymity was provided compared to those who did not have anonymity.

### Query explaination
1. Subquery - It groups the survey data by whether anonymity was offered or not, counting how many people there are in each group total and how many of them sought help seek_help_yes.
2. Outer SELECT Statement - It uses the results from the subquery to calculate the percentage of people who sought help in each group.

### SQL query
```sql
SELECT 
    anonymity,
    total,
    seek_help_yes,
    (seek_help_yes * 100) / total AS percentage_seek_help
FROM (
    SELECT 
        anonymity,
        COUNT(*) AS total,
        SUM(CASE WHEN seek_help = 'Yes' THEN 1 ELSE 0 END) AS seek_help_yes
    FROM 
        mental_health_survey
    GROUP BY 
        anonymity
) AS anon_data;
```

### The Output

| annonymity | total | seek_help_yes | percentage_seek_help |
|------------|-------|---------------|----------------------|
| Yes        |   375 |           147 |                   39 |
| No         |    65 |             6 |                    9 |
| Don't know |   819 |            97 |                   11 |

## Conclusion

### Demographic Trends
The analysis revealed a diverse range of respondents in terms of age, gender, and geographical distribution, highlighting the importance of considering these factors in mental health interventions.

### Employment Factors
Workplace-related variables such as self-employment status, company size, and availability of benefits significantly influence mental health treatment-seeking behavior and perceptions.

### Treatment Rates
While approximately half of the respondents reported seeking treatment for mental health issues, disparities exist across different countries and employment settings, indicating the need for targeted interventions.

### Impact of Benefits
Respondents with access to mental health benefits demonstrated higher rates of treatment-seeking behavior, underscoring the importance of employer support in facilitating mental health care access.

### Family History and Anonymity
Individuals with a family history of mental health issues showed lower treatment-seeking rates, while the availability of anonymity appeared to positively influence help-seeking behavior.

