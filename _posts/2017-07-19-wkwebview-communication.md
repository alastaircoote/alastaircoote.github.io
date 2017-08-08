---
layout: post
title: Structuring the project, and communicating with a WKWebView
---

I've made some relatively boring progress lately - primarily in splitting the code up properly into discrete projects, so that people will be able to pick and choose their level of integration. Right now I'm expecting it to look like this:

1. **ServiceWorker**: the actual JS environment code executes in. Basically a wrapper around JavascriptCore that provides some of the stuff it doesn't, like `setTimeout`, `fetch` and `importScripts`. It also provides stubs for things like `ServiceWorkerRegistration`, but leaves the implementation up to the user.
2. **ServiceWorkerContainer**: ...which is an implementation of `ServiceWorkerRegistration` and the like. Basically it's the glue that joins service workers together, and provides the bridge to...
3. **SWWebView**: the actual component people will use in their apps. As the name implies, I'm hoping I can get this as near as possible to a drop-in replacement for `WKWebView`, allowing people to augment their existing apps relatively easily. But if they don't already have an app...
4. **I dunno what I'll call it**: A wrapper around SWWebView that will basically create a full navigation stack for an app. This part will be particularly complex as it moves completely outside of the Service Worker API itself.

Anyway. To more immediate concerns

## Communicating with a WKWebView

My initial hack version used `WKScriptMessageHandler` to allow webviews to send messages back to the app, then used `evaluateJavaScript` to send responses back. This had quite a few problems:

- The send and receive are totally disconnected. So I had to manually store unresolved promises in an array and pass the array index through to the native code... then send that index back with the response. The whole thing was kind of a mess, not least because I needed to keep very close track of any errors so that I didn't have an ever-ballooning array.
- It didn't work with `<iframe>`s. `evaluateJavascript` doesn't let you target which frame you want to execute the script in, so it always goes to the top-most frame. From there we'd have to (somehow) find a reference to the correct frame and pass the message down.

However, iOS 11 brings us a new API to play with: [`WKURLSchemeHandler`](https://developer.apple.com/documentation/webkit/wkurlschemehandler?changes=latest_minor). It let us register a custom URL scheme, then receive any incoming requests for that scheme and send whatever we want back. It's actually how we'll power the `fetch` event, too - but it can also serve as a method of communication. Being an HTTP call, we can pair request and response very easily in the browser. And the response will go to whatever frame send the request automatically. I'm sure there is a performance overhead in adding this HTTP/JSON overhead, but we don't use these calls *that* much, so I'm optimistic the switch will be worth it.
