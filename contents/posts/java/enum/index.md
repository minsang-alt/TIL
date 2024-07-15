---
title: "Enum"
date: 2024-07-15
update: 2024-07-15
tags:
  - Java
---

Enum을 사용하다보니 다양한 이점들이 있는 것 같습니다. Enum의 기본적인 개념을 짧게 설명하고 장점들을 소개해보겠습니다.

## Enum 이란

**Enum 클래스는 열거형 상수를 정의하기 위한 클래스** 입니다. ,(콤마)를 사용해 열거형 상수를 나열하고, 세미콜론으로 마무리합니다.

예를들어 다음과 같이 사용할 수 있습니다. 날짜 열거형을 만들어보겠습니다.

```java
public enum Day {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
}
```

이 enum 타입을 매개변수로 넘길려면, `enum 클래스이름.상수 이름`을 지정함으로써 클래스의 객체 생성이 완료된다고 생각하면 됩니다.

```java
public class EnumTest {
    public static void main(String[] args) {
        Day day = Day.MONDAY;
        printDay(day);   
    }
    
    public static void printDay(Day day) {
        System.out.println(day);
    }
}
```

### Enum 클래스의 부모는 java.lang.Enum

Enum 클래스는 java.lang.Enum 클래스를 상속받습니다. 따라서 다른 클래스를 상속받을 수 없습니다. 
컴파일러가 Enum 클래스를 컴파일할 때, 자동으로 `java.lang.Enum` 클래스를 상속받습니다.

`java.lang.Enum` 클래스의 부모는 Object 클래스입니다. 따라서 Enum 클래스는 Object 클래스의 메소드를 사용할 수 있습니다.

하지만, **Object의 `clone()`, `finalize()`, `hashCode()`, `equals`는 오버라이딩을 할 수 없게 막아놓았습니다.**
또한, `clone()`은 개발자들이 직접 사용이 불가능하여 호출될 경우 `CloneNotSupportedException`이 발생합니다.

`toString()`만 오버라이딩이 가능하기 때문에 Enum 클래스에서 `toString()`을 오버라이딩하여 사용할 수 있습니다.

`java.lang.Enum` 클래스는 다음과 같은 메소드를 제공합니다.
- `compareTo(E o)` : 매개변수로 enum 타입과의 순서(ordinal) 차이를 리턴합니다.
- `getDeclaringClass()` : enum 타입의 클래스를 리턴합니다.
- `name()` : enum 타입의 이름을 리턴합니다.
- `ordinal()` : enum 타입의 순서를 리턴합니다.
- `valueOf(Class<T> enumType, String name)` : enum 타입의 이름을 매개변수로 받아 enum 타입을 리턴합니다.

enum 클래스에는 API 문서에 없는 특수한 메소드가 하나 있습니다.
`values()` 메소드는 enum 타입의 모든 상수를 배열로 리턴합니다.

## Enum을 언제 사용해야 할까

### 1. 데이터들 간의 연관 나타내기
예를들어, 달력을 표기하기 위해서 그동안 다음과 같이 사용했다고 가정해봅시다. (자바에서 이미 제공하는 캘린더는 잠시 생각하지 말고요)

```java
public class Calendar {
    public static String JANUARY = "1월";
    public static String FEBRUARY = "2월";
    public static String MARCH = "3월";
    public static String APRIL = "4월";
    public static String MAY = "5월";
    // ...
}
```

이렇게 사용했던 개발자는 정수형 타입으로 사용되는 1,2,3,4,5,6,7,8,9,10,11,12를 사용하면 더 좋지 않을까 생각합니다.
그렇지만 이미 이 상수를 사용하고 있는 코드가 많아서 변경하기 어렵습니다. 

따라서 정수를 사용하는 클래스를 또 만들었지만 같은 역할이 여러개의 클래스로 분리되는 것이기 때문에 객체지향 프로그래밍의 원칙에 어긋납니다.

