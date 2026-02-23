# 🚀 Rust & Tauri (v2) 핵심 요약 치트시트 (Cheatsheet)

개발 도중 자주 마주치는 핵심 패턴과 문법을 빠르게 찾아볼 수 있는 요약 카드입니다.

---

## 0️⃣ Rust 기초 문법 (변수, 타입, 함수, 소유권, 구조체)

```rust
// ── 변수 선언 ──
let x = 5;              // 불변 (기본)
let mut y = 10;          // 가변 (mut 필수)
const MAX: u32 = 100;    // 상수 (타입 명시 필수)

// ── 기본 타입 ──
let n: i32 = -42;        // 부호 있는 정수
let u: u32 = 42;         // 부호 없는 정수
let f: f64 = 3.14;       // 실수
let b: bool = true;
let c: char = '🦀';

// ── 함수 ──
fn add(a: i32, b: i32) -> i32 {
    a + b   // 세미콜론 없음 = 반환 (Expression)
}

// ── 소유권 & 참조 ──
let s1 = String::from("hello");
let s2 = s1;              // Move! s1 무효화
let s3 = s2.clone();      // 깊은 복사. 둘 다 유효

fn borrow(s: &String) {}  // 불변 참조: 읽기만
fn mutate(s: &mut String) { s.push_str("!"); } // 가변 참조: 수정 가능

// ── 구조체 & impl ──
#[derive(Debug, Clone, serde::Serialize)]
struct User { name: String, age: u32 }

impl User {
    fn new(name: &str, age: u32) -> Self { Self { name: name.into(), age } }
    fn greet(&self) -> String { format!("{}님 안녕!", self.name) }
}

// ── Enum & match ──
enum Status { Active, Inactive(String) }
let s = Status::Active;
match s {
    Status::Active => println!("활성"),
    Status::Inactive(reason) => println!("비활성: {}", reason),
}

// ── 클로저 & move ──
let double = |x: i32| x * 2;
let data = String::from("hello");
std::thread::spawn(move || { println!("{}", data); }); // move = 소유권 이동

// ── 컬렉션 ──
let mut v: Vec<i32> = vec![1, 2, 3];
v.push(4);
use std::collections::HashMap;
let mut map = HashMap::new();
map.insert("key", "value");

// ── 모듈 & use ──
mod my_module { pub fn helper() {} }
use std::sync::Mutex;
```

---

## 1️⃣ Rust 에러 핸들링 (Tauri 백엔드 핵심)

Rust의 안정성은 `Result`와 `Option`에서 시작됩니다.

```rust
// ✅ Option: 값이 있거나 없을 수 있음 (Null 대용)
let some_val: Option<i32> = Some(5);
let no_val: Option<i32> = None;

// ✅ Result: 성공(`Ok`)하거나 실패(`Err`)함 -> Tauri Command에서 중요!
fn divide(a: f64, b: f64) -> Result<f64, String> {
    if b == 0.0 {
        Err("0으로 나눌 수 없습니다.".into()) // 실패 시 에러 텍스트
    } else {
        Ok(a / b) // 성공 시 계산된 값
    }
}

// ✅ `?` 연산자: 에러가 발생하면 즉시 호출자에게 Err를 반환 (Early Return)
fn process_math() -> Result<(), String> {
    let res = divide(10.0, 2.0)?; // 만약 Err면 여기서 함수 종료 후 Err 반환
    println!("결과값: {}", res); // Ok일 때 값(5.0)이 `res`에 바인딩
    Ok(())
}
```

---

## 2️⃣ IPC Command (프론트 ↔ 백엔드 통신)

### Rust 백엔드에 커맨드 등록하기 (`src-tauri/src/main.rs`)
```rust
// 1. #[tauri::command] 매크로로 JS에서 호출할 함수를 정의합니다.
#[tauri::command]
fn greet(name: &str) -> String {
    format!("안녕하세요, {}님! Rust에서 보낸 환영 인사입니다.", name)
}

fn main() {
    tauri::Builder::default()
        // 2. 정의한 함수를 핸들러 배열에 반드시 등록해야 동작합니다!
        .invoke_handler(tauri::generate_handler![greet])
        .run(tauri::generate_context!())
        .expect("Tauri 앱 실행 중 오류 발생");
}
```

