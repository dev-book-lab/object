# Chapter 03. 역할, 책임, 협력

> *"객체지향의 본질은 협력하는 객체들의 공동체를 창조하는 것이다"*

## 📌 핵심 개념

- **협력(Collaboration)**: 객체들이 애플리케이션의 기능을 구현하기 위해 수행하는 상호작용
- **책임(Responsibility)**: 객체가 협력에 참여하기 위해 수행하는 행동
- **역할(Role)**: 협력 안에서 객체가 수행하는 책임의 집합
- **메시지(Message)**: 협력을 위한 유일한 커뮤니케이션 수단
- **정보 전문가(Information Expert)**: 책임을 수행하는 데 필요한 정보를 가장 잘 알고 있는 객체

---

## 🎯 학습 목표

1. 협력이 객체 설계의 문맥(context)이 됨을 이해하기
2. 책임 주도 설계(RDD)의 과정 익히기
3. 메시지가 객체를 결정하는 이유 이해하기
4. 행동이 상태를 결정해야 하는 이유 파악하기
5. 역할을 통한 유연하고 재사용 가능한 협력 설계하기

---

## ⚠️ 중요한 전제

### Chapter 03이 다루는 것

이 챕터는 **구현(implementation)**이 아닌 **개념(concept)**과 **설계 원칙(design principle)**에 집중합니다.

```
❌ 이 챕터에서 다루지 않는 것:
- 구체적인 코드 예제
- 클래스 구현 방법
- 상속과 다형성 구현 기법

✅ 이 챕터에서 다루는 것:
- 왜 협력이 중요한가?
- 어떻게 책임을 할당하는가?
- 역할은 무엇이며 왜 필요한가?
- 객체지향 설계의 사고 과정
```

### 클래스 vs 객체지향의 본질

```
구현 메커니즘 (2차적):
├─ 클래스
├─ 상속
├─ 다형성
└─ 인터페이스

객체지향의 본질 (1차적):
├─ 협력
├─ 책임
└─ 역할
```

**핵심**:
> 클래스와 상속은 객체들의 책임과 협력이 어느 정도 자리를 잡은 후에
> 사용할 수 있는 **구현 메커니즘**일 뿐이다.

---

## 🤝 01. 협력 (Collaboration)

### 협력이란 무엇인가?

```
협력 = 객체들이 애플리케이션의 기능을 구현하기 위해 수행하는 상호작용

┌─────────┐   메시지    ┌───────────┐   메시지    ┌─────────┐
│ 객체 A   │ ────────→ │ 객체 B      │ ────────→ │  객체 C  │
└─────────┘           └───────────┘            └─────────┘
    ↑                                              │
    │                  메시지                        │
    └──────────────────────────────────────────────┘
```

**협력의 특징**:
1. **메시지 전송**을 통해서만 협력 가능
2. **요청(request)**과 **응답(response)**의 흐름
3. 객체들 사이의 **수평적 관계** (계층 구조 X)

---

### 영화 예매 시스템의 협력

Chapter 02에서 본 영화 예매 시스템을 협력의 관점에서 다시 봅시다.

#### 전체 협력 흐름

```
사용자
                      │
                      │ "2명 예매해줘"
                      ↓
                 ┌─────────┐
                 │Screening│
                 └─────────┘
                      │
                      │ "1인당 요금 계산해줘"
                      ↓
                 ┌─────────┐
                 │  Movie  │
                 └─────────┘
                      │
                      │ "할인 금액 계산해줘"
                      ↓
              ┌───────────────┐
              │DiscountPolicy │
              └───────────────┘
                      │
                      │ "조건 만족해?"
                      ↓
              ┌───────────────────┐
              │DiscountCondition  │
              └───────────────────┘
                      │
                      │ true/false
                      ↓
              ┌───────────────┐
              │DiscountPolicy │
              └───────────────┘
                      │
                      │ 할인 금액 (Money)
                      ↓
                 ┌─────────┐
                 │  Movie  │
                 └─────────┘
                      │
                      │ 1인당 요금 (Money)
                      ↓
                 ┌─────────┐
                 │Screening│
                 └─────────┘
                      │
                      │ Reservation 객체
                      ↓
                    사용자
```

#### 코드로 보는 협력

```java
// 1. Screening이 협력 시작
public class Screening {
    public Reservation reserve(Customer customer, int audienceCount) {
        // 2. Movie에게 협력 요청 (메시지 전송)
        return new Reservation(
            customer, 
            this, 
            calculateFee(audienceCount),
            audienceCount
        );
    }
    
    private Money calculateFee(int audienceCount) {
        // Movie에게 "요금 계산해줘" 메시지 전송
        return movie.calculateMovieFee(this).times(audienceCount);
    }
}

// 3. Movie가 응답하기 위해 DiscountPolicy에게 협력 요청
public class Movie {
    public Money calculateMovieFee(Screening screening) {
        // DiscountPolicy에게 "할인 금액 계산해줘" 메시지 전송
        return fee.minus(discountPolicy.calculateDiscountAmount(screening));
    }
}

// 4. DiscountPolicy가 응답하기 위해 DiscountCondition에게 협력 요청
public class DiscountPolicy {
    public Money calculateDiscountAmount(Screening screening) {
        for(DiscountCondition each : conditions) {
            // DiscountCondition에게 "조건 만족해?" 메시지 전송
            if (each.isSatisfiedBy(screening)) {
                return getDiscountAmount(screening);
            }
        }
        return Money.ZERO;
    }
}
```

---

### 협력이 설계를 위한 문맥을 결정한다

#### 🤔 질문: Movie는 왜 영화를 재생(play)하는 메서드가 없는가?

```java
public class Movie {
    // ❓ 왜 play() 메서드가 없을까?
    // public void play() { ... }
    
    // ✅ 대신 이런 메서드가 있다
    public Money calculateMovieFee(Screening screening) { ... }
}
```

**답변**: Movie는 **"영화 예매"**라는 협력에 참여하기 때문

```
영화 재생 협력에 참여하는 Movie:
class Movie {
    void play();          // ✅
    void pause();         // ✅
    void stop();          // ✅
}

영화 예매 협력에 참여하는 Movie:
class Movie {
    Money calculateFee(); // ✅
    Duration getRuntime(); // ✅
}
```

**핵심**:
> **협력**이 객체가 어떤 **행동**을 가져야 하는지 결정한다.
> 객체를 단독으로 보지 말고, **"이 객체가 어떤 협력에 참여하는가?"**를 봐야 한다.

---

### 협력 → 행동 → 상태

```
1. 협력이 결정된다
   "영화를 예매한다"
   
2. 협력에 필요한 행동이 결정된다
   Movie: "요금을 계산한다"
   
3. 행동에 필요한 상태가 결정된다
   Movie: fee(기본 요금), discountPolicy(할인 정책)
```

