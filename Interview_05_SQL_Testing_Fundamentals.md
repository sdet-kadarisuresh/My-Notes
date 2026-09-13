# PART A: SQL / DATABASE TESTING

## Section 1: SQL Basics

**Q: What is SQL? What are the types of SQL commands? (DDL, DML, DQL, DCL, TCL)** 🔥
**A:** In interviews, when I'm asked about SQL, I always start by emphasizing that it's the core language we use as SDETs to talk to relational databases. In my project, we use it constantly to validate that the data saved from the UI actually landed correctly in the backend tables. SQL commands are fundamentally broken down into five categories based on what they do.
First, we have DDL (Data Definition Language). These are commands like CREATE, ALTER, and DROP. We typically use these when we are setting up our test environments and need to create fresh tables or modify schemas.
Next is DML (Data Manipulation Language), which includes INSERT, UPDATE, and DELETE. As an SDET, I use DML every day to set up test data before a test runs, or to clean it up afterward.
Then there's DQL (Data Query Language), which is basically just the SELECT statement. This is my bread and butter for verifying records.
We also have DCL (Data Control Language) like GRANT and REVOKE, which I honestly don't use much unless I'm testing role-based access to the database itself.
Lastly, TCL (Transaction Control Language) with COMMIT and ROLLBACK. In my automation framework, I often wrap my test data setup in a transaction and rollback at the end of the test so the database stays clean.

| Category | Full Form | Commands | Purpose in Testing |
|---|---|---|---|
| DDL | Data Definition Language | CREATE, ALTER, DROP, TRUNCATE | Setting up schemas for test DBs |
| DML | Data Manipulation Language | INSERT, UPDATE, DELETE | Test data creation and cleanup |
| DQL | Data Query Language | SELECT | Validating data stored by the application |
| DCL | Data Control Language | GRANT, REVOKE | Testing DB user permissions (rare) |
| TCL | Transaction Control Language | COMMIT, ROLLBACK, SAVEPOINT | Managing test data state safely |

**Q: What is the difference between DELETE, TRUNCATE, and DROP?** 🔥
**A:** This is a classic scenario we run into all the time when tearing down test environments. 
| Feature | DELETE | TRUNCATE | DROP |
|---|---|---|---|
| Command Type | DML | DDL | DDL |
| What it does | Removes specific rows | Removes all rows, keeps structure | Removes entire table (structure + data) |
| WHERE clause | Supported | Not supported | Not supported |
| Rollback | Yes, can be rolled back | No, cannot be rolled back | No, cannot be rolled back |
| Performance | Slower (logs each row) | Faster (deallocates pages) | Fastest |

**Verbal explanation:** The way I handle this in my project is quite specific. If I just need to remove a couple of test users I created during a single test case, I use `DELETE FROM Users WHERE user_id = 'test_123'`. It's a DML command, so it logs every row deleted, which makes it slower, but I can roll it back if I make a mistake. Now, if I am resetting the entire database for a nightly regression suite and I want the tables empty but I still need the tables themselves to exist, I use `TRUNCATE`. It's a DDL command, it's super fast because it doesn't log individual row deletions, but you can't roll it back. Finally, I use `DROP` when we are completely tearing down a transient Docker container or a temporary test schema and I want the table, the data, the indexes—everything—gone forever.

**Q: What is the difference between WHERE and HAVING?** 🔥
**A:** I use both of these regularly, but they serve different purposes when we are doing data aggregation.
| Feature | WHERE | HAVING |
|---|---|---|
| Purpose | Filters rows before grouping | Filters groups after grouping |
| Used with Aggregate Functions | No | Yes |
| Execution Order | Executed before GROUP BY | Executed after GROUP BY |
| Performance | Faster (reduces data early) | Slower (operates on grouped data) |

**Verbal explanation:** In my project, say we have an `Orders` table. If I want to find all orders placed in the last week, I'll use `WHERE order_date > '2023-01-01'`. The database filters the individual rows first. But what if I want to find out which customers have placed more than 5 orders in total? I can't use `WHERE COUNT(order_id) > 5` because `WHERE` looks at individual rows, not groups. So, I have to group the data by customer first using `GROUP BY`, and then I use `HAVING COUNT(order_id) > 5` to filter those aggregated groups. A best practice I follow is to always use `WHERE` to filter out as much data as possible before the `GROUP BY`, and only use `HAVING` for conditions that actually involve the aggregate function, for performance reasons.

**Q: What is the difference between UNION and UNION ALL?** 🔥
**A:** This is another common one when combining results from different tables.
| Feature | UNION | UNION ALL |
|---|---|---|
| Duplicates | Removes duplicate rows | Keeps duplicate rows |
| Performance | Slower (requires sorting to remove dupes) | Faster (just appends data) |
| Use Case | When you strictly need distinct records | When you know there are no dupes or want them all |

**Verbal explanation:** What we typically do when we have legacy tables and active tables—like `Active_Employees` and `Archived_Employees`—and I need a complete list, I'll combine them. If I use `UNION`, the database will actually sort the entire combined result set and remove any duplicate rows. That sorting step can be expensive on large tables. From my experience, if I know for a fact that an employee cannot exist in both tables simultaneously, I will always use `UNION ALL`. It simply glues the two result sets together without the overhead of deduplication, making the query much faster. If I actually want to see duplicates, `UNION ALL` is the only way to go.

**Q: What is the difference between GROUP BY and ORDER BY?**
**A:** In my day-to-day work, I use these for entirely different reasons when analyzing data.
| Feature | GROUP BY | ORDER BY |
|---|---|---|
| Purpose | Groups rows that have the same values into summary rows | Sorts the result set in ascending or descending order |
| Output | Reduces the number of rows returned | Keeps the same number of rows, just changes their sequence |
| Required with | Often used with aggregate functions (COUNT, SUM) | Can be used on any query |

**Verbal explanation:** If I want to know how many test automation runs failed per environment, I use `GROUP BY environment`. This takes hundreds of test run records, squashes them together based on the environment name, and gives me a summary—like "QA: 5 failures, Staging: 2 failures." It changes the shape of the data. On the other hand, if I just want to see a list of all test runs, but I want the most recent ones at the top, I use `ORDER BY execution_date DESC`. `ORDER BY` doesn't change the data or group anything; it simply changes the presentation order. I often use them together: I'll group by environment to get the failure count, and then order by the count descending so the worst environment shows up first.

**Q: What are JOINs? Explain INNER JOIN, LEFT JOIN, RIGHT JOIN, FULL JOIN, CROSS JOIN, SELF JOIN** 🔥
**A:** JOINs are absolutely essential. In a normalized database, data is spread across multiple tables, and in my project, I constantly have to pull data from 3 or 4 tables to validate a single UI screen. A JOIN is just the mechanism to combine rows from two or more tables based on a related column.

