<properties
    pageTitle="Connect to SQL Database by using Node.js on Windows"
    description="Presents a Node.js code sample you can use to connect to Azure SQL Database. The sample runs on a Windows client computer."
    services="sql-database"
    documentationCenter=""
    authors="meet-bhagdev"
    manager="jeffreyg"
    editor=""/>


<tags
    ms.service="sql-database"
    ms.workload="data-management"
    ms.tgt_pltfrm="na"
    ms.devlang="nodejs"
    ms.topic="article"
    ms.date="12/17/2015"
    ms.author="meetb"/>


# Connect to SQL Database by using Node.js on Windows
> [AZURE.SELECTOR-LIST (Language | Platform & Depth)]
- [(CSharp. ADO.NET | Windows. Starter)](sql-database-develop-dotnet-simple.md)
- [(CSharp. Custom logic | Windows. Retry logic)](sql-database-develop-csharp-retry-windows.md)
- [(CSharp. Enterprise Library | Windows. Retry logic)](sql-database-develop-entlib-csharp-retry-windows.md)
- [(CSharp. Entity Framework | Windows. Relational to object)](http://msdn.microsoft.com/library/azure/ff951633.aspx)
- [(C++. ODBC | Windows. Starter)](http://msdn.microsoft.com/library/azure/hh974312.aspx)
- [(Java. JDBC | Windows. Starter)](sql-database-develop-java-simple-windows.md)
- [(Node.js | Linux. Starter)](sql-database-develop-nodejs-simple-linux.md)
- [(Node.js | Windows. Starter)](sql-database-develop-nodejs-simple-windows.md)
- [(Node.js | Mac. Starter)](sql-database-develop-nodejs-simple-mac.md)
- [(PHP | Windows. Starter)](sql-database-develop-php-simple-windows.md)
- [(PHP | Windows. Retry logic)](sql-database-develop-php-retry-windows.md)
- [(Python | Linux. Starter)](sql-database-develop-python-simple-ubuntu-linux.md)
- [(Python | Windows. Starter)](sql-database-develop-python-simple-windows.md)
- [(Python | Mac. Starter)](sql-database-develop-python-simple-mac-osx.md)
- [(Ruby | Linux. Starter)](sql-database-develop-ruby-simple-linux.md)
- [(Ruby | Mac. Starter)](sql-database-develop-ruby-simple-mac-osx.md)
- [(Ruby | Windows. Starter)](sql-database-develop-ruby-simple-windows.md)


This topic presents a Node.js code sample that you can use to connect to Azure SQL Database. The Node.js program runs on a Windows client computer. To manage the connection, the msnodesql driver is used.

## Prerequisites
The following software items must exist on your client development computer.

* [Node.js](https://nodejs.org/en/download/) - Click on the windows installer and download the appropriate msi installer. Once downloaded run the msi to install Node.js

### Install the required modules
After your machine is configured with **node**, open cmd.exe and navigate to a directory where you plan to create your Node.js project, and enter the following commands.

    npm init
    npm install tedious


**npm init** creates a node project. To retain the defaults during your project creation, press enter until the project is created. Now you see a **package.json** file in your project directory.

### A SQL database
See the [getting started page](sql-database-get-started.md) to learn how to create a sample database.  It is important you follow the guide to create an **AdventureWorks database template**. The samples shown below only work with the **AdventureWorks schema**.

## Step 1: Get Connection Details
<!--
includes/sql-database-include-connection-string-20-portalshots.md

Latest Freshness check:  2015-09-02 , GeneMi.

## Connection string
-->


### Obtain the connection string from the Azure portal


Use the [Azure preview portal](https://portal.azure.com/) to obtain the connection string necessary for your client program to interact with Azure SQL Database:


1. Click **BROWSE** > **SQL databases**.

    ![Select SQL][1-select-sql]

2. Enter the name of your database into the filter text box near the upper-left of the **SQL databases** blade.

    ![Select Database][2-select-database]]

3. Click the row for your database.

4. After the blade appears for your database, for visual convenience you can click the standard minimize controls to collapse the blades  you used for browsing and database filtering.

5. Make note of the **SQL database** name and the **Server name**.  The username will be yourusername@yourserver.

	![Get Connection Details][3-get-connection-details]

7.  Paste the connection details into your client program code.  You will need to replace the {your_password_here} with your real password.


<!--
Could not find a good link for PHP

For more information, see:<br/>[Connection Strings and Configuration Files](https://msdn.microsoft.com/library/ms378428.aspx).
-->


<!-- Image references. -->

[1-select-sql]: ./media/sql-database-include-connection-string-20-portalshots/connection-string-select-sql.png

[2-select-database]: ./media/sql-database-include-connection-string-20-portalshots/connection-string-select-database.PNG

[3-get-connection-details]: ./media/sql-database-include-connection-string-20-portalshots/connection-string-details.PNG


<!--
These three includes/ files are a sequenced set, but you can pick and choose:

includes/sql-database-include-connection-string-20-portalshots.md
includes/sql-database-include-connection-string-30-compare.md
includes/sql-database-include-connection-string-40-config.md
-->


## Step 2: Connect
The [new Connection](http://pekim.github.io/tedious/api-connection.html) function is used to connect to SQL Database.

    var Connection = require('tedious').Connection;
    var config = {
        userName: 'yourusername',
        password: 'yourpassword',
        server: 'yourserver.database.windows.net',
        // If you are on Microsoft Azure, you need this:
        options: {encrypt: true, database: 'AdventureWorks'}
    };
    var connection = new Connection(config);
    connection.on('connect', function(err) {
    // If no error, then good to proceed.
        console.log("Connected");
    });


## Step 3:  Execute a query
All SQL statements are executed using the [new Request()](http://pekim.github.io/tedious/api-request.html) function. If the statement returns rows, such as a select statement, you can retreive them using the [request.on()](http://pekim.github.io/tedious/api-request.html) function. If there are no rows, [request.on()](http://pekim.github.io/tedious/api-request.html) function returns empty lists.

    var Connection = require('tedious').Connection;
    var config = {
        userName: 'yourusername',
        password: 'yourpassword',
        server: 'yourserver.database.windows.net',
        // When you connect to Azure SQL Database, you need these next options.
        options: {encrypt: true, database: 'AdventureWorks'}
    };
    var connection = new Connection(config);
    connection.on('connect', function(err) {
        // If no error, then good to proceed.
        console.log("Connected");
        executeStatement();
    });

    var Request = require('tedious').Request;
    var TYPES = require('tedious').TYPES;

    function executeStatement() {
        request = new Request("SELECT c.CustomerID, c.CompanyName,COUNT(soh.SalesOrderID) AS OrderCount FROM SalesLT.Customer AS c LEFT OUTER JOIN SalesLT.SalesOrderHeader AS soh ON c.CustomerID = soh.CustomerID GROUP BY c.CustomerID, c.CompanyName ORDER BY OrderCount DESC;", function(err) {
          if (err) {
               console.log(err);}
        });
        var result = "";
        request.on('row', function(columns) {
            columns.forEach(function(column) {
              if (column.value === null) {
                console.log('NULL');
              } else {
                result+= column.value + " ";
              }
            });
            console.log(result);
            result ="";
        });

        request.on('done', function(rowCount, more) {
        console.log(rowCount + ' rows returned');
        });
        connection.execSql(request);
    }


## Step 4: Insert a row
In this example you will see how to execute an [INSERT](https://msdn.microsoft.com/library/ms174335.aspx) statement safely, pass parameters which protect your application from [SQL injection](https://technet.microsoft.com/library/ms161953(v=sql.105).aspx) vulnerability, and retrieve the auto-generated [Primary Key](https://msdn.microsoft.com/library/ms179610.aspx) value.  

    var Connection = require('tedious').Connection;
    var config = {
        userName: 'yourusername',
        password: 'yourpassword',
        server: 'yourserver.database.windows.net',
        // If you are on Azure SQL Database, you need these next options.
        options: {encrypt: true, database: 'AdventureWorks'}
    };
    var connection = new Connection(config);
    connection.on('connect', function(err) {
        // If no error, then good to proceed.
        console.log("Connected");
        executeStatement1();
    });

    var Request = require('tedious').Request
    var TYPES = require('tedious').TYPES;

    function executeStatement1() {
        request = new Request("INSERT SalesLT.Product (Name, ProductNumber, StandardCost, ListPrice, SellStartDate) OUTPUT INSERTED.ProductID VALUES (@Name, @Number, @Cost, @Price, CURRENT_TIMESTAMP);", function(err) {
         if (err) {
             console.log(err);}
        });
        request.addParameter('Name', TYPES.NVarChar,'SQL Server Express 2014');
        request.addParameter('Number', TYPES.NVarChar , 'SQLEXPRESS2014');
        request.addParameter('Cost', TYPES.Int, 11);
        request.addParameter('Price', TYPES.Int,11);
        request.on('row', function(columns) {
            columns.forEach(function(column) {
              if (column.value === null) {
                console.log('NULL');
              } else {
                console.log("Product id of inserted item is " + column.value);
              }
            });
        });        
        connection.execSql(request);
    }
