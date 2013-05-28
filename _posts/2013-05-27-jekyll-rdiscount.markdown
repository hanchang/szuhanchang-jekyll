---
layout: post
title: Jekyll with Rdiscount
---

I was having issues with pasting URLs with ampersands directly into my Markdown files in Jekyll because Maruku, the default Markdown parser for Jekyll uses REXML. 

REXML doesn't allow raw ampersands, they need to be escaped into `&amp;` which is a huge pain if you're just copying and pasting a URL. 

Rdiscount thankfully uses Nokogiri instead of REXML to generate HTML from Markdown and as a result doesn't suffer from this problem.

To get Jekyll set up with Rdiscount, just do the following:

    $ gem install rdiscount

    # In _config.yml
    markdown: rdiscount

Reload the jekyll server and you're good to go!