**잘못된 순서**:
```java
// ❌ 상태부터 정의
class Movie {
    String title;
    int runningTime;
    // 이제 이걸로 뭘 하지?
}
```

**올바른 순서**:
```java
// ✅ 협력 → 행동 → 상태
// 1. 협력: "영화 예매"
// 2. 행동: "요금 계산"
// 3. 상태: 요금 계산에 필요한 것들

class Movie {
    private Money fee;              // 행동을 위해 필요
    private DiscountPolicy policy;  // 행동을 위해 필요
    
    public Money calculateMovieFee() {
        // 행동을 정의
    }
}
```

---

### 자율적인 객체와 협력

#### 자율성의 의미

```java
// ❌ 자율적이지 않은 객체
public class Screening {
    public Reservation reserve(Customer customer, int audienceCount) {
        // Screening이 Movie의 일까지 다 함
        Money fee = movie.getFee();
        Money discount = movie.getDiscountPolicy()
                              .calculateDiscountAmount(this);
        Money finalFee = fee.minus(discount).times(audienceCount);
        
        return new Reservation(customer, this, finalFee, audienceCount);
    }
}

// ✅ 자율적인 객체
public class Screening {
    public Reservation reserve(Customer customer, int audienceCount) {
        // Movie에게 "알아서 계산해줘" (위임)
        return new Reservation(
            customer,
            this,
            calculateFee(audienceCount),
            audienceCount
        );
    }
    
    private Money calculateFee(int audienceCount) {
        // Movie가 어떻게 계산하는지 모름!
        return movie.calculateMovieFee(this).times(audienceCount);
    }
}
```

**자율적인 객체의 조건**:
1. 자신의 일은 스스로 처리
2. 외부에서 내부 구현을 알 수 없음
3. 메시지만 전송하고 방법은 객체가 선택

---

## 📋 02. 책임 (Responsibility)

### 책임이란 무엇인가?

```
책임 = 객체가 협력에 참여하기 위해 수행하는 행동

책임은 추상적인 개념
메시지는 구체적인 구현
```

#### 책임의 두 가지 범주

**1️⃣ 하는 것 (Doing)**

```
- 객체를 생성하거나 계산을 수행하는 등의 스스로 하는 것
- 다른 객체의 행동을 시작시키는 것
- 다른 객체의 활동을 제어하고 조절하는 것

예시:
Screening의 "하는 것" 책임:
✓ 예매를 생성한다
✓ Movie에게 요금 계산을 시작시킨다

Movie의 "하는 것" 책임:
✓ 요금을 계산한다
✓ DiscountPolicy에게 할인 계산을 시작시킨다
```

**2️⃣ 아는 것 (Knowing)**

```
- 사적인 정보에 관해 아는 것
- 관련된 객체에 관해 아는 것
- 자신이 유도하거나 계산할 수 있는 것에 관해 아는 것

예시:
Screening의 "아는 것" 책임:
✓ 상영할 영화를 안다 (movie)
✓ 상영 순번을 안다 (sequence)
✓ 상영 시간을 안다 (whenScreened)

Movie의 "아는 것" 책임:
✓ 기본 요금을 안다 (fee)
✓ 할인 정책을 안다 (discountPolicy)
```

---

### 책임 vs 메시지

```
책임 (추상적, 개념적):
"예매를 생성한다"
"요금을 계산한다"
"할인 금액을 계산한다"

메시지 (구체적, 구현):
reserve(Customer, int)
calculateMovieFee(Screening)
calculateDiscountAmount(Screening)
```

**중요**:
```
책임 ≠ 메시지

하나의 책임이 여러 메시지로 구현될 수 있고,
하나의 메시지가 여러 책임을 포함할 수도 있다.
```

---

### 책임 할당: 정보 전문가 패턴

#### Information Expert Pattern

> **책임을 수행하는 데 필요한 정보를 가장 잘 알고 있는 전문가에게
> 그 책임을 할당하라**

#### 예제: 영화 예매 시스템 설계 과정

**🎯 Step 1: 시스템 책임 파악**

```
시스템 책임: "영화를 예매한다"

┌────────────────────────────┐
│   영화 예매 시스템             │
│                            │
│   책임: 영화를 예매한다         │
└────────────────────────────┘
```

**🎯 Step 2: 첫 번째 메시지 선택**

```
질문: "영화를 예매하려면 어떤 메시지가 필요한가?"
답변: "예매하라" (reserve)

┌─────────────┐
│  메시지:      │
│  "예매하라"   │
└─────────────┘
```

**🎯 Step 3: 정보 전문가 찾기**

```
질문: "예매를 생성하려면 무엇을 알아야 하는가?"
답변: 
- 상영 시간
- 상영할 영화
- 상영 순번

질문: "이 정보를 가장 잘 알고 있는 객체는?"
답변: Screening!

┌─────────────┐
│  Screening  │
│             │
│  ✓ movie    │
│  ✓ sequence │
│  ✓ whenScre │
└─────────────┘
```

**🎯 Step 4: 책임 할당**

```
Screening에게 "예매하라" 책임 할당!

┌─────────────────────────────┐
│      Screening              │
│                             │
│  책임: 예매를 생성한다           │
│  메시지: reserve()            │
└─────────────────────────────┘
```

**🎯 Step 5: 새로운 책임 발견**

```
Screening이 예매를 생성하려면...
→ 예매 가격을 알아야 함
→ Screening은 가격 계산 정보가 부족

새로운 메시지 필요: "가격을 계산하라"
```

**🎯 Step 6: 다시 정보 전문가 찾기**

```
질문: "가격을 계산하려면 무엇을 알아야 하는가?"
답변:
- 기본 요금
- 할인 정책

질문: "이 정보를 가장 잘 알고 있는 객체는?"
답변: Movie!

┌─────────────┐
│   Movie     │
│             │
│  ✓ fee      │
│  ✓ discount │
│    Policy   │
└─────────────┘
```

**🎯 Step 7: 책임 할당 (계속)**

```
Movie에게 "가격을 계산하라" 책임 할당!

┌─────────────────────────────┐
│        Movie                │
│                             │
│  책임: 요금을 계산한다           │
│  메시지: calculateMovieFee()  │
└─────────────────────────────┘
```

**🎯 Step 8: 협력 완성**

```
이런 식으로 계속...

1. 필요한 메시지 식별
2. 정보 전문가 찾기
3. 책임 할당
4. 새로운 필요 발견
5. 반복...

최종 결과:
Screening → Movie → DiscountPolicy → DiscountCondition
```

---

### 전체 책임 할당 과정 시각화

