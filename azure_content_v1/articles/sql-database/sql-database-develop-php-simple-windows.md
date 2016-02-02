<properties
    pageTitle="Connect to SQL Database by using PHP on Windows"
    description="Presents a sample PHP program that connects to Azure SQL Database from a Windows client, and provides links to the necessary software components needed by the client."
    services="sql-database"
    documentationCenter=""
    authors="meet-bhagdev"
    manager="jeffreyg"
    editor=""/>


<tags
    ms.service="sql-database"
    ms.workload="data-management"
    ms.tgt_pltfrm="na"
    ms.devlang="php"
    ms.topic="article"
    ms.date="12/17/2015"
    ms.author="meetb"/>


# Connect to SQL Database by using PHP on Windows
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


This topic illustrates how you can connect to Azure SQL Database from a client application written in PHP that runs on Windows.


## Prerequisites


To run the PHP code sample given in this topic, your client computer must have the following software items installed:


- [Microsoft Drivers for PHP for Microsoft SQL Server](http://www.microsoft.com/download/details.aspx?id=20098)
 - `SQLSRV32.EXE` contains the latest bits.
- [Microsoft SQL Server Native Client 11.0](http://www.microsoft.com/download/details.aspx?id=36434)
- [Microsoft ODBC Driver](https://www.microsoft.com/en-us/download/details.aspx?id=36434)
- IIS Express
- [PHP 5.6 for IIS Express](http://www.microsoft.com/web/downloads/platform.aspx)
 - Download using the platform installer.
 - Make sure you use Internet Explorer to download the platform installer.


Check out our [team blog](http://blogs.msdn.com/b/sqlphp/archive/2015/05/11/getting-started-with-php-and-microsoft-sql-server.aspx) and [video](https://www.youtube.com/watch?v=0oCjiRK_tUk) to learn how to install and setup the aforementioned requirements.


<!--
This include file is probably used in the following topics:
sql-database-develop-php-simple-windows.md
sql-database-develop-php-retry-windows.md

MightyPen = genemi
meet-bhagdev
DateOfLatestFreshnessVerification = 2015-07-10
DateOfLatestContentUpdate = 2015-07-10
-->



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


## Step 2:  Connect
This **OpenConnection** function is called near the top in all of the functions that follow.

    function OpenConnection()
    {
        try
        {
            $serverName = "tcp:myserver.database.windows.net,1433";
            $connectionOptions = array("Database"=>"AdventureWorks",
                "Uid"=>"MyUser", "PWD"=>"MyPassword");
            $conn = sqlsrv_connect($serverName, $connectionOptions);
            if($conn == false)
                die(FormatErrors(sqlsrv_errors()));
        }
        catch(Exception $e)
        {
            echo("Error!");
        }
    }


## Step 3:  Execute query
The [sqlsrv_query()](http://php.net/manual/en/function.sqlsrv-query.php) function can be used to retrieve a result set from a query against SQL Database. This function essentially accepts any query and the connection object and returns a result set which can be iterated over with the use of [sqlsrv_fetch_array()](http://php.net/manual/en/function.sqlsrv-fetch-array.php).

    function ReadData()
    {
        try
        {
            $conn = OpenConnection();
            $tsql = "SELECT [CompanyName] FROM SalesLT.Customer";
            $getProducts = sqlsrv_query($conn, $tsql);
            if ($getProducts == FALSE)
                die(FormatErrors(sqlsrv_errors()));
            $productCount = 0;
            while($row = sqlsrv_fetch_array($getProducts, SQLSRV_FETCH_ASSOC))
            {
                echo($row['CompanyName']);
                echo("<br/>");
                $productCount++;
            }
            sqlsrv_free_stmt($getProducts);
            sqlsrv_close($conn);
        }
        catch(Exception $e)
        {
            echo("Error!");
        }
    }


## Step 4:  Insert a row
In this example you will see how to execute an [INSERT](https://msdn.microsoft.com/library/ms174335.aspx) statement safely, pass parameters which protect your application from [SQL injection](https://technet.microsoft.com/library/ms161953(v=sql.105).aspx) vulnerability, and retrieve the auto-generated [Primary Key](https://msdn.microsoft.com/library/ms179610.aspx) value.  

    function InsertData()
    {
        try
        {
            $conn = OpenConnection();

            $tsql = "INSERT SalesLT.Product (Name, ProductNumber, StandardCost, ListPrice, SellStartDate) OUTPUT             INSERTED.ProductID VALUES ('SQL Server 1', 'SQL Server 2', 0, 0, getdate())";
            //Insert query
            $insertReview = sqlsrv_query($conn, $tsql);
            if($insertReview == FALSE)
                die(FormatErrors( sqlsrv_errors()));
            echo "Product Key inserted is :";
            while($row = sqlsrv_fetch_array($insertReview, SQLSRV_FETCH_ASSOC))
            {   
                echo($row['ProductID']);
            }
            sqlsrv_free_stmt($insertReview);
            sqlsrv_close($conn);
        }
        catch(Exception $e)
        {
            echo("Error!");
        }
    }

## Step 5:  Rollback a transaction
This code example demonstrates the use of transactions in which you:

-Begin a transaction

-Insert a row of data, Update another row of data

-Commit your transaction if the insert and update were successful and rollback the transaction if one of them was not

    function Transactions()
    {
        try
        {
            $conn = OpenConnection();

            if (sqlsrv_begin_transaction($conn) == FALSE)
                die(FormatErrors(sqlsrv_errors()));

            $tsql1 = "INSERT INTO SalesLT.SalesOrderDetail (SalesOrderID,OrderQty,ProductID,UnitPrice)
            VALUES (71774, 22, 709, 33)";
            $stmt1 = sqlsrv_query($conn, $tsql1);

            /* Set up and execute the second query. */
            $tsql2 = "UPDATE SalesLT.SalesOrderDetail SET OrderQty = (OrderQty + 1) WHERE ProductID = 709";
            $stmt2 = sqlsrv_query( $conn, $tsql2);

            /* If both queries were successful, commit the transaction. */
            /* Otherwise, rollback the transaction. */
            if($stmt1 && $stmt2)
            {
                   sqlsrv_commit($conn);
                   echo("Transaction was commited");
            }
            else
            {
                sqlsrv_rollback($conn);
                echo "Transaction was rolled back.\n";
            }
            /* Free statement and connection resources. */
            sqlsrv_free_stmt( $stmt1);
            sqlsrv_free_stmt( $stmt2);
        }
        catch(Exception $e)
        {
            echo("Error!");
        }
    }


## Next steps
For more information regarding PHP installation and usage, see [Accessing SQL Server Databases with PHP](http://technet.microsoft.com/library/cc793139.aspx).

