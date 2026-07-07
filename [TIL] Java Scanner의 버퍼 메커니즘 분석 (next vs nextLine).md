# [TIL] Java Scanner의 버퍼 메커니즘 분석 (next vs nextLine)

<br/>

## 📌 문제 상황과 의문점
클래스를 활용한 간단한 주문 시스템을 구현하던 중, 문자열 입력을 위해 `sc.next()`를 선택했다가 과거에 겪었던 `Scanner` 관련 오류들이 떠올랐다. 강의 자료와 비교해 보니 강사님은 `nextLine()`을 적극 활용하고 있었고, 메서드 호출 사이에 버퍼를 비우는 코드가 존재했다. 이 둘의 명확한 차이와 주의점을 정리한다.

<br/><br/>

## 👉 문제 & 코드 
[실행 결과 예시]

```text
입력할 주문의 개수를 입력하세요: 3
1번째 주문 정보를 입력하세요.
상품명: 두부
가격: 2000
수량: 2
2번째 주문 정보를 입력하세요.
상품명: 김치
가격: 5000
수량: 1
3번째 주문 정보를 입력하세요.
상품명: 콜라
가격: 1500
수량: 2
상품명: 두부, 가격: 2000, 수량: 2
상품명: 김치, 가격: 5000, 수량: 1
상품명: 콜라, 가격: 1500, 수량: 2
총 결제 금액: 12000
```
<br/>
[내 풀이]

```java
public class ProductOrderMain3 {
    static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        System.out.print("주문의 개수를 입력하세요: ");
        int cnt = sc.nextInt();
        ProductOrder[] orders = new ProductOrder[cnt];

        for(int i = 0; i < cnt; i++) {
            System.out.println(i + 1 +"번째 주문 정보를 입력하세요.");
            System.out.print("상품명: ");
            String productName = sc.next();
            System.out.print("가격: ");
            int price = sc.nextInt();
            System.out.print("수량: ");
            int quantity = sc.nextInt();
            System.out.print("test: ");
            String test = sc.next();

            orders[i] = createOrder(productName, price, quantity);
        }

        printOrders(orders);
        int totalAmount = getTotalAmount(orders);
        System.out.println("총 결제 금액: " + totalAmount);
    }
}
```
<br/>
[강의 자료 풀이]

```java
public class ProductOrderMain3 {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        System.out.print("입력할 주문의 개수를 입력하세요: ");
        int n = scanner.nextInt();
        scanner.nextLine();

        ProductOrder[] orders = new ProductOrder[n];

        for (int i = 0; i < orders.length; i++) {
            System.out.println((i + 1) + "번째 주문 정보를 입력하세요.");
            
            System.out.print("상품명: ");
            String productName = scanner.nextLine();
            
            System.out.print("가격: ");
            int price = scanner.nextInt();
            
            System.out.print("수량: ");
            int quantity = scanner.nextInt();
            scanner.nextLine();

            orders[i] = createOrder(productName, price, quantity);
        }

        printOrders(orders);        
        int totalAmount = getTotalAmount(orders);
        System.out.println("총 결제 금액: " + totalAmount);
    }
}
```

<br/><br/>

## 🔍 핵심 차이점 분석

### 1. `next()` vs `nextLine()`
* **`next()` (토큰 중심):** 공백(스페이스), 탭(`\t`), 엔터(`\n`)를 기준으로 데이터를 잘라온다. 
  * *문제 발생 케이스:* 상품명에 `"쭈꾸미 볶음"`을 입력하면 `"쭈꾸미"`만 가져가고 `"볶음"`이 버퍼에 남는다. 다음 입력인 `nextInt()`가 이 문자열을 숫자로 읽으려다 `InputMismatchException`이 발생한다.
* **`nextLine()` (라인 중심):** 엔터(`\n`)가 입력될 때까지 한 줄 전체를 읽는다. 공백이 포함된 `"맛있는 쭈꾸미 볶음"`도 한 번에 안전하게 담을 수 있다.

<br/>

### 2. 앞 공백 무시 vs 뒤 공백 종료
* `nextInt()`나 `next()` 같은 토큰 중심 메서드들은 데이터가 입력되기 전 앞부분에 나오는 공백이나 엔터는 **값이 나올 때까지 무시하고 건너뛴다.** (엔터를 여러 번 쳐도 오작동하지 않는 이유)
* 하지만, 진짜 값을 읽은 후에 만나는 공백/엔터는 **종료 신호**로 인식해 값을 잘라내고, 정작 그 공백/엔터 자체는 **입력 버퍼에 찌꺼기로 남겨둔다.**

<br/><br/>

## ⚠️ `nextLine()` 사용 시 결정적 주의점 (버퍼 오염)

`nextInt()` 직후에 `nextLine()`을 호출하면 사용자의 입력을 받지 않고 그냥 넘어가는(Skip) 현상이 발생한다.

* **원인:** `nextInt()`가 숫자만 쏙 빼가고 남겨둔 엔터(`\n`) 찌꺼기가 버퍼 맨 앞에 남아있기 때문이다. 이어서 호출된 `nextLine()`은 이 엔터를 만나자마자 *"입력이 끝났구나!"* 판단하여 빈 문자열(`""`)을 반환하고 종료된다.
* **해결책:** `nextInt()`와 `nextLine()` 사이에 아무 기능도 없는 `scanner.nextLine();`을 한 줄 실행시켜 **버퍼의 엔터 찌꺼기를 청소(Consume)**해 주어야 한다.

<br/><br/>

## 💡 Deep Dive: 더 알아두면 좋은 팁

### 1. 엔터까지 지워주는 유일한 메서드
`Scanner` 클래스에서 제공하는 수많은 입력 메서드(`nextInt`, `nextDouble`, `nextBoolean`, `next` 등) 중, **엔터(`\n`)까지 통째로 삼켜서 버퍼를 깨끗하게 비워주는 메서드는 `nextLine()`이 유일**하다.

<br/>

### 2. 버퍼 비우기 처리가 필요 없는 예외 상황
`nextLine()`을 쓰고, 그 다음에 **또** `nextLine()`을 쓸 때는 중간에 버퍼를 비울 필요가 없다. `nextLine()`은 스스로 엔터까지 먹어서 치워버리기 때문에 버퍼를 깨끗한 상태로 유지하기 때문이다.

* ❌ **비워야 함:** `nextInt()` ➡️ `scanner.nextLine();` (청소) ➡️ `nextLine()`
* ⭕ **안 비워도 됨:** `nextLine()` ➡️ `nextLine()`

<br/><br/>

> 📝 **최종 요약 한 줄 평**
> "엔터를 남기는 녀석들(`next`, `nextInt` 등) 뒤에 엔터를 먹는 청소부(`nextLine`)가 올 때는 반드시 중간에 방 청소(`scanner.nextLine()`)를 해줘야 한다!"






