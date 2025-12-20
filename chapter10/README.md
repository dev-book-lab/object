# Chapter 10. 상속과 코드 재사용

> *"상속은 코드 재사용을 위해 캡슐화를 희생한다"*

## 📌 핵심 개념

이 장에서는 **상속(Inheritance)의 양면성**을 다룹니다. 상속은 강력한 코드 재사용 메커니즘이지만, 잘못 사용하면 오히려 시스템을 더 복잡하고 취약하게 만듭니다.

### 🎯 학습 목표

- 중복 코드의 진정한 의미와 DRY 원칙 이해하기
- 상속의 문제점: 취약한 기반 클래스 문제 파악하기
- 상속을 안전하게 사용하는 방법: 추상화에 의존하기
- 차이에 의한 프로그래밍의 본질 이해하기
- 다음 장(합성)을 위한 발판 마련하기

---

## 📖 목차

1. [상속과 중복 코드](#1-상속과-중복-코드)
2. [취약한 기반 클래스 문제](#2-취약한-기반-클래스-문제)
3. [Phone 다시 살펴보기](#3-phone-다시-살펴보기)
4. [차이에 의한 프로그래밍](#4-차이에-의한-프로그래밍)
5. [핵심 정리](#5-핵심-정리)

---

## 1. 상속과 중복 코드

> 📂 **코드**: [`Phone.java (step01)`](https://github.com/eternity-oop/object/blob/master/chapter10/src/main/java/org/eternity/billing/step01/Phone.java) | [`NightlyDiscountPhone.java (step01)`](https://github.com/eternity-oop/object/blob/master/chapter10/src/main/java/org/eternity/billing/step01/NightlyDiscountPhone.java)

### 1.1 중복 코드의 본질

#### 💀 중복 코드는 악의 근원

```
중복 코드는 사람들의 마음속에 의심과 불신의 씨앗을 뿌린다.

- 코드를 신뢰할 수 없게 만듦
- 변경을 어렵게 만듦
- 버그를 양산함
```

#### 🎯 중복의 진정한 기준

**많은 개발자들의 오해:**
```
"똑같이 생긴 코드 = 중복 코드"  ❌
```

**올바른 기준:**
```
중복 여부를 판단하는 기준은 변경이다.

요구사항이 변경됐을 때 두 코드를 함께 수정해야 한다면
이 코드는 중복이다.
```

**예시:**

```java
// Case 1: 모양은 같지만 중복이 아닌 경우
public class UserService {
    public void validateEmail(String email) {
        if (email == null || email.isEmpty()) {
            throw new IllegalArgumentException();
        }
    }
}

public class ProductService {
    public void validateName(String name) {
        if (name == null || name.isEmpty()) {
            throw new IllegalArgumentException();
        }
    }
}
// 이메일 검증 규칙과 상품명 검증 규칙은 다른 이유로 변경됨
// → 중복이 아님!

// Case 2: 모양은 다르지만 중복인 경우
public class Phone {
    public Money calculateFee() {
        Money result = Money.ZERO;
        for(Call call : calls) {
            result = result.plus(
                amount.times(call.getDuration().getSeconds() / seconds.getSeconds())
            );
        }
        return result;
    }
}

public class NightlyDiscountPhone {
    public Money calculateFee() {
        Money result = Money.ZERO;
        for(Call call : calls) {
            if (call.getFrom().getHour() >= LATE_NIGHT_HOUR) {
                result = result.plus(
                    nightlyAmount.times(call.getDuration().getSeconds() / seconds.getSeconds())
                );
            } else {
                result = result.plus(
                    regularAmount.times(call.getDuration().getSeconds() / seconds.getSeconds())
                );
            }
        }
        return result;
    }
}
// "통화 요금 계산 정책"이 변경되면 둘 다 변경해야 함
// → 중복이다!
```

### 1.2 DRY 원칙

#### 📜 Don't Repeat Yourself

**정의:**
```
모든 지식은 시스템 내에서
단일하고, 애매하지 않고, 정말로 믿을 만한
표현 양식을 가져야 한다.
```

**핵심:**
```
"지식(knowledge)"의 중복을 제거하는 것이지
단순히 "코드(code)"의 중복을 제거하는 것이 아니다.
```

### 1.3 중복 코드와 변경

#### 📱 전화 요금 계산 시스템

**초기 요구사항:**
```
일반 요금제: 단위 시간당 고정 요금
```

```java
public class Phone {
    private Money amount;        // 단위 요금
    private Duration seconds;    // 단위 시간
    private List<Call> calls = new ArrayList<>();

    public Money calculateFee() {
        Money result = Money.ZERO;
        
        for(Call call : calls) {
            result = result.plus(
                amount.times(call.getDuration().getSeconds() / seconds.getSeconds())
            );
        }
        
        return result;
    }
}
```

#### 🌙 새로운 요구사항: 심야 할인 요금제

**문제: 복사-붙여넣기로 해결**

```java
public class NightlyDiscountPhone {
    private static final int LATE_NIGHT_HOUR = 22;
    
    private Money nightlyAmount;   // 심야 요금
    private Money regularAmount;   // 일반 요금
    private Duration seconds;
    private List<Call> calls = new ArrayList<>();

    public Money calculateFee() {
        Money result = Money.ZERO;
        
        for(Call call : calls) {
            // 📌 심야 시간대 확인 로직만 추가됨
            if (call.getFrom().getHour() >= LATE_NIGHT_HOUR) {
                result = result.plus(
                    nightlyAmount.times(call.getDuration().getSeconds() / seconds.getSeconds())
                );
            } else {
                result = result.plus(
                    regularAmount.times(call.getDuration().getSeconds() / seconds.getSeconds())
                );
            }
        }
        
        return result;
    }
}
```

**복사-붙여넣기의 함정:**

```
✅ 장점:
- 빠른 구현
- 기존 코드에 영향 없음

❌ 단점:
- 중복 코드 생성
- 변경 시 일관성 유지 어려움
- 버그 양산
```

### 1.4 중복 코드 수정의 악몽

#### 💸 새로운 요구사항: 세금 추가

```java
// Phone: 세금 추가
public class Phone {
    private double taxRate;  // 세율 추가
    
    public Phone(Money amount, Duration seconds, double taxRate) {
        this.amount = amount;
        this.seconds = seconds;
        this.taxRate = taxRate;  // 세율 초기화
    }
    
    public Money calculateFee() {
        Money result = Money.ZERO;
        
        for(Call call : calls) {
            result = result.plus(
                amount.times(call.getDuration().getSeconds() / seconds.getSeconds())
            );
        }
        
        // ✅ 세금 계산 추가
        return result.plus(result.times(taxRate));
    }
}
```

```java
// NightlyDiscountPhone: 세금 추가 (중복 코드!)
public class NightlyDiscountPhone {
    private double taxRate;  // 세율 추가 (중복!)
    
    public NightlyDiscountPhone(Money nightlyAmount, Money regularAmount, 
                                Duration seconds, double taxRate) {
        this.nightlyAmount = nightlyAmount;
        this.regularAmount = regularAmount;
        this.seconds = seconds;
        this.taxRate = taxRate;  // 세율 초기화 (중복!)
    }
    
    public Money calculateFee() {
        Money result = Money.ZERO;
        
        for(Call call : calls) {
            if (call.getFrom().getHour() >= LATE_NIGHT_HOUR) {
                result = result.plus(
                    nightlyAmount.times(call.getDuration().getSeconds() / seconds.getSeconds())
                );
            } else {
                result = result.plus(
                    regularAmount.times(call.getDuration().getSeconds() / seconds.getSeconds())
                );
            }
        }
        
        // ❌ 버그: minus가 아니라 plus여야 함!
        return result.minus(result.times(taxRate));
    }
}
```

**문제점 분석:**

```
1. 중복 코드 발견의 어려움
   - 많은 코드 더미 속에서 중복 찾기 어려움
   
2. 일관성 유지 실패
   - Phone: result.plus(result.times(taxRate))  ✅
   - NightlyDiscountPhone: result.minus(...)    ❌
   
3. 중복은 중복을 낳는다
   - 세금 계산 로직이 두 곳에 중복
   - 다음 변경 시에도 두 곳 수정 필요
   - 또 다른 중복 코드 추가...
```

### 1.5 중복 제거 시도 1: 타입 코드

#### 🔄 두 클래스를 하나로 합치기

```java
public class Phone {
    private static final int LATE_NIGHT_HOUR = 22;
    enum PhoneType { REGULAR, NIGHTLY }  // 타입 코드
    
    private PhoneType type;  // 요금제 구분
    
    // 모든 필드를 하나로
    private Money amount;
    private Money regularAmount;
    private Money nightlyAmount;
    private Duration seconds;
    private List<Call> calls = new ArrayList<>();
    
    // 일반 요금제 생성자
    public Phone(Money amount, Duration seconds) {
        this(PhoneType.REGULAR, amount, Money.ZERO, Money.ZERO, seconds);
    }
    
    // 심야 할인 요금제 생성자
    public Phone(Money nightlyAmount, Money regularAmount, Duration seconds) {
        this(PhoneType.NIGHTLY, Money.ZERO, nightlyAmount, regularAmount, seconds);
    }
    
    private Phone(PhoneType type, Money amount, Money nightlyAmount,
                  Money regularAmount, Duration seconds) {
        this.type = type;
        this.amount = amount;
        this.regularAmount = regularAmount;
        this.nightlyAmount = nightlyAmount;
        this.seconds = seconds;
    }
    
    public Money calculateFee() {
        Money result = Money.ZERO;
        
        for(Call call : calls) {
            // ❌ 타입 코드로 분기
            if (type == PhoneType.REGULAR) {
                result = result.plus(
                    amount.times(call.getDuration().getSeconds() / seconds.getSeconds())
                );
            } else {
                if (call.getFrom().getHour() >= LATE_NIGHT_HOUR) {
                    result = result.plus(
                        nightlyAmount.times(call.getDuration().getSeconds() / seconds.getSeconds())
                    );
                } else {
                    result = result.plus(
                        regularAmount.times(call.getDuration().getSeconds() / seconds.getSeconds())
                    );
                }
            }
        }
        
        return result;
    }
}
```

#### ❌ 타입 코드의 문제점

```
1. 낮은 응집도
   - 하나의 클래스가 여러 타입의 요금제를 처리
   - 관련 없는 필드들이 함께 존재
     (일반 요금제는 nightlyAmount 불필요)
   
2. 높은 결합도
   - 새로운 요금제 추가 시 calculateFee 수정 필요
   - if-else 분기가 계속 늘어남
   
3. 단일 책임 원칙 위반
   - 여러 요금제의 계산 로직이 한 곳에
   
4. 개방-폐쇄 원칙 위반
   - 확장(새 요금제)을 위해 수정(기존 코드) 필요
```

### 1.6 중복 제거 시도 2: 상속

#### 🎯 상속을 통한 코드 재사용

```java
// 부모 클래스: 기본 요금 계산
public class Phone {
    private Money amount;
    private Duration seconds;
    private List<Call> calls = new ArrayList<>();
    
    public Phone(Money amount, Duration seconds) {
        this.amount = amount;
        this.seconds = seconds;
    }
    
    public Money calculateFee() {
        Money result = Money.ZERO;
        
        for(Call call : calls) {
            result = result.plus(
                amount.times(call.getDuration().getSeconds() / seconds.getSeconds())
            );
        }
        
        return result;
    }
    
    // protected: 자식이 접근 가능
    protected List<Call> getCalls() { return calls; }
    protected Money getAmount() { return amount; }
    protected Duration getSeconds() { return seconds; }
}
```

```java
// 자식 클래스: 심야 할인 요금 계산
public class NightlyDiscountPhone extends Phone {
    private static final int LATE_NIGHT_HOUR = 22;
    
    private Money nightlyAmount;
    
    public NightlyDiscountPhone(Money nightlyAmount, Money regularAmount, 
                                Duration seconds) {
        super(regularAmount, seconds);  // 부모 생성자 호출
        this.nightlyAmount = nightlyAmount;
    }
    
    @Override
    public Money calculateFee() {
        // ❌ 부모의 calculateFee를 재사용하려는 시도
        Money result = super.calculateFee();  // 일반 요금 계산
        
        // 심야 할인 차감 계산
        Money nightlyFee = Money.ZERO;
        for(Call call : getCalls()) {
            if (call.getFrom().getHour() >= LATE_NIGHT_HOUR) {
                nightlyFee = nightlyFee.plus(
                    getAmount().minus(nightlyAmount).times(
                        call.getDuration().getSeconds() / getSeconds().getSeconds()
                    )
                );
            }
        }
        
        return result.minus(nightlyFee);  // 할인 차감
    }
}
```

#### ⚠️ 이 상속의 문제점

**1. 복잡한 로직:**
```
부모의 calculateFee()가 계산한 일반 요금에서
심야 시간대의 할인 금액을 차감하는 복잡한 방식

개발자의 가정을 이해하기 전에는 코드 이해 어려움
```

**2. 부모 클래스에 강하게 결합:**
```java
// 부모가 이렇게 계산한다는 것을 알아야 함
super.calculateFee();  // 일반 요금 전체 계산

// 부모의 내부 구현에 의존
getAmount()    // protected 메서드
getCalls()     // protected 메서드
getSeconds()   // protected 메서드
```

**3. super 호출의 위험:**

```
자식 클래스의 메서드 안에서
super 참조를 이용해 부모 클래스의 메서드를 직접 호출하면
두 클래스는 강하게 결합된다.

→ 부모 클래스 변경 시 자식 클래스도 함께 변경 필요
```

### 1.7 상속의 진짜 문제: 세금 추가 시나리오

#### 📊 세금 요구사항 추가

```java
// Phone: 세금 계산 추가
public class Phone {
    private Money amount;
    private Duration seconds;
    private List<Call> calls = new ArrayList<>();
    private double taxRate;  // 세율 추가
    
    public Phone(Money amount, Duration seconds, double taxRate) {
        this.amount = amount;
        this.seconds = seconds;
        this.taxRate = taxRate;
    }
    
    public Money calculateFee() {
        Money result = Money.ZERO;
        
        for(Call call : calls) {
            result = result.plus(
                amount.times(call.getDuration().getSeconds() / seconds.getSeconds())
            );
        }
        
        // ✅ 세금 추가
        return result.plus(result.times(taxRate));
    }
    
    public double getTaxRate() { return taxRate; }
}
```

```java
// NightlyDiscountPhone: 세금 때문에 대공사 필요!
public class NightlyDiscountPhone extends Phone {
    private static final int LATE_NIGHT_HOUR = 22;
    
    private Money nightlyAmount;
    
    public NightlyDiscountPhone(Money nightlyAmount, Money regularAmount, 
                                Duration seconds, double taxRate) {
        super(regularAmount, seconds, taxRate);  // 생성자 시그니처 변경
        this.nightlyAmount = nightlyAmount;
    }
    
    @Override
    public Money calculateFee() {
        // 부모의 calculateFee() 호출 (세금 포함)
        Money result = super.calculateFee();
        
        Money nightlyFee = Money.ZERO;
        for(Call call : getCalls()) {
            if (call.getFrom().getHour() >= LATE_NIGHT_HOUR) {
                nightlyFee = nightlyFee.plus(
                    getAmount().minus(nightlyAmount).times(
                        call.getDuration().getSeconds() / getSeconds().getSeconds()
                    )
                );
            }
        }
        
        // ❌ 세금 계산도 중복!
        return result.minus(
            nightlyFee.plus(nightlyFee.times(getTaxRate()))
        );
    }
}
```

#### 💥 문제점 폭발

```
Phone의 코드를 재사용하고 중복 코드를 제거하기 위해
Phone의 자식 클래스로 NightlyDiscountPhone을 만들었는데...

세금 계산 로직을 추가하기 위해:
1. Phone 수정
2. NightlyDiscountPhone도 수정  ← 새로운 중복 코드!

이것은 NightlyDiscountPhone이 Phone의 구현에
너무 강하게 결합되어 있기 때문에 발생하는 문제다.
```

### 1.8 상속을 위한 경고 1

```
┌─────────────────────────────────────────────────────┐
│  자식 클래스의 메서드 안에서                               │
│  super 참조를 이용해 부모 클래스의 메서드를                  │
│  직접 호출할 경우 두 클래스는 강하게 결합된다.                 │
│                                                     │
│  super 호출을 제거할 수 있는 방법을 찾아                    │
│  결합도를 제거하라.                                      │
└─────────────────────────────────────────────────────┘
```

---

## 2. 취약한 기반 클래스 문제

> 📂 **코드**: [`InstrumentedHashSet.java`](https://github.com/eternity-oop/object/blob/master/chapter10/src/main/java/org/eternity/instrumented/InstrumentedHashSet.java) | [`Playlist.java`](https://github.com/eternity-oop/object/blob/master/chapter10/src/main/java/org/eternity/playlist/step01/Playlist.java)

### 2.1 취약한 기반 클래스 문제란?

#### 🎯 Fragile Base Class Problem

**정의:**
```
부모 클래스의 작은 변경에도
자식 클래스가 컴파일 오류나 실행 에러에 시달리는 현상

상속이라는 문맥 안에서
결합도가 초래하는 문제점을 가리키는 용어
```

**핵심:**
```
상속 관계를 추가할수록
전체 시스템의 결합도가 높아진다.

상속은 객체지향 프로그래밍의
근본적인 취약성이다.
```

#### 💡 캡슐화 vs 상속

**객체지향의 기반:**
```
캡슐화를 통한 변경의 통제

구현을 퍼블릭 인터페이스 뒤로 감춤으로써
변경의 파급효과를 제어
```

**상속의 문제:**
```
부모 클래스의 퍼블릭 인터페이스가 아닌
구현을 변경하더라도
자식 클래스가 영향을 받기 쉬움

→ 상속은 캡슐화를 약화시킨다!
```

### 2.2 불필요한 인터페이스 상속 문제

#### 📚 자바 초기 버전의 설계 실수

**Stack이 Vector를 상속:**

```java
public class Stack<E> extends Vector<E> {
    public E push(E item) { ... }
    public E pop() { ... }
    public E peek() { ... }
    public boolean empty() { ... }
    public int search(Object o) { ... }
}
```

**문제 발생:**

```java
Stack<String> stack = new Stack<>();
stack.push("1st");
stack.push("2nd");
stack.push("3rd");

// ❌ Vector의 메서드 사용 가능!
stack.add(0, "4th");  // 임의의 위치에 추가

// Stack 규칙 위반!
assertEquals("3rd", stack.pop());  // 실패!
// 실제로는 "4th"가 나옴
```

**구조 분석:**

```
┌──────────────┐
│   Vector     │
│              │
│ - add()      │  ← 임의 위치 추가
│ - get()      │  ← 임의 위치 조회
│ - remove()   │  ← 임의 위치 삭제
└──────┬───────┘
       │ 상속
       ↓
┌──────────────┐
│    Stack     │
│              │
│ - push()     │  ← LIFO 규칙
│ - pop()      │  ← LIFO 규칙
│ - peek()     │  ← LIFO 규칙
└──────────────┘

문제:
Vector의 메서드가 Stack의 LIFO 규칙을 깨뜨림!
```

#### 🗂️ Properties가 Hashtable을 상속

```java
public class Properties extends Hashtable<Object, Object> {
    public synchronized Object setProperty(String key, String value) { ... }
    public String getProperty(String key) { ... }
}
```

**문제 발생:**

```java
Properties properties = new Properties();
properties.setProperty("Bjarne Stroustrup", "C++");
properties.setProperty("James Gosling", "Java");

// ❌ Hashtable의 put 사용 (타입 안전성 깨짐)
properties.put("Dennis Ritchie", 67);  // Integer!

// null 반환
assertEquals("C", properties.getProperty("Dennis Ritchie"));  // 실패!
```

**문제 분석:**

```
Properties의 의도:
- 키: String
- 값: String

Hashtable 상속으로 인한 문제:
- put(Object, Object) 메서드 사용 가능
- 타입 안전성 보장 안 됨
- getProperty는 String이 아니면 null 반환
```

### 2.3 상속을 위한 경고 2

```
┌─────────────────────────────────────────────────────┐
│  상속받은 부모 클래스의 메서드가                            │
│  자식 클래스의 내부 구조에 대한 규칙을                       │
│  깨뜨릴 수 있다.                                        │
└─────────────────────────────────────────────────────┘
```

### 2.4 메서드 오버라이딩의 오작용 문제

#### 🔢 InstrumentedHashSet: 추가 횟수 카운팅

**목적:**
```
HashSet에 요소가 추가된 횟수를 기록하고 싶다.
```

**구현 (1차 시도):**

```java
public class InstrumentedHashSet<E> extends HashSet<E> {
    private int addCount = 0;
    
    public InstrumentedHashSet() { }
    
    public InstrumentedHashSet(int initCap, float loadFactor) {
        super(initCap, loadFactor);
    }
    
    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }
    
    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }
    
    public int getAddCount() {
        return addCount;
    }
}
```

**테스트:**

```java
InstrumentedHashSet<String> languages = new InstrumentedHashSet<>();
languages.addAll(Arrays.asList("Java", "Ruby", "Scala"));

System.out.println(languages.getAddCount());
// 예상: 3
// 실제: 6  ❌
```

#### 🤔 왜 6이 나올까?

**HashSet의 내부 구현:**

```java
public class HashSet<E> {
    @Override
    public boolean addAll(Collection<? extends E> c) {
        boolean modified = false;
        for (E e : c) {
            if (add(e))  // ← 여기서 add 호출!
                modified = true;
        }
        return modified;
    }
}
```

**호출 흐름:**

```
1. languages.addAll(["Java", "Ruby", "Scala"])
   → InstrumentedHashSet.addAll() 호출
   → addCount += 3  (addCount = 3)
   
2. super.addAll() 호출
   → HashSet.addAll() 실행
   
3. HashSet.addAll() 내부에서 add() 호출
   → InstrumentedHashSet.add() 호출됨 (오버라이드!)
   → "Java" 추가: addCount++  (addCount = 4)
   → "Ruby" 추가: addCount++  (addCount = 5)
   → "Scala" 추가: addCount++ (addCount = 6)

결과: addCount = 6
```

#### 🔧 수정 시도 1: addAll 메서드 제거

```java
public class InstrumentedHashSet<E> extends HashSet<E> {
    private int addCount = 0;
    
    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }
    
    // addAll 제거
    
    public int getAddCount() {
        return addCount;
    }
}
```

**결과:**
```
✅ 문제 해결: addCount = 3

하지만...
❌ HashSet의 addAll 구현에 의존
❌ HashSet이 addAll에서 add를 호출한다는 가정
❌ 미래에 HashSet 구현이 바뀌면 깨질 수 있음
```

#### 🔧 수정 시도 2: addAll 직접 구현

```java
public class InstrumentedHashSet<E> extends HashSet<E> {
    private int addCount = 0;
    
    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }
    
    @Override
    public boolean addAll(Collection<? extends E> c) {
        boolean modified = false;
        for (E e : c) {
            if (add(e))  // add 직접 호출
                modified = true;
        }
        return modified;
    }
    
    public int getAddCount() {
        return addCount;
    }
}
```

**결과:**
```
✅ 문제 해결: addCount = 3

하지만...
❌ 코드 중복: HashSet의 addAll과 동일한 구현
❌ HashSet이 최적화를 개선하면 혜택 못 받음
```

#### 💡 핵심 문제

```
자식 클래스가
부모 클래스의 메서드를 오버라이딩할 경우

부모 클래스가 자신의 메서드를 사용하는 방법에
자식 클래스가 결합될 수 있다.

부모의 내부 구현을 알아야만
올바르게 오버라이딩할 수 있음
→ 캡슐화 위반!
```

### 2.5 상속을 위한 경고 3

```
┌─────────────────────────────────────────────────────┐
│  자식 클래스가 부모 클래스의 메서드를                        │
│  오버라이딩할 경우                                       │
│  부모 클래스가 자신의 메서드를 사용하는 방법에                 │
│  자식 클래스가 결합될 수 있다.                             │
└─────────────────────────────────────────────────────┘
```

#### 📖 조슈아 블로치의 조언

```
"클래스가 상속되기를 원한다면
 상속을 위해 클래스를 설계하고 문서화해야 하며,
 그렇지 않은 경우에는 상속을 금지시켜야 한다."
```

**문제:**
```
내부 구현을 공개하고 문서화하는 것이 옳은가?
→ 캡슐화를 위반하는 것 아닌가?

답: 설계는 트레이드오프 활동이다.
    상속은 코드 재사용을 위해 캡슐화를 희생한다.
```

### 2.6 부모 클래스와 자식 클래스의 동시 수정 문제

#### 🎵 Playlist 예제

**초기 구현:**

```java
public class Song {
    private String singer;
    private String title;
    
    public Song(String singer, String title) {
        this.singer = singer;
        this.title = title;
    }
    
    public String getSinger() { return singer; }
    public String getTitle() { return title; }
}
```

```java
public class Playlist {
    private List<Song> tracks = new ArrayList<>();
    
    public void append(Song song) {
        getTracks().add(song);
    }
    
    public List<Song> getTracks() {
        return tracks;
    }
}
```

```java
public class PersonalPlaylist extends Playlist {
    public void remove(Song song) {
        getTracks().remove(song);
    }
}
```

**사용:**

```java
Playlist playlist = new Playlist();
playlist.append(new Song("Adele", "Hello"));
playlist.append(new Song("The Beatles", "Yesterday"));

PersonalPlaylist personal = new PersonalPlaylist();
personal.append(new Song("Coldplay", "Viva La Vida"));
personal.remove(new Song("Coldplay", "Viva La Vida"));
```

#### 📊 요구사항 변경

```
Playlist에서 노래 목록뿐만 아니라
가수별 노래 제목도 함께 관리해야 한다.
```

**Playlist 수정:**

```java
public class Playlist {
    private List<Song> tracks = new ArrayList<>();
    private Map<String, String> singers = new HashMap<>();  // 추가
    
    public void append(Song song) {
        tracks.add(song);
        singers.put(song.getSinger(), song.getTitle());  // 추가
    }
    
    public List<Song> getTracks() {
        return tracks;
    }
    
    public Map<String, String> getSingers() {  // 추가
        return singers;
    }
}
```

**PersonalPlaylist도 수정 필요:**

```java
public class PersonalPlaylist extends Playlist {
    public void remove(Song song) {
        getTracks().remove(song);
        getSingers().remove(song.getSinger());  // 추가 필요!
    }
}
```

#### 💥 문제 분석

```
PersonalPlaylist는:
- 부모 클래스의 메서드를 오버라이딩하지 않았음
- 불필요한 인터페이스를 상속받지 않았음

그런데도:
- 부모 클래스를 수정할 때
- 자식 클래스를 함께 수정해야 함

이유:
부모와 자식이 내부 구현을 공유하기 때문!
```

### 2.7 상속을 위한 경고 4

```
┌─────────────────────────────────────────────────────┐
│  클래스를 상속하면 결합도로 인해                            │
│  자식 클래스와 부모 클래스의 구현을                         │
│  영원히 변경하지 않거나,                                 │
│  자식 클래스와 부모 클래스를 동시에 변경하거나                 │
│  둘 중 하나를 선택할 수밖에 없다.                          │
└─────────────────────────────────────────────────────┘
```

### 2.8 취약한 기반 클래스 문제 정리

#### 🎯 핵심 개념

```
취약한 기반 클래스 문제:
상속 관계로 연결된 자식 클래스가
부모 클래스의 변경에 취약해지는 현상

상속을 사용한다면
피할 수 없는 객체지향 프로그래밍의 근본적인 취약성
```

#### 📊 문제 유형

| 문제 유형 | 설명 | 예시 |
|----------|------|------|
| **불필요한 인터페이스 상속** | 부모의 메서드가 자식의 규칙을 깨뜨림 | Stack ← Vector |
| **메서드 오버라이딩 오작용** | 부모의 내부 구현에 의존 | InstrumentedHashSet |
| **부모-자식 동시 수정** | 내부 구현 공유로 인한 결합 | PersonalPlaylist |

#### 💡 근본 원인

```
상속은 코드 재사용을 위해
캡슐화를 희생한다.

완벽한 캡슐화를 원한다면
코드 재사용을 포기하거나
상속 이외의 다른 방법을 사용해야 한다.
```

---

## 3. Phone 다시 살펴보기

> 📂 **코드**: [`Phone.java (step06-08)`](https://github.com/eternity-oop/object/blob/master/chapter10/src/main/java/org/eternity/billing/step06/AbstractPhone.java) | [`RegularPhone.java`](https://github.com/eternity-oop/object/blob/master/chapter10/src/main/java/org/eternity/billing/step07/RegularPhone.java)

### 3.1 추상화에 의존하자

#### 🎯 해결책의 핵심

```
취약한 기반 클래스 문제를 완전히 없앨 수는 없지만
어느 정도까지 위험을 완화시키는 것은 가능하다.

열쇠는 추상화다.
부모 클래스와 자식 클래스 모두 추상화에 의존하도록 만들어야 한다.
```

#### 📏 두 가지 설계 원칙

**코드 중복을 제거하기 위해 상속을 도입할 때 따르는 원칙:**

```
1. 두 메서드가 유사하게 보인다면
   차이점을 메서드로 추출하라.
   
2. 부모 클래스의 코드를 하위로 내리지 말고
   자식 클래스의 코드를 상위로 올려라.
```

**왜 "올리기"인가?**

```
하위로 내리기:
- 구체적인 코드를 자식으로 내림
- 자식이 부모의 구체적 구현에 의존
- 결합도 증가

상위로 올리기:
- 추상적인 코드를 부모로 올림
- 부모가 추상화를 제공
- 결합도 감소
```

### 3.2 차이를 메서드로 추출하라

#### 📱 현재 코드 분석

**Phone (일반 요금제):**

```java
public class Phone {
    private Money amount;
    private Duration seconds;
    private List<Call> calls = new ArrayList<>();
    
    public Money calculateFee() {
        Money result = Money.ZERO;
        
        for(Call call : calls) {
            // 📌 개별 통화 요금 계산
            result = result.plus(
                amount.times(call.getDuration().getSeconds() / seconds.getSeconds())
            );
        }
        
        return result;
    }
}
```

**NightlyDiscountPhone (심야 할인 요금제):**

```java
public class NightlyDiscountPhone {
    private static final int LATE_NIGHT_HOUR = 22;
    
    private Money nightlyAmount;
    private Money regularAmount;
    private Duration seconds;
    private List<Call> calls = new ArrayList<>();
    
    public Money calculateFee() {
        Money result = Money.ZERO;
        
        for(Call call : calls) {
            // 📌 개별 통화 요금 계산 (조건부)
            if (call.getFrom().getHour() >= LATE_NIGHT_HOUR) {
                result = result.plus(
                    nightlyAmount.times(call.getDuration().getSeconds() / seconds.getSeconds())
                );
            } else {
                result = result.plus(
                    regularAmount.times(call.getDuration().getSeconds() / seconds.getSeconds())
                );
            }
        }
        
        return result;
    }
}
```

#### 🔍 공통점과 차이점 분석

**공통점 (변하지 않는 부분):**
```java
Money result = Money.ZERO;

for(Call call : calls) {
    result = result.plus(...);  // ← 이 구조는 동일
}

return result;
```

**차이점 (변하는 부분):**
```java
// Phone
amount.times(call.getDuration().getSeconds() / seconds.getSeconds())

// NightlyDiscountPhone  
if (call.getFrom().getHour() >= LATE_NIGHT_HOUR) {
    nightlyAmount.times(...)
} else {
    regularAmount.times(...)
}
```

#### ✂️ 메서드 추출

**Phone:**

```java
public class Phone {
    private Money amount;
    private Duration seconds;
    private List<Call> calls = new ArrayList<>();
    
    public Money calculateFee() {
        Money result = Money.ZERO;
        
        for(Call call : calls) {
            // ✅ 차이점을 메서드로 추출
            result = result.plus(calculateCallFee(call));
        }
        
        return result;
    }
    
    // ✅ 개별 통화 요금 계산을 별도 메서드로
    protected Money calculateCallFee(Call call) {
        return amount.times(call.getDuration().getSeconds() / seconds.getSeconds());
    }
}
```

**NightlyDiscountPhone:**

```java
public class NightlyDiscountPhone {
    private static final int LATE_NIGHT_HOUR = 22;
    
    private Money nightlyAmount;
    private Money regularAmount;
    private Duration seconds;
    private List<Call> calls = new ArrayList<>();
    
    public Money calculateFee() {
        Money result = Money.ZERO;
        
        for(Call call : calls) {
            // ✅ 차이점을 메서드로 추출
            result = result.plus(calculateCallFee(call));
        }
        
        return result;
    }
    
    // ✅ 개별 통화 요금 계산을 별도 메서드로
    protected Money calculateCallFee(Call call) {
        if (call.getFrom().getHour() >= LATE_NIGHT_HOUR) {
            return nightlyAmount.times(call.getDuration().getSeconds() / seconds.getSeconds());
        } else {
            return regularAmount.times(call.getDuration().getSeconds() / seconds.getSeconds());
        }
    }
}
```

#### 📊 결과

```
이제 두 클래스의 calculateFee 메서드는 완전히 동일!

공통 부분:
public Money calculateFee() {
    Money result = Money.ZERO;
    for(Call call : calls) {
        result = result.plus(calculateCallFee(call));
    }
    return result;
}

차이점은 calculateCallFee 메서드에 격리됨!
```

### 3.3 중복 코드를 부모 클래스로 올려라

#### 🏗️ 상속 계층 구성

**Step 1: 추상 클래스 생성**

```java
public abstract class AbstractPhone {
    private List<Call> calls = new ArrayList<>();
    
    // ✅ 공통 로직을 부모로
    public Money calculateFee() {
        Money result = Money.ZERO;
        
        for(Call call : calls) {
            result = result.plus(calculateCallFee(call));
        }
        
        return result;
    }
    
    // ✅ 변하는 부분은 추상 메서드로
    abstract protected Money calculateCallFee(Call call);
}
```

**Step 2: Phone을 AbstractPhone의 자식으로**

```java
public class Phone extends AbstractPhone {
    private Money amount;
    private Duration seconds;
    
    public Phone(Money amount, Duration seconds) {
        this.amount = amount;
        this.seconds = seconds;
    }
    
    @Override
    protected Money calculateCallFee(Call call) {
        return amount.times(call.getDuration().getSeconds() / seconds.getSeconds());
    }
}
```

**Step 3: NightlyDiscountPhone을 AbstractPhone의 자식으로**

```java
public class NightlyDiscountPhone extends AbstractPhone {
    private static final int LATE_NIGHT_HOUR = 22;
    
    private Money nightlyAmount;
    private Money regularAmount;
    private Duration seconds;
    
    public NightlyDiscountPhone(Money nightlyAmount, Money regularAmount, Duration seconds) {
        this.nightlyAmount = nightlyAmount;
        this.regularAmount = regularAmount;
        this.seconds = seconds;
    }
    
    @Override
    protected Money calculateCallFee(Call call) {
        if (call.getFrom().getHour() >= LATE_NIGHT_HOUR) {
            return nightlyAmount.times(call.getDuration().getSeconds() / seconds.getSeconds());
        } else {
            return regularAmount.times(call.getDuration().getSeconds() / seconds.getSeconds());
        }
    }
}
```

#### 📊 클래스 다이어그램

```
┌─────────────────────────┐
│   AbstractPhone         │  ← 추상화
│   «abstract»            │
├─────────────────────────┤
│ - calls: List<Call>     │
├─────────────────────────┤
│ + calculateFee(): Money │  ← 템플릿 메서드
│ # calculateCallFee():   │  ← 훅 메서드
│   Money «abstract»      │
└───────────┬─────────────┘
            │
            ├──────────────────────┐
            │                      │
┌───────────▼──────────┐  ┌────────▼──────────────┐
│   Phone              │  │ NightlyDiscountPhone  │
│                      │  │                       │
├──────────────────────┤  ├───────────────────────┤
│ - amount: Money      │  │ - nightlyAmount: Money│
│ - seconds: Duration  │  │ - regularAmount: Money│
├──────────────────────┤  │ - seconds: Duration   │
│ + calculateCallFee():│  ├───────────────────────┤
│   Money              │  │ + calculateCallFee(): │
└──────────────────────┘  │   Money               │
                          └───────────────────────┘

템플릿 메서드 패턴 (Template Method Pattern)
```

### 3.4 추상화가 핵심이다

#### ✅ 개선된 설계의 장점

**1. 단일 책임 원칙 (SRP) 준수:**

```
AbstractPhone:
- 전체 통화 목록을 계산하는 방법이 바뀔 경우에만 변경

Phone:
- 일반 요금제의 통화 한 건을 계산하는 방식이 바뀔 경우에만 변경

NightlyDiscountPhone:
- 심야 할인 요금제의 통화 한 건을 계산하는 방식이 바뀔 경우에만 변경

→ 각 클래스는 하나의 변경 이유만 가짐 = 높은 응집도
```

**2. 낮은 결합도:**

```java
// calculateCallFee 메서드의 시그니처가 변경되지 않는 한
// 부모 클래스의 내부 구현이 변경되어도 자식 클래스는 영향받지 않음

abstract protected Money calculateCallFee(Call call);
```

**3. 개방-폐쇄 원칙 (OCP) 준수:**

```java
// 새로운 요금제 추가 시
// 기존 코드 수정 없이 새로운 클래스만 추가

public class WeekendDiscountPhone extends AbstractPhone {
    @Override
    protected Money calculateCallFee(Call call) {
        // 주말 할인 로직
    }
}
```

#### 💡 추상화의 힘

```
모든 장점은 클래스들이 추상화에 의존하기 때문에 얻어지는 것이다.

AbstractPhone (추상화):
- 변하지 않는 전체 흐름 정의
- calculateFee() 템플릿 메서드

Phone, NightlyDiscountPhone (구체 클래스):
- 변하는 세부 사항만 구현
- calculateCallFee() 구현
```

### 3.5 의도를 드러내는 이름 선택하기

#### 📝 클래스 이름 개선

**Before:**
```
AbstractPhone   ← 추상 클래스라는 사실만 드러남
Phone           ← 일반 요금제라는 의미 불명확
NightlyDiscountPhone  ← 의미 명확
```

**After:**
```
Phone                  ← 전화기의 공통 개념
RegularPhone          ← 일반 요금제 (의미 명확)
NightlyDiscountPhone  ← 심야 할인 요금제
```

**최종 구조:**

```java
// 부모 클래스: 추상적 개념
public abstract class Phone {
    private List<Call> calls = new ArrayList<>();
    
    public Money calculateFee() {
        Money result = Money.ZERO;
        for(Call call : calls) {
            result = result.plus(calculateCallFee(call));
        }
        return result;
    }
    
    abstract protected Money calculateCallFee(Call call);
}

// 자식 클래스 1: 구체적 정책
public class RegularPhone extends Phone {
    private Money amount;
    private Duration seconds;
    
    public RegularPhone(Money amount, Duration seconds) {
        this.amount = amount;
        this.seconds = seconds;
    }
    
    @Override
    protected Money calculateCallFee(Call call) {
        return amount.times(call.getDuration().getSeconds() / seconds.getSeconds());
    }
}

// 자식 클래스 2: 구체적 정책
public class NightlyDiscountPhone extends Phone {
    // ...
}
```

### 3.6 세금 추가하기

#### 💰 요구사항: 세금 계산

**부모 클래스 수정:**

```java
public abstract class Phone {
    private double taxRate;  // 세율 추가
    private List<Call> calls = new ArrayList<>();
    
    public Phone(double taxRate) {
        this.taxRate = taxRate;
    }
    
    public Money calculateFee() {
        Money result = Money.ZERO;
        
        for(Call call : calls) {
            result = result.plus(calculateCallFee(call));
        }
        
        // ✅ 세금 계산 추가
        return result.plus(result.times(taxRate));
    }
    
    protected abstract Money calculateCallFee(Call call);
}
```

**자식 클래스들 수정:**

```java
public class RegularPhone extends Phone {
    private Money amount;
    private Duration seconds;
    
    // ✅ 생성자만 수정 (세율 전달)
    public RegularPhone(Money amount, Duration seconds, double taxRate) {
        super(taxRate);
        this.amount = amount;
        this.seconds = seconds;
    }
    
    // calculateCallFee는 수정 불필요!
    @Override
    protected Money calculateCallFee(Call call) {
        return amount.times(call.getDuration().getSeconds() / seconds.getSeconds());
    }
}
```

```java
public class NightlyDiscountPhone extends Phone {
    private static final int LATE_NIGHT_HOUR = 22;
    
    private Money nightlyAmount;
    private Money regularAmount;
    private Duration seconds;
    
    // ✅ 생성자만 수정 (세율 전달)
    public NightlyDiscountPhone(Money nightlyAmount, Money regularAmount, 
                                Duration seconds, double taxRate) {
        super(taxRate);
        this.nightlyAmount = nightlyAmount;
        this.regularAmount = regularAmount;
        this.seconds = seconds;
    }
    
    // calculateCallFee는 수정 불필요!
    @Override
    protected Money calculateCallFee(Call call) {
        if (call.getFrom().getHour() >= LATE_NIGHT_HOUR) {
            return nightlyAmount.times(call.getDuration().getSeconds() / seconds.getSeconds());
        } else {
            return regularAmount.times(call.getDuration().getSeconds() / seconds.getSeconds());
        }
    }
}
```

#### 📊 변경 비교

**이전 (상속 잘못 사용):**
```
Phone 수정:
1. taxRate 필드 추가
2. 생성자 수정
3. calculateFee에 세금 계산 추가
4. getTaxRate() 메서드 추가

NightlyDiscountPhone 수정:
1. 생성자 시그니처 변경
2. super() 호출 수정
3. calculateFee에 세금 계산 추가  ← 중복!
4. getTaxRate() 사용

→ 핵심 로직이 중복됨!
```

**현재 (추상화 의존):**
```
Phone 수정:
1. taxRate 필드 추가
2. 생성자 수정
3. calculateFee에 세금 계산 추가

RegularPhone 수정:
1. 생성자만 수정

NightlyDiscountPhone 수정:
1. 생성자만 수정

→ 핵심 로직은 한 곳에만!
```

#### 💡 중요한 교훈

```
책임을 아무리 잘 분리하더라도
인스턴스 변수의 추가는
종종 상속 계층 전반에 걸친 변경을 유발한다.

하지만:
인스턴스 초기화 로직을 변경하는 것이
두 클래스에 동일한 세금 계산 코드를 중복시키는 것보다
현명한 선택이다.
```

#### 🎯 설계 원칙

```
객체 생성 로직에 대한 변경을 막기보다는
핵심 로직의 중복을 막아라.

핵심 로직은 한 곳에 모아 놓고
조심스럽게 캡슐화해야 한다.

공통적인 핵심 로직은
최대한 추상화해야 한다.
```

### 3.7 "위로 올리기" 전략의 장점

#### 📈 실패해도 안전한 전략

**"아래로 내리기" 전략:**
```
1. 부모 클래스에 구체적인 메서드 작성
2. 자식 클래스에서 필요한 것만 오버라이드

문제:
- 실수하면 발견하기 어려움
- 자식이 부모의 구체적 구현에 의존
- 결합도 높음
```

**"위로 올리기" 전략:**
```
1. 자식 클래스들의 공통 코드 식별
2. 공통 코드를 부모로 이동
3. 차이점은 추상 메서드로

장점:
- 실수해도 공통 코드가 눈에 띔
- 점진적으로 추상화 개선 가능
- 자연스럽게 코드 품질 향상
```

#### 💡 핵심

```
"위로 올리기" 전략은
실패했더라도 수정하기 쉬운 문제를 발생시킨다.

추상화할 코드는 눈에 띄고
결국 상위 클래스로 올려지면서
코드의 품질은 높아진다.
```

---

## 4. 차이에 의한 프로그래밍

### 4.1 Programming by Difference

#### 🎯 정의

**차이에 의한 프로그래밍:**
```
기존 코드와 다른 부분만을 추가함으로써
애플리케이션의 기능을 확장하는 방법
```

**예시:**

```java
// 기존 코드
public abstract class Phone {
    public Money calculateFee() {
        Money result = Money.ZERO;
        for(Call call : calls) {
            result = result.plus(calculateCallFee(call));
        }
        return result.plus(result.times(taxRate));
    }
    
    protected abstract Money calculateCallFee(Call call);
}

// 차이점만 추가하여 새로운 기능 구현
public class WeekendDiscountPhone extends Phone {
    @Override
    protected Money calculateCallFee(Call call) {
        // 주말 여부 확인 로직만 추가
        if (isWeekend(call)) {
            return weekendAmount.times(...);
        } else {
            return regularAmount.times(...);
        }
    }
    
    private boolean isWeekend(Call call) {
        DayOfWeek dayOfWeek = call.getFrom().getDayOfWeek();
        return dayOfWeek == DayOfWeek.SATURDAY || 
               dayOfWeek == DayOfWeek.SUNDAY;
    }
}
```

### 4.2 중복 코드 제거와 코드 재사용

#### 💎 핵심 관계

```
중복 코드 제거 ⇄ 코드 재사용

동일한 행동을 가리키는
서로 다른 단어일 뿐이다.

중복 코드를 제거하기 위해
최대한 코드를 재사용해야 한다.
```

#### 🎯 상속의 목적

**상속 (Inheritance):**
```
객체지향에서 중복 코드를 제거하고
코드를 재사용할 수 있는 가장 유명한 방법

하지만:
가장 널리 알려진 방법이
가장 좋은 방법은 아니다.
```

### 4.3 코드 재사용의 진정한 가치

#### 💡 타이핑 수고를 덜어주는 것 이상

```
코드를 재사용하는 것은
단순히 문자를 타이핑하는 수고를 덜어주는
수준의 문제가 아니다.

재사용 가능한 코드란
심각한 버그가 존재하지 않는 코드다.
```

**코드 재사용의 이점:**

```
1. 검증된 코드 활용
   - 이미 테스트를 거친 코드
   - 실전에서 검증된 코드
   
2. 품질 유지
   - 버그 감소
   - 일관된 동작
   
3. 개발 효율
   - 작성 시간 절약
   - 테스트 시간 절약
```

### 4.4 상속의 양면성

#### ⚖️ 장점과 단점

**장점:**
```
✅ 강력한 코드 재사용 메커니즘
✅ 빠른 개발
✅ 코드 양 최소화
✅ 차이에 의한 프로그래밍 가능
```

**단점:**
```
❌ 부모-자식 강한 결합
❌ 취약한 기반 클래스 문제
❌ 캡슐화 약화
❌ 변경의 파급효과
```

#### 🎯 상속 사용 원칙

```
상속은 강력한 도구이지만
맹목적으로 사용하면 위험하다.

정말로 필요한 경우에만 상속을 사용하라.
```

**상속이 적절한 경우:**
```
1. is-a 관계가 명확한 경우
   RegularPhone is-a Phone ✅
   
2. 부모가 진정한 추상화인 경우
   Phone (추상 클래스) ✅
   
3. 확장을 고려해 설계된 경우
   템플릿 메서드 패턴 ✅
```

**상속을 피해야 하는 경우:**
```
1. 단순 코드 재사용 목적
   → 합성 사용
   
2. 구체 클래스 상속
   → 취약한 기반 클래스 문제
   
3. 다중 상속이 필요한 경우
   → 인터페이스 + 합성
```

### 4.5 더 나은 대안: 합성

#### 🔮 다음 장 예고

```
객체지향에 능숙한 개발자들은
상속의 단점을 피하면서
코드를 재사용할 수 있는 더 좋은 방법을 알고 있다.

바로 합성(Composition)이다.
```

**합성 vs 상속:**

```
상속 (Inheritance):
- is-a 관계
- 화이트박스 재사용 (내부가 보임)
- 컴파일 타임에 결정
- 강한 결합

합성 (Composition):
- has-a 관계
- 블랙박스 재사용 (내부가 안 보임)
- 런타임에 변경 가능
- 약한 결합
```

---

## 5. 핵심 정리

### 🎯 중복 코드와 변경

#### DRY 원칙

```
중복 여부를 판단하는 기준은 변경이다.

요구사항이 변경됐을 때
두 코드를 함께 수정해야 한다면
이 코드는 중복이다.

Don't Repeat Yourself:
모든 지식은 시스템 내에서
단일하고, 애매하지 않고, 정말로 믿을 만한
표현 양식을 가져야 한다.
```

#### 중복 코드의 문제점

```
1. 발견의 어려움
   많은 코드 더미 속에서 중복 찾기 어려움
   
2. 일관성 유지 실패
   중복 코드를 서로 다르게 수정하기 쉬움
   
3. 중복의 연쇄
   중복 코드는 새로운 중복 코드를 부름
```

### ⚠️ 취약한 기반 클래스 문제

#### 4가지 경고

**경고 1: super 호출**
```
자식 클래스의 메서드 안에서
super 참조를 이용해 부모 클래스의 메서드를
직접 호출할 경우 두 클래스는 강하게 결합된다.

super 호출을 제거할 수 있는 방법을 찾아
결합도를 제거하라.
```

**경고 2: 불필요한 인터페이스 상속**
```
상속받은 부모 클래스의 메서드가
자식 클래스의 내부 구조에 대한 규칙을
깨뜨릴 수 있다.

예: Stack extends Vector
```

**경고 3: 메서드 오버라이딩**
```
자식 클래스가 부모 클래스의 메서드를
오버라이딩할 경우
부모 클래스가 자신의 메서드를 사용하는 방법에
자식 클래스가 결합될 수 있다.

예: InstrumentedHashSet
```

**경고 4: 동시 수정**
```
클래스를 상속하면 결합도로 인해
자식 클래스와 부모 클래스의 구현을
영원히 변경하지 않거나,
동시에 변경하거나
둘 중 하나를 선택할 수밖에 없다.
```

### 💡 올바른 상속 사용법

#### 2가지 원칙

```
1. 두 메서드가 유사하게 보인다면
   차이점을 메서드로 추출하라.
   
2. 부모 클래스의 코드를 하위로 내리지 말고
   자식 클래스의 코드를 상위로 올려라.
```

#### 추상화에 의존

```
부모 클래스와 자식 클래스 모두
추상화에 의존하도록 만들어야 한다.

템플릿 메서드 패턴:
- 부모: 변하지 않는 전체 흐름 (템플릿 메서드)
- 자식: 변하는 세부 사항 (훅 메서드)
```

### 🎓 차이에 의한 프로그래밍

#### 핵심 개념

```
기존 코드와 다른 부분만을 추가함으로써
애플리케이션의 기능을 확장하는 방법

상속을 이용하면:
- 이미 존재하는 클래스의 코드를 쉽게 재사용
- 애플리케이션의 점진적인 정의 가능
- 검증된 코드 활용
```

#### 중요한 조언

```
상속은 강력한 도구이지만
맹목적으로 사용하면 위험하다.

정말로 필요한 경우에만 상속을 사용하라.

더 좋은 방법:
합성 (Composition)
→ 11장에서 학습
```

### 📊 설계 품질 지표

**좋은 상속 설계:**

```
✅ 단일 책임 원칙 준수
   - 각 클래스는 하나의 변경 이유만 가짐
   
✅ 개방-폐쇄 원칙 준수
   - 확장에는 열려 있고 수정에는 닫혀 있음
   
✅ 추상화에 의존
   - 부모와 자식 모두 추상화 의존
   
✅ 낮은 결합도
   - 부모 변경이 자식에 최소 영향
   
✅ 높은 응집도
   - 관련된 책임이 한 곳에 모임
```

**나쁜 상속 설계:**

```
❌ super 호출 남발
   - 강한 결합도
   
❌ 구체 클래스 상속
   - 취약한 기반 클래스 문제
   
❌ 메서드 오버라이딩 오작용
   - 부모의 내부 구현에 의존
   
❌ 타입 코드 사용
   - 낮은 응집도, 높은 결합도
   
❌ 불필요한 인터페이스 노출
   - 규칙 위반 가능성
```

### 💎 핵심 교훈

```
상속은 코드 재사용을 위해
캡슐화를 희생한다.

완벽한 캡슐화를 원한다면
코드 재사용을 포기하거나
상속 이외의 다른 방법을 사용해야 한다.

설계는 트레이드오프 활동이다.

다음 장에서 배울 합성은
상속의 단점을 피하면서
코드를 재사용할 수 있는 더 좋은 방법이다.
```

---

## 6. 실전 예제

### 예제 1: 도형 계산 시스템 - 잘못된 상속

#### ❌ Before: 구체 클래스 상속

```java
// 사각형
public class Rectangle {
    protected int width;
    protected int height;
    
    public Rectangle(int width, int height) {
        this.width = width;
        this.height = height;
    }
    
    public int getWidth() { return width; }
    public int getHeight() { return height; }
    
    public void setWidth(int width) {
        this.width = width;
    }
    
    public void setHeight(int height) {
        this.height = height;
    }
    
    public int getArea() {
        return width * height;
    }
}
```

```java
// ❌ 정사각형이 사각형을 상속
public class Square extends Rectangle {
    public Square(int size) {
        super(size, size);
    }
    
    // ❌ 리스코프 치환 원칙 위반
    @Override
    public void setWidth(int width) {
        super.setWidth(width);
        super.setHeight(width);  // 높이도 함께 변경
    }
    
    @Override
    public void setHeight(int height) {
        super.setWidth(height);   // 너비도 함께 변경
        super.setHeight(height);
    }
}
```

**문제 발생:**

```java
Rectangle rect = new Square(5);
rect.setWidth(10);
rect.setHeight(20);

// 정사각형: 20 * 20 = 400
// 사각형: 10 * 20 = 200
System.out.println(rect.getArea());  // 400? 200?

// Rectangle의 클라이언트는
// width와 height를 독립적으로 변경할 수 있다고 기대
// 하지만 Square는 이를 위반!
```

#### ✅ After: 추상화에 의존

```java
// 추상화
public interface Shape {
    int getArea();
    int getPerimeter();
}
```

```java
// 사각형
public class Rectangle implements Shape {
    private int width;
    private int height;
    
    public Rectangle(int width, int height) {
        this.width = width;
        this.height = height;
    }
    
    public void setWidth(int width) {
        this.width = width;
    }
    
    public void setHeight(int height) {
        this.height = height;
    }
    
    @Override
    public int getArea() {
        return width * height;
    }
    
    @Override
    public int getPerimeter() {
        return 2 * (width + height);
    }
}
```

```java
// 정사각형
public class Square implements Shape {
    private int size;
    
    public Square(int size) {
        this.size = size;
    }
    
    public void setSize(int size) {
        this.size = size;
    }
    
    @Override
    public int getArea() {
        return size * size;
    }
    
    @Override
    public int getPerimeter() {
        return 4 * size;
    }
}
```

**비교표:**

| 측면 | Before (구체 클래스 상속) | After (인터페이스 구현) |
|------|------------------------|----------------------|
| **결합도** | Rectangle에 강하게 결합 | Shape에만 의존 |
| **LSP** | 위반 (Square가 Rectangle 대체 불가) | 준수 (각자 독립적) |
| **변경** | Rectangle 변경 시 Square 영향 | 독립적으로 변경 |
| **명확성** | 혼란스러운 상속 관계 | 명확한 인터페이스 |

---

### 예제 2: 직원 관리 시스템 - 타입 코드 vs 상속

#### ❌ Before: 타입 코드 사용

```java
public class Employee {
    enum EmployeeType { REGULAR, MANAGER, SALES }
    
    private EmployeeType type;
    private String name;
    private double salary;
    private double commission;  // SALES만 사용
    private int teamSize;       // MANAGER만 사용
    
    public Employee(EmployeeType type, String name, double salary) {
        this.type = type;
        this.name = name;
        this.salary = salary;
    }
    
    // ❌ 타입 코드로 분기
    public double calculatePay() {
        switch (type) {
            case REGULAR:
                return salary;
            case MANAGER:
                return salary + (salary * 0.1 * teamSize);
            case SALES:
                return salary + commission;
            default:
                throw new IllegalStateException();
        }
    }
    
    // ❌ 타입별로 다른 필드 사용
    public void setCommission(double commission) {
        if (type != EmployeeType.SALES) {
            throw new IllegalStateException();
        }
        this.commission = commission;
    }
}
```

**문제점:**
```
1. 낮은 응집도
   - 모든 타입의 필드가 한 클래스에
   - REGULAR는 commission, teamSize 불필요
   
2. 높은 결합도
   - 새로운 직원 타입 추가 시 calculatePay 수정 필요
   
3. OCP 위반
   - 확장을 위해 수정 필요
```

#### ✅ After: 추상화와 상속

```java
// 추상 클래스
public abstract class Employee {
    private String name;
    private double salary;
    
    public Employee(String name, double salary) {
        this.name = name;
        this.salary = salary;
    }
    
    protected double getSalary() {
        return salary;
    }
    
    // 템플릿 메서드
    public abstract double calculatePay();
}
```

```java
// 일반 직원
public class RegularEmployee extends Employee {
    public RegularEmployee(String name, double salary) {
        super(name, salary);
    }
    
    @Override
    public double calculatePay() {
        return getSalary();
    }
}
```

```java
// 관리자
public class Manager extends Employee {
    private int teamSize;
    
    public Manager(String name, double salary, int teamSize) {
        super(name, salary);
        this.teamSize = teamSize;
    }
    
    public void setTeamSize(int teamSize) {
        this.teamSize = teamSize;
    }
    
    @Override
    public double calculatePay() {
        return getSalary() + (getSalary() * 0.1 * teamSize);
    }
}
```

```java
// 영업 사원
public class SalesEmployee extends Employee {
    private double commission;
    
    public SalesEmployee(String name, double salary) {
        super(name, salary);
        this.commission = 0;
    }
    
    public void setCommission(double commission) {
        this.commission = commission;
    }
    
    @Override
    public double calculatePay() {
        return getSalary() + commission;
    }
}
```

**개선 효과:**

```
1. 높은 응집도
   - 각 직원 타입은 필요한 필드만 가짐
   
2. 낮은 결합도
   - 각 타입이 독립적
   
3. OCP 준수
   - 새로운 직원 타입 추가 시 기존 코드 수정 불필요
   
4. 타입 안전성
   - setCommission은 SalesEmployee만 가짐
```

---

### 예제 3: 컬렉션 래퍼 - 상속의 함정

#### ❌ Before: ArrayList 상속

```java
// ❌ ArrayList를 상속받는 검증 리스트
public class ValidatedList<E> extends ArrayList<E> {
    private Validator<E> validator;
    
    public ValidatedList(Validator<E> validator) {
        this.validator = validator;
    }
    
    @Override
    public boolean add(E e) {
        // 검증 후 추가
        validator.validate(e);
        return super.add(e);
    }
    
    @Override
    public void add(int index, E element) {
        // 검증 후 추가
        validator.validate(element);
        super.add(index, element);
    }
    
    // ❌ addAll은 오버라이드 안 함
}
```

**문제 발생:**

```java
ValidatedList<Integer> list = new ValidatedList<>(
    value -> {
        if (value < 0) throw new IllegalArgumentException();
    }
);

list.add(10);  // ✅ 검증됨
list.add(20);  // ✅ 검증됨

// ❌ addAll은 내부에서 add를 호출하지만
//     검증이 안 될 수도 있음!
list.addAll(Arrays.asList(30, -5, 40));
```

**문제점:**
```
1. ArrayList의 내부 구현에 의존
   - addAll이 add를 호출하는지 확실하지 않음
   
2. 모든 메서드를 오버라이드해야 함
   - set, addAll, subList 등 많은 메서드
   
3. ArrayList 변경 시 영향받음
   - 취약한 기반 클래스 문제
```

#### ✅ After: 합성 사용 (미리보기)

```java
// ✅ 합성을 사용한 검증 리스트
public class ValidatedList<E> implements List<E> {
    private List<E> list = new ArrayList<>();  // 합성!
    private Validator<E> validator;
    
    public ValidatedList(Validator<E> validator) {
        this.validator = validator;
    }
    
    @Override
    public boolean add(E e) {
        validator.validate(e);
        return list.add(e);
    }
    
    @Override
    public boolean addAll(Collection<? extends E> c) {
        // 모두 검증 후 추가
        for (E e : c) {
            validator.validate(e);
        }
        return list.addAll(c);
    }
    
    // 나머지 List 메서드들은 위임
    @Override
    public E get(int index) {
        return list.get(index);
    }
    
    @Override
    public int size() {
        return list.size();
    }
    
    // ... 기타 메서드들
}
```

**합성의 장점:**
```
✅ ArrayList의 내부 구현과 무관
✅ 명시적으로 검증 로직 제어
✅ ArrayList 변경 시에도 안전
✅ 필요한 메서드만 노출 가능
```

---

### 예제 4: 상속 계층의 깊이 문제

#### ❌ Before: 깊은 상속 계층

```java
// 1단계
public class Animal {
    private String name;
    public void eat() { }
    public void sleep() { }
}

// 2단계
public class Mammal extends Animal {
    private int bodyTemperature = 37;
    public void feedMilk() { }
}

// 3단계
public class Carnivore extends Mammal {
    public void hunt() { }
}

// 4단계
public class Feline extends Carnivore {
    public void retractClaws() { }
}

// 5단계
public class Lion extends Feline {
    public void roar() { }
}

// 6단계
public class AfricanLion extends Lion {
    private String habitat = "Savanna";
}
```

**문제점:**

```
1. 이해 어려움
   AfricanLion의 동작을 이해하려면
   6단계 상속 계층을 모두 이해해야 함
   
2. 취약성 증폭
   Animal의 작은 변경이
   AfricanLion까지 영향
   
3. 재사용 어려움
   중간 계층의 불필요한 기능까지 상속
   
4. 경직성
   새로운 분류 추가 어려움
```

#### ✅ After: 얕은 상속 + 인터페이스

```java
// 인터페이스로 기능 정의
public interface Eater {
    void eat();
}

public interface Sleeper {
    void sleep();
}

public interface MilkFeeder {
    void feedMilk();
}

public interface Hunter {
    void hunt();
}

public interface ClawRetractor {
    void retractClaws();
}
```

```java
// 기본 클래스
public abstract class Animal implements Eater, Sleeper {
    private String name;
    
    public Animal(String name) {
        this.name = name;
    }
    
    @Override
    public void eat() {
        System.out.println(name + " is eating");
    }
    
    @Override
    public void sleep() {
        System.out.println(name + " is sleeping");
    }
}
```

```java
// ✅ 얕은 상속 + 필요한 인터페이스만 구현
public class Lion extends Animal 
                  implements MilkFeeder, Hunter, ClawRetractor {
    
    public Lion(String name) {
        super(name);
    }
    
    @Override
    public void feedMilk() {
        System.out.println("Feeding cubs");
    }
    
    @Override
    public void hunt() {
        System.out.println("Hunting prey");
    }
    
    @Override
    public void retractClaws() {
        System.out.println("Retracting claws");
    }
    
    public void roar() {
        System.out.println("ROAR!");
    }
}
```

**개선 효과:**

| 측면 | Before (깊은 상속) | After (얕은 상속 + 인터페이스) |
|------|------------------|----------------------------|
| **계층 깊이** | 6단계 | 2단계 |
| **이해도** | 모든 조상 클래스 이해 필요 | Animal만 이해하면 됨 |
| **유연성** | 경직됨 | 유연함 (인터페이스 조합) |
| **재사용성** | 불필요한 기능까지 상속 | 필요한 기능만 선택 |
| **변경 영향** | 전체 계층에 파급 | 최소화 |

---

## 7. 상속 설계 가이드

### 1. 상속을 사용하기 전 체크리스트

```
□ is-a 관계가 명확한가?
  예: RegularPhone is-a Phone ✅
  반례: Stack is-a Vector ❌
  
□ 부모 클래스가 진정한 추상화인가?
  ✅ abstract class Phone
  ❌ class Vector (구체 클래스)
  
□ 리스코프 치환 원칙을 만족하는가?
  자식 클래스가 부모 클래스를 완벽히 대체할 수 있는가?
  
□ 부모의 모든 퍼블릭 인터페이스가 자식에게도 적절한가?
  예: Stack은 Vector의 add(int, E) 불필요 ❌
  
□ 상속 계층이 3단계 이하인가?
  깊은 상속은 이해하기 어렵고 취약함
```

### 2. 상속 vs 합성 선택 기준

#### 상속을 사용해야 하는 경우

```
✅ is-a 관계가 명확
   RegularPhone is-a Phone
   
✅ 부모가 추상 클래스
   abstract class Phone
   
✅ 템플릿 메서드 패턴
   전체 흐름은 부모, 세부사항은 자식
   
✅ 다형성이 필수
   Phone[] phones = {regular, nightly, weekend};
```

#### 합성을 사용해야 하는 경우

```
✅ has-a 관계
   Car has-a Engine
   
✅ 구체 클래스 재사용
   ArrayList, HashMap 등
   
✅ 런타임에 동작 변경 필요
   전략 패턴, 상태 패턴
   
✅ 다중 상속 필요
   여러 기능 조합
```

### 3. 안전한 상속 설계 패턴

#### Template Method Pattern

```java
public abstract class Game {
    // 템플릿 메서드 (final로 오버라이드 방지)
    public final void play() {
        initialize();
        startPlay();
        endPlay();
    }
    
    // 훅 메서드들
    protected abstract void initialize();
    protected abstract void startPlay();
    protected abstract void endPlay();
}

public class Chess extends Game {
    @Override
    protected void initialize() {
        System.out.println("Chess Game Initialized");
    }
    
    @Override
    protected void startPlay() {
        System.out.println("Chess Game Started");
    }
    
    @Override
    protected void endPlay() {
        System.out.println("Chess Game Finished");
    }
}
```

**장점:**
- 전체 흐름은 부모가 제어
- 자식은 세부사항만 구현
- super 호출 불필요

---

### 4. 리팩토링 가이드

#### Step 1: 중복 코드 식별

```
변경 시 함께 수정해야 하는 코드를 찾는다.
모양이 같은 코드가 아니라
변경 이유가 같은 코드를 찾는 것!
```

#### Step 2: 차이점 메서드로 추출

```java
// Before
public Money calculateFee() {
    Money result = Money.ZERO;
    for(Call call : calls) {
        result = result.plus(
            amount.times(call.getDuration().getSeconds() / seconds.getSeconds())
        );
    }
    return result;
}

// After
public Money calculateFee() {
    Money result = Money.ZERO;
    for(Call call : calls) {
        result = result.plus(calculateCallFee(call));  // 추출!
    }
    return result;
}

protected Money calculateCallFee(Call call) {
    return amount.times(call.getDuration().getSeconds() / seconds.getSeconds());
}
```

#### Step 3: 공통 코드를 부모로 이동

```java
// 1. 추상 클래스 생성
public abstract class Phone {
    public Money calculateFee() {
        // 공통 로직
    }
    protected abstract Money calculateCallFee(Call call);
}

// 2. 자식 클래스들은 차이점만 구현
public class RegularPhone extends Phone {
    @Override
    protected Money calculateCallFee(Call call) {
        // RegularPhone만의 로직
    }
}
```

#### Step 4: 추상화 개선

```
- 의도를 드러내는 이름으로 변경
- 불필요한 메서드 제거
- 접근 제어자 최적화 (protected)
```

---

## 🔗 연결고리

### 이전 장과의 연결
- **Chapter 09**: 의존성 관리 원칙 → 상속에서의 의존성 문제
- DIP (의존성 역전 원칙) → 추상화에 의존하는 상속
- OCP (개방-폐쇄 원칙) → 템플릿 메서드 패턴

### 다음 장 예고
- **Chapter 11**: 합성과 유연한 설계
  - 상속의 대안: 합성
  - 합성의 장점과 사용법
  - 상속 vs 합성 선택 기준

---

<div align="center">

**[⬆ 목차로 돌아가기](../README.md)**

</div>

<div align="center">

**[⬅️ 이전: Chapter 09](../chapter09/README.md) | [다음: Chapter 11 ➡️](../chapter11/README.md)**

</div>
