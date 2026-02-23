# 📚 05. 컬렉션, 클로저, 모듈 (Collections, Closures, Modules)

## 🎯 학습 목표 (Goal)
동적 크기의 데이터를 다루는 **컬렉션(Vec, HashMap)**, 함수를 변수처럼 다루는 **클로저(Closure)와 move**, 코드를 파일별로 나누는 **모듈(mod/use) 시스템**, 그리고 Rust 의존성의 핵심인 **Cargo.toml** 관리법을 학습합니다.

---

## 💡 핵심 개념 (Core Concepts)

---

### 1. `Vec<T>` — 가변 길이 배열 (가장 많이 쓰는 컬렉션!)

JS의 `Array`에 대응하며, 크기가 자유롭게 늘어나고 줄어듭니다.

> **🐍 Python과 비교:** Python의 `list`와 거의 동일합니다. 차이점은 Rust의 `Vec<T>`는 **한 가지 타입만** 담을 수 있다는 점입니다 (Python의 `list`는 `[1, "hello", True]`처럼 막 섞어도 됨).
> | Python | Rust | 설명 |
> |---|---|---|
> | `nums = []` | `let mut nums: Vec<i32> = Vec::new();` | 빈 리스트 생성 |
> | `nums.append(10)` | `nums.push(10);` | 요소 추가 |
> | `nums[0]` | `nums[0]` | 인덱스 접근 |
> | `[x*2 for x in nums]` | `nums.iter().map(\|x\| x*2).collect()` | 함수형 변환 |

```rust
fn main() {
    // ✅ 생성 방법 3가지
    let mut nums: Vec<i32> = Vec::new();      // 1) 빈 벡터 생성 (타입 명시)
    let fruits = vec!["사과", "바나나", "체리"]; // 2) 매크로로 초기값과 함께 생성
    let zeros = vec![0; 10];                   // 3) 초기값 0을 10개로 채운 벡터

    // ✅ 요소 추가/접근
    nums.push(10);
    nums.push(20);
    nums.push(30);
    println!("첫 번째: {}", nums[0]);    // 인덱스 접근 (범위 밖이면 패닉!)
    println!("안전하게: {:?}", nums.get(99)); // .get()은 Option<&T> 반환 (안전!)

    // ✅ 순회(Iteration) — 가장 자주 쓰는 패턴
    for num in &nums {
        println!("값: {}", num);
    }

    // ✅  메서드 체이닝 (함수형 프로그래밍 스타일)
    let doubled: Vec<i32> = nums.iter()
        .map(|x| x * 2)         // 각 요소를 2배
        .filter(|x| *x > 25)    // 25 초과만 남김
        .collect();              // 결과를 다시 Vec으로 모음
    println!("결과: {:?}", doubled); // [40, 60]
}
```

---

### 2. `HashMap<K, V>` — 키-값 저장소

JS의 `Map` 또는 `Object`에 대응합니다. 설정값 관리, 캐시 등에 활용됩니다.

> **🐍 Python과 비교:** Python의 `dict`와 동일한 역할입니다.
> | Python | Rust |
> |---|---|
> | `scores = {}` | `let mut scores = HashMap::new();` |
> | `scores["Alice"] = 95` | `scores.insert("Alice".into(), 95);` |
> | `scores.get("Alice")` | `scores.get("Alice")` → `Option<&V>` |
> | `scores.setdefault("Bob", 0)` | `scores.entry("Bob".into()).or_insert(0);` |

```rust
use std::collections::HashMap; // 별도 use 선언 필요!

fn main() {
    let mut scores: HashMap<String, i32> = HashMap::new();

    // ✅ 삽입
    scores.insert("Alice".to_string(), 95);
    scores.insert("Bob".to_string(), 87);

    // ✅ 조회 — .get()은 Option<&V> 반환
    match scores.get("Alice") {
        Some(score) => println!("Alice 점수: {}", score),
        None => println!("Alice를 찾을 수 없습니다"),
    }

    // ✅ 없을 때만 삽입 (중복 방지)
    scores.entry("Charlie".to_string()).or_insert(100);

    // ✅ 순회
    for (name, score) in &scores {
        println!("{}: {}점", name, score);
    }
}
```

---

### 3. 클로저 (Closure) — 변수를 캡처하는 익명 함수

JS의 화살표 함수(`=>`)와 비슷하지만, Rust는 외부 변수를 "빌리기/이동하기"까지 세밀하게 제어합니다.

> **🐍 Python과 비교:** Python의 `lambda`와 비슷하지만 Rust 클로저는 **여러 줄**이 가능하고 훨씬 강력합니다.
> | Python | Rust |
> |---|---|
> | `double = lambda x: x * 2` | `let double = \|x\| x * 2;` |
> | `add = lambda a, b: a + b` | `let add = \|a, b\| a + b;` |
> | *(괴활호 안 한 준만 가능)* | `\|x\| { 여러 줄; 로직 }` (여러 줄 가능) |

```rust
fn main() {
    // ✅ 기본 클로저 문법: |인자| { 본문 }
    let add = |a: i32, b: i32| -> i32 { a + b };
    println!("결과: {}", add(3, 5)); // 8

    // 본문이 한 줄이면 중괄호 생략 가능
    let double = |x| x * 2;      // 타입도 추론됨!
    println!("2배: {}", double(7)); // 14

    // ✅ 외부 변수 캡처 (Capture)
    let greeting = String::from("안녕");
    let say = || println!("{}", greeting); // greeting을 불변 참조(&)로 빌림
    say();
    println!("{}", greeting); // ✅ 빌려서 보기만 했으므로 greeting 유효
}
```

