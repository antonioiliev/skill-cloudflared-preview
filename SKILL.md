---
name: cloudflared-preview
description: "Expose local dev servers with temporary public URLs via cloudflared quick tunnels. Use after building a web app or API to share a live preview link."
metadata: { "openclaw": { "emoji": "🌐" } }
---

# Presenting Builds (Public Preview URLs)

After building a web app or API, expose it with a temporary public URL using `cloudflared`:

## Quick Preview

```bash
# Start the dev server in background
bash pty:true workdir:~/project background:true command:"npm run dev -- --port 3000"

# Create a public tunnel (runs until killed)
bash background:true command:"cloudflared tunnel --url http://localhost:3000"
# Output includes a URL like: https://random-words.trycloudflare.com
```

## Combined Build + Preview

Include tunnel setup in the coding agent prompt:

```bash
bash pty:true workdir:/workspace background:true command:"pi 'Build a todo app with React.

When done, start the dev server on port 3000, then run:
  cloudflared tunnel --url http://localhost:3000
Print the public URL from cloudflared output so I can share it.

When finished, run: openclaw system event --text \"Done: preview URL printed above\" --mode now'"
```

## Notes

- URLs are temporary (`*.trycloudflare.com`) — die when the tunnel process stops
- Free, no account needed, zero config
- One tunnel per port — use different ports for multiple services
- Kill the tunnel session when preview is no longer needed
