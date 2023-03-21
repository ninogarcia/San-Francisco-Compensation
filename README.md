# Case Study - San Francisco Compensation Using Python, SQL and Tableau

## Introduction

The San Francisco Salaries case study involves a dataset of salaries for public sector employees in the city of San Francisco.

The City of San Francisco maintains a public database of employee compensation data, which is updated annually and made available to the public for transparency and accountability purposes.

This case study will follow the 5 phases of the Data Analysis process: Ask, Prepare, Process, Analyze, and Act (APPAA) to explore the factors that contribute to employee compensation in San Francisco.

By analyzing the dataset, we can gain insights into the distribution of salaries across different job titles, the average compensation for different levels of experience, the correlation between years of experience and salaries, and the pension debt for different job titles.

The findings from this case study can be used by policymakers and public sector organizations to ensure fair and equitable compensation for all public sector employees in San Francisco.

## ASK

In this case study, we will explore the San Francisco Salaries dataset to analyze different factors that contribute to employee compensation. The dataset includes a range of variables, such as job titles, total pay, years of experience, and pension debt, making it a valuable resource for understanding the compensation of public sector employees. To achieve this goal, we will address the following questions:

1. What are the top 10 job titles with the highest number of employee?
2. What is the distribution of salaries of top 10 job titles with the highest number?
3. What is the average basic pay by job title and year between 2017 and 2021?
4. What are the top 5 job titles with the highest average base pay in 2021 for the job titles that have at least 50 employees?
5. What are the top 10 job titles with the highest number of employees in 2021 that have an average total pay and benefits above the overall average?
6. Which job titles had the highest average BasePay in 2021?
7. What was the average BasePay for employees in the three job titles with the highest number of employees in 2021?
8. How many employees had a base pay less than the average base pay for their job title in each year from 2017 to 2021?
9. How many employees had a total pay and benefits greater than $200,000 in each year from 2017 to 2021?
10. Which job title had the highest average OvertimePay in 2021?

By answering these questions, we can gain valuable insights into the factors that contribute to employee compensation in San Francisco and identify areas where interventions may be necessary to ensure fair and equitable compensation for all public sector employees.

## PROCESS

### Cleaning and Preparation of data for analysis

#### USING PYTHON 

Read the CSV file
```python
import pandas as pd

df = pd.read_csv(r'...\sf-salaries.csv')
```
&nbsp;

Check for nulls
```python
df.isnull().sum() / len(df) * 100
```
&nbsp;

Remove spaces between words in each column
```python
df.columns = df.columns.str.replace(' ', '')
```
&nbsp;

Drop unessential columns
```python
df = df.drop(['EmployeeName', 'Notes', 'Agency'], axis=1)
```
&nbsp;

Remove rows with 0 and negative values in the 'BasePay' column
```python
df = df[df['BasePay'] > 0]
```
&nbsp;

Save the cleaned data to a new CSV file
```python
df.to_csv('SF_Salaries_cleaned.csv', index=False)
```
&nbsp;

## ANALYZE

We analyzed the results of our queries and discovered the following insights:

#### USING MySQL 

1. What are the top 10 job titles with the highest number of employee?
```sql
SELECT JobTitle, COUNT(*) AS TotalEmployees
FROM Salaries
GROUP BY JobTitle
ORDER BY TotalEmployees DESC
LIMIT 10;
```
&nbsp;

2. What is the distribution of salaries of top 10 job titles with the highest number?
```sql
SELECT JobTitle, COUNT(*) AS TotalEmployees, 
    MIN(BasePay) AS MinSalary, MAX(BasePay) AS MaxSalary,
    AVG(BasePay) AS AvgSalary, STDDEV(BasePay) AS StdDevSalary
FROM Salaries
WHERE JobTitle IN (
    SELECT JobTitle
    FROM (
        SELECT JobTitle, COUNT(*) AS TotalEmployees
        FROM Salaries
        GROUP BY JobTitle
        ORDER BY TotalEmployees DESC
        LIMIT 10
    ) AS top_jobs
)
GROUP BY JobTitle
ORDER BY TotalEmployees DESC;
```
&nbsp;

3. What is the average basic pay by job title and year between 2017 and 2021?
```sql
SELECT JobTitle, Year, ROUND(AVG(BasePay), 2) AS AverageBasicPay
FROM salaries
WHERE JobTitle IN (
  SELECT JobTitle
  FROM (
    SELECT JobTitle, COUNT(*) AS TotalEmployees
    FROM salaries
    GROUP BY JobTitle
    ORDER BY TotalEmployees DESC
  ) AS top_jobs
)
AND Year BETWEEN '2017' AND '2021'
GROUP BY JobTitle, Year
ORDER BY JobTitle, Year;
```
&nbsp;

