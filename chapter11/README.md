# Chapter 11. 합성과 유연한 설계

> *"객체 합성이 클래스 상속보다 더 좋은 방법이다"*

## 📌 핵심 개념

이 장에서는 **합성(Composition)의 힘**을 다룹니다. 상속의 문제점을 해결하고 진정으로 유연한 설계를 만드는 방법은 합성을 사용하는 것입니다.

### 🎯 학습 목표

- 상속의 세 가지 문제를 합성으로 해결하기
- 클래스 폭발 문제와 그 해결책 이해하기
- 합성을 통한 런타임 조합의 강력함 체험하기
- 믹스인(Mixin)의 개념과 활용 이해하기
- 상속 vs 합성의 올바른 선택 기준 파악하기

---

## 📖 목차

1. [상속을 합성으로 변경하기](#1-상속을-합성으로-변경하기)
2. [상속으로 인한 조합의 폭발적인 증가](#2-상속으로-인한-조합의-폭발적인-증가)
3. [합성 관계로 변경하기](#3-합성-관계로-변경하기)
4. [믹스인](#4-믹스인)
5. [핵심 정리](#5-핵심-정리)

---

## 1. 상속을 합성으로 변경하기

> 📂 **코드**: [`Stack.java`](https://github.com/eternity-oop/object/blob/master/chapter11/src/main/java/org/eternity/stack/Stack.java) | [`InstrumentedHashSet.java`](https://github.com/eternity-oop/object/blob/master/chapter11/src/main/java/org/eternity/instrumented/InstrumentedHashSet.java) | [`PersonalPlaylist.java`](https://github.com/eternity-oop/object/blob/master/chapter11/src/main/java/org/eternity/playlist/PersonalPlaylist.java)

### 1.1 상속 vs 합성

#### 📊 비교표

| 측면 | 상속 (Inheritance) | 합성 (Composition) |
|------|-------------------|-------------------|
| **관계** | is-a 관계 | has-a 관계 |
| **의존성 시점** | 컴파일타임 | 런타임 |
| **재사용 대상** | 구현 (코드 자체) | 인터페이스 (퍼블릭 인터페이스) |
| **결합도** | 높음 (부모의 내부 구현에 의존) | 낮음 (퍼블릭 인터페이스에만 의존) |
| **유연성** | 정적 (컴파일타임 고정) | 동적 (런타임 변경 가능) |
| **재사용 방식** | 화이트박스 재사용 | 블랙박스 재사용 |

#### 💡 핵심 통찰

```
상속:
- 부모 클래스의 내부를 알아야 함 (화이트박스)
- 컴파일타임에 관계 고정
- 변경에 취약

합성:
- 내부를 몰라도 됨 (블랙박스)
- 런타임에 관계 변경 가능
- 변경에 유연
```

### 1.2 합성으로 변경하는 방법

#### 🔄 변경 단계

```
1. 자식 클래스에 선언된 상속 관계 제거
   extends ParentClass → 제거
   
2. 부모 클래스를 인스턴스 변수로 선언
   private ParentClass instance;
   
3. 필요한 메서드는 위임(forwarding)으로 구현
   public void method() {
       instance.method();
   }
```

### 1.3 문제 1: 불필요한 인터페이스 상속

#### ❌ Before: Stack이 Vector를 상속

```java
public class Stack<E> extends Vector<E> {
    public E push(E item) {
        addElement(item);
        return item;
    }
    
    public E pop() {
        if (isEmpty()) {
            throw new EmptyStackException();
        }
        return remove(size() - 1);
    }
}
```

**문제점:**

```java
Stack<String> stack = new Stack<>();
stack.push("1st");
stack.push("2nd");
stack.push("3rd");

// ❌ Vector의 메서드로 Stack 규칙 위반!
stack.add(0, "4th");

// Stack의 LIFO 규칙이 깨짐
assertEquals("3rd", stack.pop());  // 실패!
// 실제로는 "4th"가 나옴
```

**문제 분석:**

```
Vector의 불필요한 메서드들이 Stack의 인터페이스를 오염시킴:
- add(int index, E element)  ← 임의 위치 추가
- get(int index)              ← 임의 위치 조회
- remove(int index)           ← 임의 위치 삭제

→ Stack의 LIFO 규칙을 쉽게 위반할 수 있음
```

#### ✅ After: 합성 사용

```java
public class Stack<E> {
    private Vector<E> elements = new Vector<>();  // 합성!
    
    public E push(E item) {
        elements.addElement(item);
        return item;
    }
    
    public E pop() {
        if (elements.isEmpty()) {
            throw new EmptyStackException();
        }
        return elements.remove(elements.size() - 1);
    }
}
```

**개선 효과:**

```
✅ Stack의 퍼블릭 인터페이스가 깔끔해짐
   - push(), pop()만 노출
   
✅ Vector의 불필요한 메서드 숨김
   - add(), get(), remove() 접근 불가
   
✅ Stack의 LIFO 규칙 보장
   - 임의 위치 접근 차단
```

### 1.4 문제 2: 메서드 오버라이딩의 오작용

#### ❌ Before: InstrumentedHashSet의 상속

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
        addCount += c.size();
        return super.addAll(c);
    }
    
    public int getAddCount() {
        return addCount;
    }
}
```

**문제 발생:**

```java
InstrumentedHashSet<String> set = new InstrumentedHashSet<>();
set.addAll(Arrays.asList("Java", "Ruby", "Scala"));

System.out.println(set.getAddCount());
// 예상: 3
// 실제: 6  ❌
```

**원인 분석:**

```
HashSet의 내부 구현:

public boolean addAll(Collection<? extends E> c) {
    boolean modified = false;
    for (E e : c) {
        if (add(e))  // ← InstrumentedHashSet.add() 호출!
            modified = true;
    }
    return modified;
}

호출 흐름:
1. addAll(3개) → addCount += 3  (addCount = 3)
2. super.addAll() 호출
3. HashSet.addAll()에서 add() 3번 호출
   → InstrumentedHashSet.add() 호출됨! (오버라이드)
   → addCount++ 3번  (addCount = 6)
```

#### ✅ After: 합성 사용 (기본)

```java
public class InstrumentedHashSet<E> {
    private int addCount = 0;
    private Set<E> set;  // 합성!
    
    public InstrumentedHashSet(Set<E> set) {
        this.set = set;
    }
    
    public boolean add(E e) {
        addCount++;
        return set.add(e);
    }
    
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return set.addAll(c);
    }
    
    public int getAddCount() {
        return addCount;
    }
}
```

**개선 효과:**

```
✅ HashSet의 내부 구현과 무관
   - addAll이 add를 호출하는지 몰라도 됨
   
✅ 정확한 카운팅
   - addCount = 3 (정확)
   
✅ 구현 결합도 제거
   - HashSet 변경에 영향받지 않음
```

#### ✅ After: 포워딩 메서드로 인터페이스 유지

```java
public class InstrumentedHashSet<E> implements Set<E> {
    private int addCount = 0;
    private Set<E> set;
    
    public InstrumentedHashSet(Set<E> set) {
        this.set = set;
    }
    
    @Override
    public boolean add(E e) {
        addCount++;
        return set.add(e);
    }
    
    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return set.addAll(c);
    }
    
    public int getAddCount() {
        return addCount;
    }
    
    // ✅ 포워딩 메서드들
    @Override public boolean remove(Object o) {
        return set.remove(o);
    }
    
    @Override public void clear() {
        set.clear();
    }
    
    @Override public boolean equals(Object o) {
        return set.equals(o);
    }
    
    @Override public int hashCode() {
        return set.hashCode();
    }
    
    @Override public int size() {
        return set.size();
    }
    
    @Override public boolean isEmpty() {
        return set.isEmpty();
    }
    
    @Override public boolean contains(Object o) {
        return set.contains(o);
    }
    
    @Override public Iterator<E> iterator() {
        return set.iterator();
    }
    
    @Override public Object[] toArray() {
        return set.toArray();
    }
    
    @Override public <T> T[] toArray(T[] a) {
        return set.toArray(a);
    }
    
    @Override public boolean containsAll(Collection<?> c) {
        return set.containsAll(c);
    }
    
    @Override public boolean retainAll(Collection<?> c) {
        return set.retainAll(c);
    }
    
    @Override public boolean removeAll(Collection<?> c) {
        return set.removeAll(c);
    }
}
```

#### 💡 포워딩(Forwarding)이란?

**정의:**
```
동일한 메서드 호출을 내부 객체에게 그대로 전달하는 것

포워딩 메서드(Forwarding Method):
동일한 메서드를 호출하기 위해 추가된 메서드
```

**장점:**
```
✅ 기존 클래스의 인터페이스를 그대로 제공
✅ 구현에 대한 결합 없음
✅ 일부 작동 방식만 변경 가능
```

**활용:**
```
기존 클래스의 인터페이스를 그대로 제공하면서
구현에 대한 결합 없이
일부 작동 방식을 변경하고 싶을 때 사용
```

### 1.5 문제 3: 부모-자식 동시 수정

#### ❌ Before: Playlist 상속

```java
public class Playlist {
    private List<Song> tracks = new ArrayList<>();
    private Map<String, String> singers = new HashMap<>();
    
    public void append(Song song) {
        tracks.add(song);
        singers.put(song.getSinger(), song.getTitle());
    }
    
    public List<Song> getTracks() {
        return tracks;
    }
    
    public Map<String, String> getSingers() {
        return singers;
    }
}
```

```java
public class PersonalPlaylist extends Playlist {
    public void remove(Song song) {
        getTracks().remove(song);
        getSingers().remove(song.getSinger());
    }
}
```

**문제점:**

```
Playlist의 내부 구현 변경 시
PersonalPlaylist도 함께 수정 필요

예: Playlist에 새로운 컬렉션 추가 시
    PersonalPlaylist.remove()도 수정 필요
```

#### ✅ After: 합성 사용

```java
public class PersonalPlaylist {
    private Playlist playlist = new Playlist();  // 합성!
    
    public void append(Song song) {
        playlist.append(song);
    }
    
    public void remove(Song song) {
        playlist.getTracks().remove(song);
        playlist.getSingers().remove(song.getSinger());
    }
}
```

**여전히 남은 문제:**

```
합성으로 변경해도
Playlist와 PersonalPlaylist를 함께 수정해야 하는
문제는 완전히 해결되지 않음
```

**그래도 합성이 나은 이유:**

```
✅ 파급효과를 PersonalPlaylist 내부로 캡슐화
   - Playlist 변경이 외부로 퍼지지 않음
   
✅ 향후 Playlist의 내부 구현 변경 시
   - 영향 범위가 PersonalPlaylist로 제한됨
```

### 1.6 핵심 원칙

```
┌─────────────────────────────────────────────────────┐
│  대부분의 경우                                         │
│  구현에 대한 결합보다는                                   │
│  인터페이스에 대한 결합이 더 좋다.                          │
│                                                     │
│  합성은 인터페이스에 대한 결합을 제공한다.                    │
└─────────────────────────────────────────────────────┘
```

---

## 2. 상속으로 인한 조합의 폭발적인 증가

> 📂 **코드**: [`Phone.java (step01-04)`](https://github.com/eternity-oop/object/blob/master/chapter11/src/main/java/org/eternity/billing/step01/Phone.java)

### 2.1 클래스 폭발 문제

#### 🎯 Class Explosion Problem

**정의:**
```
상속의 남용으로
하나의 기능을 추가하기 위해
필요 이상으로 많은 수의 클래스를 추가해야 하는 경우

= 조합의 폭발 (Combinational Explosion)
```

**발생 원인:**

```
1. 상속 관계는 컴파일타임에 고정
   - 런타임에 변경 불가
   
2. 다양한 조합 필요 시
   - 조합의 수만큼 클래스 추가 필요
   
3. 자식 클래스가 부모의 구현에 강하게 결합
   - 상속의 근본적인 한계
```

### 2.2 요구사항: 과금 시스템 확장

#### 📱 기존 시스템

```
기본 정책 (Basic Policy):
1. 일반 요금제 (RegularPhone)
   - 단위 시간당 고정 요금
   
2. 심야 할인 요금제 (NightlyDiscountPhone)
   - 심야 시간대 할인 요금
```

#### 📊 새로운 요구사항

```
부가 정책 (Additional Policy):
1. 세금 정책 (TaxPolicy)
   - 계산된 요금에 세금 부과
   
2. 기본 요금 할인 정책 (RateDiscountPolicy)
   - 계산된 요금에서 고정 금액 할인
```

#### 🎯 부가 정책의 특성

```
1. 기본 정책의 계산 결과에 적용
   - 기본 정책 먼저, 부가 정책은 나중
   
2. 선택적 적용 가능
   - 부가 정책 없이도 사용 가능
   - 하나만 적용 가능
   - 둘 다 적용 가능
   
3. 조합 가능
   - 세금 정책만
   - 할인 정책만
   - 세금 + 할인
   - 할인 + 세금
   
4. 임의의 순서로 적용 가능
   - 세금 → 할인
   - 할인 → 세금
```

#### 📈 조합 가능한 경우의 수

```
기본 정책: 2개
- RegularPhone
- NightlyDiscountPhone

부가 정책: 4가지 조합
- 없음
- 세금만
- 할인만
- 세금 + 할인 (순서 2가지)

총 경우의 수: 2 × 6 = 12가지
```

### 2.3 상속을 이용한 구현

#### 📝 기본 정책 (10장 복습)

```java
public abstract class Phone {
    private List<Call> calls = new ArrayList<>();
    
    public Money calculateFee() {
        Money result = Money.ZERO;
        
        for(Call call : calls) {
            result = result.plus(calculateCallFee(call));
        }
        
        return result;
    }
    
    protected abstract Money calculateCallFee(Call call);
}
```

```java
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
```

```java
public class NightlyDiscountPhone extends Phone {
    private static final int LATE_NIGHT_HOUR = 22;
    
    private Money nightlyAmount;
    private Money regularAmount;
    private Duration seconds;
    
    public NightlyDiscountPhone(Money nightlyAmount, Money regularAmount, 
                                Duration seconds) {
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

### 2.4 부가 정책 추가: 첫 번째 시도

#### ❌ super 호출 방식

```java
public class TaxableRegularPhone extends RegularPhone {
    private double taxRate;
    
    public TaxableRegularPhone(Money amount, Duration seconds, double taxRate) {
        super(amount, seconds);
        this.taxRate = taxRate;
    }
    
    @Override
    public Money calculateFee() {
        // ❌ super 호출로 부모에 강하게 결합
        Money fee = super.calculateFee();
        return fee.plus(fee.times(taxRate));
    }
}
```

**문제점:**

```
❌ 부모 클래스와 강한 결합
   - 부모의 calculateFee() 구현에 의존
   
❌ 결합도 증가
   - 부모 변경 시 자식도 영향받음
```

### 2.5 부가 정책 추가: 훅 메서드 방식

#### ✅ 추상 메서드로 결합도 낮추기

**Step 1: Phone에 훅 메서드 추가**

```java
public abstract class Phone {
    private List<Call> calls = new ArrayList<>();
    
    public Money calculateFee() {
        Money result = Money.ZERO;
        
        for(Call call : calls) {
            result = result.plus(calculateCallFee(call));
        }
        
        // ✅ 훅 메서드 호출
        return afterCalculated(result);
    }
    
    protected abstract Money calculateCallFee(Call call);
    
    // ✅ 훅 메서드: 기본 구현 제공
    protected Money afterCalculated(Money fee) {
        return fee;
    }
}
```

**훅 메서드 (Hook Method):**

```
추상 메서드와 동일하게 오버라이딩할 의도로
메서드를 추가했지만
편의를 위해 기본 구현을 제공하는 메서드

장점:
- 모든 자식 클래스가 구현하지 않아도 됨
- 필요한 자식만 오버라이드
- 중복 코드 방지
```

**Step 2: 기본 정책 클래스들**

```java
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
    
    // afterCalculated는 오버라이드 불필요 (기본 구현 사용)
}
```

```java
public class NightlyDiscountPhone extends Phone {
    // ... 필드 생략
    
    @Override
    protected Money calculateCallFee(Call call) {
        if (call.getFrom().getHour() >= LATE_NIGHT_HOUR) {
            return nightlyAmount.times(call.getDuration().getSeconds() / seconds.getSeconds());
        } else {
            return regularAmount.times(call.getDuration().getSeconds() / seconds.getSeconds());
        }
    }
    
    // afterCalculated는 오버라이드 불필요 (기본 구현 사용)
}
```

**Step 3: 세금 정책 적용**

```java
public class TaxableRegularPhone extends RegularPhone {
    private double taxRate;
    
    public TaxableRegularPhone(Money amount, Duration seconds, double taxRate) {
        super(amount, seconds);
        this.taxRate = taxRate;
    }
    
    @Override
    protected Money afterCalculated(Money fee) {
        return fee.plus(fee.times(taxRate));
    }
}
```

```java
public class TaxableNightlyDiscountPhone extends NightlyDiscountPhone {
    private double taxRate;
    
    public TaxableNightlyDiscountPhone(Money nightlyAmount, Money regularAmount, 
                                       Duration seconds, double taxRate) {
        super(nightlyAmount, regularAmount, seconds);
        this.taxRate = taxRate;
    }
    
    @Override
    protected Money afterCalculated(Money fee) {
        return fee.plus(fee.times(taxRate));
    }
}
```

**문제 발견:**

```
❌ 코드 중복!
   TaxableRegularPhone과 TaxableNightlyDiscountPhone의
   afterCalculated() 메서드가 동일함
   
원인:
   자바는 단일 상속만 지원
   → 세금 정책을 별도 클래스로 분리 불가
```

#### 📊 상속 계층 (세금 정책만)

```
         Phone (abstract)
           ↑   ↑
           │   │
   ┌───────┘   └───────┐
   │                   │
RegularPhone    NightlyDiscountPhone
   ↑                   ↑
   │                   │
   │                   │
TaxableRegularPhone  TaxableNightlyDiscountPhone
```

### 2.6 기본 요금 할인 정책 추가

```java
public class RateDiscountableRegularPhone extends RegularPhone {
    private Money discountAmount;
    
    public RateDiscountableRegularPhone(Money amount, Duration seconds, 
                                        Money discountAmount) {
        super(amount, seconds);
        this.discountAmount = discountAmount;
    }
    
    @Override
    protected Money afterCalculated(Money fee) {
        return fee.minus(discountAmount);
    }
}
```

```java
public class RateDiscountableNightlyDiscountPhone extends NightlyDiscountPhone {
    private Money discountAmount;
    
    public RateDiscountableNightlyDiscountPhone(Money nightlyAmount, 
                                                Money regularAmount, 
                                                Duration seconds, 
                                                Money discountAmount) {
        super(nightlyAmount, regularAmount, seconds);
        this.discountAmount = discountAmount;
    }
    
    @Override
    protected Money afterCalculated(Money fee) {
        return fee.minus(discountAmount);
    }
}
```

**또 다시 중복:**

```
❌ RateDiscountableRegularPhone과
   RateDiscountableNightlyDiscountPhone의
   afterCalculated() 메서드가 동일함
```

#### 📊 상속 계층 (세금 + 할인)

```
                    Phone (abstract)
                      ↑   ↑
                      │   │
            ┌─────────┘   └─────────┐
            │                       │
       RegularPhone          NightlyDiscountPhone
         ↑   ↑                   ↑   ↑
         │   │                   │   │
    ┌────┘   └────┐         ┌────┘   └────┐
    │             │         │             │
Taxable    RateDiscountable   Taxable    RateDiscountable
Regular         Regular     Nightly         Nightly
Phone           Phone       Phone           Phone

중복 코드 2개 발생!
```

### 2.7 중복의 덫: 조합 추가

#### 💥 두 부가 정책을 함께 적용

**세금 → 할인 순서:**

```java
public class TaxableAndRateDiscountableRegularPhone extends TaxableRegularPhone {
    private Money discountAmount;
    
    public TaxableAndRateDiscountableRegularPhone(Money amount, Duration seconds, 
                                                   double taxRate, 
                                                   Money discountAmount) {
        super(amount, seconds, taxRate);
        this.discountAmount = discountAmount;
    }
    
    @Override
    protected Money afterCalculated(Money fee) {
        // 세금 먼저 (부모의 afterCalculated)
        return super.afterCalculated(fee).minus(discountAmount);
    }
}
```

**할인 → 세금 순서:**

```java
public class RateDiscountableAndTaxableRegularPhone 
        extends RateDiscountableRegularPhone {
    private double taxRate;
    
    public RateDiscountableAndTaxableRegularPhone(Money amount, Duration seconds, 
                                                   Money discountAmount, 
                                                   double taxRate) {
        super(amount, seconds, discountAmount);
        this.taxRate = taxRate;
    }
    
    @Override
    protected Money afterCalculated(Money fee) {
        // 할인 먼저 (부모의 afterCalculated)
        return super.afterCalculated(fee).plus(fee.times(taxRate));
    }
}
```

**심야 할인 요금제에도 동일하게:**

```java
public class TaxableAndRateDiscountableNightlyDiscountPhone 
        extends TaxableNightlyDiscountPhone {
    // ... 동일한 패턴
}

public class RateDiscountableAndTaxableNightlyDiscountPhone 
        extends RateDiscountableNightlyDiscountPhone {
    // ... 동일한 패턴
}
```

#### 📊 최종 상속 계층

```
                         Phone (abstract)
                           ↑   ↑
                           │   │
                 ┌─────────┘   └─────────┐
                 │                       │
            RegularPhone          NightlyDiscountPhone
              ↑   ↑                   ↑   ↑
              │   │                   │   │
         ┌────┘   └────┐         ┌────┘   └────┐
         │             │         │             │
    Taxable      RateDiscount  Taxable    RateDiscount
    Regular         Regular   Nightly       Nightly
      ↑               ↑         ↑             ↑
      │               │         │             │
  ┌───┴───┐       ┌───┴───┐ ┌───┴───┐     ┌───┴───┐
  │       │       │       │ │       │     │       │
세금→할인  할인→세금  세금→할인  할인→세금  (4개 더)

총 클래스: 1 + 2 + 4 + 8 = 15개
```

### 2.8 새로운 정책 추가 시 폭발

#### 🆕 고정 요금제 추가

```
고정 요금제 (FixedFeePhone):
- 월 정액 요금

추가해야 할 클래스:
1. FixedFeePhone
2. TaxableFixedFeePhone
3. RateDiscountableFixedFeePhone
4. TaxableAndRateDiscountableFixedFeePhone
5. RateDiscountableAndTaxableFixedFeePhone

→ 5개 클래스 추가!
```

#### 🆕 약정 할인 정책 추가

```
약정 할인 정책 (ContractDiscountPolicy):
- 약정 기간에 따른 추가 할인

기존 기본 정책: 3개 (Regular, Nightly, Fixed)
기존 부가 정책 조합: 6가지
새로운 조합: 3 × (6 + 6) = 36가지

→ 더 많은 클래스 필요!
```

#### 📊 클래스 폭발 시각화

```
기본 정책 수를 B, 부가 정책 수를 A라고 할 때:

필요한 클래스 수 = B × 2^A

예:
B = 2, A = 2: 2 × 4 = 8개
B = 3, A = 2: 3 × 4 = 12개
B = 3, A = 3: 3 × 8 = 24개
B = 3, A = 4: 3 × 16 = 48개

기하급수적 증가!
```

### 2.9 클래스 폭발의 문제점

#### 💥 문제 1: 클래스 수 폭발

```
❌ 새로운 기능 추가 시
   - 불필요하게 많은 클래스 추가 필요
   
❌ 관리 어려움
   - 클래스 파일이 너무 많음
   - 어떤 클래스가 어떤 조합인지 파악 어려움
```

#### 💥 문제 2: 중복 코드 증가

```
❌ 동일한 로직이 여러 클래스에 중복
   - TaxableRegularPhone
   - TaxableNightlyDiscountPhone
   - TaxableFixedFeePhone
   → 세금 계산 로직 3번 중복
   
❌ 수정 시 모든 곳을 수정해야 함
   - 하나라도 누락하면 버그
```

#### 💥 문제 3: 단일 책임 원칙 위반

```
❌ 하나의 변경 이유에 여러 클래스 수정
   - 세금 정책 변경 시
   - 모든 Taxable* 클래스 수정 필요
```

### 2.10 핵심 원인

```
┌─────────────────────────────────────────────────────┐
│  클래스 폭발 문제의 근본 원인:                             │
│                                                     │
│  자식 클래스가 부모 클래스의 구현에                         │
│  강하게 결합되도록 강요하는                                │
│  상속의 근본적인 한계                                    │
│                                                     │
│  컴파일타임에 결정된                                     │
│  부모-자식 관계는 변경 불가                               │
│                                                     │
│  → 조합의 수만큼 클래스 추가 필요                          │
└─────────────────────────────────────────────────────┘
```

### 2.11 해결책 예고

```
최선의 방법은 상속을 포기하는 것이다.

합성을 사용하면:
✅ 컴파일타임 관계 → 런타임 관계
✅ 클래스 폭발 문제 해결
✅ 중복 코드 제거
✅ 유연한 조합
```

---

## 3. 합성 관계로 변경하기

> 📂 **코드**: [`Phone.java (step05)`](https://github.com/eternity-oop/object/blob/master/chapter11/src/main/java/org/eternity/billing/step05/Phone.java) | [`RatePolicy.java`](https://github.com/eternity-oop/object/blob/master/chapter11/src/main/java/org/eternity/billing/step05/RatePolicy.java)

### 3.1 핵심 아이디어

#### 💡 컴파일타임 → 런타임

**상속의 문제:**
```
컴파일타임에 관계 고정
→ 여러 기능 조합 필요 시
→ 조합 수만큼 클래스 추가
```

**합성의 해결:**
```
컴파일타임 관계를 런타임 관계로 변경
→ 실행 시점에 객체 조합
→ 필요한 조합을 동적으로 생성
```

#### 📊 설계 전략 비교

| 측면 | 상속 | 합성 |
|------|------|------|
| **조합 방식** | 클래스 안에 밀어넣기 | 실행 시점에 조립 |
| **클래스 수** | 조합 수만큼 | 구성 요소 수만큼 |
| **의존성** | 컴파일타임 고정 | 런타임 변경 가능 |
| **유연성** | 낮음 | 높음 |

### 3.2 기본 정책 합성하기

#### 🎯 인터페이스 정의

```java
public interface RatePolicy {
    Money calculateFee(Phone phone);
}
```

**역할:**
```
기본 정책과 부가 정책을 포괄하는 인터페이스

목적:
- Phone이 구체적인 정책을 몰라도 됨
- 다양한 정책과 협력 가능
```

#### 🏗️ 기본 정책 추상 클래스

```java
public abstract class BasicRatePolicy implements RatePolicy {
    @Override
    public Money calculateFee(Phone phone) {
        Money result = Money.ZERO;
        
        for(Call call : phone.getCalls()) {
            result = result.plus(calculateCallFee(call));
        }
        
        return result;
    }
    
    protected abstract Money calculateCallFee(Call call);
}
```

**역할:**
```
기본 정책들의 공통 로직 제공

공통점:
- 전체 통화 목록 순회
- 개별 통화 요금 합산

차이점:
- 개별 통화 요금 계산 방식
- → calculateCallFee()를 추상 메서드로
```

#### 📱 구체적인 기본 정책들

**일반 요금제:**

```java
public class RegularPolicy extends BasicRatePolicy {
    private Money amount;
    private Duration seconds;
    
    public RegularPolicy(Money amount, Duration seconds) {
        this.amount = amount;
        this.seconds = seconds;
    }
    
    @Override
    protected Money calculateCallFee(Call call) {
        return amount.times(call.getDuration().getSeconds() / seconds.getSeconds());
    }
}
```

**심야 할인 요금제:**

```java
public class NightlyDiscountPolicy extends BasicRatePolicy {
    private static final int LATE_NIGHT_HOUR = 22;
    
    private Money nightlyAmount;
    private Money regularAmount;
    private Duration seconds;
    
    public NightlyDiscountPolicy(Money nightlyAmount, Money regularAmount, 
                                 Duration seconds) {
        this.nightlyAmount = nightlyAmount;
        this.regularAmount = regularAmount;
        this.seconds = seconds;
    }
    
    @Override
    protected Money calculateCallFee(Call call) {
        if (call.getFrom().getHour() >= LATE_NIGHT_HOUR) {
            return nightlyAmount.times(call.getDuration().getSeconds() / seconds.getSeconds());
        }
        
        return regularAmount.times(call.getDuration().getSeconds() / seconds.getSeconds());
    }
}
```

#### 📞 Phone 클래스

```java
public class Phone {
    private RatePolicy ratePolicy;  // ✅ 합성!
    private List<Call> calls = new ArrayList<>();
    
    public Phone(RatePolicy ratePolicy) {
        this.ratePolicy = ratePolicy;
    }
    
    public List<Call> getCalls() {
        return Collections.unmodifiableList(calls);
    }
    
    public Money calculateFee() {
        return ratePolicy.calculateFee(this);
    }
}
```

**핵심:**
```
✅ Phone 내부에 RatePolicy 참조
   → 이것이 합성!
   
✅ 인터페이스 타입으로 선언
   → 다양한 정책과 협력 가능
   
✅ 생성자로 의존성 주입
   → 런타임에 구체적인 정책 결정
```

#### 🔧 사용 예시

**일반 요금제:**

```java
Phone phone = new Phone(
    new RegularPolicy(Money.wons(10), Duration.ofSeconds(10))
);
```

**심야 할인 요금제:**

```java
Phone phone = new Phone(
    new NightlyDiscountPolicy(
        Money.wons(5),    // 심야 요금
        Money.wons(10),   // 일반 요금
        Duration.ofSeconds(10)
    )
);
```

#### 📊 클래스 다이어그램

```
┌────────────────┐
│    Phone       │
├────────────────┤
│ - ratePolicy   │──────────▶ «interface»
│ - calls        │            RatePolicy
└────────────────┘            ────────────
                              calculateFee()
                                   △
                                   │
                    ┌──────────────┴──────────────┐
                    │                             │
            BasicRatePolicy              AdditionalRatePolicy
            (abstract)                   (abstract)
                    △                             △
                    │                             │
         ┌──────────┴──────────┐                  │
         │                     │                  │
    RegularPolicy    NightlyDiscountPolicy        │
                                                  │
                                   ┌──────────────┴──────────────┐
                                   │                             │
                            TaxablePolicy              RateDiscountablePolicy
```

### 3.3 부가 정책 적용하기

#### 🎯 부가 정책의 요구사항

```
1. 기본 정책이나 다른 부가 정책의 인스턴스를 참조
   → 어떤 정책과도 합성 가능
   
2. Phone 입장에서는 구분 불가
   → 기본 정책인지 부가 정책인지 몰라야 함
   → 동일한 역할 수행
```

**해결책:**

```
✅ RatePolicy 인터페이스 구현
   → Phone과 동일한 방식으로 협력
   
✅ 내부에 RatePolicy 인스턴스 포함
   → 다른 정책과 조합 가능
```

#### 🏗️ 부가 정책 추상 클래스

```java
public abstract class AdditionalRatePolicy implements RatePolicy {
    private RatePolicy next;  // ✅ 다음 정책
    
    public AdditionalRatePolicy(RatePolicy next) {
        this.next = next;
    }
    
    @Override
    public Money calculateFee(Phone phone) {
        // 1. 다음 정책의 요금 계산
        Money fee = next.calculateFee(phone);
        
        // 2. 부가 정책 적용
        return afterCalculated(fee);
    }
    
    protected abstract Money afterCalculated(Money fee);
}
```

**핵심 메커니즘:**

```
1. next가 참조하는 정책에게 위임
   → 기본 정책이든 부가 정책이든 상관없음
   
2. 반환된 요금에 부가 정책 적용
   → afterCalculated()로 추가 처리
   
3. 자식 클래스는 afterCalculated만 구현
   → 자신만의 부가 정책 로직
```

#### 💰 세금 정책

```java
public class TaxablePolicy extends AdditionalRatePolicy {
    private double taxRate;
    
    public TaxablePolicy(double taxRate, RatePolicy next) {
        super(next);
        this.taxRate = taxRate;
    }
    
    @Override
    protected Money afterCalculated(Money fee) {
        return fee.plus(fee.times(taxRate));
    }
}
```

#### 💵 기본 요금 할인 정책

```java
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

### 3.4 기본 정책과 부가 정책 합성하기

#### 🎯 조합 예시들

**예시 1: 일반 요금제 + 세금**

```java
Phone phone = new Phone(
    new TaxablePolicy(0.05,  // 5% 세금
        new RegularPolicy(
            Money.wons(100), 
            Duration.ofSeconds(10)
        )
    )
);
```

**실행 흐름:**

```
1. phone.calculateFee() 호출
   ↓
2. TaxablePolicy.calculateFee() 호출
   ↓
3. RegularPolicy.calculateFee() 호출 (next)
   → 기본 요금 계산: 1000원
   ↓
4. TaxablePolicy.afterCalculated(1000원)
   → 세금 추가: 1000 + (1000 × 0.05) = 1050원
   ↓
5. 최종 결과: 1050원
```

**예시 2: 일반 요금제 + 할인 + 세금**

```java
Phone phone = new Phone(
    new TaxablePolicy(0.05,  // 5% 세금 (마지막)
        new RateDiscountablePolicy(Money.wons(1000),  // 1000원 할인 (중간)
            new RegularPolicy(
                Money.wons(100), 
                Duration.ofSeconds(10)
            )  // 기본 요금 (처음)
        )
    )
);
```

**실행 흐름:**

```
1. phone.calculateFee() 호출
   ↓
2. TaxablePolicy.calculateFee()
   ↓
3. RateDiscountablePolicy.calculateFee()
   ↓
4. RegularPolicy.calculateFee()
   → 기본 요금: 10000원
   ↓
5. RateDiscountablePolicy.afterCalculated(10000원)
   → 할인: 10000 - 1000 = 9000원
   ↓
6. TaxablePolicy.afterCalculated(9000원)
   → 세금: 9000 + (9000 × 0.05) = 9450원
   ↓
7. 최종 결과: 9450원
```

**예시 3: 순서 변경 (일반 요금제 + 세금 + 할인)**

```java
Phone phone = new Phone(
    new RateDiscountablePolicy(Money.wons(1000),  // 1000원 할인 (마지막)
        new TaxablePolicy(0.05,  // 5% 세금 (중간)
            new RegularPolicy(
                Money.wons(100), 
                Duration.ofSeconds(10)
            )  // 기본 요금 (처음)
        )
    )
);
```

**실행 흐름:**

```
1. phone.calculateFee() 호출
   ↓
2. RateDiscountablePolicy.calculateFee()
   ↓
3. TaxablePolicy.calculateFee()
   ↓
4. RegularPolicy.calculateFee()
   → 기본 요금: 10000원
   ↓
5. TaxablePolicy.afterCalculated(10000원)
   → 세금: 10000 + (10000 × 0.05) = 10500원
   ↓
6. RateDiscountablePolicy.afterCalculated(10500원)
   → 할인: 10500 - 1000 = 9500원
   ↓
7. 최종 결과: 9500원
```

**예시 4: 심야 할인 요금제에도 적용**

```java
Phone phone = new Phone(
    new RateDiscountablePolicy(Money.wons(1000),
        new TaxablePolicy(0.05,
            new NightlyDiscountPolicy(
                Money.wons(5),    // 심야 요금
                Money.wons(10),   // 일반 요금
                Duration.ofSeconds(10)
            )
        )
    )
);
```

#### 📊 객체 조합 시각화

**일반 요금제 + 세금:**

```
Phone
  │
  └─▶ TaxablePolicy
        │
        └─▶ RegularPolicy
```

**일반 요금제 + 할인 + 세금:**

```
Phone
  │
  └─▶ TaxablePolicy
        │
        └─▶ RateDiscountablePolicy
              │
              └─▶ RegularPolicy
```

**심야 할인 + 세금 + 할인:**

```
Phone
  │
  └─▶ RateDiscountablePolicy
        │
        └─▶ TaxablePolicy
              │
              └─▶ NightlyDiscountPolicy
```

### 3.5 새로운 정책 추가하기

#### 🆕 고정 요금제 추가

**클래스 하나만 추가:**

```java
public class FixedFeePolicy extends BasicRatePolicy {
    private Money monthlyFee;
    
    public FixedFeePolicy(Money monthlyFee) {
        this.monthlyFee = monthlyFee;
    }
    
    @Override
    protected Money calculateCallFee(Call call) {
        return Money.ZERO;  // 통화 요금 없음
    }
    
    @Override
    public Money calculateFee(Phone phone) {
        return monthlyFee;  // 월정액만
    }
}
```

**사용:**

```java
// 고정 요금제만
Phone phone = new Phone(
    new FixedFeePolicy(Money.wons(50000))
);

// 고정 요금제 + 세금
Phone phone = new Phone(
    new TaxablePolicy(0.05,
        new FixedFeePolicy(Money.wons(50000))
    )
);

// 고정 요금제 + 할인 + 세금
Phone phone = new Phone(
    new TaxablePolicy(0.05,
        new RateDiscountablePolicy(Money.wons(5000),
            new FixedFeePolicy(Money.wons(50000))
        )
    )
);
```

#### 🆕 약정 할인 정책 추가

**클래스 하나만 추가:**

```java
public class ContractDiscountPolicy extends AdditionalRatePolicy {
    private Money discountAmount;
    
    public ContractDiscountPolicy(Money discountAmount, RatePolicy next) {
        super(next);
        this.discountAmount = discountAmount;
    }
    
    @Override
    protected Money afterCalculated(Money fee) {
        return fee.minus(discountAmount);
    }
}
```

**사용:**

```java
// 일반 요금제 + 약정 할인
Phone phone = new Phone(
    new ContractDiscountPolicy(Money.wons(3000),
        new RegularPolicy(Money.wons(100), Duration.ofSeconds(10))
    )
);

// 일반 요금제 + 약정 할인 + 기본 할인 + 세금
Phone phone = new Phone(
    new TaxablePolicy(0.05,
        new RateDiscountablePolicy(Money.wons(1000),
            new ContractDiscountPolicy(Money.wons(3000),
                new RegularPolicy(Money.wons(100), Duration.ofSeconds(10))
            )
        )
    )
);
```

#### 📊 상속 vs 합성 비교

**상속 방식:**

```
고정 요금제 추가 시:
- FixedFeePhone
- TaxableFixedFeePhone
- RateDiscountableFixedFeePhone
- TaxableAndRateDiscountableFixedFeePhone
- RateDiscountableAndTaxableFixedFeePhone

총 5개 클래스 추가

약정 할인 정책 추가 시:
- 기존 모든 조합에 대해 클래스 추가
- 수십 개의 클래스 추가 필요
```

**합성 방식:**

```
고정 요금제 추가 시:
- FixedFeePolicy 클래스 1개만 추가
- 기존 부가 정책들과 자유롭게 조합

약정 할인 정책 추가 시:
- ContractDiscountPolicy 클래스 1개만 추가
- 기존 정책들과 자유롭게 조합

→ 항상 1개 클래스만 추가!
```

### 3.6 변경의 용이성

#### 💡 단일 책임 원칙 준수

**세금 정책 변경 시:**

```
상속 방식:
❌ TaxableRegularPhone
❌ TaxableNightlyDiscountPhone
❌ TaxableFixedFeePhone
❌ TaxableAndRateDiscountableRegularPhone
❌ ... (모든 Taxable* 클래스)

합성 방식:
✅ TaxablePolicy 클래스 하나만 수정
```

**변경 영향도:**

| 정책 변경 | 상속 | 합성 |
|----------|------|------|
| **세금 정책** | 모든 Taxable* 클래스 | TaxablePolicy 1개 |
| **할인 정책** | 모든 RateDiscountable* 클래스 | RateDiscountablePolicy 1개 |
| **약정 할인** | 모든 Contract* 클래스 | ContractDiscountPolicy 1개 |

### 3.7 합성의 본질

#### 🎯 핵심 메커니즘

```
컴파일타임 의존성 ≠ 런타임 의존성

컴파일타임:
- Phone → RatePolicy 인터페이스
- 구체적인 정책은 모름

런타임:
- Phone → TaxablePolicy → RegularPolicy
- 실행 시점에 동적으로 조립
```

#### 💡 장점 요약

```
✅ 클래스 수 최소화
   - 구성 요소 수만큼만 필요
   
✅ 중복 코드 제거
   - 각 정책이 한 곳에만 존재
   
✅ 단일 책임 원칙
   - 하나의 정책 변경 시 하나의 클래스만 수정
   
✅ 유연한 조합
   - 런타임에 자유롭게 조합
   
✅ 순서 변경 용이
   - 생성자 호출 순서만 변경
```

### 3.8 객체 합성이 클래스 상속보다 더 좋은 방법이다

#### 📜 설계 원칙

```
┌─────────────────────────────────────────────────────┐
│  객체지향에서 코드를 재사용하면서도                          │
│  건전한 결합도를 유지할 수 있는 더 좋은 방법은                 │
│  합성을 이용하는 것이다.                                  │
│                                                     │
│  상속: 구현을 재사용                                    │
│  합성: 인터페이스를 재사용                                │
└─────────────────────────────────────────────────────┘
```

#### ⚠️ 상속은 언제 사용하는가?

```
이번 장에서 본 상속의 모든 단점은
구현 상속 (Implementation Inheritance)에 국한됨

인터페이스 상속 (Interface Inheritance)은 다름
→ 13장에서 다룰 예정
```

---

## 4. 믹스인

> 📂 **코드**: [Scala 예제](https://github.com/eternity-oop/object/tree/master/chapter11/src/main/scala/org/eternity/billing)

### 4.1 믹스인이란?

#### 🎯 정의

**믹스인 (Mixin):**
```
객체를 생성할 때
코드 일부를 클래스 안에 섞어 넣어
재사용하는 기법

Compile-time Composition:
컴파일 시점에 필요한 코드 조각을 조합하는
재사용 방법
```

**비교:**

| 측면 | 합성 | 믹스인 |
|------|------|--------|
| **조합 시점** | 런타임 | 컴파일타임 |
| **조합 대상** | 객체 | 코드 조각 |
| **유연성** | 높음 | 중간 |

### 4.2 믹스인 vs 상속

#### 📊 차이점

**상속:**
```
목적:
- is-a 관계 표현
- 자식을 부모와 동일한 개념 범주로 묶기

특성:
- 클래스와 클래스 사이의 관계 고정
- 부모 클래스 확장
```

**믹스인:**
```
목적:
- 코드 재사용에 특화
- 코드를 다른 코드 안에 섞어 넣기

특성:
- 관계를 유연하게 재구성 가능
- 결합도 문제 초래하지 않음
```

### 4.3 스칼라의 트레이트 (Trait)

#### 🎯 기본 정책 구현

```scala
abstract class BasicRatePolicy {
  def calculateFee(phone: Phone): Money = 
    phone.calls.map(calculateCallFee(_)).reduce(_ + _)
  
  protected def calculateCallFee(call: Call): Money
}
```

```scala
class RegularPolicy(val amount: Money, val seconds: Duration) 
  extends BasicRatePolicy {
  
  override protected def calculateCallFee(call: Call): Money = 
    amount * (call.duration.getSeconds / seconds.getSeconds)
}
```

```scala
class NightlyDiscountPolicy(
    val nightlyAmount: Money,  
    val regularAmount: Money,
    val seconds: Duration) 
  extends BasicRatePolicy {   
  
  override protected def calculateCallFee(call: Call): Money =
    if (call.from.getHour >= NightlyDiscountPolicy.LateNightHour) {
      nightlyAmount * (call.duration.getSeconds / seconds.getSeconds)
    } else {
      regularAmount * (call.duration.getSeconds / seconds.getSeconds)
    }
}
```

#### 🎭 트레이트로 부가 정책 구현

**세금 정책 트레이트:**

```scala
trait TaxablePolicy extends BasicRatePolicy {
  val taxRate: Double
  
  override def calculateFee(phone: Phone): Money = {
    val fee = super.calculateFee(phone)
    fee + fee * taxRate
  }
}
```

**핵심:**
```
✅ BasicRatePolicy를 확장 (extends)
   - 상속이 아님!
   - 사용 가능한 문맥 제한
   
✅ super 호출
   - 부모 클래스 고정 안 됨
   - 믹스인 시점에 결정됨
   
✅ 추상 필드 (taxRate)
   - 믹스인 시 구현 필요
```

**기본 요금 할인 트레이트:**

```scala
trait RateDiscountablePolicy extends BasicRatePolicy {
  val discountAmount: Money
  
  override def calculateFee(phone: Phone): Money = {
    val fee = super.calculateFee(phone)
    fee - discountAmount
  }  
}
```

### 4.4 트레이트 믹스인하기

#### 🔧 기본 사용법

**문법:**
```scala
class ClassName(params)
  extends ParentClass(params)  // 부모 클래스
  with Trait1                  // 트레이트 믹스인
  with Trait2                  // 트레이트 믹스인
```

**예시들:**

```scala
// 일반 요금제 + 세금
class TaxableRegularPolicy(
    amount: Money, 
    seconds: Duration, 
    val taxRate: Double
  ) 
  extends RegularPolicy(amount, seconds) 
  with TaxablePolicy

// 심야 할인 + 세금
class TaxableNightlyDiscountPolicy(
    nightlyAmount: Money, 
    regularAmount: Money, 
    seconds: Duration, 
    val taxRate: Double
  ) 
  extends NightlyDiscountPolicy(nightlyAmount, regularAmount, seconds) 
  with TaxablePolicy

// 일반 요금제 + 기본 할인
class RateDiscountableRegularPolicy(
    amount: Money, 
    seconds: Duration, 
    val discountAmount: Money
  )    
  extends RegularPolicy(amount, seconds) 
  with RateDiscountablePolicy

// 심야 할인 + 기본 할인
class RateDiscountableNightlyDiscountPolicy(
    nightlyAmount: Money, 
    regularAmount: Money, 
    seconds: Duration, 
    val discountAmount: Money
  )    
  extends NightlyDiscountPolicy(nightlyAmount, regularAmount, seconds) 
  with RateDiscountablePolicy
```

#### 📊 클래스 다이어그램

```
BasicRatePolicy
      △
      │
      ├──────────────────┐
      │                  │
RegularPolicy    NightlyDiscountPolicy
      △                  △
      │                  │
      │ with             │ with
TaxablePolicy    TaxablePolicy
```

### 4.5 트레이트의 선형화 (Linearization)

#### 🎯 여러 트레이트 믹스인

**예시:**

```scala
class RateDiscountableAndTaxableRegularPolicy(
    amount: Money, 
    seconds: Duration, 
    val discountAmount: Money,
    val taxRate: Double
  )
  extends RegularPolicy(amount, seconds)
  with TaxablePolicy 
  with RateDiscountablePolicy
```

**선형화 과정:**

```
1. 클래스 및 트레이트 나열
   RateDiscountableAndTaxableRegularPolicy
   → RateDiscountablePolicy
   → TaxablePolicy
   → RegularPolicy
   → BasicRatePolicy
   → AnyRef
   → Any

2. 선형화 결과 (오른쪽 → 왼쪽)
   RateDiscountableAndTaxableRegularPolicy
   ← RateDiscountablePolicy
   ← TaxablePolicy
   ← RegularPolicy
   ← BasicRatePolicy
   ← AnyRef
   ← Any
```

**실행 흐름:**

```
1. RateDiscountableAndTaxableRegularPolicy.calculateFee()
   → 없음
   ↓
2. RateDiscountablePolicy.calculateFee()
   → super.calculateFee() 호출
   ↓
3. TaxablePolicy.calculateFee()
   → super.calculateFee() 호출
   ↓
4. RegularPolicy.calculateFee()
   → BasicRatePolicy.calculateFee() 호출
   ↓
5. BasicRatePolicy.calculateFee()
   → 기본 요금 계산: 10000원
   ↓
6. RegularPolicy로 돌아옴
   → 10000원 반환
   ↓
7. TaxablePolicy.calculateFee()로 돌아옴
   → fee = 10000원
   → fee + fee * taxRate
   → 10000 + 500 = 10500원
   ↓
8. RateDiscountablePolicy.calculateFee()로 돌아옴
   → fee = 10500원
   → fee - discountAmount
   → 10500 - 1000 = 9500원
   ↓
9. 최종 결과: 9500원
```

#### 🔄 순서 변경

**순서를 바꾸려면:**

```scala
class TaxableAndRateDiscountableRegularPolicy(
    amount: Money, 
    seconds: Duration, 
    val discountAmount: Money,
    val taxRate: Double
  )
  extends RegularPolicy(amount, seconds)
  with RateDiscountablePolicy  // 순서 변경!
  with TaxablePolicy
```

**선형화 결과:**

```
TaxableAndRateDiscountableRegularPolicy
← TaxablePolicy  // 마지막 적용
← RateDiscountablePolicy  // 중간 적용
← RegularPolicy
← BasicRatePolicy
```

**실행 흐름:**

```
기본 요금: 10000원
  ↓
RateDiscountablePolicy: 10000 - 1000 = 9000원
  ↓
TaxablePolicy: 9000 + 450 = 9450원
  ↓
최종 결과: 9450원
```

### 4.6 클래스 없이 믹스인 사용하기

#### 🎯 즉석 믹스인

```scala
new RegularPolicy(Money(100), Duration.ofSeconds(10))
  with RateDiscountablePolicy
  with TaxablePolicy {
    val discountAmount = Money(100)
    val taxRate = 0.02
  }
```

**장점:**
```
✅ 클래스 선언 불필요
✅ 필요한 조합만 즉석에서 생성
✅ 테스트에 유용
```

**단점:**
```
❌ 재사용 어려움
❌ 여러 곳에서 사용 시 중복
```

**권장사항:**
```
여러 곳에서 동일한 조합 사용 시:
→ 명시적으로 클래스 정의

일회성 사용 시:
→ 즉석 믹스인 활용
```

### 4.7 쌓을 수 있는 변경 (Stackable Modification)

#### 🎯 개념

**정의:**
```
믹스인을 사용하면
특정 클래스에 대한 변경 또는 확장을
독립적으로 구현한 후
필요한 시점에 차례대로 쌓아올릴 수 있음
```

**특성:**

```
1. 믹스인은 대상 클래스의 자식처럼 동작
   with 구문은 항상 extends 뒤에 위치
   
2. 독립적인 변경 구현
   각 트레이트는 하나의 변경만 담당
   
3. 조합 가능
   여러 변경을 쌓아서 적용
   
4. 순서 변경 가능
   with 순서를 바꿔서 적용 순서 조정
```

**예시:**

```scala
// 변경 1: 세금
trait TaxablePolicy extends BasicRatePolicy {
  val taxRate: Double
  override def calculateFee(phone: Phone): Money = {
    val fee = super.calculateFee(phone)
    fee + fee * taxRate
  }
}

// 변경 2: 할인
trait RateDiscountablePolicy extends BasicRatePolicy {
  val discountAmount: Money
  override def calculateFee(phone: Phone): Money = {
    val fee = super.calculateFee(phone)
    fee - discountAmount
  }
}

// 변경들을 쌓아올리기
class Policy
  extends RegularPolicy(...)
  with TaxablePolicy      // 1층
  with RateDiscountablePolicy  // 2층
```

### 4.8 믹스인과 클래스 폭발 문제

#### ❓ 클래스가 여전히 많지 않은가?

**의문:**
```
믹스인을 사용해도
여러 조합을 위해 클래스를 만들어야 하는데
클래스 폭발 문제가 해결된 건가?
```

**답변:**

```
클래스 폭발 문제의 진짜 문제는:
❌ 클래스 수가 많다는 것이 아님
❌ 중복 코드가 기하급수적으로 늘어난다는 것!

믹스인의 해결:
✅ 각 정책이 한 곳에만 존재
✅ 중복 코드 없음
✅ 한 곳만 수정하면 모든 조합에 반영
```

**상속 vs 믹스인 비교:**

```
상속:
- TaxableRegularPhone
- TaxableNightlyDiscountPhone
- TaxableFixedFeePhone
→ 세금 계산 로직이 3곳에 중복

믹스인:
- TaxablePolicy 트레이트 하나
→ 세금 계산 로직이 한 곳에만 존재
→ 어떤 정책과도 믹스인 가능
```

### 4.9 믹스인 정리

#### 💡 핵심 특징

```
✅ 코드 재사용에 특화
   - 상속보다 유연
   
✅ 컴파일타임 조합
   - 합성보다는 정적
   - 상속보다는 동적
   
✅ 중복 코드 제거
   - 각 변경이 한 곳에만
   
✅ 독립적인 변경
   - 각 트레이트가 하나의 책임
   
✅ 쌓을 수 있는 변경
   - 여러 변경을 조합 가능
```

#### ⚖️ 장단점

**장점:**
```
✅ 상속의 결합도 문제 해결
✅ 코드 재사용 용이
✅ 유연한 조합
✅ 중복 코드 제거
```

**단점:**
```
❌ 언어 지원 필요 (Java는 미지원)
❌ 합성보다는 덜 동적
❌ 복잡한 선형화 규칙 이해 필요
```

---

## 5. 핵심 정리

### 🎯 상속 vs 합성

#### 비교표

| 측면 | 상속 | 합성 |
|------|------|------|
| **관계** | is-a | has-a |
| **의존성** | 컴파일타임 고정 | 런타임 변경 가능 |
| **재사용** | 구현 재사용 | 인터페이스 재사용 |
| **결합도** | 높음 | 낮음 |
| **재사용 방식** | 화이트박스 | 블랙박스 |
| **유연성** | 낮음 | 높음 |

#### 상속의 문제점

```
1. 불필요한 인터페이스 상속
   - 부모의 불필요한 메서드 노출
   
2. 메서드 오버라이딩 오작용
   - 부모의 내부 구현에 의존
   
3. 부모-자식 동시 수정
   - 내부 구현 공유로 인한 결합
   
4. 클래스 폭발
   - 조합 수만큼 클래스 증가
   - 중복 코드 기하급수적 증가
```

#### 합성의 장점

```
1. 낮은 결합도
   - 퍼블릭 인터페이스에만 의존
   
2. 런타임 조합
   - 실행 시점에 객체 조립
   
3. 클래스 수 최소화
   - 구성 요소 수만큼만 필요
   
4. 중복 코드 제거
   - 각 요소가 한 곳에만 존재
   
5. 단일 책임 원칙
   - 하나의 변경에 하나의 클래스만 수정
```

### 📏 설계 원칙

#### 객체 합성이 클래스 상속보다 더 좋은 방법이다

```
✅ 코드 재사용 시 합성 우선 고려
✅ 구현 결합보다 인터페이스 결합
✅ 컴파일타임보다 런타임 의존성
✅ 화이트박스보다 블랙박스 재사용
```

#### 포워딩(Forwarding)

```
기존 클래스의 인터페이스를 그대로 제공하면서
구현에 대한 결합 없이
일부 작동 방식만 변경할 때 사용

포워딩 메서드:
동일한 메서드 호출을 내부 객체에 전달하는 메서드
```

### 🎭 믹스인 (Mixin)

#### 특징

```
컴파일 시점에 코드 조각을 조합하는 재사용 기법

상속과의 차이:
- 상속: is-a 관계 표현
- 믹스인: 코드 재사용에 특화

합성과의 차이:
- 합성: 런타임 조합
- 믹스인: 컴파일타임 조합

장점:
✅ 상속의 결합도 문제 해결
✅ 유연한 조합
✅ 쌓을 수 있는 변경
```

### 💡 핵심 교훈

```
1. 상속의 한계를 이해하라
   - 클래스 폭발 문제
   - 중복 코드 증가
   - 높은 결합도
   
2. 합성을 적극 활용하라
   - 런타임 조합
   - 낮은 결합도
   - 유연한 설계
   
3. 상속은 신중하게 사용하라
   - 구현 상속의 문제점 인식
   - 인터페이스 상속은 괜찮음 (13장)
   
4. 믹스인도 고려하라
   - 언어가 지원한다면
   - 상속과 합성의 중간
```

### 🎓 실전 가이드

#### 언제 합성을 사용하는가?

```
✅ 코드 재사용이 주 목적일 때
✅ 런타임에 동작을 변경해야 할 때
✅ 여러 기능을 조합해야 할 때
✅ 기존 클래스의 일부만 필요할 때
```

#### 언제 상속을 사용하는가?

```
✅ 진정한 is-a 관계일 때
✅ 인터페이스 상속일 때 (13장)
✅ 다형성이 필요할 때
✅ 확장을 고려해 설계된 클래스일 때
```

---

## 6. 실전 예제

### 예제 1: 로깅 시스템 - 상속의 클래스 폭발

#### ❌ Before: 상속 사용

```java
// 기본 로거
public abstract class Logger {
    public void log(String message) {
        String formatted = formatMessage(message);
        writeLog(formatted);
    }
    
    protected abstract String formatMessage(String message);
    protected abstract void writeLog(String message);
}

// 파일 로거
public class FileLogger extends Logger {
    private String filePath;
    
    @Override
    protected String formatMessage(String message) {
        return message;
    }
    
    @Override
    protected void writeLog(String message) {
        // 파일에 쓰기
    }
}

// 콘솔 로거
public class ConsoleLogger extends Logger {
    @Override
    protected String formatMessage(String message) {
        return message;
    }
    
    @Override
    protected void writeLog(String message) {
        System.out.println(message);
    }
}
```

**부가 기능 추가 필요:**
```
1. 타임스탬프 추가
2. 로그 레벨 추가 (INFO, WARN, ERROR)
3. 암호화

상속으로 구현 시:
- TimestampFileLogger
- TimestampConsoleLogger
- LevelFileLogger
- LevelConsoleLogger
- TimestampAndLevelFileLogger
- TimestampAndLevelConsoleLogger
- EncryptedFileLogger
- ...

→ 클래스 폭발!
```

#### ✅ After: 합성 사용

```java
// 로그 처리 전략 인터페이스
public interface LogStrategy {
    void log(String message);
}

// 기본 파일 로거
public class FileLogStrategy implements LogStrategy {
    private String filePath;
    
    @Override
    public void log(String message) {
        // 파일에 쓰기
    }
}

// 기본 콘솔 로거
public class ConsoleLogStrategy implements LogStrategy {
    @Override
    public void log(String message) {
        System.out.println(message);
    }
}

// 부가 기능 데코레이터
public abstract class LogDecorator implements LogStrategy {
    protected LogStrategy wrapped;
    
    public LogDecorator(LogStrategy wrapped) {
        this.wrapped = wrapped;
    }
    
    @Override
    public void log(String message) {
        String processed = process(message);
        wrapped.log(processed);
    }
    
    protected abstract String process(String message);
}

// 타임스탬프 데코레이터
public class TimestampDecorator extends LogDecorator {
    public TimestampDecorator(LogStrategy wrapped) {
        super(wrapped);
    }
    
    @Override
    protected String process(String message) {
        return LocalDateTime.now() + " " + message;
    }
}

// 로그 레벨 데코레이터
public class LevelDecorator extends LogDecorator {
    private Level level;
    
    public LevelDecorator(Level level, LogStrategy wrapped) {
        super(wrapped);
        this.level = level;
    }
    
    @Override
    protected String process(String message) {
        return "[" + level + "] " + message;
    }
}

// 암호화 데코레이터
public class EncryptionDecorator extends LogDecorator {
    public EncryptionDecorator(LogStrategy wrapped) {
        super(wrapped);
    }
    
    @Override
    protected String process(String message) {
        return encrypt(message);
    }
    
    private String encrypt(String message) {
        // 암호화 로직
        return message;
    }
}
```

**사용 예시:**

```java
// 타임스탬프 + 레벨 + 파일 로거
LogStrategy logger = new TimestampDecorator(
    new LevelDecorator(Level.INFO,
        new FileLogStrategy("app.log")
    )
);

// 암호화 + 타임스탬프 + 콘솔 로거
LogStrategy secureLogger = new EncryptionDecorator(
    new TimestampDecorator(
        new ConsoleLogStrategy()
    )
);

// 순서 변경도 자유롭게
LogStrategy logger2 = new LevelDecorator(Level.ERROR,
    new TimestampDecorator(
        new EncryptionDecorator(
            new FileLogStrategy("error.log")
        )
    )
);
```

**개선 효과:**

| 측면 | 상속 | 합성 |
|------|------|------|
| **클래스 수** | 2 × 2³ = 16개 | 3 + 3 = 6개 |
| **새 기능 추가** | 모든 조합 클래스 추가 | 데코레이터 1개 추가 |
| **조합 유연성** | 컴파일타임 고정 | 런타임 자유 |

---

### 예제 2: GUI 컴포넌트 - 스타일 조합

#### ❌ Before: 상속 기반

```java
// 기본 버튼
public class Button {
    protected String text;
    
    public void render() {
        System.out.println("Button: " + text);
    }
}

// 테두리 버튼
public class BorderButton extends Button {
    @Override
    public void render() {
        System.out.println("┌─────────┐");
        super.render();
        System.out.println("└─────────┘");
    }
}

// 그림자 버튼
public class ShadowButton extends Button {
    @Override
    public void render() {
        super.render();
        System.out.println("  ▓▓▓▓▓▓▓");
    }
}

// 테두리 + 그림자 버튼 (중복!)
public class BorderShadowButton extends BorderButton {
    @Override
    public void render() {
        super.render();
        System.out.println("  ▓▓▓▓▓▓▓");
    }
}

// 그림자 + 테두리 버튼 (다른 순서, 또 중복!)
public class ShadowBorderButton extends ShadowButton {
    @Override
    public void render() {
        System.out.println("┌─────────┐");
        System.out.println("Button: " + text);
        System.out.println("└─────────┘");
        System.out.println("  ▓▓▓▓▓▓▓");
    }
}
```

**문제:**
```
스타일 3개만 추가해도:
- 기본 (3개)
- 단일 스타일 (3개)
- 2개 조합 (6개)
- 3개 조합 (2개)
= 총 14개 클래스
```

#### ✅ After: 데코레이터 패턴 (합성)

```java
// 컴포넌트 인터페이스
public interface Component {
    void render();
}

// 기본 버튼
public class Button implements Component {
    private String text;
    
    public Button(String text) {
        this.text = text;
    }
    
    @Override
    public void render() {
        System.out.println("Button: " + text);
    }
}

// 스타일 데코레이터
public abstract class ComponentDecorator implements Component {
    protected Component component;
    
    public ComponentDecorator(Component component) {
        this.component = component;
    }
}

// 테두리 스타일
public class BorderDecorator extends ComponentDecorator {
    public BorderDecorator(Component component) {
        super(component);
    }
    
    @Override
    public void render() {
        System.out.println("┌─────────┐");
        component.render();
        System.out.println("└─────────┘");
    }
}

// 그림자 스타일
public class ShadowDecorator extends ComponentDecorator {
    public ShadowDecorator(Component component) {
        super(component);
    }
    
    @Override
    public void render() {
        component.render();
        System.out.println("  ▓▓▓▓▓▓▓");
    }
}

// 색상 스타일
public class ColorDecorator extends ComponentDecorator {
    private String color;
    
    public ColorDecorator(String color, Component component) {
        super(component);
        this.color = color;
    }
    
    @Override
    public void render() {
        System.out.println("[" + color + "]");
        component.render();
    }
}
```

**사용 예시:**

```java
// 테두리 + 그림자
Component button1 = new BorderDecorator(
    new ShadowDecorator(
        new Button("Submit")
    )
);

// 색상 + 테두리 + 그림자
Component button2 = new ColorDecorator("blue",
    new BorderDecorator(
        new ShadowDecorator(
            new Button("Cancel")
        )
    )
);

// 런타임에 동적으로 추가
Component button = new Button("Click");
if (needsBorder) {
    button = new BorderDecorator(button);
}
if (needsShadow) {
    button = new ShadowDecorator(button);
}
```

---

### 예제 3: 스트림 처리 - 필터 조합

#### ❌ Before: 상속으로 필터 구현

```java
// 기본 스트림
public abstract class Stream {
    public abstract byte[] read();
}

// 파일 스트림
public class FileStream extends Stream {
    @Override
    public byte[] read() {
        // 파일에서 읽기
    }
}

// 압축 파일 스트림
public class CompressedFileStream extends FileStream {
    @Override
    public byte[] read() {
        byte[] data = super.read();
        return decompress(data);
    }
}

// 암호화 파일 스트림
public class EncryptedFileStream extends FileStream {
    @Override
    public byte[] read() {
        byte[] data = super.read();
        return decrypt(data);
    }
}

// 압축 + 암호화 파일 스트림 (중복!)
public class CompressedEncryptedFileStream extends CompressedFileStream {
    @Override
    public byte[] read() {
        byte[] data = super.read();
        return decrypt(data);
    }
}

// 암호화 + 압축 파일 스트림 (다른 순서!)
public class EncryptedCompressedFileStream extends EncryptedFileStream {
    @Override
    public byte[] read() {
        byte[] data = super.read();
        return decompress(data);
    }
}
```

#### ✅ After: 자바 I/O 스타일 (합성)

```java
// 스트림 인터페이스
public abstract class InputStream {
    public abstract int read() throws IOException;
}

// 기본 파일 스트림
public class FileInputStream extends InputStream {
    @Override
    public int read() throws IOException {
        // 파일에서 읽기
    }
}

// 필터 스트림 (데코레이터)
public abstract class FilterInputStream extends InputStream {
    protected InputStream in;
    
    protected FilterInputStream(InputStream in) {
        this.in = in;
    }
    
    @Override
    public int read() throws IOException {
        return in.read();
    }
}

// 압축 해제 필터
public class DecompressionInputStream extends FilterInputStream {
    public DecompressionInputStream(InputStream in) {
        super(in);
    }
    
    @Override
    public int read() throws IOException {
        int data = in.read();
        return decompress(data);
    }
}

// 복호화 필터
public class DecryptionInputStream extends FilterInputStream {
    public DecryptionInputStream(InputStream in) {
        super(in);
    }
    
    @Override
    public int read() throws IOException {
        int data = in.read();
        return decrypt(data);
    }
}

// 버퍼링 필터
public class BufferedInputStream extends FilterInputStream {
    private byte[] buffer;
    
    public BufferedInputStream(InputStream in) {
        super(in);
        this.buffer = new byte[8192];
    }
    
    @Override
    public int read() throws IOException {
        // 버퍼링 로직
    }
}
```

**사용 예시:**

```java
// 파일 → 압축 해제 → 복호화
InputStream stream1 = new DecryptionInputStream(
    new DecompressionInputStream(
        new FileInputStream("data.bin")
    )
);

// 파일 → 복호화 → 압축 해제 → 버퍼링
InputStream stream2 = new BufferedInputStream(
    new DecompressionInputStream(
        new DecryptionInputStream(
            new FileInputStream("secure.dat")
        )
    )
);
```

---

### 예제 4: 할인 정책 시스템

#### 📊 요구사항

```
기본 할인:
1. 금액 할인 (1000원 할인)
2. 비율 할인 (10% 할인)

부가 할인:
1. 중복 할인 (같은 할인 2회 적용)
2. 조건부 할인 (최소 금액 이상만 적용)
3. 한도 할인 (최대 할인 금액 제한)
```

#### ✅ 합성 기반 구현

```java
// 할인 정책 인터페이스
public interface DiscountPolicy {
    Money calculate(Money price);
}

// 기본 정책: 금액 할인
public class AmountDiscount implements DiscountPolicy {
    private Money discountAmount;
    
    public AmountDiscount(Money amount) {
        this.discountAmount = amount;
    }
    
    @Override
    public Money calculate(Money price) {
        return price.minus(discountAmount);
    }
}

// 기본 정책: 비율 할인
public class PercentDiscount implements DiscountPolicy {
    private double percent;
    
    public PercentDiscount(double percent) {
        this.percent = percent;
    }
    
    @Override
    public Money calculate(Money price) {
        return price.minus(price.times(percent));
    }
}

// 부가 정책 기본 클래스
public abstract class DiscountDecorator implements DiscountPolicy {
    protected DiscountPolicy base;
    
    public DiscountDecorator(DiscountPolicy base) {
        this.base = base;
    }
}

// 중복 할인
public class DuplicateDiscount extends DiscountDecorator {
    public DuplicateDiscount(DiscountPolicy base) {
        super(base);
    }
    
    @Override
    public Money calculate(Money price) {
        Money discounted = base.calculate(price);
        return base.calculate(discounted);
    }
}

// 조건부 할인
public class ConditionalDiscount extends DiscountDecorator {
    private Money minimumAmount;
    
    public ConditionalDiscount(Money minimum, DiscountPolicy base) {
        super(base);
        this.minimumAmount = minimum;
    }
    
    @Override
    public Money calculate(Money price) {
        if (price.isGreaterThan(minimumAmount)) {
            return base.calculate(price);
        }
        return price;
    }
}

// 한도 할인
public class CappedDiscount extends DiscountDecorator {
    private Money maxDiscount;
    
    public CappedDiscount(Money maxDiscount, DiscountPolicy base) {
        super(base);
        this.maxDiscount = maxDiscount;
    }
    
    @Override
    public Money calculate(Money price) {
        Money discounted = base.calculate(price);
        Money discount = price.minus(discounted);
        
        if (discount.isGreaterThan(maxDiscount)) {
            return price.minus(maxDiscount);
        }
        return discounted;
    }
}
```

**조합 예시:**

```java
// 10% 할인을 2회 적용 (중복)
DiscountPolicy policy1 = new DuplicateDiscount(
    new PercentDiscount(0.1)
);

// 1000원 할인, 단 10000원 이상일 때만, 최대 2000원까지
DiscountPolicy policy2 = new CappedDiscount(Money.wons(2000),
    new ConditionalDiscount(Money.wons(10000),
        new AmountDiscount(Money.wons(1000))
    )
);

// 사용
Money originalPrice = Money.wons(15000);
Money finalPrice = policy2.calculate(originalPrice);
```

---

## 7. 합성 설계 가이드

### 1. 합성을 사용해야 하는 신호

```
✅ 다음 상황에서는 합성을 고려하라:

1. 여러 기능의 조합이 필요할 때
   - 기능 A, B, C를 자유롭게 조합
   
2. 런타임에 동작 변경이 필요할 때
   - 실행 중 전략 교체
   
3. 클래스가 기하급수적으로 증가할 때
   - 조합 폭발 징후
   
4. 같은 코드가 여러 곳에 중복될 때
   - 상속 계층 전반의 중복
   
5. 기능 추가 시 여러 클래스 수정이 필요할 때
   - 단일 책임 원칙 위반
```

### 2. 합성 적용 패턴

#### 데코레이터 패턴

```
언제 사용:
- 기존 객체에 부가 기능을 동적으로 추가
- 여러 부가 기능을 자유롭게 조합

예시:
- I/O 스트림 (Java)
- 로깅 시스템
- GUI 컴포넌트 스타일
```

#### 전략 패턴

```
언제 사용:
- 알고리즘을 런타임에 교체
- 같은 문제를 다양한 방법으로 해결

예시:
- 정렬 알고리즘
- 압축 알고리즘
- 결제 방법
```

#### 복합체 패턴

```
언제 사용:
- 부분-전체 계층 구조
- 개별 객체와 복합 객체를 동일하게 처리

예시:
- 파일 시스템
- GUI 컨테이너
- 조직도
```

### 3. 합성 구현 체크리스트

```
□ 공통 인터페이스 정의
  - 기본 기능과 부가 기능이 같은 인터페이스 구현
  
□ 기본 구현 분리
  - 핵심 기능을 독립적인 클래스로
  
□ 데코레이터 추상 클래스
  - 부가 기능들의 공통 구조
  
□ 구체적인 데코레이터들
  - 각 부가 기능을 별도 클래스로
  
□ 의존성 주입
  - 생성자를 통한 조합
```

### 4. 주의사항

#### 과도한 합성 피하기

```java
// ❌ 너무 많은 래핑
Component component = new Decorator1(
    new Decorator2(
        new Decorator3(
            new Decorator4(
                new Decorator5(
                    new Decorator6(
                        new BaseComponent()
                    )
                )
            )
        )
    )
);

// ✅ 필요한 만큼만
Component component = new Decorator1(
    new Decorator2(
        new BaseComponent()
    )
);
```

#### 순서 의존성 문서화

```java
/**
 * 올바른 순서:
 * 1. Compression (압축)
 * 2. Encryption (암호화)
 * 3. Buffering (버퍼링)
 * 
 * 잘못된 순서:
 * Buffering → Compression → Encryption
 * (버퍼링이 가장 안쪽에 있으면 효과 없음)
 */
```

#### 디버깅 어려움 대비

```java
// 디버깅을 위한 toString 구현
public abstract class LogDecorator implements LogStrategy {
    @Override
    public String toString() {
        return getClass().getSimpleName() + 
               "(" + wrapped.toString() + ")";
    }
}

// 출력: TimestampDecorator(LevelDecorator(FileLogStrategy))
```

---

## 🔗 연결고리

### 이전 장과의 연결
- **Chapter 10**: 상속의 문제점 → 합성으로 해결
- 취약한 기반 클래스 문제 → 합성의 우수성
- 클래스 폭발 문제 → 런타임 조합

### 다음 장 예고
- **Chapter 12**: 다형성
  - 상속과 합성의 통합적 이해
  - 진정한 객체지향 설계
  - 올바른 추상화 방법

---

**[⬆ 목차로 돌아가기](#-목차)**

<div align="center">

**[⬅️ 이전: Chapter 10](../chapter10/README.md) | [다음: Chapter 12 ➡️](../chapter12/README.md)**

</div>
