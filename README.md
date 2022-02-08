# Pewlett-Hackard Employee Database Analysis

## Project Overview

The main purpose of this project was to use PostgreSQL and PgAdmin 4 to engineer a new database of employee data stored in .csv files and use queries to isolate the employees that are eligible for retirement. Additionally, to provide insight into specific deparments, a Sales and Reproduction department lists were generated.

### Resources:
- Software: PostgreSQL v13.5, PgAdmin 4
- Data: 
  - [departments.csv](https://github.com/jonathantree/Pewlett-Hackard-Analysis/blob/main/Analysis_Projects/Pewlett_Hackard_Analysis/Data/departments.csv)
  - [dept_emp.csv](https://github.com/jonathantree/Pewlett-Hackard-Analysis/blob/main/Analysis_Projects/Pewlett_Hackard_Analysis/Data/dept_emp.csv) 
  - [dept_manager.csv](https://github.com/jonathantree/Pewlett-Hackard-Analysis/blob/main/Analysis_Projects/Pewlett_Hackard_Analysis/Data/dept_manager.csv) 
  - [employees.csv](https://github.com/jonathantree/Pewlett-Hackard-Analysis/blob/main/Analysis_Projects/Pewlett_Hackard_Analysis/Data/employees.csv) 
  - [salaries.csv](https://github.com/jonathantree/Pewlett-Hackard-Analysis/blob/main/Analysis_Projects/Pewlett_Hackard_Analysis/Data/salaries.csv) 
  - [titles.csv](https://github.com/jonathantree/Pewlett-Hackard-Analysis/blob/main/Analysis_Projects/Pewlett_Hackard_Analysis/Data/titles.csv)

To develope the schema used in the database construction, a entity relationship diagram was built using [QuickDBD](https://app.quickdatabasediagrams.com/#/).
![ERD](/Analysis_Projects/Pewlett_Hackard_Analysis/EmployeeDB.png)

Using these relationships, the following schema was developed to engineer a SQL database of these data in PostgreSQL. The employee number as the primary key.

```SQL
/*
-- Creating tables for PH-EmployeeDB
*/

drop table departments cascade;
CREATE TABLE departments (
     dept_no VARCHAR(4) NOT NULL,
     dept_name VARCHAR(40) NOT NULL,
     PRIMARY KEY (dept_no),
     UNIQUE (dept_name)
);
--copy public.departments (dept_no, dept_name) FROM 'C:/Users/jonat/UO_BOO~1/Mod7/GIT_PE~1/PEWLET~1/ANALYS~1/PEWLET~1/Data/DEPART~1.CSV' CSV HEADER QUOTE '\"' ESCAPE '''';
--drop table employees cascade;

CREATE TABLE employees (
	 emp_no INT NOT NULL,
     birth_date DATE NOT NULL,
     first_name VARCHAR NOT NULL,
     last_name VARCHAR NOT NULL,
     gender VARCHAR NOT NULL,
     hire_date DATE NOT NULL,
     PRIMARY KEY (emp_no)
);

--drop table dept_manager cascade;

CREATE TABLE dept_manager (
	  dept_no VARCHAR(4) NOT NULL,
    emp_no INT NOT NULL,
    from_date DATE NOT NULL,
    to_date DATE NOT NULL,
	  FOREIGN KEY (emp_no) REFERENCES employees (emp_no),
	  FOREIGN KEY (dept_no) REFERENCES departments (dept_no),
    PRIMARY KEY (emp_no, dept_no)
);

--drop table salaries cascade;

CREATE TABLE salaries (
	  emp_no INT NOT NULL,
	  salary INT NOT NULL,
	  from_date DATE NOT NULL,
	  to_date DATE NOT NULL,
	  FOREIGN KEY (emp_no) REFERENCES employees (emp_no),
	  PRIMARY KEY (emp_no)
);

--drop table dept_employees cascade;

CREATE TABLE dept_employees (
	  emp_no INT NOT NULL,
	  dept_no VARCHAR(4) NOT NULL,
	  from_date DATE NOT NULL,
	  to_date DATE NOT NULL,
	  FOREIGN KEY (emp_no) REFERENCES employees (emp_no),
  	FOREIGN KEY (dept_no) REFERENCES departments (dept_no),
	  PRIMARY KEY (emp_no, dept_no)
);

--drop table titles cascade;

CREATE TABLE titles (
	  emp_no INT NOT NULL,
	  title VARCHAR NOT NULL,
	  from_date DATE NOT NULL,
	  to_date DATE NOT NULL,
    FOREIGN KEY (emp_no) REFERENCES employees (emp_no)
);
```
#### Retirement Eligibility
Out of the 331,603 employees at the company, 72,458 are eligible for retirement. Three additional lists were constructed to break down this portion of their workforce.
1. Employee Information: A list of employees containing their unique employee number, 
	their last name, first name, gender, and salary
2. Management: A list of managers for each department, 
	including the department number, name, and the manager's employee number, 
	last name, first name, and the starting and ending employment dates
3. Department Retirees: An updated current_emp list that includes everything 
	it currently has, but also the employee's departments

```SQL
-- List 1 of employee information
SELECT e.emp_no,
    e.first_name,
    e.last_name,
    e.gender,
    s.salary,
    de.to_date
INTO emp_info
FROM employees as e
INNER JOIN salaries as s
ON (e.emp_no = s.emp_no)
INNER JOIN dept_employees as de
ON (e.emp_no = de.emp_no)
WHERE (e.birth_date BETWEEN '1952-01-01' AND '1955-12-31')
    AND (e.hire_date BETWEEN '1985-01-01' AND '1988-12-31')
    AND (de.to_date = '9999-01-01');
```
First four results in Table 1.

| emp_no | first_name | last_name | gender | salary | to_date    |
|--------|------------|-----------|--------|--------|------------|
| 10001  | Georgi     | Facello   | M      | 60117  | 9999-01-01 |
| 10004  | Chirstian  | Koblick   | M      | 40054  | 9999-01-01 |
| 10009  | Sumant     | Peac      | F      | 60929  | 9999-01-01 |
| 10018  | Kazuhide   | Peha      | F      | 55881  | 9999-01-01 |

```SQL
-- List 2 of managers per department

SELECT  dm.dept_no,
        d.dept_name,
        dm.emp_no,
        ce.last_name,
        ce.first_name,
        dm.from_date,
        dm.to_date
INTO manager_info
FROM dept_manager AS dm
    INNER JOIN departments AS d
        ON (dm.dept_no = d.dept_no)
    INNER JOIN current_emp AS ce
        ON (dm.emp_no = ce.emp_no);
```
First four results in Table 2.
| dept_no | dept_name        | emp_no | last_name    | first_name | from_date  | to_date    |
|---------|------------------|--------|--------------|------------|------------|------------|
| d003    | Human Resources  | 110183 | Ossenbruggen | Shirish    | 1985-01-01 | 1992-03-21 |
| d004    | Production       | 110386 | Kieras       | Shem       | 1992-08-02 | 1996-08-30 |
| d007    | Sales            | 111133 | Zhang        | Hauke      | 1991-03-07 | 9999-01-01 |
| d008    | Research         | 111534 | Kambil       | Hilary     | 1991-04-08 | 9999-01-01 |


```SQL
-- List 3 of department retirees modified from previous list

SELECT ce.emp_no,
	ce.first_name,
	ce.last_name,
	d.dept_name
INTO dept_info
FROM current_emp as ce
INNER JOIN dept_employees AS de
	ON (ce.emp_no = de.emp_no)
INNER JOIN departments AS d
	ON (de.dept_no = d.dept_no);

SELECT ce.emp_no,
	ce.first_name,
	ce.last_name,
	d.dept_name
INTO dev_sales_dept_info
FROM current_emp as ce
INNER JOIN dept_employees AS de
	ON (ce.emp_no = de.emp_no)
INNER JOIN departments AS d
	ON (de.dept_no = d.dept_no)
WHERE de.dept_no IN ('d007', 'd005')

SELECT * FROM dev_sales_dept_info
ORDER by dept_name;
SELECT * FROM dev_sales_dept_info

```
First four results in Table 3.
| emp_no | first_name | last_name | dept_name   |
|--------|------------|-----------|-------------|
| 10001  | Georgi     | Facello   | Development |
| 10018  | Kazuhide   | Peha      | Development |
| 10053  | Sanjiv     | Zschoche  | Sales       |
| 10066  | Kwee       | Schusler  | Development |


## Challenge Overview

## Results

### Deliverable 1

### Deliverable 2

### Deliverable 3

## Summary