4. What are the top 5 job titles with the highest average base pay in 2021 for the job titles that have at least 50 employees?
```sql
SELECT JobTitle, AVG(BasePay) AS AverageBasePay
FROM salaries
WHERE JobTitle IN (
  SELECT JobTitle
  FROM (
    SELECT JobTitle, COUNT(*) AS TotalEmployees
    FROM salaries
    WHERE Year = '2021'
    GROUP BY JobTitle
    HAVING TotalEmployees >= 50
  ) AS top_jobs
)
GROUP BY JobTitle
ORDER BY AverageBasePay DESC
LIMIT 5;
```
&nbsp;

5. What are the top 10 job titles with the highest number of employees in 2021 that have an average total pay and benefits above the overall average?
```sql
SELECT JobTitle, COUNT(*) AS TotalEmployees
FROM salaries
WHERE JobTitle IN (
  SELECT JobTitle
  FROM (
    SELECT JobTitle, AVG(TotalPay&Benefits) AS AverageTotalPayBenefits
    FROM salaries
    WHERE Year = '2021'
    GROUP BY JobTitle
    HAVING AverageTotalPayBenefits > (
      SELECT AVG(TotalPay&Benefits)
      FROM salaries
    )
  ) AS top_jobs
)
GROUP BY JobTitle
ORDER BY TotalEmployees DESC
LIMIT 10;
```
&nbsp;

6. Which job titles had the highest average BasePay in 2021?
```sql
SELECT JobTitle, ROUND(AVG(BasePay),2) AS AvgBasePay
FROM salaries
WHERE JobTitle IN (
    SELECT JobTitle
    FROM (
        SELECT JobTitle, COUNT(*) AS TotalEmployees
        FROM salaries
        GROUP BY JobTitle
        ORDER BY TotalEmployees DESC
        LIMIT 10
    ) AS top_jobs
)
AND Year = 2021
GROUP BY JobTitle
ORDER BY AvgBasePay DESC;
```
&nbsp;

7. What was the average BasePay for employees in the three job titles with the highest number of employees in 2021?
```sql
SELECT ROUND(AVG(BasePay),2) AS AvgBasePay
FROM salaries
WHERE JobTitle IN (
    SELECT JobTitle
    FROM (
        SELECT JobTitle, COUNT(*) AS TotalEmployees
        FROM salaries
        WHERE Year = 2021
        GROUP BY JobTitle
        ORDER BY TotalEmployees DESC
        LIMIT 3
    ) AS top_jobs
)
AND Year = 2021;
```
&nbsp;

8. How many employees had a base pay less than the average base pay for their job title in each year from 2017 to 2021?
```sql
SELECT Year, JobTitle, COUNT(*) AS NumEmployeesBelowAvgBasePay
FROM (
  SELECT Year, JobTitle, BasePay, AVG(BasePay) OVER (PARTITION BY JobTitle) AS AvgBasePay
  FROM salaries
  WHERE Year BETWEEN '2017' AND '2021'
) AS base_pay_by_jobtitle
WHERE BasePay < AvgBasePay
GROUP BY Year, JobTitle
ORDER BY 2,1;
```
&nbsp;

9. How many employees had a total pay and benefits greater than $200,000 in each year from 2017 to 2021?
```sql
SELECT Year, COUNT(*) AS NumEmployeesOver200K
FROM (
  SELECT Year, Status, (BasePay + OvertimePay + OtherPay + Benefits + PensionDebt) AS TotalPayAndBenefits
  FROM salaries
) AS pay_and_benefits
WHERE TotalPayAndBenefits > 200000 AND Year BETWEEN '2017' AND '2021' AND Status = 'FT'
GROUP BY Year;
```
&nbsp;

10. Which job title had the highest average OvertimePay in 2021?
```sql
SELECT JobTitle, ROUND(AVG(OvertimePay),2) AS AvgOvertimePay
FROM salaries
WHERE JobTitle IN (
    SELECT JobTitle
    FROM (
        SELECT JobTitle, COUNT(*) AS TotalEmployees
        FROM salaries
        WHERE Year = 2020
        GROUP BY JobTitle
        ORDER BY TotalEmployees DESC
        LIMIT 10
    ) AS top_jobs
)
AND Year = 2021
GROUP BY JobTitle
ORDER BY AvgOvertimePay DESC
LIMIT 1;
```


## ACT

### Conclusion:
Based on our analysis of the salaries dataset, we can conclude that the top job titles with the highest number of employees are "Transit Operator", "Special Nurse", and "Registered Nurse". Additionally, we found that the distribution of salaries among the top 10 job titles is quite wide, with some job titles having a large range of salaries. We also found that the average basic pay varies by job title and year, with some job titles experiencing fluctuations in average pay over time.

### Recommendation:
Our analysis suggests that organizations should focus on ensuring that employees are paid fairly and equitably across job titles, particularly for job titles that have a large number of employees. Additionally, organizations should consider implementing measures to address the wide range of salaries among job titles and ensure that there is transparency and consistency in pay practices. Finally, organizations should regularly review salary data to identify any trends or fluctuations in average pay by job title and year, and adjust their compensation strategies as needed to remain competitive in the job market.
