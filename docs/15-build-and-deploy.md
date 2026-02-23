# 📦 12. 앱 패키징 및 운영체제별 빌드

## 🎯 학습 목표 (Goal)
개발을 완료한 Tauri 프로젝트를 배포용(Production)으로 최적화하여 빌드하고, 사용자에게 전달할 수 있는 무설치 바이너리나 설치 마법사(Installer) 파일(`.exe`, `.dmg`, `.app`)을 생성하는 방법을 배웁니다.

---

## 💡 핵심 개념 (Core Concepts)

### 웹사이트 빌드 vs 데스크톱 앱 빌드
- **웹 빌드 (vite build, webpack 등):** 단순히 자바스크립트 파일과 HTML, CSS를 압축(Minify)하는 과정.
- **Tauri 빌드 (`tauri build`):** 프론트엔드를 먼저 웹 빌드(압축)한 뒤, 그 결과물을 Rust 컴파일러가 OS 바이너리와 융합시켜 하나의 독립된 실행 파일(Executable)이나 설치용 패키지 마법사로 굽는(Bake) 과정입니다.

Tauri는 **"교차 컴파일(Cross-compilation)을 100% 완벽히 지원하지 않습니다."**
즉, Mac 기기에서는 `.app`과 `.dmg`만 만들 수 있고, Windows 기기에서는 `.exe`와 `.msi`만 만들 수 있습니다. 
다른 OS 버전을 뽑아내려면 [16. CI/CD 자동화](./16-ci-cd.md)가 필요합니다. 하지만 현재 여러분이 쓰는 OS 버전을 뽑아내는 건 명령어 한 줄이면 끝납니다.

---

## 💻 실습: 릴리스 빌드 (Hands-on)

앱을 실제로 구워서 폴더에서 더블클릭 해봅시다!

### Step 1: 앱 식별자(Identifier) 수정
가장 먼저 `src-tauri/tauri.conf.json` 파일을 엽니다. 초기 세팅 시 식별자가 `com.tauri.dev`로 잡혀있어 빌드가 거절될 확률이 높습니다.

```json
{
  "productName": "My Awesome App",
  "version": "1.0.0",
  "identifier": "com.mycompany.myawesomeapp", // 👈 이걸 본인만의 독창적인 영문자로 변경!
  "build": {
    "beforeDevCommand": "pnpm dev",
    "beforeBuildCommand": "pnpm build", // 👈 프론트엔드(Vite 등) 빌드 명령어 확인
    "devUrl": "http://localhost:1420",
    "frontendDist": "../dist"
  },
  "bundle": {
    "active": true,
    "icon": [
      "icons/32x32.png",
      "icons/128x128.png",
      "icons/128x128@2x.png",
      "icons/icon.icns",
      "icons/icon.ico"
    ]
  }
}
```

### Step 2: 빌드 명령어 실행
터미널에 다음 명령어를 입력합니다. 
(최초 빌드 시 모든 의존성 Crate를 릴리스 최적화용으로 처음부터 다시 컴파일하므로 **5분에서 그 이상** 매우 오래 걸릴 수 있습니다! 컴퓨터 팬이 세게 돌아가는 건 정상입니다.)

```bash
pnpm tauri build
```

### Step 3: 완성된 결과물 찾기
빌드가 다 끝나면 터미널에 초록색 글씨로 결과물의 위치가 출력됩니다. 보통 경로는 다음과 같습니다.

- **macOS:** `src-tauri/target/release/bundle/macos/My Awesome App.app` 
            (그리고 디스크 이미지 `.dmg` 파일도 확인 가능합니다)
- **Windows:** `src-tauri\target\release\bundle\nsis\MyAwesomeApp_1.0.0_x64-setup.exe`
            (그리고 단일 실행파일도 확인 가능합니다)

해당 경로로 가서 여러분이 만든 앱 아이콘을 직접 더블클릭해서 켜보세요. 프레임워크 런타임 없이도 번개처럼 빠르게 켜지는 놀라운 속도를 체감할 수 있습니다!

---

## 🛠 단축 팁: 앱 아이콘 커스텀하기

Tauri가 기본으로 제공한 징그러운 해골/톱니 아이콘을 버리고 내 아이콘을 넣고 싶나요?
Tauri CLI가 **원본 PNG 한 장만 있으면 자동으로 사이즈별(윈도우용 ico, 맥용 icns 등) 변환**을 해줍니다.

1. `1024x1024` 이상 크기의 정사각형 `app-icon.png` 이미지를 준비.
2. 터미널(Node.js가 설치된 상태)에서 실행:
   ```bash
   pnpm tauri icon /경로/app-icon.png
   ```
3. 자동으로 `src-tauri/icons` 폴더 안의 모든 아이콘이 여러분의 로고로 교체됩니다. 끝!

---

## 🚀 마무리 및 다음 단계

축하합니다! 이제 여러분은 윈도우나 맥에서 즉시 배포할 수 있는 초고속 네이티브 앱 제작 스크립트를 마스터하셨습니다. (보통 번들 사이즈가 10MB 남짓일 겁니다. Electron의 150MB에 비하면 엄청난 다이어트죠.)

그런데 방금 얘기했듯, 저는 맥북을 쓰는데 제 친구는 윈도우 컴퓨터를 쓴다면 친구에게는 앱을 전달할 수 없을까요? 내가 굳이 윈도우 컴퓨터를 한 대 더 사야 할까요?
전혀 아닙니다. 다음 장 [**16. CI/CD 파이프라인 (GitHub Actions)**](./16-ci-cd.md)에서 구글(GitHub)의 서버를 빌려 Mac, Windows, Linux 버전 3가지를 동시에 뽑아내는 클라우드 공장을 차리는 법을 알아봅니다.