| Join Type | What it returns | Use Case Example |
|---|---|---|
| INNER JOIN | Only matching rows from both tables | Find users who have placed orders |
| LEFT JOIN | All rows from left table, matched rows from right | Find all users, and their orders if any |
| RIGHT JOIN | All rows from right table, matched rows from left | Find all orders, and the users who placed them |
| FULL JOIN | All rows when there is a match in either table | Find all users and all orders, matched or not |
| CROSS JOIN | Cartesian product (every row combined with every row) | Generating a massive matrix of test data combinations |
| SELF JOIN | Table joined with itself | Finding employees and their managers in the same table |

**Verbal explanation:** The way I handle this is I think about what the base data needs to be. For `INNER JOIN`, if I query `Users INNER JOIN Orders`, I only get users who have actually placed an order. Anyone who registered but bought nothing is left out.
With `LEFT JOIN`, if I say `Users LEFT JOIN Orders`, I get EVERY user in the database. If a user hasn't placed an order, the order columns will just show up as NULL. I use this a lot when I want to find orphaned records—like `WHERE Orders.id IS NULL` to find inactive users.
`RIGHT JOIN` is just the reverse. `FULL JOIN` gives me everything from both sides, which is great for reconciliation reports between two systems.
`CROSS JOIN` I actually use specifically for test data generation. If I have a table of 10 Browsers and 5 Operating Systems, a CROSS JOIN instantly gives me all 50 combinations for a compatibility testing matrix.
And `SELF JOIN` is when a table references itself, like an Employee table where the `Manager_ID` points back to the `Employee_ID` of another row in the exact same table.

**Q: Write a query to find the second highest salary from an Employee table (3 different approaches)** 🔥
**A:** This is probably the most asked query question, and I usually provide a few ways to show I know both standard SQL and window functions.
The scenario is we have an `Employee` table with a `Salary` column.

Approach 1: Using Subquery (Works in almost all DBs)
```sql
SELECT MAX(Salary) 
FROM Employee 
WHERE Salary < (SELECT MAX(Salary) FROM Employee);
```
This is the classic way. We find the absolute maximum salary, and then find the maximum salary that is strictly less than that.

Approach 2: Using LIMIT / OFFSET (MySQL / PostgreSQL)
```sql
SELECT DISTINCT Salary 
FROM Employee 
ORDER BY Salary DESC 
LIMIT 1 OFFSET 1;
```
This is much cleaner. We sort the unique salaries in descending order, skip the first one (`OFFSET 1`), and take the next one (`LIMIT 1`).

Approach 3: Using Window Functions (SQL Server, Oracle, PostgreSQL) - 🔥 Best Practice
```sql
WITH RankedSalaries AS (
    SELECT Salary, DENSE_RANK() OVER (ORDER BY Salary DESC) as rank
    FROM Employee
)
SELECT Salary FROM RankedSalaries WHERE rank = 2;
```
From my experience, the `DENSE_RANK()` approach is what I'd use in a real project. Why? Because if they suddenly ask for the 5th highest or 10th highest, the first two approaches become incredibly messy or slow, but with window functions, I just change `rank = 2` to `rank = 5`. It's robust and handles ties perfectly.

**Q: Write a query to find duplicate records in a table** 🔥
**A:** Finding duplicates is a massive part of database testing, especially when we are validating data migrations or ETL pipelines. In my project, we often have to ensure that a combination of columns, like `email`, is unique.

To find the duplicate emails in a `Users` table:
```sql
SELECT email, COUNT(*) as duplicate_count
FROM Users
GROUP BY email
HAVING COUNT(*) > 1;
```
What we typically do here is we group the table by the column we suspect has duplicates. `COUNT(*)` counts how many times each email appears. Then, the crucial part is the `HAVING` clause—we filter out any group that only appears once, leaving us with only the emails that exist 2 or more times. If I need to see the actual full rows of the duplicates, I'd wrap this in a subquery or join it back to the main table.

**Q: Write a query to find employees who earn more than the average salary**
**A:** This requires a subquery. You can't just say `WHERE Salary > AVG(Salary)` because aggregate functions aren't allowed in the WHERE clause directly.
```sql
SELECT EmployeeID, EmployeeName, Salary
FROM Employees
WHERE Salary > (
    SELECT AVG(Salary) FROM Employees
);
```
In my experience, this is a great example of an independent subquery. The database will first execute the inner query to calculate the overall average salary across the company. Let's say it's 60,000. Then it substitutes that value into the outer query, so it effectively becomes `WHERE Salary > 60000`.

**Q: What is the difference between Primary Key, Foreign Key, and Unique Key?** 🔥
**A:** These are the building blocks of relational data integrity. As an SDET, I rely on these constraints to ensure my automation doesn't accidentally insert garbage data.

| Feature | Primary Key (PK) | Unique Key (UK) | Foreign Key (FK) |
|---|---|---|---|
| Purpose | Uniquely identifies a row in a table | Ensures all values in a column are unique | Establishes a link between data in two tables |
| Null Values | Cannot accept NULL values | Can accept one NULL value (usually) | Can accept NULL values |
| Number per table | Only ONE per table | Multiple unique keys allowed | Multiple foreign keys allowed |

**Verbal explanation:** The way I explain this is with an `Employees` table. The `Employee_ID` is the **Primary Key**. It's the absolute identifier; it cannot be null, and you can only have one PK per table. 
The `Email_Address` would be a **Unique Key**. I don't want two employees having the same email, but it's not the primary identifier of the row. I can have many unique keys on a table (like Email, SSN, Passport Number), and depending on the DB, a unique key might allow a NULL value if the employee hasn't provided an email yet.
Finally, if my Employee table has a `Department_ID`, that is a **Foreign Key**. It points to the Primary Key of a separate `Departments` table. This enforces referential integrity—it means my automation framework cannot insert an employee with a Department ID of 999 if department 999 doesn't exist in the Departments table.

**Q: What are constraints in SQL?**
**A:** Constraints are basically the rules we put on table columns to enforce data integrity. In my project, we validate these constraints heavily during database testing to ensure the application throws the correct errors if bad data is submitted.
The main ones are:
1. **NOT NULL**: Ensures a column cannot have a NULL value. (e.g., a user must have a username).
2. **UNIQUE**: Ensures all values in a column are different.
3. **PRIMARY KEY**: A combination of NOT NULL and UNIQUE.
4. **FOREIGN KEY**: Prevents actions that would destroy links between tables.
5. **CHECK**: Ensures the values in a column satisfy a specific condition. For example, `CHECK (Age >= 18)`. If my test tries to insert an age of 17, the DB will reject it.
6. **DEFAULT**: Sets a default value if no value is specified during an INSERT. For example, `DEFAULT 'Active'` for a status column.

