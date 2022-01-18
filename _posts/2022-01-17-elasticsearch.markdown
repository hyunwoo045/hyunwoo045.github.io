---
title: "[ElasticSearch] npm @elastic/elasticsearch - CRUD"
excerpt: "메모용"

header:
  overlay_image: https://images.unsplash.com/photo-1501785888041-af3ef285b470?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1350&q=80
  overlay_filter: 0.5

tags:
  - Node.js
  - elasticsearch

toc: true
toc_label: "table of content"
toc_icon: "bars"
toc_sticky: true
---

# 설치

최근에 유지 보수를 맡은 서비스에서 elastic search 를 사용하고 있더라. 뭔가 진입 장벽이 높은 느낌이라 울적함. 그래서 일단 알게 된 것들은 최대한 메모하려 한다.

```sh
$ brew update
$ brew install elasticsearch
$ brew services start elasticsearch
```

서비스 중인 실제 db를 건드릴 순 없으니 일단 로컬에 환경 설정

```sh
$ npm install @elastic/elasticsearch@7.13.0
```

`@elastic/elasticsearch` 의 7.13.0 버전을 설치함. 2022년 1월 17일 기준으로 `the client noticed that the server is not a supported distribution of elasticsearch` 이런 에러를 만났는데, 7.16.0 버전을 쓰면 version conflict 난다고 한다. 바로 다운그레이드.

<br/>

# Import

임포트는 굉장히 간단함. 간단한 auth 를 설정할 수도 있는 것으로 보이는데 일단 내가 보고 있는 서비스는 아래와 같은 아주 씸플한 코드로도 연결되는 걸 보니 별 설정 없는 듯

```js
const { Client } = require("@elastic/elasticsearch");
const client = new Client({ node: "http://127.0.0.1:9200" });
```

<br/>

# Create

MySQL 같은 관계형 데이터베이스는 테이블을 따로 만들고 나서 데이터를 삽입해야 하는데, elastic search 는 딱히 그렇지는 않은 듯. 관계형 데이터베이스에서의 table 이 곧 `index` 인 듯 함.

'my-index' 라는 인덱스에 데이터를 넣는 코드는 아래와 같다.

```js
async function testcase01() {
  await client.index({
    index: "my-index",
    body: {
      name: "MOOCHI",
      age: 18,
      address: "Yong-In",
    },
  });
}
```

당연히 좀 복잡한 구조의 객체도 만들 수 있음

```js
async function testcase02() {
  await client.index({
    index: "my-index",
    body: {
      profile: {
        name: "MOOCHI",
        job: "BAEK-SOO",
        age: 18,
      },
      contact: {
        phone: "010-1577-1577",
        email: "moochi@gmail.com",
        sns: {
          facebook: "lifewasted@facebook.com",
          twitter: "wasting@twitter.com",
        },
      },
    },
  });
}
```

<br/>

# Read

검색은 일단 2가지 방법만 알아뒀음. `client.get` 과 `client.search` 임. `get` 은 `id` 라는 각 데이터마다 가지고 있는 고유값을 가지고 검색하는 것. 따라서 결과는 항상 하나임. `search` 는 조건 검색임. 여러 결과가 나올 수 있음.

`get` 의 예시

```js
async function testcase03() {
  const { body } = await client.get({
    index: "my-index",
    id: "1",
  });
  console.log(body._source); // get 의 검색 결과
}
```

`search` 의 예시

```js
async function testcase04() {
  const { body } = await client.search({
    index: "my-index",
    body: {
      query: {
        match: {
          name: "MOOCHI",
        },
      },
    },
  });
  console.log(body.hits.hits); // search 의 검색 결과 (배열임. 히트다 히트!)
}
```

여기서 query 라는 개념을 썼는데 좀 어려움. 깊게 파 볼 필요는 확실히 있음. 복잡한 객체 구조에서 깊숙히 있는 값을 조건으로 둘려면 어케할까 하면서 삽질을 좀 했는데 아래처럼 하니까 되긴 하더라...

```js
async function testcase05() {
  const { body } = await client.search({
    index: "my-index",
    body: {
      query: {
        match: {
          "contact.sns.facebook": "lifewasted@facebook.com",
        },
      },
    },
  });
  console.log(body.hits.hits);
}
```

그리고 이번 메모에서는 계속 `match` 만 사용하지만 사실 `bool`, `gt` 같은 것도 쓸 수 있음. 근데 지금 딱히 알아본 것도 아니고 나중에 써보면 보강해야징~

# Delete

솔직히 READ 파트의 get, search 랑 비슷함. `delete`, `deleteByQuery` 로 나뉘는데 delete는 get처럼 id를 가지고 한 데이터만 삭제하고, deleteByQuery 는 쿼리를 통해 여러 데이터를 한 번에 삭제한다.

delete 의 예시

```js
async function testcase06() {
  await client.delete({
    index: "my-index",
    id: "1",
  });
  console.log("bye-bye");
}
```

deleteByQuery 의 예시

