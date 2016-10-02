---
layout: post
title: Postgres Best Practices
---

## Optimize your queries for the query cache

PostgreSQL has no cache for result of stable functions. This applies to all non-deterministic functions like NOW() and RANDOM() etc... Since the return result of the function can change, PostgreSQL decides to disable query caching for that query.

**DON'T:**

```ruby
Article.where('created_at >= NOW()')
```

**SHOULD BE**

```ruby
time_now = Time.current
Article.where('created_at >= ?', time_now)
```

## Explain your Select queries

Using the [EXPLAIN](https://www.postgresql.org/docs/9.1/static/sql-explain.html) keyword can give you insight on what PostgreSQL is doing to execute your query.

## Use Index

If there are any columns in your table that you will query by frequently, you should almost always index them.

```sql
ALTER TABLE "users" ADD INDEX ("username");
```

Postgres support: B-tree, R-tree, Hash, SP-GiST and GIN indexing types. B-tree indexing is default type, the most common and fits most common scenarios. We will talk about how to determine what type of index later :D.

## Index on Negative criteria (NOT something or another)

The index will not be used by "negative" criteria, so the below operators will make to slow query if they are used.

```sql
"IS NULL", "!=", "!>", "!<", "NOT", "NOT EXISTS", "NOT IN", "NOT LIKE"
```

## Avoid functions on the LEFT-HAND-SIDE of the operator

Functions are a handy way to provide complex tasks and they can be used both in the SELECT clause and in the WHERE clause. Nevertheless, their application in WHERE clauses may result in major performance issues. Take a look at the following example:

```sql
SELECT email FROM users WHERE DATEDIFF(MONTH, appointment_date, '2015-04-28')  > 0
```

Even if there is an index on the appointment\_date column in the table users, the query will still need to perform a full table scan. This is because we use the DATEDIFF function on the column appointment\_date. The output of the function is evaluated at run time, so the server has to visit all the rows in the table to retrieve the necessary data. To enhance performance, the following change can be made:

```sql
SELECT email FROM users WHERE appointment_date > '2015-04-30'
```

## Avoid wildcard characters at the beginning of a like operator

Whenever possible, avoid using the `LIKE` pattern in the following way

```sql
SELECT * FROM users WHERE name LIKE '%bar%'
```

The use of the `%` wildcard at the beginning of the `LIKE` pattern will prevent the database from using a suitable index if such exists. Since the system doesn’t know what the beginning of the name column is, it will have to perform a full table scan anyway. In many cases, this may slow the query execution. If the query can be rewritten in the following way:

```sql
SELECT * FROM users WHERE name LIKE 'bar%'
```

## 2 comparison operator into 1 criteria query

Example:

```sql
SELECT userid, username FROM user WHERE user_amount <=3000
```

This will cause the SQL statement comparing two times: user_amount <3000 OR user_amount = 3000 so slow queries. It maybe instead of this (if the type of user_amount is integer):

```sql
SELECT userid, username FROM user WHERE user_amount < 3001
```

## Only add Indexes when necessary

It’s tempting to add indexes to every column, however, an index helps speed up SELECT queries and WHERE clauses, but it slows down data input, with UPDATE and INSERT statements. That can hit performance; **only add indexes when necessary**.

## Index and use same column types for joins

## Do not Order by RANDOM()

If you really need random rows out of your results, there are much better ways of doing it. Granted it takes additional code, but you will prevent a bottleneck that gets exponentially worse as your data grows. The problem is, PostgresSQL will have to perform RANDOM() operation (which takes processing power) for every single row in the table before sorting it and giving you just 1 row.

**DON'T**

```ruby
Article.order('RANDOM()').first
=> SELECT  "articles".* FROM "articles"  ORDER BY RANDOM() LIMIT 1
```

**SHOULD BE**

```ruby
article_count = Article.count
random_number = rand(0...article_count)
randome_article = Article.offset(random_number).first
=> SELECT  "articles".* FROM "articles"  ORDER BY "articles"."id" ASC LIMIT 1 OFFSET 22126
```

## Avoid Using "SELECT *" in your Queries

Always specify which columns you need when you are doing your SELECT Query.

## Almost always have an id Field

In every table have an id column that is the PRIMARY KEY, AUTO_INCREMENT and one of the flavors of INT. Also preferably UNSIGNED, since the value can not be negative.

Even if you have a users table that has a unique username field, do not make that your primary key. VARCHAR fields as primary keys are slower. And you will have a better structure in your code by referring to all users with their id's internally.

## Use EXIST over IN for subquery

I highly recommend you to user EXIST & NOT EXIST in subquery for the large table.

[READ MORE](http://stackoverflow.com/a/14194444)

**In short:**

*   Use EXISTS when the subquery results are very large.
*   Use IN when the subquery results are very small.

## Fixed-length (Static) Tables are Faster

`VARCHAR`, `TEXT`, `BLOB` are not fixed-length

Convert a VARCHAR(20) field to a CHAR(20) field, it will always take 20 bytes of space regardless of what is it in.

## Do use vertical partitioning to avoid large data moves

Example:

**Example 1:** You might have a users table that contains home addresses, that do not get read often. You can choose to split your table and store the address info on a separate table. This way your main users table will shrink in size. As you know, smaller tables perform faster.

**Example 2:** You have a "last_login" field in your table. It updates every time a user logs in to the website. But every update on a table causes the query cache for that table to be flushed. You can put that field into another table to keep updates to your users table to a minimum.

But you also need to make sure you don't constantly need to join these 2 tables after the partitioning or you might actually suffer performance decline.

[READ MORE](https://en.wikipedia.org/wiki/Partition_(database))

## Break big query into chunks

When a big query like that is performed, it can lock your tables for a long time, get much memory for the server's available resources and bring your application to a halt. I put together a different script that will perform this query in chunks: 25,000, 50,000, 75,000 and 100,000 rows at a time.

Example:

* Delete query: [READ MORE](https://sqlperformance.com/2013/03/io-subsystem/chunk-deletes)
* In ruby, we can call #find_each, records will be loaded into memory in batches of the given batch size (default batch size is 1000).

```ruby
Article.where("title like 'a%'").find_each { |e| '' }
	Article Load (59.6ms)  SELECT  "articles".* FROM "articles" WHERE (title ilike 'a%')  ORDER BY "articles"."id" ASC LIMIT 1000
	Article Load (39.7ms)  SELECT  "articles".* FROM "articles" WHERE (title ilike 'a%') AND ("articles"."id" > 10558)  ORDER BY "articles"."id" ASC LIMIT 1000
	Article Load (35.1ms)  SELECT  "articles".* FROM "articles" WHERE (title ilike 'a%') AND ("articles"."id" > 21945)  ORDER BY "articles"."id" ASC LIMIT 1000
```

## Avoid NULL if possible

Unless you have a very specific reason to use a NULL value, you should always set your columns as NOT NULL. The performance improvement from changing NULL columns to NOT NULL is usually small, so don’t make it a priority to find and change them on an existing schema unless you know they are causing problems. However, if you’re planning to index columns, avoid making them nullable if possible.

## Smaller is usually better

Smaller data types are usually faster, because they use less space on the disk, in memory, and in the CPU cache. They also generally require fewer CPU cycles to process.

If a table is expected to have very few rows, there is no reason to make the primary key a BIGINT, instead of INTEGER, SMALLINT. If you do not need the time component, use DATE instead of DATETIME.

## Simple is good

Fewer CPU cycles are typically required to process operations on simpler data types. For example, integers are cheaper to compare than characters, because character sets and collations (sorting rules) make character comparisons complicated. Here are two examples:

*   You should store dates and times in PostgreSQL's built-in types instead of as strings
*   You should use integers for IP addresses.

## Optimize sub-queries

Replace a join with a subquery. For example, try this:

**DONT**

```sql
SELECT DISTINCT t1.column1 FROM t1, t2 WHERE t1.column1 = t2.column1;
```

**SHOULD BE**

```sql
SELECT DISTINCT column1 FROM t1 WHERE t1.column1 IN (SELECT column1 FROM t2)
```

[READ MORE EXAMPLES](http://dev.mysql.com/doc/refman/5.7/en/optimizing-subqueries.html)

## Transaction

* Read Committed is the default isolation level in PostgreSQL.
* Choose right isolation level. Read more \([EN](http://highscalability.com/blog/2011/2/10/database-isolation-levels-and-their-effects-on-performance-a.html) | [VI](http://www.sqlviet.com/blog/cac-mc-isolation-level)\).
Translation isolation levels:

Isolation Level | Dirty Read | Nonrepeatable Read | Phantom Read | Serialization Anomaly
----------------|------------|--------------------|--------------|----------------------
Read uncommitted | Allowed, but not in PG | Possible | Possible | Possible
Read committed | Not possible | Possible| Possible | Possible
Repeatable read | Not possible | Not possible | Allowed, but not in PG | Possible
Serializable | Not possible | Not possible | Not possible | Not possible

**Sources:**

1. [http://khanhtran.xyz/mysql-best-practices/](http://khanhtran.xyz/mysql-best-practices/)
2. [https://www.vertabelo.com/blog/technical-articles/5-tips-to-optimize-your-sql-queries](https://www.vertabelo.com/blog/technical-articles/5-tips-to-optimize-your-sql-queries)
3. [http://code.tutsplus.com/tutorials/top-20-mysql-best-practices--net-7855](http://code.tutsplus.com/tutorials/top-20-mysql-best-practices--net-7855)