**Q: What is the difference between CHAR and VARCHAR?**
**A:** This is about storage optimization, and it actually impacts how we write assertions in our tests.
| Feature | CHAR | VARCHAR |
|---|---|---|
| Length | Fixed length | Variable length |
| Storage | Pads with spaces to meet the length | Only uses required space + length byte |
| Performance | Slightly faster for fixed sizes | Better storage optimization |
| Use Case | State codes (NY, CA), Gender (M, F) | Names, Emails, Descriptions |

**Verbal explanation:** If I define a column as `CHAR(10)` and I insert the word 'Test' (which is 4 characters), the database will physically pad it with 6 spaces to make it exactly 10 characters long. When I query it back in my Java framework, I might actually have to call `.trim()` on the string before asserting it, which is an edge case SDETs need to know. 
On the other hand, if I use `VARCHAR(10)` and insert 'Test', it only stores the 4 characters plus a tiny bit of metadata. We use VARCHAR for almost everything in modern applications (like names and emails) to save space, and strictly reserve CHAR for things that are exactly the same length every time, like a 2-letter country code.

**Q: What are aggregate functions? (COUNT, SUM, AVG, MIN, MAX)** 🔥
**A:** Aggregate functions perform a calculation on a set of values and return a single summary value. In my automation, I use these to validate dashboard metrics. If the UI says "Total Sales: $500", I'll use an aggregate function in SQL to verify that.

- `COUNT()`: Returns the number of rows. `SELECT COUNT(*) FROM Users` tells me the total registered users.
- `SUM()`: Adds up a numeric column. `SELECT SUM(Order_Total) FROM Orders` gives me the total revenue.
- `AVG()`: Calculates the average. `SELECT AVG(Salary) FROM Employees`.
- `MIN()` and `MAX()`: Return the smallest and largest values. `SELECT MAX(Login_Date) FROM User_Logs` tells me the most recent time anyone logged in.

A critical best practice I always mention: Aggregate functions (except `COUNT(*)`) automatically ignore NULL values. If you are averaging salaries and someone has a NULL salary, they aren't counted in the denominator.

**Q: What is a subquery? What is a correlated subquery?**
**A:** A subquery is simply a query nested inside another query. 
A **standard subquery** executes completely independently of the outer query, runs exactly once, and passes its result out. Like `SELECT * FROM Employees WHERE DepartmentID IN (SELECT ID FROM Departments WHERE Name = 'IT')`.

A **correlated subquery**, however, depends on the outer query for its values. It essentially executes repeatedly, once for every single row evaluated by the outer query.
For example, finding employees who earn more than the average salary *of their specific department*:
```sql
SELECT EmployeeName, Salary, DepartmentID
FROM Employees e1
WHERE Salary > (
    SELECT AVG(Salary) 
    FROM Employees e2 
    WHERE e1.DepartmentID = e2.DepartmentID
);
```
In my experience, correlated subqueries can be major performance bottlenecks because they run in a loop. Whenever developers use them, I always try to see if I can rewrite them using a `JOIN` or window functions to improve performance.

**Q: What is an index? Why do we use it? Types of indexes?** 🔥
**A:** From my database testing experience, when a UI page is loading very slowly, 9 times out of 10, it's a missing database index. An index is a database object that improves the speed of data retrieval operations, much like the index at the back of a book. Without an index, the database has to do a "Full Table Scan"—checking every single row top to bottom.

There are two main types I deal with:
1. **Clustered Index**: This dictates the actual physical order of the data on the disk. Because data can only be sorted physically in one way, you can only have ONE clustered index per table (usually the Primary Key).
2. **Non-Clustered Index**: This is a separate structure from the data rows, containing the indexed columns and pointers back to the actual data. You can have multiple non-clustered indexes on a table (e.g., an index on `Email` or `Last_Name`).

While indexes drastically speed up `SELECT` queries, the trade-off is that they slow down `INSERT`, `UPDATE`, and `DELETE` operations, because the index tree has to be rebuilt every time data changes.

**Q: What is normalization? Explain 1NF, 2NF, 3NF**
**A:** Normalization is the process of organizing database tables to reduce data redundancy and dependency. In simpler terms, it's ensuring we aren't storing the same piece of information in multiple places.
- **1NF (First Normal Form)**: Every column must contain atomic (indivisible) values. No comma-separated lists in a single column.
- **2NF**: It must be in 1NF, and all non-key columns must depend on the ENTIRE primary key (important for composite keys).
- **3NF**: It must be in 2NF, and there should be no transitive dependencies. Non-key columns shouldn't depend on other non-key columns.

In my project, when analyzing schemas, if I see a "Department_Name" typed out in every single row of the "Employee" table, I know it's not normalized. We should pull Department out into its own table and just store the `Department_ID` in the Employee table.

**Q: What is the difference between IN and EXISTS?**
**A:** I use both for filtering subqueries, but their underlying execution mechanisms are completely different.
`IN` works by executing the subquery first, generating a complete list of values in memory, and then comparing the outer query's rows to that list. It is generally better when the subquery returns a small, finite list.
`EXISTS` works as a boolean check. It evaluates true as soon as it finds the *first* matching row in the subquery and immediately stops searching. It doesn't return data, just true/false.

From a performance standpoint in my automation, if the subquery returns millions of rows, `IN` will crash or take forever because it has to load them all. `EXISTS` is incredibly fast because it short-circuits upon finding a single match. Therefore, `EXISTS` is heavily preferred for large datasets.

**Q: What is the difference between = and LIKE?**
**A:** `=` is used for exact string matching. `WHERE name = 'John'` will only return rows where the name is exactly 'John', no more, no less.
`LIKE` is used for pattern matching using wildcards. `WHERE name LIKE 'John%'` will return 'John', 'Johnny', 'Johnson', etc. 
In my test scripts, if I know the exact generated ID of a test user, I use `=`. If I generated a batch of users with names like "TestUser_1", "TestUser_2", I will use `LIKE 'TestUser_%'` to find or clean up the whole batch at once. Performance-wise, `=` is always faster because it can fully utilize indexes, whereas `LIKE` with a leading wildcard (`'%John'`) will cause a full table scan.

**Q: What are wildcards in SQL? (%, _)**
**A:** Wildcards are special characters used with the `LIKE` operator to search for data patterns.
- `%` (Percent sign) represents zero, one, or multiple characters. For example, `LIKE 'A%'` finds any string starting with 'A'.
- `_` (Underscore) represents exactly ONE single character. For example, `LIKE 'T_m'` will match 'Tim' and 'Tom', but not 'Team'.

**Q: What is NULL? How to handle NULLs in SQL?**
**A:** NULL is a state, not a value. It represents missing, unknown, or inapplicable data. It is NOT equivalent to zero or a blank space. Because it's an unknown, you cannot use standard operators like `=` or `!=` with it. `NULL = NULL` actually evaluates to unknown, not true!

