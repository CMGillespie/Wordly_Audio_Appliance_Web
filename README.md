cat > /Users/chriswork/Documents/CODE/wordly_appliance_V3/README.md << 'EOF'
# Wordly Audio Appliance

Pure-browser Wordly presenter appliance. No server, no Python, no dependencies.
Single `index.html` — runs in any modern browser, designed for Raspberry Pi kiosk deployment.

## How it works

Browser captures mic audio via Web Audio API → resamples to 16kHz mono via linear interpolation → sends exact 1600-frame binary chunks → Wordly `/present` WSS endpoint → live translation delivered back via `/attend` iframe embedded in the streaming screen.

## Deployment

### GitHub Pages (default)
Live at: `https://cmgillespie.github.io/Wordly_Audio_Appliance_Web/`
Push to `main` — Pages rebuilds automatically.

### Raspberry Pi kiosk
Open Chromium in kiosk mode pointing at the Pages URL.
Portrait mode: set hardware rotation in `/boot/config.txt` — the app auto-detects portrait dimensions on load.

### Local development
Open `index.html` directly in any browser. No build step, no server needed.

## Usage

1. Tap the gear icon to open Wordly Session Config
2. Paste a Wordly join link or enter Session ID (ABCD-1234) + passcode manually
3. Select audio input device
4. Optionally set a presenter/room name
5. Optionally arm a schedule (Start / Split / End times — 24hr rolling window)
6. Save & Close
7. Tap START SESSION
8. Use language grid to quick-switch source language
9. MUTE suspends audio cleanly via WSS stop/start (no client-side suppression)
10. SPLIT creates a transcript boundary without interrupting audio
11. LEAVE disconnects this presenter; session continues for attendees
12. END SESSION terminates the session for everyone

## Audio spec

- Sample rate: 16,000 Hz
- Bit depth: 16-bit PCM
- Channels: Mono
- Frame size: exactly 1,600 samples per chunk
- Protocol: binary WebSocket frames

## WSS auth

- NO API key on WSS endpoints
- Session ID + passcode only
- Session ID always normalized to ABCD-1234 format

## Key features

- **Auto-reconnect** — detects unexpected drops, retries at 100ms / 500ms / 1s / 2s / 5s, gives up after 5 attempts
- **ALS tracking** — Auto Language Selection status messages update the active language display in real time
- **Local scheduler** — time-only Start/Split/End triggers, 24hr window, countdown pill with visual escalation
- **Portrait mode** — auto-detected on load; full reflow layout, not a CSS rotation hack
- **Caption screen** — launches `Wordly_iframe_landing` in a popup pre-configured with session ID and active language

## Downstream apps

| App | Relationship |
|-----|-------------|
| Live Event Gateway (`live_event_gateway`) | Same `index.html` + 5 Firebase script tags appended before `</body>` |
| Wordly_iframe_landing | Separate repo — self-healing attend iframe wrapper for room caption displays |
| Costco Hearing Centres | Fork — Option B language tracking (live ALS follow, 3s debounce) |

## WSS gotchas

- Connect response `languageCode` = alphabetically first language in session list — do NOT use for UI state
- Start response `languageCode` = actual configured session language — use this
- Mute = `{"type":"stop"}` / unmute = `{"type":"start",...}` — never suppress audio client-side
- Split = `{"type":"split"}` — undocumented, confirmed working, creates transcript boundary without disconnect
- Disconnect sequence: stop → 200ms → `{"type":"disconnect","end":true/false}` → 300ms → teardown
EOF