# Appendix A. 계약에 의한 설계

> *"계약은 협력에 참여하는 객체들의 의무와 이익을 명시적으로 정의한다."*

## 📌 핵심 개념

이 장에서는 **계약에 의한 설계(Design By Contract, DBC)** 를 다룹니다. 인터페이스만으로는 표현하기 어려운 협력의 제약 조건과 부수효과를 명시적으로 문서화하고 실행 시점에 검증하는 방법을 학습합니다.

### 🎯 학습 목표

- 계약에 의한 설계의 개념과 필요성 이해하기
- 사전조건, 사후조건, 불변식의 의미 파악하기
- 계약 규칙과 리스코프 치환 원칙의 관계 이해하기
- 공변성과 반공변성의 개념 학습하기
- 계약을 통한 서브타이핑 검증 방법 익히기

---

## 📖 목차

1. [협력과 계약](#1-협력과-계약)
2. [계약에 의한 설계](#2-계약에-의한-설계)
3. [계약에 의한 설계와 서브타이핑](#3-계약에-의한-설계와-서브타이핑)
4. [핵심 정리](#4-핵심-정리)

---

## 1. 협력과 계약

### 1.1 부수효과를 명시적으로

#### 😰 인터페이스의 한계

**문제:**
```java
class Event {
    // 이 두 메서드의 호출 순서는?
    public boolean isSatisfied(RecurringSchedule schedule) { ... }
    public void reschedule(RecurringSchedule schedule) { ... }
}
```

**한계:**
```
인터페이스가 알려주는 것:
✅ 메서드 이름
✅ 파라미터 타입
✅ 반환 타입

인터페이스가 알려주지 못하는 것:
❌ 메서드 호출 순서
❌ 파라미터 제약 조건
❌ 실행 후 객체 상태
❌ 부수효과
```

#### 💡 주석의 문제

```java
/**
 * reschedule 메서드를 호출하기 전에
 * 반드시 isSatisfied 메서드를 호출해서
 * true를 반환받았는지 확인해야 한다.
 */
public void reschedule(RecurringSchedule schedule) {
    if (!isSatisfied(schedule)) {
        throw new IllegalArgumentException();
    }
    // ...
}
```

**주석의 한계:**
```
1. 구현과 동기화 보장 없음
   → 시간이 지나면 거짓말이 됨

2. 실행 시점 검증 불가
   → 런타임 오류 발생 가능

3. 일반 로직과 혼재
   → 제약 조건 파악 어려움

4. 문서화 도구와 연동 어려움
   → 자동화 불가능
```

#### ✨ 계약에 의한 설계 해결책

**C# Code Contracts 사용:**

```csharp
class Event
{
    public bool IsSatisfied(RecurringSchedule schedule) { ... }
    
    public void Reschedule(RecurringSchedule schedule)
    {
        // 제약 조건 명시적 표현!
        Contract.Requires(IsSatisfied(schedule));
        // ...
    }
}
```

**장점:**

```
✅ 명시적 표현
   → 제약 조건이 코드에 드러남

✅ 실행 가능한 검증
   → 런타임에 조건 체크

✅ 자동 문서화
   → 도구가 문서 생성

✅ 일반 로직과 분리
   → 가독성 향상
```

### 1.2 현실 세계의 계약

#### 📜 계약의 특성

```
┌─────────────────────────────────────────┐
│          계약 당사자 A                     │
│                                         │
│  의무: X를 제공                            │
│  권리: Y를 받음                            │
└─────────────┬───────────────────────────┘
              │
              │ 계약서
              │
┌─────────────▼───────────────────────────┐
│          계약 당사자 B                     │
│                                         │
│  의무: Y를 제공                            │
│  권리: X를 받음                            │
└─────────────────────────────────────────┘
```

**계약의 핵심 원칙:**

```
1. 각 계약 당사자는 계약으로부터 이익을 기대
2. 이익을 얻기 위해 의무를 이행
3. 한쪽의 의무 = 반대쪽의 권리
4. 당사자의 이익과 의무는 계약서에 문서화
5. 계약 위반 시 계약은 정상적으로 완료되지 않음
```

#### 🤝 객체 협력과 계약

```
┌─────────────────────────────────────────┐
│          클라이언트                        │
│                                         │
│  의무: 사전조건 만족                         │
│  권리: 사후조건 보장받음                      │
└─────────────┬───────────────────────────┘
              │
              │ 협력 (메시지 전송)
              │
┌─────────────▼───────────────────────────┐
│           서버                           │
│                                         │
│  의무: 사후조건 만족                        │
│  권리: 사전조건 만족됨                       │
└─────────────────────────────────────────┘
```

**버트란드 마이어의 통찰:**

```
"객체 협력을 계약의 관점에서 바라보면
협력에 참여하는 객체들의 책임이 명확해진다"

협력 = 계약
메시지 = 계약 이행 요청
반환값 = 계약 이행 결과
```

---

## 2. 계약에 의한 설계

### 2.1 계약의 구성 요소

#### 📋 세 가지 핵심 요소

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  1. 사전조건 (Precondition)                           │
│     메서드가 호출되기 위해 만족돼야 하는 조건                 │
│     → 클라이언트의 의무                                 │
│                                                     │
│  2. 사후조건 (Postcondition)                          │
│     메서드 실행 후 보장해야 하는 조건                       │
│     → 서버의 의무                                      │
│                                                     │
│  3. 불변식 (Invariant)                                │
│     항상 참이라고 보장되는 조건                            │
│     → 객체 생명주기 전반에 걸친 제약                       │
│                                                     │
└─────────────────────────────────────────────────────┘
```

### 2.2 사전조건 (Precondition)

#### 🎯 정의와 책임

```
사전조건:
메서드가 정상적으로 실행되기 위해
만족해야 하는 조건

책임: 클라이언트
→ 사전조건을 만족시키는 것은 메서드를 호출하는 쪽의 책임!
```

#### 💻 구현 예시

**영화 예매 시스템:**

```csharp
public class Screening
{
    private Movie movie;
    private int sequence;
    private DateTime whenScreened;

    public Reservation Reserve(Customer customer, int audienceCount)
    {
        // 사전조건 정의
        Contract.Requires(customer != null);
        Contract.Requires(audienceCount >= 1);
        
        return new Reservation(
            customer, 
            this, 
            calculateFee(audienceCount), 
            audienceCount
        );
    }
}
```

**사전조건 해석:**

```
Reserve 메서드를 호출하기 위해서는:

1. customer는 null이 아니어야 함
   → 유효한 고객 객체 필요

2. audienceCount는 1 이상이어야 함
   → 최소 1명 이상 예매

클라이언트가 이 조건을 만족시키지 못하면
→ ContractException 발생!
```

#### 🔍 사전조건의 중요성

```
사전조건을 만족시키지 못한 경우:

❌ 나쁜 방법:
메서드 내부에서 묵묵히 처리
→ 버그 발견 지연
→ 문제 원인 파악 어려움

✅ 좋은 방법:
최대한 빨리 실패 (Fail Fast)
→ 문제 즉시 발견
→ 원인 명확히 파악
```

**Java assert 사용 예:**

```java
public class BasicRatePolicy implements RatePolicy {
    @Override
    public Money calculateFee(List<Call> calls) {
        // 사전조건
        assert calls != null : "calls는 null이 될 수 없습니다";
        
        Money result = Money.ZERO;
        for (Call call : calls) {
            result = result.plus(calculateCallFee(call));
        }
        
        return result;
    }
}
```

### 2.3 사후조건 (Postcondition)

#### 🎯 정의와 책임

```
사후조건:
메서드 실행 후
클라이언트에게 보장해야 하는 조건

책임: 서버
→ 사후조건을 만족시키는 것은 메서드를 구현하는 쪽의 책임!
```

#### 📊 사후조건의 용도

```
1. 인스턴스 변수의 상태가 올바른지 검증
   예: balance >= 0

2. 파라미터의 값이 올바르게 변경됐는지 검증
   예: list.size() == oldSize + 1

3. 반환값이 올바른지 검증
   예: result != null
```

#### 💻 구현 예시

**반환값 검증:**

```csharp
public class Screening
{
    public Reservation Reserve(Customer customer, int audienceCount)
    {
        // 사전조건
        Contract.Requires(customer != null);
        Contract.Requires(audienceCount >= 1);
        
        // 사후조건: 반환값이 null이 아님을 보장
        Contract.Ensures(Contract.Result<Reservation>() != null);
        
        return new Reservation(
            customer, 
            this, 
            calculateFee(audienceCount), 
            audienceCount
        );
    }
}
```

**Java 예시:**

```java
public class AdditionalRatePolicy implements RatePolicy {
    private RatePolicy next;

    @Override
    public Money calculateFee(List<Call> calls) {
        // 사전조건
        assert calls != null;
        
        Money fee = next.calculateFee(calls);
        Money result = afterCalculated(fee);
        
        // 사후조건: 음수 요금이 아님을 보장
        assert result.isGreaterThanOrEqual(Money.ZERO);
        
        return result;
    }
}
```

#### 🤔 사후조건 정의의 어려움

**문제 1: 여러 return 문**

```java
// ❌ 나쁜 방법
public Money calculate(int amount) {
    if (amount < 0) {
        assert false : "음수 불가";
        return Money.ZERO;
    }
    
    Money result = Money.wons(amount * rate);
    assert result.isGreaterThanOrEqual(Money.ZERO);
    return result;
}

// ✅ 좋은 방법 (Code Contracts)
public Money Calculate(int amount)
{
    Contract.Ensures(Contract.Result<Money>() >= Money.ZERO);
    
    if (amount < 0) {
        return Money.ZERO;
    }
    
    return Money.Wons(amount * rate);
}
```

**문제 2: 실행 전후 값 비교**

```csharp
// Code Contracts는 OldValue 제공
public void Add(int value)
{
    Contract.Ensures(Count == Contract.OldValue(Count) + 1);
    // ...
}
```

### 2.4 불변식 (Invariant)

#### 🎯 정의와 특성

```
불변식:
메서드 실행 전과 실행 후에
항상 참이어야 하는 조건

특성:
- 객체의 생명주기 전반에 걸쳐 유지
- 모든 생성자 실행 후 만족
- 모든 public 메서드 실행 전후 만족
- 메서드 실행 중에는 일시적으로 위반 가능
```

#### 📊 불변식의 범위

```
┌─────────────────────────────────────────┐
│          생성자 실행                       │
│              ↓                          │
│        [불변식 만족] ✅                    │
│              ↓                          │
│        메서드 실행 전                       │
│        [불변식 만족] ✅                    │
│              ↓                          │
│        메서드 실행 중                       │
│        [불변식 위반 가능] ⚠️                │
│              ↓                           │
│        메서드 실행 후                       │
│        [불변식 만족] ✅                     │
└──────────────────────────────────────────┘
```

#### 💻 구현 예시

**영화 예매 시스템:**

```csharp
public class Screening
{
    private Movie movie;
    private int sequence;
    private DateTime whenScreened;

    // 불변식 정의
    [ContractInvariantMethod]
    private void Invariant()
    {
        Contract.Invariant(movie != null);
        Contract.Invariant(sequence >= 1);
        Contract.Invariant(whenScreened > DateTime.Now);
    }
    
    public Screening(Movie movie, int sequence, DateTime whenScreened)
    {
        this.movie = movie;
        this.sequence = sequence;
        this.whenScreened = whenScreened;
        // 생성자 실행 후 자동으로 Invariant() 호출됨
    }
    
    public Reservation Reserve(Customer customer, int audienceCount)
    {
        // 메서드 실행 전 자동으로 Invariant() 호출
        // ...
        // 메서드 실행 후 자동으로 Invariant() 호출
    }
}
```

**Java 예시:**

```java
public class AdditionalRatePolicy implements RatePolicy {
    private RatePolicy next;

    public AdditionalRatePolicy(RatePolicy next) {
        changeNext(next);
    }

    protected void changeNext(RatePolicy next) {
        this.next = next;
        // 불변식 체크
        assert this.next != null;
    }

    @Override
    public Money calculateFee(List<Call> calls) {
        // 불변식 체크 (메서드 실행 전)
        assert next != null;
        
        Money fee = next.calculateFee(calls);
        Money result = afterCalculated(fee);
        
        // 불변식 체크 (메서드 실행 후)
        assert next != null;
        
        return result;
    }
}
```

#### ⚠️ protected 변수의 위험

```java
// ❌ 위험한 설계
public class Parent {
    protected int value;  // 자식이 직접 수정 가능!
    
    protected void invariant() {
        assert value >= 0;
    }
}

public class Child extends Parent {
    public void dangerousMethod() {
        // 불변식 위반!
        value = -10;  
    }
}

// ✅ 안전한 설계
public class Parent {
    private int value;  // private으로 보호
    
    protected void setValue(int value) {
        assert value >= 0;  // 검증 후 설정
        this.value = value;
    }
    
    protected void invariant() {
        assert value >= 0;
    }
}
```

**핵심 원칙:**

```
불변식 유지를 위해:

✅ 인스턴스 변수는 private
✅ protected 메서드로 제어된 접근 제공
✅ 메서드 내에서 불변식 검증

❌ protected 인스턴스 변수 지양
```

---

## 3. 계약에 의한 설계와 서브타이핑

### 3.1 리스코프 치환 원칙과 계약

#### 🎯 핵심 통찰

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  리스코프 치환 원칙 (LSP):                              │
│                                                     │
│  서브타입은 슈퍼타입과 체결한                              │
│  계약을 준수해야 한다                                    │
│                                                     │
│  = 계약 규칙 + 가변성 규칙                               │
│                                                     │
└─────────────────────────────────────────────────────┘
```

#### 📊 두 가지 규칙

```
1. 계약 규칙 (Contract Rules)
   - 사전조건과 사후조건
   - 불변식
   - 예외

2. 가변성 규칙 (Variance Rules)
   - 파라미터 타입
   - 리턴 타입
```

### 3.2 계약 규칙

#### 1️⃣ 사전조건 규칙

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  서브타입에 더 강력한 사전조건을 정의할 수 없다                │
│                                                     │
│  사전조건은 완화만 가능! (약화 가능)                        │
│                                                     │
└─────────────────────────────────────────────────────┘
```

**이유:**

```
사전조건은 클라이언트의 의무

만약 서브타입이 사전조건을 강화하면:
→ 클라이언트는 더 많은 의무를 짊어짐
→ 슈퍼타입 기준으로 작성된 코드가 동작 안 함
→ 리스코프 치환 원칙 위반!
```

**예시:**

```java
// 슈퍼타입
public class BankAccount {
    public void withdraw(Money amount) {
        // 사전조건: amount > 0
        assert amount.isGreaterThan(Money.ZERO);
        // ...
    }
}

// ❌ 잘못된 서브타입 (사전조건 강화)
public class SavingsAccount extends BankAccount {
    @Override
    public void withdraw(Money amount) {
        // 사전조건 강화: amount > 0 AND amount <= balance
        assert amount.isGreaterThan(Money.ZERO);
        assert amount.isLessThanOrEqual(balance);  // 추가 제약!
        // ...
    }
}

// 문제 발생
BankAccount account = new SavingsAccount();
account.withdraw(Money.wons(1000000));  
// 슈퍼타입 관점에서는 OK
// 서브타입에서는 실패 가능 → LSP 위반!
```

**✅ 올바른 예시 (사전조건 완화):**

```java
// 슈퍼타입
public class BankAccount {
    public void withdraw(Money amount) {
        // 사전조건: amount > 0 AND amount <= balance
        assert amount.isGreaterThan(Money.ZERO);
        assert amount.isLessThanOrEqual(balance);
        // ...
    }
}

// ✅ 올바른 서브타입 (사전조건 완화)
public class PremiumAccount extends BankAccount {
    @Override
    public void withdraw(Money amount) {
        // 사전조건 완화: amount > 0만 체크 (마이너스 통장)
        assert amount.isGreaterThan(Money.ZERO);
        // balance 체크 제거 → 완화!
        // ...
    }
}

// 문제 없음
BankAccount account = new PremiumAccount();
account.withdraw(Money.wons(1000000));
// 슈퍼타입보다 관대하므로 OK!
```

#### 2️⃣ 사후조건 규칙

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  서브타입에 더 완화된 사후조건을 정의할 수 없다                │
│                                                     │
│  사후조건은 강화만 가능! (더 엄격하게 가능)                  │
│                                                     │
└─────────────────────────────────────────────────────┘
```

**이유:**

```
사후조건은 서버의 의무

만약 서브타입이 사후조건을 완화하면:
→ 클라이언트가 기대한 것보다 적은 이익
→ 계약 위반!
→ 리스코프 치환 원칙 위반!
```

**예시:**

```java
// 슈퍼타입
public class Discounter {
    public Money discount(Money price) {
        Money result = calculateDiscount(price);
        // 사후조건: result >= 0
        assert result.isGreaterThanOrEqual(Money.ZERO);
        return result;
    }
}

// ❌ 잘못된 서브타입 (사후조건 완화)
public class NegativeDiscounter extends Discounter {
    @Override
    public Money discount(Money price) {
        Money result = calculateDiscount(price);
        // 사후조건 완화: 음수도 허용
        // assert 제거 → 완화!
        return result;  // 음수 반환 가능!
    }
}

// 문제 발생
Discounter discounter = new NegativeDiscounter();
Money result = discounter.discount(Money.wons(10000));
// 클라이언트는 양수를 기대
// 하지만 음수가 반환될 수 있음 → LSP 위반!
```

**✅ 올바른 예시 (사후조건 강화):**

```java
// 슈퍼타입
public class Discounter {
    public Money discount(Money price) {
        Money result = calculateDiscount(price);
        // 사후조건: result >= 0
        assert result.isGreaterThanOrEqual(Money.ZERO);
        return result;
    }
}

// ✅ 올바른 서브타입 (사후조건 강화)
public class PositiveDiscounter extends Discounter {
    @Override
    public Money discount(Money price) {
        Money result = calculateDiscount(price);
        // 사후조건 강화: result > 0 (0 제외)
        assert result.isGreaterThan(Money.ZERO);
        return result;
    }
}

// 문제 없음
Discounter discounter = new PositiveDiscounter();
Money result = discounter.discount(Money.wons(10000));
// 클라이언트가 기대한 것보다 더 엄격 → OK!
```

#### 3️⃣ 불변식 규칙

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  슈퍼타입의 불변식은                                     │
│  서브타입에서도 반드시 유지되어야 한다                       │
│                                                     │
└─────────────────────────────────────────────────────┘
```

**예시:**

```java
// 슈퍼타입
public class Rectangle {
    protected int width;
    protected int height;
    
    protected void invariant() {
        assert width > 0;
        assert height > 0;
    }
}

// ❌ 잘못된 서브타입 (불변식 위반)
public class Square extends Rectangle {
    @Override
    public void setWidth(int width) {
        this.width = width;
        this.height = width;  // 정사각형 유지
    }
    
    @Override
    public void setHeight(int height) {
        this.width = height;  // 정사각형 유지
        this.height = height;
    }
    
    // 불변식 추가 (부모보다 강화)
    protected void invariant() {
        super.invariant();
        assert width == height;  // 추가 제약!
    }
}

// 문제 발생
Rectangle rect = new Square();
rect.setWidth(10);
rect.setHeight(20);
// 클라이언트는 독립적인 설정을 기대
// 하지만 정사각형 제약으로 실패 → LSP 위반!
```

#### 💡 일찍 실패하기 (Fail Fast)

```
문제 발생 시:

❌ 나쁜 방법:
오류를 숨기고 계속 진행
→ 나중에 이상한 곳에서 실패
→ 원인 파악 어려움

✅ 좋은 방법:
즉시 실패하도록 만들기
→ 문제 발생 지점에서 예외
→ 원인 명확히 파악

assert를 사용하면
문제가 발생한 바로 그 위치에서
프로그램이 중단되므로
디버깅이 쉬워진다!
```

### 3.3 가변성 규칙

#### 📊 공변성, 반공변성, 무공변성

```
S가 T의 서브타입일 때:

┌─────────────────────────────────────────┐
│  공변성 (Covariance)                      │
│  S와 T의 서브타입 관계 유지                   │
│  서브타입 S가 슈퍼타입 T 대신 사용 가능          │
│  → 리스코프 치환 원칙                        │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│  반공변성 (Contravariance)                │
│  S와 T의 서브타입 관계 역전                  │
│  슈퍼타입 T가 서브타입 S 대신 사용 가능         │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│  무공변성 (Invariance)                    │
│  S와 T 사이에 관계 없음                     │
│  S 대신 T, T 대신 S 모두 불가               │
└─────────────────────────────────────────┘
```

#### 1️⃣ 리턴 타입 공변성

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  서브타입의 리턴 타입은 공변성을 가져야 한다                   │
│                                                     │
│  = 슈퍼타입의 리턴 타입의 서브타입을                        │
│    리턴할 수 있다                                      │
│                                                     │
└─────────────────────────────────────────────────────┘
```

**이유:**

```
슈퍼타입 대신 서브타입을 반환하는 것은
더 강력한 사후조건을 정의하는 것과 같음

클라이언트는 슈퍼타입을 기대
→ 서브타입을 받으면 더 구체적
→ 문제 없음!
```

**예시:**

```java
// 타입 계층
class Publisher { }
class IndependentPublisher extends Publisher { }

class Book {
    private Publisher publisher;
    
    public Book(Publisher publisher) {
        this.publisher = publisher;
    }
}

class Magazine extends Book {
    public Magazine(Publisher publisher) {
        super(publisher);
    }
}

// 리턴 타입 공변성
class BookStall {
    // 슈퍼타입 메서드
    public Book sell(IndependentPublisher publisher) {
        return new Book(publisher);
    }
}

class MagazineStore extends BookStall {
    // ✅ Java는 리턴 타입 공변성 지원
    @Override
    public Magazine sell(IndependentPublisher publisher) {
        return new Magazine(publisher);
    }
}

// 사용
BookStall store = new MagazineStore();
Book book = store.sell(new IndependentPublisher());
// Book을 기대했는데 Magazine을 받음
// Magazine은 Book의 서브타입이므로 OK!
```

**시각화:**

```
클래스 계층:          리턴 타입 계층:

BookStall            Book
    ↑                  ↑
    │                  │
    │                  │
MagazineStore      Magazine

→ 방향이 같음 = 공변성!
```

**언어별 지원:**

```
✅ Java: 리턴 타입 공변성 지원
✅ C#: 리턴 타입 공변성 지원 (C# 9.0+)
✅ Scala: 리턴 타입 공변성 지원
❌ C# (구버전): 무공변성
```

#### 2️⃣ 파라미터 타입 반공변성

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  서브타입의 메서드 파라미터는                              │
│  반공변성을 가져야 한다                                   │
│                                                     │
│  = 슈퍼타입의 파라미터 타입의 슈퍼타입을                     │
│    파라미터로 받을 수 있다                               │
│                                                     │
└─────────────────────────────────────────────────────┘
```

**이유:**

```
서브타입 대신 슈퍼타입을 파라미터로 받는 것은
더 약한 사전조건을 정의하는 것과 같음

클라이언트는 특정 타입을 전달
→ 서버가 더 일반적인 타입을 받으면
→ 당연히 처리 가능!
```

**예시 (이론적):**

```java
// 타입 계층
class Publisher { }
class IndependentPublisher extends Publisher { }

// 파라미터 타입 반공변성 (Java는 지원 안 함!)
class BookStall {
    // 슈퍼타입 메서드
    public Book sell(IndependentPublisher publisher) {
        return new Book(publisher);
    }
}

// 이론적으로는 이렇게 되어야 함
class MagazineStore extends BookStall {
    // ❌ Java에서는 오버라이딩 안됨 (오버로딩으로 간주)
    @Override
    public Magazine sell(Publisher publisher) {  // 더 일반적인 타입
        return new Magazine(publisher);
    }
}

// 사용 (이론적)
BookStall store = new MagazineStore();
store.sell(new IndependentPublisher());
// IndependentPublisher를 전달
// Publisher를 받는 메서드가 처리 → OK!
```

**시각화:**

```
클래스 계층:          파라미터 타입 계층:

BookStall            IndependentPublisher
    ↑                         ↓
    │                         │
    │                         │
MagazineStore            Publisher

→ 방향이 반대 = 반공변성!
```

**언어별 지원:**

```
❌ Java: 파라미터 타입 반공변성 미지원
❌ C#: 파라미터 타입 반공변성 미지원
✅ Scala: 함수 타입에서 반공변성 지원
```

**Scala 예시:**

```scala
class Publisher
class IndependentPublisher extends Publisher

class Book(val publisher: Publisher)
class Magazine(publisher: Publisher) extends Book(publisher)

class Orderer {
    var book: Book = null
    
    def order(store: IndependentPublisher => Book): Unit = {
        book = store(new IndependentPublisher())
    }
}

// 사용
val orderer = new Orderer()

// ✅ 정확히 일치
orderer.order((publisher: IndependentPublisher) => new Book(publisher))

// ✅ 파라미터 타입 반공변성 (더 일반적인 타입)
orderer.order((publisher: Publisher) => new Magazine(publisher))
```

#### 3️⃣ 예외 규칙

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  서브타입은 슈퍼타입이 발생시키는 예외와                      │
│  다른 타입의 예외를 발생시켜서는 안 된다                     │
│                                                     │
└─────────────────────────────────────────────────────┘
```

**이유:**

```
클라이언트는 슈퍼타입 기준으로
예외 처리를 준비함

예상치 못한 예외가 발생하면
→ 처리할 수 없음
→ 프로그램 실패!
```

**예시:**

```java
// 슈퍼타입
class Bird {
    public void fly() throws FlyException {
        // ...
    }
}

// ❌ 잘못된 서브타입
class Penguin extends Bird {
    @Override
    public void fly() throws UnsupportedOperationException {
        throw new UnsupportedOperationException("펭귄은 날 수 없음");
    }
}

// 문제 발생
Bird bird = new Penguin();
try {
    bird.fly();
} catch (FlyException e) {
    // FlyException만 처리
}
// UnsupportedOperationException은 처리 못함 → 프로그램 실패!

// ✅ 올바른 방법
class Penguin extends Bird {
    @Override
    public void fly() throws FlyException {
        throw new FlyException("펭귄은 날 수 없음");
    }
}
```

### 3.4 함수 타입과 서브타이핑

#### 🎯 함수 타입의 서브타이핑

**함수 타입:**

```scala
// 함수 시그니처
type F = (A) => B

A: 파라미터 타입
B: 리턴 타입
```

**서브타입 관계:**

```
F1 = (A1) => B1
F2 = (A2) => B2

F2가 F1의 서브타입이 되려면:

1. A2는 A1의 슈퍼타입 (반공변성)
2. B2는 B1의 서브타입 (공변성)
```

**예시:**

```scala
// 타입 정의
type BasicStore = IndependentPublisher => Book
type MagazineStore = Publisher => Magazine

// 타입 관계
Publisher >: IndependentPublisher  // Publisher가 더 일반적
Magazine <: Book                   // Magazine이 더 구체적

// 따라서
MagazineStore <: BasicStore  // 서브타입!

// 사용 가능
def order(store: IndependentPublisher => Book): Unit = {
    val book = store(new IndependentPublisher())
}

// ✅ 둘 다 가능
order((p: IndependentPublisher) => new Book(p))
order((p: Publisher) => new Magazine(p))  // 서브타입!
```

**시각화:**

```
함수 타입의 서브타이핑:

    파라미터 (반공변)      리턴 타입 (공변)
         ↓                    ↑
    Publisher            Magazine
         ↑                    ↓
IndependentPublisher        Book

BasicStore = IndependentPublisher => Book
MagazineStore = Publisher => Magazine

MagazineStore <: BasicStore
```

---

## 4. 핵심 정리

### 🎯 계약에 의한 설계의 본질

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  계약에 의한 설계:                                      │
│  협력에 필요한 제약과 부수효과를                            │
│  명시적으로 정의하고 문서화하는 기법                         │
│                                                     │
│  목적:                                               │
│  - 인터페이스만으로 전달하기 어려운 정보를 명시                │
│  - 런타임에 제약 조건 검증                                │
│  - 자동 문서화                                         │
│  - 서브타이핑 올바름 보장                                 │
│                                                     │
└─────────────────────────────────────────────────────┘
```

### 📊 계약의 세 가지 요소

| 요소 | 정의 | 책임 | 검증 시점 |
|------|------|------|----------|
| **사전조건** | 호출 전 만족 조건 | 클라이언트 | 메서드 시작 |
| **사후조건** | 실행 후 보장 조건 | 서버 | 메서드 종료 |
| **불변식** | 항상 유지 조건 | 서버 | 메서드 전후 |

### 🔑 계약 규칙과 LSP

#### 계약 규칙

```
1. 서브타입은 더 강력한 사전조건 정의 불가
   → 사전조건은 완화만 가능

2. 서브타입은 더 완화된 사후조건 정의 불가
   → 사후조건은 강화만 가능

3. 슈퍼타입의 불변식은 서브타입에서도 유지
   → 불변식은 반드시 준수

4. 서브타입은 다른 타입의 예외 발생 불가
   → 예외는 슈퍼타입과 동일 계층
```

#### 가변성 규칙

```
1. 서브타입의 리턴 타입은 공변성
   → 슈퍼타입의 리턴 타입의 서브타입 허용

2. 서브타입의 파라미터는 반공변성
   → 슈퍼타입의 파라미터의 슈퍼타입 허용
   (대부분 언어에서 미지원)
```

### 💡 실전 가이드

#### Do's ✅

```
1. 사전조건으로 파라미터 검증
   Contract.Requires(parameter != null)

2. 사후조건으로 결과 보장
   Contract.Ensures(result.isValid())

3. 불변식으로 객체 상태 유지
   Contract.Invariant(balance >= 0)

4. 일찍 실패하기 (Fail Fast)
   조건 위반 시 즉시 예외 발생

5. private 변수 + protected 메서드
   불변식 유지를 위한 안전한 설계

6. 계약 위반 시 명확한 메시지
   assert condition : "상세한 설명"
```

#### Don'ts ❌

```
1. 사전조건 강화 금지
   서브타입이 더 엄격한 조건 요구 X

2. 사후조건 완화 금지
   서브타입이 덜 보장 X

3. 불변식 위반 금지
   메서드 실행 전후 반드시 만족

4. 예상 밖 예외 발생 금지
   클라이언트가 처리할 수 없는 예외 X

5. protected 인스턴스 변수 사용 금지
   자식 클래스가 불변식 쉽게 위반

6. 계약 없이 서브타이핑 하지 말기
   LSP 위반 가능성 높음
```

### 🎓 언어별 지원 현황

#### Java

```
✅ 지원:
- assert를 통한 사전조건/사후조건/불변식
- 리턴 타입 공변성
- 예외 계층

❌ 미지원:
- 파라미터 타입 반공변성
- 전용 DBC 라이브러리 (내장)

권장:
- assert 적극 활용
- @Valid 어노테이션 (Bean Validation)
- 커스텀 검증 로직
```

#### C#

```
✅ 지원:
- Code Contracts 라이브러리
- Contract.Requires (사전조건)
- Contract.Ensures (사후조건)
- Contract.Invariant (불변식)
- 리턴 타입 공변성 (C# 9.0+)

❌ 미지원:
- 파라미터 타입 반공변성

권장:
- Code Contracts 라이브러리 사용
- 계약 자동 문서화 도구 활용
```

#### Scala

```
✅ 지원:
- require (사전조건)
- ensuring (사후조건)
- 함수 타입에서 반공변성
- 리턴 타입 공변성

권장:
- 함수형 접근과 결합
- 타입 시스템 최대 활용
```

### 📈 계약과 설계 원칙

```
계약에 의한 설계 ←→ SOLID 원칙

LSP (리스코프 치환 원칙):
  = 계약 규칙 + 가변성 규칙

OCP (개방-폐쇄 원칙):
  계약을 통한 안전한 확장

DIP (의존성 역전 원칙):
  추상화의 계약 준수
```

### 🌟 핵심 교훈

```
1. 계약은 협력을 명확하게 만든다
   → 의도를 드러내는 인터페이스의 확장

2. 사전조건은 클라이언트의 책임
   → 완화만 가능 (서브타입)

3. 사후조건은 서버의 책임
   → 강화만 가능 (서브타입)

4. 불변식은 객체의 생명줄
   → 항상 유지되어야 함

5. 일찍 실패하기
   → 문제 발생 즉시 알림

6. 계약 + LSP = 안전한 서브타이핑
   → 치환 가능성 보장

7. 공변성/반공변성 이해 필수
   → 타입 안전성 확보

8. 계약은 실행 가능한 문서
   → 코드와 함께 진화
```

### 🎯 마무리

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  "계약에 의한 설계는                                    │
│   협력에 참여하는 객체들의 책임을                          │
│   명시적으로 정의하고 검증하는 강력한 도구다"                 │
│                                                     │
│  - Bertrand Meyer                                   │
│                                                     │
│  계약을 통해:                                          │
│  - 인터페이스가 명확해지고                                │
│  - 버그가 일찍 발견되며                                  │
│  - 서브타이핑이 안전해지고                                │
│  - 코드가 문서화된다                                     │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

## 🔗 연결고리

### 이전 장과의 연결
- **Chapter 13**: 서브클래싱과 서브타이핑 → 계약으로 구체화
- **LSP**: 치환 가능성 → 계약 준수로 보장
- **인터페이스**: 시그니처 → 계약으로 확장

### 실무로의 확장
- **다음 단계**:
  - 언어별 DBC 도구 활용
  - 계약 기반 테스트 작성
  - 불변식 설계 패턴 적용
  - 타입 시스템과 계약 결합

---

<div align="center">

**[⬆ 목차로 돌아가기](../README.md)**

</div>

<div align="center">

**[⬅️ 이전: Chapter 15](../chapter15/README.md)** | **[다음: Appendix B ➡️](../appendixB/README.md)**

</div>
