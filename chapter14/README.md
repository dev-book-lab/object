# Chapter 14. 일관성 있는 협력

> *"유사한 기능을 구현하기 위해 유사한 협력 패턴을 사용하라."*

## 📌 핵심 개념

이 장에서는 **일관성 있는 협력**을 다룹니다. 유사한 요구사항을 반복적으로 추가할 때, 일관된 협력 패턴을 유지함으로써 설계의 품질을 높이고 유지보수성을 향상시키는 방법을 학습합니다.

### 🎯 학습 목표

- 일관성 있는 협력의 중요성 이해하기
- 변하는 것과 변하지 않는 것을 분리하는 방법 파악하기
- 변경을 캡슐화하는 다양한 기법 이해하기
- 협력 패턴을 발견하고 적용하는 과정 학습하기
- 조건 로직을 객체 탐색으로 전환하기
- 개념적 무결성을 유지하는 설계 만들기

---

## 📖 목차

1. [핸드폰 과금 시스템 변경하기](#1-핸드폰-과금-시스템-변경하기)
2. [설계에 일관성 부여하기](#2-설계에-일관성-부여하기)
3. [일관성 있는 기본 정책 구현하기](#3-일관성-있는-기본-정책-구현하기)
4. [핵심 정리](#4-핵심-정리)

---

## 1. 핸드폰 과금 시스템 변경하기

> 📂 **코드**: step01 - 비일관적 구현 / step02 - 일관성 있는 구현

### 1.1 기본 정책 확장

#### 📋 요구사항: 4가지 기본 정책

| 유형 | 형식 | 예시 |
|------|------|------|
| **고정요금** | A초당 B원 | 10초당 18원 |
| **시간대별** | 시간대마다 A초당 B원 | 0시~19시: 10초당 18원<br>19시~24시: 10초당 15원 |
| **요일별** | 요일마다 A초당 B원 | 평일: 10초당 38원<br>공휴일: 10초당 19원 |
| **구간별** | 통화 구간마다 A초당 B원 | 초기 1분: 10초당 50원<br>1분 이후: 10초당 20원 |

#### 🎯 부가 정책 (11장에서 구현)

```
기본 정책 + 세금 정책
기본 정책 + 요금 할인 정책
기본 정책 + 세금 정책 + 요금 할인 정책

→ 데코레이터 패턴으로 구현됨
```

#### 📊 조합 가능한 경우의 수

```
4가지 기본 정책 × 3가지 부가 정책 조합
= 총 12가지 조합

┌──────────────────────────────────────┐
│         기본 정책 (4가지)               │
├──────────────────────────────────────┤
│  • FixedFeePolicy                    │
│  • TimeOfDayDiscountPolicy           │
│  • DayOfWeekDiscountPolicy           │
│  • DurationDiscountPolicy            │
└──────────────────────────────────────┘
         ↓
┌──────────────────────────────────────┐
│      부가 정책 (데코레이터)               │
├──────────────────────────────────────┤
│  • TaxablePolicy                     │
│  • RateDiscountablePolicy            │
│  • TaxablePolicy + RateDiscountable  │
└──────────────────────────────────────┘
```

### 1.2 고정요금 방식 구현

#### 📝 FixedFeePolicy

```java
public class FixedFeePolicy extends BasicRatePolicy {
    private Money amount;      // 단위 요금
    private Duration seconds;  // 단위 시간

    public FixedFeePolicy(Money amount, Duration seconds) {
        this.amount = amount;
        this.seconds = seconds;
    }

    @Override
    protected Money calculateCallFee(Call call) {
        return amount.times(
                call.getDuration().getSeconds() / seconds.getSeconds()
        );
    }
}
```

**특징:**
```
✅ 가장 단순한 구현
✅ 통화 시간을 단위 시간으로 나눠서 요금 계산
✅ 11장의 일반 요금제와 동일
```

### 1.3 시간대별 방식 구현

#### 🎯 핵심 문제

```
시간대별 방식의 복잡성:

1. 통화는 여러 날에 걸쳐 이루어질 수 있다
   - 시작 일자 ≠ 종료 일자
   
2. 각 날짜마다 시간대가 다를 수 있다
   - 첫째 날: 23시~24시
   - 둘째 날: 0시~1시
   
3. 시간대별로 다른 요금 적용
   - 0시~19시: 10초당 18원
   - 19시~24시: 10초당 15원
```

#### 📅 DateTimeInterval 클래스

**책임: 기간 처리의 정보 전문가**

```java
public class DateTimeInterval {
    private LocalDateTime from;
    private LocalDateTime to;

    // 정적 팩토리 메서드들
    public static DateTimeInterval of(LocalDateTime from, LocalDateTime to) {
        return new DateTimeInterval(from, to);
    }

    public static DateTimeInterval toMidnight(LocalDateTime from) {
        return new DateTimeInterval(
                from,
                LocalDateTime.of(from.toLocalDate(), LocalTime.of(23, 59, 59))
        );
    }

    public static DateTimeInterval fromMidnight(LocalDateTime to) {
        return new DateTimeInterval(
                LocalDateTime.of(to.toLocalDate(), LocalTime.of(0, 0)),
                to
        );
    }

    public static DateTimeInterval during(LocalDate date) {
        return new DateTimeInterval(
                LocalDateTime.of(date, LocalTime.of(0, 0)),
                LocalDateTime.of(date, LocalTime.of(23, 59, 59))
        );
    }

    // 핵심 메서드: 일자별로 분리
    public List<DateTimeInterval> splitByDay() {
        if (days() > 0) {
            return split(days());
        }
        return Arrays.asList(this);
    }

    private List<DateTimeInterval> split(long days) {
        List<DateTimeInterval> result = new ArrayList<>();
        addFirstDay(result);      // 첫날: from ~ 자정
        addMiddleDays(result, days); // 중간날들: 0시 ~ 자정
        addLastDay(result);       // 마지막날: 자정 ~ to
        return result;
    }
}
```

**동작 예시:**

```
통화: 2024-01-05 23:00 ~ 2024-01-07 01:00

splitByDay() 결과:
[
  2024-01-05 23:00 ~ 2024-01-05 23:59,  // 첫날
  2024-01-06 00:00 ~ 2024-01-06 23:59,  // 중간날
  2024-01-07 00:00 ~ 2024-01-07 01:00   // 마지막날
]
```

#### 📞 Call 클래스

```java
public class Call {
    private DateTimeInterval interval;

    public Call(LocalDateTime from, LocalDateTime to) {
        this.interval = DateTimeInterval.of(from, to);
    }

    public Duration getDuration() {
        return interval.duration();
    }

    public List<DateTimeInterval> splitByDay() {
        return interval.splitByDay();
    }

    public DateTimeInterval getInterval() {
        return interval;
    }
}
```

#### ⏰ TimeOfDayDiscountPolicy

**책임: 시간대별 요금 계산의 정보 전문가**

```java
public class TimeOfDayDiscountPolicy extends BasicRatePolicy {
    private List<LocalTime> starts = new ArrayList<>();
    private List<LocalTime> ends = new ArrayList<>();
    private List<Duration> durations = new ArrayList<>();
    private List<Money> amounts = new ArrayList<>();

    @Override
    protected Money calculateCallFee(Call call) {
        Money result = Money.ZERO;

        // 일자별로 분리
        for (DateTimeInterval interval : call.splitByDay()) {
            // 각 시간대별 규칙 적용
            for (int loop = 0; loop < starts.size(); loop++) {
                result = result.plus(
                        amounts.get(loop).times(
                                Duration.between(
                                        from(interval, starts.get(loop)),
                                        to(interval, ends.get(loop))
                                ).getSeconds() / durations.get(loop).getSeconds()
                        )
                );
            }
        }

        return result;
    }

    private LocalTime from(DateTimeInterval interval, LocalTime from) {
        return interval.getFrom().toLocalTime().isBefore(from)
                ? from
                : interval.getFrom().toLocalTime();
    }

    private LocalTime to(DateTimeInterval interval, LocalTime to) {
        return interval.getTo().toLocalTime().isAfter(to)
                ? to
                : interval.getTo().toLocalTime();
    }
}
```

**구조:**

```
TimeOfDayDiscountPolicy
│
├─ starts:    [0:00, 19:00]
├─ ends:      [19:00, 24:00]
├─ durations: [10초, 10초]
└─ amounts:   [18원, 15원]

인덱스가 같은 요소들이 하나의 규칙을 구성
```

### 1.4 요일별 방식 구현

#### 📅 DayOfWeekDiscountRule

```java
public class DayOfWeekDiscountRule {
    private List<DayOfWeek> dayOfWeeks = new ArrayList<>();
    private Duration duration = Duration.ZERO;
    private Money amount = Money.ZERO;

    public DayOfWeekDiscountRule(List<DayOfWeek> dayOfWeeks,
            Duration duration,
            Money amount) {
        this.dayOfWeeks = dayOfWeeks;
        this.duration = duration;
        this.amount = amount;
    }

    public Money calculate(DateTimeInterval interval) {
        if (dayOfWeeks.contains(interval.getFrom().getDayOfWeek())) {
            return amount.times(
                    interval.duration().getSeconds() / duration.getSeconds()
            );
        }
        return Money.ZERO;
    }
}
```

#### 📊 DayOfWeekDiscountPolicy

```java
public class DayOfWeekDiscountPolicy extends BasicRatePolicy {
    private List<DayOfWeekDiscountRule> rules = new ArrayList<>();

    public DayOfWeekDiscountPolicy(List<DayOfWeekDiscountRule> rules) {
        this.rules = rules;
    }

    @Override
    protected Money calculateCallFee(Call call) {
        Money result = Money.ZERO;

        // 일자별로 분리
        for (DateTimeInterval interval : call.getInterval().splitByDay()) {
            // 각 규칙 적용
            for (DayOfWeekDiscountRule rule : rules) {
                result = result.plus(rule.calculate(interval));
            }
        }

        return result;
    }
}
```

**구조:**

```
DayOfWeekDiscountPolicy
│
└─ rules: [
     Rule1: 평일(월~금), 10초당 38원
     Rule2: 주말(토~일), 10초당 19원
   ]
```

### 1.5 구간별 방식 구현

#### ⏱️ DurationDiscountRule

```java
public class DurationDiscountRule extends FixedFeePolicy {
    private Duration from;  // 구간 시작
    private Duration to;    // 구간 종료

    public DurationDiscountRule(Duration from, Duration to,
            Money amount, Duration seconds) {
        super(amount, seconds);
        this.from = from;
        this.to = to;
    }

    public Money calculate(Call call) {
        // 통화 시간이 구간 범위를 벗어나면 0원
        if (call.getDuration().compareTo(to) > 0) {
            return Money.ZERO;
        }
        if (call.getDuration().compareTo(from) < 0) {
            return Money.ZERO;
        }

        // 부모 클래스 재사용을 위한 임시 Phone 생성
        Phone phone = new Phone(null);
        phone.call(new Call(
                call.getFrom().plus(from),
                call.getDuration().compareTo(to) > 0
                        ? call.getFrom().plus(to)
                        : call.getTo()
        ));

        return super.calculateFee(phone);
    }
}
```

#### 📊 DurationDiscountPolicy

```java
public class DurationDiscountPolicy extends BasicRatePolicy {
    private List<DurationDiscountRule> rules = new ArrayList<>();

    public DurationDiscountPolicy(List<DurationDiscountRule> rules) {
        this.rules = rules;
    }

    @Override
    protected Money calculateCallFee(Call call) {
        Money result = Money.ZERO;

        for (DurationDiscountRule rule : rules) {
            result = result.plus(rule.calculate(call));
        }

        return result;
    }
}
```

**구조:**

```
DurationDiscountPolicy
│
└─ rules: [
     Rule1: 0분~1분, 10초당 50원
     Rule2: 1분~∞,  10초당 20원
   ]
```

### 1.6 문제점: 비일관성

#### 🚨 구현 방식의 차이

```
FixedFeePolicy:
- 단위 시간, 단위 요금만 인스턴스 변수로 보유
- 규칙이라는 개념 없음

TimeOfDayDiscountPolicy:
- 여러 List로 시간대 정보 관리
- 인덱스로 규칙 연결

DayOfWeekDiscountPolicy:
- DayOfWeekDiscountRule 객체로 규칙 표현
- 규칙 객체의 리스트로 관리

DurationDiscountPolicy:
- DurationDiscountRule 객체로 규칙 표현
- FixedFeePolicy 상속 (코드 재사용)
```

#### 💥 비일관성의 문제

```
문제 1: 새로운 구현 추가 시
→ 어떤 방식을 따라야 할지 혼란
→ 새로운 방식으로 구현하면 일관성 더 무너짐

문제 2: 기존 구현 이해 시
→ 각 정책마다 다른 방식으로 이해해야 함
→ 학습 비용 증가

문제 3: 유지보수 시
→ 유사한 기능인데 수정 방법이 다름
→ 실수 가능성 증가
```

#### 🎯 핵심 교훈

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  유사한 기능은 유사한 방식으로 구현해야 한다!                  │
│                                                     │
│  비일관성은 설계의 적!                                   │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

## 2. 설계에 일관성 부여하기

### 2.1 일관성 있는 설계를 만드는 방법

#### 📚 첫 번째 조언: 다양한 설계 경험

```
일관성 있는 설계를 만드는 가장 좋은 방법:
→ 다양한 설계 경험을 익히는 것

하지만:
→ 단기간에 설계 경험을 쌓기는 어려움
```

#### 🎨 두 번째 조언: 디자인 패턴 학습

```
디자인 패턴이란?
→ 특정한 변경에 대해 일관성 있는 설계를 만들 수 있는
   경험 법칙을 모아놓은 설계 템플릿

장점:
1. 검증된 해결책 제공
2. 공통 어휘 제공 (의사소통 향상)
3. 변경에 대한 대응 방법 제시
```

### 2.2 협력을 일관성 있게 만드는 기본 지침

#### 📋 두 가지 핵심 지침

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  1. 변하는 개념을 변하지 않는 개념으로부터 분리하라             │
│                                                     │
│  2. 변하는 개념을 캡슐화하라                              │
│                                                     │
└─────────────────────────────────────────────────────┘
```

**연결:**
```
이 두 지침은 모든 설계 원칙의 기반:

- 단일 책임 원칙 (SRP)
- 개방-폐쇄 원칙 (OCP)
- 의존성 역전 원칙 (DIP)
- 리스코프 치환 원칙 (LSP)
- 인터페이스 분리 원칙 (ISP)

모두 변경의 캡슐화를 목표로 함!
```

### 2.3 조건 로직 대 객체 탐색

#### ❌ 조건 로직 기반 설계

**나쁜 예: ReservationAgency (2장)**

```java
public class ReservationAgency {
    public Reservation reserve(Screening screening,
            Customer customer,
            int audienceCount) {
        Movie movie = screening.getMovie();

        boolean discountable = false;

        // 조건 로직 1: 할인 조건 판단
        for (DiscountCondition condition : movie.getDiscountConditions()) {
            if (condition.getType() == DiscountConditionType.PERIOD) {
                // 기간 조건인 경우
                // ... 복잡한 로직
            } else {
                // 회차 조건인 경우
                // ... 복잡한 로직
            }

            if (discountable) {
                break;
            }
        }

        Money fee;

        // 조건 로직 2: 할인 정책 판단
        if (discountable) {
            Money discountAmount = Money.ZERO;

            switch (movie.getMovieType()) {
                case AMOUNT_DISCOUNT:
                    // 금액 할인 정책
                    discountAmount = movie.getDiscountAmount();
                    break;
                case PERCENT_DISCOUNT:
                    // 비율 할인 정책
                    discountAmount = movie.getFee()
                            .times(movie.getDiscountPercent());
                    break;
                case NONE_DISCOUNT:
                    // 할인 없음
                    discountAmount = Money.ZERO;
                    break;
            }

            fee = movie.getFee()
                    .minus(discountAmount)
                    .times(audienceCount);
        } else {
            fee = movie.getFee().times(audienceCount);
        }

        return new Reservation(customer, screening, fee, audienceCount);
    }
}
```

**문제점:**

```
1. 변경의 주기가 다른 코드가 한 클래스에 뭉쳐있음
   - 할인 조건 추가
   - 할인 정책 추가
   - 예매 로직 변경
   
2. 새로운 타입 추가 시 기존 코드 수정 필요
   - OCP 위반
   
3. 코드 이해하기 어려움
   - 복잡한 중첩된 조건문
   
4. 오류 발생 확률 높음
   - 수정 범위가 넓음
```

#### ✅ 객체 탐색 기반 설계

**좋은 예: DiscountPolicy 계층 (2장 개선)**

```java
public abstract class DiscountPolicy {
    private List<DiscountCondition> conditions = new ArrayList<>();

    public DiscountPolicy(DiscountCondition... conditions) {
        this.conditions = Arrays.asList(conditions);
    }

    public Money calculateDiscountAmount(Screening screening) {
        for (DiscountCondition each : conditions) {
            if (each.isSatisfiedBy(screening)) {
                return getDiscountAmount(screening);
            }
        }
        return screening.getMovieFee();
    }

    abstract protected Money getDiscountAmount(Screening screening);
}
```

```java
public class Movie {
    private DiscountPolicy discountPolicy;

    public Money calculateMovieFee(Screening screening) {
        // 조건 로직 없음!
        // 단순히 메시지 전송
        return fee.minus(discountPolicy.calculateDiscountAmount(screening));
    }
}
```

**장점:**

```
1. 타입 판단 로직 제거
   - Movie는 DiscountPolicy의 구체 타입을 몰라도 됨
   
2. 새로운 정책 추가 시 기존 코드 수정 불필요
   - OCP 만족
   
3. 코드 이해하기 쉬움
   - 단순한 메시지 전송
   
4. 변경의 영향 최소화
   - 각 정책은 독립적
```

#### 🎯 다형성의 역할

```
다형성은
조건 로직을 객체 사이의 이동으로 바꾼다!

조건 로직:
if (type == A) { ... }
else if (type == B) { ... }
else if (type == C) { ... }

↓ 다형성 적용

객체 탐색:
object.operation();
→ 실제 타입에 따라 적절한 메서드 실행
```

**시각화:**

```
조건 로직 방식:
Movie
  │
  ├─ if (할인 조건)
  │    └─ switch (할인 정책)
  │         ├─ case 금액할인
  │         ├─ case 비율할인
  │         └─ case 할인없음
  └─ else ...

객체 탐색 방식:
Movie ─── message ──▶ DiscountPolicy
                           ↑
                    ┌──────┼──────┐
              AmountDiscount PercentDiscount NoneDiscount
                    (다형적으로 실행)
```

### 2.4 캡슐화 다시 살펴보기

#### 🎯 캡슐화 ≠ 데이터 은닉

**전통적 정의:**
```
캡슐화 = 데이터 은닉

private 변수 + public getter/setter
```

**진정한 의미:**
```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  캡슐화란 변하는 어떤 것이든 감추는 것이다!                   │
│                                                     │
│  "Encapsulate the concept that varies"              │
│                                                     │
└─────────────────────────────────────────────────────┘
```

#### 📦 캡슐화의 종류

**1. 데이터 캡슐화 (Data Encapsulation)**

```java
public class Movie {
    private String title;        // ✅ private 데이터
    private Duration runningTime;
    private Money fee;

    // public 메서드로만 접근
    public Money getFee() {
        return fee;
    }
}
```

**목적:**
```
내부 데이터 구조 변경 시
외부 영향 최소화
```

**2. 메서드 캡슐화 (Method Encapsulation)**

```java
public class DiscountPolicy {
    // public - 외부에서 호출
    public Money calculateDiscountAmount(Screening screening) {
        for (DiscountCondition each : conditions) {
            if (each.isSatisfiedBy(screening)) {
                return getDiscountAmount(screening);
            }
        }
        return Money.ZERO;
    }

    // protected - 서브클래스에서만 오버라이드
    abstract protected Money getDiscountAmount(Screening screening);
}
```

**목적:**
```
메서드 구현 변경 시
클래스 외부에 영향 없음
```

**3. 객체 캡슐화 (Object Encapsulation)**

```java
public class Movie {
    private DiscountPolicy discountPolicy;  // ✅ 합성

    public Movie(String title, Duration runningTime,
            Money fee, DiscountPolicy discountPolicy) {
        this.title = title;
        this.runningTime = runningTime;
        this.fee = fee;
        this.discountPolicy = discountPolicy;
    }
}
```

**목적:**
```
객체 사이의 관계 변경 시
외부에 영향 없음

= 합성 (Composition)
```

**4. 서브타입 캡슐화 (Subtype Encapsulation)**

```java
public class Movie {
    private DiscountPolicy discountPolicy;  // 추상 타입

    public Money calculateMovieFee(Screening screening) {
        // 구체 타입을 알 필요 없음!
        return fee.minus(discountPolicy.calculateDiscountAmount(screening));
    }
}
```

**목적:**
```
구체적인 서브타입 종류 캡슐화

Movie는 다음을 모름:
- AmountDiscountPolicy
- PercentDiscountPolicy
- NoneDiscountPolicy

= 다형성 (Polymorphism)
```

#### 📊 캡슐화 종류별 적용 범위

| 캡슐화 종류 | 적용 대상 | 목적 | 기법 |
|-----------|---------|------|------|
| **데이터** | 개별 객체 | 데이터 구조 변경 관리 | private + getter |
| **메서드** | 개별 객체 | 메서드 구현 변경 관리 | protected, private |
| **객체** | 협력 관계 | 객체 관계 변경 관리 | 합성 |
| **서브타입** | 협력 관계 | 타입 종류 변경 관리 | 다형성 |

#### 💡 핵심 통찰

```
일반적으로:

데이터 캡슐화 + 메서드 캡슐화
→ 개별 객체의 변경 관리

객체 캡슐화 + 서브타입 캡슐화
→ 협력 참여 객체들의 관계 변경 관리

일관성 있는 협력을 만들기 위해:
→ 서브타입 캡슐화 + 객체 캡슐화 조합
  = 인터페이스 상속 + 합성
```

### 2.5 변경을 캡슐화하는 방법

#### 📋 2단계 프로세스

**1단계: 변하는 부분을 분리해서 타입 계층 만들기**

```
변하지 않는 부분으로부터 변하는 부분을 분리

변하는 부분들의 공통적인 행동을
추상 클래스나 인터페이스로 추상화

변하는 부분들이
이 추상 클래스나 인터페이스를 상속/구현

→ 변하는 부분은 변하지 않는 부분의 서브타입이 됨
```

**2단계: 변하지 않는 부분의 일부로 타입 계층 합성하기**

```
앞에서 구현한 타입 계층을
변하지 않는 부분에 합성

변하지 않는 부분에서는
변경되는 구체적인 사항에 결합되면 안 됨

의존성 주입 등을 이용해
오직 추상화에만 의존

→ 변경이 캡슐화됨!
```

#### 🎨 예시: DiscountPolicy

**1단계: 타입 계층 만들기**

```java
// 추상화
public abstract class DiscountPolicy {
    abstract protected Money getDiscountAmount(Screening screening);
}

// 변하는 부분들 (서브타입)
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

**2단계: 합성하기**

```java
// 변하지 않는 부분
public class Movie {
    private DiscountPolicy discountPolicy;  // 추상화에 의존!

    public Movie(String title, Duration runningTime,
            Money fee, DiscountPolicy discountPolicy) {
        // 의존성 주입
        this.discountPolicy = discountPolicy;
    }

    public Money calculateMovieFee(Screening screening) {
        // 구체 타입을 알 필요 없음
        return fee.minus(discountPolicy.calculateDiscountAmount(screening));
    }
}
```

**결과:**

```
변경 캡슐화 완료!

새로운 할인 정책 추가:
1. DiscountPolicy 상속
2. getDiscountAmount() 구현
3. Movie 코드 수정 불필요!

Movie는 다음을 모름:
- 구체적인 할인 정책 종류
- 할인 정책 계산 방법
- 할인 정책 내부 구현

→ 완벽한 캡슐화!
```

---

## 3. 일관성 있는 기본 정책 구현하기

> 📂 **코드**: step02 - 일관성 있는 최종 구현

### 3.1 변경 분리하기

#### 🔍 공통점 찾기

**4가지 기본 정책 분석:**

```
고정요금:
규칙 = [단위시간]당 [요금]원

시간대별:
규칙 = [시작시간]~[종료시간]까지 [단위시간]당 [요금]원

요일별:
규칙 = [요일]별 [단위시간]당 [요금]원

구간별:
규칙 = [통화구간] 동안 [단위시간]당 [요금]원
```

#### 💡 패턴 발견

```
공통 구조:
기본 정책 = 하나 이상의 "규칙"들의 집합

각 규칙:
규칙 = "적용조건" + "단위요금"

적용조건 (변하는 부분):
- 시간대
- 요일
- 통화 구간
- 항상 (고정요금)

단위요금 (변하지 않는 부분):
- [단위시간]당 [요금]원
```

#### 🎯 변경 분리

```
변하는 것:
→ 적용조건
  - 시간대별, 요일별, 구간별, 고정

변하지 않는 것:
→ 규칙
→ 단위요금
```

**시각화:**

```
┌─────────────────────────────────────┐
│         기본 정책                     │
├─────────────────────────────────────┤
│                                     │
│  ┌───────────────────────────────┐  │
│  │        규칙 1                  │  │
│  ├───────────────────────────────┤  │
│  │ 적용조건   │  단위요금             │  │
│  │ (변함)    │  (변하지 않음)        │  │
│  └───────────────────────────────┘  │
│                                     │
│  ┌───────────────────────────────┐  │
│  │        규칙 2                  │  │
│  ├───────────────────────────────┤  │
│  │ 적용조건   │  단위요금             │  │
│  │ (변함)    │  (변하지 않음)        │  │
│  └───────────────────────────────┘  │
│                                     │
│  ...                                │
│                                     │
└─────────────────────────────────────┘
```

### 3.2 변경 캡슐화하기

#### 🎨 도메인 모델

```
┌──────────────┐       ┌──────────────┐
│ BasicRate    │ 1   * │   FeeRule    │
│ Policy       ├───────│   (규칙)      │
└──────────────┘       └──────┬───────┘
                              │ 1
                              │
                    ┌─────────┴─────────┐
                    │                   │
                    │ 1                 │ 1
             ┌──────▼──────┐    ┌──────▼───────┐
             │FeeCondition │    │FeePerDuration│
             │(적용조건)     │    │(단위요금)      │
             └──────┬──────┘    └──────────────┘
                    ▲
                    │
        ┌───────────┼───────────┬───────────┐
        │           │           │           │
┌───────┴──┐ ┌──────┴───┐ ┌─────┴────┐ ┌────┴────┐
│TimeOfDay │ │DayOfWeek │ │Duration  │ │Fixed    │
│Condition │ │Condition │ │Condition │ │Condition│
└──────────┘ └──────────┘ └──────────┘ └─────────┘
```

**핵심:**
```
FeeRule은 추상화인 FeeCondition에만 의존
→ "적용조건"이라는 변경에 대해 캡슐화됨!
```

### 3.3 협력 패턴 설계하기

#### 🎯 책임 할당

**작업 1: 적용조건을 만족하는 구간 찾기**

```
누구의 책임?
→ FeeCondition (적용조건의 정보 전문가)

메서드:
List<DateTimeInterval> findTimeIntervals(Call call)
```

**작업 2: 구간에 단위요금 적용하기**

```
누구의 책임?
→ FeeRule (규칙의 정보 전문가)

메서드:
Money calculateFee(Call call)
```

#### 📊 협력 흐름

```
Phone
  │
  │ calculateFee()
  ▼
BasicRatePolicy
  │
  │ calculate(call)
  ▼
FeeRule
  │
  │ calculateFee(call)
  │
  ├──▶ FeeCondition.findTimeIntervals(call)
  │      │
  │      └─▶ List<DateTimeInterval> 반환
  │
  └──▶ FeePerDuration.calculate(interval) (각 구간마다)
         │
         └─▶ Money 반환
```

**시각화:**

```
┌─────────────┐
│   Phone     │
└──────┬──────┘
       │ calculateFee()
       ▼
┌─────────────────┐
│ BasicRatePolicy │
└──────┬──────────┘
       │ calculate(call)
       ▼
┌──────────────────────────────────┐
│          FeeRule                 │
├──────────────────────────────────┤
│                                  │
│  1. findTimeIntervals(call) ────▶│ FeeCondition
│     ← List<DateTimeInterval>     │
│                                  │
│  2. for each interval:           │
│     calculate(interval) ────────▶│ FeePerDuration
│     ← Money                      │
│                                  │
│  3. sum all Money                │
│                                  │
└──────────────────────────────────┘
```

### 3.4 추상화 수준에서 협력 패턴 구현하기

#### 🎯 FeeCondition 인터페이스

```java
public interface FeeCondition {
    /**
     * 통화 기간 중 적용조건을 만족하는 구간들을 반환
     */
    List<DateTimeInterval> findTimeIntervals(Call call);
}
```

#### 📏 FeePerDuration 클래스

```java
public class FeePerDuration {
    private Money fee;        // 단위 요금
    private Duration duration;  // 단위 시간

    public FeePerDuration(Money fee, Duration duration) {
        this.fee = fee;
        this.duration = duration;
    }

    public Money calculate(DateTimeInterval interval) {
        return fee.times(
                Math.ceil(
                        (double) interval.duration().toNanos() /
                                duration.toNanos()
                )
        );
    }
}
```

#### 🎯 FeeRule 클래스

```java
public class FeeRule {
    private FeeCondition feeCondition;      // 적용조건
    private FeePerDuration feePerDuration;  // 단위요금

    public FeeRule(FeeCondition feeCondition,
            FeePerDuration feePerDuration) {
        this.feeCondition = feeCondition;
        this.feePerDuration = feePerDuration;
    }

    public Money calculateFee(Call call) {
        return feeCondition.findTimeIntervals(call)  // 조건 만족 구간
                .stream()
                .map(each -> feePerDuration.calculate(each))  // 각 구간 요금
                .reduce(Money.ZERO, (first, second) -> first.plus(second));  // 합계
    }
}
```

#### 🏛️ BasicRatePolicy 클래스

```java
public final class BasicRatePolicy implements RatePolicy {
    private List<FeeRule> feeRules = new ArrayList<>();

    public BasicRatePolicy(FeeRule... feeRules) {
        this.feeRules = Arrays.asList(feeRules);
    }

    @Override
    public Money calculateFee(Phone phone) {
        return phone.getCalls()
                .stream()
                .map(call -> calculate(call))
                .reduce(Money.ZERO, (first, second) -> first.plus(second));
    }

    private Money calculate(Call call) {
        return feeRules
                .stream()
                .map(rule -> rule.calculateFee(call))
                .reduce(Money.ZERO, (first, second) -> first.plus(second));
    }
}
```

**핵심:**
```
✅ 추상적인 수준에서 협력 완성
✅ 구체적인 FeeCondition 구현체 없이도 동작 정의
✅ 확장 포인트는 FeeCondition 인터페이스
```

### 3.5 구체적인 협력 구현하기

#### ⏰ 시간대별 정책

```java
public class TimeOfDayFeeCondition implements FeeCondition {
    private LocalTime from;
    private LocalTime to;

    public TimeOfDayFeeCondition(LocalTime from, LocalTime to) {
        this.from = from;
        this.to = to;
    }

    @Override
    public List<DateTimeInterval> findTimeIntervals(Call call) {
        return call.getInterval().splitByDay()
                .stream()
                .filter(each -> from(each).isBefore(to(each)))
                .map(each -> DateTimeInterval.of(
                        LocalDateTime.of(each.getFrom().toLocalDate(), from(each)),
                        LocalDateTime.of(each.getTo().toLocalDate(), to(each))
                ))
                .collect(Collectors.toList());
    }

    private LocalTime from(DateTimeInterval interval) {
        return interval.getFrom().toLocalTime().isBefore(from)
                ? from
                : interval.getFrom().toLocalTime();
    }

    private LocalTime to(DateTimeInterval interval) {
        return interval.getTo().toLocalTime().isAfter(to)
                ? to
                : interval.getTo().toLocalTime();
    }
}
```

**사용:**

```java
FeeRule rule1 = new FeeRule(
        new TimeOfDayFeeCondition(LocalTime.of(0, 0), LocalTime.of(19, 0)),
        new FeePerDuration(Money.wons(18), Duration.ofSeconds(10))
);

FeeRule rule2 = new FeeRule(
        new TimeOfDayFeeCondition(LocalTime.of(19, 0), LocalTime.of(24, 0)),
        new FeePerDuration(Money.wons(15), Duration.ofSeconds(10))
);

BasicRatePolicy policy = new BasicRatePolicy(rule1, rule2);
```

#### 📅 요일별 정책

```java
public class DayOfWeekFeeCondition implements FeeCondition {
    private List<DayOfWeek> dayOfWeeks = new ArrayList<>();

    public DayOfWeekFeeCondition(DayOfWeek... dayOfWeeks) {
        this.dayOfWeeks = Arrays.asList(dayOfWeeks);
    }

    @Override
    public List<DateTimeInterval> findTimeIntervals(Call call) {
        return call.getInterval()
                .splitByDay()
                .stream()
                .filter(each ->
                        dayOfWeeks.contains(each.getFrom().getDayOfWeek()))
                .collect(Collectors.toList());
    }
}
```

**사용:**

```java
FeeRule weekdayRule = new FeeRule(
        new DayOfWeekFeeCondition(
                DayOfWeek.MONDAY, DayOfWeek.TUESDAY, DayOfWeek.WEDNESDAY,
                DayOfWeek.THURSDAY, DayOfWeek.FRIDAY
        ),
        new FeePerDuration(Money.wons(38), Duration.ofSeconds(10))
);

FeeRule weekendRule = new FeeRule(
        new DayOfWeekFeeCondition(DayOfWeek.SATURDAY, DayOfWeek.SUNDAY),
        new FeePerDuration(Money.wons(19), Duration.ofSeconds(10))
);

BasicRatePolicy policy = new BasicRatePolicy(weekdayRule, weekendRule);
```

#### ⏱️ 구간별 정책

```java
public class DurationFeeCondition implements FeeCondition {
    private Duration from;
    private Duration to;

    public DurationFeeCondition(Duration from, Duration to) {
        this.from = from;
        this.to = to;
    }

    @Override
    public List<DateTimeInterval> findTimeIntervals(Call call) {
        if (call.getInterval().duration().compareTo(from) < 0) {
            return Collections.emptyList();
        }

        return Arrays.asList(DateTimeInterval.of(
                call.getInterval().getFrom().plus(from),
                call.getInterval().duration().compareTo(to) > 0
                        ? call.getInterval().getFrom().plus(to)
                        : call.getInterval().getTo()
        ));
    }
}
```

**사용:**

```java
FeeRule initialRule = new FeeRule(
        new DurationFeeCondition(Duration.ZERO, Duration.ofMinutes(1)),
        new FeePerDuration(Money.wons(50), Duration.ofSeconds(10))
);

FeeRule afterRule = new FeeRule(
        new DurationFeeCondition(Duration.ofMinutes(1), Duration.ofHours(10)),
        new FeePerDuration(Money.wons(20), Duration.ofSeconds(10))
);

BasicRatePolicy policy = new BasicRatePolicy(initialRule, afterRule);
```

#### 💡 핵심: 일관성

```
이전 구현과의 차이:

step01 (비일관적):
- FixedFeePolicy: 단순 변수
- TimeOfDayDiscountPolicy: 여러 List
- DayOfWeekDiscountPolicy: Rule 객체
- DurationDiscountPolicy: Rule 객체 + 상속

step02 (일관적):
- 모두 FeeCondition 구현
- 모두 BasicRatePolicy + FeeRule 사용
- 동일한 협력 패턴

→ 새로운 정책 추가가 쉬워짐!
→ 코드 이해가 쉬워짐!
```

### 3.6 협력 패턴에 맞추기

#### 🎯 고정요금 방식의 문제

```
고정요금 방식:
- "규칙"이라는 개념 불필요
- 단위요금 정보만 있으면 충분

하지만:
전체 협력 패턴은 "규칙 = 적용조건 + 단위요금"

어떻게 해결?
```

#### ✅ 해결책: FixedFeeCondition

```java
public class FixedFeeCondition implements FeeCondition {
    @Override
    public List<DateTimeInterval> findTimeIntervals(Call call) {
        // 전체 통화 시간을 그대로 반환
        return Arrays.asList(call.getInterval());
    }
}
```

**사용:**

```java
FeeRule rule = new FeeRule(
        new FixedFeeCondition(),  // 항상 만족하는 조건
        new FeePerDuration(Money.wons(18), Duration.ofSeconds(10))
);

BasicRatePolicy policy = new BasicRatePolicy(rule);
```

**의미:**

```
"적용조건이 항상 만족됨"을 표현

→ 전체 통화 시간이 조건을 만족하는 구간

협력 패턴을 깨지 않고
고정요금 방식을 표현!
```

#### 💡 패턴에 맞추기의 중요성

```
고정요금 방식만 다르게 구현했다면?

새로운 개발자가 코드를 볼 때:
"왜 고정요금만 다른 방식이지?"
"어떤 방식이 더 좋은 거지?"
"새로운 정책은 어떤 방식으로 구현하지?"

→ 혼란!

일관성 있게 구현했다면:

"아, 모든 정책이 FeeCondition을 구현하는구나"
"새로운 정책도 FeeCondition을 구현하면 되겠네"

→ 명확!
```

### 3.7 지속적으로 개선하라

#### ⚠️ 주의사항

```
일관성 있는 협력 패턴을 찾았다고 해서
끝이 아니다!

처음부터 완벽한 협력 패턴을 찾기는 어렵다.

새로운 요구사항이 추가되면서
현재 협력 패턴의 한계가 드러날 수 있다.
```

#### 🔄 개선 프로세스

```
1. 새로운 요구사항 발생
   ↓
2. 현재 협력 패턴으로 수용 가능한지 평가
   ↓
3-a. 가능 → 기존 패턴 유지
   ↓
3-b. 불가능 → 리팩터링 고려
   ↓
4. 새로운 협력 패턴 탐색
   ↓
5. 점진적 리팩터링
   ↓
6. 반복
```

#### 💡 핵심 원칙

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  협력은 고정된 것이 아니다!                               │
│                                                     │
│  현재 협력 패턴이 변경의 무게를 지탱하기 어렵다면               │
│  변경을 수용할 수 있는 협력 패턴을 향해                      │
│  과감하게 리팩터링하라                                    │
│                                                     │
│  중요한 것:                                           │
│  현재 설계에 맹목적으로 일관성을 맞추는 것이 아니라             │
│  변경의 방향에 맞춰 지속적으로 개선하려는 의지                 │
│                                                     │
└─────────────────────────────────────────────────────┘
```

### 3.8 패턴을 찾아라

#### 🎯 일관성의 핵심

```
일관성 있는 협력의 핵심:
→ 변경을 분리하고 캡슐화하는 것

협력을 일관성 있게 만드는 과정:
→ 유사한 기능 구현을 위해
  반복적으로 적용할 수 있는
  협력의 구조를 찾아가는 여정

결론:
협력을 일관성 있게 만든다 
= 유사한 변경을 수용할 수 있는 협력 패턴을 발견한다
```

#### 🎨 협력 패턴의 진화

```
처음:
각 정책마다 다른 방식으로 구현
→ 비일관적

변경 분석:
공통점과 차이점 파악
→ 변하는 것과 변하지 않는 것 분리

추상화:
변하는 부분을 인터페이스로 추상화
→ FeeCondition

협력 패턴:
추상화를 중심으로 한 협력 구조
→ BasicRatePolicy - FeeRule - FeeCondition

확장:
새로운 정책 추가가 쉬워짐
→ FeeCondition 구현만 추가
```

#### 📚 관련 개념

**패턴 (Pattern):**
```
반복적으로 발생하는 문제와
그 해결책의 쌍

특징:
- 검증된 해결책
- 재사용 가능
- 문서화된 경험
- 공통 어휘 제공
```

**프레임워크 (Framework):**
```
애플리케이션의 아키텍처를
구현 코드의 형태로 제공

특징:
- 실행 가능한 코드
- 제어의 역전 (IoC)
- 확장 포인트 제공
- 협력 패턴 강제
```

#### 💡 Kent Beck의 인용

```
"객체지향 설계는
객체의 행동과 그것을 지원하기 위한 구조를
계속 수정해 나가는 작업을 반복해 나가면서
다듬어진다.

협력자들 간에 부하를 좀 더 균형 있게 배분하는
방법을 새로 만들어내면 나눠줄 책임이 바뀌게 된다.

만약 객체들이 서로 통신하는 방법을 개선해냈다면
이들 간의 상호작용은 재정의돼야 한다.

이 같은 과정을 거치면서
객체들이 자주 통신하는 경로는 더욱 효율적이게 되고,
주어진 작업을 수행하는 표준 방안이 정착된다.

협력 패턴이 드러나는 것이다!"
```

---

## 4. 핵심 정리

### 🎯 일관성 있는 협력의 가치

```
┌───────────────────────────────────────────────┐
│                                               │
│  일관성 있는 협력의 장점:                           │
│                                               │
│  1. 이해하기 쉬움                                 │
│     - 유사한 기능 = 유사한 구조                     │
│     - 한 번 이해하면 다른 것도 쉽게 이해              │
│                                               │
│  2. 수정하기 쉬움                                │
│     - 변경 포인트가 명확                          │
│     - 영향 범위 예측 가능                         │
│                                               │
│  3. 확장하기 쉬움                                │
│     - 새로운 기능 추가 방법 명확                    │
│     - 기존 코드 수정 최소화                        │
│                                               │
│  4. 재사용하기 쉬움                               │
│     - 검증된 협력 패턴 재사용                       │
│     - 설계 시간 단축                              │
│                                                │
└────────────────────────────────────────────────┘
```

### 📏 일관성을 위한 지침

#### 1. 변하는 것과 변하지 않는 것 분리

```
애플리케이션에서 달라지는 부분을 찾아내고,
달라지지 않는 부분으로부터 분리시킨다.

= 변경의 캡슐화
```

#### 2. 변하는 개념을 캡슐화

```
바뀌는 부분을 따로 뽑아서 캡슐화한다.

그렇게 하면 나중에
바뀌지 않는 부분에는 영향을 미치지 않고
그 부분만 고치거나 확장할 수 있다.
```

#### 3. 조건 로직을 객체 탐색으로 전환

```
if-else, switch 문
→ 다형성 활용

타입 판단 코드 제거
→ 메시지 전송
```

#### 4. 서브타입 캡슐화 + 객체 캡슐화

```
인터페이스 상속 + 합성

변하는 부분: 타입 계층으로 분리
변하지 않는 부분: 추상화에 의존
```

### 🔑 핵심 개념

#### 캡슐화의 진정한 의미

```
캡슐화 ≠ 데이터 은닉

캡슐화 = 변하는 어떤 것이든 감추는 것

4가지 캡슐화:
1. 데이터 캡슐화
2. 메서드 캡슐화
3. 객체 캡슐화 (합성)
4. 서브타입 캡슐화 (다형성)
```

#### 협력 패턴

```
유사한 기능을 구현하기 위해
반복적으로 적용할 수 있는
협력의 구조

= 변경을 수용할 수 있는 설계 템플릿
```

#### 개념적 무결성 (Conceptual Integrity)

```
시스템이 일관성 있는
몇 개의 협력 패턴으로 구성될 때
얻어지는 품질

→ 이해, 수정, 확장 용이
```

### 📊 step01 vs step02 비교

| 측면 | step01 (비일관적) | step02 (일관적) |
|------|------------------|----------------|
| **구조** | 각 정책마다 다름 | 모두 동일한 협력 패턴 |
| **추가 난이도** | 높음 (어떤 방식?) | 낮음 (FeeCondition 구현) |
| **이해 난이도** | 높음 (각각 학습) | 낮음 (한 번 이해) |
| **변경 영향** | 예측 어려움 | 명확함 |
| **재사용성** | 낮음 | 높음 |

### 💡 핵심 교훈

```
1. 유사한 기능은 유사한 방식으로 구현하라
   - 일관성 = 이해도 향상
   
2. 변경을 분리하고 캡슐화하라
   - 변하는 것과 변하지 않는 것 분리
   - 추상화로 캡슐화
   
3. 조건 로직을 객체 탐색으로 바꿔라
   - 다형성 활용
   - 타입 판단 제거
   
4. 협력 패턴을 발견하라
   - 반복되는 구조 찾기
   - 재사용 가능한 템플릿 만들기
   
5. 지속적으로 개선하라
   - 완벽한 패턴은 처음부터 나오지 않음
   - 요구사항 변화에 맞춰 진화
```

### 🎓 설계 원칙과의 연결

```
일관성 있는 협력은
여러 설계 원칙의 조화:

SRP (단일 책임):
- 변경 이유별로 클래스 분리
- FeeCondition, FeePerDuration 분리

OCP (개방-폐쇄):
- 새로운 FeeCondition 추가
- 기존 코드 수정 불필요

LSP (리스코프 치환):
- 모든 FeeCondition 구현체
- 동일하게 사용 가능

ISP (인터페이스 분리):
- FeeCondition 인터페이스
- 필요한 메서드만 정의

DIP (의존성 역전):
- BasicRatePolicy
- 추상화(FeeCondition)에 의존
```

---

## 🔗 연결고리

### 이전 장과의 연결
- **Chapter 11**: 합성으로 조합 폭발 해결 → 일관성 있는 협력으로 확장
- **Chapter 13**: 올바른 상속 (서브타이핑) → 일관성 있는 타입 계층
- 변경의 캡슐화 → 협력 패턴

### 다음 장 예고
- **Chapter 15**: 디자인 패턴과 프레임워크
    - 협력 패턴의 구체화
    - 재사용 가능한 설계
    - 프레임워크의 역할

---

<div align="center">

**[⬆ 목차로 돌아가기](../README.md)**

</div>

<div align="center">

**[⬅️ 이전: Chapter 13](../chapter13/README.md) | [다음: Chapter 15 ➡️](../chapter15/README.md)**

</div>
