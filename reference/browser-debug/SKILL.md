---
name: browser-debug
description: Control the user's Windows Chrome browser from a Linux VM for frontend testing and debugging, using agent-browser CLI over Chrome DevTools Protocol. Use this skill whenever you need to test UI changes, verify visual fixes, take screenshots of the running app, interact with browser elements, check console errors, or debug frontend issues. Also use when setting up browser debugging for a new project or VM. Trigger on phrases like "check the UI", "take a screenshot", "what does the page look like", "test this in the browser", "debug the frontend", "verify my changes visually", "navigate to the page", or any task that requires seeing or interacting with a web page from a headless Linux environment. When you make frontend changes, proactively use this to verify your work looks correct.
---

# Browser Debug

Control the user's Windows Chrome from this Linux VM using `agent-browser` CLI and Chrome DevTools Protocol (CDP) over an SSH reverse tunnel on port 9555.

The user browses their app normally on Windows. You can navigate, screenshot, click, type, and inspect the same browser - seeing exactly what they see.

## Before You Start: Check Connection

```bash
curl -s --max-time 3 http://localhost:9555/json/version
```

- **JSON response**: Connected. Skip to "Commands" below.
- **Connection refused or timeout**: The tunnel is down or setup is needed.
  - Check if agent-browser is installed: `which agent-browser`
  - If installed, ask the user to launch "Chrome Dev" and "Dev Tunnel" from their Windows Start Menu (these are one-click shortcuts - they'll know what you mean if they've set this up before).
  - If not installed, follow "First-Time Setup" at the bottom of this file.

## Commands

All commands below assume a project-level `agent-browser.json` exists with `{"cdp": "9555"}`. If not, append `--cdp 9555` to each command.

### Navigate

```bash
agent-browser open http://localhost:3000/some/page
```

Works with any port - localhost:3000, :3001, :4200, etc. CDP controls Chrome itself, not a specific tab.

### Snapshot (accessibility tree)

```bash
agent-browser snapshot -i
```

Returns a lightweight text representation with interactive element refs (`@e1`, `@e2`, etc.). Use these refs in click/fill/type commands. This is the primary way to "see" the page without spending tokens on a screenshot.

### Screenshot

```bash
agent-browser screenshot /tmp/page.png
```

Then `Read /tmp/page.png` to view the image. For annotated screenshots with numbered element labels:

```bash
agent-browser screenshot --annotate /tmp/annotated.png
```

### Click, Type, Fill

```bash
agent-browser click @e5          # Click element by ref
agent-browser fill @e3 "text"    # Clear field, then type
agent-browser type @e3 "text"    # Append text without clearing
agent-browser hover @e1          # Hover over element
agent-browser press Enter        # Press keyboard key
agent-browser dblclick @e1       # Double-click
agent-browser drag @e1 @e2       # Drag and drop
```

### Console and Network

```bash
agent-browser console            # Check console errors/logs
```

### Tab Management

```bash
curl -s http://localhost:9555/json  # List all open tabs with URLs
```

### Re-snapshot After Changes

Element refs become stale after any DOM change. Always re-run `agent-browser snapshot -i` after:

- Navigating to a new page
- Clicking something that changes the DOM (modals, dropdowns, navigation)
- Waiting for dynamic content or hot-reload

## Debugging Workflow

When verifying a frontend change:

1. **Navigate** to the relevant page
2. **Snapshot** to understand the page structure
3. **Screenshot** if you need to verify visual layout/styling
4. **Interact** (click, fill forms) to test behavior
5. **Console** to check for errors
6. After code changes, Vite hot-reloads automatically - wait a moment, then snapshot/screenshot again to verify

When the user reports a UI issue:

1. **Navigate** to the page they describe
2. **Screenshot** to see what they see
3. **Snapshot** to find the element's accessible name/role
4. **Grep** the codebase for that component text or role
5. **Fix** the code
6. **Screenshot** again to confirm the fix

## First-Time Setup

Only needed once per VM. The setup has two sides: this Linux VM (you handle it) and the user's Windows machine (give them instructions or a prompt for their Windows Claude Code).

### Linux VM Side (do this yourself)

#### 1. Install agent-browser

```bash
npm install -g agent-browser
```

No need for `agent-browser install` since we connect to the user's Chrome via CDP, not a local browser.

#### 2. Create project config

Create `agent-browser.json` in the project root:

```json
{
  "cdp": "9555"
}
```

#### 3. Verify later

After the user completes Windows setup:

```bash
curl -s --max-time 3 http://localhost:9555/json/version
agent-browser snapshot -i
agent-browser screenshot /tmp/test.png
```

### Windows Side (give these instructions to the user)

The user needs three one-time things on Windows. You can give them this prompt to paste into Claude Code on their Windows machine:

---

**Prompt for Windows Claude Code:**

> Set up browser debugging for remote development. I need:
>
> 1. A Start Menu shortcut called "Dev Session" that launches Chrome with `--remote-debugging-port=9555 --user-data-dir=C:\Users\<my-username>\chrome-debug --remote-allow-origins=*` and then starts an SSH reverse tunnel `ssh -N -R 9555:127.0.0.1:9555 <my-user>@<vm-ip>`. Chrome should launch in the background, the tunnel should keep the window open. If the tunnel drops, pause so I can see the error.
> 2. SSH key auth so the tunnel doesn't need a password. Generate a dedicated key (no passphrase) called `id_ed25519_devtunnel` and copy the public key to the server. Use this key in the tunnel command with `-i`.
> 3. In Cursor settings.json, add: `"remote.portsAttributes": { "9555": { "onAutoForward": "ignore" } }`

Replace `<my-username>`, `<my-user>`, and `<vm-ip>` with the actual values before giving this prompt.

---

### Cursor Setting

If using Cursor or VS Code Remote-SSH, this setting prevents the IDE from intercepting port 9555:

```json
{
  "remote.portsAttributes": {
    "9555": {
      "onAutoForward": "ignore"
    }
  }
}
```

Without this, Cursor detects the port and binds to it locally, blocking Chrome from using it.

## Critical Gotchas

These are hard-won lessons - ignore them at your peril:

- **`127.0.0.1` not `localhost`** in the SSH tunnel flag (`-R 9555:127.0.0.1:9555`). Chrome only listens on IPv4. Using `localhost` may resolve to IPv6 (`::1`), causing silent empty replies from the tunnel.

- **`--remote-allow-origins=*`** is required on Chrome or connections through the SSH tunnel receive empty responses. Chrome's security layer rejects origins that don't look local enough.

- **`--user-data-dir=<path>`** is required or Chrome silently joins an already-running instance and ignores the debug port flag. The user-data-dir path persists cookies, localStorage, and login sessions between restarts.

- **Cursor grabs ports aggressively.** Its auto-forward feature detects listening ports on the VM and binds to them on Windows. The `portsAttributes` ignore setting for port 9555 is essential.

- **Chrome must be fully closed** before relaunching with debug flags. Background Chrome processes (crash handlers, updaters) can prevent a clean start. On Windows: `taskkill /F /IM chrome.exe /T`.

## For Multiple VMs

The Chrome debug port (9555) is Chrome-level, not app-level. One Chrome instance with one debug port gives access to all tabs across all ports. To use this with multiple VMs:

- Each VM gets its own `agent-browser` install and config (same port 9555)
- The SSH tunnel points to one VM at a time
- To switch VMs, the user closes the current tunnel and opens one to the other VM
- Or: use different ports per VM (9555, 9556, etc.) and run multiple tunnels. Update `agent-browser.json` per project.
