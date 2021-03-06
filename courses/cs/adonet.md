## ADO.NET

or how to access data the *old* way


## ADO.NET
ADO.NET is an object-oriented set of libraries that allows you to interact with data sources. Commonly, the data source is a database, but it could also be a text file, an Excel spreadsheet, or an XML file.


## Data Providers
ADO.NET provides a relatively common way to interact with data sources, but comes in different sets of libraries for each way you can talk to a data source. These libraries are called Data Providers and are usually named for the protocol or data source type they allow you to interact with.


## SQL Data Provider
Since, we are interested in the SQL Server, we will use the .NET Framework Data Provider for SQL Server which resides in the `System.Data.SqlCient` namespace.
```csharp
using System.Data.SqlClient;
```


## Overview
Before jumping into the code, we will have to understand some of the important objects of ADO.NET. In a typical scenario requiring data access, we need to perform four major tasks:
1. Connecting to the database
2. Passing the request to the database, i.e., a command like select, insert, or update.
3. Getting back the results, i.e., rows and/or the number of rows effected.
4. Storing the result and displaying it to the user.


## Overview
This can be visualized as:

<img src="media/adonet-flow.jpg" width="1000">


## SqlConnection
The `SqlConnection` class is used to establish a connection to the database. The `SqlConnection` uses a `ConnectionString` to identify the database server location, authentication parameters, and other information to connect to the database.

http://www.connectionstrings.com/ is the website where you can easily find the connection string for your database. They provide the strings, for almost all of the database services and their types.


## Example ConnectionString - 1/3
```csharp
"Data Source=(LocalDB)\v11.0;AttachDbFileName=|DataDirectory|\DatabaseFileName.mdf;InitialCatalog=DatabaseName;Integrated Security=True;MultipleActiveResultSets=True"
```
* `DataSource`: For SQL Server Express, LocalDB, SQL Server, and SQL Database, this setting specifies the name of the server and the SQL Server instance on the server. For example, you can specify ServerName\Instancename. You can use ".", "(local)", or "localhost" in place of the server name to specify the local computer, and you can use an IP address instead of the server name.
* `AttachDbFileName` specifies the path and name of the database file for SQL Server Express or LocalDB databases that are not defined in the local SQL Server Express instance.


## Example ConnectionString - 2/3
```csharp
"Data Source=(LocalDB)\v11.0;AttachDbFileName=|DataDirectory|\DatabaseFileName.mdf;InitialCatalog=DatabaseName;Integrated Security=True;MultipleActiveResultSets=True"
```
* `InitialCatalog` specifies the name of the database in the SQL Server instance catalog. If omitted, ADO.NET connects to the default database for the SQL Server instance.
*  `Integrated Security` specifies whether the connection should use the user ID and password in the connection string to log on to the SQL Server instance (=false), or the current Windows account credentials should be used for authentication (=true).


## Example ConnectionString - 3/3
```csharp
"Data Source=(LocalDB)\v11.0;AttachDbFileName=|DataDirectory|\DatabaseFileName.mdf;InitialCatalog=DatabaseName;Integrated Security=True;MultipleActiveResultSets=True"
```
* The `MultipleActiveResultSets` (MARS) option makes it possible to execute multiple queries simultaneously. This is a common scenario when you use the Entity Framework.


## SqlConnection Lifecycle
The purpose of creating a `SqlConnection` object is so you can enable other ADO.NET code to work with a database. Other ADO.NET objects, such as a `SqlCommand` and a `SqlDataAdapter` take a connection object as a parameter. The sequence of operations occurring in the lifetime of a `SqlConnection` are as follows:
1. Instantiate
2. Open the connection
3. Pass the connection to other ADO.NET objects
4. Perform database operations with the other ADO.NET objects
5. Close the connection


## Demo Code
```csharp
// 1. Instantiate the connection
SqlConnection conn = new SqlConnection(
  "Data Source=(local);Initial Catalog=Northwind;Integrated Security=SSPI");
SqlDataReader rdr = null;
try {
  // 2. Open the connection
  conn.Open();
  // 3. Pass the connection to a command object
  SqlCommand cmd = new SqlCommand("select * from Customers", conn);
  // 4. Use the connection to get query results
  rdr = cmd.ExecuteReader();
} finally {
  // close the reader
  if (rdr != null)
    rdr.Close();
  // 5. Close the connection
  if (conn != null)
    conn.Close();
}
```


