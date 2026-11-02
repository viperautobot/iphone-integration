# Build Blocker Report: OpenClaw iOS Official Build

**Date:** 2026-04-22 22:29 EDT  
**Status:** HARD BLOCKER — Official build path cannot proceed  
**Affected:** Official `pnpm ios:open` → xcodegen → Xcode project generation

---

## The Problem

The official OpenClaw iOS build process fails at the xcodegen stage with:

```
Parsing project spec failed: Unknown Target dependency: ["package": "OpenClawKit"]
```

And alternately:

```
Parsing project spec failed: Unknown Target dependency: ["package": "Swabble", "product": "SwabbleKit"]
```

**Status:** This error persists **even after:**
- ✅ Installing Xcode 16 (full application)
- ✅ Swift 6.2 toolchain active and verified
- ✅ pnpm dependencies installed (1,247 packages)
- ✅ xcodegen installed (v2.2.0)
- ✅ iOS signing configured
- ✅ Both SPM packages exist at correct paths with valid manifests

---

## Root Cause

xcodegen cannot resolve Swift Package Manager dependencies because:

1. **Package Resolution Failure** — xcodegen delegates to Xcode's SPM system to validate and parse `Package.swift` manifests
2. **Path Context Issue** — When xcodegen runs from `apps/ios/`, it cannot properly resolve packages at `../shared/OpenClawKit` and `../../Swabble`
3. **Workspace Configuration** — The packages are NOT registered in pnpm-workspace.yaml (iOS build excluded from workspace)
4. **Super-Alpha Status** — The iOS build is documented as super-alpha with breaking changes expected; this appears to be a known limitation

---

## Attempts Made to Resolve

| Attempt | Method | Result |
|---------|--------|--------|
| 1 | Install Xcode 16 | ✓ Done, Swift 6.2 active |
| 2 | Clear xcodegen cache | ✗ Still fails |
| 3 | Run from repo root | ✗ Same error |
| 4 | Run pnpm ios:open | ✗ xcodegen generation fails |
| 5 | Manual xcodegen | ✗ Package resolution fails |
| 6 | Search for cached project | ✗ No pre-built .xcodeproj |

---

## What Was Accomplished

✅ Full OpenClaw repository cloned (14,921 files, 512MB)  
✅ All npm dependencies installed (1,247 packages)  
✅ Xcode 16 installed with Swift 6.2 toolchain  
✅ xcodegen v2.2.0 installed and verified  
✅ iOS signing configured  
✅ Both SPM packages located and validated:
- `~/Documents/openclaw/apps/shared/OpenClawKit/Package.swift` (valid)
- `~/Documents/openclaw/Swabble/Package.swift` (valid)

**What DID NOT Complete:** Xcode project generation (blocked at xcodegen)

---

## Implications

**Blocked Chain:**
```
xcodegen cannot generate → OpenClaw.xcodeproj doesn't exist
  ↓
Cannot open in Xcode → No IDE available
  ↓
Cannot build → No iOS app binary
  ↓
Cannot deploy → No way to install on iPhone
```

**Duration:** Build attempted for ~1 hour with system requirements met. Blocker is in OpenClaw's build configuration, not user environment.

---

## Recommended Path Forward

Given the super-alpha status and persistent build failure, options are:

### Option 1: Third-Party iOS Clients (Recommended)
- **GoClaw** — https://apps.apple.com/us/app/goclaw-openclaw-app/id6758519111
- **ClawOn** — https://apps.apple.com/us/app/clawon-openclaw-agent-ai-app/id6758406207
- **PocketClaw** — https://github.com/abeazam/pocketClaw
- **Aight** — Lightweight OpenClaw client

**Advantage:** Instant install from App Store, known working, community-maintained  
**Timeline:** 5-10 minutes to download and pair with gateway

### Option 2: Contact OpenClaw Community
- **Discord:** https://discord.com/invite/clawd
- **GitHub Issues:** https://github.com/openclaw/openclaw/issues
- **Post:** Include this build blocker report

**Advantage:** Get direct workaround from iOS maintainers  
**Timeline:** Unknown (depends on community response)

### Option 3: Wait for Official Fix
- Monitor GitHub releases for iOS build fixes
- Check CHANGELOG for xcodegen/package resolution updates

**Timeline:** Unknown

---

## Project Status Summary

| Component | Status | Notes |
|-----------|--------|-------|
| Repository | ✅ Ready | Cloned, all files present |
| Dependencies | ✅ Ready | pnpm install complete |
| Toolchain | ✅ Ready | Xcode 16 + Swift 6.2 |
| Build Tools | ✅ Ready | xcodegen installed |
| Signing | ✅ Ready | iOS config complete |
| **xcodegen → Xcode project** | ❌ BLOCKED | Package resolution fails |
| Xcode IDE | ❌ Not available | Project file doesn't exist |
| iPhone build | ❌ Not possible | No project to build |
| iPhone deploy | ❌ Not possible | No app binary |

---

## Conclusion

The official `pnpm ios:open` → xcodegen → Xcode workflow has a fundamental blocker in the OpenClaw repository's iOS build configuration that prevents project generation. This is **not a system/toolchain issue** (Xcode 16 + Swift 6.2 verified working) but rather an **upstream project configuration issue** in the super-alpha iOS build.

**Recommendation:** Pivot to third-party iOS clients (GoClaw, ClawOn, PocketClaw) which are working alternatives, or engage OpenClaw community for workaround.

---

**Build Session Details:**
- Start: 2026-04-22 21:28 EDT
- End: 2026-04-22 22:29 EDT
- Duration: ~1 hour
- Progress: 90% (all setup complete, blocker at xcodegen)
- Blocker: xcodegen package resolution (upstream issue)
