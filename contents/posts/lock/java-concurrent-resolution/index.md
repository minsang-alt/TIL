---
title: "(2) 자바에서의 동시성 이슈 해결"
date: 2024-10-28
update: 2024-10-28
tags:
  - lock
series: "동시성 이슈 제어 기법"
---

## 자바에서의 동기화 기법 - 메모리 영역

자바에서 하나의 프로세스는 메모리를 점유하고 있으며, 그 프로세스 내의 여러 스레드가 자원을 공유하게 됩니다. 여기서 공유되는 자원은 주로 힙 영역에 존재하는 객체와 `static` 변수 값입니다.

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

이 코드의 기대 결과는 `20000`이지만, 실제로는 20000보다 작은 값이 나올 수 있습니다.

이유는 두 스레드가 `non-serial schedule` 방식으로 겹쳐 실행되기 때문입니다. 위 코드는 공유 자원에 동기화 없이 접근하고 있으며, 하나의 스레드가 자원을 변경하므로 동시성 이슈가 발생할 수 있는 조건을 모두 충족합니다.

## 해결방법

### synchronized

```java
    public synchronized void increment(){
        count++;
    }
```

간단히 `increment()` 메서드에 `synchronized` 키워드를 추가하여 해결할 수 있습니다. 
이 방식은 락을 획득하지 못한 다른 스레드를 대기 상태로 만들고, 실행 중인 스레드가 락을 반납할 때 대기 중인 스레드가 실행되도록 합니다.

### Atomic 연산

Atomic 연산은 CAS(Compare-And-Swap) 연산을 사용합니다. 
CAS 연산을 통해 스레드는 직렬적(동기적)으로 처리되며, 하나의 스레드가 작업을 끝내면 다른 스레드가 실행되는 방식으로 동시성 문제를 해결합니다.

### Reentrant Lock

`ReentrantLock도` 비슷한 방식이지만, `synchronized와` 달리 대기 중인 스레드가 `blocked` 상태가 아닌 `waiting` 상태가 되어 다른 스레드가 깨어나게 할 수 있습니다. 
이로 인해 무한 대기 문제가 발생하지 않으며, 특정 조건에 따라 스레드를 깰 수 있어 생산자-소비자 문제도 해결할 수 있다는 장점이 있습니다.

## 자바에서의 동기화 기법 - 디스크 

디스크에 저장된 파일에서 동시성 이슈를 해결하려면 OS 레벨에서 제어가 필요합니다.

OS에서 프로세스 레벨 락을 획득하면 락을 얻은 프로세스만 파일에 접근할 수 있으므로, 프로세스 내 스레드 간에도 임계 영역 설정을 통해 자원을 보호해야 합니다.

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