### 프론트엔드에서 Rust 커맨드 호출하기 (TypeScript)
```typescript
import { invoke } from '@tauri-apps/api/core';

async function callRustGreeting() {
  try {
    // 첫 번째 인자는 Rust 커맨드명(string), 두 번째는 매개변수 객체
    const message: string = await invoke('greet', { name: '개발자' });
    console.log(message);
  } catch (error) {
    console.error("Rust 커맨드 실행 실패:", error);
  }
}
```

### 프론트엔드로 Custom Error 직렬화해서 던지기
```rust
use serde::Serialize;

// JS로 에러를 보내기 위해 `Serialize` 필요
#[derive(Serialize)]
struct AppError {
    message: String,
    code: u16,
}

#[tauri::command]
fn perform_action(should_fail: bool) -> Result<String, AppError> {
    if should_fail {
        Err(AppError {
            message: "의도된 작업 실패".into(),
            code: 500,
        })
    } else {
        Ok("데이터 처리 성공".into())
    }
}
```

---

## 3️⃣ 글로벌 상태 관리 (State Management)

Tauri 앱이 동작하는 동안 계속 유지되어야 하는 데이터(예: DB 연결 풀, 누적 카운터)를 다룰 때 사용합니다. 다중 스레드 접근을 위해 `Mutex`나 `RwLock`이 필수입니다.

```rust
use std::sync::Mutex;
use tauri::State;

// 1. 상태를 담을 구조체.
struct AppState {
    // 값이 변경되어야 하므로 Mutex 사용
    click_count: Mutex<i32>,
}

#[tauri::command]
// 2. 인자에 `State<'_, 구조체명>`을 넣어 Tauri가 알아서 의존성을 주입하게 합니다.
fn increment_count(state: State<'_, AppState>) -> Result<i32, String> {
    // 락(Lock) 획득
    let mut count = state.click_count.lock().map_err(|_| "상태 잠금 실패")?;
    *count += 1;
    Ok(*count)
}

fn main() {
    tauri::Builder::default()
        // 3. 앱 빌더 시작 시 초기 상태값을 manage에 등록합니다.
        .manage(AppState {
            click_count: Mutex::new(0),
        })
        .invoke_handler(tauri::generate_handler![increment_count])
        .run(tauri::generate_context!())
        .expect("error");
}
```

---

## 4️⃣ 이벤트 시스템 (Event System)

요청-응답 형태가 아니라, 한쪽에서 일방적으로 알려주거나(Emit) 구독하는(Listen) 경우 사용합니다.

### Rust 백엔드 ➡️ Frontend 송신 (Emit)
```rust
use tauri::{AppHandle, Emitter};
use std::{thread, time::Duration};

#[tauri::command]
fn start_long_task(app: AppHandle) {
    // 메인 스레드가 블록되지 않도록 백그라운드로 분리
    thread::spawn(move || {
        thread::sleep(Duration::from_secs(3));
        // "task-finished"라는 이름표를 달고 페이로드(문자열) 전송
        app.emit("task-finished", "백그라운드 작업이 완료되었습니다!").unwrap();
    });
}
```

### Frontend 이벤트 수신 및 정리 (Listen)
```typescript
import { listen } from '@tauri-apps/api/event';
import { onMount, onDestroy } from 'svelte'; // 프레임워크별 라이프사이클 다름

let unlisten: () => void;

onMount(async () => {
  // 이벤트 이름 바인딩
  unlisten = await listen<string>('task-finished', (event) => {
    console.log("알림 수신:", event.payload);
  });
});

onDestroy(() => {
  // 컴포넌트가 사라질 때 메모리 누수를 막기 위해 구독 해제(Unlisten)
  if (unlisten) unlisten();
});
```

---

## 5️⃣ 앱 생명주기 제어 (Setup Hook)

앱의 메인 창이 로딩되기 전/후에 초기화(플러그인 셋업, 로깅 초기화 등)를 실행할 때 사용합니다.

```rust
use tauri::Manager;

fn main() {
    tauri::Builder::default()
        .setup(|app| {
            // 앱 구동 시 단 한번 실행
            let window = app.get_webview_window("main").unwrap();
            
            println!("애플리케이션이 초기화 중입니다...");
            
            Ok(()) // 중요: setup은 Result<(), Box<dyn std::error::Error>>를 반환해야 함
        })
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```
