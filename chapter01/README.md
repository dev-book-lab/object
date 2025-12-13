# Chapter 01. 객체, 설계

> *"이론보다 실무가 먼저다"*

## 📌 핵심 개념

- **의존성(Dependency)**: 한 객체가 변경될 때 다른 객체도 함께 변경될 가능성
- **결합도(Coupling)**: 객체 간 의존성의 정도
- **응집도(Cohesion)**: 객체가 자신의 데이터를 스스로 처리하는 정도
- **캡슐화(Encapsulation)**: 객체 내부 구현을 감추고 인터페이스만 노출하는 것
- **자율성(Autonomy)**: 객체가 스스로 결정하고 행동하는 능력

---

## 🎯 학습 목표

1. 절차지향과 객체지향의 근본적인 차이 이해하기
2. 변경에 취약한 설계와 유연한 설계 비교하기
3. 캡슐화를 통한 의존성 관리 방법 익히기
4. 설계 트레이드오프의 필요성 인식하기

---

## 🎭 예제: 티켓 판매 애플리케이션

### 시나리오

소극장에서 관람객에게 티켓을 판매하는 시스템을 설계합니다.

**요구사항**:
- 이벤트 당첨자에게는 초대장(Invitation)을 발송
- 초대장이 있는 관람객은 무료 입장
- 초대장이 없는 관람객은 티켓 구매 후 입장
- 판매원(TicketSeller)은 매표소(TicketOffice)에서 티켓 판매

**등장 객체**:
- `Invitation`: 초대장
- `Ticket`: 티켓
- `Bag`: 관람객의 가방 (초대장, 티켓, 현금 보관)
- `Audience`: 관람객
- `TicketOffice`: 매표소 (티켓 보관, 판매 금액 관리)
- `TicketSeller`: 판매원
- `Theater`: 소극장

---

## 💻 Step 01: 절차지향적 설계

### 코드 구조

```
Theater.enter(Audience)
  ├─ audience.getBag()
  │   ├─ bag.hasInvitation()
  │   ├─ bag.setTicket()
  │   └─ bag.minusAmount()
  └─ ticketSeller.getTicketOffice()
      ├─ ticketOffice.getTicket()
      └─ ticketOffice.plusAmount()
```

### 핵심 코드

```java
public class Theater {
    private TicketSeller ticketSeller;

    public void enter(Audience audience) {
        if (audience.getBag().hasInvitation()) {
            Ticket ticket = ticketSeller.getTicketOffice().getTicket();
            audience.getBag().setTicket(ticket);
        } else {
            Ticket ticket = ticketSeller.getTicketOffice().getTicket();
            audience.getBag().minusAmount(ticket.getFee());
            ticketSeller.getTicketOffice().plusAmount(ticket.getFee());
            audience.getBag().setTicket(ticket);
        }
    }
}
```

### ❌ 문제점 분석

#### 1️⃣ 이해하기 어려운 코드

```
현실에서는: 관람객이 직접 가방을 열어 초대장을 확인하고 티켓을 넣음
코드에서는: Theater가 관람객의 가방을 마음대로 뒤짐
```

- 관람객(`Audience`)과 판매원(`TicketSeller`)이 **수동적인 객체**
- `Theater`가 모든 세부사항을 알고 처리하는 **절차지향적 구조**
- 의인화가 되지 않아 **직관적이지 않음**

#### 2️⃣ 변경에 취약한 코드

**의존성 다이어그램**:

```
Theater
  ├─ TicketSeller
  │   └─ TicketOffice
  │       └─ Ticket
  └─ Audience
      └─ Bag
          ├─ Invitation
          └─ Ticket
```

**문제 상황**:
- 관람객이 가방이 아닌 지갑을 소지한다면? → `Bag` → `Wallet`로 변경 필요
- `Theater.enter()` 메서드도 함께 수정 필요
- 판매원이 매표소가 아닌 은행에 돈을 보관한다면? → 또 수정 필요

**의존성이 높을수록**:
1. 변경 사항이 연쇄적으로 전파됨
2. 버그 발생 가능성 증가
3. 코드 수정 의지 저하

