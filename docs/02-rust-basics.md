# 🦀 02. Rust 기초 문법 (Variables, Types, Functions, Control Flow)

## 🎯 학습 목표 (Goal)
Rust 코드를 읽고 쓰기 위한 가장 기본적인 문법인 **변수 선언, 데이터 타입, 함수 정의, 제어 흐름(if/for/match)**을 익힙니다.

---

## 💡 핵심 개념 (Core Concepts)

### 1. 변수 선언 (`let`, `mut`, `const`)

Rust에서 변수는 **기본적으로 불변(immutable)**입니다! 한 번 값을 넣으면 바꿀 수 없습니다.

```rust
fn main() {
    // ✅ 기본 변수: 불변 (값을 바꿀 수 없음)
    let name = "타우리";
    // name = "Tauri"; // ❌ 컴파일 에러! 불변이라 재할당 불가

    // ✅ 가변 변수: `mut` 키워드를 붙이면 변경 가능
    let mut count = 0;
    count += 1; // ✅ 정상 동작. count는 이제 1
    println!("count = {}", count);

    // ✅ 상수: 반드시 타입 명시, 프로그램 전체에서 절대 불변
    const MAX_RETRY: u32 = 5;
    
    // ✅ 섀도잉(Shadowing): 같은 이름의 변수를 다시 `let`으로 선언 (타입도 바뀔 수 있음!)
    let x = 5;        // x는 i32 타입 정수
    let x = x + 1;    // 새로운 x가 기존 x를 가림 (shadow). 이제 x = 6
    let x = "문자열"; // 타입까지 완전히 바뀜! (i32 → &str)
    println!("x = {}", x);
}
```

> **📌 왜 기본이 불변(immutable)일까?** 
> 값이 예상치 않게 바뀌는 버그를 컴파일 시점에 원천 차단하기 위함입니다. 꼭 바꿔야 할 때만 `mut`를 의식적으로 붙이는 습관을 들이세요.

> **🐍 Python과 비교:**
> Python에서는 모든 변수가 기본적으로 재할당 가능합니다 (`x = 1` → `x = 2` 자유). 
> Rust는 정반대로, 변경하려면 `let mut`으로 명시적으로 선언해야 합니다.
> | Python | Rust |
> |---|---|
> | `x = 5` (항상 재할당 가능) | `let x = 5;` (불변, 재할당 불가) |
> | *(불변 장치 없음)* | `let mut x = 5;` (가변, 재할당 가능) |
> | `CONST = 5` (관례일 뿐, 강제 아님) | `const MAX: u32 = 5;` (컴파일러가 강제) |

---

### 2. 기본 데이터 타입 (Primitive Types)

Tauri 코드를 읽다 보면 `u32`, `u8`, `f64`, `bool` 같은 약어가 등장합니다. 이게 무엇을 뜻하는지 한눈에 정리합니다.

> **🐍 Python과 비교:** Python은 `x = 42`라고만 쓰면 내부적으로 알아서 `int`로 처리하고, 정수 크기에 제한이 없습니다(무한 정밀도). 반면 Rust는 변수가 메모리에서 몇 바이트를 차지할지 **컴파일 시점에 확정**합니다. `i32`는 4바이트, `u8`은 1바이트 등으로 정해져 있어 C/C++ 수준의 세밀한 메모리 제어가 가능합니다.

