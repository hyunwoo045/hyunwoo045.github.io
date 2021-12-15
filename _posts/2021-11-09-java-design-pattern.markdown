---
title: "Java - 디자인 패턴"
excerpt: "자바의 주요 디자인 패턴들을 정리합니다."

header:
  overlay_image: https://images.unsplash.com/photo-1501785888041-af3ef285b470?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1350&q=80
  overlay_filter: 0.5

tags:
  - Java

toc: true
toc_label: "table of content"
toc_icon: "bars"
toc_sticky: true
---

# 디자인 패턴

자주 사용하는 설계 패턴을 정형화해서 유형별로 가장 최적의 방법으로 개발을 할 수 있도록 정해둔 설계

알고리즘과 달리 명확하게 정답이 있는 형태는 아니고, 프로젝트의 상황에 맞추어 적용 가능하다.

## Gof 디자인 패턴

소프트웨어를 설계할 때에는 경험이 매우 중요하다. 그러나 모든 사람들이 다양한 경험을 가지고 있을 수는 없다.

지식을 공유하기 위해 GOF (Ganf of Four) 의 디자인 패턴이다. 객체지향 개념에 따른 설계 중 재사용할 경우 유용한 설계를 디자인 패턴으로 정리해 둔 것이다.

Gof 의 디자인 패턴은 총 23개이며, 이를 잘 이해하고 활용한다면, 경험이 부족하더라도 좋은 소프트웨어 설계가 가능하다.

<br/>

---

## 장점

1. 개발자 간의 원활한 소통
2. 소프트웨어 구조 파악 용이
3. 재사용을 통한 개발 시간 단축
4. 설계 변경 요청에 대한 유연한 대처

<br/>

---

## 단점

1. 객체지향 설계 / 구현
2. 초기 투자 비용 부담

<br/>

---

## 생성 패턴

객체를 생성하는 것과 관련된 패턴으로, 객체의 생성과 변경이 전체 시스템에 미치는 영향을 최소화하고, 코드의 유연성을 높여 준다.

- Factory Method
- Singleton
- Prototype
- Builder
- Abstract Factory
- Chaining

<br/>

---

## 구조 패턴

프로그램 내의 자료구조나 인터페이스 구조 등 프로그램 구조를 설계하는데 활용 될 수 있는 패턴 클래스, 객체들의 구성을 통해서 더 큰 구조를 만들 수 있게 해준다. 큰 규모의 시스템에서는 많은 클래스들이 서로 의존성을 가지게 되는데, 이런 복잡한 구조를 개발하기 쉽게 만들어주고, 유지 보수 하기 쉽게 만들어 준다.

- Adapter
- Composite
- Bridge
- Decorator
- Facade
- Flyweight
- Proxy

<br/>

---

## 행위 패턴

반복적으로 사용되는 객체들의 상호작용을 패턴화한 것으로, 클래스나 객체들이 상호작용하는 방법과 책임을 분산하는 방법을 제공한다. 행위 패턴은 행위 관련 패턴을 사용하여 독립적으로 일을 처리하고자 할 때 사용한다.

- Template Method
- Interpreter
- Iterator
- Observer
- Strategy
- Visitor
- Chain of responsibility
- Command
- Mediator
- State
- Memento

<br/>

---

## Singleton Pattern

어떤 객체가 유일하게 1개만 존재할 때 사용한다.

서로 자원을 공유할 때 사용. 실제 프로그래밍에서는 TCP Socket 통신에서 서버와 연결된 connect 객체에 주로 사용한다.

예제 코드

```java
/**
 * 싱글톤 패턴으로 만들어진 객체는 아래의 특성을 가져야 한다.
 * 1. 자기 자신의 인스턴스를 멤버 변수로 가진다.
 * 2. 생성자 함수는 반드시 private 으로 설정한다.
 * 3. getInstance() 와 같은 메서드로 자신이 가지고 있는 인스턴스를 반환해 주는 메서드를 생성한다.
 */
public class SocketClient {

  private static SocketClient socketClient = null;

  private SocketClient() {

  }

  public static SocketClient getInstance() {

    if (socketClient == null) {
      socketClient = new SocketClient();
    }
    return socketClient;
  }

  public void connect() {
    System.out.println("connect");
  }
}
```

여러 객체가 해당 객체를 인스턴스화 하더라도 모두 같은 객체를 바라보고 있게 된다.

테스트 해보자.

