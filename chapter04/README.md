# Chapter 04. 설계 품질과 트레이드오프

> *"캡슐화는 설계의 제1원리다"*

## 📌 핵심 개념

- **데이터 중심 설계**: 객체의 상태(데이터)를 중심으로 시스템을 분할하는 방법
- **책임 중심 설계**: 객체의 행동(책임)을 중심으로 시스템을 분할하는 방법
- **캡슐화(Encapsulation)**: 변경될 수 있는 어떤 것이라도 감추는 것
- **응집도(Cohesion)**: 모듈 내 요소들이 하나의 목적을 위해 긴밀하게 협력하는 정도
- **결합도(Coupling)**: 한 모듈이 변경되기 위해 다른 모듈의 변경을 요구하는 정도
- **추측에 의한 설계**: 협력을 고려하지 않고 객체를 설계하는 방식

---

## 🎯 학습 목표

1. 데이터 중심 설계와 책임 중심 설계의 근본적인 차이 이해하기
2. 캡슐화가 단순히 "private + getter/setter"가 아님을 깨닫기
3. 변경의 관점에서 응집도와 결합도 측정하기
4. 같은 기능도 설계 관점에 따라 품질이 달라짐을 체감하기
5. 설계는 트레이드오프의 산물임을 이해하기

---

## 🎬 같은 요구사항, 다른 접근

이번 챕터는 특별합니다. **나쁜 설계를 먼저 보여줍니다.**

### 왜 나쁜 설계를 먼저 보는가?

좋은 설계만 보는 것보다 나쁜 설계를 경험하는 것이 더 큰 통찰을 줍니다.
- "왜 안 되는지"를 체감하면 "왜 되는지"가 명확해집니다
- 실무에서 흔히 만나는 코드 패턴을 다룹니다
- 개선 과정을 통해 좋은 설계의 가치를 이해합니다

### 같은 요구사항: 영화 예매 시스템

Chapter 02와 동일한 요구사항을 다른 방식으로 구현합니다:

| 요구사항 | Chapter 02 (책임 중심) | Chapter 04 (데이터 중심) |
|---------|----------------------|------------------------|
| **할인 정책** | DiscountPolicy 추상화 | MovieType enum |
| **할인 조건** | DiscountCondition 다형성 | DiscountConditionType enum |
| **설계 철학** | 추상화와 다형성 | 타입 체크와 분기문 |

**기능은 동일하지만 설계 품질이 완전히 다릅니다.**

---

## 📐 설계 품질을 측정하는 세 가지 척도

설계를 평가하기 위한 기준을 먼저 이해해야 합니다.

### 1️⃣ 캡슐화 (Encapsulation)

> **"변경될 수 있는 어떤 것이라도 캡슐화해야 한다"**

#### 캡슐화의 힘

객체지향이 강력한 이유는 한 곳에서 일어난 변경이 전체 시스템에 영향을 끼치지 않도록 **파급효과를 조절**할 수 있기 때문입니다.

```
변경 가능성이 높은 부분 = 구현
변경 가능성이 낮은 부분 = 인터페이스

캡슐화 = 구현을 인터페이스 뒤로 숨기는 것
       = 변경의 파급효과를 통제하는 것
```

#### ❌ 잘못된 캡슐화

많은 사람이 착각하는 캡슐화:

```java
public class Movie {
    private Money fee;  // private이니까 캡슐화됐다?
    
    public Money getFee() {
        return fee;
    }
    
    public void setFee(Money fee) {
        this.fee = fee;
    }
}
```

**문제점**:
```java
// getFee()와 setFee(Money fee)가 내부 구현을 노출
// 1. Movie가 Money 타입의 fee를 가지고 있음을 외부에 알림
// 2. fee의 타입이 변경되면 모든 getFee() 사용처 수정 필요

// 사용하는 쪽
Money fee = movie.getFee();  // Money 타입에 의존
movie.setFee(Money.wons(10000));  // Money 타입에 의존

// fee를 BigDecimal로 변경한다면?
// → getFee(), setFee() 시그니처 변경
// → 모든 클라이언트 코드 수정 필요
```

**결론**: **접근자/수정자 ≈ public 속성**

#### ✅ 진정한 캡슐화

```java
public class Rectangle {
    private int left;
    private int top;
    private int right;
    private int bottom;
    
    // ❌ 나쁜 방식 - 내부 구조 노출
    public int getRight() {
        return right;
    }
    
    public void setRight(int right) {
        this.right = right;
    }
    
    public int getBottom() {
        return bottom;
    }
    
    public void setBottom(int bottom) {
        this.bottom = bottom;
    }
}

// 사용하는 쪽
public class Client {
    public void enlargeRectangle(Rectangle rectangle, int multiple) {
        // 클라이언트가 Rectangle의 내부 구현을 알아야 함
        rectangle.setRight(rectangle.getRight() * multiple);
        rectangle.setBottom(rectangle.getBottom() * multiple);
    }
}
```

**문제점**:
1. **코드 중복**: 다른 곳에서도 확대가 필요하면 같은 코드 반복
2. **변경에 취약**: right, bottom을 length, height로 바꾸면 모든 클라이언트 수정
3. **내부 노출**: Rectangle이 right, bottom으로 관리된다는 사실이 외부에 드러남

**개선**:

```java
public class Rectangle {
    private int left;
    private int top;
    private int right;
    private int bottom;
    
    // ✅ 좋은 방식 - 의도를 드러내고 구현을 숨김
    public void enlarge(int multiple) {
        right *= multiple;
        bottom *= multiple;
    }
}

// 사용하는 쪽
public class Client {
    public void enlargeRectangle(Rectangle rectangle, int multiple) {
        rectangle.enlarge(multiple);  // 간단!
    }
}
```

**개선 효과**:
1. **코드 중복 제거**: Rectangle이 enlarge 책임을 가짐
2. **변경에 강함**: 내부를 length, height로 바꿔도 enlarge() 인터페이스 유지 가능
3. **의도 명확**: "사각형을 확대한다"는 의도가 명확히 드러남

#### 캡슐화해야 할 것들

```
✅ 속성의 타입 (Money, String, int...)
✅ 내부 구현 방식 (어떤 알고리즘을 쓰는지)
✅ 내부 자료 구조 (List, Map, Array...)
✅ 객체의 종류 (MovieType, DiscountConditionType...)
✅ 협력 메커니즘 (어떤 객체와 협력하는지)
✅ 할인 정책의 종류 (금액, 비율, 없음...)
✅ 할인 조건의 판단 방식 (순번, 기간...)
```

---

### 2️⃣ 응집도 (Cohesion)

> **"모듈 내 요소들이 하나의 목적을 위해 긴밀하게 협력하는 정도"**

#### 전통적 정의의 문제

```
"모듈 내 요소들이 얼마나 강하게 연관되어 있는가?"

문제: "얼마나"가 얼마나야 높은 거지? 👈 애매함
```

#### 변경의 관점에서 측정

**높은 응집도**:
```
하나의 변경이 발생하면 → 하나의 모듈만 수정
변경의 대상과 범위가 → 명확
코드 변경이 → 쉬움
```

**낮은 응집도**:
```
하나의 변경이 발생하면 → 여러 모듈을 동시에 수정
변경과 무관한 코드도 → 영향받음
코드 변경이 → 어려움
```

#### 구체적 예시: 새로운 할인 정책 추가

**낮은 응집도 (데이터 중심 설계)**:

```java
// 새로운 "조조 할인" 정책을 추가한다면?

// 1️⃣ MovieType enum 수정
public enum MovieType {
    AMOUNT_DISCOUNT,
    PERCENT_DISCOUNT,
    NONE_DISCOUNT,
    EARLY_BIRD_DISCOUNT  // ← 추가
}

// 2️⃣ Movie 클래스 수정
public class Movie {
    private MovieType movieType;
    private Money discountAmount;
    private double discountPercent;
    private LocalTime earlyBirdDeadline;  // ← 추가
    
    public LocalTime getEarlyBirdDeadline() { ... }  // ← 추가
    public void setEarlyBirdDeadline(...) { ... }    // ← 추가
}

// 3️⃣ ReservationAgency 클래스 수정
public class ReservationAgency {
    public Reservation reserve(...) {
        switch(movie.getMovieType()) {
            case AMOUNT_DISCOUNT: ...
            case PERCENT_DISCOUNT: ...
            case NONE_DISCOUNT: ...
            case EARLY_BIRD_DISCOUNT:  // ← 추가
                if (screening.getWhenScreened().toLocalTime()
                             .isBefore(movie.getEarlyBirdDeadline())) {
                    discountAmount = Money.wons(1000);
                }
                break;
        }
    }
}
```

