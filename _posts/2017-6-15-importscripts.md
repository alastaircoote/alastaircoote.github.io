---
layout: post
title: importScripts()
---

Issue number one. I totally ignored importScripts() functionality when first hacking around with this because it didn't seem necessary, but if I want to end up with something spec-compliant, it's pretty crucial (the Google SW toolbox uses it, for one). importScripts is weird. But after reading the spec and a number of GitHub comments I think I've nailed the functionality down. This is how I'm going to implement it, anyway...

I had assumed `importScripts` was similar to the ES6 `import` command, but it really isn't. For one, it adds whatever you've imported directly into the global scope of the worker - no fancy modules or static dependency trees here. It can also be run at any point in the code - not just at the root level of the file, but inside a function, in response to an event... it's tricky.

It also has an odd relationship with caching - when you importScripts() a file it is added to a special cache collection for each worker and any future loads will go directly to the cache before trying the network (AFAIK it also ignores any headers etc too). In fact, current browser implementations will *never* check those files again, unless the main service worker file is changed. There are lots of hacks out there involving adding file hashes to your worker file and all sorts of nonsense like that, but fortunately browsers are changing their behaviour to [check both the worker JS and imported JS when updating](https://github.com/w3c/ServiceWorker/issues/839). So we'll reflect that, with a process that goes something like:

- Download and store worker JS, along with HTTP headers
- Whenever `importScripts()` is run, store JS and headers for each import
- When updating the worker, first try the worker URL, using `If-None-Match` and `If-Modified-Since`
  - If it's changed:
    - Treat this as a new worker, ignore the old import cache entirely
  - If not changed:
    - Check each import URL with the same HTTP header checks. If any have changed, treat as new worker. If not, end the update.
    
I've been storing the JS contents inside the SQLite database I've been using for all app data (I tried Core Data, but... no), but I'm wondering if it makes more sense to just store them on disk. Again, I'm motivated by minimising memory usage in the Notification Service Extension - if I could remove the SQLite dependency in that environment (which won't be trying to update or anything like that, just start, fire push event, and shut down) that would be great. Right now I'm thinking I might get that extension working before anything else, as it'll inform quite a few of the decisions I'll make further down the line.
