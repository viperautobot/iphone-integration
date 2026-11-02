# Xcode Build & Deploy to iPhone: Complete Guide

**Source:** Official Apple docs + community-verified 2026 workflow  
**Status:** Ready to execute  
**Time Estimate:** 45 minutes (first build); 2-3 minutes (rebuilds)

---

## Your Critical Question Answered

**"If I'm building in Xcode on the MacBook Air, how does the app get to my iPhone?"**

### The Answer: Direct USB Transfer + Code Signing

1. **Connect iPhone to Mac via USB cable** (Lightning or USB-C depending on iPhone model)
2. **Xcode compiles the app binary** on your MacBook Air
3. **Xcode digitally signs it** with your Apple Developer certificate (proves you created it)
4. **Xcode installs it directly on the iPhone** via USB
5. **Once installed, the app runs on iPhone** independently—no connection to Mac needed after installation

**Key Point:** The app is NOT downloaded from internet. It's built on your Mac and pushed directly to the phone via cable, signed with your developer credentials.

---

## Prerequisites (One-Time Setup)

### 1. Apple Developer Account
- **Required:** Free Apple ID (already have)
- **Cost:** $0/year for development on personal device
- **Signing:** Your account signs the app so Apple knows it came from you

### 2. Xcode 16+
```bash
# Check your current version
xcode-select --version
xcodebuild -version

# Install/update if needed (App Store → Xcode)
```

### 3. Build Tools
```bash
# Install pnpm (package manager)
npm install -g pnpm

# Install xcodegen (generates Xcode project)
pnpm add -g xcodegen

# Verify installation
pnpm --version
xcodegen --version
```

### 4. OpenClaw Source Code
```bash
cd ~/Projects  # or your preferred location
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

---

## Phase 1: Clone, Install, & Configure (15 minutes)

### Step 1.1: Install Dependencies
```bash
cd ~/Projects/openclaw
pnpm install
# This downloads all JavaScript dependencies for the entire OpenClaw project
# Time: ~5 minutes
```

### Step 1.2: Configure iOS Signing
```bash
./scripts/ios-configure-signing.sh
# This script:
# - Detects your Apple development team
# - Creates unique bundle IDs for your personal use
# - Sets up code signing certificates
# Interactive prompts will guide you through it
# Time: ~3 minutes
```

**What It Creates:**
- `apps/ios/.local-signing.xcconfig` — Your personal signing config (generated once, never change)
- Code signing certificate in Xcode (automatic)
- Unique bundle ID like `ai.openclaw.YourDevTeamName` (prevents conflicts)

### Step 1.3: Generate Xcode Project
```bash
cd apps/ios
xcodegen generate
# Generates OpenClaw.xcodeproj from source configuration
# Time: <1 minute
```

### Step 1.4: Open in Xcode
```bash
open OpenClaw.xcodeproj
# Xcode launches with the iOS project ready to build
```

**Or use the shortcut (combines 1.2 + 1.3 + 1.4):**
```bash
cd openclaw
pnpm ios:open
# One command does: signing config + xcodegen + open Xcode
```

---

## Phase 2: Connect iPhone & Configure Xcode (5 minutes)

### Step 2.1: Connect iPhone via USB
1. **Get an Apple USB cable** (Lightning or USB-C, depending on iPhone model)
2. **Plug iPhone into MacBook Air USB port**
3. **On iPhone screen:** A dialog appears asking "Trust This Computer?"
   - Tap **"Trust"**
   - Enter your iPhone passcode (optional second confirmation)

### Step 2.2: Verify iPhone Appears in Xcode

**In Xcode menu bar:**
1. Go: **Window → Devices and Simulators** (or press Shift+Cmd+2)
2. Look for "Connected" section
3. Your iPhone should be listed with its name
   - Example: `Liam's iPhone`
   - Status shows: "Connected"

**If iPhone doesn't appear:**
- ⚠️ Unplug USB, wait 5 seconds, replug
- ⚠️ Try different USB port on Mac
- ⚠️ Restart Xcode
- ⚠️ Try rebooting iPhone