**문제**: 할인 정책 하나 추가하는데 **3개의 클래스**를 수정해야 함!

**높은 응집도 (책임 중심 설계)**:

```java
// 새로운 "조조 할인" 정책을 추가한다면?

// 새 클래스 하나만 추가!
public class EarlyBirdDiscountPolicy extends DiscountPolicy {
    private LocalTime deadline;
    
    public EarlyBirdDiscountPolicy(LocalTime deadline, 
                                   DiscountCondition... conditions) {
        super(conditions);
        this.deadline = deadline;
    }
    
    @Override
    protected Money getDiscountAmount(Screening screening) {
        if (screening.getStartTime().isBefore(deadline)) {
            return Money.wons(1000);
        }
        return Money.ZERO;
    }
}

// 기존 코드는 수정 불필요!
Movie movie = new Movie(
    "아바타",
    Duration.ofMinutes(120),
    Money.wons(10000),
    new EarlyBirdDiscountPolicy(LocalTime.of(10, 0), ...)  // 새 정책 사용
);
```

**개선**: 기존 코드 수정 없이 새 클래스만 추가!

#### 단일 책임 원칙 (SRP)

로버트 마틴의 원칙:
> **"클래스는 단 한 가지의 변경 이유만 가져야 한다"**

```
SRP의 "책임" = "변경의 이유"

예:
- Movie 클래스가 변경되는 이유:
  1. 영화 정보가 변경될 때
  2. 할인 정책이 변경될 때  ← 문제!
  
→ 두 가지 이유가 있다면 응집도가 낮은 것
```

⚠️ **주의**: SRP의 "책임"은 역할-책임-협력의 "책임"과는 다른 개념입니다.

---

### 3️⃣ 결합도 (Coupling)

> **"한 모듈이 변경되기 위해 다른 모듈의 변경을 요구하는 정도"**

#### 전통적 정의의 문제

```
"다른 모듈에 대해 얼마나 많은 지식을 갖고 있는가?"

문제: "얼마나"가 얼마나면 높은 거지? 👈 애매함
```

#### 변경의 관점에서 측정

**낮은 결합도** (좋음):
```
A의 인터페이스 변경 → B 수정 필요
A의 내부 구현 변경 → B 수정 불필요 ✅

= 인터페이스에만 의존
```

**높은 결합도** (나쁨):
```
A의 인터페이스 변경 → B 수정 필요
A의 내부 구현 변경 → B도 수정 필요 ❌

= 구현에 의존
```

#### 데이터 중심 설계의 결합도 문제

```java
public class ReservationAgency {
    public Reservation reserve(Screening screening, Customer customer,
                               int audienceCount) {
        Movie movie = screening.getMovie();
        
        // Movie의 내부 구현에 강하게 결합
        Money fee = movie.getFee()  // Money 타입 노출
                         .minus(discountAmount)
                         .times(audienceCount);
        
        return new Reservation(customer, screening, fee, audienceCount);
    }
}
```

**문제점**:
```
ReservationAgency가 알아야 하는 것들:
- movie.getFee() → Money 타입을 사용한다는 사실
- movie.getMovieType() → MovieType enum을 사용한다는 사실
- movie.getDiscountAmount() → 할인 금액을 직접 저장한다는 사실
- movie.getDiscountPercent() → 할인 비율을 직접 저장한다는 사실
- condition.getType() → DiscountConditionType을 사용한다는 사실
- ...

Movie의 내부 구현 변경 → ReservationAgency도 변경 필요
```

**의존성 집결지**:
```
[Movie] ───────────┐
[DiscountCondition] ──┼──→ [ReservationAgency]
[Screening] ───────┘       (의존성의 집결지)

결과: 어떤 객체를 변경해도 ReservationAgency가 영향받음
     = 시스템이 하나의 거대한 의존성 덩어리
```

#### 결합도가 높아도 괜찮은 경우

```java
// ✅ 안정적인 모듈에 의존하는 것은 OK
String name = "홍길동";  // String은 변경될 일이 거의 없음
List<Movie> movies = new ArrayList<>();  // ArrayList도 안정적
LocalDateTime now = LocalDateTime.now();  // 표준 라이브러리

// ❌ 직접 작성한 코드는 항상 불안정
Movie movie = new Movie(...);  // 우리가 만든 코드 = 변경 가능성 높음
```

**원칙**:
- 표준 라이브러리, 성숙한 프레임워크 → 결합도 걱정 없음
- 직접 작성한 코드 → 낮은 결합도 유지 필수

#### 캡슐화와 응집도/결합도의 관계

```
                캡슐화 강화
                    ↓
        ┌───────────┴───────────┐
        ↓                       ↓
   응집도 ↑                  결합도 ↓
  (모듈 내부)              (모듈 사이)
```

**핵심**: 응집도와 결합도를 고려하기 전에 먼저 **캡슐화**를 향상시켜라!

---

## 💻 Step 01: 데이터 중심 설계