```
┌─────────────────────────────────────────────────────────┐
│                  1. 시스템 책임                            │
│               "영화를 예매한다"                             │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│               2. 메시지: "예매하라"                         │
│         정보 전문가: Screening                             │
│         (상영 정보를 가장 잘 알고 있음)                        │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│            3. Screening의 새로운 필요                      │
│           "가격을 계산해야 해!"                             │
│           → 메시지: "가격을 계산하라"                        │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│          4. 메시지: "가격을 계산하라"                         │
│         정보 전문가: Movie                                 │
│         (요금과 할인 정책을 가장 잘 알고 있음)                  │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│            5. Movie의 새로운 필요                          │
│         "할인 금액을 알아야 해!"                             │
│         → 메시지: "할인 금액을 계산하라"                       │
└─────────────────────────────────────────────────────────┘
                          ↓
                        ...
```

---

## 🎨 책임 주도 설계 (RDD)

### Responsibility-Driven Design

> **책임을 찾고, 책임을 수행할 적절한 객체를 찾아 할당하는 방식으로
> 협력을 설계하는 방법**

### RDD의 과정

```
1단계: 시스템이 사용자에게 제공해야 하는 기능(시스템 책임)을 파악한다
       ↓
2단계: 시스템 책임을 더 작은 책임으로 분할한다
       ↓
3단계: 분할된 책임을 수행할 수 있는 적절한 객체 또는 역할을 찾아 
      책임을 할당한다
       ↓
4단계: 객체가 책임을 수행하는 도중 다른 객체의 도움이 필요한 경우
      이를 책임질 적절한 객체 또는 역할을 찾는다
       ↓
5단계: 해당 객체 또는 역할에게 책임을 할당함으로써 
      두 객체가 협력하게 한다
       ↓
      (2단계로 돌아가 반복)
```

### RDD 실전 예제: 온라인 쇼핑몰 주문

**시나리오**: 사용자가 상품을 주문한다

#### Step 1: 시스템 책임

```
"상품을 주문한다"
```

#### Step 2: 첫 메시지와 정보 전문가

```
메시지: "주문하라" (placeOrder)

정보 전문가는?
- 주문하려면 사용자, 상품, 수량을 알아야 함
- 이 정보를 가장 잘 아는 것은? → Order

┌─────────────┐
│   Order     │
│             │
│ 책임:        │
│ 주문을 생성    │
└─────────────┘
```

#### Step 3: Order의 새로운 필요 발견

```
Order가 주문을 생성하려면...
→ 재고를 확인해야 함
→ Order는 재고 정보가 없음

새로운 메시지: "재고를 확인하라"

정보 전문가는?
- 재고를 가장 잘 아는 것은? → Product

┌─────────────┐
│  Product    │
│             │
│ 책임:        │
│ 재고 확인     │
└─────────────┘
```

#### Step 4: 계속 반복

```
Order → Product: "재고 확인"
Product → Stock: "재고 차감"
Order → Payment: "결제 처리"
...

최종 협력:
User → Order → Product → Stock
              ↓
           Payment
```

---

### 메시지가 객체를 결정한다

#### ❌ 잘못된 방법: 객체가 메시지를 선택

```java
// 1. 먼저 객체를 만듦
class OrderProcessor {
    void process() { ... }
    void validate() { ... }
    void save() { ... }
    void notify() { ... }
    // ... 계속 메서드 추가
}

// 2. 나중에 필요한 메시지를 찾음
// → 불필요한 메서드가 많아짐
// → 인터페이스가 비대해짐
```

#### ✅ 올바른 방법: 메시지가 객체를 선택

```
1. 필요한 메시지 식별
   "주문을 처리하라"
   
2. 이 메시지를 처리할 객체 선택
   OrderProcessor
   
3. 다음 메시지 식별
   "재고를 확인하라"
   
4. 이 메시지를 처리할 객체 선택
   InventoryChecker
   
→ 꼭 필요한 메시지만 가진 객체
→ 최소한의 인터페이스
```

#### 메시지가 객체를 선택해야 하는 이유

**1️⃣ 최소한의 인터페이스**

```java
// 메시지를 먼저 선택하면
class Movie {
    // 정말 필요한 것만
    Money calculateMovieFee(Screening screening);
}

// 객체를 먼저 만들면
class Movie {
    // 혹시 몰라서 다 만듦
    Money calculateMovieFee(Screening screening);
    Money getOriginalFee();
    Money getDiscountedFee();
    Money calculateFeeWithTax();
    Money calculateFeeWithoutTax();
    // ...
}
```

**2️⃣ 추상적인 인터페이스**

```java
// 메시지 중심 (What)
class Movie {
    Money calculateMovieFee(Screening screening); // ✅ 무엇을
}

// 구현 중심 (How)
class Movie {
    Money getFeeAfterSubtractingDiscountAmount(Screening screening); // ❌ 어떻게
}
```

---

### 행동이 상태를 결정한다

#### ❌ 잘못된 설계: 데이터 주도 설계

```java
// 1단계: 상태부터 정의
class Movie {
    private String title;
    private Duration runningTime;
    private Money fee;
    private String genre;
    private String director;
    // ... 더 많은 상태
}

// 2단계: getter/setter 기계적으로 생성
class Movie {
    public String getTitle() { return title; }
    public void setTitle(String title) { this.title = title; }
    public Duration getRunningTime() { return runningTime; }
    public void setRunningTime(Duration rt) { this.runningTime = rt; }
    // ...
}

// 3단계: 이제 뭘 하지?
// → 내부 구현이 모두 노출됨
// → 캡슐화 위반
```

**문제점**:
```
1. 상태가 노출되어 캡슐화 깨짐
2. 객체가 수동적이 됨 (다른 객체가 조작)
3. 객체의 존재 이유가 불명확
```

#### ✅ 올바른 설계: 행동 주도 설계

```java
// 1단계: 협력 정의
// "영화 예매를 위한 협력"

// 2단계: 필요한 행동 정의
// Movie는 "요금을 계산한다"

// 3단계: 행동 구현
class Movie {
    public Money calculateMovieFee(Screening screening) {
        // 행동을 구현하기 위해 필요한 상태만 추가
        return fee.minus(discountPolicy.calculateDiscountAmount(screening));
    }
    
    // 4단계: 필요한 상태 추가 (행동을 위해)
    private Money fee;
    private DiscountPolicy discountPolicy;
}
```

**장점**:
```
1. 캡슐화 유지 (상태 숨김)
2. 객체가 능동적 (스스로 처리)
3. 객체의 존재 이유 명확 (협력을 위해)
```

---

### 정리: 책임 할당의 흐름

