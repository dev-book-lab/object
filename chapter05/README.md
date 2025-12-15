# Chapter 05. 책임 할당하기

> *"객체에게 책임을 할당하는 것이 설계의 시작이다"*

## 📌 핵심 개념

- **GRASP 패턴**: 일반적인 책임 할당을 위한 소프트웨어 패턴
- **정보 전문가(Information Expert)**: 책임을 수행할 정보를 알고 있는 객체에게 책임 할당
- **창조자(Creator)**: 객체 생성 책임을 적절한 객체에게 할당
- **낮은 결합도(Low Coupling)**: 의존성을 낮춰 변경의 영향을 최소화
- **높은 응집도(High Cohesion)**: 관련된 책임을 하나의 모듈에 집중
- **다형성(Polymorphism)**: 타입에 따라 변하는 행동을 역할로 추상화
- **변경 보호(Protected Variations)**: 변경을 캡슐화하여 안정성 확보

---

## 🎯 학습 목표

1. **데이터가 아닌 책임**으로 설계를 시작하는 방법 익히기
2. **메시지가 객체를 선택**하는 과정 이해하기
3. **GRASP 패턴**을 활용한 체계적인 책임 할당 방법 습득하기
4. 협력 안에서 객체의 **자율성**을 보장하는 방법 배우기
5. **리팩터링**을 통한 책임 주도 설계로의 전환 경험하기

---

## 🔄 Chapter 04 복습: 무엇이 문제였나?

### Chapter 04의 교훈

Chapter 04에서 우리는 **데이터 중심 설계의 한계**를 배웠습니다:

```
데이터 중심 설계의 문제점:
1. 캡슐화 위반 → 내부 구현이 인터페이스에 노출
2. 높은 결합도 → 변경이 여러 클래스로 전파
3. 낮은 응집도 → 하나의 변경에 여러 모듈 수정
```

**핵심 질문**:
```
"이 객체가 포함해야 하는 데이터가 무엇인가?" ❌
         ↓
"이 객체가 수행해야 하는 책임은 무엇인가?" ✅
```

### 하지만 여전히 어렵다

책임 중심 설계가 좋다는 것은 알겠는데...

```
❓ "어떤 객체에게 어떤 책임을 할당해야 할까?"
❓ "협력을 어떻게 시작해야 할까?"
❓ "객체를 먼저 만들어야 할까, 메시지를 먼저 만들어야 할까?"
```

**Chapter 05는 이 질문들에 대한 구체적인 답을 제시합니다.**

---

## 🎬 01. 책임 주도 설계를 향해

### 📖 두 가지 핵심 원칙

책임 중심 설계로 전환하기 위한 **두 가지 필수 원칙**:

#### 원칙 1: 데이터보다 행동을 먼저 결정하라

**잘못된 접근**:
```java
// 1단계: 데이터부터 생각
class Movie {
    private String title;        // 어떤 데이터가 필요하지?
    private Duration runningTime;
    private Money fee;
    // ...
}

// 2단계: 데이터를 처리할 메서드 추가
class Movie {
    // ...
    public Money getFee() { ... }      // 데이터에 접근하는 메서드
    public void setFee(Money fee) { ... }
}
```

**올바른 접근**:
```java
// 1단계: 책임(행동)부터 생각
"Movie는 무엇을 해야 하는가?"
        → "영화 요금을 계산해야 한다"

// 2단계: 책임을 수행할 데이터 결정
class Movie {
    public Money calculateMovieFee(Screening screening) {
        // 이 책임을 수행하기 위해 필요한 데이터는?
        // → fee, discountPolicy
    }
}
```

**왜 중요한가?**

```
데이터 먼저 → 구현 먼저 결정 → 캡슐화 실패
행동 먼저 → 인터페이스 먼저 결정 → 캡슐화 성공

객체의 존재 이유 = 협력에서 책임을 수행하기 위함
데이터는 그저 책임을 수행하는 재료일 뿐
```

#### 원칙 2: 협력이라는 문맥 안에서 책임을 결정하라

**핵심 통찰**:
> 객체가 어떤 책임을 가져야 하는지는 **협력**이 결정한다

**잘못된 질문**:
```
"Movie 클래스가 필요한 것 같은데, 뭘 해야 하지?" ❌

문제점: 객체를 먼저 생각하고 책임을 나중에 생각
       → 협력을 고려하지 않은 고립된 객체
```

**올바른 질문**:
```
"영화 요금을 계산해야 하는데, 누구한테 요청하지?" ✅

장점: 메시지(필요성)를 먼저 생각하고 객체를 나중에 결정
      → 협력에 꼭 필요한 객체만 탄생
```

**메시지가 객체를 선택한다**

```
전통적 방식:
객체 → 메시지
"이 객체는 이런 메시지를 보낼 수 있어"

책임 주도 설계:
메시지 → 객체
"이 메시지가 필요해, 누가 처리할 수 있지?"
```

**예시**:

```java
// ❌ 객체 중심
Movie movie = new Movie();  // Movie부터 만들고
movie.getFee();            // 메시지를 나중에 생각

// ✅ 메시지 중심
// "영화 요금 계산"이라는 메시지가 필요하다
// → 이 메시지를 처리할 객체는? → Movie가 적합
Money fee = movie.calculateMovieFee(screening);
```

**왜 이것이 캡슐화를 보장하는가?**

```
메시지를 먼저 결정하면:
→ 송신자는 수신자의 내부를 가정할 수 없음
→ 수신자는 메시지만 처리하면 됨
→ 내부 구현이 자연스럽게 캡슐화됨

객체를 먼저 결정하면:
→ 객체가 가진 데이터부터 생각
→ 데이터를 노출하는 인터페이스 만듦
→ 캡슐화 실패
```

---

### 🎯 책임 주도 설계 프로세스

Chapter 03에서 배운 내용을 다시 정리하면:

```
1️⃣ 시스템이 사용자에게 제공해야 하는 기능 파악
   → 시스템 책임

2️⃣ 시스템 책임을 더 작은 책임으로 분할
   → 여러 객체의 책임으로 나눔

3️⃣ 분할된 책임을 수행할 객체/역할 찾기
   → 책임 할당 (GRASP 패턴 활용!)

4️⃣ 객체가 책임 수행 중 도움이 필요하면
   → 새로운 메시지 정의
   → 메시지를 처리할 객체 찾기
   → 책임 할당

5️⃣ 3-4 반복
   → 협력 공동체 완성
```

**핵심**: 항상 **책임 → 객체** 순서!

---

## 📚 02. 책임 할당을 위한 GRASP 패턴

### GRASP란?

```
GRASP
= General Responsibility Assignment Software Patterns
= 일반적인 책임 할당을 위한 소프트웨어 패턴

목적: "어떤 객체에게 어떤 책임을 할당할까?"라는
      영원한 질문에 대한 체계적인 답 제공
```

**GRASP의 가치**:
- 경험에만 의존하지 않고 **원칙**에 따라 설계
- 여러 설계 대안 중 **더 나은 선택**의 기준 제공
- 설계 의사결정을 **설명**하고 **공유**할 수 있는 언어

---

### 🗺️ 1단계: 도메인 개념에서 출발하기

> **"설계를 시작하기 전에 도메인을 대략적으로 그려보자"**

#### 영화 예매 시스템 도메인 모델

```
┌──────────┐        ┌──────────┐
│  Movie   │◆───────│ Discount │
│  (영화)   │        │ Condition│
│          │        │(할인조건)  │
└──────────┘        └──────────┘
     │
     │ has
     ↓
┌──────────┐
│Screening │
│  (상영)   │
└──────────┘
     │
     │ creates
     ↓
┌───────────┐        ┌──────────┐
│Reservation│─────→  │ Customer │
│  (예매)    │        │  (고객)   │
└───────────┘        └──────────┘
```

**도메인 개념들**:
- **영화(Movie)**: 제목, 상영시간, 기본 요금
- **상영(Screening)**: 실제 상영 일시, 상영 순번
- **할인 조건(DiscountCondition)**: 순번 조건, 기간 조건
- **예매(Reservation)**: 고객, 상영, 요금, 인원수
- **고객(Customer)**: 이름, ID

#### ⚠️ 중요한 주의사항

```
도메인 모델의 목적:
✅ 설계의 출발점 제공
✅ 책임 할당의 후보 식별
✅ 개념 간 관계 파악

❌ 완벽한 모델 만들기
❌ 데이터베이스 설계
❌ 클래스 다이어그램

"도메인 모델을 정리하는 데 너무 많은 시간을 들이지 말라"
```

**올바른 도메인 모델이란?**

```
❌ "도메인을 완벽하게 반영한 모델"
✅ "구현에 도움이 되는 모델"

도메인 모델은:
- 설계를 위한 힌트 제공
- 빠르게 그리고 빠르게 구현으로 이동
- 구현하면서 계속 수정
```

---

### 👨‍🏫 2단계: 정보 전문가에게 책임을 할당하라

> **INFORMATION EXPERT 패턴**
>
> "책임을 수행할 정보를 알고 있는 객체에게 책임을 할당하라"

#### 시작: 시스템이 제공할 기능 정의

```
사용자 관점의 기능: "영화를 예매한다"

이것을 시스템의 책임으로 변환:
→ "시스템은 영화를 예매할 책임이 있다"

이것을 메시지로 표현:
→ "예매하라" 메시지
```

#### 1️⃣ 첫 번째 질문: 메시지를 전송할 객체는 무엇을 원하는가?

```
협력을 시작할 객체(아직 미정)가 원하는 것:
→ "영화를 예매하고 싶다"

메시지 이름:
→ "예매하라" (reserve)
```

**메시지 먼저!**

```java
// 아직 객체는 모르지만 메시지는 정했다
???.reserve(customer, audienceCount);
```

#### 2️⃣ 두 번째 질문: 메시지를 수신할 적합한 객체는 누구인가?

**INFORMATION EXPERT 패턴 적용**:

```
"예매하라" 메시지를 처리하려면 어떤 정보가 필요한가?
- 상영 정보 (영화, 시간, 순번)
- 고객 정보
- 인원수

이 정보를 가장 많이 알고 있는 객체는?
→ Screening (상영)
```

**왜 Screening이 정보 전문가인가?**

