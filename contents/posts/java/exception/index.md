---
title: "예외 처리"
date: 2024-07-28
update: 2024-07-28
tags:
  - Java
  - Interview
---

## Exception 과 Error 차이 

Error는 자바 프로그램 밖에서 발생한 예외를 말합니다. 가장 치명적인 오류로 가장 흔한 예가 서버의 디스크가 고장나거나 메인보드가 맛이 가서 자바프로그램이 동작을 못하는 경우입니다.
Exception은 try-catch로 프로그램의 비정상 종료를 막을 수 있으며 프로그램을 계속 진행할 수 있습니다. 즉 쓰레드의 종료를 막습니다.
반면, Error는 프로세스에 영향을 주는 오류이며 프로그램을 더 이상 진행할 수가 없습니다. 

## RuntimeException

일반적으로 `RuntimeException`은 프로그래밍 방식으로 예외를 방지할 수 있습니다. 그리고 `RuntimeException`에서 파생된 클래스는 `uncheckedException`입니다.
따라서 `RuntimeException`은 호출코드에서 try-catch 문으로 둘러싸거나 throws 키워드를 넣는 일을 할 필요가 없습니다. 

반면 Exception 클래스에서 파생된 모든 클래스들은 `chekcedException` 입니다. 따라서 명시적으로 예외를 처리하지 않으면 컴파일 에러가 발생합니다.

CheckedException은 호출 코드에서 예외를 처리하도록 강제합니다. 이는 복구가 가능한 예외를 처리하도록 try-catch문을 사용하도록 강제합니다.
반면, RuntimeException인 언체크 예외는 복구할 수 없는 예외로 간주하여 예외처리를 강제하지 않습니다. 

## finally 블록을 사용하는 이유

`finally`와 연결된 try 블록으로 진입을 하면, 무조건 실행되는 영역이 `finally` 블록입니다.

**DB 서버와의 연결을 끊어주어야 할때**, 정상적인 흐름이 아닌 예외 상황에 DB 연결이 해제되지 않는 상황이 발생하여
문제가 발생합니다. 이때 `finally` 블록에서 DB 연결을 해제하는 코드를 작성하면, 예외 발생 여부와 상관없이 항상 DB 연결을 제대로 해제할 수 있습니다.

그외에도 반납해주어야 하는 자원들은 `Closable`인터페이스를 구현하고 있으며, 사용 후에 close 메소드를 호출해줘야 합니다.

## try-with-resources 란?

`finally` 블록안에서도 예외가 발생할 수 있다면 `try-catch`문을 또 둘러쌓야하는 상황이 발생합니다.또는 **자원의 null 값**도 체크해야하기 때문에 이는 **코드가 복잡**해집니다.
또는 **에러 스택 트레이스가 누락되어 디버깅이 어렵습니다**.

```java
import java.io.IOException;
...
finally{
    if(is !=null){
        try{
            is.close();
        }catch(IOException e){
            ...
        }
    }
    
    if(bis != null){...}
}
```

하지만 `try-with-resources`를 사용해 자원을 반납하도록 하면 이런 문제점을 다 해결할 수 있습니다. 
**try에 자원 객체를 전달하면, try 코드블록이 끝나면 자동으로 자원을 종료해주는 기능** 입니다.

**예시**
```java
try(Something1 s1 = new Something1();
    Something2 s2 = new Something2()){
    ...
}
```

이는 `AutoCloseable` 인터페이스를 구현한 클래스만 사용할 수 있지만 기존에 `Cloaseable` 인터페이스를 구현한 클래스들은 코드의 수정없이 바로 사용할 수 있습니다.
왜냐면 아래와 같이 `Closeable` 부모 인터페이스로 `AutoCloseable`를 추가했기 때문입니다.

```java
public interface Closeable extends AutoCloseable {
    public void close() throws IOException;
}
```




