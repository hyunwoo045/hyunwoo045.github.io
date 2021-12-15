---
title: "MQTT - aedes로 구현하기"
excerpt: "MQTT (Message Queuing Telemetry Transport) 에 대해 정리합니다."

header:
  overlay_image: https://images.unsplash.com/photo-1501785888041-af3ef285b470?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1350&q=80
  overlay_filter: 0.5

tags:
  - aedes
  - MQTT

toc: true
toc_label: "table of content"
toc_icon: "bars"
toc_sticky: true
---

TCP/IP 위에서 동작하는 ISO 표준 프로토콜인 MQTT (Message Queuing Telemetry Transport) 에 대해 정리해보겠습니다!

참고자료: <br/>
[moscajs/aedes 공식 github 페이지](https://github.com/moscajs/aedes#install)<br/>
[MQTT 연동 IoT 서비스](https://www.hardcopyworld.com/?p=3369)<br/>
[MQTT 란?](https://medium.com/@jspark141515/mqtt%EB%9E%80-314472c246ee)<br/>

실습 코드: [hyunwoo045/mqtt-aedes-tutorial](https://github.com/hyunwoo045/mqtt-aedes-tutorial)

틀린 부분이 있을 수 있습니다. hyunwoo045@gmail.com 로 지적해주시면 수정하겠습니다. 감사합니다 :D

<br/>

## MQTT

사람들이 채팅창에 모여서 그룹을 짓고, 그 안에서 메시지를 주고 받는 것처럼, MQTT는 인터넷 네트워크에서 `topic` 을 바탕으로 그룹화된 여러 장치들끼리 서로 메시지를 주고 받을 수 있도록 해주는 프로토콜 입니다. TCP/IP 기반으로 대역폭이 작은 네트워크에서 동작할 수 있도록 설계되었습니다. 하지만 HTTP, TCP등의 통신과 같이 클라이언트-서버 구조로 이루어 진 것이 아닌 `Publisher-Broker-Subscriber` 의 구조를 가지고 있습니다.

![MQTT 기본 구조](https://miro.medium.com/max/1170/1*lKWgSNIYc1Pil5FFoAHMkA.png)<br/>
출처: [MQTT 란?](https://medium.com/@jspark141515/mqtt%EB%9E%80-314472c246ee)

Publisher 가 Topic 을 발행하고, Subscriber 가 Topic 에 '구독' 합니다. Publisher 가 Topic 에 대하여 어떤 메시지를 발행하면 해당 Topic 에 '구독'한 Subscriber 는 그 메시지를 받아서 볼 수 있습니다. 한 Topic 에 대하여 여러 Subscriber 가 구독할 수 있기 때문에 1:N 통신 구축에 유용합니다.

위 단락이 내용이 썩 와닿지 않을 수 있습니다 <s>(저는 그랬어요)</s>. 유튜브에 비유해서 생각해보죠.

![MQTT를 Youtube에 비유한다면?](/images/2021-09-27-MQTT-aedes/mqtt-example.png)

유튜브 크리에이터(Publisher)가 한 명 있습니다. 크리에이터는 자신의 채널에 영상(message)을 업로드 합니다. 이 채널(Topic)을 구독한 구독자(Subscriber)들은 구독 페이지에서 이 크리에이터가 올린 영상을 볼 수 있습니다. 물론 각각 구독자들도 크리에이터(Publisher)가 될 수 있고, 영상을 업로드 할 수 있고, 누군가가 이 크리에이터의 채널에 구독할 수 있습니다. MQTT 의 구조와 흡사하죠?

그렇다면 `Broker` 는 뭘까요? MQTT 는 단순히 Publisher 와 Subscriber 간에 메시지를 주고 받는 구조를 말하는 하나의 프로토콜에 불과합니다. 이를 실제로 구조화 해 줄 무언가가 필요합니다. 그것이 브로커입니다. 크리에이터가 영상을 유튜브라는 중간 매체에 올리고 구독자가 이를 통해 영상을 시청할 수 있듯이, Publisher 와 Subscriber 는 Broker 라고 하는 중계자를 통해 메시지를 주고 받을 수 있는 것입니다.

MQTT를 구현하는 브로커들은 아래와 같습니다.

- Mosquitto
- Aedes (mosca)
- HiveMQ
- ActiveMQ
- RabbitMQ

<br/>

## MQTT 브로커 구현

Aedes 로 브로커를 구현해보겠습니다. Mosca 의 author 인 `Matteo Collina` 가 최근 새롭게 작업중인 패키지입니다. <s>아마도</s> 비슷한 구조일 테니 따끈따끈한 신상을 사용해보도록 하겠습니다.

빈 프로젝트 폴더를 만들고 `mqtt` 와 `aedes` 를 설치합니다.

```bash
# MQTT-tutorial/
$ npm init
$ npm install mqtt aedes
```

`broker.js` 파일을 만들고 기본 설정을 작성합니다. `localhost:1883` 혹은 `127.0.0.1:1883` 을 사용하도록 합니다.

```javascript
// ./broker.js
const aedes = require("aedes")();
const server = require("net").createServer(aedes.handle);
const port = 1883;

// localhost listen to port 1883
server.listen(port, () => {
  console.log("Server started and listening on port", port);
});
```

준비가 끝났습니다. `node broker.js` 로 코드를 동작시키면 서버가 작동하고 broker 로써의 역할을 할 수 있습니다. 하지만 이렇게만 작성하면 뭔가 밋밋하고, 실제로 어떤 동작들을 하고 있는지 알 수 없으니 구독자가 접속했을 때와 구독자가 subscribe 했을 때, 누군가 publish 했을 때 로그를 남기도록 코드를 추가하겠습니다.

```javascript
// ./broker.js
// (...)
aedes.on("client", function (client) {
  console.log(`Client Connected: ${client.id} to ${aedes.id}`);
});

aedes.on("subscribe", function (subscriptions, client) {
  const newTopic = subscriptions[subscriptions.length - 1].topic;
  console.log(`New Subscription! - "${newTopic}" by "${client.id}"`);
});

aedes.on("publish", (packet) => {
  console.log(packet.payload.toString());
});
```

테스트해보기 위해 Subscriber 코드를 작성해 봅니다. `sub.js` 파일을 생성하고 아래 코드를 작성할게요. 연결되었을 때 'test' 라고 하는 `topic` 에 구독하도록 설정하고, 해당 `topic` 에 message 가 발행될 때 마다 message 를 출력하도록 합니다.

```javascript
// MQTT subscriber

const mqtt = require("mqtt");
const client = mqtt.connect("mqtt://localhost:1883");
const topic = "test";

client.on("message", (topic, message) => {
  message = message.toString();
  console.log("Subscriber: ", message);
});

client.on("connect", () => {
  client.subscribe(topic);
});
```

또한 테스트 용도로 Publisher 를 작성합니다. `pub.js` 파일을 생성하고 아래 코드를 작성할게요. 5초마다 "test" 토픽에 "Hello mqtt!" 를 발행합니다.

```javascript
// MQTT publisher
const mqtt = require("mqtt");
const client = mqtt.connect("mqtt://localhost:1883");
const topic = "test";
const message = "Hello mqtt!";

client.on("connect", () => {
  setInterval(() => {
    client.publish(topic, message);
    console.log("Message sent!", message);
  }, 5000);
});
```

준비를 마쳤습니다. 테스트 구동해보겠습니다. 3개의 터미널을 열어 실시간으로 각 코드(`broker.js`, `sub.js`, `pub.js`)가 어떻게 동작하는지 확인해보세요.

```bash
$ node broker.js
Server started and listening on port 1883
```

`node sub.js` 했을 때 `broker.js` 에 아래와 같은 결과가 나타납니다!

```bash
Client Connected: mqttjs_b307f6d5 to 07731bb4-ec90-4a9b-b62f-386842465246
mqttjs_b307f6d5
New Subscription! - "test" by "mqttjs_b307f6d5"
{"clientId":"mqttjs_b307f6d5","subs":[{"topic":"test","qos":0}]}
```

제가 직접 찍은 로그들이 출력되네요. `mqttjs_b307f6d5` 라는 클라이언트가 연결되었고, `test` 토픽에 구독하였음을 알 수 있습니다.

`node pub.js` 하여 publisher 를 실행해보죠. 5초마다 "Hello mqtt!" 를 발행하며, `sub.js` 와 `broker.js` 에도 5초마다 해당 메시지가 찍히는 것을 기대합니다.

![MQTT Plain Test](/images/2021-09-27-MQTT-aedes/mqtt-plain-test.png)<br/>
기대한 대로 잘 나오네요 :D

<br/>

## MQTT server over WebSocket

웹에서 동작되도록 브로커를 구현해보겠습니다. 기존의 `broker.js` 로 구현한 서버로는 웹에서 사용할 수 없습니다. 웹 소켓을 열어 웹과 서버가 통신할 수 있도록 해야 합니다. 따라서 `broker.js` 파일을 아래와 같이 수정해주세요.

호스트는 마찬가지로 로컬이며 포트만 수정합니다. 따라서 `localhost:8888` 혹은 `127.0.0.1:8888` 에서 동작합니다.

```javascript
const aedes = require("aedes")();
const httpServer = require("http").createServer();
const ws = require("websocket-stream");
const port = 8888;

ws.createServer({ server: httpServer }, aedes.handle);

httpServer.listen(port, () => {
  console.log("HTTP Server started and listening on port", port);
});
```

간단한 웹 애플리케이션을 만들어 봅시다. 저는 저에게 친숙한 vue.js 로 만들게요. mqtt 모듈을 사용해야하니 mqtt 도 설치해줍니다.

```
$ vue create vue-mqtt-test
$ cd vue-mqtt-test
$ npm install mqtt
```

아주 간단한 UI를 만들었고 아래와 같은 모습입니다. [레파지토리](https://github.com/hyunwoo045/mqtt-aedes-tutorial/tree/master/frontend/src) 의 `App.vue` 파일을 참고해주세요.

![Vue MQTT-test Home](/images/2021-09-27-MQTT-aedes/vue-mqtt-home.png)씸플.

- Connect 버튼을 클릭하면 mqtt 서버에 연결합니다.
- 메시지를 입력하고 Publish 버튼을 클릭하면 메시지가 발행됩니다.
- 3번째 요소에 '[발행된 시간] 메시지' 형태로 랜더링 됩니다.

`template` 태그와 `style` 태그를 제외하고 `script` 태그의 내용만 살펴보겠습니다. 코드 설명은 간단한 주석으로 대체하겠습니다.

```html
<script>
  import mqtt from "mqtt";
  export default {
    data() {
      return {
        // mqtt broker 서버 config, localhost:8888 에 연결.
        connection: {
          host: "localhost",
          port: 8888,
        },
        topic: "test", // 'test' 토픽에 구독하게 함
        receiveNews: [], // 발행된 메시지들을 쌓는 배열
        message: "", // 발행할 메시지 (input 태그에 바인딩)
        // mqtt.client 객체
        client: {
          connected: false,
        },
      };
    },

    methods: {
      /* 
        f: createConnetion
        이전 `sub.js` 와 유사한 구조입니다. mqtt.connnect, mqtt.subscribe 한 후
        메시지를 받을 때 마다 state.receiveNews 에 푸시하여 배열에 저장합니다.
      */
      createConnection() {
        const { host, port } = this.connection;
        const connectUrl = `ws://${host}:${port}`;
        try {
          this.client = mqtt.connect(connectUrl);
          this.client.connected = true;
        } catch (error) {
          console.log("mqtt.connect error", error);
        }
        this.client.on("connect", () => {
          this.client.subscribe(this.topic);
        });
        this.client.on("message", (topic, message) => {
          let today = new Date();
          this.receiveNews.push(
            `[${today.toLocaleString()}] ${message.toString()}`
          );
        });
      },
      /*
        f: publishHandler
        'test' 에 state.message 를 publish 합니다.
      */
      publishHandler() {
        this.client.publish(this.topic, this.message);
      },
    },
  };
</script>
```

`node broker.js` 로 서버를 시작하고 웹 앱을 2개 열어 여러 구독자에게 메시지가 잘 전달되는지 확인해 봅시다!

![MQTT Test on VUE](/images/2021-09-27-MQTT-aedes/vue-mqtt-test.png)<br/>
한 쪽에서 메시지를 보내면 다른쪽에도 메시지가 잘 나타나네요.

<br/>

## 마치며

지금까지 아주 간단한 코드로 MQTT 프로토콜이 어떻게 동작하는지와 aedes 의 기본 사용법을 기록하였습니다. 다음에 데이터베이스와의 연결과 사용자 인증 기능을 추가하는 실습을 해보고 또 기록 남기도록 하겠습니다.
