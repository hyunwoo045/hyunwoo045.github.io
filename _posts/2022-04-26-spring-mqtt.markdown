---
title: "Spring Boot - MQTT 메시지 채널 구성"
excerpt: "MQTT 메시지 채널 구축하기"

header:
  overlay_image: https://images.unsplash.com/photo-1501785888041-af3ef285b470?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1350&q=80
  overlay_filter: 0.5

tags:
  - Java
  - Spring Boot
  - Swagger

toc: true
toc_label: "table of content"
toc_icon: "bars"
toc_sticky: true
---

# MQTT

예전에 MQTT 에 대해 다뤄본 적이 있었다. 그 땐 nodejs 로 브로커를 구현했었고 이번엔 스프링부트에서 구현해보자. [예전 MQTT 포스트](https://hyunwoo045.github.io/MQTT-aedes/)

<br/>

# 기본 설정

테스트하기 위해 로컬에 mqtt 브로커를 설치. homebrew 를 통해 mosquitto 브로커를 설치하고 "MQTT Explorer" 를 설치하여 잘 되나 확인해볼거임.

대충 이런 모습의 창.

![](/images/2022-04-26-spring-mqtt/1.png)

스프링부트 Integration 에 포함되어 있는 `spring-integration-mqtt` 를 사용.

```
implementation 'org.springframework.boot:spring-boot-starter-integration'
implementation 'org.springframework.integration:spring-integration-mqtt'
```

<br/>

# OutboundChannel

작업 과정에서 메시지를 내보내는 로직만 필요했기 때문에 일단은 OutboundChannel 만 구성함. MQTT 연결 정보를 설정한 `DefaultMqttPahoClientFactory`로 MQTT 클라이언트를 등록하여 빈으로 등록. 그리고 `MqttPahoMessageHandler` 메시지 발행 채널을 구성.

```java
@Configuration
@IntegrationComponentScan
public class MqttConfig {
  private static final String MQTT_USERNAME = "username";
  private static final String MQTT_PASSWORD = "password";
  private static final String BROKER_URL = "tcp://localhost:1883";

  @Bean
  public MqttPahoClientFactory mqttClientFactory() {
    DefaultMqttPahoClientFactory factory = new DefaultMqttPahoClientFactory();
    MqttConnectOptions options = new MqttConnectOptions();
    options.setServerURIs(new String[] { BROKER_URL });
    options.setUserName(MQTT_USERNAME);
    options.setPassword(MQTT_PASSWORD.toCharArray());
    factory.setConnectionOptions(options);
    return factory;
  }

  @Bean
  @ServiceActivator(inputChannel = "mqttOutboundChannel")
  public MessageHandler mqttOutbound() {
    MqttPahoMessageHandler messageHandler =
        new MqttPahoMessageHandler("testClient", mqttClientFactory());
    messageHandler.setAsync(true);
    messageHandler.setDefaultTopic("testTopic");
    return messageHandler;
  }

  @Bean
  public MessageChannel mqttOutboundChannel() {
    return new DirectChannel();
  }

  @MessagingGateway(defaultRequestChannel = "mqttOutboundChannel")
  public interface MyGateway {
    void sendToMqtt(String data, @Header(MqttHeaders.TOPIC) String topic);
  }
}
```

마지막에 `@MessagingGateway` 가 선언된 메시지 게이트웨이를 생성하여 메시지를 발송할 수 있도록 함.

아래와 같이 테스트해보자.

```java
@SpringBootTest
public class MqttConfigTest {

  @Autowired
  private MqttConfig.MyGateway myGateway;

  @Autowired
  private ObjectMapper mapper;

  @Test
  void mqttTest() {
    myGateway.sendToMqtt("Hello", "/message/topic");
    ObjectNode obj = mapper.createObjectNode();
    obj.put("apiCode", "group");
    obj.put("mode", "add");
    obj.put("group", "1234ABCD");
    myGateway.sendToMqtt(obj.toString(), "/message/group");
  }
}
```

![](/images/2022-04-26-spring-mqtt/2.png)

잘 날아감

<br/>

# 갈무리

InboundChannel 을 구성해야 할 일이 있으면 추가할 것임.

<br/>

## 참고 자료

- [Spring MQTT Support - Spring 공식 문서](https://docs.spring.io/spring-integration/reference/html/mqtt.html#mqtt-inbound)
- [스프링 부트 MQTT 클라이언트 메시지 채널 구성하기](https://kdevkr.github.io/spring-boot-integration-mqtt/)
- [[Spring Integration] MQTT 연동](https://velog.io/@csh0034/Spring-Integration-MQTT-%EC%97%B0%EB%8F%99)
