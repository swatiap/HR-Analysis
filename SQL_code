create database if not exists human_resources;

use human_resources;

-- Create 'departments' table
CREATE TABLE departments (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50),
    manager_id INT
);

-- Create 'employees' table
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50),
    hire_date DATE,
    job_title VARCHAR(50),
    department_id INT REFERENCES departments(id)
);

-- Create 'projects' table
CREATE TABLE projects (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50),
    start_date DATE,
    end_date DATE,
    department_id INT REFERENCES departments(id)
);

-- Insert data into 'departments'
INSERT INTO departments (name, manager_id)
VALUES ('HR', 1), ('IT', 2), ('Sales', 3);

-- Insert data into 'employees'
INSERT INTO employees (name, hire_date, job_title, department_id)
VALUES ('John Doe', '2018-06-20', 'HR Manager', 1),
       ('Jane Smith', '2019-07-15', 'IT Manager', 2),
       ('Alice Johnson', '2020-01-10', 'Sales Manager', 3),
       ('Bob Miller', '2021-04-30', 'HR Associate', 1),
       ('Charlie Brown', '2022-10-01', 'IT Associate', 2),
       ('Dave Davis', '2023-03-15', 'Sales Associate', 3);

-- Insert data into 'projects'
INSERT INTO projects (name, start_date, end_date, department_id)
VALUES ('HR Project 1', '2023-01-01', '2023-06-30', 1),
       ('IT Project 1', '2023-02-01', '2023-07-31', 2),
       ('Sales Project 1', '2023-03-01', '2023-08-31', 3);
       
UPDATE departments
SET manager_id = (SELECT id FROM employees WHERE name = 'John Doe')
WHERE name = 'HR';

UPDATE departments
SET manager_id = (SELECT id FROM employees WHERE name = 'Jane Smith')
WHERE name = 'IT';

UPDATE departments
SET manager_id = (SELECT id FROM employees WHERE name = 'Alice Johnson')
WHERE name = 'Sales';

-----------------------------------------------------------------------------------------
# extracting all the data present into 'departments' table
select * from departments;

# extracting all the data present into 'employees' table
select * from employees;

# extracting all the data present into 'projects' table
select * from projects;

################################################# Human Resources ################################################

/*
1. Find the longest ongoing project for each department.
*/
select d.name as 'Department Name', p.name as 'Project Name', timestampdiff(month, p.start_date, p.end_date) as 'Duration(in months)'
from projects as p
inner join departments as d on p.department_id = d.id
order by timestampdiff(month, p.start_date, p.end_date) desc;

-----------------------------------------------------------------------------------------
/*
2. Find all employees who are not managers.
*/
select e.name as 'Employee Name', e.job_title as 'Designation'
from employees as e
inner join departments as d on e.department_id = d.id
where e.id not in (d.manager_id);

-----------------------------------------------------------------------------------------
/*
3. Find all employees who have been hired after the start of a project in their department.
*/
select e.name as 'Employee Name', e.hire_date as 'Hire Date', p.start_date as 'Project Start Date' 
from employees as e 
inner join departments as d on e.department_id = d.id
inner join projects as p on d.id = p.department_id
where e.hire_date > p.start_date
order by e.name;

----------------------------------------------------------------------------------------
/*
4. Rank employees within each department based on their hire date (earliest hire gets the highest rank).
*/
with emp_rank as(
					select e.id as e_id, d.id, e.hire_date, rank() over (partition by d.id order by e.hire_date) as e_rank
                    from employees as e 
                    inner join departments as d on e.department_id = d.id
                    )
select e.name as 'Employee Name', d.name as 'Department Name', er.hire_date, er.e_rank as 'Rank'
from employees as e 
inner join departments as d on e.department_id = d.id
inner join emp_rank as er on e.id = er.e_id
order by d.name;

---------------------------------------------------------------------------------------
/*
5. Find the duration between the hire date of each employee and the hire date of the next employee hired in the same department.
*/
with emp_cte as(
			select e.id as e_id, e.hire_date, e.department_id,
            lead(e.id) over (partition by e.department_id order by e.hire_date) as next_emp_id,
            lead(e.name) over (partition by e.department_id order by e.hire_date) as next_emp_name,
            lead(e.hire_date) over(partition by e.department_id order by e.hire_date) as next_emp_hire_date
            from employees as e
)
select e.name as 'Employee Name', e.hire_date as 'Hire Date', e_cte.next_emp_name as 'Next Employee Name', e_cte.next_emp_hire_date as 'Next Employee Hire Date', 
concat(
	timestampdiff(year, e.hire_date, e_cte.next_emp_hire_date), ' years ',
    timestampdiff(month, e.hire_date, e_cte.next_emp_hire_date) % 12, ' months ',
    timestampdiff(day, e.hire_date, e_cte.next_emp_hire_date) % 30, ' days '
    ) as 'Duration Difference(in month)'
from employees as e
inner join emp_cte as e_cte on e.id = e_cte.e_id 
where e_cte.next_emp_name is not null;
