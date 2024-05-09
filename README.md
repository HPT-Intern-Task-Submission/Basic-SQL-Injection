Basic SQL Injection
============
Definition
-------------
SQL Injection is a server-side vulnerability which attackers can inject malicious SQL command to exfiltrate, alter or even delete database. SQL command can be injected via different places in the application such as user's input field, search field. The bug often occurs because of the lack of input validation and sanitization.

Impact
---------------
Depending on which commands attackers are able to inject, SQL injection will have perspective impact on the server. Once attackers can have SQL injection, they can retrieve the data from others user, collect information about the server. In worse cases, they can also manipulate the data in database and finally destroy the whole system. SQL injection in the real world is still popular and every developer should be aware of.

Cause
------------
The occurrence of SQL injection often lies on the carelessness of developers in validating user's input. Let's have a look at the code:
```
<?php
// Get input
$username = $_POST['username'];
$password = $_POST['password'];

// Connect to database
$connection = mysql_connect("localhost", "username", "password", "database");

// Query to check user
$query = "SELECT * FROM users WHERE username='$username' AND password='$password'";
$result = mysql_query($connection, $query);

// Check if user exists
if(mysqli_num_rows($result) > 0) {
    echo "Login successful!";
} else {
    echo "Invalid username or password";
}

// Close connection
mysqli_close($connection);
?>
```
In this case, the `username` and `password` are put directly in to the SQL statement which poses a risk to SQL injection. The attacker just simply enter `username` like `admin ' or 1=1 --` and they will successfully log in as administrator. 

Let's analyze the payload. First, attacker inserts `admin` following by a single quote `'` to break out of the `username` parameter. After that, he add an `or` operator  and a **True** conditional statement to make sure the SQL statement will always return **True**. Finally, he add `--` to comment the rest of the SQL statement. To sum up, the complete SQL command will select all users who has `username` = `admin` or **True**, the `WHERE` clause is always True so the server will return all the users and log user in as the first `username` found in the result.


Mitigation
------------------
Having reviewed the vulnerable code, we now understand that validating user's input is a very important step. Developers shouldn't directly import user's input into the SQL query. One of the solution is to use **Parameterized Queries**. Here's the updated version of the vulnerable code above:
```
<?php
// Get input
$username = $_POST['username'];
$password = $_POST['password'];

// Connect to database
$connection = mysqli_connect("localhost", "username", "password", "database");

// Query to check user
$query = "SELECT * FROM users WHERE username=? AND password=?";
$stmt = mysqli_prepare($connection, $query);
mysqli_stmt_bind_param($stmt, "ss", $username, $password);
mysqli_stmt_execute($stmt);
$result = mysqli_stmt_get_result($stmt);

// Check if user exists
if(mysqli_num_rows($result) > 0) {
    echo "Login successful!";
} else {
    echo "Invalid username or password";
}

// Close connection
mysqli_close($connection);
?>
```
In this updated version, we see that the value of `username` and `password` is a question mark `?`. It is the placeholder for the `username` and `password` entered by user. To explain, when the SQL statement is executed, it will first execute the whole statement and later look for the value of `username` and `password`. In this case, the server will match the whole string `admin ' or 1=1 --` as `username` instead of `admin` as the vulnerable code. Therefore, the server cannot find any `username` with `admin ' or 1=1 --`  so the attacker cannot log in.