```
┌──────────────────────────────────────┐
│      협력을 식별한다                     │
│   "영화를 예매한다"                      │
└──────────────────────────────────────┘
                ↓
┌──────────────────────────────────────┐
│    필요한 메시지를 결정한다                │
│   "예매하라"                           │
└──────────────────────────────────────┘
                ↓
┌──────────────────────────────────────┐
│   정보 전문가를 찾는다                    │
│   Screening이 상영 정보 전문가            │
└──────────────────────────────────────┘
                ↓
┌──────────────────────────────────────┐
│   책임을 할당한다                        │
│   Screening.reserve()                │
└──────────────────────────────────────┘
                ↓
┌──────────────────────────────────────┐
│   필요한 상태를 결정한다                   │
│   movie, sequence, whenScreened      │
└──────────────────────────────────────┘
                ↓
            (반복...)
```

---

## 🎭 03. 역할 (Role)

### 역할이란 무엇인가?

```
역할 = 협력 안에서 객체가 수행하는 책임의 집합

┌─────────────────────────────┐
│         역할                 │
│                             │
│   책임 1: ...                │
│   책임 2: ...                │
│   책임 3: ...                │
│                             │
│   ↑ 여러 객체가 교대로          │
│      이 역할을 수행 가능        │
└─────────────────────────────┘
        ↑       ↑       ↑
        │       │       │
      객체 A   객체 B    객체 C
```

### 역할의 필요성: 문제 상황

#### 시나리오: 할인 정책이 2가지인 경우

```
질문: "할인 금액을 계산하라" 메시지에 응답할 객체는?

답변 1: AmountDiscountPolicy (금액 할인)
답변 2: PercentDiscountPolicy (비율 할인)

❓ 두 가지 협력을 따로 만들어야 하나?
```

#### ❌ 역할 없이 설계하면?

```
협력 1: 금액 할인

Movie ────→ AmountDiscountPolicy ────→ SequenceCondition
                                  └───→ PeriodCondition

협력 2: 비율 할인

Movie ────→ PercentDiscountPolicy ────→ SequenceCondition
                                  └───→ PeriodCondition
```

**문제점**:
```
1. 협력이 중복됨
2. Movie 코드가 중복됨
3. 새로운 할인 정책마다 새로운 협력 필요
4. 유연하지 않음
```

---

### 역할을 통한 해결

#### ✅ 역할 도입

```
핵심 통찰:
AmountDiscountPolicy와 PercentDiscountPolicy 모두
"할인 금액을 계산한다"는 동일한 책임을 수행

→ 이 책임을 역할로 추상화!
```

**역할 이름**: DiscountPolicy

```
         ┌──────────────────┐
         │ DiscountPolicy   │  ← 역할 (추상화)
         │   (역할)          │
         │                  │
         │ 책임:             │
         │ 할인 금액 계산      │
         └──────────────────┘
                 △
                 │
         ┌───────┴───────┐
         │               │
┌────────────────┐ ┌────────────────┐
│ Amount         │ │ Percent        │  ← 객체들
│ DiscountPolicy │ │ DiscountPolicy │
└────────────────┘ └────────────────┘
```

#### 하나로 통합된 협력

```
통합된 협력:

Movie ────→ DiscountPolicy ────→ DiscountCondition
               (역할)

실행 시점에:
- AmountDiscountPolicy가 DiscountPolicy 역할 수행
- PercentDiscountPolicy가 DiscountPolicy 역할 수행
- NoneDiscountPolicy가 DiscountPolicy 역할 수행
```

**장점**:
```
1. ✅ 협력 하나로 통합
2. ✅ 중복 코드 제거
3. ✅ 새로운 할인 정책 추가 쉬움
4. ✅ 유연한 설계
```

---

### 역할 구현 방법

#### Java에서 역할 구현

```java
// 1. 인터페이스로 역할 정의
public interface DiscountPolicy {
    Money calculateDiscountAmount(Screening screening);
}

// 2. 구체적인 객체들이 역할을 구현
public class AmountDiscountPolicy implements DiscountPolicy {
    @Override
    public Money calculateDiscountAmount(Screening screening) {
        // 금액 할인 구현
    }
}

public class PercentDiscountPolicy implements DiscountPolicy {
    @Override
    public Money calculateDiscountAmount(Screening screening) {
        // 비율 할인 구현
    }
}

// 3. 역할에 의존하는 협력
public class Movie {
    private DiscountPolicy discountPolicy;  // 역할에 의존!
    
    public Money calculateMovieFee(Screening screening) {
        // 어떤 객체든 DiscountPolicy 역할을 수행하면 OK
        return fee.minus(discountPolicy.calculateDiscountAmount(screening));
    }
}
```

---

### 역할 vs 객체: 언제 무엇을 사용하는가?

#### 판단 기준

```
┌─────────────────────────────────────┐
│ 이 책임을 수행하는 대상이                 │
│ 몇 종류인가?                           │
└─────────────────────────────────────┘
         │
         ├─── 1종류 → 객체로 시작
         │            (나중에 필요하면 역할로)
         │
         └─── 2종류 이상 → 역할로 시작
```

#### 예시 1: 한 종류 → 객체

```java
// Screening은 한 종류만 있음
public class Screening {
    public Reservation reserve(Customer customer, int audienceCount) {
        // ...
    }
}

// 역할로 만들 필요 없음!
```

#### 예시 2: 여러 종류 → 역할

```java
// DiscountPolicy는 여러 종류가 있음
// → 역할로 정의!

interface DiscountPolicy { }

class AmountDiscountPolicy implements DiscountPolicy { }
class PercentDiscountPolicy implements DiscountPolicy { }
class NoneDiscountPolicy implements DiscountPolicy { }
```

---

### 역할 발견 과정

#### 객체로 시작해서 역할로 진화

```
1단계: 구체적인 객체로 시작
Movie ────→ AmountDiscountPolicy

2단계: 또 다른 객체 필요 발견
Movie ────→ PercentDiscountPolicy

3단계: 중복 발견
"두 객체가 같은 책임을 수행하네?"

4단계: 공통 책임 추출 → 역할 탄생!
         ┌──────────────┐
         │DiscountPolicy│  ← 역할
         └──────────────┘
                △
         ┌──────┴──────┐
    Amount         Percent

5단계: 역할 기반으로 협력 재설계
Movie ────→ DiscountPolicy (역할)
```

---

### 역할과 추상화

#### 추상화의 두 가지 장점

**1️⃣ 상위 수준 정책 표현**

```
구체적 (추상화 전):
"Movie는 AmountDiscountPolicy에게 할인 금액 계산을 요청하고,
 그 결과를 기본 요금에서 뺀다"

추상적 (추상화 후):
"Movie는 DiscountPolicy에게 할인 금액 계산을 요청하고,
 그 결과를 기본 요금에서 뺀다"

→ 세부사항(금액 할인인지, 비율 할인인지)을 감춤
→ 본질(할인 정책 적용)에 집중
```

**2️⃣ 유연한 설계**

