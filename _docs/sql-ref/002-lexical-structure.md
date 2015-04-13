---
title: "Lexical Structure"
parent: "SQL Reference"
---

A SQL statement used in Drill can include one or more of the following parts:

* Clause, such as FROM
* Command, such as SELECT 
* Expression, a combination of one or more values, operators, and SQL functions that evaluates to a value. For example, users.firstname is a period expression
* Function, scalar and aggregate, such as sum
* Literal value

  * [Boolean](/docs/lexical-structure#boolean)
  * [Identifier](/docs/lexical-structure#identifier)
  * [Integer](/docs/lexical-structure#integer)
  * [Numeric constant](/docs/lexical-structure#numeric-constant)
  * [String](/docs/lexical-structure#string)

* Operator, such as [NOT] IN, LIKE, and AND
* Predicate, such as a > b in `SELECT * FROM myfile WHERE a > b`.
* [Storage plugin and workspace reference](/docs/lexical-structure#storage-plugin-and-workspace-references)
* Whitespace
* Comment in the following format: 

        /* This is a comment. */

The upper/lowercase sensitivity of the parts differs.

## Case-sensitivity

SQL function and command names are case-insensitive. Storage plugin and workspace names are case-sensitive. Column and table names are case-insensitive unless enclosed in double quotation marks. The double-quotation mark character can be used as an escape character for the double quotation mark.

Although column names are case-insensitive in Drill, the names might be otherwise in the storage format:

* JSON: insensitive
* Hive: insensitive
* Parquet: insensitive
* MapR-DB: case-sensitive
* HBase: case-sensitive

Keywords are case-insensitive. For example, the keywords SELECT and select are equivalent. This document shows keywords in uppercase.

The sys.options table name and values are case-sensitive. The following query works:

    SELECT * FROM sys.options where NAME like '%parquet%';

When using the ALTER command, specify the name in lower case. For example:

    ALTER SESSION  set `store.parquet.compression`='snappy';

## Storage Plugin and Workspace References

Storage plugin and workspace names are case-sensitive. The case of the name used in the query and the name in the storage plugin definition need to match. For example, defining a storage plugin named `dfs` and then referring to the plugin as `DFS` fails, but this query succeeds:

    SELECT * FROM dfs.`/Users/drilluser/ticket_sales.json`;

## Literal Values

This section describes how to construct literals.

### Boolean
Boolean values are true or false and are case-insensitive. Do not enclose the values in quotation marks.

### Date and Time
Format dates using dashes (-) to separate year, month, and day. Format time using colons (:) to separate hours, minutes and seconds. Format timestamps using a date and a time. These literals are shown in the following examples:

* Date: 2008-12-15

* Time: 22:55:55.123...

* Timestamp: 2008-12-15 22:55:55.12345

If you have dates and times in other formats, use a [data type conversion function](/data-type-conversion/#other-data-type-conversions) in your queries.

### Identifier
An identifier is a letter followed by any sequence of letters, digits, or the underscore. For example, names of tables, columns, and aliases are identifiers. Maximum length is 1024 characters. Enclose the following identifiers in back ticks:

* Keywords
* Identifiers that SQL cannot parse. 

For example, enclose the SQL keywords date and time in back ticks when referring to column names, but not when referring to data types:

    CREATE TABLE dfs.tmp.sampleparquet AS 
    (SELECT trans_id, 
    cast(`date` AS date) transdate, 
    cast(`time` AS time) transtime, 
    cast(amount AS double) amountm,
    user_info, marketing_info, trans_info 
    FROM dfs.`/Users/drilluser/sample.json`);

Table and column names are case-insensitive. Use back ticks to enclose names that contain special characters. Special characters are those other than the 52 Latin alphabet characters. For example, space and @ are special characters. 

The following example shows the keyword Year enclosed in back ticks. Because the column alias contains the special space character, also enclose the alias in back ticks, as shown in the following example:

    SELECT extract(year from transdate) AS `Year`, t.user_info.cust_id AS `Customer Number` FROM dfs.tmp.`sampleparquet` t;

    +------------+-----------------+
    |    Year    | Customer Number |
    +------------+-----------------+
    | 2013       | 28              |
    | 2013       | 86623           |
    | 2013       | 11              |
    | 2013       | 666             |
    | 2013       | 999             |
    +------------+-----------------+
    5 rows selected (0.051 seconds)

### Integer
An integer value consists of an optional minus sign, -, followed by one or more digits.

### Numeric constant

Numeric constants include integers, floats, and values in E notation.

* Integers: 0-9 and a minus sign prefix
* Float: a series of one or more decimal digits, followed by a period, ., and one or more digits in decimal places. There is no optional + sign. Leading or trailing zeros are required before and after decimal points. For example, 0.52 and 52.0. 
* E notation: Approximate-value numeric literals in scientific notation consist of a mantissa and exponent. Either or both parts can be signed. For example: 1.2E3, 1.2E-3, -1.2E3, -1.2E-3. Values consist of an optional negative sign (using -), a floating point number, letters e or E, a positive or negative sign (+ or -), and an integer exponent. For example, the following JSON file has data in E notation in two records.

        {"trans_id":0,
         "date":"2013-07-26",
         "time":"04:56:59",
         "amount":-2.6034345E+38,
         "trans_info":{"prod_id":[16],
         "purch_flag":"false"
        }}

        {"trans_id":1,
         "date":"2013-05-16",
         "time":"07:31:54",
         "amount":1.8887898E+38,
         "trans_info":{"prod_id":[],
         "purch_flag":"false"
        }}
  Aggregating the data in Drill produces scientific notation in the output:

        SELECT sum(amount) FROM dfs.`/Users/khahn/Documents/sample2.json`;

        +------------+
        |   EXPR$0   |
        +------------+
        | -7.146447E37 |
        +------------+
        1 row selected (0.044 seconds)

Drill represents invalid values, such as the square root of a negative number, as NaN.

### String
Strings are characters enclosed in single quotation marks. To use a single quotation mark itself (apostrophe) in a string, escape it using a single quotation mark. For example, the value Martha's Vineyard in the SOURCE column in the `vitalstat.json` file contains an apostrophe:

    +------------+
    |   SOURCE   |
    +------------+
    | Martha's Vineyard |
    | Monroe County |
    +------------+
    2 rows selected (0.053 seconds)

To refer to the string Martha's Vineyard in a query, use single quotation marks to enclose the string and escape the apostophe using a single quotation mark:

    SELECT * FROM dfs.`/Users/drilluser/vitalstat.json` t 
    WHERE t.source = 'Martha''s Vineyard';





