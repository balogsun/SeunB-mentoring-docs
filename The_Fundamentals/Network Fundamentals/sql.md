Introduction to Databases
•	A database is a set of related information
•	Database - A collection of organized data. A set of related information.
•	database systems are computerized data storage and retrieval mechanisms
•	database system stores data electronically rather than on paper
•	Database system is able to retrieve data more quickly, index data in multiple ways, and deliver up-to-the-minute information to its user community.
https://dev.mysql.com/downloads/installer/	
https://dev.mysql.com/downloads/mysql/	
https://www.youtube.com/watch?v=u96rVINbAUI	
https://www.w3schools.com/sql/sql_create_db.asp	

<img width="848" height="398" alt="image" src="https://github.com/user-attachments/assets/c36dec93-7328-4704-acd0-6ece196926e6" />

The Relational Model
•	data is represented as sets of tables 
•	redundant data is used to link records in different tables
Relational databases
•	Relational databases - It defines the relationship between 2 entities
•	SQL is a programming language in database arena
•	SQL is the language for generating, manipulating, and retrieving data from a relational database
•	SQL is used for writing applications, performing administrative tasks, or generating reports
Purpose of SQL:
Structured Query Language (SQL) is a textual language used by DB servers
•	SQL commands are used to perform operations on DB:
INSERT, SELECT, UPDATE, and DELETE
•	Programmers use these commands to manipulate data in the DB
SQL query
•	the result of an SQL query is a table
•	This table is called “result set”
•	a new permanent table can be created in a relational database simply by storing the result set of a query
•	A query can use both permanent tables and the result sets from other queries as inputs
•	Activity: Discuss the security concerns about this feature.
SQL statements
•	SQL schema statements: Statements used to create database objects (tables, indexes, constraints, etc.). 
•	used to define the data structures stored in the database
•	Used by administrator
•	SQL data statements: statements used to create, manipulate, and retrieve the data stored in a database are known as the SQL data statements
•	Used by administrator, programmer or report writer 
•	SQL transaction statements: used to begin, end, and roll back transactions
Launch the mysql tool, and create your database and database user:
6.	Launch a command window and login to MySQL as root: mysql -u root –p
7.	Create a database for the sample data: create database bank; 
8.	Create the lrngsql database user with full privileges on the bank database: 
•	create user 'lrngsql'@'localhost' identified by 'toor’;
•	grant all privileges on bank.* to 'lrngsql'@'localhost;
9.	Exit the mysql tool: quit; 
10.	Log in to MySQL as lrngsql: mysql -u lrngsql –p 
11.	Attach to the bank database: use bank;
You now have a MySQL server, a database, and a database user
create a database tables and populate them with sample data:
12.	Download the script at http://examples.oreilly.com/learningsql/ and save it under  c:\temp\LearningSQLExample.sql
13.	If you have logged out of the mysql tool, Log in to MySQL as lrngsql Attach to the bank database: mysql -u lrngsql -p;    use bank;
14.	Run it from the mysql utility: Type source c:\temp\LearningSQLExample.sql; and press Enter. 

EXPLAINING WHAT SQL INJECTION MEANS. 
An SQL injection is a type of code injection hacking technique in which an attacker uses a piece of structured query language code to alter a database and gain access to valuable discreet information. SQL injection codes have the capability to destroy a database if care is not taken.

<img width="940" height="449" alt="image" src="https://github.com/user-attachments/assets/3aa38024-1ace-4d5f-9818-d3c79b9ed602" />

 
SQL Injection – Risk to DB:
•	SQL injection is a code injection technique that might destroy your database.
•	SQL injection is one of the most common web hacking techniques.
•	SQL injection is the placement of malicious code in SQL statements, via web page input.
•	SQL injection can bypass a firewall and can affect a fully patched system
•	Through SQL Injection attacker can obtain unauthorized access to a database and can create, read, update, alter, or delete data stored in the back-end database
Scope of SQL Injection:
•	Identify injectable parameters.
•	Identify the database type and version.
•	Discover database schema.
•	Extracting data.
•	Insert, modify or delete data.
•	Denial of service to authorized users by locking or deleting tables.
•	Bypassing authentication.
•	Privilege escalation.
•	Execute remote commands by calling stored functions within the DBMS which are reserved for administrators.
Ways to inject SQL:
•	Injected through user input.
•	Injection through cookie fields contains attack strings.
•	Injection through Server Variables.
•	Second-Order Injection where hidden statements are executed at another time by another function.
SQL Injection Example:
 <img width="940" height="646" alt="image" src="https://github.com/user-attachments/assets/217e9f87-85d0-4c12-a935-baaffa59d2aa" />

