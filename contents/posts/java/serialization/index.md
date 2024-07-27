---
title: "Serialization"
date: 2024-07-27
update: 2024-07-27
tags:
  - Java
  - Interview
---
## 직렬화란

**직렬화는 객체의 상태를 바이트 스트림으로 변환**하는 것입니다. 
역직렬화는 그 반대입니다. 다르게 말하면 직렬화는 **Java 객체를 정적 바이트 스트림(시퀀스)으로 변환**한 다음 
데이터베이스 또는 파일에 저장하거나 네트워크를 통해 전송할 수 있는 것입니다.

## transient 키워드는 왜 필요한가

어떤 객체를 전송 또는 저장하려 할때, 직렬화된 출력에 민감한 데이터가 포함하면 보안 위험이 발생할 수 있습니다. 여기서 `transient` 키워드가 도움될 수 있습니다.

하지만 transient 키워드를 사용할 때는 해당 필드의 값이 기본값으로 설정되기 때문에 주의해야 합니다.

### 예제
```java
import java.io.*;

class User implements Serializable {
  String username;
  transient String password;
  public User(String username, String password) {
    this.username = username;
    this.password = password;
  }
}

public class Test {
  public static void main(String [] args) {
    User u = new User("testUser", "secret123");
    try {
      FileOutputStream fileOut = new FileOutputStream("./test.txt");
      ObjectOutputStream out = new ObjectOutputStream(fileOut);
      out.writeObject(u);
      out.close();
      fileOut.close();
    } catch (IOException i) {
      i.printStackTrace();
    }
  }
}

# Output:
# 'user.txt' 파일에는 'User' 개체에 대한 직렬화된 데이터가 포함됩니다. 
# 단, '비밀번호'는 임시로 표시되어 있으므로 포함되지 않습니다.
```

## transient 외에 다른 방법

**첫번째 방법으로는** `Externalizable 인터페이스` 는 `serialization`의 읽기 쓰기 메소드를 정의할 수 있도록 하여 직렬화하는 데이터와 수행방법을 제어할 수 있습니다.

```java
  public void writeExternal(ObjectOutput out) throws IOException {
    out.writeObject(name);
    out.writeInt(age);
  }

  public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
    name = (String) in.readObject();
    age = in.readInt();
  }
```

**두번째 방법으로는** `Custom Serialization`이 있습니다. 클래스에 사용자 정의 `writeObject` 및 `readObject` 메서드를 정의하여 직렬화 프로세스를 제어할 수 있습니다. 
이는 직렬화 중에 커스텀 처리 또는 유효성 검사를 수행해야 하는 경우 유용할 수 있습니다.

```java
  private void writeObject(ObjectOutputStream oos) throws IOException {
    oos.defaultWriteObject();
    oos.writeInt(age);
  }

  private void readObject(ObjectInputStream ois) throws IOException, ClassNotFoundException {
    ois.defaultReadObject();
    age = ois.readInt();
  }
```

## SerialVersionUID를 선언해야 하는 이유

직렬화 가능한 클래스에 `serialVersionUID`라는 버전 번호를 연결합니다. 
이 버전 번호는 역직렬화 할때 직렬화된 객체의 발신자와 수신자가 호환되는 지 즉, 발신자의 객체에 대한 클래스를 로드했는지 확인하는 데 사용됩니다.

만약 역직렬화하는 곳에서 다른 `serialVersionUID`를 가진 객체에 대한 클래스를 로드했을경우 `InvalidClassException`이 발생합니다. 

```java
ANY-ACCESS-MODIFIER static final long serialVersionUID = 42L;
```
만약 명시적으로 위와 같이 선언하지 않으면, 먼저 IDE에서 클래스이름, 속성, 접근제어자 기반으로 자체적으로 생성하거나 JVM이 런타임에 자동적으로 생성합니다. 
이때 각 플랫폼의 컴파일러마다 `serialVersionUID`는 다릅니다. 따라서 다른 플랫폼으로 해당 객체를 직렬화하여 전송하면 `InvalidClassException`이 발생합니다.

결론적으로 선언을 해야하는 이유는 **직렬화된 데이터의 버전관리를 위함**입니다. 

만약 시간이 흘러 바이트스트림을 객체로 역직렬화 할때 해당 클래스가 변경사항이 생기면 함부로 역직렬화하다가 문제가 생길 수 있습니다. 그래서 이를 방지하기 위해 
차라리 버전관리를 해서 예외가 발생하도록 하는 것 같습니다. 어려운 말로 데이터의 무결성과 애플리케이션의 안정성을 위함입니다.


## Serialization 주의점

직렬화 하려는 객체중 필드 하나가 객체 참조 변수인 경우 이 객체도 역시 직렬화가 가능하도록 해야합니다. 그렇지 않으면 `NotSerialzedException`이 발생합니다.

```java
public class Person implements Serializable {
    private int age;
    private String name;
    private Address country; // must be serializable too
}
```

## Reference

https://www.baeldung.com/java-serialization

https://ioflood.com/blog/what-is-transient-in-java/

https://stackoverflow.com/questions/285793/what-is-a-serialversionuid-and-why-should-i-use-it