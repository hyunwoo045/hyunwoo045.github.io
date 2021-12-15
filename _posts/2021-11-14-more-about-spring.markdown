---
title: "Spring, 조금만 더 들여다보기"
excerpt: "스프링에 대해 조금 더 깊게 들여다보자."

header:
  overlay_image: https://images.unsplash.com/photo-1501785888041-af3ef285b470?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1350&q=80
  overlay_filter: 0.5

tags:
  - Java
  - Spring

toc: true
toc_label: "table of content"
toc_icon: "bars"
toc_sticky: true
---

## More about Spring

- Spring 은 2004년 3월에 출시하였고 지금까지 단 한 번도 자바 엔터프라이즈 어플리케이션 개발의 최고의 자리를 차지하고 있다.
- 스프링 프레임워크의 구성은 20여가지로 구성되고, 필요한 모듈만 선택하여 사용할 수 있다.
- 현재 많은 서비스들이 단일 아키텍쳐(모놀리스)에서 마이크로서비스 아키텍쳐로 변환 중인데, 여기에 맞춰서 스프링도 진화하고 있다.
- 여러 모듈들이 있지만 단연 Spring Boot, Spring Cloud, Spring Data, Spring Batch, Spring Security 에 중점을 둔다.

스프링은 "테스트의 용이성", "느슨한 결합"에 중점을 두고 개발되었다. 스프링 개발 이전에는 테스트가 매우 어려웠다. 느슨한 결합이 된 애플리케이션 개발이 힘든 상태였고, 특히나 DB와 같이 외부에 의존성을 두는 경우 단위테스트가 불가능했다.

스프링과 다른 프레임워크의 가장 큰 차이점은 IoC 를 통한 개발을 진행한다는 점이고, AOP를 사용하여 로깅, 트랙잭션 관리, 시큐리티 적용에 있어 AspectJ와 같이 완벽하게 구현된 AOP와 통합하여 사용이 가능하다는 점에 있다.

Spring 삼각형에 대해 자세히 알아보자. (IoC/DI, PSA, AOP, POJO)

<br/>

---

## IoC (Inversion of Control)

제어의 역전. 스프링에서는 일반적인 Java 객체를 new 로 생성하여 개발자가 관리하는 것이 아닌 Spring Container 에 모두 맡긴다. 즉, 개발자에서 프레임워크로 제어의 객체 관리의 권한이 넘어갔음으로 '제어의 역전' 이라고 한다. 다른 프레임워크에선 이런 형태를 찾아 볼 수 없음.

<br/>

## DI (Dependency Injection)

Spring 이 객체 생명 주기를 알아서 관리해주는데, 객체를 사용하기 위해 주입을 받는다고 하는데 이를 DI 라고 한다. 외부로부터 내가 사용할 객체를 주입받는다. 주입을 해주는 애는 Spring Container 임.

이에 장점은

- 의존성으로부터 격리시켜 코드 테스트할 때 용이하다.
- DI를 통하여, 불가능한 상황을 Mock와 같은 기술을 통하여, 안정적으로 테스트 가능하다.
- 코드를 확장하거나 변경할 때 영향을 최소화 한다. (추상화)
- 순환참조를 막을 수 있다.

<br/>

예시를 살펴보자. 문자열을 인코딩하는 객체를 만들어달라는 의뢰를 받았다. Base64 기반으로 인코딩 하는 객체이다. 아~ 간단하지 하면서 만들었다.

```java
class Base64Encoder {
  public String encode(String message) {
    return Base64.getEncoder().encodeToString(message.getBytes());
  }
}
```

그런데 어느 날 이번엔 Url Encoder 도 추가로 만들어 달라고 하는 요청이 들어왔다. 그래서 그 것도 막 만들었다.

```java
class UrlEncoder {
  public String encode(String message) {
    try {
      return URLEncoder.encode(message,"UTF-8");
    } catch (UnsupportedEncodingException e) {
      e.printStackTrace();
      return null;
    }
  }
}
```

```java
public class Main {
  public static void main(String[] args) {
    String url = "www.naver.com/books/it?page=10&size=20&name=spring-boot";

    Base64Encoder base64Encoder = new Base64Encoder();
    String base64Result = base64Encoder.encode(url);
    System.out.println(base64Result);

    UrlEncoder urlEncoder = new UrlEncoder();
    String urlResult = urlEncoder.encode(url);
    System.out.println(urlResult);
  }
}
```

테스트도 해보니까 잘 되더라. 근데 가만보니 두 인코더가 `encode()` 라고 비슷한 일을 하고 있다. 그래서 Interface 를 만들어서 추상화를 시켜주었다.

```java
interface IEncoder {
  public String encode(String message);
}

class Base64Encoder implements IEncoder {
  @Override
  public String encode(String message) {
    return Base64.getEncoder().encodeToString(message.getBytes());
  }
}

class UrlEncoder implements IEncoder {
  @Override
  public String encode(String message) {
    try {
      return URLEncoder.encode(message,"UTF-8");
    } catch (UnsupportedEncodingException e) {
      e.printStackTrace();
      return null;
    }
  }
}

public class Main {
  public static void main(String[] args) {
    String url = "www.naver.com/books/it?page=10&size=20&name=spring-boot";

    IEncoder base64Encoder = new Base64Encoder();
    String base64Result = base64Encoder.encode(url);
    System.out.println(base64Result);

    IEncoder urlEncoder = new UrlEncoder();
    String urlResult = urlEncoder.encode(url);
    System.out.println(urlResult);
  }
}
```

