---
layout: post
title:  "Prevent SQL Injection in EF Core"
date:   2018-02-02 16:16:01 -0600
categories: Software Development
tags:   [ EF, Entity Framework, SQL Injection, T-SQL ]
---

### In-line SQL and EF Core
EF Core has always supported direct SQL queries against databases like below. 
```csharp
var searchTerm = "magic";
var posts = blogContext.Posts.FromSql("SELECT * FROM Posts Where Title={0}", searchTerm)
    .OrderBy(x => x.PostedDate);
```

If you notice the EF Core debug logs You'll notice a line like below.
```sql
SELECT * FROM Posts Where Title = @p0
```
### Great where is the SQL Injection?

Lets change our EF Core statement to use string interpolation like below.
```csharp
var searchTerm = "magic";
var posts = blogContext.Posts.FromSql($"SELECT * FROM Posts Where Title={searchTerm}")
    .OrderBy(x => x.PostedDate);
```

Now what you'll notice in the EF Core debug logs is...

```sql
SELECT * FROM Posts Where Title = magic
```
```csharp
System.Data.SqlClient.SqlException (0x80131904): Incorrect syntax near 'magic'.
```

### How do I use string interpolation then?
If we notice closely in EF Core code. There is an overload for FromSql that takes a FormattableString type.
So instead of sending a flat interpolation, send it as FormattableString

```csharp
var searchTerm = "magic";
FormattableString query = $@"SELECT * FROM Posts Where Title={searchTerm}";
var posts = blogContext.Posts.FromSql(query)
    .OrderBy(x => x.PostedDate);
```

Now the EF Core debug logs go back to the way they should be, with `@p0` instead of raw string.