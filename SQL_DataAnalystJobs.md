# Data Analyst Jobs 

## Exploring the dataset using SQL

I use this data from Kaggle [click here](https://www.kaggle.com/datasets/intelai/data-analysis-jobs) which is titled as Data Analysis Jobs, a dataset from  MUHAMMAD ABDULAZIZ. Dataset contains job postings available on the City of New York’s official jobs site.

### The platform the I've used is MySQL

I want to import the data which is in csv file into the MySQL platform. So the first step is to create Database and a table

```sql
CREATE DATABASE DataAnalystInfo;
USE DataAnalystInfo;
CREATE TABLE data_analyst_jobs(
	job_id INT,
    agency VARCHAR(255),
    posting_type VARCHAR(255),
    num_of_positions INT,
    business_title VARCHAR(255),
    civil_service_title VARCHAR(255),
    title_code_no VARCHAR(50),
    level VARCHAR(50),
    salary_range_from FLOAT,
    salary_range_to FLOAT,
    salary_frequency VARCHAR(255),
    work_location TEXT,
    division_work_unit	TEXT,			
	job_description TEXT,		
	min_qual_requirements TEXT,		
	preferred_skills TEXT,		
	additional_info TEXT,			
	to_apply TEXT,			
	hours_shift	TEXT,			
	work_location_1	VARCHAR(255),	
	recruitment_contact VARCHAR(255),		
	residency_requirement TEXT,			
	post_until DATE
);
```

The next step is to import the data from csv file to MySQL using LOAD DATA INFILE command

```sql
LOAD DATA INFILE 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/data_analysis_jobs.csv'  -- I put the data on the secure-file-priv directory to load it.
INTO TABLE data_analyst_jobs
FIELDS TERMINATED BY ','
OPTIONALLY ENCLOSED BY '"' -- so that the commas inside quotations won't be recognized as delimeter
LINES TERMINATED BY '\n'
IGNORE 1 ROWS -- to ignore the header 
( 
	job_id,
	agency,
	posting_type, 
	num_of_positions,
	business_title,
	civil_service_title,
	title_code_no,
	level,
	salary_range_from, 
	salary_range_to,  
	salary_frequency, 
	work_location, 
	division_work_unit, 
	job_description,  
	min_qual_requirements,
	preferred_skills, 
	additional_info, 
	to_apply, 
	hours_shift, 
	work_location_1 ,
	recruitment_contact, 
	residency_requirement,
	@post_until
)
SET post_until = STR_TO_DATE(@post_until,'%d-%M-%Y'); -- The date format on csv is like this '29-DEC-2021'  and this code convert these values to a correct DATE type format so it can be stored in post_until column
```

After the data was succesfully imported, I want to identify if there are duplicate rows

```sql
SELECT COUNT(*) AS total_rows,COUNT(DISTINCT job_id, 
	agency,
	posting_type, 
	num_of_positions,
	business_title,
	civil_service_title,
	title_code_no,
	level,
	salary_range_from, 
	salary_range_to,  
	salary_frequency, 
	work_location, 
	division_work_unit, 
	job_description,  
	min_qual_requirements,
	preferred_skills, 
	additional_info, 
	to_apply, 
	hours_shift, 
	work_location_1 ,
	recruitment_contact, 
	residency_requirement,
	post_until) AS total_distinct_rows
FROM data_analyst_jobs;
/* the result query shows total_rows 851, total_distinct_rows 837. There are actually 14 duplicates. */
```

In this case, I created a new table to store the unique rows only

```sql
CREATE TABLE data_analyst_jobs_distinct AS
SELECT DISTINCT job_id, 
	agency,
	posting_type, 
	num_of_positions,
	business_title,
	civil_service_title,
	title_code_no,
	level,
	salary_range_from, 
	salary_range_to,  
	salary_frequency, 
	work_location, 
	division_work_unit, 
	job_description,  
	min_qual_requirements,
	preferred_skills, 
	additional_info, 
	to_apply, 
	hours_shift, 
	work_location_1 ,
	recruitment_contact, 
	residency_requirement,
	post_until
FROM data_analyst_jobs;

-- delete the first table
DROP TABLE data_analyst_jobs;

-- rename the second table
RENAME TABLE data_analyst_jobs_distinct TO data_analyst_jobs;
```

Now I can start exploring the table. Upon exploring I've identified that the salary_frequency has two distinct values by running

```sql
SELECT DISTINCT salary_frequency
FROM data_analyst_jobs
```

In order to properly analyze the salary, I need to have a consistent salary_frequency so I computed the "estimated_salary" (this is a custom field I created by averaging the salary_from and salary_to) of records with salary frequency of hourly to annual. 
 
 ```sql
CREATE TEMPORARY TABLE aggregated_table AS -- it is easier to store the query on a table so it can easily be referenced
SELECT   *, 
		(salary_range_From + salary_range_to)/2 AS estimated_salary
FROM data_analyst_jobs
WHERE salary_frequency="Annual" -- excludes salary frequency of 'hourly')
UNION ALL -- there are two distinct values of expected salary "Annual" and "Hourly" So there is a need to adjust the salary on hourly first then UNION them
SELECT   *,
		((salary_range_from + salary_range_to)/2)*8*20*12 AS estimated_salary -- adjusted expected salary to annual
FROM data_analyst_jobs
WHERE salary_frequency="Hourly"; -- excludes salary_frequency of 'Annual'
 ```

Now that I am comfortable with the table that I have, I can start querying from it.
I am interested to know what are the business_title present and how is the pay on each.

```sql
SELECT DISTINCT job_id, -- when DISTINCT is not here, the rows being returned have duplicate like data but not really duplicate, the difference is the posting_type (External/Internal). So It is better to have a DISTINCT to remove duplicate like data.
		business_title,
		estimated_salary
FROM aggregated_table
ORDER BY estimated_salary;

/* To group the data by business title  */

SELECT DISTINCT job_id, 
		business_title,
		AVG(estimated_salary)
FROM aggregated_table
GROUP BY business_title
ORDER BY estimated_salary;

```


The five lowest average estimated salary are the: 
- College Aide
- Computer Programmer Analyst College Aide
- Research Intern
- Programs Intern
- WSCP SUPPORT STAFF. 
I can see similarities here, all job title seems to be entry levels.

The five highest average estimated salary are the: 
- COMPUTER SYSTEMS MANAGER
- Assistant Commissioner
- Chief Technology Officer
- Information Security Specialist
- Executive Director
I can see a common thing to them which is being in a high position  ( Have Comissioner, Chief, Executive, Director on business titles) , which is no surprise.

In order to gain more insights, instead of grouping them by business titles, 
I grouped them by the agency so I can see which agency has the highest/lowest average expected salary. 

```sql
SELECT DISTINCT job_id, 
		agency,
		estimated_salary
FROM aggregated_table
ORDER BY estimated_salary;

/*To group the data by agency */
SELECT DISTINCT job_id, 
		agency,
		AVG(estimated_salary)
FROM aggregated_table
GROUP BY agency
ORDER BY AVG(estimated_salary);

```

The five lowest average estimated salary are the:
- DISTRICT ATTORNEY RICHMOND COU
- DEPARTMENT OF PROBATION
- DEPARTMENT OF BUILDINGS
- ADMIN TRIALS AND HEARINGS
- DEPARTMENT OF INVESTIGATION

The five highest average estimated salary are the: 
- DEPARTMENT OF CORRECTION
- NYC EMPLOYEES RETIREMENT SYS
- DEPARTMENT OF SANITATION
- DEPARTMENT OF TRANSPORTATION
- BOARD OF CORRECTION

Honestly, I don't see any similarities of departments within lowest and highest. 
But I can hypothesize that
- maybe the role of analyst in the agencies with the higher average estimated salary is more integral compare to other agencies, thus they are compensated more and the average expected salary is bigger. 
- And also maybe there are more proportion of people with higher job title in agencies with high average estimated salary compare to other agencies.

We can actualy test my second hypothesis by running the following query

``` sql
SELECT DISTINCT job_id, 
		business_title,
        agency,
		estimated_salary
FROM aggregated_table
WHERE agency="DEPARTMENT OF CORRECTION"
ORDER BY estimated_salary DESC;
/*
Top 5 business title results in DEPARTMENT OF CORRECTION agency:
-Senior Advisor to the Chief of Department
-Senior Advisor to the Bureau Chief of Operations
-Senior Advisor to the FDC, Legal & Policy
-Senior Advisor to the Senior Deputy Commissioner
-Assistant Commissioner, Strategic Initiatives
*/

SELECT DISTINCT job_id, 
		business_title,
        agency,
		estimated_salary
FROM aggregated_table
WHERE agency="NYC EMPLOYEES RETIREMENT SYS"
ORDER BY estimated_salary DESC;
/*
Top 5 business title results NYC EMPLOYEES RETIREMENT SYS agency:
-COMPUTER SYSTEMS MANAGER
-COMPUTER SYSTEMS MANAGER
-COMPUTER SPECIALIST (SOFTWARE)
-CERTIFIED IT DEVELOPER (APPLICATIONS)
-COMPUTER SPECIALIST (SOFTWARE)
*/

SELECT DISTINCT job_id, 
		business_title,
        agency,
		estimated_salary
FROM aggregated_table
WHERE agency="DEPARTMENT OF SANITATION"
ORDER BY estimated_salary DESC;
/*
Top 5 business title results in DEPARTMENT OF SANITATION agency:
-Java Developer
-Java Developer
-.Net/Javascript Developer
-.Net Developer
-.Net Developer
*/

SELECT DISTINCT job_id, 
		business_title,
        agency,
		estimated_salary
FROM aggregated_table
WHERE agency="DEPARTMENT OF TRANSPORTATION"
ORDER BY estimated_salary DESC;
/*
Top 5 business title results in DEPARTMENT OF TRANSPORTATION agency:
-Chief Technology Officer
-Senior IT Architect 12-103
-Network Engineer
-IT Project Specialist 6-124
-IT Project Specialist 7-128
*/

SELECT DISTINCT job_id, 
		business_title,
        agency,
		estimated_salary
FROM aggregated_table
WHERE agency="BOARD OF CORRECTION"
ORDER BY estimated_salary DESC;
/*
Top 5 business title results in DEPARTMENT OF TRANSPORTATION agency:
-Deputy Executive Director of Oversight and Evaluation
-Director of Public Accountability
-Director of Environmental Safety
-Director of Programming and Community Support
-Director of Physical and Mental Wellbeing
*/

```

From the result , we can say that it strongly supports my second hypothesis.
