# NoSQL vs SQL

## Difference

SQL stores data in tables where each row represents an entity and each column represents a data point about that entity. NoSQL databases have different data storage models. The main ones are key-value, document, graph. etc.

Schema: In SQL, each record conforms to a fixed schema, meaning the columns must be decided and chosen before data entry and each row must have data for each column. The schema can be altered later, but it involves modifying the whole database and going offline. In NoSQL, schemas are dynamic. Columns can be added on the way and each ‘row’ does not have to contain data for each ‘column.’

The vast majority of relational databases are ACID(Atomicity, Consistency, Isolation, Durability) compliant. Most of the NoSQL solutions sacrifice ACID compliance for performance and scalability.

### Storage

SQL: Store data in tables.
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

ACID compliant: data reliability and safe guarantee.

Predefined schema: Data is structured and unchanging in the long term.

In most common situations, SQL databases only support vertically scalable.

### NoSQL database

Undecided structure/Require making frequent updates to the data structure. Data has little or no structure.

Rapid development: Frequent updates to the data structure.

Need Cloud-based storage / spread across multiple servers to scale up (horizontally scalable). (Make the most of cloud computing and storage. Cloud-based storage requires data to be easily spread across multiple servers to scale up.)

### Most common types of NoSQL

#### Key-Value Stores

Data is stored in an array of key-value pairs.
e.g. Redis, Voldemort and Dynamo

#### Document Databases

In these databases data is stored in documents
e.g. CouchDB, MongoDB

#### Wide-Column Databases

Instead of ‘tables,’ in columnar databases we have column families, which are containers for rows. (Unlike relational databases, we don’t need to know all the columns up front, and each row doesn’t have to have the same number of columns.)
e.g. Cassandra and HBase.

#### Graph Databases

These databases are used to store data whose relations are best represented in a graph.
e.g. Neo4J and InfiniteGraph.

## Reference

[1] <https://aaronice.gitbook.io/system-design/distributed-systems/sql-vs-nosql>

[2] Move from <https://wangby511.github.io/2021/10/29/New-System-Design/> in 2022/1/8.
