# NoSQl injection 

---

### ***Prerequisite questions***
---
**1. What is nosql injection used against ?**
=> NOSQL injection is used against applications that interact with NOSQL databases.

**2. What is nosql databases? Name some.**
=> Nosql databases are non relational databases that process structed,semi-structured and unstructured data with flexible schema and alot of scalability and flexibility.

Popular databases : MongoDb , CASSANDRA , Redis.
Types of stores : key-value pairs , document stores, widecolumn stores, graph databases.

---

###Types of Nosql injection : 
1. Syntax based = break nosql query syntax.
{ "username": "admin' || '1'=='1", "password": "anything" }

2. Operator injection : You inject special operators ($ne, $gt, $regex) into the query.
{ "username": "admin", "password": { *"$ne"*: null } }

---


###TYPE 1 - NoSQL syntax injection 
notes :  test each input by submitting fuzz strings and special characters that trigger a database error or some other detectable behavior. If you know the API language of the target database, use special characters and fuzz strings that are relevant to that language. Otherwise, use a variety of fuzz strings to target multiple API languages.

Also ensure fuzz strings are encoded rightly. 
example : 

FUZZ string - 
```
'"`{
;$Foo}
$Foo \xYZ
```
For this attack we url encode :   
```
https://insecure-website.com/product/lookup?category='%22%60%7b%0d%0a%3b%24Foo%7d%0d%0a%24Foo%20%5cxYZ%00
```

