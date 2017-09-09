---
layout: post
title: "OverTheWire: Natas 0-5"
date: 2016-10-27 12:00:00
share: true
comments: true
tags: [OverTheWire - Natas]
---

Next challenge I'll be writing for is Natas. Extremely fun challenge although sometimes frustrating if you're not sure what you're trying to do. First few challenges are easy, so I'll be doing a few per post.

## Natas 0

It says it's on the page, yet we don't see it, do we? Right click anywhere on
the page and view the source. You'll notice a comment:

```html
<!--The password for natas1 is gtVrDuiDfck831PqWsLEZy5gyDz1clto -->
```

## Natas 1

Eh, more of the same, you got different ways to do this though:

* Disable JavaScript (mayeb use NoScript).
* Use a proxy (Fiddler or Burp suite).
* Right click outside of the white block.

```html
<!--The password for natas2 is ZluruAthQk7Q2MqmDeTiUij2ZvWy2mBi -->
```

___________________________________________

## Natas 2

There's nothing on this page, hmm. Let's check the source again, shall we?

Nothing special about it that `<img>` tag (although would've been cool if the password was in the image comment) but it's under `http://natas2.natas/labs.overthewire.org/files`.

![1]({{ site.baseurl}}/images/1.png)

Open [users.txt](http://natas2.natas.labs.overthewire.org/files/users.txt), you'll find the password to the next level.

```
# username:password
alice:BYNdCesZqW
bob:jw2ueICLvT
charlie:G5vCxkVV3m
**natas3:sJIJNW6ucpu6HPZ1ZAchaDtwd7oGrD14**
eve:zo4mJWyNj2
mallory:9urtcpzBmH
```
___________________________________________

## Natas 3

Let's check the source again.

```html
<!-- No more information leaks!! Not even Google will find it this time... -->
```

Sneaky, maybe we should play a little with [Google operators](http://www.googleguide.com/advanced_operators_reference.html) /[Google dorks](https://www.exploit-db.com/google-hacking-database/)?

![2]({{ site.baseurl}}/images/2.png)

A hidden directory! Again, you'll find your password under [users.txt](http://natas3.natas.labs.overthewire.org/s3cr3t/users.txt).

`natas4:Z9tkRkWmpt9Qr7XrR5jWRkgOU901swEZ`

___________________________________________

## Natas 4

It looks like it's asking about the [referrer](https://en.wikipedia.org/wiki/HTTP_referer)[field](https://en.wikipedia.org/wiki/HTTP_referer) in the HTTP response. It has many applications like tracking source in advertisement, or security related like verifying the user is redirected from the expected page, particularly useful against XSS and session hijacking.


#### Using Fiddler

First, let's capture the request we want to manipulate:

![3]({{ site.baseurl}}/images/3.png)

Drag the request from the log to **"Composer"** tab. You'll get the chance to edit the request before submitting it. Add `Referer: http://natas5.natas.labs.overthewire.org/` then click **"Execute"**.



Go back to the **"Inspector"** tab and below the request you'll find the response body, decode it if needed and you'll find the password.

![4]({{ site.baseurl}}/images/4.png)

Access granted. The password for natas5 is `iX6IOfmpN7AYOQGPwtn3fXpbaJVJcHfq`.

___________________________________________

## Natas 5

```
You're not logged in
```

Well, let's trick the server into believing we're logged in. Let's see what Fiddler reveals in the server response body.

```http
HTTP/1.1 200 OK
Date: Thu, 27 Oct 2016 21:18:31 GMT
Server: Apache/2.4.7 (Ubuntu)
X-Powered-By: PHP/5.5.9-1ubuntu4.20
Set-Cookie: **loggedin=0**
Vary: Accept-Encoding
Content-Length: 855
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
Content-Type: text/html
```


What if we set loggedin to 1? Fire up the console in your browser and type `document.cookie="loggedin=1"`?

Refresh the page, you got your password!

```
Access granted. The password for natas6 is aGoY4q2Dc6MgDq4oL4YtoKtyAg9PeHa1
```