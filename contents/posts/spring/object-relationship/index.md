---
title: "1장 오브젝트와 의존관계"
date: 2024-08-21
update: 2024-08-21
tags:
  - Spring
series: 토비의 스프링 정리
---

## 스프링이란

스프링은 객체지향 설계와 구현에 관해 특정한 모델과 기법을 억지로 강요하지 않습니다. 

하지만 객체를 어떻게 효과적으로 설계하고 구현하고, 사용하고, 이를 개선해나갈 것인가에 대한 명쾌한 기준을 마련해줍니다. 

동시에 스프링은 객체지향 기술과 설계,구현에 관한 실용적인 전략과 검증된 베스트 프렉티스를 평범한 개발자도 자연스럽고 손쉽게 적용할 수 있도록 프레임워크 형태로 제공합니다.

## 디자인 패턴이란

자주 만나는 문제를 해결하기 위해 사용할 수 있는 재사용 가능한 솔루션을 말합니다.

모든 패턴은 간결한 이름이 있어서, 잘 알려진 패턴을 적용하고자 할 때 간단히 패턴 이름을 언급하는 것만으로도 설계의 의도와 해결책을 함께 설명할 수 있다는 장점이 있습니다.
대부분 객체지향적 설계 원칙을 이용해 문제를 해결합니다. 

각 패턴을 적용할 상황, 해결해야 할 문제, 솔루션의 구조와 각 요소의 역할 및 핵심 의도가 무엇인지를 기억해둬야 합니다.

### 템플릿 메소드 패턴

이렇게 슈퍼클래스에 기본적인 로직의 흐름을 만들고, 그 기능의 일부를 추상 메소드나 오버라이딩이 가능한 protected 메소드 등으로 만든 뒤 서브클래스에서 이런 메소드를 필요에 맞게 구현해서 사용하는 방법입니다.

새로운 하위 클래스를 쉽게 추가할 수 있어, 확장성이 좋고 특정 알고리즘의 구조는 유지하기 때문에 일관성을 유지할 수 있는 역할을 갖고 있습니다. 

유사한 단계를 거치지만 세부 구현이 다르거나, 알고리즘의 특정 부분만 확장하고 싶을 때 사용하는 디자인 패턴 입니다.

```java
public abstract class Super{
    public void templateMethod(){
        //기본 알고리즘 코드..
        
        abstractMethod();
        hookMethod();
        //...
    }
    
    protected void hookMethod(){}
    
    public abstract void abstractMethod();
    
}
```

### 펙토리 메소드 패턴

서브 클래스에서 객체 생성 방법과 클래스를 결정할 수 있도록 미리 정의해둔 메소드를 팩토리 메소드라고 하고, 이 방식을 통해 객체 생성 방법을 나머지 로직, 즉 
슈퍼클래스의 기본 코드에서 독립 시키는 방법을 팩토리 메소드 패턴 이라 합니다.

### 전략 패턴

개방폐쇄원칙(OCP)의 실현에 가장 잘 들어 맞는 디자인 패턴입니다. 

자신의 기능(맥락-context)에서, 필요에 따라 변경이 필요한 기능을 인터페이스를 통해 통째로 외부로 분리시키고, 이 기능을 구현한 구체적인 클래스를 필요에 따라 바꿔서 사용할 수 있습니다. 


## 관계설정 책임의 분리

UserDao가 다른 책임을 가진 클래스의 구체적인 클래스를 알면 안됩니다. 만약 구체적인 클래스가 변경이 일어나면 UserDao도 역시 변경이 일어납니다. 다음 코드와 같습니다.

```java
public class UserDao{
    Connection connectionMaker;
    public UserDao(){
        connectionMaker = new DConnectionMaker();
    }
}
```

이런식으로 구성되면 구현 클래스가 변경이 일어나면 UserDao 역시 변경이 일어나기 때문에 책임의 분리가 필요한 시점입니다.

