---
layout: post
title: URLSession and GZip
---

Another short entry, just to make a search engine indexable note about something I've discovered. It appears that when using `URLSession` in iOS, there is no way to stop the OS from decompression gzipped content. Normally you'd want it to do this, but in my case I would actually prefer to preserve the gzipped body on disk. Worse, having the gunzipped body means that the `Content-Length` header is no longer accurate, and when you're using SQLite 3's blob streaming functions (which I am, more on that later) you need to know the length upfront.

Long story short, I have no answer on accessing the gzipped body. It doesn't seem to be possible, at least not right now. But there is some good news: `URLSessionTask` has a property named `countOfBytesExpectedToReceive`, which is accurate for gunzipped content. So, in my Fetch implementation I'm now rewriting headers as they are passed to JS - removing the `Content-Encoding` header and rewriting the `Content-Length` one. Not great. But better than nothing.