---

### `move` 클로저 ⭐ — Tauri의 `thread::spawn`에서 필수!

```rust
use std::thread;

fn main() {
    let data = String::from("중요 데이터");

    // ❌ 일반 클로저: 다른 스레드에선 참조의 수명이 보장되지 않아 에러!
    // thread::spawn(|| { println!("{}", data); }); // 컴파일 에러

    // ✅ move 클로저: data의 소유권을 클로저(= 새 스레드) 안으로 '이동'시킴
    thread::spawn(move || {
        println!("다른 스레드에서: {}", data);
    }).join().unwrap();

    // println!("{}", data); // ❌ 소유권이 이동되었으므로 사용 불가
}
```

> **📌 08장(이벤트 시스템)에서 `thread::spawn(move || { app.emit(...) })` 를 사용했죠?**
> `move`는 `app` 핸들의 소유권을 새 스레드로 넘겨줘서 안전하게 사용하게 하는 키워드입니다.

---

### 4. 모듈 시스템 (`mod`, `use`, `pub`) — 코드를 파일별로 나누기

프로젝트가 커지면 모든 것을 `lib.rs` 한 파일에 담을 수 없습니다.

> **🐍 Python과 비교:**
> | Python | Rust | 설명 |
> |---|---|---|
> | `import os` | `use std::fs;` | 표준 라이브러리 임포트 |
> | `from os.path import join` | `use std::path::PathBuf;` | 특정 항목만 가져오기 |
> | `__init__.py` 파일로 모듈화 | `mod commands;` 로 파일 등록 | 모듈 선언 방식 |
> | 기본적으로 public | 기본적으로 **private** (`pub` 필요) | 접근 제어 |

#### 같은 파일 안에서의 모듈
```rust
// src-tauri/src/lib.rs

// 모듈 선언: 이름 공간(namespace)을 만듦
mod utils {
    // pub 키워드가 없으면 외부에서 접근 불가 (기본 비공개)
    pub fn greet(name: &str) -> String {
        format!("안녕, {}!", name)
    }

    fn internal_helper() {
        // 이 함수는 utils 모듈 밖에서 호출 불가 (private)
    }
}

fn main() {
    // 모듈 경로로 호출
    let msg = utils::greet("학습자");
    println!("{}", msg);
}
```

#### 별도 파일로 분리 (실전 프로젝트 필수 패턴)
```text
src-tauri/src/
├── lib.rs          // 메인 진입점
├── commands.rs     // 커맨드 모음 파일
└── models.rs       // 데이터 구조체 모음 파일
```

```rust
// src-tauri/src/commands.rs
pub fn greet(name: &str) -> String {
    format!("안녕, {}!", name)
}
```

```rust
// src-tauri/src/lib.rs
mod commands;  // commands.rs 파일을 모듈로 등록! (파일 이름 = 모듈 이름)

#[cfg_attr(mobile, tauri::mobile_entry_point)]
pub fn run() {
    tauri::Builder::default()
        .invoke_handler(tauri::generate_handler![commands::greet])
        .run(tauri::generate_context!())
        .expect("error");
}
```

> **`use` 키워드:** 경로를 짧게 축약할 때 사용합니다.
> ```rust
> use std::collections::HashMap; // 이후 HashMap만으로 사용 가능
> use serde::Serialize;          // Serialize만으로 사용 가능
> ```

---

### 5. `Cargo.toml` — Rust 의존성 관리 (npm의 `package.json`)

> **🐍 Python과 비교:**
> | Python | Rust |
> |---|---|
> | `requirements.txt` / `pyproject.toml` | `Cargo.toml` |
> | `pip install requests` | `cargo add reqwest` |
> | `site-packages/` | 전역 캐시 (`~/.cargo/`) |
> | `python main.py` | `cargo run` |
> | `pip freeze` | `cargo tree` |

```toml
# src-tauri/Cargo.toml

[package]
name = "my-tauri-app"
version = "0.1.0"
edition = "2021"    # Rust 에디션 (2021이 현재 표준)

[dependencies]
# Tauri 핵심 프레임워크
tauri = { version = "2", features = ["tray-icon"] }
# JSON 직렬화/역직렬화 (Tauri 통신의 근간)
serde = { version = "1", features = ["derive"] }
serde_json = "1"
# Tauri 플러그인들
tauri-plugin-sql = { version = "2", features = ["sqlite"] }
tauri-plugin-dialog = "2"
tauri-plugin-fs = "2"

[build-dependencies]
tauri-build = { version = "2", features = [] }
```

> **📌 npm과의 비교:**
> | npm (JS) | Cargo (Rust) |
> |---|---|
> | `package.json` | `Cargo.toml` |
> | `npm install serde` | `cargo add serde` |
> | `node_modules/` | 전역 캐시 (`~/.cargo/`) |
> | `npm run build` | `cargo build` |

---

## 🚀 마무리 및 다음 단계

Rust 기초 문법 시리즈(02~05장)를 모두 끝냈습니다! 🎉
이제 Tauri 코드에서 등장하는 거의 모든 Rust 문법을 해석할 수 있는 기반이 완성되었습니다.

이제 본격적으로 Tauri 프레임워크의 핵심 메커니즘으로 넘어갈 차례입니다.
**TypeScript 프론트엔드에서 Rust 백엔드 함수를 어떻게 호출하는지?**
다음 장 [**06. IPC & Commands 통신**](./06-ipc-and-commands.md)에서 브릿지(Bridge) 패턴의 진수를 배워봅시다!
