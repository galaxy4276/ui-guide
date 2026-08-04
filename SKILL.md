---
name: ui-guide
description: General-purpose skill that documents a feature you just built/fixed as a
  user-facing UI guide (markdown). Web projects use claude-in-chrome; native projects
  (iOS/Android) use the simulator/emulator to capture real screenshots into
  docs/<slug>/assets/, then weave them into a prose walkthrough. Auto-detects project
  type to pick the capture backend.
  Triggers - "make a UI guide", "write a guide", "screenshot guide",
  "document this flow", "capture the screens and write it up", "/ui-guide"
---

# UI Guide

Walk through what you just built by operating the real screen, capturing
screenshots along the way, and write a markdown guide a user can follow.
Doubles as verification and documentation.

Same procedure whether the project is web or native (iOS/Android) —
**only the capture backend switches, automatically, based on project type.**

## Pin the scope

Decide what to document from the conversation. If ambiguous, don't ask —
pull a feature name from the most recent diff/report and turn it into a
slug (e.g. `checkout-flow-fix`).

## Detect project type

Before capturing, scan the target project root to pick a capture backend.

| Signal | Verdict | Capture backend |
|---|---|---|
| `ios/*.xcworkspace` or `ios/*.xcodeproj` exists | iOS native (plain or RN/Expo) | iOS Simulator |
| `android/build.gradle`(`.kts`) or `android/app/build.gradle` exists | Android native (plain or RN/Expo) | Android Emulator |
| Neither of the above, and `package.json` has a web framework (next/vite/react-dom etc.) or a browser-reachable dev server | Web | claude-in-chrome |
| Both `ios/` and `android/` exist (React Native/Expo) | Native, ambiguous — ask the user which platform to capture on | Whichever they pick |

If it's genuinely ambiguous (e.g. an Expo project that also has a web
target), ask the user which platform. In a monorepo mixing all three, first
pin down which app/target the feature from Scope actually belongs to.

## Line up test data

Reproducing the flow needs a real case on screen. Before capturing:

1. Check the target project's data source (local DB, mock server, app local
   storage, etc.) for a record/state that reproduces this flow.
2. If none exists, don't invent one — **propose two alternatives and get
   confirmation**:
   - **A. Inject a fixture directly**: DB insert, API call, or app local
     storage write. State the minimal spec first.
   - **B. Generate via an external API / real flow**: through a connected
     platform (payments, e-commerce, etc.).
   - **Never proceed without confirmation on either** — these are writes,
     hard to undo.
