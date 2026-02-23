# 🦀 Rust & Tauri 마스터 클래스: 기초부터 실전 배포까지

## 📖 프로젝트 개요
이 레포지토리는 웹 기술(HTML, CSS, JS/TS)과 **Rust**의 강력함을 결합하여, 가볍고 빠르며 메모리 안전한 크로스 플랫폼 데스크톱 애플리케이션을 개발하는 프레임워크인 **Tauri(v2 기준)**를 마스터하기 위한 종합 학습 가이드입니다.
단순한 문법 나열을 넘어, 빈 폴더에서 시작해 실제 동작하는 애플리케이션을 구축하고 배포하는 과정까지 현업 실무 중심의 모범 사례(Best Practice)를 제공합니다.

---

## 📂 프로젝트 구조
```text
.
├── README.md             // 프로젝트 소개, 시작 가이드, 전체 학습 목차 플래너
├── CHEATSHEET.md         // 핵심 문법 및 패턴 위주의 빠른 참조 요약 카드
├── docs/                 // 00번부터 시작하는 단계별 학습 마크다운 문서들
│   ├── 00-environment-setup.md
│   ├── 01-tauri-architecture.md
│   └── ... 
└── examples/             // 학습자가 참고할 수 있는 완성본 데모 코드
    ├── basic-app/        // 기초 단계(00~중반)를 통합한 데모 앱
    └── final-project/    // 심화 단계(후반)를 적용한 실전 완성형 앱
```

---

## 🚀 시작하기

### 1️⃣ 필수 환경 설정 확인
Tauri 개발을 위해서는 **Node.js**와 **Rust** 컴파일러 환경이 모두 필요합니다.
프로젝트별로 독립적인 Node.js 버전을 사용할 수 있도록 **nvm(Node Version Manager)**을 통해 설치하는 것을 권장합니다.

```bash
# 1. nvm 설치 (이미 설치되어 있다면 생략)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash

# 2. nvm으로 Node.js LTS 버전 설치 및 활성화
nvm install --lts
nvm use --lts

# 3. 설치 확인
node -v   # 예: v24.x.x
npm -v    # 예: 11.x.x

# 4. pnpm 설치 (이 프로젝트의 패키지 매니저)
npm install -g pnpm
pnpm -v   # 예: 10.x.x

# 5. Rust 컴파일러 및 패키지 매니저 확인
rustc --version   # Rust 컴파일러: .rs 소스 코드를 네이티브 바이너리로 변환
cargo --version   # Rust 패키지 매니저 겸 빌드 도구: 의존성 관리, 빌드, 테스트 실행 등
```
> 💡 프로젝트 루트에 `.nvmrc` 파일을 만들어 Node 버전을 고정하면, `nvm use` 한 줄로 팀원 모두가 동일한 환경을 재현할 수 있습니다.

> 환경이 아직 준비되지 않았다면 [00. 환경 구축 가이드](docs/00-environment-setup.md)를 먼저 참고하여 설치를 완료하세요.

### 2️⃣ 새로운 Tauri 프로젝트 템플릿 생성
빈 프로젝트를 직접 구축하며 학습하고 싶다면, 아래의 명령어를 통해 CLI 스캐폴딩을 시작하세요:
```bash
# 최신 Tauri 앱 생성 마법사 실행
pnpm create tauri-app@latest

# 프롬프트 안내에 따라 아래와 같이 환경을 선택합니다:
# ✔ Project name · my-learning-app
# ✔ Identifier · com.hojungyun.my-learning-app
# ✔ Choose which language to use for your frontend · TypeScript / JavaScript
# ✔ Choose your package manager · pnpm
# ✔ Choose your UI template · Svelte
# ✔ Choose your UI flavor · TypeScript

cd my-learning-app
pnpm install      # 프론트엔드 의존성 설치
pnpm tauri dev    # Tauri 앱(프론트+백엔드) 로컬 개발 서버 실행!
# ⚠️ 최초 실행 시 ".svelte-kit/tsconfig.json을 찾을 수 없다"는 경고가 뜰 수 있습니다.
#    이는 SvelteKit이 첫 빌드 시 자동 생성하는 폴더이므로 무시해도 됩니다. 두 번째 실행부터 사라집니다.
```

