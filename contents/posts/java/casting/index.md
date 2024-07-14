---
title: "참조 자료형의 형 변환"
date: 2024-07-14
update: 2024-07-14
tags:
  - Java
---

**다형성**을 이해하기 위해서는 먼저 형변환에 대해 확실히 알아야 합니다. 이 절의 실습을 위해 먼저 Parent와 Child 클래스를 만들겠습니다.
```java
public class Parent {
    
    public Parent() {
    }
    
}

static class Child extends Parent {
    
    public Child() {
    }
    
    public void printName() {
        System.out.println("Child");
    }
} 
```

위와 같이 상속관계가 성립되면, 다음과 같이 객체를 생성할 수 있습니다. 

```java
Parent obj = new Child();
```

또한 원시타입의 형변환에서 int -> long 으로 확장될 때는 명시적 형변환이 필요없는것 처럼
**자식 클래스에서 부모클래스로 형변환이 일어날 때는 명시적 형변환이 필요가 없습니다.**


## 주의할 점 
```java
public class InheritanceCasting {
    //...
    public void cast() {
        Parent parent = new Parent();
        Child child = new Child();
        
        Parent parent2 = child; // 1.
        Child child2 = parent; // 2.
        
        Child child3 = (Child) parent2; // 3.
    }
    
}
```
1번은 정상적으로 캐스팅이 되지만 2번은 컴파일이 되지 않습니다. 명시적 형변환을 사용하여 `Child child2 = (Child)parent`로 하면 컴파일 에러는 발생하지 않지만 `ClassCastException`이
발생합니다.

하지만 3번 같은 경우에는 `parent2`에 Child 클래스의 객체인 `child`을 대입한 상태입니다. 따라서 실제로는 Child 클래스이기 때문에 `parent2`를 Child 클래스로 형 변환해도 문제가 없습니다.

### 결론

참조 자료형의 형변환은 상속 관계에 있는 클래스 간에 이루어지며, **자식에서 부모로의 업캐스팅은 문제없이 무조건 가능하며**, **부모에서 자식으로의 다운캐스팅**은 **부모의 참조가 실제 자식객체를 가르킬때만 명시적 형변환**이 가능합니다.

## instanceof

```java
public void cast(){
    
    Parent[] parents = new Parent[3];
    
    parents[0] = new Child();
    parents[1] = new Parent();
    parents[2] = new Child();
}
```
이렇게 일반적으로 **여러 개의 값을 처리하거나, 매개 변수로 값을 전달할 때는 보통 부모 클래스의 타입으로 보냅니다**. 이렇게 하지않으면 배열과 같이 여러 값을 한번에 보낼 때 각 타입별로 구분해서
메소드를 만들어야 하는 문제가 생기기 때문입니다.

그런데 `Parent`타입의 배열을 받고 Parent인지 Child인지 구분이 필요합니다. 이때 사용하는 것이 `instanceof`라는 예약어 입니다.
```java
private void typeCheck(Parent[] parents){
    
    for(Parent tempParent : parents){
        
        if(tempParent instanceof Child){
            System.out.println("this is Child");
            Child tempChild = (Child) tempParent;
            tempChild.printName();
            
        }else if(tempParent instanceof Parent){
            System.out.println("this is Parent");
        }
    }
}
```

**출력**

```text
this is Child
Child
this is Parent
this is Child
Child
```

이렇게 `instanceof`를 사용하면 정확하게 타입을 확인할 수 있고 메소드도 문제없이 사용할 수 있습니다. 

하지만 유의할 점이 있습니다. `tempParent`는 부모타입인 Parent와 자식타입인 Child 모두에 true을 반환합니다. 따라서 if - else if 문의 위치가 바뀌면 제대로된 타입검증이 불가능합니다.

따라서 `instanceof`는 타입 확인을 할 때 부모 타입도 true 결과를 반환하므로 **가장 하위에 있는 자식 타입부터 확인**해야 정상적으로 수행이 가능합니다. 


## Reference

https://product.kyobobook.co.kr/detail/S000210144588