•	This query returns all employee records with their salaries.
•	The query is designed to retrieve a single employee record.
•	Adding OR 1=1 in the input field will change the SQL query.
•	Since 1=1 is always true, the query returns all employees and their salaries.
Defending against SQL Injection:
The root cause of almost every SQL injection is invalid input checking. Here is a list of prevention methods:
•	Input Validation
•	Input Checking Functions
•	Validate Input Sources
•	Access Rights
Configure database error reporting:
CLASSWORK:

 2. Create a MySQL database and respective tables 
3. Add constraints for attributes 
4. Define Primary and/or Foreign Key(s) for each table 
Create database Columbia_db; 
use columbia_db;
CREATE TABLE doctors (
    StaffID char(8) NOT NULL primary key,
    LastName varchar(255),
    FirstName varchar(255),
    Sex varchar(255),
	Grade varchar(255),
    Address varchar(255)
);
use columbia_db;
CREATE TABLE staff (
    StaffID char(8) NOT NULL primary key,
    LastName varchar(255),
    FirstName varchar(255),
    Sex varchar(255),
	Grade varchar(255),
    Address varchar(255)
	FOREIGN KEY (StaffID) REFERENCES doctors(StaffID)
);

use columbia_db;
CREATE TABLE patients (
    RegNum char(8) NOT NULL primary key,
    LastName varchar(255),
    FirstName varchar(255),
    Sex varchar(255),
	Age int,
    Address varchar(255)
	results varchar(255),
);
create database insurance;
use insurance;
CREATE TABLE clientinfo (
    SerialNo int NOT NULL primary key,
    LastName varchar(255),
    FirstName varchar(255),
	DriverID char (15) NOT NULL unique,
    Address varchar(255) default 'Ontario'
);

create table vehicle_info (
   SerialNo int NOT NULL,
   Model varchar(255),
   Year_of_purchase char(4),
   DriverID char(12) NOT NULL unique,
   Accident_Info varchar(255),
   FOREIGN KEY (SerialNo) REFERENCES clientinfo(SerialNo)
);

foreign key = references the primary key of another table
FOREIGN KEY (SerialNo column in vehicle_info) REFERENCES clientinfo(SerialNo column)
5. Add 5-10 new records in each table 
Add records to the tables using INSERT INTO statement. 

```sql
INSERT INTO staff (StaffID, LastName, FirstName, Sex, Grade, Address) 
VALUES ('1111111', 'romeno', 'uno', 'Male', 'Junior Exec', 'Queens'); 
VALUES ('1111112', 'domuli', 'reno', 'Male', 'Junior Exec', 'Queens'); 
VALUES ('1111113', 'tifani', 'jones', 'Male', 'Junior Exec', 'Finch'); 
VALUES ('1111114', 'jelopa', 'mireen', 'FeMale', 'Junior Exec', 'manitoba'); 
VALUES ('1111115', 'haniff', 'toff', 'FeMale', 'Junior Exec', 'kichener');
```
Paste the output of each table using the syntax. 
SELECT * from doctors;
 <img width="808" height="572" alt="image" src="https://github.com/user-attachments/assets/f0f434cf-0c2b-4a65-8d7c-ec2ab5c38236" />

SELECT * from patients;
 <img width="791" height="611" alt="image" src="https://github.com/user-attachments/assets/df78381c-7289-431d-a9f3-d8260e20f19e" />

