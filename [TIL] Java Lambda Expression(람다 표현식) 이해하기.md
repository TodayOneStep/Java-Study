# [TIL] Java Lambda Expression(람다 표현식) 이해하기

> Today I Learned - Java Lambda

---

# Lambda(람다)란?

람다(Lambda Expression)는 **메서드를 하나의 식(Expression)으로 표현하는 문법**이다.

익명 클래스(Anonymous Class)를 더욱 간결하게 작성할 수 있도록 도와주며,
Java 8부터 도입되었다.

람다를 사용하면 코드가 짧아지고 가독성이 좋아지며, 함수형 프로그래밍 스타일을 사용할 수 있다.

---

# 왜 사용할까?

기존에는 인터페이스를 구현하기 위해 익명 클래스를 많이 사용했다.

### 기존 방식

```java
Runnable runnable = new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello");
    }
};
```

람다를 사용하면

```java
Runnable runnable = () -> System.out.println("Hello");
```

코드가 훨씬 간결해진다.

---

# 람다의 기본 문법

```java
(매개변수) -> { 실행문 }
```

예시

```java
(int a, int b) -> {
    return a + b;
}
```

더 줄이면

```java
(a, b) -> a + b;
```

---

# 문법 규칙

## 1. 매개변수 타입 생략 가능

컴파일러가 타입을 추론한다.

```java
(a, b) -> a + b
```

---

## 2. 매개변수가 하나라면 괄호 생략 가능

```java
name -> System.out.println(name);
```

하지만 명확성을 위해 괄호를 사용하는 경우도 많다.

```java
(name) -> System.out.println(name);
```

---

## 3. 실행문이 한 줄이면 중괄호 생략 가능

```java
() -> System.out.println("Hello");
```

---

## 4. return만 있는 경우 return과 중괄호 생략 가능

```java
(a, b) -> a + b
```

---

# Functional Interface(함수형 인터페이스)

람다는 **함수형 인터페이스(Function Interface)** 에서만 사용할 수 있다.

함수형 인터페이스란

> 추상 메서드가 하나만 존재하는 인터페이스

예시

```java
@FunctionalInterface
public interface Calculator {
    int sum(int a, int b);
}
```

람다 사용

```java
Calculator cal = (a, b) -> a + b;

System.out.println(cal.sum(3, 5));
```

출력

```
8
```

---

# @FunctionalInterface

```java
@FunctionalInterface
public interface MyFunction {

    void run();

}
```

`@FunctionalInterface`를 붙이면

- 추상 메서드가 하나인지 컴파일러가 검사
- 실수로 메서드를 추가하는 것을 방지

예를 들어

```java
@FunctionalInterface
interface Test {

    void run();

    void print();

}
```

컴파일 에러가 발생한다.

---

# Java에서 제공하는 대표적인 함수형 인터페이스

## Consumer

값을 받아 소비한다.

반환값이 없다.

```java
Consumer<String> consumer = s -> System.out.println(s);

consumer.accept("Hello");
```

---

## Supplier

값을 반환한다.

```java
Supplier<String> supplier = () -> "Java";

System.out.println(supplier.get());
```

---

## Function

입력을 받아 다른 타입으로 변환한다.

```java
Function<String, Integer> function = s -> s.length();

System.out.println(function.apply("Lambda"));
```

출력

```
6
```

---

## Predicate

참 또는 거짓을 반환한다.

```java
Predicate<Integer> predicate = n -> n > 10;

System.out.println(predicate.test(15));
```

출력

```
true
```

---

# 람다와 Stream

람다는 Stream API와 함께 사용할 때 가장 강력하다.

예시

```java
List<String> names = List.of("Kim", "Lee", "Park");

names.stream()
     .forEach(name -> System.out.println(name));
```

메서드 참조를 사용하면 더 간단하다.

```java
names.forEach(System.out::println);
```

---

# Method Reference(메서드 참조)

람다를 더 줄일 수 있는 문법이다.

람다

```java
name -> System.out.println(name)
```

메서드 참조

```java
System.out::println
```

또 다른 예시

```java
String::length
```

```java
Integer::parseInt
```

---

# 람다 사용 시 장점

- 코드가 짧아진다.
- 가독성이 좋아진다.
- 익명 클래스 사용이 줄어든다.
- Stream API와 함께 사용하기 좋다.
- 함수형 프로그래밍 스타일을 사용할 수 있다.

---

# 람다 사용 시 단점

- 처음에는 문법이 낯설다.
- 지나치게 복잡한 람다는 오히려 가독성이 떨어질 수 있다.
- 디버깅이 어려운 경우가 있다.

---

# 익명 클래스 vs 람다

### 익명 클래스

```java
Runnable runnable = new Runnable() {
    @Override
    public void run() {
        System.out.println("Run");
    }
};
```

### 람다

```java
Runnable runnable = () -> System.out.println("Run");
```

람다가 훨씬 간결하다.

---

# 정리

- Lambda는 Java 8에서 추가된 기능이다.
- 메서드를 하나의 식으로 표현할 수 있다.
- 함수형 인터페이스에서만 사용할 수 있다.
- 코드가 간결해지고 가독성이 높아진다.
- Stream API와 함께 자주 사용된다.
- 메서드 참조(`::`)를 사용하면 더 간결하게 작성할 수 있다.

---

# 핵심 키워드

- Lambda Expression
- Functional Interface
- @FunctionalInterface
- Consumer
- Supplier
- Function
- Predicate
- Stream API
- Method Reference
- Java 8

