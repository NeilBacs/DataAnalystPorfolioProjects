### The platform that I've used is SQL Server

I've imported the cleaned excel file into SQL Server.



Now I can start exploring the table. Upon exploring I've identified that the salary_frequency has two distinct values by running

```sql
SELECT DISTINCT Salary_Frequency
FROM Data_Analyst_Jobs;

--OUTPUTS Hourly and Annual
```

In order to properly analyze the salary, I need to have a consistent salary_frequency so I computed the "estimated_salary" (this is a custom field I created by averaging the salary_from and salary_to) of records with salary frequency of hourly to annual. 
 
 ```sql
CREATE View Unified_Salary  AS -- it is easier to store the query on a view so it can easily be referenced
SELECT   *, 
		(salary_range_From + salary_range_to)/2 AS estimated_salary
FROM data_analyst_jobs
WHERE salary_frequency='Annual' -- excludes salary frequency of 'hourly'
UNION ALL -- there are two distinct values of expected salary "Annual" and "Hourly" So there is a need to adjust the salary on hourly first then UNION them
SELECT   *,
		((salary_range_from + salary_range_to)/2)*8*20*12 AS estimated_salary -- adjusted expected salary to annual
FROM data_analyst_jobs
WHERE salary_frequency='Hourly'; -- excludes salary_frequency of 'Annual'

 ```
 


Now that I am comfortable with the data that I have, I can start querying from it.
I am interested to know what are the business_title present and how is the pay on each.

```sql
SELECT  DISTINCT business_title, estimated_salary
FROM Unified_Salary
ORDER BY estimated_salary;

/* To group the data by business title  */

SELECT business_title, AVG(estimated_salary)
FROM Unified_Salary
GROUP BY business_title
ORDER BY AVG(estimated_salary);

```


The five **lowest** average estimated salary are the: 
- Computer Programmer Analyst College Aide
- College Aide
- Programs Intern
- Research Intern
- WSCP SUPPORT STAFF. 

I can see similarities here, all job title seems to be **entry levels**.

We can output the highest estimated salary on top by just adding DESC in order by clause

```sql
SELECT business_title, AVG(estimated_salary)
FROM Unified_Salary
GROUP BY business_title
ORDER BY AVG(estimated_salary) DESC;

```

The five **highest** average estimated salary are the: 
- Assistant Commissioner
- Chief Technology Officer
- Information Security Specialist
- Executive Director, Portfolio Planning and Management
- COMPUTER SYSTEMS MANAGER

I can see a common thing to them which is being in a **high position**  ( Have Manager, Comissioner, Chief, Executive, Director on business titles) , which is no surprise.

In order to gain more insights, instead of grouping them by business titles, 
I grouped them by the agency so I can see which agency has the highest/lowest average estimated salary. 

```sql

SELECT agency, AVG(estimated_salary)
FROM Unified_Salary
GROUP BY agency
ORDER BY AVG(estimated_salary);

```

The five **lowest** average estimated salary are the:
- DISTRICT ATTORNEY RICHMOND COU
- DEPARTMENT OF PROBATION
- DEPARTMENT OF BUILDINGS
- ADMIN TRIALS AND HEARINGS
- DEPARTMENT OF INVESTIGATION

The five **highest** average estimated salary are the: 
- DEPARTMENT OF CORRECTION
- NYC EMPLOYEES RETIREMENT SYS
- DEPARTMENT OF SANITATION
- DEPARTMENT OF TRANSPORTATION
- BOARD OF CORRECTION

Honestly, I don't see any similarities of departments within lowest and highest. 
But I can **hypothesize** that
- maybe the role of analyst in the agencies with the higher average estimated salary is more integral compare to other agencies, thus they are compensated more and the average expected salary is bigger. 
- And also maybe there are more proportion of people with higher job title in agencies with high average estimated salary compare to other agencies.

We can actualy test my second hypothesis by looking at the business titles who earn the most on each agency.

Let's check first on the agencies with the top 5 average estimated salary.
``` sql
SELECT DISTINCT job_id,
	business_title,
        agency,
	estimated_salary
FROM Unified_Salary
WHERE agency='DEPARTMENT OF CORRECTION'
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
FROM Unified_Salary
WHERE agency='NYC EMPLOYEES RETIREMENT SYS'
ORDER BY estimated_salary DESC;
/*
Top 5 business title results NYC EMPLOYEES RETIREMENT SYS agency:
-COMPUTER SYSTEMS MANAGER
-COMPUTER SYSTEMS MANAGER (differ on estimated salary on the top)
-COMPUTER SPECIALIST (SOFTWARE)
-CERTIFIED IT DEVELOPER (APPLICATIONS)
-COMPUTER SPECIALIST (SOFTWARE)
*/

SELECT DISTINCT job_id, 
		business_title,
        	agency,
		estimated_salary
FROM Unified_Salary
WHERE agency='DEPARTMENT OF SANITATION'
ORDER BY estimated_salary DESC;
/*
Top 5 business title results in DEPARTMENT OF SANITATION agency:
-Java Developer
-Java Developer (differ on estimated salary on the top)
-.Net/Javascript Developer
-.Net Developer
-.Net Developer (differ on Job ID on the top)
*/

SELECT DISTINCT job_id, 
		business_title,
        	agency,
		estimated_salary
FROM Unified_Salary
WHERE agency='DEPARTMENT OF TRANSPORTATION'
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
FROM Unified_Salary
WHERE agency='BOARD OF CORRECTION'
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

Now let's check on the agencies with the bottom 5 average estimated salary.
```sql
SELECT DISTINCT job_id, 
		business_title,
        	agency,
		estimated_salary
FROM Unified_Salary
WHERE agency='DISTRICT ATTORNEY RICHMOND COU'
ORDER BY estimated_salary DESC;
/*
Top business title results in DISTRICT ATTORNEY RICHMOND COU agency:
(Returned only 1 row)
- Body Worn Camera Analyst
*/


SELECT DISTINCT job_id, 
		business_title,
        	agency,
		estimated_salary
FROM Unified_Salary
WHERE agency='DEPARTMENT OF PROBATION'
ORDER BY estimated_salary DESC;
/*
Top business title results in DEPARTMENT OF PROBATION agency:
(Returned only 2 rows)
- Investigator Analyst
- Human Resources Specialist
*/


SELECT DISTINCT job_id, 
		business_title,
        	agency,
		estimated_salary
FROM Unified_Salary
WHERE agency='DEPARTMENT OF BUILDINGS'
ORDER BY estimated_salary DESC;
/*
Top business title results in DEPARTMENT OF BUILDINGS agency:
(Returned only 1 row)
- Budget Analyst
*/

SELECT DISTINCT job_id, 
		business_title,
        	agency,
		estimated_salary
FROM Unified_Salary
WHERE agency='ADMIN TRIALS AND HEARINGS'
ORDER BY estimated_salary DESC;
/*
Top business title results in ADMIN TRIALS AND HEARINGS agency:
(Returned only 1 row)
- Procurement Analyst
*/

SELECT DISTINCT job_id, 
		business_title,
        	agency,
		estimated_salary
FROM Unified_Salary
WHERE agency='DEPARTMENT OF INVESTIGATION'
ORDER BY estimated_salary DESC;
/*
Top business title results in DEPARTMENT OF INVESTIGATION agency:
(Returned only 1 row)
- Senior Programmer
- Data Analyst
- Policy Analystt
*/
```

**We can infer from the result that my second hypothesis is substantially supported.**
