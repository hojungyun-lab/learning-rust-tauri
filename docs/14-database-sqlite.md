# 🗄️ 11. 로컬 데이터베이스 (SQLite)

## 🎯 학습 목표 (Goal)
Tauri의 공식 SQL 플러그인을 사용하여 로컬 환경에 SQLite 데이터베이스 파일을 생성하고, 프론트엔드에서 안전하게 쿼리(Query)를 실행하는 방법을 배웁니다.

---

## 💡 핵심 개념 (Core Concepts)

단순한 환경설정 값은 브라우저의 `localStorage`나 텍스트 파일(JSON)에 저장해도 충분합니다.
하지만 사용자의 채팅 기록, 가계부 내역, 수만 건의 로그 데이터 등을 저장하고 검색하려면 **관계형 데이터베이스(RDB)** 가 필수입니다.

Tauri는 앱 내부에 경량화된 DB인 **SQLite**를 구동할 수 있도록 `@tauri-apps/plugin-sql`을 제공합니다.
이 플러그인의 가장 큰 장점은 **Rust(백엔드) 코드를 직접 짤 필요 없이, 프론트엔드에서 바로 비동기로 SQL을 날릴 수 있다는 점**입니다. (내부적으로는 Rust를 거쳐 안전하게 실행됩니다).

---

## 💻 실습: 할 일(Todo) 테이블 만들고 데이터 추가하기 (Hands-on)

### Step 1: 플러그인 설치
SQLite용 드라이버를 지정하여 플러그인을 설치합니다. (MySQL, Postgres도 지원하지만 로컬 데스크톱 앱은 99% SQLite를 씁니다.)

```bash
pnpm tauri add sql
```
이후 `src-tauri/src/lib.rs` 파일의 `plugin()` 체인에 자동으로 `tauri_plugin_sql::Builder::new().build()` 코드가 삽입됩니다.

### Step 2: DB 파일 경로 설정 및 초기화
Tauri에서는 `sqlite:sqlite.db` 와 같이 이름을 지정하면 OS별 애플리케이션 전용 데이터 폴더(예: Windows의 `%APPDATA%`, macOS의 `~/Library/Application Support`)에 자동으로 DB 파일을 생성해 줍니다. 매우 안전하고 깔끔합니다.

### Step 3: 프론트엔드에서 SQL 쿼리 실행하기 (`src/routes/+page.svelte`)

```svelte
<!-- src/routes/+page.svelte -->
<script lang="ts">
  import Database from '@tauri-apps/plugin-sql';

  interface Todo {
    id: number;
    title: string;
    done: boolean;
  }

  let todos = $state<Todo[]>([]);

  async function initDatabase() {
    try {
      // 1️⃣ 데이터베이스 연결 (파일이 없으면 자동 생성됨)
      // 여기서 생성되는 'test.db'는 OS의 로컬 앱 데이터 폴더에 숨겨져 저장됩니다.
      const db = await Database.load('sqlite:test.db');

      // 2️⃣ 테이블 생성 구문 (최초 1회만 실행됨)
      await db.execute(`
        CREATE TABLE IF NOT EXISTS todos (
          id INTEGER PRIMARY KEY AUTOINCREMENT,
          title TEXT NOT NULL,
          done BOOLEAN DEFAULT 0
        )
      `);

      // 3️⃣ 데이터 삽입 (Execute)
      // SQL Injection 해킹 방지를 위해 무조건 배열 파라미터(Binding) $1, $2 방식을 사용해야 합니다!
      const result = await db.execute(
        'INSERT INTO todos (title, done) VALUES ($1, $2)',
        ['Tauri SQL 플러그인 학습하기', false]
      );
      console.log("추가된 데이터 행 수:", result.rowsAffected);

      // 4️⃣ 데이터 조회 (Select)
      // Select 결과는 지정한 객체 배열 형태로 깔끔하게 반환됩니다.
      todos = await db.select<Todo[]>(
        'SELECT * FROM todos WHERE done = $1',
        [false]
      );

      console.log("현재 남아있는 할 일:", todos);

    } catch (err) {
      console.error("데이터베이스 처리 중 에러 발생:", err);
    }
  }
</script>

<div>
  <button onclick={initDatabase}>DB 초기화 및 테스트</button>
  {#each todos as todo}
    <p>{todo.id}. {todo.title} - {todo.done ? '✅' : '⬜'}</p>
  {/each}
</div>
```

### 🔐 주의: Capabilities 설정
파일 읽기와 마찬가지로, SQL 플러그인도 허용 목록에 추가해야 동작합니다!
`src-tauri/capabilities/default.json`을 열람하고 아래가 있는지 확인하세요. (npm add 명령어 실행 시 보통 자동 추가됩니다.)
```json
"permissions": [
    "core:default",
    "sql:default" // 이거 필수!
]
```

---

## 🚀 마무리 및 다음 단계

`plugin-sql`을 사용하면 로컬 캐싱, 사용자 로그, 오프라인 우선(Offline-First) 동기화 앱 등 거의 모든 데스크톱 DB 요구사항을 충족할 수 있습니다.

이제 여러분의 앱은 기능적으로 완성되었습니다!
"그런데 친구나 고객에게 이 앱을 주려면 어떻게 해야 하죠? 소스 코드를 통째로 넘겨주고 `pnpm tauri dev`를 치라고 할 순 없잖아요."

맞습니다. 다음 장 [**15. 앱 패키징 및 최적화**](./15-build-and-deploy.md)에서 클릭 한 번으로 실행되는 깔끔한 `.exe`, `.app`, `.dmg` 설치 파일(Installer)을 만드는 법을 다룹니다.
