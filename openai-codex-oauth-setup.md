# Setting Up OpenAI Codex with ChatGPT Plus OAuth in OpenClaw

This guide explains how to install OpenAI Codex CLI in your Docker container and configure OpenClaw to use `gpt-5.3-codex` with your ChatGPT Plus subscription via OAuth.

## Prerequisites

- ChatGPT Plus subscription ($20/month)
- OpenClaw running in a Docker container
- Access to the host machine (to open browser for OAuth)

---

## Part 1: Install Codex CLI in Docker Container

### Step 1.1: Enter the Docker Container

```bash
# Find your container name
docker ps

# Enter the container
docker exec -it openclaw-container-name /bin/bash
```

Or use the OpenClaw exec tool (if exec is configured):

```bash
openclaw exec
```

### Step 1.2: Install Codex CLI

The recommended way is via Homebrew (if available) or direct binary:

```bash
# Option A: Homebrew (if available)
brew install openai-codex-cli

# Option B: Direct binary download (recommended for Docker)
# Download the latest binary for Linux x86_64
cd /tmp
curl -L -o codex https://github.com/openai/codex-cli/releases/latest/download/codex-linux-x86_64

# Make it executable
chmod +x codex

# Move to PATH
sudo mv codex /usr/local/bin/codex

# Verify installation
codex --version
```

### Step 1.3: Authenticate with ChatGPT OAuth

This is the key step — authenticate using your ChatGPT Plus subscription instead of an API key.

```bash
# Start OAuth login (this will output a URL)
codex auth login

# You'll see output like:
# Opening https://auth0.openai.com/authorize?...
```

**Important:** Since you're in Docker, you need to access this URL from your **host machine**:

1. Copy the URL from the output
2. Open it in your host browser
3. Complete the OAuth flow (log into ChatGPT)
4. The token will be cached in `~/.codex/auth.json` inside the container

### Step 1.4: Verify Authentication

```bash
# Check auth status
codex auth status

# Should show: Connected to ChatGPT (Plus)
```

---

## Part 2: Configure OpenClaw to Use Codex

### Step 2.1: Understand OpenClaw Model Configuration

OpenClaw supports custom model providers. You'll need to add Codex as a provider in your `openclaw.json`.

### Step 2.2: Add Codex Provider to OpenClaw Config

Edit your OpenClaw config:

```bash
# Backup current config
cp ~/.openclaw/openclaw.json ~/.openclaw/openclaw.json.backup

# Edit the config
vi ~/.openclaw/openclaw.json
```

Add the Codex provider under `models.providers`:

```json
{
  "models": {
    "mode": "merge",
    "providers": {
      "openai-codex": {
        "baseUrl": "https://api.openai.com/v1",
        "api": "chat_completions",
        "auth": {
          "type": "oauth",
          "tokenFile": "/root/.codex/auth.json",
          "scopes": ["chatgpt"]
        },
        "models": [
          {
            "id": "gpt-5.3-codex",
            "name": "GPT-5.3 Codex",
            "contextWindow": 200000,
            "maxTokens": 100000,
            "reasoning": true,
            "input": ["text"],
            "cost": {
              "input": 0,
              "output": 0
            }
          }
        ]
      }
    }
  }
}
```

### Step 2.3: Alternative — Use Codex CLI Directly via Exec

Since Codex CLI handles OAuth natively, you can also invoke it directly via OpenClaw's exec tool:

```bash
# In your workspace, create a script
cat > /home/node/.openclaw/workspace/scripts/codex.sh << 'EOF'
#!/bin/bash
# Use Codex CLI with OAuth token
codex "$@"
EOF

chmod +x /home/node/.openclaw/workspace/scripts/codex.sh
```

---

## Part 3: Using Codex in OpenClaw

### Method A: Set as Default Model

Update `agents.defaults.model.primary`:

```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "openai-codex/gpt-5.3-codex"
      }
    }
  }
}
```

### Method B: Use via Session Command

In an active session, switch models:

```
/model openai-codex/gpt-5.3-codex
```

### Method C: Spawn Sub-agent with Codex

```bash
# In your session:
@subagent --model openai-codex/gpt-5.3-codex --task "Write a Python script to..."
```

---

## Part 4: Troubleshooting

### Issue: OAuth Callback Blocked

If you see "localhost blocked" error:

```bash
# Codex stores auth at ~/.codex/auth.json
# You can manually copy token from host if needed

# On HOST machine, get token:
cat ~/.codex/auth.json

# Copy to container:
docker exec -it <container> mkdir -p ~/.codex
docker exec -it <container> bash -c 'cat > ~/.codex/auth.json' < ~/.codex/auth.json
```

### Issue: Token Expiration

OAuth tokens refresh automatically. If expired:

```bash
# Re-authenticate
codex auth logout
codex auth login
```

### Issue: Model Not Found

Check available models:

```bash
codex models list
```

---

## Part 5: Cost & Billing

| Auth Method | Cost | Billing |
|-------------|------|---------|
| **ChatGPT OAuth (Plus)** | Included in $20/mo | Uses your Plus quota |
| **API Key** | Pay-per-use | From platform.openai.com |

With ChatGPT OAuth:
- Uses your Plus subscription quota
- No additional API charges
- Limited by Plus rate limits

---

## Summary

1. ✅ Install Codex CLI in Docker: `curl -L -o codex ...`
2. ✅ Authenticate with OAuth: `codex auth login` → open URL in host browser
3. ✅ Configure OpenClaw: Add `openai-codex` provider to `openclaw.json`
4. ✅ Set default model: `"primary": "openai-codex/gpt-5.3-codex"`
5. ✅ Restart OpenClaw: `openclaw gateway restart`

---

## References

- [OpenAI Codex Documentation](https://developers.openai.com/codex)
- [Codex Authentication](https://developers.openai.com/codex/auth/)
- [Codex Models](https://developers.openai.com/codex/models/)
- [OpenClaw Configuration](https://docs.openclaw.ai)
