---
layout: post
title:  "Starting ranktracker.io"
date:   2013-08-02
tags: ranktracker.io
---

I should have done this a long time ago.

One of my projects is in need of a Google rank tracker for SEO, and I can't really justify the monthly ongoing costs for the keyword volume that we need to check since it would be upwards of $1000 per month. As a result, I've decided that it is time to buckle down and create my own rank tracking service; not only will it save on monthly costs, but it can also be offered as a SaaS product upsell / addon for other customers, not to mention the existing market. Lastly, it just sounds FUN - I've been wanting to do this for awhile now, but didn't have the business justification to do so. 

Now that the stars have aligned, I've decided to open up and be quite transparent about how I'm building the application. This will help serve as both a work journal of sorts so I can document the decisions I've made along with their reasons, and also hopefully help some folks out who are looking to do similar things.

## Domain / Branding

I've picked up the domain name ranktracker.io. I chose it because it's short, descriptive, and I love .io domains! The exact keyword match helps with ranking and the .io TLD helps with reputability, since the majority of the services on the .io TLD tend to be very high quality. I intend to maintain if not exceed that standard. :)

## Architecture

I'm looking at the following tech stack:

* **[Mechanize](http://mechanize.rubyforge.org/) + [Nokogiri](http://nokogiri.org/)** for scraping Google
* **[Graphite](http://graphite.wikidot.com/)** for storing the ranking data (since it's time series)
* **Rails** for handling user input and output
* **PostgreSQL** for storing user data (login, plan, tuples)
* **Dashing? Graphene?**: for data visualization

This stack should meet my business requirements, which from a practical perspective consists of just checking every single [keyword, locale, URL] tuple entered by a user and storing the Google rank of that URL for the keyword and locale specified. I don't see a need to keep over a year of data, and I plan on only checking a tuple once per day.

I'm not entirely sure if Graphite is actually the best tool for the job. Since the internal WhisperDB was built to handle time series data, I think it should be a far superior data store for ranking data compared against a standard relational database. Redis might be another choice, but it will require multiple data structures to properly store the time series data, and if there is a ton of volume I'm concerned that its in-memory propery will turn from an advantage into an expensive liability. Graphite does allow me to query the data by just appending `format=json` to the URL, so it should do the trick. As a bonus, since Graphite was made for visualizations I should be able to easily create ranking charts from the data. I'm going to start with Graphite and see what happens.

The only hard decision is the choice of [Dashing](http://shopify.github.io/dashing/) vs [Graphene](http://jondot.github.io/graphene/) for the front end, but I can delay making that choice until later.

> Decision: Put off deciding whether to use Dashing or Graphene until I need a visualization framework.

## Graphite

First things first - I need to set up Graphite, then configure its retention for 1 day frequency and 1 year history. I investigated [Hosted Graphite](https://www.hostedgraphite.com/) which looks good, but their plans don't allow for enough metrics to be cost effective for me in the long run. However, I'll use them in the short term to avoid wasting time seting up a server while testing out my MVP, since there's a possibility that I may not stick with Graphite. Once I make sure that I'm confident with Graphite as the time series data store, I can migrate to my own server. I'm slightly concerned that the Hosted Graphite plans don't allow customizable data periods, but that should be a visualization and not a storage issue since my frequency of one day is way longer than their minimum resolution of 180 seconds.

> Decision: Use Hosted Graphite temporarily until MVP is proven.

## Scraping Google

The next part is getting the data! I'm not going to worry about the "other" search engines at this point, since Google makes up 66+% of the search volume as of June 2013. A Google search result parser is quite easy to write; there are only two difficult parts. One is figuring out whether to include universal search results, such as local / shopping / video / news results that fall outside of the top 10 organic results but still show up on the first page. The other is to avoid being blocked by Google since scraping violates the ToS.

The first issue, whether or not to include universal search results, will most likely be customer driven. Right now I have code that does both, but the code does not distinguish between local vs shopping vs video vs news, and Google will undoubtedly create additional ones (like restaurant reviews) in the future. As a result, I think in the beginning it's easier to just keep it simple and only count the top 10 organic results. Later on I'll most likely modify it to support universal results.

The CSS selector `h3.r > a` will retrieve ALL results including universal, whereas `li[@class='g'] > div.rc > h3.r > a` will only pick up the top 10 organic results. I'll go back and figure out the individual quirks of local vs shopping vs video vs news results later.

> Decision: Just include top 10 organic results for now, (possibly) move to universal search results down the road if customer demand exists. 

For obvious reasons, I'm not going to talk about the second issue of avoiding getting blocked when scraping Google. Needless to say, I plan on being as gentle as possible when scraping Google to make sure that I don't waste too much of their computing resources, hence another reason for just updating daily. A lot of other rank tracking services update multiple times in a day, which I find wasteful - that kind of resolution isn't useful for something like rank tracking. I'll also employ rate limiting and rotate IP addresses via proxies as well.

## Pushing results to Graphite

Since I'm using Mechanize/Nokogiri to scrape, I need a Ruby Graphite client to push the rank of the target URL for a keyword to the Graphite server.

Although I could use Hosted Graphite's nice [HTTP POST API hook](http://docs.hostedgraphite.com/#http-post), obviously that won't work when I switch over to my own server, so that's not in consideration.

I looked at the various Rubygems for Graphite and found out that I probably don't need them. I'm most likely just going to use a bare TCP socket like this:

    require 'socket'

    conn = TCPSocket.new 'carbon.server.tld', 2003
    conn.puts "rankings.#{locale}.#{keyword}.#{url} rank\n" # where rank is an integer
    conn.close

Notice that I put the locale first; this is to make the Graphite interface more usable, since it groups items hiearchically by the path component, which is delineated by dots. [On the documentation](http://graphite.wikidot.com/getting-your-data-into-graphite), they clearly state that "Volatile path components should be kept as deep into the hierarchy as possible", presumably for the purpose of maintaining your sanity.

Unfortunately, my worst fears were confirmed after some Googling around - I was worried that special characters such as spaces, dashes, slashes, and colons would present a problem if they were used in the metric name. It turns out that this is definitely the case. I've summarized my discoveries about [which characters are valid in a Graphite metric name](#) in another post, so you can look there if you need details.

It's getting late now so I'll wrap it up for today and continue tomorrow!
