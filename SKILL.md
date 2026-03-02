---
name: cloudflared-preview
description: "Expose local dev servers with temporary public URLs via cloudflared quick tunnels. Use after building a web app or API to share a live preview link."
metadata: { "openclaw": { "emoji": "🌐" } }
---

# Presenting Builds (Public Preview URLs)

After building a web app or API, expose it with a temporary public URL using `cloudflared`:

## Step 0: Ensure cloudflared Is Available

Before using cloudflared, check if it's installed and auto-install if missing:

```bash
# Check and install cloudflared if needed
bash command:"if ! command -v cloudflared >/dev/null 2>&1 && [ ! -x /tmp/cloudflared ]; then
  ARCH=$(dpkg --print-architecture 2>/dev/null || case $(uname -m) in x86_64) echo amd64;; aarch64) echo arm64;; armv7l) echo armhf;; i686) echo 386;; *) echo amd64;; esac)
  curl -fsSL \"https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-${ARCH}\" -o /tmp/cloudflared
  chmod +x /tmp/cloudflared
  echo \"cloudflared installed to /tmp/cloudflared (arch: ${ARCH})\"
else
  echo \"cloudflared already available\"
fi"

# Set the path variable for subsequent commands
CLOUDFLARED=$(command -v cloudflared 2>/dev/null || echo /tmp/cloudflared)
```

All subsequent commands should use `$CLOUDFLARED` instead of bare `cloudflared`.

## Quick Preview

```bash
# Start the dev server in background
bash pty:true workdir:~/project background:true command:"npm run dev -- --port 3000"

# Create a public tunnel (runs until killed)
bash background:true command:"CLOUDFLARED=$(command -v cloudflared 2>/dev/null || echo /tmp/cloudflared); $CLOUDFLARED tunnel --url http://localhost:3000"
# Output includes a URL like: https://random-words.trycloudflare.com
```

## Extracting the Public URL

The tunnel URL appears in cloudflared's output. To retrieve it from a background session:

```bash
# Poll the background session for the trycloudflare URL
process action:poll sessionId:<tunnel-session-id>
# Look for a line containing *.trycloudflare.com — that's your public URL
```

## Combined Build + Preview

Include tunnel setup in the coding agent prompt:

```bash
bash pty:true workdir:/workspace background:true command:"pi 'Build a todo app with React.

When done, start the dev server on port 3000, then run:
  CLOUDFLARED=\$(command -v cloudflared 2>/dev/null || echo /tmp/cloudflared)
  if [ ! -x \"\$CLOUDFLARED\" ]; then
    ARCH=\$(dpkg --print-architecture 2>/dev/null || case \$(uname -m) in x86_64) echo amd64;; aarch64) echo arm64;; armv7l) echo armhf;; i686) echo 386;; *) echo amd64;; esac)
    curl -fsSL \"https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-\${ARCH}\" -o /tmp/cloudflared
    chmod +x /tmp/cloudflared
    CLOUDFLARED=/tmp/cloudflared
  fi
  \$CLOUDFLARED tunnel --url http://localhost:3000
Print the public URL from cloudflared output so I can share it.

When finished, run: openclaw system event --text \"Done: preview URL printed above\" --mode now'"
```

## Notes

- URLs are temporary (`*.trycloudflare.com`) — die when the tunnel process stops
- Free, no account needed, zero config
- One tunnel per port — use different ports for multiple services
- Kill the tunnel session when preview is no longer needed
- Architecture detection covers `amd64`, `arm64`, `armhf`, and `386`
- If cloudflared is pre-installed (e.g. in the sandbox image), the auto-install step is skipped
