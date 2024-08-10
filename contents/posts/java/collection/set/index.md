---
title: "Set"
date: 2024-07-22
update: 2024-07-22
tags:
  - Java
  - Collection 

series: "Collection"
---

![](img.png)

**Set은 순서에 상관없이, 어떤 데이터가 존재하는 지를 확인하는 용도로 많이 사용**됩니다. 즉, **중복되는 것을 방지**하고, 원하는 값이 
포함되어 있는지를 확인하는 것이 용도입니다.

자바에서 Set 인터페이스를 구현한 대표 클래스는 `HashSet` , `TreeSet`, `LinkedHashSet`이 있습니다.

- `HashSet` : 순서가 전혀 필요없는 데이터를 해시 테이블에 저장합니다. Set 중에 가장 성능이 좋습니다.
- `TreeSet` : 저장된 데이터의 값에 따라서 정렬되는 셋입니다. **Red-Black** 트리 타입으로 값이 저장되며, HashSet 보다 약간 성능이 느립니다.
- `LinkedHashSet` : 연결된 목록 타입으로 구현된 해시 테이블에 데이터를 저장합니다. 저장된 순서에 따라서 값이 정렬됩니다. 성능이 가장 나쁩니다.

## HashSet

### HashSet 클래스의 상속관계

```java
java.lang.Object
 ㄴ java.util.AbstractCollection<E>
    ㄴ java.util.AbstractSet<E>
       ㄴ java.util.HashSet<E>
```
