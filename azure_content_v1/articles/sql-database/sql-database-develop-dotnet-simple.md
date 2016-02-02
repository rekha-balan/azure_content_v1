<properties
    pageTitle="Connect to SQL Database by using .NET (C#)"
    description="Use the sample code in this quick start to build a modern application with C# and backed by a powerful relational database in the cloud with Azure SQL Database."
    services="sql-database"
    documentationCenter=""
    authors="tobbox"
    manager="jeffreyg"
    editor=""/>


<tags
    ms.service="sql-database"
    ms.workload="sql-database"
    ms.tgt_pltfrm="na"
    ms.devlang="dotnet"
    ms.topic="article"
    ms.date="12/17/2015"
    ms.author="tobiast"/>


# Using SQL Database from .NET (C#)
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


## Prerequisites
### .NET Framework
.NET Framework is pre-installed with Windows. For Linux and Mac OS X you can download .NET Framework from the [Mono Project](http://www.mono-project.com/).

### A SQL database
See the [getting started page](sql-database-get-started.md) to learn how to create a sample database.  It is important you follow the guide to create an **AdventureWorks database template**. The samples shown below only work with the **AdventureWorks schema**.  

## Step 1:  Get Connection String
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

5. On the blade for your database, click **Show database connection strings**.

6. If you intend to use the ADO.NET connection library, copy the string labeled **ADO.NET**.

	![Copy the ADO.NET connection string for your database][3-get-connection-string]

7. Paste the connection string information into your client program code.  You will need to replace the {your_password_here} with your real password.



For more information, see: [Connection Strings and Configuration Files](http://msdn.microsoft.com/library/ms254494.aspx).

<!-- Image references. -->

[1-select-sql]: ./media/sql-database-include-connection-string-20-portalshots/connection-string-select-sql.png


[2-select-database]: ./media/sql-database-include-connection-string-20-portalshots/connection-string-select-database.PNG

[3-get-connection-string]: ./media/sql-database-include-connection-string-20-portalshots/connection-string-dotnet.PNG


<!--
These three includes/ files are a sequenced set, but you can pick and choose:

includes/sql-database-include-connection-string-20-portalshots.md
includes/sql-database-include-connection-string-30-compare.md
includes/sql-database-include-connection-string-40-config.md
-->


## Step 2:  Connect
The [System.Data.SqlClient.SqlConnection class](https://msdn.microsoft.com/library/system.data.sqlclient.sqlconnection.aspx) is used to connect to SQL Database.

```
using System.Data.SqlClient;

class Sample
{
  static void Main()
  {
      using(var conn = new SqlConnection("Server=tcp:yourserver.database.windows.net,1433;Database=yourdatabase;User ID=yourlogin@yourserver;Password={yourpassword};Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;"))
      {
          conn.Open();
      }
  }
}
```

## Step 3: Execute a query
The [System.Data.SqlClient.SqlCommand](https://msdn.microsoft.com/library/system.data.sqlclient.sqlcommand.aspx) and [SqlDataReader](https://msdn.microsoft.com/library/system.data.sqlclient.sqldatareader.aspx) classes can be used to retrieve a result set from a query against SQL Database. Note that System.Data.SqlClient also supports retrieving data into an offline [System.Data.DataSet](https://msdn.microsoft.com/library/system.data.dataset.aspx).   

```
using System;
using System.Data.SqlClient;

class Sample
{
    static void Main()
    {
      using(var conn = new SqlConnection("Server=tcp:yourserver.database.windows.net,1433;Database=yourdatabase;User ID=yourlogin@yourserver;Password={yourpassword};Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;"))
        {
            var cmd = conn.CreateCommand();
            cmd.CommandText = @"
                    SELECT
                        c.CustomerID
                        ,c.CompanyName
                        ,COUNT(soh.SalesOrderID) AS OrderCount
                    FROM SalesLT.Customer AS c
                    LEFT OUTER JOIN SalesLT.SalesOrderHeader AS soh ON c.CustomerID = soh.CustomerID
                    GROUP BY c.CustomerID, c.CompanyName
                    ORDER BY OrderCount DESC;";

            conn.Open();

            using(var reader = cmd.ExecuteReader())
            {
                while(reader.Read())
                {
                    Console.WriteLine("ID: {0} Name: {1} Order Count: {2}", reader.GetInt32(0), reader.GetString(1), reader.GetInt32(2));
                }
            }                    
        }
    }
}

```  

## Step 4: Insert a row
In this example you will see how to execute an [INSERT](https://msdn.microsoft.com/library/ms174335.aspx) statement safely, pass parameters which protect your application from [SQL injection](https://technet.microsoft.com/library/ms161953(v=sql.105).aspx) vulnerability, and retrieve the auto-generated [Primary Key](https://msdn.microsoft.com/library/ms179610.aspx) value.  

```
using System;
using System.Data.SqlClient;

class Sample
{
    static void Main()
    {
        using(var conn = new SqlConnection("Server=tcp:yourserver.database.windows.net,1433;Database=yourdatabase;User ID=yourlogin@yourserver;Password={yourpassword};Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;"))
        {
            var cmd = conn.CreateCommand();
            cmd.CommandText = @"
                INSERT SalesLT.Product (Name, ProductNumber, StandardCost, ListPrice, SellStartDate)
                OUTPUT INSERTED.ProductID
                VALUES (@Name, @Number, @Cost, @Price, CURRENT_TIMESTAMP)";

            cmd.Parameters.AddWithValue("@Name", "SQL Server Express");
            cmd.Parameters.AddWithValue("@Number", "SQLEXPRESS1");
            cmd.Parameters.AddWithValue("@Cost", 0);
            cmd.Parameters.AddWithValue("@Price", 0);

            conn.Open();

            int insertedProductId = (int)cmd.ExecuteScalar();

            Console.WriteLine("Product ID {0} inserted.", insertedProductId);
        }
    }
}
```
