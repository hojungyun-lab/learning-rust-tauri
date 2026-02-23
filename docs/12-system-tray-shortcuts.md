# 🍱 09. 시스템 트레이와 단축키 (Tray & Shortcuts)

## 🎯 학습 목표 (Goal)
메인 창을 닫아도 시스템 백그라운드에 남아 동작하는 트레이 아이콘(macOS의 메뉴바, Windows의 작업표시줄 아이콘)을 만들고, OS 전역에서 작동하는 글로벌 단축키를 설정합니다.

---

## 💡 핵심 개념 (Core Concepts)

- **System Tray:** 창(Window)에 종속되지 않는 독립적인 데스크톱 UI입니다. Tauri v2부터는 트레이 관련 기능도 별도의 플러그인(`@tauri-apps/plugin-global-shortcut`)이나 API 공간으로 분리 및 강화되었습니다.
- 앱을 "닫기(X)" 눌렀을 때, 프로그램이 완전히 종료(`exit()`)되지 않고 창만 닫힌 채(`hide()`) 트레이에 남아있게 만드는 것이 핵심 패턴입니다.

---

## 💻 실습: 트레이 초기화 및 이벤트 (Hands-on)

### Step 1: `tauri.conf.json` 아이콘 설정
이 부분은 기본 생성 시 설정되어 있지만, 앱 빌드 트레이에 보일 아이콘 경로가 맞는지 확인해야 합니다. (보통 `icons/32x32.png` 등을 참조함)

### Step 2: Rust 쪽에 트레이 및 메뉴 생성 코드 작성
트레이는 GUI의 시작 전에 준비하는 것이 좋으므로 `main.rs` 혹은 `lib.rs`의 `setup()` 훅 안에 작성합니다.

```rust
// src-tauri/src/lib.rs
use tauri::{
    menu::{Menu, MenuItem},
    tray::{MouseButton, TrayIconBuilder, TrayIconEvent},
    Manager,
};

#[cfg_attr(mobile, tauri::mobile_entry_point)]
pub fn run() {
    tauri::Builder::default()
        // 앱 초기화(setup) 단계에서 트레이를 묶습니다.
        .setup(|app| {
            // 1️⃣ 트레이에서 우클릭하면 나올 메뉴 생성
            let quit_i = MenuItem::with_id(app, "quit", "앱 완전 종료", true, None::<&str>)?;
            let show_i = MenuItem::with_id(app, "show", "메인 창 열기", true, None::<&str>)?;
            let menu = Menu::with_items(app, &[&show_i, &quit_i])?;

            // 2️⃣ 트레이 아이콘 빌드 (이벤트 핸들러 부착)
            let _tray = TrayIconBuilder::new()
                .menu(&menu)      // 방금 만든 메뉴 연결
                .show_menu_on_left_click(false) // 윈도우/맥 동작 차이 보정
                // 트레이 아이콘 자체가 클릭/우클릭 될 때의 이벤트 리스너
                .on_tray_icon_event(|tray, event| match event {
                    // 좌클릭 시 메인 창을 다시 띄우기
                    TrayIconEvent::Click { button: MouseButton::Left, .. } => {
                        let app = tray.app_handle();
                        if let Some(window) = app.get_webview_window("main") {
                            window.show().unwrap(); // 숨겨진 창 다시 보이기
                            window.set_focus().unwrap(); // 최상단 포커스
                        }
                    }
                    _ => {}
                })
                // 메뉴 아이템(quit, show 등)이 클릭될 때의 감지기
                .on_menu_event(|app, event| match event.id.as_ref() {
                    "quit" => app.exit(0), // 앱 프로세스 완전 종료!
                    "show" => {
                        if let Some(window) = app.get_webview_window("main") {
                            window.show().unwrap();
                        }
                    }
                    _ => {}
                })
                .build(app)?;

            Ok(())
        })
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

### Step 3: 창(X) 버튼 눌렀을 때의 동작 가로채기
트레이를 만들었으니, 창의 `X` 버튼을 누르면 종료가 아니라 '숨기기'가 되도록 윈도우 이벤트를 가로채야 합니다.

```rust
// 바로 위 Builder 밑에 추가합니다.
        .on_window_event(|window, event| match event {
            // 사용자가 [X] 버튼을 눌러 종료(CloseRequested)를 시도할 때!
            tauri::WindowEvent::CloseRequested { api, .. } => {
                // 이벤트 전파 방지 = 진짜로 창이 닫히는 걸 막아버림
                api.prevent_close();
                // 대신 창을 눈에 안 보이게 숨김
                window.hide().unwrap();
            }
            _ => {}
        })
```

### (참고) 글로벌 단축키 플러그인
앱이 안 떠 있을 때도 `Cmd/Ctrl + Shift + T`를 누르면 화면에 팝업이 뜨게 하려면:
1. `pnpm tauri add global-shortcut` 설치
2. Rust 플러그인 등록 후 단축키 등록 API 호출 (자세한 API는 플러그인 문서 참조)

---

## 🚀 마무리 및 다음 단계

이제 우리 앱은 메신저나 백신 프로그램처럼 운영체제의 훌륭한 백그라운드 거주자가 되었습니다!
그런데 [10장](./10-native-apis.md) 파일 시스템 파트에서, "v2는 권한이 기본적으로 막혀있어 에러가 난다" 고 언급했었죠?
Tauri v2의 구조적 안정성을 책임지는 가장 중요한 개념인 **Capabilities(접근 권한 통제)** 시스템을 살펴보기 위해 [**13. 보안 및 Capabilities**](./13-security-capabilities.md)로 이동해봅시다.
