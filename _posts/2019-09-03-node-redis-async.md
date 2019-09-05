---
layout: post
title: "node.js에서 redis클라이언트 비동기로 사용하기"
tags: [node, redis, async]
comments: true
---

node.js에서 redis 연결시 많이 사용하는 [node_redis](https://github.com/NodeRedis/node_redis)는 기본적으로 비동기 메소드를 지원하지 않는다. 따라서 ES5의 Promise패턴이나 ES7의 async-await같은 기능을 바로 이용할 수는 없다. 그런데 node.js v8.0.0 이상 버전부터 추가된 [util.promisify](https://nodejs.org/api/util.html#util_util_promisify_original) 를 이용하면 비동기로 동작하지 않는 메소드도 Promise 스타일의 비동기 콜백을 사용 할 수 있다.

# 기본 설정

redis client를 초기화 했다고 할 때, get 메소드를 비동기 메소드로 만든다. 이 때, 비동기로 동작하는 메소드의 실행컨텍스트를 유지시켜 주기위해 해당 컨텍스트를 바인드 해 준다.

```javascript
const {promisify} = require('util');
const getAsync = promisify(client.get).bind(client);
```

# 비동기 호출방법

## 1. Promise 를 이용한 방법

```javascript
// We expect a value 'foo': 'bar' to be present
// So instead of writing client.get('foo', cb); you have to write:
return getAsync('foo').then(function(res) {
    console.log(res); // => 'bar'
});
```

### 2. async-await 를 이용한 방법
```javascript
async myFunc() {
    const res = await getAsync('foo');
    console.log(res);
}
```

# 참고링크

* https://github.com/NodeRedis/node_redis#promises
* https://nodejs.org/api/util.html#util_util_promisify_origina