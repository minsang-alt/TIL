---
title: "StringBuilder와 StringBuffer의 차이"
date: 2024-07-30
update: 2024-07-30
tags:
  - Java
  - Interview
---

## String, StringBuffer, StringBuilder

String 클래스는 **불변(immutable)**이기 때문에 **문자열을 변경할 때마다 새로운 객체를 생성**합니다. 이러한 작업은 **메모리 낭비**를 초래하며 새롭게 객체를 생성하는 비용 때문에 **성능 저하**를 일으킵니다.

이 단점을 보완하기 위해 `StringBuffer`와 `StringBuilder` 클래스가 등장했습니다.

이 둘의 기능은 같지만 `StringBuilder`는 **Thread safe 하지 않습니다**. 하지만 그만큼 속도가 빠릅니다.
반면, `StringBuffer`는 멀티스레드 환경에 유리하게 **임계영역을 설정했기때문에 Thread safe 하지만**  속도가 느립니다.

## 문자열을 연결하는 내부구현 차이

### String + 연산

String의 + 연산은 버전마다 계속 최적화 되었습니다. 
**자바 5버전부터는 성능 향상을 목적으로 컴파일시 `StringBuilder` 혹은 `StringBuffer`로 변환되도록 변경되었습니다.**

하지만 loop 같은 상황에서 `StringBuilder`가 계속 생성된다는 단점과 `StringBuilder`나 `StringBuffer`의 생성자로 생성하지 않으면 초기 용량이 16 character로 셋팅되는 데 이는 적은 용량이여서 재할당으로 인한 추가 비용이 발생하기 쉬웠습니다.

**JDK 9부터 `StringConcatFactory`을 이용하여 더 효율적으로 처리**를 합니다.(이는 String을 매번 이어붙이지 않고 `MethodHandle`을 통해 기억하고 있다가 한번만 byte 배열에 할당하는 방식입니다.)

### String concat 연산

항상 `concat` 메소드를 사용할 때마다 `new` 키워드를 사용해 새롭게 객체를 생성하기 때문에 매우 비효율적입니다.

### StringBuffer와 StringBuilder

문자열을 `CharSequence`의 내부 버퍼에 저장하고 참조변수 `this`를 통해 본인 인스턴스를 반환하기 때문에 객체를 새롭게 생성하는 과정이 없어 성능이 좋습니다.

## Reference

https://jerry92k.tistory.com/50