## SqlCommand
A `SqlCommand` object allows you to specify what type of interaction you want to perform with a database. For example, you can do select, insert, modify, and delete commands on rows of data in a database table.


## SqlCommand
Similar to other C# objects, you instantiate a `SqlCommand` object via the new instance declaration, as follows:
```csharp
SqlCommand cmd = new SqlCommand("select CategoryName from Categories", conn);
```


## Quering data
When using a SQL `SELECT` command, you retrieve a data set for viewing. To accomplish this with a `SqlCommand` object, you would use the `ExecuteReader` method, which returns a `SqlDataReader` object. The example below shows how to use the `SqlCommand` object to obtain a `SqlDataReader` object:
```csharp
// 1. Instantiate a new command with a query and connection
 SqlCommand cmd = new SqlCommand("select CategoryName from Categories", conn);

// 2. Call Execute reader to get query results
 SqlDataReader rdr = cmd.ExecuteReader();
```


## Getting single values
Sometimes all we need from a database is a single value, which could be a count, sum, average, or other aggregated value from a data set. Performing an `ExecuteReader` and calculating the result in the code is not the most efficient way to do this. The best choice is to let the database perform the work and return just the single value we need. The following example shows how to do this with the `ExecuteScalar` method:
```csharp
// 1. Instantiate a new command
 SqlCommand cmd = new SqlCommand("select count(*) from Categories", conn);

 // 2. Call ExecuteNonQuery to send command
 int count = (int)cmd.ExecuteScalar();
 ```


## Inserting data
To insert data into a database, use the `ExecuteNonQuery` method of the `SqlCommand` object. The following code shows how to insert data into a database table:
```csharp
// prepare command string
string insertString = "insert into Categories (CategoryName, Description) values ('Miscellaneous', 'Whatever doesn''t fit elsewhere')";

// 1. Instantiate a new command with a query and connection
SqlCommand cmd = new SqlCommand(insertString, conn);

// 2. Call ExecuteNonQuery to send command
cmd.ExecuteNonQuery();
```


## Updating data
The ExecuteNonQuery method is also used for updating data. The following code shows how to update data:
```csharp
// prepare command string
 string updateString = @"
 update Categories
 set CategoryName = 'Other'
 where CategoryName = 'Miscellaneous'";

 // 1. Instantiate a new command with command text only
 SqlCommand cmd = new SqlCommand(updateString);

 // 2. Set the Connection property
 cmd.Connection = conn;

 // 3. Call ExecuteNonQuery to send command
 cmd.ExecuteNonQuery();
 ```


## Deleting data
You can also delete data using the `ExecuteNonQuery` method. The following example shows how to delete a record from a database:
```csharp
// prepare command string
 string deleteString = @"
 delete from Categories
 where CategoryName = 'Other'";

 // 1. Instantiate a new command
 SqlCommand cmd = new SqlCommand();

 // 2. Set the CommandText property
 cmd.CommandText = deleteString;

 // 3. Set the Connection property
 cmd.Connection = conn;

 // 4. Call ExecuteNonQuery to send command
 cmd.ExecuteNonQuery();
```


## Parameterizing the query
Parameterizing the query is done by using the `SqlParameter` passed into the command. For example, you might want to search for the records where a criteria matches. You can denote that criteria, by passing the variable name into the query and then adding the value to it using the `SqlParameter` object.
```csharp
// Create the command
SqlCommand insertCommand = new SqlCommand("INSERT INTO TableName
(FirstColumn, SecondColumn, ThirdColumn, ForthColumn)
VALUES (@0, @SecondParameter, @aDate, @3)", conn);

// Add the parameters.
insertCommand.Parameters.Add(new SqlParameter("0", 10));
insertCommand.Parameters.Add(new SqlParameter("SecondParameter", "Test Column"));
insertCommand.Parameters.Add(new SqlParameter("aDate", DateTime.Now));
insertCommand.Parameters.Add(new SqlParameter("3", false));
```