To handle it in my queries:
1. I use `IS NULL` or `IS NOT NULL` to filter. (`WHERE phone_number IS NULL`).
2. I use functions like `COALESCE()` or `NVL()` (in Oracle) to provide a default value if a NULL is encountered. For example, `SELECT COALESCE(bonus, 0) FROM Salaries` will return 0 if the bonus column is NULL, preventing null pointer exceptions when that data gets passed back to my Java code.

---

## Section 2: SQL Coding Questions (MUST PREPARE)

**Q: Find the Nth highest salary** 🔥
**A:** As I mentioned earlier, the best way to handle this dynamically in a modern database is using the `DENSE_RANK()` window function. Let's say N is 4.

```sql
WITH RankedSalaries AS (
    SELECT 
        Employee_ID,
        Salary,
        DENSE_RANK() OVER (ORDER BY Salary DESC) as salary_rank
    FROM Employees
)
SELECT Employee_ID, Salary
FROM RankedSalaries
WHERE salary_rank = 4;
```
I specifically use `DENSE_RANK` instead of `RANK` because if two people share the top salary, `RANK` would call them both 1, and the next person 3. `DENSE_RANK` calls them both 1, and the next person 2, which is usually what the business logically means by "Nth highest".

**Q: Find employees who joined in the last 30 days** 🔥
**A:** In my automation frameworks, we constantly query for recent data. The syntax varies slightly by DB, but the logic is comparing the joining date to the current system date minus 30 days.

For SQL Server:
```sql
SELECT * FROM Employees 
WHERE Join_Date >= DATEADD(day, -30, GETDATE());
```

For PostgreSQL / MySQL:
```sql
SELECT * FROM Employees 
WHERE Join_Date >= CURRENT_DATE - INTERVAL '30 days';
```

**Q: Write a query to count employees per department**
**A:** This is a classic `GROUP BY` requirement. We need to join the Employee and Department tables to get the readable department name, not just the ID.

```sql
SELECT d.DepartmentName, COUNT(e.EmployeeID) as EmployeeCount
FROM Departments d
LEFT JOIN Employees e ON d.DepartmentID = e.DepartmentID
GROUP BY d.DepartmentName;
```
I purposefully use a `LEFT JOIN` here from the Departments table. If a department exists but has zero employees, an INNER JOIN would hide that department completely. A LEFT JOIN ensures the department still shows up in the report with an EmployeeCount of 0.

**Q: Write a query to find departments with more than 5 employees**
**A:** Building on the previous query, we just add a `HAVING` clause because we are filtering on an aggregated metric.

```sql
SELECT d.DepartmentName, COUNT(e.EmployeeID) as EmployeeCount
FROM Departments d
JOIN Employees e ON d.DepartmentID = e.DepartmentID
GROUP BY d.DepartmentName
HAVING COUNT(e.EmployeeID) > 5;
```

**Q: Write a query to find employees with duplicate emails** 🔥
**A:** 
```sql
SELECT Email, COUNT(*) as Count
FROM Employees
GROUP BY Email
HAVING COUNT(*) > 1;
```
If the interviewer asks me to delete those duplicates and keep only one (like the one with the lowest ID), I would do a self-join delete or use a CTE with `ROW_NUMBER()`:
```sql
WITH CTE AS (
    SELECT Email, 
           ROW_NUMBER() OVER (PARTITION BY Email ORDER BY EmployeeID) as row_num
    FROM Employees
)
DELETE FROM CTE WHERE row_num > 1;
```
This is a very advanced and clean way to handle test data cleanup.

**Q: Write a query to swap male and female values in a table**
**A:** This usually implies updating a single column `Gender` where 'M' becomes 'F' and 'F' becomes 'M'. We use a CASE statement within an UPDATE command.

```sql
UPDATE Employees
SET Gender = CASE 
    WHEN Gender = 'M' THEN 'F'
    WHEN Gender = 'F' THEN 'M'
    ELSE Gender
END;
```

**Q: Write a query to find the cumulative sum of salaries**
**A:** Cumulative sum (or running total) is done using window functions. We sum the salaries, ordered by some logical sequence like Employee ID or Date.

```sql
SELECT 
    EmployeeID, 
    Salary,
    SUM(Salary) OVER (ORDER BY EmployeeID) as Cumulative_Salary
FROM Employees;
```

**Q: Write a self-join query to find employees and their managers**
**A:** If we have an `Employees` table where `ManagerID` points to the `EmployeeID` of another row.

```sql
SELECT 
    e.EmployeeName as Employee, 
    m.EmployeeName as Manager
FROM Employees e
LEFT JOIN Employees m ON e.ManagerID = m.EmployeeID;
```
I use a `LEFT JOIN` here because the CEO or top-level executives will have a NULL ManagerID. If I used an INNER JOIN, the CEO would disappear from the report.

**Q: Write a query to delete duplicate rows keeping one**
**A:** (Covered in the CTE example in question 25, which is the most robust way to do this).

**Q: Write a query to find employees whose salary is above department average** 🔥
**A:** This requires a correlated subquery or a CTE.

```sql
SELECT e.EmployeeName, e.Salary, e.DepartmentID
FROM Employees e
WHERE e.Salary > (
    SELECT AVG(Salary) 
    FROM Employees d 
    WHERE d.DepartmentID = e.DepartmentID
);
```
Alternatively, using Window Functions:
```sql
WITH DeptAvgs AS (
    SELECT EmployeeName, Salary, DepartmentID,
           AVG(Salary) OVER (PARTITION BY DepartmentID) as DeptAvg
    FROM Employees
)
SELECT EmployeeName, Salary FROM DeptAvgs WHERE Salary > DeptAvg;
```
I always mention the window function approach in interviews because it scans the table only once, making it vastly more efficient than the correlated subquery.

---

## Section 3: Database Testing

**Q: What is database testing? How do you do it in your project?** 🔥
**A:** Database testing is the process of validating the data integrity, consistency, and structure within the backend database. In my current project, we don't just trust that if the UI says "Saved Successfully", it actually worked. What we typically do is a 3-step process. 
First, the automation script interacts with the UI or API to create a record—say, a new user profile. 
Second, we establish a JDBC connection to the database. 
Third, we run a `SELECT` query fetching that specific record and assert that the values in the database exactly match the test data we inputted. 
We also test constraints (trying to push nulls into non-null fields via API and checking for correct DB rejections), trigger executions, and stored procedure logic.

**Q: How do you validate data between UI and database?**
**A:** The way I handle UI to DB validation is by passing my Test Data Object through the whole flow. Let's say I fill out a Registration form. I have a Java POJO containing the First Name, Last Name, and Email. Selenium drives the browser to input this data and click submit. 
Then, I extract the generated User ID from the UI confirmation screen. I pass that ID into a DAO (Data Access Object) class in my framework, which executes a `SELECT * FROM Users WHERE id = ?`. The ResultSet is mapped back into another POJO. Finally, my TestNG assertion layer compares the original Test Data POJO with the Database POJO field by field using `Assert.assertEquals()`.