---

## 🔧 Step 02: 객체지향적 설계 (1차 리팩토링)

### 설계 원칙

> **"객체는 자신의 데이터를 스스로 처리해야 한다"**

### 리팩토링 과정

#### Before: Theater가 모든 것을 처리

```java
// Theater가 직접 처리
if (audience.getBag().hasInvitation()) {
    Ticket ticket = ticketSeller.getTicketOffice().getTicket();
    audience.getBag().setTicket(ticket);
}
```

#### After: 각 객체가 자율적으로 처리

```java
// Theater는 인터페이스만 호출
public class Theater {
    public void enter(Audience audience) {
        ticketSeller.sellTo(audience);
    }
}
```

### TicketSeller의 자율성 확보

```java
public class TicketSeller {
    private TicketOffice ticketOffice;

    public void sellTo(Audience audience) {
        ticketOffice.plusAmount(audience.buy(ticketOffice.getTicket()));
    }
}
```

**변경 내용**:
- `getTicketOffice()` 제거 → 내부 구현 숨김
- `sellTo()` 메서드 추가 → 판매 로직 캡슐화

### Audience의 자율성 확보

```java
public class Audience {
    private Bag bag;

    public Long buy(Ticket ticket) {
        if (bag.hasInvitation()) {
            bag.setTicket(ticket);
            return 0L;
        } else {
            bag.setTicket(ticket);
            bag.minusAmount(ticket.getFee());
            return ticket.getFee();
        }
    }
}
```

**변경 내용**:
- `getBag()` 제거 → 내부 구현 숨김
- `buy()` 메서드 추가 → 구매 로직 캡슐화

### ✅ 개선 효과

#### 1️⃣ 의존성 감소

```
Before: Theater → TicketSeller, TicketOffice, Audience, Bag, Ticket
After:  Theater → TicketSeller, Audience
```

#### 2️⃣ 변경의 국지화

- `Bag` 구현 변경 → `Audience`만 수정
- `TicketOffice` 구현 변경 → `TicketSeller`만 수정
- `Theater`는 영향받지 않음

#### 3️⃣ 응집도 향상

각 객체가 자신의 데이터를 직접 처리:
- `Audience`: 티켓 구매 로직
- `TicketSeller`: 티켓 판매 로직
- `Theater`: 입장 처리만 담당

---

## 🎯 Step 03: 추가 캡슐화 (2차 리팩토링)

### Bag의 자율성 확보

#### Before: Audience가 Bag 내부를 직접 제어

```java
public Long buy(Ticket ticket) {
    if (bag.hasInvitation()) {
        bag.setTicket(ticket);
        return 0L;
    } else {
        bag.setTicket(ticket);
        bag.minusAmount(ticket.getFee());
        return ticket.getFee();
    }
}
```

#### After: Bag이 스스로 처리

```java
public class Bag {
    public Long hold(Ticket ticket) {
        if (hasInvitation()) {
            setTicket(ticket);
            return 0L;
        } else {
            setTicket(ticket);
            minusAmount(ticket.getFee());
            return ticket.getFee();
        }
    }

    private boolean hasInvitation() {
        return invitation != null;
    }

    private void setTicket(Ticket ticket) {
        this.ticket = ticket;
    }

    private void minusAmount(Long amount) {
        this.amount -= amount;
    }
}
```

```java
public class Audience {
    public Long buy(Ticket ticket) {
        return bag.hold(ticket);
    }
}
```

**개선 효과**:
- `Bag`의 내부 구현을 `private`으로 숨김
- `Audience`는 `Bag`의 퍼블릭 인터페이스만 사용
- `Bag`의 구현 변경이 `Audience`에 영향 없음

### ⚠️ TicketOffice의 자율성 vs 트레이드오프

#### 시도: TicketOffice도 자율적으로 만들기

```java
public class TicketOffice {
    public void sellTicketTo(Audience audience) {
        plusAmount(audience.buy(getTicket()));
    }
}

public class TicketSeller {
    public void sellTo(Audience audience) {
        ticketOffice.sellTicketTo(audience);
    }
}
```

