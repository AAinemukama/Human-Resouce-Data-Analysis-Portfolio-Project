# Human Resource Data Analysis Project

## Overview
This project involves cleaning, standardizing, and analyzing human resource data using SQL Server. The goal is to provide a comprehensive outlook of the company's workforce, examining various demographic factors and employment trends.

## Key Objectives
- Analyze employee demographics (gender, age, race/ethnicity)
- Examine workforce distribution across locations and departments
- Investigate employee turnover and retention rates
- Track changes in employee count over time

## Data Source
- Kaggle

## Technologies Used
- SQL Server for analysis
- Power Bi for data Visualization(refer to the data visualization repository)

## Data Preparation
The initial steps involved data cleaning and standardization:
- Altering column data types for dates (birthdate, hire_date, termdate)
- Creating derived columns (age, full_name, age_group)

## Description of fields
- Employment ID as id
- Date of birth as birthdate
- Gender as gender
- Race as race
- Department as department
- Job Title as jobtitle
- Location of office or work place as loation
- Date of hire of employee as hire_date
- Date of termination  of employee as termdate1
- City of location of work place as location_city
- Age of employee as age
- Age group of employee as age_group

## Key Analyses

1. Gender distribution
2. Race/ethnicity breakdown
3. Age distribution
4. Remote vs. headquarters employee count
5. Average length of employment for terminated employees
6. Gender distribution across departments
7. Job title distribution
8. Departmental turnover rates
9. Employee distribution by location (city and state)
10. Employee count changes over time
11. Tenure distribution by department
12. Years of service distribution

# SQL Queries used
## Data Preparation and Standardization

### Retriveing all data from the HR table to provide a complete view of the dataset.
```sql

select *
from hr
```

### Ensuring Date Columns Have a Consistent DATE Data Type
```sql

select birthdate
from hr

alter table hr
alter column birthdate date
go

select hire_date 
from hr

alter table hr
alter column hire_date date
go

select termdate
from hr

alter table hr
add termdate1 date

update hr
set termdate1= cast(termdate as date)
where termdate is not null
```

### Adding and Populating Age and Full Name Columns
```sql

alter table hr
add age int

update hr
set age=datediff(year, birthdate,getdate()) 

alter table hr
add full_name varchar(100)

update hr
set full_name=concat(first_name,' ', last_name)

```
### Retrieve Employees with Upcoming Contract Expirations
```sql

select *
from hr where termdate1 > getdate() ---we take it that this is the period when their contracts expire
```

## QUESTIONS TO BE ANSWERED USING THE ANALYSIS

### 1. What is the gender breakdown of employees in the company?
```sql

select gender,count(gender) as Countpergender
from hr
where termdate1 > getdate() or termdate1 is null
group by gender
order by gender
```

### 2. What is the race/ethnicity breakdown of employees in the company? These queries help in understanding the diversity of the workforce by providing insights into the racial and ethnic composition of both the overall employee base and the current employees.

#### Overall 
```sql

select race, count(race) as countperrace
from hr
group by race
order by countperrace desc
```
#### Current Employees
```sql

select race, count(race) as countperrace
from hr
where termdate1 > getdate() or termdate1 is null
group by race
order by countperrace desc
```

### 3. What is the age distribution of employees in the company?
#### Age Range of Current employees (This query retrieves the youngest and oldest ages among current employees ,excluding those with past termination dates).
```sql

select min(age) as youngest, max(age) as oldest
from hr
where termdate1 > getdate() or termdate1 is null
```

#### Grouping Employees by Age
##### This section describes two methods for grouping employees into different age categories and analyzing the distribution. These methods provide insights into the age distribution of current employees, helping to understand the demographic composition of the workforce.

###### Method A: Using a subquery
```sql

select age_group, count(*) as group_count 
from
(
select age,
  case
  when age>=18 and age <=24 then '18-24'
  when age>=25 and age <=34 then '25-34'
  when age>=35 AND age <=44 then '35-44'
  when age>=45 AND age <=54 then '45-54'
  when age>=55 AND age <=64 then '55-64'
  else '65+'
  end as age_group
from hr
where termdate1 > getdate() or termdate1 is null
) as subquery
group by age_group
order by age_group
```

###### Methode B: Creating a New Column 
- Add Age Group Column
```sql
alter table hr
add age_group nvarchar(20)
```

