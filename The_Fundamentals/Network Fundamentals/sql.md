## Introduction to Databases

* A database is a set of related information
* Database – A collection of organized data. A set of related information.
* Database systems are computerized data storage and retrieval mechanisms
* Database system stores data electronically rather than on paper
* Database system is able to retrieve data more quickly, index data in multiple ways, and deliver up-to-the-minute information to its user community.

Links:

* [https://dev.mysql.com/downloads/installer/](https://dev.mysql.com/downloads/installer/)
* [https://dev.mysql.com/downloads/mysql/](https://dev.mysql.com/downloads/mysql/)
* [https://www.youtube.com/watch?v=u96rVINbAUI](https://www.youtube.com/watch?v=u96rVINbAUI)
* [https://www.w3schools.com/sql/sql\_create\_db.asp](https://www.w3schools.com/sql/sql_create_db.asp)

<img width="848" height="398" alt="image" src="https://github.com/user-attachments/assets/c36dec93-7328-4704-acd0-6ece196926e6" />

---

## The Relational Model

* Data is represented as sets of tables
* Redundant data is used to link records in different tables

### Relational databases

* Relational databases – It defines the relationship between 2 entities
* SQL is a programming language in database arena
* SQL is the language for generating, manipulating, and retrieving data from a relational database
* SQL is used for writing applications, performing administrative tasks, or generating reports

**Purpose of SQL**
Structured Query Language (SQL) is a textual language used by DB servers

* SQL commands are used to perform operations on DB: `INSERT`, `SELECT`, `UPDATE`, and `DELETE`
* Programmers use these commands to manipulate data in the DB

#### SQL query

* The result of an SQL query is a table
* This table is called “result set”
* A new permanent table can be created in a relational database simply by storing the result set of a query
* A query can use both permanent tables and the result sets from other queries as inputs
* **Activity:** Discuss the security concerns about this feature.

#### SQL statements

* **SQL schema statements:** Statements used to create database objects (tables, indexes, constraints, etc.).

  * Used to define the data structures stored in the database
  * Used by administrator
* **SQL data statements:** Statements used to create, manipulate, and retrieve the data stored in a database.

  * Used by administrator, programmer or report writer
* **SQL transaction statements:** Used to begin, end, and roll back transactions

---

## Launch the MySQL Tool, and Create Your Database and Database User

1. Launch a command window and login to MySQL as root:

   ```bash
   mysql -u root -p
   ```
2. Create a database for the sample data:

   ```sql
   CREATE DATABASE bank;
   ```
3. Create the `lrngsql` database user with full privileges on the `bank` database:

   ```sql
   CREATE USER 'lrngsql'@'localhost' IDENTIFIED BY 'toor';
   GRANT ALL PRIVILEGES ON bank.* TO 'lrngsql'@'localhost';
   ```
4. Exit the mysql tool:

   ```sql
   QUIT;
   ```
5. Log in to MySQL as `lrngsql`:

   ```bash
   mysql -u lrngsql -p
   ```
6. Attach to the `bank` database:

   ```sql
   USE bank;
   ```

You now have a MySQL server, a database, and a database user.

---

## Create Database Tables and Populate Them with Sample Data

12. Download the script at [http://examples.oreilly.com/learningsql/](http://examples.oreilly.com/learningsql/) and save it under `C:\temp\LearningSQLExample.sql`
13. If you have logged out of the mysql tool, log in to MySQL as `lrngsql` and attach to the `bank` database:

    ```bash
    mysql -u lrngsql -p
    USE bank;
    ```
14. Run it from the mysql utility:

    ```sql
    SOURCE C:\temp\LearningSQLExample.sql;
    ```

---

## Explaining What SQL Injection Means

An SQL injection is a type of code injection hacking technique in which an attacker uses a piece of structured query language code to alter a database and gain access to valuable discreet information. SQL injection codes have the capability to destroy a database if care is not taken.

<img width="940" height="449" alt="image" src="https://github.com/user-attachments/assets/3aa38024-1ace-4d5f-9818-d3c79b9ed602" />

---

## SQL Injection – Risk to DB

* SQL injection is a code injection technique that might destroy your database.
* SQL injection is one of the most common web hacking techniques.
* SQL injection is the placement of malicious code in SQL statements, via web page input.
* SQL injection can bypass a firewall and can affect a fully patched system
* Through SQL Injection attacker can obtain unauthorized access to a database and can create, read, update, alter, or delete data stored in the back-end database

### Scope of SQL Injection

* Identify injectable parameters.
* Identify the database type and version.
* Discover database schema.
* Extracting data.
* Insert, modify or delete data.
* Denial of service to authorized users by locking or deleting tables.
* Bypassing authentication.
* Privilege escalation.
* Execute remote commands by calling stored functions within the DBMS which are reserved for administrators.

### Ways to Inject SQL

* Injected through user input.
* Injection through cookie fields contains attack strings.
* Injection through server variables.
* Second-order Injection where hidden statements are executed at another time by another function.

<img width="940" height="646" alt="image" src="https://github.com/user-attachments/assets/217e9f87-85d0-4c12-a935-baaffa59d2aa" />

* This query returns all employee records with their salaries.
* The query is designed to retrieve a single employee record.
* Adding `OR 1=1` in the input field will change the SQL query.
* Since `1=1` is always true, the query returns all employees and their salaries.

---

## Defending Against SQL Injection

The root cause of almost every SQL injection is invalid input checking. Here is a list of prevention methods:

* Input Validation
* Input Checking Functions
* Validate Input Sources
* Access Rights

### Configure Database Error Reporting

---

## Classwork

1. Create a MySQL database and respective tables
2. Add constraints for attributes
3. Define Primary and/or Foreign Key(s) for each table

```sql
CREATE DATABASE Columbia_db;
USE Columbia_db;

CREATE TABLE doctors (
    StaffID CHAR(8) NOT NULL PRIMARY KEY,
    LastName VARCHAR(255),
    FirstName VARCHAR(255),
    Sex VARCHAR(255),
    Grade VARCHAR(255),
    Address VARCHAR(255)
);

CREATE TABLE staff (
    StaffID CHAR(8) NOT NULL PRIMARY KEY,
    LastName VARCHAR(255),
    FirstName VARCHAR(255),
    Sex VARCHAR(255),
    Grade VARCHAR(255),
    Address VARCHAR(255),
    FOREIGN KEY (StaffID) REFERENCES doctors(StaffID)
);

CREATE TABLE patients (
    RegNum CHAR(8) NOT NULL PRIMARY KEY,
    LastName VARCHAR(255),
    FirstName VARCHAR(255),
    Sex VARCHAR(255),
    Age INT,
    Address VARCHAR(255),
    results VARCHAR(255)
);
```

```sql
CREATE DATABASE insurance;
USE insurance;

CREATE TABLE clientinfo (
    SerialNo INT NOT NULL PRIMARY KEY,
    LastName VARCHAR(255),
    FirstName VARCHAR(255),
    DriverID CHAR(15) NOT NULL UNIQUE,
    Address VARCHAR(255) DEFAULT 'Ontario'
);

CREATE TABLE vehicle_info (
    SerialNo INT NOT NULL,
    Model VARCHAR(255),
    Year_of_purchase CHAR(4),
    DriverID CHAR(12) NOT NULL UNIQUE,
    Accident_Info VARCHAR(255),
    FOREIGN KEY (SerialNo) REFERENCES clientinfo(SerialNo)
);
```

* `foreign key = references the primary key of another table`
* `FOREIGN KEY (SerialNo column in vehicle_info) REFERENCES clientinfo(SerialNo column)`

4. Add 5–10 new records in each table using `INSERT INTO` statements.

```sql
INSERT INTO staff (StaffID, LastName, FirstName, Sex, Grade, Address) 
VALUES 
('1111111', 'romeno', 'uno', 'Male', 'Junior Exec', 'Queens'),
('1111112', 'domuli', 'reno', 'Male', 'Junior Exec', 'Queens'),
('1111113', 'tifani', 'jones', 'Male', 'Junior Exec', 'Finch'),
('1111114', 'jelopa', 'mireen', 'FeMale', 'Junior Exec', 'manitoba'),
('1111115', 'haniff', 'toff', 'FeMale', 'Junior Exec', 'kichener');
```

Paste the output of each table using the syntax:

```sql
SELECT * FROM doctors;
```

<img width="808" height="572" alt="image" src="https://github.com/user-attachments/assets/f0f434cf-0c2b-4a65-8d7c-ec2ab5c38236" />

```sql
SELECT * FROM patients;
```

<img width="791" height="611" alt="image" src="https://github.com/user-attachments/assets/df78381c-7289-431d-a9f3-d8260e20f19e" />

```sql
SELECT * FROM staff;
```

<img width="940" height="701" alt="image" src="https://github.com/user-attachments/assets/d0ef4cad-5c5e-47cf-a6f6-4a8d2423bcf5" />

---

## 1. Aggregation by Date in Toad for Oracle

A simple **aggregation** by date, useful for daily volume reports.

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

---

## MySQL Queries Functions

### Querying data

* **SELECT** – show you how to use simple `SELECT` statement to query the data from a single table.

  * The `SELECT` statement is used to select data from a database. The data returned is stored in a result table, called the result-set.
* Below is a screenshot showing an example of the output of running the select query on the “offices” tables.

<img width="808" height="531" alt="image" src="https://github.com/user-attachments/assets/d28021d5-bada-4b51-bdbe-a0cb683809ab" />

### Sorting data

* **ORDER BY** – show you how to sort the result set using `ORDER BY` clause. The custom sort order with the `FIELD` function will be also covered.
* The `ORDER BY` is used to sort the result-set in ascending or descending order.
* Below is a screenshot showing an example of the output of running the “order by” commands and its sorts the employee number in an ascending order.

<img width="826" height="564" alt="image" src="https://github.com/user-attachments/assets/dcb59928-7f5e-405b-8d5f-a368eb2626ad" />

### Filtering data

* **OR** – introduce you to the `OR` operator and show you how to combine the `OR` operator with the `AND` operator to filter data.
* **IN** – show you how to use the `IN` operator in the `WHERE` clause to determine if a value matches any value in a list or a subquery.
* **LIKE** – provide you with technique to query data based on a specific pattern.
* **LIMIT** – use `LIMIT` to constrain the number of rows returned by `SELECT` statement.
* **IS NULL** – test whether a value is `NULL` or not by using `IS NULL` operator.

The `WHERE` query is used to filter records. It is used to extract only those records that fulfill a specified condition.

Below is a screenshot showing an example of the output of running the “where” query and thus showing the records of `customerNumber` 112 filtered and listed from the `payments` table.

<img width="861" height="640" alt="image" src="https://github.com/user-attachments/assets/40695cf1-5bf4-43e0-bc90-ac28414e2b7a" />

### The SELECT DISTINCT query

Inside a table, a column often contains many duplicate values; and sometimes you only want to list the distinct values.

```sql
SELECT DISTINCT paymentdate FROM payments;
```

<img width="792" height="557" alt="image" src="https://github.com/user-attachments/assets/20f94399-9324-43e8-9c89-fbc7251cfdb8" />

* The `AND` operator displays a record if all the conditions separated by `AND` are `TRUE`.
* The `BETWEEN` operator selects values within a given range. The values can be numbers, text, or dates.

Below is a screenshot showing an example of how the `AND` and `BETWEEN` statements are used in the same statement. It is used to show values between two specific dates range from 31st June 2004 to 31st October 2004.

<img width="940" height="634" alt="image" src="https://github.com/user-attachments/assets/93aedf7a-7a2c-4958-99cd-9700c2f008f4" />

---

## Defining a Schema

A database schema defines how data is organized within a relational database. It represents the logical configuration of all or part of a relational database.

---

## Explaining MySQL JOIN Commands and Types of JOIN

An SQL JOIN is used to combine rows from two or more tables, based on a related column between them.

* **INNER JOIN** keyword selects records that have matching values in both tables.
* **LEFT JOIN** keyword returns all records from the left table on the left and the matching records from the table on the right.

Below is an example of the left join query.

<img width="806" height="505" alt="image" src="https://github.com/user-attachments/assets/15af09c1-bcbe-4d1e-9c9d-13d134b4c9fd" />

* **RIGHT JOIN** query displays all records from the right table and the matching records from the left table.

Below is an example screenshot of the right join query.

<img width="940" height="624" alt="image" src="https://github.com/user-attachments/assets/ddfdf570-e1dd-4a2d-a1a0-eca8f1b38ffb" />

---

## 2. Self-Join to List Managers and Direct Reports

Demonstrates a **self-join** to reveal hierarchical relationships within a single table.

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

---

## 3. Multi-Table JOIN with Aggregation and LEFT JOIN

Combines **multi-table joins**, a **LEFT JOIN**, and **aggregation** to produce per-customer spending summaries—even capturing customers with zero payments.

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

---

## The TIMESTAMP Data Type

The `TIMESTAMP` data type allows you to store date and time data including year, month, day, hour, minute and second.

---

## How to Import Data into MySQL

1. Select **Server** tab, then click on **Data Import** tab.

2. Click on **Import from Self-Contained File**, choose the path where the data file resides.

   <img width="940" height="634" alt="image" src="https://github.com/user-attachments/assets/ac42ab22-07f2-41b3-b0d4-0428576f8582" />

3. Start the import.

4. Refresh the **Schema** button and see new database information on the left tab of the workbench.

   <img width="940" height="512" alt="image" src="https://github.com/user-attachments/assets/4f3f9593-7191-4473-8ea1-a9cfb4ff359c" />

---

## Defining the ALIAS Command

Alias is used when you want to temporarily give a table or column another name. It is also often used to make columns or table names more readable, but they only exist for the duration of the query executed. The `AS` keyword is used in a query to create an alias.

**ALIASES:**

```sql
SELECT column_name AS alias_name
FROM table_name;

SELECT column_name(s)
FROM table_name AS alias_name;

SELECT CustomerID AS ID, CustomerName AS Customer
FROM Customers;
```

---

## Other CLI Scenarios

```sql
CREATE DATABASE TsomDB;
DROP DATABASE databasename;

BACKUP DATABASE testDB
TO DISK = 'D:\backups\testDB.bak'
WITH DIFFERENTIAL;

CREATE TABLE table_name (
    column1 datatype,
    column2 datatype,
    column3 datatype,
    ...
);

ALTER TABLE students (
    column1 datatype constraint,
    column2 datatype constraint,
    column3 datatype constraint,
    ...
);
```

```sql
USE tsomdb;
CREATE TABLE students (
    StudentID INT NOT NULL PRIMARY KEY,
    RegNum CHAR(8),
    LastName VARCHAR(255),
    FirstName VARCHAR(255),
    Course VARCHAR(255),
    Address VARCHAR(255),
    City VARCHAR(255),
    age INT,
    CONSTRAINT PK_Person PRIMARY KEY (StudentID, LastName)
);

-- foreign key = references the primary key of another table

SELECT * FROM students;
SELECT StudentID, LastName FROM students;

USE tsomdb;
SHOW TABLES;

SHOW DATABASES;

ALTER TABLE students
ADD Email VARCHAR(255);

ALTER TABLE students
DROP COLUMN Email;
ALTER TABLE students
DROP COLUMN DOB;
ALTER TABLE students
ADD DateOfBirth DATE;

ALTER TABLE students
RENAME COLUMN DateOfBirth TO DOB;
ALTER TABLE students
MODIFY COLUMN DOB YEAR;
ALTER TABLE students
MODIFY COLUMN Email VARCHAR(255) NOT NULL;
ALTER TABLE students
MODIFY COLUMN Address VARCHAR(255) SET DEFAULT Toronto;

INSERT INTO students (StudentID, RegNum, LastName, FirstName, Course, Address, City)
VALUES ('08081212', '18Df181', 'John', 'Stavanger', 'Cyb', 'ON', 'Finch'),
       ('08081213', '18Df182', 'Josh', 'Stavanger', 'Cyb', 'ON', 'Finch');

ALTER TABLE students ADD PRIMARY KEY(StudentID);

SELECT * FROM db_name.table_name;
SELECT * FROM classicmodels.customers;
SELECT * FROM classicmodels.productlines;

SELECT * FROM classicmodels.employees
ORDER BY employeenumber;

SELECT customername, contactlastname 
FROM classicmodels.customers;

USE classicmodels;
SELECT ordernumber, orderdate, status, customernumber
FROM orders
WHERE orderdate BETWEEN '2003-01-10' AND '2003-12-01';

SELECT * FROM employees WHERE NOT jobtitle = 'sales rep';
SELECT * FROM employees WHERE jobtitle LIKE '%vp%';
SELECT MAX(amount) FROM payments;
SELECT checknumber, paymentdate, MAX(amount) AS highestorder
FROM payments
GROUP BY 1;
SELECT * FROM classicmodels.payments 
WHERE amount = (SELECT MAX(amount) FROM payments);
SELECT COUNT(customernumber) FROM orders WHERE customernumber = 121;
SELECT AVG(amount) FROM payments WHERE paymentdate BETWEEN '2003-01-01' AND '2003-12-31';
SELECT AVG(amount) FROM payments WHERE paymentdate BETWEEN '2004-01-01' AND '2004-12-31';
SELECT * FROM payments;
SELECT orders.*, customers.customername, customers.phone, customers.postalcode
FROM orders
RIGHT JOIN customers ON orders.customernumber = customers.customernumber;
```

---

## Practice and Provide Solution

Please write a query for the following output –
The total amount spent by each customer on all the orders placed
>**Required columns** – Customer Name (Fname, Lname), postalcode, credit limit, amount spent

```sql
SELECT
  contactfirstname,
  city,
  state,
  orders.ordernumber,
  MAX(orders.orderdate) AS latest_order 
FROM customers
LEFT JOIN orders ON customers.customernumber = orders.customernumber;
```
