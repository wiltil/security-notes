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

Then the 
