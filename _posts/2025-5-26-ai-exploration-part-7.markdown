---
layout: post
title: "AI Exploration: Part VII - MCP"
date: 2025-5-26
tags: ["ai"]
published: false
---

**Contents**
* TOC
{:toc}

# Problem Statement
Let's build a simple app that uses MCP (Model Context Protocol) in order learn what MCP is and how it works

# MCP App
I thought it might be fun to write an app to track the International Space Station (ISS) and say which city the station is closest too. Turns out the ISS is moving around 17500 mph, or 4.861 miles per second.[^1] It circles the earth approximately every 90 mins, so the geo-location will change pretty fast. 

[https://github.com/JacksonKuo/app-mcp-iss](https://github.com/JacksonKuo/app-mcp-iss)

Turns out there an convenient page that publishes ISS updates. [iss-now.json](http://api.open-notify.org/iss-now.json). And we can use the second site to verify our app is working properly: [www.astroviewer.net](https://www.astroviewer.net/iss/en/). 

#### Model Context Protocol
The best place to learn about MCP is from [https://modelcontextprotocol.io/](https://modelcontextprotocol.io/). In a nutshell, MCP is a shim server that provides an interface to an LLM. There are server and client examples available for a weather app that are a great starting point to understanding MCP:

* [https://modelcontextprotocol.io/quickstart/server](https://modelcontextprotocol.io/quickstart/server)
* [https://modelcontextprotocol.io/quickstart/client](https://modelcontextprotocol.io/quickstart/client)

The examples however focus on using Claude Desktop and not OpenAI, which I'll have to refactor in my app for ChatGPT. The ISS JSON page doesn't require any parameters, which makes setup a bit easier. 

#### MCP Server
The server is pretty easy to implement. A couple of things to note. The transport is `stdio`, so I don't have to continually run a server, the client will call and instantiate this server at runtime. The comment below `get_position()` is actually required and is the description of the method that the LLM will use to decide which tool method to call. 

```python
from typing import Any
import httpx
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("iss")
API_BASE = "http://api.open-notify.org"

async def make_iss_request(url: str) -> dict[str, Any] | None:
    """Make a request to the ISS API with proper error handling."""
    async with httpx.AsyncClient() as client:
        try:
            response = await client.get(url, headers={}, timeout=30.0)
            ...

@mcp.tool()
async def get_position() -> str:
    """Get ISS geolocation.

    Args:

    """
    url = f"{API_BASE}/iss-now.json"
    data = await make_iss_request(url)
    ...
    return data

if __name__ == "__main__":
    print("running")
    mcp.run(transport='stdio')
```

#### MCP Client
The MCP Client is a bit more complex. 

[https://github.com/JacksonKuo/app-mcp-iss/blob/main/mcp-client-completion.py](https://github.com/JacksonKuo/app-mcp-iss/blob/main/mcp-client-completion.py)

# Issues

[https://platform.openai.com/docs/guides/responses-vs-chat-completions](https://platform.openai.com/docs/guides/responses-vs-chat-completions)

[https://platform.openai.com/docs/guides/function-calling?api-mode=responses](https://platform.openai.com/docs/guides/function-calling?api-mode=responses)

[https://platform.openai.com/docs/guides/function-calling?api-mode=chat](https://platform.openai.com/docs/guides/function-calling?api-mode=chat)

[https://platform.openai.com/docs/guides/tools-remote-mcp](https://platform.openai.com/docs/guides/tools-remote-mcp)

# UV

# References
[^1]: [https://spotthestation.nasa.gov/faq.cfm](https://spotthestation.nasa.gov/faq.cfm)