```java
class AClass {
  private SocketClient socketClient;
  // 해당 객체의 생성자 메서드로 socketClient 를 초기화한다.
  // 이 때 싱글톤 객체의 생성자는 private 으로 선언되어 있기 때문에, new 키워드로 생성이 불가하고
  // 구현한 getInstance() 메서드로 해당 객체의 인스턴스를 가져와야 한다.
  public AClass() {
    this.socketClient = SocketClient.getInstance();
  }
  public SocketClient getInstance() {
    return this.socketClient;
  }
}

class BClass {
  private SocketClient socketClient;
  public BClass {
    this.socketClient = SocketClient.getInstance();
  }
  public SocketClient getInstance() {
    return this.socketClient;
  }
}

public class Main {
    public static void main(String[] args) {
	      AClass aClass = new AClass();
        BClass bClass = new BClass();

        SocketClient aClient = aClass.getSocketClient();
        SocketClient bClient = bClass.getSocketClient();

        System.out.println(aClient.equals(bClient));  // -> true
    }
}
```

<br/>

---

## Adapter Pattern

호환성이 없는 기존 클래스의 인터페이스를 변환하여 재사용할 수 있도록 한다. SOLID 중 개방폐쇄원칙(OCP)를 따른다.

예제

```java
interface Electronic110V {
  public void powerOn();
}

interface Electronic220V {
  public void powerOn();
}
```

110V, 220V 컨센트가 있다고 하자.

```java
class HairDryer implements Electronic110V {
  @Override
  public void powerOn() {
    System.out.println("Hair Dryer ON! 110V");
  }
}

class TV implements Electronic220V {
  @Override
  public void powerOn() {
    System.out.println("TV ON! 220V");
  }
}
```

두 가전이 있다. 하나는 헤어드라이어고 110V 콘센트에 꽂아야 한다. 나머지 하나는 TV이며 이 아이는 220V 콘센트에 꽂아야 한다.

```java
public class Home {
  public static void main () {
    HairDryer hairDryer = new HairDryer();
    TV tv = new TV();

    connect(hairDryer);
    connect(tv);  // -> 여기서 에러. Electronic110V 인스턴스를 넣어야 하는데 Electronic220V 를 넣었음.
  }

  public static void connect(Electronic110V electronic110V) {
    electornic110V.powerOn();
  }
}
```

나의 집이다. 곧 무너질 것 같은 옛날 집이라 그런지 110V 콘센트를 사용한다. 우리 집에는 헤어드라이기랑 TV가 있다. 둘 다 연결을 해보려 했는데, 헤어드라이기는 잘 연결됬고 TV는 안된다. TV는 220V 콘센트가 필요하기 때문이다.

돼지코를 사야겠다.

```java
// 기본적으로 110V 콘센트에 꽂을거임.
class Adapter implements Electronic110V {

  // 이 어댑터에는 220V 가전을 꽂을 수 있음
  private Electronic220V electronic220V;

  public Adapter(Electronic220V electronic220V) {
    this.electronic220V = electronic220V;
  }

  // 그래서 콘센트에는 110V를 꽂은거지만 사실은 220V를 꽂은거임 :D
  @Override
  public void connect() {
    electronic220V.connect();
  }
}
```

돼지코도 샀으니 집에 다시 TV를 꽂아보자.

```java
public class Home {
  public static void main () {
    HairDryer hairDryer = new HairDryer();
    TV tv = new TV();

    connect(hairDryer);

    // 사온 어댑터에 TV를 꽂아주고 어댑터를 콘센트에 연결
    Adapter adapter = new Adapter(tv);
    adapter.connect();
  }

  public static void connect(Electronic110V electronic110V) {
    electornic110V.powerOn();
  }
}
```

이와 같이 서로 비슷한 동작을 가지고 있지만 다른 인터페이스여서 호환이 안될 때 사용할 수 있는 패턴이다.

<br/>

---

## Proxy Pattern

Proxy 는 대리인이라는 뜻이다. 뭔가를 대신해서 동작하는 것이다. SOLID 중에서 개방폐쇄 원칙(OCP)과 의존역전원칙(DIP)을 따른다.

예제

```java
class Html {
  private String url;
  public Html(String url) {
    this.url = url;
  }
}
```

HTML 파일이 하나 있다.

```java
interface IBrowser {
  show();
}
```

HTML 파일은 브라우저가 랜더링해서 보여준다.

```java
class Browser implements IBrowser {

  private String url;

  public Browser(String url) {
    this.url = url;
  }

  @Override
  public Html show() {
    System.out.println("Browser loading html from: "+url);
    return new Html(url);
  }
}
```

IBrowser 인터페이스를 정의해보자. 멤버 변수로 url 이 있고, 보여달라고 할 때마다 보여준다.

```java
public class Test {
  public static void main(String[] args) {
    Browser browser = new Browser("www.naver.com");
    browser.show();
    browser.show();
    browser.show();
    browser.show();
    browser.show();
  }
}
```

```
Browser loading html from: www.naver.com
Browser loading html from: www.naver.com
Browser loading html from: www.naver.com
Browser loading html from: www.naver.com
Browser loading html from: www.naver.com
```

