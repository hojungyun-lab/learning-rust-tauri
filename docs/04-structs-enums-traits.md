# 🧱 04. 구조체, 열거형, 트레이트 (Structs, Enums, Traits)

## 🎯 학습 목표 (Goal)
Rust의 사용자 정의 타입 시스템인 **구조체(struct), 열거형(enum), impl 블록, 트레이트(trait), 그리고 제네릭(Generics)**을 이해하고, Tauri 코드에서 상시 등장하는 `#[derive(Serialize, Clone)]`의 의미를 완벽히 파악합니다.

---

## 💡 핵심 개념 (Core Concepts)

---

### 1. 구조체 (Struct) — 데이터를 묶는 설계도

TypeScript의 `interface`나 `class`처럼, 관련된 데이터를 하나의 이름으로 묶습니다.

> **🐍 Python과 비교:** Python의 `class`와 유사하지만, Rust의 `struct`은 **데이터만** 담습니다. 메서드는 별도의 `impl` 블록에 작성합니다.
> ```python
> # Python
> class User:
>     def __init__(self, name: str, age: int):
>         self.name = name  # 생성자에서 필드 선언
>         self.age = age
> ```
> ```rust
> // Rust: 데이터 정의와 메서드가 분리되어 있습니다
> struct User { name: String, age: u32 }  // 데이터만
> impl User { fn greet(&self) { ... } }    // 메서드는 따로
> ```

```rust
// ✅ 구조체 정의
struct User {
    name: String,
    email: String,
    age: u32,
    active: bool,
}

fn main() {
    // ✅ 인스턴스 생성
    let user1 = User {
        name: String::from("홍길동"),
        email: String::from("hong@email.com"),
        age: 30,
        active: true,
    };

    // 필드 접근
    println!("이름: {}", user1.name);

    // ✅ 가변 구조체: 내부 필드를 수정하려면 전체를 mut으로 선언
    let mut user2 = User {
        name: String::from("김철수"),
        email: String::from("kim@email.com"),
        age: 25,
        active: false,
    };
    user2.active = true; // 가변이므로 수정 가능
}
```

---

### 2. `impl` 블록 — 구조체에 기능(메서드) 달아주기

```rust
struct Rectangle {
    width: f64,
    height: f64,
}

// impl 블록으로 Rectangle에 메서드를 부착합니다.
impl Rectangle {
    // 🔹 연관 함수 (Associated Function): `self`가 없음. 
    // JS의 static 메서드와 비슷. `Rectangle::new(3.0, 4.0)` 형태로 호출
    fn new(width: f64, height: f64) -> Self {
        Self { width, height } // Self = Rectangle 자기 자신
    }

    // 🔹 메서드: 첫 번째 인자가 `&self` (인스턴스의 불변 참조)
    fn area(&self) -> f64 {
        self.width * self.height
    }

    // 🔹 가변 메서드: `&mut self`로 내부 값을 변경할 수 있음
    fn scale(&mut self, factor: f64) {
        self.width *= factor;
        self.height *= factor;
    }
}

fn main() {
    let mut rect = Rectangle::new(10.0, 5.0);
    println!("넓이: {}", rect.area()); //  50.0
    rect.scale(2.0);
    println!("확대 후 넓이: {}", rect.area()); // 200.0
}
```

---

### 3. 열거형 (Enum) — 가능한 경우의 수를 꼼꼼히 나열

Rust의 `enum`은 단순한 상수 목록이 아니라, **각 항목(variant)이 서로 다른 데이터를 품을 수 있습니다.** 
사실 여러분이 이미 써본 `Option<T>`과 `Result<T, E>`도 enum입니다!

> **🐍 Python과 비교:** Python의 `enum.Enum`은 단순 상수 목록입니다 (`class Color(Enum): RED=1`). Rust의 enum은 각 variant에 **서로 다른 타입의 데이터**를 담을 수 있어 Python의 `Union` 타입, 또는 다형성(`isinstance` 검사)을 대체하는 훨씬 강력한 도구입니다.

```rust
// ✅ 기본 Enum
enum Direction {
    Up,
    Down,
    Left,
    Right,
}

// ✅ 데이터를 품은 Enum (Tauri에서 다양한 이벤트 분류에 활용)
enum AppMessage {
    TextMsg(String),             // 문자열 데이터를 품은 항목
    NumberMsg(i32),              // 정수를 품은 항목
    DetailedMsg { from: String, body: String }, // 구조체처럼 이름 붙은 필드
    Quit,                        // 데이터 없는 단순 항목
}

fn handle_message(msg: AppMessage) {
    // match는 enum과 궁합이 완벽합니다!
    match msg {
        AppMessage::TextMsg(text) => println!("텍스트: {}", text),
        AppMessage::NumberMsg(n) => println!("숫자: {}", n),
        AppMessage::DetailedMsg { from, body } => {
            println!("{}로부터: {}", from, body);
        }
        AppMessage::Quit => println!("종료 메시지 수신"),
    }
}

fn main() {
    handle_message(AppMessage::TextMsg("안녕!".into()));
    handle_message(AppMessage::Quit);
}
```

**💡 `Option`과 `Result`의 정체:**
```rust
// 표준 라이브러리에 이미 이렇게 정의되어 있습니다!
// enum Option<T> { Some(T), None }
// enum Result<T, E> { Ok(T), Err(E) }

// 우리가 쓰는 Some(42), None, Ok("성공"), Err("실패")는
// 전부 이 enum의 variant(항목)인 것입니다!
```

