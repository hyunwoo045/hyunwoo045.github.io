---
title: "Artillery 로 성능 테스트하기"
excerpt: "NodeJS 환경에서 성능 테스트를 해보기 위해 Artillery 알아봅니다."

header:
  overlay_image: https://images.unsplash.com/photo-1501785888041-af3ef285b470?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1350&q=80
  overlay_filter: 0.5

tags:
  - Node.js
  - Artillery

toc: true
toc_label: "table of content"
toc_icon: "bars"
toc_sticky: true
---

## Artillery 로 부하 테스트

사전 조건으로 `Node.js` 를 설치해야 합니다.

<br/>

---

## 참고 자료

[Node 환경에서 Artillery 를 활용한 스트레스 테스트](https://velog.io/@hax0r/Node-%ED%99%98%EA%B2%BD%EC%97%90%EC%84%9C-Artillery-%EB%A5%BC-%ED%99%9C%EC%9A%A9%ED%95%9C-%EC%8A%A4%ED%8A%B8%EB%A0%88%EC%8A%A4-%ED%85%8C%EC%8A%A4%ED%8A%B8)<br/>
[Artillery 를 통한 NODE 환경에서 스트레스 테스트](https://blog.hax0r.info/2020-04-19/stress-test-in-node-with-artillery/)<br/>
[JMeter vs Artillery](https://azevedorafaela.com/2020/06/09/load-tests-jmeter-vs-artillery/)<br/>
[Artillery Docs - 공식 문서](https://www.artillery.io/docs/guides/guides/http-reference)<br/>

<br/>

---

## Artillery 설치

- [설치 가이드](https://artillery.io/docs/guides/getting-started/installing-artillery.html)

- 터미널에 아래 명령어를 입력
  - `$ npm install -g artillery` - 로컬 전역에 설치
  - `$ npm install -D artillery` - 프로젝트에 개발 의존성으로 설치

<br/>

---

## 스크립트 작성

yaml 파일을 작성하여 테스트를 진행하는 방법을 기록합니다.

```yaml
config:
  target: http://localhost:3000
  phases:
    - duration: 100
      arrivalRate: 1
scenarios:
  - name: "방 조회"
    flow:
      - get:
          url: "/chat/room"
```

설명

- target: 인스턴스로 접속할 url 을 적어줍니다.
  - 로컬 서버가 port 3000 에서 돌고 있으니 `http://localhost:3000` 을 입력
- duration: 성능을 측정하는 시간
- arrivalRate: 매초 새로운 가상 유저를 만드는 수

<br/>

---

## 스크립트 실행

아래 명령어로 간단하게 테스트를 실행합니다.

`$ artillery run --output report.json ./test.yaml`

명령어를 실행하고 나면 현재 디렉토리에 `report.json` 파일이 생깁니다. 이를 보기 좋게 HTML 파일로 변환할 수 있습니다.

`$ artillery report ./report.json`

위 명령을 실행하면 역시나 같은 디렉토리에 `report.json.html` 파일에 생깁니다. 이 파일을 실행하면 테스트 결과를 보기 좋게 렌더링 해 준 화면을 볼 수 있습니다.

![](/images/artillery/artillery-report.png)

그림에 보이는 두 개의 그래프에서 지연 시간을 체크할 수 있습니다. 세로가 Latency 를 나타냅니다.

x축에 min, max, p95 등이 있습니다. 이 내용은 아래와 같습니다.

- max: 가장 오래 걸린 요청
- min: 가장 빠르게 온 요청
- p95: 전체 HTTP transanction 중 가장 빠른 것 부터 95% 까지
- p50: 전체 HTTP transanction 중 가장 빠른 것부터 50% 까지
- p99: 전체 HTTP transanction 중 가장 빠른 것부터 99%

그래프 위에는 표를 확인할 수 있습니다.

직관적으로 잘 나와 있으니 보면서 알아가면 되겠습니다.

<br/>

---

## 시나리오 작성

실제 유저가 서비스를 사용하는 시나리오를 작성해보겠습니다. 현재 제가 테스트에 사용하고 있는 예시는 채팅앱의 API 서버 입니다. 유저는 '방 조회 -> 방 만들기 -> 만든 방에 입장' 과 같이 행동할 수 있겠습니다.

```yaml
config:
  target: http://localhost:3000
  phases:
    - duration: 10
      arrivalRate: 1
scenarios:
  - name: "방 조회, 생성, 입장"
    flow:
      - get:
          url: "/chat/room"
      - post:
          url: "/chat/room"
          json:
            roomName: "현우의 채팅방"
            userId: "hyunwoo045"
          capture:
            json: "$._id"
            as: "_id"
      - get:
          url: "/chat/room/enter/{{ _id }}"
```

scenarios.flow 에 API 를 더 추가하는 방식으로 시나리오를 구현할 수 있겠습니다. 여기서 추가로 더 알아야 할 것은 응답으로 받아온 값을 `capture` 를 통해 기억해두었다가 다음 flow 에서 사용할 수 있다는 점입니다. 위 예시에서는 방을 생성하며 받아온 방의 고유 번호인 `_id` 값을 `_id` 라는 변수로써 기억해두었다가 방 입장 url 에 동적으로 입력해 준 모습입니다.

아래는 테스트 결과 입니다.

![](/images/artillery/artillery-report-2.png)

<br/>

---

# 추가 코드 작성법 (21. 12. 08 추가)

조금 더 나은 테스트 환경을 위한 코드 작성법을 기록합니다.

<br/>

---

## 외부 파일에서 데이터 가져오기

사용자가 서비스들을 이용할 때에 특정 서비스에는 항상 같은 값만 입력하지는 않습니다. 예를 들어 인터넷 쇼핑몰에서 제품을 검색할 때 제품은 고정값이 아니죠. 이런 시나리오를 테스트 할 때 외부에서 키워드들을 가지고 와서 랜덤하게 사용할 수 있습니다. `.csv` 파일을 이용합니다. `keywords.csv` 파일을 하나 생성합니다.

```csv
Apple
Banana
Cherry
Mango
Pineapple
```

Artillery 에 `config.payload` 설정에서 파일을 세팅할 수 있습니다. 그리고 `payload.fields` 에서 설정한 명칭으로 `sceneraio` 에서 사용할 수 있겠습니다.

```yaml
config:
  target: "http://localhost:3000"
  phases:
    - duration: 60
      arrivalRate: 5
  payload:
    path: "keywords.csv"
    fields:
      - "keyword"
scenarios:
  - name: "Search and buy"
    flow:
      - post:
          url: "/search"
          json:
            keyword: "{{ keyword }}"
```

로컬 서버를 열고 로그를 찍어보세요. `[ Apple, Banana, Cherry, Mango, Pineapple ]` 이 번갈아가면서 요청 body 로 들어가는 것을 볼 수 있습니다.

아래와 같은 방식으로도 사용 가능합니다.

```csv
id1,password1
id2,password2
id3,password3
```

```yaml
config:
  payload:
    path: "users.csv"
    fields:
      - "id"
      - "password"
scenarios:
  - flow:
      - post:
          url: "/auth"
          json:
            id: "{{ id }}"
            password: "{{ password }}"
```

<br/>

---

## Header 값 설정

API 호출할 때에 헤더를 설정해야 하는 경우도 있죠. 헤더 설정은 간단합니다. `config.defaults.headers` 에 디폴트 헤더값을 설정할 수 있고, `scenarios` 내부에서 개별 API 호출에 해당하는 헤더를 설정할 수도 있습니다.

```yaml
config:
  phases:
    - duration: 60
      arrivalRate: 5
  defaults:
    headers:
      content-type: application/json
      accepts: application/json
scenarios:
  flow:
    - get:
        url: "/"
        headers:
          Authorization: "ThisismyBearerToken:)"
```

<br/>

---

## 테스트 일시정지

테스트 시나리오 중에 일시정지하는 시나리오도 추가할 수 있습니다. 쇼핑몰을 이용하는 이용자가 검색 후 결과 리스트를 받은 후에 잠깐 고민한다는 시나리오를 예시로 들 수 있겠습니다. `sceneraios.flow.think` 를 설정합니다.

```yaml
scenarios:
  - flow:
      - post:
          url: "/search"
          json:
            kw: "{{ keyword }}"
          capture:
            - json: "$.results[0].id"
              as: "productId"
      - get:
          url: "/product/{{ productId }}/details"
      - think: 5
```

<br/>

---

## 여러 환경의 테스트 코드 설정하기

로컬 환경에서의 테스트 코드, 개발 서버 환경에서의 테스트 코드, 상용 서버 환경에서의 테스트 코드를 각각의 파일에 작성하고 저장해두는 것은 뭔가 귀찮습니다. `config.environments` 를 통해 한 파일에 여러 환경을 세팅할 수 있습니다.

```yaml
config:
  target: "http://helloworld.local:3000"
  phases:
    - duration: 10
      arrivalRate: 1
  environments:
    production:
      target: "http://helloworld.prod:3000"
      phases:
        - duration: 120
          arrivalRate: 10
    staging:
      target: "http://localhost:3000"
      phases:
        - duration: 1200
          arrivalRate: 20
```

각각의 환경에서 알맞는 테스트 코드를 실행하기 위해서는 CLI에서 `-e` 플래그를 사용하면 됩니다. 예로 `artillery run -e staging my-script.yaml` 이런식으로요!

<br/>

---

## 실패 환경 설정하기

테스트가 성공인지 실패인지에 대한 조건을 설정할 수 있습니다. 예를 들어 `p95 지연시간`은 200ms 가 넘어서는 안된다거나, error rate 가 5% 가 넘으면 안된다와 같이요. 설정한 조건에 부합하지 않으면 Artillery 는 non-zero code 로 종료될 것입니다.

```yaml
config:
  ensure:
    p95: 200
    maxErrorRate: 5
```