- Populate Age Group Column
```sql

update hr
set age_group =
  case
  when age>=18 and age <=24 then '18-24'
  when age>=25 and age <=34 then '25-34'
  when age>=35 AND age <=44 then '35-44'
  when age>=45 AND age <=54 then '45-54'
  when age>=55 AND age <=64 then '55-64'
  else '65+'
  end
```
- Count Employees in Each Age Group
```sql

select age_group, count(age_group) as age_groupcount
from hr
where termdate1 > getdate() or termdate1 is null
group by age_group
order by age_group desc
```

- Count Employees in Eacg Age group by Gender
```sql

select age_group, gender, count(*) as countofagegrppergender
from hr
where termdate1 > getdate() or termdate1 is null
group by age_group,gender
order by age_group,gender
```

### 4. How many employees work at the headquarters vs remote locations?
#### Employee Distribution by Location
##### This query counts the number of current employees (excluding those with past termination dates) at each location, providing insights into the distribution of the workforce across different locations
```sql

select location, count(*) as countperlocation
from hr
where termdate1 > getdate() or termdate1 is null
group by location
```

### 5. What is the average length of employment for employees who have been terminated?
#### Average Length of Employment for Terminated Employees-This helps in understanding the typical duration of employment for those who have left the company
```sql

select avg(datediff(year,hire_date,termdate1)) as avgyrsofemplymnt
from hr
where termdate1 <= getdate() and termdate1 is not null
```

### 6. How does the gender distribution vary across departments?
#### Gender Distribution by Department
```sql

select department, gender, count(gender) as countdept
from hr
where termdate1 > getdate() or termdate1 is null
group by department,gender
order by department
```

### 7. What is the distribution of job titles across the company?
#### Job Title Distribution- This helps in understanding the prevalence of different job titles among the current workforce.
```sql

select jobtitle, count(jobtitle) as countjobtitle
from hr
where termdate1 > getdate() or termdate1 is null
group by jobtitle
order by countjobtitle desc
```

### 8. Which department has the highest turnover rate?
#### Turnover Rate by Department- This helps in identifying which departments have the highest turnover rates, providing insights into employee retention and departmental stability.
```sql

select department,total_count,terminated_count,(terminated_count *100)/total_count as turnover_rate 
from (
select department,
	count(department) as total_count, sum(case 
	when termdate1 <= getdate() and termdate is not null then 1 else 0 end) as terminated_count
    from hr
    group by department) as subquery
    order by turnover_rate desc
```

### 9. What is the distribution of employees across locations by city and state?
#### Employee Distribution by State- This helps in understanding how employees are distributed across various states
```sql

select location_state, count(*) as countbystate
from hr
where termdate1 > getdate() or termdate1 is null
group by location_state
order by countbystate desc
```

### 10. How has the company's employee count changed over time based on hire and term dates?
#### Annual Hires and Terminations Analysis- This helps in understanding the overall hiring and retention trends over the years.
```sql

select year, hires, terminations, hires-terminations as net_change, (hires-terminations)*100/hires as net_change_percent
from(
select YEAR(hire_date) as year, count(*) as hires, sum(case when termdate1<=getdate() and termdate1 is not null then 1 else 0 end) as terminations
from hr
group by YEAR(hire_date)
) as subquery	
order by year asc
```

### 11. What is the tenure distribution for each department?
#### Tenure Distribution by Department-  It helps in understanding the tenure distribution across different departments, highlighting which departments have longer or shorter average employee tenures
```sql

select department, avg(datediff(year,hire_date,termdate1)) as avgyrsofemplymnt
from hr
where termdate1 <= getdate() and termdate1 is not null
group by department
order by avgyrsofemplymnt desc
```
### 12. How many employees have worked with the comapany for each specific number of years?
#### Employee Tenure Distribution- Understanding the tenure distribution helps in planning for future workforce needs and developing effective retention strategies.
```sql
select datediff(year, hire_date, getdate()) as years_of_service,count(*) as employee_count
from hr
where termdate1 > getdate() or termdate1 is null
group by datediff(year, hire_date, getdate())
order by years_of_service desc
```
## Data Visualization
- Click the link to download the Power BI dashboard (https://github.com/AAinemukama/Data-Visualizations-Power-BI-and-Excel/blob/main/Human_Resource.pbix)
