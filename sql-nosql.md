# NoSQL vs SQL

## Difference

SQL stores data in tables where each row represents an entity and each column represents a data point about that entity. NoSQL databases have different data storage models. The main ones are key-value, document, graph. etc.

Schema: In SQL, each record conforms to a fixed schema, meaning the columns must be decided and chosen before data entry and each row must have data for each column. The schema can be altered later, but it involves modifying the whole database and going offline. In NoSQL, schemas are dynamic. Columns can be added on the way and each ‘row’ does not have to contain data for each ‘column.’

The vast majority of relational databases are ACID(Atomicity, Consistency, Isolation, Durability) compliant. Most of the NoSQL solutions sacrifice ACID compliance for performance and scalability.

### Storage

SQL: Store data in tables. Relational databases store data in rows and columns. Each row contains all the information about one entity and each column contains all the separate data points.

NoSQL: It has different data storage models. See Most common types of NoSQL below.

### Schema

SQL: Each record conforms to a fixed/predefined schema. Schema can be altered, but it requires modifying the whole database.

NoSQL: Unstructured and have a dynamic schema.

### Querying

SQL: Use SQL (structured query language) for defining and manipulating the data.

NoSQL: UnQL (unstructured query language). Different databases have different syntax.

### Scalability

SQL: Vertically scalable (by increasing the horsepower: memory, CPU, etc) and expensive. Harder to do scale across multiple servers (can be challenging and time-consuming).

NoSQL: Horizontally scalable (by adding more servers) and cheap. Many NoSQL DB also distribute data automatically.

### ACID

Atomicity, consistency, isolation, durability

SQL: ACID compliant. Data reliability. Guarantee of transactions.

NoSQL: Most sacrifice ACID compliance for performance and scalability.

## Which one to choose

### SQL

We need to ensure ACID compliance like many e-commerce and financial applications.

Our data is structured and unchanging.: Data is structured and unchanging in the long term.

### NoSQL database

Undecided structure/Require making frequent updates to the data structure. Data has little or no structure.

Rapid development: Frequent updates to the data structure.

Need Cloud-based storage / spread across multiple servers to scale up (horizontally scalable). (Make the most of cloud computing and storage. Cloud-based storage requires data to be easily spread across multiple servers to scale up.)

### Most common types of NoSQL

#### Key-Value Stores

Data is stored in an array of key-value pairs.
e.g. Redis, Voldemort and Dynamo

#### Document Databases

In these databases data is stored in documents. Each document can have an entirely different structure.
e.g. CouchDB, MongoDB

#### Wide-Column Databases

A wide-column database is a type of NoSQL database in which the names and format of the columns can vary across rows, even within the same table. Wide-column databases are also known as column family databases. Because data is stored in columns, queries for a particular value in a column are very fast, as the entire column can be loaded and searched quickly. Related columns can be modeled as part of the same column family.

e.g. Cassandra and HBase.

#### Graph Databases

These databases are used to store data whose relations are best represented in a graph.
e.g. Neo4J and InfiniteGraph.

## Reference

[1] <https://aaronice.gitbook.io/system-design/distributed-systems/sql-vs-nosql>

[2] Move from <https://wangby511.github.io/2021/10/29/New-System-Design/> in 2022/1/8.

[3] <https://www.scylladb.com/glossary/wide-column-database/>
