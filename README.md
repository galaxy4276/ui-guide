# ui-guide

English | [한국어](README.ko.md)

A Claude Code skill that auto-generates screenshot-based UI guides. It walks
through a feature you just built or fixed on the real screen, captures
screenshots along the way, and writes a markdown guide a user can follow
step by step.

- **Web projects** → drives and captures via the [claude-in-chrome](https://docs.claude.com/en/docs/claude-code) extension
- **iOS native (plain or React Native/Expo)** → iOS Simulator, captured with `xcrun simctl`
- **Android native (plain or React Native/Expo)** → Android Emulator, captured with `adb`

Project type is auto-detected (presence of `ios/`, `android/`, `package.json`).

## Install

Claude Code skills registered under `~/.claude/skills/<name>/SKILL.md` are
available in **every project**. To scope it to a single project instead, put
it in that project's `.claude/skills/`.

```bash
git clone https://github.com/galaxy4276/ui-guide.git
mkdir -p ~/.claude/skills
ln -s "$(pwd)/ui-guide" ~/.claude/skills/ui-guide
```

Or just copy the single `SKILL.md` file:

```bash
mkdir -p ~/.claude/skills/ui-guide
curl -o ~/.claude/skills/ui-guide/SKILL.md \
  https://raw.githubusercontent.com/galaxy4276/ui-guide/main/SKILL.md
```

## Usage

In a Claude Code session:

```
Make a UI guide
```

or

```
/ui-guide
```

Any request to document a feature triggers it. Output lands in the target
project as `docs/<slug>/GUIDE.md` + `docs/<slug>/assets/*.png`.

## Requirements

| Platform | Needs |
|---|---|
| Web | Claude Code connected to the [claude-in-chrome](https://docs.claude.com) extension |
| iOS | Xcode Command Line Tools (`xcrun simctl`), computer-use access to Simulator.app |
| Android | Android SDK platform-tools (`adb`), computer-use access to Android Emulator |

## Principles

- If no test data exists, it never fabricates one — it proposes injecting a
  fixture or generating one through an external API, and proceeds only
  **after user confirmation**.
- It never describes a step it didn't actually screenshot — writing about an
  unverified screen would make the guide itself false.

## License

MIT
