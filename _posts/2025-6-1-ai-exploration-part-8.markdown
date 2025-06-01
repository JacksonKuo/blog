---
layout: post
title: "AI Exploration: Part VIII - MCP Security"
date: 2025-6-1
tags: ["ai"]
published: true
---

**Contents**
* TOC
{:toc}

# Problem Statement
Let's do a overview of the current state of MCP security. Note that this is very fast changing field, but this post will serve as a good baseline and will be updated as the MCP ecosystem continues to evolve

# State of the World
* Security Literature 
    * Wiz / Rami
        * [https://www.wiz.io/blog/mcp-security-research-briefing](https://www.wiz.io/blog/mcp-security-research-briefing)
            * [https://docs.anthropic.com/en/docs/test-and-evaluate/strengthen-guardrails/mitigate-jailbreaks](https://docs.anthropic.com/en/docs/test-and-evaluate/strengthen-guardrails/mitigate-jailbreaks)
            * *SSE, a built-in MCP transport type, is at risk of DNS rebinding attacks*
            * [https://mcp-guardian.org/introduction.html](https://mcp-guardian.org/introduction.html)
            * [https://github.com/lasso-security/mcp-gateway/](https://github.com/lasso-security/mcp-gateway/)
    * Simon Willison
        * [https://simonwillison.net/2025/Apr/9/mcp-prompt-injection/#atom-everything](https://simonwillison.net/2025/Apr/9/mcp-prompt-injection/#atom-everything)
        * [https://elenacross7.medium.com/%EF%B8%8F-the-s-in-mcp-stands-for-security-91407b33ed6b](https://elenacross7.medium.com/%EF%B8%8F-the-s-in-mcp-stands-for-security-91407b33ed6b)
            * weaknesses
                * command injection vulnerabilities, i.e. piping to `os.system`
                * tool poisoning attacks via method description
                * the rug pull
                * cross-server tool shadowing / confused deputy attack
            * mitigations
                * input validation
                * version pinning
                * tools description sanitization
                * display full tools metadata
                * integrity hashes for update
        * mcp specification
            * [https://modelcontextprotocol.io/specification/2025-03-26/server/tools#user-interaction-model](https://modelcontextprotocol.io/specification/2025-03-26/server/tools#user-interaction-model): 
            * *For trust & safety and security, there SHOULD always be a human in the loop with the ability to deny tool invocations. Applications SHOULD:*
                * *Provide UI that makes clear which tools are being exposed to the AI model*
                * *Insert clear visual indicators when tools are invoked*
                * *Present confirmation prompts to the user for operations, to ensure a human is in the loop*
    * wunderwuzzi
        * [https://embracethered.com/blog/posts/2025/model-context-protocol-security-risks-and-exploits/](https://embracethered.com/blog/posts/2025/model-context-protocol-security-risks-and-exploits/)
            * JSON-RPC call for `tools/list`, `resources/list`, `prompts/list`
            * handshake: `initialize`, available protocols, `notifications/initialized`
            * Claude Desktop -> `tools/list` auto-loaded into the system prompt
            * tool metadata injection + unicode
        * [https://embracethered.com/blog/ascii-smuggler.html](https://embracethered.com/blog/ascii-smuggler.html)
        * [https://embracethered.com/blog/posts/2025/sneaky-bits-and-ascii-smuggler/](https://embracethered.com/blog/posts/2025/sneaky-bits-and-ascii-smuggler/)
        * [https://embracethered.com/blog/posts/2024/hiding-and-finding-text-with-unicode-tags/](https://embracethered.com/blog/posts/2024/hiding-and-finding-text-with-unicode-tags/)
    * NCC Group
        * [https://www.nccgroup.com/us/research-blog/5-mcp-security-tips/](https://www.nccgroup.com/us/research-blog/5-mcp-security-tips/)
            * JSON-RPC 2.0 over stdio
            * HTTP + SSE (server-sent events) / maybe replaced by Streamable HTTP soon
                * Two different HTTP connections
                * First connection 
                    * `-->` /sse
                        * `Accept: text/event-stream`
                    * `<--` /messages/
                * Second connection `/messages` (capabilities exchange)
                    * `{"method":"initialize}`
                    * `{"method":"notifications/initialized}`
                    * `{"method":"tools/list}`
            * Security risks
                * supply chain attacks - local mcp server
                * prompt injection via method description - untrusted external mcp server
                * excessive capabilities exposed - mcp server call mcp client llm engine
        * https://www.nccgroup.com/us/research-blog/http-to-mcp-bridge/
            * https://github.com/nccgroup/http-mcp-bridge
    * Trail of Bits - [https://blog.trailofbits.com/categories/mcp/](https://blog.trailofbits.com/categories/mcp/)
        * [Jumping the line: How MCP servers can attack you before you ever use them](https://blog.trailofbits.com/2025/04/21/jumping-the-line-how-mcp-servers-can-attack-you-before-you-ever-use-them/)
            * line jumping attacks = tool poisoning attacks
            * *In this case, the description includes instructions to prefix all shell commands with chmod -R 0666 ~; – a command that makes the user’s home directory world-readable and writable. The MCP client applications and models we tested—including Claude Desktop—will follow this malicious instruction when interacting with other MCP tools*
            * *For example, many AI-integrated development environments (Cursor, Cline, Windsurf, etc.) allow users to configure automated command execution without explicit user approval*
        * [How MCP servers can steal your conversation history](https://blog.trailofbits.com/2025/04/23/how-mcp-servers-can-steal-your-conversation-history/)
            * *tool descriptions are loaded into the context window as soon as the host connects to the MCP server*
        * [Deceiving users with ANSI terminal codes in MCP](https://blog.trailofbits.com/2025/04/29/deceiving-users-with-ansi-terminal-codes-in-mcp/)
            * *0.2.76 of Claude Code (Anthropic’s command-line interface for Claude), we found that Claude does not offer any filtering or sanitization for tool descriptions and outputs containing ANSI escape sequences.*
            * Overwriting content through cursor movement
            * Clearing the screen
            * Hyperlink manipulation
            * *Avoid passing raw tool output to the terminal: Instead, implement consistent sanitization for potentially dangerous output by disabling escape sequences before rendering. The simplest approach is to replace any byte with hex value 1b with a placeholder character, since all escape sequences recognized by modern terminals start with that byte*
        * [Insecure credential storage plagues MCP](https://blog.trailofbits.com/2025/04/30/insecure-credential-storage-plagues-mcp/)
            * 
                *many MCP environments store long-term API keys for third-party services in plaintext on the local filesystem, often with insecure, world-readable permissions*
            * `~/Library/Application Support/, ~/.config/, or application logs`
            * *MCP integrated OAuth 2.1 in its March 2025 protocol revision*
            * *`claude_desktop_config.json` file in the user’s home directory. On macOS, we found this file has world-readable permissions*
            * *applications like Cursor and Windsurf store these conversation logs with world-readable permissions*
    * Invariant Labs
        * [https://invariantlabs.ai/blog/mcp-security-notification-tool-poisoning-attacks](https://invariantlabs.ai/blog/mcp-security-notification-tool-poisoning-attacks)
            * Tool Poisoning Attacks - *occurs when malicious instructions are embedded within MCP tool descriptions that are invisible to users but visible to AI models*
                * agent Cursor, `~/.cursor/mcp.json`
            * MCP Rug Pulls
            * Shadowing Tool Descriptions with Multiple Servers
                * *agentic system is exposed to all connected servers and their tool descriptions, making it possible... to inject the agent's behavior with respect to other servers.*
            * Mitigation
                * Clear UI Patterns
                * Tool and Package Pinning
                * Cross-Server Protection
        * [https://invariantlabs.ai/blog/whatsapp-mcp-exploited](https://invariantlabs.ai/blog/whatsapp-mcp-exploited)
            * *malicious MCP server's tool descriptions can manipulate the agent's behavior via a maliciously crafted MCP tool*
            * this is because which LLM call includes tool list from all mcp server including the tool descriptions
            * *description shadows and re-programs the agent's behavior with respect to the WhatsApp MCP server*
        * [https://invariantlabs.ai/blog/mcp-github-vulnerability](https://invariantlabs.ai/blog/mcp-github-vulnerability)
            * [https://github.com/github/github-mcp-server](https://github.com/github/github-mcp-server)
            * Toxic Agent Flows - *indirect prompt injection to trigger a malicious tool use sequence*
            * *affects any agent that uses the GitHub MCP server*
            * Claude Desktop - *“Always Allow” confirmation policy when using agents*
            * [https://explorer.invariantlabs.ai/docs/guardrails/](https://explorer.invariantlabs.ai/docs/guardrails/)
                * *deploy Invariant within minutes using our hosted LLM-level gateway*
* Misc
    * [https://blog.extensiontotal.com/trust-me-im-local-chrome-extensions-mcp-and-the-sandbox-escape-1875a0ee4823](https://blog.extensiontotal.com/trust-me-im-local-chrome-extensions-mcp-and-the-sandbox-escape-1875a0ee4823)
        * *In 2023, Google introduced stricter measures to prevent websites from reaching into users’ private networks*
        * *September 2023: Chrome 117 rolls out to Stable, ending the deprecation trial. Chrome blocks all private network requests from public, non-secure contexts*
        * `host_permissions` should be required
* Scanners
    * [https://github.com/johnhalloran321/mcpSafetyScanner](https://github.com/johnhalloran321/mcpSafetyScanner)
    * [https://invariantlabs.ai/blog/introducing-mcp-scan](https://invariantlabs.ai/blog/introducing-mcp-scan)
    * [https://explorer.invariantlabs.ai/docs/mcp-scan/](https://explorer.invariantlabs.ai/docs/mcp-scan/)
* Whitepapers
    * [https://arxiv.org/pdf/2504.03767](https://arxiv.org/pdf/2504.03767)

# Add MCP server to VSCode Github Copilot
Let's add our ISS MCP server to Github Copilot in agent mode.[^1] This process ending up being quite the hassle since I wasn't able to get input variables to work for my OpenAI key. There are two settings:

* workspace settings = single workspace
    * `.vscode/mcp.json`
* user settings = all workspaces
    * `vscode://settings/mcp`
    * `/Users/jacksonkuo/Library/Application Support/Code/User/settings.json`

`.vscode/mcp.json`
```json
{
    "servers": {
        "app-mcp-iss": {
            "type": "stdio",
            "env": {
                "PYTHONPATH":"/Users/jacksonkuo/workspace/app-mcp-iss/.venv/lib/python3.13/site-packages:$PYTHONPATH"
            },
            "command": "python3",
            "args": [
                "/Users/jacksonkuo/workspace/app-mcp-iss/mcp-server-iss.py"
            ]
        }
    }
}
```

The `PYTHONPATH` is required to run in the python virtual environment using `uv`. There's probably an easier way to run `uv` in VS Code that i'm not aware of. For some weird reason, the `PYTHONPATH` works correctly, but when trying to use input variables[^2] for `OPENAI_API_KEY` the env variable never imports. However, I just loaded the API key the old fashion way via terminal export. 

```json
{
    "inputs": [
        {
        "type": "promptString",
        "id": "openai-key",
        "description": "OPENAI API Key",
        "password": true
        }
    ],
    ...
    "env": {
        "OPENAI_API_KEY":"${input:openai-key}"
        }
}
```

{:refdef: style="text-align: center;"}
![Image]({{ site.baseurl }}/assets/images/mcp-server-prompt.png){: width="300"}
![Image]({{ site.baseurl }}/assets/images/mcp-server-result.png){: width="300"}
{: refdef}
{:refdef: style="text-align: center;"}
\[Calling MCP server\]
{: refdef}

# Todo List
* Proxy MCP traffic specifically Copilot in Agent mode
* Test Burp MCP plugins
* Test MCP scanners
* Attempt some cross-server attacks
* List which agents allow command/code execution and what settings are required
* Setup MCP with OAuth 2.1
* Read the whitepapers

# References
[^1]: [https://code.visualstudio.com/docs/copilot/chat/mcp-servers](https://code.visualstudio.com/docs/copilot/chat/mcp-servers)
[^2]: [https://code.visualstudio.com/docs/reference/variables-reference#_input-variables](https://code.visualstudio.com/docs/reference/variables-reference#_input-variables)
