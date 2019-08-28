---
layout: post
title: "node.js에서 redis에 publish할 때 오류 발생"
tags: [node, redis, pub/sub]
comments: true
---

node.js에서 redis 사용시, 하나의 연결을 가지고 메세지 수신 및 발신을 전부 처리하고자 할 경우 발생하는 오류 및 대응방법에 대해 정리 해 본다.

# 사건의 시작

node.js에서 redis에 메세지를 publish 하다가 다음과 같은 오류가 발생했다.

> ERR only (P)SUBSCRIBE / (P)UNSUBSCRIBE / QUIT allowed in this context

이 때의 코드는 대략 다음과 같았다.

```javascript
var conn = redis.createClient();

conn.subscribe('example_messages');
conn.on('message', (channel, message) => { doSonething() });

AnotherJob.on('end', async (err, message) =>{
    // 여기서 publish 하는 도중 오류가 발생한다.
    conn.publish('do_more_something', message);
} );
```

# 문제의 원인

[redis의 Pub/Sub 문서](https://redis.io/topics/pubsub)에 보면 다음과 같은 설명이 있다. 

> Messages sent by other clients to these channels will be pushed by Redis to all the subscribed clients.

> A client subscribed to one or more channels should not issue commands, although it can subscribe and unsubscribe to and from other channels. The replies to subscription and unsubscription operations are sent in the form of messages, so that the client can just read a coherent stream of messages where the first element indicates the type of message. The commands that are allowed in the context of a subscribed client are SUBSCRIBE, PSUBSCRIBE, UNSUBSCRIBE, PUNSUBSCRIBE, PING and QUIT.

즉, subscribe 하고 있는 클라이언트에서는 publish 하지 말라는 말이다.


# 문제의 해결

별거 없다. publish를 위한 클라이언트를 하나더 생성해서 publish와 subscribe가 다른 클라이언트에서 동작하도록 해주면 된다.

```javascript
var conn = redis.createClient();
var connForPub = redis.createClient();

conn.subscribe('example_messages');
conn.on('message', (channel, message) => { doSonething() });

AnotherJob.on('end', async (err, message) =>{
    // 이렇게 발신 전용 연결을 하나 더 만들어서 사용하니 해결이 되었다.
    connForPub.publish('do_more_something', message);
} );
```

# 결론

한참을 헤메었지만 결국 답은 Document 안에 다 있었다.
뭔가 안될땐 공식 문서를 잘 읽어 보자.

* https://redis.io/topics/pubsub
* https://stackoverflow.com/questions/22668244/should-i-use-separate-connections-for-pub-and-sub-with-redis