```java
// 새로운 할인 정책 추가
public class SeasonDiscountPolicy implements DiscountPolicy {
    @Override
    public Money calculateDiscountAmount(Screening screening) {
        // 계절별 할인 구현
    }
}

// Movie 코드 변경 필요 없음!
Movie movie = new Movie(
    "아바타",
    Duration.ofMinutes(120),
    Money.wons(10000),
    new SeasonDiscountPolicy(...)  // ✅ 그냥 사용 가능
);
```

---

### 배우와 배역의 은유

#### 연극에서의 배역

```
연극: "햄릿"

배역: 햄릿
- 배역은 연극이 상영되는 동안만 존재
- 서로 다른 배우가 같은 배역 연기 가능
- 한 배우가 여러 연극에서 다른 배역 연기 가능

배우 A ────→ 햄릿 (연극 1)
       └───→ 오델로 (연극 2)

배우 B ────→ 햄릿 (연극 1)
       └───→ 맥베스 (연극 3)
```

#### 객체지향에서의 역할

```
협력: "영화 예매"

역할: DiscountPolicy
- 역할은 협력이 진행되는 동안만 존재
- 서로 다른 객체가 같은 역할 수행 가능
- 한 객체가 여러 협력에서 다른 역할 수행 가능

AmountDiscountPolicy ────→ DiscountPolicy (영화 예매 협력)

PercentDiscountPolicy ───→ DiscountPolicy (영화 예매 협력)
```

**핵심**:
```
배우는 여러 배역을 연기할 수 있지만,
한 연극 안에서는 하나의 배역만 연기

객체는 여러 역할을 수행할 수 있지만,
한 협력 안에서는 하나의 역할만 수행
```

---

## 🔄 전체 흐름 정리

### 객체지향 설계의 흐름

```
1. 협력을 설계한다
   "사용자가 영화를 예매한다"
   ↓
2. 협력에 필요한 메시지를 식별한다
   "예매하라"
   ↓
3. 메시지를 처리할 객체를 선택한다 (정보 전문가)
   Screening (상영 정보 전문가)
   ↓
4. 객체에게 책임을 할당한다
   Screening.reserve()
   ↓
5. 책임을 수행하는 데 필요한 상태를 결정한다
   movie, sequence, whenScreened
   ↓
6. 객체가 다른 객체의 도움이 필요한가?
   Yes → 2번으로 (새로운 메시지 식별)
   No  → 완료
   ↓
7. 여러 객체가 같은 책임을 수행하는가?
   Yes → 역할로 추상화
   No  → 객체 그대로 유지
```

---

## 💡 실전 적용 가이드

### 책임 주도 설계 체크리스트

```
설계 시작 전:
□ 어떤 협력이 필요한가?
□ 사용자에게 어떤 기능을 제공하는가?

책임 할당 시:
□ 이 책임을 수행하려면 무엇을 알아야 하는가?
□ 그 정보를 가장 잘 알고 있는 객체는 무엇인가?
□ 메시지를 먼저 선택했는가? (객체가 아닌)

설계 검증 시:
□ 객체가 상태가 아닌 행동으로 정의되었는가?
□ 여러 객체가 같은 책임을 수행하는가? (역할 필요?)
□ 협력이 유연하고 확장 가능한가?
```

---

### 잘못된 설계 패턴 발견하기

#### ❌ 패턴 1: 거대한 God Object

```java
// 모든 책임을 혼자 다 함
public class MovieSystem {
    public Reservation reserve(...) {
        // 예매 로직
        // 가격 계산 로직
        // 할인 계산 로직
        // 조건 확인 로직
        // 모든 것을 여기서!
    }
}
```

**해결**: 책임을 분산 → 협력하는 객체들로

---

#### ❌ 패턴 2: 데이터 클래스

```java
public class Movie {
    private String title;
    private Money fee;
    
    // getter/setter만 있음
    public String getTitle() { ... }
    public void setTitle(String title) { ... }
    public Money getFee() { ... }
    public void setFee(Money fee) { ... }
}
```

**해결**: 행동을 추가 → 자율적인 객체로

---

#### ❌ 패턴 3: 중복된 협력

```java
// 금액 할인용 Movie
public class AmountMovie { ... }

// 비율 할인용 Movie
public class PercentMovie { ... }

// 거의 같은 코드가 중복!
```

**해결**: 역할로 통합 → 하나의 협력으로

---

## 🤔 자주 하는 질문

### Q1. 협력을 먼저 생각한다는 게 구체적으로 뭔가요?

**A**: 객체를 먼저 만들지 말고, **"누가 누구에게 무엇을 요청하는가?"**를 먼저 생각하라는 의미

```
❌ 잘못된 순서:
1. Movie 클래스 만들기
2. DiscountPolicy 클래스 만들기
3. 이제 이걸로 뭘 하지?

✅ 올바른 순서:
1. "영화를 예매한다" (협력)
2. "예매하려면 누구에게 뭘 요청해야 하지?" (메시지)
3. "Screening에게 '예매하라' 요청" (객체 선택)
4. "Screening은 누구에게 뭘 요청해야 하지?" (다음 메시지)
5. 반복...
```

---

### Q2. 정보 전문가를 어떻게 찾나요?

**A**: **"이 책임을 수행하려면 무엇을 알아야 하는가?"**를 질문

```
예시: "예매를 생성하라"

질문: 예매를 생성하려면 무엇을 알아야 하는가?
- 어떤 영화를 상영하는지
- 언제 상영하는지
- 몇 번째 상영인지

질문: 이 정보를 누가 가장 잘 알고 있는가?
→ Screening!

결론: Screening이 정보 전문가
```

---

### Q3. 역할과 객체를 언제 구분하나요?

**A**: **설계 초반에는 구분하지 마세요!**

```
초반:
- 일단 구체적인 객체로 시작
- 협력을 설계하며 책임 할당

중반:
- "어? 이 객체랑 저 객체가 같은 책임을 수행하네?"
- 공통점 발견

후반:
- 공통 책임을 역할로 추출
- 역할 기반으로 재설계

결론:
필요할 때 객체에서 역할을 분리하는 것이 가장 좋은 방법!
```

---

### Q4. 메시지가 객체를 선택한다는 게 무슨 의미인가요?

**A**: **메시지를 먼저 정하고, 그 메시지를 처리할 객체를 나중에 선택**

```
❌ 객체가 메시지를 선택:
1. Movie 객체가 있다
2. Movie가 뭘 할 수 있을까?
3. calculate(), get(), set(), ... (메서드 나열)
→ 불필요한 메서드 많음

✅ 메시지가 객체를 선택:
1. "요금을 계산하라" 메시지가 필요하다
2. 이 메시지를 누가 처리할까?
3. Movie가 적절하다
4. Movie.calculateMovieFee() 메서드 추가
→ 꼭 필요한 메서드만
```

---

### Q5. 상태가 아닌 행동 중심이라는데, 상태는 언제 정의하나요?