다섯번 로딩이 잘 되었다.

하지만 매번 `show()` 할 때 마다 로딩을 해야 하는 것은 좀 비효율적인 것 같다. 캐싱해서 한 번만 로딩하고 가져다 쓰면 좋겠다. 당연히 실제 캐싱을 하는 건 아니고 흉내만 내보자.

브라우저가 하는 일은 대신 하는 객체인 BrowserProxy 를 만든다.

```java
public class BrowserProxy implements IBrowser {

  private String url;
  private Html html;

  public BrowserProxy(String url) {
    this.url = url;
  }

  @Override
  public Html show() {

    // html 이 저장된 적이 없다면 html 을 받아오고 저장해둔다.
    if (html == null) {
      this.html = new Html(url);
      System.out.println("BrowserProxy loading html from: "+url);
    }
    // html 이 저장되어 있으면 더 이상 로딩하지 않고 가져온다.
    System.out.println("BrowserProxy use cache html: "+ url);
    return html;
  }
}
```

<br/>

---

## Decorator Pattern

기존 뼈대(클래스)는 유지하되, 이후 필요한 형태로 꾸밀 때 사용한다. 확장이 필요한 경우 상속의 대안으로도 활용한다. SOLID 중에 개방폐쇄원칙(OCP)와 의존 역전 원칙(DIP)를 따른다.

커피를 생각하면 이해가 쉽다. 기본적으로 커피는 에스프레소에 물을 부으면 아메리카노가 되고, 우유를 부으면 라떼가 되는 등, 에스프레소에 데코레이터를 추가하면 다른 무언가가 되는 구조라고 생각하면 된다.

예제

```java
interface ICar {

  int getPrice();
  void showPrice();
}


class Audi implements ICar{
  private int price;
  public Audi(int cost) {
    this.price = cost;
  }

  @Override
  public int getPrice() {
    return price;
  }

  @Override
  public void showPrice() {
    System.out.println("Audi 의 가격은 "+this.price);
  }
}
```

자동차라는 인터페이스가 있고, 우리는 그 것으로 아우디를 만들었다. 아우디는 각 모델에 따라서 가격이 천차만별로 달라진다.

```java
class AudiDecorator implements ICar {
  protected ICar audi;
  protected String modelName;
  protected int modelPrice;

  public AudiDecorator(ICar audi, String modelName, int modelPrice) {
    this.audi = audi;
    this.modelName = modelName;
    this.modelPrice = modelPrice;
  }

  /* 이 부분에서 decorating 이 들어간다. 각 모델에 맞는 가격이 추가 된다. */
  @Override
  public int getPrice() {
    return audi.getPrice() + modelPrice;
  }

  @Override
  public void showPrice() {
    System.out.println(modelName + " 의 가격은 " + getPrice() + "원 입니다.");
  }
}
```

모델을 구현해보자.

```java
class A3 extends AudiDecorator {
  public A3(ICar audi, String modelName) {
    super(audi, modelName, 1000);
  }
}

class A4 extends AudiDecorator {
  public A4(ICar audi, String modelName) {
    super(audi, modelName, 2000);
  }
}

class A5 extends AudiDecorator {
  public A5(ICar audi, String modelName) {
    super(audi, modelName, 3000);
  }
}
```

네. A3 모델은 기존 아우디에 1000원이 추가되고, A4 는 2000원이 A5 는 3000원이 추가되는 구현.

```java
import com.company.design.decorator.*;

public class Main {
  public static void main(String[] args) {
    ICar audi = new Audi(1000);  // 기존 아우디는 1000원이고
    ICar audi3 = new A3(audi, "A3");  // A3 모델로 다운캐스팅하여 생성.
    audi3.showPrice();

    ICar audi4 = new A4(audi, "A4");  // A4 모델로 다운캐스팅하여 생성.
    audi4.showPrice();

    ICar audi5 = new A5(audi, "A5");  // A5 모델로 다운캐스팅하여 생성.
    audi5.showPrice();
  }
}
```

```
Audi 의 가격은 1000
A3 의 가격은 2000원 입니다.
A4 의 가격은 3000원 입니다.
A5 의 가격은 4000원 입니다.
```

결과가 잘 출력되었다.

<br/>

---

## Observer Pattern

Event Listener 가 가장 대표적인 예시. Observer 라고 하는 관찰자가 변화가 생기면 관련된 객체들에게 알려준다.

<br/>

---

## Facade Pattern

<br/>

---

## Strategy Pattern

유사한 행위들을 캡슐화하여, 객체의 행위를 바꾸고 싶은 경우 직접 변경하는 것이 아닌 전략만 변경하여, 유연하게 확장하는 패턴.

SOLID 중에서 개방폐쇄원칙(OCP)과 의존 역전 원칙(DIP)를 따른다.
