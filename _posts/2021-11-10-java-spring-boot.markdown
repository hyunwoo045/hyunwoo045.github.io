---
title: "Spring Boot - Methods"
excerpt: "Java 의 Spring boot 기초 노트"

header:
  overlay_image: https://images.unsplash.com/photo-1501785888041-af3ef285b470?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1350&q=80
  overlay_filter: 0.5

tags:
  - Java
  - Spring Boot

toc: true
toc_label: "table of content"
toc_icon: "bars"
toc_sticky: true
---

## Spring Boot

특징

- 단순히 실행되며, 프로덕션 제품 수준의 스프링 기반 어플리케이션을 쉽게 만들 수 있다.
- Spring 구성이 거의 필요하지 않다.
- java, -jar로 실행하는 java 어플리케이션을 만들 수 있다.

주요 목표

- Spring 개발에 대해 빠르고, 광범위하게 적용할 수 있는 환경
- 기본값 설정이 있지만 설정을 바꿀 수 있다.
- 대규모 프로젝트에 공통적인 비 기능 제공 (보안, 모니터링 등)
- XML 구성 요구사항이 전혀 없음 (@Annotation 으로 아주 간단히 대체할 수 있음)

Build Tool

1. Maven
2. Gradle

Servlet Containers

1. Tomcat 9.x
2. Jetty 9.4
3. Undertow 2.0
4. Netty

프로젝트를 다운받는 법

