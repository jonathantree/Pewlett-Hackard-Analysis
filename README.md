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

```SQL

``



## Challenge Overview

## Results

### Deliverable 1

### Deliverable 2

### Deliverable 3

## Summary