**Q: How do you connect to a database from Selenium/Java? (JDBC)**
**A:** Selenium itself cannot connect to a database; it's just a browser automation tool. We use Java's JDBC (Java Database Connectivity) API. 
The standard flow is:
1. Load the JDBC Driver (e.g., `Class.forName("com.mysql.cj.jdbc.Driver")`).
2. Establish a connection using `DriverManager.getConnection(url, username, password)`.
3. Create a `Statement` or `PreparedStatement`.
4. Execute the query using `.executeQuery()` for SELECTs or `.executeUpdate()` for INSERT/UPDATE/DELETE.
5. Iterate through the `ResultSet`.
6. Close the connection in a `finally` block to prevent memory leaks.

**Q: What is JDBC? Explain Connection, Statement, ResultSet**
**A:** 
```java
// Real code example from my framework
public String getUserEmail(String userId) throws SQLException {
    String email = null;
    String query = "SELECT email FROM users WHERE id = ?";
    
    // 1. Connection
    try (Connection conn = DriverManager.getConnection(DB_URL, USER, PASS);
         // 2. PreparedStatement
         PreparedStatement pstmt = conn.prepareStatement(query)) {
        
        pstmt.setString(1, userId);
        
        // 3. ResultSet
        try (ResultSet rs = pstmt.executeQuery()) {
            if (rs.next()) {
                email = rs.getString("email");
            }
        }
    } // Auto-closed due to try-with-resources
    return email;
}
```
In this snippet:
- **Connection**: Manages the physical TCP connection to the DB server.
- **PreparedStatement**: Compiles the SQL query. I always use PreparedStatement over regular Statement because it prevents SQL injection and is pre-compiled for better performance.
- **ResultSet**: The object that holds the tabular data returned by the database. We use a cursor (`rs.next()`) to read through it row by row.

**Q: How do you use database queries in your automation framework for test data setup/cleanup?**
**A:** Relying on the UI to create test data before a test is incredibly slow and flaky. In my framework, I heavily rely on DB queries for `@BeforeMethod` setup and `@AfterMethod` cleanup. 
For example, if I have a test that validates a user can check out a shopping cart, I don't want to script the UI steps to create a user, search for an item, add to cart, etc. Instead, in my setup method, I execute a JDBC batch insert that injects a mocked user, an item, and an active cart session directly into the tables. The UI test then simply logs in as that user and goes straight to the checkout page. After the test, my cleanup method executes a `DELETE` query based on that specific test user's ID to keep the environment pristine.

---

# PART B: TESTING FUNDAMENTALS

## Section 4: SDLC & STLC

**Q: What is SDLC? Explain its phases** 🔥
**A:** SDLC stands for Software Development Life Cycle. It's the overarching process that the entire engineering organization follows to design, develop, and deliver high-quality software. As an SDET, I am involved in almost every phase, not just the testing phase.

| Phase | What happens | SDET Involvement |
|---|---|---|
| Requirement Analysis | Gathering business needs | Reviewing for testability, pointing out edge cases early |
| Design | Architecture, DB schema, UI mockups | Identifying automation frameworks needed, test data planning |
| Development (Coding) | Devs write the code | Writing automation scripts against APIs/mock UIs, code reviews |
| Testing | Finding defects, validating requirements | Executing automation suites, manual exploratory testing, logging bugs |
| Deployment | Moving code to production | Running post-deployment sanity/smoke suites |
| Maintenance | Bug fixes, minor updates | Updating regression suites, monitoring production logs |

**Verbal explanation:** In my project, we follow an Agile SDLC. During requirement analysis (our backlog refinement sessions), I actively participate by asking "How are we going to test this?" before a single line of code is written. By the time development starts, I am already writing my TestNG tests based on the Swagger API docs. We don't wait for the "Testing" phase to start testing; testing is a continuous thread throughout the entire SDLC.

**Q: What is STLC? Explain its phases** 🔥
**A:** STLC is the Software Testing Life Cycle. If SDLC is the whole software factory, STLC is specifically the QA department's process within that factory.

| Phase | What happens | Deliverable |
|---|---|---|
| Requirement Analysis | Analyzing what to test based on PRD/Stories | RTM, Clarification queries |
| Test Planning | Defining strategy, tools, scope, and resources | Test Plan document |
| Test Case Development | Writing manual steps and automation scripts | Test Cases, Automation Code |
| Environment Setup | Configuring servers, DBs, test data | Ready test environment |
| Test Execution | Running tests, logging bugs | Defect Reports, Test Logs |
| Test Closure | Evaluating coverage, generating final reports | Test Closure Report |

**Verbal explanation:** What we typically do is, once a sprint starts (Requirement Analysis), I create sub-tasks for automation. During Test Case Development, I map out scenarios in JIRA and simultaneously script them in Java/Selenium. Before Dev finishes, I ensure the Test Environment is seeded with the right SQL scripts. Once the feature branch is merged (Execution), my Jenkins pipeline runs the scripts, and any failures are raised as bugs. Finally, at Test Closure, I provide a metrics report to the Scrum master showing our pass/fail rate.

**Q: What is the difference between SDLC and STLC?** 🔥
**A:** 
| Feature | SDLC (Software Development Life Cycle) | STLC (Software Testing Life Cycle) |
|---|---|---|
| Definition | End-to-end process of developing the software | Specific process of validating the software |
| Scope | Encompasses the entire project (Dev, QA, Ops, Product) | Limited to the testing team / QA activities |
| Focus | Building a high-quality product | Ensuring the product built meets the requirements |
| Phases | Requirements, Design, Code, Test, Deploy | Requirements Analysis, Test Plan, Execution, Closure |

**Verbal explanation:** SDLC is the parent process, and STLC is a subset of it. The STLC essentially sits inside the "Testing" phase of the SDLC, but in modern Agile teams, STLC activities overlap heavily with SDLC phases. For example, my STLC Requirement Analysis happens at the exact same time as the SDLC Requirement Analysis. 

**Q: What is the difference between Waterfall and Agile?**
**A:** 
| Feature | Waterfall | Agile |
|---|---|---|
| Approach | Sequential, linear | Iterative, incremental |
| Flexibility | Rigid; hard to change requirements later | Highly flexible; embraces changing requirements |
| Delivery | One single massive release at the end | Continuous delivery of small, working features (Sprints) |
| Testing phase | Done only after all development is complete | Continuous, happens concurrently with development |

**Verbal explanation:** I've worked in both, and the difference is night and day. In Waterfall, we used to get a 100-page spec document, Dev would code for 3 months, and then QA would get 2 weeks to test it. It was a nightmare because bugs found at the end were structurally hard to fix. In my current Agile project, we work in 2-week sprints. We take a tiny feature, code it, write automation for it, and deploy it. If the client changes their mind next week, it's no big deal—we just pivot for the next sprint.