[https://start.spring.io](https://start.spring.io)

핵심 요약

1. 어플리케이션 개발에 필수 요소들만 모아두었다.
2. 간단한 설정으로 개발 및 커스텀이 가능하다.
3. 간단하고, 빠르게 어플리케이션 실행 및 배포가 가능하다.
4. 대규모프로젝트(운영환경)에 필요한 비 기능적 기능도 제공한다.
5. 오랜 경험에서 나오는 안정적인 운영이 가능하다.
6. Spring에서 불편한 설정이 없어졌다. (XML 설정 등)

<br/>

---

## API 만들기

IntelliJ 를 사용한다면 Spring Boot 프로젝트를 생성하여 `localhost:8080` 으로 동작할 수 있는 서버를 자동으로 만들 수 있게 된다.

메인 클래스의 모습은 아래와 같고,

```java
package com.example.hello;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class HelloApplication {

  public static void main(String[] args) {
    SpringApplication.run(HelloApplication.class, args);
  }
}
```

directory tree 는 아래와 같다

```
.
├── main
│   ├── java
│   │   └── com
│   │       └── example
│   │           └── hello
│   │               └── HelloApplication.java
│   └── resources
│       ├── application.properties
│       ├── static
│       └── templates
└── test
    └── java
        └── com
            └── example
                └── hello
                    └── HelloApplicationTests.java
```

<br/>

## GET Method

Spring boot 에서 GET Method 를 받아 처리를 해주는 Controller 를 생성하는 법을 알아보자.

com.hello.example 아래에 controller 패키지를 생성한다. 그리고 GetApiController 클래스를 생성한다. GET Method 를 받아 처리하는 방법은 아래 주석으로 대체함.

```java
import com.example.hello.dto.UserRequest;
import org.springframework.web.bind.annotation.*;

import java.util.Map;

@RestController
@RequestMapping("/api/get")
public class GetApiController {

  /**
   * 1. 가장 기본적인 방법
   * http://localhost:9090/api/get/hello
   */
  @GetMapping("/hello")
  public String hello() {
    return "Hello!";
  }

  /**
   * http://localhost:9090/api/get/hello
   * 위와 똑같으나 Annotation 에 path 라는 변수를 명시해줌.
   */
  @GetMapping(path = "/hello")
  public String hello() {
    return "Hello!";
  }

  /**
   * 2. @RequestMapping Annotation 을 사용.
   * http://localhost:9090/api/get/hi
   * method 라는 매개변수를 아래와 같이 지정하지 않으면 GET, POST, PUT, DELETE 등 모든 메서드에 mapping 된다.
   */
  @RequestMapping(path = "/hi", method = RequestMethod.GET)
  public String hi() {
    return "Hi!";
  }

  /**
   * 3. path variable를 처리 하는 방법
   * 메서드의 매개변수에 @PathVariable 이라고 annotation 을 명시해주어야 한다.
   */
  // http://localhost:9090/api/get/path-variable/{name}
  @GetMapping("/path-variable/{name}")
  public String pathVariable(@PathVariable String name) {
    System.out.println("PathVariable: " + name);
    return name;
  }

  /**
   * 4. query parameter 를 처리 하는 방법
   * 4-1. key 값이 명확히 무엇인지 모를 때는 @RequestParam 이라고 annotatino 으로 명시한 매개변수를 Mapping 하는 방식으로 처리 가능.
   */
  // http://localhost:9090/api/get/query-param?user=steve&email=steve@gmail.com&age=30
  @GetMapping(path="query-param")
  public String queryParam(@RequestParam Map<String, String> queryParam) {
    StringBuilder sb = new StringBuilder();

    queryParam.entrySet().forEach(entry-> {
      System.out.println(entry.getKey());
      System.out.println(entry.getValue());
      System.out.println();

      sb.append(entry.getKey() + " = " + entry.getValue() + "\n");
    });

    return sb.toString();
  }

  /**
   * 4-2. key 값이 명확할 때는 아래와 같이 처리 가능함.
   */
  @GetMapping("query-param02")
  public String queryParam02(
      @RequestParam String name,
      @RequestParam String email,
      @RequestParam int age
  ) {
    System.out.println(name);
    System.out.println(email);
    System.out.println(age);

    return name+" "+email+" "+age;
  }

  /**
   * 4-3. key 가 너무 많아지면 DTO 객체를 생성하여 관리할 수 있다. 이 때는 @RequestParam annotation 을 사용하지 않아도 된다.
   */
  @GetMapping("query-param03")
  public String queryParam02(UserRequest userRequest) {
    System.out.println(userRequest.getName());
    System.out.println(userRequest.getEmail());
    System.out.println(userRequest.getAge());

    return userRequest.toString();
  }
}

/* 4-3. 의 DTO 예시 */
class UserRequest {
  private String name;
  private String email;
  private int age;
  public String getName() {
    return name;
  }
  public void setName(String name) {
    this.name = name;
  }
  public String getEmail() {
    return email;
  }
  public void setEmail(String email) {
    this.email = email;
  }
  public int getAge() {
    return age;
  }
  public void setAge(int age) {
    this.age = age;
  }
  @Override
  public String toString() {
    return "UserRequest{" +
        "name='" + name + '\'' +
        ", email='" + email + '\'' +
        ", age=" + age +
        '}';
  }
}
```

<br/>

---

## POST Method

Request body 라는 것을 클라이언트가 보내게 되는데, 형태로는 JSON, XML 등이 있지만 application/json 이 가장 많이 쓰임.

JSON 의 키는 camelCase 나 snake_case 를 많이 사용하고, snake_case 가 주로 사용이 많이 됨.

```json
{
  "phone_number": "010-1111-2222",
  "age": 10,
  "isAgree": false,
  "user": {
    // object value 예시
    "address": "경기도 용인시",
    "email": "abcde@gmail.com",
    "password": "1234"
  },
  "chat_history": [
    // object array 예시, 배열값은 object 도 가능.
    {
      "message": "안녕하세요!"
    },
    {
      "message": "반갑습니다 ^^"
    }
  ]
}
```

아래 예시 template 을 다루는 POST Method 코드를 살펴보자

```json
{
  "account": "",
  "email": "",
  "password": "",
  "address": ""
}
```

```java
@RestController
@RequestMapping("/api")
public class PostApiController {

  @PostMapping("/post")
  public void post(@RequestBody Map<String, Object> requestData) {
    requestData.entrySet().forEach(stringObjectEntry -> {
      System.out.println("key: " + stringObjectEntry.getKey());
      System.out.println("value: " + stringObjectEntry.getValue());
    });
  }

  @PostMapping("/post02")
  public void post02(@RequestBody PostRequestDto requestData) {
    System.out.println(requestData);
  }
}
```

```java
package com.example.post.dto;

import com.fasterxml.jackson.annotation.JsonProperty;

public class PostRequestDto {
  private String account;
  private String email;
  private String address;
  private String password;

  @JsonProperty("phone_number")
  private String phoneNumber;

  public String getPhoneNumber() {
    return phoneNumber;
  }

  public void setPhoneNumber(String phoneNumber) {
    this.phoneNumber = phoneNumber;
  }

  public String getAccount() {
    return account;
  }

  public void setAccount(String account) {
    this.account = account;
  }

  public String getEmail() {
    return email;
  }

  public void setEmail(String email) {
    this.email = email;
  }

  public String getAddress() {
    return address;
  }

  public void setAddress(String address) {
    this.address = address;
  }

  public String getPassword() {
    return password;
  }

  public void setPassword(String password) {
    this.password = password;
  }

  @Override
  public String toString() {
    return "PostRequestDto{" +
        "account='" + account + '\'' +
        ", email='" + email + '\'' +
        ", address='" + address + '\'' +
        ", password='" + password + '\'' +
        ", phoneNumber='" + phoneNumber + '\'' +
        '}';
  }
}
```

GET Method 를 구현할 때와 거의 똑같다. 다만 controller 메서드의 매개변수로 `@RequestBody` 를 사용한다는 것에 주의해야 한다. 또한, PostRequestDto 객체에 맴버 변수 phoneNumber 에 `@JsonProperty` 라는 어노테이션을 적용했다.

기본적으로 HTTP Request 의 JSON key 는 대문자를 사용하지 않는다. 하지만 Java 프로젝트에서 변수명을 생성할 때 camelCase 로 작성하는 경우가 많다. `@JsonProperty` 어노테이션을 적용하지 않은 상태에서 API Call을 해보면 phoneNumber 에 요청값이 매핑되지 않는다. phone_number 와 phoneNumber 는 너무나도 당연히 다르기 때문이다. 이를 `@JsonProperty` 를 통해 phoneNumber 는 phone_number 이라는 것을 알려주는 것이고, 이를 적용하면 phone_number 의 요청값이 phoneNumber 에 잘 반영되는 것을 볼 수 있다.

이와 같이 표기법이 달라 매핑이 되지 않는 것을 해결하는 방법에는 `@JsonProperty` 를 적용하는 것 말고 객체 전체에 rule 을 적용하는 방법이 있다. `@JsonNaming` 을 활용하는 예시이다.

```java
@JsonNaming(value = PropertyNamingStrategy.SnakeCaseStrategy.class)
public class PostRequestDto {

  private String name;
  private int age;
  private String phoneNumber;

  //.... 생략
}
```

<br/>

---

## PUT Method

POST 방식과 마찬가지로 `@RequestBody` 로 요청을 받는다. `@PutMapping` 로 API 생성한다.

```java
import com.example.put.dto.PostRequestDto;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api")
public class PutApiController {
  @PutMapping("/put")
  public PostRequestDto put(@RequestBody PostRequestDto requestDto) {
    System.out.println(requestDto);

    return requestDto;
  }

  /**
   * Path Variable 를 사용하는 방식은 GET Method 에서 사용한 것과 동일하다.
   */
  @PutMapping("/put/{userId}")
  public PostRequestDto put02(@RequestBody PostRequestDto requestDto, @PathVariable(name = "userId") Long id) {
    System.out.println(id);
    return requestDto;
  }
}
```

```java
package com.example.put.dto;

import com.fasterxml.jackson.databind.PropertyNamingStrategy;
import com.fasterxml.jackson.databind.annotation.JsonNaming;

import java.util.List;

/* Class 전체에 NamingStrategy 를 설정하는 방식. snake_case 로 들어온 key는 camelCase 인 멤버 변수로 자동으로 mapping 된다. */
@JsonNaming(value = PropertyNamingStrategy.SnakeCaseStrategy.class)
public class PostRequestDto {

  private String name;
  private int age;
  private List<CarDto> carList;

  /* Getter/Setter, toString() 코드 생략 */
}
```

```java
import com.fasterxml.jackson.annotation.JsonProperty;
import com.fasterxml.jackson.databind.annotation.JsonNaming;

public class CarDto {

  private String name;

  @JsonProperty("car_number")
  private String carNumber;

  /* Getter/Setter, toString() 코드 생략 */
}
```

<br/>

---

## DELETE Method

DELETE Method 는 그 특성상 많은 데이터를 요청으로 받지 않는다. DB 에 특정 레코드를 지우는데에는 고유값만 필요하다거나 하는 특성 때문이다. 그래서 아주 간단한 예제만 기록하고 넘어간다.

```java
@RestController
@RequestMapping("/api")
public class DeleteApiController {

  @DeleteMapping("/delete/{userId}")
  public void delete(@PathVariable String userId, @RequestParam String account) {
    System.out.println(userId);
    System.out.println(account);
  }
}
```

역시나 마찬가지로 Path Variable 은 `@PathVariable` 을 명시해주고, query parameter 는 `@RequestParam` 을 명시해주고 사용한다.

그리고 DELETE Method 는 그 어떤 값에도 200 OK 를 응답으로 돌려준다.

<br/>

---

## Response 내려주기

Response 는 다양한 형태로 내려줄 수 있다.

```java
@RestController
@RequestMapping("/api")
public class ApiController {

  @GetMapping("/text")
  public String text(@RequestParam String account) {
    return account;
  }
}
```

가장 대표적인 TEXT 를 내려주는 방법이다. 하지만 실제로 Text를 그대로 Response 로 보내주는 사례는 거의 없다.

```java
@RestController
@RequestMapping("/api")
public class ApiController {
  @PostMapping("/post")
  public User json(@RequestBody User user) {
    return user;
  }
}
```

JSON 을 내려주는 대표적인 방법이다. 여기서 의문이 드는 것은 분명 Request Body는 JSON의 형태인데, 컴파일러는 어떻게 Request Body를 객체로 인식할 수 있느냐이다. 이는 "Object Mapper" 라는 녀석이 JSON 과 Object 를 매핑해주기 때문이다. 즉,

1. Request Body (JSON) 를 Object 로 Object Mapper 가 변환
2. Method 가 실행되어 Object 가 반환됨.
3. 2번의 Object 를 Object Mapper 가 다시 JSON 으로 반환.
4. Response 로 보내줌.

의 순서로 진행됨.

위와 같은 방식으로 직접 객체를 JSON 의 형태로 보내줄 수 있고, ResponseEntity 라는 객체로 요구사항에 맞추어 status code 를 수정한다거나 header 를 약간 수정하는 방식으로 response 를 내려 줄 수 도 있다.

```java
@RestController
@RequestMapping("/api")
public class ApiController {
  @PutMapping("/put")
  public ResponseEntity<User> put(@RequestBody User user){
    // CREATED 는 201.
    return ResponseEntity.status(HttpStatus.CREATED).body(user);
  }
}
```

네.

API 를 제작하다보면 RestController 뿐만 아니라 PageController 를 제작하게 될 때도 있다.

```java
@Controller
public class PageController{
  @RequestMapping("/main")
  public String main() {
    return "main.html";
  }
}
```

위 코드는 `resources` 디렉토리에 static 폴더 안에 들어있는 파일을 찾아 (예제의 경우 `main.html`) HTML 을 반환해주는 코드이다. `main()` 메서드는 String 값을 반환함에도 불구하고 HTML 문서를 찾아 이를 반환하는 것이 특이하다.

디렉토리 트리를 참고.

```bash
.
├── main
│   ├── java
│   │   └── com
│   │       └── example
│   │           └── response
│   │               ├── ResponseApplication.java
│   │               ├── controller
│   │               │   ├── ApiController.java
│   │               │   └── PageController.java
│   │               └── dto
│   │                   └── User.java
│   └── resources
│       ├── application.properties
│       ├── static
│       │   └── main.html  # 여기에요!
│       └── templates
└── test
    └── java
        └── com
            └── example
                └── response
                    └── ResponseApplicationTests.java
```

어쨌든 Page Controller 내부에서도 HTML 문서를 반환하는 것이 아닌 Response 를 해야 하는 경우도 있는데 이는 아래와 같이 처리하면 된다.

```java
@Controller
public class PageController {

  @RequestMapping("/main")
  public String main() {
    return "main.html";
  }

  // ResponseEntity 를 response!
  @ResponseBody
  @GetMapping("/user")
  public User user() {
    var user = new User();
    user.setName("Steve");
    user.setAddress("Seoul");

    return user;
  }
}
```

```java
public class User {
  private String name;
  private int age;
  private String phoneNumber;
  private String address;

  /* Getter/Setter, toString() 코드 생략 :) */
}
```

추가로 위의 코드는 user 라고 하는 변수에 타입 추론 기능을 넣은 것이고, 멤버 변수 4개 중 name과 address를 set 해주고 이를 response 로 보내주었다.

이 때 age 와 phoneNumber 의 경우 아무런 값을 할당해주지 않았기 때문에 각각의 default 값이 반환되게 된다. String 의 경우는 null, int 는 0 이다.

```json
{
  "name": "Steve",
  "age": 0,
  "phoneNumber": null,
  "address": "Seoul"
}
```

null 값이 있으면 클라이언트에게 이를 전달하지 않고 싶을 수도 있다. 이 때는 아래와 같이 annotation 을 이용할 수 있다.

```java
@JsonInclude(JsonInclude.Include.NON_NULL)
public class User {
  private String name;
  private int age;
  private String phoneNumber;
  private String address;

  /* Getter/Setter, toString() 코드 생략 :) */
}
```

```json
// 결과
{
  "name": "Steve",
  "age": 0,
  "address": "Seoul"
}
```

age 의 경우는 null 이 default 가 아니라서 생략되지 않았는데, 간단히 아래와 같이 int 타입을 Integer 로 바꿔주면 default 값이 null 이 되게 된다.

```java
@JsonInclude(JsonInclude.Include.NON_NULL)
public class User {
  private String name;
  private Integer age;
  private String phoneNumber;
  private String address;

  /* Getter/Setter, toString() 코드 생략 :) */
  // getter 메서드도 Integer 로 수정해야 한다 :D
  public Integer getAge() {
    return age;
  }
}
```

```json
{
  "name": "Steve",
  "address": "Seoul"
}
```

이제 age 도 잘 생략됨.

어쨌든 RestController 라는 좋은 어노테이션이 있기 때문에, Page Controller 에서는 Rest API Controller 를 생성하지 않는 것이 좋다.
