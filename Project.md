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
This query aims to simplify the representation of gender across the dataset, thereby simplifying the analysis and interpretation of gender-related insights. By standardizing diverse gender entries into coherent categories, it enhances the dataset's coherence and usability, fostering a more structured and insightful analytical process.

#### Query Explanation
This query aims to standardize the values in the `gender` column of the `mental_health_survey` table to ensure consistency across the dataset. It starts with an update operation, adjusting the gender values based on specific patterns found in the existing data.

The `CASE` statement evaluates each row's `gender` value and assigns it a standardized category. If the `gender` value contains patterns similar to 'm' or 'male', it's categorized as 'Male'. Similarly, if the `gender` value resembles 'f', 'female', or 'girl', it's categorized as 'Female'. Any other values or patterns not matching these criteria are categorized as 'Other'.

After updating the `gender` column, the subsequent `SELECT` statement retrieves the updated `gender` values from the `mental_health_survey` table.

### SQL Query
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
Here, the `ROUND()` function rounds the result of the `AVG()` function to the nearest integer and the `AVG()` function calculates the average age of all respondents in the dataset.

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
This query cuonts the occurrences of work interference reported by both self-employed and non-self-employed respondents.

### Query Explanation
For this query, the `FILTER` clause is used to selectively count only the rows that meet the specified condition within the `COUNT()` function. It ensures that only instances where `work_interfere` is marked as `Often` are included in the count, while other instances are excluded.

### SQL Query
```sql
SELECT
    self_employed,
    COUNT(*) FILTER (WHERE work_interfere = 'Often') AS interference_count
FROM
    mental_health_survey
GROUP BY
    self_employed;
```
### The Output
| self-employed | interference_count |
|---------------|--------------------|
|            No |                114 | 
|            NA |                  2 | 
|           Yes |                 28 |

### 4. Total respondents with treatment
This query counts the total number of respondents who have sought treatment for mental health issues.

#### Query Explanation
This query works by first filtering the rows from the `mental_health_survey` table where the `treatment` column equals `Yes`. Then, it counts the occurrences of these filtered rows using the `COUNT()` function. It assigns the result to the alias `total_respondents_with_treatment`, providing the count of respondents who reported receiving treatment for mental health issues.

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
This query uses a subquery within the `SELECT` statement to calculate the total number of respondents who have sought treatment for mental health issues. It then divides this count by the total number of respondents in the table to determine the percentage of respondents who sought treatment.

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
This query helps us understand the prevalence of mental health treatment-seeking behavior across different countries. By calculating the treatment rate for each country, we can see how many respondents sought treatment for mental health issues relative to the total number of respondents in each country.

#### Query Explanation
Here, the inner subquery calculates the treatment rate for each country represented in the table, which is done by grouping the data by country and counting the total number of respondents `(COUNT(*))` and the number of respondents who sought treatment `(SUM(CASE WHEN treatment = 'Yes' THEN 1 ELSE 0 END))`.

Then, the treatment rate is calculated as the percentage of respondents who sought treatment `(SUM(treatment = 'Yes'))` out of the total number of respondents for each country. This calculation is expressed as `SUM(CASE WHEN treatment = 'Yes' THEN 1 ELSE 0 END) * 100 / COUNT(*)`.

After the treatment rates have been calculated for each country in the subquery, the outer query selects the country and treatment rate columns from the subquery result. Then, the `RANK()` window function assigns a rank to each country based on their treatment rates. We use the `RANK()` function to order the countries by treatment rate in descending order and assign a rank to each row accordingly.

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
In this query, data from the `mental_health_survey` table is grouped based on whether respondents have mental health benefits `(benefits = 'Yes')` or not. For each group, the query calculates the average treatment percentage. This is achieved by averaging the value of 1 for respondents who sought treatment `(treatment = 'Yes')` and 0 for those who didn't within each group.

The `CASE` statement categorizes respondents into two groups - `With Benefits` if they have mental health benefits and `Without Benefits` if they don't. For each benefit status group, the query calculates the treatment percentage by averaging the values obtained from the previous step and multiplying by 100 to express it as a percentage.

The result set includes the benefit status `benefit_status` and the corresponding treatment percentage `treatment_percentage` for each group.

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
Here, we examine the correlation between having a family history of mental health issues and the likelihood of seeking treatment. The query calculates the percentage of individuals who have sought treatment among those with and without a family history of mental health issues.

### Query Explanation
In this query, the common table expression (CTE) named `family_history_count` is defined. Within this CTE, the data is grouped based on the presence or absence of a family history of mental health issues among respondents. This grouping is achieved using the `GROUP BY` clause applied to the family_history column.

The main query then selects data from the CTE and includes the family history status `family_history`, the total count of respondents `total_count`, the count of respondents who sought treatment `treatment_count`, and the percentage of treated respondents out of the total count `percentage_treated`. The percentage of treated respondents is calculated by dividing the treatment count by the total count and multiplying by 100 `(treatment_count * 100) / total_count AS percentage_treated`.

Practically, this query allows us to analyze the correlation between having a family history of mental health issues and the likelihood of seeking treatment. By aggregating data based on family history status and calculating the treatment percentage for each group, we can understand the impact of family history on treatment-seeking behavior.

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
This query analyzes data from the `mental_health_survey` table, specifically focusing on respondents' anonymity status regarding seeking help for mental health issues. It starts by grouping the data based on anonymity status, where respondents are categorized as either having anonymity `anonymity = 'Yes'` or not. Within each group, the query calculates three key metrics.

Firstly, it computes the total count of respondents `total` within each anonymity category. Then, it determines the count of respondents who feel comfortable seeking help for mental health issues `seek_help_yes` within each anonymity group.

Subsequently, the query calculates the percentage of respondents who sought help `seek_help_yes` out of the total count `total` within each anonymity category. This is accomplished by dividing the count of respondents who sought help by the total count and multiplying the result by 100 to express it as a percentage.

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

