Census Data Analysis with Google Cloud PostgreSQL and Tableau

Project Overview

This project involves setting up a PostgreSQL server on Google Cloud, pulling data from [data.census.gov](https://data.census.gov), and using this data to create visualizations with Tableau. The analysis focuses on exploring the relationships between salary, age, and rent over work day start time, segmented by race.


1. Google Cloud PostgreSQL Server Setup:
Created a Google Cloud Project in the Google Cloud Console and enabled the Cloud SQL Admin API
Created a PostgreSQL Instance and set the region to Northern Virginia 
Configure the Instance with an instance ID as well as password
Authorized Network Access by adding my IP address to the authorized networks 
Connected with pgAdmin using the provided IP, username and password.
2.  Data Sources:
   - Data was sourced from [data.census.gov](https://data.census.gov).
   - The datasets used include:
     - Census Data - Race Time Age
     - Census Data - Race Time Rent
     - Census Data - Race Time Salary

3. Data Cleaning and Transformation
Imported all three csv files into google sheets
Data was cleaned and transformed using Google Sheets  ensuring consistency and usability.
Added Null Values to where there were blanks initially 
Exploratory data analysis to understand and relationships

4. Database Creation

Created a database that would make querying fast easy and simple.

```
CREATE TABLE races (
    race_id SERIAL PRIMARY KEY,
    race_name VARCHAR(255) NOT NULL
);

CREATE TABLE times (
    time_id SERIAL PRIMARY KEY,
    year INTEGER NOT NULL
);

CREATE TABLE salaries (
    salary_id SERIAL PRIMARY KEY,
    race_id INTEGER NOT NULL,
    time_id INTEGER NOT NULL,
    salary NUMERIC NOT NULL,
    FOREIGN KEY (race_id) REFERENCES races(race_id),
    FOREIGN KEY (time_id) REFERENCES times(time_id)
);

CREATE TABLE ages (
    age_id SERIAL PRIMARY KEY,
    race_id INTEGER NOT NULL,
    time_id INTEGER NOT NULL,
    age INTEGER NOT NULL,
    FOREIGN KEY (race_id) REFERENCES races(race_id),
    FOREIGN KEY (time_id) REFERENCES times(time_id)
);

CREATE TABLE rents (
    rent_id SERIAL PRIMARY KEY,
    race_id INTEGER NOT NULL,
    time_id INTEGER NOT NULL,
    rent NUMERIC NOT NULL,
    FOREIGN KEY (race_id) REFERENCES races(race_id),
    FOREIGN KEY (time_id) REFERENCES times(time_id)
);
```

Data was then loaded by using the PSQL tool:

Example:

```
psql -c "\COPY races(race_name) FROM '/path/to/Census Data - Race Time Age.csv' DELIMITER ',' CSV HEADER;"

```

5. Data Analysis 

The analysis focused on exploring the trends and relationships between salary, age, and rent over time, segmented by race. 
Example Query: 

```
WITH SalaryData AS (
    SELECT 
        s.time_period_id,
        s.race_id,
        s.salary,
        tp.time_period,
        rc.race
    FROM 
        census_salary s
    JOIN 
        race rc ON s.race_id = rc.id
    JOIN 
        time_period tp ON s.time_period_id = tp.id
    WHERE 
        rc.race = 'White alone'
)
SELECT 
    race,
    MAX(salary) AS max_salary,
    MIN(salary) AS min_salary,
    MAX(time_period) FILTER (WHERE salary = (SELECT MAX(salary) FROM SalaryData)) AS max_salary_period,
    MIN(time_period) FILTER (WHERE salary = (SELECT MIN(salary) FROM SalaryData)) AS min_salary_period
FROM 
    SalaryData
GROUP BY 
    race;

```
This query returned the following:

<img width="686" alt="image" src="https://github.com/trevorlovet1/Census-Data-Project/assets/112558354/371bd08f-0d26-4dd0-bd09-521265f7ced2">

The data above shows that according to the survey: The data points where White Americans made the most money was an 8:30 AM start time and they made the least amount of money with a work day start at 4:05 PM

See below a similar query with the race selected as African American:

<img width="736" alt="image" src="https://github.com/trevorlovet1/Census-Data-Project/assets/112558354/889781ca-0208-48a9-ba72-0fc2cf7bc033">

The data above shows that according to the survey: The data points where Black Americans made the most money was an 11:30 PM start time and they made the least amount of money with a work day start at 5:05 PM

What about Asian Americans?

<img width="686" alt="image" src="https://github.com/trevorlovet1/Census-Data-Project/assets/112558354/033ac1b1-02aa-4835-83e0-fbbae33f572b">

The data above shows that according to the survey: The data points where Asian Americans made the most money was an 7:25 PM start time and they made the least amount of money with a work day start at 4:55 PM

One data point here for each race provides some insight but it would be helpful to have the top 5-6 data points for each race



