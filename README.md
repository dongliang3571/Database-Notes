# SQL-Notes

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

### Difference between `VARCHAR` and `CHAR`
  - `VARCHAR` is variable-length.
  - `CHAR` is fixed length.
  - If your content is a fixed size, you'll get better performance with `CHAR`.


## MySql

MySQL Server doesn't support the `SELECT ... INTO TABLE` Sybase SQL extension. Instead, MySQL Server supports the `INSERT INTO ... SELECT` standard SQL syntax, which is basically the same thing.

```sql
INSERT INTO tbl_temp2 (fld_id)
    SELECT tbl_temp1.fld_order_id
    FROM tbl_temp1 WHERE tbl_temp1.fld_order_id > 100;
```


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



## Microsoft SQL Server

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
SELECT name, database_id, create_date  
FROM sys.databases ;  
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

```sql
USE [Database Name]
SELECT COLUMN_NAME,* 
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'YourTableName' AND TABLE_SCHEMA='YourSchemaName'
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

### What is dbo
dbo is the default schema in SQL Server. You can create your own schemas to allow you to better manage your object namespace.

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
