# Model Request Logging

This document describes how to enable detailed logging of model requests and responses in OpenClaw.

## Changes Made

The following changes have been made to add model request/response logging:

1. **Modified `src/agents/pi-embedded-runner/run/attempt.ts`**:
   - Added logging wrapper around the `streamFn` function that logs:
     - Model provider, ID, base URL, and API type
     - Message count in the context
     - First 2 messages (truncated to 100 characters)
     - Response chunks (truncated to 200 characters)

## Log Output Format

When a model request is made, you'll see output like:

```
[Model Request] {
  provider: 'openai',
  model: 'gpt-4o',
  baseUrl: 'https://api.openai.com/v1',
  api: 'openai-completions',
  messageCount: 5,
  timestamp: '2026-03-09T23:45:00.000Z'
}
[Model Request] message 0: role=user content=Hello...
[Model Request] message 1: role=assistant content=Hi there...
[Model Response] chunk: {"choices":[{"delta":{"content":"Hello"}}]}
```

## Building and Running

### Prerequisites

- Node.js 22+
- pnpm package manager (install via one of the following methods):

  ```bash
  # Using npm (if Node.js is installed)
  npm install -g pnpm
  
  # Using corepack (comes with Node.js 16+)
  corepack enable pnpm
  
  # Using bun
  bun install -g pnpm
  ```

### Build Commands

```bash
# Install dependencies
pnpm install

# Build the project
pnpm build

# If canvas bundling fails, you can skip it:
SKIP_CANVAS_BUNDLE=1 pnpm build
```

### Running with Model Logging

```bash
# Run gateway with model request logging
pnpm openclaw gateway run

# Or use development mode (auto-reload)
pnpm dev gateway run

# For direct execution after build
openclaw gateway run
```

### Environment Variables for Enhanced Logging

You can also enable additional diagnostic logging:

```bash
# Enable cache trace logging
OPENCLAW_CACHE_TRACE=true pnpm openclaw gateway run

# Enable verbose pi-ai logging
DEBUG=pi-ai:* pnpm openclaw gateway run
```

## Verifying Local Version is Running

To confirm you're running the locally compiled version:

1. **Check startup message**: When running `pnpm dev gateway run`, you should see:
   ```
   [OpenClaw] Running LOCAL development version - model request logging enabled
   ```

2. **Check process info**: The local version runs `node scripts/run-node.mjs gateway run`, while global version runs `openclaw gateway run`.

3. **Trigger model request logging**: Send a message through any configured channel (Telegram, Feishu, etc.) or use the OpenAI-compatible API:

   ```bash
   # Enable OpenAI-compatible endpoint in config (~/.openclaw/config.json):
   # {
   #   "gateway": {
   #     "openaiCompletions": {
   #       "enabled": true
   #     }
   #   }
   # }
   
   # Restart gateway, then send a test request:
   curl -X POST http://127.0.0.1:18789/v1/chat/completions \
     -H "Content-Type: application/json" \
     -H "Authorization: Bearer sk-test" \
     -d '{
       "model": "moonshot/kimi-k2.5",
       "messages": [{"role": "user", "content": "Hello"}],
       "max_tokens": 10
     }'
   ```

   You should see `[Model Request]` and `[Model Response]` logs in the gateway console.

## Troubleshooting Common Issues

### Port Conflicts

If you get "address already in use" errors:

```bash
# Check which process is using port 18789
netstat -ano | findstr :18789

# Stop conflicting process (replace PID with actual process ID)
taskkill /F /PID <PID>

# Or run on a different port
pnpm dev gateway run --port 18790
```

### HTTP Connection Refused

The gateway primarily uses WebSocket connections. For HTTP APIs, ensure:
- OpenAI-compatible endpoint is enabled in config
- You're using the correct port (default: 18789)

### No Model Request Logs Appearing

Model request logs only appear when:
1. A model is actually called (e.g., agent responds to a message)
2. The model provider is configured with valid API keys
3. The request passes through the `streamFn` wrapper

To test:
1. Configure a model provider with API key
2. Send a message through a configured channel
3. Check console for `[Model Request]` logs

### Global vs Local Version Conflicts

If you have both globally installed `openclaw` and local development version:

```bash
# Check which version is running
where openclaw
# Global: C:\Users\Username\AppData\Roaming\npm\openclaw
# Local: Uses local openclaw.mjs

# Ensure local version is running
taskkill /f /im node.exe  # Stop all node processes
pnpm dev gateway run
```

## Notes

- The logging is added at `src/agents/pi-embedded-runner/run/attempt.ts:1291-1320`
- Logs are printed to stdout/stderr using `console.log`
- Response chunks are logged as they arrive (streaming mode)
- For complete response logging, you may need to collect all chunks (current implementation logs each chunk individually)
- `localhost` and `127.0.0.1` refer to the same service on Windows