> 📂 **전체 코드**: [step01 디렉토리](https://github.com/eternity-oop/object/tree/master/chapter04/src/main/java/org/eternity/movie/step01)

데이터 중심 설계의 전형적인 패턴을 보여줍니다.

### 설계 접근법

```
질문: "이 객체가 어떤 데이터를 포함해야 하는가?"

순서:
1. 필요한 데이터가 무엇인지 결정
2. 데이터를 보관할 클래스 만들기
3. 데이터를 캡슐화하기 위해 getter/setter 추가
4. 나중에 데이터를 사용하는 로직 작성

결과: 데이터와 프로세스의 분리 = 절차지향
```

---

### 📦 MovieType & DiscountConditionType - 타입 열거형

> 📂 **코드**: [`MovieType.java`](https://github.com/eternity-oop/object/blob/master/chapter04/src/main/java/org/eternity/movie/step01/MovieType.java), [`DiscountConditionType.java`](https://github.com/eternity-oop/object/blob/master/chapter04/src/main/java/org/eternity/movie/step01/DiscountConditionType.java)

```java
public enum MovieType {
    AMOUNT_DISCOUNT,    // 금액 할인 정책
    PERCENT_DISCOUNT,   // 비율 할인 정책
    NONE_DISCOUNT       // 미적용
}

public enum DiscountConditionType {
    SEQUENCE,       // 순번 조건
    PERIOD          // 기간 조건
}
```

**역할**:
- 객체의 종류를 구분하는 타입
- switch문이나 if-else로 타입별 처리

**문제점**:
- 새로운 타입 추가 시 모든 switch문 수정 필요
- 객체의 종류를 외부에 노출

---

### 📦 Movie - 영화 정보

> 📂 **코드**: [`Movie.java`](https://github.com/eternity-oop/object/blob/master/chapter04/src/main/java/org/eternity/movie/step01/Movie.java)

#### 전체 코드

```java
package org.eternity.movie.step01;

import org.eternity.money.Money;
import java.time.Duration;
import java.util.Arrays;
import java.util.Collections;
import java.util.List;

public class Movie {
    private String title;
    private Duration runningTime;
    private Money fee;
    private List<DiscountCondition> discountConditions;
    
    // ⚠️ 데이터 중심 설계의 특징적 패턴
    private MovieType movieType;           // 객체 종류를 저장하는 필드
    private Money discountAmount;          // 금액 할인용 (배타적 사용)
    private double discountPercent;        // 비율 할인용 (배타적 사용)
    
    // 생성자 1: 비율 할인 영화
    public Movie(String title, Duration runningTime, Money fee, 
                 double discountPercent, DiscountCondition... discountConditions) {
        this(MovieType.PERCENT_DISCOUNT, title, runningTime, fee, 
             Money.ZERO, discountPercent, discountConditions);
    }
    
    // 생성자 2: 금액 할인 영화
    public Movie(String title, Duration runningTime, Money fee, 
                 Money discountAmount, DiscountCondition... discountConditions) {
        this(MovieType.AMOUNT_DISCOUNT, title, runningTime, fee, 
             discountAmount, 0, discountConditions);
    }
    
    // 생성자 3: 할인 없는 영화
    public Movie(String title, Duration runningTime, Money fee) {
        this(MovieType.NONE_DISCOUNT, title, runningTime, fee, Money.ZERO, 0);
    }
    
    // 실제 생성자
    private Movie(MovieType movieType, String title, Duration runningTime, 
                  Money fee, Money discountAmount, double discountPercent,
                  DiscountCondition... discountConditions) {
        this.movieType = movieType;
        this.title = title;
        this.runningTime = runningTime;
        this.fee = fee;
        this.discountAmount = discountAmount;
        this.discountPercent = discountPercent;
        this.discountConditions = Arrays.asList(discountConditions);
    }
    
    // ❌ 모든 필드에 대한 getter/setter
    public MovieType getMovieType() {
        return movieType;
    }
    
    public void setMovieType(MovieType movieType) {
        this.movieType = movieType;
    }
    
    public Money getFee() {
        return fee;
    }
    
    public void setFee(Money fee) {
        this.fee = fee;
    }
    
    public List<DiscountCondition> getDiscountConditions() {
        return Collections.unmodifiableList(discountConditions);
    }
    
    public void setDiscountConditions(List<DiscountCondition> discountConditions) {
        this.discountConditions = discountConditions;
    }
    
    public Money getDiscountAmount() {
        return discountAmount;
    }
    
    public void setDiscountAmount(Money discountAmount) {
        this.discountAmount = discountAmount;
    }
    
    public double getDiscountPercent() {
        return discountPercent;
    }
    
    public void setDiscountPercent(double discountPercent) {
        this.discountPercent = discountPercent;
    }
}
```

#### 데이터 중심 설계의 전형적 패턴

**패턴 1: 타입 구분 필드**

```java
private MovieType movieType;  // 객체 종류를 저장
```

- 객체의 종류를 나타내는 enum 필드
- switch문이나 if-else로 타입별 처리 필요
- **문제**: 새로운 타입 추가 시 모든 분기문 수정

**패턴 2: 배타적 사용 필드**

```java
private Money discountAmount;      // AMOUNT_DISCOUNT일 때만 사용
private double discountPercent;    // PERCENT_DISCOUNT일 때만 사용
```

- 타입에 따라 하나만 사용되는 필드들
- 메모리 낭비 + 혼란 야기
- **문제**: 어떤 필드를 써야 하는지 타입을 확인해야 함

**패턴 3: 무분별한 getter/setter**

```java
public MovieType getMovieType() { return movieType; }
public void setMovieType(MovieType movieType) { ... }
public Money getDiscountAmount() { return discountAmount; }
public void setDiscountAmount(Money discountAmount) { ... }
```

- 모든 필드에 대해 접근자/수정자 제공
- **문제**: 내부 구현이 인터페이스에 그대로 노출

#### 왜 이렇게 설계했는가?

```
개발자의 생각 과정:

1단계: "일단 Movie가 가져야 할 데이터가 뭐가 있을까?"
      → 제목, 러닝타임, 가격, 할인조건, 할인정책...

2단계: "할인 정책은 여러 종류가 있으니 enum으로 구분하자"
      → MovieType 추가

3단계: "각 정책마다 필요한 데이터를 모두 넣자"
      → discountAmount, discountPercent 추가

4단계: "나중에 어떻게 쓰일지 모르니 모든 필드에 getter/setter를 만들자"
      → 무분별한 접근자/수정자 추가

결과: 캡슐화 실패! 내부 구현이 모두 노출됨
```

---

### 📦 DiscountCondition - 할인 조건

> 📂 **코드**: [`DiscountCondition.java`](https://github.com/eternity-oop/object/blob/master/chapter04/src/main/java/org/eternity/movie/step01/DiscountCondition.java)

#### 전체 코드

```java
package org.eternity.movie.step01;

import java.time.DayOfWeek;
import java.time.LocalTime;

public class DiscountCondition {
    private DiscountConditionType type;  // 조건 타입
    
    // 순번 조건용
    private int sequence;
    
    // 기간 조건용
    private DayOfWeek dayOfWeek;
    private LocalTime startTime;
    private LocalTime endTime;
    
    // ❌ 모든 필드 노출
    public DiscountConditionType getType() {
        return type;
    }
    
    public void setType(DiscountConditionType type) {
        this.type = type;
    }
    
    public int getSequence() {
        return sequence;
    }
    
    public void setSequence(int sequence) {
        this.sequence = sequence;
    }
    
    public DayOfWeek getDayOfWeek() {
        return dayOfWeek;
    }
    
    public void setDayOfWeek(DayOfWeek dayOfWeek) {
        this.dayOfWeek = dayOfWeek;
    }
    
    public LocalTime getStartTime() {
        return startTime;
    }
    
    public void setStartTime(LocalTime startTime) {
        this.startTime = startTime;
    }
    
    public LocalTime getEndTime() {
        return endTime;
    }
    
    public void setEndTime(LocalTime endTime) {
        this.endTime = endTime;
    }
}
```

#### 문제점 분석

**1. 배타적 사용 필드**

```
type == SEQUENCE일 때:
- sequence만 사용
- dayOfWeek, startTime, endTime은 무의미

type == PERIOD일 때:
- dayOfWeek, startTime, endTime만 사용
- sequence는 무의미
```

**2. 내부 구조 완전 노출**

```java
// getter를 통해 내부 구조가 그대로 노출
condition.getType();        // DiscountConditionType 보유 노출
condition.getSequence();    // int sequence 보유 노출
condition.getDayOfWeek();   // DayOfWeek 보유 노출
condition.getStartTime();   // LocalTime startTime 보유 노출
condition.getEndTime();     // LocalTime endTime 보유 노출
```

**3. 외부에서 마음대로 조작 가능**

```java
// 외부에서 DiscountCondition의 모든 것을 마음대로 조작 가능
DiscountCondition condition = new DiscountCondition();
condition.setType(DiscountConditionType.SEQUENCE);
condition.setSequence(1);

// 나중에 누군가가 타입을 바꿔버릴 수도...
condition.setType(DiscountConditionType.PERIOD);  // sequence는?
```

---

### 📦 Screening - 상영 정보

> 📂 **코드**: [`Screening.java`](https://github.com/eternity-oop/object/blob/master/chapter04/src/main/java/org/eternity/movie/step01/Screening.java)

```java
package org.eternity.movie.step01;

import java.time.LocalDateTime;

public class Screening {
    private Movie movie;
    private int sequence;
    private LocalDateTime whenScreened;
    
    // ❌ 모든 상태 노출
    public Movie getMovie() {
        return movie;
    }
    
    public void setMovie(Movie movie) {
        this.movie = movie;
    }
    
    public LocalDateTime getWhenScreened() {
        return whenScreened;
    }
    
    public void setWhenScreened(LocalDateTime whenScreened) {
        this.whenScreened = whenScreened;
    }
    
    public int getSequence() {
        return sequence;
    }
    
    public void setSequence(int sequence) {
        this.sequence = sequence;
    }
}
```

**역할**:
- 영화 상영 정보 보관
- 어떤 영화를, 몇 번째로, 언제 상영하는지

**문제점**:

```java
// 외부에서 Screening의 모든 것을 마음대로 조작 가능
Screening screening = new Screening();
screening.setMovie(anotherMovie);           // 영화 바꿔치기
screening.setSequence(999);                 // 순번 조작
screening.setWhenScreened(wrongTime);       // 시간 조작

// 심지어 이런 것도 가능...
screening.setMovie(null);  // 영화 없는 상영?!
screening.setSequence(-1); // 음수 순번?!
```

---

### 📦 Reservation & Customer - 예약과 고객

> 📂 **코드**: [`Reservation.java`](https://github.com/eternity-oop/object/blob/master/chapter04/src/main/java/org/eternity/movie/step01/Reservation.java), [`Customer.java`](https://github.com/eternity-oop/object/blob/master/chapter04/src/main/java/org/eternity/movie/step01/Customer.java)

```java
public class Reservation {
    private Customer customer;
    private Screening screening;
    private Money fee;
    private int audienceCount;
    
    public Reservation(Customer customer, Screening screening, 
                       Money fee, int audienceCount) {
        this.customer = customer;
        this.screening = screening;
        this.fee = fee;
        this.audienceCount = audienceCount;
    }
    
    public Customer getCustomer() { return customer; }
    public void setCustomer(Customer customer) { this.customer = customer; }
    
    public Screening getScreening() { return screening; }
    public void setScreening(Screening screening) { this.screening = screening; }
    
    public Money getFee() { return fee; }
    public void setFee(Money fee) { this.fee = fee; }
    
    public int getAudienceCount() { return audienceCount; }
    public void setAudienceCount(int audienceCount) { this.audienceCount = audienceCount; }
}

public class Customer {
    private String name;
    private String id;
    
    public Customer(String name, String id) {
        this.id = id;
        this.name = name;
    }
}
```

---

### 📦 ReservationAgency - 모든 로직의 집결지 ⭐

> 📂 **코드**: [`ReservationAgency.java`](https://github.com/eternity-oop/object/blob/master/chapter04/src/main/java/org/eternity/movie/step01/ReservationAgency.java)

이 클래스가 데이터 중심 설계의 핵심 문제를 보여줍니다.

#### 전체 코드 (45줄의 복잡한 로직)

```java
package org.eternity.movie.step01;

import org.eternity.money.Money;

public class ReservationAgency {
    public Reservation reserve(Screening screening, Customer customer,
                               int audienceCount) {
        Movie movie = screening.getMovie();
        
        // ========== 1단계: 할인 조건 판단 ==========
        boolean discountable = false;
        
        // Movie의 모든 할인 조건을 순회하며 체크
        for(DiscountCondition condition : movie.getDiscountConditions()) {
            
            // 조건 타입별로 분기
            if (condition.getType() == DiscountConditionType.PERIOD) {
                // ⚠️ DiscountCondition 내부 구현에 의존
                // - getDayOfWeek(), getStartTime(), getEndTime() 사용
                // - Screening의 whenScreened 접근
                discountable = 
                    screening.getWhenScreened().getDayOfWeek()
                             .equals(condition.getDayOfWeek()) &&
                    condition.getStartTime()
                             .compareTo(screening.getWhenScreened().toLocalTime()) <= 0 &&
                    condition.getEndTime()
                             .compareTo(screening.getWhenScreened().toLocalTime()) >= 0;
            } else {
                // ⚠️ DiscountCondition의 sequence 필드 접근
                // ⚠️ Screening의 sequence 필드 접근
                discountable = condition.getSequence() == screening.getSequence();
            }
            
            if (discountable) {
                break;  // 하나라도 만족하면 할인 가능
            }
        }
        
        // ========== 2단계: 할인 금액 계산 ==========
        Money fee;
        if (discountable) {
            Money discountAmount = Money.ZERO;
            
            // ⚠️ Movie 타입별로 분기 처리
            switch(movie.getMovieType()) {
                case AMOUNT_DISCOUNT:
                    // ⚠️ Movie의 discountAmount 필드 직접 접근
                    discountAmount = movie.getDiscountAmount();
                    break;
                    
                case PERCENT_DISCOUNT:
                    // ⚠️ Movie의 fee와 discountPercent 필드 직접 접근
                    discountAmount = movie.getFee()
                                          .times(movie.getDiscountPercent());
                    break;
                    
                case NONE_DISCOUNT:
                    discountAmount = Money.ZERO;
                    break;
            }
            
            // 최종 요금 = (영화 가격 - 할인 금액) × 인원수
            fee = movie.getFee().minus(discountAmount).times(audienceCount);
        } else {
            // 할인 불가능하면 정가
            fee = movie.getFee().times(audienceCount);
        }
        
        return new Reservation(customer, screening, fee, audienceCount);
    }
}
```

#### 문제점 총정리

**1. 모든 데이터 객체에 강하게 결합**

```java
// ReservationAgency가 알아야 하는 것들:
movie.getMovieType()                  // Movie의 movieType 필드
movie.getFee()                        // Movie의 fee 필드
movie.getDiscountAmount()             // Movie의 discountAmount 필드
movie.getDiscountPercent()            // Movie의 discountPercent 필드
movie.getDiscountConditions()         // Movie의 discountConditions 필드
condition.getType()                   // DiscountCondition의 type 필드
condition.getSequence()               // DiscountCondition의 sequence 필드
condition.getDayOfWeek()              // DiscountCondition의 dayOfWeek 필드
condition.getStartTime()              // DiscountCondition의 startTime 필드
condition.getEndTime()                // DiscountCondition의 endTime 필드
screening.getMovie()                  // Screening의 movie 필드
screening.getSequence()               // Screening의 sequence 필드
screening.getWhenScreened()           // Screening의 whenScreened 필드
```

**2. 절차지향 프로그래밍**

```
데이터 객체들 (Movie, DiscountCondition, Screening)
       +
프로세스 객체 (ReservationAgency)
       =
데이터와 프로세스의 분리 = 절차지향
```

**3. 변경에 극도로 취약**

```
변경 시나리오별 영향 범위:

1. Movie 필드 변경
   → ReservationAgency 수정 필요

2. DiscountCondition 필드 변경
   → ReservationAgency 수정 필요

3. Screening 필드 변경
   → ReservationAgency 수정 필요

4. 새로운 할인 정책 추가
   → MovieType enum 수정
   → Movie에 필드 추가
   → ReservationAgency의 switch문 수정

5. 새로운 할인 조건 추가
   → DiscountConditionType enum 수정
   → DiscountCondition에 필드 추가
   → ReservationAgency의 if-else 수정
```

---

### 🎯 Step 01 구조적 문제 정리

#### 의존성 다이어그램

```
                        ┌─────────────────────┐
                        │  ReservationAgency  │
                        │   (모든 로직)         │
                        └─────────────────────┘
                                 │
                    ┌────────────┼────────────┬─────────┐
                    ↓            ↓            ↓         ↓
            ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌───────────┐
            │  Movie   │  │ Screening│  │ Customer │  │Reservation│
            │ (데이터)   │  │ (데이터)  │  │  (데이터)  │  │ (데이터)    │
            └──────────┘  └──────────┘  └──────────┘  └───────────┘
                 │
                 ↓
         ┌─────────────────┐
         │DiscountCondition│
         │    (데이터)       │
         └─────────────────┘
```

**특징**:
- ReservationAgency가 모든 객체에 의존
- 나머지 객체들은 데이터만 제공
- 전형적인 **데이터-프로세스 분리** 구조
- **God Object** 패턴 (ReservationAgency가 모든 것을 알고 처리)

#### 책임 분배도

```
ReservationAgency: 100%
├─ 할인 조건 판단 로직
├─ 할인 금액 계산 로직
├─ 예매 금액 계산 로직
└─ 예매 정보 생성 로직

Movie: 0% (데이터만 제공)
DiscountCondition: 0% (데이터만 제공)
Screening: 0% (데이터만 제공)
Customer: 0% (데이터만 제공)
```

---

## 💻 Step 02: 1차 개선 - 자율적 객체로

> 📂 **전체 코드**: [step02 디렉토리](https://github.com/eternity-oop/object/tree/master/chapter04/src/main/java/org/eternity/movie/step02)

### 🎯 개선 전략

**핵심 원칙**:
> **"스스로 자신의 데이터를 책임지는 객체"**

**질문의 변화**:

```
Before (Step 01):
"이 객체가 어떤 데이터를 포함해야 하는가?"

After (Step 02):
"이 객체가 어떤 데이터를 포함해야 하는가?"
"이 객체가 데이터에 대해 수행해야 하는 오퍼레이션은 무엇인가?" ← 추가!
```

**적용 방법**:
```
1. ReservationAgency에서 각 객체와 관련된 로직을 찾는다
2. 해당 로직을 객체 내부로 옮긴다
3. 외부에서는 메시지를 통해서만 요청한다
```

---

### 📦 DiscountCondition - 스스로 판단하기

> 📂 **코드**: [`DiscountCondition.java`](https://github.com/eternity-oop/object/blob/master/chapter04/src/main/java/org/eternity/movie/step02/DiscountCondition.java)

#### 개선된 코드

```java
package org.eternity.movie.step02;

import java.time.DayOfWeek;
import java.time.LocalTime;

public class DiscountCondition {
    private DiscountConditionType type;
    private int sequence;
    private DayOfWeek dayOfWeek;
    private LocalTime startTime;
    private LocalTime endTime;
    
    // ✅ 생성자로 타입 결정 (불변성)
    public DiscountCondition(int sequence) {
        this.type = DiscountConditionType.SEQUENCE;
        this.sequence = sequence;
    }
    
    public DiscountCondition(DayOfWeek dayOfWeek, 
                            LocalTime startTime, LocalTime endTime) {
        this.type = DiscountConditionType.PERIOD;
        this.dayOfWeek = dayOfWeek;
        this.startTime = startTime;
        this.endTime = endTime;
    }
    
    // ⚠️ 여전히 타입 노출
    public DiscountConditionType getType() {
        return type;
    }
    
    // ✅ 핵심: 할인 가능 여부를 스스로 판단
    public boolean isDiscountable(DayOfWeek dayOfWeek, LocalTime time) {
        if (type != DiscountConditionType.PERIOD) {
            throw new IllegalArgumentException();
        }
        
        return this.dayOfWeek.equals(dayOfWeek) &&
               this.startTime.compareTo(time) <= 0 &&
               this.endTime.compareTo(time) >= 0;
    }
    
    public boolean isDiscountable(int sequence) {
        if (type != DiscountConditionType.SEQUENCE) {
            throw new IllegalArgumentException();
        }
        
        return this.sequence == sequence;
    }
}
```

#### 개선 효과

**Before (Step 01)**: ReservationAgency가 판단

```java
// ReservationAgency가 DiscountCondition 내부를 직접 조작
if (condition.getType() == DiscountConditionType.PERIOD) {
    discountable = screening.getWhenScreened().getDayOfWeek()
                            .equals(condition.getDayOfWeek()) &&
                   condition.getStartTime()
                            .compareTo(screening.getWhenScreened().toLocalTime()) <= 0 &&
                   condition.getEndTime()
                            .compareTo(screening.getWhenScreened().toLocalTime()) >= 0;
}
```

**After (Step 02)**: DiscountCondition이 스스로 판단

```java
// DiscountCondition이 스스로 판단 (타입 체크는 Movie가)
if (condition.getType() == DiscountConditionType.PERIOD) {
    if (condition.isDiscountable(whenScreened.getDayOfWeek(), 
                                 whenScreened.toLocalTime())) {
        return true;
    }
}
```

**변경 영향 분석**:

```
시나리오: 기간 조건 판단 로직 변경
예: "평일 10~12시" → "평일 && (10~12시 또는 14~16시)"

Step 01:
- DiscountCondition의 필드 구조 변경 필요
- ReservationAgency의 판단 로직 수정 필요
→ 2개 클래스 수정

Step 02:
- DiscountCondition의 isDiscountable만 수정
- 외부 코드는 수정 불필요
→ 1개 클래스만 수정
```

#### 여전히 남은 문제

```java
// ⚠️ 메서드 시그니처가 내부 구조를 노출
public boolean isDiscountable(DayOfWeek dayOfWeek, LocalTime time) {
    // DayOfWeek, LocalTime을 내부에 가지고 있음을 노출
}

public boolean isDiscountable(int sequence) {
    // int sequence를 내부에 가지고 있음을 노출
}

// ⚠️ 타입을 여전히 외부에 노출
public DiscountConditionType getType() {
    return type;
}
```

---

### 📦 Movie - 스스로 계산하기

> 📂 **코드**: [`Movie.java`](https://github.com/eternity-oop/object/blob/master/chapter04/src/main/java/org/eternity/movie/step02/Movie.java)

#### 개선된 코드

```java
package org.eternity.movie.step02;

import org.eternity.money.Money;
import java.time.Duration;
import java.time.LocalDateTime;
import java.util.Arrays;
import java.util.List;

public class Movie {
    private String title;
    private Duration runningTime;
    private Money fee;
    private List<DiscountCondition> discountConditions;
    
    private MovieType movieType;
    private Money discountAmount;
    private double discountPercent;
    
    // ... 생성자들 (Step 01과 동일)
    
    // ⚠️ 여전히 타입 노출
    public MovieType getMovieType() {
        return movieType;
    }
    
    // ✅ 할인 가능 여부 판단을 스스로 수행
    public boolean isDiscountable(LocalDateTime whenScreened, int sequence) {
        for(DiscountCondition condition : discountConditions) {
            if (condition.getType() == DiscountConditionType.PERIOD) {
                if (condition.isDiscountable(whenScreened.getDayOfWeek(), 
                                            whenScreened.toLocalTime())) {
                    return true;
                }
            } else {
                if (condition.isDiscountable(sequence)) {
                    return true;
                }
            }
        }
        return false;
    }
    
    // ✅ 각 할인 타입별 요금 계산을 스스로 수행
    public Money calculateAmountDiscountedFee() {
        if (movieType != MovieType.AMOUNT_DISCOUNT) {
            throw new IllegalArgumentException();
        }
        return fee.minus(discountAmount);
    }
    
    public Money calculatePercentDiscountedFee() {
        if (movieType != MovieType.PERCENT_DISCOUNT) {
            throw new IllegalArgumentException();
        }
        return fee.minus(fee.times(discountPercent));
    }
    
    public Money calculateNoneDiscountedFee() {
        if (movieType != MovieType.NONE_DISCOUNT) {
            throw new IllegalArgumentException();
        }
        return fee;
    }
}
```

#### 책임의 이동

**Step 01**:
```
ReservationAgency가 Movie의 데이터를 가져와서 계산
- movie.getDiscountAmount()
- movie.getFee().times(movie.getDiscountPercent())
```

**Step 02**:
```
Movie가 자신의 데이터로 스스로 계산
- movie.calculateAmountDiscountedFee()
- movie.calculatePercentDiscountedFee()
```

#### 여전히 남은 문제

```java
// ⚠️ 메서드 이름이 할인 정책의 종류를 노출
public Money calculateAmountDiscountedFee() { ... }
public Money calculatePercentDiscountedFee() { ... }
public Money calculateNoneDiscountedFee() { ... }

// 이 메서드들의 존재 자체가:
// "Movie에는 금액 할인, 비율 할인, 할인 없음 세 가지가 있다"
// 라는 사실을 외부에 알림

// ⚠️ 타입을 여전히 외부에 노출
public MovieType getMovieType() {
    return movieType;
}
```

---

### 📦 Screening - 요금 계산 책임

> 📂 **코드**: [`Screening.java`](https://github.com/eternity-oop/object/blob/master/chapter04/src/main/java/org/eternity/movie/step02/Screening.java)

#### 개선된 코드

```java
package org.eternity.movie.step02;

import org.eternity.money.Money;
import java.time.LocalDateTime;

public class Screening {
    private Movie movie;
    private int sequence;
    private LocalDateTime whenScreened;
    
    public Screening(Movie movie, int sequence, LocalDateTime whenScreened) {
        this.movie = movie;
        this.sequence = sequence;
        this.whenScreened = whenScreened;
    }
    
    // ✅ 예매 요금 계산을 스스로 수행
    public Money calculateFee(int audienceCount) {
        switch (movie.getMovieType()) {
            case AMOUNT_DISCOUNT:
                if (movie.isDiscountable(whenScreened, sequence)) {
                    return movie.calculateAmountDiscountedFee()
                               .times(audienceCount);
                }
                break;
                
            case PERCENT_DISCOUNT:
                if (movie.isDiscountable(whenScreened, sequence)) {
                    return movie.calculatePercentDiscountedFee()
                               .times(audienceCount);
                }
                
            case NONE_DISCOUNT:
                return movie.calculateNoneDiscountedFee()
                           .times(audienceCount);
        }
        
        return movie.calculateNoneDiscountedFee().times(audienceCount);
    }
}
```

#### 협력의 변화

**Step 01**:
```
ReservationAgency
  ├─> Movie의 모든 필드 직접 접근
  ├─> DiscountCondition의 모든 필드 직접 접근
  ├─> Screening의 모든 필드 직접 접근
  └─> 스스로 모든 것 계산
```

**Step 02**:
```
ReservationAgency
  └─> Screening에게 "요금 계산해줘"
        └─> Movie에게 "할인 가능해?", "할인된 요금 계산해줘"
              └─> DiscountCondition에게 "조건 만족해?"
```

---

### 📦 ReservationAgency - 극적으로 단순해짐

> 📂 **코드**: [`ReservationAgency.java`](https://github.com/eternity-oop/object/blob/master/chapter04/src/main/java/or g/eternity/movie/step02/ReservationAgency.java)

#### 개선된 코드

```java
package org.eternity.movie.step02;

import org.eternity.money.Money;

public class ReservationAgency {
    public Reservation reserve(Screening screening, Customer customer, 
                               int audienceCount) {
        Money fee = screening.calculateFee(audienceCount);
        return new Reservation(customer, screening, fee, audienceCount);
    }
}
```

#### 극적인 변화

**Before (Step 01)**: **45줄**의 복잡한 로직
```java
// 할인 조건 판단 (20줄)
boolean discountable = false;
for(DiscountCondition condition : movie.getDiscountConditions()) {
    if (condition.getType() == DiscountConditionType.PERIOD) {
        // 복잡한 판단 로직...
    }
}

// 할인 금액 계산 (15줄)
switch(movie.getMovieType()) {
    case AMOUNT_DISCOUNT:
        // 계산 로직...
}

// 예매 금액 계산 (10줄)
fee = movie.getFee().minus(discountAmount).times(audienceCount);
```

**After (Step 02)**: **3줄**의 단순한 협력
```java
Money fee = screening.calculateFee(audienceCount);
return new Reservation(customer, screening, fee, audienceCount);
```

**변화의 본질**:
```
Step 01: ReservationAgency가 모든 것을 "제어" (Controller)
Step 02: ReservationAgency는 "조율"만 담당 (Coordinator)
```

---

### 🎯 Step 02 개선 효과

#### 의존성 다이어그램 변화

**Step 01**:
```
ReservationAgency ──→ Movie
                  ├──→ DiscountCondition
                  ├──→ Screening
                  └──→ Reservation
                  └──→ Customer

(ReservationAgency가 모든 것에 의존)
```

**Step 02**:
```
ReservationAgency ──→ Screening ──→ Movie ──→ DiscountCondition
                  └──→ Reservation
                  └──→ Customer

(의존성 체인이 자연스러운 협력 구조로)
```

**결합도 개선**:
- ReservationAgency의 직접 의존성: 5개 → 3개
- 전이적 의존성은 있지만 간접적임
- 의존성 체인이 자연스러운 협력 흐름을 따름

#### 책임 분배 변화

**Step 01**:
```
ReservationAgency: 100%  (모든 로직)
Movie: 0%               (데이터만)
DiscountCondition: 0%   (데이터만)
Screening: 0%           (데이터만)
```

**Step 02**:
```
ReservationAgency: 10%   (조율)
Screening: 30%           (요금 계산)
Movie: 40%               (할인 판단 및 계산)
DiscountCondition: 20%   (조건 판단)
```

#### 변경 영향 분석

| 변경 시나리오 | Step 01 | Step 02 |
|-------------|---------|---------|
| **할인 조건 판단 로직 변경** | DiscountCondition + ReservationAgency | DiscountCondition만 |
| **할인 금액 계산 로직 변경** | Movie + ReservationAgency | Movie만 |
| **예매 요금 계산 로직 변경** | ReservationAgency | Screening만 |

**결론**: 변경 영향 범위가 명확히 줄어듦!

---

## 🚨 하지만 Step 02도 여전히 부족하다

> **"데이터를 처리하는 메서드를 데이터를 가진 객체에 넣었다고 끝이 아니다"**

Step 02는 Step 01보다 훨씬 나아졌지만, 여전히 **데이터 중심 설계의 근본적 문제**를 안고 있습니다.

---

### 🔍 문제 1: 캡슐화 위반

#### DiscountCondition의 캡슐화 위반

```java
public class DiscountCondition {
    // ❌ 타입을 외부에 노출
    public DiscountConditionType getType() {
        return type;
    }
    
    // ❌ 파라미터가 내부 구조를 노출
    public boolean isDiscountable(DayOfWeek dayOfWeek, LocalTime time) {
        // "내부에 DayOfWeek, LocalTime 필드가 있다"는 사실을 노출
    }
    
    public boolean isDiscountable(int sequence) {
        // "내부에 int sequence 필드가 있다"는 사실을 노출
    }
}
```

**문제 상황**:
```
1. getType() → DiscountConditionType을 포함하고 있음을 노출
2. isDiscountable(DayOfWeek, LocalTime) → 이 타입들이 있음을 노출
3. isDiscountable(int) → int sequence가 있음을 노출
```

**구체적 변경 시나리오**:

```java
// 현재 구조
private DayOfWeek dayOfWeek;
private LocalTime startTime;
private LocalTime endTime;

public boolean isDiscountable(DayOfWeek dayOfWeek, LocalTime time) { ... }

// 만약 내부를 LocalDateTime으로 변경한다면?
private LocalDateTime startDateTime;
private LocalDateTime endDateTime;

public boolean isDiscountable(LocalDateTime dateTime) { ... }  // 변경!

// 결과:
// 1. Movie의 코드 수정 필요
public boolean isDiscountable(LocalDateTime whenScreened, int sequence) {  // 변경!
    for(DiscountCondition condition : discountConditions) {
        if (condition.getType() == DiscountConditionType.PERIOD) {
            if (condition.isDiscountable(whenScreened)) {  // 변경!
                return true;
            }
        }
    }
}

// 2. Screening의 코드도 수정 필요
public Money calculateFee(int audienceCount) {
    if (movie.isDiscountable(whenScreened, sequence)) {  // 파라미터 변경!
        ...
    }
}
```

→ **내부 구현 변경이 외부 코드까지 영향**

#### Movie의 캡슐화 위반

```java
public class Movie {
    // ❌ 메서드 이름이 할인 정책의 종류를 노출
    public Money calculateAmountDiscountedFee() { ... }
    public Money calculatePercentDiscountedFee() { ... }
    public Money calculateNoneDiscountedFee() { ... }
}
```

**문제 상황**:
```
메서드 이름 자체가 내부 구현을 노출:
- "금액 할인, 비율 할인, 할인 없음" 세 가지 정책이 존재함을 외부에 알림
```

**구체적 변경 시나리오**:

```java
// 새로운 "시즌 할인" 정책 추가

// 1. Movie에 새 메서드 추가
public Money calculateSeasonDiscountedFee() {
    if (movieType != MovieType.SEASON_DISCOUNT) {
        throw new IllegalArgumentException();
    }
    // 시즌 할인 계산...
}

// 2. MovieType에 추가
public enum MovieType {
    AMOUNT_DISCOUNT,
    PERCENT_DISCOUNT,
    NONE_DISCOUNT,
    SEASON_DISCOUNT  // 추가
}

// 3. Screening의 switch문 수정
public Money calculateFee(int audienceCount) {
    switch (movie.getMovieType()) {
        case AMOUNT_DISCOUNT: ...
        case PERCENT_DISCOUNT: ...
        case NONE_DISCOUNT: ...
        case SEASON_DISCOUNT:  // 추가!
            if (movie.isDiscountable(whenScreened, sequence)) {
                return movie.calculateSeasonDiscountedFee()
                           .times(audienceCount);
            }
    }
}
```

→ **새로운 정책 추가 시 여러 곳 수정**

---

### 🔍 문제 2: 높은 결합도

#### Movie와 DiscountCondition의 강한 결합

```java
public class Movie {
    public boolean isDiscountable(LocalDateTime whenScreened, int sequence) {
        for(DiscountCondition condition : discountConditions) {
            // ❌ DiscountCondition의 타입에 의존
            if (condition.getType() == DiscountConditionType.PERIOD) {
                // ❌ DiscountCondition의 메서드 시그니처에 의존
                if (condition.isDiscountable(whenScreened.getDayOfWeek(), 
                                            whenScreened.toLocalTime())) {
                    return true;
                }
            } else {
                if (condition.isDiscountable(sequence)) {
                    return true;
                }
            }
        }
        return false;
    }
}
```

**DiscountCondition 변경 시 Movie도 변경되는 경우들**:

**시나리오 1: 할인 조건 명칭 변경**
```java
// DiscountConditionType 변경
public enum DiscountConditionType {
    SEQUENCE,
    PERIOD,      // → TIME_SLOT으로 변경
}

// Movie도 변경 필요!
if (condition.getType() == DiscountConditionType.TIME_SLOT) {  // 변경!
    // ...
}
```

**시나리오 2: 할인 조건 종류 추가**
```java
// DiscountConditionType에 추가
public enum DiscountConditionType {
    SEQUENCE,
    PERIOD,
    COMBINED    // 순번 + 기간 조합 조건
}

// Movie에 새 분기 추가 필요!
if (condition.getType() == DiscountConditionType.PERIOD) {
    // ...
} else if (condition.getType() == DiscountConditionType.COMBINED) {  // 추가!
    // 복합 조건 처리 로직...
} else {
    // ...
}
```

**시나리오 3: 조건 판단 정보 변경 (앞에서 본 예)**
```java
// DiscountCondition의 isDiscountable 파라미터 변경
public boolean isDiscountable(LocalDateTime dateTime) {
    // DayOfWeek, LocalTime → LocalDateTime으로 통합
}

// Movie의 isDiscountable도 변경 필요!
public boolean isDiscountable(LocalDateTime whenScreened, int sequence) {
    // 파라미터 변경!
}

// Screening도 변경 필요!
public Money calculateFee(int audienceCount) {
    if (movie.isDiscountable(whenScreened, sequence)) {
        // whenScreened가 이미 LocalDateTime이므로 OK
    }
}
```

**연쇄 변경의 악순환**:
```
DiscountCondition 변경
    ↓
Movie.isDiscountable 파라미터 변경
    ↓
Screening.calculateFee 변경
```

---

### 🔍 문제 3: 낮은 응집도

#### Screening의 낮은 응집도

```java
public class Screening {
    public Money calculateFee(int audienceCount) {
        // ❌ Movie의 타입을 직접 체크
        switch (movie.getMovieType()) {
            case AMOUNT_DISCOUNT:
                if (movie.isDiscountable(whenScreened, sequence)) {
                    return movie.calculateAmountDiscountedFee()
                               .times(audienceCount);
                }
                break;
            case PERCENT_DISCOUNT:
                if (movie.isDiscountable(whenScreened, sequence)) {
                    return movie.calculatePercentDiscountedFee()
                               .times(audienceCount);
                }
            case NONE_DISCOUNT:
                return movie.calculateNoneDiscountedFee()
                           .times(audienceCount);
        }
        return movie.calculateNoneDiscountedFee().times(audienceCount);
    }
}
```

**할인 조건 판단 방법 변경 시 수정 대상**:

```
1. DiscountCondition.isDiscountable() 메서드
   예: DayOfWeek, LocalTime → LocalDateTime으로 변경

2. Movie.isDiscountable() 메서드
   예: 파라미터를 LocalDateTime으로 변경

3. Screening.calculateFee() 메서드
   예: movie.isDiscountable() 호출 부분 수정
```

→ **하나의 변경을 위해 3개 모듈 동시 수정 = 낮은 응집도**

#### 새로운 할인 정책 추가 시

```
1. MovieType enum 수정
   SEASON_DISCOUNT 추가

2. Movie 클래스 수정
   - private SeasonInfo seasonInfo 필드 추가
   - public Money calculateSeasonDiscountedFee() 메서드 추가
   - getter/setter 추가

3. Screening 클래스 수정
   - calculateFee()의 switch문에 case 추가

변경 범위: 3개 클래스
```

---

## 💡 캡슐화의 진정한 의미

### 추측에 의한 설계 전략

데이터 중심 설계가 빠지기 쉬운 함정입니다.

```
개발자의 생각:
"나중에 어떻게 쓰일지 모르니까..."
"일단 모든 필드에 getter/setter를 만들어두자!"
"혹시 필요할지도 모르니까 다 public으로!"

문제:
→ 객체가 어떤 문맥에서 사용될지 고려하지 않음
→ 무분별한 접근자 메서드 추가
→ 내부 구현이 인터페이스에 그대로 노출
→ 캡슐화 실패
```

**올바른 접근**:
```
1. 객체가 참여할 협력을 먼저 생각
2. 협력에 필요한 책임 식별
3. 책임을 수행하기 위한 오퍼레이션 정의
4. 오퍼레이션 수행에 필요한 데이터 결정

순서: 협력 → 책임 → 행동 → 데이터
```

---

## 🎯 데이터 중심 설계의 근본 문제

### 1️⃣ 너무 이른 시기에 데이터 결정

#### 문제의 본질

```
데이터 중심 설계: "이 객체가 포함해야 하는 데이터가 무엇인가?"
                              ↓
                    너무 이른 시기에 내부 구현 결정
                              ↓
                    구현이 인터페이스에 노출
                              ↓
                          캡슐화 실패
```

**왜 데이터가 구현인가?**

```
데이터 = 객체가 어떻게 구현되는지를 결정하는 요소
      = 가장 변하기 쉬운 부분
      = 구현의 일부

예:
- Money fee                    → Money 타입 사용 (구현 결정)
- MovieType movieType          → enum으로 타입 구분 (구현 결정)
- List<DiscountCondition>      → List 사용 (구현 결정)
```

**데이터부터 결정하면 왜 문제인가?**

```
1. 불안정한 것부터 결정
   데이터(구현) → 메서드(인터페이스)
   = 불안정한 것이 안정적인 것에 영향

2. 내부 구현이 인터페이스에 스며듦
   public Money getFee()           → Money 타입 노출
   public MovieType getMovieType() → MovieType 노출

3. 변경에 취약
   데이터 변경 → 인터페이스 변경 → 모든 클라이언트 수정
```

---

### 2️⃣ 협력을 고려하지 않고 객체 고립

#### 문제의 본질

```
데이터 중심 설계 순서:
1. 데이터 결정 (고립된 상태에서)
2. 오퍼레이션 결정
3. 협력 관계 고민
4. 억지로 끼워맞추기

문제:
→ 이미 구현이 결정되어 있음
→ 인터페이스를 억지로 맞춤
→ 유연하지 못한 설계
```

**책임 중심 설계 순서**:

```
1. 협력 문맥 파악
   "영화 예매 시스템에서 영화 요금을 어떻게 계산할까?"

2. 필요한 책임 식별
   - 영화는 자신의 요금을 계산할 책임
   - 할인 정책은 할인 금액을 계산할 책임

3. 책임을 수행할 오퍼레이션 정의
   Movie.calculateMovieFee(Screening screening)
   DiscountPolicy.calculateDiscountAmount(Screening screening)

4. 오퍼레이션 수행에 필요한 데이터 결정
   Movie: fee, discountPolicy
   DiscountPolicy: conditions

결과:
→ 협력에 적합한 인터페이스
→ 구현은 인터페이스 뒤로 숨김
→ 유연한 설계
```

---

## 🎓 핵심 개념 총정리

### 절차지향 vs 객체지향

| 구분 | 절차지향 (Step 01) | 객체지향 (Chapter 02) |
|------|-------------------|----------------------|
| **데이터와 프로세스** | 분리 | 동일 객체 내 위치 |
| **책임 소재** | ReservationAgency에 집중 | 각 객체에 분산 |
| **변경 영향 범위** | 여러 곳에 파급 | 해당 객체로 국한 |
| **결합도** | 높음 | 낮음 |
| **응집도** | 낮음 | 높음 |
| **이해하기** | 어려움 | 쉬움 |
| **테스트** | 어려움 | 쉬움 |

### 좋은 설계의 조건

```
좋은 설계 = 오늘의 기능 + 내일의 변경 수용

높은 응집도: 하나의 변경 → 하나의 모듈만 수정
낮은 결합도: 내부 구현 변경 → 다른 모듈 영향 없음
강한 캡슐화: 변경 가능성을 내부에 숨김
```

### 설계의 무게 중심

```
데이터 중심: 내부로 (구현 집착)
           ↓
    외부와의 협력은 나중에
           ↓
     억지로 끼워맞추기

 책임 중심: 외부로 (협력 집중)
           ↓
     내부 구현은 나중에
           ↓
      자연스러운 설계
```

---

## 🔬 자주 하는 질문

### Q1. getter/setter를 절대 쓰면 안 되나요?

**A**: 상황에 따라 다릅니다.

#### 사용해도 되는 경우

**DTO (Data Transfer Object)**:
```java
// 계층 간 데이터 전달이 목적
@Data
public class MovieResponse {
    private String title;
    private Long fee;
    private String posterUrl;
    
    public String getTitle() { return title; }  // OK
    public Long getFee() { return fee; }  // OK
}
```

**VO (Value Object)**:
```java
// 단순 값 표현
public class Money {
    private final BigDecimal amount;
    
    public BigDecimal getAmount() { return amount; }  // OK
    
    // 하지만 이렇게 하는 것이 더 좋음
    public Money plus(Money other) {
        return new Money(amount.add(other.amount));
    }
}
```

#### 사용하면 안 되는 경우

**도메인 객체**:
```java
// ❌ 상태를 외부에 노출
public class Order {
    private OrderStatus status;
    
    public OrderStatus getStatus() { return status; }
    public void setStatus(OrderStatus status) { this.status = status; }
}

// 외부에서 상태 조작
order.setStatus(OrderStatus.CANCELLED);  // 검증 로직 없음!

// ✅ 의도를 드러내는 메서드
public class Order {
    private OrderStatus status;
    
    public void cancel() {
        if (status != OrderStatus.PENDING) {
            throw new IllegalStateException("취소할 수 없는 상태입니다");
        }
        this.status = OrderStatus.CANCELLED;
        // 추가 로직: 환불 처리, 재고 복구 등
    }
}

// 외부에서 의도를 표현
order.cancel();  // 검증 로직 포함!
```

#### 핵심 원칙

```
getter/setter 자체가 문제가 아니라,
객체가 스스로 책임을 다하지 못하게 만드는 것이 문제

판단 기준:
1. 단순 조회만 필요한가? → getter OK
2. 행동이 필요한가? → 메서드로 캡슐화
3. 외부에서 상태를 변경해야 하는가? → 대부분 설계 문제
```

---

### Q2. 데이터 중심은 항상 나쁜가요?

**A**: 문맥에 따라 다릅니다.

#### 데이터 중심이 적합한 경우

**1. 단순한 CRUD**:
```java
// 게시판 시스템의 Post
public class Post {
    private Long id;
    private String title;
    private String content;
    private LocalDateTime createdAt;
    
    // getter, setter
    // 복잡한 로직 없음
}
```

**2. 안정적인 도메인**:
```java
// 설정 정보
public class SystemConfig {
    private String dbUrl;
    private int maxConnections;
    
    // getter, setter
}
```

#### 책임 중심이 필수인 경우

**1. 복잡한 비즈니스 로직**:
```java
public class Order {
    public void cancel() {
        validateCancellable();
        refund();
        restoreStock();
        this.status = OrderStatus.CANCELLED;
    }
}
```

**2. 요구사항 변경이 잦은 도메인**:
```java
// 할인 정책 (자주 변경됨)
public abstract class DiscountPolicy {
    public Money calculateDiscountAmount(Screening screening) {
        // 템플릿 메서드 패턴
    }
}
```

---

### Q3. Step 02를 어떻게 개선하나요?

**A**: Chapter 02의 **추상화와 다형성**이 해답입니다.

```java
// Step 02: 타입 체크 + switch문
switch (movie.getMovieType()) {
    case AMOUNT_DISCOUNT: ...
    case PERCENT_DISCOUNT: ...
}

// Chapter 02: 다형성
public abstract class DiscountPolicy {
    abstract Money getDiscountAmount(Screening screening);
}

public class AmountDiscountPolicy extends DiscountPolicy { ... }
public class PercentDiscountPolicy extends DiscountPolicy { ... }
```

---

## 🚀 실전 적용: Spring Boot 예제

### Before: 데이터 중심

```java
@Service
public class OrderService {
    public OrderResponse create(OrderRequest request) {
        // Service가 모든 로직 처리
        Order order = new Order();
        order.setUserId(request.getUserId());
        order.setProductId(request.getProductId());
        
        Product product = productRepository.findById(request.getProductId());
        if (product.getStock() < request.getQuantity()) {
            throw new OutOfStockException();
        }
        
        Long totalPrice = product.getPrice() * request.getQuantity();
        order.setTotalPrice(totalPrice);
        
        product.setStock(product.getStock() - request.getQuantity());
        
        orderRepository.save(order);
        productRepository.save(product);
        
        return new OrderResponse(order);
    }
}

// Domain은 데이터만
public class Order {
    private Long userId;
    private Long productId;
    private Long totalPrice;
    
    // getter, setter만...
}
```

### After: 책임 중심

```java
@Service
public class OrderService {
    public OrderResponse create(OrderRequest request) {
        // Service는 조율만
        Product product = productRepository.findById(request.getProductId());
        
        Order order = Order.create(
            request.getUserId(),
            product,
            request.getQuantity()
        );
        
        orderRepository.save(order);
        productRepository.save(product);
        
        return new OrderResponse(order);
    }
}

// Domain이 로직 처리
public class Order {
    private Long userId;
    private Long productId;
    private Long totalPrice;
    
    public static Order create(Long userId, Product product, Integer quantity) {
        product.decreaseStock(quantity);
        Long totalPrice = product.calculatePrice(quantity);
        return new Order(userId, product.getId(), quantity, totalPrice);
    }
    
    private Order(Long userId, Long productId, Integer quantity, Long totalPrice) {
        this.userId = userId;
        this.productId = productId;
        this.totalPrice = totalPrice;
    }
}

public class Product {
    public void decreaseStock(Integer quantity) {
        if (this.stock < quantity) {
            throw new OutOfStockException();
        }
        this.stock -= quantity;
    }
    
    public Long calculatePrice(Integer quantity) {
        return this.price * quantity;
    }
}
```

---

## ✨ 핵심 정리

### 설계 원칙

```
1. 데이터가 아닌 책임에 초점
   → "이 객체는 무엇을 할 수 있는가?"

2. 협력 문맥에서 책임 결정
   → "다른 객체와 어떻게 협력하는가?"

3. 구현이 아닌 인터페이스에 의존
   → "어떻게가 아니라 무엇을"

4. 변경될 수 있는 모든 것 캡슐화
   → "변경의 파급효과를 최소화"
```

### 이 챕터의 교훈

```
설계는 트레이드오프의 산물
"완벽한 설계"는 없다
"더 나은 설계"는 있다
중요한 것은 변경에 대응하는 능력

나쁜 설계를 경험하는 것도 배움
"왜 안 되는지" 아는 것이
"왜 되는지" 이해하는 첫걸음
```

---

<div align="center">

**[← Chapter 03](../chapter03/README.md) | [Chapter 05 →](../chapter05/README.md)**

</div>
