---
layout:     post
title:      The Main NoSQL Database Types
subtitle:   Introduction to NoSQL Databases
date:       2019-04-01
author:     Mr.Humorous 🥘
header-img: img/database/header.jpg
catalog: true
tags:
    - Database
---

## 1. Five Main NoSQL Database Types
- [Wide-Column Store](#2-wide-column-store)
- [Document Store](#3-document-store)
- [Key-Value Data Store](#4-key-value-data-store)
- [Graph Store](#5-graph-store)
- [Multi-Model](#6-multi-model)

## 2. Wide-Column Store
As the name suggests, wide-column stores use columns to store data. You can group related columns into column families. Individual rows then constitute a column family.

Sounds a bit confusing, doesn’t it? Visuals should help. Here’s what an individual row within a column family looks like in Cassandra:
![Individual row in column family illustration](/img/database/mainNoSQLDBTypes/wideColumnStore/individualRowInColumnFamily.png)

A typical column family contains a row key as the first column, which uniquely identifies that row within the column family. The following columns then contain a column key, which uniquely identifies that column within the row, so that you can query their corresponding column values.

To better illustrate, here’s how data in a table would translate to a column family:
![Table Translate To Column Family Illustration](/img/database/mainNoSQLDBTypes/wideColumnStore/tableTranslateToColumnFamily.png)

### 2.1 Examples of Column Databases
- [Cassandra](https://cassandra.apache.org/) – Free, open-source
- [HBase](https://hbase.apache.org/) – Free, open-source

### 2.2 Use Cases
Column stores offer very high performance and a highly scalable architecture. Because they’re fast to load and query, they’ve been popular among companies and organizations dealing with big data, IoT, and user recommendation and personalization engines.
- Spotify uses Cassandra to store user profile attributes and metadata about artists, songs, etc. for better personalization
- Facebook initially built its revamped Messages on top of HBase, but is now also used for other Facebook services like the Nearby Friends feature and search indexing
- Outbrain uses Cassandra to serve over 190 billion personalized content recommendations each month

## 3. Document Store
Document store databases use JSON, XML, or BSON documents to store data. You can pretty much fill in a document with whatever data you want. Documents can have different structures, but you can index common fields for faster performance.

Here’s an example of a document in MongoDB:
![MongoDB Illustration](/img/database/mainNoSQLDBTypes/documentStore/mongoDB.png)

### 3.1 Examples of Document Databases
- [MongoDB](https://www.mongodb.com/) – Free, open-source
- [Couchbase](https://www.couchbase.com/) – Free, open-source

### 3.2 Use Cases
- SEGA uses MongoDB for handling 11 million in-game accounts
- Cisco moved its VSRM (video session and research manager) platform to Couchbase to achieve greater scalability
- Aer Lingus uses MongoDB with Studio 3T to handle ticketing and internal apps
- Built on MongoDB, The Weather Channel’s iOS and Android apps deliver weather alerts to 40 million users in real-time

## 4. Key-Value Data Store
Key-value databases store data in unique key-value pairs, meaning each key is associated only with one value in a collection, like in a dictionary. This makes querying a key-value data store fast because of its simple model. There’s no query language needed. Data retrieval is a simple matter of using get, put, and delete commands.

### 4.1 Examples
- [Redis](https://redis.io/) – Free, open-source
- [Memcached](https://memcached.org/) – Free, open-source

### 4.2 Use cases
Key-value data stores are ideal for storing user profiles, blog comments, product recommendations, and session information.
- Twitter uses Redis to deliver your Twitter timeline
- Pinterest uses Redis to store lists of users, followers, unfollowers, boards, and more
- Coinbase uses Redis to enforce rate limits and guarantee correctness of Bitcoin transactions
- Quora uses Memcached to cache results from slower, persistent databases

## 5. Graph Store
Nodes and relationships are the bases of graph databases. A node represents an entity, like a user, category, or a piece of data. A relationship represents how two nodes are associated.

Graph databases use nodes that contain lists of relationship records. These records represent relationships with other nodes, which eliminates the (time-consuming) search-match operation found in relational databases.

The image below shows how a relational database like MySQL works, which use memory-intensive and more complicated join operations to search entire tables to find a match:
![Relational DB Find A Match Illustration](/img/database/mainNoSQLDBTypes/graphStore/relationalDBFindAMatch.png)

Compare that to a graph database, which already predetermines relationships based on connected nodes, making queries much faster and more economical.
![Graph Store Find A Match Illustration](/img/database/mainNoSQLDBTypes/graphStore/graphStoreFindAMatch.png)

### 5.1 Examples of Graph Databases
- [Neo4j](https://neo4j.com/) – Free, open-source
- [JanusGraph](http://janusgraph.org/) – Free, open-source

### 5.2 Use Cases
Graph databases are great for establishing data relationships especially when dealing with large data sets. They offer blazing fast query performance that relational databases cannot compete with, especially as data grows much deeper.
- Walmart uses Neo4j to provide customers personalized, relevant product recommendations and promotions
- Medium uses Neo4j to build their social graph to enhance content personalization
- Cisco uses Neo4j to mine customer support cases to anticipate bugs

## 6. Multi-Model
It’s clear that the four previous NoSQL database types have their respective advantages, which is why the multi-model database has emerged as another popular option in recent years. As its name suggests, a multi-model database combines the capabilities of wide-column, graph, document, and key-value databases. More supported data models within one database means more versatility for you – the end user – in the way you store your data.

### 6.1 Examples of Multi-Model Databases
- [Microsoft Azure Cosmos DB](https://docs.microsoft.com/en-us/azure/cosmos-db/introduction) – Pricing starts at $0.25/GB, free Emulator available
- [OrientDB](https://orientdb.com/orientdb/) – Free, open source

### 6.2 Use Cases
- RTE’s Investigations Unit uses OrientDB to establish data connections across various sources for investigative purposes
- Azure CosmosDB supports Citrix’s Identity Platform, which enables single-sign on for more than 400,000 organizations and 100 million individuals worldwide
- ASOS uses Azure Cosmos DB to deliver real-time product recommendations and instant order updates for 15.4 million customers
