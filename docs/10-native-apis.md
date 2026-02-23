# 🖥️ 07. 네이티브 API (Native APIs) - 파일 시스템 및 다이얼로그

## 🎯 학습 목표 (Goal)
Tauri의 기본 플러그인(`@tauri-apps/plugin-fs`, `@tauri-apps/plugin-dialog`)을 사용하여 OS 고유의 파일 열기/저장 다이얼로그를 띄우고, 로컬 파일을 읽고 쓰는 방법을 배웁니다.

---

## 💡 핵심 개념 (Core Concepts)

브라우저의 JavaScript는 사용자의 로컬 하드디스크(`C:\`나 `/Users/`)에 직접 접근할 수 없지만, Tauri는 이를 허용합니다.
Rust 백엔드에 직접 커맨드를 작성하여 파일을 제어할 수도 있지만, **Tauri v2부터는 가장 자주 쓰이는 기능들을 공식 "플러그인(Plugin)" 형태로 제공**하여 프론트엔드 코드만으로도 네이티브 기능을 즉시 제어할 수 있게 되었습니다.

> **⚠️ 플러그인 생태계 (v2의 특징)**
> v1에서는 `@tauri-apps/api`에 모든 기능이 뭉쳐 있었지만, v2부터는 앱의 패키지 용량(번들 사이즈)을 줄이기 위해 필요한 플러그인만 선택적으로 설치(`pnpm tauri add fs` 등)하도록 변경되었습니다.

---

## 💻 실습: 파일 선택 및 내용 읽기 (Hands-on)

텍스트 파일을 선택하는 OS 기본 다이얼로그 창을 띄우고, 그 파일의 내용을 프론트엔드 화면에 출력해봅니다.

### Step 1: 플러그인 설치 (터미널)
다이얼로그(Dialog)와 파일 시스템(FS) 플러그인을 프로젝트에 추가합니다.

```bash
pnpm tauri add dialog
pnpm tauri add fs
```
*이 명령어를 치면 `src-tauri/Cargo.toml`과 `package.json`에 의존성이 자동 추가되고, `lib.rs`에 초기화 코드(`.plugin()`)가 자동으로 삽입됩니다!*

### Step 2: 프론트엔드 호출 로직 작성 (`src/routes/+page.svelte`)

```svelte
<!-- src/routes/+page.svelte -->
<script lang="ts">
  // 이제 특정 플러그인 패키지에서 함수를 임포트합니다.
  import { open } from "@tauri-apps/plugin-dialog";
  import { readTextFile } from "@tauri-apps/plugin-fs";

  let fileContent = $state('');

  async function openAndReadFile() {
    try {
      // 1️⃣ 파일을 선택하는 네이티브 창을 엽니다.
      const selectedPath = await open({
        multiple: false,       // 다중 선택 불가
        directory: false,      // 폴더가 아닌 파일만 선택
        filters: [{
          name: 'Text Files',  // 드롭다운에 표시될 이름
          extensions: ['txt', 'md', 'json'] // 허용할 확장자
        }]
      });

      // 사용자가 취소(Cancel)를 누르면 null이 반환됩니다.
      if (selectedPath === null) {
        console.log("파일 선택이 취소되었습니다.");
        return;
      }

      // 파일 경로는 string 또는 string[] 으로 올 수 있으므로 단일 파일 처리
      const filePath = Array.isArray(selectedPath) ? selectedPath[0] : selectedPath;
      console.log(`선택된 파일 경로: ${filePath}`);

      // 2️⃣ 선택한 파일의 텍스트 내용을 프론트엔드에서 직접 읽습니다!
      fileContent = await readTextFile(filePath);

    } catch (error) {
      console.error("파일 처리 중 오류:", error);
    }
  }
</script>

<div>
  <button onclick={openAndReadFile}>파일 열기</button>
  <pre>{fileContent}</pre>
</div>
```

### 🔐 주의: Security 타겟 (Capabilities)
위 코드를 바로 실행하면 `readTextFile`에서 권한 에러(Permission Denied)가 납니다. 
Tauri v2는 기본적으로 모든 파일 접근을 막아두었습니다!
이것을 해결하는 방법은 **[13. 보안 및 Capabilities](./13-security-capabilities.md)** 장에서 자세히 다룹니다. (빠른 테스트를 원한다면 `src-tauri/capabilities/default.json`에 fs 권한을 추가해야 합니다.)

---

## 🚀 마무리 및 다음 단계

Tauri의 공식 플러그인들을 조합하면 Rust 코드를 단 한 줄도 짜지 않고도 알림창 띄우기(`plugin-notification`), 클립보드 제어(`plugin-clipboard-manager`), 기기 진동(`plugin-haptics`) 등을 구현할 수 있습니다.

지금까지 우리는 오직 '메인 창' 하나에서만 작업했습니다.
클릭 시 새로운 팝업 창이나 설정 전용 창을 띄우려면 어떻게 할까요? 다음 장 [**11. 멀티 윈도우 관리**](./11-multi-window.md)를 살펴봅시다.
