---
layout: post
title:  "Retrieving Rank Data from Graphite"
date:   2013-08-05
tags: ranktracker.io
---

Now that I've inserted rank data into Graphite, I need to make sure that I can retrieve it and possibly store / cache it somewhere for faster access if Graphite is not performant enough.

However, I just learned that escaping special characters via URL encoding won't be enough when I tested some random keywords for my own projects. Basically, since the metrics are written to disk as a flat file in a directory structure according to the metric name, delineated by dots, special characters such as apostrophes that are valid for URLs will unfortunately break things when used in a filename.

So instead, it looks like the best alternative is to just name the metric after the primary id of the tuple, and retrieve the information from the database. This setback will require additional testing to make sure dependencies are satisfied and performance doesn't suffer, so I'll implement it tomorrow and test as it's getting late.

I'm also thinking of creating a Redis list to store the ranking for the last 30 days of each tuple, with each item in the list representing the rank of the URL for the keyword. As a result, the Redis list will have a maximum of 30 entries, and when that limit is reached I can first `rpop` the old entry out and then `lpush` the new entry in as a manual sort of LRU cache.

    LPUSH rank:engine.locale:keyword RANK
    RPOP rank:engine.locale:keyword if LLEN > 30

As a back of the envelope calculation, this conservatively should take up approximately 250 bytes per tuple, so with just 250 MB of memory I should be able to store 1 million tuples. Since this is all in-memory, it should be super quick. I won't do this unless Graphite lookup performance is bad though, as it adds additional complexity and definitely falls under premature optimization. Regardless, it's nice to know that there's a backup plan in case speed becomes an issue, even if I don't implement it.
