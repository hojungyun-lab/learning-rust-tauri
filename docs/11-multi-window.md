# 🪟 08. 멀티 윈도우 관리

## 🎯 학습 목표 (Goal)
메인 창 외에 설정 창, 알림 전용 토스트 팝업, 서브 화면 등 새로운 창(Window)을 동적으로 생성하고, 창들끼리 데이터를 교환하는 방법을 학습합니다.

---

## 💡 핵심 개념 (Core Concepts)

데스크톱 앱의 장점은 여러 개의 창을 띄울 수 있다는 점입니다.
- **`tauri.conf.json`에 정적 등록:** 앱이 켜질 때부터 띄우고 싶은 창. 보통 "main" 창 하나만 등록합니다.
- **프론트엔드(TS)에서 동적 생성:** 버튼을 누를 때마다 띄우는 창 (예: 포토샵의 도구 박스, 카카오톡의 개별 채팅창).

새로 띄워진 창은 각자 별도의 메모리(JavaScript 텍스트, 변수 등)를 가집니다. 즉, A창의 TS 변수를 B창이 직접 읽을 수 없습니다. 창과 창이 대화하려면 [08. 이벤트 시스템](./08-event-system.md)을 활용하거나 저장소(State/DB)를 거쳐야 합니다.

---

## 💻 실습: 팝업 창 동적 생성 (Hands-on)

"설정(Settings)" 버튼을 누르면 약관이나 옵션이 나오는 새로운 미니 윈도우를 띄워보겠습니다.

### Step 1: HTML 라우트(경로) 준비
보통 메인 창은 `/index.html`을 바라봅니다. 새 창은 `/settings.html` (또는 프론트엔드 라우터의 `/settings` 경로)를 바라보게 해야 합니다. 프로젝트 루트의 프론트엔드 폴더에 `settings.html`을 하나 만들어 둡니다.

### Step 2: 프론트엔드에서 새 창 띄우기 로직

메인 Svelte 컴포넌트에 다음을 추가합니다.

```svelte
<!-- src/routes/+page.svelte -->
<script lang="ts">
  import { WebviewWindow } from "@tauri-apps/api/webviewWindow";

  async function openSettingsWindow() {
    // 1️⃣ 첫 번째 인자 "settings-window"는 창의 고유 ID(라벨)입니다.
    // 동일한 라벨의 창을 2번 띄우려 하면 에러(이미 존재함)가 납니다.
    const webview = new WebviewWindow("settings-window", {
      url: "settings.html", // SPA 라우터라면 "/settings" 같은 라우트 경로 지정
      title: "설정 창",
      width: 400,
      height: 500,
      center: true,         // 화면 중앙에 배치
      resizable: false,     // 크기 조절 불가
      alwaysOnTop: true,    // 항상 위 (옵션)
    });

    // 창이 완전히 초기화(렌더링 준비)되고 나서 실행되는 이벤트
    webview.once("tauri://created", () => {
      console.log("설정 창 생성 성공!");
    });

    // 에러 처리 (예: 이미 창이 열려있는 경우)
    webview.once("tauri://error", (e) => {
      console.warn("창 생성 실패(또는 이미 존재):", e);
    });
  }
</script>

<button onclick={openSettingsWindow}>설정 열기</button>
```

### Step 3: 백엔드(Rust)에서 제어하기 (참고)

창을 꼭 프론트엔드에서만 띄워야 하는 건 아닙니다. 백그라운드 작업이 끝났을 때 백엔드(Rust)에서 창을 강제로 닫거나 띄울 수도 있습니다.

```rust
// src-tauri/src/lib.rs
use tauri::Manager;

#[tauri::command]
fn close_settings(app: tauri::AppHandle) {
    // "settings-window"라는 라벨을 가진 창을 찾아서 닫음
    if let Some(window) = app.get_webview_window("settings-window") {
        window.close().unwrap();
    }
}
```

---

## 🚀 마무리 및 다음 단계

위처럼 `WebviewWindow` 객체를 이용하면 디자인이 없는 투명한 창(transparent: true), 테두리가 없는 창(decorations: false) 등을 조합하여 바탕화면에 떠다니는 위젯 같은 UI도 만들 수 있습니다.

"근데 앱을 최소화하거나 닫았을 때, 카카오톡처럼 작업 표시줄(시간 옆 트레이)로 숨게 만들 수는 없을까?"
다음 장 [**12. 시스템 트레이와 전역 단축키**](./12-system-tray-shortcuts.md)에서 데스크톱 앱의 백그라운드 실행을 완성해보겠습니다.
