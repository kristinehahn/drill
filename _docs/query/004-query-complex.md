---
title: "Querying Complex Data"
parent: "Query Data"
---
Apache Drill queries do not require prior knowledge of the actual data you are
trying to access, regardless of its source system or its schema and data
types. The sweet spot for Apache Drill is a SQL query workload against
"complex data": data made up of various types of records and fields, rather
than data in a recognizable relational form (discrete rows and columns). Drill
is capable of discovering the form of the data when you submit the query.
Nested data formats such as JSON (JavaScript Object Notation) files and
Parquet files are not only _accessible_: Drill provides special operators and
functions that you can use to _drill down _into these files and ask
interesting analytic questions.

These operators and functions include:

  * References to nested data values
  * Access to repeating values in arrays and arrays within arrays (array indexes)

The SQL query developer needs to know the data well enough to write queries
that identify values of interest in the target file. For example, the writer
needs to know what a record consists of, and its data types, in order to
reliably request the right "columns" in the select list. Although these data
values do not manifest themselves as columns in the source file, Drill will
return them in the result set as if they had the predictable form of columns
in a table. Drill also optimizes queries by treating the data as "columnar"
rather than reading and analyzing complete records. (Drill uses similar
parallel execution and optimization capabilities to commercial columnar MPP
databases.)

Given a basic knowledge of the input file, the developer needs to know how to
use the SQL extensions that Drill provides and how to use them to "reach into"
the nested data. The following examples show how to write both simple queries
against JSON files and interesting queries that unpack the nested data. The
examples show how to use the Drill extensions in the context of standard SQL
SELECT statements. For the most part, the extensions use standard JavaScript
notation for referencing data elements in a hierarchy.

### Before You Begin

The examples in this section operate on JSON data files. In order to write
your own queries, you need to be aware of the basic data types in these files:

  * string (all data inside double quotes), such as `"0001"` or `"Cake"`
  * numeric types: integers, decimals, and floats, such as `0.55` or `10`
  * null values
  * boolean values: true, false

Check that you have the following configuration setting for JSON files in the
Drill Web UI (`dfs` storage plugin configuration):

    "json" : {
      "type" : "json"
    }

Click on any of the following links to see examples of complex queries:

  * [Sample Data: Donuts](/confluence/display/DRILL/Sample+Data%3A+Donuts)
  * [Query 1: Selecting Flat Data](/confluence/display/DRILL/Query+1%3A+Selecting+Flat+Data)
  * [Query 2: Using Standard SQL Functions, Clauses, and Joins](/confluence/display/DRILL/Query+2%3A+Using+Standard+SQL+Functions%2C+Clauses%2C+and+Joins)
  * [Query 3: Selecting Nested Data for a Column](/confluence/display/DRILL/Query+3%3A+Selecting+Nested+Data+for+a+Column)
  * [Query 4: Selecting Multiple Columns Within Nested Data](/confluence/display/DRILL/Query+4%3A+Selecting+Multiple+Columns+Within+Nested+Data)