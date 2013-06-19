---
layout: post
title:  "Nullmailer on Ubuntu 12.04"
date:   2013-05-26 22:33:58
tags: 
---

# Nullmailer on Ubuntu 12.04

We needed an easy solution to send email from some boxes without going through all the hassle of setting up Postfix, and
found nullmailer to be a good solution.

Here are the steps:

    sudo apt-get install nullmailer
    sudo echo "smtp.mandrillapp.com smtp --port=587 --starttls --user=xxx@example.com --pass=ccc" > /etc/nullmailer/remotes"
    sudo echo example.com > /etc/nullmailer/defaultdomain
    sudo echo "user@example.com" > /etc/nullmailer/adminaddr
    sudo service nullmailer reload
    # To test:
    echo "Test" | sendmail someone@example.com
    tail -f /var/log/mail.log /var/log/mail.err

Unfortuately the default version in Ubuntu 12.04 is Nullmailer 1.05 which doesn't support TLS :(

There was a PPA but now it's down, so I'm just using it without TLS for now.