```js
async function testcase07() {
  /* 여러 데이터가 삭제되는 감동을 위해 데이터를 좀만 추가하자 */
  await client.index({
    index: "my-index",
    body: {
      name: "MOOCHI",
      job: "백수",
    },
  });

  await client.index({
    index: "my-index",
    refresh: true,
    body: {
      name: "HWKIM",
      job: "백수",
    },
  });

  await client.index({
    index: "my-index",
    refresh: true,
    body: {
      name: "GREAT GATSBY",
      job: "돈 많은 부자",
    },
  });

  await client.deleteByQuery({
    index: "my-index",
    body: {
      query: {
        match: {
          job: "백수",
        },
      },
    },
  });
}
```

백수를 모두 없애보자.

```js
async function testcase08() {
  const { body } = client.search({
    index: "my-index",
    body: {
      query: { match: {} },
    },
  });
  console.log(body.hits.hits);
}
```

위대한 게츠비만 살아남았을 것이다.

<br/>

# Update

얘도 비슷하다. `update` 가 있고 `updateByQuery` 가 있다. 이하 생략

```js
// 특정 필드 하나만 수정
async function testcase09() {
  await client.update({
    index: "my-index",
    id: "1",
    body: {
      name: "POOR GATSBY",
    },
  });
}

// 데이터 전체를 수정
async function testcase10() {
  const newBody = {
    name: "POOR GATSBY",
    job: "백수",
  };
  await client.update({
    index: "my-index",
    id: "1",
    body: {
      doc: {
        ...newBody,
      },
    },
  });
}
```

body 를 케이크를 먹는 것처럼 쉽게 수정하기 위해서는 doc 이라는 field 를 이용하자.

`updateByQuery` 예시

```js
async function testcase11() {
  await client.index({
    index: "my-index",
    refresh: true,
    body: {
      name: "hwkim",
      nickname: "moochi",
    },
  });

  await client.updateByQuery({
    index: "my-index",
    body: {
      query: {
        match: {
          name: "hwkim",
        },
      },
      script: {
        lang: "painless",
        source: "ctx._source.nickname = 'sosospoon'",
      },
    },
  });
}
```

`updateByQuery` 의 body 에는 script 라는 필드를 사용하게 되는데, 아직은 정확히 이게 뭐다, 저게 뭐다고 설명할 순 없다... (일단은 되는 거 기록ㅠ) `painless` 는 뭐고, `ctx` 는 뭔지는 나중에 알아보는 걸로 하고 `updateByQuery` 는 말 그대로 body.query 에 해당하는 여러 데이터를 수정하겠다는 것이다.

배열, 객체 데이터를 어떻게 핸들링 할 것이냐가 진짜 좀 까다로웠는데 (도대체 배열 변수에 `add({ name: "빡빡이" })` 같이 객체를 직접 넣는 코드는 안되는 건지이잉) 우선은 아래와 같이 하는 것으로 방안을 찾았음.

```js
async function testcase12() {
  await client.index({
    index: "my-index",
    refresh: true,
    body: {
      name: "hwkim",
      nickname: "moochi",
      hobby: [],
      sns: {
        facebook: "",
        instagram: "",
      },
    },
  });

  // 배열에 push
  await client.updateByQuery({
    index: "my-index",
    body: {
      query: {
        match: {
          name: "hwkim",
        },
      },
      script: {
        lang: "painless",
        source: `ctx._source.hobby.add('game')`,
      },
    },
  });

  // params 필드 활용 (위에거랑 같이 쓰지마셈)
  await client.updateByQuery({
    index: "my-index",
    body: {
      query: {
        match: {
          name: "hwkim",
        },
      },
      script: {
        lang: "painless",
        params: {
          facebook: "www.facebook.com",
          instagram: "www.instagram.com",
        },
        source: `ctx._source.sns.facebook = params.facebook;
        ctx.source.sns.instagram = params.instagram;`,
      },
    },
  });
}
```

아래와 같이 응용 가능.

```js
async function testcase13() {
  const sns = {
    facebook: "www.facebook.com",
    instagram: "www.instagram.com",
  };
  await client.updateByQuery({
    index: "my-index",
    body: {
      query: {
        match: {
          name: "hwkim",
        },
      },
      script: {
        lang: "painless",
        params: {
          sns,
        },
        source: `ctx._source.sns = params.sns`,
      },
    },
  });
}
```

위에서 징징 거렸던 배열에 객체 집어넣는 건 아래와 같이 (도대체 이거 때문에 몇 시간을 날려 먹었는지)

```js
async function testcase14() {
  const hobby = {
    game: "ori",
    exercise: "breath",
  };
  await client.updateByQuery({
    index: "my-index",
    body: {
      query: {
        match: {
          name: "hwkim"
        }
      }
      script: {
        lang: "painless",
        params: {
          hobby,
        },
        source: `ctx._source.hobby.add(params.hobby)`
      }
    }
  })
}
```

사실 엘라스틱서치의 진가는 이런 CRUD 에 있는게 아닌 거 같은데 지금 보고 있는 서비스는 단순히 데이터베이스를 대체할 용도로 쓰고 있는 것 같아 여기까지만 일단 기록한다.
