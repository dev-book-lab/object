# Appendix B. 타입 계층의 구현

> *"클래스가 아니라 타입에 집중하라. 중요한 것은 행동이다."*

## 📌 핵심 개념

이 장에서는 **타입 계층을 구현하는 다양한 방법**을 다룹니다. 클래스, 인터페이스, 추상 클래스, 덕 타이핑, 믹스인 등 여러 구현 기법의 장단점과 적용 시나리오를 학습합니다.

### 🎯 학습 목표

- 타입과 클래스의 차이 이해하기
- 다양한 타입 계층 구현 방법 학습하기
- 각 방법의 장단점과 트레이드오프 파악하기
- 인터페이스와 추상 클래스의 조합 이해하기
- 덕 타이핑의 개념과 특성 학습하기
- 믹스인을 통한 코드 재사용 이해하기

---

## 📖 목차

1. [타입과 클래스](#1-타입과-클래스)
2. [다양한 타입 계층 구현 방법](#2-다양한-타입-계층-구현-방법)
3. [추상 클래스와 인터페이스 결합하기](#3-추상-클래스와-인터페이스-결합하기)
4. [덕 타이핑](#4-덕-타이핑)
5. [믹스인과 타입 계층](#5-믹스인과-타입-계층)
6. [핵심 정리](#6-핵심-정리)

---

## 1. 타입과 클래스

### 1.1 타입 vs 클래스

#### 🎯 핵심 차이

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  타입 (Type):                                        │
│  객체의 퍼블릭 인터페이스                                 │
│  → 객체가 외부에 제공하는 행동의 집합                       │
│                                                     │
│  클래스 (Class):                                     │
│  객체의 구현                                          │
│  → 내부 상태와 메서드 구현을 정의                          │
│                                                     │
└─────────────────────────────────────────────────────┘
```

**비교:**

| 측면 | 타입 | 클래스 |
|------|------|--------|
| **관심사** | 행동 (What) | 구현 (How) |
| **범위** | 퍼블릭 인터페이스 | 내부 상태 + 메서드 |
| **추상화** | 높음 | 낮음 |
| **변경** | 어려움 | 상대적으로 쉬움 |

#### 💡 타입 계층의 본질

```
타입 계층이란:

동일한 메시지에 대한
행동 호환성을 전제로 한 구조

핵심:
- 행동의 일관성
- 치환 가능성
- 다형성
```

**중요한 사실:**

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  타입 계층을 구현한다고 해서                               │
│  서브타이핑 관계가 자동으로 보장되는 것은 아니다                │
│                                                     │
│  → 리스코프 치환 원칙(LSP)을 준수해야 함!                   │
│                                                     │
└─────────────────────────────────────────────────────┘
```

### 1.2 타입을 구현하는 방법

```
타입 계층 구현 방법:

1. 클래스 상속
2. 인터페이스 구현
3. 추상 클래스 상속
4. 인터페이스 + 추상 클래스
5. 덕 타이핑
6. 믹스인

→ 각 방법의 장단점과 트레이드오프를 이해해야 함!
```

---

## 2. 다양한 타입 계층 구현 방법

### 2.1 클래스를 이용한 타입 계층 구현

#### 🎯 클래스 = 타입 + 구현

```
클래스는:
- 객체의 타입을 정의
- 동시에 구현도 정의

→ 타입과 구현이 강하게 결합!
```

**예시:**

```java
// 클래스로 타입 정의
public class SalariedEmployee {
    private String name;
    private Money basePay;

    public SalariedEmployee(String name, Money basePay) {
        this.name = name;
        this.basePay = basePay;
    }

    public Money calculatePay(double taxRate) {
        return basePay.minus(basePay.times(taxRate));
    }
}

public class HourlyEmployee {
    private String name;
    private Money basePay;
    private int timeCard;

    public HourlyEmployee(String name, Money basePay, int timeCard) {
        this.name = name;
        this.basePay = basePay;
        this.timeCard = timeCard;
    }

    public Money calculatePay(double taxRate) {
        return basePay.times(timeCard)
                      .minus(basePay.times(timeCard).times(taxRate));
    }
}
```

**문제점:**

```
1. 타입과 구현의 강한 결합
   → 구현 변경 시 타입도 영향받음

2. 다중 상속 불가
   → 하나의 객체가 여러 타입을 가질 수 없음

3. 코드 중복
   → 유사한 구현을 공유하기 어려움

4. 유연성 부족
   → 다양한 구현 방법 제약
```

#### ⚠️ 구체 클래스 상속의 위험

```java
// ❌ 나쁜 방법: 구체 클래스 상속
public class Manager extends SalariedEmployee {
    private Money bonus;
    
    public Manager(String name, Money basePay, Money bonus) {
        super(name, basePay);
        this.bonus = bonus;
    }
    
    @Override
    public Money calculatePay(double taxRate) {
        // 부모 구현에 강하게 결합됨!
        return super.calculatePay(taxRate).plus(bonus);
    }
}
```

**문제:**

```
구체 클래스 상속:
→ 부모 클래스의 구현에 강하게 결합
→ 부모 변경 시 자식에 파급 효과
→ 캡슐화 위반
→ 유지보수 어려움
```

### 2.2 인터페이스를 이용한 타입 계층 구현

#### 🎯 타입과 구현의 분리

```
인터페이스:
- 타입만 정의 (구현 X)
- 퍼블릭 인터페이스만 명시
- 구현은 클래스에 위임

→ 타입과 구현 완전 분리!
```

**예시:**

```java
// 타입 정의
public interface Employee {
    Money calculatePay(double taxRate);
}

// 구현 1
public class SalariedEmployee implements Employee {
    private String name;
    private Money basePay;

    public SalariedEmployee(String name, Money basePay) {
        this.name = name;
        this.basePay = basePay;
    }

    @Override
    public Money calculatePay(double taxRate) {
        return basePay.minus(basePay.times(taxRate));
    }
}

// 구현 2
public class HourlyEmployee implements Employee {
    private String name;
    private Money basePay;
    private int timeCard;

    public HourlyEmployee(String name, Money basePay, int timeCard) {
        this.name = name;
        this.basePay = basePay;
        this.timeCard = timeCard;
    }

    @Override
    public Money calculatePay(double taxRate) {
        return basePay.times(timeCard)
                      .minus(basePay.times(timeCard).times(taxRate));
    }
}
```

**사용:**

```java
public class TaxOffice {
    public Money calculate(Employee employee, double taxRate) {
        // 인터페이스에만 의존!
        return employee.calculatePay(taxRate);
    }
}

// 다형성
TaxOffice office = new TaxOffice();
office.calculate(new SalariedEmployee("Alice", Money.wons(3000000)), 0.1);
office.calculate(new HourlyEmployee("Bob", Money.wons(15000), 160), 0.1);
```

#### 💎 인터페이스의 장점

```
1. 타입과 구현 분리
   ✅ 구현 변경이 타입에 영향 없음

2. 다중 타입 구현 가능
   ✅ 하나의 클래스가 여러 인터페이스 구현

3. 낮은 결합도
   ✅ 클라이언트가 인터페이스에만 의존

4. 높은 유연성
   ✅ 새로운 구현 추가 용이
```

**다중 인터페이스 구현 예시:**

```java
// 게임 객체 타입 계층
public interface GameObject {
    String getName();
}

public interface Displayable extends GameObject {
    Point getPosition();
    void update(Graphics graphics);
}

public interface Collidable extends Displayable {
    boolean collideWith(Collidable other);
}

public interface Effect extends GameObject {
    void activate();
}

// 하나의 클래스가 여러 타입 구현
public class Explosion implements Displayable, Effect {
    @Override
    public String getName() { return "Explosion"; }
    
    @Override
    public Point getPosition() { return new Point(0, 0); }
    
    @Override
    public void update(Graphics graphics) { /* ... */ }
    
    @Override
    public void activate() { /* ... */ }
}

// Player는 Collidable
public class Player implements Collidable {
    @Override
    public String getName() { return "Player"; }
    
    @Override
    public Point getPosition() { return new Point(0, 0); }
    
    @Override
    public void update(Graphics graphics) { /* ... */ }
    
    @Override
    public boolean collideWith(Collidable other) { return false; }
}
```

**타입 계층 시각화:**

```
        GameObject
            ↑
            │
    ┌───────┴───────┐
    │               │
Displayable      Effect
    ↑               ↑
    │               │
Collidable      (Explosion은 둘 다 구현)
    ↑
    │
Player, Monster
```

#### 🔍 인터페이스로 알 수 있는 사실

```
1. 여러 클래스가 동일한 타입 구현 가능
   Employee ← SalariedEmployee
           ← HourlyEmployee

2. 하나의 클래스가 여러 타입 구현 가능
   Explosion → Displayable
            → Effect
```

#### ⚠️ 인터페이스의 한계

```
❌ 코드 중복 문제

인터페이스는 구현을 제공하지 않으므로
각 클래스가 동일한 로직을 중복 구현해야 함

→ Java 8 이전의 한계
```

### 2.3 추상 클래스를 이용한 타입 계층 구현

#### 🎯 구현 공유 + 결합도 낮추기

```
추상 클래스:
- 타입 정의 (추상 메서드)
- 일부 구현 제공 (구체 메서드)
- 인스턴스 생성 불가

→ 구현 공유하면서도 의존성 역전!
```

**예시:**

```java
// 추상 클래스로 타입 정의
public abstract class DiscountPolicy {
    private List<DiscountCondition> conditions;

    public DiscountPolicy(DiscountCondition... conditions) {
        this.conditions = Arrays.asList(conditions);
    }

    // 구체 메서드 (구현 공유)
    public Money calculateDiscountAmount(Screening screening) {
        for (DiscountCondition each : conditions) {
            if (each.isSatisfiedBy(screening)) {
                return getDiscountAmount(screening);
            }
        }
        return Money.ZERO;
    }

    // 추상 메서드 (타입 정의)
    abstract protected Money getDiscountAmount(Screening screening);
}

// 구현 1
public class AmountDiscountPolicy extends DiscountPolicy {
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

// 구현 2
public class PercentDiscountPolicy extends DiscountPolicy {
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

#### 💎 추상 클래스의 특징

**1. 의존성 역전 원칙 (DIP) 준수**

```
┌──────────────────────────┐
│   DiscountPolicy         │  ← 추상 클래스
│   (추상)                  │
│                          │
│   calculateDiscountAmount│  ← 구체 메서드
│   getDiscountAmount      │  ← 추상 메서드
└────────┬─────────────────┘
         │ depends on
         ▼
    (추상 메서드)
         ▲
         │ implements
         │
┌────────┴─────────────────┐
│   AmountDiscountPolicy   │
│   (구체)                  │
└──────────────────────────┘

부모와 자식 모두 추상 메서드에 의존!
→ 의존성 역전
```

**2. 상속을 염두에 둔 설계**

```
구체 클래스 상속:
❌ 우연한 상속
❌ 구현 세부사항 노출
❌ 취약한 기반 클래스 문제

추상 클래스 상속:
✅ 의도된 상속
✅ 추상화를 통한 확장 포인트 제공
✅ 안전한 확장
```

**3. 인스턴스 생성 불가**

```java
// ❌ 컴파일 에러
DiscountPolicy policy = new DiscountPolicy() { ... };

// ✅ 서브클래스를 통해서만 인스턴스화
DiscountPolicy policy = new AmountDiscountPolicy(...);
```

#### 📊 구체 클래스 vs 추상 클래스

| 측면 | 구체 클래스 상속 | 추상 클래스 상속 |
|------|----------------|-----------------|
| **의존 대상** | 구체적 구현 | 추상 메서드 |
| **결합도** | 높음 | 낮음 |
| **확장성** | 낮음 | 높음 |
| **목적** | 코드 재사용 | 타입 계층 정의 |
| **DIP** | 위반 | 준수 |

---

## 3. 추상 클래스와 인터페이스 결합하기

### 3.1 골격 구현 추상 클래스

#### 🎯 최선의 조합

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  인터페이스: 타입 정의                                   │
│  추상 클래스: 공통 구현 제공                              │
│                                                     │
│  → 골격 구현 추상 클래스                                 │
│    (Skeletal Implementation Abstract Class)         │
│                                                     │
└─────────────────────────────────────────────────────┘
```

**패턴:**

```
1. 인터페이스로 타입 정의
2. 추상 클래스로 기본 구현 제공
3. 구체 클래스로 특화된 구현

→ 다중 타입 + 코드 재사용!
```

#### 💻 구현 예시

**Java 8 이전 방식:**

```java
// 1. 인터페이스로 타입 정의
public interface DiscountPolicy {
    Money calculateDiscountAmount(Screening screening);
}

// 2. 골격 구현 추상 클래스
public abstract class DefaultDiscountPolicy implements DiscountPolicy {
    private List<DiscountCondition> conditions;

    public DefaultDiscountPolicy(DiscountCondition... conditions) {
        this.conditions = Arrays.asList(conditions);
    }

    // 공통 구현
    @Override
    public Money calculateDiscountAmount(Screening screening) {
        for (DiscountCondition each : conditions) {
            if (each.isSatisfiedBy(screening)) {
                return getDiscountAmount(screening);
            }
        }
        return Money.ZERO;
    }

    // 확장 포인트
    abstract protected Money getDiscountAmount(Screening screening);
}

// 3. 구체 클래스
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

// 4. 다른 계층에서도 인터페이스 구현 가능
public class CustomDiscountPolicy implements DiscountPolicy {
    // 골격 구현 사용 안 함
    // 완전히 다른 방식으로 구현
    
    @Override
    public Money calculateDiscountAmount(Screening screening) {
        // 자체 구현
        return Money.ZERO;
    }
}
```

**구조:**

```
<<interface>>
DiscountPolicy
    ↑
    │ implements
    ├──────────────────────┐
    │                      │
DefaultDiscountPolicy  CustomDiscountPolicy
(추상 클래스)           (독립 구현)
    ↑
    │ extends
    ├──────────────┐
    │              │
AmountDiscount  PercentDiscount
Policy          Policy
```

#### 💎 골격 구현의 장점

**1. 다양한 구현 방법 지원**

```
상황 A: 기본 구현 활용
→ DefaultDiscountPolicy 상속

상황 B: 완전히 다른 방식
→ DiscountPolicy 직접 구현

상황 C: 다른 부모 클래스 존재
→ DiscountPolicy 구현
→ DefaultDiscountPolicy 로직을 위임으로 활용
```

**2. 기존 클래스 확장 용이**

```java
// 이미 다른 부모가 있는 경우
public class SpecialMovie extends AbstractMovie {
    // 이미 상속 중
}

// 인터페이스 추가로 새로운 타입 확장!
public class SpecialMovie extends AbstractMovie 
                           implements DiscountPolicy {
    // DiscountPolicy 타입으로도 사용 가능
    
    @Override
    public Money calculateDiscountAmount(Screening screening) {
        // 구현
        return Money.ZERO;
    }
}
```

### 3.2 Java 8 디폴트 메서드

#### 🎯 인터페이스에 구현 추가

```java
// 인터페이스에 디폴트 메서드
public interface DiscountPolicy {
    // 디폴트 메서드 (구현 제공)
    default Money calculateDiscountAmount(Screening screening) {
        for (DiscountCondition each : getConditions()) {
            if (each.isSatisfiedBy(screening)) {
                return getDiscountAmount(screening);
            }
        }
        return Money.ZERO;
    }

    // 추상 메서드
    List<DiscountCondition> getConditions();
    Money getDiscountAmount(Screening screening);
}

// 구현 클래스
public class AmountDiscountPolicy implements DiscountPolicy {
    private Money discountAmount;
    private List<DiscountCondition> conditions;

    public AmountDiscountPolicy(Money discountAmount, 
                                DiscountCondition... conditions) {
        this.discountAmount = discountAmount;
        this.conditions = Arrays.asList(conditions);
    }

    @Override
    public List<DiscountCondition> getConditions() {
        return conditions;
    }

    @Override
    public Money getDiscountAmount(Screening screening) {
        return discountAmount;
    }
}
```

#### ⚠️ 디폴트 메서드의 한계

**문제 1: 캡슐화 약화**

```java
public interface DiscountPolicy {
    default Money calculateDiscountAmount(Screening screening) {
        // getConditions()를 사용하려면
        // 이 메서드가 public이어야 함!
        for (DiscountCondition each : getConditions()) {
            if (each.isSatisfiedBy(screening)) {
                return getDiscountAmount(screening);
            }
        }
        return Money.ZERO;
    }

    // ❌ 외부에 노출될 필요 없는데 public이어야 함
    List<DiscountCondition> getConditions();
    Money getDiscountAmount(Screening screening);
}
```

**추상 클래스 방식:**

```java
public abstract class DefaultDiscountPolicy {
    // ✅ private 가능
    private List<DiscountCondition> conditions;

    public Money calculateDiscountAmount(Screening screening) {
        // private 필드 직접 접근
        for (DiscountCondition each : conditions) {
            if (each.isSatisfiedBy(screening)) {
                return getDiscountAmount(screening);
            }
        }
        return Money.ZERO;
    }

    // ✅ protected 가능
    protected abstract Money getDiscountAmount(Screening screening);
}
```

**문제 2: 디폴트 메서드의 원래 목적**

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  디폴트 메서드의 목적:                                   │
│  추상 클래스를 대체하는 것이 아니라                         │
│  하위 호환성 문제를 해결하는 것!                           │
│                                                     │
└─────────────────────────────────────────────────────┘
```

**실제 사용 사례:**

```java
// Java 8에서 기존 Collection 인터페이스에
// stream() 메서드 추가

public interface Collection<E> {
    // 기존 메서드들...
    
    // 새로 추가된 디폴트 메서드
    default Stream<E> stream() {
        return StreamSupport.stream(spliterator(), false);
    }
}

// 기존 구현 클래스들은 수정 없이도
// stream() 메서드 사용 가능!
List<String> list = Arrays.asList("a", "b", "c");
list.stream().forEach(System.out::println);
```

### 3.3 선택 가이드

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  단순한 경우:                                          │
│  → 인터페이스 또는 추상 클래스 중 하나만 사용                 │
│                                                     │
│  복잡한 경우:                                          │
│  → 인터페이스 + 골격 구현 추상 클래스                       │
│                                                     │
│  단일 상속 계층:                                       │
│  → 추상 클래스 고려                                     │
│                                                     │
│  다중 타입 필요:                                       │
│  → 인터페이스 고려                                      │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

## 4. 덕 타이핑

### 4.1 덕 테스트

#### 🦆 James Whitcomb Riley의 명언

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  "어떤 새가 오리처럼 걷고,                                │
│   오리처럼 헤엄치며,                                     │
│   오리처럼 꽥꽥 소리를 낸다면                              │
│   나는 이 새를 오리라고 부를 것이다"                        │
│                                                     │
│  - James Whitcomb Riley                             │
│                                                     │
└─────────────────────────────────────────────────────┘
```

**핵심:**

```
덕 타이핑 (Duck Typing):

어떤 대상의 행동이 오리와 같다면
그것을 오리라는 타입으로 취급해도 무방

→ 행동이 타입을 결정!
```

### 4.2 덕 타이핑의 특징

#### 🎯 행동 중심의 타입

```
전통적 타입 시스템:
- 명시적 타입 선언 필요
- 컴파일 타임에 타입 검사
- Employee 인터페이스 구현 필요

덕 타이핑:
- 타입 선언 불필요
- 런타임에 행동 검사
- calculatePay 메서드만 있으면 OK
```

#### 💻 언어별 예시

**Ruby (동적 타입 언어):**

```ruby
# 타입 선언 없음!
class SalariedEmployee
  def initialize(name, base_pay)
    @name = name
    @base_pay = base_pay
  end
  
  def calculate_pay(tax_rate)
    @base_pay - (@base_pay * tax_rate)
  end
end

class HourlyEmployee
  def initialize(name, base_pay, time_card)
    @name = name
    @base_pay = base_pay
    @time_card = time_card
  end
  
  def calculate_pay(tax_rate)
    (@base_pay * @time_card) - (@base_pay * @time_card) * tax_rate
  end
end

# 덕 타이핑!
def calculate(employee, tax_rate)
  # employee가 calculate_pay 메서드만 있으면 OK
  employee.calculate_pay(tax_rate)
end

# 사용
calculate(SalariedEmployee.new("Alice", 3000000), 0.1)
calculate(HourlyEmployee.new("Bob", 15000, 160), 0.1)
```

**C# (dynamic 키워드):**

```csharp
public class SalariedEmployee
{
    private string name;
    private decimal basePay;

    public SalariedEmployee(string name, decimal basePay)
    {
        this.name = name;
        this.basePay = basePay;
    }

    public decimal CalculatePay(decimal taxRate)
    {
        return basePay - basePay * taxRate;
    }
}

public class HourlyEmployee
{
    private string name;
    private decimal basePay;
    private int timeCard;

    public HourlyEmployee(string name, decimal basePay, int timeCard)
    {
        this.name = name;
        this.basePay = basePay;
        this.timeCard = timeCard;
    }

    public decimal CalculatePay(decimal taxRate)
    {
        return (basePay * timeCard) - (basePay * timeCard) * taxRate;
    }
}

public class TaxOffice
{
    // dynamic 사용 - 런타임 타입 체크
    public decimal Calculate(dynamic employee, decimal taxRate)
    {
        return employee.CalculatePay(taxRate);
    }
}
```

**C++ (템플릿):**

```cpp
// 타입 선언 없음
class SalariedEmployee {
private:
    string name;
    long base_pay;

public:
    SalariedEmployee(string name, long base_pay)
        : name(name), base_pay(base_pay) {}

    long calculate_pay(double tax_rate) {
        return base_pay - (base_pay * tax_rate);
    }
};

class HourlyEmployee {
private:
    string name;
    long base_pay;
    int time_card;

public:
    HourlyEmployee(string name, long base_pay, int time_card)
        : name(name), base_pay(base_pay), time_card(time_card) {}

    long calculate_pay(double tax_rate) {
        return (base_pay * time_card) - (base_pay * time_card) * tax_rate;
    }
};

// 템플릿 - 컴파일 타임 덕 타이핑
template <typename T>
long calculate(T employee, double tax_rate) {
    return employee.calculate_pay(tax_rate);
}

// 사용
calculate(SalariedEmployee("Alice", 3000000), 0.1);
calculate(HourlyEmployee("Bob", 15000, 160), 0.1);
```

### 4.3 덕 타이핑의 장단점

#### ✅ 장점

```
1. 유연성
   - 명시적 타입 선언 불필요
   - 새로운 타입 추가 용이

2. 간결성
   - 인터페이스 정의 불필요
   - 보일러플레이트 코드 감소

3. 빠른 프로토타이핑
   - 타입 계층 설계 없이 개발 가능
   - 실험적 코드 작성 용이
```

**예시:**

```ruby
# 새로운 타입 추가 - 인터페이스 수정 불필요!
class ContractEmployee
  def initialize(name, hourly_rate, hours)
    @name = name
    @hourly_rate = hourly_rate
    @hours = hours
  end
  
  def calculate_pay(tax_rate)
    (@hourly_rate * @hours) - (@hourly_rate * @hours) * tax_rate
  end
end

# 즉시 사용 가능
calculate(ContractEmployee.new("Charlie", 50000, 100), 0.1)
```

#### ❌ 단점

```
1. 타입 안전성 부족
   - 컴파일 타임 오류 검출 불가
   - 런타임 오류 발생 가능

2. IDE 지원 약함
   - 자동 완성 제한적
   - 리팩토링 도구 사용 어려움

3. 가독성 저하
   - 명시적 타입 정보 부재
   - 코드 이해 어려움

4. 디버깅 어려움
   - 오류 발생 시점이 늦음
   - 오류 원인 파악 어려움
```

**문제 예시:**

```ruby
# 오타가 있어도 컴파일 타임에 발견 못함
def calculate(employee, tax_rate)
  # calculate_pay가 아니라 calcuate_pay (오타!)
  employee.calcuate_pay(tax_rate)
end

# 런타임에만 오류 발생!
calculate(SalariedEmployee.new("Alice", 3000000), 0.1)
# NoMethodError: undefined method `calcuate_pay'
```

### 4.4 C++ 템플릿 - 컴파일 타임 덕 타이핑

#### 🎯 최선의 양립

```
C++ 템플릿:
- 덕 타이핑의 유연성
- 정적 타입의 안전성

→ 컴파일 타임에 타입 체크!
```

**작동 방식:**

```cpp
template <typename T>
long calculate(T employee, double tax_rate) {
    return employee.calculate_pay(tax_rate);
}

// 컴파일러가 각 타입별로 함수 생성
// calculate<SalariedEmployee>(...)
// calculate<HourlyEmployee>(...)

// 컴파일 타임에 calculate_pay 메서드 존재 여부 확인!
```

**장점:**

```
✅ 유연성 (덕 타이핑)
✅ 타입 안전성 (정적 타입)
✅ 성능 (인라인 가능)
```

**단점:**

```
❌ 컴파일 시간 증가
❌ 바이너리 크기 증가 (각 타입별로 함수 생성)
❌ 복잡한 오류 메시지
```

### 4.5 Java와 덕 타이핑

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  Java는 덕 타이핑을 지원하지 않는다                        │
│                                                     │
│  이유:                                               │
│  - 정적 타입 언어                                      │
│  - 명시적 타입 선언 필수                                 │
│  - 컴파일 타임 타입 체크                                 │
│                                                     │
│  대안:                                               │
│  - 인터페이스 사용                                      │
│  - 리플렉션 (성능 문제)                                  │
│  - dynamic proxy (복잡함)                             │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

## 5. 믹스인과 타입 계층

### 5.1 믹스인이란?

#### 🎯 정의

```
믹스인 (Mixin):

객체를 생성할 때
코드 일부를 섞어 넣을 수 있도록 만들어진
일종의 추상 서브클래스

목적:
다양한 객체 구현 안에서
동일한 행동을 중복 코드 없이 재사용
```

**특징:**

```
믹스인 ≠ 상속

상속:
- is-a 관계
- 단일 계층
- 타입 계층 정의

믹스인:
- has-a 행동
- 다중 조합 가능
- 행동 재사용
```

### 5.2 Scala의 트레이트

#### 💻 Ordered 트레이트 예시

```scala
// Money 클래스
case class Money(amount: Long) extends Ordered[Money] {
    // + 연산자
    def +(that: Money): Money = Money(this.amount + that.amount)
    
    // - 연산자
    def -(that: Money): Money = Money(this.amount - that.amount)
    
    // compare 메서드만 구현!
    def compare(that: Money): Int = (this.amount - that.amount).toInt
}

// Ordered 트레이트가 제공하는 것들:
// < (less than)
// > (greater than)
// <= (less than or equal)
// >= (greater than or equal)

// 사용
println(Money(10) < Money(20))  // true
println(Money(30) > Money(20))  // true
```

**Ordered 트레이트 내부 (간소화):**

```scala
trait Ordered[A] {
    // 추상 메서드 - 구현 필요
    def compare(that: A): Int
    
    // 믹스인이 제공하는 메서드들
    def <(that: A): Boolean = (this compare that) < 0
    def >(that: A): Boolean = (this compare that) > 0
    def <=(that: A): Boolean = (this compare that) <= 0
    def >=(that: A): Boolean = (this compare that) >= 0
}
```

#### 💎 믹스인의 효과

```
간결한 인터페이스
    ↓
  믹스인
    ↓
풍부한 인터페이스

Money 클래스:
- 구현한 것: compare 메서드 (1개)
- 얻은 것: <, >, <=, >= 메서드 (4개)

→ 최소한의 노력으로 풍부한 기능!
```

### 5.3 Java의 디폴트 메서드로 믹스인 구현

#### 💻 예시

```java
// Comparable을 확장한 Ordered 인터페이스
public interface Ordered<T> extends Comparable<T> {
    // 추상 메서드
    int compareTo(T other);
    
    // 디폴트 메서드 (믹스인)
    default boolean lessThan(T other) {
        return compareTo(other) < 0;
    }
    
    default boolean greaterThan(T other) {
        return compareTo(other) > 0;
    }
    
    default boolean lessThanOrEqual(T other) {
        return compareTo(other) <= 0;
    }
    
    default boolean greaterThanOrEqual(T other) {
        return compareTo(other) >= 0;
    }
}

// Money 클래스
public class Money implements Ordered<Money> {
    private long amount;
    
    public Money(long amount) {
        this.amount = amount;
    }
    
    // compareTo만 구현
    @Override
    public int compareTo(Money other) {
        return Long.compare(this.amount, other.amount);
    }
    
    // lessThan, greaterThan 등은 자동으로 사용 가능!
}

// 사용
Money m1 = new Money(10);
Money m2 = new Money(20);

System.out.println(m1.lessThan(m2));           // true
System.out.println(m1.greaterThan(m2));        // false
System.out.println(m1.lessThanOrEqual(m2));    // true
```

#### ⚠️ Java 디폴트 메서드의 한계

**문제: 캡슐화 약화**

```java
public interface DiscountPolicy {
    // 믹스인으로 제공하고 싶은 메서드
    default Money calculateDiscountAmount(Screening screening) {
        // 문제: getConditions()를 사용하려면 public이어야 함!
        for (DiscountCondition each : getConditions()) {
            if (each.isSatisfiedBy(screening)) {
                return getDiscountAmount(screening);
            }
        }
        return Money.ZERO;
    }
    
    // ❌ 외부에 노출될 필요 없는데 public!
    List<DiscountCondition> getConditions();
    
    // ❌ 외부에 노출될 필요 없는데 public!
    Money getDiscountAmount(Screening screening);
}
```

**이상적인 형태 (불가능):**

```java
public interface DiscountPolicy {
    default Money calculateDiscountAmount(Screening screening) {
        // private 메서드 사용하고 싶지만...
        // 인터페이스는 private 메서드 불가 (Java 8)
        for (DiscountCondition each : getConditions()) {
            // ...
        }
        return Money.ZERO;
    }
    
    // 이렇게 하고 싶지만 불가능
    private List<DiscountCondition> getConditions();
    private Money getDiscountAmount(Screening screening);
}
```

**참고: Java 9부터는 private 메서드 가능**

```java
// Java 9+
public interface DiscountPolicy {
    default Money calculateDiscountAmount(Screening screening) {
        // private 메서드 호출 가능!
        return calculateInternal(screening);
    }
    
    // Java 9부터 가능
    private Money calculateInternal(Screening screening) {
        // 구현...
        return Money.ZERO;
    }
}
```

### 5.4 디폴트 메서드의 진짜 목적

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  디폴트 메서드의 목적:                                   │
│                                                     │
│  추상 클래스를 대체하려는 것이 아니라 ✗                      │
│  하위 호환성 문제를 해결하려는 것이다 ✓                      │
│                                                     │
└─────────────────────────────────────────────────────┘
```

**배경:**

```
Java 8 이전:

interface Collection<E> {
    // 기존 메서드들
    boolean add(E e);
    boolean remove(Object o);
    // ...
}

// 수많은 구현 클래스들
class ArrayList<E> implements Collection<E> { ... }
class LinkedList<E> implements Collection<E> { ... }
class HashSet<E> implements Collection<E> { ... }
// ...
```

**문제:**

```
Java 8에서 stream() 추가하고 싶다면?

interface Collection<E> {
    // 새 메서드 추가
    Stream<E> stream();  // ← 모든 구현 클래스가 컴파일 에러!
}

→ 하위 호환성 깨짐!
```

**해결책: 디폴트 메서드**

```java
interface Collection<E> {
    // 기존 메서드들...
    
    // 디폴트 메서드로 추가
    default Stream<E> stream() {
        return StreamSupport.stream(spliterator(), false);
    }
}

// 기존 구현 클래스들은 수정 없이도 동작!
List<String> list = new ArrayList<>();
list.stream().forEach(System.out::println);
```

### 5.5 믹스인 사용 시 주의사항

```
✅ 믹스인 사용이 좋은 경우:
- 간결한 인터페이스 → 풍부한 인터페이스
- 여러 클래스에서 동일한 행동 재사용
- 행동 조합이 필요한 경우

❌ 믹스인을 피해야 하는 경우:
- 캡슐화가 중요한 경우
- 복잡한 내부 상태 관리가 필요한 경우
- 추상 클래스로도 충분한 경우
```

---

## 6. 핵심 정리

### 🎯 타입 계층 구현 방법 비교

| 방법 | 장점 | 단점 | 사용 시나리오 |
|------|------|------|--------------|
| **클래스 상속** | 간단함 | 강한 결합, 다중 상속 불가 | 단순한 단일 계층 |
| **인터페이스** | 낮은 결합도, 다중 타입 | 코드 중복 | 다중 타입 필요 |
| **추상 클래스** | 구현 공유, DIP | 단일 상속 제약 | 단일 계층 + 구현 공유 |
| **인터페이스 + 추상** | 최선의 조합 | 복잡성 증가 | 복잡한 타입 계층 |
| **덕 타이핑** | 높은 유연성 | 타입 안전성 부족 | 동적 언어, 빠른 개발 |
| **믹스인** | 행동 재사용 | 캡슐화 약화 | 간결→풍부 인터페이스 |

### 📊 결정 트리

```
타입 계층 구현 필요
    ↓
단일 상속 계층인가?
    ↓ YES                    ↓ NO
추상 클래스 사용         인터페이스 사용
    ↓                        ↓
구현 공유 필요?         구현 공유 필요?
    ↓ YES                    ↓ YES
그대로 사용              골격 구현 추상 클래스 추가
                            ↓
                        인터페이스 + 추상 클래스
```

### 💡 핵심 원칙

```
1. 클래스가 아니라 타입에 집중
   → 행동이 중요

2. 타입과 구현의 분리
   → 인터페이스 활용

3. 구현 공유 시 추상화에 의존
   → 추상 클래스, DIP

4. 리스코프 치환 원칙 준수
   → 타입 계층의 필수 조건

5. 트레이드오프 이해
   → 상황에 맞는 선택
```

### 🎓 실전 가이드

#### Do's ✅

```
1. 인터페이스로 타입 정의
   - 퍼블릭 인터페이스만 명시
   - 구현은 클래스에 위임

2. 추상 클래스로 공통 구현 제공
   - 코드 중복 제거
   - 의존성 역전 원칙 준수

3. 골격 구현 패턴 활용
   - 유연성과 재사용성 모두 확보
   - 복잡한 타입 계층에 적합

4. 믹스인으로 행동 재사용
   - 간결한 → 풍부한 인터페이스
   - 여러 클래스에서 동일 행동 공유

5. LSP 항상 준수
   - 서브타입은 슈퍼타입 대체 가능해야
   - 계약 규칙 준수
```

#### Don'ts ❌

```
1. 구체 클래스 상속 지양
   - 강한 결합도
   - 캡슐화 위반

2. 타입과 구현 혼동 금지
   - 클래스 ≠ 타입
   - 행동에 집중

3. 디폴트 메서드 남용 금지
   - 추상 클래스 대체 목적 아님
   - 캡슐화 약화 주의

4. 덕 타이핑 무분별 사용 금지
   - 타입 안전성 고려
   - 적절한 상황에만 사용

5. LSP 위반 금지
   - 타입 계층의 근본
   - 치환 가능성 필수
```

### 🌟 마무리 메시지

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  타입 계층 구현은 도구일 뿐이다                            │
│                                                     │
│  중요한 것은:                                          │
│  - 행동의 일관성                                        │
│  - 치환 가능성                                         │
│  - 유연한 설계                                         │
│                                                     │
│  올바른 도구를 선택하라:                                  │
│  - 상황에 맞는 방법                                     │
│  - 트레이드오프 이해                                     │
│  - LSP 항상 준수                                       │
│                                                     │
│  "클래스가 아니라 타입에 집중하라"                          │
│                                                     │
└─────────────────────────────────────────────────────┘
```

### 🔑 핵심 교훈

```
1. 타입 = 행동의 집합
   → 퍼블릭 인터페이스가 타입 정의

2. 클래스는 타입 구현 방법 중 하나
   → 타입과 구현 분리 중요

3. 인터페이스 + 추상 클래스 = 최선
   → 유연성과 재사용성 모두 확보

4. 덕 타이핑은 양날의 검
   → 유연성 vs 안전성 트레이드오프

5. 믹스인은 행동 재사용 도구
   → 간결함을 풍부하게

6. LSP는 타입 계층의 기본
   → 항상 준수해야 함

7. 설계는 트레이드오프
   → 상황에 맞는 선택이 중요

8. 클래스가 아니라 타입에 집중
   → 이것이 객체지향의 본질
```

---

## 🔗 연결고리

### 이전 장과의 연결
- **Appendix A**: 계약 → 타입 계층에서 LSP 준수
- **Chapter 13**: 서브타이핑 → 다양한 구현 방법
- **전체 책**: 역할, 책임, 협력 → 타입으로 구체화

### 실무로의 확장
- **다음 단계**:
  - 언어별 특성 이해 (Java, Scala, Kotlin 등)
  - 디자인 패턴 학습 (GoF 패턴)
  - 타입 시스템 깊이 학습
  - 함수형 프로그래밍의 타입 시스템

---

<div align="center">

**[⬆ 목차로 돌아가기](../README.md)**

</div>

<div align="center">

**[⬅️ 이전: Appendix A](../appendixA/README.md)** | **[다음: Appendix C ➡️](#)**

</div>
