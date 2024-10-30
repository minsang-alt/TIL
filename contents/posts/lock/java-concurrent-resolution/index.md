---
title: "(2) 자바에서의 동시성 이슈 해결"
date: 2024-10-30
update: 2024-10-30
tags:
  - lock
series: "동시성 이슈 제어 기법"
---

## 자바에서의 동기화 기법 - 메모리 영역

자바 메모리같은 경우, 하나의 프로세스에서 점유하고 있고 해당 프로세스 내에서
여러 스레드들이 공유하고 있는 자원이 있습니다.

이 자원은 힙 영역에 존재하는 객체, static 변수 값을 말합니다. 

```java
public class Counter {
    public int count = 0;
    
    // 임계영역
    public void increment(){
        count++;
    }
}

public class CounterTest {
    public static void main(String[] args) {
        Counter counter = new Counter();
        
        Runnable task = () -> {
            for (int i = 0; i < 10000; i++) {
                counter.increment();
            }
        };
        
        Thread thread1 = new Thread(task);
        Thread thread2 = new Thread(task);
        
        thread1.start();
        thread2.start();
        
        thread1.join();
        thread2.join();

        System.out.println("total count: " + counter.count);
    }
}
```

기대결과는 20000이 나와야 하지만, 위 코드에서는 20000보다 작은 값이 나올 수 있습니다.

왜냐하면 스레드 2개가 non-serial schedule 방식으로 겹쳐서 실행하고 있습니다.
또한 공유자원에 접근하고, 동기화없이 접근하며 하나의 스레드가 자원을 변경한다는 조건을 모두 충족하기 때문에
동시성 이슈가 발생한 것입니다.

## 해결방법

### synchronized

```java
    public synchronized void increment(){
        count++;
    }
```

단순히 임계영역인 increment()에 synchronized 키워드를 붙이면 됩니다. 

이는 락을 획득하지 못한 다른 스레드가 블로킹이 되어 대기하다가 실행되고 있는 스레드가 끝나 락을 반납했을때
실행됩니다.

### Atomic 연산

Atomic 연산은 CAS 연산을 사용합니다. 이 또한 스레드가 직렬적(동기적)으로 하나의 스레드가 끝나면 다른
스레드가 실행되는 방식으로 동시성 이슈를 해결합니다.

### Reentrant Lock

비슷한 방식이지만, 
synchronized와 다르게 대기하고 있는 스레드가 blocked 상태가 아닌 wait 상태인점이 있고 따라서 다른 스레드에 의해 깨울 수 있어 무한대기 문제를 해결하고 

특정 공간에 속한 스레드를 깨울 수 있기 때문에 생산자-소비자 문제도 해결할 수 있다는 장점이 있습니다.

## 자바에서의 동기화 기법 - 디스크 

디스크에 있는 파일에서 동시성 이슈를 해결하기 위해서는 OS레벨 제어가 필요합니다.

OS에서 프로세스 레벨 락을 획득하고, 락을 얻은 프로세스는 파일에 접근가능하므로 
프로세스 내의 스레드끼리 공유할때는 임계영역의 설정도 필요합니다.

```java
public class FileLockExample {
    
    public void writeToFile(String fileName, String data) {
        // 프로세스 레벨 락을 위한 FileLock 사용
        try (RandomAccessFile file = new RandomAccessFile(fileName, "rw");
             FileChannel channel = file.getChannel()) {
            
            // 파일에 대한 배타적 락 획득 시도
            FileLock lock = channel.lock();
            try {
                // 임계 영역 시작
                synchronized (this) {  // 프로세스 내 스레드 동기화
                    file.seek(file.length());  // 파일 끝으로 이동
                    file.writeBytes(data);     // 데이터 쓰기
                }
                // 임계 영역 끝
            } finally {
                lock.release();  // 락 해제
            }
            
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    
    public static void main(String[] args) {
        FileLockExample example = new FileLockExample();
        
        // 여러 스레드에서 파일 쓰기 시도
        Runnable writeTask = () -> {
            String threadName = Thread.currentThread().getName();
            example.writeToFile("test.txt", 
                "Written by " + threadName + "\n");
        };
        
        // 5개의 스레드 생성
        for (int i = 0; i < 5; i++) {
            new Thread(writeTask).start();
        }
    }
}
```

