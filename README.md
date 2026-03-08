# Tool Proxy

OpenAI-compatible local proxy for sitting between OpenCode and NanoGPT.

Current mode is a proxy-managed tool bridge:

- OpenCode still sends normal OpenAI `tools`
- the proxy strips native tool calling before forwarding upstream
- the proxy injects a strict text tool protocol into the prompt
- the model replies in that text protocol
- the proxy converts that reply back into real OpenAI `tool_calls`

This bypasses unreliable native tool calling upstream while preserving the normal tool interface downstream.

## Quick Start

1. Extract the proxy files into a folder.
2. Open a terminal in that folder.
3. Start the proxy:

```powershell
node server.js
```

4. In OpenCode, point the provider base URL to:

```text
http://127.0.0.1:8787
```

5. Keep using your normal NanoGPT API key and model selection in OpenCode.

## Optional Overrides

```powershell
$env:UPSTREAM_BASE_URL = "https://nano-gpt.com/api/v1"
$env:PROXY_HOST = "127.0.0.1"
$env:PROXY_PORT = "8787"
node server.js
```

You only need those environment variables if you want to change the defaults. Normally, `node server.js` is enough.

Health check:

```powershell
Invoke-WebRequest http://127.0.0.1:8787/health
```

## How The Bridge Works

For tool-enabled requests:

1. OpenCode sends `tools` to the proxy.
2. The proxy removes native `tools` before sending upstream.
3. The proxy injects a strict tool protocol into the system prompt.
4. The proxy also appends a short protocol reminder to bridged user turns.
5. The model must answer using one of these marker envelopes:

Tool use:

```text
[[OPENCODE_TOOL]]
{"tool_calls":[{"name":"write","arguments":{"filePath":"a.txt","content":"hello"}}]}
[[/OPENCODE_TOOL]]
```

Final answer:

```text
[[OPENCODE_FINAL]]
done
[[/OPENCODE_FINAL]]
```

6. The proxy parses that envelope and converts it into OpenAI-style `tool_calls` for OpenCode.

## Notes

- Reasoning streams live.
- Tool and final content are buffered until the proxy can classify them safely.
- This means reliability is prioritized over raw token-by-token passthrough for tool turns.
- Requests without `tools` are forwarded normally.
- For bridged tool turns, the proxy caps upstream `temperature` and `top_p` to reduce protocol drift.
- This share version does not write request or stream logs.

## Verification

```powershell
node --check server.js
node selftest.js
```
