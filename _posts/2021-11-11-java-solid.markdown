---
title: "Java 객체지향 설계 5원칙"
excerpt: "자바의 객체지향 설계 5원칙. SOLID"

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

# 객체지향 설계 5원칙

좋은 소프트웨어 설계를 위해서는 어떠한 원칙들을 지켜야 할까? 함께 알아보자

<br/>

---

## 응집도와 결합도

좋은 소프트웨어의 설계는 '낮은 결합도', '높은 응집도' 이다.

결합도 - 객체 간의 상호 의존 정도를 나타내는 지표 <br/>
응집도 - 하나의 객체 내부에 존재하는 구성 요소들의 기능적 관련성. 응집도가 높은 모듈은 하나의 책임에 집중하고 독립성이 높아져, 재사용 및 유지보수가 용이하다.

<br/>

---

## SRP (Single Responsibility Principle) 단일 책임 원칙

어떠한 클래스를 변경해야 하는 이유는 한 가지 뿐이여야 한다.

```java
class Mammal {
  private String type;
  public int speed;

  public Mammal(String type) {
    this.type = type;
    this.speed = 0;
  }

  public void move() {
    if (this.type == "human") {
      speed += 10;
    } else if (this.type == "turtle") {
      speed += 1;
    } else if (this.type == "tiger") {
      speed += 50;
    }
  }
}
```

위의 코드는 동물을 나타내는 클래스. `move()` 메서드에서 각 종족의 이동 속도를 나타내고 있다. 만약의 인간의 이동속도를 고치고 싶으면 이 객체를, 또한 호랑이의 이동 속도를 고치고 싶다 해도 이 객체로 들어와서 수정하게 된다. 만약 종족이 더 늘어난다면 Mammal 객체를 수정해야 하는 이유도 늘어나게 되고 관리가 어렵고 실수를 저지르기 쉬워진다. 아래와 같이 수정하자.

```java
class Human extends Mammal {
  public void move() {
    this.speed += 10;
  }
}

class Turtle extends Mammal {
  public void move() {
    this.speed += 1;
  }
}

class Tiger extends Mammal {
  public void move() {
    this.speed += 50;
  }
}
```

<br/>

---

## OCP (Open Closed Principle) 개방 폐쇄 원칙

자신의 확장에는 열려 있고, 주변의 변화에 대해서는 닫혀 있어야 한다.

상위 클래스 또는 인터페이스를 중간에 둠으로써, 자신은 변화에 대해서는 폐쇄적인지만, 인터페이스는 외부의 변화에 대해서 확장을 개방해 줄 수 있다.

<br/>

---

## LSP (Liskov Substitution Principle) 리스코프 치환 원칙

서브 타입은 언제나 자신의 기반 타입으로 교체 할 수 있어야 한다.

원숭이의 특징을 가져다가 고릴라를 만들었다. 또한 원숭이의 특징을 가져다가 오랑우탄을 만들었다. 그렇다고 해서 고릴라가 원숭이를 대신할 수도, 오랑우탄이 원숭이를 대신할 수는 없다. LSP 원칙에 위배 된다.

'자동차'가 있다. 자동차의 특징을 가져다가 제네시스를, 자동차의 특징을 가져다가 그렌저를 만들었다. 제네시스는 자동차라고 할 수도, 그렌저도 자동차라고 할 수 있다. 이 것은 리스코프 치환 원칙을 잘 지킨 것이다.

<br/>

## ISP (Interface Segregation Principle) 인터페이스 분리 원칙

클라이언트는 자신이 사용하지 않는 메서드에 의존 관계를 맺으면 안된다.

단일책임원칙(SRP) 와 약간 반대되는 개념이다. 프로젝트의 요구 사항과 설계에 따라서 SRP / ISP 둘 중 하나를 선택해야 한다.

<br/>

---

## DIP (Dependency Inversion Principle) 의존 역전 원칙

자신보다 변하기 쉬운 것에 의존하지 말아야 한다.
