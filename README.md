# Setting CommandTimeout in ADO.NET (SqlCommand.CommandTimeout)

In a C# data export project using ADO.NET or Entity Framework, the CommandTimeout property determines how long a database command will wait before timing out.

Setting CommandTimeout in ADO.NET (SqlCommand)

If you are using ADO.NET with SqlCommand, you can set the CommandTimeout property as follows:

```
using (SqlConnection conn = new SqlConnection(connectionString))
{
    conn.Open();
    using (SqlCommand cmd = new SqlCommand("your_sql_query", conn))
    {
        cmd.CommandTimeout = 300; // Timeout in seconds (5 minutes)
        cmd.ExecuteNonQuery();
    }
}
```

Setting CommandTimeout in Entity Framework (DbContext)

If you're using Entity Framework, you can set the timeout on the DbContext command like this:

```
using (var context = new YourDbContext())
{
    context.Database.SetCommandTimeout(300); // Timeout in seconds (5 minutes)
    
    var result = context.YourDbSet.ToList(); // Example query
}
```

Setting CommandTimeout in Entity Framework Core

In EF Core, you can specify CommandTimeout while configuring the database context:

```
services.AddDbContext<YourDbContext>(options =>
    options.UseSqlServer(connectionString, sqlOptions => sqlOptions.CommandTimeout(300)));
```

## Best Practices
+ Use a higher CommandTimeout if exporting large datasets.
+ Avoid excessively high values; it could indicate an underlying performance issue.
+ Consider asynchronous execution (ExecuteNonQueryAsync(), ToListAsync()) for better responsiveness.


# Best Practices for Using CommandTimeout in Large Data Exports:

## 1. Set an Appropriate Timeout:

If the query takes a long time, increase the timeout to a value that allows it to complete.

Example: Setting it to 5 minutes (300 seconds):
```
cmd.CommandTimeout = 300;
```

## 2. Optimize Your SQL Query:

+ Use proper indexes.
+ Avoid **SELECT *** (fetch only necessary columns).
+ Optimize joins and filters.

## 3. Use Asynchronous Execution (for Non-Blocking UI):

In ADO.NET:
```
await cmd.ExecuteNonQueryAsync();
```

In Entity Framework:
```
await context.YourDbSet.ToListAsync();
```

## 4. Stream Data Instead of Loading Everything at Once:

Use SqlDataReader with CommandBehavior.SequentialAccess for large exports.
```
using (SqlDataReader reader = cmd.ExecuteReader(CommandBehavior.SequentialAccess))
```

In EF Core, use AsNoTracking():
```
var data = await context.YourDbSet.AsNoTracking().ToListAsync();
```

## 5. Use Bulk Copy for Large Data Transfers:

SqlBulkCopy is efficient for exporting large data to another database or file:

```
using (SqlBulkCopy bulkCopy = new SqlBulkCopy(connection))
{
    bulkCopy.DestinationTableName = "YourTable";
    await bulkCopy.WriteToServerAsync(dataTable);
}
```

# Example: Using SqlDataReader with SequentialAccess for Large Exports
```
using (SqlConnection conn = new SqlConnection(connectionString))
{
    conn.Open();
    using (SqlCommand cmd = new SqlCommand("SELECT LargeColumn FROM YourTable", conn))
    {
        cmd.CommandTimeout = 600; // Increase timeout for large exports (10 minutes)
        
        using (SqlDataReader reader = cmd.ExecuteReader(CommandBehavior.SequentialAccess))
        {
            while (reader.Read())
            {
                // Example: Reading a large text column sequentially
                using (StreamWriter writer = new StreamWriter("exported_data.txt"))
                {
                    char[] buffer = new char[8192]; // Buffer size
                    long bytesRead;
                    long startIndex = 0;
                    
                    while ((bytesRead = reader.GetChars(0, startIndex, buffer, 0, buffer.Length)) > 0)
                    {
                        writer.Write(buffer, 0, (int)bytesRead);
                        startIndex += bytesRead; // Move to the next chunk
                    }
                }
            }
        }
    }
}
```

## Why Use SequentialAccess?
✅ Memory Efficiency: Prevents large binary or text fields from being fully loaded into memory.
✅ Better Performance: Streams data efficiently for large exports like file writing or API responses.
✅ Handles Large Blobs (Images, Files, XML, JSON, etc.)

# Alternative: Exporting Large Data to CSV
```
using (SqlConnection conn = new SqlConnection(connectionString))
{
    conn.Open();
    using (SqlCommand cmd = new SqlCommand("SELECT LargeColumn FROM YourTable", conn))
    {
        cmd.CommandTimeout = 600; // Increase timeout for large exports (10 minutes)
        
        using (SqlDataReader reader = cmd.ExecuteReader(CommandBehavior.SequentialAccess))
        {
            while (reader.Read())
            {
                // Example: Reading a large text column sequentially
                using (StreamWriter writer = new StreamWriter("data_export.csv"))
                {
                    while (reader.Read())
                    {
                        writer.WriteLine($"{reader["Column1"]},{reader["Column2"]}");
                    }
                }
            }
        }
    }
}
```

# References
* https://github.com/gtechsltn/NET9_SecureWebApp
* https://github.com/gtechsltn/NET48_WinSvc_DataExport
* https://github.com/gtechsltn/SqlCommandTimeout
* https://github.com/gtechsltn/ConsoleApp_Log4Net
* https://github.com/gtechsltn/SqlServerInsertFiles
* https://github.com/gtechsltn/ConsoleApp_SharpZipLib
