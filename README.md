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
