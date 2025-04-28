---
layout: post
title: "Browser Extension: URL Blocker"
date: 2025-4-29
tags: ["browser"]
published: false
---

**Contents**
* TOC
{:toc}

# Problem Statement
Create a chrome V3 Manifest extension that will block certain URLs. Let's use YT shorts as a good test case, which will have the added benefit of preventing me getting sucked into the mindless scrolling vortex.

# Manifest Differences
Years ago I wrote a V2 Manifest chrome extension that checks for a `x-java-serialized-object` header[^1]. But with V2 Manifest phase out in 10-10-2024[^2], what are the main differences between V2 and V3?[^3]

| Component | V2 | V3 |
|---|---|---|
| manifest_version | 2 | 3 | 
| background | script | service workers | 
| permissions | pemissions | host permission seperated |
| web request listeners | `chrome.webRequest...addListener` | rules |
| web request permissions | webRequestBlocking | declarativeNetRequest | 

# URL Blocker
I could have used UBlock Origin Lite or Tampermonkey or any of the multitudes of blockers on there, but I'm highly distrustful of chrome extensions and avoid running extensions unless I've personally written the code. 

V3 Manifest uses Rules to block traffic. There sample code located here: [https://github.com/GoogleChrome/chrome-extensions-samples/tree/main/api-samples/declarativeNetRequest/url-blocker](https://github.com/GoogleChrome/chrome-extensions-samples/tree/main/api-samples/declarativeNetRequest/url-blocker). 

Unfortunately, the V3 rules 


# Permissions

host_permissions


# References

[^1]: [https://github.com/JacksonKuo/java-serial-header-check](https://github.com/JacksonKuo/java-serial-header-check), indicative of a deseralization vulnerabilty. 

[^2]: [https://blog.chromium.org/2024/05/manifest-v2-phase-out-begins.html](https://blog.chromium.org/2024/05/manifest-v2-phase-out-begins.html)

[^3]: [https://developer.chrome.com/docs/extensions/develop/migrate/checklist](https://developer.chrome.com/docs/extensions/develop/migrate/checklist)

[^4]: [https://developer.chrome.com/docs/extensions/reference/api/declarativeNetRequest](https://developer.chrome.com/docs/extensions/reference/api/declarativeNetRequest)

