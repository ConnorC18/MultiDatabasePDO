![MySQL & PHP](https://codegeekz.com/wp-content/uploads/php-mysql-logo-large.gif)

# MultiDatabasePDO
This is a **free**, **easy to use**, **lightweight** and **powerful** PHP library that allows you to connect to multiple MySQL databases through PDO. I've always wondered why MySQL doesn't have built in horizontal scaling thats simple for everyone. I've come up with a solution, just have multiple databases with the same table names and columns, and this library will allow you to scale, have as many databases as you want! Before you start please make sure you understand [the basics of PDO](https://secure.php.net/manual/en/book.pdo.php).

## Features
✔ Connect to multiple MySQL databases using PDO, without having performance issues!<br>
✔ Get rows from multiple databases from the same named table.<br>
✔ Perform insert queries efficiently by only doing 1 query instead of adding into all databases/tables.<br>
✔ Free to use, and it's really easy too, which is great!

## Getting Started
Simply include the file `MultiDatabasePDO.php` in your autoload PHP class or include header file on all pages.
Then you can connect to all your databases easily by doing:
```php
$multiPDO = new MultiDatabasePDO([
    ["mysql", "1.1.1.1", "database_1", "username", "password"],
    ["mysql", "2.2.2.2", "database_2", "username", "password"]
]);
```

Now we need to check for any errors using a simple function called `hasAnyErrors()`! You can get the failed connections by calling the function `getFailedConnections()`.
```php
if($multiPDO->hasAnyErrors()) {
    error_log("Error connecting to database(s): " . $multiPDO->getFailedConnections());
    exit("Error connecting to our main databases! Please try again later.");
}
```

## Before Using, Read This
There are some differences between this library and the standard PDO library, where some functions and parameters are different, here are some differences I can think of:
* You can't pass in an array of placeholders/values in the `execute()` method, use `bindValue()` for each placeholder.
* You can't bind PHP variables directly, use `bindValue()` method for assigning values to each placeholder.
* You can't use `ORDER BY`, `LIMIT` or `OFFSET` in your SQL queries, instead please [see this guide](#organising-results).
* Avoid using `AUTO INCREMENT` for columns, instead if you have an ID column [make use of this function here](#random-id-generator).

## The Example Tables
For example purposes, imagine we have the following tables, both called "Users". Each example in this README below will be using these tables and their values/columns. Note that you have to use the same columns for every table in ALL your databases.<br>

**"Users" table, from database 1.**<br>

| ID (int)      | Username (text)     | PassHash (text)     | Email (text)         | FirstName (text) | LastName (text) |
| ------------- | ------------------- | ------------------- | -------------------- | ---------------- | --------------- |
| 1             | WulfGamesYT         | ThLfkbQFyvDx        | wulf@example.com     | Liam             | Allen           |
| 2             | IndianaJones55      | npxCn975RSaP        | im@indiana.jones     | Indiana          | Jones           |
| 3             | YaBoiTableFlipper69 | BT7V2U6VJv2d        | yaboi@gmail.com      | Steve            | Jones           |

**"Users" table, from database 2.**<br>

| ID (int)      | Username (text)     | PassHash (text)     | Email (text)         | FirstName (text) | LastName (text) |
| ------------- | ------------------- | ------------------- | -------------------- | ---------------- | --------------- |
| 4             | ReallyDude          | 6XBmD4bzGP87        | reallydude@yahoo.com | Liam             | Mason           |
| 5             | HellYeaBoi          | LeyTpTwvvMUM        | hellyea@gmail.com    | Julie            | Crosby          |

## Example Query #1: SELECT
To select rows from ALL databases and ALL tables, you can simply do, like normal PDO in PHP:
```php
$selectQuery = $multiPDO->prepare("SELECT ID, Username, Email FROM Users WHERE Username = :username");
$selectQuery->bindValue(":username", "WulfGamesYT");
$selectQuery->execute();
while($row = $selectQuery->getNextRow()) { var_dump($row); }
```

That will produce some example output like:
```
array(3) {
  ["ID"]=>
  int(1)
  ["Username"]=>
  string(11) "WulfGamesYT"
  ["Email"]=>
  string(16) "wulf@example.com"
}
```

## Example Query #2: INSERT
Say if we had a form and you can POST the info to your PHP file, and you want to insert 1 new record into a table from a database called "Users", all you need to do is the following. Note that this will be inserted into the second table in the example tables above because it has the lowest row count.
```php
$insertQuery = $multiPDO->prepare("INSERT INTO Users VALUES (NULL, :username, :pass, :email, :firstname, :lastname)");
$insertQuery->bindValue(":username", $_POST["username"]);
$insertQuery->bindValue(":pass", password_hash($_POST["password"], PASSWORD_DEFAULT));
$insertQuery->bindValue(":email", $_POST["email"]);
$insertQuery->bindValue(":firstname", $_POST["name-first"]);
$insertQuery->bindValue(":lastname", $_POST["name-last"]);
$insertQuery->execute(true, "Users");
```

Notice that with the `execute()` method we pased in 2 parameters, this is required for inserting new rows, because it tells the class we're inserting (by passing in: true) a new row into a table called "Users". Don't put untrusted user input as the second parameter as SQL Injection can occur.

## Example Query #3: UPDATE
This is basically the same as doing a SELECT query, this will update ALL tables in ALL databases that match the WHERE clause if specified, for example:
```php
$updateQuery = $multiPDO->prepare("UPDATE Users SET Username = :newusername WHERE Username = :oldusername");
$updateQuery->bindValue(":newusername", "MyFancyUsername");
$updateQuery->bindValue(":oldusername", "WulfGamesYT");
$updateQuery->execute();
```

Now if we ran a SELECT query on ALL the tables named "Users" we will see the updated row.

## Example Query #4: DELETE
Again, all we need to do is:
```php
$deleteQuery = $multiPDO->prepare("DELETE FROM Users WHERE Username = :username");
$deleteQuery->bindValue(":username", "MyFancyUsername");
$deleteQuery->execute();
```

Now if we ran a SELECT query on ALL the tables named "Users" we will see the updated row.

## Organising Results
It's important to note you can't use `ORDER BY`, `LIMIT` or `OFFSET` in your SQL queries. Instead you have to use the following functions that are available, which make it easy to organise your final results/rows.

**Ordering Results (instead of "ORDER BY"):**
You can order your results just like you can in SQL queries with "ASC" or "DESC" passed into the second parameter to the `sortBy()` method.

This is how you order number columns:
```php
$selectQuery = $multiPDO->prepare("SELECT * FROM Users");
$selectQuery->execute();

//Now sort by the "ID" column with data type: int(11) in descending order.
$selectQuery->sortBy("ID", "DESC");

while($row = $selectQuery->getNextRow()) { var_dump($row); }
```

This is how you order string/object columns:
```php
$selectQuery = $multiPDO->prepare("SELECT * FROM Users");
$selectQuery->execute();

//Now sort by the ID column with data type: int(11) in descending order.
$selectQuery->sortBy("Username", "ASC");

while($row = $selectQuery->getNextRow()) { var_dump($row); }
```

You can order multiple columns, or multiple times if you want. In the example below we will be ordering a column called "FirstName" in descending order, then a column called "LastName". This will list users in the table in alphabetical order, if they have the same first name then it will also order by the last name. Put the least important order column first, then the most important at the end as you can see in the code:
```php
$selectQuery = $multiPDO->prepare("SELECT * FROM Users");
$selectQuery->execute();

//Now sort both the columns.
$selectQuery->sortBy("LastName", "ASC");
$selectQuery->sortBy("FirstName", "ASC");

while($row = $selectQuery->getNextRow()) { var_dump($row); }
```

## Random ID Generator
Instead of `AUTO INCREMENT`, or if you need a way of generating unique strings in your tables for a column, you can make use of a function called `GenerateRandomID()`. Here is an example of how to use it when inserting new rows into your tables:
```php
//Here we generate a truly random string for the "ID" column in the "Users" table.
$randomID = $multiPDO->GenerateRandomID("ID", "Users");

$insertQuery = $multiPDO->prepare("INSERT INTO Users VALUES (:id, :username, :pass, :email, :firstname, :lastname)");
$insertQuery->bindValue(":id", $randomID);
$insertQuery->bindValue(":username", $_POST["username"]);
$insertQuery->bindValue(":pass", password_hash($_POST["password"], PASSWORD_DEFAULT));
$insertQuery->bindValue(":email", $_POST["email"]);
$insertQuery->bindValue(":firstname", $_POST["name-first"]);
$insertQuery->bindValue(":lastname", $_POST["name-last"]);
$insertQuery->execute(true, "Users");
```

## Have Questions?
If you need to ask a question, reach out to me on Twitter.
Twitter: https://www.twitter.com/WulfGamesYT
