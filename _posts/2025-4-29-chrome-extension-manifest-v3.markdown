---
layout: post
title: "Chrome Extension - Manifest V3"
date: 2025-4-29
tags: ["browser"]
published: true
---

**Contents**
* TOC
{:toc}

# Problem Statement
Create a Manifest V3 chrome extension that will block certain URLs. Let's use Youtube shorts as a good test case, which will have the added benefit of preventing me getting sucked into the mindless scrolling vortex.

# Manifest Differences
Years ago I wrote a Manifest V2 chrome extension that checks for a `x-java-serialized-object` header[^1]. But with the Manifest V2 phase out in 10-10-2024[^2], let's figure out the differences between V2 and V3[^3] [^4]

| Component | V2 | V3 |
|---|---|---|
| manifest_version | 2 | 3 | 
| webpage scripts | content scripts | content scripts | 
| extension scripts | background scripts | background service workers | 
| permissions | permissions | host permission separated |
| web request listeners | `chrome.webRequest.onBeforeRequest.addListener` | Rules |
| web request permissions | webRequestBlocking | declarativeNetRequest | 

# Shorts Blocker
I could have used UBlock Origin Lite, Tampermonkey, or any of the multitudes of custom extensions, but I'm highly distrustful of chrome extensions and avoid running extensions unless I've personally written the code. 

#### Version 1
The new Manifest V3 uses Rules to block traffic. Google has sample code available[^5]. Unfortunately, the V3 rules are a little finicky for certain websites. For simple sites like `example.com` the result is successful.

```js
[
  {
    "id": 1,
    "priority": 1,
    "action": { "type": "block" },
    "condition": {
      "urlFilter": "||youtube.com*shorts",
      "resourceTypes": ["main_frame", "sub_frame"]
    }
  }
]
```

> **example.com is blocked**  
> This page has been blocked by an extension.  
> Try disabling your extensions.  
> `ERR_BLOCKED_BY_CLIENT`

However when filtering for `https://www.youtube.com/shorts/OkR5f2hElwU` things get kinda weird. The `urlFilter` I use is `||youtube.com*shorts`, however the error is now:

> **This site can’t be reached**  
> The webpage at https://www.youtube.com/shorts/OkR5f2hElwU might be temporarily down or it may have moved permanently to a new web address.  
> `ERR_FAILED`  

An error page is still an error page, I guess. However the bigger issue is that Shorts are still accessible through navigation. Even when `sub_frame` is added to the `resourceTypes`. Youtube does not use regular redirects when clicking Shorts, but instead in SPA fashion updates the DOM and then rewrites the url address to `/shorts/xxx`. This page update does not trigger the extension block rule at all. 

Note that `declarativeNetRequest` does not need `host_permissions` to block a site. `host_permissions` is required when attempting to redirect requests.[^6]

#### Version 2
Okay. So instead of a Rules block, let's try to use a Rules redirect and see if that'll work...

Turns out Rules redirects run into the same problem. Going directly to a `/shorts` page will properly redirect according to the extension. However for SPA the extension isn't able to catch the navigation to Shorts. 

#### Version 3
Let's write a extension that will work. Instead of rules, we'll use a script that run a `redirect.js` script on every `document_start` for only `www.youtube.com`. If the pathname starts with `/shorts/` then redirect to the home page. 

```js
    "host_permissions": ["https://www.youtube.com/*"],
    "content_scripts": [
        {
        "matches": ["*://www.youtube.com/*"],
        "js": ["redirect.js"],
        "run_at": "document_start"
        }
    ]
```

```js
setInterval(() => {
    if (location.pathname.startsWith("/shorts/")) {
        location.href = "https://www.youtube.com/";
    }
}, 500);
```

This version isn't very efficient, as `setInterval` runs every 500 ms on every `www.youtube.com` page. Let's see if we can come up with a more ideal version.

#### Version 4
There is an experimental Navigation API[^7] that was specifically tailored for intercepting SPA navigation events:

> This is a successor to previous web platform features such as the History API and window.location, which solves their shortcomings and is specifically aimed at the needs of single-page applications (SPAs).

Using an event listener should be much more efficient than repeatedly calling JS every 500 ms. Note this method doesn't proc on direct navigation to Shorts URLs, i.e. directly going to `https://www.youtube.com/shorts/OkR5f2hElwU`, but direct navigation is pretty rare and not something I'm super concerned about. 

```js
window.navigation.addEventListener("navigate", (event) => {
    const url = new URL(event.destination.url);
    if (url.pathname.startsWith("/shorts/")) {
        location.href = "https://www.youtube.com/";
    }
})
```

# References
[^1]: [https://github.com/JacksonKuo/java-serial-header-check](https://github.com/JacksonKuo/java-serial-header-check), indicative of a deserialization vulnerability. 

[^2]: [https://blog.chromium.org/2024/05/manifest-v2-phase-out-begins.html](https://blog.chromium.org/2024/05/manifest-v2-phase-out-begins.html)

[^3]: [https://developer.chrome.com/docs/extensions/develop/migrate/checklist](https://developer.chrome.com/docs/extensions/develop/migrate/checklist)

[^4]: [https://developer.chrome.com/docs/extensions/reference/api/declarativeNetRequest](https://developer.chrome.com/docs/extensions/reference/api/declarativeNetRequest)

[^5]: [https://github.com/GoogleChrome/chrome-extensions-samples/tree/main/api-samples/declarativeNetRequest/url-blocker](https://github.com/GoogleChrome/chrome-extensions-samples/tree/main/api-samples/declarativeNetRequest/url-blocker)

[^6]: [https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API/declarativeNetRequest](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API/declarativeNetRequest)

[^7]: [https://developer.mozilla.org/en-US/docs/Web/API/Navigation_API](https://developer.mozilla.org/en-US/docs/Web/API/Navigation_API)