SELECT * from staff;
 <img width="940" height="701" alt="image" src="https://github.com/user-attachments/assets/d0ef4cad-5c5e-47cf-a6f6-4a8d2423bcf5" />


## 1. Aggregation by Date in Toad for Oracle
A simple **aggregation** by date, useful for daily volume reports.

* Toad for Oracle, running a simple `SELECT … GROUP BY` to inspect daily transaction counts.

**SQL:**

```sql
SELECT
  SYSDATE,
  COUNT(1),
  tran_date
FROM tbaadm.dtd
GROUP BY tran_date;
```

**What it does:**

1. **`SYSDATE`** returns the current timestamp for each row of output (so you can see when the query ran).
2. **`COUNT(1)`** tallies the number of rows in `tbaadm.dtd` for each distinct `tran_date`.
3. **`GROUP BY tran_date`** collapses all records into one row per date, letting you compare volumes day-by-day.

You ran it repeatedly (“1st check”, “2nd check”, “3rd check”) to verify that:

* The **count** per date stays consistent.
* The **SYSDATE** column changes, confirming successive executions.
 <img width="940" height="501" alt="image" src="https://github.com/user-attachments/assets/2f6f6fad-f765-4d1e-8f23-aaaa8592c39e" />


MySQL QUERIES FUNCTIONS:
Querying data:
SELECT – show you how to use simple SELECT statement to query the data from a single table.

The SELECT statement is used to select data from a database. The data returned is stored in a result table, called the result-set.
Below is a screenshot showing an example of the output of running the select query on the “offices” tables.
 <img width="808" height="531" alt="image" src="https://github.com/user-attachments/assets/d28021d5-bada-4b51-bdbe-a0cb683809ab" />

Sorting data:
ORDER BY – show you how to sort the result set using ORDER BY clause. The custom sort order with the FIELD function will be also covered.
The ORDER BY is used to sort the result-set in ascending or descending order.
Below is a screenshot showing an example of the output of running the “order by” commands and its sorts the employee number in an ascending order.

<img width="826" height="564" alt="image" src="https://github.com/user-attachments/assets/dcb59928-7f5e-405b-8d5f-a368eb2626ad" />

Filtering data:

OR – introduce you to the OR operator and show you how to combine the OR operator with the AND operator to filter data.
IN – show you how to use the IN operator in the WHERE clause to determine if a value matches any value in a list or a subquery.

LIKE – provide you with technique to query data based on a specific pattern.
LIMIT – use LIMIT to constrain the number of rows returned by SELECT statement
IS NULL – test whether a value is NULL or not by using IS NULL operator.
The “WHERE” query is used to filter records. It is used to extract only those records that fulfill a specified condition.
Below is a screenshot showing an example of the output of running the “where” query and thus showing the records of “customerNumber” 112 filtered and listed from the payments table.

<img width="861" height="640" alt="image" src="https://github.com/user-attachments/assets/40695cf1-5bf4-43e0-bc90-ac28414e2b7a" />

The SELECT DISTINCT query: Inside a table, a column often contains many duplicate values; and sometimes you only want to list the distinct values.
Below is a screenshot showing an example showing how this query works.
SELECT DISTINCT paymentdate FROM payments;
 
<img width="792" height="557" alt="image" src="https://github.com/user-attachments/assets/20f94399-9324-43e8-9c89-fbc7251cfdb8" />


The AND operator displays a record if all the conditions separated by AND are TRUE.
The “BETWEEN” operator selects values within a given range. The values can be numbers, text, or dates.
Below is a screenshot showing an example of  how the “AND” and “BETWEEN” statements are used in the same statement.  Its is used to show values between two specific dates range from 31st June 2004 to 31st October 2004.

<img width="940" height="634" alt="image" src="https://github.com/user-attachments/assets/93aedf7a-7a2c-4958-99cd-9700c2f008f4" />

DEFINING A SCHEMA
A database schema defines how data is organized within a relational database. It represents the logical configuration of all or part of a relational database.