**A**: **행동을 결정한 후에 상태를 정의**

```
순서:
1. 협력: "영화 예매"
2. 행동: Movie는 "요금을 계산한다"
3. 구현: calculateMovieFee() 메서드 작성
4. 상태: "어? 계산하려면 fee랑 discountPolicy가 필요하네"
5. 추가: private Money fee, private DiscountPolicy policy

핵심:
"이 상태가 필요해서 추가한다"가 아니라
"이 행동을 하려면 이 상태가 필요하네" 순서!
```

---

## 📚 핵심 정리

### 3문장 요약

```
1. 협력이 객체의 행동을 결정하고, 행동이 상태를 결정한다.

2. 메시지를 먼저 선택하고, 정보 전문가에게 책임을 할당한다.

3. 여러 객체가 같은 책임을 수행하면 역할로 추상화한다.
```

### 설계 원칙

```
📌 협력 중심
- 객체를 협력의 관점에서 바라봐라
- "이 객체가 어떤 협력에 참여하는가?"

📌 책임 할당
- 정보 전문가에게 책임을 할당하라
- 메시지를 먼저 선택하라

📌 행동 우선
- 상태가 아닌 행동으로 객체를 정의하라
- 협력 → 행동 → 상태 순서

📌 역할 활용
- 같은 책임은 역할로 추상화하라
- 필요할 때 객체에서 역할을 분리하라
```

---

## 📚 실전 예제: 온라인 쇼핑몰

### 잘못된 설계 (데이터 중심)

```java
// ❌ 상태부터 정의
public class Order {
    private Long userId;
    private Long productId;
    private Integer quantity;
    private Long totalPrice;
    
    // getter/setter만 가득
    public Long getUserId() { return userId; }
    public void setUserId(Long userId) { this.userId = userId; }
    // ...
}

public class Product {
    private Long id;
    private String name;
    private Long price;
    private Integer stock;
    
    // getter/setter만 가득
    public Long getPrice() { return price; }
    public void setPrice(Long price) { this.price = price; }
    // ...
}

// Service가 모든 로직 처리
public class OrderService {
    public Order createOrder(Long userId, Long productId, Integer quantity) {
        Product product = productRepository.findById(productId);
        
        // ❌ 재고 확인을 Service가 함
        if (product.getStock() < quantity) {
            throw new OutOfStockException();
        }
        
        // ❌ 재고 차감을 Service가 함
        product.setStock(product.getStock() - quantity);
        
        // ❌ 가격 계산을 Service가 함
        Long totalPrice = product.getPrice() * quantity;
        
        // ❌ Order 생성을 Service가 함
        Order order = new Order();
        order.setUserId(userId);
        order.setProductId(productId);
        order.setQuantity(quantity);
        order.setTotalPrice(totalPrice);
        
        orderRepository.save(order);
        productRepository.save(product);
        
        return order;
    }
}
```

**문제점**:
1. Order, Product는 데이터만 보관 (수동적)
2. OrderService가 모든 로직 처리 (God Object)
3. 캡슐화 위반 (getter/setter 남발)
4. Product, Order의 책임 불명확

---

### 올바른 설계 (책임 주도)

#### Step 1: 협력 정의

```
"사용자가 상품을 주문한다"

필요한 협력:
- 재고 확인
- 재고 차감
- 가격 계산
- 주문 생성
```

#### Step 2: 메시지 식별 및 책임 할당

```
1. "주문을 생성하라"
   → 정보 전문가: Order
   → Order가 주문 정보를 가장 잘 알고 있음

2. "재고를 확인하라"
   → 정보 전문가: Product
   → Product가 재고 정보를 가장 잘 알고 있음

3. "재고를 차감하라"
   → 정보 전문가: Product
   → Product가 재고를 가장 잘 알고 있음

4. "가격을 계산하라"
   → 정보 전문가: Product
   → Product가 가격 정보를 가장 잘 알고 있음
```

#### Step 3: 구현

```java
// ✅ 자율적인 객체
public class Product {
    private Long id;
    private String name;
    private Long price;
    private Integer stock;
    
    // ✅ 책임: 재고를 확인하고 차감한다
    public void decreaseStock(Integer quantity) {
        if (!hasEnoughStock(quantity)) {
            throw new OutOfStockException(
                "재고 부족: 현재 " + stock + "개, 요청 " + quantity + "개"
            );
        }
        this.stock -= quantity;
    }
    
    // ✅ 책임: 재고가 충분한지 확인한다
    private boolean hasEnoughStock(Integer quantity) {
        return this.stock >= quantity;
    }
    
    // ✅ 책임: 가격을 계산한다
    public Money calculatePrice(Integer quantity) {
        return Money.wons(this.price * quantity);
    }
    
    // getter만 제공 (setter 없음)
    public Long getId() { return id; }
    public Money getPrice() { return Money.wons(price); }
}

// ✅ 자율적인 객체
public class Order {
    private Long userId;
    private Long productId;
    private Integer quantity;
    private Money totalPrice;
    private OrderStatus status;
    
    // ✅ 책임: 주문을 생성한다
    public static Order create(
        Long userId,
        Product product,
        Integer quantity
    ) {
        // 1. Product에게 재고 차감 요청 (협력)
        product.decreaseStock(quantity);
        
        // 2. Product에게 가격 계산 요청 (협력)
        Money totalPrice = product.calculatePrice(quantity);
        
        // 3. 주문 생성
        return new Order(
            userId,
            product.getId(),
            quantity,
            totalPrice
        );
    }
    
    // ✅ private 생성자: 외부에서 직접 생성 불가
    private Order(
        Long userId,
        Long productId,
        Integer quantity,
        Money totalPrice
    ) {
        this.userId = userId;
        this.productId = productId;
        this.quantity = quantity;
        this.totalPrice = totalPrice;
        this.status = OrderStatus.PENDING;
    }
    
    // ✅ 책임: 주문을 취소한다
    public void cancel() {
        if (this.status != OrderStatus.PENDING) {
            throw new IllegalStateException("취소 불가능한 상태");
        }
        this.status = OrderStatus.CANCELLED;
    }
    
    // getter만 제공
    public OrderStatus getStatus() { return status; }
}

// ✅ Service는 조율만 담당
public class OrderService {
    public Order createOrder(Long userId, Long productId, Integer quantity) {
        // 1. 조회
        Product product = productRepository.findById(productId);
        
        // 2. 도메인 객체에게 위임 (협력 요청)
        Order order = Order.create(userId, product, quantity);
        
        // 3. 저장
        orderRepository.save(order);
        productRepository.save(product);
        
        return order;
    }
}
```

#### 협력 흐름