즉 UserDao가 어떤 구현 클래스의 객체를 이용할지는 독립적인 관심사 입니다. 그래서 **UserDao와 UserDao가 사용할 특정 구현 클래스 사이의 관계를 설정해주는 것은 
분리되어 다른 제3자가 맡아야 합니다.** 다른 관심사와 함께 있는 것은 확장성을 떨어뜨리는 문제이기 때문입니다.

이 문장을 기억해야 합니다. **구현체와의 의존관계는 클래스 다이어그램에서 나타나면 안되고, 런타임 시점의 오브젝트 간 관계에서 나타나야 합니다.**

## 제3자. 관계 설정 역할을 하는 스프링 IoC

### IoC 란

**IoC, 제어의 역전이란,** 자신의 흐름대로 관계를 설정하고 객체를 생성하지 않고, 제어권을 넘겨서 제어 권한을 위임받은 특별한 오브젝트가 오브젝트를 생성하고 관계를 설정합니다. 

제어의 역전 개념은 폭넓게 적용됩니다. 

예를들어, **템플릿 메소드 패턴**도 서브클래스가 정의해놓으면 상위 클래스의 템플릿 메소드에서 필요할 때 호출하여 사용하기 때문에 제어의 역전을 적용한 패턴이라 볼 수가 있습니다.
또 **프레임워크도** 개발한 클래스를 등록해두면, 프레임워크가 흐름을 주도하는 중에 개발자가 만든 애플리케이션 코드를 사용하도록 만드는 방식입니다. 

제어의 역전에서는 애플리케이션 컴포넌트의 생성과 관계설정, 사용, 생명주기 관리등을 관장하는 존재가 필요합니다. 이때 IoC 프레임워크의 도움을 받으면 애플리케이션 전반적으로 가능하도록 도와줍니다.
**대표적인 IoC 프레임워크가 바로 스프링이 제공하는 IoC 입니다.**

### 스프링 IoC 용어 정리

- 빈(Bean) : 스프링에게 직접 관리되는 오브젝트입니다.
- 빈 팩토리(Bean Factory) : 인터페이스이고, 스프링의 IoC를 담당하는 핵심 컨테이너를 가르킵니다. 빈을 등록하고, 생성하고, 조회하고, 돌려주고 등등 부가적인 빈 관리하는 기능을 담당합니다. 
- 애플리케이션 컨텍스트(Application Context) : 빈 팩토리를 확장했으며, 그 뿐아니라 스프링이 제공하는 애플리케이션 지원 기능이 모두 포함되어 있습니다. 
- 설정 정보(meta data) : 애플리케이션 컨택스트 또는 빈 팩토리가 IoC를 적용하기 위해 사용하는 메타정보를 말합니다.
- 컨테이너 또는 IoC 컨테이너 : 스프링 컨테이너 또는 애플리케이션 컨텍스트와 같은 말입니다. 

### 그냥 오브젝트 팩토리 클래스에 비하여 스프링 컨테이너를 사용했을 때 장점

1. 클라이언트는 구체적인 팩토리 클래스를 알 필요없습니다. 특정 팩토리 구현 클래스가 아니고 일관되게 원하는 오브젝트를 가져올 수 있기 때문입니다.
2. 애플리케이션 컨텍스트는 종합 IoC 서비스를 제공합니다. 오브젝트 생성, 다른 오브젝트와의 관계 설정, 오브젝트가 만들어지는 방식,시점,전략을 다르게 가져갈 수 있는 등 다양한 기능을 제공합니다.
3. 애플리케이션 컨텍스트는 빈을 검색하는 다양한 방법을 제공합니다. `getBean()` 등


## 싱글톤 패턴이란

어떤 클래스를 애플리케이션 내에서 단일 오브젝트로 존재하게 하고, 이를 애플리케이션 여러 곳에서 공유합니다.

자바에서 구현하는 방법은 다음과 같습니다.

