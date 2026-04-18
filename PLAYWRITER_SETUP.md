# Playwriter bridge – agent-browser → Windows Chrome

How `agent-browser` on this Linux VM drives Chrome on the Windows work laptop without opening `--remote-debugging-port` on Chrome.

## What this is

```
Windows laptop                                 Linux VM
┌───────────────────────────────┐              ┌──────────────────────────┐
│ Chrome + Playwriter extension │              │ agent-browser --cdp      │
│         │                     │              │   http://127.0.0.1:19988 │
│         ▼ ws://127.0.0.1:19988│              │         │                │
│   playwriter serve (Node)     │              │         ▼                │
│   127.0.0.1:19988 ────────────┼──ssh -R ─────┼──► 127.0.0.1:19988       │
│                               │  (outbound)  │    (tunnel endpoint)     │
└───────────────────────────────┘              └──────────────────────────┘
```

Why this shape: the corporate EDR flagged the old setup on the Chrome `--remote-debugging-port` listener being exposed to a remote host – the classic info-stealer C2 signature. With Playwriter, Chrome is driven through the extension's `chrome.debugger` API. **No debug port is opened on Chrome.** The only thing tunneled is a localhost Node dev server.

Key property: **Chrome never listens on a debug port.** If any step seems to require `--remote-debugging-port`, stop.

Relay must run on the Windows laptop, not the VM: Playwriter's extension hard-rejects any `/extension` WebSocket from a non-`127.0.0.1` client (verified in `cdp-relay.js:996` of the installed package).

## Windows: start / stop

Two PowerShell windows, both stay open.

Window A – relay (binds `127.0.0.1:19988` only):

```powershell
playwriter serve --host 127.0.0.1
```

Window B – SSH reverse tunnel to the VM:

```powershell
ssh -N -R 19988:127.0.0.1:19988 acoote@5.161.94.201
```

Order: relay first, then tunnel.

Healthy signals:
- Window A logs `listening on 127.0.0.1:19988` (or similar) and then logs an incoming `/extension` WebSocket when Chrome connects.
- Window B shows no output (that's `-N` doing its job) and doesn't exit.

To stop: `Ctrl+C` in each window.

Port `19988` is hardcoded in Playwriter – don't change it.

## Windows: Chrome profile + extension

- Dedicated Chrome profile **Playwriter Dev** – do not sign in to anything (no Atlas, no work accounts).
- Install the Playwriter extension in that profile:
  `https://chromewebstore.google.com/detail/playwriter-mcp/jfeammnjpkecdekppnclgkkffahnhfhe`
- Pin the extension so the toolbar icon is visible.

Per-tab enable model: the extension attaches via `chrome.debugger` per tab. **Each tab you want automated must be enabled by clicking the Playwriter icon on that tab.** `chrome://` pages can't be attached to.

## VM: `agent-browser` target

`~/.bashrc` exports the relay endpoint:

```bash
export AGENT_BROWSER_CDP="http://127.0.0.1:19988"
# export AGENT_BROWSER_CDP="http://127.0.0.1:9555"  # OLD reverse-tunnel target – uncomment to flip back
```

Smoke test after Windows-side is up:

```bash
curl -s http://127.0.0.1:19988/json/version  # expect Chrome-style JSON
agent-browser --cdp "$AGENT_BROWSER_CDP" open https://example.com
```

Per-project `agent-browser.json` files should be updated only after the validation matrix below passes. Keep the old value in a comment when flipping.

## Troubleshooting

**Port 19988 busy on the VM.** Usually a dead `ssh -R` from a prior session on the laptop. Check on the VM:

```bash
ss -tlnp 'sport = :19988'
```

If something's bound and it's not the current sshd-forwarded socket, kill the stale ssh on the laptop side and re-run Window B.

**Extension not connecting.** Window A should log an incoming WebSocket when the extension dials. If nothing logs:
- The Playwriter icon needs to be clicked on an `http(s)` tab – `chrome://` and `about:` pages won't attach.
- Confirm the extension is installed in the **Playwriter Dev** profile (not the default work profile).
- Confirm Window A is still bound to `127.0.0.1:19988` on the laptop (`netstat -ano | findstr 19988`).

**`curl /json/version` returns nothing from the VM.** Either Window A isn't running or Window B isn't up. Both must be alive. The laptop must be reachable from itself at `127.0.0.1:19988` first – if `curl http://127.0.0.1:19988/json/version` on the laptop doesn't work, Window A is the issue, not the tunnel.

**Tab attaches but actions fail.** The tab may have been closed / navigated in a way that detached the debugger. Re-click the Playwriter icon to reattach.

## Known-good / known-broken commands

Validated 2026-04-18 against Playwriter/0.1.0 on Windows laptop, Chrome in the Playwriter Dev profile. Test pages: `https://example.com`, `https://httpbin.org/forms/post`.

| Action | Command | Status |
|---|---|---|
| Navigate | `agent-browser --cdp "$AGENT_BROWSER_CDP" open https://example.com` | ✓ worked |
| Screenshot | `agent-browser screenshot /tmp/pw-1.png` | ✓ worked (40 KB PNG) |
| A11y snapshot | `agent-browser snapshot -i` | ✓ worked (tree with `@eN` refs) |
| Click | `agent-browser click @eN` on a link | ✓ worked (navigated to iana.org) |
| Fill input | `agent-browser fill @eN "hello"` then re-snapshot | ✓ worked (value reflected) |
| `eval` | `agent-browser eval "<js>"` | ✓ worked |
| Console logs | `agent-browser console` (read a console.log marker) | ✓ worked |
| Network request | `agent-browser network requests --filter <host>` after a `fetch` | ✓ worked |

### Expected degradations

Inherent to the `chrome.debugger` subset – flag and move on, don't work around:

- `Target.createTarget` – programmatic new windows/contexts unavailable via extension API.
- `Browser.setDownloadBehavior` – no VM-driven laptop download paths.
- File uploads where the path lives on the VM – only laptop-side paths work.
- `Page.printToPDF` – may be unsupported depending on extension permissions.
- Multi-context / multi-tab workflows – single attached target at a time; each tab must be icon-enabled.

## Fallback: Microsoft `@playwright/mcp`

Documented only. Not executed unless Playwriter is abandoned.

Same relay-on-Chrome-host architecture; also exposes a CDP surface. Path:

1. `npm install -g @playwright/mcp` on Windows.
2. Start its server on `127.0.0.1` (check its `--host` / `--port` flags at the time).
3. Use the same `ssh -N -R <port>:127.0.0.1:<port>` shape from the laptop.
4. Update `AGENT_BROWSER_CDP` on the VM to the new port.

If the EDR flags `ssh -R` generically regardless of payload, do not substitute `--remote-debugging-port` back – that was the original trigger. Escalate instead.

## Non-negotiables

- **No `--remote-debugging-port` on Windows Chrome.** Ever.
- **No direct network exposure of Chrome.** Relay binds `127.0.0.1` on the laptop; CDP traffic reaches the VM only via the authenticated SSH channel.
- **Outbound SSH only** (initiated from the laptop).
- **Dedicated Chrome profile** – no work logins in the Playwriter Dev profile.
