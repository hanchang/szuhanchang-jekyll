---
layout: post
title:  "Rearchitecting Data Flow"
date:   2013-08-06
tags: ranktracker.io
---

Continuing off from yesterday, it looks like using the primary key of the {keyword, locale, URL} tuple to track for the Graphite metric name should work quite nicely. I've tested my worker and cronjob rake file with mock database entries, and will make the actual model / migration today and modify the rake task to actually read from the database to retrieve a list of all the tuples. 

Obviously in the future the list of all tuples will be quite large, so I'll go ahead and use the `find_each` method to retrieve them in batches and thereby reduce memory consumption. It's probably a bit of premature optimization but since I'm confident that this will be required in the future and it's quite trivial to implement, I'm going to go ahead and just do it in this case.

I've also used the `whenever` gem to set up the daily cron job that runs the rake task. The hard part will be making sure to safely and correctly `rescue` any exceptions that pop up; otherwise if a particular keyword raises an error, it will prevent the remaining keywords in the list from running their daily rank check, which is not acceptable.

That said, I'm probably just going to use `whenever` as a way to keep a record of the cronjobs required for the project in source control since I plan on running the app off of Heroku. Heroku's ephemeral filesystem doesn't allow `whenever` to do its thang. Instead, I have to use their Scheduler addon, which is no big deal, but just an important point to remember.

Setting up a testing framework is also important, although I forgot to mention it. I'm using RSpec and Capybara, and only plan on writing unit tests for model along with integration tests to make sure the business logic works as expected. I find that these two test types are usually adequate for making sure things work properly.

Other than that, there's not much else to report. Tomorrow I will start researching and thinking about how to do the API. Basically, I need to decide whether or not to do a frontend app that consumes the API, or use Rails to generate the views and/or JSON responses based on the format (HTML vs JSON). Obviously it isn't a false dilemma in that I can do a bit of both and probably will, but I'm leaning much more strongly towards the latter mostly because that's what I'm familiar with.

In the meanwhile, my local development box is chugging along and checking rank daily on a handful of tuples that I care about, then storing the data into Graphite.

