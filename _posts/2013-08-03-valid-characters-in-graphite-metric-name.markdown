---
layout: post
title:  "Valid Characters in Graphite Metric Names"
date:   2013-08-03
tags: ranktracker.io
---

When defining metric names in Graphite, you have to be careful not to assign special characters.

This is due to the underlying implementation; Graphite's Whisper backend stores data as flat files in a directory structure specified by the metric name.

For example, a metric name of `server.service.uptime` would be stored in the file uptime.wsp within a directory structure of `server/service/uptime.wsp`.

As a result, any characters that are invalid in a filename such as a slash (/) or space ( ) will present a problem.

Clearly, the dot is a special character because it delineates each metric's path component, but this is an easy fix; just substitute all dots for underscores.

Fortunately for the rest of the special characters, you can just URL encode any metric name with special characters to make it valid for Graphite, and then URL decode it when you need to reconstruct the information.