```
OrderService (조율자)
    │
    │ 1. "주문을 생성하라"
    ↓
Order.create()
    │
    │ 2. "재고를 차감하라"
    ↓
Product.decreaseStock()
    │
    │ 2-1. "재고가 충분한가?"
    │
    ├─→ hasEnoughStock() [private]
    │     └─→ true/false
    │
    │ 3. "가격을 계산하라"
    ↓
Product.calculatePrice()
    │
    │ Money 반환
    ↓
Order 생성 완료
```

---

### 개선 효과 비교

| 측면 | 데이터 중심 설계 | 책임 주도 설계 |
|------|-----------------|---------------|
| **캡슐화** | getter/setter 남발 | 행동만 노출 |
| **응집도** | Service에 모든 로직 집중 | 각 객체가 자신의 로직 처리 |
| **결합도** | Service가 모든 객체에 의존 | 인터페이스를 통한 협력 |
| **변경 영향** | Service 변경 시 전체 영향 | 해당 객체만 변경 |
| **테스트** | Service만 테스트 가능 | 각 객체 독립적 테스트 |
| **재사용성** | 낮음 (Service에 종속) | 높음 (객체 단위 재사용) |

---

## 🎓 심화 학습

### CRC 카드 (Class-Responsibility-Collaboration)

책임 주도 설계를 위한 도구로, 실제 카드를 사용해서 설계합니다.

```
┌─────────────────────────────────┐
│         Screening               │
├─────────────────────────────────┤
│ 책임 (Responsibilities):         │
│ - 예매를 생성한다                   │
│ - 예매 가격을 계산한다               │
├─────────────────────────────────┤
│ 협력자 (Collaborators):           │
│ - Movie                         │
│ - Reservation                   │
└─────────────────────────────────┘

┌─────────────────────────────────┐
│         Movie                   │
├─────────────────────────────────┤
│ 책임:                            │
│ - 영화 요금을 계산한다               │
├─────────────────────────────────┤
│ 협력자:                           │
│ - DiscountPolicy                │
└─────────────────────────────────┘
```

**사용 방법**:
1. 각 객체마다 카드 작성
2. 책임을 카드에 기록
3. 협력자를 파악하여 기록
4. 카드를 배치하며 협력 설계
5. 책임 재배치 및 정제

---

### 정보 전문가 vs 낮은 결합도

때로는 정보 전문가가 아닌 다른 객체에게 책임을 할당하는 것이 나을 수 있습니다.

#### 예제: 주문 총액 계산

```java
// 방법 1: 정보 전문가 (Order)
public class Order {
    private List<OrderItem> items;
    
    public Money calculateTotal() {
        Money total = Money.ZERO;
        for (OrderItem item : items) {
            // ❌ Order가 OrderItem의 내부를 알아야 함
            total = total.plus(
                item.getProduct().getPrice()
                    .times(item.getQuantity())
            );
        }
        return total;
    }
}

// 방법 2: 낮은 결합도 (OrderItem에게 위임)
public class Order {
    private List<OrderItem> items;
    
    public Money calculateTotal() {
        Money total = Money.ZERO;
        for (OrderItem item : items) {
            // ✅ OrderItem에게 위임 (캡슐화 유지)
            total = total.plus(item.calculatePrice());
        }
        return total;
    }
}

public class OrderItem {
    private Product product;
    private Integer quantity;
    
    // ✅ OrderItem이 자신의 가격 계산
    public Money calculatePrice() {
        return product.getPrice().times(quantity);
    }
}
```

**트레이드오프**:
- 정보 전문가: Order가 총액 정보를 가장 잘 알지만, OrderItem 내부를 알아야 함
- 낮은 결합도: OrderItem에게 위임하여 캡슐화 유지

**결정**: 낮은 결합도를 선택 (캡슐화가 더 중요)

---

## 💭 자주 묻는 질문 (심화)

### Q6. 정보 전문가를 찾을 때, 객체는 이미 존재해야 하나요?

**A**: 아니요! 정보 전문가 패턴은 **"어떤 객체가 필요한지"**를 찾는 과정입니다.

```
❌ 잘못된 이해:
"이미 만들어진 객체들 중에서 정보를 많이 아는 객체를 선택한다"

✅ 올바른 이해:
"이 책임을 수행하려면 무엇을 알아야 하는가?
 → 그 정보를 알고 있어야 할 객체는 무엇인가?
 → 그 객체가 존재하지 않으면 만든다"
```

**실제 과정**:

```
1. 협력 시나리오: "상품을 주문한다"

2. 메시지: "주문을 생성하라"

3. 질문: "주문을 생성하려면 무엇을 알아야 하는가?"
   답변: 사용자, 상품, 수량

4. 질문: "이 정보를 알고 있어야 할 객체는?"
   답변: Order (문맥상 주문 정보를 담당)

5. 결론: Order 객체가 필요하다!
   → Order 객체를 만든다
   → Order에게 "주문을 생성하라" 책임 할당
```

---

### Q7. 행동이 상태를 결정한다는데, 정보 전문가는 상태를 본다는 게 모순 아닌가요?

**A**: 정보 전문가도 행동 중심입니다. **출발점은 항상 행동**입니다.

```
순서 정리:

1. 협력 필요 발견
   "영화를 예매한다"

2. 필요한 행동(책임) 파악
   "예매를 생성한다"
   "요금을 계산한다"

3. 행동을 수행하려면 무엇을 알아야 하는가?
   "상영 시간, 영화, 순번을 알아야 한다"  ← 여기서 상태 등장

4. 그 정보를 알고 있어야 할 객체는?
   "Screening"

5. Screening 설계
   - 행동: reserve(), calculateFee()
   - 상태: movie, sequence, whenScreened  ← 행동 때문에 필요
```

**핵심**:
> 상태를 먼저 정의하는 것이 아니라,
> **"이 행동을 하려면 무엇을 알아야 하는가?"**를 질문하는 것

---

### Q8. 역할은 언제 만들어야 하나요?

**A**: 설계 초반에는 객체로 시작하고, **중복이 발견되면** 역할로 추출합니다.

#### 시나리오: 결제 시스템

**Phase 1: 객체로 시작**

```java
// 협력: "결제를 처리한다"
// 메시지: "결제하라"
// 정보 전문가: CreditCardPayment

public class Order {
    public void pay(CreditCardPayment payment) {
        payment.process(this.totalPrice);
    }
}

public class CreditCardPayment {
    public void process(Money amount) {
        // 신용카드 결제 처리
    }
}
```

**Phase 2: 새로운 결제 수단 추가**

```java
// 카카오페이 추가됨
public class KakaoPayPayment {
    public void process(Money amount) {
        // 카카오페이 결제 처리
    }
}

// ❌ 문제: Order를 두 개 만들어야 하나?
public class Order {
    public void pay(CreditCardPayment payment) { ... }
    public void pay(KakaoPayPayment payment) { ... }  // 중복!
}
```

