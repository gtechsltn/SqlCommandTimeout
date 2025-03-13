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
