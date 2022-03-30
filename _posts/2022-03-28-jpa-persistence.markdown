---
title: "Spring - JPA 프로그래밍 (3)"
excerpt: "Spring Data, JPA 프로그래밍 공부 노트"

header:
  overlay_image: https://images.unsplash.com/photo-1501785888041-af3ef285b470?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1350&q=80
  overlay_filter: 0.5

tags:
  - Java
  - Spring Boot
  - JPA

toc: true
toc_label: "table of content"
toc_icon: "bars"
toc_sticky: true
---

# 영속성

사라지지 않고 지속적으로 접근 가능하다는 의미. 메모리에 저장되는 데이터는 서비스가 종료되면 사라진다. 데이터가 사라지지 않도록 하기 위해서는 파일로 저장하거나 데이터베이스에 저장하는 등의 방식을 사용해야함. JPA 도 Java Persistence API 의 줄임말로 데이터의 영속성을 위한 프레임워크임. 이를 이용하기 위해서 영속성 설정을 해야 하지만 Spring boot 환경에서는 빌드 환경에서 implemetation 만 설정해주어도 자동으로 JPA 의 영속성 기능들을 사용할 수 있게 된다.

```js
// gradle 환경에서..
dependencies {
  //...
  implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
}
```

- `application.yaml`

```yaml
spring:
  h2:
    console:
      enabled: true
  jpa:
    show-sql: true
    properties:
      hibernate:
        format_sql: true
    generate-ddl: true
    hibernate:
      ddl-auto: create-drop
  datasource:
    url: jdbc:mysql://localhost:3306/book_manager
    username: root
    password:
server:
  port: 8080
```

이렇게 설정하고 자동 생성되어 있는 `contextLoads()` 테스트를 실행한 후 로그를 살펴보면 아래와 같은 로그가 생기는 것을 볼 수 있음

```shell
INFO 7434 --- [    Test worker] org.hibernate.dialect.Dialect            : HHH000400: Using dialect: org.hibernate.dialect.MySQL8Dialect
```

얼추봐도 MySQL 8 버전의 dialect 를 사용한다는 것을 유츄할 수 있다. dialect 는 JPA 가 쿼리를 생성할 때에 어떤 데이터베이스의 언어에 맞춰 쿼리를 생성할 것인가에 대한 정의라고 생각하면 될 듯. (dialect = 방언, 같은 언어여도 지역마다 언어가 조금씩 다르듯이..). 위 `application.yaml` 파일의 spring.datasource 영역을 지우고 테스트를 다시 실행해보면 이 전에 게속 사용하던 H2 DB dialect 를 사용하는 것을 볼 수 있음.

- `generate-ddl` 항목은 프로젝트 빌드 시 필요한 테이블을 자동으로 생성할 것인지에 대한 여부. MySQL 에 필요한 테이블들을 생성하지 않고 이전에 작성했던 테스트들을 돌려보면 테이블이 없다는 에러가 뜸.
- `hibernate.ddl-auto` : `None`, `create-drop`, `create` 등의 설정을 할 수 있으며 어느 시점에 ddl query 를 자동 적용할 것인지에 대한 설정임.

<br/>

## @Transactional 과 flush

일반적으로 트랜잭션은 "데이터베이스의 상태를 변경하는 작업" 혹은 "한번에 수행되어야 하는 연산들"을 의미함. 이 의미를 놓고 `@Transactional` 을 보면 이해가 약간 되는듯 싶기도 하다.

```java
import com.fastcampus.jpa.bookmanager.domain.User;
import com.fastcampus.jpa.bookmanager.repository.UserRepository;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import javax.transaction.Transactional;

@SpringBootTest
@Transactional
public class EntityManagerTest {
  @Autowired
  private UserRepository userRepository;

  @BeforeEach
  void saveUser() {
    User user = new User();
    user.setName("moochi.Kim");
    user.setEmail("moochi@gmail.com");

    userRepository.save(user);
  }

  @Test
  void cacheFindTest2() {
    User user = userRepository.findById(1L).get();

    user.setName("moochi");
    userRepository.save(user);
    System.out.println("1 >>> " + userRepository.findById(1L).get());

    user.setEmail("moochikim@gmail.com");
    userRepository.save(user);
    System.out.println("2 >>> " + userRepository.findById(1L).get());
  }
}
```

대충 이런 테스트가 있다 치자. 주목해야 할 점은 테스트 메서드에 `@Transactional` 이 붙어있다는 점. 이 테스트를 실행하고 로그상 쿼리를 보면

