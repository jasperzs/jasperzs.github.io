---
layout:     post
title:      File System vs. Database
subtitle:   Pros, Cons and Use Cases Comparison
date:       2018-10-10
author:     Mr.Humorous 🥘
header-img: img/fileSystemVsDatabase/header.jpeg
catalog: true
tags:
    - File System
    - Database
---

## 1. File System

### 1.1 Pros
+ __Performance can be better.__ If you store large files in db then it may slow down the performance because a Select * query to retrieve the filename will also load the file data.
+ __Saving the files and downloading them in the file system is much simpler.__ A simple Save as function is enough. Downloading can be done by addressing an URL with the location of the saved file.
+ __Migrating the data is an easy process here.__ You can just copy and paste the folder to your desired destination while ensuring right write permissions.
+ __Economical__ in most of the cases to expand your web server rather than paying for certain Databases.
+ __Easy to migrate__ it to Cloud storage like Amazon S3 or CDNs etc in the future.

### 1.2 Cons
+ __Loosely packed.__ No ACID (Atomicity, Consistency, Isolation, Durability) operations. Consider a scenario if your files are deleted from the location manually, you might not know whether the file exists or not.
+ __Low Security.__ Since your files can be saved in a folder where you should have provided write permissions, it is prone to safety issues and invites troubles like hacking.

### 1.3 Preferred Use Cases
+ If your application is liable to handle Large files of size more than 5MB and the massive number say thousands of file uploads.
+ If your application will have a large number of users.


## 2. Database

### 2.1 Pros
+ __ACID (Atomicity, Consistency, Isolation, Durability)__ consistency which includes a rollback of an update that is complicated when the files are stored outside the database.
+ __Files are in sync__ with the database so cannot be orphaned from it which gives you an upper hand in tracking transactions.
+ __Backups automatically__ include file binaries.
+ __More Secure__ than saving in a File System.

### 2.2 Cons
+ May have to __convert the files to blob__ in order to store it in db.
+ Database __Backups__ will become more __hefty and heavy__.
+ __Memory ineffective.__ Often RDBMS’s are RAM driven. So all data has to go to RAM first.

### 2.3 Preferred Use Cases
+ If your user’s file needs to be more tightly coupled, secured and confidential.
+ If your application will not demand a large number of files from a large number of users.

## 3. Questions to help decide
+ Do you need object management?
+ Do you need object relationship management?
+ Do you need transactional data operations?
+ Do you need concurrent access to your data?
+ Do you need indexed data for fast lookups?
+ Do you need intelligent memory managed data?
+ Do you need data redundancy?
+ Will your data management requirements change?

Answering "yes" to any of the above questions may indicate a DBMS is a more appropriate data store.
