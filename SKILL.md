---
name: ui-guide
description: 방금 구현/수정한 기능을 유저 관점 UI 가이드(md)로 문서화하는 범용 스킬.
  웹 프로젝트는 claude-in-chrome, 네이티브 프로젝트(iOS/Android)는 시뮬레이터·에뮬레이터로
  실제 화면 스크린샷을 찍어 docs/<slug>/assets/에 저장하고, 흐름을 산문으로 서술한 md에
  이미지를 삽입해 완성형 가이드를 만든다. 프로젝트 종류를 자동 감지해 캡처 방식을 고른다.
  트리거 - "UI 가이드 만들어줘", "가이드 작성해줘", "스크린샷 가이드",
  "사용 흐름 문서화", "화면 캡처해서 정리해줘", "/ui-guide"
---

# UI Guide

방금 작업한 기능/수정 건을 실제 화면으로 직접 조작하며 스크린샷을 찍고,
유저가 화면을 따라올 수 있는 md 가이드로 남긴다. 검증 겸 문서화.

프로젝트가 웹이든 iOS/Android 네이티브든 같은 절차를 따르되, **캡처 방식만 프로젝트 종류에 맞춰 자동으로 갈라진다.**

## Scope 정하기

세션 대화에서 무엇을 문서화할지 먼저 확정한다. 애매하면 사용자에게 되묻기보다
직전 작업 diff/보고 내용에서 기능명을 뽑아 슬러그로 쓴다(예: `checkout-flow-fix`).

## 프로젝트 종류 감지

캡처 시작 전에 대상 프로젝트 루트를 훑어 어떤 캡처 백엔드를 쓸지 정한다.

| 신호 | 판정 | 캡처 백엔드 |
|---|---|---|
| `ios/*.xcworkspace` 또는 `ios/*.xcodeproj` 존재 | iOS 네이티브(순수 또는 RN/Expo) | iOS 시뮬레이터 |
| `android/build.gradle`(`.kts`) 또는 `android/app/build.gradle` 존재 | Android 네이티브(순수 또는 RN/Expo) | Android 에뮬레이터 |
| 위 둘 다 없고 `package.json`에 웹 프레임워크(next/vite/react-dom 등) 또는 브라우저로 접근 가능한 dev 서버 | 웹 | claude-in-chrome |
| `ios/`와 `android/` 둘 다 있음 (React Native/Expo) | 네이티브 우선 — 사용자에게 iOS/Android 중 어디로 캡처할지 확인 | 확인 후 해당 백엔드 |

애매하면(예: Expo 프로젝트인데 웹 타겟도 있음) 사용자에게 플랫폼을 되묻는다. 셋 다 코드가 섞인
모노레포라면 Scope에서 정한 기능이 어느 앱/타겟에 속하는지부터 먼저 확정한다.

## 테스트 데이터 확보

가이드 흐름을 재현하려면 실제 화면에 뜨는 케이스가 있어야 한다. 캡처 시작 전에 확인:

1. 대상 프로젝트의 데이터 소스(로컬 DB, 목 서버, 앱 로컬 스토리지 등)에서 해당 흐름을 재현할
   레코드/상태가 있는지 먼저 확인한다.
2. 없으면 자체 판단으로 만들지 말고 **대안 2가지를 사용자에게 제시하고 컨펌 받는다**:
   - **A. Fixture 직접 주입**: DB INSERT, API 호출, 앱 로컬 스토리지 조작 등으로 테스트 레코드를
     직접 생성. 어떤 값으로 만들지(최소 스펙) 먼저 제시.
   - **B. 외부 API/실제 흐름으로 생성**: 연동된 플랫폼(결제·이커머스 등)을 통해 실제 흐름으로 생성.
   - 둘 다 **컨펌 없이 진행 금지** — 쓰기 작업이라 되돌리기 어려움.
3. 컨펌 받으면 생성 → 캡처 끝난 뒤 테스트용으로만 만든 데이터면 정리 여부도 같이 확인
   (특히 공유 DB/운영 환경이면 방치 금지).
4. 사용자가 "이미 있는 사례로만 써라"라고 하면 대안 제시 없이 그 케이스로만 진행.

## 폴더 구조

```
docs/<slug>/
  GUIDE.md
  assets/
    01-xxx.png
    02-xxx.png
```

- `<slug>`: kebab-case, 기능/이슈 단위 (테스트별 폴더 분리 — 여러 건이면 폴더도 여러 개)
- 이미지 파일명: `NN-짧은설명.png` zero-padded 순번
- 대상 프로젝트가 이미 자체 문서/가이드 컨벤션(CLAUDE.md 등)을 정해뒀으면 그걸 우선한다 —
  위 구조는 그런 컨벤션이 없을 때의 기본값이다.

## Workflow

0. **프로젝트 종류 감지 + 테스트 데이터 확보** (위 두 섹션 절차대로 먼저 확인/컨펌).