EXPLAINING MySQL JOIN COMMANDS AND TYPES OF JOIN COMMANDS
An SQL JOIN is used to combine rows from two or more tables, based on a related column between them.
The INNER JOIN keyword selects records that have matching values in both tables.
The LEFT JOIN keyword returns all records from the left table on the left and the matching records from the table on the right.
Below is an example of the left join query.

<img width="806" height="505" alt="image" src="https://github.com/user-attachments/assets/15af09c1-bcbe-4d1e-9c9d-13d134b4c9fd" />

The RIGHT JOIN query displays all records from the right table and the matching records from the left table.
Below is and example screenshot of the right join query.

<img width="940" height="624" alt="image" src="https://github.com/user-attachments/assets/ddfdf570-e1dd-4a2d-a1a0-eca8f1b38ffb" />

## 2. Self-Join to List Managers and Direct Reports
Demonstrates a **self-join** to reveal hierarchical relationships within a single table.
* The SQL IDE (looks like MySQL Workbench or similar), querying the same `employees` table twice to relate employees to their managers.

**SQL:**

```sql
SELECT
  manager.firstname      AS manager_firstname,
  manager.lastname       AS manager_lastname,
  manager.employeeNumber AS managerId,
  directreport.firstname AS directreport_firstname,
  directreport.lastname  AS directreport_lastname,
  directreport.employeeNumber AS directreportId
FROM 
  employees AS manager,
  employees AS directreport
WHERE 
  manager.employeeNumber = directreport.reportsto;
```

**What it does:**

1. **Two aliases** (`manager`, `directreport`) both point to the same table.
2. The `WHERE` clause matches each employee’s `reportsto` (their manager’s ID) to the manager’s `employeeNumber`.
3. **Result:** a two-column hierarchy list—every row tells you which manager (first four columns) oversees which direct report (last three columns).

<img width="734" height="468" alt="image" src="https://github.com/user-attachments/assets/0d814acb-1b03-4db0-8da3-c60406f2e7f0" />


## 3. Multi-Table JOIN with Aggregation and LEFT JOIN
Combines **multi-table joins**, a **LEFT JOIN**, and **aggregation** to produce per-customer spending summaries—even capturing customers with zero payments.
The SQL IDE, combining `customers`, `orders`, `orderdetails`, and optionally `payments` to compute how much each customer has spent.

**SQL:**

```sql
SELECT
  c.contactFirstname AS Fname,
  c.contactLastname  AS Lname,
  c.postalCode,
  c.creditLimit,
  SUM(od.quantityOrdered * od.priceEach) AS amount_spent
FROM
  customers c
  JOIN orders o              ON c.customerNumber = o.customerNumber
  JOIN orderdetails od       ON o.orderNumber    = od.orderNumber
  LEFT JOIN payments p       ON c.customerNumber = p.customerNumber
GROUP BY
  c.customerName;  -- or c.customerNumber
```

**What it does:**

1. **`JOIN` chain:**

   * `customers` → `orders` ensures you only count customers with orders.
   * `orders` → `orderdetails` multiplies quantity by unit price for each line item.
2. **`LEFT JOIN payments`** includes customers even if they haven’t made any payments yet.
3. **`SUM(...)`** aggregates each customer’s total spending across all their order lines.
4. **`GROUP BY`** collapses everything into one row per customer, returning their name, contact info, credit limit, and total amount spent.

<img width="752" height="493" alt="image" src="https://github.com/user-attachments/assets/906fc89c-5db1-4277-bd39-abd3ed9a78d6" />


THE TIMESTAMP data type
The TIMESTAMP data type allows you to store date and time data including year, month, day, hour, minute and second. 

HOW TO IMPORT DATA INTO MYSQL
Select server tab, next click on “data import” tab, then click on “import from self-contained file”, choose path where the data file resides. Screenshot below.

<img width="940" height="634" alt="image" src="https://github.com/user-attachments/assets/ac42ab22-07f2-41b3-b0d4-0428576f8582" />
 
Start the import:
Refresh the schema button and see new database information on the left tab of the workbench.
See screenshot below:

<img width="940" height="512" alt="image" src="https://github.com/user-attachments/assets/4f3f9593-7191-4473-8ea1-a9cfb4ff359c" />

