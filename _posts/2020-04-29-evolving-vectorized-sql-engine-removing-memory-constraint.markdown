---
layout: post
title:  "Evolving Vectorized SQL Engine: Removing Memory Constraint"
date:   2020-04-29 23:04:45 -0400
---

In the upcoming CockroachDB 20.1 release, we will finally be removing the experimental flag for
our new Vectorized SQL Execution Engine. Currently, CockroachDB will run the query through the
new vectorized engine using a set of heuristics, such as the size of the returning result and estimated
memory consumption.  Users can also use vectorized engine execute all queries using the following
command in their SQL shell.

```
set vectorize=on;
```

The vectorized execution engine is an interesting piece of technology that we have be building to
significantly speed up SQL queries. In our early benchmarks, the new vectorized engine can be up to
[40x][1] faster than our existing row-at-a-time SQL engine. You can read more about how we built the
vectorized engine more in our previous [blog posts][2].

The new vectorized engine enabled us to build SQL operators that are friendly to modern CPU caching architecture
and branch predictor. These are the keys to how vectorized engine can greatly outperform our
existing row-at-a time engine. However, these improvement does come at a cost. Columnar memory layout
is great when it comes to speeding up the operations, but sometimes it makes it difficult to implement complex
operations such as SQL joins or groupings. Also we need to be careful with our implementations that we don't
take any shortcut, which can cause the vectorized engine to essentially perform like the old row-at-a-time
engine.

When first rolled out the new vectorized engine in CockroachDB 19.2 release. It was considered an experimental
feature at the time due to one key issues: memory constraints. Although most of the SQL operators in vectorized
engine performed streaming operations, some does require buffering inputs in memory. The vectorized engine at
the time did not have any ability to fall back to disk when the buffering operator runs out of memory. This is
a significant road block that prevented us fully turn on the vectorized engine and limiting its use case to the
SQL tables that can fit inside database the memory.

In order to make vectorized engine suitable for all use cases, the SQL execution team at Cockroach Labs laid out plans
to address the memory constraint in [this RFC][3] later last year. The idea was essentially to use a combination
of disk spilling infrastructure and algorithmic improvement to reduce memory buffering, and fall back to disk when
necessary. As of CockroachDB 20.1, we are happy to say that we believe vectorized engine is now suitable for
production usage.

For past four months, I was working as a backend engneering intern with incredibly talented engineers from the
SQL Execution team to unlock the full potential of vectorized engine for the upcoming CockroachDB 20.1 release.
During this time, I focused on improving the algorithm used in `hash aggregation` and `unordered distinct` operators
to reduce memory usage and speed up execution. In this blog post, I will be going over the ideas behind the new
algorithms we implemented and how the new algorithms address the deficiency of the existing algorithms.

As a note, it is strongly encourage to read our previous [blog post][1] before proceed with this blog post.


# Unordered Distinct


# Hash Aggregation



[1]: https://www.cockroachlabs.com/blog/vectorized-hash-joiner
[2]: https://www.cockroachlabs.com/blog/how-we-built-a-vectorized-execution-engine/
[3]: https://github.com/cockroachdb/cockroach/blob/master/docs/RFCS/20191113_vectorized_external_storage.md