### Step 2.3: Select iPhone as Build Target

In the **Xcode toolbar** (top of window):
1. Find the **Scheme dropdown** (left side, next to play/stop buttons)
2. It currently shows: `OpenClaw` (the app name)
3. Click on the **Target/Device dropdown** (right side)
4. Select your iPhone from the list
   - Example: `Liam's iPhone` (not "Generic iOS Device")

**Should now show:**
- Scheme: `OpenClaw`
- Target: `Liam's iPhone` (your actual device name)

---

## Phase 3: Build & Deploy to iPhone (15-20 minutes)

### Step 3.1: Verify Build Configuration
1. In Xcode, ensure: **Product → Scheme → Edit Scheme**
2. Under "Run" tab:
   - **Configuration:** `Debug` (required, not Release)
   - ✅ Confirmed

### Step 3.2: Build & Install
**Option A: Simple (Recommended)**
```bash
# In Xcode, press: ⌘R (Cmd+R)
# Or click: Product → Run (the play button)
```

**Option B: From Terminal**
```bash
cd ~/Projects/openclaw/apps/ios
xcodebuild -scheme OpenClaw -configuration Debug -destination 'id=<DEVICE_UUID>'
# Note: Find DEVICE_UUID by running: xcodebuild -showBuildSettings
```

**What Happens:**
1. Xcode compiles the app (~10-15 minutes on first build)
2. Code signs it with your developer certificate
3. Pushes binary to iPhone via USB
4. Installs the app on iPhone home screen
5. Launches the app automatically

**Console Output:**
- Green checkmark: ✅ Build succeeded
- App appears on iPhone home screen
- OpenClaw app launches automatically

### Step 3.3: First Launch on iPhone

When the app first launches:
1. iOS may ask for permission requests:
   - **"Allow OpenClaw to access your location?"** → Tap "Allow" (or "Allow While Using")
   - **"Allow OpenClaw to access your camera?"** → Tap "Allow"
   - **"Allow OpenClaw to access Contacts?"** → Tap "Allow"
2. App displays main OpenClaw UI
3. **Settings tab** → Find "Gateway" option

---

## Phase 4: Pair with MacBook Air Gateway (5 minutes)

### Step 4.1: Gateway Auto-Discovery
In the OpenClaw iOS app:
1. Go to **Settings**
2. Under "Gateway Discovery", look for available gateways
3. You should see: `shamosbot's-MacBook-Air` (automatic Bonjour discovery)
4. Tap to select it

### Step 4.2: Approve Pairing on Gateway Host

**On your MacBook Air terminal:**
```bash
# List pending pairing requests
openclaw devices list

# Look for your iPhone's request (status: pending)
# Copy the requestId (UUID format)

# Approve the request
openclaw devices approve <requestId>

# Verify connection
openclaw nodes status
openclaw gateway call node.list --params "{}"
```

### Step 4.3: Verify Connection on iPhone
- Back in the iOS app, Settings should now show:
  - ✅ Gateway: "shamosbot's-MacBook-Air" **Connected**
  - Status indicator shows green/active

---

## Common Issues & Troubleshooting

| Problem | Cause | Solution |
|---------|-------|----------|
| **iPhone not appearing in Xcode** | USB not recognized or trust not granted | Unplug/replug; tap "Trust" on iPhone; restart Xcode |
| **"Code Signing Identity" error** | Signing config failed | Re-run `./scripts/ios-configure-signing.sh` |
| **Build fails with bundle ID error** | App ID not registered in Xcode | Check `apps/ios/.local-signing.xcconfig` created correctly |
| **Simulator selected instead of device** | Wrong build target | Select iPhone from device dropdown (not "Generic iOS Device") |
| **App crashes on launch** | Missing framework or signing issue | Check Xcode console for detailed error; rebuild clean: `Cmd+Shift+K` then `Cmd+B` |
| **"iPhone OS 15+ required"** | iPhone iOS version too old | Update iPhone iOS in Settings → General → Software Update |
| **Gateway pairing fails** | Gateway not running or network issue | Verify gateway: `openclaw gateway status` |