**Q: What is the difference between Verification and Validation?** 🔥
**A:** This is a fundamental concept.
| Feature | Verification | Validation |
|---|---|---|
| Question | "Are we building the product right?" | "Are we building the right product?" |
| Type | Static testing (without executing code) | Dynamic testing (executing code) |
| Activities | Reviews, walkthroughs, inspections of documents/code | Unit, Integration, System, and UAT testing |
| Who does it | Devs, QA, Product (Peer reviews) | QA and End Users |

**Verbal explanation:** Verification is checking the documents and processes. When I sit in a meeting and review a JIRA user story to ensure it has proper acceptance criteria, or when I do a code review of another SDET's pull request, that is Verification. I am not running the application. Validation is the actual testing. When my Selenium script clicks a button and verifies a database record is updated, that is Validation. I am executing the software to prove it meets the customer's exact needs.

---

## Section 5: Types of Testing

**Q: What is the difference between Smoke Testing, Sanity Testing, and Regression Testing?** 🔥
**A:** I handle all three of these using different automation suites in my Jenkins CI/CD pipeline.

| Feature | Smoke Testing | Sanity Testing | Regression Testing |
|---|---|---|---|
| Purpose | Verify critical functionality of a new build | Verify a specific bug fix or new feature works | Verify old functionality didn't break due to new code |
| Scope | Very shallow, broad coverage | Deep, narrow coverage | Deep, broad coverage |
| When it's done | Immediately after a build is deployed | After a minor code change/fix | Before a major release |
| Automation | 100% automated | Usually manual or lightly automated | 100% automated |

**Verbal explanation:** When developers push code and Jenkins deploys a new build to the QA environment, my **Smoke Test** suite (maybe 10 critical test cases like Login and Add to Cart) runs immediately. If it fails, the build is rejected instantly. 
If it passes, and the release contains a fix for the Payment Gateway, I will do **Sanity Testing** specifically on the Payment Gateway to ensure the fix actually worked. 
Finally, before we push the release to Production, I run the full **Regression Suite** (maybe 500 automated tests) overnight to ensure that fixing the Payment Gateway didn't accidentally break the User Profile page.

**Q: What is the difference between Functional and Non-Functional Testing?** 🔥
**A:**
| Feature | Functional Testing | Non-Functional Testing |
|---|---|---|
| Focus | "What" the system does | "How well" the system does it |
| Examples | Login, Checkout, API responses, DB updates | Performance, Security, Usability, Reliability |
| Tools | Selenium, RestAssured, Appium | JMeter, LoadRunner, OWASP ZAP |
| Execution | Usually done first | Usually done after functional is stable |

**Verbal explanation:** In my day-to-day, I focus heavily on functional testing. If a user clicks 'Submit', does the order process? I write Selenium scripts to prove that. But functional testing doesn't care if the 'Submit' button takes 30 seconds to process. That's where Non-Functional testing comes in. A colleague might run a JMeter script simulating 10,000 users clicking 'Submit' simultaneously to test the system's performance and scalability. Both are critical for a production-ready application.

**Q: What is the difference between System Testing and Integration Testing?**
**A:** 
| Feature | Integration Testing | System Testing |
|---|---|---|
| Scope | Testing interaction between 2 or more modules | Testing the entire integrated application as a whole |
| Focus | Data flow and interfaces between modules | End-to-end business requirements |
| Who does it | Devs (Component level) or QA (System level) | QA / SDETs |

**Verbal explanation:** In microservices architecture, Integration Testing is vital. If Service A (User Service) needs to call Service B (Payment Service), I write API integration tests to ensure Service A sends the JSON payload correctly and Service B parses it. However, System Testing is when I fire up the frontend web UI, log in, navigate the site, make a purchase, and check the database. I am testing the entire system exactly as a real user would.

**Q: What is the difference between Black Box, White Box, and Grey Box Testing?** 🔥
**A:**
| Feature | Black Box Testing | White Box Testing | Grey Box Testing |
|---|---|---|---|
| Knowledge of internal code | None | Full source code access | Partial knowledge (APIs, DB schemas) |
| Focus | Inputs and Outputs | Internal logic, branches, statements | End-to-end functionality + backend validation |
| Who performs it | Manual QA / End Users | Developers | SDETs / Automation Engineers |
| Example | Testing UI buttons | Writing JUnit / TestNG unit tests | Hitting an API and checking DB state |

**Verbal explanation:** As an SDET, I primarily live in the Grey Box world. I don't need to read the specific Java loops the frontend developer wrote (White box), but I am not just blindly clicking the UI either (Black box). I have access to the architecture, the database, and the Swagger API docs. So my tests might click a UI element, intercept the network API call to assert the payload, and then run a JDBC query to verify the backend state.

**Q: What is UAT (User Acceptance Testing)? Who performs it?**
**A:** UAT is the final phase of testing before production deployment. It is performed by the actual end-users, clients, or business stakeholders (like the Product Owner). 
In my project, QA does not perform UAT. Our job is to prove the software meets the technical specifications. The business users' job during UAT is to prove the software works for their day-to-day business operations in the real world. We often support UAT by setting up the data for the business users, but they execute the scenarios and give the final sign-off.

**Q: What is Alpha testing vs Beta testing?**
**A:** 
| Feature | Alpha Testing | Beta Testing |
|---|---|---|
| Who performs it | Internal employees / internal QA teams | Real external end-users |
| Environment | Developer/QA controlled environments | Real-world, uncontrolled environments |
| Goal | Find bugs before releasing to clients | Gauge customer feedback, find edge-case environmental bugs |

**Verbal explanation:** We rarely do Beta testing for internal enterprise apps, but for B2C apps, Alpha testing is what we do in-house before the launch. Beta testing is when we release the app to 1% of real users in the App Store to see how it performs on thousands of weird devices and network conditions we couldn't simulate in the lab.

**Q: What is Exploratory Testing? When do you use it?**
**A:** Exploratory testing is simultaneous learning, test design, and test execution. There are no pre-written test scripts. 
In my workflow, after I finish automating all the defined acceptance criteria for a new feature, I spend an hour doing exploratory testing. I try to "break" the system by doing unexpected things—using the back button during a transaction, submitting weird data types, interrupting network connections. It relies entirely on the tester's intuition and domain knowledge. Our automation catches regression bugs, but exploratory testing is where we find the really nasty, complex new defects.

**Q: What is Ad-hoc Testing?**
**A:** Ad-hoc testing is completely informal testing with no documentation, no test design, and no specific goal other than finding bugs. It's often called "Monkey testing." While exploratory testing is structured around exploring specific areas of an app using domain knowledge, ad-hoc is completely random. I might just ask a developer to randomly click around the UI for 5 minutes before I start my formal suite, just to see if the server crashes.

