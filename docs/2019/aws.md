blog post ideas:
  - git fetch -p
  - git aliases
  - aws
  - net worth predicter

# AWS Post

The official boto3 documentation doesn't explicitly say that all of its calls to
the Athena API are asynchronous. But a good clue to this is the fact that there
is no method named `run_query`. Instead, there's `start_query_execution`. 

At first, when I saw this, I thought, "Hey, they just chose a weird name for the
method you use to run a query in Athena." After all, the documentation for this
method just says:

```
Runs the SQL query statements contained in the Query. Requires you to have
access to the workgroup in which the query ran.
```

About a week of debugging later, though, I realized that it was named that way
because that's exactly what this method does: **start** a query execution. If you
need to run several queries in succession, and each successive query depends on
the last[^1], then you need to figure out some way to have each next query wait
until the previous one finishes before it starts. Otherwise, your code turns
into a giant [race condition](link).

[^1] For example, if you're running multiple DDL scripts...

The way around this is...
