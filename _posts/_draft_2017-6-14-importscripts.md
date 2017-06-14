---
layout: post
title: A development blog.
---

Dilemma number one. I totally ignored importScripts() functionality when first hacking around with this because it didn't seem necessary - I was using a JS bundler anyway, which gave me a single worker.js file. But when I tried to double-back over to add importScripts() later on I created a huge mess, because it's kind of difficult to graft that functionality on later.