**Q: What is Re-testing vs Regression Testing?** 🔥
**A:** This is a very common interview question to trap candidates.
| Feature | Re-testing | Regression Testing |
|---|---|---|
| Purpose | To verify a specific defect has been fixed | To ensure a fix didn't break anything else |
| Scope | Very narrow (just the failed test case) | Broad (entire application or module) |
| Automation | Often done manually | Almost always automated |
| Status | Done on "Fixed" bugs | Done on "Passed" test cases |

**Verbal explanation:** If I raise a bug saying "Login fails with special characters", the developer fixes it and assigns it back to me. I will execute that exact same scenario again. That is **Re-testing**. Once I confirm it passes, I then trigger my entire Jenkins test suite to ensure that his fix to the login module didn't accidentally break the Password Reset page. That is **Regression Testing**.

**Q: What is Boundary Value Analysis and Equivalence Partitioning? Give examples**
**A:** These are Black Box test design techniques to ensure maximum coverage with minimum test cases.
- **Equivalence Partitioning (EP)** divides input data into valid and invalid partitions. If a text box accepts ages 18-60, the valid partition is 18-60. Invalid partitions are <18 and >60. I only need to test one value from each (e.g., 10, 35, 70).
- **Boundary Value Analysis (BVA)** focuses on the edges of those partitions, because bugs frequently cluster at boundaries (due to >= vs > errors in code). For the age 18-60 field, I would test the boundaries: 17, 18, 19, 59, 60, 61.
In my automation, whenever I write a data-driven test for a numeric field, I exclusively use BVA values as my parameters.

---

## Section 6: Test Artifacts

**Q: What is a Test Plan? What does it contain?** 🔥
**A:** A Test Plan is the master document that outlines the overall strategy, scope, resources, and schedule for testing a project. As an SDET lead, I help formulate this at the beginning of a major release.
It typically contains:
1. **Scope**: In-scope and Out-of-scope items (e.g., "We will test Web, but Mobile is out of scope for Phase 1").
2. **Strategy**: Manual vs. Automation ratios, types of testing (Functional, Load).
3. **Hardware/Software Requirements**: Test environment specs.
4. **Roles & Responsibilities**: Who is doing what.
5. **Schedule**: Deadlines aligned with sprints.
6. **Entry/Exit Criteria**: When we start testing, and when we are confident to sign off.

**Q: What is a Test Strategy? How is it different from Test Plan?**
**A:** 
| Feature | Test Strategy | Test Plan |
|---|---|---|
| Level | High-level organizational document | Project-level document |
| Flexibility | Static (rarely changes) | Dynamic (changes with project needs) |
| Creator | QA Manager / Test Architect | Test Lead / QA Lead |

**Verbal explanation:** The Test Strategy is established at the company level. It defines things like "Our organization will use Selenium for UI and RestAssured for API testing, and all code must have 80% coverage." It applies to all projects. A Test Plan is created for a specific project or release, detailing exactly *how* that specific project will execute the strategy. 

**Q: What is a Test Case? How do you write a good test case?** 🔥
**A:** A test case is a specific set of conditions, steps, and expected results used to verify a particular feature. In JIRA/Zephyr, I format my test cases to be easily readable by any new team member or convertible to automation code.
A good test case contains:
- **Test Case ID**: Unique identifier.
- **Title**: Clear summary (e.g., "Verify successful login with valid credentials").
- **Pre-conditions**: E.g., "User account must exist in database".
- **Test Data**: E.g., username: admin, password: password123.
- **Steps**: Explicit actions (1. Navigate to URL, 2. Enter username...).
- **Expected Result**: What the system *should* do.
- **Actual Result**: Left blank until execution.

**Q: What is the difference between Test Case and Test Scenario?** 🔥
**A:**
| Feature | Test Scenario | Test Case |
|---|---|---|
| Definition | A high-level functionality to be tested | Detailed steps to test that functionality |
| Focus | "What" to test | "How" to test |
| Detail | Very brief (one liner) | Highly detailed with steps and data |

**Verbal explanation:** A Test Scenario is broad: "Validate the Login Functionality." From that single scenario, I can write dozens of detailed Test Cases: "Test Case 1: Login with valid credentials," "Test Case 2: Login with invalid password," "Test Case 3: Login with blank username." We usually define scenarios during sprint planning to estimate effort, and then write the detailed test cases during the sprint.

**Q: What is Requirements Traceability Matrix (RTM)?**
**A:** An RTM is a document (or a JIRA dashboard) that maps business requirements to test cases. The entire goal is to ensure 100% test coverage—that no requirement is left untested. 
In my project, every test case in Zephyr must be linked to a JIRA User Story. If the product owner asks, "Is the new payment gateway fully tested?", I can pull the RTM, show them the specific requirement ID, and immediately show them the 15 test cases linked to it, along with their pass/fail status.

---

## Section 7: Defect Management

**Q: What is a Bug/Defect? What is the Defect Life Cycle?** 🔥
**A:** A defect is a deviation between the expected result and the actual result in the software. When I find a bug, it goes through a specific lifecycle in JIRA:
1. **New**: I log the bug.
2. **Assigned**: The lead reviews it and assigns it to a developer.
3. **Open/In Progress**: The developer is fixing the code.
4. **Fixed/Resolved**: Dev pushes the fix to the QA environment.
5. **Ready for Retest**: It's back in my queue.
6. **Closed**: I retest it, it passes, I close it.
7. **Reopened**: I retest it, it still fails, I kick it back to Dev.
8. **Deferred / Rejected**: Sometimes Product decides the bug is too minor to fix right now (Deferred), or Dev argues it's actually an intended feature (Rejected).

**Q: What is the difference between Severity and Priority?** 🔥
**A:** This is a guaranteed interview question.
| Feature | Severity | Priority |
|---|---|---|
| Definition | The technical impact of the bug on the system | The business urgency to fix the bug |
| Driven by | QA Engineer based on technical failure | Product Owner based on business impact |
| Types | Blocker, Critical, Major, Minor | P1 (Immediate), P2, P3, P4 |

**Verbal explanation:** Severity is about how badly the software is broken. Priority is about how fast we need to fix it. As an SDET, I assign Severity based on whether the app crashes or just looks slightly misaligned. The Product Owner assigns Priority based on how many customers are complaining about it.

