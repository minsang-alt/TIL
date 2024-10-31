---
title: "(3) 데이터베이스 동시성 제어"
date: 2024-10-29
update: 2024-10-29
tags:
  - lock
series: "동시성 이슈 제어 기법"
---

데이터베이스에서 트랜잭션을 **하나씩 순차적으로 실행**하면 동시성 문제가 발생하지 않습니다. 그러나 이 방법은 성능 저하를 초래할 수 있습니다.

현실적으로는 여러 서버에서 네트워크를 통해 다수의 트랜잭션이 동시에 데이터베이스에 접근합니다. 따라서 동시성 문제를 해결하는 다른 방법이 필요합니다.

## 동시성 문제를 해결하면서 성능을 유지하는 방법

동시에 여러 트랜잭션이 실행될 때에도, 마치 하나씩 순차적으로 실행된 것처럼 보이도록 만들어야 합니다.

### 🤔 먼저 알아야 할 개념: Conflicting Operations

트랜잭션 간 충돌이 발생할 수 있는 조건:
1. 서로 다른 트랜잭션에서 수행되는 연산일 것
2. 같은 데이터를 사용할 것
3. 최소 하나가 쓰기 연산일 것

이 세 가지 조건을 모두 만족할 때 “충돌 가능성”이 있다고 판단합니다.

### 트랜잭션의 순차 실행
![](img.png)


위 그림에서 충돌 조건을 충족하는 연산은 `T1-R(B)-> T2-W(B)` , `T1-W(B) -> T2-R(B)` , `T1-W(B) -> T2-W(B)` 입니다.
하지만 T1과 T2가 순서대로 실행되므로 충돌이 발생하지 않았습니다.

### 트랜잭션의 동시 실행 

![](img_1.png)

성능 저하를 막기 위해 동시 실행하는 경우, 충돌 조건을 충족하는 연산은 `T1-R(B) -> T2-W(B)`, `T1-W(B) -> T2-W(B)`, `T2-R(B) -> T1-W(B)`입니다. 
실제로 동시성 문제가 발생했습니다.

동시에 실행하더라도 서로 다른 스케줄에서 연산 순서를 일치시킨다면 동시성 문제를 피할 수 있습니다. 
유일하게 순서가 일치하지 않는 `T2-R(B) -> T1-W(B)` 부분의 순서를 `T1-W(B) -> T2-R(B)`로 맞추면 문제가 해결됩니다.

아래처럼 순서를 변경하면 동시성 문제 없이 트랜잭션을 동시 실행할 수 있습니다.

![](img_2.png)

이렇게 충돌 가능성이 있는 임계 영역에서 트랜잭션들이 **일관된 순서**를 지키면 데이터베이스에서 동시성 제어가 가능합니다.
이를 통해 순차 실행 시의 성능 저하 문제를 해결하면서도, 동시성 문제를 방지할 수 있습니다. 이 방식을 **Conflicting Equivalent**라 하며, 이는 Serializable Schedule 방식으로 동작함을 의미합니다.

## Serializable Schedule: 동시성 제어와 성능을 모두 잡는 방법

위와 같은 연산 순서를 보장(Conflicting Equivalent)하기 위해 여러 가지 구현 방식이 존재합니다.

### Isolation - Serializable (X)

Serializable은 Java에서 흔히 사용되는 방법으로, 동시성 문제가 없는 가장 간단한 방법입니다. 
그러나 하나씩 순차적으로 실행되기 때문에 긴 연산이 포함된 트랜잭션이 있을 경우 성능 저하가 발생합니다. 이 때문에 실무에서는 잘 사용되지 않습니다.

보통은 **READ COMMITTED** 격리 수준을 설정하는데, 이로 인해 **Non-Repeatable Read** 문제가 발생할 수 있어 동시성 문제가 생길 수 있습니다. 따라서, READ COMMITTED를 사용하되 다른 동시성 제어 방법을 함께 적용해야 합니다.

### 데이터베이스에서의 낙관적 락 

낙관적 락은 데이터 충돌이 거의 없다고 가정하는 방식입니다. 트랜잭션이 완료되기 전에 다른 트랜잭션이 데이터를 수정하지 않았는지 **버전 정보를 통해** 확인합니다. (충돌 감지)

```java
// 상품 엔티티
class Product {
    private Long id;
    private String name;
    private int stock;
    private Long version; // 버전 정보
}

// 상품 구매 시
public void purchase(Long productId) {
    Product product = productRepository.findById(productId);
    Long originalVersion = product.getVersion();
    
    // 재고 감소
    product.decreaseStock(1);
    
    // 저장 시 버전 체크
    if (originalVersion != product.getVersion()) {
        throw new OptimisticLockException("다른 사용자가 이미 수정했습니다");
    }
}
```

