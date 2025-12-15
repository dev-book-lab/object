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

### 도메인 모델

```
┌─────────────┐
│   Theater   │  소극장 - 관람객을 입장시킴
└─────────────┘
       │
       ├──────────────────────┐
       │                      │
┌──────▼──────┐      ┌────────▼──────┐
│TicketSeller │      │   Audience    │
│  (판매원)     │      │   (관람객)     │
└─────────────┘      └───────────────┘
       │                      │
       │                      │
┌──────▼──────┐      ┌────────▼──────┐
│TicketOffice │      │     Bag       │
│  (매표소)     │      │   (가방)       │
└─────────────┘      └───────────────┘
       │                      │
       │                      ├──────────┐
┌──────▼──────┐      ┌────────▼──────┐   │
│   Ticket    │      │  Invitation   │   │
│  (티켓)      │      │   (초대장)      │   │
└─────────────┘      └───────────────┘   │
                              │          │
                              └──────────┘
```

---

## 💻 Step 01: 절차지향적 설계 - 모든 것을 Theater가 처리

> 📂 **전체 코드**: [step01 디렉토리](https://github.com/eternity-oop/object/tree/master/chapter01/src/main/java/org/eternity/theater/step01)

이 단계에서는 `Theater`가 모든 로직을 처리합니다. 마치 극장 직원이 관람객의 가방을 직접 뒤지고, 판매원의 매표소를 마음대로 열어보는 것과 같습니다.

### 📦 1-1. 기본 클래스들 (데이터만 보관)

#### [`Invitation.java`](https://github.com/eternity-oop/object/blob/master/chapter01/src/main/java/org/eternity/theater/step01/Invitation.java) - 초대장

```java
public class Invitation {
    private LocalDateTime when;  // 초대 일자
}
```

💡 **설명**: 초대장은 언제 사용 가능한지만 알고 있습니다. 아무런 행동도 하지 않는 단순 데이터 클래스입니다.

---

#### [`Ticket.java`](https://github.com/eternity-oop/object/blob/master/chapter01/src/main/java/org/eternity/theater/step01/Ticket.java) - 티켓

```java
public class Ticket {
    private Long fee;  // 티켓 가격

    public Long getFee() {
        return fee;
    }
}
```

💡 **설명**: 티켓은 가격 정보만 알고 있고, 외부에서 가격을 조회할 수 있게 getter만 제공합니다.

---

#### [`Bag.java`](https://github.com/eternity-oop/object/blob/master/chapter01/src/main/java/org/eternity/theater/step01/Bag.java) - 관람객의 가방

```java
public class Bag {
    private Long amount;           // 현금
    private Invitation invitation; // 초대장
    private Ticket ticket;         // 티켓

    // 생성자 1: 초대장이 없는 관람객
    public Bag(long amount) {
        this(null, amount);
    }

    // 생성자 2: 초대장이 있는 관람객
    public Bag(Invitation invitation, long amount) {
        this.invitation = invitation;
        this.amount = amount;
    }

    // ❌ 문제: 내부 상태를 외부에 그대로 노출
    public boolean hasInvitation() {
        return invitation != null;
    }

    public boolean hasTicket() {
        return ticket != null;
    }

    public void setTicket(Ticket ticket) {
        this.ticket = ticket;
    }

    public void minusAmount(Long amount) {
        this.amount -= amount;
    }

    public void plusAmount(Long amount) {
        this.amount += amount;
    }
}
```

💡 **설명**:
- 가방은 현금, 초대장, 티켓을 보관합니다
- **문제점**: 모든 내부 상태를 변경할 수 있는 메서드들이 public으로 노출되어 있습니다
- 이는 외부에서 가방 내부를 마음대로 조작할 수 있다는 의미입니다

---

#### [`Audience.java`](https://github.com/eternity-oop/object/blob/master/chapter01/src/main/java/org/eternity/theater/step01/Audience.java) - 관람객

```java
public class Audience {
    private Bag bag;  // 관람객이 소지한 가방

    public Audience(Bag bag) {
        this.bag = bag;
    }

    // ❌ 문제: 가방을 외부에 그대로 노출
    public Bag getBag() {
        return bag;
    }
}
```

💡 **설명**:
- 관람객은 가방을 하나 가지고 있습니다
- **문제점**: `getBag()`으로 가방을 노출하면, 외부에서 관람객의 가방을 마음대로 열어볼 수 있습니다
- 현실에서는 다른 사람이 내 가방을 함부로 열어보면 안 되는 것처럼, 코드에서도 문제입니다

---

#### [`TicketOffice.java`](https://github.com/eternity-oop/object/blob/master/chapter01/src/main/java/org/eternity/theater/step01/TicketOffice.java) - 매표소

```java
public class TicketOffice {
    private Long amount;              // 판매 금액
    private List<Ticket> tickets = new ArrayList<>();  // 판매할 티켓들

    public TicketOffice(Long amount, Ticket... tickets) {
        this.amount = amount;
        this.tickets.addAll(Arrays.asList(tickets));
    }

    // ❌ 문제: 티켓을 외부에서 마음대로 가져갈 수 있음
    public Ticket getTicket() {
        return tickets.remove(0);  // 첫 번째 티켓을 꺼내서 줌
    }

    // ❌ 문제: 금액을 외부에서 마음대로 조작할 수 있음
    public void minusAmount(Long amount) {
        this.amount -= amount;
    }

    public void plusAmount(Long amount) {
        this.amount += amount;
    }
}
```

💡 **설명**:
- 매표소는 판매 금액과 티켓 목록을 관리합니다
- **문제점**: 누구든지 `getTicket()`을 호출해서 티켓을 가져갈 수 있고, 금액도 마음대로 조작할 수 있습니다

---

#### [`TicketSeller.java`](https://github.com/eternity-oop/object/blob/master/chapter01/src/main/java/org/eternity/theater/step01/TicketSeller.java) - 판매원

```java
public class TicketSeller {
    private TicketOffice ticketOffice;  // 판매원이 근무하는 매표소

    public TicketSeller(TicketOffice ticketOffice) {
        this.ticketOffice = ticketOffice;
    }

    // ❌ 문제: 매표소를 외부에 그대로 노출
    public TicketOffice getTicketOffice() {
        return ticketOffice;
    }
}
```

💡 **설명**:
- 판매원은 매표소에서 근무합니다
- **문제점**: `getTicketOffice()`로 매표소를 노출하면, 판매원이 없어도 매표소에 직접 접근할 수 있습니다

---

### 🏛️ 1-2. Theater - 모든 것을 처리하는 전지전능한 극장

#### [`Theater.java`](https://github.com/eternity-oop/object/blob/master/chapter01/src/main/java/org/eternity/theater/step01/Theater.java)

```java
public class Theater {
    private TicketSeller ticketSeller;

    public Theater(TicketSeller ticketSeller) {
        this.ticketSeller = ticketSeller;
    }

    public void enter(Audience audience) {
        // 1️⃣ 관람객의 가방을 직접 열어본다
        if (audience.getBag().hasInvitation()) {
            // 2️⃣ 초대장이 있으면
            
            // 3️⃣ 판매원의 매표소에서 티켓을 가져온다
            Ticket ticket = ticketSeller.getTicketOffice().getTicket();
            
            // 4️⃣ 관람객의 가방에 티켓을 넣어준다
            audience.getBag().setTicket(ticket);
        } else {
            // 5️⃣ 초대장이 없으면
            
            // 6️⃣ 역시 판매원의 매표소에서 티켓을 가져온다
            Ticket ticket = ticketSeller.getTicketOffice().getTicket();
            
            // 7️⃣ 관람객의 가방에서 티켓 가격만큼 현금을 뺀다
            audience.getBag().minusAmount(ticket.getFee());
            
            // 8️⃣ 판매원의 매표소에 그 금액을 추가한다
            ticketSeller.getTicketOffice().plusAmount(ticket.getFee());
            
            // 9️⃣ 관람객의 가방에 티켓을 넣어준다
            audience.getBag().setTicket(ticket);
        }
    }
}
```

### 🔍 1-3. 코드 흐름 상세 분석

```
Theater.enter(audience)가 호출되면:

1. audience.getBag()으로 관람객의 가방을 얻는다
   └─> Audience는 자기 의지 없이 가방을 내어준다

2. bag.hasInvitation()으로 초대장 유무를 확인한다
   └─> Bag은 자기 의지 없이 초대장 유무를 알려준다

3-1. 초대장이 있는 경우:
   a. ticketSeller.getTicketOffice()로 매표소를 얻는다
      └─> TicketSeller는 자기 의지 없이 매표소를 내어준다
   
   b. ticketOffice.getTicket()으로 티켓을 가져온다
      └─> TicketOffice는 자기 의지 없이 티켓을 내어준다
   
   c. audience.getBag().setTicket(ticket)으로 티켓을 넣는다
      └─> Bag은 자기 의지 없이 티켓을 받는다

3-2. 초대장이 없는 경우:
   a~b. 위와 동일하게 티켓을 가져온다
   
   c. audience.getBag().minusAmount(ticket.getFee())
      └─> Bag은 자기 의지 없이 돈이 빠져나간다
   
   d. ticketSeller.getTicketOffice().plusAmount(ticket.getFee())
      └─> TicketOffice는 자기 의지 없이 돈이 들어온다
   
   e. audience.getBag().setTicket(ticket)
      └─> Bag은 자기 의지 없이 티켓을 받는다
```

### ❌ 1-4. 문제점 분석

#### 🤔 문제 1: 이해하기 어려운 코드

**현실 세계**:
1. 관람객이 스스로 가방을 열어 초대장을 확인한다
2. 관람객이 판매원에게 표를 구매한다
3. 판매원이 매표소에서 티켓을 꺼내서 관람객에게 준다
4. 관람객이 스스로 돈을 내고 티켓을 받는다

**현재 코드**:
1. Theater가 관람객의 가방을 강제로 열어본다
2. Theater가 판매원의 매표소를 마음대로 연다
3. Theater가 관람객의 돈을 빼간다
4. Theater가 관람객의 가방에 티켓을 넣어준다

➡️ **관람객과 판매원은 완전히 수동적이며, Theater가 모든 것을 처리합니다**

---

#### 🔗 문제 2: 변경에 취약한 코드 (높은 결합도)

Theater가 알아야 하는 것들:
```java
enter(Audience audience) {
    audience.getBag()              // Audience의 내부 구조
        .hasInvitation()           // Bag의 내부 구조  
    ticketSeller.getTicketOffice() // TicketSeller의 내부 구조
        .getTicket()               // TicketOffice의 내부 구조
    audience.getBag()
        .minusAmount()             // Bag의 메서드
    ticketSeller.getTicketOffice()
        .plusAmount()              // TicketOffice의 메서드
}
```

**의존성 다이어그램**:
```
         Theater
        /   |   \
       /    |    \
      ▼     ▼     ▼
Audience  Ticket  TicketSeller
    |               |
    ▼               ▼
   Bag         TicketOffice
    |
    ▼
Invitation
```

Theater는 **6개 클래스**에 의존하고 있습니다!

**변경 시나리오 1**: 관람객이 가방이 아닌 지갑을 소지하면?
```java
// ❌ 이렇게 많은 곳을 수정해야 함
public class Audience {
    private Wallet wallet;  // Bag → Wallet 변경
    
    public Wallet getWallet() {  // getBag() → getWallet() 변경
        return wallet;
    }
}

public class Theater {
    public void enter(Audience audience) {
        if (audience.getWallet().hasInvitation()) {  // 수정 필요
            Ticket ticket = ticketSeller.getTicketOffice().getTicket();
            audience.getWallet().setTicket(ticket);  // 수정 필요
        } else {
            // ... 모든 getBag()을 getWallet()으로 변경
        }
    }
}
```

**변경 시나리오 2**: 판매원이 매표소가 아닌 은행에 돈을 보관하면?
- TicketSeller의 내부 구조 변경
- Theater.enter() 메서드 전체 수정 필요

➡️ **하나의 변경이 여러 클래스의 수정을 유발합니다**

---

#### 📊 문제 3: 낮은 응집도

각 클래스가 해야 할 일:
- `Bag`: 자신의 돈과 티켓을 관리해야 함
- `Audience`: 티켓 구매 결정을 내려야 함
- `TicketSeller`: 티켓 판매 업무를 수행해야 함
- `TicketOffice`: 티켓과 판매 금액을 관리해야 함

하지만 실제로는:
- `Theater`가 모든 일을 처리함
- 나머지 클래스들은 데이터만 보관하는 수동적 객체

➡️ **연관된 기능이 한 곳에 모여있지 않고 흩어져 있습니다**

---

## 🔧 Step 02: 객체지향적 설계 - 자율적인 객체들

> 📂 **전체 코드**: [step02 디렉토리](https://github.com/eternity-oop/object/tree/master/chapter01/src/main/java/org/eternity/theater/step02)

### 💡 2-1. 설계 원칙

> **"객체는 자신의 데이터를 스스로 처리해야 한다"**

**개선 방향**:
1. `Audience`가 자신의 `Bag`을 스스로 관리
2. `TicketSeller`가 자신의 `TicketOffice`를 스스로 관리
3. `Theater`는 각 객체에게 "일을 시키기"만 함

### 🎯 2-2. 핵심 변경사항

#### 변경 1: Theater - 구체적인 방법을 모르게 만들기

**Before (Step 01)**:
```java
public void enter(Audience audience) {
    // Theater가 구체적인 방법을 다 알고 있음
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
```

**After (Step 02)**: [`Theater.java`](https://github.com/eternity-oop/object/blob/master/chapter01/src/main/java/org/eternity/theater/step02/Theater.java)
```java
public void enter(Audience audience) {
    // Theater는 "판매하라"고만 요청함
    // "어떻게" 판매하는지는 TicketSeller가 알아서 함
    ticketSeller.sellTo(audience);
}
```

💡 **핵심**:
- Theater는 더 이상 티켓 판매 과정의 세부사항을 몰라도 됨
- `ticketSeller.sellTo(audience)` 한 줄로 의도만 전달
- "어떻게"는 `TicketSeller`에게 위임

---

#### 변경 2: TicketSeller - 판매 로직을 내부로 캡슐화

**Before (Step 01)**: [`TicketSeller.java`](https://github.com/eternity-oop/object/blob/master/chapter01/src/main/java/org/eternity/theater/step01/TicketSeller.java)
```java
public class TicketSeller {
    private TicketOffice ticketOffice;

    public TicketSeller(TicketOffice ticketOffice) {
        this.ticketOffice = ticketOffice;
    }

    // ❌ 매표소를 외부에 노출
    public TicketOffice getTicketOffice() {
        return ticketOffice;
    }
    
    // 판매 로직이 없음 - Theater가 대신 처리
}
```

**After (Step 02)**: [`TicketSeller.java`](https://github.com/eternity-oop/object/blob/master/chapter01/src/main/java/org/eternity/theater/step02/TicketSeller.java)
```java
public class TicketSeller {
    private TicketOffice ticketOffice;

    public TicketSeller(TicketOffice ticketOffice) {
        this.ticketOffice = ticketOffice;
    }

    // ✅ 1. getTicketOffice() 메서드 삭제
    //    → 매표소를 외부에 노출하지 않음
    
    // ✅ 2. 판매 로직을 TicketSeller 내부로 이동
    public void sellTo(Audience audience) {
        // a. 매표소에서 티켓을 가져온다
        Ticket ticket = ticketOffice.getTicket();
        
        // b. 관람객에게 티켓을 판다
        //    관람객이 지불한 금액을 반환받는다
        Long payment = audience.buy(ticket);
        
        // c. 받은 금액을 매표소에 보관한다
        ticketOffice.plusAmount(payment);
    }
}
```

💡 **핵심**:
- `getTicketOffice()` 제거 → 매표소를 외부에 감춤
- `sellTo()` 추가 → 판매 로직을 캡슐화
- TicketSeller가 매표소를 어떻게 사용하는지는 외부에서 알 수 없음

**흐름 설명**:
```
1. ticketOffice.getTicket()
   └─> "매표소야, 티켓 하나 줘"
   
2. audience.buy(ticket)
   └─> "관람객님, 이 티켓 사시겠어요?"
   └─> 관람객이 지불한 금액을 반환받음
   
3. ticketOffice.plusAmount(payment)
   └─> "매표소야, 이 금액 보관해줘"
```

---

#### 변경 3: Audience - 구매 로직을 내부로 캡슐화

**Before (Step 01)**: [`Audience.java`](https://github.com/eternity-oop/object/blob/master/chapter01/src/main/java/org/eternity/theater/step01/Audience.java)
```java
public class Audience {
    private Bag bag;

    public Audience(Bag bag) {
        this.bag = bag;
    }

    // ❌ 가방을 외부에 노출
    public Bag getBag() {
        return bag;
    }
    
    // 구매 로직이 없음 - Theater가 대신 처리
}
```

**After (Step 02)**: [`Audience.java`](https://github.com/eternity-oop/object/blob/master/chapter01/src/main/java/org/eternity/theater/step02/Audience.java)
```java
public class Audience {
    private Bag bag;

    public Audience(Bag bag) {
        this.bag = bag;
    }

    // ✅ 1. getBag() 메서드 삭제
    //    → 가방을 외부에 노출하지 않음
    
    // ✅ 2. 구매 로직을 Audience 내부로 이동
    public Long buy(Ticket ticket) {
        // a. 가방에 초대장이 있는지 확인
        if (bag.hasInvitation()) {
            // b-1. 초대장이 있으면 무료
            bag.setTicket(ticket);  // 티켓만 받음
            return 0L;               // 지불 금액 0원
        } else {
            // b-2. 초대장이 없으면 유료
            bag.setTicket(ticket);           // 티켓을 받고
            bag.minusAmount(ticket.getFee()); // 돈을 지불
            return ticket.getFee();           // 지불한 금액 반환
        }
    }
}
```

💡 **핵심**:
- `getBag()` 제거 → 가방을 외부에 감춤
- `buy()` 추가 → 구매 로직을 캡슐화
- Audience가 가방을 어떻게 사용하는지는 외부에서 알 수 없음

**흐름 설명**:
```
1. bag.hasInvitation()
   └─> "내 가방에 초대장이 있나?"
   
2-1. 초대장이 있으면:
   a. bag.setTicket(ticket)
      └─> "가방에 티켓 넣어야지"
   b. return 0L
      └─> "무료니까 0원 지불"
   
2-2. 초대장이 없으면:
   a. bag.setTicket(ticket)
      └─> "가방에 티켓 넣고"
   b. bag.minusAmount(ticket.getFee())
      └─> "가방에서 돈 꺼내서 지불"
   c. return ticket.getFee()
      └─> "이만큼 지불했어요"
```

---

### 📊 2-3. 변경 전후 비교

#### 호출 흐름 비교

**Before (Step 01)**:
```
Theater.enter(audience)
  ├─ audience.getBag().hasInvitation()         // Theater가 직접 접근
  ├─ ticketSeller.getTicketOffice().getTicket() // Theater가 직접 접근
  ├─ audience.getBag().setTicket()             // Theater가 직접 조작
  ├─ audience.getBag().minusAmount()           // Theater가 직접 조작
  └─ ticketSeller.getTicketOffice().plusAmount() // Theater가 직접 조작
```

**After (Step 02)**:
```
Theater.enter(audience)
  └─ ticketSeller.sellTo(audience)             // Theater는 요청만 함
       ├─ ticketOffice.getTicket()             // TicketSeller가 처리
       ├─ audience.buy(ticket)                 // 관람객에게 위임
       │    ├─ bag.hasInvitation()             // Audience가 처리
       │    ├─ bag.setTicket()
       │    └─ bag.minusAmount()
       └─ ticketOffice.plusAmount()            // TicketSeller가 처리
```

#### 의존성 비교

**Before (Step 01)**:
```
Theater가 알아야 하는 것:
- Audience의 getBag()
- Bag의 hasInvitation(), setTicket(), minusAmount()
- TicketSeller의 getTicketOffice()
- TicketOffice의 getTicket(), plusAmount()
- Ticket의 getFee()

총 6개 클래스에 의존
```

**After (Step 02)**:
```
Theater가 알아야 하는 것:
- TicketSeller의 sellTo()
- Audience (인자로 전달만 함)

총 2개 클래스에 의존
```

---

### ✅ 2-4. 개선 효과

#### 효과 1: 캡슐화 (Encapsulation)

**TicketSeller의 캡슐화**:
```java
// Before: 매표소가 노출됨
ticketSeller.getTicketOffice().getTicket();

// After: 매표소가 감춰짐
ticketSeller.sellTo(audience);
```

**Audience의 캡슐화**:
```java
// Before: 가방이 노출됨  
audience.getBag().hasInvitation();

// After: 가방이 감춰짐
audience.buy(ticket);
```

---

#### 효과 2: 변경의 국지화

**시나리오**: 관람객이 가방 대신 지갑을 소지하도록 변경

**Before (Step 01)**: 여러 곳 수정 필요
```java
// 1. Audience 수정
public class Audience {
    private Wallet wallet;  // 변경
    public Wallet getWallet() { return wallet; }  // 변경
}

// 2. Theater 수정 (❌ Theater까지 수정해야 함!)
public void enter(Audience audience) {
    if (audience.getWallet().hasInvitation()) {  // 변경
        // ...
    }
}
```

**After (Step 02)**: Audience만 수정
```java
// 1. Audience만 수정
public class Audience {
    private Wallet wallet;  // 변경
    
    public Long buy(Ticket ticket) {
        // wallet을 사용하도록 내부 구현만 변경
        if (wallet.hasInvitation()) {  // 변경
            wallet.setTicket(ticket);
            return 0L;
        } else {
            wallet.setTicket(ticket);
            wallet.minusAmount(ticket.getFee());
            return ticket.getFee();
        }
    }
}

// 2. Theater는 수정 불필요! (✅ Theater는 그대로)
public void enter(Audience audience) {
    ticketSeller.sellTo(audience);  // 변경 없음!
}
```

---

#### 효과 3: 응집도 향상

**Before (Step 01)**:
```
관람객의 돈을 관리하는 로직이 어디에 있나?
→ Theater.enter()에 있음 (❌ 잘못된 위치)

판매원의 티켓 판매 로직이 어디에 있나?
→ Theater.enter()에 있음 (❌ 잘못된 위치)
```

**After (Step 02)**:
```
관람객의 돈을 관리하는 로직이 어디에 있나?
→ Audience.buy()에 있음 (✅ 올바른 위치)

판매원의 티켓 판매 로직이 어디에 있나?
→ TicketSeller.sellTo()에 있음 (✅ 올바른 위치)
```

---

## 🎯 Step 03: 추가 캡슐화 - Bag도 자율적으로

> 📂 **전체 코드**: [step03 디렉토리](https://github.com/eternity-oop/object/tree/master/chapter01/src/main/java/org/eternity/theater/step03)

Step 02에서는 `Audience`와 `TicketSeller`를 개선했지만, 여전히 문제가 있습니다:
- `Audience`가 `Bag`의 내부를 직접 조작함
- `Bag`은 여전히 수동적임

### 🔍 3-1. Bag의 문제점 발견

**현재 Audience.buy() 코드**:
```java
public Long buy(Ticket ticket) {
    if (bag.hasInvitation()) {      // Audience가 Bag 내부를 확인하고
        bag.setTicket(ticket);       // Audience가 Bag 내부를 조작하고
        return 0L;
    } else {
        bag.setTicket(ticket);       // Audience가 Bag 내부를 조작하고
        bag.minusAmount(ticket.getFee()); // Audience가 Bag 내부를 조작함
        return ticket.getFee();
    }
}
```

**문제**: `Audience`가 `Bag`을 마치 자기 부품처럼 직접 제어하고 있습니다.

---

### 🔧 3-2. Bag에 자율성 부여하기

#### 변경: Bag이 스스로 티켓을 보관하도록

**Before (Step 02)**: [`Bag.java`](https://github.com/eternity-oop/object/blob/master/chapter01/src/main/java/org/eternity/theater/step02/Bag.java)
```java
public class Bag {
    private Long amount;
    private Invitation invitation;
    private Ticket ticket;

    // 생략...

    // ❌ 외부에서 내부 상태를 직접 조작 가능
    public boolean hasInvitation() {
        return invitation != null;
    }

    public void setTicket(Ticket ticket) {
        this.ticket = ticket;
    }

    public void minusAmount(Long amount) {
        this.amount -= amount;
    }
}
```

**After (Step 03)**: [`Bag.java`](https://github.com/eternity-oop/object/blob/master/chapter01/src/main/java/org/eternity/theater/step03/Bag.java)
```java
public class Bag {
    private Long amount;
    private Ticket ticket;
    private Invitation invitation;

    // ✅ 1. 티켓 보관 로직을 Bag 내부로 이동
    public Long hold(Ticket ticket) {
        // a. 초대장이 있는지 스스로 확인
        if (hasInvitation()) {
            // b-1. 초대장이 있으면
            setTicket(ticket);  // 티켓만 보관
            return 0L;          // 지불 금액 0원
        } else {
            // b-2. 초대장이 없으면
            setTicket(ticket);           // 티켓을 보관하고
            minusAmount(ticket.getFee()); // 돈을 지불
            return ticket.getFee();       // 지불 금액 반환
        }
    }

    // ✅ 2. 내부 메서드들을 private으로 변경
    //    → 외부에서 직접 조작할 수 없음
    private void setTicket(Ticket ticket) {
        this.ticket = ticket;
    }

    private boolean hasInvitation() {
        return invitation != null;
    }

    private void minusAmount(Long amount) {
        this.amount -= amount;
    }
}
```

💡 **핵심**:
- `hold()` 메서드 추가 → 티켓 보관 로직을 Bag이 담당
- `hasInvitation()`, `setTicket()`, `minusAmount()`를 `private`으로 변경
- 외부에서는 `hold()`만 호출 가능, 내부 구현은 알 수 없음

**흐름 설명**:
```
Bag.hold(ticket)가 호출되면:

1. hasInvitation()
   └─> "내가 초대장을 가지고 있나?"
   
2-1. 초대장이 있으면:
   a. setTicket(ticket)
      └─> "티켓을 내 안에 넣자"
   b. return 0L
      └─> "무료니까 0원"
   
2-2. 초대장이 없으면:
   a. setTicket(ticket)
      └─> "티켓을 내 안에 넣고"
   b. minusAmount(ticket.getFee())
      └─> "내 안의 돈에서 차감하자"
   c. return ticket.getFee()
      └─> "이만큼 지불했어"
```

---

#### Audience 단순화

**Before (Step 02)**: [`Audience.java`](https://github.com/eternity-oop/object/blob/master/chapter01/src/main/java/org/eternity/theater/step02/Audience.java)
```java
public Long buy(Ticket ticket) {
    // Audience가 Bag 내부를 직접 제어
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

**After (Step 03)**: [`Audience.java`](https://github.com/eternity-oop/object/blob/master/chapter01/src/main/java/org/eternity/theater/step03/Audience.java)
```java
public Long buy(Ticket ticket) {
    // Audience는 Bag에게 "보관해줘"라고만 요청
    // "어떻게" 보관하는지는 Bag이 알아서 처리
    return bag.hold(ticket);
}
```

💡 **핵심**:
- `Audience.buy()`가 훨씬 단순해짐
- Audience는 Bag이 초대장을 어떻게 확인하는지, 돈을 어떻게 관리하는지 몰라도 됨
- `bag.hold(ticket)` 한 줄로 의도만 전달

---

### 📊 3-3. Bag 캡슐화 효과

#### 변경 시나리오: Bag의 내부 구현 변경

예를 들어, 초대장 확인 로직을 다음과 같이 변경한다면:

**Before (Step 02에서 변경 시)**:
```java
// 1. Bag 수정
public boolean hasInvitation() {
    return invitation != null && invitation.isValid();  // 유효성 검사 추가
}

// 2. Audience도 수정해야 할 수도 있음
public Long buy(Ticket ticket) {
    if (bag.hasInvitation()) {  // 이 부분의 동작이 변경됨
        // ...
    }
}
```

**After (Step 03에서 변경 시)**:
```java
// 1. Bag만 수정 (외부에는 영향 없음)
private boolean hasInvitation() {
    return invitation != null && invitation.isValid();  // 유효성 검사 추가
}

// 2. Audience는 수정 불필요!
public Long buy(Ticket ticket) {
    return bag.hold(ticket);  // 변경 없음!
}
```

---

### ⚠️ 3-4. TicketOffice 캡슐화 시도와 트레이드오프

지금까지의 개선으로 다음과 같은 구조가 되었습니다:
```
Theater
  └─ ticketSeller.sellTo(audience)
       └─ audience.buy(ticket)
            └─ bag.hold(ticket)
```

그렇다면 `TicketOffice`도 자율적으로 만들 수 있지 않을까요?

#### 시도: TicketOffice도 자율적으로 만들기

**현재 (Step 02~03)**: [`TicketSeller.java`](https://github.com/eternity-oop/object/blob/master/chapter01/src/main/java/org/eternity/theater/step02/TicketSeller.java)
```java
public void sellTo(Audience audience) {
    // TicketSeller가 TicketOffice를 직접 조작
    ticketOffice.plusAmount(audience.buy(ticketOffice.getTicket()));
}
```

**시도: TicketOffice가 스스로 판매하도록**: [`TicketOffice.java`](https://github.com/eternity-oop/object/blob/master/chapter01/src/main/java/org/eternity/theater/step03/TicketOffice.java)
```java
public class TicketOffice {
    private Long amount;
    private List<Ticket> tickets = new ArrayList<>();

    // TicketOffice가 직접 판매 로직을 처리
    public void sellTicketTo(Audience audience) {
        plusAmount(audience.buy(getTicket()));
    }

    private Ticket getTicket() {
        return tickets.remove(0);
    }

    private void plusAmount(Long amount) {
        this.amount += amount;
    }
}
```

**TicketSeller 단순화**: [`TicketSeller.java`](https://github.com/eternity-oop/object/blob/master/chapter01/src/main/java/org/eternity/theater/step03/TicketSeller.java)
```java
public void sellTo(Audience audience) {
    // TicketSeller는 TicketOffice에게 "판매해줘"라고만 요청
    ticketOffice.sellTicketTo(audience);
}
```

---

#### 문제 발생: 새로운 의존성 추가

**Before (Step 02)**:
```
TicketOffice는 독립적인 객체
- Ticket 목록만 관리
- 금액만 관리
- Audience를 몰라도 됨
```

**After (시도)**:
```
TicketOffice가 Audience에 의존
public void sellTicketTo(Audience audience) {
    plusAmount(audience.buy(getTicket()));  // Audience를 알아야 함!
}
```

**의존성 다이어그램**:
```
Before:
TicketSeller ──> TicketOffice
      │
      └─────> Audience

After:
TicketSeller ──> TicketOffice ──> Audience
                      │
                      └─> 새로운 의존성 추가!
```

---

#### 트레이드오프 분석

| 측면 | TicketOffice 자율성 부여 | 기존 구조 유지 |
|------|------------------------|-------------|
| **TicketOffice 자율성** | ✅ 높음 | ❌ 낮음 |
| **결합도** | ❌ TicketOffice → Audience 의존성 추가 | ✅ 의존성 없음 |
| **TicketOffice 재사용성** | ❌ Audience 없이 사용 불가 | ✅ 독립적으로 사용 가능 |
| **설계 일관성** | ✅ 모든 객체가 자율적 | ❌ TicketOffice만 수동적 |

**고려사항**:
1. `TicketOffice`는 `Audience`라는 개념 없이도 존재할 수 있어야 함
2. 매표소는 단순히 티켓과 금액을 관리하는 저수준 객체
3. `Audience`와의 의존성은 도메인적으로 부자연스러움

---

#### 최종 결정: Step 02 구조 유지

```java
// 최종 코드 (Step 02 구조)
public class TicketSeller {
    public void sellTo(Audience audience) {
        // TicketSeller가 TicketOffice를 사용하는 방법을 알고 있음
        ticketOffice.plusAmount(audience.buy(ticketOffice.getTicket()));
    }
}
```

**결정 이유**:
1. **TicketOffice의 자율성** < **Audience와의 결합도 제거**
2. TicketOffice는 범용적인 구성요소로 유지하는 것이 더 중요
3. TicketSeller가 TicketOffice 사용 방법을 아는 것은 자연스러움

💡 **핵심 교훈**:
> **"설계는 트레이드오프의 산물이다"**
>
> 모든 것을 완벽하게 만들 수는 없습니다. 상황에 따라 무엇이 더 중요한지 판단하고, 때로는 한 쪽을 포기해야 합니다.

---

## 🤔 핵심 개념 정리

### 1️⃣ 절차지향 vs 객체지향

#### 절차지향 (Step 01)
```java
// 데이터와 프로세스 분리
public class Theater {
    public void enter(Audience audience) {
        // Theater가 모든 프로세스를 처리
        // Audience, Bag, TicketSeller, TicketOffice는 데이터만 제공
        if (audience.getBag().hasInvitation()) {
            Ticket ticket = ticketSeller.getTicketOffice().getTicket();
            audience.getBag().setTicket(ticket);
        }
        // ...
    }
}
```

**특징**:
- 데이터(Audience, Bag 등)와 프로세스(Theater.enter())가 분리
- 프로세스가 데이터를 가져다가 처리
- 데이터 변경 시 프로세스도 변경 필요

#### 객체지향 (Step 02~03)
```java
// 데이터와 프로세스 통합
public class Theater {
    public void enter(Audience audience) {
        ticketSeller.sellTo(audience);  // 요청만 함
    }
}

public class TicketSeller {
    public void sellTo(Audience audience) {
        // TicketSeller가 자신의 데이터를 스스로 처리
        ticketOffice.plusAmount(audience.buy(ticketOffice.getTicket()));
    }
}

public class Audience {
    public Long buy(Ticket ticket) {
        // Audience가 자신의 데이터를 스스로 처리
        return bag.hold(ticket);
    }
}
```

**특징**:
- 데이터와 프로세스가 같은 객체 안에 존재
- 객체가 자신의 데이터를 스스로 처리
- 데이터 변경 시 해당 객체만 변경하면 됨

**비교표**:

| 구분 | 절차지향 (Step 01) | 객체지향 (Step 02~03) |
|------|-------------------|---------------------|
| **데이터와 프로세스** | 분리 | 동일 객체 내 위치 |
| **책임 소재** | Theater에 집중 | 각 객체에 분산 |
| **변경 영향 범위** | 여러 곳에 파급 | 해당 객체로 국한 |
| **결합도** | 높음 (6개 클래스 의존) | 낮음 (2개 클래스 의존) |
| **응집도** | 낮음 (기능이 흩어짐) | 높음 (기능이 응집) |
| **이해하기** | 어려움 | 쉬움 |

---

### 2️⃣ 좋은 설계란?

```
좋은 설계 = 오늘 요구하는 기능을 구현 + 내일의 변경을 수용
```

**로버트 마틴의 소프트웨어 모듈 3가지 목적**:

#### 1. 실행 중 제대로 동작
```java
// Step 01도 Step 02도 기능은 똑같이 동작함
theater.enter(audience);  // 둘 다 정상 작동
```

#### 2. 변경을 위해 존재
```java
// Step 01: 변경이 어려움
// Bag → Wallet 변경 시 Theater도 수정 필요

// Step 02~03: 변경이 쉬움
// Bag → Wallet 변경 시 Audience만 수정
```

#### 3. 코드 읽는 사람과 의사소통
```java
// Step 01: 이해하기 어려움
if (audience.getBag().hasInvitation()) {
    Ticket ticket = ticketSeller.getTicketOffice().getTicket();
    audience.getBag().setTicket(ticket);
}
// "Theater가 왜 관람객 가방을 열어봐?"

// Step 02~03: 이해하기 쉬움
ticketSeller.sellTo(audience);
// "판매원이 관람객에게 판매하는구나!"
```

---

### 3️⃣ 캡슐화 (Encapsulation)

#### 개념
> **"객체 내부 구현을 감추고 인터페이스만 노출하는 것"**

#### 나쁜 예 (Step 01)
```java
public class Audience {
    private Bag bag;
    
    public Bag getBag() {  // ❌ 내부를 노출
        return bag;
    }
}

// 사용하는 쪽
audience.getBag().setTicket(ticket);  // Bag 내부를 직접 조작
```

**문제점**:
- `Audience`가 `Bag`을 어떻게 관리하는지 외부에 노출
- 나중에 `Bag` → `Wallet`으로 바꾸면 모든 `getBag()` 호출 코드를 수정해야 함

#### 좋은 예 (Step 02~03)
```java
public class Audience {
    private Bag bag;  // ✅ 내부를 감춤
    
    public Long buy(Ticket ticket) {  // ✅ 인터페이스만 노출
        return bag.hold(ticket);
    }
}

// 사용하는 쪽
audience.buy(ticket);  // 내부 구현을 몰라도 됨
```

**장점**:
- `Audience`가 `Bag`을 어떻게 관리하는지 외부에서 알 수 없음
- `Bag` → `Wallet`으로 바꿔도 `buy()` 내부만 수정하면 됨
- `audience.buy(ticket)` 호출 코드는 변경 불필요

#### 캡슐화 체크리스트

```java
// ❌ 캡슐화 위반 패턴
class SomeClass {
    public InternalData getData() { ... }      // getter로 내부 노출
    public void setData(InternalData d) { ... } // setter로 내부 노출
}

// ✅ 캡슐화 준수 패턴
class SomeClass {
    private InternalData data;  // private으로 감춤
    
    public Result doSomething() {  // 의미 있는 행동 제공
        // data를 사용한 로직
    }
}
```

---

### 4️⃣ 응집도 (Cohesion)

#### 개념
> **"객체가 자신의 데이터를 스스로 처리하는 정도"**

#### 낮은 응집도 (Step 01)
```java
public class Bag {
    private Long amount;
    private Ticket ticket;
    
    // 데이터만 보관
    public void setTicket(Ticket ticket) { ... }
    public void minusAmount(Long amount) { ... }
}

public class Theater {
    public void enter(Audience audience) {
        // Theater가 Bag의 데이터를 처리
        audience.getBag().setTicket(ticket);
        audience.getBag().minusAmount(ticket.getFee());
    }
}
```

**문제**:
- `Bag`의 데이터를 `Theater`가 처리
- `Bag`과 관련된 로직이 `Theater`에 흩어짐
- "Bag에 티켓 넣기"와 "돈 빼기"가 서로 다른 곳에 있음

#### 높은 응집도 (Step 02~03)
```java
public class Bag {
    private Long amount;
    private Ticket ticket;
    
    // 자신의 데이터를 스스로 처리
    public Long hold(Ticket ticket) {
        if (hasInvitation()) {
            setTicket(ticket);
            return 0L;
        } else {
            setTicket(ticket);
            minusAmount(ticket.getFee());  // 자신의 돈을 스스로 관리
            return ticket.getFee();
        }
    }
    
    private void setTicket(Ticket ticket) { ... }
    private void minusAmount(Long amount) { ... }
}
```

**장점**:
- `Bag`의 데이터를 `Bag`이 처리
- `Bag`과 관련된 로직이 `Bag` 안에 응집
- "티켓 넣기"와 "돈 빼기"가 `hold()` 안에 함께 있음

#### 응집도 판단 기준

```
높은 응집도:
- 연관된 데이터와 로직이 한 곳에 모여있음
- 객체가 자신의 일을 스스로 처리
- 객체 외부에서 내부를 몰라도 됨

낮은 응집도:
- 연관된 로직이 여러 곳에 흩어져 있음
- 다른 객체가 대신 일을 처리
- 객체 외부에서 내부를 알아야 함
```

---

### 5️⃣ 결합도 (Coupling)

#### 개념
> **"객체 간 의존성의 정도"**

#### 높은 결합도 (Step 01)
```java
public class Theater {
    public void enter(Audience audience) {
        // Theater가 많은 객체에 의존
        if (audience.getBag().hasInvitation()) {           // Audience, Bag에 의존
            Ticket ticket = ticketSeller.getTicketOffice() // TicketSeller, TicketOffice에 의존
                                        .getTicket();
            audience.getBag().setTicket(ticket);           // Ticket에 의존
        }
        // ...
    }
}
```

**의존 관계**:
```
Theater가 알아야 하는 것:
1. Audience의 getBag() 메서드
2. Bag의 hasInvitation(), setTicket(), minusAmount() 메서드
3. TicketSeller의 getTicketOffice() 메서드
4. TicketOffice의 getTicket(), plusAmount() 메서드
5. Ticket의 getFee() 메서드

→ 5개 클래스의 10개 이상 메서드에 의존
```

#### 낮은 결합도 (Step 02~03)
```java
public class Theater {
    public void enter(Audience audience) {
        // Theater가 적은 객체에 의존
        ticketSeller.sellTo(audience);  // TicketSeller의 sellTo()만 의존
    }
}
```

**의존 관계**:
```
Theater가 알아야 하는 것:
1. TicketSeller의 sellTo() 메서드

→ 1개 클래스의 1개 메서드에만 의존
```

#### 결합도가 중요한 이유

**높은 결합도의 문제**:
```java
// Bag 내부 구현 변경
public class Bag {
    public void putTicket(Ticket ticket) {  // setTicket → putTicket 변경
        this.ticket = ticket;
    }
}

// Theater도 수정 필요! (❌ 변경 파급)
public class Theater {
    public void enter(Audience audience) {
        audience.getBag().putTicket(ticket);  // setTicket → putTicket 변경
    }
}
```

**낮은 결합도의 장점**:
```java
// Bag 내부 구현 변경
public class Bag {
    public Long hold(Ticket ticket) {
        putTicket(ticket);  // 내부적으로만 변경
        // ...
    }
    
    private void putTicket(Ticket ticket) {  // private 메서드 변경
        this.ticket = ticket;
    }
}

// Theater는 수정 불필요! (✅ 변경 국지화)
public class Theater {
    public void enter(Audience audience) {
        ticketSeller.sellTo(audience);  // 변경 없음!
    }
}
```

---

### 6️⃣ 책임의 이동 (Shift of Responsibility)

#### 절차지향: 책임이 한 곳에 집중

```
            [Theater]
             모든 책임
        /   |   |   |   \
       /    |   |   |    \
     티켓  초대장  돈  금액   티켓
      발급  확인  결제  보관  전달
       |    |    |    |    |
       ▼    ▼    ▼    ▼    ▼
(Audience, Bag, TicketSeller, TicketOffice는 수동적)
```

**코드**:
```java
public void enter(Audience audience) {
    // Theater가 모든 책임을 짐
    if (audience.getBag().hasInvitation()) {        // 1. 초대장 확인
        Ticket ticket = ticketSeller                // 2. 티켓 발급
                         .getTicketOffice()
                         .getTicket();
        audience.getBag().setTicket(ticket);        // 3. 티켓 전달
    } else {
        Ticket ticket = ticketSeller                // 4. 티켓 발급
                         .getTicketOffice()
                         .getTicket();
        audience.getBag().minusAmount(...);         // 5. 돈 결제
        ticketSeller.getTicketOffice().plusAmount(...); // 6. 금액 보관
        audience.getBag().setTicket(ticket);        // 7. 티켓 전달
    }
}
```

#### 객체지향: 책임이 분산

```
    [Theater]           [TicketSeller]      [Audience]         [Bag]
     입장 처리      →        티켓 판매     →      티켓 구매    →    티켓/돈 보관
        ↓                     ↓                 ↓                ↓
     sellTo()               buy()             hold()          내부 처리
```

**코드**:
```java
// Theater: 입장 처리 책임
public void enter(Audience audience) {
    ticketSeller.sellTo(audience);
}

// TicketSeller: 티켓 판매 책임
public void sellTo(Audience audience) {
    ticketOffice.plusAmount(audience.buy(ticketOffice.getTicket()));
}

// Audience: 티켓 구매 책임
public Long buy(Ticket ticket) {
    return bag.hold(ticket);
}

// Bag: 티켓/돈 보관 책임
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
```

**핵심**:
```
절차지향: 하나의 객체가 모든 책임을 짐
객체지향: 각 객체가 자신의 책임을 짐
```

---

### 7️⃣ 의인화 (Anthropomorphism)

#### 개념
> **"현실에서는 수동적인 객체를 능동적으로 설계하는 것"**

#### 현실 세계
```
가방은 수동적임:
- 사람이 가방을 열어야 함
- 사람이 가방에서 돈을 꺼내야 함
- 가방은 스스로 아무것도 못 함
```

#### 객체지향 세계
```java
public class Bag {
    public Long hold(Ticket ticket) {
        // 가방이 스스로 판단하고 행동함
        if (hasInvitation()) {
            setTicket(ticket);
            return 0L;
        } else {
            setTicket(ticket);
            minusAmount(ticket.getFee());
            return ticket.getFee();
        }
    }
}
```

**의인화 예시**:
```
현실:
- 가방: 수동적 → 객체지향: 능동적 (스스로 티켓 보관)
- 티켓: 수동적 → 객체지향: 능동적 (스스로 가격 제공)
- 매표소: 수동적 → 객체지향: 능동적 (스스로 티켓 관리)
```

**왜 의인화가 필요한가?**:
1. **자율성**: 객체가 스스로 결정하고 행동
2. **캡슐화**: 내부 구현을 숨길 수 있음
3. **유연성**: 변경에 대응하기 쉬움
4. **이해하기 쉬움**: 객체 간 협력이 명확함

---

## 💡 실전 적용 가이드

### 1️⃣ 설계 개선 체크리스트

코드 리뷰 시 다음을 확인하세요:

```java
// ❌ 나쁜 코드 패턴
class BadExample {
    public void doSomething(SomeObject obj) {
        // 1. 다른 객체의 내부를 가져옴
        InternalData data = obj.getData();
        
        // 2. 가져온 데이터를 직접 조작
        if (data.check()) {
            data.modify();
        }
        
        // 3. 다시 넣어줌
        obj.setData(data);
    }
}

// ✅ 좋은 코드 패턴
class GoodExample {
    public void doSomething(SomeObject obj) {
        // 객체에게 행동을 요청
        obj.process();
    }
}
```

**체크 항목**:
- [ ] 객체가 자신의 데이터를 직접 처리하는가?
- [ ] 외부에서 객체 내부를 직접 제어하지 않는가?
- [ ] getter가 꼭 필요한 경우에만 사용되는가?
- [ ] setter를 사용하지 않는가?
- [ ] 변경 시 한 곳만 수정하면 되는가?
- [ ] 인터페이스만으로 협력이 가능한가?
- [ ] 객체의 역할이 명확한가?

---

### 2️⃣ 리팩토링 단계별 가이드

#### Step 1: getter/setter 사용처 찾기

```java
// 문제 있는 코드 찾기
obj.getData().process();
obj.setData(newData);
```

#### Step 2: 로직을 데이터 소유 객체로 이동

```java
// Before
class Client {
    public void doSomething(DataHolder holder) {
        Data data = holder.getData();  // getter 사용
        data.process();
        holder.setData(data);  // setter 사용
    }
}

// After
class DataHolder {
    private Data data;
    
    public void process() {  // 로직을 내부로 이동
        data.process();
    }
}

class Client {
    public void doSomething(DataHolder holder) {
        holder.process();  // 인터페이스만 사용
    }
}
```

#### Step 3: getter/setter 제거

```java
class DataHolder {
    private Data data;
    
    // ❌ 제거
    // public Data getData() { return data; }
    // public void setData(Data data) { this.data = data; }
    
    // ✅ 의미 있는 메서드 제공
    public void process() {
        data.process();
    }
}
```

#### Step 4: private 메서드 정리

```java
class DataHolder {
    private Data data;
    
    // public 인터페이스
    public void process() {
        if (isValid()) {
            doProcess();
        }
    }
    
    // private 구현
    private boolean isValid() { ... }
    private void doProcess() { ... }
}
```

---

### 3️⃣ 트레이드오프 판단 기준

```
자율성 증가 시도
     ↓
새로운 의존성 발생?
     ↓
   /   \
 Yes    No
  ↓      ↓
트레이드  리팩토링
오프 검토  진행
  ↓
어느 것이 더 중요한가?
  ↓
┌─────────────────┬─────────────────┐
│ 자율성 증가        │ 결합도 감소        │
│ 설계 일관성        │ 재사용성          │
│ 캡슐화            │ 독립성           │
└─────────────────┴─────────────────┘
```

**예시: TicketOffice 캡슐화 판단**

| 기준 | TicketOffice 자율화 | 현재 구조 유지 |
|------|-------------------|--------------|
| 자율성 | ✅ 높음 | ❌ 낮음 |
| 결합도 | ❌ Audience 의존 추가 | ✅ 의존 없음 |
| 재사용성 | ❌ Audience 필요 | ✅ 독립적 사용 |
| 도메인 적합성 | ❌ 부자연스러움 | ✅ 자연스러움 |
| **결정** | ❌ 채택 안 함 | ✅ 채택 |

---

### 4️⃣ 실전 예제: Service 레이어 리팩토링

#### Before: 전형적인 빈약한 도메인 모델

```java
// Controller
@PostMapping("/orders")
public OrderResponse createOrder(@RequestBody OrderRequest request) {
    return orderService.createOrder(request);
}

// Service (모든 로직이 여기에)
@Service
public class OrderService {
    public OrderResponse createOrder(OrderRequest request) {
        // 1. 주문 생성
        Order order = new Order();
        order.setUserId(request.getUserId());
        order.setProductId(request.getProductId());
        order.setQuantity(request.getQuantity());
        
        // 2. 재고 확인
        Product product = productRepository.findById(request.getProductId());
        if (product.getStock() < request.getQuantity()) {
            throw new OutOfStockException();
        }
        
        // 3. 가격 계산
        Long totalPrice = product.getPrice() * request.getQuantity();
        if (request.hasCoupon()) {
            totalPrice = totalPrice - request.getCouponDiscount();
        }
        order.setTotalPrice(totalPrice);
        
        // 4. 재고 차감
        product.setStock(product.getStock() - request.getQuantity());
        
        // 5. 저장
        orderRepository.save(order);
        productRepository.save(product);
        
        return new OrderResponse(order);
    }
}

// Domain (데이터만 보관)
public class Order {
    private Long userId;
    private Long productId;
    private Integer quantity;
    private Long totalPrice;
    
    // getter, setter만 있음
}

public class Product {
    private Long id;
    private Long price;
    private Integer stock;
    
    // getter, setter만 있음
}
```

**문제점**:
- Service가 모든 로직을 처리
- Domain은 데이터만 보관
- Service가 Order, Product 내부를 마음대로 조작

#### After: 풍부한 도메인 모델

```java
// Controller (변경 없음)
@PostMapping("/orders")
public OrderResponse createOrder(@RequestBody OrderRequest request) {
    return orderService.createOrder(request);
}

// Service (조율만 담당)
@Service
public class OrderService {
    public OrderResponse createOrder(OrderRequest request) {
        // 1. 도메인 객체 조회
        Product product = productRepository.findById(request.getProductId());
        
        // 2. 도메인 객체에게 행동 요청
        Order order = Order.create(
            request.getUserId(),
            product,
            request.getQuantity(),
            request.getCoupon()
        );
        
        // 3. 저장
        orderRepository.save(order);
        productRepository.save(product);
        
        return new OrderResponse(order);
    }
}

// Domain (로직을 스스로 처리)
public class Order {
    private Long userId;
    private Long productId;
    private Integer quantity;
    private Long totalPrice;
    
    // ✅ 생성 로직을 스스로 처리
    public static Order create(
        Long userId,
        Product product,
        Integer quantity,
        Coupon coupon
    ) {
        // 재고 확인을 Product에게 위임
        product.decreaseStock(quantity);
        
        // 가격 계산을 자신이 처리
        Long totalPrice = product.calculatePrice(quantity);
        if (coupon != null) {
            totalPrice = coupon.discount(totalPrice);
        }
        
        return new Order(userId, product.getId(), quantity, totalPrice);
    }
    
    private Order(Long userId, Long productId, Integer quantity, Long totalPrice) {
        this.userId = userId;
        this.productId = productId;
        this.quantity = quantity;
        this.totalPrice = totalPrice;
    }
    
    // getter만 제공 (setter 없음)
}

public class Product {
    private Long id;
    private Long price;
    private Integer stock;
    
    // ✅ 재고 차감을 스스로 처리
    public void decreaseStock(Integer quantity) {
        if (this.stock < quantity) {
            throw new OutOfStockException();
        }
        this.stock -= quantity;
    }
    
    // ✅ 가격 계산을 스스로 처리
    public Long calculatePrice(Integer quantity) {
        return this.price * quantity;
    }
    
    // getter만 제공 (setter 없음)
}

public class Coupon {
    private Long discountAmount;
    
    // ✅ 할인을 스스로 처리
    public Long discount(Long price) {
        return price - discountAmount;
    }
}
```

**개선 효과**:
- Service는 조율만 담당
- Domain이 자신의 로직을 스스로 처리
- 각 Domain의 책임이 명확함
- 변경 시 해당 Domain만 수정하면 됨

---

## 💭 생각해보기

### Q1. 절차지향이 무조건 나쁜가?

**A**: 아니다. 설계는 트레이드오프의 산물이다.

**절차지향이 적합한 경우**:
```java
// 단순한 데이터 변환
public class CsvConverter {
    public String convert(List<Data> dataList) {
        StringBuilder sb = new StringBuilder();
        for (Data data : dataList) {
            sb.append(data.toString()).append("\n");
        }
        return sb.toString();
    }
}
```

- 변경이 거의 없는 단순한 로직
- 빠른 구현이 필요한 일회성 코드
- 데이터 변환, 포맷팅 등의 유틸리티 기능

**객체지향이 필요한 경우**:
```java
// 복잡한 비즈니스 로직
public class Order {
    public Money calculateTotalPrice() {
        Money itemsPrice = calculateItemsPrice();
        Money shippingFee = calculateShippingFee();
        Money discount = calculateDiscount();
        return itemsPrice.plus(shippingFee).minus(discount);
    }
}
```

- 변경이 잦은 비즈니스 로직
- 복잡한 도메인 규칙
- 장기간 유지보수가 필요한 코드

**핵심**:
> 변경이 잦은 부분은 즉시 객체지향적으로 개선해야 하지만,
> 변경이 거의 없는 단순한 부분은 절차지향으로 작성해도 무방하다.

---

### Q2. 모든 객체를 자율적으로 만들어야 하는가?

**A**: 상황에 따라 판단해야 한다.

**Case 1: TicketOffice의 경우**
```java
// 자율적으로 만들면?
public class TicketOffice {
    public void sellTicketTo(Audience audience) {
        plusAmount(audience.buy(getTicket()));
    }
}
```

**문제**:
- `TicketOffice`가 `Audience`에 의존하게 됨
- 매표소는 관람객이라는 개념 없이도 존재할 수 있어야 함
- 도메인적으로 부자연스러움

**결정**: 자율성 < 결합도 제거

**Case 2: Entity와 VO의 경우**
```java
// Money (Value Object)
public class Money {
    private final BigDecimal amount;
    
    // ✅ 자율적으로 만드는 것이 좋음
    public Money plus(Money other) {
        return new Money(this.amount.add(other.amount));
    }
    
    public Money minus(Money other) {
        return new Money(this.amount.subtract(other.amount));
    }
}

// OrderId (Identifier)
public class OrderId {
    private final Long value;
    
    // ❌ 굳이 자율적으로 만들 필요 없음
    // 단순한 식별자는 데이터만 보관해도 충분
}
```

**판단 기준**:
```
자율성을 부여할 때:
✅ 도메인 규칙을 포함하는 경우
✅ 변경이 자주 발생하는 경우
✅ 자연스러운 책임 분배가 가능한 경우

자율성을 생략할 때:
✅ 단순한 데이터 보관만 하는 경우
✅ 새로운 의존성이 부자연스러운 경우
✅ 재사용성이 더 중요한 경우
```

---

### Q3. 실무에서 어떻게 적용하는가?

**A**: 점진적으로 개선하라.

#### Step 1: 문제 인식하기
```java
// ❌ 절차지향 코드 발견
public class UserService {
    public void updateUserInfo(User user, UserUpdateRequest request) {
        user.setName(request.getName());
        user.setEmail(request.getEmail());
        user.setPhone(request.getPhone());
        
        // 검증 로직이 Service에 있음
        if (!isValidEmail(user.getEmail())) {
            throw new InvalidEmailException();
        }
        if (!isValidPhone(user.getPhone())) {
            throw new InvalidPhoneException();
        }
        
        userRepository.save(user);
    }
}
```

#### Step 2: 작은 부분부터 리팩토링
```java
// ✅ 검증 로직을 User로 이동
public class User {
    private String name;
    private String email;
    private String phone;
    
    public void update(String name, String email, String phone) {
        validateEmail(email);
        validatePhone(phone);
        
        this.name = name;
        this.email = email;
        this.phone = phone;
    }
    
    private void validateEmail(String email) {
        if (!isValidEmail(email)) {
            throw new InvalidEmailException();
        }
    }
    
    private void validatePhone(String phone) {
        if (!isValidPhone(phone)) {
            throw new InvalidPhoneException();
        }
    }
}

public class UserService {
    public void updateUserInfo(User user, UserUpdateRequest request) {
        user.update(
            request.getName(),
            request.getEmail(),
            request.getPhone()
        );
        userRepository.save(user);
    }
}
```

#### Step 3: 테스트로 검증
```java
@Test
void 이메일_검증() {
    User user = new User("홍길동", "test@test.com", "010-1234-5678");
    
    assertThatThrownBy(() -> 
        user.update("홍길동", "invalid-email", "010-1234-5678")
    ).isInstanceOf(InvalidEmailException.class);
}
```

#### Step 4: 지속적으로 개선
```java
// 더 나아가기: Email, Phone을 VO로 분리
public class User {
    private String name;
    private Email email;      // ✅ VO로 분리
    private Phone phone;      // ✅ VO로 분리
    
    public void update(String name, Email email, Phone phone) {
        this.name = name;
        this.email = email;  // 검증은 VO가 담당
        this.phone = phone;  // 검증은 VO가 담당
    }
}

public class Email {
    private final String value;
    
    public Email(String value) {
        validate(value);
        this.value = value;
    }
    
    private void validate(String value) {
        if (!isValid(value)) {
            throw new InvalidEmailException();
        }
    }
}
```

**실전 팁**:
1. 한 번에 모든 것을 바꾸려 하지 말 것
2. getter 남발하는 부분부터 찾아서 개선
3. 테스트 코드로 변경 영향 확인
4. 팀원들과 코드 리뷰하며 점진적 개선
5. 레거시 코드는 변경이 필요한 부분만 리팩토링

---

<div align="center">

**[⬆ 목차로 돌아가기](../README.md)**

</div>

<div align="center">

**[Chapter 02 →](../chapter02/README.md)**

</div>
