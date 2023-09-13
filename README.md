# Data_Analysis_Project

### Project Overview
The Data Analysis in this project will consist of importing data for Data Cleaning, then addressing various predefined questions related to this dataset. Following that, Data Visualization will be performed to present the dataset's information.

### Tools
- MySQL for Data Cleaning and Analysis
- PowerBI for Cleating report

### Data Cleaning and Preparation
1. In the process of Data Cleaning, import the Human Resources Data into the MySQL program.
2. Then, inspect all the data in the Human Resources Data using the SQL command SELECT * FROM hr; .
3. Rectify any errors found in the table, such as:
 - Incorrect column names.
 - Checking data structure and data type.
 - Changing the data type of columns that should be Dates from TEXT to DATE.
 - Standardizing the Date format in the entire table.
 - Calculating the age of employees and creating a new age column.
 - Verifying and correcting in column age.
4. After making corrections and updating the dataset, proceed with the analysis and answer any questions.
   
### Questions
1. What is the gender breakdown of employees in the company ?
2. What is the race/ethinicity breakdown  of employee in the company ?
3. What is the age distribution of employees in the company ?
4. How many employees work at headquarters versus remote locations ?
5. What is the average length of employment for employees who have been terminated ?
6. How dose the gender distribution very across departments and job title ?
7. What are the top 5 most common job titles across the company?
8. Which department has the highest turnover rate ?
9. What is the distribution  of employees across location by city and state ?
10. How has the company's employees  count change over time base on hire and term date ?
11. What is the tenure distribution for each department ?

### Data Cleaning in MySQL
Check dataset
```sql
USE project_01 ; -- Select dataset to use
SELECT * FROM hr; -- Show all data for check overall of data
DESCRIBE hr; -- Show database structure
```
Edit table
```sql
ALTER TABLE hr -- for change column name
CHANGE COLUMN ๏ปฟid emp_id VARCHAR(20) NULL;

SELECT count(*) FROM hr;
SHOW COLUMNS FROM hr;
SELECT birthdate FROM hr ;

SET sql_safe_updates = 0; -- Allow for run command Update table

-- Update date in table(birthdate and hire_date)
UPDATE hr -- Change form birthdate in YYYY-MM-DD
SET birthdate = CASE
	WHEN birthdate LIKE '%/%' THEN date_format(str_to_date(birthdate,'%m/%d/%Y'),'%Y-%m-%d')
    WHEN birthdate LIKE '%-%' THEN date_format(str_to_date(birthdate,'%m-%d-%Y'),'%Y-%m-%d')
    ELSE NULL
END ;

ALTER TABLE hr -- Change data type of birthdate from text to date
MODIFY COLUMN birthdate DATE ;

UPDATE hr -- Change form hire_date in YYYY-MM-DD
SET hire_date = CASE
	WHEN hire_date LIKE '%/%' THEN date_format(str_to_date(hire_date,'%m/%d/%Y'),'%Y-%m-%d')
    WHEN hire_date LIKE '%-%' THEN date_format(str_to_date(hire_date,'%m-%d-%Y'),'%Y-%m-%d')
    ELSE NULL
END ;

ALTER TABLE hr -- Change data type of hire_date from text to date
MODIFY COLUMN hire_date DATE;

ALTER TABLE hr -- Change data type of hire_date from text to date
MODIFY COLUMN hire_date DATE;

UPDATE hr -- Date information organized
SET termdate = IF(termdate IS NOT NULL AND termdate != '', date(str_to_date(termdate, '%Y-%m-%d %H:%i:%s UTC')), '0000-00-00')
WHERE true;

SELECT termdate from hr;

SET sql_mode = 'ALLOW_INVALID_DATES'; -- for allow date collect or not collect without error

ALTER TABLE hr -- Change data type of termdate from text to date
MODIFY COLUMN termdate DATE;

-- Add colunm age and Calculate age
ALTER TABLE hr ADD COLUMN age INT;

UPDATE hr -- Calculate age by timestampdiff() Ex. TIMESTAMPDIFF(unit, datetime_expr1, datetime_expr2) unit as a year, min, second
SET age = timestampdiff(YEAR, birthdate, CURDATE());

SELECT birthdate, age FROM hr ;

SELECT -- Check employee age
	MIN(age) AS youngest,
    MAX(age) AS oldest
FROM hr ;

SELECT count(*) AS sum_of_age_under_18
FROM hr WHERE age < 18;

SELECT count(*) AS sum_of_age_over_18
FROM hr WHERE age >18;

SELECT count(*) AS worng_age -- Check a minus age in hr
FROM hr WHERE age < 0;

UPDATE hr -- Edit minus age
SET age = ABS(age)
WHERE age < 0 ;
```

### Data Analysis and answer a question in MySQL 