### 3️⃣ 데모(Examples) 코드 직접 실행해보기
이 레포지토리에 포함된 완성된 코드를 분석하고 싶다면 `examples/` 디렉터리를 활용하세요.
```bash
# 기초 데모 앱 실행
cd examples/basic-app
pnpm install
pnpm tauri dev
```

---

## 📚 학습 목차 (커리큘럼)

초보자도 차근차근 따라올 수 있도록 개념별로 단계를 세분화했습니다. 각 문서를 순장적으로 학습하는 것을 권장합니다.

| 번호 | 범주 | 주제 | 핵심 내용 |
|---|---|---|---|
| `00` | **기초 단계** | [환경 구축 (Environment Setup)](docs/00-environment-setup.md) | OS별 Rust & Tauri 기본 개발 환경 세팅 |
| `01` | 기초 단계 | [Tauri 아키텍처와 생명주기](docs/01-tauri-architecture.md) | 프론트엔드와 웹뷰, Core 프로세스의 연동 체계 이해 |
| `02` | **Rust 언어** | [Rust 기초 문법](docs/02-rust-basics.md) | 변수, 타입, 함수, 제어흐름, match 패턴 매칭 |
| `03` | Rust 언어 | [소유권과 참조](docs/03-ownership-borrowing.md) | 소유권 3대 규칙, 빌림(&T), String vs &str, 라이프타임 |
| `04` | Rust 언어 | [구조체, 열거형, 트레이트](docs/04-structs-enums-traits.md) | struct, enum, impl, trait, derive 매크로, 제네릭(`<T>`) |
| `05` | Rust 언어 | [컬렉션, 클로저, 모듈](docs/05-collections-closures-modules.md) | Vec, HashMap, 클로저/move, mod/use, Cargo.toml |
| `06` | **프레임워크 핵심** | [IPC & Commands 통신](docs/06-ipc-and-commands.md) | TS 프론트엔드 ↔ Rust 백엔드 브릿징, 비동기 데이터 통신 |
| `07` | 프레임워크 핵심 | [상태 관리 (State Management)](docs/07-state-management.md) | Rust 런타임에서의 스레드 안전한(Mutex) 글로벌 상태 주입 및 제어 |
| `08` | 프레임워크 핵심 | [이벤트 시스템 (Event System)](docs/08-event-system.md) | Emit & Listen을 활용한 프론트/백엔드 간 양방향 이벤트 분배 |
| `09` | 프레임워크 핵심 | [에러 핸들링 및 디버깅](docs/09-error-handling.md) | Rust `Result`를 프론트엔드 예외로 맵핑하여 우아하게 처리하기 |
| `10` | **고급 & 실전** | [네이티브 모듈 (fs, dialog)](docs/10-native-apis.md) | 파일 시스템 읽기/쓰기 및 OS 네이티브 팝업, 알림 호출 |
| `11` | 고급 & 실전 | [멀티 윈도우 관리](docs/11-multi-window.md) | 동적으로 새로운 창 띄우기 및 다중 윈도우 간 통신 라우팅 |
| `12` | 고급 & 실전 | [시스템 트레이와 전역 단축키](docs/12-system-tray-shortcuts.md) | 트레이(메뉴바) 아이콘 등록 및 백그라운드 상태 유지를 위한 단축키 |
| `13` | 고급 & 실전 | [보안 및 Capabilities (v2 API)](docs/13-security-capabilities.md) | 플러그인 접근 권한 제어와 안전한 IPC 타겟 화이트리스트 설정 |
| `14` | 고급 & 실전 | [로컬 데이터베이스 (SQLite)](docs/14-database-sqlite.md) | `tauri-plugin-sql`을 통해 로컬 앱에 영속적 데이터 저장 최적화 |
| `15` | 고급 & 실전 | [앱 패키징 및 최적화](docs/15-build-and-deploy.md) | 프로덕션 릴리스 빌드, 커스텀 아이콘 등록, 번들 용량 다이어트 |
| `16` | 고급 & 실전 | [CI/CD 파이프라인 (GitHub Actions)](docs/16-ci-cd.md) | Mac, Windows, Linux 바이너리 동시 빌드 자동화 구축 |

---

## 📋 빠른 참조 (Cheatsheet)
각 가이드를 학습하는 도중 즉각적인 힌트가 필요할 땐 [CHEATSHEET.md](./CHEATSHEET.md)를 확인하세요. 에러 처리, Command 등록 등 가장 자주 쓰이는 코드 스니펫만 모아 두었습니다.
