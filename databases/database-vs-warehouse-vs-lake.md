# Databases vs. Data Warehouses vs. Data Lakes

## Normalization

*Normalization* refers to the process of structuring a database in such a way that data redundancy is minimized or obviated altogether.

This is in contrast with *denormalization*, which seeks to increase performance in a normalized database structure by way of introducing reundancy (for example: adding redundant columns to a table to avoid `JOIN` queries).

### Unnormalized

Unnormalized databases can run up against insertion anomalies, update anomalies, and deletion anomalies.

#### Insertion anomalies

Insertion anomalies happen when tables try to represent multiple atomic objects simultaneously.

#### Deletion anomalies

If a table expresses multiple essential features about an object

### 1NF (first normal form)

A database violates 1NF principles if:

* The *row order* in any table matters at all, or expresses any information.
* Columns have multiple data types.
* Tables lack primary keys (thus a table cannot express uniqueness).
* Rows store "repeating" groups of data - for example, a value like `Three oranges, two apples, five bananas`.

A database conforms to 1NF principles if:

* Row ordering never matters.
* Columns have strict data types defined.
* Tables have primary key columns.
* Databases use junction tables to express `1:*` relationships (essentially). 

## OLTP vs. OLAP

*Databases* are generally optimized for *OLTP* ("online transaction processing") operations. OLTP is optimized for `INSERT`, `UPDATE`, and `DELETE` transactions.

*Data warehouses* are generally optimized for *OLAP* ("online analytical processing") operations.

## TODO

Add notes on:

* 2NF - 5NF
* Unnormalized database update anomalies

## Sources
https://www.coursera.org/articles/data-warehouse
https://aws.amazon.com/compare/the-difference-between-olap-and-oltp
https://stackoverflow.com/a/21900244/3474146
https://www.bbc.co.uk/bitesize/guides/zc93tv4/revision/2
