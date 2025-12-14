# Chapter 02. 객체지향 프로그래밍

> *"진정한 객체지향 패러다임으로의 전환은 클래스가 아닌 객체에 초점을 맞춰야만 얻을 수 있다"*

## 📌 핵심 개념

- **협력(Collaboration)**: 객체들이 애플리케이션의 기능을 구현하기 위해 수행하는 상호작용
- **메시지(Message)**: 객체가 다른 객체와 협력하기 위해 사용하는 의사소통 메커니즘
- **다형성(Polymorphism)**: 동일한 메시지에 대해 객체마다 다른 방식으로 응답하는 능력
- **추상화(Abstraction)**: 세부사항을 감추고 본질적인 개념만 남기는 것
- **상속(Inheritance)**: 코드를 재사용하고 다형성을 구현하는 메커니즘
- **합성(Composition)**: 객체를 포함해서 코드를 재사용하는 방법

---

## 🎯 학습 목표

1. 도메인 개념에서 객체와 클래스로 전환하는 과정 이해하기
2. 협력하는 객체들의 공동체로 프로그램을 바라보기
3. 상속과 다형성을 활용한 유연한 설계 구현하기
4. 추상화를 통해 변경에 유연한 코드 만들기
5. 상속과 합성의 차이점과 트레이드오프 이해하기

---

## 🎬 예제: 영화 예매 시스템

### 도메인 설명

온라인 영화 예매 시스템을 구축합니다. Chapter 01의 극장 예제보다 복잡한 비즈니스 규칙을 가지고 있습니다.

**핵심 용어**:

| 용어 | 설명 | 예시 |
|------|------|------|
| **영화 (Movie)** | 영화의 기본 정보 | 제목, 상영시간, 기본 요금 |
| **상영 (Screening)** | 실제 관람 이벤트 | "아바타" 12월 15일 10:00 상영 |
| **할인 조건 (Discount Condition)** | 할인 여부를 결정하는 규칙 | 순번 조건, 기간 조건 |
| **할인 정책 (Discount Policy)** | 할인 요금 계산 방법 | 금액 할인, 비율 할인 |
| **예매 (Reservation)** | 고객의 예매 정보 | 고객, 상영, 인원, 요금 |

**중요한 구분**:
```
영화 (Movie)
  ↓ has
여러 개의 상영 (Screening)

예시:
"아바타"라는 영화는 하나
  ├─ 12월 15일 10:00 상영 (1회차)
  ├─ 12월 15일 14:00 상영 (2회차)
  └─ 12월 15일 18:00 상영 (3회차)

사용자가 예매하는 대상 = 상영 (특정 일시의 이벤트)
```

**할인 규칙**:

1. **할인 조건** (하나라도 만족하면 할인 가능):
   - 순번 조건: "1회차 상영", "10회차 상영" 등
   - 기간 조건: "월요일 10:00~12:00", "목요일 14:00~21:00" 등

2. **할인 정책** (영화당 하나만 적용):
   - 금액 할인: 800원 할인
   - 비율 할인: 10% 할인
   - 할인 없음: 0원 할인

### 도메인 구조

```
┌─────────────┐         ┌──────────────┐
│   Movie     │◆──────→ │ Discount     │
│   (영화)     │         │ Policy       │
│             │         │ (할인 정책)    │
└─────────────┘         └──────────────┘
       │                        │
       │ has                    │ has
       ↓                        ↓
┌─────────────┐         ┌──────────────┐
│  Screening  │         │ Discount     │
│   (상영)     │         │ Condition    │
│             │         │ (할인 조건)    │
└─────────────┘         └──────────────┘
       │
       │ creates
       ↓
┌─────────────┐
│ Reservation │
│   (예매)     │
└─────────────┘
```

---

## 📦 기초 클래스들

### 👤 Customer - 고객

