---
layout: post
title: "Database: SQL Refresher Course"
date: 2025-5-15
tags: ["database"]
published: false
---

**Contents**
* TOC
{:toc}

# Problem Statement
Refresh on SQL fundamentals using O'Reilly's Learning SQL book: [https://www.amazon.com/gp/product/B085HDSWR3/](https://www.amazon.com/gp/product/B085HDSWR3/).

# Chapters

* Chapter 1 - A little background
    * Postgres allows a max of 1600 columns per table
    * *primary key: uniquely identifies a row in that table*
    * *primary key consisting of two or more columns is known as a compound key*
    * natural key: A natural key is a column or set of columns that already exist in the table, e.g. first_name[^1]
    * surrogate key: system generated (could be GUID, sequence, unique identifier, etc.) value with no business meaning[^1]
    * *Some of the tables also include information used to navigate to another table...These columns are known as foreign keys*
    * using foreign keys find rows in other tables is known as a join
    * *normalization: refining a database design to ensure that each independent piece of information is in only one place (except for foreign keys)*
    * *result set: result of an SQL query is a table*
    * *foreign keys: one or more columns that can be used together to identify a single row in another table*
    * *SQL schema statements: define the data structures stored in the database*
    * *database elements created via SQL schema statements are stored in a special set of tables called the data dictionary... known collectively as metadata*
    * *manner in which a statement is executed is left to a component of your database engine known as the optimizer*
    * *all examples for this book be run against a MySQL (version 8.0) database,*
* Chapter 2 - Creating and populating a database
    * `show databases;`
    * `SELECT now();`
    * *With some database servers, you won’t be able to issue a query without a from clause that names at least one table... Oracle provides a table called dual, which consists of a single column called dummy that contains a single row of data.*
    * `SELECT now() FROM dual;`
    * *When defining a character column, you must specify the maximum size of any string to be stored in the column.*
    * `char(20)    /* fixed-length*`
    * `varchar(20) /* variable-length*`
    * *maximum length for char columns is currently 255 bytes, whereas varchar columns can be up to 65,535 bytes.*
    * *MySQL can store data using various character sets, both single- and multibyte.*
    * `SHOW CHARACTER SET;`
    * `varchar(20) character set latin1`
    * *If the data being loaded into a text column exceeds the maximum size for that type, the data will be truncated.*
    * *When using text columns for sorting or grouping, only the first 1,024 bytes are used*
    * *dates and/or times. This type of data is referred to as temporal*
    * timestamp/datetime YYYY-MM-DD HH:MI:SS, difference is datetime has a larger range of years 1000-9999, compared to timestamp 1970-2038. Differs per database
    * *normalization, which is the process of ensuring that there are no duplicate (other than foreign keys) or compound columns in your database design.*
    * ```
    CREATE TABLE person  
    (person_id SMALLINT UNSIGNED,   
        fname VARCHAR(20),   
        lname VARCHAR(20),  
        country VARCHAR(20),    
        CONSTRAINT pk_person PRIMARY KEY (person_id)  
    );
    ```
    * *You can add several types of constraints to a table definition.*
    * *primary key constraint: creating a constraint on the table... created on the person_id column and given the name pk_person*
    * *check constraint: constrains the allowable values for a particular column*
    * `eye_color CHAR(2) CHECK (eye_color IN ('BR','BL','GR')),`
    * `eye_color ENUM('BR','BL','GR')`
    * *describe command (or desc for short) to look at the table definition*
    * `desc person;`
    * *foreign key constraint. This constrains the values of the person_id column in the favorite_food table to include only values found in the person table.*
    * `CONSTRAINT fk_fav_food_person_id FOREIGN KEY (person_id)`
* Chapter 3 - Query primer
* Chapter 4 - Filtering
* Chapter 5 - Querying multiple tables
* Chapter 6 - Working with sets
* Chapter 7 - Data generation, manipulation, and conversion
* Chapter 8 - Grouping and aggregates
* Chapter 9 - Subqueries
* Chapter 10 - Joins revisited
* Chapter 11 - Conditional logics
* Chapter 12 - Transactions
* Chapter 13 - Indexes and constraints
* Chapter 14 - Views
* Chapter 15 - Metadata
* Chapter 16 - Analytics functions
* Chapter 17 - Working with large databases
* Chapter 18 - SQL and big data

# References
[^1]: [https://www.mssqltips.com/sqlservertip/5431/surrogate-key-vs-natural-key-differences-and-when-to-use-in-sql-server/](https://www.mssqltips.com/sqlservertip/5431/surrogate-key-vs-natural-key-differences-and-when-to-use-in-sql-server/)

[^2]: []()
[^3]: []()
[^4]: []()
[^5]: []()