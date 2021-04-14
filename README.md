# Database-Notes

## Database Replication

https://dev.mysql.com/doc/refman/5.5/en/replication.html

## ElasticSearch

### Supported Geo-Shapes

https://www.elastic.co/guide/en/elasticsearch/reference/current/geo-shape.html#input-structure

### Join/Parent-child

https://medium.com/swlh/parent-and-child-joins-with-elasticsearch-7-381f6cca73fe
https://www.elastic.co/guide/en/elasticsearch/reference/current/parent-join.html

### Glossary of terms

https://www.elastic.co/guide/en/elasticsearch/reference/current/glossary.html#index

### What are mapping types?

https://www.elastic.co/guide/en/elasticsearch/reference/7.9/removal-of-types.html

### Analogy to relational database

**Before ElasticSearch 8.0 (before removing the Type)**

The easiest and most familiar layout clones what you would expect from a relational database. You can (very roughly) think of an index like a database.

```
MySQL         => Databases => Tables => Columns/Rows
Elasticsearch => Indices   => Types  => Documents with Properties
```

An Elasticsearch cluster can contain multiple Indices (databases), which in turn contain multiple Types (tables). These types hold multiple Documents (rows), and each document has Properties(columns).

So in your car manufacturing scenario, you may have a SubaruFactory index. Within this index, you have three different types:

`People`

`Cars`

`Spare_Parts`

Each type then contains documents that correspond to that type (e.g. a Subaru Imprezza doc lives inside of the Cars type. This doc contains all the details about that particular car).

Searching and querying takes the format of: http://localhost:9200/[index]/[type]/[operation]

So to retrieve the Subaru document, I may do this:

```xml
<strong>$ curl -XGET localhost:9200/SubaruFactory/Cars/SubaruImprezza
</strong>
```

**After ElasticSearch 8.0 (after removing the Type)**

After Type is removed, there will be only one Type in a Index, that is

```
MySQL =>         Databases => Tables  => Columns/Rows
Elasticsearch => <N/A>     => Indices => Documents with Properties
```


### Advantages

- Elasticsearch is developed on Java, which makes it compatible on almost every platform.

- Elasticsearch is real time, in other words after one second the added document is searchable in this engine.

- Elasticsearch is distributed, which makes it easy to scale and integrate in any big organization.

- Creating full backups are easy by using the concept of gateway, which is present in Elasticsearch.

- Handling multi-tenancy is very easy in Elasticsearch when compared to Apache Solr.

- Elasticsearch uses JSON objects as responses, which makes it possible to invoke the Elasticsearch server with a large number of different programming languages.

- Elasticsearch supports almost every document type except those that do not support text rendering.

### Key Concept

https://www.elastic.co/guide/en/elasticsearch/reference/current/glossary.html

The key concepts of Elasticsearch are as follows −

**analysis**

Analysis is the process of converting full text to terms. Depending on which analyzer is used, these phrases: FOO BAR, Foo-Bar, foo,bar will probably all result in the terms foo and bar. These terms are what is actually stored in the index. A full text query (not a term query) for FoO:bAR will also be analyzed to the terms foo,bar and will thus match the terms stored in the index. It is this process of analysis (both at index time and at search time) that allows elasticsearch to perform full text queries. Also see text and term.

**cluster**

A cluster consists of one or more nodes which share the same cluster name. Each cluster has a single master node which is chosen automatically by the cluster and which can be replaced if the current master node fails.

**document**

A document is a JSON document which is stored in elasticsearch. It is like a row in a table in a relational database. Each document is stored in an index and has a type and an id. A document is a JSON object (also known in other languages as a hash / hashmap / associative array) which contains zero or more fields, or key-value pairs. The original JSON document that is indexed will be stored in the _source field, which is returned by default when getting or searching for a document.

**id**

The ID of a document identifies a document. The index/id of a document must be unique. If no ID is provided, then it will be auto-generated. (also see routing)

**field**

A document contains a list of fields, or key-value pairs. The value can be a simple (scalar) value (eg a string, integer, date), or a nested structure like an array or an object. A field is similar to a column in a table in a relational database. The mapping for each field has a field type (not to be confused with document type) which indicates the type of data that can be stored in that field, eg integer, string, object. The mapping also allows you to define (amongst other things) how the value for a field should be analyzed.

**index**

An index is like a table in a relational database. It has a mapping which contains a type, which contains the fields in the index. An index is a logical namespace which maps to one or more primary shards and can have zero or more replica shards.

**mapping**

A mapping is like a schema definition in a relational database. Each index has a mapping, which defines a type, plus a number of index-wide settings. A mapping can either be defined explicitly, or it will be generated automatically when a document is indexed.

**node**

A node is a running instance of elasticsearch which belongs to a cluster. Multiple nodes can be started on a single server for testing purposes, but usually you should have one node per server. At startup, a node will use unicast to discover an existing cluster with the same cluster name and will try to join that cluster.

**primary shard**

Each document is stored in a single primary shard. When you index a document, it is indexed first on the primary shard, then on all replicas of the primary shard. By default, an index has 5 primary shards. You can specify fewer or more primary shards to scale the number of documents that your index can handle. You cannot change the number of primary shards in an index, once the index is created. See also routing

**replica shard**

Each primary shard can have zero or more replicas. A replica is a copy of the primary shard, and has two purposes:

increase failover: a replica shard can be promoted to a primary shard if the primary fails
increase performance: get and search requests can be handled by primary or replica shards. By default, each primary shard has one replica, but the number of replicas can be changed dynamically on an existing index. A replica shard will never be started on the same node as its primary shard.

**routing**

When you index a document, it is stored on a single primary shard. That shard is chosen by hashing the routing value. By default, the routing value is derived from the ID of the document or, if the document has a specified parent document, from the ID of the parent document (to ensure that child and parent documents are stored on the same shard). This value can be overridden by specifying a routing value at index time, or a routing field in the mapping.

**shard**

A shard is a single Lucene instance. It is a low-level “worker” unit which is managed automatically by elasticsearch. An index is a logical namespace which points to primary and replica shards. Other than defining the number of primary and replica shards that an index should have, you never need to refer to shards directly. Instead, your code should deal only with an index. Elasticsearch distributes shards amongst all nodes in the cluster, and can move shards automatically from one node to another in the case of node failure, or the addition of new nodes.

**source field**

By default, the JSON document that you index will be stored in the _source field and will be returned by all get and search requests. This allows you access to the original object directly from search results, rather than requiring a second step to retrieve the object from an ID.

**term**

A term is an exact value that is indexed in elasticsearch. The terms foo, Foo, FOO are NOT equivalent. Terms (i.e. exact values) can be searched for using term queries. See also text and analysis.

**text**

Text (or full text) is ordinary unstructured text, such as this paragraph. By default, text will be analyzed into terms, which is what is actually stored in the index. Text fields need to be analyzed at index time in order to be searchable as full text, and keywords in full text queries must be analyzed at search time to produce (and search for) the same terms that were generated at index time. See also term and analysis.

**type**

A type used to represent the type of document, e.g. an email, a user, or a tweet. Types are deprecated and are in the process of being removed. See Removal of mapping types.

### Sharding

https://medium.com/@jeeyoungk/how-sharding-works-b4dec46b3f6

A database shard is a horizontal partition, the partitioning can be done by either building separate smaller databases (each with its own tables, indices, and transaction logs), or by splitting selected elements, for example just one table.

Horizontal partitioning involves putting different rows into different tables. For example, customers with ZIP codes less than 50000 are stored in CustomersEast, while customers with ZIP codes greater than or equal to 50000 are stored in CustomersWest. The two partition tables are then CustomersEast and CustomersWest, while a view with a union might be created over both of them to provide a complete view of all customers.

Vertical partitioning involves creating tables with fewer columns and using additional tables to store the remaining columns. Normalization also involves this splitting of columns across tables, but vertical partitioning goes beyond that and partitions columns even when already normalized. Different physical storage might be used to realize vertical partitioning as well; storing infrequently used or very wide columns on a different device, for example, is a method of vertical partitioning. Done explicitly or implicitly, this type of partitioning is called "row splitting" (the row is split by its columns). A common form of vertical partitioning is to split dynamic data (slow to find) from static data (fast to find) in a table where the dynamic data is not used as often as the static. Creating a view across the two newly created tables restores the original table with a performance penalty, however performance will increase when accessing the static data e.g. for statistical analysis.

### Example of vertical partitioning

```python
fetch_user_data(user_id) -> db["USER"].fetch(user_id)
fetch_photo(photo_id) ->    db["PHOTO"].fetch(photo_id)

# Example of horizontal partitioning
fetch_user_data(user_id) -> user_db[user_id % 2].fetch(user_id)
```

### Difference between a Database(what we see as a DB user) & a Storage Engine & Relational engine

When you submit a query to SQL Server, a number of processes on the server go to work on that query. The purpose of all these processes is to manage the system such that it will SELECT, INSERT, UPDATE or DELETE the data.These processes kick into action every time we submit a query to the system. The processes for meeting the requirements of queries break down roughly into two stages:

- 1-Processes that occur in the relational engine.

- 2-Processes that occur in the storage engine. 

In the relational engine, the query is parsed and then processed by the query optimizer, which generates an execution plan. The plan is sent (in a binary format) to the storage engine, which then uses that plan as a basis to retrieve or modify the underlying data. The storage engine is where processes such as locking, index maintenance, and transactions occur.

### Isolation

https://www.youtube.com/watch?v=7nv-7XQI7p0

https://www.youtube.com/watch?v=ZtPj09tJjnQ

| Isolation Level | Transactions | Dirty Reads | Non-Repeatable Reads | Phantom Reads |
| ----------------|--------------|-------------|----------------------|---------------|
| TRANSACTION_None | Not supported	| Not applicable | Not applicable | Not applicable | 
| TRANSACTION_READ_COMMITTED | Supported | Prevented | Allowed  | Allowed |
| TRANSACTION_READ_UNCOMMITTED | Supported | Allowed | Allowed | Allowed |
| TRANSACTION_REPEATABLE_READ | Supported | Prevented | Prevented | Allowed |
| TRANSACTION_SERIALIZABLE | Supported | Prevented | Prevented | Prevented |

**Dirty Read**

**Dirty read** occurs when one transaction is changing the record, and the other transaction can read this record before the first transaction has been committed or rolled back. This is known as a dirty read scenario because there is always the possibility that the first transaction may rollback the change, resulting in the second transaction having read an invalid data.

Transaction A begins.

```sql
UPDATE EMPLOYEE SET SALARY = 10000 WHERE EMP_ID= ‘123’;
```

Transaction B begins.

```sql
SELECT * FROM EMPLOYEE;
```

(Transaction B sees data which is updated by transaction A. But, those updates have not yet been committed.)

**Non-Repeatable**

A non-repeatable read occurs, when during the course of a transaction, a row is retrieved twice and the values within the row differ between reads.

A **non-repeatable** read occurs when transaction A retrieves a row, transaction B subsequently updates the row, and transaction A later retrieves the same row again. Transaction A retrieves the same row twice but sees different data.

In **Mysql**, REPEATABLE READ is different from SQL Server.

Example:

```
+------+----------------------------+--------------+-------+------+
| id   | title                      | author       | price | qty  |
+------+----------------------------+--------------+-------+------+
| 1001 | Java for dummies           | Tan Ah Teck  | 11.11 |   11 |
| 1002 | More Java for dummies      | Tan Ah Teck  | 22.22 |   22 |
| 1003 | More Java for more dummies | Mohammad Ali | 33.33 |   33 |
| 1004 | A Cup of Java              | Kumar        | 44.44 |   44 |
| 1005 | A Teaspoon of Java         | Kevin Jones  | 55.55 |   55 |
| 1006 | JAVA                       | dong         | 33.33 |    3 |
| 1007 | JAdsfA                     | dsdfong      | 33.33 |    3 |
| 1008 | JsdfAdsfA                  | dssdfdfong   | 33.33 |    3 |
| 1009 | JsdfAdsfA                  | dssdfdfong   | 33.33 |    3 |
-------------------------------------------------------------------
```

transaction 1 with REPEATABLE READ isolation level