```
Hibernate:
    insert
    into
        user
        (created_at, updated_at, email, gender, name)
    values
        (?, ?, ?, ?, ?)
Hibernate:
    insert
    into
        user_history
        (created_at, updated_at, email, name, user_id)
    values
        (?, ?, ?, ?, ?)
1 >>> User(super=BaseEntity(createdAt=2022-03-29T08:44:51.133, updatedAt=2022-03-29T08:44:51.133), id=1, name=moochi, email=moochi@gmail.com, gender=null)
2 >>> User(super=BaseEntity(createdAt=2022-03-29T08:44:51.133, updatedAt=2022-03-29T08:44:51.133), id=1, name=moochi, email=moochikim@gmail.com, gender=null)
2022-03-29 08:44:51.216  INFO 11647 --- [    Test worker] o.s.t.c.transaction.TransactionContext   : Rolled back transaction for
```

로그는 정상적으로 수정한 데이터대로 잘 나왔지만, 로그 전체를 살펴보더라도 update 쿼리가 존재하지 않고, 로그를 찍은 후 바로 `@Transactional` 의 로직대로 롤백이 일어나는 것을 볼 수 있음. 이 말인 즉슨 실제 데이터베이스에는 아무 업데이트 내용이 반영되지 않았다는 의미임. 업데이트가 일어난대로 바로바로 데이터베이스에 반영되도록 하고 싶다면 2가지 방법을 이용하면 된다.

1. `@Transactional` 어노테이션을 뺀다.

트랜잭션의 의미대로 메서드 내의 모든 로직을 수행한 후 commit 하겠다는 의미이므로 라인마다 commit 이 일어나게 하고 싶다면 `@Transactional` 어노테이션을 빼자.

2. `flush()`

JpaRepository 인터페이스를 구현하는 repository 의 flush 기능을 이용하자. 커밋이 필요한 위치마다 flush 를 사용하여 영속성을 부여한다. 위 예시를 아래와 같이 고치면 update 쿼리를 볼 수 있음.

```java
import com.fastcampus.jpa.bookmanager.domain.User;
import com.fastcampus.jpa.bookmanager.repository.UserRepository;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import javax.transaction.Transactional;

@SpringBootTest
@Transactional
public class EntityManagerTest {
  @Autowired
  private UserRepository userRepository;

  @BeforeEach
  void saveUser() {
    User user = new User();
    user.setName("Moochi");
    user.setEmail("moochi@gmail.com");

    userRepository.save(user);
  }

  @Test
  void cacheFindTest2() {
    User user = userRepository.findById(1L).get();

    user.setName("moochi");
    userRepository.save(user);
    userRepository.flush();
    System.out.println("1 >>> " + userRepository.findById(1L).get());

    user.setEmail("moochikim@gmail.com");
    userRepository.save(user);
    userRepository.flush();
    System.out.println("2 >>> " + userRepository.findById(1L).get());
  }
}
```

```
Hibernate:
    insert
    into
        user
        (created_at, updated_at, email, gender, name)
    values
        (?, ?, ?, ?, ?)
Hibernate:
    insert
    into
        user_history
        (created_at, updated_at, email, name, user_id)
    values
        (?, ?, ?, ?, ?)
Hibernate:
    update
        user
    set
        created_at=?,
        updated_at=?,
        email=?,
        gender=?,
        name=?
    where
        id=?
Hibernate:
    insert
    into
        user_history
        (created_at, updated_at, email, name, user_id)
    values
        (?, ?, ?, ?, ?)
Hibernate:
    update
        user_history
    set
        created_at=?,
        updated_at=?,
        email=?,
        name=?,
        user_id=?
    where
        id=?
1 >>> User(super=BaseEntity(createdAt=2022-03-29T08:47:05.329, updatedAt=2022-03-29T08:47:05.408), id=1, name=moochi, email=moochi@gmail.com, gender=null)
Hibernate:
    update
        user
    set
        created_at=?,
        updated_at=?,
        email=?,
        gender=?,
        name=?
    where
        id=?
Hibernate:
    insert
    into
        user_history
        (created_at, updated_at, email, name, user_id)
    values
        (?, ?, ?, ?, ?)
2 >>> User(super=BaseEntity(createdAt=2022-03-29T08:47:05.329, updatedAt=2022-03-29T08:47:05.436), id=1, name=moochi, email=moochikim@gmail.com, gender=null)
```

네.

## LifeCycle

1. 비영속 상태 : 영속성 컨텍스트가 해당 entity 객체를 관리하지 않는 상태

생성한 Entity 객체에서 Column 에 `@Transient` 를 붙히는 것은 영속화에서 제외하겠다는 의미