버전 정보가 처음 값과 다를 경우 다른 트랜잭션이 수정을 했다고 판단하여 예외 처리를 합니다. **데이터베이스는 충돌을 감지하지만, 실패 처리는 하지 않습니다.**

장점
- 락을 사용하지 않아 높은 처리량을 유지합니다.
- 데드락이 발생하지 않습니다.

단점
- 충돌이 자주 발생하면 재시도로 인한 오버헤드가 증가합니다.
- 실패 후 복구 과정을 직접 코드로 작성해야 하므로 개발자의 작업이 필요합니다.

사용 예시
- 사용자 수가 적거나 충돌이 거의 발생하지 않는 경우 유용합니다.

### 데이터베이스의 비관적 락 

비관적 락은 **공유락과** **베타락을** 사용하여 동시성 문제를 제어하는 방식입니다.

- **공유락**은 읽기 전용 락으로, 이 락이 걸려 있는 동안 다른 커넥션은 베타락을 얻을 수 없습니다.
- **베타락**은 읽기와 쓰기 작업이 모두 가능한 락으로, 이 락이 걸려 있는 동안 다른 커넥션은 어떠한 락도 얻을 수 없습니다.

장점
- 데이터에 직접 락을 걸어 데이터 충돌을 방지합니다.
- Serializable 격리 수준이 트랜잭션 내 모든 자원에 락을 거는 것과 달리, 비관적 락은 특정 자원에만 락을 걸어 **병목 현상을 줄입니다.**

단점
- 데드락 발생 가능성이 높습니다.
- 락 대기로 인한 병목 현상이 발생할 수 있습니다

**여기서 말하는 자원이란 어떤건가요?**

데이터베이스마다 테이블일수도, 특정 row일 수도 있습니다. MySQL은 인덱스를 기준으로 락을 걸 수 있습니다.
즉 PK도 인덱스이므로 PK인 row에 락을 겁니다. 

**사용예시**

```java
@Service
class OrderService {
    @PersistenceContext
    private EntityManager em;
    
    @Transactional
    public void createOrder(Long productId, int quantity) {
        try {
            // 비관적 락으로 쓰기작업으로 상품 조회
            Product product = em.find(Product.class, productId,
                                    LockModeType.PESSIMISTIC_WRITE); // 쿼리: select * from product where product_id = :productId FOR UPDATE
            
            // 재고 감소
            product.decreaseStock(quantity);
            
            // 주문 생성
            Order order = new Order(product, quantity);
            em.persist(order);
            
        } catch (PessimisticLockException e) {
            throw new OrderFailedException("일시적인 오류. 다시 시도해주세요.");
        }
    }
}
```

트랜잭션이 종료될 때까지 락을 유지하며, 락을 획득하지 못하면 타임아웃까지 기다리다 예외가 발생합니다.

**비관적락이 데드락 위험이 높은 이유가 뭔가요?**

데드락이 발생하려면 4가지가 충족되어야 합니다. 

1. 상호배제

```java
// 한 번에 하나의 트랜잭션만 자원 사용 가능
@Lock(LockModeType.PESSIMISTIC_WRITE)  // 배타적 락
Product product = em.find(Product.class, id);
```

2. 점유와 대기 (Hold and Wait)

```java
@Transactional
public void transfer(Long fromId, Long toId) {
    // 자원1 점유하고 있으면서
    Account from = em.find(Account.class, fromId, 
                          LockModeType.PESSIMISTIC_WRITE);
    
    // 다른 자원2를 요청
    Account to = em.find(Account.class, toId,  // 대기 발생 가능
                        LockModeType.PESSIMISTIC_WRITE);
}
```

3. 비선점

```java
// 다른 트랜잭션이 락을 강제로 빼앗을 수 없음
Transaction 1:
Account from = em.find(Account.class, 1L, LockModeType.PESSIMISTIC_WRITE);
// 트랜잭션2는 이 락을 강제로 해제 불가능
```

4. 순환 대기

```java
// 트랜잭션 1
@Transactional
public void process1() {
    Account acc1 = em.find(Account.class, 1L,   // acc1 락 획득
                          LockModeType.PESSIMISTIC_WRITE);
    Thread.sleep(100);  // 데드락 상황 만들기
    Account acc2 = em.find(Account.class, 2L,   // acc2 락 대기
                          LockModeType.PESSIMISTIC_WRITE);
}

// 트랜잭션 2
@Transactional
public void process2() {
    Account acc2 = em.find(Account.class, 2L,   // acc2 락 획득
                          LockModeType.PESSIMISTIC_WRITE);
    Thread.sleep(100);  // 데드락 상황 만들기
    Account acc1 = em.find(Account.class, 1L,   // acc1 락 대기
                          LockModeType.PESSIMISTIC_WRITE);
}
```

MySQL 등에서는 데드락 발생 시 이를 감지하고 실패 처리를 통해 락을 반납하는 기능도 지원합니다.