```sql
USE project_01 ;
SELECT * FROM hr;
SET sql_mode = 'ALLOW_INVALID_DATES'; -- for allow date collect or not collect without error

-- 1. What is the gender breakdown of employees in the company ?
SELECT gender, count(*) AS  count
 FROM hr
 WHERE age >= 18 
 GROUP BY gender ;

-- 2. What is the race/ethinicity breakdown  of employee in the company ?
SELECT race, count(*) AS count 
FROM hr
WHERE age >= 18 
GROUP BY race 
ORDER BY COUNT(*) DESC ;

-- 3. What is the age distribution of employees in the company ?
-- Show age of employees
SELECT 
MIN(age) AS youngest,
MAX(age) AS oldest
FROM hr
WHERE age > 18 ;

SELECT 
	CASE 
		WHEN age >=18 AND age <= 24 THEN '18-24'
        WHEN age >=25 AND age <= 34 THEN '25-34'
        WHEN age >=35 AND age <= 44 THEN '35-44'
        WHEN age >=45 AND age <= 54 THEN '45-54'
        WHEN age >=55 AND age <= 64 THEN '55-64'
        ELSE '65+'
	END AS age_group, count(*) AS count
FROM hr
WHERE age > 18 
GROUP BY age_group
ORDER BY age_group;

SELECT -- Distribution of employees by age group and gender.
	CASE 
		WHEN age >=18 AND age <= 24 THEN '18-24'
        WHEN age >=25 AND age <= 34 THEN '25-34'
        WHEN age >=35 AND age <= 44 THEN '35-44'
        WHEN age >=45 AND age <= 54 THEN '45-54'
        WHEN age >=55 AND age <= 64 THEN '55-64'
        ELSE '65+'
	END AS age_group, gender, count(*) AS count
FROM hr
WHERE age > 18 
GROUP BY age_group, gender
ORDER BY age_group, gender;

-- 4. How many employees work at headquarters versus remote locations ?
SELECT location, COUNT(*) AS count
FROM hr 
WHERE age > 18 
GROUP BY location ;

-- 5. What is the average length of employment for employees who have been terminated ?
SELECT
	ROUND(AVG(datediff(termdate,hire_date))/365,2) AS avg_length_employment --  ROUND() is rounding numeric values and show decimal 
FROM hr
WHERE termdate <= curdate() AND age > 18;

-- 6. How dose the gender distribution very across departments and job title ?
SELECT department, gender, COUNT(*) AS count
FROM hr
WHERE age > 18
GROUP BY department, gender
ORDER BY department ;

-- 7. What is the average length of employment for employees who have been terminated ?
SELECT jobtitle, COUNT(*) AS count 
FROM hr
WHERE age >= 18 
GROUP BY jobtitle
ORDER BY COUNT(*) DESC 
LIMIT 5 ;

-- 8. Which department has the highest turnover rate ?
SELECT department,
	total_count,
    terminated_count,
    terminated_count/total_count AS termination_rate
FROM (
 SELECT department,
 COUNT(*) AS total_count,
 SUM(CASE WHEN termdate <> '0000-00-00' AND termdate  <= curdate()THEN 1 ELSE 0 END) AS terminated_count
 FROM hr
 WHERE age >= 18
GROUP BY department
) AS subquery
ORDER BY termination_rate DESC ;

-- 9. What is the distribution  of employees across location by city and state ?
SELECT  location_state, COUNT(*) AS count
FROM hr
WHERE age >= 18 AND termdate = '0000-00-00'
GROUP BY location_state
ORDER BY count DESC ;

-- 10. How has the company's employees  count change over time base on hire and term date ?
SELECT
	YEAR(hire_date) AS year, -- define year for calculate number of hire
    COUNT(*) AS hires,
    SUM(CASE WHEN termdate <> '0000-00-00' AND termdate <= curdate()THEN 1 ELSE 0 END) AS terminations,
    ROUND((COUNT(*) - (CASE WHEN termdate <> '0000-00-00' AND termdate <= curdate() THEN 1 ELSE 0 END))/ (COUNT(*) * 100), 2) AS net_change_percent
-- Used ROUND((hires - terminations)/ (hires * 100), 2) AS net_change_percent >> Error 
FROM hr
WHERE age >= 18
GROUP BY YEAR(hire_date)
ORDER BY YEAR(hire_date) ASC ;

-- 11. What is the tenure distribution for each department ?
SELECT 
department, 
ROUND(AVG(datediff(termdate, hire_date)/365), 0) AS avg_tenure
FROM hr
WHERE  termdate <> '0000-00-00' AND termdate <= curdate()  AND age >= 18
GROUP BY department ;
```

### Answer the question
What has been found from this dataset includes.

Ans 1. The highest number of employees is male, followed by females and then non-conforming. 

Ans 2. From this dataset, it is found that the highest number of employees are of White ethnicity, followed by Asian, Two or More Races, Black or African American, Hispanic or Latino, American Indian or Alaska Native, and the lowest number is Native Hawaiian or Other Pacific Islander.

Ans 3. When categorizing employees by age group, it is observed that the age group 35-44 years has the highest number of employees, which is 2346 people, accounting for 30%.

Ans 4.There are 5813 employees working at Headquarters, while 1891 employees work remotely, making up 75% and 25% respectively.

Ans 5. Average length of employment is 7 and a half years.

Ans 6. The number of male and female employees is roughly equal in every department, but in most cases, the number of male employees is slightly higher.

Ans 7. The top 5 departments with the highest number of employees are Research Assistant II, Business Analyst, Human Resources Analyst II, Research Assistant I, and Account Executive.

Ans 8. The Auditing department has the highest turnover rate.

Ans 9. The majority of employees reside in Ohio, accounting for approximately 60%, while the number of employees in other cities is relatively similar.

Ans 10. The change in the number of employees from 2000 to 2020 shows a gradual decrease in numbers over time.

Ans 11. The average tenure in each department ranges from 5 to 12 years.
