# Pewlett-Hackard Employee Database Analysis

## Project Overview

The main purpose of this project was to use PostgreSQL and PgAdmin 4 to engineer a new database of employee data stored in .csv files and use queries to isolate the employees that are eligible for retirement. Additionally, to provide insight into specific deparments, a Sales and Development department lists were generated.

### Resources

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

Using these relationships, the following schema was developed to engineer a SQL database of these data in PostgreSQL. The employee number is the primary key.

```SQL
/*
-- Creating tables for PH-EmployeeDB
*/

CREATE TABLE departments (
      dept_no VARCHAR(4) NOT NULL,
      dept_name VARCHAR(40) NOT NULL,
     PRIMARY KEY (dept_no),
     UNIQUE (dept_name)
);

CREATE TABLE employees (
 emp_no INT NOT NULL,
      birth_date DATE NOT NULL,
      first_name VARCHAR NOT NULL,
      last_name VARCHAR NOT NULL,
      gender VARCHAR NOT NULL,
      hire_date DATE NOT NULL,
     PRIMARY KEY (emp_no)
);

CREATE TABLE dept_manager (
 dept_no VARCHAR(4) NOT NULL,
     emp_no INT NOT NULL,
     from_date DATE NOT NULL,
     to_date DATE NOT NULL,
    FOREIGN KEY (emp_no) REFERENCES employees (emp_no),
    FOREIGN KEY (dept_no) REFERENCES departments (dept_no),
    PRIMARY KEY (emp_no, dept_no)
);

CREATE TABLE salaries (
   emp_no INT NOT NULL,
   salary INT NOT NULL,
   from_date DATE NOT NULL,
   to_date DATE NOT NULL,
    FOREIGN KEY (emp_no) REFERENCES employees (emp_no),
    PRIMARY KEY (emp_no)
);

CREATE TABLE dept_employees (
   emp_no INT NOT NULL,
   dept_no VARCHAR(4) NOT NULL,
   from_date DATE NOT NULL,
   to_date DATE NOT NULL,
    FOREIGN KEY (emp_no) REFERENCES employees (emp_no),
    FOREIGN KEY (dept_no) REFERENCES departments (dept_no),
    PRIMARY KEY (emp_no, dept_no)
);

CREATE TABLE titles (
    emp_no INT NOT NULL,
    title VARCHAR NOT NULL,
    from_date DATE NOT NULL,
    to_date DATE NOT NULL,
    FOREIGN KEY (emp_no) REFERENCES employees (emp_no)
);
```

#### Retirement Eligibility

- Out of the 331,603 employees at the company, 72,458 are eligible for retirement. 

Three additional lists were constructed to break down this portion of their workforce.

1. Employee Information: A list of employees containing their unique employee number, their last name, first name, gender, and salary
2. Management: A list of managers for each department, including the department number, name, and the manager's employee number, last name, first name, and the starting and ending employment dates
3. Department Retirees: An updated current_emp list that includes everything it currently has, but also the employee's departments

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

Now that the database has been constructed with tables that are useful from joins on the original six tables, the task is to create two new lists. These lists will contribute to developing a mentor program that eligible employees can be selected from and a enter mentorship with a retiring employee.

1. The first was to create a Retirement Titles table that holds all the titles of employees who were born between January 1, 1952 and December 31, 1955. Because some employees may have multiple titles in the database—for example, due to promotions, the `DISTINCT ON` statement was used to create a table that contains the most recent title of each employee. Then, the `COUNT()` function was used to create a table that has the number of retirement-age employees by most recent job title. Finally, because we want to include only current employees in our analysis, a final filter on the data was used that excluded those employees who have already left the company.

```SQL
SELECT DISTINCT ON (rt.emp_no) rt.emp_no,
rt.first_name,
rt.last_name,
rt.title

INTO unique_titles
FROM retirement_titles AS rt
WHERE (rt.to_date = '9999-01-01')
ORDER BY rt.emp_no, rt.to_date DESC;

SELECT COUNT(ut.title), ut.title
INTO count_retiring_titles
FROM unique_titles as ut
GROUP BY ut.title
ORDER BY ut.count DESC;
```

