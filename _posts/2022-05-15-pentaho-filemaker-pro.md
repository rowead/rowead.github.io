---
layout: article
title: Pentaho and Filemaker Pro
excerpt: >
  Connect to Filemaker Pro with JDBC for simple Change Data Capture.
published: true
lightbox: true
---
You can connect to almost any database out there with ETL tools like Pentaho PDI (Kettle) and the new Apache Hop
but there are some that need a generic JDBC/ODBC connection.

You can achieve this for Filemaker Pro databases by grabbing a copy of the JDBC driver jar and placing it into the "lib"
folder in Pentaho's install folder. You can get the details for the drivers from Filemaker's docs, [Installing FileMaker JDBC client drivers](https://support.claris.com/s/article/Installing-FileMaker-JDBC-client-drivers-1503693055862?language=en_US){:target="_blank"}. You will also need to turn on JDBC/ODBC connections within the Filemaker Pro server

Set up the connection with the following settings:

<div class="center">
  {% include fig-caption.html url="/assets/images/2022-05/pentaho-filemaker-connection.png" class="image--fullwidth" description="Database Connection settings in Pentaho Kettle for a Filemaker Database" %}
</div>

| Setting                                              | Value                                                                           |
|------------------------------------------------------|---------------------------------------------------------------------------------|
| Connection Type                                      | "Generic Database"                                                              |
| Access                                               | "Native (JDBC)"                                                                 |
| Custom Connection URL                                | "jdbc:filemaker://SERVER_NAME/DATABASE?useUnicode=true&characterEncoding=UTF-8" |
| Custom driver class name | "com.filemaker.jdbc.Driver"                                                     |

If you have any problems with Pentaho failing to convert fields to its native field format (Dates, Number and
Calculated fields), try converting them to strings using the STRVAL() function in the connection SQL. This will allow you
to get a better sense of the field's contents and use Pentaho's "Select Values" step and its "Meta-data" tab to convert
fields into the data types you need.

<div class="center">
  {% include fig-caption.html url="/assets/images/2022-05/pentaho-filemaker-sql.png" class="image--fullwidth" description="Typical SQL using STRVAL for tricky Filemaker fields" %}
</div>

Now you can easily do some powerful things like simple Change Data Capture by creating a [Slowly Changing Dimension Type 2](https://www.kimballgroup.com/2008/09/slowly-changing-dimensions-part-2/){:target="_blank"} from your table.

<div class="center">
  {% include fig-caption.html url="/assets/images/2022-05/cdc.png" class="image--xl" description="Transformation creating a SCD Type 2 from a Filemaker Pro Table." %}
</div>

You can run this from your desktop, but I manage this in git and use Jenkins to pull the code and run it on a schedule.