```rust
fn main() {
    // ── 정수형 (Integer) ──
    let age: u32 = 25;          // u32: 부호 없는 32비트 정수 (0 ~ 4,294,967,295)
    let temperature: i32 = -10; // i32: 부호 있는 32비트 정수 (음수 가능)
    let byte_val: u8 = 255;     // u8: 0~255 범위 (1바이트). Tauri 이벤트의 percent 등에 자주 사용
    let big: i64 = 9_000_000;   // i64: 큰 수. 밑줄(_)은 가독성 구분자 (9000000과 동일)

    // ── 실수형 (Floating Point) ──  
    let pi: f64 = 3.141592;     // f64: 64비트 부동소수점 (기본 추론 타입)
    let ratio: f32 = 0.75;      // f32: 32비트 부동소수점 (메모리 절약용)

    // ── 논리형 (Boolean) ──
    let is_admin: bool = true;

    // ── 문자형 (Character) ──
    let emoji: char = '🦀';     // 4바이트 유니코드 문자 (한 글자)

    // ── 문자열 ──
    let greeting: &str = "안녕"; // 문자열 슬라이스(참조). 03장 소유권에서 상세 설명
    let owned: String = String::from("안녕하세요"); // 소유권을 가진 힙 할당 문자열

    // ── 튜플 & 배열 ──
    let point: (f64, f64) = (10.5, 20.3);  // 여러 타입을 묶을 수 있는 고정 크기 튜플
    let first = point.0;                    // 인덱스로 접근
    let numbers: [i32; 3] = [1, 2, 3];     // 고정 길이 배열 (모두 같은 타입)

    // ✨ Rust는 타입 추론(Type Inference)이 강력합니다!
    let inferred = 42;      // 컴파일러가 `i32`로 자동 추론
    let inferred_f = 3.14;  // 컴파일러가 `f64`로 자동 추론

    println!("{} {} {} {}", age, pi, is_admin, emoji);
}
```

---

### 3. 함수 정의 (`fn`)

Rust의 함수 문법은 TypeScript와 비슷하지만, **반환 타입은 `->` 화살표** 뒤에 적습니다.

> **🐍 Python과 비교:**
> ```python
> # Python
> def add(a: int, b: int) -> int:  # 타입 힌트는 '권장'일 뿐, 강제 아님
>     return a + b
> ```
> ```rust
> // Rust
> fn add(a: i32, b: i32) -> i32 {  // 타입 명시가 '필수'이며, 틀리면 컴파일 에러
>     a + b  // return 없이 마지막 표현식이 반환값 (세미콜론 없음 주의!)
> }
> ```
> Python의 `def` → Rust의 `fn`, Python의 `return` → Rust는 마지막 줄에 세미콜론을 안 붙이면 자동 반환.

```rust
// ✅ 기본 함수: 인자와 반환 타입 명시 필수
fn add(a: i32, b: i32) -> i32 {
    a + b   // ⚡ 세미콜론(;)을 안 붙이면 "이 값을 반환해!"라는 뜻! (암시적 반환)
}

// ✅ 반환값이 없는 함수 (→ 유닛 타입 `()`)
fn say_hello(name: &str) {
    println!("안녕, {}!", name);
    // 반환 타입을 안 적으면 `()` (빈 튜플, void 같은 것)을 반환
}

// ✅ 조기 반환 (Early Return) 도 가능
fn check_positive(n: i32) -> &'static str {
    if n < 0 {
        return "음수입니다";  // `return` 키워드 + 세미콜론
    }
    "양수입니다" // 마지막 줄: 세미콜론 없이 암시적 반환
}

fn main() {
    let result = add(10, 20);
    println!("합계: {}", result); // 합계: 30

    say_hello("학습자");

    println!("{}", check_positive(-5)); // "음수입니다"
}
```

> **⚡ 핵심 포인트: 세미콜론(`;`) 유무가 의미를 바꿉니다!**
> - `a + b` (세미콜론 없음) → "이 표현식의 값을 반환해" (Expression)
> - `a + b;` (세미콜론 있음) → "이 줄을 실행하고 아무것도 반환하지 마" (Statement)

---

### 4. 매크로 (`!`): 함수처럼 생겼지만 느낌표가 붙은 것

> **🐍 Python과 비교:**
> | 기능 | Python | Rust |
> |---|---|---|
> | 콘솔 출력 | `print(f"안녕 {name}")` | `println!("안녕 {}", name);` |
> | 문자열 포맷 | `f"파일 {n}개"` (f-string) | `format!("파일 {}개", n)` |
> | 디버그 출력 | `print(repr(x))` | `dbg!(x);` |

```rust
fn main() {
    // println!: 콘솔 출력 매크로. {} 는 플레이스홀더
    println!("안녕 {}!", "세계");           // 안녕 세계!
    println!("{} + {} = {}", 1, 2, 1 + 2); // 1 + 2 = 3

    // format!: 문자열을 생성하는 매크로 (println과 같지만 출력하지 않고 String 반환)
    let msg = format!("파일 {}개 처리 완료", 10);

    // vec!: Vec(가변 길이 배열)을 간편하게 생성하는 매크로
    let numbers = vec![1, 2, 3, 4, 5];

    // dbg!: 디버깅용 매크로. 변수명과 값을 stderr에 출력 (개발 중 매우 유용!)
    let x = 42;
    dbg!(x); // [src/main.rs:13] x = 42
}
```

