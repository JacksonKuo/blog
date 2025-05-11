---
layout: post
title: "Database: Lambda Backup"
date: 2025-5-12
tags: ["database"]
published: false
---

**Contents**
* TOC
{:toc}

# Problem Statement
Setup a DB with columns that have IP and JSON data. Using Python and SQL, efficiently query both IP and JSON data using indexing and serialize the output to a file in S3.

#### Database Backup 

* Setup database
* Run script to save log to file
    * runtime options
        * export cmd to CSV 
        * python lambda
        * stored procedure
    * parsing
        * json.load / alchemy
    * state
        * env var
        * db column
        * bucket file itself

Assume python script runs every hour. 
Assume you have some state mechanism
Assume lambda can fail

Efficiency and Speed
* indexing
* serialize

columnar db

# References
[^1]: [https://nerderati.com/a-python-epoch-timestamp-timezone-trap/](https://nerderati.com/a-python-epoch-timestamp-timezone-trap/)