#### 문제 발생

```
Before: TicketOffice → (의존성 없음)
After:  TicketOffice → Audience
```

**새로운 의존성 발생**:
- `TicketOffice`가 `Audience`에 의존하게 됨
- `TicketOffice`는 `Audience` 없이도 독립적으로 존재할 수 있어야 함
- **자율성**을 높이려다 **결합도**가 증가함

#### 설계 결정

```java
// 최종적으로 Step 02 구현으로 복구
public class TicketSeller {
    public void sellTo(Audience audience) {
        ticketOffice.plusAmount(audience.buy(ticketOffice.getTicket()));
    }
}
```

**트레이드오프**:
- `TicketOffice`의 자율성 < `Audience`와의 결합도 제거
- 현재 상황에서는 **결합도를 낮추는 것이 더 중요**
- 설계는 항상 트레이드오프의 산물

---

## 🤔 핵심 개념 정리

### 1️⃣ 절차지향 vs 객체지향

| 구분 | 절차지향 | 객체지향 |
|------|---------|---------|
| **데이터와 프로세스** | 분리 | 동일 모듈 내 위치 |
| **책임 소재** | 프로세스에 집중 | 데이터를 가진 객체에 분산 |
| **변경 영향** | 여러 곳에 파급 | 해당 객체로 국한 |
| **결합도** | 높음 | 낮음 |
| **응집도** | 낮음 | 높음 |

### 2️⃣ 좋은 설계의 조건

```
좋은 설계 = 오늘 요구하는 기능을 구현 + 내일의 변경을 수용
```

**로버트 마틴의 소프트웨어 모듈 3가지 목적**:
1. **실행 중 제대로 동작**: 모듈의 존재 이유
2. **변경 용이성**: 간단한 작업만으로 변경 가능
3. **코드 가독성**: 특별한 훈련 없이 이해 가능

### 3️⃣ 캡슐화와 응집도

#### 캡슐화 (Encapsulation)

```java
// ❌ 나쁜 예: 내부를 노출
public Bag getBag() {
    return bag;
}

// ✅ 좋은 예: 인터페이스만 노출
public Long buy(Ticket ticket) {
    return bag.hold(ticket);
}
```

**효과**:
- 외부에서 내부 구현을 알 수 없음
- 내부 변경이 외부에 영향 없음

#### 응집도 (Cohesion)

```java
// ❌ 낮은 응집도: Theater가 모든 것을 처리
public void enter(Audience audience) {
    if (audience.getBag().hasInvitation()) {
        // Bag의 일을 Theater가 함
    }
}

// ✅ 높은 응집도: 각자의 일을 스스로 처리
public Long buy(Ticket ticket) {
    return bag.hold(ticket);  // Bag이 자신의 일을 함
}
```

**원칙**:
> 밀접하게 연관된 작업만 수행하고, 연관성 없는 작업은 다른 객체에 위임

### 4️⃣ 책임의 이동 (Shift of Responsibility)

#### 절차지향: 책임이 한 곳에 집중

```
[Theater]
  ↓ 모든 책임
(Audience, TicketSeller, Bag, TicketOffice 모두 수동적)
```

#### 객체지향: 책임이 분산

```
[Theater] → 입장 처리
[TicketSeller] → 티켓 판매
[Audience] → 티켓 구매
[Bag] → 티켓/현금 관리
[TicketOffice] → 티켓 재고 관리
```

**핵심**:
- 각 객체가 자신의 **데이터를 스스로 처리**
- **자율적인 객체들의 협력**으로 기능 구현

---

## 💡 실전 적용 가이드

### 1️⃣ 설계 개선 체크리스트

코드 리뷰 시 다음을 확인하세요:

- [ ] 객체가 자신의 데이터를 직접 처리하는가?
- [ ] 외부에서 객체 내부를 직접 제어하지 않는가?
- [ ] 변경 시 한 곳만 수정하면 되는가?
- [ ] 인터페이스만으로 협력이 가능한가?
- [ ] 객체의 역할이 명확한가?

### 2️⃣ 캡슐화 패턴