1. 클래스 밖에서 생성못하도록 생성자를 private으로 만든다.
2. 생성된 싱글톤 오브젝트를 저장할 수 있는 자신과 같은 타입의 스태틱 필드를 정의
3. `getInstance()`를 만들고 최초로 호출되는 시점에서 한번만 오브젝트가 만들어지게 합니다. 생성된 오브젝트는 스태틱 필드에 저장
4. 한번 오브젝트가 만들어지고난 후 `getInstance()` 메소드를 통해 이미 저장해둔 오브젝트를 넘깁니다.

```java
public class UserDao {
    private static UserDao INSTANCE;
    
    private UserDao(ConnectionMaker connectionMaker){
        this.conntectionMaker = connectionMaker;
    }
    
    public static synchronized UserDao getINSTANCE(){
        if(INSTANCE == null) INSTANCE = new UserDao(???);
        return INSTANCE;
    }
}
```

### 싱글톤 패턴의 한계

- private 생성자를 갖고 있어 상속이 불가능합니다. 이는 상속을 이용한 다형성 적용이 불가능하기 때문에 객체지향적이지 않습니다.
- 싱글톤은 테스트하기 어렵습니다. 생성자를 통해 다이나믹하게 주입 불가능합니다.
- 서버환경에서는 싱글톤이 하나만 만들어지는 것을 보장하지 않습니다.
- 싱글톤의 사용은 전역 상태를 만들 수 있기 때문에 여기저기서 수정할 수 있어서 객체지향적으로 바람직하지 못합니다.

### 싱글톤 레지스트리

스프링은 오브젝트를 싱글톤으로 구현하도록 적극 지지합니다. 

하지만 위처럼 여러 단점 때문에 스프링이 직접 싱글톤 형태의 오브젝트를 만들고 관리하는 기능을 제공합니다. 

private 생성자가 아니고 static도 안쓰고 오로지 평범한 자바 클래스라도
IoC 방식의 컨테이너를 사용해서 싱글톤 방식으로 관리할 수 있습니다. 어떠한 제약도 없기 때문에 위 단점을 모두 커버합니다. 이 또한 스프링 컨테이너를 사용하는 이유중 하나입니다.

다만 주의할 점은, 멀티스레드환경에서 읽기/쓰기 다 되는 인스턴스 변수를 가지고 있으면 안됩니다.


## DI(의존 관계 주입)

앞서 IoC는 일반적이고 포괄적인 개념입니다. 이를 가지고서 IoC = 스프링이다 라고 말하기는 좀 근거가 부족합니다. 

그래서 의존관계 주입 이라는 좀 더 의도가 명확히 드러나는 이름을 사용합니다.
**즉, 스프링 IoC의 기능의 대표적인 동작원리는 의존관계 주입이라는 차별적인 기능을 제공합니다.**

의존관계 주입은 구체적인 의존 오브젝트와 그것을 사용할 주체인 오브젝트를 런타임 시에 연결해주는 작업을 말합니다. 다음 세가지 조건을 충족하면 DI라고 할 수 있습니다.

1. 클래스모델이나 코드에는 런타임 시점의 의존관계가 드러나지 않습니다. 이렇게 구현하려면 인터페이스에만 의존하고 있어야 합니다.
2. 런타임 시점의 의존관계는 컨테이너나 팩토리 같은 제 3자의 존재가 결정합니다.
3. 의존관계는 사용할 오브젝트에 대한 레퍼런스를 외부에서 주입(제공)해줌으로써 만들어집니다.

### 오브젝트의 레퍼런스를 전달해주는 방법

1. 생성자를 통한 주입 
2. 메소드를 통한 주입 (수정자 메소드 setter, 일반 메소드를 이용한 주입)
3. XML을 통한 주입
4. 그외에도 다양한 주입 방식 (Autowired)


## Reference

토비의 스프링 Vol.1