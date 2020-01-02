title: Data Analysis eXpression query language
date: 2019-12-27 10:32
category: data modeling, dax
tags: data, feature engineering
slug: learning-dax
author: Alex Gonzalez
summary: The first part of an overview of the DAX language for data analysis.
header_cover: images/dax-code.png
status: published


## Data Analysis eXpressions (DAX)

DAX is a functional language, that is, execution flows from function calls read from inner to outer context in nested calls.

Here is a short description of DAX syntax conventions:

* Use [Measure] and 'Table'[Column] and never otherwise. Quotes can be omitted for single word table names.
* Use spaces before opening and closing parentheses, and before any operand and operator.
* Space before an in-line argument, and indented line feed for multi-line function calls.

### Functions

#### Aggregations

* `SUM`, `AVERAGE`, `MIN`, `MAX`...
    * `SUM ( Sales[Price] )` works!
    * `SUM ( Sales[Price] + Sales[Quantity] )` does not!

Expressions within aggregators need to be calculated with an **iterator**. These are versions of the function with an X in their name, i.e. `SUMX`. These functions iterate over a table and evaluate the expression for every row, which means the concept of row context comes to life here.
These functions always receive two inputs: the table to iterate and expression to evaluate.

Note that `SUM` is just syntax sugar. `SUM ( Sales[Price] )` is translated in-engine to `SUMX ( Sales, Sales[Price] )`

#### Counting values

* `COUNT` /`COUNTA`: anything but blanks
* `COUNTBLANK`: blanks and empty strings 
* `COUNTROWS`: rows in a table
* `DISTINCTCOUNT`: syntax sugar for `COUNTROWS ( DISTINCT ( 'Table'[Column] ) )` 

#### Relational functions

* `RELATED` / `RELATEDTABLE`: both follow relationships in the data model and return the value of a column or all the rows in relationship with the current one.
    * All relationships must follow the same direction.

#### Table-valued functions

These return full tables which are often used as the input for further calculations.

You can mix table functions, for example: `FILTER ( ALL ( 'Sales' ); Product[Name] = "Laptop" )` puts a filter over the entire table, ignoring any other filter context.

* `FILTER`, `ALL`, `VALUES`, `DISTINCT`, `ADDCOLUMNS`, `SUMMARIZE`...
* `DISTINCT` returns the unique values of a column that are visible in the current context. 
* `VALUES` is the same, respects the filter context and returns the additional blank row if it's visible within the context. This row is created to guarantee referential integrity when values in the fact table can't be found in the dimension table.
* `ALL` returns all the rows of a table while ignoring every filter. It's useful to calculate grand totals and relative percentages.
* `ALLNOBLANKROW` is the same, but excludes the blank row.
* `ALLSELECTED` returns the elements of a table as visible outside the current visualization. That is, it ignores all filters from the current visual but respects all others.
    * Very dangerous function as it is difficult to debug.
* `ADDCOLUMNS` adds one or more columns to a table expression, keeping all existing columns. It's an iterator, so you can access columns of the table and perform expressions and calculations.

> ##### Quick tip!
> 
> I often use `ADDCOLUMNS` to generate a Date table, For example:


```ruby
ADDCOLUMNS (
    ALL ( 'Sales'[sk_Date] ); # the table that will be iterated
    "Month"; # the name of the first column to add
    MONTH ( 'Sales'[sk_Date] ); # the definition of the first column to add
    "Year"; 
    YEAR ( 'Sales'[sk_Date] ); 
    ...
)
```
> This is a very flexible way of generating your own date dimension according to your specific needs.
> 
> You can also generate a dimension table by scanning the columns of your different fact tables and appending them. For example:
> 
```ruby
DISTINCT (
    UNION (
        ALL ( LaptopOrders[OrderID] );
        ALL ( MobileOrders[OrderID] );
    )
)
```

### The IN operator

Verifies whether a value exists in a list of possible values, i.e. `Company[Code] IN {"Microsoft", "Apple", "Dell"}`.
Note that the curly brackets notation generates a temporary table behind the scenes to perform the filtering.

### More functions

* `DIVIDE ( )`: handles safe division by zero and allows for custom alternate values. No more `IF ( ISERROR (...) )`!
  * Also, better performance.
* `SELECTEDVALUE ( )`: provides a shortcut to access the filtered value of a dimension to display in a chart or card.
  * For example, you can create a custom measure that concatenates text with the selected value and display dynamic title in a chart.
  * Prior to this function, the same objective could be accomplished through something along the lines of `IF ( HASONEVALUE ( ... ) )`, which is much longer and error-prone.

### CALCULATE ( )

`CALCULATE` is the main function for DAX expressions, as it is used in many different scenarios to add, change or remove filters. Its syntax is as follows:

```
CALCULATE (
    Expression;
    Filter1;
    ...;
    FilterN 
)
```

`CALCULATE ` calls with multiple filters can have very different behaviors:

```ruby
CALCULATE (
    SUM ( Orders[ItemId] );
    Company[Name] IN {"Microsoft", "Apple"};
    Date[Year] = 2019
)
```
This will result in the amount of booking items for the filtered companies, in 2019.
However, the following expression has a very different result:

```ruby
CALCULATE (
    CALCULATE (
        SUM ( Orders[ItemId] );
        Company[Code] IN {"Microsoft", "Apple"};
    );
    Company[Code] IN {"Microsoft", "Dell"}
)
```
In this case, the use of nested `CALCULATE` statements does not intersect the filter. The result of the expression is **the amount of items only for Microsoft**, since the inner filter overwrites the outer.

You can override this behavior to intersect filter by using the `KEEPFILTERS` function:

```ruby
CALCULATE (
    CALCULATE (
        SUM ( Orders[ItemId] );
        KEEPFILTERS ( Company[Code] IN {"Microsoft", "Apple"} );
    );
    Company[Code] IN {"Microsoft", "Dell"}
)
```

In this context, only Microsoft remains visible at the end of the evaluation.


### Variables

Newer feature in DAX. Allows for reduced repetition of expressions and other nifty tricks, such as accessing outer row context.
You can store strings, numbers, and even tables in a variable.

Variables can be used to store the previous row context for later access:

```ruby
Product[Rank] =

VAR _CurrentPrice = Product[Price]

RETURN

COUNTROWS (
    FILTER(
        Product;
        Product[Price] > _CurrentPrice
    )
) + 1
```

In this case, you save the value of the previous price during the outer context evaluation, and use it to check whether the price in the inner context is greater.

## Evaluation contexts

The filter and row contexts are one of the most important concepts in DAX.

The filter context is defined by:

- row selection  
- column selection  
- report filters  
- slicer selection 


The row context is the concept of "current row" and you have a row context whenever you iterate a table, either explicitly (using an iterator) or implicitly (in a calculated column).

>  
> ##### Quick tip!
> 
> Starting from a row context, you can use related to access columns in other tables:


```ruby
SalePrice =

SUMX (
    Orders;
    Orders[Cost] * RELATED ( Product[Markup] )
)
```
> You need RELATED because the row context is iterating the sales table. The Markup columns is in another table, but the row context allows for the relationship to be propagated. 
> 
> 
```ruby
AverageDiscountedIncomeByProduct =

AVERAGEX (
    Products;
    SUMX (
        RELATEDTABLE ( Orders );
        Orders[SalePrice] * Products[Discount]
    )
)

```

The inner expression can safely reference two columns coming from different tables because there are **two row contexts**: the first one introduced by `AVERAGEX` and the second one by `SUMX`. Also note that `RELATEDTABLE` uses the row context to determine the set of rows to return, that is, the Orders of the current product which is the one being currently iterated by `AVERAGEX`.

### Context transition

The context transition in DAX is the transformation of row contexts into an equivalent filter context performed by `CALCULATE` functions.
When `CALCULATE` is executed within a row context, it transforms the context into the equivalent filter context and applies it to the data model before computing its expression.
This is an expensive operation, so it's better to avoid it when iterating large tables.

This concept is easier to understand through examples:

```ruby
Product[SumOfUnitsSold] = SUM ( Orders[UnitsSold] )
Product[SumOfUnitsSoldCalculate] = CALCULATE ( SUM ( Orders[UnitsSold] ) )
```

The first calculated column returns the grand total of `Orders[UnitsSold]`, because no filter context is active, whereas the one with `CALCULATE` returns the sum of `Orders[UnitsSold]` for the current product only, because the filter context containing the current product is automatically propagated to sales due to the relationship between the two tables.

It is important to note that context transition happens before further filters in `CALCULATE`. Thus, filters in `CALCULATE` might override filters produced by context transition as seen on previous sections.


## DAX Studio 

DAX Studio is a tool to write, execute, and analyze DAX queries in Power BI, Power Pivot for Excel, and Analysis Services Tabular. It can connect to any of sources and access their data model, such as an open Power BI file.

It includes an Object Browser, query editing and execution, formula and measure editing, syntax highlighting and formatting, integrated tracing and query execution breakdowns.

This tool supercharges you report development toolbelt and allows to access fairly advanced features such as:

* __Query tracing and server timings__: provides insights on execution time, and differentiates between time spent on formula or query engine to facilitate debugging.
* __Filter dumps__: you can create a measure automatically that shows the current filter context for each visual and data point. This filter dump can be set on a tooltip to make debugging infinitely easier.
* __Measure definitions__: similarly, you can automatically document all of your calculated measures with a single click inside of DAX Studio.
* __Language formatting__: you can also format your DAX queries using the syntax tree developed by SQLBI and used in [DAX Formatter](https://www.daxformatter.com/).
* __Dynamic Management Views (DMVs)__: the tool also grants access to several pre-defined DMVs that allow for finer control of the data model embedded in the file. For example, you can automate the documentation and definition of your model using a series of queries against the DMVs.


## Closing thoughts

This is but a small subset of all the capabilities of DAX and merely scratches the surface of the language. Some resources like [sqlbi.com](sqlbi.com) or [dax.guide](dax.guide) are great steps to take next.

If you are stuck with a certain problem, try looking or posting a question on the DAX tag in the [Power BI community forums](https://community.powerbi.com/). They are a friendly and knowledgeable bunch that will most certainly help you as long as you follow the site's rules and etiquette.

Hopefully the techniques and tools mentioned in this post as well as the simplicity of the language and it's Excel-like functionality encourage you to try the language or dive deeper into its guts. In any case, I'm more than happy to answer any questions you might have.