## using `using`
In C# there are some objects which use the resources of the system. Which need to be removed, closed, flushed and disposed etc. In C# you can either write the code to Create a new instance to the resource, use it, close it, flush it, dispose it. Or on the other hand you can simply just use the `using` statement block in which the object created is closed, flushed and disposed and the resources are then allowed to be used again by other processes.


## `using` example
```csharp
SqlConnection conn = new SqlConnection("connection string");
conn.Open();

// use the connection here

conn.Close();
conn.Dipose();
// connections don't get flushed
```
becomes
```csharp
using (SqlConnection conn = new SqlConnection("connection string"))
{
  conn.Open();
  // use the connection here
}
```


## Reading Data with the `SqlDataReader`
`SqlDataReader` is a type that is good for reading data in the most efficient manner possible. You can *not* use it for writing data. `SqlDataReader`s are often described as fast-forward firehose-like streams of data.
You can read from `SqlDataReader` objects in a forward-only sequential manner. Once you’ve read some data, you must save it because you will not be able to go back and read it again.


## Creating a `SqlDataReader` Object
Getting an instance of a SqlDataReader is a little different than the way you instantiate other ADO.NET objects. You must call `ExecuteReader` on a `SqlCommand` object, like this:
```csharp
SqlDataReader rdr = cmd.ExecuteReader();
```
The `ExecuteReader` method of the `SqlCommand` object returns a `SqlDataReader` instance. Creating a `SqlDataReader` with the new operator doesn’t do anything for you.


## Reading data
The typical method of reading from the data stream returned by the SqlDataReader is to iterate through each row with a while loop. The following code shows how to accomplish this:
```csharp
while (rdr.Read())
{
	// get the results of each column
	string contact = (string) rdr["ContactName"];
	string company = (string) rdr["CompanyName"];
	string city    = (string) rdr["City"];

	// print out the results
	Console.Write("{0,-25}", contact);
	Console.Write("{0,-20}", city);
	Console.Write("{0,-25}", company);
	Console.WriteLine();
}
```


## Closing `SqlDataReader`
Always remember to close and dispose your `SqlDataReader`, just like you need to close the `SqlConnection`. In fact, `SqlCommand` also requires disposing (there is no Close method). In order to be safe, it is recommended to use the `using` statement to let the Garbage Collector handle all three objects.


## Recommended pattern
```csharp
using(SqlConnection connection = new SqlConnection("connection string"))
{
  connection.Open();
  using(SqlCommand cmd = new SqlCommand("SELECT * FROM SomeTable", connection))
  {
  	using (SqlDataReader reader = cmd.ExecuteReader())
  	{
  		if (reader != null)
  		{
  			while (reader.Read())
  			{
  			  //do something
  			}
  		}
  	} // reader closed and disposed up here
  } // command disposed here
} //connection closed and disposed here
```


## Working with Disconnected Data
A `DataSet` is an in-memory data store that can hold numerous tables. `DataSets` only hold data and do not interact with a data source. It is the `SqlDataAdapter` that manages connections with the data source and gives us disconnected behavior. The `SqlDataAdapter` opens a connection only when required and closes it as soon as it has performed its task.


##Creating a `DataSet` Object
There isn’t anything special about instantiating a `DataSet`. You just create a new instance, just like any other object:
```csharp
DataSet dsCustomers = new DataSet();
```
Right now, the `DataSet` is empty and you need a `SqlDataAdapter` to load it.


## Creating A `SqlDataAdapter`
The `SqlDataAdapter` holds the SQL commands and connection object for reading and writing data. You initialize it with a SQL select statement and connection object:
```csharp
SqlDataAdapter daCustomers = new SqlDataAdapter("select CustomerID, CompanyName from Customers", conn);
```
The code above creates a new `SqlDataAdapter`, **daCustomers**. The SQL select statement specifies what data will be read into a `DataSet`. The connection object, **conn**, should have already been instantiated, but *not opened*. It is the `SqlDataAdapter`’s responsibility to open and close the connection.


