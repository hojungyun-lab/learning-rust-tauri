# 🛡️ 10. 보안 및 Capabilities (v2 API 핵심)

## 🎯 학습 목표 (Goal)
Tauri v2에서 새롭게 도입된 강력한 권한 통제 시스템인 **Capabilities** 구조를 파악하고, 프론트엔드가 꼭 필요한 디렉터리와 시스템 API에만 접근하도록 화이트리스트(Whitelist)를 개방하는 방법을 배웁니다.

---

## 💡 핵심 개념 (Core Concepts)

### 왜 내 `fs.readTextFile()` 은 거절(Denied) 당했을까요?
만약 누군가 브라우저 해킹(XSS 공격)을 통해 여러분의 프론트엔드에 악성 JS를 심었다고 가정해봅시다. 이 스크립트가 로컬 디스크의 `.ssh/id_rsa` 폴더를 통째로 읽어 서버로 빼돌린다면 끔찍한 일입니다.

이를 방지하기 위해 Tauri v1에서는 `tauri.conf.json`의 allowlist를 썼고, **Tauri v2에서는 완전히 독립된 `src-tauri/capabilities/` 폴더 기반의 더욱 정교한 모델**을 도입했습니다.
- 프론트엔드는 **명시적으로 허락된 명령어(Command)와 허락된 범위(Scope)** 에만 접근할 수 있습니다.
- 모든 플러그인과 IPC 커맨드는 기본적으로 `Deny(거절)` 상태입니다!

---

## 💻 실습: Capabilities 부여하기 (Hands-on)

### 구조 확인하기
Tauri v2 앱 스캐폴드를 만들면 `src-tauri/capabilities/default.json` (또는 `core.json`) 파일이 존재합니다. 열어보면 대략 이런 형태입니다.

```json
{
  "$schema": "../gen/schemas/desktop-schema.json",
  "identifier": "default",
  "description": "Capability for the main window",
  "windows": ["main"],
  "permissions": [
    "core:default",
    // 여기에 우리가 추가해야 할 플러그인의 권한들을 넣습니다.
  ]
}
```
- `"windows": ["main"]` : 이 권한들은 오직 "main" 창의 프론트엔드 영역에서만 허용된다는 뜻입니다. (설정창에선 권한을 축소하는 식의 분리가 가능!)

### Step 1: 파일 시스템 읽기 권한(Permission) 추가
[10. Native APIs](./10-native-apis.md) 에서 파일 열기 다이얼로그와 파일 읽기를 사용했었습니다.
다이얼로그로 파일을 여는 것, 그리고 파일을 '읽는' 행위 모두 특정 권한 심볼이 필요합니다.

```json
// src-tauri/capabilities/default.json 의 permissions 배열에 추가:
  "permissions": [
    "core:default",
    "dialog:default", 
    "fs:default" 
  ]
```

### Step 2: Scope (허용 범위 정밀 타격)
`fs:default`를 열어두면 프론트엔드는 "사용자 기본 설정에 해당하는 파일 접근"이 가능해졌습니다. 하지만 완전히 자유로운 것은 아닙니다.
다이얼로그를 통해 선택한 파일은 *의도된 행동*이므로 읽힙니다.
만약 프론트엔드가 JS를 통해 **사용자 몰래 고정된 경로(예: `$HOME/Documents/secret.txt`)를 읽고자 한다면?**
추가적인 보안 스코프 객체를 통해 파일 트리의 특정 경로를 명시해야 합니다.

`src-tauri/capabilities/fs-scoped.json` 이라는 파일을 별도로 만들어도 되고, `default.json`에 합쳐도 됩니다.
```json
{
  "identifier": "fs-scoped",
  "windows": ["main"],
  "permissions": [
    {
      "identifier": "fs:allow-read-text-file",
      "allow": [
        // "$HOME" 과 같은 OS 종속 환경변수를 Tauri가 자동 치환해줍니다.
        { "path": "$HOME/Documents/*" }
      ]
    }
  ]
}
```

### 내가 Crate(Rust)로 직접 만든 커맨드도 막히나요?
아니요! 여러분이 작성한 `#[tauri::command] fn my_rust_func()` 는 기본적으로 권한 시스템의 체크를 우회합니다.
Rust 백엔드 내부에서 하는 짓(로직)은 개발자 본인의 책임하에 100% 신뢰 구역으로 치기 때문입니다. 프론트엔드 플러그인(`@tauri-apps/api/*`)을 통해 타우리의 네이티브 시스템에 접근할 때만 Capabilities가 통제합니다.

---

## 🚀 마무리 및 다음 단계

개발 중 `IPC 403 Forbidden` 과 비슷한 에러콘솔이 뜬다면 거의 99% 이 Capabilities 설정이 빠져 있어서입니다. 새로 플러그인을 설치(`pnpm tauri add xxx`)할 때마다 꼭 관련 권한을 `default.json`에 추가하는 습관을 들여야 합니다.

OS 접근을 넘어, 로컬 데이터베이스를 애플리케이션 안에 콕 박아놓고 수만 건의 캐시나 기록(History)을 영구적으로 저장하고 싶지 않으신가요?
다음 장 [**14. 로컬 데이터베이스 연동 (SQLite)**](./14-database-sqlite.md)에서 파일 베이스 DB의 정석인 SQLite 연동법을 다룹니다.