---

## Rebuild Workflow (After First Build)

Once the app is installed, rebuilds are much faster:

```bash
# After making code changes:
# 1. In Xcode, press Cmd+B (build)
#    - Just recompiles changed files (~1-2 seconds)

# 2. Press Cmd+R (run)
#    - Installs updated app to iPhone (~10 seconds)
#    - App relaunches automatically

# 3. Test on iPhone
```

---

## What Gets Installed on iPhone

After deployment, on your iPhone:
1. ✅ OpenClaw app icon appears on home screen
2. ✅ App is fully functional—no Mac connection needed after install
3. ✅ You can disconnect the USB cable and use the app freely
4. ✅ App persists on iPhone even if you restart your Mac
5. ✅ To update: rebuild in Xcode and run again (reinstalls new version)

---

## Security & Code Signing Explained

**Why all this complexity?**

Apple requires code signing for security:
1. **You sign the app** with your developer certificate (proves you made it)
2. **iOS verifies the signature** before installing (prevents malware)
3. **The certificate is tied to your Apple ID** (Apple can track who created what)

**What's required:**
- ✅ Free Apple ID (you have this)
- ✅ Developer certificate (Xcode creates automatically)
- ✅ Provisioning profile (Xcode manages automatically)
- ❌ $99/year Apple Developer Program membership (NOT required for testing on personal device)

**Duration:** Without Developer Program membership, the app signature is valid for **7 days**. After 7 days, you must rebuild/reinstall via Xcode. With membership ($99/year), the signature lasts 1 year.

---

## Quick Reference: Commands Cheat Sheet

```bash
# Full fresh setup (one-time)
cd ~/Projects/openclaw
pnpm install
./scripts/ios-configure-signing.sh
cd apps/ios && xcodegen generate && open OpenClaw.xcodeproj

# Or use shortcut:
pnpm ios:open

# In Xcode (after that):
Cmd+R     → Build, sign, install, and run on connected iPhone
Cmd+B     → Just build (no install)
Cmd+Shift+K → Clean build
Shift+Cmd+2 → Open Devices and Simulators window

# On gateway host (MacBook Air):
openclaw devices list      # Show pending pairing requests
openclaw devices approve <id>  # Approve iPhone pairing
openclaw nodes status      # Verify connection
```

---

## Next Steps After App Installs

1. ✅ App appears on iPhone home screen
2. ✅ Open it and grant all permission requests
3. ✅ Go to Settings → Gateway
4. ✅ Tap discovered "shamosbot's-MacBook-Air"
5. ✅ On MacBook Air terminal: `openclaw devices approve <id>`
6. ✅ App should now show **Connected** in Settings
7. ✅ Open Chat tab and test sending a message

---

## Important Notes for You

**Key Points to Remember:**

1. **You ARE the developer** — Your Apple ID signs the app. Apple trusts it came from you.

2. **Xcode does the heavy lifting** — Code signing, compilation, installation all happen automatically in Xcode. You just click "Run".

3. **No internet required** — The app is built locally on your Mac and installed via USB. No cloud upload.

4. **App stays on iPhone** — After installation, the iPhone is completely independent. You can unplug and use the app anywhere.

5. **7-day limitation** — Without paying $99/year, the app signature expires after 7 days and needs rebuilding. This is Apple's policy, not a limitation of our setup.

6. **Bundle ID must be unique** — The `./scripts/ios-configure-signing.sh` creates a unique ID per developer to prevent conflicts.

---

## Resources

- **Apple Xcode Deploy Docs:** https://developer.apple.com/documentation/xcode/distributing-your-app-to-registered-devices
- **OpenClaw iOS README:** https://github.com/openclaw/openclaw/blob/main/apps/ios/README.md
- **Code Signing Explained:** https://developer.apple.com/support/code-signing/

---

**Last Updated:** 2026-04-22 21:20 EDT  
**Author:** V.I.P.E.R  
**Status:** Ready for Execution
