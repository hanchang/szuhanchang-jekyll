---
layout: post
title:  "Switching from Graphite to Redis for Time Series Database"
date:   2013-08-14
tags: ranktracker.io
---

Ouch, I just discovered a bit of bad news - HostedGraphite doesn't allow direct access to the URI API that I need to retrieve metrics on the plan I'm using; I'd need to upgrade to a mid-tier plan at $30/month, so I might consider just setting up my own Graphite server at this point. 

Well, this is what I get for not thoroughly investigating a third-party service before using it. It's not a ton of money, and $30 is probably worth less than the time I'd need to set up my own Graphite server, but I'm not even sure if Graphite is the best choice of time series databases anymore.

I'm looking deeper into [Cube](http://square.github.io/cube/) now. It seems a lot more flexible; with Graphite, if I wanted to increase how often I collected ranking data from daily to hourly, I'd have to completely rehaul the system. With Cube I can just add another event. Also, with Cube I can compute statistics on the database instead of having to grab all the data, then computing it on the server side. I do have a distrust of MongoDB though, which serves as the underlying data store, but I do like the flexibility. 

On the flip side, the biggest loss from the (hypothetical) switch will come from the fact that I no longer have a cap on the disk space required - this might be a bigger deal as time goes on, but for now it's definitely not a big deal; plus storage is cheap now. Another loss is the fact that a LOT of libraries are preconfigured to handle Graphite data out of the box, whereas only Cubism is ready to handle Cube data (and Cubism doesn't fit the needs of ranktracker.io at all since it graphs one data point every pixel).

However, I'm torn between the two solutions, so I guess there's no choice other than to try to them both out and see how it goes... I'll install both stacks on my local dev box, shove data in them, and see how it easy it is to retrieve and manipulate the data. I'll start with Graphite first since that's what I was going to go with originally.

I need to install a local version of Graphite instead of paying the $30 because I need to test the retention policy and resolution. I plan to use a 10 year retention with daily resolution for ranktracker.io, whereas HostedGraphite has just 2 month retention on the smallest plan with API access, but 30 second resolution, which doesn't fit my needs at all so it's not worth sticking with them long term.

## Graphite Installation

Wow, that was painless! I just installed Graphite in about 10 minutes through [this fantastic Gist](https://gist.github.com/jgeurts/3112065). I've configured it for a 1 day, 10 year retention for now. I tried adding a more granular 6 hour, 7 days retention but since Graphite requires that every retention set be proportional, I don't think it will work.

Unfortunately, after testing out Graphite's Render URL API, I've determined that it's not a good fit for the project. In order to display the most recent rankings and their deltas, I'd need to make an HTTP request for each keyword and then parse the response on the client side. Not only will that be quite slow even in parallel, but I really don't like the idea of having most of my business logic sitting around in JavaScript waiting to be copied, not to mention the access control nightmares that I'd have to face when trying to protect my Graphite server.

## Redis to the rescue!

I make it pretty well known that I'm deeply in love with Redis. For most of my purposes it works beautifully, and I think for ranktracker.io, again it shines!

The new plan is to use a Redis List as a round robin database for each keyword; the key will be the keyword's id and the values will be `"timestamp:rank"`. New values will be inserted by the distributed workers into Redis via RPUSH, and I can retrieve the last week or month's rankings via a quick LRANGE and some parsing. I can also then format the data server-side for graphing, and since everything is in-memory it should be super fast.

Obviously the first concern is storage space - how much memory will I need? After a quick back of the envelope calculation, it turns out not too much:

A UNIX epoch timestamp is currently 10 characters long and will continue to be until November 20, 2286. I only check rank up to position 100, so the rank can only take up 3 characters max. The separator colon is an additional character which winds up to be 114 characters for each list item.

Redis stores keys and values as ASCII which requires 1 byte per character, so each entry into the keyword's list is 114 bytes. At a maximum of one year (365 days) per keyword ranking history, that turns out to be 42kb for one keyword per year. I'm not counting the key though, which is another 36 characters if I'm using UUIDs, along with a namespace string and separator, but since most rankings should NOT be the three character maximum I'm going to let it slide. The overhead for the list structure is probably within those bounds as well.

114 * 365 = 42k/keyword/year, so with 1 GB of memory dedicated to Redis I can store a little over 23,000 keyword ranking histories for up to a year. If I need additional storage, I can spin up new Redis clusters as well, so I'm not too worried. Considering an 8 GB VPS on DigitalOcean is only $80/month, capable of handling 184,000+ keywords, I'm really not too worried about scale.

I'll need to check if I've reached the 365 value limit before inserting via LLEN, and if so do an RPOP before LPUSHing the new rank entry in. Thankfully, every single one of those operations is O(1). Goddamn that is sexy. The only operation that is slightly slower is LRANGE which runs in O(S+N), where S is the offset and N is the number of nodes within the range; N will always be at most 365 (if requested via the API) and most commonly will be 30 since I plan on showing the last 30 days / month worth of rankings, so it's effectively constant time as well. Have I mentioned how much I love Redis?