**Q: Give examples of: High Sev/High Pri, High Sev/Low Pri, Low Sev/High Pri, Low Sev/Low Pri**
**A:**
- **High Severity, High Priority**: The "Checkout" button crashes the app entirely. (Core functionality broken, blocks revenue immediately).
- **High Severity, Low Priority**: The app crashes if a user inputs a 500-character string into a legacy page that only 2 users visit per year. (Technically a severe crash, but business impact is zero, so it's low priority to fix).
- **Low Severity, High Priority**: The company logo on the main homepage is spelled wrong. (Technically the app functions perfectly, severity is low. But it's an embarrassing branding issue, so the CEO wants it fixed immediately).
- **Low Severity, Low Priority**: A typo in the Terms & Conditions text footer.

**Q: What is a defect report? What fields does it contain?**
**A:** A defect report is what I log in JIRA. A poorly written bug report wastes everyone's time. My bug reports always include:
- **Summary**: Concise title (e.g., "500 Error on submitting blank payment form").
- **Environment**: OS, Browser, App Version, DB branch.
- **Steps to Reproduce**: 1, 2, 3 exact steps.
- **Expected Result vs Actual Result**.
- **Evidence**: Attached screenshots, screen recordings, network payload JSONs, and server logs.
- **Severity/Priority**.

**Q: What tools do you use for defect tracking?**
**A:** In my current project, we exclusively use JIRA. We have a workflow configured where bugs are automatically linked to the user stories they originated from. If my CI/CD pipeline fails, I have Jenkins integrated with JIRA via APIs so that a failure automatically generates a Defect ticket, populates the stack trace into the description, and tags the developer who made the commit.

---

## Section 8: Agile/Scrum

**Q: What is Agile? What are Agile principles?** 🔥
**A:** Agile is an iterative approach to software development and project management. Instead of delivering everything at the end like Waterfall, Agile delivers work in small, consumable increments. 
In my experience, the core principles we follow are: valuing individuals and interactions over rigid processes, delivering working software frequently (every 2 weeks), collaborating heavily with the customer, and being extremely flexible to change requirements late in the game. As an SDET, Agile means I am automating features exactly as they are being built, not months later.

**Q: What is Scrum? Explain the Scrum process** 🔥
**A:** Scrum is the most popular framework for implementing Agile. 
The process works like this: The Product Owner maintains a list of everything the product needs (Product Backlog). The team takes a small chunk of that list and commits to building it during a 2-week period called a Sprint (Sprint Backlog). During the Sprint, the team meets daily for 15 minutes to sync up. At the end of the Sprint, we demo the working software to stakeholders, and then hold a retrospective to figure out how to improve our process for the next Sprint.

**Q: What are Scrum ceremonies? Explain each** 🔥
**A:** I participate in four main ceremonies in my project:
1. **Sprint Planning**: At the start of the sprint, we review user stories, point them (estimate effort), and decide what we can commit to delivering. I ensure automation effort is included in the estimates.
2. **Daily Standup**: A 15-minute daily sync. I answer three questions: What did I do yesterday? What am I doing today? Are there any blockers?
3. **Sprint Review / Demo**: At the end of the sprint, we demo the working features. I often drive the demo using my automated test scripts to show the stakeholders it works.
4. **Sprint Retrospective**: We discuss what went well, what went wrong, and action items. If a bug slipped into production, this is where we discuss how QA can prevent it next time.

**Q: What are Scrum roles?** 🔥
**A:** 
1. **Product Owner (PO)**: Represents the business. They write the user stories, prioritize the backlog, and decide what features are built next.
2. **Scrum Master**: The facilitator. They don't manage the team, but they ensure Scrum practices are followed and remove any blockers (e.g., if our test environment goes down, the Scrum Master chases the DevOps team to fix it).
3. **Development Team**: Cross-functional team of developers, QA/SDETs, and designers who actually build and test the software. We are self-organizing.

**Q: What is a Sprint? How long is it? What happens during a sprint?** 🔥
**A:** A Sprint is a time-boxed iteration where a specific amount of work is completed and made ready for review. In my project, our Sprints are exactly 2 weeks long. 
During the sprint, Developers write code, and I simultaneously write the automation framework additions for that code. Once Devs merge a feature, I execute my tests, log bugs, and they fix them. The goal is that by the last day of the sprint, the feature is 100% coded, 100% tested, and potentially deployable to production. No work carries over if we can help it.

**Q: What is a User Story? What is the format? What are Acceptance Criteria?** 🔥
**A:** A User Story is a requirement written from the perspective of the end-user. 
The standard format we use is: "As a [Role], I want [Action], so that [Benefit]." 
For example: "As a customer, I want to reset my password, so that I can regain access to my account."
**Acceptance Criteria (AC)** are the technical conditions that must be met for the story to be considered "Done." As an SDET, ACs are critical for me—I literally translate the ACs directly into my TestNG `@Test` methods. If an AC says "Password must be 8 characters," I write a test validating exactly that.

**Q: What is Story Point estimation? What is Planning Poker?**
**A:** We don't estimate work in "hours" or "days"; we use Story Points based on the Fibonacci sequence (1, 2, 3, 5, 8, 13). Points measure complexity, risk, and effort.
We use Planning Poker: The PO reads a story, and everyone (Devs and QA) secretly selects a card with a point value. We reveal them simultaneously. If a Dev votes '2' but I vote '8', we discuss why. Often, I point out that while the coding is easy, writing the complex test data setup and automation framework integrations will be very difficult, so the team agrees to increase the points.

**Q: What is your role as an SDET in Agile/Scrum? What do you do in each ceremony?** 🔥
**A:** My role is to ensure quality is baked in from day one, not bolted on at the end. 
- In **Backlog Refinement/Planning**, I advocate for testability. I ask Devs to add specific IDs to UI elements so my Selenium scripts don't break.
- During the **Sprint**, I write automation code in parallel with developers writing app code. I run tests continuously in CI/CD pipelines.
- In **Standups**, I communicate defect trends and testing blockers.
- In **Retrospectives**, I provide metrics—how many tests automated, what defects slipped, and how we can improve our CI/CD pipeline speed.

**Q: What is Definition of Done (DoD)? What is Definition of Ready (DoR)?**
**A:** 
- **Definition of Ready (DoR)**: The criteria a user story must meet *before* we can pull it into a sprint. For me, DoR means the story has clear acceptance criteria, wireframes are attached, and API contracts are defined. If it doesn't have these, I block it from entering the sprint.
- **Definition of Done (DoD)**: The checklist a story must pass *before* we can call it complete at the end of the sprint. In my project, our DoD explicitly states: Code reviewed, Unit tests >80%, E2E automation scripts merged and passing in Jenkins, and zero Sev-1/Sev-2 open defects.

**Q: What is a Sprint Backlog vs Product Backlog?**
**A:**
| Feature | Product Backlog | Sprint Backlog |
|---|---|---|
| Content | Every single feature, bug, and idea for the entire product's future | Only the specific items committed to for the current 2-week Sprint |
| Ownership | Product Owner | The Development/QA Team |
| Lifespan | Exists as long as the product exists (constantly changing) | Lives only for the duration of the Sprint |

**Verbal explanation:** The Product Backlog is the master wish-list. It might have 500 items. The Sprint Backlog is the realistic subset—maybe 15 items—that our team agreed to tackle right now. As an SDET, I mostly focus on the Sprint Backlog for my daily execution, but I keep an eye on the top of the Product Backlog to plan future automation framework changes.
