---
title: "(1) 단방향 해시"
date: 2024-08-06
update: 2024-08-06
tags:
  - 암호화 
series: "암호화 방식"
---

## 해싱

### 해시 함수

해시함수는 **가변길이 입력데이터를 사용하여 고정길이 출력 값을 생성**합니다. 이 출력 값을 다이제스트(digest), hashcode, hash 라고 부릅니다.
특징은 다음과 같습니다.

-  **해싱은 단방향 프로세스** 입니다. 따라서 digest 값을 원본으로 바꿀 수 없습니다. 예를들어, 사용자가 입력한 비밀번호를 해싱하고 이를 다시 원본 비밀번호로 얻을 수 없습니다. (복호화 불가능)
- 동일한 입력을 해시함수에 전달하면 항상 동일한 해시코드가 생성됩니다.
- 해시는 가능한 중복되지않고 고르게 분산됩니다. 즉, 입력값이 달라도 같은 해시코드를 출력할 가능성이 있습니다.
- 충돌 위험을 최소화 할 수 있도록 해시함수 로직은 복잡해야 합니다.

### 암호화 해시 함수

해시함수인데, 향상된 보안 수준을 제공합니다. 따라서 비밀번호 확인, 데이터 무결성 확인, 블록체인과 같은 암호화 목적으로 사용합니다.

대표적인 특징으로, 암호화 해시함수는 **충돌에 완전히 저항**할 수 있습니다. 대조적으로 Object 클래스의 `hashcode()`는 int형 정수를 반환합니다. 즉, 범위가 -2147483648에서 2147483647로 제한됩니다. 이는 충돌가능성을 야기합니다.  

## 해싱 알고리즘

`MD5`은 128비트 길이의 다이제스트를 생성합니다. 길이가 짧은만큼 굉장히 충돌이 빠른데, 레인보우테이블등으로 공격에 매우 취약합니다. 현재는 암호화목적으로 절대 사용하면 안됩니다.
[취약사례](https://namu.wiki/w/%EB%BD%90%EB%BF%8C%20%EA%B0%9C%EC%9D%B8%EC%A0%95%EB%B3%B4%20%ED%95%B4%ED%82%B9%20%EC%82%AC%EA%B1%B4)

`SHA-256`은 SHA 시리즈 중 하나로, 가장 널리 쓰이는 암호화 해시 함수입니다. 고정 크기의 256비트 해시를 생성하며, 단방향 함수입니다.
자바는 `SHA-256` 해싱을 위한 내장 `MessageDigest` 클래스를 제공합니다. 

MessageDigest는 스레드로부터 안전하지 않다는 점이 있습니다. 그외에도 외부라이브러리인
`Guava`, `apache commons codecs`, `bouncy castle`등으로 구현할 수 있습니다.

```java
String originalString = "테스트";
MessageDigest digest = MessageDigest.getInstance("SHA-256");
byte[] encodedHash = digest.digest(originalString.getBytes(StandardCharsets.UTF_16));

System.out.println("bytesToHex(encodedHash) = " + bytesToHex(encodedHash));
```

**bytesToHex 함수**

```java
private static String bytesToHex(byte[] hash) {
    StringBuilder hexString = new StringBuilder(2 * hash.length);
    for (int i = 0; i < hash.length; i++) {
        String hex = Integer.toHexString(0xff & hash[i]);
        if (hex.length() == 1) {
            hexString.append('0');
        }
        hexString.append(hex);
    }
    return hexString.toString();
}
```

## 암호화 공격

`SHA-256`과 같이 단방향 해시함수로 패스워드를 암호화하면 문제점이 있습니다.

### 레인보우 테이블 

동일한 메시지가 동일한 다이제스트를 갖는다는 특징 때문에, **공격자가 미리 만든 다이제스트를 가능한 한 많이 확보**하고,
이를 탈취한 다이제스트와 비교해 원본 메시지를 찾아내거나, 동일한 다이제스트를 만드는 다른 메시지(해시충돌로 인해)를 찾아낼 수 있습니다.

### 속도

해시함수는 빠른 처리 속도를 갖고 있습니다. 즉, 무차별 대입 공격에 취약합니다. 공격자는 매우 빠른 속도로 임의의 문자열의 다이제스트와
해킹할 대상의 다이제스트를 비교할 수 있습니다.

## 단방향 해시 보완하기

### 솔팅 (salting)

원본 메시지에 임의의 문자열(솔팅)을 추가하여 다이제스트를 생성합니다. 예를들어, 사용자마다 다른 솔팅을 사용하고 다이제스트화하면 공격자는 다이제스트를 탈취해도 위에 같은 공격방식으로
원본 메시지를 구하기 어려울 것 입니다.

### 키 스트레칭 (key stretching)

입력한 패스워드의 다이제스트를 생성하고, 생성된 다이제스트를 입력값을 다시 다이제스트를 생성하는 과정을 n번 반복합니다.

### Adaptive Key Derivation Function 

위 2가지 방식을 조합하여 다이제스트를 구현한 함수이며, 대표적으로 `PBKDF2`, `bcrypt`, `scrypt` 있습니다.

## 정리

- `bcrypt`는 강력한 패스워드 다이제스트를 생성하는 시스템을 구축했으며 빠른 적용을 위해 사용합니다.
- `scrypt`는 매우 민감한 정보를 다루고, 보안 시스템을 구현하는 데 많은 비용을 투자할 때 사용합니다.
- `SHA-256, SHA-512` 등의 해시 함수는 **패스워드 인증이 아닌 메시지 인증(MAC)과 무결성 체크를 위해 사용**합니다.
- `PBKDF2-HMAC-SHA-256/SHA-512` 는 ISO-27001의 보안 규정을 준수하고, 서드파티의 라이브러리에 의존하지 않으면서 사용자 패스워드의 다이제스트를 생성할 때 사용합니다.



### 

## Reference

https://www.baeldung.com/cs/hashing

https://d2.naver.com/helloworld/318732

https://www.baeldung.com/sha-256-hashing-java