**Phase 3: 역할 추출**

```java
// ✅ 공통 책임 발견: "결제를 처리한다"
// → 역할로 추출!

public interface Payment {  // 역할
    void process(Money amount);
}

public class CreditCardPayment implements Payment {
    @Override
    public void process(Money amount) {
        // 신용카드 결제
    }
}

public class KakaoPayPayment implements Payment {
    @Override
    public void process(Money amount) {
        // 카카오페이 결제
    }
}

// ✅ 하나의 협력으로 통합
public class Order {
    public void pay(Payment payment) {  // 역할에 의존
        payment.process(this.totalPrice);
    }
}
```

**정리**:
```
1. 처음: 구체적인 객체로 시작
2. 추가 요구사항: 비슷한 객체가 또 필요
3. 중복 발견: "어? 둘이 같은 책임을 수행하네?"
4. 역할 추출: 공통 책임을 역할로 만들기
```

---

### Q9. 협력을 먼저 설계한다는 게 구체적으로 무엇인가요?

**A**: **시나리오를 먼저 작성**하는 것입니다.

#### 실전 예제: 게시판 시스템

**❌ 잘못된 방법 (클래스부터)**

```
1. Board 클래스 만들기
   - 제목, 내용, 작성자, 날짜...

2. Comment 클래스 만들기
   - 내용, 작성자, 날짜...

3. 이제 이걸로 뭘 하지? 🤔
```

**✅ 올바른 방법 (협력부터)**

```
1. 사용자 스토리 작성
   "사용자가 게시글을 작성한다"
   "사용자가 게시글에 댓글을 단다"
   "사용자가 게시글을 수정한다"

2. 첫 번째 협력 시나리오
   "사용자가 게시글을 작성한다"
   
   a. 누가 이 요청을 받는가?
      → BoardService? Board?
   
   b. "게시글을 작성하라" 메시지
      → 정보 전문가는?
      → Board (제목, 내용 등을 알아야 함)
   
   c. Board가 작성되려면 뭐가 필요한가?
      → 작성자 정보
      → 작성자는 누가 아는가?
      → User

3. 협력 설계
   User ────→ Board.create()
       "게시글을 작성하라"

4. 두 번째 협력 시나리오
   "사용자가 댓글을 단다"
   
   Board ────→ Comment.create()
        "댓글을 추가하라"

5. 이제 클래스 구현
   (협력이 명확하므로 구현이 쉬움)
```

---

## 🔧 실전 적용: 단계별 가이드

### 1단계: 사용자 스토리 작성

```
예제: 쇼핑몰 장바구니

사용자 스토리:
- 사용자가 상품을 장바구니에 담는다
- 사용자가 장바구니의 상품을 삭제한다
- 사용자가 장바구니의 총액을 확인한다
- 사용자가 장바구니의 상품을 주문한다
```

### 2단계: 협력 시나리오 작성

```
시나리오 1: "상품을 장바구니에 담는다"

1. 사용자가 "담기" 버튼 클릭
2. 시스템이 상품 정보 확인
3. 장바구니에 상품 추가
4. 장바구니 개수 업데이트
```

### 3단계: 메시지 식별

```
필요한 메시지:
- "장바구니에 담아라" (addToCart)
- "상품 정보를 가져와라" (getProduct)
- "수량을 증가시켜라" (increaseQuantity)
```

### 4단계: 책임 할당

```
"장바구니에 담아라"
→ 정보 전문가: Cart
   (장바구니 항목들을 알고 있음)

"상품 정보를 가져와라"
→ 정보 전문가: ProductRepository
   (상품 정보를 알고 있음)

"수량을 증가시켜라"
→ 정보 전문가: CartItem
   (항목별 수량을 알고 있음)
```

### 5단계: 협력 설계

```
CartService (조율)
    │
    │ "상품을 가져와라"
    ↓
ProductRepository
    │
    │ Product 반환
    ↓
CartService
    │
    │ "장바구니에 담아라"
    ↓
Cart.addItem(product, quantity)
    │
    │ "같은 상품이 있나?"
    │
    ├─→ YES: CartItem.increaseQuantity()
    │
    └─→ NO: new CartItem(product, quantity)
```

### 6단계: 구현

```java
public class Cart {
    private List<CartItem> items;
    
    public void addItem(Product product, Integer quantity) {
        CartItem existing = findItem(product.getId());
        
        if (existing != null) {
            existing.increaseQuantity(quantity);
        } else {
            items.add(new CartItem(product, quantity));
        }
    }
    
    private CartItem findItem(Long productId) {
        return items.stream()
            .filter(item -> item.hasProduct(productId))
            .findFirst()
            .orElse(null);
    }
    
    public Money calculateTotal() {
        return items.stream()
            .map(CartItem::calculatePrice)
            .reduce(Money.ZERO, Money::plus);
    }
}

public class CartItem {
    private Product product;
    private Integer quantity;
    
    public void increaseQuantity(Integer amount) {
        this.quantity += amount;
    }
    
    public Money calculatePrice() {
        return product.getPrice().times(quantity);
    }
    
    public boolean hasProduct(Long productId) {
        return this.product.getId().equals(productId);
    }
}
```

---

## 📊 설계 품질 체크리스트

### ✅ 좋은 설계의 신호

```
□ 각 객체가 명확한 책임을 가지고 있다
□ 협력이 명확하게 보인다
□ 객체가 자신의 데이터를 스스로 처리한다
□ getter/setter가 최소화되어 있다
□ Service 레이어가 조율만 한다
□ 도메인 객체에 비즈니스 로직이 있다
□ 새로운 기능 추가 시 기존 코드 수정이 최소화된다
```

### ⚠️ 나쁜 설계의 신호

```
□ Service가 모든 로직을 처리한다 (God Object)
□ 도메인 객체가 getter/setter만 있다 (Anemic Domain)
□ 한 객체가 다른 객체 내부를 마음대로 조작한다
□ 협력 없이 모든 것을 한 곳에서 처리한다
□ 객체의 존재 이유가 불명확하다
□ 비슷한 코드가 여러 곳에 중복된다
```

---

## 🎯 핵심 원칙 요약

### 설계 순서

```
1. 협력 시나리오 작성
   ↓
2. 필요한 메시지 식별
   ↓
3. 정보 전문가 찾기
   ↓
4. 책임 할당
   ↓
5. 협력 완성
   ↓
6. 역할 추출 (필요시)
```

### 기억할 3가지

```
1. 협력이 객체를 만든다
   (객체부터 만들지 마라)

2. 메시지가 인터페이스를 결정한다
   (객체가 메시지를 선택하지 않는다)

3. 행동이 상태를 결정한다
   (상태부터 정의하지 마라)
```

---

<div align="center">

**[⬆ 목차로 돌아가기](../README.md)**

</div>
