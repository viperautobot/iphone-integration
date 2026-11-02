# iPhone Integration Project

**Status:** ⚠️ Build Blocked — Official Xcodegen Path Fails (2026-04-22 22:29 EDT)
**Objective:** Enable V.I.P.E.R to communicate and interface with an iPhone via OpenClaw iOS app
**Current Blocker:** xcodegen cannot resolve SPM package dependencies (OpenClawKit, Swabble) even with Xcode 16 + Swift 6.2 installed

---

## Overview

Deploy OpenClaw's iOS node on Mr. Shamos's spare iPhone to enable:
- **Real-time messaging** from anywhere without computer access
- **Full visual feedback** via Canvas support on iPhone screen
- **Voice control** with wake detection and talk mode
- **Device capabilities** camera, location, contacts, calendar, reminders, notifications
- **Seamless continuity** between Mac and iPhone interfaces

---

## Current State

### OpenClaw iOS App Status
- **Distribution:** Internal alpha (not on public App Store)
- **Availability Methods:**
  1. **TestFlight Beta** (recommended) - `https://testflight.apple.com/join/cutCZt5s`
  2. **Local build from source** via Xcode (dev only)
  3. **Third-party apps** (GoClaw, ClawOn, Aight, OpenClaw Messenger)

### Key Architecture
- **iPhone as Node:** The iOS app connects to the Gateway (Mac mini) as a **node**, not as a traditional client
- **Gateway Requirement:** Requires an OpenClaw Gateway running on another device (macOS, Linux, Windows)
- **Network Path:**
  - Local LAN via Bonjour (automatic discovery)
  - Tailnet via DNS-SD (cross-network)
  - Manual host/port entry (fallback)

---

## Prerequisites

### Infrastructure
1. ✅ **OpenClaw Gateway** - Already running on Mac mini (192.168.1.218:9443)
   - Verify gateway is running: `openclaw gateway status`
   - Must be accessible from iPhone on same LAN (192.168.1.0/24)

2. **iPhone** - Any model, iOS 15.1+
   - Connected to same Wi-Fi network as Mac mini
   - Apple ID for TestFlight access

3. **Pairing Authority** - Gateway host (Mac mini)
   - Must approve node pairing requests via CLI

---

## Setup Steps

### Phase 1: Build & Deploy iOS App via Xcode (⏱️ 45 minutes first build)

**RECOMMENDED PATH:** Build from source using Xcode

**Full step-by-step guide:** `XCODE_BUILD_AND_DEPLOY.md`

Quick overview:
1. Clone OpenClaw repo
2. Run `pnpm ios:open` (handles dependencies + signing + opens Xcode)
3. Connect iPhone via USB
4. Press Cmd+R in Xcode
5. App builds, signs, installs directly to iPhone
6. App appears on iPhone home screen

**Why this method:**
- ✅ Works with your MacBook Air gateway (direct APNs, no relay needed)
- ✅ Official recommended path
- ✅ Full control and debugging
- ✅ Latest code from main branch

**Other options (NOT recommended):**
- ⚠️ TestFlight - Relay incompatibility with local gateways
- ⚠️ Third-party clients (GoClaw, ClawOn, etc.) - Unsupported, unpredictable

---

### Phase 2: Gateway Discovery & Pairing (⏱️ 5 minutes)

**On iPhone:**
1. Open OpenClaw app
2. Go to **Settings**
3. Under "Gateway Discovery," the Mac mini should appear automatically as:
   - `shamosbot's-macbook-air` (Bonjour)
   - Or manually enter: `192.168.1.218` port `9443` (if manual mode needed)
4. Tap to select gateway
5. Accept TLS certificate prompt (self-signed, expected)
6. **Pairing request will be pending**

**On Gateway Host (Mac mini):**
1. Open terminal
2. List pending requests:
   ```bash
   openclaw devices list
   ```
3. Note the `requestId` for the iPhone pairing
4. Approve it:
   ```bash
   openclaw devices approve <requestId>
   ```
5. Verify connection:
   ```bash
   openclaw nodes status
   openclaw gateway call node.list --params "{}"
   ```

---

### Phase 3: Verify Connection (⏱️ 3 minutes)