1. **캡처 백엔드 준비** — 감지된 종류에 따라 아래 중 하나:

   ### 웹 → claude-in-chrome
   - ToolSearch(한 번에): `select:mcp__claude-in-chrome__tabs_context_mcp,mcp__claude-in-chrome__navigate,mcp__claude-in-chrome__computer,mcp__claude-in-chrome__tabs_create_mcp`
   - `tabs_context_mcp` → 없으면 `tabs_create_mcp`. 세션 기존 탭 재사용은 사용자가 명시할 때만.
   - 캡처: `computer` 툴 `action: "screenshot"` + **`save_to_disk: true`**.

   ### iOS 네이티브 → iOS 시뮬레이터
   - 부팅된 시뮬레이터 확인: `xcrun simctl list devices | grep Booted`. 없으면
     `xcrun simctl boot "<device name>"` 후 `open -a Simulator`로 창을 띄운다.
   - 앱 설치/실행이 필요하면 `xcrun simctl install booted <path-to-.app>` →
     `xcrun simctl launch booted <bundle-id>`.
   - **조작(탭/스와이프/타이핑)**: computer-use MCP로 Simulator.app 창을 직접 클릭/입력한다
     (macOS 네이티브 앱이라 `request_access(["Simulator"])` 후 tier "full"로 자유 조작 가능).
     좌표는 반드시 스크린샷을 먼저 보고 잡는다.
   - **캡처(저장용)**: 화면이 원하는 상태가 되면 computer-use 스크린샷이 아니라
     `xcrun simctl io booted screenshot docs/<slug>/assets/NN-설명.png`를 Bash로 직접 실행한다 —
     시뮬레이터 프레임/OS 크롬 없이 기기 화면만 깨끗하게 저장된다.

   ### Android 네이티브 → Android 에뮬레이터
   - 실행 중 기기 확인: `adb devices`. 없으면 AVD 부팅:
     `emulator -avd <avd-name> &` (또는 Android Studio에서 실행) 후 `adb wait-for-device`.
   - 앱 설치/실행이 필요하면 `adb install -r <path-to.apk>` →
     `adb shell monkey -p <package> -c android.intent.category.LAUNCHER 1`.
   - **조작(탭/스와이프/타이핑)**: computer-use MCP로 에뮬레이터 창을 직접 클릭/입력한다
     (`request_access(["Android Emulator"])` 후 tier "full"). 좌표는 스크린샷을 먼저 보고 잡는다.
     정밀한 좌표가 필요하면 `adb shell input tap X Y` / `adb shell input text "..."` /
     `adb shell input swipe X1 Y1 X2 Y2`를 Bash로 직접 써도 된다(기기 좌표계라 더 정확할 때가 많다).
   - **캡처(저장용)**: `adb exec-out screencap -p > docs/<slug>/assets/NN-설명.png`를 Bash로 실행 —
     에뮬레이터 창 크롬 없이 기기 화면만 저장된다.

2. **폴더 생성**: `mkdir -p docs/<slug>/assets` (Bash).

3. **흐름 재현 + 캡처**: 실제 화면을 단계별로 조작하며, 상태가 바뀌는 의미있는 지점마다 위에서
   고른 백엔드로 캡처한다.
   - 캡처 대상: 시작 화면 → 조작 직후 변화 → 최종 결과. 아무 변화 없는 중간 클릭은 생략.
   - 웹이 아닌 경우 저장 경로를 직접 지정하므로 별도 `mv` 불필요. 웹은 `save_to_disk` 결과 경로를
     받아 즉시 `docs/<slug>/assets/NN-설명.png`로 이동(Bash `mv`).
   - 로그인/계정 필요 시 계정 정보는 사용자에게 요청(자격증명 직접 입력 금지 원칙 준수).

4. **GUIDE.md 작성**: 아래 템플릿대로. 스크린샷 안 찍은 단계는 서술하지 말 것 —
   실제로 확인한 화면만 가이드에 남긴다. frontmatter는 대상 프로젝트에 자체 컨벤션이 있으면
   그걸 따르고, 없으면 최소 형태(title/created)만 쓴다.

   ```md
   ---
   title: <기능명> 사용 가이드
   created: <YYYY-MM-DD>
   ---

   # <기능명> 가이드

   <한두 문장으로 무엇이 왜 바뀌었는지, 어떤 플랫폼(웹/iOS/Android)에서 확인했는지>

   ## 1. <단계 설명>

   <무엇을 어떻게 조작하는지 산문으로 1~3문장>

   ![](assets/01-xxx.png)

   ## 2. <다음 단계>
   ...
   ```

5. **검증**: 이미지 파일 실제 존재 확인(`ls docs/<slug>/assets/`), md 내 상대경로가
   실제 파일명과 정확히 일치하는지 확인.

6. **보고**: 로컬 경로(`docs/<slug>/GUIDE.md`) 안내. 어떤 캡처 백엔드(웹/iOS/Android)를
   썼는지도 같이 알린다.

## 주의사항

- 스크린샷 못 찍은 단계를 상상해서 쓰지 않는다 — 확인 안 된 화면을 지어내면 가이드 자체가
  거짓 문서가 된다.
- 브라우저/앱 조작 중 alert·confirm·시스템 팝업을 유발하는 요소는 클릭 금지(세션 멈춤 위험).
- 여러 이슈를 한 번에 문서화할 땐 `docs/` 아래 이슈별 폴더를 분리 — 한 GUIDE.md에 몰아넣지 않는다.
- 네이티브 캡처는 반드시 `request_access`로 대상 앱(Simulator / Android Emulator)을 먼저
  승인받고 시작한다. 승인 안 된 상태로 computer-use를 호출하면 실패한다.
- iOS/Android 둘 다 없는 순수 웹 프로젝트에서 네이티브 캡처를 시도하지 않는다 — 프로젝트 종류
  감지 결과를 반드시 먼저 확인한다.
