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

## Troubleshooting

If you encounter build issues:

1. **Network issues**: Some dependencies may fail to download. Try:
   ```bash
   pnpm install --force
   ```

2. **Canvas bundle failures**: Skip the canvas step:
   ```bash
   SKIP_CANVAS_BUNDLE=1 pnpm build
   ```

3. **TypeScript errors**: Run type checking:
   ```bash
   pnpm tsgo
   ```

## Notes

- The logging is added at `src/agents/pi-embedded-runner/run/attempt.ts:1291-1320`
- Logs are printed to stdout/stderr using `console.log`
- Response chunks are logged as they arrive (streaming mode)
- For complete response logging, you may need to collect all chunks (current implementation logs each chunk individually)