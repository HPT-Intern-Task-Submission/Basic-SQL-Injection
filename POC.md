POC for SQL Injection
==================
PortSwigger Lab: `SQL injection vulnerability in WHERE clause allowing retrieval of hidden data`
------------------
### Lab Description
![Lab description](https://github.com/HPT-Intern-Task-Submission/Basic-SQL-Injection/blob/main/image/Lab_description.png)

This lab requires us to retrieve all hidden data. Looking at the description, the SQL query which process our search will be like this: `SELECT * FROM products WHERE category = 'Gifts' AND released = 1`. As we can see, the value of the category parameter is inserted directly into the query which is a simple case for us. However, let's first explore the page more.

### Here's the main page:

![Lab main page](https://github.com/HPT-Intern-Task-Submission/Basic-SQL-Injection/blob/main/image/Lab_main_page.png)

The page looks quite simple as it display some products for us. What caught my attention is the `Refine your search` area. Maybe this will allow us to filter the category we want to look for. So let's explore this functionality. I will click on `Corporate gifts` to see what will happen.

### What we recieve:
![Corporate gifts](https://github.com/HPT-Intern-Task-Submission/Basic-SQL-Injection/blob/main/image/Corporate%20gifts.png)

As I expected, the server will return all products related to the category.  What we should pay attention is the url: `https://0a15006e03c84fc0805471e000690040.web-security-academy.net/filter?category=Corporate+gifts`. Maybe the `category` parameter is vulnerable to SQL injection, so why don't we trying to inject something here. 

### Let's first start with a single quote `'` to see how server responses:
![Single quote inject](https://github.com/HPT-Intern-Task-Submission/Basic-SQL-Injection/blob/main/image/single_quote_inject.png)

Bump! It seems that we have successfully injected as the server returns error, I will explain the more in below section. Server returns with error is one of the indications to know if you can inject SQL command. 

### Now let's inject our whole payload to see what will happen next:
![Solved](https://github.com/HPT-Intern-Task-Submission/Basic-SQL-Injection/blob/main/image/Solved.png)

Ya-hoooo!!! We solved the lab. This is the complete query: `SELECT * FROM products WHERE category = 'Corporate gifts'or 2>1 --' AND released = 1`. Let's inspect the whole payload one more time to have a better understanding of what have just happened. The original query: `SELECT * FROM products WHERE category = 'Corporate gifts' AND released = 1`. Our payload `Corporate gifts'or 2>1 --`. First, the single quote `'` will break out of the `category` parameter which cause a SQL syntax error, this is the reason why the server returned error. Next we have an `or` operator followed by a **True** comparison statement `2>1` to make sure the `WHERE` clause will always return **True**. As usual, we often see people inject `1=1` which is often blocked by WAF,  we can instead inject other **True** comparisons. Finally, `--` will comment out all the remaining part of the query, which ensure that there will be no syntax error. Because the `WHERE` returns **True**, the server will return all the items in products table. 

>We've just solved a very basic SQL injection lab. In the real world, developers implement a lot of input validation to prevent this vulnerability. Therefore, we need to learn more and more advanced techniques to exploit this bug. Having solved this lab, I also gained some basic understanding of SQL injection which is a good foundation for me to level up my skills. Happy learning!!!
	