**Expected iPhone UI:**
- ✅ Connected indicator in app header
- ✅ Settings shows gateway as "Connected"
- ✅ Chat tab is active and responsive

**From Gateway Terminal:**
```bash
openclaw nodes invoke --node "iPhone" --command canvas.snapshot --params '{"maxWidth":900,"format":"jpeg"}'
```

---

## Capabilities Unlocked

### Chat & Messaging
- **Full OpenClaw chat** on iPhone
- **Canvas rendering** (interactive visuals, charts, UI)
- **Voice input/output** (requires audio permissions)

### Device Actions (Node Commands)
- **Camera:** Snap photos / record video clips
- **Screen:** Screenshot of current app
- **Location:** GPS coordinates (requires location permission)
- **Voice:** Push-to-talk mode, voice wake detection
- **Calendar/Reminders:** Read/integrate with native apps
- **Contacts:** Access contact list
- **Notifications:** Local alerts from agents

### Canvas Features
- **Interactive visuals** rendered in WKWebView on iPhone
- **Drawing/painting** support
- **Navigation** between Canvas states
- **A2UI integration** for rich UI components

---

## Known Limitations & Workarounds

| Issue | Status | Workaround |
|-------|--------|-----------|
| **Foreground only** | Alpha limitation | Keep app in foreground for reliable operation |
| **Background audio** | May be suspended | Voice features are best-effort when app backgrounded |
| **TestFlight lag** | Typical for beta | Official App Store release coming soon |
| **No VPN support** | Design choice | Use Tailnet or manual host if VPN blocks LAN |
| **Permissions hardening** | In progress | Grant all requested permissions for full feature access |

---

## Network Configuration

### Bonjour (LAN) - Default
- ✅ **Automatic discovery** if iPhone & Mac mini on same Wi-Fi
- ✅ No manual config needed
- ✅ Best for home setup

### Tailnet (Cross-Network)
- Requires DNS-SD unicast zone
- Use when Wi-Fi blocked or remote access needed
- Config: `openclaw.internal.` domain with Tailscale split DNS

### Manual Entry
- **Host:** `192.168.1.218` (Mac mini IP)
- **Port:** `9443` (default OpenClaw gateway port)
- **TLS Fingerprint:** Accept prompt on first connect

---

## Next Steps

### Immediate (This Session)
1. ✅ Research complete - **DONE**
2. Mr. Shamos downloads TestFlight on spare iPhone
3. Join OpenClaw TestFlight: https://testflight.apple.com/join/cutCZt5s
4. Install beta build

### Follow-Up (Next Session)
1. Open OpenClaw app on iPhone
2. Discover Mac mini gateway in Settings
3. Approve pairing from Mac mini terminal
4. Test Canvas rendering
5. Test voice features
6. Configure Canvas host URL if needed

---

## Security Notes

- ✅ **TLS 1.3** between iPhone and gateway (self-signed OK for LAN)
- ✅ **Pairing flow** prevents unauthorized node access (requires gateway approval)
- ⚠️ **TestFlight beta** - Treat as internal/alpha, not production stable
- ⚠️ **Local build** - Only use for development; manual APNs auth required for push
- ✅ **Official builds** - Future App Store version uses relay-backed push (more secure)

---

## Resources

- **Official Docs:** https://docs.openclaw.ai/platforms/ios
- **GitHub README:** https://github.com/openclaw/openclaw/blob/main/apps/ios/README.md
- **TestFlight Link:** https://testflight.apple.com/join/cutCZt5s
- **Discord Community:** https://discord.com/invite/clawd

---

## Project Timeline

| Phase | Duration | Status |
|-------|----------|--------|
| Research | 1 hour | ✅ Complete (2026-04-22 21:07 EDT) |
| Download TestFlight | 5 minutes | ⏳ Pending |
| Install iOS App | 10 minutes | ⏳ Pending |
| Gateway Pairing | 5 minutes | ⏳ Pending |
| Connection Verify | 3 minutes | ⏳ Pending |
| Canvas Test | 10 minutes | ⏳ Pending |
| **Total Time Estimate** | **~30 minutes** | - |

---

**Last Updated:** 2026-04-22 21:07 EDT
**Author:** V.I.P.E.R
**Project Owner:** Mr. Shamos
