---
layout: post
title: "AI Exploration: Part VII - MCP"
date: 2025-5-26
tags: ["ai"]
published: true
---

**Contents**
* TOC
{:toc}

# Problem Statement
Let's build a simple app that uses MCP (Model Context Protocol) in order learn what MCP is and how it works

# International Space Station Tracker
I thought it might be fun to write an app to track the International Space Station (ISS) and say which city the station is closest too. Turns out the ISS is moving around 17500 mph, or 4.861 miles per second.[^1] It circles the earth approximately every 90 mins, so the geo-location will change pretty fast. 

[https://github.com/JacksonKuo/app-mcp-iss](https://github.com/JacksonKuo/app-mcp-iss)

Turns out there an convenient page that publishes ISS updates. [iss-now.json](http://api.open-notify.org/iss-now.json). And we can use the second site to verify our app is working properly: [www.astroviewer.net](https://www.astroviewer.net/iss/en/). 

# Model Context Protocol
The best place to learn about MCP is from [https://modelcontextprotocol.io/](https://modelcontextprotocol.io/). In a nutshell, MCP is a shim server that provides an interface to LLMs. There are server and client examples available for an MCP weather app that is a great starting point to understanding MCP:

* [https://modelcontextprotocol.io/quickstart/server](https://modelcontextprotocol.io/quickstart/server)
* [https://modelcontextprotocol.io/quickstart/client](https://modelcontextprotocol.io/quickstart/client)

The example weather app however leverages Claude Desktop and not OpenAI, which I'll have to refactor to use ChatGPT. The ISS JSON page doesn't require any parameters, which makes setup a bit easier. The python setup uses `astral-sh/uv` package manager[^2]:

```bash
brew install uv
export PATH="$HOME/.local/bin:$PATH"

# don't forget to export OPENAI_API_KEY
uv venv
source .venv/bin/activate
uv add "mcp[cli]" httpx openai
uv init app-mcp-iss

uv run mcp-server-iss.py
uv run mcp-client-responses.py
```

Note that I'm not using OpenAI's Remote MCP because that service can only interact with internet-facing MCP servers.[^3] 

# MCP Server
The server is pretty easy to implement: [https://github.com/JacksonKuo/app-mcp-iss/blob/main/mcp-server-iss.py](https://github.com/JacksonKuo/app-mcp-iss/blob/main/mcp-server-iss.py). A couple of things to note. The transport uses `stdio`, so I don't have to continually run a server, the client will call and instantiate this server at runtime from the CLI. The description comment below `get_position()` is actually required and is the description of the function. The LLM needs this description in order to understand what the function does and ultimately what tools the LLM should call. 

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

# MCP Client
The MCP Client is a bit more complex. First, there is a specific flow for providing ChatGPT with external data:

* `-->` User leverages the MCP Client to call the MCP Server to get list of tools
* `-->` User asks ChatGPT a prompt that includes list of available tools and their descriptions
* `<--` ChatGPT responds by requesting a specific tool be called with parameters
* `-->` User leverages the MCP Client to call the MCP Server tool function
* `<->` MCP Server calls the external data source and passes data back to user
* `-->` User calls ChatGPT with the combined 3 messages containing:
    - Message 1: Original prompt
    - Message 2: ChatGPT requesting for tool call
    - Message 3: Tool call result
* `<--` ChatGPT responds to original prompt using tool call result 

Secondly there are two different methods to fetch external data. The original being Chat Completions API[^4] and the newer being Responses API[^5]. The key differences being slightly different input and output[^6]:
1. Responses output being easier to parse
2. Different checks for call type
    * Responses: `response.output[0].type == "function_call`
    * Completions: `response.choices[0].finish_reason == "tool_calls":`
3. No `tool_choice="auto"` for Responses
    
I have written clients using both, see below:
* [https://github.com/JacksonKuo/app-mcp-iss/blob/main/mcp-client-completion.py](https://github.com/JacksonKuo/app-mcp-iss/blob/main/mcp-client-completion.py)
* [https://github.com/JacksonKuo/app-mcp-iss/blob/main/mcp-client-responses.py](https://github.com/JacksonKuo/app-mcp-iss/blob/main/mcp-client-responses.py)

#### Chat Completions API
```python
response = self.openai.chat.completions.create(
    model="gpt-4o-mini",
    messages=messages,
    tools=available_tools,
    tool_choice="auto"
)
print(response.choices[0].message.content)
```

```json
[{
    "index": 0,
    "message": {
        "role": "assistant",
        "content": "..."
        "refusal": null
    },
    "logprobs": null,
    "finish_reason": "stop"
}]
```

#### Responses API
```python
response = self.openai.responses.create(
    model="gpt-4o-mini",
    input=messages,
    tools=available_tools,
)
print(response.output_text)
```

```json
[{
    "id": "msg_67b73f697ba4819183a15cc17d011509",
    "type": "message",
    "role": "assistant",
    "content": [{
        "type": "output_text",
        "text": "...",
        "annotations": []
    }]
}]
```

# Results
```bash
uv run mcp-client-responses.py

ChatGPT answer: The current geolocation of the ISS is:

Longitude: 165.1391
Latitude: -47.1014

The closest city to this position is likely Auckland, New Zealand, though it is still quite far away.
```

{:refdef: style="text-align: center;"}
![Image]({{ site.baseurl }}/assets/images/mcp-iss.png)
{: refdef}
{:refdef: style="text-align: center;"}
[\[ISS ground track from Astroviewer\]](https://www.astroviewer.net/iss/en/)
{: refdef}

# References
[^1]: [https://spotthestation.nasa.gov/faq.cfm](https://spotthestation.nasa.gov/faq.cfm)

[^2]: [https://github.com/astral-sh/uv](https://github.com/astral-sh/uv)

[^3]: [https://platform.openai.com/docs/guides/tools-remote-mcp](https://platform.openai.com/docs/guides/tools-remote-mcp)

[^4]: [https://platform.openai.com/docs/guides/function-calling?api-mode=chat](https://platform.openai.com/docs/guides/function-calling?api-mode=chat)

[^5]: [https://platform.openai.com/docs/guides/function-calling?api-mode=responses](https://platform.openai.com/docs/guides/function-calling?api-mode=responses)

[^6]: [https://platform.openai.com/docs/guides/responses-vs-chat-completions](https://platform.openai.com/docs/guides/responses-vs-chat-completions)