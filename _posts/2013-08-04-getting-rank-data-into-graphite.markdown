---
layout: post
title:  "Getting Rank Data into Graphite"
date:   2013-08-04
tags: ranktracker.io
---

Continuing off my first post about starting ranktracker.io, I'm now at the point where I need to write the Mechanize/Nokogiri crawler that scrapes Google for the rank of the URL I'm interested in, and stores that data into Graphite.

I have a gem that I've just updated to find the rank of a URL on Google; I don't check for the exact URL actually, just the subdomain + domain. I'll add other gems for the remaining search engines later.

Next, I need a worker that uses the gem's functionality to pull the rank and then insert that data into Graphite. Since the worker needs to do this daily, it's gotta be scheduled via something. I think cron is still the best tool for this, and will use the [Whenever gem](https://github.com/javan/whenever) to make sure that the crontab moves with the app.

The worker will be a standalone Ruby program that is distributed amongst multiple machines, and `whenever` will just summon the workers via a Rake task that looks up all the requested tuples for the day and farms out each tuple to a particular machine.

I forgot to mention that I'm going to try using Rails 4 for this project, and make the backend an API as well. Here's what the Rails stack will look like:

* Slim as the template engine
* Postgres as the data store
* Puma as the web server

This is probably a risky move, as I'm not confident that the gems I may need will support Rails 4. However, since I'd like for ranktracker.io to be relatively future-proof, I figure I'd do a bit of extra work upfront in order to save on work down the road. Plus, in the worst case I can help out by upgrading a gem to Rails 4 if I really really need it, and give back to the community. Plus, I'm just really curious to mess around with the new functionality of Rails 4, especially the Russian doll caching and emphasis on making it easier to connect with clientside JavaScript frameworks!

I've finished setting up the above now, and tested it both locally as well as on the distributed network. All looks well, so tomorrow will be focused on retrieving the data from Graphite!