> 📂 **코드**: [`Customer.java`](https://github.com/eternity-oop/object/blob/master/chapter02/src/main/java/org/eternity/movie/step01/Customer.java)

```java
package org.eternity.movie.step01;

public class Customer {
    private String name;  // 고객 이름
    private String id;    // 고객 ID
    
    public Customer(String name, String id) {
        this.id = id;
        this.name = name;
    }
}
```

💡 **설명**:
- 예매를 진행하는 고객의 정보를 담는 클래스
- 현재는 단순히 이름과 ID만 보관
- 실제로는 getter 메서드나 추가 비즈니스 로직이 있을 수 있음
- **Chapter 02의 핵심은 고객이 아닌 영화 요금 계산이므로 단순하게 구현**

---

## 💰 Money - 금액을 표현하는 값 객체

> 📂 **코드**: [`Money.java`](https://github.com/eternity-oop/object/blob/master/chapter02/src/main/java/org/eternity/money/Money.java)

### 왜 Long이 아닌 Money 클래스를 만드는가?

```java
// ❌ 나쁜 예: 원시 타입 사용
long price = 10000;
long discount = 800;
long finalPrice = price - discount;

// 문제점:
price = discount * 2;  // 가격에 할인율을 곱함 (실수)
long duration = 120;
price = price + duration;  // 가격에 상영시간을 더함 (타입 안전성 없음)
```

```java
// ✅ 좋은 예: 값 객체 사용
Money price = Money.wons(10000);
Money discount = Money.wons(800);
Money finalPrice = price.minus(discount);

// 장점:
// 1. 타입 안전성: Money끼리만 연산 가능
// 2. 의미 명확: plus, minus, times 등 도메인 용어 사용
// 3. 불변성: 한 번 생성되면 변경 불가
```

### 전체 코드 분석

```java
package org.eternity.money;

import java.math.BigDecimal;
import java.util.Objects;

public class Money {
    // ✅ 상수로 0원을 제공 (자주 사용되므로)
    public static final Money ZERO = Money.wons(0);

    // ✅ final: 불변성 보장
    // ✅ BigDecimal: 정확한 금액 계산 (double은 부정확)
    private final BigDecimal amount;

    // ✅ 팩토리 메서드 패턴: 생성 방법을 명확하게 표현
    public static Money wons(long amount) {
        return new Money(BigDecimal.valueOf(amount));
    }

    public static Money wons(double amount) {
        return new Money(BigDecimal.valueOf(amount));
    }

    // ✅ package-private 생성자: 팩토리 메서드를 통해서만 생성
    Money(BigDecimal amount) {
        this.amount = amount;
    }

    // ✅ 불변 객체 패턴: 연산 결과를 새 객체로 반환
    public Money plus(Money amount) {
        return new Money(this.amount.add(amount.amount));
    }

    public Money minus(Money amount) {
        return new Money(this.amount.subtract(amount.amount));
    }

    public Money times(double percent) {
        return new Money(this.amount.multiply(BigDecimal.valueOf(percent)));
    }

    // ✅ 비교 연산: 금액 크기 비교
    public boolean isLessThan(Money other) {
        return amount.compareTo(other.amount) < 0;
    }

    public boolean isGreaterThanOrEqual(Money other) {
        return amount.compareTo(other.amount) >= 0;
    }

    // ✅ 값 객체의 동등성: 내용이 같으면 같은 객체
    @Override
    public boolean equals(Object object) {
        if (this == object) {
            return true;
        }

        if (!(object instanceof Money)) {
            return false;
        }

        Money other = (Money)object;
        return Objects.equals(amount.doubleValue(), other.amount.doubleValue());
    }

    @Override
    public int hashCode() {
        return Objects.hashCode(amount);
    }

    @Override
    public String toString() {
        return amount.toString() + "원";
    }
}
```

### Money 클래스의 핵심 설계 원칙

#### 1️⃣ 불변성 (Immutability)

```java
Money price = Money.wons(10000);
Money discount = price.minus(Money.wons(1000));

// price는 변하지 않음!
System.out.println(price);     // 10000원
System.out.println(discount);  // 9000원
```

**왜 불변성이 중요한가?**
```java
// 가변 객체의 문제
MutableMoney price = new MutableMoney(10000);
MutableMoney fee1 = price;
MutableMoney fee2 = price;

fee1.minus(1000);  // fee1을 변경했는데...

// price, fee1, fee2 모두 9000원! (예상치 못한 변경)
System.out.println(price);  // 9000원
System.out.println(fee1);   // 9000원
System.out.println(fee2);   // 9000원
```

#### 2️⃣ 값 객체 (Value Object)

```java
Money money1 = Money.wons(1000);
Money money2 = Money.wons(1000);

// 값이 같으면 같은 객체로 취급
money1.equals(money2);  // true

// 참조가 달라도 상관없음
money1 == money2;  // false (다른 인스턴스)
```

#### 3️⃣ 타입 안전성 (Type Safety)

```java
// ❌ 컴파일 에러: Money에 int를 더할 수 없음
Money price = Money.wons(10000);
price = price + 1000;  // 컴파일 에러!

// ✅ Money끼리만 연산 가능
price = price.plus(Money.wons(1000));
```

---

## 🎬 실제 사용 시나리오

코드가 어떻게 실행되는지 먼저 살펴봅시다.

### 영화 생성 및 예매 전체 과정

```java
// 1️⃣ 할인 조건 생성
DiscountCondition sequenceCondition1 = new SequenceCondition(1);     // 1회차
DiscountCondition sequenceCondition2 = new SequenceCondition(10);    // 10회차
DiscountCondition periodCondition = new PeriodCondition(
    DayOfWeek.MONDAY,
    LocalTime.of(10, 0),
    LocalTime.of(11, 59)
);

// 2️⃣ 할인 정책 생성 (금액 할인)
DiscountPolicy discountPolicy = new AmountDiscountPolicy(
    Money.wons(800),              // 800원 할인
    sequenceCondition1,
    sequenceCondition2,
    periodCondition
);

// 3️⃣ 영화 생성
Movie avatar = new Movie(
    "아바타",
    Duration.ofMinutes(120),
    Money.wons(10000),            // 기본 요금 10,000원
    discountPolicy
);

// 4️⃣ 상영 생성 (1회차, 월요일 10:30)
Screening screening = new Screening(
    avatar,
    1,                            // 1회차
    LocalDateTime.of(2024, 12, 16, 10, 30)  // 월요일 10:30
);

// 5️⃣ 예매 진행
Customer customer = new Customer("홍길동", "user123");
Reservation reservation = screening.reserve(customer, 2);  // 2명 예매

// 📊 최종 결과
// 기본 요금: 10,000원
// 할인 금액: 800원 (1회차 조건 만족 또는 월요일 10:00~12:00 조건 만족)
// 1인당 요금: 9,200원
// 2명 총 요금: 18,400원
```

### 실행 흐름 추적

```
screening.reserve(customer, 2) 호출
    │
    ├─> calculateFee(2) 호출
    │     │
    │     ├─> movie.calculateMovieFee(this) 호출
    │     │     │
    │     │     ├─> discountPolicy.calculateDiscountAmount(screening) 호출
    │     │     │     │
    │     │     │     ├─> sequenceCondition1.isSatisfiedBy(screening)
    │     │     │     │     └─> screening.isSequence(1) = true ✅
    │     │     │     │
    │     │     │     ├─> getDiscountAmount(screening) 호출
    │     │     │     │     └─> return Money.wons(800)
    │     │     │     │
    │     │     │     └─> return Money.wons(800)
    │     │     │
    │     │     ├─> fee.minus(Money.wons(800))
    │     │     └─> return Money.wons(9200)  // 10,000 - 800
    │     │
    │     ├─> Money.wons(9200).times(2)
    │     └─> return Money.wons(18400)
    │
    └─> new Reservation(customer, this, Money.wons(18400), 2)
```

이제 각 클래스가 어떻게 구현되어 있는지 상세히 살펴보겠습니다.

---

## 🎬 협력의 시작: Screening - 상영

> 📂 **코드**: [`Screening.java`](https://github.com/eternity-oop/object/blob/master/chapter02/src/main/java/org/eternity/movie/step01/Screening.java)

### 역할과 책임

`Screening`은 다음을 알고 있습니다:
- 어떤 영화를 상영하는지 (`movie`)
- 몇 회차 상영인지 (`sequence`)
- 언제 상영하는지 (`whenScreened`)

`Screening`의 책임:
- 예매 생성하기 (`reserve()`)
- 상영 정보 제공하기 (시작 시간, 순번, 요금)

### 전체 코드 분석

```java
package org.eternity.movie.step01;

import org.eternity.money.Money;
import java.time.LocalDateTime;

public class Screening {
    private Movie movie;               // 상영할 영화
    private int sequence;              // 상영 순번
    private LocalDateTime whenScreened; // 상영 시작 시간

    public Screening(Movie movie, int sequence, LocalDateTime whenScreened) {
        this.movie = movie;
        this.sequence = sequence;
        this.whenScreened = whenScreened;
    }

    // ✅ 캡슐화: 시작 시간 조회
    public LocalDateTime getStartTime() {
        return whenScreened;
    }

    // ✅ 캡슐화: 순번 확인
    //    외부에서 sequence를 직접 접근하지 않고
    //    "이 상영이 N번째인가?"라는 질문에 답변
    public boolean isSequence(int sequence) {
        return this.sequence == sequence;
    }

    // ✅ 데메테르 법칙 준수
    //    외부에 movie를 노출하지 않고
    //    movie에게 물어본 결과를 전달
    public Money getMovieFee() {
        return movie.getFee();
    }

    // ✅ 핵심 책임: 예매 생성
    //    Screening이 예매의 생성을 책임짐
    public Reservation reserve(Customer customer, int audienceCount) {
        return new Reservation(
            customer,
            this,
            calculateFee(audienceCount),  // 요금 계산
            audienceCount
        );
    }

    // ✅ private: 내부 구현 감춤
    //    요금 계산 로직은 외부에 노출하지 않음
    private Money calculateFee(int audienceCount) {
        // movie에게 1인당 요금을 물어보고
        // 인원수를 곱해서 총 요금 계산
        return movie.calculateMovieFee(this).times(audienceCount);
    }
}
```

### 협력 과정 상세 분석

```
사용자가 예매를 요청하면:

screening.reserve(customer, 2)
     ↓
1. calculateFee(2) 호출 (private)
     ↓
2. movie.calculateMovieFee(this) 호출
     │
     │  [Movie가 할인 정책에게 위임]
     │  discountPolicy.calculateDiscountAmount(screening)
     │        ↓
     │  [할인 정책이 할인 조건 확인]
     │  for each condition:
     │    if condition.isSatisfiedBy(screening):
     │      return getDiscountAmount(screening)
     │        ↓
     │  [할인 금액 반환]
     │  ← Money (할인 금액)
     │
     ← Money (할인 적용된 1인 요금)
     ↓
3. 1인 요금 × 2명 = 총 요금
     ↓
4. new Reservation(customer, this, totalFee, 2)
     ↓
5. 예매 완료!
```

### 왜 이렇게 설계했는가?

#### 🔒 캡슐화: 묻지 말고 시켜라

```java
// ❌ 나쁜 예: 외부에서 Screening 내부를 조작
Screening screening = ...;
Movie movie = screening.getMovie();  // movie를 꺼내고
Money fee = movie.getFee();           // fee를 꺼내고
Money discount = ...;                 // 할인을 계산하고
Money totalFee = fee.minus(discount).times(audienceCount);

// ✅ 좋은 예: Screening에게 요청
Screening screening = ...;
Reservation reservation = screening.reserve(customer, audienceCount);
```

#### 🎯 단일 책임 원칙 (SRP)

`Screening`의 단 하나의 책임: **예매를 생성한다**

```java
// Screening이 하는 일:
1. 예매 생성하기
   └─ 내부적으로 요금 계산 필요
       └─ Movie에게 1인당 요금 물어보기
           └─ 인원수 곱하기

// Screening이 하지 않는 일:
✗ 할인 정책 결정하기 → Movie의 책임
✗ 할인 조건 확인하기 → DiscountCondition의 책임
✗ 할인 금액 계산하기 → DiscountPolicy의 책임
```

---

## 🎥 중심 객체: Movie - 영화

> 📂 **코드**: [`Movie.java`](https://github.com/eternity-oop/object/blob/master/chapter02/src/main/java/org/eternity/movie/step01/Movie.java)

### 핵심 설계: 추상화에 의존하기

```java
package org.eternity.movie.step01;

import org.eternity.money.Money;
import java.time.Duration;

public class Movie {
    private String title;           // 제목
    private Duration runningTime;   // 상영 시간
    private Money fee;              // 기본 요금
    
    // ✅ 핵심: 추상화(DiscountPolicy)에 의존
    //    구체 클래스가 아닌 인터페이스/추상 클래스에 의존
    private DiscountPolicy discountPolicy;

    public Movie(String title, Duration runningTime, Money fee, 
                 DiscountPolicy discountPolicy) {
        this.title = title;
        this.runningTime = runningTime;
        this.fee = fee;
        this.discountPolicy = discountPolicy;
    }

    public Money getFee() {
        return fee;
    }

    // ✅ 핵심 메서드: 영화 요금 계산
    public Money calculateMovieFee(Screening screening) {
        // 기본 요금에서 할인 금액을 뺀다
        // "어떻게" 할인하는지는 discountPolicy가 알고 있음
        return fee.minus(discountPolicy.calculateDiscountAmount(screening));
    }
}
```

### 🤔 놀라운 사실: Movie는 할인 정책을 모른다!

```java
public Money calculateMovieFee(Screening screening) {
    return fee.minus(discountPolicy.calculateDiscountAmount(screening));
}
```

**이 코드의 놀라운 점**:

1. **Movie는 할인 정책의 종류를 모른다**
   ```java
   // Movie 클래스 어디에도 이런 코드가 없음:
   if (discountPolicy instanceof AmountDiscountPolicy) {
       // 금액 할인
   } else if (discountPolicy instanceof PercentDiscountPolicy) {
       // 비율 할인
   }
   ```

2. **Movie는 할인 금액 계산 방법을 모른다**
   ```java
   // Movie는 그저 요청만 할 뿐
   discountPolicy.calculateDiscountAmount(screening);
   
   // 실제 계산은 DiscountPolicy가 담당
   // - AmountDiscountPolicy: 800원 차감
   // - PercentDiscountPolicy: 10% 차감
   ```

3. **Movie는 할인 조건을 모른다**
   ```java
   // 순번 조건인지, 기간 조건인지 전혀 모름
   // DiscountPolicy가 알아서 처리
   ```

### 왜 이게 중요한가?

#### 변경에 유연한 코드

```java
// 새로운 할인 정책 추가: 고정 금액 1000원 할인
public class FixedDiscountPolicy extends DiscountPolicy {
    @Override
    protected Money getDiscountAmount(Screening screening) {
        return Money.wons(1000);
    }
}

// Movie 코드 변경 없이 바로 사용 가능!
Movie avatar = new Movie(
    "아바타",
    Duration.ofMinutes(120),
    Money.wons(10000),
    new FixedDiscountPolicy(...)  // 새 정책 적용
);
```

**Movie 클래스는 단 한 줄도 수정하지 않았습니다!**

이것이 바로 **"구현이 아닌 인터페이스에 의존하라"**의 의미입니다.

---

## 💳 예매 결과: Reservation - 예매

> 📂 **코드**: [`Reservation.java`](https://github.com/eternity-oop/object/blob/master/chapter02/src/main/java/org/eternity/movie/step01/Reservation.java)

```java
package org.eternity.movie.step01;

import org.eternity.money.Money;

public class Reservation {
    private Customer customer;      // 예매한 고객
    private Screening screening;    // 예매한 상영
    private Money fee;              // 예매 요금
    private int audienceCount;      // 인원수

    public Reservation(Customer customer, Screening screening, 
                      Money fee, int audienceCount) {
        this.customer = customer;
        this.screening = screening;
        this.fee = fee;
        this.audienceCount = audienceCount;
    }
}
```

💡 **설명**:
- 예매 정보를 담는 단순한 데이터 클래스
- 예매 성공의 결과물
- 실제로는 getter 메서드나 비즈니스 로직이 추가될 수 있음

---

## 🎫 Step 01: 추상 클래스 기반 할인 정책

> 📂 **전체 코드**: [step01 디렉토리](https://github.com/eternity-oop/object/tree/master/chapter02/src/main/java/org/eternity/movie/step01)

### 설계 구조

```
                    ┌──────────────────┐
                    │ DiscountPolicy   │
                    │ (abstract class) │
                    └──────────────────┘
                            △
                            │ extends
              ┌─────────────┼──────────────┐
              │             │              │
┌─────────────────┐ ┌───────────────┐ ┌─────────────────┐
│ Amount          │ │ Percent       │ │ None            │
│ DiscountPolicy  │ │ DiscountPolicy│ │ DiscountPolicy  │
└─────────────────┘ └───────────────┘ └─────────────────┘
```

### 📜 DiscountPolicy - 추상 클래스

> 📂 **코드**: [`DiscountPolicy.java`](https://github.com/eternity-oop/object/blob/master/chapter02/src/main/java/org/eternity/movie/step01/DiscountPolicy.java)

```java
package org.eternity.movie.step01;

import org.eternity.money.Money;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

public abstract class DiscountPolicy {
    // ✅ 할인 조건 목록
    //    하나의 할인 정책은 여러 개의 할인 조건을 가질 수 있음
    private List<DiscountCondition> conditions = new ArrayList<>();

    // ✅ 가변 인자: 여러 개의 할인 조건을 받을 수 있음
    public DiscountPolicy(DiscountCondition... conditions) {
        this.conditions = Arrays.asList(conditions);
    }

    // ✅ TEMPLATE METHOD 패턴
    //    전체적인 흐름은 여기서 정의
    //    세부 구현은 자식 클래스에 위임
    public Money calculateDiscountAmount(Screening screening) {
        // 할인 조건을 하나씩 확인
        for(DiscountCondition each : conditions) {
            // 조건을 만족하는 것이 하나라도 있으면
            if (each.isSatisfiedBy(screening)) {
                // 할인 금액을 계산해서 반환
                // (계산 방법은 자식 클래스가 결정)
                return getDiscountAmount(screening);
            }
        }

        // 만족하는 조건이 없으면 할인 없음
        return Money.ZERO;
    }

    // ✅ 추상 메서드: 자식 클래스가 반드시 구현
    //    "얼마나" 할인할지는 자식 클래스가 결정
    abstract protected Money getDiscountAmount(Screening screening);
}
```

### 🎨 디자인 패턴: TEMPLATE METHOD

**핵심 아이디어**:
> 알고리즘의 골격을 정의하고, 세부 단계는 자식 클래스에게 위임

```
calculateDiscountAmount()의 흐름:

1. 할인 조건 목록을 순회한다           ← 부모가 정의
     ↓
2. 각 조건을 하나씩 확인한다           ← 부모가 정의
     ↓
3. 조건을 만족하면?
     ↓
4. getDiscountAmount() 호출          ← 자식이 구현
     ↓
5. 할인 금액 반환                     ← 부모가 정의
```

**왜 이렇게 설계했는가?**

```java
// ✅ 공통 로직(할인 조건 확인)은 재사용
// ✅ 변하는 부분(할인 금액 계산)만 자식 클래스가 구현

// 예시 1: 금액 할인
class AmountDiscountPolicy extends DiscountPolicy {
    protected Money getDiscountAmount(Screening screening) {
        return Money.wons(800);  // 항상 800원
    }
}

// 예시 2: 비율 할인
class PercentDiscountPolicy extends DiscountPolicy {
    protected Money getDiscountAmount(Screening screening) {
        return screening.getMovieFee().times(0.1);  // 10% 할인
    }
}
```

### 💰 AmountDiscountPolicy - 금액 할인

> 📂 **코드**: [`AmountDiscountPolicy.java`](https://github.com/eternity-oop/object/blob/master/chapter02/src/main/java/org/eternity/movie/step01/pricing/AmountDiscountPolicy.java)

```java
package org.eternity.movie.step01.pricing;

import org.eternity.money.Money;
import org.eternity.movie.step01.DiscountCondition;
import org.eternity.movie.step01.DiscountPolicy;
import org.eternity.movie.step01.Screening;

public class AmountDiscountPolicy extends DiscountPolicy {
    private Money discountAmount;  // 할인 금액

    public AmountDiscountPolicy(Money discountAmount, 
                               DiscountCondition... conditions) {
        super(conditions);  // 부모에게 할인 조건 전달
        this.discountAmount = discountAmount;
    }

    @Override
    protected Money getDiscountAmount(Screening screening) {
        // 항상 고정 금액 할인
        return discountAmount;
    }
}
```

**사용 예시**:
```java
// "아바타"는 800원 금액 할인
//  - 1회차 상영 시
//  - 10회차 상영 시
//  - 월요일 10:00~12:00
Movie avatar = new Movie(
    "아바타",
    Duration.ofMinutes(120),
    Money.wons(10000),
    new AmountDiscountPolicy(
        Money.wons(800),                      // 800원 할인
        new SequenceCondition(1),             // 1회차
        new SequenceCondition(10),            // 10회차
        new PeriodCondition(                  // 월요일 오전
            DayOfWeek.MONDAY,
            LocalTime.of(10, 0),
            LocalTime.of(11, 59)
        )
    )
);

// 예매 시:
// 10000원 - 800원 = 9200원 × 인원수
```

### 📊 PercentDiscountPolicy - 비율 할인

> 📂 **코드**: [`PercentDiscountPolicy.java`](https://github.com/eternity-oop/object/blob/master/chapter02/src/main/java/org/eternity/movie/step01/pricing/PercentDiscountPolicy.java)

```java
package org.eternity.movie.step01.pricing;

import org.eternity.money.Money;
import org.eternity.movie.step01.DiscountCondition;
import org.eternity.movie.step01.DiscountPolicy;
import org.eternity.movie.step01.Screening;

public class PercentDiscountPolicy extends DiscountPolicy {
    private double percent;  // 할인 비율

    public PercentDiscountPolicy(double percent, 
                                DiscountCondition... conditions) {
        super(conditions);
        this.percent = percent;
    }

    @Override
    protected Money getDiscountAmount(Screening screening) {
        // 영화 요금의 일정 비율만큼 할인
        return screening.getMovieFee().times(percent);
    }
}
```

**사용 예시**:
```java
// "타이타닉"은 10% 비율 할인
//  - 2회차 상영 시
//  - 목요일 14:00~21:00
Movie titanic = new Movie(
    "타이타닉",
    Duration.ofMinutes(180),
    Money.wons(11000),
    new PercentDiscountPolicy(
        0.1,                                  // 10% 할인
        new SequenceCondition(2),             // 2회차
        new PeriodCondition(                  // 목요일 오후
            DayOfWeek.THURSDAY,
            LocalTime.of(14, 0),
            LocalTime.of(20, 59)
        )
    )
);

// 예매 시:
// 11000원 - (11000원 × 0.1) = 9900원 × 인원수
```

### ⛔ NoneDiscountPolicy - 할인 없음 (step01의 문제점)

> 📂 **코드**: [`NoneDiscountPolicy.java`](https://github.com/eternity-oop/object/blob/master/chapter02/src/main/java/org/eternity/movie/step01/pricing/NoneDiscountPolicy.java)

```java
package org.eternity.movie.step01.pricing;

import org.eternity.money.Money;
import org.eternity.movie.step01.DiscountPolicy;
import org.eternity.movie.step01.Screening;

public class NoneDiscountPolicy extends DiscountPolicy {
    @Override
    protected Money getDiscountAmount(Screening screening) {
        return Money.ZERO;  // 할인 없음
    }
}
```

### ❌ Step 01의 문제점

```java
public class NoneDiscountPolicy extends DiscountPolicy {
    // ❌ 문제: getDiscountAmount를 구현해야 함
    //    하지만 이 메서드는 절대 호출되지 않음!
    
    // 왜냐하면 부모의 calculateDiscountAmount에서:
    // for(DiscountCondition each : conditions) {
    //     if (each.isSatisfiedBy(screening)) {
    //         return getDiscountAmount(screening);  ← 조건이 없으면 실행 안됨
    //     }
    // }
    // return Money.ZERO;  ← 항상 여기서 반환
    
    @Override
    protected Money getDiscountAmount(Screening screening) {
        return Money.ZERO;  // 의미 없는 구현
    }
}
```

**개념적 혼란**:

```
NoneDiscountPolicy의 의도:
"할인 조건이 없다" → calculateDiscountAmount를 오버라이드해야 자연스러움

하지만 현재 구조:
"할인 금액 계산 방법이 없다" → getDiscountAmount를 오버라이드 (부자연스러움)
```

이 문제를 **Step 02**에서 해결합니다!

---

## 🔍 할인 조건: DiscountCondition

> 📂 **코드**: [`DiscountCondition.java`](https://github.com/eternity-oop/object/blob/master/chapter02/src/main/java/org/eternity/movie/step01/DiscountCondition.java)

```java
package org.eternity.movie.step01;

public interface DiscountCondition {
    // 이 상영이 할인 조건을 만족하는가?
    boolean isSatisfiedBy(Screening screening);
}
```

### 🔢 SequenceCondition - 순번 조건

> 📂 **코드**: [`SequenceCondition.java`](https://github.com/eternity-oop/object/blob/master/chapter02/src/main/java/org/eternity/movie/step01/pricing/SequenceCondition.java)

```java
package org.eternity.movie.step01.pricing;

import org.eternity.movie.step01.DiscountCondition;
import org.eternity.movie.step01.Screening;

public class SequenceCondition implements DiscountCondition {
    private int sequence;  // 할인 대상 순번

    public SequenceCondition(int sequence) {
        this.sequence = sequence;
    }

    public boolean isSatisfiedBy(Screening screening) {
        // 상영 순번이 일치하면 할인
        return screening.isSequence(sequence);
    }
}
```

**예시**:
```java
new SequenceCondition(1)   // 1회차 상영 시 할인
new SequenceCondition(10)  // 10회차 상영 시 할인
```

### 📅 PeriodCondition - 기간 조건

> 📂 **코드**: [`PeriodCondition.java`](https://github.com/eternity-oop/object/blob/master/chapter02/src/main/java/org/eternity/movie/step01/pricing/PeriodCondition.java)

```java
package org.eternity.movie.step01.pricing;

import org.eternity.movie.step01.DiscountCondition;
import org.eternity.movie.step01.Screening;
import java.time.DayOfWeek;
import java.time.LocalTime;

public class PeriodCondition implements DiscountCondition {
    private DayOfWeek dayOfWeek;  // 요일
    private LocalTime startTime;  // 시작 시간
    private LocalTime endTime;    // 종료 시간

    public PeriodCondition(DayOfWeek dayOfWeek, 
                          LocalTime startTime, 
                          LocalTime endTime) {
        this.dayOfWeek = dayOfWeek;
        this.startTime = startTime;
        this.endTime = endTime;
    }

    public boolean isSatisfiedBy(Screening screening) {
        // 1. 요일이 일치하고
        // 2. 시작 시간 이후이고
        // 3. 종료 시간 이전이면
        // → 할인
        return screening.getStartTime().getDayOfWeek().equals(dayOfWeek) &&
               startTime.compareTo(screening.getStartTime().toLocalTime()) <= 0 &&
               endTime.compareTo(screening.getStartTime().toLocalTime()) >= 0;
    }
}
```

**예시**:
```java
// 월요일 10:00~12:00
new PeriodCondition(
    DayOfWeek.MONDAY,
    LocalTime.of(10, 0),
    LocalTime.of(11, 59)
)

// 목요일 14:00~21:00
new PeriodCondition(
    DayOfWeek.THURSDAY,
    LocalTime.of(14, 0),
    LocalTime.of(20, 59)
)
```

---

## 🎯 Step 02: 인터페이스 기반 할인 정책 (개선)

> 📂 **전체 코드**: [step02 디렉토리](https://github.com/eternity-oop/object/tree/master/chapter02/src/main/java/org/eternity/movie/step02)

### Step 01 → Step 02 변경사항

**문제**: `NoneDiscountPolicy`가 `getDiscountAmount`를 구현해야 하는데, 이 메서드는 절대 호출되지 않음

**해결**: `DiscountPolicy`를 인터페이스로 분리

```
Step 01:
DiscountPolicy (abstract class)
   ├─ AmountDiscountPolicy
   ├─ PercentDiscountPolicy
   └─ NoneDiscountPolicy ← getDiscountAmount를 억지로 구현

Step 02:
DiscountPolicy (interface)
   ├─ DefaultDiscountPolicy (abstract class)
   │    ├─ AmountDiscountPolicy
   │    └─ PercentDiscountPolicy
   └─ NoneDiscountPolicy ← calculateDiscountAmount를 자연스럽게 구현
```

### 📜 DiscountPolicy - 인터페이스

> 📂 **코드**: [`DiscountPolicy.java`](https://github.com/eternity-oop/object/blob/master/chapter02/src/main/java/org/eternity/movie/step02/DiscountPolicy.java)

```java
package org.eternity.movie.step02;

import org.eternity.money.Money;

// ✅ 인터페이스로 변경
//    모든 할인 정책이 구현해야 하는 계약
public interface DiscountPolicy {
    Money calculateDiscountAmount(Screening screening);
}
```

### 📜 DefaultDiscountPolicy - 기본 구현

> 📂 **코드**: [`DefaultDiscountPolicy.java`](https://github.com/eternity-oop/object/blob/master/chapter02/src/main/java/org/eternity/movie/step02/DefaultDiscountPolicy.java)

```java
package org.eternity.movie.step02;

import org.eternity.money.Money;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

// ✅ 기존 DiscountPolicy의 구현을 그대로 가져옴
//    할인 조건을 확인하는 정책들의 기본 구현
public abstract class DefaultDiscountPolicy implements DiscountPolicy {
    private List<DiscountCondition> conditions = new ArrayList<>();

    public DefaultDiscountPolicy(DiscountCondition... conditions) {
        this.conditions = Arrays.asList(conditions);
    }

    @Override
    public Money calculateDiscountAmount(Screening screening) {
        for(DiscountCondition each : conditions) {
            if (each.isSatisfiedBy(screening)) {
                return getDiscountAmount(screening);
            }
        }

        return Money.ZERO;
    }

    abstract protected Money getDiscountAmount(Screening screening);
}
```

### ✅ AmountDiscountPolicy - 변경 없음

```java
// DefaultDiscountPolicy를 상속
public class AmountDiscountPolicy extends DefaultDiscountPolicy {
    private Money discountAmount;

    public AmountDiscountPolicy(Money discountAmount, 
                               DiscountCondition... conditions) {
        super(conditions);
        this.discountAmount = discountAmount;
    }

    @Override
    protected Money getDiscountAmount(Screening screening) {
        return discountAmount;
    }
}
```

### ✅ PercentDiscountPolicy - 변경 없음

```java
// DefaultDiscountPolicy를 상속
public class PercentDiscountPolicy extends DefaultDiscountPolicy {
    private double percent;

    public PercentDiscountPolicy(double percent, 
                                DiscountCondition... conditions) {
        super(conditions);
        this.percent = percent;
    }

    @Override
    protected Money getDiscountAmount(Screening screening) {
        return screening.getMovieFee().times(percent);
    }
}
```

### ✨ NoneDiscountPolicy - 개선!

> 📂 **코드**: [`NoneDiscountPolicy.java`](https://github.com/eternity-oop/object/blob/master/chapter02/src/main/java/org/eternity/movie/step02/pricing/NoneDiscountPolicy.java)

```java
package org.eternity.movie.step02.pricing;

import org.eternity.money.Money;
import org.eternity.movie.step02.DiscountPolicy;
import org.eternity.movie.step02.Screening;

// ✅ DiscountPolicy 인터페이스를 직접 구현
//    DefaultDiscountPolicy를 상속하지 않음
public class NoneDiscountPolicy implements DiscountPolicy {
    @Override
    public Money calculateDiscountAmount(Screening screening) {
        // ✅ 자연스러움: "할인 금액 계산하면 0원"
        return Money.ZERO;
    }
}
```

**개선 효과**:

```
Step 01 (부자연스러움):
NoneDiscountPolicy
  → getDiscountAmount 구현 (호출되지 않는 메서드)
  → "할인 금액을 어떻게 계산하는가?"에 답변

Step 02 (자연스러움):
NoneDiscountPolicy  
  → calculateDiscountAmount 구현
  → "할인 금액을 계산하면 얼마인가?"에 답변
```

### 설계 구조 비교

```
Step 02 최종 구조:

                 ┌──────────────────┐
                 │ DiscountPolicy   │
                 │   (interface)    │
                 └──────────────────┘
                          △
                          │ implements
           ┌──────────────┼──────────────┐
           │                             │
┌──────────────────────┐      ┌──────────────────┐
│ DefaultDiscountPolicy│      │NoneDiscountPolicy│
│  (abstract class)    │      │                  │
└──────────────────────┘      └──────────────────┘
           △
           │ extends
     ┌─────┴─────┐
     │           │
┌────────┐  ┌────────┐
│ Amount │  │Percent │
│Discount│  │Discount│
│ Policy │  │ Policy │
└────────┘  └────────┘
```

---

## 🎭 핵심 개념: 다형성 (Polymorphism)

### 컴파일 시간 vs 실행 시간 의존성

```java
// 컴파일 시간: Movie는 DiscountPolicy에 의존
public class Movie {
    private DiscountPolicy discountPolicy;
    
    public Money calculateMovieFee(Screening screening) {
        return fee.minus(discountPolicy.calculateDiscountAmount(screening));
    }
}

// 실행 시간: 실제로는 구체 클래스에 의존
Movie avatar = new Movie(
    "아바타",
    Duration.ofMinutes(120),
    Money.wons(10000),
    new AmountDiscountPolicy(...)  // ← 실행 시점에 결정!
);
```

**다이어그램**:

```
컴파일 시간 의존성:
Movie ──────→ DiscountPolicy (인터페이스)

실행 시간 의존성:
Movie ──────→ AmountDiscountPolicy (구체 클래스)
       또는
Movie ──────→ PercentDiscountPolicy (구체 클래스)
       또는
Movie ──────→ NoneDiscountPolicy (구체 클래스)
```

### 동적 바인딩 (Late Binding)

```java
Movie movie = ...;
Screening screening = ...;

// 이 메서드 호출 시:
movie.calculateMovieFee(screening)
  ↓
discountPolicy.calculateDiscountAmount(screening)
  ↓
// 실제 실행되는 메서드는?
// → 실행 시점에 연결된 객체에 따라 결정!

// AmountDiscountPolicy인 경우:
DefaultDiscountPolicy.calculateDiscountAmount()
  → AmountDiscountPolicy.getDiscountAmount()
  
// PercentDiscountPolicy인 경우:
DefaultDiscountPolicy.calculateDiscountAmount()
  → PercentDiscountPolicy.getDiscountAmount()
  
// NoneDiscountPolicy인 경우:
NoneDiscountPolicy.calculateDiscountAmount()
```

### 메시지와 메서드의 분리

```java
// 메시지: "할인 금액을 계산해줘"
discountPolicy.calculateDiscountAmount(screening)

// 메서드: 실제 실행되는 코드
// → AmountDiscountPolicy일 때: 고정 금액 반환
// → PercentDiscountPolicy일 때: 비율 계산 후 반환
// → NoneDiscountPolicy일 때: 0원 반환
```

**핵심**:
> 같은 메시지를 보내도, 받는 객체에 따라 다른 메서드가 실행된다!

---

## 🔄 실행 시점 할인 정책 변경

### 상속의 한계

```java
// ❌ 상속으로 할인 정책을 구현했다면?
public class AmountDiscountMovie extends Movie {
    private Money discountAmount;
    // ...
}

public class PercentDiscountMovie extends Movie {
    private double percent;
    // ...
}

// 문제: 실행 시점에 정책 변경 불가!
Movie avatar = new AmountDiscountMovie(...);
// 비율 할인으로 바꾸려면?
// → 객체를 새로 만들어야 함 (불가능)
```

### 합성의 유연성

```java
// ✅ 합성으로 할인 정책을 구현했으므로
public class Movie {
    private DiscountPolicy discountPolicy;
    
    // 실행 시점에 정책 변경 가능!
    public void changeDiscountPolicy(DiscountPolicy discountPolicy) {
        this.discountPolicy = discountPolicy;
    }
}

// 사용:
Movie avatar = new Movie(
    "아바타",
    Duration.ofMinutes(120),
    Money.wons(10000),
    new AmountDiscountPolicy(Money.wons(800), ...)
);

// 나중에 정책 변경!
avatar.changeDiscountPolicy(
    new PercentDiscountPolicy(0.1, ...)
);
```

**상속 vs 합성**:

| 측면 | 상속 | 합성 |
|------|------|------|
| **결합 시점** | 컴파일 시점 | 실행 시점 |
| **변경 가능성** | 불가능 | 가능 |
| **캡슐화** | 위반 (부모 내부 노출) | 유지 (인터페이스만 사용) |
| **유연성** | 낮음 | 높음 |
| **사용 목적** | 인터페이스 재사용 | 구현 재사용 |

---

---

## 🔬 실행 시점 상세 추적

각 할인 정책별로 실제 코드가 어떻게 실행되는지 완벽하게 추적해봅시다.

### 시나리오 1: 금액 할인 (AmountDiscountPolicy)

```java
Movie avatar = new Movie(
    "아바타",
    Duration.ofMinutes(120),
    Money.wons(10000),
    new AmountDiscountPolicy(
        Money.wons(800),
        new SequenceCondition(1)
    )
);

Screening screening = new Screening(avatar, 1, ...);  // 1회차
screening.reserve(customer, 2);
```

**실행 추적**:

```
1. screening.reserve(customer, 2)
   ↓
2. calculateFee(2)
   ↓
3. movie.calculateMovieFee(this)
   ├─ movie.fee = Money.wons(10000)
   └─ movie.discountPolicy = AmountDiscountPolicy 인스턴스
   ↓
4. discountPolicy.calculateDiscountAmount(screening)
   ↓ [DefaultDiscountPolicy.calculateDiscountAmount 실행]
   
5. for (DiscountCondition each : conditions) {
       // conditions = [SequenceCondition(1)]
   ↓
6. condition.isSatisfiedBy(screening)
   ├─ SequenceCondition.isSatisfiedBy(screening) 호출
   └─ screening.isSequence(1)
       └─ this.sequence == 1 → true ✅
   ↓
7. getDiscountAmount(screening)  // 추상 메서드 호출
   ↓ [실제로 AmountDiscountPolicy.getDiscountAmount 실행]
   
8. return discountAmount;
   └─ return Money.wons(800)
   ↓
9. movie.calculateMovieFee에서 계속
   fee.minus(Money.wons(800))
   └─ Money.wons(10000).minus(Money.wons(800))
       └─ return Money.wons(9200)
   ↓
10. calculateFee에서 계속
    Money.wons(9200).times(2)
    └─ return Money.wons(18400)
    ↓
11. new Reservation(customer, screening, Money.wons(18400), 2)
```

**핵심 포인트**:
- 4번에서 `discountPolicy`는 컴파일 시점에는 `DiscountPolicy` 타입
- 하지만 실행 시점에는 `AmountDiscountPolicy` 인스턴스
- 7번에서 `getDiscountAmount()`는 **동적 바인딩**으로 `AmountDiscountPolicy`의 메서드 실행

---

### 시나리오 2: 비율 할인 (PercentDiscountPolicy)

```java
Movie titanic = new Movie(
    "타이타닉",
    Duration.ofMinutes(180),
    Money.wons(11000),
    new PercentDiscountPolicy(
        0.1,  // 10% 할인
        new SequenceCondition(2)
    )
);

Screening screening = new Screening(titanic, 2, ...);  // 2회차
screening.reserve(customer, 3);
```

**실행 추적**:

```
1~6. [위와 동일]
   SequenceCondition(2).isSatisfiedBy(screening)
   └─ screening.isSequence(2) → true ✅
   ↓
7. getDiscountAmount(screening)
   ↓ [PercentDiscountPolicy.getDiscountAmount 실행]
   
8. screening.getMovieFee().times(percent)
   ├─ screening.getMovieFee()
   │   └─ movie.getFee()
   │       └─ return Money.wons(11000)
   └─ Money.wons(11000).times(0.1)
       └─ return Money.wons(1100)  // 11,000 × 10%
   ↓
9. fee.minus(Money.wons(1100))
   └─ Money.wons(11000).minus(Money.wons(1100))
       └─ return Money.wons(9900)
   ↓
10. Money.wons(9900).times(3)
    └─ return Money.wons(29700)
    ↓
11. new Reservation(customer, screening, Money.wons(29700), 3)
```

**핵심 포인트**:
- 같은 `getDiscountAmount()` 호출이지만
- `PercentDiscountPolicy`의 메서드가 실행됨
- 이것이 **다형성(Polymorphism)**

---

### 시나리오 3: 할인 없음 (NoneDiscountPolicy)

```java
Movie starWars = new Movie(
    "스타워즈",
    Duration.ofMinutes(210),
    Money.wons(10000),
    new NoneDiscountPolicy()
);

Screening screening = new Screening(starWars, 1, ...);
screening.reserve(customer, 2);
```

**실행 추적**:

```
1~3. [위와 동일]
   ↓
4. discountPolicy.calculateDiscountAmount(screening)
   ↓ [NoneDiscountPolicy.calculateDiscountAmount 실행]
   
5. return Money.ZERO;  // 바로 0원 반환!
   └─ DefaultDiscountPolicy를 거치지 않음
   ↓
6. fee.minus(Money.ZERO)
   └─ Money.wons(10000).minus(Money.ZERO)
       └─ return Money.wons(10000)
   ↓
7. Money.wons(10000).times(2)
   └─ return Money.wons(20000)
   ↓
8. new Reservation(customer, screening, Money.wons(20000), 2)
```

**핵심 포인트**:
- `NoneDiscountPolicy`는 할인 조건을 확인하지 않음
- `calculateDiscountAmount()`를 직접 구현
- 이것이 **Step 02의 개선점**

---

### 시나리오 4: 할인 조건 불만족

```java
Movie avatar = new Movie(
    "아바타",
    Duration.ofMinutes(120),
    Money.wons(10000),
    new AmountDiscountPolicy(
        Money.wons(800),
        new SequenceCondition(1)  // 1회차만 할인
    )
);

Screening screening = new Screening(avatar, 5, ...);  // 5회차 (조건 불만족)
screening.reserve(customer, 2);
```

**실행 추적**:

```
1~5. [위와 동일]
   ↓
6. condition.isSatisfiedBy(screening)
   └─ screening.isSequence(1)
       └─ this.sequence(5) == 1 → false ❌
   ↓
7. for문 종료 (조건을 만족하는 것이 없음)
   ↓
8. return Money.ZERO;  // DefaultDiscountPolicy에서 0원 반환
   ↓
9. fee.minus(Money.ZERO)
   └─ return Money.wons(10000)  // 할인 없음
   ↓
10. Money.wons(10000).times(2)
    └─ return Money.wons(20000)
```

**핵심 포인트**:
- 할인 조건을 만족하지 않으면 `getDiscountAmount()`가 호출되지 않음
- `DefaultDiscountPolicy`에서 `Money.ZERO` 반환
- 결과적으로 할인 없는 정상 가격

---

## 🏗️ 전체 협력 과정

### 예매 전체 흐름

```
1. 사용자 요청
screening.reserve(customer, 2)

2. Screening: 요금 계산 시작
calculateFee(2)
  ↓
movie.calculateMovieFee(this)

3. Movie: 할인 정책에 위임
discountPolicy.calculateDiscountAmount(screening)

4-1. AmountDiscountPolicy인 경우:
  ↓
DefaultDiscountPolicy.calculateDiscountAmount(screening)
  ↓
for each condition:
  if condition.isSatisfiedBy(screening):  // 조건 확인
    ↓
    getDiscountAmount(screening)  // 800원 반환
  ↓
return Money.wons(800)

4-2. PercentDiscountPolicy인 경우:
  ↓
DefaultDiscountPolicy.calculateDiscountAmount(screening)
  ↓
for each condition:
  if condition.isSatisfiedBy(screening):  // 조건 확인
    ↓
    getDiscountAmount(screening)  // 10% 계산
  ↓
return screening.getMovieFee().times(0.1)

4-3. NoneDiscountPolicy인 경우:
  ↓
NoneDiscountPolicy.calculateDiscountAmount(screening)
  ↓
return Money.ZERO

5. Movie: 할인 적용
fee.minus(할인금액)

6. Screening: 인원수 곱하기
1인요금.times(2)

7. Screening: 예매 생성
new Reservation(customer, this, 총요금, 2)

8. 예매 완료!
```

### 객체들의 협력 다이어그램

```
Customer         Screening         Movie         DiscountPolicy
   │                │                │                   │
   │  reserve()     │                │                   │
   │───────────────>│                │                   │
   │                │ calculateFee() │                   │
   │                │───────────────>│                   │
   │                │                │calculateDiscount()│
   │                │                │──────────────────>│
   │                │                │                   │
   │                │                │  [할인 조건 확인]    │
   │                │                │                   │
   │                │                │<──────────────────│
   │                │                │   할인 금액         │
   │                │<───────────────│                   │
   │                │   1인 요금       │                   │
   │                │                │                   │
   │                │ [인원수 곱하기]   │                   │
   │                │                │                   │
   │<───────────────│                │                   │
   │  Reservation   │                │                   │
```

---

## 💡 핵심 설계 원칙

### 1️⃣ 객체에 초점을 맞춰라

```java
// ❌ 클래스 중심 사고
"어떤 클래스가 필요한가?"
"Movie 클래스를 만들자"
"DiscountPolicy 클래스를 만들자"

// ✅ 객체 중심 사고
"영화 예매를 위해 어떤 객체들이 협력하는가?"
"상영 객체가 예매를 생성한다"
"영화 객체가 요금을 계산한다"
"할인 정책 객체가 할인 금액을 결정한다"
```

### 2️⃣ 협력하는 공동체로 바라봐라

```java
// 각 객체의 책임:
Screening  → 예매 생성
Movie      → 요금 계산
Discount   → 할인 결정
Condition  → 조건 확인

// 협력:
Screening이 Movie에게 요청
  → Movie가 DiscountPolicy에게 요청
    → DiscountPolicy가 Condition에게 요청
```

### 3️⃣ 도메인 구조를 따라라

```
도메인 개념              →    클래스
──────────────────────────────────────
영화 (Movie)            →    Movie
상영 (Screening)        →    Screening
할인 정책 (Policy)      →    DiscountPolicy
할인 조건 (Condition)   →    DiscountCondition
예매 (Reservation)      →    Reservation
```

### 4️⃣ 인터페이스에 의존하라

```java
// ✅ 좋은 설계
public class Movie {
    private DiscountPolicy discountPolicy;  // 인터페이스
}

// ❌ 나쁜 설계
public class Movie {
    private AmountDiscountPolicy discountPolicy;  // 구체 클래스
}
```

### 5️⃣ 캡슐화하라

```java
// ✅ 좋은 캡슐화
public class Screening {
    public Money getMovieFee() {
        return movie.getFee();  // Movie에게 요청
    }
}

// ❌ 나쁜 캡슐화
public class Screening {
    public Money getMovieFee() {
        return movie.fee;  // Movie 내부 노출
    }
}
```

---

## 🎨 디자인 패턴 정리

### TEMPLATE METHOD 패턴

**정의**: 알고리즘의 골격을 정의하고, 일부 단계를 서브클래스로 연기

**적용**: `DefaultDiscountPolicy.calculateDiscountAmount()`

```java
// 템플릿 메서드
public Money calculateDiscountAmount(Screening screening) {
    // 1. 조건 확인 (공통)
    for(DiscountCondition each : conditions) {
        if (each.isSatisfiedBy(screening)) {
            // 2. 할인 계산 (변하는 부분 - 서브클래스가 구현)
            return getDiscountAmount(screening);
        }
    }
    // 3. 할인 없음 (공통)
    return Money.ZERO;
}

// 서브클래스가 구현할 부분
abstract protected Money getDiscountAmount(Screening screening);
```

### STRATEGY 패턴

**정의**: 알고리즘 군을 정의하고 캡슐화하여 교환 가능하게 만듦

**적용**: `Movie`와 `DiscountPolicy`

```java
// Context
public class Movie {
    private DiscountPolicy discountPolicy;  // Strategy
    
    public Money calculateMovieFee(Screening screening) {
        // Strategy에게 위임
        return fee.minus(discountPolicy.calculateDiscountAmount(screening));
    }
}

// Strategy 교체 가능
movie.changeDiscountPolicy(new AmountDiscountPolicy(...));
movie.changeDiscountPolicy(new PercentDiscountPolicy(...));
```

---

## 🤔 실전 적용 가이드

### 1️⃣ 새로운 할인 정책 추가하기

**요구사항**: 첫 예매는 무료, 이후 정상 가격

```java
public class FirstFreeDiscountPolicy extends DefaultDiscountPolicy {
    public FirstFreeDiscountPolicy(DiscountCondition... conditions) {
        super(conditions);
    }
    
    @Override
    protected Money getDiscountAmount(Screening screening) {
        // 영화 요금 전액 할인
        return screening.getMovieFee();
    }
}

// 사용
Movie event = new Movie(
    "이벤트 영화",
    Duration.ofMinutes(90),
    Money.wons(8000),
    new FirstFreeDiscountPolicy(
        new SequenceCondition(1)  // 1회차만 무료
    )
);
```

**변경된 코드**: 0줄!
- Movie: 변경 없음
- Screening: 변경 없음
- 다른 할인 정책들: 변경 없음

### 2️⃣ 새로운 할인 조건 추가하기

**요구사항**: 생일인 사람은 할인

```java
public class BirthdayCondition implements DiscountCondition {
    private MonthDay birthday;
    
    public BirthdayCondition(MonthDay birthday) {
        this.birthday = birthday;
    }
    
    @Override
    public boolean isSatisfiedBy(Screening screening) {
        return MonthDay.from(screening.getStartTime()).equals(birthday);
    }
}

// 사용
Movie birthday = new Movie(
    "생일 이벤트",
    Duration.ofMinutes(100),
    Money.wons(10000),
    new PercentDiscountPolicy(
        0.5,  // 50% 할인
        new BirthdayCondition(MonthDay.of(12, 25))
    )
);
```

**변경된 코드**: 0줄!

### 3️⃣ 여러 할인 정책 중 최대 할인 선택

**요구사항**: 금액 할인과 비율 할인 중 큰 것 선택

```java
public class MaxDiscountPolicy implements DiscountPolicy {
    private List<DiscountPolicy> policies;
    
    public MaxDiscountPolicy(DiscountPolicy... policies) {
        this.policies = Arrays.asList(policies);
    }
    
    @Override
    public Money calculateDiscountAmount(Screening screening) {
        Money maxDiscount = Money.ZERO;
        
        for (DiscountPolicy policy : policies) {
            Money discount = policy.calculateDiscountAmount(screening);
            if (discount.isGreaterThan(maxDiscount)) {
                maxDiscount = discount;
            }
        }
        
        return maxDiscount;
    }
}

// 사용
Movie flexible = new Movie(
    "유연한 할인",
    Duration.ofMinutes(120),
    Money.wons(10000),
    new MaxDiscountPolicy(
        new AmountDiscountPolicy(Money.wons(1500), ...),
        new PercentDiscountPolicy(0.1, ...)
    )
);
```

---

## 💭 생각해보기

### Q1. 왜 Movie는 할인 정책을 직접 구현하지 않는가?

**A**: 단일 책임 원칙 (SRP)과 개방-폐쇄 원칙 (OCP)을 지키기 위해

```java
// ❌ Movie가 직접 구현하면?
public class Movie {
    private DiscountType type;  // AMOUNT, PERCENT, NONE
    private Money discountAmount;
    private double percent;
    
    public Money calculateMovieFee(Screening screening) {
        Money discount = Money.ZERO;
        
        // 할인 조건 확인
        boolean satisfied = false;
        for (Condition c : conditions) {
            if (c.check(screening)) {
                satisfied = true;
                break;
            }
        }
        
        if (satisfied) {
            // 할인 타입에 따라 분기
            if (type == DiscountType.AMOUNT) {
                discount = discountAmount;
            } else if (type == DiscountType.PERCENT) {
                discount = fee.times(percent);
            }
        }
        
        return fee.minus(discount);
    }
}

// 문제점:
// 1. Movie가 여러 책임을 가짐 (요금 계산 + 할인 정책)
// 2. 새 할인 정책 추가 시 Movie 수정 필요
// 3. 조건문이 Movie 곳곳에 퍼짐
```

### Q2. 할인 조건은 왜 인터페이스인가?

**A**: 다양한 할인 조건을 동일하게 취급하기 위해

```java
// 순번 조건과 기간 조건을 동일하게 취급
List<DiscountCondition> conditions = Arrays.asList(
    new SequenceCondition(1),
    new PeriodCondition(DayOfWeek.MONDAY, ...)
);

// 인터페이스 덕분에 같은 방식으로 확인
for (DiscountCondition each : conditions) {
    if (each.isSatisfiedBy(screening)) {
        // ...
    }
}
```

### Q3. 상속과 합성을 언제 사용하는가?

**A**:

**상속을 사용할 때**:
```java
// "is-a" 관계일 때
AmountDiscountPolicy is a DiscountPolicy  // ✅
PercentDiscountPolicy is a DiscountPolicy  // ✅

// 인터페이스를 재사용할 때 (다형성)
DiscountPolicy policy = new AmountDiscountPolicy(...);
```

**합성을 사용할 때**:
```java
// "has-a" 관계일 때
Movie has a DiscountPolicy  // ✅
Screening has a Movie       // ✅

// 실행 시점에 변경이 필요할 때
movie.changeDiscountPolicy(new PercentDiscountPolicy(...));
```

**조합해서 사용**:
```java
// Movie와 DiscountPolicy: 합성
public class Movie {
    private DiscountPolicy discountPolicy;  // 합성
}

// DiscountPolicy 계층: 상속
public class AmountDiscountPolicy extends DefaultDiscountPolicy {
    // 상속
}
```

---

<div align="center">

**[⬆ 목차로 돌아가기](../README.md)**

</div>
