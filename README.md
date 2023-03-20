# Case Study - San Francisco Compensation Using Python, SQL and Tableau

## Introduction

The San Francisco Salaries case study involves a dataset of salaries for public sector employees in the city of San Francisco.

The City of San Francisco maintains a public database of employee compensation data, which is updated annually and made available to the public for transparency and accountability purposes.

This case study will follow the 5 phases of the Data Analysis process: Ask, Prepare, Process, Analyze, and Act (APPAA) to explore the factors that contribute to employee compensation in San Francisco.

By analyzing the dataset, we can gain insights into the distribution of salaries across different job titles, the average compensation for different levels of experience, the correlation between years of experience and salaries, and the pension debt for different job titles.

The findings from this case study can be used by policymakers and public sector organizations to ensure fair and equitable compensation for all public sector employees in San Francisco.

## ASK

In this case study, we will explore the San Francisco Salaries dataset to analyze different factors that contribute to employee compensation. The dataset includes a range of variables, such as job titles, total pay, years of experience, and pension debt, making it a valuable resource for understanding the compensation of public sector employees. To achieve this goal, we will address the following questions:

1. What are the job titles with the highest number of employees?
2. What is the distribution of salaries across different job titles?
3. What is the average salary for each job title?
4. What is the highest and lowest salary for each job title?
5. What is the total pay for each employee?
6. What is the average total pay for each job title?
7. Is there a correlation between years of experience and salaries for different job titles in San Francisco?
8. What is the average pension debt for each job title?

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

3. What is the average salary for each job title?
```sql
SELECT JobTitle, AVG(TotalPay) AS AverageSalary
FROM Salaries
GROUP BY JobTitle
ORDER BY AverageSalary DESC;
```
&nbsp;

4. What is the highest salary for each job title?
```sql
SELECT JobTitle, MAX(TotalPay) AS HighestSalary
FROM Salaries
GROUP BY JobTitle;
```
&nbsp;

5. What is the lowest salary for each job title?
```sql
SELECT JobTitle, MIN(TotalPay) AS LowestSalary
FROM Salaries
GROUP BY JobTitle;
```
&nbsp;

6. What is the total pay for each employee?
```sql
SELECT JobTitle, TotalPay
FROM Salaries;
```
&nbsp;

7. What is the average total pay for each job title?
```sql
SELECT JobTitle, AVG(TotalPay) AS AverageTotalPay
FROM Salaries
GROUP BY JobTitle
ORDER BY AverageTotalPay DESC;
```
&nbsp;

8. What is the correlation between years of experience and salaries for different job titles in San Francisco?
```sql
SELECT JobTitle, AVG(TotalPay), AVG(YearsExperience) AS AvgExperience, 
       CORR(TotalPay, YearsExperience) AS Correlation
FROM Salaries
GROUP BY JobTitle
ORDER BY Correlation DESC;
```
&nbsp;

9. What is the average pension debt for each job title?
```sql
SELECT JobTitle, AVG(PensionDebt) AS AveragePensionDebt
FROM Salaries
GROUP BY JobTitle
ORDER BY AveragePensionDebt DESC;
```
&nbsp;