---

### 5. 제어 흐름 (Control Flow)

#### `if` / `else` — 표현식(Expression)으로도 사용 가능!
```rust
fn main() {
    let score = 85;

    // 기본 조건문
    if score >= 90 {
        println!("A등급");
    } else if score >= 80 {
        println!("B등급");
    } else {
        println!("C등급");
    }

    // ✨ if는 값을 반환하는 '표현식'! (삼항 연산자 대용)
    let grade = if score >= 90 { "A" } else { "B" };
    println!("등급: {}", grade);
}
```

#### `for` 반복 (가장 많이 쓰는 루프)
```rust
fn main() {
    // 범위(Range) 반복: 1, 2, 3, 4, 5 (..= 은 이하, .. 은 미만)
    for i in 1..=5 {
        println!("{}번째 반복", i);
    }

    // 배열/벡터 순회
    let fruits = vec!["사과", "바나나", "체리"];
    for fruit in &fruits {   // `&`를 안 붙이면 소유권이 이동되어 이후 fruits 사용 불가!
        println!("과일: {}", fruit);
    }
}
```

#### `loop` — 무한 루프 (+ `break`로 탈출)
```rust
fn main() {
    let mut counter = 0;
    let result = loop {
        counter += 1;
        if counter == 10 {
            break counter * 2; // ✨ loop도 break와 함께 값을 반환할 수 있음!
        }
    };
    println!("결과: {}", result); // 결과: 20
}
```

---

### 6. 패턴 매칭 (`match`) ⭐ — Tauri 코드에서 대량 사용!

`match`는 JavaScript의 `switch`를 10배 강력하게 만든 것입니다. Tauri 트레이 이벤트 핸들링(09장), Result 처리 등에서 핵심적으로 사용됩니다.

> **🐍 Python과 비교:** Python 3.10에서 추가된 `match/case`와 유사하지만, Rust의 `match`는 **컴파일러가 모든 경우의 수를 빠짐없이 처리했는지 강제로 검사**합니다(Exhaustiveness Check). 하나라도 빼먹으면 컴파일이 안 됩니다!
> ```python
> # Python 3.10+
> match status:
>     case "active": print("활성")
>     case "inactive": print("비활성")
>     case _: print("알 수 없음")
> ```

```rust
fn describe_number(n: i32) -> &'static str {
    match n {
        // 정확한 값 매칭
        0 => "영",
        1 => "일",
        // 범위 매칭 (2 이상 9 이하)
        2..=9 => "한 자리 수",
        // 가드(Guard) 조건 추가 가능
        x if x < 0 => "음수",
        // `_` 는 나머지 모든 경우 (default)
        _ => "큰 수",
    }
}

fn main() {
    println!("{}", describe_number(5));   // "한 자리 수"
    println!("{}", describe_number(-3));  // "음수"
    println!("{}", describe_number(100)); // "큰 수"

    // ✨ Option 타입과의 조합 (매우 자주 사용!)
    let maybe_value: Option<i32> = Some(42);
    match maybe_value {
        Some(val) => println!("값이 있습니다: {}", val),
        None      => println!("비어 있습니다"),
    }

    // ✨ if let: match를 하나의 패턴에 대해서만 간소화
    if let Some(val) = maybe_value {
        println!("간편 추출: {}", val);
    }
}
```

---

## 🚀 마무리 및 다음 단계

이번 장에서 Rust의 가장 기본적인 건물의 "벽돌"들을 배웠습니다. 변수, 타입, 함수, 제어흐름 — 이것만으로도 간단한 Rust 코드를 읽을 수 있을 것입니다.

하지만 Rust에서 가장 독특하고 혁명적인 기능이 아직 남아 있습니다. 
**"왜 함수 인자에 `&`가 붙어있고, `String`과 `&str`은 뭐가 다른 거야?"**
다음 장 [**03. 소유권과 참조 (Ownership & Borrowing)**](./03-ownership-borrowing.md)에서 Rust만의 메모리 관리 철학을 파헤쳐 봅시다.
