<properties
    pageTitle="PHP retry logic to connect to SQL Database | Microsoft Azure"
    description="Presents a sample PHP program that connects to Azure SQL Database from a Windows client with transient fault handling, and provides links to the necessary software components needed by the client."
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


# Connect to SQL Database by using PHP on Windows with Transient Fault Handling
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


## Step 2:  Connect and Query
The demo program is designed so that a transient error during an attempt to connect leads to a retry. But a transient error during query command causes the program to discard the connection and create a new connection, before retrying the query command. We neither recommend nor disrecommend this design choice. The demo program illustrates some of the design flexibility that is available to you.

<br>The length of this code sample is due mostly to the catch exception logic. A shorter version of this Program.cs file is available this [here](sql-database-develop-php-simple-windows.md).
<br>The Main method is in Program.cs. The callstack runs as follows:

* Main calls ConnectAndQuery.
* ConnectAndQuery calls EstablishConnection.
* EstablishConnection calls IssueQueryCommand.

The [sqlsrv_query()](http://php.net/manual/en/function.sqlsrv-query.php) function can be used to retrieve a result set from a query against SQL Database. This function essentially accepts any query and the connection object and returns a result set which can be iterated over with the use of [sqlsrv_fetch_array()](http://php.net/manual/en/function.sqlsrv-fetch-array.php).

    <?php
        // Variables to tune the retry logic.  
        $connectionTimeoutSeconds = 30;  // Default of 15 seconds is too short over the Internet, sometimes.
        $maxCountTriesConnectAndQuery = 3;  // You can adjust the various retry count values.
        $secondsBetweenRetries = 4;  // Simple retry strategy.
        $errNo = 0;
        $serverName = "tcp:yourdatabase.database.windows.net,1433";
        $connectionOptions = array("Database"=>"AdventureWorks",
           "Uid"=>"yourusername", "PWD"=>"yourpassword", "LoginTimeout" => $connectionTimeoutSeconds);
        $conn;
        $errorArr = array();
        for ($cc = 1; $cc <= $maxCountTriesConnectAndQuery; $cc++)
        {
            $errorArr = array();
            $ctr = 0;
            // [A.2] Connect, which proceeds to issue a query command.
            $conn = sqlsrv_connect($serverName, $connectionOptions);  
            if( $conn == true)
            {
                echo "Connection was established";
                echo "<br>";

                $tsql = "SELECT [CompanyName] FROM SalesLT.Customer";
                $getProducts = sqlsrv_query($conn, $tsql);
                if ($getProducts == FALSE)
                    die(FormatErrors(sqlsrv_errors()));
                $productCount = 0;
                $ctr = 0;
                while($row = sqlsrv_fetch_array($getProducts, SQLSRV_FETCH_ASSOC))
                {   
                    $ctr++;
                    echo($row['CompanyName']);
                    echo("<br/>");
                    $productCount++;
                    if($ctr>10)
                        break;
                }
                sqlsrv_free_stmt($getProducts);
                break;
            }
            // Adds any the error codes from the SQL Exception to an array.
            else {  
                if( ($errors = sqlsrv_errors() ) != null) {
                    foreach( $errors as $error ) {
                        $errorArr[$ctr] = $error['code'];
                        $ctr = $ctr + 1;
                    }
                }
                $isTransientError = TRUE;
                // [A.4] Check whether sqlExc.Number is on the whitelist of transients.
                $isTransientError = IsTransientStatic($errorArr);
                if ($isTransientError == TRUE)  // Is a static persistent error...
                {
                    echo("Persistent error suffered, SqlException.Number==". $errorArr[0].". Program Will terminate.");
                    echo "<br>";
                    // [A.5] Either the connection attempt or the query command attempt suffered a persistent SqlException.
                    // Break the loop, let the hopeless program end.
                    exit(0);
                }
                // [A.6] The SqlException identified a transient error from an attempt to issue a query command.
                // So let this method reloop and try again. However, we recommend that the new query
                // attempt should start at the beginning and establish a new connection.
                if ($cc >= $maxCountTriesConnectAndQuery)
                {
                    echo "Transient errors suffered in too many retries - " . $cc ." Program will terminate.";
                    echo "<br>";
                    exit(0);
                }
                echo("Transient error encountered.  SqlException.Number==". $errorArr[0]. " . Program might retry by itself.");  
                echo "<br>";
                echo $cc . " Attempts so far. Might retry.";
                echo "<br>";
                // A very simple retry strategy, a brief pause before looping. This could be changed to exponential if you want.
                sleep(1*$secondsBetweenRetries);
            }
            // [A.3] All has gone well, so let the program end.
        }
        function IsTransientStatic($errorArr) {
            $arrayOfTransientErrorNumbers = array(4060, 10928, 10929, 40197, 40501, 40613);
            $result = array_intersect($arrayOfTransientErrorNumber, $errorArr);
            $count = count($result);
            if($count >= 0) //change to > 0 later.
                return TRUE;
        }
    ?>

## Next steps
For more information regarding PHP installation and usage, see [Accessing SQL Server Databases with PHP](http://technet.microsoft.com/library/cc793139.aspx).

