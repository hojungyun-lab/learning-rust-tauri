# 🎮 예제 코드 (Examples)

이 디렉터리에는 학습 문서(`docs/`)에서 배운 내용을 토대로 완성된 **실행 가능한 Tauri v2 데모 애플리케이션**들이 들어있습니다.

## 1. basic-app (기초 통합 데모)
- **목적:** 00~06장까지의 기초 학습 내용을 통합하여 테스트해 볼 수 있는 Svelte + TypeScript 프로젝트입니다.
- **주요 기능:**
  - 프론트엔드와 백엔드 간 Data 통신 (IPC Commands)
  - 글로벌 상태 카운터 처리 (State & Mutex)
  - 채널을 통한 로딩 바 비동기 수신 (Events)
- **실행 방법:**
  ```bash
  cd basic-app
  pnpm install
  pnpm tauri dev
  ```

## 2. final-project (실전 심화 완성 앱)
- **목적:** 07~13장까지의 고급 API를 모두 접목시켜 만든 모던 데스크톱 애플리케이션 뼈대입니다. (React + TS 기반)
- **주요 기능:**
  - `plugin-sql`을 활용한 로컬 SQLite DB CRUD 연동
  - 파일 시스템 및 네이티브 다이얼로그 접근 (Capabilities 제어 포함)
  - 백그라운드 구동을 위한 System Tray 아이콘 및 메뉴 제어
  - 서브 윈도우 팝업 동적 생성
- **실행 방법:**
  ```bash
  cd final-project
  pnpm install
  pnpm tauri dev
  ```
