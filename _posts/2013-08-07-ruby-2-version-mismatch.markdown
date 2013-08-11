---
layout: post
title:  "Your Ruby version is 2.0.0, but your Gemfile specified 2.0.0-p195"
date:   2013-08-07
tags: ruby
---

I had an annoying problem recently - I had installed Ruby 2.0.0-p0 when it first came out, and then upgraded to Ruby 2.0.0-p195 shortly thereafter.

In rvm, I set Ruby 2.0.0-p195 as the default Ruby, but for some reason when I specified my Gemfile to use Ruby 2.0.0, it would default to 2.0.0-p0.

This became a problem since whenever I opened a new terminal window, it would select Ruby 2.0.0-p0 instead of p195 as I preferred. As a result, `bundle install` would install new gems to p0 and not p195, which was super annoying.

My first reaction was to add the patch version number to the Gemfile like this:

    source 'https://rubygems.org'
    ruby '2.0.0-p195'

Unfortunately, that raised the following error message: `Your Ruby version is 2.0.0, but your Gemfile specified 2.0.0-p195`

The solution turned out to be really easy - I just deleted Ruby 2.0.0-p0:

    rvm remove 2.0.0-p0

Now my life is much better!