## INSERT, UPDATE, DELETE
The code showed how to specify the select statement, but didn’t show the `INSERT`, `UPDATE`, and `DELETE` statements. These are added to the `SqlDataAdapter` after it is instantiated.

There are two ways to add `INSERT`, `UPDATE`, and `DELETE` commands: manually via `SqlDataAdapter` properties or with a `SqlCommandBuilder`.


## `SqlCommandBuilder - 1/2`
```csharp
SqlCommandBuilder cmdBldr = new SqlCommandBuilder(daCustomers);
```
Notice in the code above that the `SqlCommandBuilder` is instantiated with a single constructor parameter of the `SqlDataAdapter` instance. The `SqlCommandBuilder` will read the SQL SELECT statement (specified when the `SqlDataAdapter` was instantiated), infer the INSERT, UPDATE, and DELETE commands, and assign the new commands to the Insert, Update, and Delete properties of the `SqlDataAdapter`, respectively.


## `SqlCommandBuilder - 2/2`
The `SqlCommandBuilder` has limitations. It works when you do a simple select statement on a single table. However, when you need a join of two or more tables or must do a stored procedure, it won’t work.


## Filling the `DataSet`
Once you have a `DataSet` and `SqlDataAdapter` instances, you need to fill the `DataSet`. To do it we use the `Fill` method of the `SqlDataAdapter`:
```csharp
daCustomers.Fill(dsCustomers, "Customers");
```
The `Fill` method, in the code above, takes two parameters: a `DataSet` and a table name. The `DataSet` must be instantiated before trying to fill it with data. The second parameter is the name of the table that will be created in the `DataSet`.


## Updating Changes
After modifications are made to the data, you’ll want to write the changes back to the database. The following code shows how to use the Update method of the `SqlDataAdapter` to push modifications back to the database.
```csharp
daCustomers.Update(dsCustomers, "Customers");
```
The `Update` method, above, is called on the `SqlDataAdapter` instance that originally filled the dsCustomers `DataSet`. The second parameter to the Update method specifies which table, from the `DataSet`, to update.


## Stored procedures
A stored procedure is a pre-defined, reusable routine that is stored in a database. SQL Server compiles stored procedures, which makes them more efficient to use. Therefore, rather than dynamically building queries in your code, you can take advantage of the reuse and performance benefits of stored procedures.


## Executing a Stored Procedure
In addition to commands built with strings, the SqlCommand type can be used to execute stored procedures. There are two tasks require to make this happen: let the SqlCommand object know which stored procedure to execute and tell the SqlCommand object that it is executing a stored procedure. These two steps are shown below:
```csharp
// 1. create a command object identifying the stored procedure
SqlCommand cmd  = new SqlCommand("Ten Most Expensive Products", conn);

// 2. set the command object so it knows to execute a stored procedure
cmd.CommandType = CommandType.StoredProcedure;
```


## Sending Parameters to Stored Procedures
Using parameters for stored procedures is the same as using parameters for query string commands. The following code shows this:
```csharp
// 1. create a command object identifying the stored procedure
SqlCommand cmd  = new SqlCommand("CustOrderHist", conn);

// 2. set the command object so it knows to execute a stored procedure
cmd.CommandType = CommandType.StoredProcedure;

// 3. add parameter to command, which will be passed to the stored procedure
cmd.Parameters.Add(new SqlParameter("@CustomerID", custId));
```


## Catching the errors from SQL Server
SQL Server generates the errors for you to catch and work on them. In the namespace we're working there are two classes that work with the errors and exceptions thrown by SQL Server, 
1. `SqlException`
2. `SqlError`

These are used to catch the exceptions in the code and get the error details respectively. `SqlException` always contains at least one instance of `SqlError`.


## Catching the errors from SQL Server
### Example
```csharp
try
{
  //Do something here
}
catch (SqlException ex)
{
  for (int i = 0; i < exception.Errors.Count; i++)
  {
    Console.WriteLine("Error: " + exception.Errors[i].ToString());
  }
}
```
