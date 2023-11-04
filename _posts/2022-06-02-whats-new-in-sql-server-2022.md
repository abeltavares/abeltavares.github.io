---
layout: post
title: "What's new in SQL Server 2022"
description: "Taking a look at some of the new language enhancements coming in SQL Server 2022"
date: 2022-06-02T12:30:00-07:00
tags: T-SQL
image: img/postbanners/2022-06-02-whats-new-in-sql-server-2022.png
---

<style> .hljs { max-height: 1000px; } </style>

> Note: It seems since this post was written, some functions have been added, or some syntax has changed. So until I have time to update this post with the latest information, keep that in mind.

I've been exicted to play with the new features and language enhancements in SQL Server 2022 so I've been keeping an eye on the Microsoft Docker repository for the 2022 image. Well they finally added it! I immediately pulled the image and started playing with it.

I want to focus on the language enhancements as those are the easiest to demonstrate, and I feel that's what you'll be able to take advantage of the quickest after upgrading.

[Here's the official post from Microsoft.](https://docs.microsoft.com/en-us/sql/sql-server/what-s-new-in-sql-server-2022){:target="_blank"}

----

Table of contents:

* TOC
{:toc}

----

## Docker Tag

I won't go into the details of how to set up or use Docker, but you should definitely set aside some time to learn it. You can copy paste the command supplied by Microsoft [on their Docker Hub page for SQL Server](https://hub.docker.com/_/microsoft-mssql-server){:target="_blank"}, but this is the one I prefer to use:

```powershell
docker run -it `
    --name sqlserver `
    -e ACCEPT_EULA='Y' `
    -e MSSQL_SA_PASSWORD='yourStrong(!)Password' `
    -e MSSQL_AGENT_ENABLED='True' `
    -p 1433:1433 `
    mcr.microsoft.com/mssql/server:2022-latest;
```

This sets it up to always use the same name of "sqlserver" for the container, this keeps you from creating multiple SQL server containers. It keeps it in interactive mode so you can watch for system errors, and it starts it up with SQL Agent running. Also, this will automatically download and run the SQL Server image if you don't already have it.

You won't need to worry about loading up any specific databases for this blog post, but if that's something you'd like to learn how to do, [I've blogged about it here]({% post_url 2021-11-04-restore-database-in-docker %}){:target="_blank"}.

----

## GENERATE_SERIES()

[Microsoft Documentation](https://docs.microsoft.com/en-us/sql/t-sql/functions/generate-series-transact-sql){:target="_blank"}

I want to cover this function first so we can use it to help us with building sample data for the rest of this post.

Generating a series of incrementing (or decrementing) values is extremely useful. If you've never used a "tally table" or a "numbers table" plenty of other SQL bloggers have covered it and I highly recommend looking up their posts.

A few uses for tally tables:

* Can often be the solution that avoids reverting to what Jeff Moden likes to call "RBAR"...Row-By-Agonizing-Row. Tally tables can help you perform iterative / incremental tasks without having to build any looping mechanisms. In fact, one of the fastest solutions for splitting strings (prior to `STRING_SPLIT()`) is using a tally table. Up until recently (we'll cover that later), that tally table string split function was still one of the best methods even despite `STRING_SPLIT()` being available.

* Can help you with reporting, such as building a list of dates so that you don't have gaps in your aggregated report that is grouped by day or month. If you group sales by month, but a particular month had no sales you can use the tally table to fill the gaps with "0" sales.

* They're great for helping you generate sample data as you'll see throughout this post.

Prior to this new function, the best way I've seen to generate a tally table is using the CTE method, like so:

```tsql
WITH c1 AS (SELECT x.x FROM (VALUES(1),(1),(1),(1),(1),(1),(1),(1),(1),(1)) x(x))  -- 10
    , c2(x) AS (SELECT 1 FROM c1 x CROSS JOIN c1 y)                                -- 10 * 10
    , c3(x) AS (SELECT 1 FROM c2 x CROSS JOIN c2 y CROSS JOIN c2 z)                -- 100 * 100 * 100
    , c4(rn) AS (SELECT 0 UNION SELECT ROW_NUMBER() OVER (ORDER BY (SELECT 1)) FROM c3)  -- Add zero record, and row numbers
SELECT TOP(1000) x.rn
FROM c4 x
ORDER BY x.rn;
```

This will generate rows with values from 0 to 1,000,000. In this sample, it is using a `TOP(1000)` and an `ORDER BY` to only return the first 1,000 rows (0 - 999). It can be easily modified to generate more or less rows or ranges of rows and it's extremely fast.

Another method I personally figured out while trying to work on a code golf problem was using XML:

```tsql
DECLARE @x xml = REPLICATE(CONVERT(varchar(MAX),'<n/>'), 999); --Table size
WITH c(rn) AS (
    SELECT 0 
    UNION ALL
    SELECT ROW_NUMBER() OVER (ORDER BY (SELECT 1))
    FROM @x.nodes('n') x(n)
)
SELECT c.rn
FROM c;
```

This method is more for fun, and I typically wouldn't use this in a production environment. I'm sure it's plenty stable, I just prefer the CTE method more. This method also returns 1000 records (0 - 999).

Now, in comes the `GENERATE_SERIES()` function. You specify where it starts, where it ends and what to increment by (optional). Though, this is certainly not a direct drop in replacement for the options above and I'll show you why.

```tsql
SELECT [value]
FROM GENERATE_SERIES(START = 0, STOP = 999, STEP = 1);
```

This is pretty awesome, and it definitely beats typing all that other junk from the other options and it's a lot more straight forward and intuitive to read.

I think it's great that you can customize it to increment, decrement, change the range and even change the datatype by supplying decimal values. You can also set the "STEP" size (i.e. only return every N values). I could see this coming in handy for generating date tables. For example, generate a list of dates going back every 30 days or every 14 days.

```tsql
-- List of dates going back every 30 days for 180 days
SELECT DateValue = CONVERT(date, DATEADD(DAY, [value], '2022-06-01'))
FROM GENERATE_SERIES(START = -30, STOP = -180, STEP = -30);

/* Result:
| DateValue  |
|------------|
| 2022-05-02 |
| 2022-04-02 |
| 2022-03-03 |
| 2022-02-01 |
| 2022-01-02 |
| 2021-12-03 |
*/
```

You could certainly do this with the CTE method, it just wouldn't be as obvious as this.

However, I quickly discovered one major caveat...performance. This `GENERATE_SERIES()` function is an absolute pig 🐷. I don't know why it's so slow, maybe they're still working out the kinks, or maybe it will improve in a future update.

Here's how it stacks up on my local machine in docker.

Generating 1,000,001 rows from 0 to 1,000,000:

| Method              | CPU Time (ms) | Elapsed Time (ms) |
|---------------------|---------------|-------------------|
| CTE                 |  1,361        |    500            |
| XML                 |    775        |    784            |
| `GENERATE_SERIES()` | 47,801        | 44,371            |

Unless there's something wrong with my docker image...this doesn't seem ready for prime time. I could see this being used in utility scripts, sample scripts (like this blog post), reporting procs, etc. Where you only need to generate a small set of records, but if you need to generate a large set of records often, it seems you're best sticking with the CTE method for now.

I could possibly see this being useful when used inline where you need to generate a different number of records for each row (e.g. in an `APPLY` operator). However, even then, seeing how slow this is, you might be better off building your own TVF using the CTE method 🤷‍♂️. So while it may be shorter and much easier to use, I'm not sure if the performance trade-off is worth it. Hopefully it's just my machine?

Now that we've got this one out of the way, we can use it to help us with generating sample data for the rest of this post.

----
