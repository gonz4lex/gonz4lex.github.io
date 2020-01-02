title: An introduction to Power BI for data scientists
date: 2019-12-6 17:42
category: power bi
tags: data science
slug: an-introduction-to-power-bi-for-data-scientists
author: Alex Gonzalez
summary: Power BI is a great free tool that can be used to quickly prototype visualizations for exploratory data analysis.
header_cover: images/power-bi.png
status: published

Get started with Power BI to speed up your exploratory data analysis when looking for trends and features than might be useful for prediction or classification with machine learning algorithms.

## Getting Data

Connecting to a data source in Power BI is quite straightforward. Just navigate to the Home tab in the ribbon and select **Get Data** to find a dropdown menu of possible options. Common sources are SQL Server, Analysis Services (SSAS), Excel and Dynamics 365, among others. In most cases, such as SQL Server, you can also specify the query you want to run against the database.
Note that when connecting to a source you can either query the data directly through **DirectQuery** or **Import** modes, but you cannot combine both methods into the same solution. There is also the **LiveConnection** mode, a variation of DirectQuery geared towards SSAS Tabular that we will not cover here.

There are some crucial differences between the two connection modes:
* In DirectQuery, the data stays at the source and is not imported into your machine's memory and the size of the `.pbix` file is dramatically lower than an import solution. However, each event during report visualization will trigger live queries that might harm the user experience. 
* In Import mode, the data model is loaded into Power BI desktop and remains static until a refresh is triggered. This makes for smooth user experience but increase file size and memory consumption.


When you have selected the source, Power BI will present you with a preview of your data. If everything looks OK, you can continue towards the next step. Otherwise, you can review your query and make any necessary changes.

You can also use a folder as your source, allowing you to import several files at once such as a series of Excel or CSV files. They are loaded as binary data types and double clicking them loads their values.

## Clean and Transform

## Data Modeling

There some key aspects to take into account when modeling data with Power BI, such as data categories and summarization; or establishing adequate relationships among your tables. You can also create custom hierarchies such as Year/Month/Week or Country/Destination/Region by dragging dragging columns on top of each other in the Report or Data View.

### Data types, categories and format strings

There are several different types of data to work with in Power BI, such as integer, decimal/fixed, currency, date/time/datetime, string and binary objects.
The data type is what allows certain functions and operations to be performed on a specific column, and it's **not the same as the data format**.

The format string is the way that this data will be visually represented, such as percentage, with thousand separator, with decimals... There also many different formats for date and time objects which can also be changed flexibly with the DAX `FORMAT ( )` function.

Text data can be categorized, which is useful when you have geographical data or URLs in your table. Setting a data category for these columns allows for advanced features such as URL icons, embedded images and map visuals. Numerical data is uncategorized by default, since we use data types to differentiate it.
Both types of data need *summarization*, which is the default statistic that will be calculated by visuals using the measure. Possible options are `sum`, `average`, `minimum`, `maximum`, `count` and `distinct` for numerical types, while text types only have count and distinct. You can also use `first` and `last` within some visuals. 



It's important to make a distinction between modeling your data within the Power Query editor or within Power BI Desktop itself, since different operations can be performed and the computational load is different. Usually, we use the *Query Editor* to clean, shape and transform data from the source, while the *Data View* (i.e. Power Pivot) is often used to model the imported data and define calculations for analytics.

### Power Query editor and the `M` language

Power Query is a data discovery and query tool, good for shaping and mashing up data even before you’ve added the first row of data to the table. Inside the Power Query editor you can use the `M` programming language to shape your data, creating step by step scripts that iteratively clean, transform and add columns to your model.
Moreover, you can leverage the amazing Power BI user interface that makes this data engineering process very straight-forward and with zero necessary knowledge of `M`.

Some of the operations you can perform are:

* Duplicate, transform or split columns
  * You can use regular expressions, functions and calculations to engineer new features.
* Change data types and formats
  * Power Query can also detect automatically the best data type for your column if you are not sure.
* Pivot, unpivot and sort columns
* Merge (join) and append (union) queries
* Remove or keep duplicates, errors, nulls, headers...
* Group by columns and aggregate measures

### Enhance your model with the Data View

Not only can you explore all of your data interactively within this view, but you can also further improve the analytical capabilities of your model by using DAX expressions that leverage different filtering contexts and time frames to create complex features.

### Create calculated columns

You can create calculated columns, for example, to establish a relationship between different tables, or to create indexes that could later be used as reference by iterator functions.
Creating a calculated column consumes memory in your machine as it expands your data model if your tables have many records. Consider creating them only when necessary or for extremely complex calculations that should be run live.

Remember to custom sort your columns, i.e. `MonthName` should be sorted by `MonthNumber` and not alphabetically by itself.

### Create calculated measures

Measures are calculations that exists in your Power BI data model and is computed on the fly at each event. With DAX you can define a measure once and then slice it by all the different fields in your model, as long as the necessary relationships exist.

Since they are computed at query time, calculated measures consume more CPU than a column, but they do not grow the tables in your data model.

### Create calculated tables

Calculated tables allow you to express a whole new range of modeling capabilities, such as performing different types of joins and creating new tables on the fly based on the results of a formula.
For example, you can create a date dimension with DAX by using the date to derive all necessary attributes such as month, week number or day of the week, or create summary tables with aggregated values.

### Relationships in the data model

Data relationships are similar to how a cube is structured in Analysis Services. In the Model View you can access a diagrammatic view of your data, and you can define how your tables are related.
To create a relationship, just click on a column on your fact table and drag it into the key on the dimension table. This generates an arrow indicating the active relationship, which you can double click to access advanced options such as setting the cardinality or the direction of the relationship.

Your options for cardinality are Many to One, and One to One. Many to One is the fact to dimension type relationship, for example a sales table with multiple rows per product being matched up with a table listing products in their own unique row.

## Visualization and report design

Visuals are the main element of Power BI reporting, and they have a wide range of formatting options that allows for almost limitless customization. 


### Editing interactions

When you have multiple visualizations on the same report page, selecting a particular segment by clicking or using a slicer will affect all the visuals on that page. In some cases, though, you may want to slice only specific visuals. For example, you might not want a scatter plot to be cross-filtered because that would remove crucial meaning from the visualization.
You can define how visuals interacts with each other when the user selects data points. 

### Z-order

When you have lots of elements on a report, Power BI lets you manage how they overlap with each other. How items are layered, or arranged on top of one another, is often referred to as the z-order.
This is particularly useful when you want to use shapes to create backgrounds, highlights or containers. For example, you might use a shape to create the background for your report title, or to enclose a set of KPIs in a container. You can also set a semi-transparent shape over a graph section to highlight it.

### Bookmarks

Bookmarks are a great feature that allow for some nice tricks with user interaction. Essentially, a bookmark is a snapshot of your report page at a given time. They can save filter state, formatting options, visual placement, etc.
You can use them to change between report pages, create a guided navigation for the user, change themes, show help or filter panes, send emails and many more things. 


## Time Intelligence

Time intelligence manages complex time-based calculations with reduced user input. It needs a date table with no date gaps, which Power BI creates automagically if the setting `Auto Date/Time` is enabled, which is the default case.
It is recommended to create your own date table instead of relying on this one, since it's not possible to adjust if necessary and can actually only be viewed from DAX Studio.

If you decide to work with it, you can use dot notation to access the time calculations, i.e. `Orders[Date].[Year]` to automatically extract the year of that date.