```sql
SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;
START TRANSACTION;
select * from books; -- select 1
select sleep(10); -- sleep 1
select * from books; -- select 2
COMMIT;
```

transaction 2

```sql
DELETE from books WHERE id=1009; -- delete 1
```

If I execute transaction 2 while transaction 1 is doing sleep 1. select 2 will produce same result as select 1, because select 2 read the snapshot established by the select 1. But transaction 2 is not blocked

result will be:

```
+------+----------------------------+--------------+-------+------+
| id   | title                      | author       | price | qty  |
+------+----------------------------+--------------+-------+------+
| 1001 | Java for dummies           | Tan Ah Teck  | 11.11 |   11 |
| 1002 | More Java for dummies      | Tan Ah Teck  | 22.22 |   22 |
| 1003 | More Java for more dummies | Mohammad Ali | 33.33 |   33 |
| 1004 | A Cup of Java              | Kumar        | 44.44 |   44 |
| 1005 | A Teaspoon of Java         | Kevin Jones  | 55.55 |   55 |
| 1006 | JAVA                       | dong         | 33.33 |    3 |
| 1007 | JAdsfA                     | dsdfong      | 33.33 |    3 |
| 1008 | JsdfAdsfA                  | dssdfdfong   | 33.33 |    3 |
| 1009 | JsdfAdsfA                  | dssdfdfong   | 33.33 |    3 |
-------------------------------------------------------------------
```

Although **SQL Server** gives the same output however transaction 2 will be blocked until transaction 1 finishes.

If we do INSERT instead of DELETE in transaction, that will be phantom read. INSERT will not be blocked.

Note that 1009 is still there because select 1 is before delete 1 because these restriction only takes effect when first read happens. If delete happens before select 1, result will be:

```
+------+----------------------------+--------------+-------+------+
| id   | title                      | author       | price | qty  |
+------+----------------------------+--------------+-------+------+
| 1001 | Java for dummies           | Tan Ah Teck  | 11.11 |   11 |
| 1002 | More Java for dummies      | Tan Ah Teck  | 22.22 |   22 |
| 1003 | More Java for more dummies | Mohammad Ali | 33.33 |   33 |
| 1004 | A Cup of Java              | Kumar        | 44.44 |   44 |
| 1005 | A Teaspoon of Java         | Kevin Jones  | 55.55 |   55 |
| 1006 | JAVA                       | dong         | 33.33 |    3 |
| 1007 | JAdsfA                     | dsdfong      | 33.33 |    3 |
| 1008 | JsdfAdsfA                  | dssdfdfong   | 33.33 |    3 |
-------------------------------------------------------------------
```

**Phantom Read**

A **phantom read** occurs when transaction A retrieves a set of rows satisfying a given condition, transaction B subsequently inserts or updates a row such that the row now meets the condition in transaction A, and transaction A later repeats the conditional retrieval. Transaction A now sees an additional row. This row is referred to as a phantom.

same example:

```
+------+----------------------------+--------------+-------+------+
| id   | title                      | author       | price | qty  |
+------+----------------------------+--------------+-------+------+
| 1001 | Java for dummies           | Tan Ah Teck  | 11.11 |   11 |
| 1002 | More Java for dummies      | Tan Ah Teck  | 22.22 |   22 |
| 1003 | More Java for more dummies | Mohammad Ali | 33.33 |   33 |
| 1004 | A Cup of Java              | Kumar        | 44.44 |   44 |
| 1005 | A Teaspoon of Java         | Kevin Jones  | 55.55 |   55 |
| 1006 | JAVA                       | dong         | 33.33 |    3 |
| 1007 | JAdsfA                     | dsdfong      | 33.33 |    3 |
| 1008 | JsdfAdsfA                  | dssdfdfong   | 33.33 |    3 |
| 1009 | JsdfAdsfA                  | dssdfdfong   | 33.33 |    3 |
-------------------------------------------------------------------
```

transaction 1 with SERIALIZABLE isolation level

```sql
SET SESSION TRANSACTION ISOLATION LEVEL SERIALIZABLE;
START TRANSACTION;
select * from books; -- select 1
select sleep(10); -- sleep 1
select * from books; -- select 2
COMMIT;
```

transaction 2

```sql
DELETE from books WHERE id=1009; -- delete 1
```

If I execute transaction 2 while transaction 1 is doing sleep 1. transaction 2 will wait until transaction 1 finishes and output will be:

```
+------+----------------------------+--------------+-------+------+
| id   | title                      | author       | price | qty  |
+------+----------------------------+--------------+-------+------+
| 1001 | Java for dummies           | Tan Ah Teck  | 11.11 |   11 |
| 1002 | More Java for dummies      | Tan Ah Teck  | 22.22 |   22 |
| 1003 | More Java for more dummies | Mohammad Ali | 33.33 |   33 |
| 1004 | A Cup of Java              | Kumar        | 44.44 |   44 |
| 1005 | A Teaspoon of Java         | Kevin Jones  | 55.55 |   55 |
| 1006 | JAVA                       | dong         | 33.33 |    3 |
| 1007 | JAdsfA                     | dsdfong      | 33.33 |    3 |
| 1008 | JsdfAdsfA                  | dssdfdfong   | 33.33 |    3 |
| 1009 | JsdfAdsfA                  | dssdfdfong   | 33.33 |    3 |
-------------------------------------------------------------------
```

### Transactions & Locking tables

https://www.sqlshack.com/locking-sql-server/

short version:

- transactions as securing things inside your code path. 
- Locks secure things across "parallel" code paths. Until deadlocks hit..

long version:

Locking tables prevents other DB users from affecting the rows/tables you've locked. But locks, in and of themselves, will NOT ensure that your logic comes out in a consistent state.

Think of a banking system. When you pay a bill online, there's at least two accounts affected by the transaction: Your account, from which the money is taken. And the receiver's account, into which the money is transferred. And the bank's account, into which they'll happily deposit all the service fees charged on the transaction. Given (as everyone knows these days) that banks are extraordinarily stupid, let's say their system works like this:

```sql
$balance = "GET BALANCE FROM your ACCOUNT";
if ($balance < $amount_being_paid) {
    charge_huge_overdraft_fees();
}
$balance = $balance - $amount_being paid;
UPDATE your ACCOUNT SET BALANCE = $balance;

$balance = "GET BALANCE FROM receiver ACCOUNT"
charge_insane_transaction_fee();
$balance = $balance + $amount_being_paid
UPDATE receiver ACCOUNT SET BALANCE = $balance
```

Now, with no locks and no transactions, this system is vulnerable to various race conditions, the biggest of which is multiple payments being performed on your account, or the receiver's account in parallel. While your code has your balance retrieved and is doing the huge_overdraft_fees() and whatnot, it's entirely possible that some other payment will be running the same type of code in parallel. They'll be retrieve your balance (say, $100), do their transactions (take out the $20 you're paying, and the $30 they're screwing you over with), and now both code paths have two different balances: $80 and $70. Depending on which ones finishes last, you'll end up with either of those two balances in your account, instead of the $50 you should have ended up with ($100 - $20 - $30). In this case, "bank error in your favor".

Now, let's say you use locks. Your bill payment ($20) hits the pipe first, so it wins and locks your account record. Now you've got exclusive use, and can deduct the $20 from the balance, and write the new balance back in peace... and your account ends up with $80 as is expected. But... uhoh... You try to go update the receiver's account, and it's locked, and locked longer than the code allows, timing out your transaction... We're dealing with stupid banks, so instead of having proper error handling, the code just pulls an exit(), and your $20 vanishes into a puff of electrons. Now you're out $20, and you still owe $20 to the receiver, and your telephone gets repossessed.

So... enter transactions. You start a transaction, you debit your account $20, you try to credit the receiver with $20... and something blows up again. But this time, instead of exit(), the code can just do rollback, and poof, your $20 is magically added back to your account.

In the end, it boils down to this:

Locks keep anyone else from interfering with any database records you're dealing with. Transactions keep any "later" errors from interfering with "earlier" things you've done. Neither alone can guarantee that things work out ok in the end. But together, they do.

in tomorrow's lesson: The Joy of Deadlocks.

**table-level & row-level Locking **

https://dev.mysql.com/doc/refman/5.5/en/innodb-locking.html

https://makandracards.com/makandra/31937-differences-between-transactions-and-locking

**Row-level locking**

InnoDB implements standard row-level locking where there are two types of locks, shared (S) locks and exclusive (X) locks.

A shared (S) lock permits the transaction that holds the lock to read a row.

An exclusive (X) lock permits the transaction that holds the lock to update or delete a row.

If transaction T1 holds a shared (S) lock on row r, then requests from some distinct transaction T2 for a lock on row r are handled as follows:

A request by T2 for an S lock can be granted immediately. As a result, both T1 and T2 hold an S lock on r.

A request by T2 for an X lock cannot be granted immediately.

If a transaction T1 holds an exclusive (X) lock on row r, a request from some distinct transaction T2 for a lock of either type on r cannot be granted immediately. Instead, transaction T2 has to wait for transaction T1 to release its lock on row r.

`SELECT ... LOCK IN SHARE MODE`

Sets a shared mode lock on any rows that are read. Other sessions can read the rows, but cannot modify them until your transaction commits. If any of these rows were changed by another transaction that has not yet committed, your query waits until that transaction ends and then uses the latest values.

`SELECT ... FOR UPDATE`

For index records the search encounters, locks the rows and any associated index entries, the same as if you issued an UPDATE statement for those rows. Other transactions are blocked from updating those rows, from doing SELECT ... LOCK IN SHARE MODE, or from reading the data in certain transaction isolation levels. Consistent reads ignore any locks set on the records that exist in the read view. (Old versions of a record cannot be locked; they are reconstructed by applying undo logs on an in-memory copy of the record.)

**What happens if two mySQL queries both try to lock the same row at the same time?**

One transaction will simply block -- waiting to acquire the lock. The other transaction will proceed. As soon as the other transaction is done -- either by commit or rollback, the first transaction will proceed.

A deadlock happens when a transaction has acquired a lock on object A, and attempts to acquire a lock on object B at the same time as another transaction has already acquired a lock on object B and is attempting to acquire a lock on object A. Both transactions will then block, waiting on each other. That's the definition of a deadlock: two transactions blocked waiting on a lock that the other has.

### ACID properties

**Atomicity**

Atomicity requires that each transaction be "all or nothing": if one part of the transaction fails, then the entire transaction fails, and the database state is left unchanged. An atomic system must guarantee atomicity in each and every situation, including power failures, errors and crashes. To the outside world, a committed transaction appears (by its effects on the database) to be indivisible ("atomic"), and an aborted transaction does not happen.

**Consistency**

The consistency property ensures that any transaction will bring the database from one valid state to another. Any data written to the database must be valid according to all defined rules, including constraints, cascades, triggers, and any combination thereof. This does not guarantee correctness of the transaction in all ways the application programmer might have wanted (that is the responsibility of application-level code), but merely that any programming errors cannot result in the violation of any defined rules.

**Isolation**

The isolation property ensures that the concurrent execution of transactions results in a system state that would be obtained if transactions were executed sequentially, i.e., one after the other. Providing isolation is the main goal of concurrency control. Depending on the concurrency control method (i.e., if it uses strict - as opposed to relaxed - serializability), the effects of an incomplete transaction might not even be visible to another transaction.

**Durability**

The durability property ensures that once a transaction has been committed, it will remain so, even in the event of power loss, crashes, or errors. In a relational database, for instance, once a group of SQL statements execute, the results need to be stored permanently (even if the database crashes immediately thereafter). To defend against power loss, transactions (or their effects) must be recorded in a non-volatile memory.

## The SQL vs NoSQL(specifically, Document-Oriented database, not including redis, apache cassandra) Holy War

Before we go further, let’s dispel a number of myths …

**MYTH: NoSQL supersedes SQL**

That would be like saying boats were superseded by cars because they’re a newer technology. SQL and NoSQL do the same thing: store data. They take different approaches, which may help or hinder your project. Despite feeling newer and grabbing recent headlines, NoSQL is not a replacement for SQL — it’s an alternative.

**MYTH: NoSQL is better / worse than SQL**

Some projects are better suited to using an SQL database. Some are better suited to NoSQL. Some could use either interchangeably. This article could never be a SitePoint Smackdown, because you cannot apply the same blanket assumptions everywhere.