```
Screening이 알고 있는 것:
✅ 어떤 영화가 상영되는지 (Movie 참조)
✅ 언제 상영되는지 (whenScreened)
✅ 몇 번째 상영인지 (sequence)

예매에 필요한 핵심 정보를 모두 보유!
```

#### 책임 할당

```java
public class Screening {
    private Movie movie;
    private int sequence;
    private LocalDateTime whenScreened;

    // ✅ "예매하라" 메시지에 응답할 책임
    public Reservation reserve(Customer customer, int audienceCount) {
        // 책임을 수행하는 방법은 아직 미정
    }
}
```

**INFORMATION EXPERT 패턴의 핵심**:

```
정보 ≠ 데이터

"정보를 안다" ≠ "데이터를 저장한다"

정보를 안다 = 다음 중 하나:
1. 데이터를 직접 가지고 있다
2. 데이터를 가진 객체를 알고 있다
3. 필요한 정보를 계산할 수 있다
```

---

#### 3️⃣ 책임 수행을 위한 협력 필요

Screening이 예매 책임을 수행하려면:

```
필요한 것:
1. 예매 요금 계산
   → Screening이 직접 할 수 있나? ❌
   → 영화 요금은 Movie가 알고 있음
   
2. Reservation 생성
   → Screening이 할 수 있나? ✅
```

**새로운 메시지 탄생**:

```java
public class Screening {
    public Reservation reserve(Customer customer, int audienceCount) {
        // 1. 영화 요금 계산이 필요하다
        // → "가격을 계산하라" 메시지 전송
        Money fee = movie.calculateMovieFee(this);

        // 2. 예매 생성
        return new Reservation(customer, this, fee, audienceCount);
    }
}
```

**협력의 시작**:

```
Screening ─────"calculateMovieFee"────→ Movie
    │
    └─ "나는 요금 계산을 못해"
    └─ "Movie에게 도움을 요청"
```

---

#### 4️⃣ 연쇄적 책임 할당: Movie

**다시 INFORMATION EXPERT 패턴 적용**:

```
메시지: "영화 요금을 계산하라"
정보 전문가: Movie (요금과 할인 정책을 알고 있음)
```

```java
public class Movie {
    private Money fee;
    private List<DiscountCondition> discountConditions;

    // ✅ "영화 요금 계산하라" 메시지에 응답
    public Money calculateMovieFee(Screening screening) {
        // 할인 가능한지 확인 필요
        // → DiscountCondition에게 물어봐야 함
    }
}
```

**또 다른 협력 필요**:

```
Movie ─────"isSatisfiedBy"────→ DiscountCondition
  │
  └─ "할인 조건 만족 여부를 내가 직접 판단할 수 없어"
  └─ "DiscountCondition에게 물어봐야 해"
```

---

#### 5️⃣ 최종 협력: DiscountCondition

```java
public class DiscountCondition {
    private DiscountConditionType type;
    private int sequence;
    private DayOfWeek dayOfWeek;
    private LocalTime startTime;
    private LocalTime endTime;

    // ✅ "조건을 만족하는가?" 메시지에 응답
    public boolean isSatisfiedBy(Screening screening) {
        if (type == DiscountConditionType.PERIOD) {
            return isSatisfiedByPeriod(screening);
        }
        return isSatisfiedBySequence(screening);
    }
}
```

---

#### 🎯 완성된 협력 구조

```
[사용자]
   ↓
reserve
   ↓
[Screening] ────calculateMovieFee───→ [Movie]
   │                                     ↓
   │                               isSatisfiedBy
   │                                     ↓
   │                             [DiscountCondition]
   │
   └──→ [Reservation] 생성
```

**책임의 흐름**:

```
1. Screening: "예매를 생성한다"
   ├─ 협력 필요: Movie에게 "요금 계산" 요청
   
2. Movie: "영화 요금을 계산한다"
   ├─ 협력 필요: DiscountCondition에게 "조건 만족 여부" 확인
   
3. DiscountCondition: "할인 조건 만족 여부를 판단한다"
   └─ 스스로 해결 가능!
```

**INFORMATION EXPERT의 위력**:

```
✅ 각 객체는 자신이 아는 정보로 책임을 수행
✅ 모르는 것은 아는 객체에게 위임
✅ 자연스럽게 협력 구조 형성
✅ 응집도 높고 결합도 낮은 설계
```

---

### 🔗 3단계: 낮은 결합도와 높은 응집도

> **설계는 트레이드오프의 산물**

같은 기능을 구현하는 방법은 여러 가지가 있습니다. 이때 어떤 설계를 선택해야 할까요?

#### 설계 대안 비교

**상황**: Movie가 할인 조건을 확인하는 방법

**대안 1**: Screening이 DiscountCondition과 직접 협력

```
[Screening] ────calculateMovieFee───→ [Movie]
     │
     └────isSatisfiedBy───→ [DiscountCondition]
```

**대안 2**: Movie가 DiscountCondition과 협력 (현재 설계)

```
[Screening] ────calculateMovieFee───→ [Movie]
                                         ↓
                                   isSatisfiedBy
                                         ↓
                                [DiscountCondition]
```

**어떤 것이 더 나을까?**

---

#### LOW COUPLING 패턴 (낮은 결합도)

> "여러 설계 중 결합도가 더 낮은 것을 선택하라"

**대안 1 분석**:

```
Screening의 의존성:
- Movie (이미 의존)
- DiscountCondition (새로운 의존성!) ❌

문제점:
→ Screening이 DiscountCondition에도 결합
→ DiscountCondition 변경 시 Screening도 영향
→ 결합도 증가
```

**대안 2 분석**:

```
Screening의 의존성:
- Movie (이미 의존)

Movie의 의존성:
- DiscountCondition (이미 의존)

장점:
→ 새로운 결합도가 추가되지 않음
→ 기존 협력 관계 재사용
→ 결합도 증가 없음 ✅
```

**결론**: 대안 2 선택!

---

#### HIGH COHESION 패턴 (높은 응집도)

> "여러 설계 중 응집도가 더 높은 것을 선택하라"

**대안 1 분석**:

```
Screening의 책임:
1. 예매 생성 (핵심 책임)
2. 할인 조건 판단 (부가 책임) ❌

문제점:
→ Screening이 할인과 관련된 책임까지 떠안음
→ 할인 정책 변경 시 Screening도 수정
→ 응집도 저하
```

**대안 2 분석**:

```
Screening의 책임:
1. 예매 생성 (핵심 책임만)

Movie의 책임:
1. 요금 계산
2. 할인 조건 판단

장점:
→ 각 객체가 자신의 핵심 책임에만 집중
→ 변경 이유가 명확
→ 응집도 향상 ✅
```

**결론**: 대안 2 선택!

---

#### 패턴 적용의 시점

```
INFORMATION EXPERT 패턴:
→ 첫 번째 책임 할당 시 사용
→ "누가 이 정보를 알고 있지?"

LOW COUPLING & HIGH COHESION 패턴:
→ 여러 대안이 있을 때 사용
→ "어떤 것이 더 나은 설계지?"
```

**실전 팁**:

```
1. INFORMATION EXPERT로 후보 찾기
2. 후보가 여러 개면 LOW COUPLING/HIGH COHESION으로 결정
3. 항상 트레이드오프 고려
```

---

### 🏗️ 4단계: 창조자에게 객체 생성 책임을 할당하라

> **CREATOR 패턴**
>
> "생성되는 객체와 관련이 깊은 객체가 생성 책임을 진다"

#### 상황: Reservation을 누가 만들어야 할까?

```
영화 예매의 최종 결과 = Reservation 인스턴스
→ 누군가는 Reservation을 생성할 책임이 있다
```

**CREATOR 패턴의 원칙**:

```
객체 A를 생성해야 할 때,
다음 조건을 최대한 많이 만족하는 B에게 생성 책임 할당:

□ B가 A를 포함하거나 참조한다
□ B가 A를 기록한다  
□ B가 A를 긴밀하게 사용한다
□ B가 A를 초기화하는 데 필요한 데이터를 가지고 있다
```

#### Reservation 생성 책임 할당

**후보 검토**:

```java
// Reservation의 구성 요소
public class Reservation {
    private Customer customer;     // 고객
    private Screening screening;   // 상영
    private Money fee;            // 요금
    private int audienceCount;    // 인원수
}
```

**Screening 분석**:

```
Screening이 만족하는 조건:
✅ Reservation과 긴밀하게 관련
✅ Reservation 생성에 필요한 대부분의 정보 보유
   - screening 자신
   - fee (계산 가능)
   - customer, audienceCount (파라미터로 받음)

결론: Screening이 CREATOR! ✅
```

```java
public class Screening {
    public Reservation reserve(Customer customer, int audienceCount) {
        Money fee = movie.calculateMovieFee(this);

        // ✅ CREATOR 패턴: Screening이 Reservation 생성
        return new Reservation(customer, this, fee, audienceCount);
    }
}
```

---

#### CREATOR 패턴의 장점

**1. 낮은 결합도 유지**

```
Screening과 Reservation은 어차피 협력해야 함
→ 이미 결합되어 있음
→ 생성 책임을 줘도 새로운 결합도 추가 안 됨

vs. 다른 객체에게 생성 책임을 주면?
→ 불필요한 결합도 증가
```

**2. 응집도 향상**

```
Screening의 책임:
- 예매 진행
- 예매 정보(Reservation) 생성

→ 관련된 책임이 한곳에 모임
→ 응집도 up!
```

**3. 정보 은닉**

```
Reservation 생성에 필요한 정보가
Screening 내부에 캡슐화됨

외부는 reserve() 메시지만 보내면 됨
```

---

## 💻 03. 구현을 통한 검증

지금까지 설계한 내용을 실제 코드로 구현해봅시다.

### 📦 Screening - 예매 책임의 시작점

