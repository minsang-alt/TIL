---
title: "static에 대한 설명"
date: 2024-07-27
update: 2024-07-27
tags:
  - Java
  - Interview
---

## Static 이란

자바에서 Static 키워드는 클래스 레벨의 변수나 메소드에 사용되며, 이를 통해 객체 생성 없이 해당 변수나 메소드에 
접근할 수 있습니다. **Static 멤버는 클래스가 메모리에 로드될 때 단 한 번만 생성**되며, 모든 인스턴스가 공유하는 자원입니다.

## Static 메모리 위치 

**자바 8 이전에는** Permanent 영역에 클래스의 meta data, interned String, Constant Pool, Static 변수, 메소드 등이 할당되었습니다.
하지만 Permanent 영역의 큰 단점이 JVM에 의해 관리되며 할당된 크기가 제한되며, OOM 발생 확률이 매우 높았습니다. 

따라서 **자바 8이후로** **static variable, static method, static 객체**는 힙 영역에 저장되고
Class Meta Data, Constant Pool, reference은 method 영역(Meta Space)에 저장되고 참조대상이 없어졌을 때 GC 대상이 됩니다.

## 정적 변수, 정적 메소드를 사용하는 이유

정적변수 같은 경우, 각 객체마다 고유한 변수가 아니고, 변수 값이 객체와 독립적인 경우 일때, 모든 인스턴스가 해당 변수를 알아야할 경우에 사용됩니다.

정적 메소드의 경우 유틸리티 작업을 수행하는 데 유용합니다. 객체 선언 없이 호출할 수 있습니다. 따라서 메모리 효율 관점에서도 좋습니다.

하지만 객체지향의 개념을 생각해보면 객체란, 상태와 행위로 이루어진 일종의 대상 입니다. 또한 객체지향의 특성 중 하나인 캡슐화 원칙도 깨집니다. 그런데 static은 이 **객체지향 패러다임을 깨뜨립니다.**
또한, 오버라이딩을 할 수 없는 static 멤버들 때문에 **클래스를 확장하기도 어렵습니다.** 




## Reference 

https://jgrammer.tistory.com/entry/JAVA-Java8%EB%B6%80%ED%84%B0%EB%8A%94-static%EC%9D%B4-heap%EC%98%81%EC%97%AD%EC%97%90-%EC%A0%80%EC%9E%A5%EB%90%9C%EB%8B%A4

