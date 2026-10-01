# Wordly Audio Appliance

Pure-browser Wordly presenter appliance. No server, no Python, no dependencies.
Single `index.html`. Runs in any modern browser. Designed for Raspberry Pi kiosk deployment.

Current version: v3.2f. The version shows in the footer of every screen. It is set by `APP_VERSION` in `index.html`.

## How it works

Browser captures mic audio via Web Audio API → resamples to 16kHz mono via linear interpolation → sends exact 1600-frame binary chunks → Wordly `/present` WSS endpoint → live translation delivered back via `/attend` iframe embedded in the streaming screen.

## Deployment

### GitHub Pages (default)
Live at: `https://cmgillespie.github.io/Wordly_Audio_Appliance_Web/`
Push to `main`. Pages rebuilds automatically.

### Raspberry Pi kiosk
Open Chromium in kiosk mode pointing at the Pages URL.
Portrait mode: set hardware rotation in `/boot/config.txt`. The app detects portrait dimensions on load.

### Pi touchscreen with a second monitor
When an HDMI caption screen is connected, map the touchscreen to the Pi display in `~/.config/labwc/rc.xml`:

`<touch deviceName="Goodix Capacitive TouchScreen" mapToOutput="DSI-2" mouseEmulation="yes"/>`

Find the device name in `/proc/bus/input/devices`. Find the output name with `wlr-randr`. Reload labwc with `pkill -HUP labwc`.

### Local development
Run `python3 -m http.server 8000` in the repo folder and open `http://localhost:8000`.

## Usage

1. Tap **App Setup**
2. Paste a Wordly join link, or enter Session ID (ABCD-1234) and passcode manually
3. Check the audio input. On first use, the app picks an external microphone and remembers it. If the saved device is missing, the app picks a new one and saves it
4. Optionally set a presenter or room name
5. Optionally arm a schedule (Start / Split / End times, 24-hour rolling window)
6. Use **Test Connection** and **Orientation** if needed
7. Save & Close
8. Tap START SESSION
9. Use the language grid to quick-switch the source language
10. MUTE suspends audio cleanly via WSS stop/start. The app never suppresses audio on the client
11. SPLIT creates a transcript boundary without interrupting audio
12. LEAVE disconnects this presenter. The session continues for attendees
13. END SESSION terminates the session for everyone

### End Open Session
This button is on the idle screen. Use it when a session was left open with LEAVE.

1. The app asks you to confirm.
2. The app reconnects with the saved Session ID and passcode, then sends the end message.
3. The app gives up after 4 seconds if Wordly is unreachable.

If no session was left open, the app shows "No open session to end."

## Audio spec

- Sample rate: 16,000 Hz
- Bit depth: 16-bit PCM
- Channels: Mono
- Frame size: exactly 1,600 samples per chunk
- Protocol: binary WebSocket frames

## WSS auth

- NO API key on WSS endpoints
- Session ID and passcode only
- Session ID is always normalized to ABCD-1234 format

## Key features

- **Mic auto-pick**: saved device first, then the most likely external microphone. The app never saves the system `default` device by itself
- **Auto-reconnect**: detects unexpected drops, retries at 100ms / 500ms / 1s / 2s / 5s, gives up after 5 attempts
- **ALS tracking**: Auto Language Selection status messages update the active language display in real time
- **Local scheduler**: time-only Start/Split/End triggers, 24-hour window, countdown pill with visual escalation
- **Portrait mode**: detected on load. Full reflow layout, not a CSS rotation
- **Caption screen**: launches `Wordly_iframe_landing` in a popup with the session ID and active language
- **Cloud Control lockout**: appears in App Setup on the LEG build only (`isLegBuild()`)

## Branding

- Colors follow the Wordly Design System dark mode
- Font is Plus Jakarta Sans, loaded from Google Fonts. Without a network, the app falls back to Arial
- The Wordly logo is an inline SVG in `index.html`
- Legal footer and version number show on every screen

## Downstream apps

| App | Relationship |
|-----|-------------|
| Live Event Gateway (`live_event_gateway`) | Same `index.html` + 5 Firebase script tags appended before `</body>` |
| Wordly_iframe_landing | Separate repo. Self-healing attend iframe wrapper for room caption displays |
| Costco Hearing Centres | Fork. Option B language tracking (live ALS follow, 3s debounce) |

## WSS gotchas

- Connect response `languageCode` = alphabetically first language in session list. Do NOT use it for UI state
- Start response `languageCode` = actual configured session language. Use this
- Mute = `{"type":"stop"}` / unmute = `{"type":"start",...}`. Never suppress audio client-side
- Split = `{"type":"split"}`. Undocumented, confirmed working, creates transcript boundary without disconnect
- Disconnect sequence: stop → 200ms → `{"type":"disconnect","end":true/false}` → 300ms → teardown