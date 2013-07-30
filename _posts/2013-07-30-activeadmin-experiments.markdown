---
layout: post
title:  "ActiveAdmin as a Framework"
date:   2013-07-30
tags: 
---

I've been working on a project and been stuck on whether or not to use ActiveAdmin as the site itself (since it's mostly just a CRUD app), so I wanted to write down the pros and cons of each to remind myself of my decision. I'll update the post as things change.

Pros of ActiveAdmin

* Rapid development, can have something up and running quickly; in particular, I need the filtering capabilities of the sidebar and didn't want to write each one by hand, since there are a lot of columns...
* It handles auth for you via Devise already

Cons of ActiveAdmin

* Namespacing is weird, defaults to /admin so all routes will have that prepended
* Customization can be done, but only through a DSL called Arbre
* Will be hard to find additional developers to help out if necessary in the future, since it's not standard Rails

I actually interviewed at a company which started off using ActiveAdmin as the framework for an internal app, but they mentioned they regretted it and were working on porting it back to Rails.

Writing this down helped me make my decision - I'm going to ditch ActiveAdmin and write it myself. I'll probably come up with a DSL for writing the filters anyways, so it'll be fine. If it's abstract enough maybe I can package it into a gem for others...?
