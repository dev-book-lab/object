# Chapter 06. 메시지와 인터페이스

> *"훌륭한 객체지향 코드를 얻기 위해서는 클래스가 아니라 객체를 지향해야 한다"*

## 📌 핵심 개념

- **메시지(Message)**: 객체들이 협력하기 위해 사용하는 유일한 의사소통 수단
- **퍼블릭 인터페이스(Public Interface)**: 객체가 외부에 공개하는 메시지의 집합
- **디미터 법칙(Law of Demeter)**: 객체의 내부 구조에 결합되지 않도록 협력 경로를 제한
- **묻지 말고 시켜라(Tell, Don't Ask)**: 객체의 상태를 묻지 말고 원하는 것을 시켜라
- **의도를 드러내는 인터페이스**: 구현이 아닌 의도를 표현하는 메서드 이름
- **명령-쿼리 분리(Command-Query Separation)**: 명령과 쿼리를 명확하게 분리

---

## 🎯 학습 목표

1. **메시지 중심 사고**로 객체지향 설계의 본질 이해하기
2. **퍼블릭 인터페이스 설계 원칙**들을 체계적으로 학습하기
3. **디미터 법칙**을 올바르게 이해하고 적용하기
4. **명령-쿼리 분리**로 예측 가능한 코드 작성하기
5. 원칙들 사이의 **트레이드오프** 판단 능력 기르기

---

## 🔑 Chapter 핵심 메시지

### 왜 메시지가 중요한가?

```
클래스로 구성되지만, 메시지로 정의된다

애플리케이션 = 클래스들의 집합 (X)
애플리케이션 = 메시지들의 협력 (O)

설계의 품질 = 퍼블릭 인터페이스의 품질
```

**이번 챕터의 핵심**:
```
좋은 퍼블릭 인터페이스를 만드는 방법
→ 4가지 원칙과 기법 제시
→ 원칙의 한계와 트레이드오프 이해
```

---

## 🎬 01. 협력과 메시지

### 📡 클라이언트-서버 모델

> 객체 간 협력을 이해하는 전통적인 메타포

```
[Client] ──메시지 전송──→ [Server]
    ↑                        ↓
    └──────응답(결과)──────────┘

클라이언트: 메시지를 전송하는 객체
서버: 메시지를 수신하고 처리하는 객체
```

**중요한 사실**:

```
한 객체는 협력 안에서:
- 클라이언트이면서 동시에
- 서버이기도 함

예시:
Screening ──→ Movie ──→ DiscountCondition
(클라이언트) (서버이자    (서버)
              클라이언트)
```

---

### 📨 메시지와 메시지 전송

#### 메시지의 구성

```java
condition.isSatisfiedBy(screening);
    ↑           ↑            ↑
 수신자    오퍼레이션명    인자(argument)
```

**메시지 전송(Message Sending/Passing)**:
```
메시지 전송 = 수신자 + 오퍼레이션명 + 인자

메시지: isSatisfiedBy(screening)
메시지 전송: condition.isSatisfiedBy(screening)
```

---

### 🎭 메시지와 메서드의 차이

> **핵심**: 메시지를 보낸다고 해서 어떤 코드가 실행될지 알 수 없다

```java
DiscountCondition condition = ...;  // 실제 타입은 모름
condition.isSatisfiedBy(screening);
```

**실행 시점에 결정**:

```
condition이 PeriodCondition이면
→ PeriodCondition.isSatisfiedBy() 실행

condition이 SequenceCondition이면
→ SequenceCondition.isSatisfiedBy() 실행

메시지 ≠ 메서드
메시지 = 무엇을 할지 (what)
메서드 = 어떻게 할지 (how)
```

---

### 📚 용어 정리

체계적 이해를 위한 용어 정의:

| 용어 | 정의 | 예시 |
|------|------|------|
| **메시지** | 객체 간 협력을 위한 의사소통 메커니즘 | `calculateFee(audienceCount)` |
| **메시지 전송** | 메시지에 수신자를 추가한 것 | `screening.calculateFee(2)` |
| **메시지 전송자** | 메시지를 보내는 객체 (클라이언트) | `Theater` |
| **메시지 수신자** | 메시지를 받는 객체 (서버) | `Screening` |
| **오퍼레이션** | 퍼블릭 인터페이스에 포함된 메시지 | `calculateFee` |
| **메서드** | 메시지에 응답하기 위해 실행되는 코드 | `PeriodCondition의 isSatisfiedBy 구현` |
| **시그니처** | 오퍼레이션/메서드의 이름 + 파라미터 목록 | `calculateFee(int)` |
| **퍼블릭 인터페이스** | 외부에 공개된 메시지들의 집합 | `Screening의 public 메서드들` |

---

### 🎯 다형성의 진정한 의미

> **오퍼레이션 관점에서의 다형성**

```java
// 같은 오퍼레이션 호출
DiscountCondition condition1 = new PeriodCondition(...);
DiscountCondition condition2 = new SequenceCondition(...);

condition1.isSatisfiedBy(screening);  // Period 메서드 실행
condition2.isSatisfiedBy(screening);  // Sequence 메서드 실행
```

**다형성 = 동일한 오퍼레이션 호출 → 서로 다른 메서드 실행**

```
메시지 송신자의 관점:
"나는 isSatisfiedBy라는 메시지만 안다"
"어떻게 처리되는지는 모른다"
"신경 쓰지 않는다"

→ 이것이 바로 캡슐화!
```

---

## 🎨 02. 인터페이스와 설계 품질

### 좋은 인터페이스의 조건

```
1. 최소한의 인터페이스 (Minimal Interface)
   - 꼭 필요한 오퍼레이션만 포함
   - "적을수록 좋다"

2. 추상적인 인터페이스 (Abstract Interface)
   - "어떻게"가 아닌 "무엇을"
   - 구현 세부사항 숨김
```

**달성 방법**: 책임 주도 설계!

```
메시지를 먼저 선택하고
그 후에 객체를 선택하면
자동으로 좋은 인터페이스가 만들어진다
```

---

### 🚂 디미터 법칙 (Law of Demeter)

> **"낯선 자에게 말하지 마라 (Don't Talk to Strangers)"**

#### Chapter 04의 나쁜 예제

> 📂 **코드**: [`ReservationAgency.java`](https://github.com/eternity-oop/object/blob/master/chapter04/src/main/java/org/eternity/movie/step01/ReservationAgency.java)

```java
public class ReservationAgency {
    public Reservation reserve(Screening screening, Customer customer,
                               int audienceCount) {
        // ❌ Screening의 내부 깊숙이 침투
        Movie movie = screening.getMovie();
        
        boolean discountable = false;
        for(DiscountCondition condition : movie.getDiscountConditions()) {
            if (condition.getType() == DiscountConditionType.PERIOD) {
                // ❌ 기차 충돌!
                discountable = 
                    screening.getWhenScreened().getDayOfWeek()
                             .equals(condition.getDayOfWeek()) &&
                    condition.getStartTime()
                             .compareTo(screening.getWhenScreened()
                                                 .toLocalTime()) <= 0 &&
                    condition.getEndTime()
                             .compareTo(screening.getWhenScreened()
                                                 .toLocalTime()) >= 0;
            } else {
                discountable = 
                    condition.getSequence() == screening.getSequence();
            }
            
            if (discountable) {
                break;
            }
        }
        // ...
    }
}
```

**문제점 분석**:

```
ReservationAgency가 의존하는 것들:
1. Screening       (직접 의존)
2. Movie           (Screening을 통해)
3. DiscountCondition (Movie를 통해)
4. LocalDateTime   (Screening을 통해)
5. DayOfWeek       (LocalDateTime을 통해)

결합도 폭발! 💥
```

**변경의 파급 효과**:

```
시나리오 1: Screening이 Movie를 직접 포함하지 않게 변경
→ ReservationAgency 수정 필요

시나리오 2: DiscountCondition이 sequence를 포함하지 않게 변경
→ ReservationAgency 수정 필요

시나리오 3: sequence 타입이 int → Long으로 변경
→ ReservationAgency 수정 필요

ReservationAgency = 변경의 집결지!
```

---

#### 디미터 법칙의 정의

> **메서드가 메시지를 전송할 수 있는 대상을 제한**

```
메서드 M이 메시지를 전송할 수 있는 대상:

✅ this 객체
✅ 메서드의 매개변수
✅ this의 속성(인스턴스 변수)
✅ this 속성인 컬렉션의 요소
✅ 메서드 내에서 생성된 지역 객체

❌ 위 대상들로부터 얻은 객체 (X)
```

**쉬운 표현**:
```
"오직 하나의 도트(.)만 사용하라"

단, 주의: 이것은 비유일 뿐!
(뒤에서 자세히 설명)
```

---

#### Chapter 05의 좋은 예제

> 📂 **코드**: [`ReservationAgency.java` (개선 후)](https://github.com/eternity-oop/object/blob/master/chapter05/src/main/java/org/eternity/movie/step01/Screening.java)

```java
public class ReservationAgency {
    public Reservation reserve(Screening screening, Customer customer,
                               int audienceCount) {
        // ✅ Screening에게만 메시지 전송
        Money fee = screening.calculateFee(audienceCount);
        
        return new Reservation(customer, screening, fee, audienceCount);
    }
}
```

**개선 효과**:

```
ReservationAgency의 의존성:
1. Screening       (직접 의존)
2. Customer        (매개변수)
3. Reservation     (생성)

Screening 내부 구조 변경?
→ ReservationAgency는 영향 없음! ✅
```

---

#### 🚂 기차 충돌 (Train Wreck)

디미터 법칙을 위반하는 전형적인 코드:

```java
// ❌ 기차 충돌의 전형
screening.getMovie().getDiscountConditions();
    ↓         ↓              ↓
   1량차      2량차          3량차

// ❌ 더 심한 경우
screening.getMovie()
         .getDiscountConditions()
         .get(0)
         .getDayOfWeek()
         .getValue();
```

**왜 나쁜가?**

```
1. 내부 구조 노출
   - Screening이 Movie를 가짐
   - Movie가 DiscountCondition 리스트를 가짐
   - 모든 구조가 외부에 드러남

2. 강한 결합
   - Screening의 내부 변경 → 호출 코드 수정
   - Movie의 내부 변경 → 호출 코드 수정
   - DiscountCondition 변경 → 호출 코드 수정

3. 이해하기 어려움
   - 각 단계의 타입을 모두 알아야 함
   - 중간에 null이 나오면? NPE 폭탄!
```

---

#### ✅ 디미터 법칙을 따르는 코드

```java
// ❌ Before: 기차 충돌
screening.getMovie().getDiscountConditions();

// ✅ After: 하나의 메시지
screening.calculateFee(audienceCount);
```

**리팩터링 과정**:

```java
// Step 1: Screening에 메서드 추가
public class Screening {
    private Movie movie;
    
    // ✅ 내부에서 Movie에게 위임
    public Money calculateFee(int audienceCount) {
        return movie.calculateMovieFee(this).times(audienceCount);
    }
}

// Step 2: Movie에 메서드 추가
public class Movie {
    private List<DiscountCondition> discountConditions;
    
    // ✅ 내부에서 DiscountCondition들과 협력
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
}
```

**핵심**:

```
각 객체는 자신의 이웃에게만 메시지를 보낸다
→ Screening → Movie → DiscountCondition
→ 체인이 아닌 위임!
```

---

#### 🎯 디미터 법칙의 장점

**1. 부끄럼타는 코드 (Shy Code)**

```java
// 부끄럼타는 코드 = 필요한 것만 공개
public class Screening {
    private Movie movie;          // ❌ 보여주지 않음
    private LocalDateTime when;   // ❌ 보여주지 않음
    
    // ✅ 꼭 필요한 것만
    public Money calculateFee(int audienceCount) {
        return movie.calculateMovieFee(this).times(audienceCount);
    }
}
```

**2. 높은 응집도**

```java
// 정보와 행동이 함께
public class Movie {
    private List<DiscountCondition> discountConditions;  // 정보
    
    // ✅ 정보를 사용하는 행동도 여기에
    private boolean isDiscountable(Screening screening) {
        return discountConditions.stream()
                .anyMatch(condition -> condition.isSatisfiedBy(screening));
    }
}
```

**3. 낮은 결합도**

```
변경의 파급효과 차단:

Movie 내부 구조 변경
→ Movie 클래스만 수정
→ Screening은 영향 없음
→ Theater는 영향 없음

캡슐화 = 변경의 방파제
```

---

### 💬 묻지 말고 시켜라 (Tell, Don't Ask)

> **객체의 상태를 묻지 말고, 원하는 것을 시켜라**

#### Theater 예제: Step 01 (나쁜 코드)

> 📂 **코드**: [`Theater.java` (Step 01)](https://github.com/eternity-oop/object/blob/master/chapter06/src/main/java/org/eternity/theater/step01/Theater.java)

```java
public class Theater {
    private TicketSeller ticketSeller;
    
    public void enter(Audience audience) {
        // ❌ 묻고 있다: "초대권 있니?"
        if (audience.getBag().hasInvitation()) {
            Ticket ticket = ticketSeller.getTicketOffice().getTicket();
            // ❌ 직접 조작
            audience.getBag().setTicket(ticket);
        } else {
            Ticket ticket = ticketSeller.getTicketOffice().getTicket();
            // ❌ 묻고, 판단하고, 조작
            audience.getBag().minusAmount(ticket.getFee());
            ticketSeller.getTicketOffice().plusAmount(ticket.getFee());
            audience.getBag().setTicket(ticket);
        }
    }
}
```

**문제점**:

```
Theater가 하는 일:
1. Audience의 Bag에게 "초대권 있니?" 물음
2. 있으면 이렇게, 없으면 저렇게 판단
3. Bag의 상태 직접 변경
4. TicketOffice의 상태 직접 변경

→ Theater = 절차적 코드의 집결지
→ Audience, Bag, TicketSeller = 데이터 덩어리
```

---

#### Theater 예제: Step 03 (좋은 코드)

> 📂 **코드**: [`Theater.java` (Step 03)](https://github.com/eternity-oop/object/blob/master/chapter06/src/main/java/org/eternity/theater/step03/Theater.java)

```java
// ✅ Theater: 시키기만 함
public class Theater {
    private TicketSeller ticketSeller;
    
    public void enter(Audience audience) {
        // ✅ TicketSeller에게 시킴
        ticketSeller.sellTo(audience);
    }
}

// ✅ TicketSeller: 자신의 일을 처리
public class TicketSeller {
    private TicketOffice ticketOffice;
    
    public void sellTo(Audience audience) {
        // ✅ Audience에게 시킴
        ticketOffice.plusAmount(
            audience.buy(ticketOffice.getTicket())
        );
    }
}

// ✅ Audience: 자신의 일을 처리
public class Audience {
    private Bag bag;
    
    public Long buy(Ticket ticket) {
        // ✅ Bag에게 시킴
        return bag.hold(ticket);
    }
}

// ✅ Bag: 자신의 상태를 스스로 관리
public class Bag {
    private Long amount;
    private Invitation invitation;
    private Ticket ticket;
    
    public Long hold(Ticket ticket) {
        // ✅ 스스로 판단하고 처리
        if (hasInvitation()) {
            this.ticket = ticket;
            return 0L;
        } else {
            this.ticket = ticket;
            minusAmount(ticket.getFee());
            return ticket.getFee();
        }
    }
    
    private boolean hasInvitation() {
        return invitation != null;
    }
    
    private void minusAmount(Long amount) {
        this.amount -= amount;
    }
}
```

---

#### Before vs After 비교

| 측면 | Before (묻고 조작) | After (시킴) |
|------|-------------------|-------------|
| **Theater 코드 길이** | 15줄 | 3줄 |
| **책임 분산** | Theater가 모든 로직 | 각 객체가 자신의 로직 |
| **Bag 캡슐화** | hasInvitation() 노출 | private으로 감춤 |
| **Audience 역할** | 데이터 홀더 | 스스로 구매 결정 |
| **TicketSeller 역할** | 데이터 홀더 | 스스로 판매 처리 |
| **결합도** | 높음 | 낮음 |
| **응집도** | 낮음 | 높음 |

---

#### 묻지 말고 시켜라의 핵심

```
❌ 나쁜 패턴:
1. 객체의 상태를 묻는다 (getter 호출)
2. 그 상태로 판단한다 (if-else)
3. 객체의 상태를 바꾼다 (setter 호출)

✅ 좋은 패턴:
1. 객체에게 할 일을 시킨다 (메서드 호출)
2. 객체가 스스로 판단한다
3. 객체가 스스로 상태를 바꾼다
```

**정보 전문가 패턴과의 관계**:

```
정보를 가진 객체가 책임을 진다
→ 정보와 행동이 함께 있다
→ 묻지 말고 시킬 수 있다

순환 관계:
묻지 말고 시켜라 → 정보 전문가에게 책임 할당
정보 전문가 패턴 → 묻지 말고 시킬 수 있게 됨
```

---

### 🎭 의도를 드러내는 인터페이스

> **"어떻게"가 아닌 "무엇을"을 표현하라**

#### 나쁜 메서드 이름 (구현 중심)

```java
// ❌ 어떻게 하는지를 나타냄
public class PeriodCondition {
    public boolean isSatisfiedByPeriod(Screening screening) {
        return dayOfWeek.equals(screening.getWhenScreened().getDayOfWeek()) &&
               startTime.compareTo(screening.getWhenScreened().toLocalTime()) <= 0 &&
               endTime.compareTo(screening.getWhenScreened().toLocalTime()) >= 0;
    }
}

public class SequenceCondition {
    public boolean isSatisfiedBySequence(Screening screening) {
        return sequence == screening.getSequence();
    }
}
```

**문제점**:

```
1. 구현 방법이 드러남
   - "Period로 확인한다"
   - "Sequence로 확인한다"
   → 내부 구현이 메서드 이름에 노출

2. 동일한 목적, 다른 이름
   - 둘 다 "할인 조건을 만족하는가?"를 확인
   - 하지만 메서드 이름이 달라서 알기 어려움

3. 변경에 취약
   - Period → Time으로 변경하면?
   - 메서드 이름도 바꿔야 함
   - 호출하는 모든 코드 수정 필요
```

---

#### 좋은 메서드 이름 (의도 중심)

```java
// ✅ 무엇을 하는지를 나타냄
public interface DiscountCondition {
    boolean isSatisfiedBy(Screening screening);
}

public class PeriodCondition implements DiscountCondition {
    @Override
    public boolean isSatisfiedBy(Screening screening) {
        return dayOfWeek.equals(screening.getWhenScreened().getDayOfWeek()) &&
               startTime.compareTo(screening.getWhenScreened().toLocalTime()) <= 0 &&
               endTime.compareTo(screening.getWhenScreened().toLocalTime()) >= 0;
    }
}

public class SequenceCondition implements DiscountCondition {
    @Override
    public boolean isSatisfiedBy(Screening screening) {
        return sequence == screening.getSequence();
    }
}
```

**개선 효과**:

```
1. 의도가 명확
   - "조건을 만족하는가?"
   - 어떻게 확인하는지는 중요하지 않음

2. 일관된 인터페이스
   - 모든 할인 조건이 같은 메시지에 응답
   - 타입 계층으로 묶을 수 있음

3. 변경에 강함
   - 내부 구현 변경 → 메서드 이름 유지
   - 새로운 조건 추가 → 인터페이스만 구현
```

---

#### Kent Beck의 조언

> **"의도를 드러내는 선택자 (Intention Revealing Selector)"**

```
메서드 이름을 지을 때:

1. 매우 다른 두 번째 구현을 상상하라
2. 그 구현에도 동일한 이름을 붙일 수 있나?
3. 가능하다면 그것이 가장 추상적인 이름!

예시:
"isSatisfiedByPeriod" → 두 번째 구현: "isSatisfiedBySequence"
  → 다른 이름! 추상화 실패

"isSatisfiedBy" → 두 번째 구현: "isSatisfiedBy"
  → 같은 이름! 추상화 성공
```

---

#### Theater 예제의 메서드 이름 개선

**Step 02 (구현 중심)**:

```java
public class TicketSeller {
    // ❌ 어떻게: "티켓을 설정한다"
    public void setTicket(Audience audience) {
        ticketOffice.plusAmount(
            audience.setTicket(ticketOffice.getTicket())
        );
    }
}

public class Audience {
    // ❌ 어떻게: "티켓을 설정한다"
    public Long setTicket(Ticket ticket) {
        return bag.setTicket(ticket);
    }
}
```

**Step 03 (의도 중심)**:

```java
public class TicketSeller {
    // ✅ 무엇을: "관객에게 판매한다"
    public void sellTo(Audience audience) {
        ticketOffice.plusAmount(
            audience.buy(ticketOffice.getTicket())
        );
    }
}

public class Audience {
    // ✅ 무엇을: "티켓을 구매한다"
    public Long buy(Ticket ticket) {
        return bag.hold(ticket);
    }
}

public class Bag {
    // ✅ 무엇을: "티켓을 보관한다"
    public Long hold(Ticket ticket) {
        if (hasInvitation()) {
            this.ticket = ticket;
            return 0L;
        } else {
            this.ticket = ticket;
            minusAmount(ticket.getFee());
            return ticket.getFee();
        }
    }
}
```

---

#### 의도를 드러내는 인터페이스의 가치

```
1. 클라이언트 관점
   - 무엇을 원하는지 명확
   - "티켓을 판매하고 싶어" → sellTo()
   - "티켓을 구매하고 싶어" → buy()

2. 설계자 관점
   - 구현 자유도 확보
   - 내부를 바꿔도 인터페이스 유지
   - 리팩터링이 쉬움

3. 협력 관점
   - 역할이 명확해짐
   - 책임이 드러남
   - 다형성 적용이 자연스러움
```

---

## ⚖️ 03. 원칙의 함정

> **"원칙을 맹신하지 마라"**

### 핵심 메시지

```
설계 = 트레이드오프의 산물

좋은 설계자 vs 초보 설계자:
- 초보: 원칙을 맹목적으로 따름
- 숙련자: 상황에 따라 원칙을 조율함

"언제 원칙이 유용하고
 언제 유용하지 않은지를
 판단할 수 있는 능력"
```

---

### 🚫 함정 1: 디미터 법칙은 하나의 도트를 강제하는 규칙이 아니다

#### Stream API는 디미터 법칙 위반인가?

```java
// ❓ 이것은 디미터 법칙 위반인가?
IntStream.of(1, 15, 20, 3, 9)
         .filter(x -> x > 10)
         .distinct()
         .count();
```

**답**: **위반 아님!** ✅

**이유**:

```
디미터 법칙의 핵심:
"객체의 내부 구조가 외부로 노출되는가?"

Stream의 경우:
- of() → IntStream 반환
- filter() → IntStream 반환
- distinct() → IntStream 반환
- count() → long 반환

모두 IntStream을 IntStream으로 변환
→ 내부 구조 노출 없음
→ 객체를 다른 객체로 변환할 뿐
```

---

#### 판단 기준

```java
// ❌ 디미터 법칙 위반
screening.getMovie().getDiscountConditions();
         ↓            ↓
      Movie를      DiscountCondition
      노출함        리스트를 노출함
      
→ Screening의 내부 구조(Movie 보유)가 드러남

// ✅ 디미터 법칙 위반 아님
IntStream.of(1, 2, 3).filter(x -> x > 1).count();
         ↓                ↓           ↓
    IntStream        IntStream    long
    
→ 같은 타입의 변환만 수행
→ 내부 구조 노출 없음
```

**질문 리스트**:

```
여러 개의 도트를 사용한 코드를 볼 때:

1. 객체의 내부 구조를 알아야 하는가?
   YES → 디미터 법칙 위반
   NO → OK

2. 중간 단계의 타입이 노출되는가?
   YES → 디미터 법칙 위반
   NO → OK

3. 변경 시 여러 곳을 수정해야 하는가?
   YES → 디미터 법칙 위반
   NO → OK
```

---

### 🚫 함정 2: 결합도와 응집도의 충돌

> **"무조건 묻지 말고 시켜라는 정답이 아니다"**

#### 상황: 컬렉션 처리

```java
// ❓ 이것을 어떻게 개선할까?
for (Movie each : movies) {
    total += each.getFee();
}
```

**선택지 1**: Movies 클래스 생성

```java
// ✅ 응집도는 높아지지만...
public class Movies {
    private List<Movie> movies;
    
    public Money calculateTotalFee() {
        Money total = Money.ZERO;
        for (Movie each : movies) {
            total = total.plus(each.getFee());
        }
        return total;
    }
}
```

**문제**:
```
Movies 클래스의 존재 이유는?
→ "여러 Movie를 묶기 위해"

하지만:
- calculateTotalFee 외에 무슨 책임이 있나?
- 이 클래스는 정말 필요한가?
- 일급 컬렉션을 만들 가치가 있나?
```

**선택지 2**: 그냥 묻는다

```java
// ✅ 때로는 묻는 것이 나을 수 있다
for (Movie each : movies) {
    total += each.getFee();
}
```

**이유**:
```
1. Movie는 데이터인가 객체인가?
   - getFee()는 단순 속성 접근
   - 복잡한 로직 없음
   - 데이터에 가까움

2. 응집도 vs 결합도
   - Movies 클래스 추가 → 결합도 증가
   - 얻는 이득이 적음
   - 오히려 복잡도만 증가
```

---

#### 판단 기준

```
묻지 말고 시켜라를 적용하기 전 체크리스트:

□ 묻는 대상이 진짜 객체인가, 아니면 자료구조인가?
  → 자료구조면 묻는 것이 자연스러움

□ 시키기 위해 위임 메서드를 추가하면 응집도가 높아지나?
  → 아니라면 굳이 시킬 필요 없음

□ 클라이언트가 객체 내부를 알아야 결정을 내리는가?
  → YES면 시켜야 함
  → NO면 물어도 됨

□ 새로운 클래스를 추가하는 것이 정말 가치 있는가?
  → 일급 컬렉션이 꼭 필요한가?
```

---

#### Robert C. Martin (클린 코드)

> **"객체 vs 자료구조"**

```
객체:
- 추상화된 인터페이스
- 내부 숨김
- 행동 중심
→ 디미터 법칙 적용

자료구조:
- 데이터 노출
- getter/setter
- 데이터 중심
→ 당연히 내부 접근 OK
```

**예시**:

```java
// 자료구조 - 묻는 것이 자연스러움
public class Point {
    public double x;
    public double y;
}

double distance = Math.sqrt(
    Math.pow(p1.x - p2.x, 2) + 
    Math.pow(p1.y - p2.y, 2)
);

// 객체 - 시켜야 함
public class Point {
    public double distanceTo(Point other) {
        return Math.sqrt(
            Math.pow(x - other.x, 2) + 
            Math.pow(y - other.y, 2)
        );
    }
}

double distance = p1.distanceTo(p2);
```

---

### 결론: 원칙의 적용

```
원칙은 도구다:
- 맹신 금지
- 상황 판단
- 트레이드오프

좋은 설계자:
- 원칙을 안다
- 언제 적용할지 안다
- 언제 깰지도 안다

"원칙을 따르는 것보다
 왜 따르는지 아는 것이 중요"
```

---

## 🔀 04. 명령-쿼리 분리 원칙

> **"질문이 답변을 수정해서는 안 된다"**

### 용어 정의

```
루틴 (Routine)
- 어떤 절차를 묶어 호출 가능하도록 이름을 부여한 것
- 프로시저 + 함수

프로시저 (Procedure)
- 부수효과를 발생시킬 수 있음
- 값을 반환할 수 없음
- 상태를 변경

함수 (Function)
- 값을 반환할 수 있음
- 부수효과를 발생시킬 수 없음
- 상태를 변경하지 않음
```

**객체지향에서**:

```
명령 (Command) = 프로시저
- 객체의 상태를 수정
- 반환값 없음
- 부수효과 있음

쿼리 (Query) = 함수
- 객체의 정보를 반환
- 상태 변경 없음
- 부수효과 없음
```

---

### 명령-쿼리 분리 원칙 (CQS)

> **Command-Query Separation Principle**

```
모든 오퍼레이션은 다음 중 하나여야 한다:

1. 명령 (Command)
   - 상태를 변경한다
   - 값을 반환하지 않는다
   - void 반환

2. 쿼리 (Query)
   - 상태를 변경하지 않는다
   - 값을 반환한다
   - 읽기 전용

❌ 명령이면서 동시에 쿼리여서는 안 된다!
```

---

### 반복 일정 예제: 잘못된 설계

> 📂 **코드**: [`Event.java` (Step 01)](https://github.com/eternity-oop/object/blob/master/chapter06/src/main/java/org/eternity/event/step01/Event.java)

```java
public class Event {
    private String subject;
    private LocalDateTime from;
    private Duration duration;
    
    // ❌ 명령과 쿼리가 섞여있음!
    public boolean isSatisfied(RecurringSchedule schedule) {
        if (from.getDayOfWeek() != schedule.getDayOfWeek() ||
            !from.toLocalTime().equals(schedule.getFrom()) ||
            !duration.equals(schedule.getDuration())) {
            
            // ❌ 쿼리인 척 하지만 상태를 변경!
            reschedule(schedule);
            return false;
        }
        
        return true;
    }
    
    private void reschedule(RecurringSchedule schedule) {
        // 상태 변경!
        from = LocalDateTime.of(
            from.toLocalDate().plusDays(daysDistance(schedule)),
            schedule.getFrom()
        );
        duration = schedule.getDuration();
    }
}
```

---

#### 버그 시나리오

```java
// 반복 일정: 매주 수요일 10:30~11:00
RecurringSchedule schedule = new RecurringSchedule(
    "회의", 
    DayOfWeek.WEDNESDAY,
    LocalTime.of(10, 30),
    Duration.ofMinutes(30)
);

// 이벤트: 2019년 5월 8일 (수요일) 10:30~11:00
Event meeting = new Event(
    "회의",
    LocalDateTime.of(2019, 5, 8, 10, 30),
    Duration.ofMinutes(30)
);

// 🐛 버그 발생!
assert meeting.isSatisfied(schedule) == true;   // ✅ 통과 (조건 만족)
assert meeting.isSatisfied(schedule) == true;   // ❌ 실패!
```

**왜 실패했는가?**

```
첫 번째 호출:
1. 조건 체크: 일치함
2. return true
3. 상태 변경 없음

두 번째 호출:
1. 조건 체크: 여전히 일치함
2. return true
3. 상태 변경 없음

아, 잠깐... 이건 정상 동작 아닌가?

실제로는:
- 이벤트 날짜가 다른 경우를 생각해보자!
```

**실제 버그 시나리오**:

```java
// 이벤트: 2019년 5월 9일 (목요일) 10:30~11:00
Event meeting = new Event(
    "회의",
    LocalDateTime.of(2019, 5, 9, 10, 30),  // 목요일!
    Duration.ofMinutes(30)
);

// 첫 번째 호출
boolean result1 = meeting.isSatisfied(schedule);
// 1. 조건 체크: 불일치 (목요일 != 수요일)
// 2. reschedule() 호출 → 수요일로 변경!
// 3. return false
// result1 = false ✅

// 두 번째 호출
boolean result2 = meeting.isSatisfied(schedule);
// 1. 조건 체크: 일치! (이미 수요일로 변경됨)
// 2. reschedule() 호출 안 됨
// 3. return true
// result2 = true ✅

// 🐛 같은 메서드, 다른 결과!
```

**문제의 본질**:

```
isSatisfied()는:
- 이름: "조건을 만족하는가?" (쿼리처럼 보임)
- 동작: 상태를 변경함 (명령처럼 동작)

→ 명령과 쿼리가 섞임
→ 부수효과 발생
→ 예측 불가능
→ 버그 양산
```

---

### 반복 일정 예제: 올바른 설계

> 📂 **코드**: [`Event.java` (Step 02)](https://github.com/eternity-oop/object/blob/master/chapter06/src/main/java/org/eternity/event/step02/Event.java)

```java
public class Event {
    private String subject;
    private LocalDateTime from;
    private Duration duration;
    
    // ✅ 쿼리: 상태 변경 없음
    public boolean isSatisfied(RecurringSchedule schedule) {
        if (from.getDayOfWeek() != schedule.getDayOfWeek() ||
            !from.toLocalTime().equals(schedule.getFrom()) ||
            !duration.equals(schedule.getDuration())) {
            return false;
        }
        return true;
    }
    
    // ✅ 명령: 명확하게 분리
    public void reschedule(RecurringSchedule schedule) {
        from = LocalDateTime.of(
            from.toLocalDate().plusDays(daysDistance(schedule)),
            schedule.getFrom()
        );
        duration = schedule.getDuration();
    }
    
    private long daysDistance(RecurringSchedule schedule) {
        return schedule.getDayOfWeek().getValue() - 
               from.getDayOfWeek().getValue();
    }
}
```

---

#### 사용 코드

```java
// ✅ 의도가 명확
Event meeting = new Event(
    "회의",
    LocalDateTime.of(2019, 5, 9, 10, 30),  // 목요일
    Duration.ofMinutes(30)
);

RecurringSchedule schedule = new RecurringSchedule(
    "회의",
    DayOfWeek.WEDNESDAY,
    LocalTime.of(10, 30),
    Duration.ofMinutes(30)
);

// 조건 확인 (쿼리)
if (!meeting.isSatisfied(schedule)) {
    // 일정 변경 (명령)
    meeting.reschedule(schedule);
}

// 몇 번을 호출해도 같은 결과
assert meeting.isSatisfied(schedule) == true;
assert meeting.isSatisfied(schedule) == true;  // ✅ 통과!
```

---

### 명령-쿼리 분리의 장점

**1. 예측 가능성**

```java
// 쿼리는 몇 번 호출해도 안전
boolean result1 = meeting.isSatisfied(schedule);
boolean result2 = meeting.isSatisfied(schedule);
boolean result3 = meeting.isSatisfied(schedule);
// result1 == result2 == result3 ✅

// 순서를 바꿔도 결과가 같음
boolean a = meeting.isSatisfied(schedule);
meeting.getDuration();
boolean b = meeting.isSatisfied(schedule);
// a == b ✅
```

**2. 디버깅 용이**

```java
// ❌ 명령과 쿼리가 섞인 경우
boolean result = meeting.isSatisfied(schedule);  // 상태 변경!
// 디버거로 확인하려고 호출 → 상태가 바뀜 → 버그!

// ✅ 분리된 경우
boolean result = meeting.isSatisfied(schedule);  // 상태 변경 없음
// 디버거로 몇 번을 확인해도 안전
```

**3. 코드 이해**

```java
// ✅ 이름만 봐도 알 수 있음
boolean isValid = order.isValid();        // 쿼리
order.cancel();                           // 명령
Money total = order.calculateTotal();     // 쿼리
order.confirm();                          // 명령
```

---

### 참조 투명성 (Referential Transparency)

> **함수형 프로그래밍의 핵심 개념**

```
참조 투명성이란?
"표현식 e를 e의 결과값으로 대체해도 프로그램의 의미가 변하지 않는 것"

예시:
int x = add(1, 2);      // x = 3
int y = add(1, 2);      // y = 3
int z = x + y;          // z = 6

대체:
int z = add(1, 2) + add(1, 2);  // z = 6
int z = 3 + 3;                   // z = 6

결과가 같음! → 참조 투명
```

---

#### 수학에서의 함수

```
f(x) = x + 1

f(1) = 2
f(1) + f(1) = 4
f(1) * 2 = 4

f(1)을 2로 바꿔도 결과는 같음:
2 + 2 = 4
2 * 2 = 4

수학의 함수는 참조 투명!
```

---

#### 명령-쿼리 분리와의 관계

```
쿼리:
- 상태를 변경하지 않음
- 같은 입력 → 같은 출력
- 참조 투명성 만족

명령:
- 상태를 변경함
- 참조 투명성 불만족
- 하지만 명확하게 분리되어 있어 안전

혼합:
- 언제 상태가 바뀌는지 모름
- 참조 투명성 파괴
- 예측 불가능
```

---

#### 불변성 (Immutability)

> **참조 투명성을 달성하는 강력한 방법**

```java
// ✅ 불변 객체
public final class Money {
    private final BigDecimal amount;
    
    public Money plus(Money other) {
        // 자신을 변경하지 않고 새 객체 반환
        return new Money(amount.add(other.amount));
    }
    
    public Money minus(Money other) {
        return new Money(amount.subtract(other.amount));
    }
}

// 사용
Money m1 = Money.wons(1000);
Money m2 = m1.plus(Money.wons(500));  // m1은 변하지 않음

// 참조 투명성
Money result1 = m1.plus(Money.wons(500)).times(2);
Money result2 = m1.plus(Money.wons(500)).times(2);
// result1 == result2 항상 성립
```

---

### 명령-쿼리 분리 실전 가이드

#### 명령 메서드 (Commands)

```java
// ✅ 좋은 명령 메서드
public void cancel() {
    this.status = OrderStatus.CANCELLED;
    // return 없음
}

public void addItem(Item item) {
    this.items.add(item);
    // return 없음
}

// ⚠️ 예외: 편의를 위한 반환
public Order placeOrder() {
    this.status = OrderStatus.PLACED;
    return this;  // 메서드 체이닝을 위한 this 반환은 OK
}
```

---

#### 쿼리 메서드 (Queries)

```java
// ✅ 좋은 쿼리 메서드
public boolean isEmpty() {
    return items.isEmpty();
    // 상태 변경 없음
}

public Money calculateTotal() {
    return items.stream()
                .map(Item::getPrice)
                .reduce(Money.ZERO, Money::plus);
    // 상태 변경 없음
}

// ❌ 나쁜 쿼리 메서드
public int getSize() {
    this.accessCount++;  // 상태 변경!
    return items.size();
}
```

---

#### 예외 상황

```
때로는 명령과 쿼리를 섞어야 할 때도 있다:

1. Stack.pop()
   - 요소를 제거하고 (명령)
   - 그 요소를 반환 (쿼리)
   - 하지만: 이름이 명확하므로 OK

2. 원자적 연산 (Atomic Operations)
   - compareAndSet()
   - 비교와 설정을 원자적으로 수행
   - 동시성 제어를 위해 필요

판단 기준:
- 이름이 명확한가?
- 부수효과가 예상 가능한가?
- 다른 방법이 없는가?
```

---

## 🎓 05. 책임에 초점을 맞춰라

> **"모든 원칙의 근본은 책임이다"**

### 메시지를 먼저 선택하면

```
메시지 우선 → 모든 원칙이 자연스럽게 따라온다

1. 디미터 법칙
   메시지를 먼저 선택
   → 수신자의 내부를 모르는 상태에서 설계
   → 내부 구조에 결합될 수 없음

2. 묻지 말고 시켜라
   클라이언트 관점에서 메시지 선택
   → 필요한 것을 표현
   → 묻지 않고 시킴

3. 의도를 드러내는 인터페이스
   메시지 = 클라이언트의 의도
   → 의도가 이름에 반영됨

4. 명령-쿼리 분리
   협력 속에서 역할 고민
   → 예측 가능성 필요
   → 명령과 쿼리 분리
```

---

### 책임 주도 설계 복습

```
1. 시스템이 제공할 기능 파악
   → 시스템의 책임

2. 책임을 더 작은 책임으로 분할
   → 메시지 정의

3. 메시지를 처리할 객체 선택
   → 정보 전문가에게 할당

4. 협력 중 도움 필요?
   → 새로운 메시지 정의
   → 다른 객체에게 전송

5. 반복
   → 협력 완성
```

---

### 실전 적용 순서

```
Step 1: 메시지 먼저
"영화 요금을 계산하고 싶어"
→ calculateMovieFee()

Step 2: 수신자 선택
"누가 이 정보를 알고 있지?"
→ Movie가 정보 전문가

Step 3: 메서드 이름 결정
"클라이언트가 원하는 것은?"
→ 영화 요금 계산 (의도)

Step 4: 명령인가 쿼리인가?
"상태가 바뀌나?"
→ 아니오 → 쿼리

Step 5: 협력 필요한가?
"Movie 혼자 할 수 있나?"
→ 아니오 → DiscountCondition에게 메시지
```

---

## 💡 핵심 정리

### 4가지 원칙 요약

| 원칙 | 핵심 질문 | 답변 |
|------|-----------|------|
| **디미터 법칙** | 누구와 얘기할까? | 가까운 이웃만 |
| **묻지 말고 시켜라** | 어떻게 요청할까? | 묻지 말고 시켜라 |
| **의도를 드러내는 인터페이스** | 무엇을 이름으로? | 의도를 표현 |
| **명령-쿼리 분리** | 어떤 종류인가? | 명령 또는 쿼리 |

---

### 원칙 적용 체크리스트

```
□ 메시지를 먼저 선택했는가?
□ 객체의 내부 구조가 드러나는가? (디미터)
□ 객체의 상태를 묻고 판단하는가? (Tell, Don't Ask)
□ 메서드 이름이 구현을 드러내는가? (의도)
□ 명령과 쿼리가 섞여있는가? (CQS)
□ 원칙을 맹신하고 있지는 않은가? (트레이드오프)
```

---

### 설계 의사결정 가이드

```
상황별 판단:

1. 기차 충돌 발견
   → 위임 메서드 추가
   → 응집도 확인
   → 가치 있으면 적용

2. getter 사용 고민
   → 묻는 대상이 객체인가 자료구조인가?
   → 객체면 시키기
   → 자료구조면 묻기 OK

3. 메서드 이름 고민
   → "매우 다른 두 번째 구현" 상상
   → 같은 이름을 붙일 수 있나?
   → 가능하면 추상적 이름 선택

4. 명령과 쿼리 구분
   → 상태를 변경하는가?
   → YES면 명령, void 반환
   → NO면 쿼리, 값 반환
```

---

## ❓ 자주 하는 질문

### Q1. 디미터 법칙을 지키면 위임 메서드가 너무 많아지는데?

**A**: 맞습니다. 그래서 **트레이드오프**가 필요합니다.

```
판단 기준:

1. 위임 메서드가 객체의 응집도를 높이는가?
   YES → 추가
   NO → 고민

2. 해당 메서드가 퍼블릭 인터페이스의 일부로 적합한가?
   YES → 추가
   NO → 재고려

3. 클라이언트 코드가 더 깔끔해지는가?
   YES → 추가
   NO → 현재 상태 유지

예시:
// Person.getAddress().getStreet()
// vs
// Person.getStreet()

Person이 주소를 관리한다면 → getStreet() 추가 가치 있음
Person이 단순히 Address를 참조한다면 → 굳이 필요 없을 수 있음
```

---

### Q2. getter는 항상 나쁜가요?

**A**: 아닙니다. **문맥에 따라 다릅니다.**

```
getter가 나쁜 경우:
❌ 객체의 상태를 기반으로 외부에서 판단
❌ getter 결과로 객체 상태 변경
❌ 객체의 책임을 빼앗음

getter가 괜찮은 경우:
✅ 단순 조회 (UI 표시)
✅ DTO 변환
✅ 불변 값 객체
✅ 데이터베이스 저장

예시:
// ❌ 나쁜 사용
if (order.getStatus() == OrderStatus.PLACED) {
    order.setStatus(OrderStatus.CONFIRMED);
}

// ✅ 좋은 사용
orderView.showStatus(order.getStatus());

// ✅ 더 좋은 설계
order.confirm();  // 상태 전환 로직은 Order 내부에
```

---

### Q3. 명령-쿼리 분리를 지키면 코드가 길어지는데?

**A**: 때로는 **편의성**과 **원칙** 사이의 균형이 필요합니다.

```
허용 가능한 예외:

1. 메서드 체이닝
   return this;  // 명령이지만 this 반환

2. Builder 패턴
   builder.name("홍길동")   // 상태 변경
          .age(30)          // 상태 변경
          .build();         // 객체 반환

3. Stack, Queue
   T pop()  // 제거 + 반환

판단 기준:
- 이름이 명확한가?
- 부수효과가 예상 가능한가?
- 널리 쓰이는 관례인가?
```

---

### Q4. 모든 원칙을 다 지켜야 하나요?

**A**: **상황에 따라 판단**하세요.

```
원칙은 도구입니다:
- 맹신하지 마세요
- 문맥을 고려하세요
- 트레이드오프하세요

좋은 설계자:
1. 원칙을 이해한다
2. 언제 적용할지 안다
3. 언제 깰지도 안다
4. 그 이유를 설명할 수 있다

"원칙을 지키는 것이 목적이 아니라
 좋은 설계를 하는 것이 목적"
```

---

## 🚀 실전 적용 가이드

### Before: 디미터 법칙 위반 코드

```java
public class OrderService {
    public void processOrder(Order order) {
        // ❌ 기차 충돌
        Customer customer = order.getCustomer();
        Address address = customer.getAddress();
        String city = address.getCity();
        
        if (city.equals("서울")) {
            // 서울 배송
        }
        
        // ❌ 상태를 묻고 판단
        if (order.getItems().size() > 5) {
            order.setDiscountRate(0.1);
        }
        
        // ❌ 직접 계산
        BigDecimal total = BigDecimal.ZERO;
        for (OrderItem item : order.getItems()) {
            total = total.add(
                item.getPrice().multiply(
                    new BigDecimal(item.getQuantity())
                )
            );
        }
    }
}
```

---

### After: 원칙 적용 코드

```java
public class OrderService {
    public void processOrder(Order order) {
        // ✅ 묻지 말고 시켜라
        if (order.isSeoulDelivery()) {
            // 서울 배송
        }
        
        // ✅ 객체에게 책임 부여
        order.applyBulkDiscount();
        
        // ✅ 계산 책임은 Order에게
        Money total = order.calculateTotal();
    }
}

public class Order {
    private Customer customer;
    private List<OrderItem> items;
    private BigDecimal discountRate = BigDecimal.ZERO;
    
    // ✅ 쿼리: 상태 변경 없음
    public boolean isSeoulDelivery() {
        return customer.isSeoulResident();
    }
    
    // ✅ 명령: 명확한 이름, void 반환
    public void applyBulkDiscount() {
        if (items.size() > 5) {
            this.discountRate = new BigDecimal("0.1");
        }
    }
    
    // ✅ 쿼리: 계산만 수행
    public Money calculateTotal() {
        Money subtotal = items.stream()
            .map(OrderItem::calculatePrice)
            .reduce(Money.ZERO, Money::plus);
            
        return subtotal.minus(
            subtotal.times(discountRate.doubleValue())
        );
    }
}

public class Customer {
    private Address address;
    
    // ✅ 의도를 드러내는 이름
    public boolean isSeoulResident() {
        return address.isInCity("서울");
    }
}

public class OrderItem {
    private Money price;
    private int quantity;
    
    // ✅ 자신의 책임
    public Money calculatePrice() {
        return price.times(quantity);
    }
}
```

---

## 📖 최종 요약

### 핵심 메시지

```
1. 클래스가 아닌 메시지 중심으로 설계하라
2. 퍼블릭 인터페이스의 품질이 설계의 품질이다
3. 4가지 원칙을 이해하고 적용하라
4. 원칙을 맹신하지 말고 트레이드오프하라
5. 책임에 초점을 맞추면 원칙은 자연스럽게 따라온다
```

---

### 다음 Chapter 예고

```
Chapter 07: 객체 분해

- 프로시저 추상화와 데이터 추상화
- 프로시저 중심 vs 데이터 중심 vs 책임 중심
- 모듈의 세 가지 목적
- 정보 은닉과 모듈
- 추상 데이터 타입과 클래스
```

---

## 🔬 실행 흐름 상세 추적: Theater 시스템

### Step 01 → Step 03 변화 과정 완전 분석

코드가 어떻게 진화하는지 단계별로 추적해봅시다.

---

#### 🎬 Step 01: 절차적 설계 (나쁜 예)

> 📂 **코드**: [`Theater.java` (Step 01)](https://github.com/eternity-oop/object/blob/master/chapter06/src/main/java/org/eternity/theater/step01/Theater.java)

```
초기화 상태:
┌─────────────┐
│   Theater   │
│  - ticketSeller: TicketSeller
└─────────────┘
       │
       ↓
┌─────────────┐
│ TicketSeller│
│  - ticketOffice: TicketOffice
└─────────────┘
       │
       ↓
┌─────────────┐
│TicketOffice │
│  - amount: 10000L
│  - tickets: [Ticket1, Ticket2, ...]
└─────────────┘

관객 도착:
┌─────────────┐
│  Audience   │
│  - bag: Bag
└─────────────┘
       │
       ↓
┌─────────────┐
│     Bag     │
│  - amount: 5000L
│  - invitation: null (or Invitation)
│  - ticket: null
└─────────────┘
```

**실행 추적**:

```
theater.enter(audience) 호출
    ↓
┌──────────────────────────────────────────────────────────┐
│ 1. Theater.enter(Audience audience) 시작                  │
└──────────────────────────────────────────────────────────┘
    ↓
    if (audience.getBag().hasInvitation())  // ❌ 묻기 시작
    
    ├─ audience.getBag() 호출
    │  └─ Bag 반환 (Audience 내부 노출)
    │
    └─ bag.hasInvitation() 호출
       └─ boolean 반환 (Bag 내부 로직 사용)
    
    ↓
┌──────────────────────────────────────────────────────────┐
│ 2. if문 분기                                               │
└──────────────────────────────────────────────────────────┘
    
    Case A: 초대권 있는 경우
    ├─ Ticket ticket = ticketSeller.getTicketOffice().getTicket()
    │  ├─ ticketSeller.getTicketOffice() → TicketOffice 반환
    │  └─ ticketOffice.getTicket() → Ticket 반환
    │
    └─ audience.getBag().setTicket(ticket)
       ├─ audience.getBag() → Bag 반환
       └─ bag.setTicket(ticket) → Ticket 설정
    
    Case B: 초대권 없는 경우
    ├─ Ticket ticket = ticketSeller.getTicketOffice().getTicket()
    │  └─ (위와 동일)
    │
    ├─ audience.getBag().minusAmount(ticket.getFee())
    │  ├─ audience.getBag() → Bag 반환
    │  ├─ ticket.getFee() → Long 반환
    │  └─ bag.minusAmount(fee) → 금액 차감
    │
    ├─ ticketSeller.getTicketOffice().plusAmount(ticket.getFee())
    │  ├─ ticketSeller.getTicketOffice() → TicketOffice 반환
    │  ├─ ticket.getFee() → Long 반환
    │  └─ ticketOffice.plusAmount(fee) → 금액 추가
    │
    └─ audience.getBag().setTicket(ticket)
       └─ (위와 동일)

┌──────────────────────────────────────────────────────────┐
│ 3. 종료                                                   │
└──────────────────────────────────────────────────────────┘

총 메서드 호출 수: 6~10회
총 객체 접근: Audience, Bag, TicketSeller, TicketOffice, Ticket
```

**문제점 카운트**:

```
1. getBag() 호출 횟수: 3~4회
2. getTicketOffice() 호출 횟수: 2~3회
3. 내부 구조 노출: Bag, TicketOffice
4. 책임 위치: 모두 Theater에 집중
5. Theater가 아는 것: 5개 클래스의 내부 구조
```

---

#### 🎬 Step 02: 중간 단계

> 📂 **코드**: [`Theater.java` (Step 02)](https://github.com/eternity-oop/object/blob/master/chapter06/src/main/java/org/eternity/theater/step02/Theater.java)

```java
// Theater
public void enter(Audience audience) {
    ticketSeller.setTicket(audience);  // ✅ 1단계 개선
}

// TicketSeller
public void setTicket(Audience audience) {
    ticketOffice.plusAmount(
        audience.setTicket(ticketOffice.getTicket())
    );
}

// Audience
public Long setTicket(Ticket ticket) {
    return bag.setTicket(ticket);  // ✅ Bag에게 위임
}

// Bag
public Long setTicket(Ticket ticket) {
    if (hasInvitation()) {
        this.ticket = ticket;
        return 0L;
    } else {
        this.ticket = ticket;
        minusAmount(ticket.getFee());
        return ticket.getFee();
    }
}
```

**개선 포인트**:

```
✅ Theater가 Bag에 직접 접근하지 않음
✅ Audience가 Bag의 세부사항 처리
⚠️ 하지만 메서드 이름이 구현 중심 (setTicket)
```

---

#### 🎬 Step 03: 최종 설계 (좋은 예)

> 📂 **코드**: [`Theater.java` (Step 03)](https://github.com/eternity-oop/object/blob/master/chapter06/src/main/java/org/eternity/theater/step03/Theater.java)

```
실행 추적:

theater.enter(audience) 호출
    ↓
┌──────────────────────────────────────────────────────────┐
│ 1. Theater.enter(Audience audience)                      │
│    ticketSeller.sellTo(audience);                        │
└──────────────────────────────────────────────────────────┘
    ↓
┌──────────────────────────────────────────────────────────┐
│ 2. TicketSeller.sellTo(Audience audience)                │
│    Long amount = audience.buy(ticketOffice.getTicket()); │
│    ticketOffice.plusAmount(amount);                      │
└──────────────────────────────────────────────────────────┘
    ↓                           ↓
    │                       ticketOffice.getTicket()
    │                           ↓
    │                       [Ticket 반환]
    ↓
┌──────────────────────────────────────────────────────────┐
│ 3. Audience.buy(Ticket ticket)                           │
│    return bag.hold(ticket);                              │
└──────────────────────────────────────────────────────────┘
    ↓
┌──────────────────────────────────────────────────────────┐
│ 4. Bag.hold(Ticket ticket)                               │
│    if (hasInvitation()) {                                │
│        this.ticket = ticket;                             │
│        return 0L;                                        │
│    } else {                                              │
│        this.ticket = ticket;                             │
│        minusAmount(ticket.getFee());                     │
│        return ticket.getFee();                           │
│    }                                                     │
└──────────────────────────────────────────────────────────┘
    ↓
    [Long 반환] → Audience → TicketSeller
    ↓
┌──────────────────────────────────────────────────────────┐
│ 5. TicketSeller.sellTo 계속                               │
│    ticketOffice.plusAmount(amount);                      │
└──────────────────────────────────────────────────────────┘
    ↓
    [종료]

총 메서드 호출 수: 5회
총 public 메서드: 4개 (sellTo, buy, hold, plusAmount)
```

---

#### 📊 Step별 비교표

| 측면 | Step 01 | Step 02 | Step 03 |
|------|---------|---------|---------|
| **Theater 코드** | 15줄 | 3줄 | 3줄 |
| **메서드 호출 수** | 6~10회 | 4~5회 | 5회 |
| **getBag() 호출** | 3~4회 | 0회 | 0회 |
| **getTicketOffice()** | 2~3회 | 1회 | 1회 |
| **의사결정 위치** | Theater | Theater → TicketSeller | 각 객체 |
| **메서드 이름** | get/set | setTicket | sellTo, buy, hold |
| **디미터 법칙** | ❌ 위반 | ⚠️ 부분 준수 | ✅ 준수 |
| **Tell, Don't Ask** | ❌ 위반 | ⚠️ 부분 준수 | ✅ 준수 |
| **의도 표현** | ❌ 불명확 | ⚠️ 구현 중심 | ✅ 명확 |

---

### 🔍 메시지 흐름 시각화

**Step 01의 메시지 흐름**:

```
[Theater]
    │
    ├──→ audience.getBag()
    │        └──→ [Bag 반환]
    │
    ├──→ bag.hasInvitation()
    │        └──→ [boolean 반환]
    │
    ├──→ ticketSeller.getTicketOffice()
    │        └──→ [TicketOffice 반환]
    │
    ├──→ ticketOffice.getTicket()
    │        └──→ [Ticket 반환]
    │
    ├──→ audience.getBag()
    │        └──→ [Bag 반환]
    │
    └──→ bag.setTicket(ticket)

기차처럼 길게 연결된 호출 체인!
```

**Step 03의 메시지 흐름**:

```
[Theater]
    │
    └──→ ticketSeller.sellTo(audience)
             │
             ├──→ audience.buy(ticket)
             │        │
             │        └──→ bag.hold(ticket)
             │                 └──→ [Long 반환]
             │
             └──→ ticketOffice.plusAmount(amount)

깔끔한 위임 체인!
```

---

## 🏗️ 설계 원칙 적용 Before & After

### 예제 1: 주문 시스템

#### Before: 모든 원칙 위반

```java
public class OrderProcessor {
    public void process(Order order) {
        // ❌ 디미터 법칙 위반 (기차 충돌)
        String city = order.getCustomer()
                          .getAddress()
                          .getCity();
        
        // ❌ 묻지 말고 시켜라 위반
        if (order.getStatus() == OrderStatus.PENDING) {
            if (order.getItems().size() > 0) {
                // ❌ 상태를 묻고 직접 변경
                order.setStatus(OrderStatus.CONFIRMED);
                
                BigDecimal total = BigDecimal.ZERO;
                for (OrderItem item : order.getItems()) {
                    // ❌ 구현 노출, 직접 계산
                    total = total.add(
                        item.getPrice()
                            .multiply(new BigDecimal(item.getQuantity()))
                    );
                }
                order.setTotalAmount(total);
            }
        }
        
        // ❌ 명령-쿼리 분리 위반
        boolean wasProcessed = order.markAsProcessed();  // 상태 변경 + 반환
        if (wasProcessed) {
            sendNotification(order);
        }
    }
}
```

#### After: 모든 원칙 적용

```java
public class OrderProcessor {
    public void process(Order order) {
        // ✅ 디미터 법칙: Order에게만 메시지
        if (order.isSeoulDelivery()) {
            applySeoulDiscount(order);
        }
        
        // ✅ 묻지 말고 시켜라: Order가 스스로 처리
        order.confirm();
        
        // ✅ 명령-쿼리 분리
        if (order.isProcessed()) {  // 쿼리
            order.markAsProcessed();  // 명령
            sendNotification(order);
        }
    }
}

public class Order {
    private Customer customer;
    private List<OrderItem> items;
    private OrderStatus status;
    private Money totalAmount;
    
    // ✅ 의도를 드러내는 인터페이스
    public boolean isSeoulDelivery() {
        return customer.livesInSeoul();
    }
    
    // ✅ 명령: void 반환
    public void confirm() {
        validateConfirmable();
        this.status = OrderStatus.CONFIRMED;
        this.totalAmount = calculateTotal();
    }
    
    // ✅ 쿼리: 상태 변경 없음
    public boolean isProcessed() {
        return status == OrderStatus.PROCESSED;
    }
    
    // ✅ 명령: 명확하게 분리
    public void markAsProcessed() {
        if (status != OrderStatus.CONFIRMED) {
            throw new IllegalStateException("주문이 확정되지 않았습니다");
        }
        this.status = OrderStatus.PROCESSED;
    }
    
    // ✅ 정보 전문가: Order가 자신의 총액 계산
    private Money calculateTotal() {
        return items.stream()
                   .map(OrderItem::calculateAmount)
                   .reduce(Money.ZERO, Money::plus);
    }
}

public class Customer {
    private Address address;
    
    // ✅ 의도를 드러내는 메서드
    public boolean livesInSeoul() {
        return address.isInCity("서울");
    }
}

public class OrderItem {
    private Money price;
    private int quantity;
    
    // ✅ 정보 전문가: 자신의 금액 계산
    public Money calculateAmount() {
        return price.times(quantity);
    }
}
```

---

### 예제 2: 게시판 시스템

#### Before: 절차적 설계

```java
public class PostService {
    public void updatePost(Long postId, PostUpdateRequest request) {
        Post post = postRepository.findById(postId)
            .orElseThrow(() -> new PostNotFoundException());
        
        // ❌ 상태를 묻고 판단
        if (post.getAuthor().getId().equals(request.getUserId())) {
            // ❌ 직접 상태 변경
            post.setTitle(request.getTitle());
            post.setContent(request.getContent());
            post.setUpdatedAt(LocalDateTime.now());
            
            // ❌ 비즈니스 로직이 서비스에
            if (post.getContent().length() > 10000) {
                throw new ContentTooLongException();
            }
            
            postRepository.save(post);
        } else {
            throw new UnauthorizedException();
        }
    }
}
```

#### After: 객체지향 설계

```java
public class PostService {
    public void updatePost(Long postId, PostUpdateRequest request) {
        Post post = postRepository.findById(postId)
            .orElseThrow(() -> new PostNotFoundException());
        
        // ✅ 묻지 말고 시켜라
        post.update(
            request.getUserId(),
            request.getTitle(),
            request.getContent()
        );
        
        postRepository.save(post);
    }
}

public class Post {
    private Long id;
    private User author;
    private String title;
    private String content;
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;
    
    // ✅ 모든 로직이 Post 내부에
    public void update(Long userId, String newTitle, String newContent) {
        // ✅ 권한 검증도 Post가 수행
        validateAuthor(userId);
        
        // ✅ 비즈니스 규칙도 Post가 관리
        validateContent(newContent);
        
        this.title = newTitle;
        this.content = newContent;
        this.updatedAt = LocalDateTime.now();
    }
    
    private void validateAuthor(Long userId) {
        if (!author.hasId(userId)) {
            throw new UnauthorizedException("작성자만 수정할 수 있습니다");
        }
    }
    
    private void validateContent(String content) {
        if (content.length() > 10000) {
            throw new ContentTooLongException("내용이 너무 깁니다");
        }
    }
}

public class User {
    private Long id;
    
    // ✅ 의도를 드러내는 인터페이스
    public boolean hasId(Long id) {
        return this.id.equals(id);
    }
}
```

---

## 🎯 패턴별 실전 적용 가이드

### 1. 디미터 법칙 적용하기

#### Step 1: 기차 충돌 찾기

```java
// 코드 리뷰 체크리스트
□ 연속된 getter 호출이 있는가?
□ 3단계 이상 접근하는가?
□ 중간 객체의 타입을 알아야 하는가?
```

#### Step 2: 위임 메서드 추가

```java
// Before
String street = person.getAddress().getStreet();

// After
String street = person.getStreet();

// Person에 추가
public String getStreet() {
    return address.getStreet();
}
```

#### Step 3: 응집도 확인

```
질문: "이 메서드가 Person의 책임인가?"
- YES → 유지
- NO → 재고려
```

---

### 2. 묻지 말고 시켜라 적용하기

#### Step 1: 상태 기반 로직 찾기

```java
// 패턴 찾기
if (object.getState() == SOME_STATE) {
    object.setState(NEW_STATE);
}
```

#### Step 2: 메서드 추출

```java
// Before
if (order.getStatus() == OrderStatus.PENDING) {
    order.setStatus(OrderStatus.CONFIRMED);
}

// After
order.confirm();
```

#### Step 3: 검증 로직 포함

```java
public void confirm() {
    if (status != OrderStatus.PENDING) {
        throw new IllegalStateException(
            "대기 중인 주문만 확정할 수 있습니다"
        );
    }
    this.status = OrderStatus.CONFIRMED;
}
```

---

### 3. 의도를 드러내는 인터페이스 만들기

#### Step 1: 구현 중심 이름 찾기

```java
// ❌ 구현이 드러나는 이름
public boolean isSatisfiedByPeriod(Screening screening)
public boolean checkSequenceCondition(Screening screening)
public Money calculateAmountDiscount()
public Money calculatePercentDiscount()
```

#### Step 2: 의도 중심으로 변경

```java
// ✅ 의도가 드러나는 이름
public boolean isSatisfiedBy(Screening screening)
public boolean isSatisfiedBy(Screening screening)
public Money calculateDiscountAmount()
public Money calculateDiscountAmount()
```

#### Step 3: 인터페이스로 통합

```java
public interface DiscountCondition {
    boolean isSatisfiedBy(Screening screening);
}

public interface DiscountPolicy {
    Money calculateDiscountAmount();
}
```

---

### 4. 명령-쿼리 분리 적용하기

#### Step 1: 혼합된 메서드 찾기

```java
// ❌ 명령과 쿼리가 섞임
public boolean isSatisfied(RecurringSchedule schedule) {
    if (/* 조건 불만족 */) {
        reschedule(schedule);  // 상태 변경!
        return false;
    }
    return true;
}
```

#### Step 2: 명령과 쿼리 분리

```java
// ✅ 쿼리: 상태 변경 없음
public boolean isSatisfied(RecurringSchedule schedule) {
    return /* 조건 체크만 */;
}

// ✅ 명령: 명확하게 분리
public void reschedule(RecurringSchedule schedule) {
    // 상태 변경
}
```

#### Step 3: 사용 코드 개선

```java
// Before (혼합)
if (!event.isSatisfied(schedule)) {
    // 이미 재스케줄링됨
}

// After (분리)
if (!event.isSatisfied(schedule)) {
    event.reschedule(schedule);
}
```

---

## 💎 고급 주제

### 명령-쿼리 책임 분리 (CQRS)

> **Command Query Responsibility Segregation**

```
CQS의 확장:
- 명령 모델과 쿼리 모델을 아예 분리
- 읽기와 쓰기를 다른 데이터 저장소로

장점:
- 읽기 최적화
- 쓰기 최적화
- 독립적인 확장

단점:
- 복잡도 증가
- 최종 일관성 (Eventual Consistency)
```

---

### 함수형 프로그래밍과의 관계

```
명령-쿼리 분리 → 함수형 프로그래밍

쿼리 = 순수 함수
- 부수효과 없음
- 참조 투명성
- 테스트 쉬움

명령 = 부수효과
- 상태 변경
- I/O 작업
- 외부 시스템 호출

함수형의 이상:
"부수효과를 가능한 한 줄이고
 순수 함수로 핵심 로직 작성"
```

---

## 📚 더 읽어보기

### 추천 자료

```
1. "Object-Oriented Software Construction" - Bertrand Meyer
   → CQS 원칙의 원조

2. "Clean Code" - Robert C. Martin
   → Tell, Don't Ask 상세 설명

3. "The Pragmatic Programmer" - Andy Hunt, Dave Thomas
   → Law of Demeter 실전 적용

4. "Domain-Driven Design" - Eric Evans
   → 도메인 모델에서의 메시지 설계

5. "Growing Object-Oriented Software" - Steve Freeman
   → Mock을 사용한 메시지 중심 TDD
```

---

## 🎓 최종 정리

### 4가지 원칙의 관계도

```
              메시지 우선 설계
                    ↓
        ┌───────────┴───────────┐
        ↓                       ↓
    디미터 법칙              묻지 말고 시켜라
        ↓                       ↓
    구조적 결합도 ↓             책임 명확화
        ↓                       ↓
        └───────────┬───────────┘
                    ↓
           의도를 드러내는 인터페이스
                    ↓
                명령-쿼리 분리
                    ↓
               예측 가능한 협력
```

---

### 설계 품질 체크리스트

```
□ 메시지를 먼저 선택했는가?
□ 기차 충돌이 있는가?
□ getter로 상태를 확인하고 setter로 변경하는가?
□ 메서드 이름이 구현을 드러내는가?
□ 명령 메서드가 값을 반환하는가?
□ 쿼리 메서드가 상태를 변경하는가?
□ 원칙을 맹신하고 있지는 않은가?
□ 트레이드오프를 고려했는가?
```

---

### 마지막 조언

```
설계 원칙은 도구입니다:

1. 이해하라
   - 왜 필요한지
   - 어떤 문제를 해결하는지

2. 적용하라
   - 실전에서 연습
   - 실수하고 배우기

3. 판단하라
   - 언제 적용할지
   - 언제 깰지

4. 설명하라
   - 팀과 공유
   - 의사결정 근거 제시

"완벽한 설계는 없다.
 더 나은 설계만 있을 뿐이다."
```

---

<div align="center">

**[⬆ 목차로 돌아가기](../README.md)**

</div>

<div align="center">

**[← Chapter 05](../chapter05/README.md) | [Chapter 07 →](../chapter07/README.md)**

</div>
