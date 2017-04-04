# SQL-Notes

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


### GROUP BY and ORDER BY, WHERE and HAVING
- ORDER BY alters the order in which items are returned.
- GROUP BY will aggregate records by the specified columns which allows you to perform aggregation functions on non-grouped columns (such as SUM, COUNT, AVG, etc).

- WHERE doesn't work on aggregation like `SELECT * FROM TABLE GROUP BY ATTRUBUTE WHERE attr>2`, but it works for regular grouping like 'SELECT * FROM TABLE ORDER BY WHERE SOMETHING'
- HAVING works for `GROUP BY`, for example `SELECT * FROM TABLE GROUP BY ATTRUBUTE HAVING attr>2`

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
### Xquey methods

- **query()**

Specifies an XQuery against an instance of the xml data type. The result is of xml type. The method returns an instance of untyped XML

**Syntax**

```sql
query ('XQuery')  

-- Arguments 'XQuery'
-- Is a string , an XQuery expression, that queries for XML nodes such as elements, attributes, in an XML instance.
```

**Example**

The following example declares a variable @myDoc of xml type and assigns an XML instance to it. The query() method is then used to specify an XQuery against the document.

The query retrieves the <Features> child element of the <ProductDescription> element:

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

- **value()**

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

A. Using the value() method against an xml type variable

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

B. Using the value() method to retrieve a value from an xml type column

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
Using the value() and exist() methods to retrieve values from an xml type column

## BEGIN...END (Transact-SQL)
