# [TIL] Java 메모리 구조(Java Memory Structure)와 JVM 메모리 영역 이해하기

> 📅 Today I Learned  
> Java 프로그램이 실행될 때 JVM이 메모리를 어떻게 관리하는지 학습했다.

---

## 📚 JVM(Java Virtual Machine)이란?

JVM(Java Virtual Machine)은 Java 프로그램을 실행하기 위한 가상 머신이다.

Java 코드는 컴파일 후 **ByteCode(.class)** 로 변환되며, JVM이 이를 운영체제에 맞게 실행한다.

JVM은 프로그램 실행을 위해 메모리를 여러 영역으로 나누어 관리한다.

---

# JVM 메모리 구조

```text
JVM Memory

┌───────────────────────────────┐
│          Method Area          │
├───────────────────────────────┤
│             Heap              │
├───────────────────────────────┤
│       Stack (Thread별)        │
├───────────────────────────────┤
│         PC Register           │
├───────────────────────────────┤
│     Native Method Stack       │
└───────────────────────────────┘
```

---

## 1. Method Area (메서드 영역)

프로그램에서 공통으로 사용하는 정보를 저장하는 공간이다.

### 저장되는 데이터

- 클래스 정보
- 메서드 정보
- static 변수
- final 상수
- Runtime Constant Pool

### 예제

```java
class Person {
    static int count = 0;
}
```

`count`는 Method Area에 저장된다.

### 특징

- 모든 스레드가 공유
- 프로그램 종료 시까지 유지되는 데이터가 많다.

---

## 2. Heap (힙 영역)

객체(Object)와 배열(Array)이 생성되는 공간이다.

Java에서 `new` 키워드를 사용하면 대부분 Heap에 저장된다.

### 예제

```java
Person p = new Person();
```

### 메모리 구조

```text
Stack
----------------
p --------┐
          │
Heap      │
----------------
Person 객체 ◀─┘
```

### 특징

- 가장 큰 메모리 영역
- 모든 스레드가 공유
- Garbage Collector(GC)가 관리한다.

---

## 3. Stack (스택 영역)

메서드 호출 시 생성되는 지역 변수와 매개변수를 저장하는 공간이다.

메서드가 종료되면 자동으로 제거된다.

### 예제

```java
public void hello() {
    int age = 20;
}
```

`age`는 Stack에 저장된다.

### Stack Frame

메서드가 호출될 때마다 새로운 Stack Frame이 생성된다.

```text
main()

↓

hello()

↓

print()
```

메서드가 종료되면 **후입선출(LIFO)** 방식으로 제거된다.

---

## Stack과 Heap의 관계

```java
Person p = new Person();
```

```text
Stack
-------------------
p (참조 주소)

↓

Heap
-------------------
Person 객체
```

Stack에는 객체 자체가 아닌 **참조 주소(Reference)** 가 저장된다.

실제 객체는 Heap에 존재한다.

---

## 4. PC Register

각 스레드마다 하나씩 존재하는 메모리 영역이다.

현재 실행 중인 JVM 명령어의 주소를 저장한다.

즉,

> "현재 프로그램이 어디까지 실행되었는지"를 기억하는 공간이다.

---

## 5. Native Method Stack

Java가 아닌 C/C++ 등 Native 메서드를 실행할 때 사용하는 메모리이다.

JNI(Java Native Interface)를 사용할 때 사용된다.

---

# 객체 생성 과정

```java
Person p = new Person();
```

실행 순서

1. Stack에 참조 변수 `p` 생성
2. Heap에 Person 객체 생성
3. 객체의 주소를 `p`가 참조

```text
Stack

p
│
▼

Heap

Person Object
```

---

# Garbage Collection(GC)

Heap에서 더 이상 참조되지 않는 객체를 자동으로 삭제하는 기능이다.

```java
Person p = new Person();

p = null;
```

이제 객체를 참조하는 변수가 없으므로 GC의 대상이 된다.

> 즉시 삭제되는 것이 아니라 Garbage Collector가 실행되는 시점에 메모리가 회수된다.

---

# Stack과 Heap 비교

| Stack | Heap |
|--------|------|
| 지역 변수 저장 | 객체 저장 |
| 메서드 종료 시 제거 | GC가 제거 |
| 속도가 빠름 | 상대적으로 느림 |
| Thread별 생성 | 모든 Thread 공유 |

---

# 실행 예제

```java
public class Main {

    public static void main(String[] args) {

        int a = 10;

        Person p = new Person();

        test();
    }

    static void test() {
        int b = 20;
    }
}
```

### 실행 과정

#### Stack

```text
main()

a = 10

p = 객체 주소

↓

test()

b = 20
```

`test()` 종료

```text
b 제거
```

`main()` 종료

```text
a 제거

p 제거
```

Heap에 있는 Person 객체는 더 이상 참조되지 않으므로 Garbage Collection 대상이 된다.

---

# 핵심 정리

| 메모리 영역 | 저장 내용 |
|------------|----------|
| Method Area | 클래스 정보, static 변수, 메서드 정보 |
| Heap | 객체, 배열 |
| Stack | 지역 변수, 매개변수, 참조 변수 |
| PC Register | 현재 실행 중인 명령어 위치 |
| Native Method Stack | Native 메서드 실행 |

---

# 핵심 포인트

- `new`로 생성한 객체는 Heap에 저장된다.
- 참조 변수는 Stack에 저장된다.
- `static` 변수는 Method Area에 저장된다.
- 메서드가 종료되면 Stack 메모리는 자동으로 제거된다.
- Heap 메모리는 Garbage Collector(GC)가 관리한다.
- 참조되지 않는 객체는 GC의 대상이 된다.

---

## ✨ 오늘의 한 줄 정리

> Java 메모리는 Method Area, Heap, Stack 등으로 구분되며, 객체와 변수의 저장 위치를 이해하면 객체 생성 과정과 Garbage Collection의 동작 원리를 명확하게 이해할 수 있다.