> 📂 **코드**: [`Screening.java`](https://github.com/eternity-oop/object/blob/master/chapter05/src/main/java/org/eternity/movie/step01/Screening.java)

```java
package org.eternity.movie.step01;

import org.eternity.money.Money;
import java.time.LocalDateTime;

public class Screening {
    private Movie movie;              // 상영할 영화
    private int sequence;             // 상영 순번
    private LocalDateTime whenScreened; // 상영 시작 시간
    
    // ✅ CREATOR 패턴: Reservation 생성 책임
    public Reservation reserve(Customer customer, int audienceCount) {
        return new Reservation(customer, this, 
                              calculateFee(audienceCount), 
                              audienceCount);
    }
    
    // ✅ 요금 계산은 Movie에게 위임
    private Money calculateFee(int audienceCount) {
        // Movie에게 "영화 요금 계산" 메시지 전송
        return movie.calculateMovieFee(this).times(audienceCount);
    }
    
    // ✅ Screening이 협력에 필요한 정보만 제공
    public LocalDateTime getWhenScreened() {
        return whenScreened;
    }
    
    public int getSequence() {
        return sequence;
    }
}
```

**설계 포인트**:

```
1. 메시지 중심
   - calculateMovieFee() 메시지를 Movie에게 전송
   - Movie의 내부 구현은 전혀 모름

2. 캡슐화
   - calculateFee()는 private
   - 외부에는 reserve()만 노출

3. 협력
   - Movie와 협력하여 요금 계산
   - 스스로 할 수 없는 것은 위임
```

---

### 📦 Movie - 요금 계산 책임

> 📂 **코드**: [`Movie.java`](https://github.com/eternity-oop/object/blob/master/chapter05/src/main/java/org/eternity/movie/step01/Movie.java)

```java
package org.eternity.movie.step01;

import org.eternity.money.Money;
import java.time.Duration;
import java.util.List;

public class Movie {
    private String title;
    private Duration runningTime;
    private Money fee;
    private List<DiscountCondition> discountConditions;
    
    // ⚠️ 아직 개선되지 않은 부분 (Step 01)
    private MovieType movieType;
    private Money discountAmount;
    private double discountPercent;
    
    // ✅ INFORMATION EXPERT: 요금 계산의 정보 전문가
    public Money calculateMovieFee(Screening screening) {
        // 할인 가능 여부 확인
        if (isDiscountable(screening)) {
            return fee.minus(calculateDiscountAmount());
        }
        return fee;
    }
    
    // ✅ DiscountCondition과 협력
    private boolean isDiscountable(Screening screening) {
        return discountConditions.stream()
                .anyMatch(condition -> condition.isSatisfiedBy(screening));
    }
    
    // ⚠️ 개선이 필요한 부분 (곧 다룰 예정)
    private Money calculateDiscountAmount() {
        switch(movieType) {
            case AMOUNT_DISCOUNT:
                return calculateAmountDiscountAmount();
            case PERCENT_DISCOUNT:
                return calculatePercentDiscountAmount();
            case NONE_DISCOUNT:
                return calculateNoneDiscountAmount();
        }
        throw new IllegalStateException();
    }
    
    private Money calculateAmountDiscountAmount() {
        return discountAmount;
    }
    
    private Money calculatePercentDiscountAmount() {
        return fee.times(discountPercent);
    }
    
    private Money calculateNoneDiscountAmount() {
        return Money.ZERO;
    }
}
```

---

### 📦 DiscountCondition - 조건 판단 책임

> 📂 **코드**: [`DiscountCondition.java`](https://github.com/eternity-oop/object/blob/master/chapter05/src/main/java/org/eternity/movie/step01/DiscountCondition.java)

```java
package org.eternity.movie.step01;

import java.time.DayOfWeek;
import java.time.LocalTime;

public class DiscountCondition {
    private DiscountConditionType type;
    
    // 순번 조건용
    private int sequence;
    
    // 기간 조건용
    private DayOfWeek dayOfWeek;
    private LocalTime startTime;
    private LocalTime endTime;
    
    // ✅ INFORMATION EXPERT: 조건 판단의 정보 전문가
    public boolean isSatisfiedBy(Screening screening) {
        if (type == DiscountConditionType.PERIOD) {
            return isSatisfiedByPeriod(screening);
        }
        return isSatisfiedBySequence(screening);
    }
    
    private boolean isSatisfiedByPeriod(Screening screening) {
        return dayOfWeek.equals(screening.getWhenScreened().getDayOfWeek()) &&
               startTime.compareTo(screening.getWhenScreened().toLocalTime()) <= 0 &&
               endTime.compareTo(screening.getWhenScreened().toLocalTime()) >= 0;
    }
    
    private boolean isSatisfiedBySequence(Screening screening) {
        return sequence == screening.getSequence();
    }
}
```

---

### 🚨 Step 01의 문제점

코드를 구현했지만, 여전히 문제가 있습니다:

```
1. DiscountCondition의 낮은 응집도
   - 순번 조건과 기간 조건이 하나의 클래스에 혼재
   - 변경 이유가 2개 이상
   
2. Movie의 낮은 응집도
   - 금액/비율/없음 할인이 하나의 클래스에 혼재
   - MovieType으로 타입 체크
   
3. 캡슐화 위반
   - getWhenScreened(), getSequence() 등 내부 노출
```

**이 문제들을 어떻게 해결할까?**

→ 다음 섹션에서 다룹니다!

---

## 🔧 04. DiscountCondition 개선하기

### 🔍 문제 진단: 변경에 취약한 클래스

DiscountCondition은 **3가지 이유**로 변경될 수 있습니다:

```
변경 시나리오 1: 새로운 할인 조건 추가
→ isSatisfiedBy()의 if-else 수정
→ 새로운 데이터 필요하면 속성 추가

변경 시나리오 2: 순번 조건 판단 로직 변경
→ isSatisfiedBySequence() 수정
→ sequence 속성 변경 가능

변경 시나리오 3: 기간 조건 판단 로직 변경
→ isSatisfiedByPeriod() 수정
→ dayOfWeek, startTime, endTime 속성 변경 가능
```

**결론**: 응집도가 낮다! → 클래스 분리 필요

---

### 📊 응집도가 낮다는 증거 찾기

#### 방법 1: 인스턴스 변수 초기화 시점 관찰

```java
// 순번 조건 생성
DiscountCondition sequenceCondition = new DiscountCondition();
sequenceCondition.setType(DiscountConditionType.SEQUENCE);
sequenceCondition.setSequence(1);
// ❌ dayOfWeek, startTime, endTime은 초기화 안 됨!

// 기간 조건 생성
DiscountCondition periodCondition = new DiscountCondition();
periodCondition.setType(DiscountConditionType.PERIOD);
periodCondition.setDayOfWeek(DayOfWeek.MONDAY);
periodCondition.setStartTime(LocalTime.of(10, 0));
periodCondition.setEndTime(LocalTime.of(12, 0));
// ❌ sequence는 초기화 안 됨!
```

**문제**:
```
응집도 높은 클래스 = 모든 속성을 함께 초기화
응집도 낮은 클래스 = 일부 속성만 초기화

→ DiscountCondition은 응집도가 낮다!
```

#### 방법 2: 메서드가 사용하는 속성 그룹 관찰

```java
public class DiscountCondition {
    // 그룹 1: 순번 조건 속성
    private int sequence;
    
    // 그룹 2: 기간 조건 속성
    private DayOfWeek dayOfWeek;
    private LocalTime startTime;
    private LocalTime endTime;
    
    // 그룹 1만 사용하는 메서드
    private boolean isSatisfiedBySequence(Screening screening) {
        return sequence == screening.getSequence();
    }
    
    // 그룹 2만 사용하는 메서드
    private boolean isSatisfiedByPeriod(Screening screening) {
        return dayOfWeek.equals(...) &&
               startTime.compareTo(...) <= 0 &&
               endTime.compareTo(...) >= 0;
    }
}
```

**문제**:
```
메서드들이 사용하는 속성에 따라 그룹이 나뉨
→ 서로 다른 관심사가 한 클래스에 혼재
→ 응집도가 낮다!
```

---

### 📐 클래스 응집도 판단 체크리스트

```
다음 중 하나라도 해당하면 응집도가 낮은 것:

□ 클래스가 하나 이상의 이유로 변경돼야 한다
□ 인스턴스 초기화 시점에 경우에 따라 다른 속성 초기화
□ 메서드 그룹이 사용하는 속성 그룹으로 나뉜다
```

---

### 🔀 해결 방법 1: 타입 분리하기

가장 직관적인 해결책: 두 타입을 별도 클래스로 분리

#### PeriodCondition

> 📂 **코드**: [`PeriodCondition.java`](https://github.com/eternity-oop/object/blob/master/chapter05/src/main/java/org/eternity/movie/step02/PeriodCondition.java)

```java
public class PeriodCondition {
    private DayOfWeek dayOfWeek;
    private LocalTime startTime;
    private LocalTime endTime;
    
    public PeriodCondition(DayOfWeek dayOfWeek, 
                          LocalTime startTime, 
                          LocalTime endTime) {
        this.dayOfWeek = dayOfWeek;
        this.startTime = startTime;
        this.endTime = endTime;
    }
    
    public boolean isSatisfiedBy(Screening screening) {
        return dayOfWeek.equals(screening.getWhenScreened().getDayOfWeek()) &&
               startTime.compareTo(screening.getWhenScreened().toLocalTime()) <= 0 &&
               endTime.compareTo(screening.getWhenScreened().toLocalTime()) >= 0;
    }
}
```

#### SequenceCondition

> 📂 **코드**: [`SequenceCondition.java`](https://github.com/eternity-oop/object/blob/master/chapter05/src/main/java/org/eternity/movie/step02/SequenceCondition.java)

```java
public class SequenceCondition {
    private int sequence;
    
    public SequenceCondition(int sequence) {
        this.sequence = sequence;
    }
    
    public boolean isSatisfiedBy(Screening screening) {
        return sequence == screening.getSequence();
    }
}
```

**개선 효과**:

```
✅ 각 클래스의 응집도 향상
✅ 변경 이유가 하나로 명확
✅ 인스턴스 변수 초기화 시점 일치
✅ 모든 메서드가 모든 속성 사용
```

---

### 🚨 하지만 새로운 문제 발생

Movie는 두 클래스 모두와 협력해야 합니다:

```java
public class Movie {
    // ❌ 두 개의 리스트를 따로 관리?
    private List<PeriodCondition> periodConditions;
    private List<SequenceCondition> sequenceConditions;
    
    private boolean isDiscountable(Screening screening) {
        // ❌ 두 리스트를 각각 확인?
        return checkPeriodConditions(screening) ||
               checkSequenceConditions(screening);
    }
    
    private boolean checkPeriodConditions(Screening screening) {
        return periodConditions.stream()
                .anyMatch(condition -> condition.isSatisfiedBy(screening));
    }
    
    private boolean checkSequenceConditions(Screening screening) {
        return sequenceConditions.stream()
                .anyMatch(condition -> condition.isSatisfiedBy(screening));
    }
}
```

**새로운 문제점**:

```
1. 결합도 증가
   - Movie가 두 클래스 모두에 의존
   - 변경 시 Movie도 함께 수정

2. 새로운 할인 조건 추가 어려움
   - 새 클래스 추가
   - Movie에 새 리스트 추가
   - Movie에 새 체크 메서드 추가
   
3. 캡슐화 위반
   - 할인 조건 종류가 Movie에 노출
```

**해결책**: 다형성을 통한 역할 추상화!

---

### 🎭 해결 방법 2: 다형성을 통해 분리하기

> **POLYMORPHISM 패턴**
>
> "객체의 타입에 따라 변하는 행동이 있다면
>  타입을 분리하고 각 타입에 책임을 할당하라"

#### 역할(Role) 도입

Movie의 관점에서 보면:
```
PeriodCondition과 SequenceCondition은
같은 책임을 수행한다 = 같은 역할!

"할인 조건을 만족하는가?" 라는 동일한 메시지에 응답
```

**역할로 추상화**:

> 📂 **코드**: [`DiscountCondition.java`](https://github.com/eternity-oop/object/blob/master/chapter05/src/main/java/org/eternity/movie/step03/DiscountCondition.java), [`PeriodCondition.java`](https://github.com/eternity-oop/object/blob/master/chapter05/src/main/java/org/eternity/movie/step03/PeriodCondition.java), [`SequenceCondition.java`](https://github.com/eternity-oop/object/blob/master/chapter05/src/main/java/org/eternity/movie/step03/SequenceCondition.java)

```java
// ✅ 역할 정의
public interface DiscountCondition {
    boolean isSatisfiedBy(Screening screening);
}

// ✅ 역할 구현 1
public class PeriodCondition implements DiscountCondition {
    private DayOfWeek dayOfWeek;
    private LocalTime startTime;
    private LocalTime endTime;
    
    @Override
    public boolean isSatisfiedBy(Screening screening) {
        return dayOfWeek.equals(screening.getWhenScreened().getDayOfWeek()) &&
               startTime.compareTo(screening.getWhenScreened().toLocalTime()) <= 0&&
               endTime.compareTo(screening.getWhenScreened().toLocalTime()) >= 0;
    }
}

// ✅ 역할 구현 2
public class SequenceCondition implements DiscountCondition {
    private int sequence;
    
    @Override
    public boolean isSatisfiedBy(Screening screening) {
        return sequence == screening.getSequence();
    }
}
```

---

#### Movie는 역할에만 의존

```java
public class Movie {
    // ✅ 하나의 리스트로 통합!
    private List<DiscountCondition> discountConditions;
    
    public Money calculateMovieFee(Screening screening) {
        if (isDiscountable(screening)) {
            return fee.minus(calculateDiscountAmount());
        }
        return fee;
    }
    
    // ✅ 단순하고 명확!
    private boolean isDiscountable(Screening screening) {
        return discountConditions.stream()
                .anyMatch(condition -> condition.isSatisfiedBy(screening));
    }
}
```

**개선 효과**:

```
✅ Movie는 DiscountCondition 역할에만 의존
✅ 구체적인 타입(Period/Sequence)은 몰라도 됨
✅ 새로운 할인 조건 추가 시 Movie 수정 불필요
✅ 각 조건 클래스의 내부 구현 자유롭게 변경 가능
```

---

#### POLYMORPHISM 패턴의 핵심

```
조건문 사용 (나쁜 설계):
if (condition.getType() == PERIOD) {
    // 기간 조건 처리
} else if (condition.getType() == SEQUENCE) {
    // 순번 조건 처리
}
→ 새로운 타입 추가 시 모든 조건문 수정

다형성 사용 (좋은 설계):
condition.isSatisfiedBy(screening)
→ 새로운 타입 추가 시 새 클래스만 추가
```

**POLYMORPHISM 패턴 정의**:

```
When: 객체의 타입에 따라 변하는 행동이 있을 때
How: 타입을 명시적으로 정의하고
     각 타입에 다형적으로 행동하는 책임 할당
Don't: 타입을 검사해서 조건문으로 분기 처리
```

---

### 🛡️ PROTECTED VARIATIONS 패턴

> **변경 보호 패턴**
>
> "변화가 예상되는 지점을 식별하고
>  안정적인 인터페이스 뒤로 캡슐화하라"

```
변화하는 부분:
- Period할인 조건
- Sequence할인 조건
- 앞으로 추가될 새로운 할인 조건

안정적인 부분:
- DiscountCondition 인터페이스
- isSatisfiedBy() 메시지

결과:
Movie ──→ [DiscountCondition 인터페이스]
             ↑              ↑
             │              │
      PeriodCondition  SequenceCondition
      
→ 변경이 인터페이스 뒤로 캡슐화됨
→ Movie는 변경으로부터 보호됨
```

**새로운 할인 조건 추가 시**:

```java
// ✅ 새 클래스만 추가
public class CombinedCondition implements DiscountCondition {
    private PeriodCondition periodCondition;
    private SequenceCondition sequenceCondition;
    
    @Override
    public boolean isSatisfiedBy(Screening screening) {
        return periodCondition.isSatisfiedBy(screening) &&
               sequenceCondition.isSatisfiedBy(screening);
    }
}

// ✅ Movie는 수정 불필요!
Movie movie = new Movie(
    "아바타",
    Duration.ofMinutes(120),
    Money.wons(10000),
    new CombinedCondition(...)  // 그냥 추가
);
```

---

## 🎬 05. Movie 클래스 개선하기

Movie도 DiscountCondition과 동일한 문제를 가지고 있습니다.

### 🔍 문제점 분석

```java
public class Movie {
    // 세 가지 할인 정책 타입이 하나의 클래스에 혼재
    private MovieType movieType;      // 타입 구분
    private Money discountAmount;     // 금액 할인용
    private double discountPercent;   // 비율 할인용
    
    private Money calculateDiscountAmount() {
        switch(movieType) {
            case AMOUNT_DISCOUNT:
                return calculateAmountDiscountAmount();
            case PERCENT_DISCOUNT:
                return calculatePercentDiscountAmount();
            case NONE_DISCOUNT:
                return calculateNoneDiscountAmount();
        }
        throw new IllegalStateException();
    }
}
```

**변경 이유**:

```
1. 금액 할인 정책 계산 방식 변경
2. 비율 할인 정책 계산 방식 변경
3. 할인 없음 정책 변경
4. 새로운 할인 정책 추가

→ 4가지 변경 이유 = 낮은 응집도
```

---

### 🎭 POLYMORPHISM 패턴 적용

**1단계: 추상 클래스로 역할 정의**

> 📂 **코드**: [`Movie.java`](https://github.com/eternity-oop/object/blob/master/chapter05/src/main/java/org/eternity/movie/step04/Movie.java)

```java
public abstract class Movie {
    private String title;
    private Duration runningTime;
    private Money fee;
    private List<DiscountCondition> discountConditions;
    
    public Movie(String title, Duration runningTime, Money fee,
                 DiscountCondition... discountConditions) {
        this.title = title;
        this.runningTime = runningTime;
        this.fee = fee;
        this.discountConditions = Arrays.asList(discountConditions);
    }
    
    // ✅ 공통 로직: 템플릿 메서드
    public Money calculateMovieFee(Screening screening) {
        if (isDiscountable(screening)) {
            return fee.minus(calculateDiscountAmount());
        }
        return fee;
    }
    
    private boolean isDiscountable(Screening screening) {
        return discountConditions.stream()
                .anyMatch(condition -> condition.isSatisfiedBy(screening));
    }
    
    protected Money getFee() {
        return fee;
    }
    
    // ✅ 변하는 부분: 자식 클래스가 구현
    abstract protected Money calculateDiscountAmount();
}
```

**2단계: 구체적인 할인 정책 클래스들**

> 📂 **코드**: [`AmountDiscountMovie.java`](https://github.com/eternity-oop/object/blob/master/chapter05/src/main/java/org/eternity/movie/step04/AmountDiscountMovie.java), [`PercentDiscountMovie.java`](https://github.com/eternity-oop/object/blob/master/chapter05/src/main/java/org/eternity/movie/step04/PercentDiscountMovie.java), [`NoneDiscountMovie.java`](https://github.com/eternity-oop/object/blob/master/chapter05/src/main/java/org/eternity/movie/step04/NoneDiscountMovie.java)

```java
// 금액 할인
public class AmountDiscountMovie extends Movie {
    private Money discountAmount;
    
    public AmountDiscountMovie(String title, Duration runningTime,
                              Money fee, Money discountAmount,
                              DiscountCondition... discountConditions) {
        super(title, runningTime, fee, discountConditions);
        this.discountAmount = discountAmount;
    }
    
    @Override
    protected Money calculateDiscountAmount() {
        return discountAmount;
    }
}

// 비율 할인
public class PercentDiscountMovie extends Movie {
    private double percent;
    
    public PercentDiscountMovie(String title, Duration runningTime,
                               Money fee, double percent,
                               DiscountCondition... discountConditions) {
        super(title, runningTime, fee, discountConditions);
        this.percent = percent;
    }
    
    @Override
    protected Money calculateDiscountAmount() {
        return getFee().times(percent);
    }
}

// 할인 없음
public class NoneDiscountMovie extends Movie {
    public NoneDiscountMovie(String title, Duration runningTime, Money fee) {
        super(title, runningTime, fee);
    }
    
    @Override
    protected Money calculateDiscountAmount() {
        return Money.ZERO;
    }
}
```

---

### 🎯 최종 설계 구조

```
                ┌──────────────┐
                │    Movie     │
                │ (abstract)   │
                └──────────────┘
                       △
                       │
      ┌────────────────┼────────────────┐
      │                │                │
┌────────────┐  ┌─────────────┐  ┌────────────┐
│  Amount    │  │  Percent    │  │   None     │
│ Discount   │  │  Discount   │  │  Discount  │
│   Movie    │  │    Movie    │  │   Movie    │
└────────────┘  └─────────────┘  └────────────┘
```

---

### 📊 개선 전후 비교

**Before**:

```java
// 하나의 클래스에 모든 정책
Movie movie = new Movie(...);
movie.setMovieType(MovieType.AMOUNT_DISCOUNT);
movie.setDiscountAmount(Money.wons(800));

// 새로운 정책 추가 시:
// 1. MovieType에 enum 추가
// 2. Movie에 필드 추가
// 3. calculateDiscountAmount()에 case 추가
```

**After**:

```java
// 정책별로 별도 클래스
Movie avatar = new AmountDiscountMovie(
    "아바타",
    Duration.ofMinutes(120),
    Money.wons(10000),
    Money.wons(800),
    ...
);

// 새로운 정책 추가 시:
// 1. Movie를 상속하는 새 클래스만 추가
// 2. calculateDiscountAmount() 구현
// → 기존 코드 수정 불필요!
```

---

### 🔄 도메인의 구조가 코드의 구조를 이끈다

**도메인 모델**:

```
Movie
  ├─ AmountDiscountMovie (금액 할인 영화)
  ├─ PercentDiscountMovie (비율 할인 영화)
  └─ NoneDiscountMovie (할인 없는 영화)
```

**코드 구조**:

```java
abstract class Movie
class AmountDiscountMovie extends Movie
class PercentDiscountMovie extends Movie
class NoneDiscountMovie extends Movie
```

**→ 도메인 구조가 코드 구조에 반영됨!**

---

## 🔄 06. 변경과 유연성

### 설계 vs 변경

개발자가 변경에 대비하는 두 가지 방법:

```
1. 코드를 이해하고 수정하기 쉽게 단순하게 설계
   → 대부분의 경우 이것이 정답

2. 코드를 수정하지 않고 변경을 수용하도록 유연하게 설계
   → 변경이 반복적으로 발생할 때
```

### 현재 설계의 한계

**문제 상황**:
```
"영화 개봉 후 반응을 보고 할인 정책을 변경하고 싶어요"
"런타임에 정책을 바꿀 수 있나요?"
```

**현재 설계**:
```java
// ❌ 상속 기반 설계의 한계
AmountDiscountMovie avatar = new AmountDiscountMovie(...);

// 정책을 변경하려면?
// → 객체를 새로 만들어야 함
PercentDiscountMovie newAvatar = new PercentDiscountMovie(...);

// 문제: 기존 avatar 객체와의 연결이 끊김
```

---

### 💡 합성으로 전환

**새로운 설계**:

```java
// Movie가 정책을 "포함"
public class Movie {
    private String title;
    private Duration runningTime;
    private Money fee;
    private DiscountPolicy discountPolicy;  // ← 합성!
    
    public Money calculateMovieFee(Screening screening) {
        return fee.minus(discountPolicy.calculateDiscountAmount(screening));
    }
    
    // ✅ 런타임에 정책 변경 가능!
    public void changeDiscountPolicy(DiscountPolicy discountPolicy) {
        this.discountPolicy = discountPolicy;
    }
}

// 할인 정책 계층
public abstract class DiscountPolicy {
    abstract Money calculateDiscountAmount(Screening screening);
}

public class AmountDiscountPolicy extends DiscountPolicy { ... }
public class PercentDiscountPolicy extends DiscountPolicy { ... }
public class NoneDiscountPolicy extends DiscountPolicy { ... }
```

**사용**:

```java
// 생성 시 정책 설정
Movie avatar = new Movie(
    "아바타",
    Duration.ofMinutes(120),
    Money.wons(10000),
    new AmountDiscountPolicy(...)  // 처음엔 금액 할인
);

// 런타임에 정책 변경!
avatar.changeDiscountPolicy(
    new PercentDiscountPolicy(...)  // 비율 할인으로 변경
);
```

---

### 상속 vs 합성

| 측면 | 상속 | 합성 |
|------|------|------|
| **결합 시점** | 컴파일 타임 (정적) | 런타임 (동적) |
| **유연성** | 낮음 (변경 불가) | 높음 (변경 가능) |
| **의존성** | 부모 클래스에 강하게 결합 | 인터페이스에 느슨하게 결합 |
| **캡슐화** | 부모 내부 노출 | 인터페이스만 노출 |
| **사용 시기** | 타입 계층이 명확할 때 | 유연한 변경이 필요할 때 |

**결론**:
```
유사한 변경이 반복적으로 발생한다면
→ 복잡성이 증가하더라도
→ 유연성을 추가하는 것이 좋다
→ 합성 사용!
```

---

## 🔨 07. 책임 주도 설계의 대안

### 현실의 어려움

```
책임 주도 설계는 훌륭하지만...
❌ 처음부터 완벽하게 하기 어렵다
❌ 객체와 책임 사이에서 방황하게 됨
❌ 막막할 때가 많다
```

### 💡 실용적 대안: 리팩터링

```
1. 일단 빠르게 작동하는 코드 작성
2. 코드를 보면서 책임 식별
3. 올바른 위치로 책임 이동
4. 겉으로 보이는 동작은 유지
5. 내부 구조만 개선

= 리팩터링(Refactoring)
```

**Martin Fowler**:
> "처음부터 완벽한 설계를 갖추는 것보다,
>  빠르게 실행 가능한 코드를 만들고
>  지속적으로 리팩터링하는 것이 낫다"

---

### 📝 메서드 응집도: 몬스터 메서드 해체하기

#### Before: 거대한 메서드

> 📂 **코드**: [`ReservationAgency.java`](https://github.com/eternity-oop/object/blob/master/chapter05/src/main/java/org/eternity/movie/step05/ReservationAgency.java) (리팩터링 전)

```java
public class ReservationAgency {
    public Reservation reserve(Screening screening, Customer customer,
                               int audienceCount) {
        Movie movie = screening.getMovie();
        
        // ❌ 45줄의 거대한 메서드
        boolean discountable = false;
        for(DiscountCondition condition : movie.getDiscountConditions()) {
            if (condition.getType() == DiscountConditionType.PERIOD) {
                discountable = screening.getWhenScreened().getDayOfWeek()
                                       .equals(condition.getDayOfWeek()) &&
                              condition.getStartTime()
                                       .compareTo(screening.getWhenScreened()
                                                          .toLocalTime()) <= 0 &&
                              condition.getEndTime()
                                       .compareTo(screening.getWhenScreened()
                                                          .toLocalTime()) >= 0;
            } else {
                discountable = condition.getSequence() == screening.getSequence();
            }
            if (discountable) {
                break;
            }
        }
        
        Money fee;
        if (discountable) {
            Money discountAmount = Money.ZERO;
            switch(movie.getMovieType()) {
                case AMOUNT_DISCOUNT:
                    discountAmount = movie.getDiscountAmount();
                    break;
                case PERCENT_DISCOUNT:
                    discountAmount = movie.getFee().times(movie.getDiscountPercent());
                    break;
                case NONE_DISCOUNT:
                    discountAmount = Money.ZERO;
                    break;
            }
            fee = movie.getFee().minus(discountAmount).times(audienceCount);
        } else {
            fee = movie.getFee().times(audienceCount);
        }
        
        return new Reservation(customer, screening, fee, audienceCount);
    }
}
```

**몬스터 메서드의 문제점**:

```
1. 한눈에 파악 불가능
   → 코드 이해에 시간 소요

2. 수정 포인트 찾기 어려움
   → 어디를 고쳐야 할지 막막

3. 부분 수정이 전체에 영향
   → 버그 발생 위험

4. 재사용 불가능
   → 복사-붙여넣기 중복 발생

5. 낮은 응집도
   → 여러 이유로 변경됨
```

---

#### After: 응집도 높은 메서드들

```java
public class ReservationAgency {
    // ✅ 의도가 명확한 public 메서드
    public Reservation reserve(Screening screening, Customer customer,
                               int audienceCount) {
        boolean discountable = checkDiscountable(screening);
        Money fee = calculateFee(screening, discountable, audienceCount);
        return createReservation(screening, customer, audienceCount, fee);
    }
    
    // ✅ 할인 가능 여부 확인 - 단일 책임
    private boolean checkDiscountable(Screening screening) {
        return screening.getMovie().getDiscountConditions().stream()
                .anyMatch(condition -> condition.isDiscountable(screening));
    }
    
    // ✅ 요금 계산 - 단일 책임
    private Money calculateFee(Screening screening, boolean discountable,
                               int audienceCount) {
        if (discountable) {
            return screening.getMovie().getFee()
                    .minus(calculateDiscountedFee(screening.getMovie()))
                    .times(audienceCount);
        }
        return screening.getMovie().getFee().times(audienceCount);
    }
    
    // ✅ 할인 금액 계산 - 단일 책임
    private Money calculateDiscountedFee(Movie movie) {
        switch(movie.getMovieType()) {
            case AMOUNT_DISCOUNT:
                return calculateAmountDiscountedFee(movie);
            case PERCENT_DISCOUNT:
                return calculatePercentDiscountedFee(movie);
            case NONE_DISCOUNT:
                return calculateNoneDiscountedFee(movie);
        }
        throw new IllegalArgumentException();
    }
    
    // 각 할인 타입별 계산 메서드들...
}
```

**개선 효과**:

```
✅ 주석이 필요 없을 정도로 명확
✅ 각 메서드는 하나의 일만 수행
✅ 변경 지점이 명확
✅ 재사용 가능
✅ 테스트하기 쉬움
```

---

### 🎯 객체를 자율적으로 만들자

메서드를 작게 쪼갰지만, 여전히 문제가 있습니다:

```
ReservationAgency가 여전히 모든 데이터에 접근
→ 응집도는 여전히 낮음
```

**해결책**: 메서드를 적절한 객체로 이동!

#### 1단계: DiscountCondition으로 이동

```java
// Before: ReservationAgency가 판단
private boolean checkDiscountable(Screening screening) {
    for(DiscountCondition condition : movie.getDiscountConditions()) {
        if (condition.getType() == DiscountConditionType.PERIOD) {
            // DiscountCondition의 데이터로 직접 판단
            discountable = screening.getWhenScreened().getDayOfWeek()
                                   .equals(condition.getDayOfWeek()) && ...
        }
    }
}

// After: DiscountCondition 스스로 판단
public class DiscountCondition {
    public boolean isDiscountable(Screening screening) {
        if (type == DiscountConditionType.PERIOD) {
            return isSatisfiedByPeriod(screening);
        }
        return isSatisfiedBySequence(screening);
    }
    
    private boolean isSatisfiedByPeriod(Screening screening) {
        return screening.getWhenScreened().getDayOfWeek().equals(dayOfWeek) &&
               startTime.compareTo(screening.getWhenScreened().toLocalTime()) <= 0 &&
               endTime.compareTo(screening.getWhenScreened().toLocalTime()) >= 0;
    }
}
```

#### 2단계: Movie로 이동

```java
// Before: ReservationAgency가 계산
private Money calculateDiscountedFee(Movie movie) {
    switch(movie.getMovieType()) {
        case AMOUNT_DISCOUNT:
            return movie.getDiscountAmount();
        case PERCENT_DISCOUNT:
            return movie.getFee().times(movie.getDiscountPercent());
    }
}

// After: Movie 스스로 계산
public class Movie {
    public Money calculateMovieFee(Screening screening) {
        if (isDiscountable(screening)) {
            return fee.minus(calculateDiscountAmount());
        }
        return fee;
    }
}
```

#### 3단계: Screening으로 이동

```java
// Before: ReservationAgency가 생성
public Reservation reserve(Screening screening, Customer customer,
                           int audienceCount) {
    Money fee = calculateFee(...);
    return new Reservation(customer, screening, fee, audienceCount);
}

// After: Screening이 스스로 생성
public class Screening {
    public Reservation reserve(Customer customer, int audienceCount) {
        Money fee = movie.calculateMovieFee(this).times(audienceCount);
        return new Reservation(customer, this, fee, audienceCount);
    }
}
```

---

### 🎯 리팩터링의 결과

**Before**:
```
ReservationAgency: 모든 로직 (100%)
Movie: 데이터만 (0%)
DiscountCondition: 데이터만 (0%)
Screening: 데이터만 (0%)
```

**After**:
```
ReservationAgency: 조율 (0%)
Screening: 예매 생성 (30%)
Movie: 요금 계산 (40%)
DiscountCondition: 조건 판단 (30%)
```

**→ 책임 주도 설계와 동일한 결과!**

---

## 🎓 핵심 정리

### GRASP 패턴 요약

| 패턴 | 질문 | 답변 |
|------|------|------|
| **INFORMATION EXPERT** | 누구에게 책임을 줄까? | 정보를 아는 자에게 |
| **CREATOR** | 누가 생성할까? | 관련 깊은 자가 생성 |
| **LOW COUPLING** | 여러 대안 중 어느 것? | 결합도 낮은 것 |
| **HIGH COHESION** | 여러 대안 중 어느 것? | 응집도 높은 것 |
| **POLYMORPHISM** | 타입별 행동이 다르면? | 다형성으로 해결 |
| **PROTECTED VARIATIONS** | 변경을 어떻게 다룰까? | 인터페이스로 캡슐화 |

---

### 책임 할당의 핵심 원칙

```
1. 데이터가 아닌 행동(책임)을 먼저 결정하라
2. 협력이라는 문맥 안에서 책임을 결정하라
3. 메시지가 객체를 선택하게 하라
4. 정보를 아는 객체에게 책임을 할당하라
5. 결합도와 응집도를 항상 고려하라
6. 다형성으로 타입을 캡슐화하라
7. 변경을 인터페이스 뒤로 숨겨라
```

---

### 설계 프로세스

```
완벽한 설계를 처음부터 만들 필요 없다!

1. 도메인 대략 그리기 (빠르게)
2. 시스템 기능 파악
3. 메시지 정의
4. GRASP 패턴으로 책임 할당
5. 구현하면서 계속 개선

또는:

1. 일단 작동하는 코드 작성
2. 몬스터 메서드 분해
3. 메서드를 적절한 객체로 이동
4. 다형성으로 조건문 제거
5. 지속적인 리팩터링
```

---

### 마지막 조언

```
📖 "완벽한 설계는 없다. 더 나은 설계만 있을 뿐"

🎯 설계는 트레이드오프
   - 여러 설계 대안이 존재
   - 문맥에 따라 최선이 달라짐
   - GRASP 패턴은 선택의 기준 제공

🔄 설계는 진화한다
   - 처음부터 완벽할 수 없음
   - 구현하면서 발견하고 개선
   - 리팩터링은 선택이 아닌 필수

💪 연습이 중요하다
   - GRASP 패턴을 체화하려면 반복 연습
   - 다양한 도메인에 적용해보기
   - 설계 의사결정을 설명하는 연습
```

---

## 🔬 실행 흐름 상세 추적

코드가 실제로 어떻게 실행되는지 완벽하게 추적해봅시다.

### 시나리오: 아바타 영화 예매 (2명, 1회차, 월요일 10:30)

#### 준비 단계

```java
// 1. 할인 조건 생성
DiscountCondition sequenceCondition = new SequenceCondition(1);  // 1회차
DiscountCondition periodCondition = new PeriodCondition(
    DayOfWeek.MONDAY,
    LocalTime.of(10, 0),
    LocalTime.of(12, 0)
);

// 2. 영화 생성 (금액 할인)
Movie avatar = new AmountDiscountMovie(
    "아바타",
    Duration.ofMinutes(120),
    Money.wons(10000),      // 기본 요금 10,000원
    Money.wons(800),        // 800원 할인
    sequenceCondition,
    periodCondition
);

// 3. 상영 생성
Screening screening = new Screening(
    avatar,
    1,                      // 1회차
    LocalDateTime.of(2024, 12, 16, 10, 30)  // 월요일 10:30
);

// 4. 고객
Customer customer = new Customer("홍길동", "user123");

// 5. 예매 시작!
Reservation reservation = screening.reserve(customer, 2);
```

---

#### 실행 추적: 메시지의 여행

```
┌─────────────────────────────────────────────────────────────┐
│ 1. screening.reserve(customer, 2) 호출                       │
└─────────────────────────────────────────────────────────────┘
                        ↓
    ┌────────────────────────────────────────────────┐
    │ [Screening.reserve 메서드 실행]                   │
    │                                                │
    │ public Reservation reserve(Customer customer,  │
    │                           int audienceCount) { │
    │     return new Reservation(                    │
    │         customer,                              │
    │         this,                                  │
    │         calculateFee(audienceCount),  ← 호출    │
    │         audienceCount                          │
    │     );                                         │
    │ }                                              │
    └────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────────┐
│ 2. calculateFee(2) 호출                                      │
└─────────────────────────────────────────────────────────────┘
                        ↓
    ┌────────────────────────────────────────────────┐
    │ [Screening.calculateFee 메서드 실행]              │
    │                                                │
    │ private Money calculateFee(int audienceCount) {│
    │     return movie.calculateMovieFee(this)  ← 호출│
    │                 .times(audienceCount);         │
    │ }                                              │
    └────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────────┐
│ 3. movie.calculateMovieFee(screening) 호출                   │
│    (movie = AmountDiscountMovie 인스턴스)                     │
└─────────────────────────────────────────────────────────────┘
                        ↓
    ┌────────────────────────────────────────────────┐
    │ [Movie.calculateMovieFee 메서드 실행]             │
    │                                                │
    │ public Money calculateMovieFee(                │
    │         Screening screening) {                 │
    │     if (isDiscountable(screening)) {  ← 호출    │
    │         return fee.minus(                      │
    │             calculateDiscountAmount());        │
    │     }                                          │
    │     return fee;                                │
    │ }                                              │
    └────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────────┐
│ 4. isDiscountable(screening) 호출                            │
└─────────────────────────────────────────────────────────────┘
                        ↓
    ┌────────────────────────────────────────────────┐
    │ [Movie.isDiscountable 메서드 실행]                │
    │                                                │
    │ private boolean isDiscountable(                │
    │         Screening screening) {                 │
    │     return discountConditions.stream()         │
    │         .anyMatch(condition ->                 │
    │            condition.isSatisfiedBy(screening));│
    │ }                                              │
    │                                                │
    │ → discountConditions 순회 시작                   │
    └────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────────┐
│ 5. 첫 번째 조건: sequenceCondition.isSatisfiedBy(screening)    │
└─────────────────────────────────────────────────────────────┘
                        ↓
    ┌────────────────────────────────────────────────┐
    │ [SequenceCondition.isSatisfiedBy 실행]          │
    │                                                │
    │ public boolean isSatisfiedBy(                  │
    │         Screening screening) {                 │
    │     return sequence == screening.getSequence();│
    │         //  1     ==    1                      │
    │         //  true! ✅                           │
    │ }                                              │
    └────────────────────────────────────────────────┘
                        ↓
                     true 반환
                        ↓
    ┌────────────────────────────────────────────────┐
    │ anyMatch가 true를 만나면 즉시 종료                  │
    │ → periodCondition은 체크하지 않음                  │
    └────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────────┐
│ 6. isDiscountable()이 true 반환                               │
│    → 할인 가능!                                               │
└─────────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────────┐
│ 7. calculateDiscountAmount() 호출                            │
│    (AmountDiscountMovie의 메서드)                             │
└─────────────────────────────────────────────────────────────┘
                        ↓
    ┌────────────────────────────────────────────────┐
    │ [AmountDiscountMovie.calculateDiscountAmount]  │
    │                                                │
    │ protected Money calculateDiscountAmount() {    │
    │     return discountAmount;                     │
    │         // Money.wons(800) 반환                 │
    │ }                                              │
    └────────────────────────────────────────────────┘
                        ↓
                Money.wons(800) 반환
                        ↓
┌─────────────────────────────────────────────────────────────┐
│ 8. calculateMovieFee에서 계속                                 │
│    fee.minus(discountAmount)                                │
│    = Money.wons(10000).minus(Money.wons(800))               │
│    = Money.wons(9200)                                       │
└─────────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────────┐
│ 9. calculateFee에서 계속                                      │
│    Money.wons(9200).times(2)                                │
│    = Money.wons(18400)                                      │
└─────────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────────┐
│ 10. reserve에서 Reservation 생성                              │
│     new Reservation(customer, this, Money.wons(18400), 2)   │
└─────────────────────────────────────────────────────────────┘
                        ↓
                      완료! 🎉
```

---

### 최종 결과

```
예매 정보:
- 고객: 홍길동
- 영화: 아바타
- 상영: 1회차, 월요일 10:30
- 인원: 2명
- 1인당 요금: 9,200원 (10,000원 - 800원)
- 총 요금: 18,400원
```

---

### 핵심 포인트

**1. 메시지 체인**

```
Screening → Movie → DiscountCondition
   ↓          ↓           ↓
 reserve   calculate   isSatisfied
```

**2. 다형성의 힘**

```
Movie.calculateMovieFee()는:
- AmountDiscountMovie인 경우 → calculateDiscountAmount()
- PercentDiscountMovie인 경우 → calculateDiscountAmount()
- NoneDiscountMovie인 경우 → calculateDiscountAmount()

같은 메시지, 다른 구현!
```

**3. 조기 종료 (Short Circuit)**

```
anyMatch()는 첫 번째 true를 만나면 즉시 종료
→ sequenceCondition이 true → periodCondition 체크 안 함
→ 효율적!
```

---

## 🚀 실전 적용: Spring Boot 예제

### Before: 데이터 중심 설계

```java
// ❌ Service에 모든 로직
@Service
public class OrderService {
    @Autowired
    private OrderRepository orderRepository;
    
    @Autowired
    private ProductRepository productRepository;
    
    @Autowired
    private CustomerRepository customerRepository;
    
    public OrderResponse createOrder(OrderRequest request) {
        // Service가 모든 것을 처리
        
        // 1. 데이터 조회
        Customer customer = customerRepository.findById(request.getCustomerId())
            .orElseThrow(() -> new CustomerNotFoundException());
        Product product = productRepository.findById(request.getProductId())
            .orElseThrow(() -> new ProductNotFoundException());
        
        // 2. 비즈니스 로직 (Service가 직접 처리)
        if (product.getStock() < request.getQuantity()) {
            throw new OutOfStockException();
        }
        
        BigDecimal totalPrice = product.getPrice()
                                       .multiply(new BigDecimal(request.getQuantity()));
        
        if (customer.getGrade() == CustomerGrade.VIP) {
            totalPrice = totalPrice.multiply(new BigDecimal("0.9")); // 10% 할인
        } else if (customer.getGrade() == CustomerGrade.GOLD) {
            totalPrice = totalPrice.multiply(new BigDecimal("0.95")); // 5% 할인
        }
        
        // 3. 재고 감소 (Service가 직접 조작)
        product.setStock(product.getStock() - request.getQuantity());
        productRepository.save(product);
        
        // 4. 주문 생성
        Order order = new Order();
        order.setCustomerId(customer.getId());
        order.setProductId(product.getId());
        order.setQuantity(request.getQuantity());
        order.setTotalPrice(totalPrice);
        order.setStatus(OrderStatus.PENDING);
        
        Order savedOrder = orderRepository.save(order);
        
        return new OrderResponse(savedOrder);
    }
}

// 데이터만 가진 Order
@Entity
public class Order {
    @Id @GeneratedValue
    private Long id;
    
    private Long customerId;
    private Long productId;
    private Integer quantity;
    private BigDecimal totalPrice;
    private OrderStatus status;
    
    // getter, setter만...
}
```

**문제점**:

```
1. Service가 모든 로직 수행 (God Object)
2. Domain 객체는 데이터만 (Anemic Domain Model)
3. 할인 정책 변경 시 Service 수정
4. 새로운 고객 등급 추가 시 Service 수정
5. 재고 관리 로직이 Service에 노출
```

---

### After: 책임 중심 설계 (GRASP 패턴 적용)

#### 1. INFORMATION EXPERT: 도메인 객체가 책임 수행

```java
// ✅ Customer가 할인 계산 책임
@Entity
public class Customer {
    @Id @GeneratedValue
    private Long id;
    
    private String name;
    
    @Enumerated(EnumType.STRING)
    private CustomerGrade grade;
    
    // ✅ INFORMATION EXPERT: 할인율을 아는 Customer가 계산
    public Money calculateDiscountedPrice(Money originalPrice) {
        return grade.applyDiscount(originalPrice);
    }
}

// ✅ POLYMORPHISM: 등급별 다형성
public enum CustomerGrade {
    NORMAL {
        @Override
        Money applyDiscount(Money price) {
            return price;  // 할인 없음
        }
    },
    GOLD {
        @Override
        Money applyDiscount(Money price) {
            return price.multiply(0.95);  // 5% 할인
        }
    },
    VIP {
        @Override
        Money applyDiscount(Money price) {
            return price.multiply(0.9);  // 10% 할인
        }
    };
    
    abstract Money applyDiscount(Money price);
}
```

```java
// ✅ Product가 재고 관리 책임
@Entity
public class Product {
    @Id @GeneratedValue
    private Long id;
    
    private String name;
    private Money price;
    private Integer stock;
    
    // ✅ INFORMATION EXPERT: 재고를 아는 Product가 관리
    public void decreaseStock(int quantity) {
        if (this.stock < quantity) {
            throw new OutOfStockException(
                String.format("재고 부족: 현재 %d개, 요청 %d개", 
                             this.stock, quantity));
        }
        this.stock -= quantity;
    }
    
    public Money calculatePrice(int quantity) {
        return price.multiply(quantity);
    }
}
```

```java
// ✅ Order가 주문 생성 책임 (CREATOR)
@Entity
public class Order {
    @Id @GeneratedValue
    private Long id;
    
    @ManyToOne(fetch = FetchType.LAZY)
    private Customer customer;
    
    @ManyToOne(fetch = FetchType.LAZY)
    private Product product;
    
    private Integer quantity;
    private Money totalPrice;
    
    @Enumerated(EnumType.STRING)
    private OrderStatus status;
    
    // ✅ CREATOR: Order가 자신을 생성
    public static Order create(Customer customer, Product product, 
                              int quantity) {
        // 1. 재고 확인 및 감소 (Product의 책임)
        product.decreaseStock(quantity);
        
        // 2. 가격 계산 (Product와 Customer의 협력)
        Money productPrice = product.calculatePrice(quantity);
        Money finalPrice = customer.calculateDiscountedPrice(productPrice);
        
        // 3. Order 생성
        Order order = new Order();
        order.customer = customer;
        order.product = product;
        order.quantity = quantity;
        order.totalPrice = finalPrice;
        order.status = OrderStatus.PENDING;
        
        return order;
    }
    
    // ✅ 상태 변경도 Order의 책임
    public void cancel() {
        if (status != OrderStatus.PENDING) {
            throw new IllegalStateException(
                "취소 가능한 상태가 아닙니다: " + status);
        }
        
        // 재고 복구
        product.increaseStock(quantity);
        this.status = OrderStatus.CANCELLED;
    }
}
```

```java
// ✅ Service는 조율만
@Service
@Transactional
public class OrderService {
    @Autowired
    private OrderRepository orderRepository;
    
    @Autowired
    private ProductRepository productRepository;
    
    @Autowired
    private CustomerRepository customerRepository;
    
    public OrderResponse createOrder(OrderRequest request) {
        // 1. 엔티티 조회
        Customer customer = customerRepository.findById(request.getCustomerId())
            .orElseThrow(() -> new CustomerNotFoundException());
        Product product = productRepository.findById(request.getProductId())
            .orElseThrow(() -> new ProductNotFoundException());
        
        // 2. Order에게 생성 책임 위임 (CREATOR)
        Order order = Order.create(customer, product, request.getQuantity());
        
        // 3. 저장
        Order savedOrder = orderRepository.save(order);
        
        return new OrderResponse(savedOrder);
    }
}
```

---

### 개선 효과 비교

| 측면 | Before | After |
|------|--------|-------|
| **Service 라인 수** | ~50줄 | ~15줄 |
| **비즈니스 로직 위치** | Service | Domain |
| **할인 정책 변경** | Service 수정 | CustomerGrade만 수정 |
| **재고 관리 변경** | Service 수정 | Product만 수정 |
| **테스트** | Service 통합 테스트 | Domain 단위 테스트 |
| **재사용성** | 낮음 (Service 의존) | 높음 (독립적) |

---

### GRASP 패턴 적용 분석

**INFORMATION EXPERT**:
```
- Customer: 할인율 정보 → 할인 계산 책임
- Product: 재고 정보 → 재고 관리 책임
- Product: 가격 정보 → 가격 계산 책임
```

**CREATOR**:
```
- Order가 자신을 생성
  ✅ Customer, Product를 참조
  ✅ 초기화 데이터 보유
```

**POLYMORPHISM**:
```
- CustomerGrade enum의 다형성
  → 새 등급 추가 시 enum만 수정
```

**LOW COUPLING**:
```
- Service는 Repository와 Domain에만 의존
- Domain 객체들은 서로 협력하지만 느슨하게 결합
```

**HIGH COHESION**:
```
- Customer: 고객 관련 책임만
- Product: 상품 관련 책임만
- Order: 주문 관련 책임만
```

---

## ❓ 자주 하는 질문 (Q&A)

### Q1. GRASP 패턴을 모두 외워야 하나요?

**A**: 이름을 외우는 것보다 **원리를 이해**하는 것이 중요합니다.

```
GRASP 패턴의 본질:

1. INFORMATION EXPERT
   → "이 정보를 아는 애가 책임져"

2. CREATOR
   → "이미 관련있는 애가 만들어"

3. LOW COUPLING & HIGH COHESION
   → "덜 얽히고, 더 집중하게"

4. POLYMORPHISM
   → "타입 체크 말고 다형성 써"

5. PROTECTED VARIATIONS
   → "변하는 건 인터페이스 뒤로"
```

**실전 적용**:
```java
// 질문 1: "누가 할인 금액을 계산해야 하지?"
// → 할인 정보를 아는 DiscountPolicy

// 질문 2: "누가 Reservation을 만들어야 하지?"
// → Reservation과 관련 깊은 Screening

// 질문 3: "두 설계 중 어느 게 나을까?"
// → 결합도와 응집도를 비교

이렇게 자연스럽게 패턴을 적용하게 됨!
```

---

### Q2. 책임 주도 설계는 항상 정답인가요?

**A**: 아닙니다. **문맥에 따라 다릅니다.**

#### 책임 주도 설계가 적합한 경우

```
✅ 복잡한 비즈니스 로직
✅ 요구사항 변경이 잦은 도메인
✅ 장기간 유지보수가 필요한 시스템
✅ 여러 개발자가 협업하는 프로젝트
```

**예시**:
```java
// 전자상거래 주문 시스템
// - 할인 정책이 자주 변경
// - 결제 방식이 다양
// - 배송 옵션이 복잡
→ 책임 주도 설계 적합!
```

#### 단순한 설계가 나은 경우

```
✅ 단순한 CRUD
✅ 요구사항이 안정적
✅ 짧은 개발 기간
✅ 작은 규모의 프로젝트
```

**예시**:
```java
// 간단한 게시판 시스템
@Service
public class PostService {
    public PostResponse create(PostRequest request) {
        Post post = new Post();
        post.setTitle(request.getTitle());
        post.setContent(request.getContent());
        return new PostResponse(postRepository.save(post));
    }
}
// → 이 정도면 충분할 수 있음
```

---

### Q3. Service 계층은 항상 얇아야 하나요?

**A**: **"트랜잭션 경계"와 "조율"**이 주 역할입니다.

#### Service의 올바른 역할

```java
@Service
@Transactional
public class OrderService {
    // ✅ 트랜잭션 관리
    // ✅ 여러 도메인 객체 조율
    // ✅ 인프라스트럭처 레이어와의 연결
    
    public OrderResponse createOrder(OrderRequest request) {
        // 1. 엔티티 조회 (인프라)
        Customer customer = customerRepository.findById(...);
        Product product = productRepository.findById(...);
        
        // 2. 도메인 로직 실행 (조율)
        Order order = Order.create(customer, product, quantity);
        
        // 3. 저장 (인프라)
        orderRepository.save(order);
        productRepository.save(product);
        
        // 4. 이벤트 발행 (인프라)
        eventPublisher.publish(new OrderCreatedEvent(order));
        
        return new OrderResponse(order);
    }
}
```

#### Service에 있어도 되는 로직

```
✅ 여러 Aggregate 간 조율
✅ 트랜잭션 관리
✅ 이벤트 발행
✅ 외부 API 호출
✅ 메시징
```

#### Domain에 있어야 하는 로직

```
✅ 비즈니스 규칙
✅ 도메인 불변식
✅ 상태 변경
✅ 정책 결정
```

---

### Q4. getter는 항상 나쁜가요?

**A**: **"무엇을 위한 getter인가?"**가 중요합니다.

#### 나쁜 getter

```java
// ❌ 내부 의사결정을 외부에 노출
public class Order {
    private OrderStatus status;
    
    public OrderStatus getStatus() {
        return status;
    }
}

// 외부에서 판단
if (order.getStatus() == OrderStatus.PENDING) {
    // 취소 로직
}
```

**문제**: Order의 비즈니스 로직이 외부로 누출

#### 좋은 getter

```java
// ✅ 단순 조회용
public class Order {
    private OrderStatus status;
    
    // 비즈니스 의사결정은 내부에서
    public boolean isCancellable() {
        return status == OrderStatus.PENDING;
    }
    
    // 외부 표현용 getter는 OK
    public OrderStatus getStatus() {
        return status;
    }
}

// 외부에서 사용
if (order.isCancellable()) {  // ✅ 의도 명확
    order.cancel();
}

// UI에 표시
view.showStatus(order.getStatus());  // ✅ 단순 조회
```

#### 판단 기준

```
getter를 사용하는 이유가:

1. 의사결정을 위한 것
   → ❌ 나쁨, 메서드로 캡슐화

2. 단순 조회/표시를 위한 것
   → ✅ 괜찮음

3. DTO 변환을 위한 것
   → ✅ 괜찮음
```

---

### Q5. 모든 조건문을 다형성으로 바꿔야 하나요?

**A**: **"변경의 빈도"를 고려하세요.**

#### 다형성으로 바꾸면 좋은 경우

```java
// ❌ 자주 변경되는 타입별 분기
public Money calculateDiscount() {
    switch(discountType) {
        case AMOUNT: return Money.wons(1000);
        case PERCENT: return fee.times(0.1);
        case NONE: return Money.ZERO;
        case SEASONAL: ...  // 추가
        case MEMBER: ...    // 추가
        case COUPON: ...    // 추가
    }
}

// ✅ 다형성으로 개선
public abstract class DiscountPolicy {
    abstract Money calculate();
}
```

**이유**: 새로운 할인 타입이 자주 추가됨

#### 조건문이 나은 경우

```java
// ✅ 안정적인 분기
public class Money {
    public Money plus(Money other) {
        if (other == null) {  // 예외 처리
            return this;
        }
        return new Money(amount.add(other.amount));
    }
}

// ✅ 단순한 상태 체크
public void process() {
    if (status == Status.READY) {
        execute();
    }
}
```

**이유**: 변경이 거의 없고 단순함

---

### Q6. 리팩터링은 언제 해야 하나요?

**A**: **"보이스카우트 규칙"**을 따르세요.

```
"캠핑장을 떠날 때는
 왔을 때보다 깨끗하게 만들어라"

→ "코드를 체크아웃할 때보다
   체크인할 때 더 깨끗하게"
```

#### 리팩터링 타이밍

**1. 기능 추가 전**:
```
새 기능을 추가하기 전에
기존 코드를 이해하기 쉽게 만들기
```

**2. 코드 리뷰 시**:
```
PR을 보다가 개선점이 보이면
작은 리팩터링 제안
```

**3. 버그 수정 중**:
```
버그가 발생한 코드는
구조에 문제가 있을 가능성 높음
```

**4. 중복 발견 시**:
```
3번째 중복이 발생하면
반드시 리팩터링 (Rule of Three)
```

#### 리팩터링 하지 말아야 할 때

```
❌ 배포 직전
❌ 테스트가 없을 때
❌ 요구사항이 불명확할 때
❌ 시간이 너무 촉박할 때
```

---

## 💡 실전 팁과 모범 사례

### 1. 메시지 먼저 생각하는 연습

```java
// ❌ 객체부터 생각
"Order 클래스가 필요해"
"Order는 뭘 가지고 있을까?"

// ✅ 메시지부터 생각
"주문을 취소해야 해"
"누가 취소 메시지를 처리할까?"
order.cancel();
```

**연습 방법**:
```
1. 기능 요구사항을 메시지로 표현
2. 메시지를 처리할 객체 찾기
3. 객체에 책임 할당
```

---

### 2. 작은 메서드, 명확한 이름

```java
// ❌ 거대한 메서드
public void processOrder(OrderRequest request) {
    // 100줄의 로직...
}

// ✅ 작은 메서드들
public void processOrder(OrderRequest request) {
    validateRequest(request);
    Customer customer = findCustomer(request);
    Product product = findProduct(request);
    Order order = createOrder(customer, product, request);
    notifyCustomer(order);
}
```

---

### 3. Tell, Don't Ask

```java
// ❌ Ask (묻지 말고)
if (order.getStatus() == OrderStatus.PENDING) {
    order.setStatus(OrderStatus.CONFIRMED);
}

// ✅ Tell (시켜라)
order.confirm();
```

---

### 4. Law of Demeter (최소 지식 원칙)

```java
// ❌ 기차 충돌
customer.getAddress().getCity().getName();

// ✅ 필요한 것만 묻기
customer.getCityName();
```

---

### 5. 불변성 활용

```java
// ✅ 불변 값 객체
public class Money {
    private final BigDecimal amount;
    
    public Money plus(Money other) {
        return new Money(amount.add(other.amount));  // 새 객체 반환
    }
}
```

---

## 📖 더 읽어보기

### 추천 도서

```
1. "오브젝트" - 조영호
   → 이 책!

2. "엔터프라이즈 애플리케이션 아키텍처 패턴" - Martin Fowler
   → 레이어드 아키텍처와 패턴

3. "도메인 주도 설계" - Eric Evans
   → DDD와 책임 할당

4. "클린 코드" - Robert C. Martin
   → 코드 품질과 원칙

5. "Refactoring" - Martin Fowler
   → 리팩터링 카탈로그
```

### 온라인 자료

```
- Martin Fowler's Blog (martinfowler.com)
- Kent Beck's Twitter
- Uncle Bob's Blog
```

---

## 🎓 최종 요약

### 핵심 원칙 5가지

```
1. 데이터가 아닌 책임으로 시작하라
2. 메시지가 객체를 선택하게 하라
3. 협력 문맥에서 책임을 결정하라
4. 정보를 아는 자에게 책임을 주라
5. 변경을 고려하여 설계하라
```

### GRASP 패턴 체크리스트

```
□ INFORMATION EXPERT: 정보 전문가에게 책임 할당했는가?
□ CREATOR: 생성 책임이 적절한 곳에 있는가?
□ LOW COUPLING: 결합도를 낮췄는가?
□ HIGH COHESION: 응집도를 높였는가?
□ POLYMORPHISM: 타입 체크 대신 다형성을 사용했는가?
□ PROTECTED VARIATIONS: 변경을 캡슐화했는가?
```

### 실천 방법

```
1주차: 기존 코드에서 몬스터 메서드 찾아 분해하기
2주차: Service의 비즈니스 로직을 Domain으로 이동하기
3주차: 조건문을 다형성으로 바꾸기
4주차: 새 기능을 책임 주도로 설계하기
```

### 마지막 조언

```
완벽한 설계는 없습니다.
더 나은 설계만 있을 뿐입니다.

처음부터 잘할 수 없습니다.
하지만 계속 개선할 수 있습니다.

설계는 예술이 아닙니다.
원칙과 패턴으로 배울 수 있습니다.

지금 당장 시작하세요! 🚀
```

---

<div align="center">

**[⬆ 목차로 돌아가기](../README.md)**

</div>

<div align="center">

**[← Chapter 04](../chapter04/README.md) | [Chapter 06 →](../chapter06/README.md)**

</div>