3. Once confirmed, create it → after capturing, check whether the
   test-only data should be cleaned up (especially on a shared DB or
   production-like environment — don't leave it lingering).
4. If the user says "just use what's already there," skip the alternatives
   and use that case only.

## Folder layout

```
docs/<slug>/
  GUIDE.md
  assets/
    01-xxx.png
    02-xxx.png
```

- `<slug>`: kebab-case, one per feature/issue (split into separate folders
  for multiple issues — don't merge them)
- Image filenames: `NN-short-description.png`, zero-padded sequence
- If the target project already has its own docs/guide convention
  (CLAUDE.md, etc.), follow that instead — the layout above is only the
  default when no such convention exists.

## Workflow

0. **Detect project type + line up test data** (per the two sections above,
   confirm first).

1. **Prep the capture backend** — one of the following, based on what was
   detected:

   ### Web → claude-in-chrome
   - ToolSearch (single call): `select:mcp__claude-in-chrome__tabs_context_mcp,mcp__claude-in-chrome__navigate,mcp__claude-in-chrome__computer,mcp__claude-in-chrome__tabs_create_mcp`
   - `tabs_context_mcp` → if empty, `tabs_create_mcp`. Only reuse an
     existing tab if the user explicitly asks.
   - Capture: `computer` tool, `action: "screenshot"` with
     **`save_to_disk: true`**.

   ### iOS native → iOS Simulator
   - Check for a booted simulator: `xcrun simctl list devices | grep
     Booted`. If none, `xcrun simctl boot "<device name>"` then
     `open -a Simulator` to bring up the window.
   - If the app needs installing/launching: `xcrun simctl install booted
     <path-to-.app>` → `xcrun simctl launch booted <bundle-id>`.
   - **Interaction (tap/swipe/type)**: drive it with computer-use MCP,
     clicking/typing directly into the Simulator.app window (it's a native
     macOS app, so `request_access(["Simulator"])` grants tier "full").
     Always look at a screenshot before picking coordinates.
   - **Capture (for saving)**: once the screen is in the desired state,
     don't use a computer-use screenshot — run
     `xcrun simctl io booted screenshot docs/<slug>/assets/NN-description.png`
     via Bash instead. This saves a clean device frame with no simulator
     bezel or OS chrome.

   ### Android native → Android Emulator
   - Check for a running device: `adb devices`. If none, boot an AVD:
     `emulator -avd <avd-name> &` (or launch from Android Studio), then
     `adb wait-for-device`.
   - If the app needs installing/launching: `adb install -r
     <path-to.apk>` → `adb shell monkey -p <package> -c
     android.intent.category.LAUNCHER 1`.
   - **Interaction (tap/swipe/type)**: drive it with computer-use MCP,
     clicking/typing directly into the emulator window
     (`request_access(["Android Emulator"])` grants tier "full"). Always
     look at a screenshot before picking coordinates. For precise
     coordinates, `adb shell input tap X Y` / `adb shell input text "..."`
     / `adb shell input swipe X1 Y1 X2 Y2` via Bash also work, and are
     often more accurate since they use device coordinates directly.
   - **Capture (for saving)**: `adb exec-out screencap -p >
     docs/<slug>/assets/NN-description.png` via Bash — saves the device
     screen with no emulator window chrome.

2. **Create the folder**: `mkdir -p docs/<slug>/assets` (Bash).

3. **Reproduce the flow + capture**: operate the real screen step by step,
   capturing with the backend chosen above at every meaningful state change.
   - What to capture: the starting screen → the change right after an
     action → the final result. Skip intermediate clicks that change
     nothing.
   - Non-web backends write the save path directly, so no `mv` needed.
     For web, take the `save_to_disk` result path and immediately `mv` it
     to `docs/<slug>/assets/NN-description.png`.
   - If login/an account is required, ask the user for credentials
     (never enter credentials yourself — see credential-handling policy).

4. **Write GUIDE.md**: follow the template below. Don't narrate a step you
   didn't screenshot — only document screens you actually verified. Use
   the target project's own frontmatter convention if it has one;
   otherwise keep it minimal (title/created).

   ```md
   ---
   title: <feature name> guide
   created: <YYYY-MM-DD>
   ---

   # <feature name> guide

   <one or two sentences: what changed and why, which platform (web/iOS/Android) this was verified on>

   ## 1. <step description>

   <1-3 sentences of prose describing what to do>

   ![](assets/01-xxx.png)

   ## 2. <next step>
   ...
   ```

5. **Verify**: confirm the image files actually exist (`ls
   docs/<slug>/assets/`), and that every relative path in the markdown
   matches an actual filename exactly.

6. **Report**: point to the local path (`docs/<slug>/GUIDE.md`) and state
   which capture backend (web/iOS/Android) was used.

## Cautions

- Never invent a step you didn't screenshot — describing an unverified
  screen makes the guide itself false.
- Never click anything that triggers an alert/confirm/system popup during
  browser or app interaction — it can freeze the session.
- When documenting multiple issues at once, split them into separate
  folders under `docs/` — don't cram them into one GUIDE.md.
- Native capture always requires `request_access` for the target app
  (Simulator / Android Emulator) first. Calling computer-use without that
  approval fails.
- Never attempt native capture on a pure web project with neither `ios/`
  nor `android/` — always confirm the project-type detection result first.
