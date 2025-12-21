# Chapter 15. 디자인 패턴과 프레임워크

> *"디자인 패턴은 협력 템플릿을, 프레임워크는 코드 템플릿을 제공한다."*

## 📌 핵심 개념

이 장에서는 **디자인 패턴**과 **프레임워크**를 다룹니다. 반복적으로 발생하는 문제를 해결하기 위한 검증된 설계 지식을 어떻게 재사용할 수 있는지, 그리고 설계와 코드를 함께 재사용하는 방법을 학습합니다.

### 🎯 학습 목표

- 디자인 패턴의 본질과 가치 이해하기
- 패턴 분류 체계 파악하기 (아키텍처, 디자인, 이디엄, 분석)
- 주요 디자인 패턴의 목적과 구조 학습하기
- 프레임워크의 개념과 특징 이해하기
- 제어 역전 원리의 의미 파악하기
- 설계 재사용과 코드 재사용의 조화 이해하기

---

## 📖 목차

1. [디자인 패턴과 설계 재사용](#1-디자인-패턴과-설계-재사용)
2. [프레임워크와 코드 재사용](#2-프레임워크와-코드-재사용)
3. [핵심 정리](#3-핵심-정리)

---

## 1. 디자인 패턴과 설계 재사용

### 1.1 소프트웨어 패턴

#### 🎯 패턴이란?

**Martin Fowler의 정의:**
```
패턴 정의는
하나의 실무 컨텍스트(practical context)에서
유용하게 사용해왔고

다른 실무 컨텍스트에서도
유용할 것이라고 예상되는
아이디어(idea)다
```

**핵심 특징:**

```
1. 반복적으로 발생하는 문제와 해법의 쌍

2. 알려진 문제와 해법을 문서화
   → 지식 전달 가능

3. 추상적 원칙과 실제 코드 사이의 간극을 메움
   → 실질적인 코드 작성 돕기

4. 실무 경험에서 비롯됨
   → 검증된 해결책
```

#### 📏 3의 규칙 (Rule of Three)

```
패턴으로 인정받기 위한 조건:

최소 3가지의 서로 다른 시스템에
특별한 문제 없이 적용할 수 있고
유용한 경우에만 패턴으로 간주

→ 경험을 통한 검증 필요
```

#### 💎 패턴의 가치

**1. 경험의 효과적 전달**

```
패턴 = 경험의 산물

실무 경험이 적은 초보자도:
→ 패턴을 익히고 반복 적용
→ 유연하고 품질 높은 소프트웨어 개발 방법 습득
```

**2. 공통 어휘 제공**

```
❌ 패턴을 모르는 경우:
"인터페이스를 하나 추가하고 
이 인터페이스를 구체화하는 클래스를 만든 후 
객체의 생성자나 setter 메서드에 할당해서 
런타임 시에 알고리즘을 바꿀 수 있게 하자"

✅ 패턴을 아는 경우:
"STRATEGY 패턴을 적용하자"

→ 높은 수준의 의사소통 가능!
```

**3. 추상 원칙의 구체화**

```
추상적인 설계 원칙
    ↓
패턴 (구체적 템플릿)
    ↓
실제 코드

→ 간극을 메워주는 다리 역할
```

### 1.2 패턴 분류

#### 📊 4가지 패턴 분류

```
┌─────────────────────────────────────────┐
│      아키텍처 패턴                         │
│   (Architecture Pattern)                │
│   - 소프트웨어 전체 구조                     │
│   - 서브시스템 정의                         │
│   - MVC, Layered Architecture           │
└─────────────┬───────────────────────────┘
              │
┌─────────────▼───────────────────────────┐
│      디자인 패턴                           │
│   (Design Pattern)                      │
│   - 협력하는 컴포넌트 구조                    │
│   - STRATEGY, TEMPLATE METHOD           │
│   - DECORATOR, COMPOSITE                │
└─────────────┬───────────────────────────┘
              │
┌─────────────▼───────────────────────────┐
│      이디엄                               │
│   (Idiom)                               │
│   - 특정 언어에 국한                        │
│   - COUNT POINTER (C++)                 │
└─────────────────────────────────────────┘

         분석 패턴 (Analysis Pattern)
      ┌────────────────────────────┐
      │  도메인 개념적 문제 해결         │
      │  업무 모델링 구조              │
      └────────────────────────────┘
```

#### 🏛️ 아키텍처 패턴 (Architecture Pattern)

**정의:**
```
소프트웨어 전체 구조를 결정하기 위해 사용

제공하는 것:
- 미리 정의된 서브시스템
- 서브시스템의 책임 정의
- 서브시스템 간 관계 조직화 규칙
```

**예시:**
```
MVC (Model-View-Controller)
Layered Architecture (계층형 아키텍처)
Microservices Architecture
Event-Driven Architecture
```

**특징:**
```
범위: 시스템 전체
적용 단계: 아키텍처 설계
영향: 전체 시스템 구조
```

#### 🎨 디자인 패턴 (Design Pattern)

**정의:**
```
특정 상황 내에서
일반적인 설계 문제를 해결하며

협력하는 컴포넌트들 사이에서
반복적으로 발생하는 구조를 서술
```

**특징:**
```
범위: 컴포넌트 간 협력
적용 단계: 상세 설계
언어 독립적: 프로그래밍 언어나 패러다임에 독립적
```

**예시:**
```
생성 패턴:
- FACTORY METHOD
- ABSTRACT FACTORY
- SINGLETON

구조 패턴:
- ADAPTER
- DECORATOR
- COMPOSITE

행위 패턴:
- STRATEGY
- TEMPLATE METHOD
- OBSERVER
```

#### 💻 이디엄 (Idiom)

**정의:**
```
특정 프로그래밍 언어에만 국한된 하위 레벨 패턴

주어진 언어의 기능을 사용해
컴포넌트 간 특정 측면을 구현하는 방법 서술
```

**예시: COUNT POINTER (C++)**

```cpp
// C++의 이디엄
class CountPointer {
    int* ptr;
    int* refCount;
    
    CountPointer(int* p) : ptr(p), refCount(new int(1)) {}
    
    ~CountPointer() {
        if (--(*refCount) == 0) {
            delete ptr;
            delete refCount;
        }
    }
};
```

**특징:**
```
언어 종속적:
- C++ COUNT POINTER 이디엄
- Java에서는 불필요 (가비지 컬렉터)
- Python의 컨텍스트 매니저 (with 문)
- Go의 defer 패턴
```

#### 📈 분석 패턴 (Analysis Pattern)

**정의:**
```
도메인 내의 개념적 문제를 해결하는 데 초점

업무 모델링 시 발견되는
공통적인 구조를 표현하는 개념들의 집합
```

**특징:**
```
기술적 문제 X
도메인 개념적 문제 O

예시:
- Account Transaction 패턴
- Party 패턴 (사람/조직 모델링)
- Inventory 패턴
```

**비교:**

| 패턴 종류 | 범위 | 목적 | 예시 |
|----------|------|------|------|
| **아키텍처** | 시스템 전체 | 전체 구조 결정 | MVC, Layered |
| **디자인** | 컴포넌트 협력 | 설계 문제 해결 | STRATEGY, DECORATOR |
| **이디엄** | 언어 수준 | 언어 특화 구현 | COUNT POINTER |
| **분석** | 도메인 | 개념적 문제 해결 | Account Transaction |

### 1.3 패턴과 책임-주도 설계

#### 🎯 패턴의 구성 요소

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  패턴의 구성 요소는 클래스가 아니라 '역할'이다!                │
│                                                     │
└─────────────────────────────────────────────────────┘
```

**의미:**

```
역할은 추상적 개념
→ 다양한 방법으로 구현 가능

1. 하나의 객체가 여러 역할 수행 가능
2. 다수의 클래스가 동일한 역할 구현 가능
3. 구현 방법에 제한 없음
```

**예시:**

```
STRATEGY 패턴:

역할:
- Context (컨텍스트)
- Strategy (전략)
- ConcreteStrategy (구체 전략)

구현 선택:
- Strategy를 인터페이스로? 추상 클래스로?
- ConcreteStrategy 개수는?
- Context가 Strategy를 어떻게 사용?

→ 역할과 책임만 유사하면
  다양한 구현 가능!
```

#### 💡 핵심 통찰

```
디자인 패턴을 따른다 =
특정 구현 방식 강제 X

역할, 책임, 협력 관점에서
유사성을 공유한다 O

→ 창의적 적용 가능!
```

**역할 기반 패턴 적용:**

```
1. 패턴이 제시하는 역할 이해
   ↓
2. 현재 문제의 역할 식별
   ↓
3. 역할 간 책임 할당
   ↓
4. 협력 설계
   ↓
5. 구체적 구현 (다양한 방법 가능)
```

### 1.4 캡슐화와 디자인 패턴

#### 🎨 주요 디자인 패턴과 변경 캡슐화

**핵심:**
```
모든 디자인 패턴의 목적:
→ 변경을 캡슐화하는 것

패턴마다 캡슐화하는 변경이 다름
```

#### 1️⃣ STRATEGY 패턴

**목적: 알고리즘의 변경을 캡슐화**

**구조:**

```
┌──────────────┐
│   Context    │
│              │
│ + operation()│
└──────┬───────┘
       │ uses
       │
       ▼
┌───────────────┐
│  <<interface>>│
│   Strategy    │
│               │
│ + algorithm() │
└──────┬────────┘
       △
       │
   ┌───┴───┬───────────┐
   │       │           │
┌──▼───┐ ┌─▼────┐ ┌───▼──┐
│Strat │ │Strat │ │Strat │
│egy A │ │egy B │ │egy C │
└──────┘ └──────┘ └──────┘
```

**영화 예매 시스템 예시:**

```java
// Context
public class Movie {
    private DiscountPolicy discountPolicy;  // Strategy
    
    public Money calculateMovieFee(Screening screening) {
        // 알고리즘 실행 위임
        return fee.minus(discountPolicy.calculateDiscountAmount(screening));
    }
}

// Strategy
public abstract class DiscountPolicy {
    abstract protected Money getDiscountAmount(Screening screening);
}

// ConcreteStrategy
public class AmountDiscountPolicy extends DiscountPolicy {
    @Override
    protected Money getDiscountAmount(Screening screening) {
        return discountAmount;
    }
}

public class PercentDiscountPolicy extends DiscountPolicy {
    @Override
    protected Money getDiscountAmount(Screening screening) {
        return screening.getMovieFee().times(percent);
    }
}
```

**특징:**

```
✅ 객체 합성 사용
✅ 런타임에 알고리즘 변경 가능
✅ 낮은 결합도
✅ 높은 유연성

변경 캡슐화:
- 알고리즘의 종류
- 알고리즘의 구현 방법
```

#### 2️⃣ TEMPLATE METHOD 패턴

**목적: 알고리즘의 변경을 캡슐화 (상속 사용)**

**구조:**

```
┌──────────────────────┐
│   AbstractClass      │
│                      │
│ + templateMethod()   │
│ # primitiveOp1()     │
│ # primitiveOp2()     │
└──────────┬───────────┘
           △
           │
     ┌─────┴─────────│───┐
     │                   │
┌────▼─────────┐     ┌───▼──────────┐
│Concrete      │     │Concrete      │
│Class A       │     │Class B       │
│              │     │              │
│primitiveOp1()│     │primitiveOp2()│
│primitiveOp2()│     │primitiveOp1()│
└──────────────┘     └──────────────┘
```

**영화 예매 시스템 TEMPLATE METHOD 버전:**

```java
// AbstractClass
public abstract class Movie {
    private Money fee;
    
    // Template Method
    public Money calculateFee(Screening screening) {
        return fee.minus(calculateDiscountAmount(screening));
    }
    
    // Primitive Operation (추상 메서드)
    abstract protected Money calculateDiscountAmount(Screening screening);
}

// ConcreteClass
public class AmountDiscountMovie extends Movie {
    private Money discountAmount;
    
    @Override
    protected Money calculateDiscountAmount(Screening screening) {
        return discountAmount;
    }
}

public class PercentDiscountMovie extends Movie {
    private double percent;
    
    @Override
    protected Money calculateDiscountAmount(Screening screening) {
        return getMovieFee().times(percent);
    }
}
```

**특징:**

```
✅ 상속 사용
✅ 컴파일 타임에 알고리즘 결정
✅ STRATEGY보다 단순

❌ 런타임 변경 불가
❌ STRATEGY보다 높은 결합도

변경 캡슐화:
- 알고리즘의 세부 단계
- 단계의 구체적 구현
```

#### 🆚 STRATEGY vs TEMPLATE METHOD

| 측면 | STRATEGY | TEMPLATE METHOD |
|------|----------|-----------------|
| **기법** | 객체 합성 | 상속 |
| **변경 시점** | 런타임 | 컴파일 타임 |
| **결합도** | 낮음 | 높음 |
| **복잡도** | 높음 | 낮음 |
| **유연성** | 높음 | 낮음 |

**선택 기준:**

```
STRATEGY 선택:
- 런타임 알고리즘 변경 필요
- 알고리즘을 독립적으로 재사용
- 낮은 결합도 중요

TEMPLATE METHOD 선택:
- 알고리즘이 고정적
- 단순한 구현 선호
- 코드 중복 제거가 주 목적
```

#### 3️⃣ DECORATOR 패턴

**목적: 선택적 행동의 순서와 개수 변경을 캡슐화**

**구조:**

```
┌──────────────┐
│ <<interface>>│
│  Component   │
│              │
│ + operation()│
└──────┬───────┘
       △
       │
   ┌───┴────────────────┐
   │                    │
┌──▼────────┐    ┌──────▼───────┐
│Concrete   │    │  Decorator   │
│Component  │    │              │
└───────────┘    │ - component  │
                 │ + operation()│
                 └──────┬───────┘
                        △
                        │
                  ┌─────┴────┐
                  │          │
           ┌──────▼───┐ ┌────▼──────┐
           │Concrete  │ │Concrete   │
           │DecoratorA│ │DecoratorB │
           └──────────┘ └───────────┘
```

**핸드폰 과금 시스템 예시:**

```java
// Component
public interface RatePolicy {
    Money calculateFee(Phone phone);
}

// ConcreteComponent
public abstract class BasicRatePolicy implements RatePolicy {
    @Override
    public Money calculateFee(Phone phone) {
        Money result = Money.ZERO;
        for (Call call : phone.getCalls()) {
            result = result.plus(calculateCallFee(call));
        }
        return result;
    }
    
    protected abstract Money calculateCallFee(Call call);
}

// Decorator
public abstract class AdditionalRatePolicy implements RatePolicy {
    private RatePolicy next;  // 감싸질 객체
    
    public AdditionalRatePolicy(RatePolicy next) {
        this.next = next;
    }
    
    @Override
    public Money calculateFee(Phone phone) {
        Money fee = next.calculateFee(phone);
        return afterCalculated(fee);
    }
    
    abstract protected Money afterCalculated(Money fee);
}

// ConcreteDecorator
public class TaxablePolicy extends AdditionalRatePolicy {
    private double taxRatio;
    
    public TaxablePolicy(double taxRatio, RatePolicy next) {
        super(next);
        this.taxRatio = taxRatio;
    }
    
    @Override
    protected Money afterCalculated(Money fee) {
        return fee.plus(fee.times(taxRatio));
    }
}

public class RateDiscountablePolicy extends AdditionalRatePolicy {
    private Money discountAmount;
    
    public RateDiscountablePolicy(Money discountAmount, RatePolicy next) {
        super(next);
        this.discountAmount = discountAmount;
    }
    
    @Override
    protected Money afterCalculated(Money fee) {
        return fee.minus(discountAmount);
    }
}
```

**사용 예:**

```java
// 일반 요금제
Phone phone = new Phone(
    new RegularPolicy(...)
);

// 일반 요금제 + 세금
Phone phone = new Phone(
    new TaxablePolicy(0.05,
        new RegularPolicy(...)
    )
);

// 일반 요금제 + 세금 + 요금 할인
Phone phone = new Phone(
    new RateDiscountablePolicy(Money.wons(1000),
        new TaxablePolicy(0.05,
            new RegularPolicy(...)
        )
    )
);

// 순서 변경 가능!
Phone phone = new Phone(
    new TaxablePolicy(0.05,
        new RateDiscountablePolicy(Money.wons(1000),
            new RegularPolicy(...)
        )
    )
);
```

**특징:**

```
✅ 런타임에 행동 동적 추가
✅ 행동의 순서 변경 가능
✅ 행동의 개수 제한 없음
✅ 단일 책임 원칙 (각 데코레이터는 하나의 책임)

변경 캡슐화:
- 선택적 행동의 종류
- 행동의 조합 순서
- 행동의 개수
```

#### 📊 패턴별 캡슐화 비교

| 패턴 | 캡슐화 대상 | 변경 시점 | 기법 |
|------|------------|----------|------|
| **STRATEGY** | 알고리즘 종류 | 런타임 | 합성 |
| **TEMPLATE METHOD** | 알고리즘 단계 | 컴파일 | 상속 |
| **DECORATOR** | 행동의 순서/개수 | 런타임 | 합성 |

### 1.5 패턴은 출발점이다

#### ⚠️ 패턴 남용의 위험

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  패턴은 출발점이지 목적지가 아니다!                         │
│                                                     │
│  패턴이 목표가 되어서는 안 된다                            │
│                                                     │
└─────────────────────────────────────────────────────┘
```

**초보자의 함정:**

```
패턴의 강력함에 매료됨
    ↓
모든 설계에 패턴 적용 시도
    ↓
명확한 트레이드오프 없이 남용
    ↓
불필요하게 복잡한 설계
```

#### 💡 올바른 패턴 사용

**1. 문제부터 시작**

```
❌ 나쁜 순서:
패턴 선택 → 문제에 적용

✅ 좋은 순서:
문제 분석 → 적합한 패턴 탐색
```

**2. 트레이드오프 고려**

```
패턴 적용 전 질문:

1. 이 패턴이 정말 필요한가?
2. 더 단순한 해결책은 없는가?
3. 복잡도 증가가 정당한가?
4. 팀이 이해할 수 있는가?
```

**3. 목적에 맞게 수정**

```
패턴을 맹목적으로 따르지 말고
현재 요구사항과 기술에 맞게 수정

패턴 = 가이드라인
      ≠ 엄격한 규칙
```

#### 🎯 패턴의 진정한 가치

```
패턴을 알고 있다면:

현재 문제에 딱 들어맞지 않더라도
참조할 수 있는 모범적
역할과 책임의 집합을 알고 있는 것

→ 큰 도움이 된다!
```

**활용 방법:**

```
1. 비슷한 문제 식별
   ↓
2. 패턴 구조 참조
   ↓
3. 현재 상황에 맞게 조정
   ↓
4. 역할과 책임 재배치
   ↓
5. 새로운 협력 패턴 창출
```

#### 📋 패턴 적용 체크리스트

```
✅ 해결하려는 문제가 명확한가?
✅ 패턴이 문제 해결에 적합한가?
✅ 트레이드오프를 이해했는가?
✅ 팀이 패턴을 이해하는가?
✅ 유지보수 비용을 고려했는가?
✅ 더 단순한 대안을 검토했는가?

모두 YES → 패턴 적용 고려
하나라도 NO → 재검토 필요
```

#### 💎 핵심 원칙

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  정당한 이유 없이 사용된 모든 패턴은                        │
│  설계를 복잡하게 만드는 장애물이다                          │
│                                                     │
│  패턴은 비싸다                                          │
│  반드시 그 가치가 있을 때만 사용이 정당화된다                  │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

## 2. 프레임워크와 코드 재사용

### 2.1 코드 재사용 대 설계 재사용

#### 🎯 재사용의 두 측면

**설계 재사용:**
```
장점:
✅ 추상적 수준에서 재사용
✅ 다양한 상황 적용 가능
✅ 검증된 구조

단점:
❌ 매번 유사한 코드 작성
❌ 개발 시간 증가
❌ 일관성 유지 어려움
```

**코드 재사용:**
```
장점:
✅ 빠른 개발
✅ 검증된 구현
✅ 일관성 보장

단점:
❌ 특정 상황에만 적용
❌ 유연성 부족
❌ 커스터마이징 어려움
```

#### 💡 이상적인 재사용

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  가장 이상적인 재사용:                                   │
│  설계 재사용 + 코드 재사용을 적절히 조합                     │
│                                                     │
│  → 프레임워크!                                         │
│                                                     │
└─────────────────────────────────────────────────────┘
```

### 2.2 프레임워크란?

#### 📚 프레임워크의 정의

**구조적 측면:**
```
추상 클래스나 인터페이스를 정의하고
인스턴스 사이의 상호작용을 통해
시스템 전체 혹은 일부를 구현해 놓은
재사용 가능한 설계
```

**목적 측면:**
```
애플리케이션 개발자가
현재의 요구사항에 맞게
커스터마이징할 수 있는
애플리케이션의 골격(skeleton)
```

#### 🏗️ 프레임워크의 구성 요소

**1. 애플리케이션 아키텍처**

```
전체 시스템의 구조와
주요 컴포넌트 정의

예:
- MVC 구조
- 계층형 아키텍처
- 이벤트 기반 아키텍처
```

**2. 기반 코드**

```
문제 해결에 필요한
설계 결정과 구현 코드

예:
- HTTP 요청 처리
- 데이터베이스 연결
- 트랜잭션 관리
```

**3. 확장 가능한 추상 클래스와 인터페이스**

```
부분적으로 구현된 클래스들

개발자가 상속/구현해서
애플리케이션 로직 추가
```

**4. 재사용 가능한 컴포넌트**

```
추가 작업 없이 바로 사용 가능한
다양한 종류의 컴포넌트

예:
- 로깅 컴포넌트
- 캐싱 컴포넌트
- 보안 컴포넌트
```

#### 💎 프레임워크의 핵심 가치

```
프레임워크는 코드를 재사용함으로써
설계 아이디어를 재사용한다

설계 자체의 재사용을 중요시한다
```

**예시: 핸드폰 과금 시스템을 프레임워크로**

```
프레임워크 제공:
- RatePolicy 인터페이스
- BasicRatePolicy 추상 클래스
- AdditionalRatePolicy 추상 클래스
- Phone 클래스
- Call 클래스

개발자 구현:
- 구체적인 BasicRatePolicy (고정요금, 시간대별 등)
- 구체적인 AdditionalRatePolicy (세금, 할인 등)

→ 핵심 협력 구조는 프레임워크가 제공!
```

### 2.3 상위 정책과 하위 정책으로 패키지 분리하기

#### 📦 패키지 분리의 필요성

```
프레임워크 = 여러 애플리케이션에서 재사용

→ 변하는 것과 변하지 않는 것을
  '배포 단위'로 분리 필요!
```

#### 🎯 분리 전략

**변하지 않는 부분 (상위 정책):**

```
billing.framework 패키지

- RatePolicy (인터페이스)
- BasicRatePolicy (추상 클래스)
- AdditionalRatePolicy (추상 클래스)
- Phone
- Call

→ 프레임워크로 배포
→ 안정적
→ 재사용 가능
```

**변하는 부분 (하위 정책):**

```
billing.policies 패키지

- RegularPolicy
- NightlyDiscountPolicy
- TaxablePolicy
- RateDiscountablePolicy

→ 애플리케이션별로 구현
→ 변경 가능
→ 상위 정책에 의존
```

#### 📊 패키지 의존성 방향

```
┌──────────────────────────────┐
│   billing.policies           │
│   (하위 정책 - 애플리케이션)       │
│                              │
│   - RegularPolicy            │
│   - NightlyDiscountPolicy    │
│   - TaxablePolicy            │
└────────────┬─────────────────┘
             │ depends on
             │
             ▼
┌──────────────────────────────┐
│   billing.framework          │
│   (상위 정책 - 프레임워크)        │
│                              │
│   - RatePolicy               │
│   - BasicRatePolicy          │
│   - AdditionalRatePolicy     │
│   - Phone                    │
│   - Call                     │
└──────────────────────────────┘

핵심:
세부 사항(하위)이 추상화(상위)에 의존!
= 의존성 역전 원칙 (DIP)
```

#### 💡 분리의 효과

```
상위 정책 패키지가 충분히 안정적이면:

1. 하위 정책 패키지로부터 완벽히 분리
2. 별도의 배포 단위로 만듦
3. 여러 애플리케이션에서 재사용

→ 프레임워크 탄생!
```

### 2.4 제어 역전 원리

#### 🎯 의존성 역전 원리 (DIP)

**Robert C. Martin의 말:**

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  "좋은 객체지향 설계의 증명이 바로 의존성의 역전이다"           │
│                                                     │
│  "그 의존성이 역전돼 있지 않다면                           │
│   절차적 설계를 갖는 것이다"                              │
│                                                     │
└─────────────────────────────────────────────────────┘
```

**의존성 역전:**

```
전통적 설계 (하향식):
상위 모듈 → 하위 모듈 의존

객체지향 설계 (의존성 역전):
상위 모듈 ← 하위 모듈 의존
     ↑
   추상화
```

#### 🔄 제어 역전 원리 (Inversion of Control)

**정의:**

```
의존성을 역전시키면
제어 흐름의 주체도 역전된다!

제어 역전 원리
= Hollywood 원리
  "Don't call us, we'll call you"
```

**전통적 방식 (라이브러리):**

```
개발자 코드가 주체

main() {
    Library lib = new Library();
    lib.method1();  // 개발자가 호출
    lib.method2();  // 개발자가 호출
}

→ 개발자가 흐름 제어
```

**프레임워크 방식:**

```
프레임워크가 주체

Framework {
    hookMethod1();  // 프레임워크가 호출
    hookMethod2();  // 프레임워크가 호출
}

개발자는 hook만 구현
→ 프레임워크가 흐름 제어
```

#### 🎨 훅 (Hook)

**정의:**

```
프레임워크에서
일반적인 해결책만 제공하고

애플리케이션에 따라 달라질 수 있는
특정한 동작은 비워둔 부분

→ 이렇게 완성되지 않은 채로 남겨진 동작
```

**특징:**

```
1. 프레임워크가 호출 시점 결정
2. 개발자는 구현만 제공
3. 제어는 프레임워크가 가짐
```

**예시: 핸드폰 과금 시스템의 훅**

```java
// 프레임워크가 제공
public abstract class BasicRatePolicy implements RatePolicy {
    @Override
    public Money calculateFee(Phone phone) {
        Money result = Money.ZERO;
        
        for (Call call : phone.getCalls()) {
            // 훅 호출! (프레임워크가 호출 시점 결정)
            result = result.plus(calculateCallFee(call));
        }
        
        return result;
    }
    
    // 훅 (개발자가 구현해야 할 부분)
    protected abstract Money calculateCallFee(Call call);
}

// 개발자가 구현
public class RegularPolicy extends BasicRatePolicy {
    @Override
    protected Money calculateCallFee(Call call) {
        // 구체적인 요금 계산 로직
        return amount.times(
            call.getDuration().getSeconds() / seconds.getSeconds()
        );
    }
}
```

#### 📊 제어 흐름 비교

**라이브러리:**

```
개발자 코드
    │
    │ calls
    ▼
라이브러리 코드

제어: 개발자
```

**프레임워크:**

```
프레임워크 코드
    │
    │ calls
    ▼
개발자 코드 (훅)

제어: 프레임워크
```

#### 🎯 핸드폰 과금 시스템의 제어 흐름

```
Phone.calculateFee()
    ↓
RatePolicy.calculateFee()  ← 프레임워크가 제공
    ↓
BasicRatePolicy.calculateFee()  ← 프레임워크가 제공
    ↓
calculateCallFee()  ← 개발자가 구현 (훅)
    ↓
RegularPolicy.calculateCallFee()  ← 개발자 코드 실행

전체 협력 흐름: 프레임워크가 담당
특정 정책 구현: 애플리케이션 개발자가 담당

→ 제어가 프레임워크로 역전!
```

#### 💡 제어 역전의 의미

```
과거 (라이브러리):
우리가 직접 라이브러리 코드 호출

현재 (프레임워크):
프레임워크가 우리 코드를 호출

→ 제어가 우리에게서 프레임워크로 넘어감!
```

**개발자의 역할 변화:**

```
라이브러리 시대:
능동적 → 필요할 때 호출

프레임워크 시대:
수동적 → 호출될 때까지 대기
```

#### 🌟 프레임워크의 핵심

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  협력을 제어하는 것은 프레임워크                            │
│                                                     │
│  애플리케이션 개발자의 코드는                              │
│  상위 모듈(프레임워크)에 의해 호출된다                       │
│                                                     │
│  제어가 프레임워크로 넘어갔다                              │
│  = 의존성이 역전되었다!                                  │
│                                                     │
└─────────────────────────────────────────────────────┘
```

### 2.5 상위 정책 재사용

#### 🎯 핵심 통찰

```
상위 정책을 재사용한다는 것 =
도메인에 존재하는 핵심 개념들 사이의
협력 관계를 재사용한다
```

**예시:**

```
핸드폰 과금 도메인의 핵심 개념:
- 요금 정책
- 기본 정책
- 부가 정책
- 통화
- 요금 계산

이들 사이의 협력:
Phone → RatePolicy
BasicRatePolicy → FeeRule
AdditionalRatePolicy → next

→ 이 협력 구조가 프레임워크!
```

#### 💎 객체지향 설계에서의 재사용

```
객체지향 설계에서 재사용성은
객체들 사이의 공통적인 협력 흐름으로부터 나온다

역할 사이의 공통적인 협력 흐름이 있다면
다양한 문맥에 맞는 책임을 구현하면서
설계를 재사용할 수 있다
```

**시각화:**

```
공통 협력 흐름 (프레임워크):
Phone → RatePolicy → BasicRatePolicy

문맥별 구현 (애플리케이션):
- 일반 요금제 시스템: RegularPolicy
- 심야 요금제 시스템: NightlyPolicy
- 시간대별 시스템: TimeOfDayPolicy

→ 협력은 재사용하되
  구체적 책임은 각자 구현!
```

---

## 3. 핵심 정리

### 🎯 디자인 패턴과 프레임워크의 본질

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  디자인 패턴:                                          │
│  특정한 변경을 일관성 있게 다룰 수 있는                      │
│  협력 템플릿                                           │
│                                                     │
│  프레임워크:                                           │
│  특정한 변경을 일관성 있게 다룰 수 있는                      │
│  확장 가능한 코드 템플릿                                  │
│                                                     │
└─────────────────────────────────────────────────────┘
```

### 📊 디자인 패턴 vs 프레임워크

| 측면 | 디자인 패턴 | 프레임워크 |
|------|-----------|-----------|
| **제공** | 협력 템플릿 | 코드 템플릿 |
| **재사용** | 설계 재사용 | 설계 + 코드 재사용 |
| **형태** | 문서, 아이디어 | 실행 가능한 코드 |
| **적용** | 개발자가 구현 | 확장 포인트 구현 |
| **제어** | 개발자 | 프레임워크 (제어 역전) |

### 🔑 핵심 개념 정리

#### 1. 패턴의 본질

```
패턴 = 반복적 문제 + 검증된 해법

특징:
- 실무 경험에서 비롯
- 3의 규칙 (최소 3곳에서 검증)
- 공통 어휘 제공
- 역할 기반 (클래스 기반 X)
```

#### 2. 패턴 분류

```
아키텍처 패턴: 시스템 전체 구조
    ↓
디자인 패턴: 컴포넌트 협력 구조
    ↓
이디엄: 언어별 구현 방법

분석 패턴: 도메인 개념 구조
```

#### 3. 주요 디자인 패턴

```
STRATEGY:
- 목적: 알고리즘 변경 캡슐화
- 기법: 객체 합성
- 장점: 런타임 변경 가능

TEMPLATE METHOD:
- 목적: 알고리즘 변경 캡슐화
- 기법: 상속
- 장점: 단순한 구조

DECORATOR:
- 목적: 행동 순서/개수 변경 캡슐화
- 기법: 객체 합성
- 장점: 동적 조합
```

#### 4. 프레임워크의 핵심

```
구성:
- 애플리케이션 아키텍처
- 기반 코드
- 추상 클래스/인터페이스
- 재사용 가능한 컴포넌트

원리:
- 의존성 역전 (DIP)
- 제어 역전 (IoC)
- 훅 (Hook)
```

#### 5. 제어 역전

```
라이브러리: 개발자가 호출
프레임워크: 프레임워크가 호출

제어의 주체가 역전됨!
```

### 💡 실전 가이드

#### 디자인 패턴 사용 시

```
✅ Do:
- 문제부터 시작
- 트레이드오프 고려
- 목적에 맞게 수정
- 팀과 공유

❌ Don't:
- 패턴을 목표로 삼기
- 맹목적으로 따르기
- 모든 곳에 적용
- 복잡도 무시
```

#### 프레임워크 설계 시

```
핵심 원칙:

1. 상위 정책과 하위 정책 분리
   - 변하지 않는 것: 프레임워크
   - 변하는 것: 애플리케이션

2. 의존성 역전 적용
   - 하위가 상위에 의존
   - 추상화에 의존

3. 적절한 훅 제공
   - 확장 포인트 명확히
   - 호출 시점 문서화

4. 공통 협력 추출
   - 재사용 가능한 구조
   - 역할 기반 설계
```

### 🎓 학습 단계 (Alistair Cockburn)

#### 3단계 숙련도

```
1️⃣ 따라 하는 수준 (Shu):
- 모방
- 주어진 절차를 따름
- 결과물 얻는 안정감

2️⃣ 분리 수준 (Ha):
- 다양한 절차 학습
- 트레이드오프 이해
- 상황에 맞는 절차 적용
- 판단력과 유연함

3️⃣ 거침없는 수준 (Ri):
- 해법을 직관적으로 떠올림
- 절차를 넘어선 창의성
- 가장 적합한 방법 선택
```

#### 학습 경로

```
디자인 패턴 학습
    ↓
리팩터링 연습
    ↓
테스트 주도 개발 (TDD)
    ↓
'분리 수준'으로 성장
    ↓
실무 경험 축적
    ↓
'거침없는 수준'으로 진화
```

### 📚 설계자의 문서화

```
거침없는 수준의 설계자:
- 직관적으로 설계 완료

↓ 문서화

분리 수준의 관점:
- 트레이드오프 설명
- 대안 비교
- 결정 이유 제시

↓ 문서화

따라 하는 수준의 사람도 이해:
- 단계별 설명
- 구체적 예시
- 명확한 가이드
```

### 🌟 마무리 메시지

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  소프트웨어를 설계하는                                    │
│  단 하나의 방법이란 존재하지 않는다                          │
│                                                     │
│  중요한 것은:                                          │
│  현재의 설계에 맹목적으로 일관성을 맞추는 것이 아니라            │
│  변경의 방향에 맞춰                                     │
│  지속적으로 개선하려는 의지다                              │
│                                                     │
│  디자인 패턴과 프레임워크는                                │
│  그 여정의 훌륭한 동반자다                                │
│                                                     │
└─────────────────────────────────────────────────────┘
```

### 🎯 핵심 교훈

```
1. 패턴은 경험의 산물
   → 검증된 해법 활용

2. 패턴은 출발점
   → 맹목적 적용 지양

3. 역할 기반 사고
   → 구현 방법은 다양

4. 변경을 캡슐화
   → 패턴의 목적 이해

5. 프레임워크 = 설계 + 코드
   → 재사용의 최고 형태

6. 제어 역전
   → 프레임워크의 핵심

7. 협력 흐름 재사용
   → 상위 정책의 가치

8. 지속적 개선
   → 완벽한 설계는 없음
```

---

## 🔗 연결고리

### 이전 장과의 연결
- **Chapter 14**: 일관성 있는 협력 → 패턴으로 구체화
- **전체 책**: 역할, 책임, 협력 → 패턴의 기초
- 변경의 캡슐화 → 모든 패턴의 목적

### 실무로의 확장
- **다음 단계**: 
  - 디자인 패턴 깊이 학습 (GoF 패턴)
  - 리팩터링 실천
  - TDD 적용
  - 프레임워크 소스 분석
  - 오픈소스 기여

---

<div align="center">

**[⬆ 목차로 돌아가기](../README.md)**

</div>

<div align="center">

**[⬅️ 이전: Chapter 14](../chapter14/README.md)** | **[다음: Appendix A ➡️](../appendixA/README.md)**

---

**🎉 오브젝트 여정의 완성을 축하합니다! 🎉**

이제 실무에서 이 지식을 활용하며  
**분리 수준**을 향해 나아가세요!

</div>
