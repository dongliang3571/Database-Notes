# SQL-Notes

### Difference between `VARCHAR` and `CHAR`
  - `VARCHAR` is variable-length.
  - `CHAR` is fixed length.
  - If your content is a fixed size, you'll get better performance with `CHAR`.


## MySql

`DELIMITER //` changes the statement delimiter from `;` to `//`. This is so you can write `;` in your trigger definition without the MySQL client misinterpreting that as meaning you're done with it.

Note that when changing back, it's `DELIMITER ;`, not `DELIMITER;` as I've seen people try to do.
