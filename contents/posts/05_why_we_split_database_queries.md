---
title: "Why we split database queries in Elixir"
date: "2026-05-16"
description: "The 65,535 parameter limit, performance cost, and how to pick a chunk size"
---
## The chunk size question
Sooner or later you will run into the advice (or you will reach this conclusion yourself) that a large insert or update should be broken into smaller pieces before it hits the database. In Elixir we usually do it with `Enum.chunk_every/2`:

```elixir
records
|> Enum.chunk_every(1000)
|> Enum.each(&Repo.insert_all(MySchema, &1))
```

The question that always shows up next is: what number do I put in `chunk_every`? People throw around values between 500 and 5000, but where does that range actually come from? Here are the four reasons that shape it.

### First Tip
#### There is a hard PostgreSQL limit at 65,535 parameters per query.
Why?
This one is not a guideline, it is a wall. The PostgreSQL [limits page](https://www.postgresql.org/docs/current/limits.html) lists a maximum of 65,535 query parameters per statement. The number is not arbitrary - it comes from the Bind message in the PostgreSQL wire protocol, which encodes the parameter count as an unsigned 16-bit integer. `2^16 - 1 = 65535`, and that is the magic number. Now translate it into your table: if you insert into a table with 20 columns, one row uses 20 parameters, so you can fit `65000 / 20 = 3250` rows in a single query before the limit catches you. That is a realistic ceiling, and plenty of tables have more than 20 columns. Ecto and Postgrex will raise an error before anything touches the database, so you will not corrupt data - you will just get a failed job at the worst possible moment unless you chunk up front.

### Second Tip
#### Query time scales linearly with parameters, but the constant is not free.
Why?
If you plot insert time against parameter count for 100, 10,000 and 65,000 rows, the line is almost perfectly linear. That is a compliment to how mature PostgreSQL is. The catch is what makes up that constant: parsing the Bind message, planning the statement, shipping the payload over the network, writing it to the WAL, and updating every index on the table. A query with 65k parameters is megabytes of data on the wire and tens of megabytes of WAL on the server side. While that query runs, it holds row locks the whole time, and every other transaction that wants the same rows waits. Smaller chunks mean shorter locks and less contention. Linear scaling does not mean cheap - it means predictable, and you still pay the bill.

### Third Tip
#### Big chunks put pressure on the BEAM, not just on the database.
Why?
Before any of those parameters reach Postgres, they live in your Elixir process as a list of maps or structs. Every parameter is an allocation on the process heap, so a 65k-parameter chunk is a big heap and a big garbage collection bill. If you have an Oban or Broadway pipeline with dozens of workers doing this in parallel, you are multiplying that heap by the worker count. The fix is the same `Enum.chunk_every/2` you were going to use anyway, but combined with `Stream.each` or `Stream.chunk_every/2` so each batch is freed before the next one is built:

```elixir
records
|> Stream.chunk_every(1000)
|> Stream.each(&Repo.insert_all(MySchema, &1))
|> Stream.run()
```

The database limit is the hard ceiling, but in practice you will feel BEAM memory pressure long before you hit 65k parameters.

### Fourth Tip
#### Pick the chunk size on purpose, not by copy-paste.
Why?
The 500 to 5000 range that people quote is a useful default, but a number you picked deliberately is always better. A safe starting formula is `floor(50_000 / column_count)`, which leaves a margin under the 65k limit and works for almost any table. Go lower than the default when the table has wide JSONB or text columns, a lot of indexes, or the operation blocks readers in production. Go higher (closer to 5000) when the table is narrow, has few indexes, and you are running a one-off migration in a maintenance window. And then measure - wrap the call in `:timer.tc/1` on your own data, with your own indexes, on your own hardware. A benchmark from someone else's blog is a starting point, not a verdict.
