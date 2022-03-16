# Chapter 2 Data Models and Query Languages

CREATED 2022/03/09 FINISHED 03/14

## Relational Model vs Document Model

an unordered collection of tuples (rows in SQL)

### The Birth of NoSQL

* A need for greater scalability

* A widespread preference for free and open source software

* Specialized query operations

* A desire for more dynamic and expressive data model

### The Object-Relational Mismatch

E.g. For a data structure like a resume, a JSON representation can be quite appropriate which is mostly a self-contained document. It has better **locality** than the multi-table schema where you need to perform a messy multi-way join between users table and its subordinate tables.

### Many-to-One and Many-to-Many Relationships

Using an ID is more meaningful to database instead of using a text string, which can bring duplication.

Data has a tendency of becoming more interconnected as features are added to applications - like making organizations and schools as entities, making recommendations as a reference to user's profile.

### Are Document Databases Repeating History?

the network model - a record could have multiple parents. The links between records in this model are more like **pointers** and the way of accessing a record following a path is called **an accessed path**. The code for querying and updating is complicated and inflexible.

the relational model - simply a collection of tuples(rows).

The related item is reference by a unique identifier, which is called a **foreign key** in the relational model and a **document reference** in the document model.

### Relational Versus Document Databases Today

If the data in your application code has a document-like structure and use many-to-many relationships, the document model is a good idea. But is has poor support for joins.

Schema flexibility in the document model - **schema on read** : the structure of the data is implicit and only interpreted when read. It is similar to dynamic(runtime) type checking in programming languages. **schema on write** : the traditional approach of relational databases where the schema is explicit and the database ensures all written data confirms to it. It is similar to static(compile-time) type checking.

Data locality for queries - If your application often needs to access the entire document, there is a performance advantage to this storage locality. It is generally recommended that you keep documents fairly small size.

Hybrid mode of combining document and relational databases.

## Queries Languages for Data

### Declarative Queries on the Web

CSS and XSL are both declarative languages.

### MapReduce Querying

It is neither a declarative query language nor a fully imperative query API, but somewhere in between.

## Graph-Like Data Models

The many-to-many relationships are very common in the data. A graph consists of two kinds of objects: vertices (nodes) and edges.

The graph is not only limited to homogeneous data - vertices can represent people, locations, events, comments etc in Facebook.

### Property Graphs

each vertex contains: a unique identifier, a set of incoming edges, a set of outgoing edges and a collection of properties.

each edge contains: a unique identifier, start vertex, end vertex, label and properties.

This graph has a great deal of flexibility for data modeling and it is good for evolvability, like adding features.

### The Cypher Query Language

Cypher is a declarative query language for property graphs, created for the Neo4j graph database.

E.g. find the names of all the people who emigrated from the United States to Europe.

### Graph Queries in SQL

If we put graph data into a relational structure, the querying by SQL will become very difficult and complex.

### Triple-Stores and SPARQL

In a triple store, all information is stored in the form of very simple **three-part statements: subject, predicate, object**. e.g. Jim likes bananas.

Two kinds of objects: (Lucy, age, 33) - a vertex Lucy with property {"age": 33}. (A marriedTo B) - both person A and B are vertices.

E.g. **_:lucy  :bornIn  _:idaho**

The **Resource Description Framework (RDF)** is intended as a mechanism for different websites to publish data in a consistent format.

SPARQL is a query language for triple-stores using the RDF data model.

### The Foundation: Datalog

Datalog is a much older language than SPARQL or Cypher. It is very similar to the triple-store model: we write it in **predicate(subject, object)** format.

We define two new predicates: **within_recursive** and **migrated**. Rules can refer to other rules, just like functions can call other functions or recursively call themselves. E.g. **within_recursive** can tell us all the locations in North America.

## Summary

Document database - data comes in self-contained documents and relationships are rare between one and another.

Graph database - anything is potentially related to everything.
