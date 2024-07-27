---
title: "클래스 멤버변수 초기화 순서"
date: 2024-07-27
update: 2024-07-27
tags:
  - Java
  - Interview
---
## 멤버변수 초기화 순서

```java
public class InitializationExample {
     
     static int pk = callMe(1);
     int i = callMe(3);
   
     static {
        callMe(2);
     }
  
     {
       i= callMe(4); 
     }
 
     public InitializationExample() { i = callMe(5); }
   
  private static int callMe(int pk) {
    System.out.println(pk);
    return pk;
  }
   
 
  public static void main(String[] args) {
    InitializationExample init = new InitializationExample();
  }
  
}
```

1. **static 변수 & static block** : 클래스가 로드될때, 링킹단계에서 먼저 static 변수에 기본값으로 초기화하고 초기화단계에서 static 변수에 명시적 초기화 및 static 블록이 실행됩니다. static 블록은 클래스 로딩할 시점에
딱 한번만 실행됩니다.
2. **인스턴스 변수 & 인스턴스 블록** : Application 실행 도중 객체가 생성될때, Heap 영역에 해당 객체가 생성되고나서 기본 값으로 초기화합니다. 그후 명시적 초기화, 인스턴스 초기화 블록에 의해 변수가 초기화 됩니다.
3. **생성자** : 마지막으로 생성자에 의해 변수가 초기화 됩니다.

따라서 위 코드는 아래와 같은 순서대로 초기화 됩니다.

```java
1. static int pk = callMe(1); 
2. static { callMe(2); }
3. int i = callMe(3);
4. {i= callMe(4);}
5. public InitializationExample() { i = callMe(5); }
```

## Reference

https://coderanch.com/t/669246/certification/Initialization-Order-java-class