인터페이스도 잘 만들었고 테스트도 잘 되는데 아직 추상화도 안되어 있고, 기존 코드와 거의 동일하다. DI 를 넣어보자.

```java
class Encoder {
  private IEncoder iEncoder;

  public Encoder() {
    this.iEncoder = new Base64Encoder();
  }

  public String encode(String message) {
    iEncoder.encode(message);
  }
}
```

Encoder 라고 하는 객체가 IEncoder 인터페이스를 멤버 변수로 가지고 있으면서 생성될 때 Base64Encoder 로 지정해준다. Main 함수는 따라서 아래와 같이 변할 수 있다.

```java
public class Main {
  public static void main(String[] args) {
    String url = "www.naver.com/books/it?page=10&size=20&name=spring-boot";

    Encoder encoder = new Encoder();
    String result = encoder.encode(url);
    System.out.println(result);  // -> Base64 로 encoding 된 결과.
  }
}
```

Base64 로 인코딩이 잘 되는 것을 테스트를 통해 확인하였다. 그런데 또 어느 날 URL Encoding 이 잘 되는지를 테스트 해달라는 요청이 들어왔다. 바로 Encoder 객체로 들어가서 아래와 같이 수정하였다.

```java
class Encoder {
  private IEncoder iEncoder;

  public Encoder() {
    // this.iEncoder = new Base64Encoder();
    this.iEncoder = new UrlEncoder();
  }

  public String encode(String message) {
    iEncoder.encode(message);
  }
}
```

Main 함수는 변하지 않는 상태에서 테스트를 해보았을 때 URL Encoding 이 잘 되는 것을 확인하였다. 하지만 매번 테스트 내용이 바뀔 때 마다 Encoder 라는 객체의 코드를 수정하는 것은 비효율적이고 본질을 건드린다는 위험을 안는다.

DI 개념을 도입하면 아래와 같이 수정할 수 있다.

```java
class Encoder {
  private IEncoder iEncoder;

  // 생성자를 통해 외부로부터 객체를 주입받는다.
  public Encoder(IEncoder iEncoder) {
    this.iEncoder = iEncoder;
  }

  public String encode(String message) {
    iEncoder.encode(message);
  }
}

public class Main {
  public static void main(String[] args) {
    String url = "www.naver.com/books/it?page=10&size=20&name=spring-boot";

    Encoder encoder = new Encoder(new Base64Encoder()); // 외부에서 사용할 객체를 주입.

    // Url Encoder 를 사용하고 싶으면 단순히 이렇게 쓰기만 하면 됨.
    // Encoder encoder = new Encoder(new UrlEncoder());

    String result = encoder.encode(url);
    System.out.println(result);
  }
}
```

이제 더 이상 개발자가 Encoder 의 코드를 관리할 필요가 없다. 단순히 Encoder 객체에 필요한 인코더(Base64 혹은 URL)를 주입함으로써 필요한 메서드를 가져다가 사용할 수 있게 된다.

이러한 방식으로 외부에서 사용할 객체를 주입을 하는 것을 보고 DI(Dependency Injection)이라고 한다. Encoder 객체는 IEncoder 멤버 변수를 가지고 있으며 `encode()` 메서드는 이에 의존적인 상태이다. 외부 객체에 의존적인 부분을 생성자 함수에서 매개 변수로 주입받음으로써 `encode()` 메서드가 완성이 될 수 있다.

<br/>

---

## AOP (Aspect Oriented Programming)

관점 지향 프로그래밍. 스프링 어플리케이션은 대부분 특별한 경우를 제외하고는 MVC. 웹 어플리케이션에서는 Web Layer, Business Layer, Data Layer 로 정의한다.

- Web Layer : REST API 를 제공하며, Client 중심의 로직 적용
- Business Layer : 내부 정책에 따른 Logic 을 개발하며 주로 해당 부분을 개발
- Data Layer : 데이터 베이스 및 외부와의 연동을 처리

<br/>

주요 Annotation

| Annotation      | 의미                                                                              |
| --------------- | --------------------------------------------------------------------------------- |
| @Aspect         | 자바에서 널리 사용하는 AOP 프레임워크에 포함되며 <br/> AOP를 정의하는 객체에 할당 |
| @Pointcut       | 기능을 어디에 적용시킬지 메소드나 Annotation 등 <br/> AOP를 적용시킬 지점을 설정  |
| @Before         | 메소드 실행하기 이전                                                              |
| @After          | 메소드가 성공적으로 실행된 후, 예외가 발생하더라도 실행                           |
| @AfterReturning | 메소드 호출 성공 실행 시 (Not Throws)                                             |
| @AfterThrowing  | 메소드 호출 실패 예외 발생 (Throws)                                               |
| @Around         | Before / after 모두 제어                                                          |
