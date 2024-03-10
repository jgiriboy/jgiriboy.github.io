---
title: Rust(러스트) Trait
author: Gieun Jeong
date: 2024-03-11 02:30:00 +0900
categories: [Rust 기초]
tags: [Rust, 러스트, 프로그래밍언어]
# pin: true
# math: true
# mermaid: true
# image:
#   path: /commons/devices-mockup.png
#   lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
#   alt: Responsive rendering of Chirpy theme on multiple devices.
---

러스트의 `trait`은 다른 언어의 `interface`와 유사하다. `trait`을 사용하면, **공유하는 동작**을 정의할 수 있다. 글의 내용은 [The Rust Programming Language 한국어 번역](https://rinthel.github.io/rust-lang-book-ko/ch10-02-traits.html) 의 `trait` 파트를 공부하며 정리하였다.

---
### 트레잇 정의하기
만약 어떤 `struct` 혹은 `enum` 처럼 어떤 타입의 동작을 정의했다고 해보자. 해당 **타입의 동작은 메소드를 구현하여 정의**될 수 있다. 대부분의 타입은 일반적으로 해당 타입의 인스턴스 정보를 출력하는 `Display` 같은 동작을 구현한다. 이렇게 서로 다른 타입이지만 같은 동작을 하는 경우, 동작을 공유한다고 할 수 있다.

공식 문서에서는 `NewsArticle` 구조체와 `Tweet` 구조체에서, 내용을 요약하여 출력하는 `summary`라는 동일한 메소드가 존재하는 상황을 가정하였다. 아래 해당 `trait`을 정리하였음.

```rust
// Naming Convetion: camel case
// pub을 넣었기 때문에 다른 크레이트에서 이 트레잇을 구현할 수 있음
pub trait Summarizable {
    // 일단 함수 파라미터와 리턴타입만 정의
    fn summary(&self) -> String;

    // 여러개의 메소드가 가능함
    // fn ...
    // fn ...
}
```

만약 어떤 타입이 `Summarizable` 트레잇을 가지고 있다면, 컴파일러는 해당 타입이 `summary`라는 이름을 정확히 가지도록 강제할 것이다.

---
### 특정 타입에 대한 Trait 구현

아래 코드는 모두 [공식 문서](https://rinthel.github.io/rust-lang-book-ko/ch10-02-traits.html)의 코드와 동일하다.

```rust
pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summarizable for NewsArticle {
    fn summary(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub retweet: bool,
}

impl Summarizable for Tweet {
    fn summary(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}
```

트레잇이 구현되면 사용하는 것은 일반 메소드 사용방법과 동일하다.
```rust
let tweet = Tweet {
    username: String::from("horse_ebooks"),
    content: String::from("of course, as you probably already know, people"),
    reply: false,
    retweet: false,
};

// 일반 메소드 사용하는 방법과 동일하게 사용
println!("1 new tweet: {}", tweet.summary());
```

---

### 기본 동작 구현
만약 `trait`이 기본적인 동작을 가지고 있고, 필요할 때 `override`하기를 원할 수도 있다. 그럴 때는 그냥 `trait`의 동작을 구현해주면 된다.

```rust
pub trait Summarizable {
    // 기본 동작을 구현해 주었음
    fn summary(&self) -> String {
        String::from("(Read more...)")
    }
}
```
<br/>
사용할 때 `summary`의 커스텀 동작은 필요없고 기본 동작만 필요하다면 아래처럼 빈 `impl` 블록을 명시해주면 된다.
`summary` 동작을 구현하지 않았지만, 기본 동작을 가지고 있기 때문에 문제없이 메소드를 호출할 수 있다.

```rust
impl Summarizable for NewsArticle {}
```

<br/>
기본 구현 내부에서는 동일한 트레잇 내부의 다른 메소드를 호출할 수 있다. 이 때 다른 메소드는 **기본 구현을 가지고 있지 않아도 된다.** 대신 기본 구현된 메소드를 호출하기 위해서는 다른 메소드를 구현하도록 강제한다.

```rust
pub trait Summarizable {
    // 이 메소드를 구현해야 summary를 호출할 수 있다
    fn author_summary(&self) -> String;

    // 기본 구현이 author_summary를 호출하고 있다.
    // author_summary를 구현해야 이 메소드를 호출할 수 있다.
    fn summary(&self) -> String {
        format!("(Read more from {}...)", self.author_summary())
    }
}
```

<br />
이 경우 `Summarizable`을 구현하고 싶다면 아래처럼 `author_summary`만 구현하면 된다.

```rust
impl Summarizable for Tweet {
    // author_summary만 구현하면 됨
    fn author_summary(&self) -> String {
        String::from("hello")
    }
}
```

---
### 트레잇 바운드
제네릭 타입을 사용하는 트레잇을 한번 구현해보자. 이 때 제네릭 타입이 모든 것에 대응하도록 하지 않고, 제네릭 타입이 트레잇을 구현하도록 제한하려고 한다.<br/>

아래 코드는 파라미터 `item` 에서 `summary`메소드를 호출하는 함수 `notify`를 정의한 것이다. 이 때 `item`의 타입은 제네릭 타입 `T`이다. 에러없이 `item`에서 `summary`를 호출하기 위해서는 적절하게 트레잇의 메소드가 구현되어야 할 것이다. 이 때 `<T: Summarizable>` 처럼, `T`는 반드시 `Summarizable` 트레잇을 구현해야만 한다고 명시해준다. 이렇게 명시하는 것을 트레잇 바운드라고 한다.
```rust
pub fn notify<T: Summarizable>(item: T) {
    println!("Breaking news! {}", item.summary());
}
```

위의 `notify` 함수의 인자로 아까 있었던 `Tweet` 구조체는 넘길 수 있다. 하지만 `String`, `i32`와 같은 타입은 `Summarizable` 트레잇을 구현하지 않았으므로 파라미터로 넘겨줄 수 없다.

<br/>
하나의 제네릭 타입에 대해 여러 트레잇을 구현하도록 하고 싶다면 `+`를 사용하면 된다. `summary` 메소드 뿐만 아니라 형식화된 출력을 원한다면 `T: Summarizable + Display`를 사용할 수 있다. 이는 T가 `Summarizable`과 `Display` 모두 구현해야 함을 의미한다.

<br/>
여러 개의 제네릭 타입을 가진 함수의 경우, 각 제네릭은 고유의 트레잇 바운드를 가진다.

```rust
fn some_function<T: Display + Clone, U: Clone + Debug>(t: T, u: U) -> i32 {}
```

하지만 위와 같은 코드는 가독성을 해치므로, `where`를 활용하여 트레잇 바운드를 뒤로 옮겨서 아래와 같이 재작성할 수 있다.

```rust
fn some_function<T, U>(t: T, u: U) -> i32
where T: Display + Clone,
      U: Clone + Debug
{}
```