2. Create a mentorship-eligibility table that holds the current employees who were born between January 1, 1965 and December 31, 1965.

```SQL
SELECT DISTINCT ON(e.emp_no) e.emp_no,
    e.first_name,
    e.last_name,
    e.birth_date,
    de.to_date,
    de.from_date,
    t.title

INTO mentorship_eligibility
FROM employees as e
INNER JOIN titles as t
    ON (e.emp_no = t.emp_no)
INNER JOIN dept_employees as de
    ON (e.emp_no = de.emp_no)
WHERE (de.to_date = '9999-01-01')
    AND (e.birth_date BETWEEN '1965-01-01' AND '1965-12-31')
ORDER BY e.emp_no;
```

## Results

### Table 4. The Number of Retiring Employees by Title

![Table 6](/Analysis_Projects/Pewlett_Hackard_Analysis/retirement_titles_head.png)

### Table 5. The Employees Eligible for the Mentorship Program

![Table 5](/Analysis_Projects/Pewlett_Hackard_Analysis/mentorship_eligibility_head.png)

## Summary

1. 72,458 are eligible for retirement and their positions will need to be filled. A breakdown of the number of positions by title is provided in the subsequent table.

### Table 6. Count of total retiring employee positions that will need to be fillled

| count | title              |
|-------|--------------------|
| 25916 | Senior Engineer    |
| 24926 | Senior Staff       |
| 9285  | Engineer           |
| 7636  | Staff              |
| 3603  | Technique Leader   |
| 1090  | Assistant Engineer |
| 2     | Manager            |

2. The total number of employees that were eligible for the mentorship program using the criteria stated above is only 1,549. This means that through the mentorship program, only 2% of the openings will be fulfilled if every employee on that list enters the program.

### *The large gap of employees available internally to fill these positons means that different considerations need to be made to see if there are more employees that will be able to fill these roles*

### Additional Queries

Changing the eligibility for the mentorship program to a five year span `BETWEEN '1960-01-01' AND '1965-12-31'` rather than just a one year span yields a total of 93,756 employees that could enter the program. With these considerations for eligibility, there could be enough employees to fill the retiring positions and allow for some employees who may not be eligible for other reason other than their age such as qualifications. The following query generates this list:

```SQL
SELECT DISTINCT ON(e.emp_no) e.emp_no,
    e.first_name,
    e.last_name,
    e.birth_date,
    de.to_date,
    de.from_date,
    t.title

INTO mentorship_eligibility_2
FROM employees as e
INNER JOIN titles as t
    ON (e.emp_no = t.emp_no)
INNER JOIN dept_employees as de
    ON (e.emp_no = de.emp_no)
WHERE (de.to_date = '9999-01-01')
    AND (e.birth_date BETWEEN '1960-01-01' AND '1965-12-31')
ORDER BY e.emp_no;
```

To better understand how many of these employees from this new eligibility currently occupy roles that qualify them for promotion, the following query was can be performed:

```SQL
SELECT COUNT(me.title), me.title
INTO count_eligible_titles
FROM mentorship_eligibility_2 as me
GROUP BY me.title
ORDER BY me.count DESC;
```

### Table 7. Number of employees for each title from the new eligibility list

| count | title              |
|-------|--------------------|
| 33069 | Engineer           |
| 31975 | Senior Staff       |
| 9975  | Staff              |
| 9374  | Senior Engineer    |
| 4735  | Assistant Engineer |
| 4618  | Technique Leader   |
| 10    | Manager            |

Table 6.
| count | title              |
|-------|--------------------|
| 25916 | Senior Engineer    |
| 24926 | Senior Staff       |
| 9285  | Engineer           |
| 7636  | Staff              |
| 3603  | Technique Leader   |
| 1090  | Assistant Engineer |
| 2     | Manager            |

Conclusions:

1. There could be enough engineers that are able to fill the senior engineer positions.
2. There are not enough staff to fill the senior staff positions.
3. There are not enough assistant engineers to fill the Engineer postion.

There are an insufficient number of employees to fill every position and a massive external hiring effort will be needed to fill this void.