**MYTH: SQL vs NoSQL is a clear distinction**

This is not necessarily true. Some SQL databases are adopting NoSQL features and vice versa. The choices are likely to become increasingly blurred, and NewSQL hybrid databases could provide some interesting options in the future.

### SQL Tables vs NoSQL Documents

SQL databases provide a store of related data tables. For example, if you run an online book store, book information can be added to a table named book:

| ISBN | title | author | format | price |
|------|-------|--------|--------|-------|
| 9780992461225 | JavaScript: Novice to Ninja | Darren Jones | ebook | 	29.00 |
| 9780994182654 | Jump Start Git | Shaumik Daityari | ebook | 	29.00 |

Every row is a different book record. The design is rigid; you cannot use the same table to store different information or insert a string where a number is expected.

NoSQL databases store JSON-like field-value pair documents, e.g.

```javascript
{
  ISBN: 9780992461225,
  title: "JavaScript: Novice to Ninja",
  author: "Darren Jones",
  format: "ebook",
  price: 29.00
}
```

Similar documents can be stored in a collection, which is analogous to an SQL table. However, you can store any data you like in any document; the NoSQL database won’t complain. For example:

```javascript
{
  ISBN: 9780992461225,
  title: "JavaScript: Novice to Ninja",
  author: "Darren Jones",
  year: 2014,
  format: "ebook",
  price: 29.00,
  description: "Learn JavaScript from scratch!",
  rating: "5/5",
  review: [
    { name: "A Reader", text: "The best JavaScript book I've ever read." },
    { name: "JS Expert", text: "Recommended to novice and expert developers alike." }
  ]
}
```

SQL tables create a strict data template, so it’s difficult to make mistakes. NoSQL is more flexible and forgiving, but being able to store any data anywhere can lead to consistency issues.

### SQL Schema vs NoSQL Schemaless

In an SQL database, it’s impossible to add data until you define tables and field types in what’s referred to as a schema. The schema optionally contains other information, such as —

- primary keys — unique identifiers such as the ISBN which apply to a single record
- indexes — commonly queried fields indexed to aid quick searching
- relationships — logical links between data fields
- functionality such as triggers and stored procedures.

Your data schema must be designed and implemented before any business logic can be developed to manipulate data. It’s possible to make updates later, but large changes can be complicated.

In a NoSQL database, data can be added anywhere, at any time. There’s no need to specify a document design or even a collection up-front. For example, in MongoDB the following statement will create a new document in a new book collection if it’s not been previously created:

```javascript
db.book.insert(
  ISBN: 9780994182654,
  title: "Jump Start Git",
  author: "Shaumik Daityari",
  format: "ebook",
  price: 29.00
);
```

(MongoDB will automatically add a unique _id value to each document in a collection. You may still want to define indexes, but that can be done later if necessary.)

A NoSQL database may be more suited to projects where the initial data requirements are difficult to ascertain. That said, don’t mistake difficulty for laziness: neglecting to design a good data store at project commencement will lead to problems later.

### SQL Normalization vs NoSQL Denormalization

Presume we want to add publisher information to our book store database. A single publisher could offer more than one title so, in an SQL database, we create a new publisher table:

| id | name | country | email |
|------|-------|--------|--------|
| SP001 | SitePoint | Australia | feedback@sitepoint.com |

We can then add a publisher_id field to our book table, which references records by publisher.id:

| ISBN | title | author | format | price | publisher_id |
|------|-------|--------|--------|-------|--------------|
| 9780992461225 | JavaScript: Novice to Ninja | Darren Jones | ebook | 	29.00 | SP001 |
| 9780994182654 | Jump Start Git | Shaumik Daityari | ebook | 	29.00 | SP001 |

This minimizes data redundancy; we’re not repeating the publisher information for every book — only the reference to it. This technique is known as normalization, and has practical benefits. We can update a single publisher without changing book data.

We can use normalization techniques in NoSQL. Documents in the book collection —

```javascript
{
  ISBN: 9780992461225,
  title: "JavaScript: Novice to Ninja",
  author: "Darren Jones",
  format: "ebook",
  price: 29.00,
  publisher_id: "SP001"
}
```

reference a document in a publisher collection:

```javascript
{
  id: "SP001"
  name: "SitePoint",
  country: "Australia",
  email: "feedback@sitepoint.com"
}
```

**However, this is not always practical**, for reasons that will become evident below. We may opt to denormalize our document and repeat publisher information for every book:

```javascript
{
  ISBN: 9780992461225,
  title: "JavaScript: Novice to Ninja",
  author: "Darren Jones",
  format: "ebook",
  price: 29.00,
  publisher: {
    name: "SitePoint",
    country: "Australia",
    email: "feedback@sitepoint.com"
  }
}
```

This leads to faster queries, but updating the publisher information in multiple records will be significantly slower.

### SQL Relational JOIN vs NoSQL

SQL queries offer a powerful JOIN clause. We can obtain related data in multiple tables using a single SQL statement. For example:

```sql
SELECT book.title, book.author, publisher.name
FROM book
LEFT JOIN book.publisher_id ON publisher.id;
```

This returns all book titles, authors and associated publisher names (presuming one has been set).

NoSQL has no equivalent of JOIN, and this can shock those with SQL experience. If we used normalized collections as described above, we would need to fetch all book documents, retrieve all associated publisher documents, and manually link the two in our program logic. This is one reason denormalization is often essential.

### SQL vs NoSQL Data Integrity

Most SQL databases allow you to enforce data integrity rules using foreign key constraints (unless you’re still using the older, defunct MyISAM storage engine in MySQL). Our book store could —

ensure all books have a valid publisher_id code that matches one entry in the publisher table, and
not permit publishers to be removed if one or more books are assigned to them.
The schema enforces these rules for the database to follow. It’s impossible for developers or users to add, edit or remove records, which could result in invalid data or orphan records.

The same data integrity options are not available in NoSQL databases; you can store what you want regardless of any other documents. Ideally, a single document will be the sole source of all information about an item.

### SQL vs NoSQL Transactions (refer to [Multiversion concurrency control](#Multiversion concurrency control))

In SQL databases, two or more updates can be executed in a transaction — an all-or-nothing wrapper that guarantees success or failure. For example, presume our book store contained order and stock tables. When a book is ordered, we add a record to the order table and decrement the stock count in the stock table. If we execute those two updates individually, one could succeed and the other fail — thus leaving our figures out of sync. Placing the same updates within a transaction ensures either both succeed or both fail.

In a NoSQL database, modification of a single document is atomic. In other words, if you’re updating three values within a document, either all three are updated successfully or it remains unchanged. However, there’s no transaction equivalent for updates to multiple documents. There are transaction-like options, but, at the time of writing, these must be manually processed in your code.

### SQL vs NoSQL CRUD Syntax

Creating, reading updating and deleting data is the basis of all database systems. In essence —

SQL is a lightweight declarative language. It’s deceptively powerful, and has become an international standard, although most systems implement subtly different syntaxes.
NoSQL databases use JavaScripty-looking queries with JSON-like arguments! Basic operations are simple, but nested JSON can become increasingly convoluted for more complex queries.

A quick comparison:

![crud1](https://github.com/dongliang3571/Database-Notes/blob/master/screenshots/crud-1.png?raw=true)
![crud2](https://github.com/dongliang3571/Database-Notes/blob/master/screenshots/curd-2.png?raw=true)

### SQL vs NoSQL Performance

Perhaps the most controversial comparison, NoSQL is regularly quoted as being faster than SQL. This isn’t surprising; NoSQL’s simpler denormalized store allows you to retrieve all information about a specific item in a single request. There’s no need for related JOINs or complex SQL queries.

That said, your project design and data requirements will have most impact. A well-designed SQL database will almost certainly perform better than a badly designed NoSQL equivalent and vice versa.

### SQL vs NoSQL Scaling

As your data grows, you may find it necessary to distribute the load among multiple servers. This can be tricky for SQL-based systems. How do you allocate related data? Clustering is possibly the simplest option; multiple servers access the same central store — but even this has challenges.

NoSQL’s simpler data models can make the process easier, and many have been built with scaling functionality from the start. That is a generalization, so seek expert advice if you encounter this situation.

### SQL vs NoSQL Practicalities

Finally, let’s consider security and system problems. The most popular NoSQL databases have been around a few years; they are more likely to exhibit issues than more mature SQL products. Many problems have been reported, but most boil down to a single issue: knowledge.

Developers and sysadmins have less experience with newer database systems, so mistakes are made. Opting for NoSQL because it feels fresher, or because you want to avoid schema design inevitably, leads to problems later.

### SQL vs NoSQL Summary

SQL and NoSQL databases do the same thing in different ways. It’s possible choose one option and switch to another later, but a little planning can save time and money.

**Projects where SQL is ideal:**

- logical related discrete data requirements which can be identified up-front
- data integrity is essential
- standards-based proven technology with good developer experience and support.

**Projects where NoSQL is ideal:**

- unrelated, indeterminate or evolving data requirements
- simpler or looser project objectives, able to start coding immediately
- speed and scalability is imperative.

In the case of our book store, an SQL database appears the most practical option — especially when we introduce ecommerce facilities requiring robust transaction support. In the next article, we’ll discuss further project scenarios, and determine whether an SQL or NoSQL database would be the best solution.

## Different NoSql database

- **Key/Value**: Redis, Tokyo Cabinet, Memcached
- **ColumnFamily**: Cassandra, HBase
- **Document**: MongoDB, CouchDB

The main differences are the data model and the querying capabilities.

**Key-value stores**

The first type is very simple and probably doesn't need any further explanation.

**Data model: more than key-value stores**

Although there is some debate on the correct name for databases such as Cassandra, I'd like to call them **column-family stores**. Although key-value pairs are an essential part of Cassandra, it's not limited to just that. It allows you to nest key-value pairs, so a key could refer to multiple sub-key-value pairs.

You cannot nest key-value pairs indefinitely though. Your limited to three levels (column families) or four levels of nesting (super-column families). In case the term column family doesn't ring a bell, see the WTF is a SuperColumn article, it's a good explanation of Cassandra's data model.

**Document databases**, such as CouchDB and MongoDB store entire documents in the form of JSON objects. You can think of these objects as nested key-value pairs. Unlike Cassandra, you can nest key-value pairs as much as you want. JSON also supports arrays and understands different data types, such as strings, numbers and boolean values.

**Querying**

I believe column-family stores can only be queried by key, or by writing map-reduce functions. You cannot query the values like you would in an SQL database. If your application needs more complex queries, your application will have to create and maintain indexes in order to access the desired data.

Document databases support queries by key and map-reduce functions as well, but also allow you to do basic queries by value, such as "Give me all users with more than 10 posts". Document databases are more flexible in this way.

## Different SQL languages

- SQL is a query language to operate on sets.

It is more or less standardized, and used by almost all relational database management systems: SQL Server, Oracle, MySQL, PostgreSQL, DB2, Informix, etc.

- PL/SQL is a proprietary procedural language used by Oracle
- PL/pgSQL is a procedural language used by PostgreSQL
- TSQL is a proprietary procedural language used by Microsoft in SQL Server.

Procedural languages are designed to extend SQL's abilities while being able to integrate well with SQL. Several features such as local variables and string/data processing are added. These features make the language Turing-complete.

### ACID

Let's first discuss what ACID means. ACID is an acronym **(Atomicity, Consistency, Isolation, Durability)** that defines a set of properties that will guarantee reliability in the world of database transactions. **Atomicity** means that if you have multiple statements in a transaction, all parts of the transaction need to be successful - if any one of them fails, the whole transaction will also fail. **Consistency** means that there is a guarantee that the database will go from one valid state into another when a transaction commits. **Isolation** means that if you execute your transactions concurrently, those transactions are unaware of each other and that they are executed serially. Finally, **durability** means that if a transaction commits (i.e. saves data to the database, does an update or deletes something) those changes are going to persist, even in the case of a system failure. - See more at: https://developer.marklogic.com/blog/how-is-marklogic-different-from-mongodb#sthash.aiILcZAE.dpuf

### Multiversion concurrency control (MCC or MVCC)

Multiversion concurrency control (MCC or MVCC), is a concurrency control method commonly used by database management systems to provide concurrent access to the database and in programming languages to implement transactional memory.[1]

If someone is reading from a database at the same time as someone else is writing to it, it is possible that the reader will see a half-written or inconsistent piece of data. There are several ways of solving this problem, known as concurrency control methods. The simplest way is to make all readers wait until the writer is done, which is known as a lock. This can be very slow, so MVCC takes a different approach: each user connected to the database sees a snapshot of the database at a particular instant in time. Any changes made by a writer will not be seen by other users of the database until the changes have been completed (or, in database terms: until the transaction has been committed.)

When an MVCC database needs to update an item of data, it will not overwrite the old data with new data, but instead marks the old data as obsolete and adds the newer version elsewhere. Thus there are multiple versions stored, but only one is the latest. This allows readers to access the data that was there when they began reading, even if it was modified or deleted part way through by someone else. It also allows the database to avoid the overhead of filling in holes in memory or disk structures but requires (generally) the system to periodically sweep through and delete the old, obsolete data objects. For a document-oriented database it also allows the system to optimize documents by writing entire documents onto contiguous sections of disk—when updated, the entire document can be re-written rather than bits and pieces cut out or maintained in a linked, non-contiguous database structure.

## Postgresql

`$PGDATA` where the data directory are

for Fedora distribution, it locates on, for example, `/var/lib/pgsql/9.5`
for Ubuntu distribution, it locates on, for example, `/var/lib/postgresql/9.5/main`

or you can do

```bash
$ psql -U postgres
postgres=# show data_directory

# output is /var/lib/postgresql/9.5/main
```

or

```bash
$ pg_config

BINDIR = /usr/lib/postgresql/9.5/bin
DOCDIR = /usr/share/doc/postgresql-doc-9.5
HTMLDIR = /usr/share/doc/postgresql-doc-9.5
INCLUDEDIR = /usr/include/postgresql
PKGINCLUDEDIR = /usr/include/postgresql
INCLUDEDIR-SERVER = /usr/include/postgresql/9.5/server
LIBDIR = /usr/lib/x86_64-linux-gnu
PKGLIBDIR = /usr/lib/postgresql/9.5/lib
LOCALEDIR = /usr/share/locale
MANDIR = /usr/share/postgresql/9.5/man
SHAREDIR = /usr/share/postgresql/9.5
SYSCONFDIR = /etc/postgresql-common
```

`$libdir`

for Fedora distribution it locates on, for example, `/usr/pgsql/lib` or `usr/lib/pgsql/lib`
for Ubuntu distribution it locates onm for example, `/usr/lib/postgresql/9.5/lib`

**`pg_dump`**

```
pg_dump -U <username> -W <password> -p <port> -F <format> -f <file_path_to_dump>
```

**`pg_restore`**

## MySql

MySQL Server doesn't support the `SELECT ... INTO TABLE` Sybase SQL extension. Instead, MySQL Server supports the `INSERT INTO ... SELECT` standard SQL syntax, which is basically the same thing.

```sql
INSERT INTO tbl_temp2 (fld_id)
    SELECT tbl_temp1.fld_order_id
    FROM tbl_temp1 WHERE tbl_temp1.fld_order_id > 100;
```

### Change password for a user

MySQL 5.7.6 and later:

```
ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass';
```

MySQL 5.7.5 and earlier:

```
SET PASSWORD FOR 'root'@'localhost' = PASSWORD('MyNewPass');
```
### Auto Commit

By default, MySQL runs with `autocommit` mode enabled. This means that as soon as you execute a statement that updates (modifies) a table, MySQL stores the update on disk to make it permanent. The change cannot be rolled back.

You can disablt that with `SET autocommit=1` explicitly ()

or do it implicitly with `START TRANSACTION`, by starting a transaction it will automatically disable autocommit.

Note that, in **MyISAM**, there is no concept of transaction, so you have to use `SET autocommit=0` to prevent autocommit. InnoDB has transaction, so using `START TRANSACTION` is recommended way to do it.

### CREATE TABLE

```sql
CREATE TABLE tiny_to_long (
    id INT(11) NOT NULL AUTO_INCREMENT PRIMARY KEY,
    tiny_url VARCHAR(255) NOT NULL UNIQUE,
    long_url VARCHAR(3000) NOT NULL
) ENGINE=INNODB AUTO_INCREMENT=1;

-- When using UNIQUE constraint, size of it cannot be too large.
```

`AUTO_INCREMENT` columns start from 1 by default. The automatically generated value can never be lower than 0. If you want different inital value for `AUTO_INCREMENT`, just do `AUTO_INCREMENT=other_value`

`ENGINE = INNODB` is default storage engine for mysql. More options can be `ENGINE = CSV`, `ENGINE = MEMORY`

### Difference between `VARCHAR` and `CHAR`
  - `VARCHAR` is variable-length.
  - `CHAR` is fixed length.
  - If your content is a fixed size, you'll get better performance with `CHAR`.

### Stored program(procedures)

Each stored program contains a body that consists of an SQL statement. This statement may be a compound statement made up of several statements separated by semicolon (;) characters. For example, the following stored procedure has a body made up of a BEGIN ... END block that contains a SET statement and a REPEAT loop that itself contains another SET statement:

```sql
CREATE PROCEDURE dorepeat(p1 INT)
BEGIN
  SET @x = 0;
  REPEAT SET @x = @x + 1; UNTIL @x > p1 END REPEAT;
END;
```

**If you use the mysql client program to define a stored program containing semicolon characters, a problem arises. By default, mysql itself recognizes the semicolon as a statement delimiter, so you must redefine the delimiter temporarily to cause mysql to pass the entire stored program definition to the server.**

`DELIMITER //` changes the statement delimiter from `;` to `//`. This is so you can write `;` in your trigger definition without the MySQL client misinterpreting that as meaning you're done with it.

Note that when changing back, it's `DELIMITER ;`, not `DELIMITER;` as I've seen people try to do.

```sql
mysql> delimiter //

mysql> CREATE PROCEDURE dorepeat(p1 INT)
    -> BEGIN
    ->   SET @x = 0;
    ->   REPEAT SET @x = @x + 1; UNTIL @x > p1 END REPEAT;
    -> END
    -> //
Query OK, 0 rows affected (0.00 sec)

mysql> delimiter ;

mysql> CALL dorepeat(1000);
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT @x;
+------+
| @x   |
+------+
| 1001 |
+------+
1 row in set (0.00 sec)
```

**You can redefine the delimiter to a string other than //, and the delimiter can consist of a single character or multiple characters. You should avoid the use of the backslash (\) character because that is the escape character for MySQL.**

The following is an example of a function that takes a parameter, performs an operation using an SQL function, and returns the result. In this case, it is unnecessary to use delimiter because the function definition contains no internal ; statement delimiters:

```sql
mysql> CREATE FUNCTION hello (s CHAR(20))
mysql> RETURNS CHAR(50) DETERMINISTIC
    -> RETURN CONCAT('Hello, ',s,'!');
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT hello('world');
+----------------+
| hello('world') |
+----------------+
| Hello, world!  |
+----------------+
1 row in set (0.00 sec)
```

Notice that above example use `CREATE FUNCTION` rather than `CREATE PROCEDURE`.

Function must have a `RETURN` type, and the function body must contain a `RETURN` value statement.

You can't mix in stored procedures with ordinary SQL, whilst with stored function you can.
e.g. `SELECT get_foo(myColumn) FROM  mytable` is **NOT** valid if `get_foo()` is a **procedure**, but you can do that if `get_foo()` is a **function**. The price is that functions have more limitations than a procedure.

### Procedure Parameters

**Each parameter is an IN parameter by default.** To specify otherwise for a parameter, use the keyword OUT or INOUT before the parameter name.

**Note:** Specifying a parameter as IN, OUT, or INOUT is valid only for a PROCEDURE. For a FUNCTION, parameters are always regarded as IN parameters.

1. An `IN` parameter passes a value into a procedure. The procedure might modify the value, but the modification is not visible to the caller when the procedure returns. 
2. An `OUT` parameter passes a value from the procedure back to the caller. **Its initial value is `NULL` within the procedure**, and its value is visible to the caller when the procedure returns. 
3. An `INOUT` parameter is initialized by the caller, can be modified by the procedure, and any change made by the procedure is visible to the caller when the procedure returns.

```sql
DELIMITER //
create procedure insert_sales_proc(IN stor_id char(4),IN ordernum varchar(20), IN orderdate varchar(40))
    begin
    insert into sales values(stor_id, ordernum,orderdate);
    end//

create procedure insert_salesdetail_proc (IN stor_id char(4), IN ord_num varchar(20), IN title_id varchar(6), IN qty smallint, IN discount float)
     begin
     insert salesdetail values(stor_id,ord_num,title_id,qty,discount);
     end//

DELIMITER ;
```

The following example shows a simple stored procedure that uses an OUT parameter:
```sql
mysql> delimiter //

mysql> CREATE PROCEDURE simpleproc (OUT param1 INT)
    -> BEGIN
    ->   SELECT COUNT(*) INTO param1 FROM t;
    -> END//
Query OK, 0 rows affected (0.00 sec)

mysql> delimiter ;

mysql> CALL simpleproc(@a);
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT @a;
+------+
| @a   |
+------+
| 3    |
+------+
1 row in set (0.00 sec)
```

### Trigger

A trigger is a named database object that is associated with a table, and that activates when a particular event occurs for the table. The trigger becomes associated with the table named tbl_name, which must refer to a permanent table. You cannot associate a trigger with a TEMPORARY table or a view.

Syntax:

```sql
CREATE
    [DEFINER = { user | CURRENT_USER }]
    TRIGGER trigger_name
    trigger_time trigger_event
    ON tbl_name FOR EACH ROW
    [trigger_order]
    trigger_body

trigger_time: { BEFORE | AFTER }

trigger_event: { INSERT | UPDATE | DELETE }

trigger_order: { FOLLOWS | PRECEDES } other_trigger_name
```

example:

```sql
mysql> CREATE TABLE account (acct_num INT, amount DECIMAL(10,2));
Query OK, 0 rows affected (0.03 sec)

mysql> CREATE TRIGGER ins_sum BEFORE INSERT ON account
    -> FOR EACH ROW SET @sum = @sum + NEW.amount;
Query OK, 0 rows affected (0.06 sec)
```

```sql
CREATE TRIGGER PriceTrig
  AFTER UPDATE of pirce ON sells
  REFERENCING
    OLD ROW AS ooo
    NEW ROW AS nnn
  FOR EACH ROW
```

**Row Level trigger** and **Statement Level Trigger**

- a row level trigger will fire for each and every row that is affected by the DML statement (note this is also true for INSERT statements that are based on a SELECT or are using a multi-row syntax to insert more than one row at a time)
- a statement level trigger will fire once for the whole statement.

Oracle, PostgreSQL and DB2 support both, row level and statement level triggers. **Microsoft SQL Server** only supports statement level triggers and MySQL only supports row level triggers.

### GROUP BY and ORDER BY, WHERE and HAVING
- ORDER BY alters the order in which items are returned.
- GROUP BY will aggregate records by the specified columns which allows you to perform aggregation functions on non-grouped columns (such as SUM, COUNT, AVG, etc).

- WHERE doesn't work on aggregation like `SELECT * FROM TABLE GROUP BY ATTRUBUTE WHERE attr>2`, but it works for regular grouping like 'SELECT * FROM TABLE ORDER BY WHERE SOMETHING'
- HAVING works for `GROUP BY`, for example `SELECT * FROM TABLE GROUP BY ATTRUBUTE HAVING attr>2`

### Copy data from one table into a new table(the new table must not exist already)

```sql
SELECT *
INTO newtable [IN externaldb]
FROM oldtable
WHERE condition;
```

## Joins

**Inner join**

![alt text](https://github.com/dongliang3571/SQL-Notes/blob/master/screenshots/inner_join.png?raw=true "Inner join")

**Left outter join**

![alt text](https://github.com/dongliang3571/SQL-Notes/blob/master/screenshots/left_outter_join.png?raw=true "Left outter join")

**Right outter join**

![alt text](https://github.com/dongliang3571/SQL-Notes/blob/master/screenshots/right_outter_join.png?raw=true "Right outter join")

**Full outter join**

![alt text](https://github.com/dongliang3571/SQL-Notes/blob/master/screenshots/full_outter_join.png?raw=true "Full outter join")

## Group by

The GROUP BY statement is often used with aggregate functions (`COUNT`, `MAX`, `MIN`, `SUM`, `AVG`) to group the result-set by one or more columns.

| CustomerID | CustomerName	| ContactName	| Address	City | PostalCode | Country |
|------------|--------------|-------------|--------------|------------|---------|
| 1 | Alfreds Futterkiste	| Maria Anders | Obere Str. 57 | Berlin	| 12209	| Germany |
| 2 |	Ana Trujillo Emparedados y helados | Ana Trujillo | Avda. de la Constitución 2222	| México D.F.	| 05021	| Mexico |
| 3	| Antonio Moreno Taquería	| Antonio Moreno | Mataderos 2312	| México D.F.	| 05023	| Mexico |
| 4 | Around the Horn	| Thomas Hardy | 120 Hanover Sq.	| London | WA1 1DP |	UK |
| 5	| Berglunds snabbköp | Christina Berglund	| Berguvsvägen 8 | Luleå | S-958 | 22	| Sweden |

after run following sql

```
SELECT COUNT(CustomerID), Country
FROM Customers
GROUP BY Country;
```

You will get:

| COUNT(CustomerID) | Country |
|------------|---------|
| 1 | Germany |
| 2 | Mexico |
| 1 |	UK |
| 1	| Sweden |

Note that There are two `CustomerID` with coutry being `Mexico`, that's why the `count` is `2`. What `group by` does is that it divide the table into multiple groups, and apply search on each group and return.

similarly can do `SUM`

```
SELECT SUM(CustomerID), Country
FROM Customers
GROUP BY Country;
```

do `AVG`, `MIN`, `MAX` so forth.

## Microsoft SQL Server

### `master` database
The `master` database records all the system-level information for a SQL Server system. This includes instance-wide metadata such as logon accounts, endpoints, linked servers, and system configuration settings. In SQL Server, system objects are no longer stored in the master database; instead, they are stored in the Resource database. Also, master is the database that records the existence of all other databases and the location of those database files and records the initialization information for SQL Server. Therefore, SQL Server cannot start if the master database is unavailable.

**Note that you will be connected to `master` automatically once you log in to shell of the database server**

### Delimited identifiers

The square brackets [] are used to delimit identifiers. This is necessary if the column name is a reserved keyword or contains special characters such as a space or hyphen.

Are enclosed in double quotation marks (") or brackets ([ ]). Identifiers that comply with the rules for the format of identifiers may or may not be delimited.

```sql
SELECT *
FROM [TableX]         --Delimiter is optional.
WHERE [KeyCol] = 124  --Delimiter is optional.
Identifiers that do not comply with all of the rules for identifiers must be delimited in a Transact-SQL statement.
```
```sql
SELECT *
FROM [My Table]      --Identifier contains a space and uses a reserved keyword.
WHERE [order] = 10   --Identifier is a reserved keyword.
```

### View a list of databases on a MS sql server

```sql
SELECT name, database_id, create_date FROM sys.databases;  
GO 
```

### View a list of tables in a database


**For a particular database**

1. First approach

```sql
SELECT TABLE_NAME FROM <DATABASE_NAME>.INFORMATION_SCHEMA.TABLES WHERE TABLE_TYPE = 'BASE TABLE'
```

2. Second approach

```sql
SELECT TABLE_NAME FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_TYPE = 'BASE TABLE' AND TABLE_CATALOG='dbName' --(for MySql, use: TABLE_SCHEMA='dbName' )
```

### View a list of columns in a database

hard way:

```sql
USE [Database Name]
SELECT COLUMN_NAME,* 
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'YourTableName' AND TABLE_SCHEMA='YourSchemaName'
```

easy way with builtin stored procedure:

```sql
exec sp_columns @table_name='<table_name>'
```

The `INFORMATION_SCHEMA.TABLES` view allows you to get information about all tables and views within a database. By default it will show you this information for every single table and view that is in the database.

| Column name | Data type | Description |
| ------------|-----------|-------------|
| TABLE_CATALOG	| nvarchar(128) | Table qualifier |
| TABLE_SCHEMA | nvarchar(128) | Name of schema that contains the table |
| TABLE_NAME | sysname | Table name |
| TABLE_TYPE | varchar(10) | Table qualifier |

```sql
SELECT * FROM INFORMATION_SCHEMA.TABLES
```

will give you output of:

| TABLE_CATALOG | TABLE_SCHEMA | TABLE_NAME | TABLE_TYPE |
| ------------|-----------|-------------|-------|
| master	      | dbo	          | spt_fallback_db	      | BASE TABLE |
| master	      | dbo	          | spt_fallback_dev	    | BASE TABLE |
| master	      | dbo	          | spt_fallback_usg	    | BASE TABLE | 
| master	      | dbo	          | MSreplication_options	| BASE TABLE |
| master	      | dbo	          | spt_values	          | VIEW |
| master	      | dbo	          | spt_monitor	          | BASE TABLE |

### What is table schema(`TABLE_SCHEMA`)

schema : database : table <-> floor plan : house : room

### Schema(namespace, it's different from table schema)

You cannot have two objects with the same name in a namespace.

In Orace, schema equals to user, and schema is a namespace. You cannot have two objects of the same name in a schema. Triggers, views, tables, indexes - are examples of objects. They all have names which must be unique in the namespace (schema). Column is not an object, so you can have multiple columns with the same name as long as they belong to different tables/views.

In SQL Srever 2005 an up, schema (within a database) is also a namespace, as in Oracle. Schema is a group (a set) of db objects within a database. No two objects (indexes, triggers, tables) can have the same name in a single schema.

In SQL Server 2000 and below, there is no schema. Instead of schema, there is owner of the object (user that owns the object). So, below a user of a database there cannot be two objects with the same name. In version 2005, SQL Server introduced the schema to separate a user (owner) from the group of objects (schema). E.g. You can drop a user, but schema and all objects within it will stay untouched. It's because you didn' drop the schema (group of objects), you just dropped the user (that has some rights on the objects). 

### What is dbo
dbo is the default schema in SQL Server. You can create your own schemas to allow you to better manage your object namespace.

### SQL Server Index Basics

[Good resouce SQl Server Index Basics](https://www.simple-talk.com/sql/learn-sql-server/sql-server-index-basics/)
[Good resouce SQl Server Index: The Basics](http://www.sqlteam.com/article/sql-server-indexes-the-basics)
[Good resouce SQl Server Index (the best one I think)](http://odetocode.com/articles/70.aspx)

One of the most important routes to high performance in a SQL Server database is the index. Indexes speed up the querying process by providing swift access to rows in the data tables, similarly to the way a book’s index helps you find information quickly within that book. In this article, I provide an overview of SQL Server indexes and explain how they’re defined within a database and how they can make the querying process faster. Most of this information applies to indexes in both SQL Server 2005 and 2008; the basic structure has changed little from one version to the next. In fact, much of the information also applies to SQL Server 2000. This does not mean there haven’t been changes. New functionality has been added with each successive version; however, the underlying structures have remained relatively the same. So for the sake of brevity, I stick with 2005 and 2008 and point out where there are differences in those two versions.

**Index Structures**

Indexes are created on columns in tables or views. The index provides a fast way to look up data based on the values within those columns. For example, if you create an index on the primary key and then search for a row of data based on one of the primary key values, SQL Server first finds that value in the index, and then uses the index to quickly locate the entire row of data. Without the index, a table scan would have to be performed in order to locate the row, which can have a significant effect on performance.

You can create indexes on most columns in a table or a view. The exceptions are primarily those columns configured with large object (LOB) data types, such as image, text, and varchar(max). You can also create indexes on XML columns, but those indexes are slightly different from the basic index and are beyond the scope of this article. Instead, I’ll focus on those indexes that are implemented most commonly in a SQL Server database.

An index is made up of a set of pages (index nodes) that are organized in a B-tree structure. This structure is hierarchical in nature, with the root node at the top of the hierarchy and the leaf nodes at the bottom, as shown figure below.

![Image of B-tree](https://github.com/dongliang3571/Database-Notes/blob/master/screenshots/index_B_tree.jpg?raw=true)

When a query is issued against an indexed column, the query engine starts at the root node and navigates down through the intermediate nodes, with each layer of the intermediate level more granular than the one above. The query engine continues down through the index nodes until it reaches the leaf node. For example, if you’re searching for the value 123 in an indexed column, the query engine would first look in the root level to determine which page to reference in the top intermediate level. In this example, the first page points the values 1-100, and the second page, the values 101-200, so the query engine would go to the second page on that level. The query engine would then determine that it must go to the third page at the next intermediate level. From there, the query engine would navigate to the leaf node for value 123. The leaf node will contain either the entire row of data or a pointer to that row, depending on whether the index is clustered or nonclustered.

**Clustered Indexes**

A clustered index stores the actual data rows at the leaf level of the index. Returning to the example above, that would mean that the entire row of data associated with the primary key value of 123 would be stored in that leaf node. An important characteristic of the clustered index is that the indexed values are sorted in either ascending or descending order. As a result, there can be only one clustered index on a table or view. In addition, data in a table is sorted only if a clustered index has been defined on a table.

Note: A table that has a clustered index is referred to as a clustered table. A table that has no clustered index is referred to as a heap.

**Nonclustered Indexes**

Unlike a clustered indexed, the leaf nodes of a nonclustered index contain only the values from the indexed columns and row locators that point to the actual data rows, rather than contain the data rows themselves. This means that the query engine must take an additional step in order to locate the actual data.

A row locator’s structure depends on whether it points to a clustered table or to a heap. If referencing a clustered table, the row locator points to the clustered index, using the value from the clustered index to navigate to the correct data row. If referencing a heap, the row locator points to the actual data row.

Nonclustered indexes cannot be sorted like clustered indexes; however, you can create more than one nonclustered index per table or view. SQL Server 2005 supports up to 249 nonclustered indexes, and SQL Server 2008 support up to 999. This certainly doesn’t mean you should create that many indexes. Indexes can both help and hinder performance, as I explain later in the article.

In addition to being able to create multiple nonclustered indexes on a table or view, you can also add included columns to your index. This means that you can store at the leaf level not only the values from the indexed column, but also the values from non-indexed columns. This strategy allows you to get around some of the limitations on indexes. For example, you can include non-indexed columns in order to exceed the size limit of indexed columns (900 bytes in most cases).

**Index Types**

In addition to an index being clustered or nonclustered, it can be configured in other ways:

Composite index: An index that contains more than one column. In both SQL Server 2005 and 2008, you can include up to 16 columns in an index, as long as the index doesn’t exceed the 900-byte limit. Both clustered and nonclustered indexes can be composite indexes.

Unique Index: An index that ensures the uniqueness of each value in the indexed column. If the index is a composite, the uniqueness is enforced across the columns as a whole, not on the individual columns. For example, if you were to create an index on the FirstName and LastName columns in a table, the names together must be unique, but the individual names can be duplicated.
A unique index is automatically created when you define a primary key or unique constraint:

Primary key: When you define a primary key constraint on one or more columns, SQL Server automatically creates a unique, clustered index if a clustered index does not already exist on the table or view. However, you can override the default behavior and define a unique, nonclustered index on the primary key.

Unique: When you define a unique constraint, SQL Server automatically creates a unique, nonclustered index. You can specify that a unique clustered index be created if a clustered index does not already exist on the table.
Covering index: A type of index that includes all the columns that are needed to process a particular query. For example, your query might retrieve the FirstName and LastName columns from a table, based on a value in the ContactID column. You can create a covering index that includes all three columns.

**Special case that index wont help**

One big point is that an index won't help at all for certain kinds of searches. For example:

`SELECT * FROM [MyTable] WHERE [MyVarcharColumn] LIKE '%' + @SearchText + '%'`

No amount of normal indexing will help that query. It's forever doomed to be slow. That LIKE expression is just not sargable.

Why? You first need to understand how indexes work. They basically take the columns being indexed along with the primary key (record pointer) into a new table. They then sort that table on the indexed column rather than the key. When you do a lookup using the index, it can very quickly find the row(s) you want because this index is sorted to facilitate a more efficient search using algorithms like binary search and others.

Now look at that query again. By placing a wildcard in front of the search text, you've just told the database that you don't know for sure what your column starts with. No amount of sorting will help; you still need to go through the entire table to be sure you find every record that matches the expression. And that means any normal index on the column is worthless for this query.

If you want to search a text column for a search string anywhere in the column, you need to use something a little different: a full-text index.

Now for contrast look at this query:

`SELECT * FROM [MyTable] WHERE [MyVarcharColumn] LIKE @SearchText + '%'`

This will work perfectly fine with an normal index, because you know how you expect the column to start. It can still match up with the sorted values stored in the index, and so we can say that it is sargable.

### What is XQuery?

- XQuery is to XML what SQL is to databases.
- XQuery is designed to query XML data.

```xml
for $x in doc("books.xml")/bookstore/book
where $x/price>30
order by $x/title
return $x/title
```

an example of books.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<bookstore>

<book category="COOKING">
  <title lang="en">Everyday Italian</title>
  <author>Giada De Laurentiis</author>
  <year>2005</year>
  <price>30.00</price>
</book>

<book category="CHILDREN">
  <title lang="en">Harry Potter</title>
  <author>J K. Rowling</author>
  <year>2005</year>
  <price>29.99</price>
</book>

<book category="WEB">
  <title lang="en">XQuery Kick Start</title>
  <author>James McGovern</author>
  <author>Per Bothner</author>
  <author>Kurt Cagle</author>
  <author>James Linn</author>
  <author>Vaidyanathan Nagarajan</author>
  <year>2003</year>
  <price>49.99</price>
</book>

<book category="WEB">
  <title lang="en">Learning XML</title>
  <author>Erik T. Ray</author>
  <year>2003</year>
  <price>39.95</price>
</book>

</bookstore>
```

### XQuery (SQL Server)

Transact-SQL supports **a subset of the XQuery language** that is used for querying the xml data type. XQuery is a language that can query structured or semi-structured XML data. With the xml data type support provided in the Database Engine, documents can be stored in a database and then queried by using XQuery.

### XPath
XPath (XML Path Language) is a query language for selecting nodes from an XML document, which is **subset of XQuery**

### XQuery abd XPath

XQuery is based on the existing XPath query language, with support added for better iteration, better sorting results, and the ability to construct the necessary XML. XQuery operates on the XQuery Data Model. This is an abstraction of XML documents, and the XQuery results that can be typed or untyped. The type information is based on the types provided by the W3C XML Schema language. If no typing information is available, XQuery handles the data as untyped. This is similar to how XPath version 1.0 handles XML.

To query an XML instance stored in a variable or column of xml type, you use the xml Data Type Methods. For example, you can declare a variable of xml type and query it by using the query() method of the xml data type.

```sql
DECLARE @x xml  -- declare variable @x with data type xml
SET @x = '<ROOT><a>111</a></ROOT>'  -- set @x to xml some data
SELECT @x.query('/ROOT/a') -- using query() method to retrive node <a> which is the direct child of <ROOT>
-- output will be <a>111</a>
```

In the following example, the query is specified against the Instructions column of xml type in ProductModel table in the AdventureWorks database.

```sql
SELECT Instructions.query('declare namespace AWMI="http://schemas.microsoft.com/sqlserver/2004/07/adventure-works/ProductModelManuInstructions";           
    /AWMI:root/AWMI:Location[@LocationID=10]  
') as Result   
FROM  Production.ProductModel  
WHERE ProductModelID=7  
```
## Xquey methods

### `query()`

Specifies an XQuery against an instance of the xml data type. The result is of xml type. The method returns an instance of untyped XML

**Syntax**

```sql
query ('XQuery')  

-- Arguments 'XQuery'
-- Is a string , an XQuery expression, that queries for XML nodes such as elements, attributes, in an XML instance.
```

**Example**

The following example declares a variable @myDoc of xml type and assigns an XML instance to it. The query() method is then used to specify an XQuery against the document.

The query retrieves the `<Features>` child element of the `<ProductDescription>` element:

```sql
declare @myDoc xml  
set @myDoc = '<Root>  
<ProductDescription ProductID="1" ProductName="Road Bike">  
<Features>  
  <Warranty>1 year parts and labor</Warranty>  
  <Maintenance>3 year parts and labor extended maintenance is available</Maintenance>  
</Features>  
</ProductDescription>  
</Root>'  
SELECT @myDoc.query('/Root/ProductDescription/Features') 
```

**This is the result:**

```sql
<Features>  
  <Warranty>1 year parts and labor</Warranty>  
  <Maintenance>3 year parts and labor extended maintenance is available</Maintenance>  
</Features>    
```

### `value()`

Performs an XQuery against the XML and returns a value of SQL type. This method returns a scalar value.
You typically use this method to extract a value from an XML instance stored in an xml type column, parameter, or variable. In this way, you can specify SELECT queries that combine or compare XML data with data in non-XML columns.

**Syntax**
```sql
value (XQuery, SQLType)  
-- Arguments
-- 'XQuery'
-- Is the XQuery expression, a string literal, that retrieves data inside the XML instance. The XQuery must return at most one value. -- -- Otherwise, an error is returned.
-- 'SQLType'
-- Is the preferred SQL type, a string literal, to be returned. The return type of this method matches the SQLType parameter. SQLType ---- cannot be an xml data type, a common language runtime (CLR) user-defined type, image, text, ntext, or sql_variant data type. SQLType -- can be an SQL, user-defined data type.
```

**Example**

**A. Using the value() method against an xml type variable**

In the following example, an XML instance is stored in a variable of xml type. The value() method retrieves the ProductID attribute value from the XML. The value is then assigned to an int variable.

```sql
DECLARE @myDoc xml  
DECLARE @ProdID int  
SET @myDoc = '<Root>  
<ProductDescription ProductID="1" ProductName="Road Bike">  
<Features>  
  <Warranty>1 year parts and labor</Warranty>  
  <Maintenance>3 year parts and labor extended maintenance is available</Maintenance>  
</Features>  
</ProductDescription>  
</Root>'  

SET @ProdID =  @myDoc.value('(/Root/ProductDescription/@ProductID)[1]', 'int' )  
SELECT @ProdID  
```

**This is the result:**

Value 1 is returned as a result.

**B. Using the value() method to retrieve a value from an xml type column**

The following query is specified against an xml type column (CatalogDescription) in the AdventureWorks database. The query retrieves ProductModelID attribute values from each XML instance stored in the column.

```sql
SELECT CatalogDescription.value('             
    declare namespace PD="http://schemas.microsoft.com/sqlserver/2004/07/adventure-works/ProductModelDescription";             
       (/PD:ProductDescription/@ProductModelID)[1]', 'int') AS Result             
FROM Production.ProductModel             
WHERE CatalogDescription IS NOT NULL             
ORDER BY Result desc             
```

Note the following from the previous query:

  - The namespace keyword is used to define a namespace prefix.
  - Per static typing requirements, [1] is added at the end of the path expression in the value() method to explicitly indicate that the path expression returns a singleton.
  
This is the partial result:

```
35
34
...
```

**why need [1]**

The type infers a higher cardinality than what the data actually contains. This occurs frequently, because the xml data type can contain more than one top-level element, and an XML schema collection cannot constrain this. In order to reduce the static type and guarantee that there is indeed at most one value being passed, you should use the positional predicate [1]. For example, to add 1 to the value of the attribute c of the element b under the top-level a element, you must write (/a/b/@c)[1]+1. Additionally, the DOCUMENT keyword can be used together with an XML schema collection.

**Note that, basically it's index to an array, but it starts from index 1 rather than 0**

```sql
declare @x xml  
set @x='<ManuInstructions ProductModelID="1" ProductModelName="SomeBike" >  
<Location LocationID="L1" >  
  <Step>Manu step 1 at Loc 1</Step>  
  <Step>Manu step 2 at Loc 1</Step>  
  <Step>Manu step 3 at Loc 1</Step>  
</Location>  
<Location LocationID="L2" >  
  <Step>Manu step 1 at Loc 2</Step>  
  <Step>Manu step 2 at Loc 2</Step>  
  <Step>Manu step 3 at Loc 2</Step>  
</Location>  
</ManuInstructions>'  
SELECT @x.query('  
   for $step in /ManuInstructions/Location[1]/Step  
   return string($step)  
')  
```

This is the result:

```sql
Manu step 1 at Loc 1 Manu step 2 at Loc 1 Manu step 3 at Loc 1 
```

But if you have same query but with `[2]`

```sql
declare @x xml  
set @x='<ManuInstructions ProductModelID="1" ProductModelName="SomeBike" >  
<Location LocationID="L1" >  
  <Step>Manu step 1 at Loc 1</Step>  
  <Step>Manu step 2 at Loc 1</Step>  
  <Step>Manu step 3 at Loc 1</Step>  
</Location>  
<Location LocationID="L2" >  
  <Step>Manu step 1 at Loc 2</Step>  
  <Step>Manu step 2 at Loc 2</Step>  
  <Step>Manu step 3 at Loc 2</Step>  
</Location>  
</ManuInstructions>'  
SELECT @x.query('  
   for $step in /ManuInstructions/Location[2]/Step  -- change from Location[1] to Location[2]
   return string($step)  
')  
```

and result is:

```
Manu step 1 at Loc 2 Manu step 2 at Loc 2 Manu step 3 at Loc 2
```

Another example:

```sql
declare @x xml;
declare @f bit;
set @x='<ManuInstructions ProductModelID="1" ProductModelName="SomeBike" >  
<Location LocationID="L1" >  
  <Step>Manu step 1 at Loc 1</Step>  
  <Step>Manu step 2 at Loc 1</Step>  
  <Step>Manu step 3 at Loc 1</Step>  
</Location>  
<Location LocationID="L2" >  
  <Step>Manu step 1 at Loc 2</Step>  
  <Step>Manu step 2 at Loc 2</Step>  
  <Step>Manu step 3 at Loc 2</Step>  
</Location>  
</ManuInstructions>';

select @x.query('/ManuInstructions/Location/Step[1]');
```

And result is:

```sql
<Step>Manu step 1 at Loc 1</Step><Step>Manu step 1 at Loc 2</Step>
```

Another example:

```sql
declare @x xml;
declare @f bit;
set @x='<ManuInstructions ProductModelID="1" ProductModelName="SomeBike" >  
<Location LocationID="L1" >  
  <Step>Manu step 1 at Loc 1</Step>  
  <Step>Manu step 2 at Loc 1</Step>  
  <Step>Manu step 3 at Loc 1</Step>  
</Location>  
<Location LocationID="L2" >  
  <Step>Manu step 1 at Loc 2</Step>  
  <Step>Manu step 2 at Loc 2</Step>  
  <Step>Manu step 3 at Loc 2</Step>  
</Location>  
</ManuInstructions>';

select @x.query('/ManuInstructions/Location[1]/Step[1]');
```

Result is :

```
<Step>Manu step 1 at Loc 1</Step>
```

**Summary**

`@x.query('/ManuInstructions/Location/Step[1]')` is iterate through all `<Location>` elements and return the first `<Step>` element in each `<Location>`, `Location[1]` takes `<Location LocationID="L1">`, and `Location[2]` takes `<Location LocationID="L2">`.


**C. Using the value() and exist() methods to retrieve values from an xml type column**

The following example shows using both the value() method and the exist() method of the xml data type. The value() method is used to retrieve ProductModelID attribute values from the XML. The exist() method in the WHERE clause is used to filter the rows from the table.

The query retrieves product model IDs from XML instances that include warranty information (the `<Warranty>` element) as one of the features. The condition in the WHERE clause uses the exist() method to retrieve only the rows satisfying this condition.

```sql
SELECT CatalogDescription.value('  
     declare namespace PD="http://schemas.microsoft.com/sqlserver/2004/07/adventure-works/ProductModelDescription";  
           (/PD:ProductDescription/@ProductModelID)[1] ', 'int') as Result  
FROM  Production.ProductModel  
WHERE CatalogDescription.exist('  
     declare namespace PD="http://schemas.microsoft.com/sqlserver/2004/07/adventure-works/ProductModelDescription";  
     declare namespace wm="http://schemas.microsoft.com/sqlserver/2004/07/adventure-works/ProductModelWarrAndMain";  

     /PD:ProductDescription/PD:Features/wm:Warranty ') = 1  
```

Note the following from the previous query:

  - The CatalogDescription column is a typed XML column. This means that it has a schema collection associated with it. In the XQuery Prolog, the namespace declaration is used to define the prefix that is used later in the query body.
  - If the `exist()` method returns a 1 (True), it indicates that the XML instance includes the `<Warranty>` child element as one of the features.
  - The `value()` method in the SELECT clause then retrieves the ProductModelID attribute values as integers.
  
This is the partial result:

```sql
Result       
-----------  
19           
23           
...  
```

**D. Using the `exist()` method instead of the `value()` method**

For performance reasons, instead of using the `value()` method in a predicate to compare with a relational value, use `exist()` with `sql:column()`. For example:

```sql
CREATE TABLE T (c1 int, c2 varchar(10), c3 xml)  
GO  

SELECT c1, c2, c3   
FROM T  
WHERE c3.value( '/root[1]/@a', 'integer') = c1  
GO  
```

This can be written in the following way:

```sql
SELECT c1, c2, c3   
FROM T  
WHERE c3.exist( '/root[@a=sql:column("c1")]') = 1  
GO  
```

### `exist()`

Returns a bit that represents one of the following conditions:
  - 1, representing True, if the XQuery expression in a query returns a nonempty result. That is, it returns at least one XML node.
  - 0, representing False, if it returns an empty result.
  - NULL if the xml data type instance against which the query was executed contains NULL.

**Syntax**

```
exist (XQuery)

-- Arguments
-- 'XQuery'
-- Is an XQuery expression, a string literal.
```

**Examples**

In the following example, @x is an xml type variable (untyped xml) and @f is an integer type variable that stores the value returned by the exist() method. The exist() method returns True (1) if the date value stored in the XML instance is 2002-01-01.

```sql
declare @x xml;  
declare @f bit;  
set @x = '<root Somedate = "2002-01-01Z"/>';  
set @f = @x.exist('/root[(@Somedate cast as xs:date?) eq xs:date("2002-01-01Z")]');  
select @f;  
```

In comparing dates in the exist() method, note the following:

  - The code `cast as xs:date?` is used to cast the value to xs:date type for purposes of comparison.
  - The value of the @Somedate attribute is untyped. In comparing this value, it is implicitly cast to the type on the right side of the comparison, the xs:date type.
  - Instead of cast as xs:date(), you can use the xs:date() constructor function.
  
**Anothen examlpe**

```sql
declare @x xml;
declare @f bit;
set @x='<ManuInstructions ProductModelID="1" ProductModelName="SomeBike" >  
<Location LocationID="L1" >  
  <Step>Manu step 1 at Loc 1</Step>  
  <Step>Manu step 2 at Loc 1</Step>  
  <Step>Manu step 3 at Loc 1</Step>  
</Location>  
<Location LocationID="L2" >  
  <Step>Manu step 1 at Loc 2</Step>  
  <Step>Manu step 2 at Loc 2</Step>  
  <Step>Manu step 3 at Loc 2</Step>  
</Location>  
</ManuInstructions>';

set @f = @x.exist('/ManuInstructions/Location[@LocationID="L3"]');
select @f;
```

This will return 0, because there is no `<Location>` with attribute `LocationID` equal to "L3"

The following example is similar to the previous one, except it has a `<Somedate>` element.

```sql
DECLARE @x xml;  
DECLARE @f bit;  
SET @x = '<Somedate>2002-01-01Z</Somedate>';  
SET @f = @x.exist('/Somedate[(text()[1] cast as xs:date ?) = xs:date("2002-01-01Z") ]')  
SELECT @f;  
```

Note the following from the previous query:

  - The `text()` method returns a text node that contains the untyped value `2002-01-01`. (The XQuery type is xdt:untypedAtomic.) You must explicitly cast this typed value from x to xsd:date, because implicit casting is not supported in this case.

**Example: Specifying the exist() method against a typed xml variable**

The following example illustrates the use of the exist() method against an xml type variable. It is a typed XML variable, because it specifies the schema namespace collection name, ManuInstructionsSchemaCollection.

In the example, a manufacturing instructions document is first assigned to this variable and then the exist() method is used to find whether the document includes a `<Location>` element whose LocationID attribute value is 50.

The exist() method specified against the @x variable returns 1 (True) if the manufacturing instructions document includes a `<Location>` element that has LocationID=50. Otherwise, the method returns 0 (False).

```sql
DECLARE @x xml (Production.ManuInstructionsSchemaCollection);  
SELECT @x=Instructions  
FROM Production.ProductModel  
WHERE ProductModelID=67;  
--SELECT @x  
DECLARE @f int;  
SET @f = @x.exist(' declare namespace AWMI="http://schemas.microsoft.com/sqlserver/2004/07/adventure-works/ProductModelManuInstructions";  
    /AWMI:root/AWMI:Location[@LocationID=50]  
');  
SELECT @f;  
```

**Example: Specifying the exist() method against an xml type column**

The following query retrieves product model IDs whose catalog descriptions do not include the specifications, `<Specifications>` element:

```sql
SELECT ProductModelID, CatalogDescription.query('  
declare namespace pd="http://schemas.microsoft.com/sqlserver/2004/07/adventure-works/ProductModelDescription";  
    <Product   
        ProductModelID= "{ sql:column("ProductModelID") }"   
        />  
') AS Result  
FROM Production.ProductModel  
WHERE CatalogDescription.exist('  
    declare namespace  pd="http://schemas.microsoft.com/sqlserver/2004/07/adventure-works/ProductModelDescription";  
     /pd:ProductDescription[not(pd:Specifications)]'  
    ) = 1;  
```

### `nodes()`

The `nodes()` method is useful when you want to shred an xml data type instance into relational data. It allows you to identify nodes that will be mapped into a new row.

Every xml data type instance has an implicitly provided context node. For the XML instance stored in a column or variable, this is the document node. The document node is the implicit node at the top of every xml data type instance.

The result of the nodes() method is a rowset that contains logical copies of the original XML instances. In these logical copies, the context node of every row instance is set to one of the nodes identified with the query expression, so that subsequent queries can navigate relative to these context nodes.

You can retrieve multiple values from the rowset. For example, you can apply the value() method to the rowset returned by nodes() and retrieve multiple values from the original XML instance. Note that the value() method, when applied to the XML instance, returns only one value.

**Syntax**

```sql
nodes (XQuery) as Table(Column)  

-- Arguments
-- 'XQuery'
-- Is a string literal, an XQuery expression. If the query expression constructs nodes, these constructed nodes are exposed in the resulting rowset. If the query expression results in an empty sequence, the rowset will be empty. If the query expression statically results in a sequence that contains atomic values instead of nodes, a static error is raised.
-- 'Table(Column)'
-- Is the table name and the column name for the resulting rowset.
```

As an example, assume that you have the following table:

```sql
T (ProductModelID int, Instructions xml)  
```

The following manufacturing instructions document is stored in the table. Only a fragment is shown. Note that there are three manufacturing locations in the document.

```sql
<root>  
  <Location LocationID="10"...>  
     <step>...</step>  
     <step>...</step>  
      ...  
  </Location>  
  <Location LocationID="20" ...>  
       ...  
  </Location>  
  <Location LocationID="30" ...>  
       ...  
  </Location>  
</root>  
```

A `nodes()` method invocation with the query expression /root/Location would return a rowset with three rows, each containing a logical copy of the original XML document, and with the context item set to one of the `<Location>` nodes:

```sql
Product  
ModelID      Instructions  
----------------------------------  
1       <root>  
             <Location LocationID="20" ... />  
             <Location LocationID="30" .../></root>  
1      <root><Location LocationID="10" ... />  

             <Location LocationID="30" .../></root>  
1      <root><Location LocationID="10" ... />  
             <Location LocationID="20" ... />  
             </root>  
```

You can then query this rowset by using xml data type methods. The following query extracts the subtree of the context item for each generated row:

```sql
SELECT T2.Loc.query('.')  
FROM   T  
CROSS APPLY Instructions.nodes('/root/Location') as T2(Loc)   
```

This is the result:

```sql
ProductModelID  Instructions  
----------------------------------  
1        <Location LocationID="10" ... />  
1        <Location LocationID="20" ... />  
1        <Location LocationID="30" .../>  
```

Note that the rowset returned has maintained the type information. You can apply xml data type methods, such as query(), value(), exist(), and nodes(), to the result of a nodes() method. However, you cannot apply the modify() method to modify the XML instance.

Also, the context node in the rowset cannot be materialized. That is, you cannot use it in a SELECT statement. However, you can use it in IS NULL and COUNT(*).

### `CROSS APPLY`

To use JOINs (with whatever syntax), both sets you are joining must be **self-sufficient**, i. e. the sets should not depend on each other. You can query both sets without ever knowing the contents on another set.

But for some tasks the sets are not self-sufficient. For instance, let's consider the following query:

```
We have table1 and table2. table1 has a column called rowcount.

For each row from table1 we need to select first rowcount rows from table2, ordered by table2.id
```

We cannot come up with a join condition here. The join condition, should it exist, would involve the row number, which is not present in table2, and there is no way to calculate a row number only from the values of columns of any given row in table2.

That's where the CROSS APPLY can be used.

CROSS APPLY is a Microsoft's extension to SQL, which was originally intended to be used with table-valued functions (TVF's).

The query above would look like this:

```sql
SELECT  *
FROM    table1
CROSS APPLY
(
SELECT  TOP (table1.rowcount) *
FROM    table2
ORDER BY
id
) t2
```

```
For each from table1, select first table1.rowcount rows from table2 ordered by id
```

### Xquery expression

**Primary Expressions**

**Variable References**

A variable reference in XQuery is a QName preceded by a $ sign. This implementation supports only unprefixed variable references. For example, the following query defines the variable $i in the FLWOR expression.

```sql
DECLARE @var XML  
SET @var = '<root>1</root>'  
SELECT @var.query('  
 for $i in /root return data($i)')  
GO  
```

return 1

The following query will not work because a namespace prefix is added to the variable name.

```sql
DECLARE @var XML  
SET @var = '<root>1</root>'  
SELECT @var.query('  
DECLARE namespace x="http://X";  
for $x:i in /root return data($x:i)')  
GO  
```

You can use the sql:variable() extension function to refer to SQL variables, as shown in the following query.

```sql
DECLARE @price money  
SET @price=2500  
DECLARE @x xml  
SET @x = ''  
SELECT @x.query('<value>{sql:variable("@price") }</value>') 
```

This is the result.

<value>2500</value>


**Sequence Expressions**

**Filtering Sequences**

You can filter the sequence returned by an expression by adding a predicate to the expression. For more information, see Path Expressions (XQuery). For example, the following query returns a sequence of three `<a>` element nodes:

```sql
declare @x xml  
set @x = '<root>  
<a attrA="1">111</a>  
<a></a>  
<a></a>  
</root>'  
SELECT @x.query('/root/a')  
```

This is the result:

```sql
<a attrA="1">111</a>  
<a />  
<a />  
```

To retrieve only `<a>` elements that have the attribute attrA, you can specify a filter in the predicate. The resulting sequence will have only one `<a>` element.


```sql
declare @x xml  
set @x = '<root>  
<a attrA="1">111</a>  
<a></a>  
<a></a>  
</root>'  
SELECT @x.query('/root/a[@attrA]')
```

This is the result:

```sql
<a attrA="1">111</a> 
```

For more information about how to specify predicates in a path expression, see Specifying Predicates in a Path Expression Step[https://docs.microsoft.com/en-us/sql/xquery/path-expressions-specifying-predicates].

The following example builds a sequence expression of subtrees and then applies a filter to the sequence.

```sql
declare @x xml  
set @x = '  
<a>  
  <c>C under a</c>  
</a>  
<b>    
   <c>C under b</c>  
</b>  
<c>top level c</c>  
<d></d>  
'  
```

The expression in (/a, /b) constructs a sequence with subtrees /a and /b and from the resulting sequence the expression filters element `<c>`.

```sql
SELECT @x.query('  
  (/a, /b)/c  
')  
```

This is the result:

```sql
<c>C under a</c>  
<c>C under b</c> 
```

The following example applies a predicate filter. The expression finds elements `<a>` and `<b>` that contain element `<c>`.

```sql
declare @x xml  
set @x = '  
<a>  
  <c>C under a</c>  
</a>  
<b>    
   <c>C under b</c>  
</b>  

<c>top level c</c>  
<d></d>  
'  
SELECT @x.query('  
  (/a, /b)[c]  
')  
```

This is the result:

```sql
<a>  
  <c>C under a</c>  
</a>  
<b>  
  <c>C under b</c>  
</b>  
```


## BEGIN...END (Transact-SQL)

## IN (Transact-SQL)

Determines whether a specified value matches any value in a subquery or a list.

**Syntax**

```sql
-- Syntax for SQL Server, Azure SQL Database, Azure SQL Data Warehouse, Parallel Data Warehouse  

test_expression [ NOT ] IN   
    ( subquery | expression [ ,...n ]  
    )   

-- Arguments
-- 'test_expression'
-- Is any valid expression.
-- 'subquery'
-- Is a subquery that has a result set of one column. This column must have the same data type as test_expression.
-- 'expression[ ,... n ]'
-- Is a list of expressions to test for a match. All expressions must be of the same type as test_expression.
```

**Caution**

Any null values returned by subquery or expression that are compared to test_expression using IN or NOT IN return UNKNOWN. Using null values in together with IN or NOT IN can produce unexpected results.

Examples

A. Comparing OR and IN

The following example selects a list of the names of employees who are design engineers, tool designers, or marketing assistants.

```sql
-- Uses AdventureWorks  

SELECT p.FirstName, p.LastName, e.JobTitle  
FROM Person.Person AS p  
JOIN HumanResources.Employee AS e  
    ON p.BusinessEntityID = e.BusinessEntityID  
WHERE e.JobTitle = 'Design Engineer'   
   OR e.JobTitle = 'Tool Designer'   
   OR e.JobTitle = 'Marketing Assistant';  
GO  
```

However, you retrieve the same results by using IN.

```sql
-- Uses AdventureWorks  

SELECT p.FirstName, p.LastName, e.JobTitle  
FROM Person.Person AS p  
JOIN HumanResources.Employee AS e  
    ON p.BusinessEntityID = e.BusinessEntityID  
WHERE e.JobTitle IN ('Design Engineer', 'Tool Designer', 'Marketing Assistant');  
GO 
```

### Prefix `N` in Transact-SQL statment

It's declaring the string as nvarchar data type, rather than varchar

You may have seen Transact-SQL code that passes strings around using an N prefix. This denotes that the subsequent string is in Unicode (the N actually stands for National language character set). Which means that you are passing an NCHAR, NVARCHAR or NTEXT value, as opposed to CHAR, VARCHAR or TEXT.

Prefix Unicode character string constants with the letter N. Without the N prefix, the string is converted to the default code page of the database. This default code page may not recognize certain characters.

### Keyword `GO` and `BEGIN...END`

GO is like the end of a script.

You could have multiple CREATE TABLE statements, separated by GO. It's a way of isolating one part of the script from another, but submitting it all in one block.

BEGIN and END are just like { and } in C/++/#, Java, etc.

They bound a logical block of code. I tend to use BEGIN and END at the start and end of a stored procedure, but it's not strictly necessary there. Where it IS necessary is for loops, and IF statements, etc, where you need more then one step...

```sql
IF EXISTS (SELECT * FROM my_table WHERE id = @id)
BEGIN
   INSERT INTO Log SELECT @id, 'deleted'
   DELETE my_table WHERE id = @id
END
```

## List all relationships and how the link tow tables

```sql
SELECT
    fk.name 'FK Name',
    tp.name 'Parent table',
    cp.name, cp.column_id,
    tr.name 'Refrenced table',
    cr.name, cr.column_id
FROM 
    sys.foreign_keys fk
INNER JOIN 
    sys.tables tp ON fk.parent_object_id = tp.object_id
INNER JOIN 
    sys.tables tr ON fk.referenced_object_id = tr.object_id
INNER JOIN 
    sys.foreign_key_columns fkc ON fkc.constraint_object_id = fk.object_id
INNER JOIN 
    sys.columns cp ON fkc.parent_column_id = cp.column_id AND fkc.parent_object_id = cp.object_id
INNER JOIN 
    sys.columns cr ON fkc.referenced_column_id = cr.column_id AND fkc.referenced_object_id = cr.object_id
ORDER BY
    tp.name, cp.column_id
```

Dump this into Excel, and you can slice and dice - based on the parent table, the referenced table or anything else.

## Cassandra

https://medium.com/jorgeacetozi/cassandra-architecture-and-write-path-anatomy-51e339bcfe0c

**Note: All nodes in cassandra cluster need to have same time otherwise either consistency will be broken or other problems**

In Cassandra, **keyspaces** is corresponding to **database** in sql, **column family** us corresponding to **table** in sql. In **column family** we have **row key**, i.e unique identifier for each row, we can retrieve a row by using this **row key**. For each row, we will have multiple pair of **keys** and **values**, keys are colums, values are corresponding values for each column.

- CQL clients on localhost/127.0.0.1:9042 (as shell for cassandra uses CQL service)
- Binding thrift service to localhost/127.0.0.1:9160 (as python or any other language's interface uses thrift service)

**If cassandra is installed with homebrew**, use this [link](https://gist.github.com/hkhamm/a9a2b45dd749e5d3b3ae)

Assume you install cassandra with homebrew:

In `/usr/local/etc/cassandra/cassandra.yaml`, make `start_rpc` `true` and restart so thrift will start working.

```bash
brew services start cassandra # start the cassandra in the background

brew services stop cassandra # stop cassandra

brew services restart cassandra # stop cassandra and then start it in the background
```

if you want to run cassandra in the foreground and check logs in real time, you do

```bash
cassandra -f
```

### prevent cassandra commit log grow too much

In `cassandra.yaml`, find `commitlog_total_space_in_mb` and give a number

### Some common error

**Error 1**

`java.lang.IllegalStateException: Unknown commitlog version 6`

**Solution**

Go to `/usr/local/var/lib/cassandra/commitlog`, delete all logs there

**Error 2**

`Detected unreadable sstables /usr/local/var/lib/cassandra/data/gputptest-abcdedf-5040777060fe11e5a8557fcd8340170b-KeyCache-b.db ...... please check NEWS.txt and ensure that you have upgraded through all required intermediate versions, running upgradesstables`

**Solution**

If the data is not important to you, go to `/usr/local/var/lib/cassandra/data/` and delete all `*.db` files.

**Error 3**

`Connection error: ('Unable to connect to any servers', {'127.0.0.1': error(61, "Tried connecting to [('127.0.0.1', 9042)]. Last error: Connection refused")})`

**Solution**

Cassandra server is not launched, just do `cassandra -f` to luanch it in foreground or `brew services start cassandra`

**Error 4**

`Connection error: ('Unable to connect to any servers', {'127.0.0.1': ProtocolError("cql_version '3.4.3' is not supported by remote (w/ native protocol). Supported versions: [u'3.4.2']",)})`

**Solution**

`cqlsh` accepts a argument `--cqlversion` to specify `cql_version`, just do `cqlsh --cqlversion='3.4.2'` will do.

### Cqlsh

`cqlsh` is cassandra shell, you can use cql(cassandra query language) to operate cassandra just as SQL.

Some frequent used query:

 - show all keyspaces
 
  ```sql
  select * from system_schema.keyspaces;
  # or
  describe keyspaces
  ```
  
 - show all column families in a keyspace
 
  ```sql
  select columnfamily_name from system.schema_columnfamilies where keyspace_name = <name>;
  # or
  describe tables
  #or
  describe columnfamilies
  ```
 
 - show all rows in a column families
  ```sql
  select * from <column_family_name>;
  ```

 - convert blob type to string using `blobastext`
  ```sql
  select blobastext(column1) from task_results ;
  ```

**Python client driver**
  - pycassa(deprecated), if encounter error `AttributeError: 'TBufferedTransport' object has no attribute 'trans'`, try to enforcing dependency `thrift` to version of **0.9.3**. [link](https://pycassa.github.io/pycassa/index.html)
  - python-driver, [link](https://datastax.github.io/python-driver/)
  
 **Notice:** when using pycassa with cassandra, the consistency has to match the replicas(replication factor) that you have. i.e. `READ_CONSISTENCY` and `WRITE_CONSISTENCY` and their value `ONE`, `TWO`, `QUORUM` and so on. If they're not match, `Connection 4438698256 (localhost) in pool 4438697552 failed: UnavailableException(_message=None)` will throw.
 
### Consistency

Consistency refers to how up-to-date and synchronized all replicas of a row of Cassandra data are at any given moment.

Reliability of read and write operations depends on the consistency used to verify the operation. Strong consistency can be guaranteed when the following condition is true:

`R + W > N`

where

- R is the consistency level of read operations
- W is the consistency level of write operations
- N is the number of replicas

Eventual consistency occurs if the following condition is true:

`R + W =< N`

where

- R is the consistency level of read operations
- W is the consistency level of write operations
- N is the number of replicas

**Contact point**

https://stackoverflow.com/questions/55520942/i-need-to-pass-all-nodes-to-cassandra-client

Question 1: Why do I need to pass these nodes?

To make initial contact with the cluster. If the connection is made then there is no use with these contact points.

Question 2: Do I need to pass all nodes? Or is one sufficient? (All nodes have the information about all other nodes, right?)

You can pass only one node as contact point but the problem is if that node is down when the driver tries to contact then, it won't be able to connect to cluster. So if you provide another contact point it will try to connect with it even if the first one failed. It would be better if you use your Cassandra seed list as contact points.

Question 3: Does the client choose the best node to connect knowing all nodes? Does the client know what data is stored in each node?

Once the initial connection is made the client driver will have the metadata about the cluster. The client will know what data is stored in each node and also which node can be queried with less latency. you can configure all these using load balancing policies

Refer: https://docs.datastax.com/en/developer/python-driver/3.10/api/cassandra/policies/

Question 4: I'm starting to use cassandra for first time, and I'm using kubernetes for the first time. I deployed a cassandra cluster with 3 cassandra nodes. I deployed another one machine and in this machine I want to connect to cassandra by a Python Cassandra client. Do I need to pass all cassandra IPs to Python Cassandra client? Or is it sufficient to put the cassandra DNS given by Kubernetes?

If the hostname can be resolved then it is always better to use DNS instead of IP. I don't see any disadvantage.

**Read consistency levels**

[consistency levels](http://docs.datastax.com/en/cassandra/3.0/cassandra/dml/dmlConfigConsistency.html#dmlConfigConsistency__dml-config-read-consistency)

### Transaction

https://sqlperformance.com/2018/11/sql-performance/understanding-log-buffer-flushes

## MongoDB

#### Replica Set

One primary node and a few secondary nodes form a replica set

https://docs.mongodb.com/manual/replication/#replication-in-mongodb

#### Sharding

A cluster of `mongos`(router), a cluster of `config` nodes and a cluster of shards, each shard is a replica set

https://docs.mongodb.com/manual/sharding/#sharded-cluster

#### Oplog

https://medium.com/@atharva.inamdar/understanding-mongodb-oplog-249f3996f528

Oplog is just another collection(`oplog.rs`) under `local` database. 

We are able to dump oplogs like we dump any other collections.

**mongodump**

How to dump oplogs
```bash
mongodump --host=localhost:27017 --authenticationDatabase=admin -u=username -p=password -d=local -c=oplog.rs --query='{"op": "i", "ns": "database.collection", "ts": { "$lt": Timestamp(1605837774, 1)}}'
```

**mongorestore**

How to restore from oplogs

```bash
mongorestore --host=localhost:27017 --authenticationDatabase=admin -u=username -p=password --oplogReplay oplog_BSON_from_mongodump
```

#### How to restore deleted mongo documents

https://tarunbatra.com/blog/data/Restore-deleted-MongoDB-documents-using-Oplog/
