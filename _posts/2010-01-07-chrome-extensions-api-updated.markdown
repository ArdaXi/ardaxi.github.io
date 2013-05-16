---
author: admin
comments: true
date: 2010-01-07 08:17:37
layout: post
slug: chrome-extensions-api-updated
title: Chrome Extensions API updated
wordpress_id: 59
tags:
- AdSweep
---

It's been quite some time since I've had time to review the whole Chrome API again. As it turns out, it's been greatly updated, to the point where we can abandon the whole content-script idea. The main features I'm interested in are insertCSS and executeScript. These let extensions inject CSS and Javascript into a page from the background page. So, now, there's a background script that waits for pages to start loading, which is when it'll download rules to hide ads. It also caches rules in the extension's database with an option to download all rules, which should make the experience a lot smoother. I'm putting together a beta ASAP.

<!-- more -->Basically, it's the same thing as what I announced a few days ago, only this is even easier. I also plan on making a normal one-page script for any Opera/Firefox users, this will take a bit longer though and I'm not sure I can cache...
