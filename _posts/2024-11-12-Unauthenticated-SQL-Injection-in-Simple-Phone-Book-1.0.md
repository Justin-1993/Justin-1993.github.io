---
layout: post
title: Unauthenticated SQL Injection in Simple Phone Book 1.0
date: 2024-11-12 10:59 -0800
tag: [exploit-db]
category: [Vulnerability Research]
---

# Simple Phone Book 1.0 - 'Username' SQL Injection (Unauthenticated)

**Exploit ID**: 50223  
**Author**: Justin White  
**Platform**: PHP  
**Date Published**: 2021-08-23  
**Vulnerable Application**: [Simple Phone Book 1.0](https://www.sourcecodester.com/php/13011/phone-bookphone-directory.html)

## Description

This vulnerability allows unauthenticated SQL Injection in the Simple Phone Book 1.0 web application. The exploit targets the login functionality by injecting SQL commands in the username parameter, allowing an attacker to bypass authentication.

## Vulnerable Parameters

- **Username**: `username1`
- **Password**: `password`

## Proof of Concept (PoC)

To exploit this vulnerability and bypass authentication, use the following input:

- **Username**: `' or sleep(5)='-- -`
- **Password**: `' '`

This input causes a 5-second delay, confirming the SQL Injection vulnerability.

## Vulnerable Code

In `index.php` on line 13, the following code is vulnerable:

``` php
$sql = mysqli_query($dbcon, "SELECT * FROM userdetails WHERE username = '$username' AND password = '$password'");
```

## Additional Information

- **Tested On**: Linux (Ubuntu 20.04) with LAMPP

## Remediation

To fix this vulnerability, implement parameterized queries to prevent SQL injection.

[View this exploit on Exploit-DB](https://www.exploit-db.com/exploits/50223)
