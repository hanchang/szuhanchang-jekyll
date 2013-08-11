---
layout: post
title:  "Finally time for Ranktracker.io MVC"
date:   2013-08-08
tags: ranktracker.io
---

It's finally time to generate models and controllers! Previously I was working on the infrastructure for getting workers to retrieve data from Google and insert into Graphite; now that all of that is set up and tested to be functional, I'm going to work on the Rails components. I've been putting it off because I wanted to test the unknown variables (workers and Graphite) first, but now that they're ready to go I can get back to the tried and true MVC framework tasks.

## KeywordTuple

First and foremost will be the Keyword model which basically consists of the keyword, the locale / country for the search engine to search in (.com, .ca, .com.mx, .com.au, .co.uk, etc.), and finally the URL target. I'll start with that for now and layer on additional complexity later. As for validations, I'll keep it simple and just make sure each item is present and do some minor formatting checks.

Next will be integration with the rake task that is run daily for checking rank; instead of hardcoding each keyword tuple in an array and iterating as I was before, I'll retrieve every keyword tuple from the database and send to the worker to check rank.

I've disabled editing and updating keyword tuples, since I think it'll be much easier and cleaner to just have users either delete and recreate them if they need to modify something.

I've also written tests to make sure that editing and updating keyword tuples don't work.

## Devise on Rails 4

That was super easy - next step is to layer some additional complexity on top. The first order of business is Devise for authentication, so we can add the concept of users into the system. I'm using the `rails4` branch of Devise since I'm using Rails 4.

    # In Gemfile
    gem 'devise', github: 'plataformatec/devise', branch: 'rails4'

    $ rails generate devise:install

## Bootstrap 3

After testing to make sure that users can sign up, log in, and log out, it's time to add a bit of UI. Integrating Bootstrap 3 was very easy because I chose to use CDNs for loading the CSS and JS of both Bootstrap and jQuery instead of downloading the files and self-hosting. The only downside to this is if I'm on a plane or somewhere without internet access, but if that's the case I'm screwed anyways since I can't look stuff up if I need to either.

Since it's so new, there aren't any Bootstrap 3 themes out yet, but as soon as there are I'll probably pick one up and integrate it with the project. At the moment, I just need something to get me started, so the default will work just fine.

## Scoping out the Competition

I've been checking out competitors' sites to see how they structure the user flow and get sort of a sanity check on how I'm designing my interface.

## Design Decisions

* Since I'm using Bootstrap 3, that means everything will be mobile first. This also means no modals and minimal Javascript so that users on a phone or tablet won't have to worry about the screen size blocking elements in the form (I hate when that happens to me).
* I'm going to keep menus to a minimum and use more breadcrumb-like navigation to ensure a smooth browsing process.

## Assigning Keywords to Users

Now I need to create associations between keywords and users. Obviously in a multi-user system, people will only be interested in the keyword/locale/url tuples that they enter themselves, so a user needs to `has_many` keywords and a keyword needs to `belongs_to` a user.

## Interesting B2B Services

I found these while browsing around my competition, I may also consider using their services so I figured I'd write them down before I forgot:

* qualaroo.com - conversion rate optimization through targeted realtime surveys
* getambassador.com - affiliate / referral program for 