```java
// 1. getter 제거
// Before
public Bag getBag() { return bag; }
audience.getBag().setTicket(ticket);

// After  
public Long buy(Ticket ticket) { return bag.hold(ticket); }
audience.buy(ticket);

// 2. 로직을 데이터 소유 객체로 이동
// Before: Theater가 처리
if (audience.getBag().hasInvitation()) { ... }

// After: Bag이 처리
bag.hold(ticket);  // Bag 내부에서 invitation 확인
```

### 3️⃣ 트레이드오프 판단 기준

```
자율성 추구 ──────┬────── 새로운 의존성 발생?
                 │
                 ├─ No  → 리팩토링 진행
                 │
                 └─ Yes → 트레이드오프 검토
                           ↓
                    어느 것이 더 중요한가?
                    - 자율성 증가?
                    - 결합도 감소?
```

### 4️⃣ 의인화(Anthropomorphism)

객체지향에서는 **모든 것을 능동적으로** 만들어야 합니다:

```java
// ❌ 현실: 가방은 수동적
가방.열기();
가방에서_초대장_꺼내기();

// ✅ 객체지향: 가방도 능동적
bag.hold(ticket);  // 가방이 스스로 티켓을 보관함
```

---

## ✨ 핵심 원칙

### 📐 설계 원칙

1. **데이터와 프로세스를 하나의 객체 안에**
   - 절차지향의 분리 구조를 거부
   - 객체가 자신의 데이터를 책임지도록

2. **캡슐화를 통한 의존성 관리**
   - 내부 구현을 숨기고 인터페이스만 노출
   - 불필요한 세부사항을 감춤

3. **협력하는 자율적 객체들**
   - 각 객체는 능동적이고 자율적
   - 메시지를 통해서만 협력

### 🎯 실천 방법

```java
// 1. 객체에게 무엇을 하라고 시키지 말고, 무엇을 해달라고 요청하라
// ❌ Don't
audience.getBag().minusAmount(ticket.getFee());

// ✅ Do
audience.buy(ticket);

// 2. 구현이 아닌 인터페이스에 의존하라
// ❌ Don't
Bag bag = audience.getBag();
bag.setTicket(ticket);

// ✅ Do
audience.buy(ticket);  // Audience의 인터페이스 사용

// 3. 변경을 국지화하라
// 한 객체의 변경이 다른 객체로 전파되지 않도록
```

---

## 📚 함께 읽으면 좋은 내용

- **책임 주도 설계 (Responsibility-Driven Design)**: Chapter 05에서 자세히 다룸
- **SOLID 원칙**: 
  - **SRP** (Single Responsibility): 응집도와 관련
  - **OCP** (Open-Closed): Chapter 09에서 다룸
- **의존성 역전 원칙 (DIP)**: Chapter 08에서 다룸

---

## 🔗 참고 자료

- [원본 코드 - eternity-oop/object](https://github.com/eternity-oop/object/tree/master/chapter01)
- [저자 블로그 - 객체지향의 사실과 오해](https://eternity-object.tistory.com/)

---

## 💭 생각해보기

### Q1. 절차지향이 무조건 나쁜가?

**A**: 아니다. 설계는 트레이드오프의 산물이다.

- 변경이 거의 없고 빠른 구현이 필요하다면 절차지향도 좋은 선택
- 하지만 **변경이 잦은 부분**은 즉시 객체지향적으로 개선해야 함

### Q2. 모든 객체를 자율적으로 만들어야 하는가?

**A**: 상황에 따라 판단해야 한다.

- `TicketOffice`의 경우처럼 자율성 추구가 오히려 결합도를 높일 수 있음
- **변경 가능성**, **의존성**, **복잡도**를 종합적으로 고려

### Q3. 실무에서 어떻게 적용하는가?

**A**: 점진적으로 개선하라.

1. 기존 코드가 절차지향적이라면 인식하기
2. 변경이 잦은 부분부터 리팩토링
3. getter 남발하는 부분 찾아 캡슐화
4. 테스트 코드로 변경 영향 확인

---

<div align="center">

**[⬆ 목차로 돌아가기](../../Downloads/README.md)**

</div>
