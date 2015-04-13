---
title: "Data Type Conversion"
parent: "SQL Functions"
---
Drill supports the following functions for casting and converting data types:

* [CAST](/docs/data-type-conversion#cast)
* [CONVERT_TO and CONVERT_FROM](/docs/data-type-conversion#convert_to-and-convert_from)
* [Other Data Type Conversions](/docs/data-type-conversion#other-data-type-conversions)

## CAST

The CAST function converts an entity, such as an expression that evaluates to a single value, from one type to another.

### Syntax

    cast (<expression> AS <data type>)

*expression*

A combination of one or more values, operators, and SQL functions that evaluate to a value

*data type*

The target data type, such as INTEGER or DATE, to which to cast the expression

### Usage Notes

If the SELECT statement includes a WHERE clause that compares a column of an unknown data type, cast both the value of the column and the comparison value in the WHERE clause. For example:

    SELECT c_row, CAST(c_int AS DECIMAL(28,8)) FROM mydata WHERE CAST(c_int AS DECIMAL(28,8)) > -3.0

Do not use the CAST function for converting binary data types to other types. Although CAST works for converting VARBINARY to VARCHAR, CAST does not work in other cases for converting all binary data. Use CONVERT_TO and CONVERT_FROM for converting to or from binary data. 

Refer to the following tables for information about the data types to use for casting:

* [Supported Data Types for Casting](/docs/supported-data-types-for-casting)
* [Explicit Type Casting Maps](/docs/explicit-type-casting-maps)


### Examples

The following examples show how to cast a string to a number, a number to a string, and one type of number to another.

#### Casting a character string to a number
You cannot cast a character string that includes a decimal point to an INT or BIGINT. For example, if you have "1200.50" in a JSON file, attempting to select and cast the string to an INT fails. As a workaround, cast to a FLOAT or DECIMAL type, and then to an INT. 

The following example shows how to cast a character to a DECIMAL having two decimal places.

    SELECT CAST('1' as DECIMAL(28, 2)) FROM sys.version;
    +------------+
    |   EXPR$0   |
    +------------+
    | 1.00       |
    +------------+

#### Casting a number to a character string
The first example shows Drill casting a number to a VARCHAR having a length of 3 bytes: The result is a 3-character string, 456. Drill supports the CHAR and CHARACTER VARYING alias.

    SELECT CAST(456 as VARCHAR(3)) FROM sys.version;
    +------------+
    |   EXPR$0   |
    +------------+
    | 456        |
    +------------+
    1 row selected (0.08 seconds)

    SELECT CAST(456 as CHAR(3)) FROM sys.version;
    +------------+
    |   EXPR$0   |
    +------------+
    | 456        |
    +------------+
    1 row selected (0.093 seconds)

#### Casting from one type of number to another

Cast an integer to a decimal.

    SELECT CAST(-2147483648 AS DECIMAL(28,8)) FROM sys.version;
    +------------+
    |   EXPR$0   |
    +------------+
    | -2.147483648E9 |
    +------------+
    1 row selected (0.08 seconds)

### Casting Intervals

To cast INTERVAL data use the following syntax:

    CAST (column_name AS INTERVAL)
    CAST (column_name AS INTERVAL DAY)
    CAST (column_name AS INTERVAL YEAR)

For example, a JSON file named intervals.json contains the following objects:

    { "INTERVALYEAR_col":"P1Y", "INTERVALDAY_col":"P1D", "INTERVAL_col":"P1Y1M1DT1H1M" }
    { "INTERVALYEAR_col":"P2Y", "INTERVALDAY_col":"P2D", "INTERVAL_col":"P2Y2M2DT2H2M" }
    { "INTERVALYEAR_col":"P3Y", "INTERVALDAY_col":"P3D", "INTERVAL_col":"P3Y3M3DT3H3M" }

1. Set the storage format to Parquet.

        ALTER SESSION SET `store.format` = 'parquet';

        +------------+------------+
        |     ok     |  summary   |
        +------------+------------+
        | true       | store.format updated. |
        +------------+------------+
        1 row selected (0.037 seconds)

Use a CTAS statement to cast text from a JSON file to year and day intervals and to write the data to a Parquet table:

    CREATE TABLE dfs.tmp.parquet_intervals AS 
    (SELECT CAST( INTERVALYEAR_col as interval year) INTERVALYEAR_col, 
            CAST( INTERVALDAY_col as interval day) INTERVALDAY_col 
    FROM dfs.`/Users/drill/intervals.json`);

<!-- Text and include output -->

## CONVERT_TO and CONVERT_FROM

The CONVERT_TO and CONVERT_FROM functions encode and decode
data to and from another data type.

## Syntax  

    CONVERT_TO (column, type)

    CONVERT_FROM(column, type)

*column* is the name of a column Drill reads.

*type* is one of the data types listed in the [CONVERT_TO/FROM Data Types](/docs/data-types#convert_to-and-convert_from-data-types) table.


### Usage Notes

You can use the CONVERT_TO and CONVERT_FROM functions to encode and decode data that is binary or complex. For example, HBase stores
data as encoded VARBINARY data. To read HBase data in Drill, convert every column of an HBase table *from* binary to an SQL data type while selecting the data. To write HBase or Parquet binary data, convert SQL data *to* binary data and store the data in an HBase or Parquet while creating a table as a selection (CTAS).

Do not use the CAST function for converting binary data types to other types. Although CAST works for converting VARBINARY to VARCHAR, CAST does not work in some other binary conversion cases. CONVERT functions work for binary conversions and are also more efficient to use than CAST.

## Usage Notes
Use the CONVERT_TO function to change the data type to binary when sending data back to a binary data source, such as HBase, MapR, and Parquet, from a Drill query. CONVERT_TO also converts an SQL data type to complex types, including HBase byte arrays, JSON and Parquet arrays, and maps. CONVERT_FROM converts from complex types, including HBase arrays, JSON and Parquet arrays and maps to an SQL data type. 

### Examples

This example shows how to use the CONVERT_FROM function to convert complex HBase data to a readable type. The example summarizes and continues the ["Query HBase"](/docs/querying-hbase) example. The ["Query HBase"](/docs/querying-hbase) example stores the following data in the students table on the Drill Sandbox:  

    USE maprdb;

    SELECT * FROM students;
        
    +------------+------------+------------+
    |  row_key   |  account   |  address   |
    +------------+------------+------------+
    | [B@e6d9eb7 | {"name":"QWxpY2U="} | {"state":"Q0E=","street":"MTIzIEJhbGxtZXIgQXY=","zipcode":"MTIzNDU="} |
    | [B@2823a2b4 | {"name":"Qm9i"} | {"state":"Q0E=","street":"MSBJbmZpbml0ZSBMb29w","zipcode":"MTIzNDU="} |
    | [B@3b8eec02 | {"name":"RnJhbms="} | {"state":"Q0E=","street":"NDM1IFdhbGtlciBDdA==","zipcode":"MTIzNDU="} |
    | [B@242895da | {"name":"TWFyeQ=="} | {"state":"Q0E=","street":"NTYgU291dGhlcm4gUGt3eQ==","zipcode":"MTIzNDU="} |
    +------------+------------+------------+
    4 rows selected (1.335 seconds)

You use the CONVERT_FROM function to decode the binary data to render it readable, selecting a data type to use from the [list of supported types](/docs/data-type-conversion/#convert_to-and-convert_from-data-types). JSON supports strings. To convert binary to strings, use the UTF8 type.:

    SELECT CONVERT_FROM(row_key, 'UTF8') AS studentid, 
           CONVERT_FROM(students.account.name, 'UTF8') AS name, 
           CONVERT_FROM(students.address.state, 'UTF8') AS state, 
           CONVERT_FROM(students.address.street, 'UTF8') AS street, 
           CONVERT_FROM(students.address.zipcode, 'UTF8') AS zipcode FROM students;

    +------------+------------+------------+------------+------------+
    | studentid  |    name    |   state    |   street   |  zipcode   |
    +------------+------------+------------+------------+------------+
    | student1   | Alice      | CA         | 123 Ballmer Av | 12345      |
    | student2   | Bob        | CA         | 1 Infinite Loop | 12345      |
    | student3   | Frank      | CA         | 435 Walker Ct | 12345      |
    | student4   | Mary       | CA         | 56 Southern Pkwy | 12345      |
    +------------+------------+------------+------------+------------+
    4 rows selected (0.504 seconds)

This example converts from VARCHAR to a JSON map:

    SELECT CONVERT_FROM('{x:100, y:215.6}' ,'JSON') AS MYCOL FROM sys.version;
    +------------+
    |   MYCOL    |
    +------------+
    | {"x":100,"y":215.6} |
    +------------+
    1 row selected (0.073 seconds)

This example uses a list of BIGINT as input and returns a repeated list of vectors:

    SELECT CONVERT_FROM('[ [1, 2], [3, 4], [5]]' ,'JSON') AS MYCOL1 FROM sys.version;
    +------------+
    |   mycol1   |
    +------------+
    | [[1,2],[3,4],[5]] |
    +------------+
    1 row selected (0.054 seconds)

This example uses a map as input to return a repeated list vector (JSON).

    SELECT CONVERT_FROM('[{a : 100, b: 200}, {a:300, b: 400}]' ,'JSON') AS MYCOL1  FROM sys.version;
    +------------+
    |   MYCOL1   |
    +------------+
    | [{"a":100,"b":200},{"a":300,"b":400}] |
    +------------+
    1 row selected (0.074 seconds)

#### Set up a storage plugin for working with HBase files

This example assumes you are working in the Drill Sandbox. The `maprdb` storage plugin definition is limited, so you modify the `dfs` storage plugin slightly and use that plugin for this example.

1. Copy/paste the `dfs` storage plugin definition to a newly created plugin called myplugin.

2. Change the root location to "/mapr/demo.mapr.com/tables". This change allows you to query tables for reading in the tables directory by workspace.table name. This change allows you to read a table in the `tables` directory. You can write a converted version of the table in the `tmp` directory because the writable property is true.

        {
          "type": "file",
          "enabled": true,
          "connection": "maprfs:///",
          "workspaces": {
            "root": {
              "location": "/mapr/demo.mapr.com/tables",
              "writable": true,
              "defaultInputFormat": null
            },
         
            . . .

            "tmp": {
              "location": "/tmp",
              "writable": true,
              "defaultInputFormat": null
            }

            . . .
         
          "formats": {
            . . .
            "maprdb": {
              "type": "maprdb"
            }
          }
        }

#### Convert the binary HBase students table to JSON data
First, you set the storage format to JSON. Next, you use the CREATE TABLE AS SELECT (CTAS) statement to convert from a selected file of a different format, HBase in this example, to the storage format. You then convert the JSON file to Parquet using a similar procedure. Set the storage format to Parquet, and use a CTAS statement to convert to Parquet from JSON. In each case, you [select UTF8](/docs/data-type-conversion/#convert_to-and-convert_from-data-types) as the file format because the data you are converting from and then to consists of strings.

1. Start Drill on the Drill Sandbox and set the default storage format from Parquet to JSON.

        ALTER SESSION SET `store.format`='json';

2. Use CONVERT_FROM queries to convert the binary data in the HBase students table to JSON, and store the JSON data in a file. You select a data type to use from the supported. JSON supports strings. To convert binary to strings, use the UTF8 type.

        CREATE TABLE tmp.`to_json` AS SELECT 
            CONVERT_FROM(row_key, 'UTF8') AS `studentid`, 
            CONVERT_FROM(students.account.name, 'UTF8') AS name, 
            CONVERT_FROM(students.address.state, 'UTF8') AS state, 
            CONVERT_FROM(students.address.street, 'UTF8') AS street, 
            CONVERT_FROM(students.address.zipcode, 'UTF8') AS zipcode 
        FROM root.`students`;

        +------------+---------------------------+
        |  Fragment  | Number of records written |
        +------------+---------------------------+
        | 0_0        | 4                         |
        +------------+---------------------------+
        1 row selected (0.41 seconds)
4. Navigate to the output. 

        cd /mapr/demo.mapr.com/tmp/to_json
        ls
   Output is:

        0_0_0.json

5. Take a look at the output of `to_json`:

        {
          "studentid" : "student1",
          "name" : "Alice",
          "state" : "CA",
          "street" : "123 Ballmer Av",
          "zipcode" : "12345"
        } {
          "studentid" : "student2",
          "name" : "Bob",
          "state" : "CA",
          "street" : "1 Infinite Loop",
          "zipcode" : "12345"
        } {
          "studentid" : "student3",
          "name" : "Frank",
          "state" : "CA",
          "street" : "435 Walker Ct",
          "zipcode" : "12345"
        } {
          "studentid" : "student4",
          "name" : "Mary",
          "state" : "CA",
          "street" : "56 Southern Pkwy",
          "zipcode" : "12345"
        }

6. Set up Drill to store data in Parquet format.

        ALTER SESSION SET `store.format`='parquet';
        +------------+------------+
        |     ok     |  summary   |
        +------------+------------+
        | true       | store.format updated. |
        +------------+------------+
        1 row selected (0.056 seconds)

7. Use CONVERT_TO to convert the JSON data to a binary format in the Parquet file.

        CREATE TABLE tmp.`json2parquet` AS SELECT 
            CONVERT_TO(studentid, 'UTF8') AS id, 
            CONVERT_TO(name, 'UTF8') AS name, 
            CONVERT_TO(state, 'UTF8') AS state, 
            CONVERT_TO(street, 'UTF8') AS street, 
            CONVERT_TO(zipcode, 'UTF8') AS zip 
        FROM tmp.`to_json`;

        +------------+---------------------------+
        |  Fragment  | Number of records written |
        +------------+---------------------------+
        | 0_0        | 4                         |
        +------------+---------------------------+
        1 row selected (0.414 seconds)
8. Take a look at the binary Parquet output:

        SELECT * FROM tmp.`json2parquet`;
        +------------+------------+------------+------------+------------+
        |     id     |    name    |   state    |   street   |    zip     |
        +------------+------------+------------+------------+------------+
        | [B@224388b2 | [B@7fc36fb0 | [B@77d9cd57 | [B@7c384839 | [B@530dd5e5 |
        | [B@3155d7fc | [B@7ad6fab1 | [B@37e4b978 | [B@94c91f3 | [B@201ed4a |
        | [B@4fb2c078 | [B@607a2f28 | [B@75ae1c93 | [B@79d63340 | [B@5dbeed3d |
        | [B@2fcfec74 | [B@7baccc31 | [B@d91e466 | [B@6529eb7f | [B@232412bc |
        +------------+------------+------------+------------+------------+
        4 rows selected (0.12 seconds)

9. Use CONVERT_FROM to convert the Parquet data to a readable format:

        SELECT CONVERT_FROM(id, 'UTF8') AS id, 
               CONVERT_FROM(name, 'UTF8') AS name, 
               CONVERT_FROM(state, 'UTF8') AS state, 
               CONVERT_FROM(street, 'UTF8') AS address, 
               CONVERT_FROM(zip, 'UTF8') AS zip 
        FROM tmp.`json2parquet2`;

        +------------+------------+------------+------------+------------+
        |     id     |    name    |   state    |  address   |    zip     |
        +------------+------------+------------+------------+------------+
        | student1   | Alice      | CA         | 123 Ballmer Av | 12345      |
        | student2   | Bob        | CA         | 1 Infinite Loop | 12345      |
        | student3   | Frank      | CA         | 435 Walker Ct | 12345      |
        | student4   | Mary       | CA         | 56 Southern Pkwy | 12345      |
        +------------+------------+------------+------------+------------+
        4 rows selected (0.182 seconds)

## Other Data Type Conversions
Drill supports the format for date and time literals shown in the following examples:

* 2008-12-15

* 22:55:55.123...

If you have dates and times in other formats, use a data type conversion functions to perform the following conversions:

* A timestamp, integer, decimal, or double to a character string.
* A character string to a date
* A character string to a number

The following table lists data type formatting functions that you can
use in your Drill queries as described in this section:

**Function**| **Return Type**  
---|---  
TO_CHAR(timestamp, format)| text  
TO_CHAR(int, format)| text  
TO_CHAR(double precision, format)| text  
TO_CHAR(numeric, format)| text  
TO_DATE(text, format)| date  
TO_NUMBER(text, format)| numeric  
TO_TIMESTAMP(text, format)| timestamp
TO_TIMESTAMP(double precision)| timestamp

### Format Specifiers for Numerical Conversions
Use the following format specifiers for converting numbers:
<table >
     <tr >
          <th align=left>Symbol
          <th align=left>Location
          <th align=left>Meaning
     <tr valign=top>
          <td><code>0</code>
          <td>Number
          <td>Digit
     <tr >
          <td><code>#</code>
          <td>Number
          <td>Digit, zero shows as absent
     <tr valign=top>
          <td><code>.</code>
          <td>Number
          <td>Decimal separator or monetary decimal separator
     <tr >
          <td><code>-</code>
          <td>Number
          <td>Minus sign
     <tr valign=top>
          <td><code>,</code>
          <td>Number
          <td>Grouping separator
     <tr >
          <td><code>E</code>
          <td>Number
          <td>Separates mantissa and exponent in scientific notation.
              <em>Need not be quoted in prefix or suffix.</em>
     <tr valign=top>
          <td><code>;</code>
          <td>Subpattern boundary
          <td>Separates positive and negative subpatterns
     <tr >
          <td><code>%</code>
          <td>Prefix or suffix
          <td>Multiply by 100 and show as percentage
     <tr valign=top>
          <td><code>&#92;u2030</code>
          <td>Prefix or suffix
          <td>Multiply by 1000 and show as per mille value
     <tr >
          <td><code>&#164;</code> (<code>&#92;u00A4</code>)
          <td>Prefix or suffix
          <td>Currency sign, replaced by currency symbol.  If
              doubled, replaced by international currency symbol.
              If present in a pattern, the monetary decimal separator
              is used instead of the decimal separator.
     <tr valign=top>
          <td><code>'</code>
          <td>Prefix or suffix
          <td>Used to quote special characters in a prefix or suffix,
              for example, <code>"'#'#"</code> formats 123 to
              <code>"#123"</code>.  To create a single quote
              itself, use two in a row: <code>"# o''clock"</code>.
 </table>

### Format Specifiers for Date/Time Conversions

Use the following format specifiers for date/time conversions:

<table>
  <tr>
    <th>Symbol</th>
    <th>Meaning</th>
    <th>Presentation</th>
    <th>Examples</th>
  </tr>
  <tr>
    <td>G</td>
    <td>era</td>
    <td>text</td>
    <td>AD</td>
  </tr>
  <tr>
    <td>C</td>
    <td>century of era (&gt;=0)</td>
    <td>number</td>
    <td>20</td>
  </tr>
  <tr>
    <td>Y</td>
    <td>year of era (&gt;=0)</td>
    <td>year</td>
    <td>1996</td>
  </tr>
  <tr>
    <td>x</td>
    <td>weekyear</td>
    <td>year</td>
    <td>1996</td>
  </tr>
  <tr>
    <td>w</td>
    <td>week of weekyear</td>
    <td>number</td>
    <td>27</td>
  </tr>
  <tr>
    <td>e</td>
    <td>day of week</td>
    <td>number</td>
    <td>2</td>
  </tr>
  <tr>
    <td>E</td>
    <td>day of week</td>
    <td>text</td>
    <td>Tuesday; Tue</td>
  </tr>
  <tr>
    <td>y</td>
    <td>year</td>
    <td>year</td>
    <td>1996</td>
  </tr>
  <tr>
    <td>D</td>
    <td>day of year</td>
    <td>number</td>
    <td>189</td>
  </tr>
  <tr>
    <td>M</td>
    <td>month of year</td>
    <td>month</td>
    <td>July; Jul; 07</td>
  </tr>
  <tr>
    <td>d</td>
    <td>day of month</td>
    <td>number</td>
    <td>10</td>
  </tr>
  <tr>
    <td>a</td>
    <td>halfday of day</td>
    <td>text</td>
    <td>PM</td>
  </tr>
  <tr>
    <td>K</td>
    <td>hour of halfday (0~11)</td>
    <td>number</td>
    <td>0</td>
  </tr>
  <tr>
    <td>h</td>
    <td>clockhour of halfday (1~12)number</td>
    <td>12</td>
    <td></td>
  </tr>
  <tr>
    <td>H</td>
    <td>hour of day (0~23)</td>
    <td>number</td>
    <td>0</td>
  </tr>
  <tr>
    <td>k</td>
    <td>clockhour of day (1~24)</td>
    <td>number</td>
    <td>24</td>
  </tr>
  <tr>
    <td>m</td>
    <td>minute of hour</td>
    <td>number</td>
    <td>30</td>
  </tr>
  <tr>
    <td>s</td>
    <td>second of minute</td>
    <td>number</td>
    <td>55</td>
  </tr>
  <tr>
    <td>S</td>
    <td>fraction of second</td>
    <td>number</td>
    <td>978</td>
  </tr>
  <tr>
    <td>z</td>
    <td>time zone</td>
    <td>text</td>
    <td>Pacific Standard Time; PST</td>
  </tr>
  <tr>
    <td>Z</td>
    <td>time zone offset/id</td>
    <td>zone</td>
    <td>-0800; -08:00; America/Los_Angeles</td>
  </tr>
  <tr>
    <td>'</td>
    <td>single quotation mark, escape for text delimiter</td>
    <td>literal</td>
    <td></td>
  </tr>
</table>

For more information about specifying a format, refer to one of the following format specifier documents:

* [Java DecimalFormat class](http://docs.oracle.com/javase/7/docs/api/java/text/DecimalFormat.html) format specifiers 
* [Java DateTimeFormat class](http://joda-time.sourceforge.net/apidocs/org/joda/time/format/DateTimeFormat.html) format specifiers

## TO_CHAR

TO_CHAR converts a number, date, time, or timestamp expression to a character string.

### Syntax

    TO_CHAR (expression, 'format');

*expression* is a float, integer, decimal, date, time, or timestamp expression. 

*'format'* is a format specifier enclosed in single quotation marks that sets a pattern for the output formatting. 

### Usage Notes

You can use the ‘z’ option to identify the time zone in TO_TIMESTAMP to make sure the timestamp has the timezone in it, as shown in the TO_TIMESTAMP description.

### Examples

Convert a FLOAT to a character string.

    SELECT TO_CHAR(125.789383, '#,###.###') FROM sys.version;
    +------------+
    |   EXPR$0   |
    +------------+
    | 125.789    |
    +------------+

Convert an integer to a character string.

    SELECT TO_CHAR(125, '#,###.###') FROM sys.version;
    +------------+
    |   EXPR$0   |
    +------------+
    | 125        |
    +------------+
    1 row selected (0.083 seconds)

Convert a date to a character string.

    SELECT TO_CHAR((CAST('2008-2-23' AS DATE)), 'yyyy-MMM-dd') FROM sys.version;
    +------------+
    |   EXPR$0   |
    +------------+
    | 2008-Feb-23 |
    +------------+

Convert a time to a string.

    SELECT TO_CHAR(CAST('12:20:30' AS TIME), 'HH mm ss') FROM sys.version;
    +------------+
    |   EXPR$0   |
    +------------+
    | 12 20 30   |
    +------------+
    1 row selected (0.07 seconds)


Convert a timestamp to a string.

    SELECT TO_CHAR(CAST('2015-2-23 12:00:00' AS TIMESTAMP), 'yyyy MMM dd HH:mm:ss') FROM sys.version;
    +------------+
    |   EXPR$0   |
    +------------+
    | 2015 Feb 23 12:00:00 |
    +------------+
    1 row selected (0.075 seconds)

## TO_DATE
Converts a character string or a UNIX epoch timestamp to a date.

### Syntax

    TO_DATE (expression [, 'format']);

*expression* is a character string enclosed in single quotation marks or a Unix epoch timestamp in milliseconds, not enclosed in single quotation marks. 

*'format'* is a format specifier enclosed in single quotation marks that sets a pattern for the output formatting. Use this option only when the expression is a character string, not a UNIX epoch timestamp. 

### Usage Notes
Specify a format using patterns defined in [Java DateTimeFormat class](http://joda-time.sourceforge.net/apidocs/org/joda/time/format/DateTimeFormat.html). The TO_TIMESTAMP function takes a Unix epoch timestamp. The TO_DATE function takes a UNIX epoch timestamp in milliseconds.

To compare dates in the WHERE clause, use TO_DATE on the value in the date column and in the comparison value. For example:

    SELECT <fields> FROM <plugin> WHERE TO_DATE(<field>, <format>) < TO_DATE (<value>, <format>);

For example:

    SELECT TO_DATE(`date`, 'yyyy-MM-dd') FROM `sample.json`;

    +------------+
    |   EXPR$0   |
    +------------+
    | 2013-07-26 |
    | 2013-05-16 |
    | 2013-06-09 |
    | 2013-07-19 |
    | 2013-07-21 |
    +------------+
    5 rows selected (0.134 seconds)

    SELECT TO_DATE(`date`, 'yyyy-MM-dd') FROM `sample.json` WHERE TO_DATE(`date`, 'yyyy-MM-dd') < TO_DATE('2013-07-20', 'yyyy-MM-dd');
    
    +------------+
    |   EXPR$0   |
    +------------+
    | 2013-05-16 |
    | 2013-06-09 |
    | 2013-07-19 |
    +------------+
    3 rows selected (0.177 seconds)

### Examples
The first example converts a character string to a date. The second example extracts the year to verify that Drill recognizes the date as a date type. 

    SELECT TO_DATE('2015-FEB-23', 'yyyy-MMM-dd') FROM sys.version;
    +------------+
    |   EXPR$0   |
    +------------+
    | 2015-02-23 |
    +------------+
    1 row selected (0.077 seconds)

    SELECT EXTRACT(year from mydate) `extracted year` FROM (SELECT TO_DATE('2015-FEB-23', 'yyyy-MMM-dd') AS mydate FROM sys.version);

    +------------+
    |   myyear   |
    +------------+
    | 2015       |
    +------------+
    1 row selected (0.128 seconds)

The following example converts a UNIX epoch timestamp to a date.

    SELECT TO_DATE(1427849046000) FROM sys.version;
    +------------+
    |   EXPR$0   |
    +------------+
    | 2015-04-01 |
    +------------+
    1 row selected (0.082 seconds)

## TO_NUMBER

TO_NUMBER converts a character string to a formatted number using a format specification.

### Syntax

    TO_NUMBER ('string', 'format');

*'string'* is a character string enclosed in single quotation marks. 

*'format'* is one or more [Java DecimalFormat class](http://docs.oracle.com/javase/7/docs/api/java/text/DecimalFormat.html) specifiers enclosed in single quotation marks that set a pattern for the output formatting.


### Usage Notes
The data type of the output of TO_NUMBER is a numeric. You can use the following [Java DecimalFormat class](http://docs.oracle.com/javase/7/docs/api/java/text/DecimalFormat.html) specifiers to set the output formatting. 

* #  
  Digit placeholder. 

* 0  
  Digit placeholder. If a value has a digit in the position where the zero '0' appears in the format string, that digit appears in the output; otherwise, a '0' appears in that position in the output.

* .  
  Decimal point. Make the first '.' character in the format string the location of the decimal separator in the value; ignore any additional '.' characters.

* ,  
  Comma grouping separator. 

* E
  Exponent. Separates mantissa and exponent in scientific notation. 

### Examples

    SELECT TO_NUMBER('987,966', '######') FROM sys.version;
    +------------+
    |   EXPR$0   |
    +------------+
    | 987.0      |
    +------------+

    SELECT TO_NUMBER('987.966', '###.###') FROM sys.version;
    +------------+
    |   EXPR$0   |
    +------------+
    | 987.966    |
    +------------+
    1 row selected (0.063 seconds)

    SELECT TO_NUMBER('12345', '##0.##E0') FROM sys.version;
    +------------+
    |   EXPR$0   |
    +------------+
    | 12345.0    |
    +------------+
    1 row selected (0.069 seconds)

## TO_TIME
Converts a character string to a time.

### Syntax

    TO_TIME (expression [, 'format']);

*expression* is a character string enclosed in single quotation marks or milliseconds, not enclosed in single quotation marks. 

*'format'* is a format specifier enclosed in single quotation marks that sets a pattern for the output formatting. Use this option only when the expression is a character string, not milliseconds. 

## Usage 
Specify a format using patterns defined in [Java DateTimeFormat class](http://joda-time.sourceforge.net/apidocs/org/joda/time/format/DateTimeFormat.html).

### Examples

    SELECT TO_TIME('12:20:30', 'HH:mm:ss') FROM sys.version;
    +------------+
    |   EXPR$0   |
    +------------+
    | 12:20:30   |
    +------------+
    1 row selected (0.067 seconds)

Convert 828550000 milliseconds (23 hours 55 seconds) to the time.

    SELECT to_time(82855000) FROM sys.version;
    +------------+
    |   EXPR$0   |
    +------------+
    | 23:00:55   |
    +------------+
    1 row selected (0.086 seconds)

## TO_TIMESTAMP

### Syntax

    TO_TIMESTAMP (expression [, 'format']);

*expression* is a character string enclosed in single quotation marks or a UNIX epoch timestamp, not enclosed in single quotation marks. 

*'format'* is a format specifier enclosed in single quotation marks that sets a pattern for the output formatting. Use this option only when the expression is a character string, not a UNIX epoch timestamp. 

### Usage 
Specify a format using patterns defined in [Java DateTimeFormat class](http://joda-time.sourceforge.net/apidocs/org/joda/time/format/DateTimeFormat.html). The TO_TIMESTAMP function takes a Unix epoch timestamp. The TO_DATE function takes a UNIX epoch timestamp in milliseconds.

### Examples

Convert a date to a timestamp. 

    SELECT TO_TIMESTAMP('2008-2-23 12:00:00', 'yyyy-MM-dd HH:mm:ss') FROM sys.version;
    +------------+
    |   EXPR$0   |
    +------------+
    | 2008-02-23 12:00:00.0 |
    +------------+

Convert Unix Epoch time to a timestamp.

    SELECT TO_TIMESTAMP(1427936330) FROM sys.version;
    +------------+
    |   EXPR$0   |
    +------------+
    | 2015-04-01 17:58:50.0 |
    +------------+
    1 row selected (0.094 seconds)

Convert a UTC date to a timestamp offset from the UTC time zone code.

    SELECT TO_TIMESTAMP('2015-03-30 20:49:59.0 UTC', 'YYYY-MM-dd HH:mm:ss.s z') AS Original, 
           TO_CHAR(TO_TIMESTAMP('2015-03-30 20:49:59.0 UTC', 'YYYY-MM-dd HH:mm:ss.s z'), 'z') AS New_TZ 
    FROM sys.version;

    +------------+------------+
    |  Original  |   New_TZ   |
    +------------+------------+
    | 2015-03-30 20:49:00.0 | UTC        |
    +------------+------------+
    1 row selected (0.129 seconds)

## Time Zone Limitation
Currently Drill does not support conversion of a date, time, or timestamp from one time zone to another. The workaround is to configure Drill to use [UTC](http://www.timeanddate.com/time/aboututc.html)-based time, convert your data to UTC timestamps, and perform date/time operation in UTC.  

1. Take a look at the Drill time zone configuration by running the TIMEOFDAY function. This function returns the local date and time with time zone information.

        SELECT TIMEOFDAY() FROM sys.version;

        +------------+
        |   EXPR$0   |
        +------------+
        | 2015-04-02 15:01:31.114 America/Los_Angeles |
        +------------+
        1 row selected (1.199 seconds)

2. Configure the default time zone format in <drill installation directory>/conf/drill-env.sh by adding `-Duser.timezone=UTC` to DRILL_JAVA_OPTS. For example:

        export DRILL_JAVA_OPTS="-Xms1G -Xmx$DRILL_MAX_HEAP -XX:MaxDirectMemorySize=$DRILL_MAX_DIRECT_MEMORY -XX:MaxPermSize=512M -XX:ReservedCodeCacheSize=1G -ea -Duser.timezone=UTC"

3. Restart sqlline.

4. Confirm that Drill is now set to UTC:

        SELECT TIMEOFDAY() FROM sys.version;

        +------------+
        |   EXPR$0   |
        +------------+
        | 2015-04-02 17:05:02.424 UTC |
        +------------+
        1 row selected (1.191 seconds)

You can use the ‘z’ option to identify the time zone in TO_TIMESTAMP to make sure the timestamp has the timezone in it. Also, use the ‘z’ option to identify the time zone in a timestamp using the TO_CHAR function. For example:

    SELECT TO_TIMESTAMP('2015-03-30 20:49:59.0 UTC', 'YYYY-MM-dd HH:mm:ss.s z') AS Original, 
           TO_CHAR(TO_TIMESTAMP('2015-03-30 20:49:59.0 UTC', 'YYYY-MM-dd HH:mm:ss.s z'), 'z') AS TimeZone 
           FROM sys.version;

    +------------+------------+
    |  Original  |  TimeZone  |
    +------------+------------+
    | 2015-03-30 20:49:00.0 | UTC        |
    +------------+------------+
    1 row selected (0.299 seconds)

<!-- DRILL-448 Support timestamp with time zone -->


<!-- Apache Drill    
Apache DrillDRILL-1141
ISNUMERIC should be implemented as a SQL function
SELECT count(columns[0]) as number FROM dfs.`bla` WHERE ISNUMERIC(columns[0])=1
 -->