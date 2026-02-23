# 🏭 13. CI/CD 파이프라인 (GitHub Actions)

## 🎯 학습 목표 (Goal)
GitHub Actions를 활용하여 내 컴퓨터의 OS와 상관없이 코드를 푸시(Push)할 때마다 **Windows(.exe), macOS(.dmg), Linux(.AppImage)** 3대장 데스크톱 앱을 클라우드에서 동시에 자동 빌드(Cross-platform Build)하는 파이프라인을 구축합니다.

---

## 💡 핵심 개념 (Core Concepts)

[15장](./15-build-and-deploy.md)에서 배웠듯, Rust와 Tauri는 해당 실행 파일이 돌아갈 목표 운영체제 환경과 동일한 툴체인(환경)에서 빌드되는 것을 권장합니다.
내가 Mac 기기만 가지고 있더라도, 코드를 GitHub에 업로드만 하면 GitHub가 제공하는 Windows와 Ubuntu 가상 서버(Runner)가 내 코드를 내려받아 각각 컴파일하고 배포용 설치 파일로 구워주는 것이 이번 장의 목표입니다.

---

## 💻 실습: GitHub Actions 워크플로 만들기 (Hands-on)

### Step 1: 자동화 스크립트 파일 생성
Tauri 공식에서 권장하는 액션(`tauri-apps/tauri-action`)을 사용하면 매우 쉽게 구축할 수 있습니다.
프로젝트 최상단 루트 폴더에서 `.github/workflows/` 폴더를 만들고 `release.yml` 이라는 파일을 생성합니다.

```yaml
# .github/workflows/release.yml 파일
name: "Release_Tauri_App"

# GitHub에 "v1.0.0" 같은 태그(Tag)를 달아서 푸시했을 때만 이 자동화 공장이 돌아갑니다.
on:
  push:
    tags:
      - "v*"

jobs:
  release:
    # 3가지 환경에서 동시에 돌린다는 매트릭스(Matrix) 전략
    strategy:
      fail-fast: false
      matrix:
        platform: [macos-latest, ubuntu-22.04, windows-latest]

    runs-on: ${{ matrix.platform }}
    steps:
      # 1. 내 레포지토리 코드 다운로드 (Checkout)
      - uses: actions/checkout@v4

      # 2. NodeJS 환경 세팅 (pnpm install 용도)
      - name: setup node
        uses: actions/setup-node@v4
        with:
          node-version: 24

      # 3. Rust 환경 세팅 (최신 안정화 버전)
      - name: install Rust stable
        uses: dtolnay/rust-toolchain@stable

      # 4. Linux(Ubuntu) 환경일 때만 필요한 추가 C++ 의존성 설치
      - name: install dependencies (ubuntu only)
        if: matrix.platform == 'ubuntu-22.04'
        run: |
          sudo apt-get update
          sudo apt-get install -y libgtk-3-dev libwebkit2gtk-4.0-dev libappindicator3-dev librsvg2-dev patchelf

      # 5. pnpm 설치
      - name: install pnpm
        run: npm install -g pnpm

      # 6. 프론트엔드 의존성 설치
      - name: install frontend dependencies
        run: pnpm install

      # 6. 😎 대망의 Tauri 공식 액션! 
      # 이 액션 블록이 알아서 `tauri build`를 치고 결과물을 끌어옵니다.
      - uses: tauri-apps/tauri-action@v0
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          # 여기서 만들어진 .exe, .dmg 파일등을 
          # 깃헙 저장소 우측의 [Releases] 페이지 초안에 자동으로 업로드해줌!
          tagName: 앱 릴리스 ${{ github.ref_name }}
          releaseName: "App Release ${{ github.ref_name }}"
          releaseBody: "자동화된 빌드 버전입니다."
          releaseDraft: true # 바로 배포하지 않고 사용자가 확인 후 [배포] 누르도록 초안으로 설정
```

### Step 2: GitHub에 업로드 및 실행

1. 코드를 Commit 합니다.
2. Git Tag를 하나 만듭니다. (터미널에서 `$ git tag v1.0.0` 입력)
3. 코드를 푸시하고 태그도 푸시합니다. (`$ git push origin main && git push origin v1.0.0`)
4. 여러분의 GitHub Repository 웹페이지 상단의 **[Actions]** 탭에 들어가면, 금방 만든 `Release_Tauri_App` 작업이 빙글빙글 돌며 OS 3개 채널에서 동시에 컴파일을 땀흘리며 하고 있는 것을 볼 수 있습니다. (Rust 빌드 특성상 15~20분 정도 소요될 수 있습니다)

작업이 완료되면 레포지토리의 **[Releases]** 메뉴로 가보세요!
그곳에서 `.exe`, `.msi`, `.dmg` 파일들이 사람들의 클릭을 기다리고 있을 것입니다.

---

## 🎉 마무리 총평! (Conclusion)

여기까지 완주하셨다면 여러분은 **완벽한 하나의 독립된 크로스플랫폼 데스크톱 앱을 혼자서 처음부터 배포까지 구축할 수 있는 풀스택 프레젠터**가 된 것입니다.
`Node.js(TS)`의 엄청난 UI 자유도 발전 속도와, `Rust`의 무시무시한 퍼포먼스(빠른 속도, 작은 용량)가 결합된 Tauri는 현재 데스크톱 프레임워크 1황으로 자리잡아 가고 있습니다.

이제 여러분만의 창의적인 데스크톱 아이디어를 Tauri 위에서 구현해보세요! 문서 중간에 막히면 언제든 기본으로 돌아와 `CHEATSHEET.md`와 각 기초 단계 문서를 열어보시면 됩니다.

> 🎓 **진짜 실전 코드가 궁금하신가요?**
> 지금부터는 글로 배우는 걸 멈추고 직접 코드를 씹어먹어볼 차례입니다.
> `examples/` 디렉터리에 있는 `basic-app` 프로젝트를 살펴보며 문서에서 배운 내용들이 어떻게 결합되어 있는지 전체 그림을 확인해보세요.
