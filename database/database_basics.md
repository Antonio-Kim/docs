## Basic of PostgreSQL

This section summarizes the basics of SQL, specifically focused on PostgreSQL due to its data type. The section consists of:

- Setting up environment
- Create database table
- Retrieving data from table
- Data Types in PostgreSQL

### Setting up Environment

The example used here is through Windows 11. Go to https://www.postgresql.org/download/windows/ and download the installer. Continue clicking on next and once you arrive to component screen, select all of pgAdmin Tool, Stack Builder, and Command Line Tools. PostgreSQL should be selected by default. Next, enter in password, and leave the port number to be 5432, which is the default port for PostgreSQL. Get to end and install all the components. In the Stack Builder, click on PostGIS component, and if applicable, click on Python Extention. At the time of writing, this was not available.

### Create database table

PostgreSQL is a Database Management System - a software that allows users to define, manage, and query data stored in database. Database is collection of objects includes schemas, functions, and etc. When you create database in pgAdmin, you create _Database Server_.

```sql
-- Create Database
CREATE DATABASE analysis;

-- Create Table
CREATE TABLE teachers (
	id bigserial,
	first_name varchar(25),
	last_name varchar(50),
	school varchar(50),
	hire_date date,
	salary numeric
);

-- Add row to table
INSERT INTO teachers Values (first_name, last_name, school, hire_date, salary)
VALUES
('Janet', 'Smith', 'F.D. Roosevelt HS', '2011-10-30', 36200),
('Lee', 'Reynolds', 'F.D. Roosevelt HS', '1993-05-22', 65000),
('Samuel', 'Cole' 'F.D. Roosevelt HS', '2005-08-01', 43500);
```

### Retrieving data from table

```sql
-- View all the data
SELECT * FROM teachers;
TABLE teachers;

-- Retrieve specific columns from table
SELECT last_name, first_name, salary FROM teachers;

-- Sorting Data, DESC and ASC
SELECT first_name, last_name, salary
FROM teachers
ORDER BY salary DESC;

-- Sorting Data, multiple columns
SELECT last_name, school, hire_date
FROM teachers
ORDER BY school ASC, hire_date DESC;

-- Get Unique Values
SELECT DISTINCT school
FROM teachers;

-- Filter row using WHERE
SELECT last_name, school, hire_date
FROM teachers
WHERE school = 'F.D. Roosevelt HS';
```

#### Commonly used operator for filter

| Operator | Function                          | Example                              |
| -------- | --------------------------------- | ------------------------------------ |
| =        | Equal to                          | WHERE school = 'F.D. Roosevelt HS'   |
| !=       | Not equal to                      | Where school != 'F.D. Roosevelt Hs'  |
| >        | Greater than                      | Where salary > 20000                 |
| <        | Less than                         | Where salary <= 60500                |
| >=       | Greater than or equal to          | Where salary >= 20000                |
| <=       | Less than or equal to             | Where salary <= 60500                |
| IN       | Match one of a set of values      | WHERE last_name IN ('Bush', 'Roush') |
| LIKE     | Match a pattern (case sensitive)  | Where first_name LIKE 'Sam%'         |
| ILIKE    | Match a petter (case insensitive) | WHERE first_name ILIKE 'sam%'        |
| NOT      | Negates a condition               | WHERE first_name NOT ILIKE 'sam%'    |

Here the wildcard for LIKE and ILIKE are commonly used with two operators: **%** and **\_**. The % matches one or more characters, whereas \_ matches just one character.

### Data Types in PostgreSQL

Data types in SQL usually fall into three categories: **characters**, **numbers**, and **dates**.

#### Characters

- char(n): fixed length column where regardless of the input, the column will store n characters. PostgreSQL will pad the remaining with spaces. Note: **Outdated type**
- varchar(n): variable length colun where maximum length is specified by n. It will not store extra spaces.
- text: unlimited length of characters. Not part of SQL standard.

NB: There is no performance differences betwee the three types.

#### Numbers

Three standard SQL types for integers are:

- smallint: 2 bytes with range between -32,768 to 32,767
- integer: 4 bytes with range between -2,147,483,648 to 2,147,483,647
- bigint: 8 bytes with range between -9,223,372,036,854,775,808 to 9,223,372,036,854,775,807

Space matters in this type so keep in mind the values you deal with. If you don't need to 2 billion, it's safe to use integer.

We also have auto-increment numbers that increases as you add row to the table.

- smallserial: 2 bytes with range between 1 to 32,767
- serial: 4 bytes with range between 1 to 2,147,483,647
- bigserial: 8 bytes with range between 1 to 9,223,372,036,854,775,807

However, do try using IDENTITY instead of serial. This also auto-increments but IDENTITY is part of ANSI SQL and does not allow users to insert value into IDENTITY Column. IDENTITY has two options:

- GENERATE ALWAYS AS IDENTITY: tells database to always fill column with auto-increment value. A user cannot insert a value into the column unless manually overriding the setting
- GENERATE BY DEFAULTS AS IDENTITY: tells database to fill the oclumn with auto-increment value, but also possibility of duplicate values, which can be problematic.

```sql
CREATE TABLE people (
	id integer GENERATE ALWAYS AS IDENTITY,
	person_name varchar(100)
);
```

Standard SQL types for floating values are:

- numeric, decimal(precision, scale): fixed point, up to 131,072 digits before the decimal points, and up to 16,383 digits after the decimal points
- real: 6 decimal digits precision
- double precision: 15 decimal digits precision

Scale means number of digits right of the decimal, and precision means total digits in the number. Both numeric and decimal are interchangeable.

#### Dates

Here are four major types of dates in PostgreSQL

| Data type | Storage Size | Description    | Range                 |
| --------- | ------------ | -------------- | --------------------- |
| timestamp | 8 bytes      | Date and time  | 4713 BC to 294276 AD  |
| date      | 4 bytes      | Date (no time) | 4613 BC to 5874897 AD |
| time      | 8 bytes      | Time (no date) | 00:00:00 to 24:00:00  |
| interval  | 16 bytes     | Time interval  | +/- 178,000,000 years |

In most cases, dealing with dates will require _with time zone_ added.
