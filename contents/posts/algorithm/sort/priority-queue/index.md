---
title: "우선순위 큐"
date: 2024-07-10
update: 2024-07-11
tags:
  - 알고리즘
---

우선순위 큐는 우선순위가 높은 데이터 부터 먼저 처리하는 큐이다.

### 동작 순서
1. 빈 우선순위 큐 선언 (작은 값일수록 우선순위 기준이 높다고 가정)
2. 10을 삽입한다. 빈 큐이므로 별다른 우선순위 생각하지 않고 맨 앞에 추가 
3. 이어서 5 삽입을 한다. 5는 10보다 작으므로 우선순위가 높다. 따라서 5을 10 앞으로 위치시킨다.
4. poll(추출)할때는 5가 10보다 앞에 있으므로 poll하면 5가 나온다.

### 힙으로 우선순위 큐를 구현해야 하는 이유 

우선순위 큐를 힙으로 구현할 때 최대힙, 최소힙과 같이 특정 값을 루트 노드에 유지하는 특징이 앞서 우선순위 큐의 핵심 동작과 맞아 떨어진다.

게다가 자바는 힙으로 구현된 `PriorityQueue` 클래스로 제공한다. 이를 활용하면 쉽게 우선순위 큐를 구현할 수 있다.

### 우선순위 큐 선언시 우선순위 기준 세우기
```java
// 데이터 자체에 우선순위를 직접 정할때 Comparator 객체를 생성하여 인자에 넘긴다
PriorityQUeue<Integer> pq = new PriorityQueue<>(Comparator.comparingInt(o->o.charAt(1)));
pq.add("15");
pq.add("23");
System.out.println(pq.poll()); // 23

// 값이 클수록 우선순위가 높다고 가정할때
PriorityQUeue<Integer> pq = new PriorityQueue<>(Collections.reverseOrder());
```

### 값을 한꺼번에 우선순위 큐에 넣을 때
```java
ArrayList<Integer> list = new ArrayList<>();
list.add(10);
list.add(4);
list.add(1);

PriorityQueue<Integer> pq = new PriorityQueue<>(list);
System.out.println(pq.poll()); // 1
System.out.println(pq.poll()); // 4
System.out.println(pq.poll()); // 10

System.out.println(pq); // [ ]

pq.addAll(list);
System.out.println(pq); // [1, 10, 4]
```
`System.out.println(pq); // [1, 10, 4]` 의 출력값은 우선순위 큐가 add 할때, 정렬된 상태가 아님을 알 수 있다. <br>

코드를 보면 es[]에 힙 자료구조 원칙 대로 저장한다. 이는 poll()할 때 정렬된 상태로 NlogN만에 뽑을 수가 있다.
```java
private static <T> void siftUpComparable(int k, T x, Object[] es) {
    Comparable<? super T> key = (Comparable<? super T>) x;
    while (k > 0) {
        int parent = (k - 1) >>> 1;
        Object e = es[parent];
        if (key.compareTo((T) e) >= 0)
            break;
        es[k] = e;
        k = parent;
    }
    es[k] = key;
}
```
### 우선순위 큐의 시간 복잡도
<br>

- `addAll()`은 O(NlogN)
- `add() / poll()` 둘 다 O(logN)