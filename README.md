# SQL-Notes

### Difference between `VARCHAR` and `CHAR`
  - `VARCHAR` is variable-length.
  - `CHAR` is fixed length.
  - If your content is a fixed size, you'll get better performance with `CHAR`.


## MySql


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