DEFINING THE ALIAS COMMAND:
Alias is used when you want to temporarily give a table or column another name.
It is also often used to make columns or table names more readable, but they only exist for the duration of the query executed. The AS keyword is used in a query to create an alias. 
ALIASES:
SELECT column_name AS alias_name
FROM table_name;

SELECT column_name(s)
FROM table_name AS alias_name;

SELECT CustomerID AS ID, CustomerName AS Customer
FROM Customers;

OTHER CLI SCENARIOS:
CREATE DATABASE TsomDB;
DROP DATABASE databasename;

BACKUP DATABASE testDB
TO DISK = 'D:\backups\testDB.bak';
WITH DIFFERENTIAL;

CREATE TABLE table_name (
    column1 datatype,
    column2 datatype,
    column3 datatype,
   ....
);

ALTER TABLE students (
    column1 datatype constraint,
    column2 datatype constraint,
    column3 datatype constraint,
    ....
);


use tsomdb;
CREATE TABLE students (
    StudentID int NOT NULL primary key,
    RegNum char(8),
    LastName varchar(255),
    FirstName varchar(255),
    Course varchar(255),
    Address varchar(255),
    City varchar(255),
    age int,
    CONSTRAINT PK_Person PRIMARY KEY (StudentID,LastName)
);

foreign key = references the primary key of another table

SELECT * FROM students;
SELECT StudentID, LastName FROM students;

use tsomdb;
show tables;

show databases;

ALTER TABLE students
ADD Email varchar(255);

ALTER TABLE students
DROP COLUMN Email;

ALTER TABLE students
DROP COLUMN DOB;

ALTER TABLE students
ADD DateOfBirth date;

ALTER TABLE table_name
ALTER TABLE students
RENAME COLUMN old_name to new_name;
RENAME COLUMN DateOfBirth to DOB;

ALTER TABLE students
ALTER COLUMN column_name datatype;
MODIFY COLUMN column_name datatype; (OLDER VERSION)
MODIFY COLUMN DOB year;
MODIFY COLUMN Email varchar(255) not null; 
MODIFY COLUMN  Address varchar(255) set default toronto;


INSERT INTO students (StudentID, RegNum, LastName, FirstName, Course, Address, City)
VALUES ('08081212', '18Df181', 'John', 'Stavanger', 'Cyb', 'ON', 'Finch');

INSERT INTO students (StudentID, RegNum, LastName, FirstName, Course, Address, City)
VALUES ('08081213', '18Df182', 'Josh', 'Stavanger', 'Cyb', 'ON', 'Finch');

alter table students add primary key(StudentID);


SELECT * FROM db_name.table_name;
select * from classicmodels.customers;
SELECT * FROM classicmodels.productlines;

SELECT * FROM classicmodels.employees
ORDER BY employeenumber;

select customername, contactlastname from classicmodels.customers;

use classicmodels;
select ordernumber, orderdate, status, customernumber from orders where orderdate between "2003-01-10" and "2003-12-01";
select * from employees where not jobtitle = 'sales rep';
select * from employees where jobtitle like '%vp%';
select max(amount) from payments;
select checknumber, paymentdate, max(amount) AS highestorder from payments group by 1;
select * from classicmodels.payments where amount = (select max(amount) from payments);
select count(customernumber) from orders where customernumber = 121;
select avg(amount) from payments where paymentdate between '2003-01-01' and '2003-12-31';
select avg(amount) from payments where paymentdate between '2004-01-01' and '2004-12-31';
select * from payments;
 select orders.*, customers.customername, customers.phone, customers.postalcode from orders
 right join customers on orders.customernumber = customers.customernumber

================================================================
PRACTICE AND PROVIDE SOLUTION:

Please write a query for the following output -
The total amount spent by each customer on all the orders placed
Required columns - Customer Name (Fname, Lname), postalcode, credit limit, amount spent

select contactfirstname, city, state, orders.ordernumber, max(orders.orderdate) as latest_order 
from customers
left join orders on customers.customernumber = orders.customernumber;
===============================================================