애초에 Enum을 사용했다면 다음과 같이 사용할 수 있습니다.

```java
public enum Calendar {
    JANUARY("1월",1),
    FEBRUARY("2월",2),
    MARCH("3월",3),
    APRIL("4월",4),
    MAY("5월",5);
    
    private final String month;
    private final int monthNumber;
    
    Calendar(String month, int monthNumber) {
        this.month = month;
        this.monthNumber = monthNumber;
    }
    
    public String getMonth() {
        return month;
    }
    
    public int getMonthNumber() {
        return monthNumber;
    }
}
```

이렇게 사용하면, 코드의 가독성이 높아지고, 데이터들 간의 연관성을 나타낼 수 있습니다. 또 비슷한 요구사항이 생긴다면 여기에 더 추가하면 됩니다.

### 2. 상태와 행위를 한곳에서 관리

예를들어, 달 마다 할인 이벤트를 진행한다고 가정해봅시다. 이때, 달마다 할인율이 다르게 적용되어야 합니다.
만약 enum이 아닌 상수와 할인율을 분리한다면 다음과 같이 사용됩니다.

```java
public class Discount {
    public static final double JANUARY_DISCOUNT = 0.1;
    public static final double FEBRUARY_DISCOUNT = 0.2;
    public static final double MARCH_DISCOUNT = 0.3;
    public static final double APRIL_DISCOUNT = 0.4;
    public static final double MAY_DISCOUNT = 0.5;
    // ...
    
    
}

public class DiscountTest {
    public static void main(String[] args) {
        double discount = 0;
        String month = "JANUARY";
        
        if(month.equals("JANUARY")) {
            discount = Discount.JANUARY_DISCOUNT;
        } else if(month.equals("FEBRUARY")) {
            discount = Discount.FEBRUARY_DISCOUNT;
        } else if(month.equals("MARCH")) {
            discount = Discount.MARCH_DISCOUNT;
        } else if(month.equals("APRIL")) {
            discount = Discount.APRIL_DISCOUNT;
        } else if(month.equals("MAY")) {
            discount = Discount.MAY_DISCOUNT;
        }
        
        System.out.println(discount);
    }
}

```

**이렇게 사용해서 생기는 문제점은 다음과 같습니다.**
1. 계산코드를 중복 생성할 위험이 있습니다.
2. 할인율을 변경할 때, 코드를 수정해야 합니다.
3. 문자열과 메소드가 분리되어 있기 때문에, 이 계산 메소드를 써야함을 알 수 없어 누락할 수 있습니다.
4. Code에 따라 특정 메소드를 강제할 방법이 없습니다.

**이를 Enum을 사용하면 다음과 같이 사용할 수 있습니다.**

```java
public enum Discount {
    JANUARY("1", value -> value * 0.1),
    FEBRUARY("2", value -> value * 0.2),
    MARCH("3", value -> value * 0.3);
    
    private final String month;
    private final Function<Double, Double> discount;
    
    Discount(String month, Function<Double, Double> discount) {
        this.month = month;
        this.discount = discount;
    }
    
    public double calculate(double value) {
        return discount.apply(value);
    }
}
```

이렇게 사용하면, 할인율을 변경할 때, 코드를 수정할 필요가 없습니다. 또한, 할인율을 계산하는 메소드를 강제할 수 있습니다.


## 결론 

**Enum은 연관을 나타낼 수 있다는 점**에 더 다양하게 활용할 수 있습니다. 

하지만 **Enum을 사용할 때 인지 DB의 데이터를 가져오는 게 더 나을 지** 판단해야 합니다. 자칫 Enum을 사용했는 데 데이터의 변경이 심하다면 
코드를 **다시 수정하고 컴파일하는 번거로움**이 생길 수 있습니다. **필요할 때 적절하게 사용**한다면 정말 좋은 도구가 될 것 같습니다.


## Reference

- https://product.kyobobook.co.kr/detail/S000210144588
- https://techblog.woowahan.com/2527/