---

### 4. 트레이트 (Trait) — "이 타입은 이런 능력이 있다"는 계약서

트레이트는 TypeScript의 `interface`와 비슷합니다. "특정 메서드를 구현해야 한다"는 약속입니다.

> **🐍 Python과 비교:** Python의 `abc.ABC` (추상 기반 클래스)와 가장 유사합니다.
> ```python
> from abc import ABC, abstractmethod
> class Summarizable(ABC):
>     @abstractmethod
>     def summary(self) -> str: ...
> 
> class Article(Summarizable):
>     def summary(self) -> str: return f"[{self.title}]"
> ```
> Rust에서는 `trait Summarizable { fn summary(&self) -> String; }` 로 정의하고, `impl Summarizable for Article { ... }` 로 구현합니다. Python의 보호적 상속(class 상속) 대신, Rust는 **어떤 타입에든 능력을 나중에 붙일 수 있는** 유연한 구조입니다.

```rust
// ✅ 트레이트 정의: "요약 가능하다"는 능력
trait Summarizable {
    fn summary(&self) -> String;
}

struct Article {
    title: String,
    content: String,
}

struct Tweet {
    username: String,
    body: String,
}

// Article에 대해 Summarizable 트레이트 구현
impl Summarizable for Article {
    fn summary(&self) -> String {
        format!("[기사] {}: {}...", self.title, &self.content[..20])
    }
}

// Tweet에 대해서도 같은 트레이트 구현 (다른 방식으로!)
impl Summarizable for Tweet {
    fn summary(&self) -> String {
        format!("@{}: {}", self.username, self.body)
    }
}
```

---

### 5. `#[derive(…)]` 매크로 — 트레이트 자동 구현 ⭐

매번 `impl Serialize for X { ... }` 같은 코드를 일일이 손으로 쓰기엔 너무 번거롭습니다.
Rust는 자주 쓰이는 트레이트를 **`#[derive]` 매크로로 자동 생성**해줍니다!

> **🐍 Python과 비교:** Python의 `@dataclass` 데코레이터와 역할이 비슷합니다. `@dataclass`가 `__init__`, `__repr__`, `__eq__` 등을 자동으로 만들어주는 것처럼, Rust의 `#[derive(Debug, Clone)]`도 해당 트레이트의 구현 코드를 자동 생성합니다.
> ```python
> from dataclasses import dataclass
> @dataclass                       # ≈ Rust의 #[derive(Debug, Clone)]
> class User:
>     name: str
>     level: int
> ```

```rust
use serde::{Serialize, Deserialize};

// 이 한 줄로 4가지 능력이 자동 추가됩니다!
#[derive(Debug, Clone, Serialize, Deserialize)]
struct UserProfile {
    name: String,
    level: u32,
}

// Debug   → {:?} 포맷으로 디버그 출력 가능
// Clone   → .clone()으로 깊은 복사 가능
// Serialize   → Rust 구조체 → JSON 변환 (프론트엔드로 보낼 때 필수!)
// Deserialize → JSON → Rust 구조체 변환 (프론트엔드에서 받을 때!)

fn main() {
    let user = UserProfile { name: "홍길동".into(), level: 5 };
    println!("{:?}", user);         // Debug 덕분에 출력 가능
    let user2 = user.clone();      // Clone 덕분에 복사 가능
}
```

> **📌 Tauri에서 가장 중요한 derive:**
> - `Serialize` : Rust 데이터를 JSON으로 변환 → JS 프론트엔드로 전송 (응답)
> - `Deserialize` : JS에서 보낸 JSON을 Rust 구조체로 변환 (요청 수신)
> - `Clone` : 이벤트 페이로드 등에서 데이터 복제 시 필요

---

### 6. 제네릭 (Generics, `<T>`) — 타입을 나중에 결정하겠다!

```rust
// T는 "아직 정해지지 않은 어떤 타입"을 뜻하는 플레이스홀더
fn first_element<T>(list: &[T]) -> &T {
    &list[0]
}

fn main() {
    let numbers = vec![10, 20, 30];
    let names = vec!["Alice", "Bob"];

    println!("{}", first_element(&numbers)); // T가 i32로 결정됨
    println!("{}", first_element(&names));   // T가 &str로 결정됨
}
```

**Tauri에서 이미 마주쳤던 제네릭들:**
```rust
Result<String, String>    // T=String, E=String
Option<i32>               // T=i32
Mutex<u32>                // T=u32
State<'_, AppState>       // T=AppState (라이프타임 + 제네릭 조합)
Vec<String>               // T=String
```

---

## 🚀 마무리 및 다음 단계

이제 Tauri 코드에 등장하는 `struct`, `impl`, `#[derive(Serialize)]`, `enum`, `match`, `<T>` 같은 기호들이 더 이상 외계어가 아닐 것입니다!

Rust의 필수 무기 하나만 더 남았습니다. 데이터를 동적으로 모으는 **컬렉션(Vec, HashMap)**, 변수를 포획하는 **클로저(Closure)와 `move`**, 그리고 파일을 나누는 **모듈(mod/use)** 시스템을 다음 장 [**05. 컬렉션, 클로저, 모듈**](./05-collections-closures-modules.md)에서 마저 마무리합니다!
