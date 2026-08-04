# ui-guide

[English](README.md) | 한국어

Claude Code용 UI 가이드 자동 생성 스킬. 방금 구현/수정한 기능을 실제 화면으로 조작하며
스크린샷을 찍고, 유저가 따라올 수 있는 마크다운 가이드로 남긴다.

- **웹 프로젝트** → [claude-in-chrome](https://docs.claude.com/en/docs/claude-code) 확장으로 브라우저 조작·캡처
- **iOS 네이티브(순수 또는 React Native/Expo)** → iOS 시뮬레이터, `xcrun simctl`로 캡처
- **Android 네이티브(순수 또는 React Native/Expo)** → Android 에뮬레이터, `adb`로 캡처

프로젝트 종류는 자동 감지한다(`ios/`, `android/`, `package.json` 존재 여부).

## 설치

Claude Code 스킬은 `~/.claude/skills/<name>/SKILL.md` 형태로 등록하면 **모든 프로젝트**에서
바로 쓸 수 있다. 특정 프로젝트에서만 쓰려면 그 프로젝트의 `.claude/skills/`에 넣는다.

```bash
git clone https://github.com/galaxy4276/ui-guide.git
mkdir -p ~/.claude/skills
ln -s "$(pwd)/ui-guide" ~/.claude/skills/ui-guide
```

또는 그냥 `SKILL.md` 하나만 복사해도 된다:

```bash
mkdir -p ~/.claude/skills/ui-guide
curl -o ~/.claude/skills/ui-guide/SKILL.md \
  https://raw.githubusercontent.com/galaxy4276/ui-guide/main/SKILL.md
```

## 사용

Claude Code 세션에서:

```
UI 가이드 만들어줘
```

또는

```
/ui-guide
```

기능을 문서화하고 싶다고 말하면 트리거된다. 결과물은 대상 프로젝트의
`docs/<slug>/GUIDE.md` + `docs/<slug>/assets/*.png`로 생성된다.

## 요구사항

| 플랫폼 | 필요한 것 |
|---|---|
| 웹 | Claude Code에 [claude-in-chrome](https://docs.claude.com) 확장 연결 |
| iOS | Xcode Command Line Tools (`xcrun simctl`), computer-use 접근 권한(Simulator.app) |
| Android | Android SDK platform-tools (`adb`), computer-use 접근 권한(Android Emulator) |

## 동작 원칙

- 테스트 데이터가 없으면 임의로 지어내지 않고, fixture 직접 주입 또는 외부 API 생성 중
  대안을 제시하고 **사용자 컨펌 후에만** 진행한다.
- 스크린샷 찍지 못한 단계는 가이드에 쓰지 않는다 — 확인 안 된 화면을 서술하면 문서가 거짓이 된다.

## License

MIT
