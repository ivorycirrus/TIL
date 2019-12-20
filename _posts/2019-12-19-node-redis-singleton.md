---
layout: post
title: "Express router에 Redis 클라이언트 1개만 생성하기"
tags: [node, redis, singleton]
comments: true
---

Node.js에서 웹 프레임워크를 고민할 때 Express는 좋은 선택이 될 수 있다. Express에서는 URL패턴을 구분해서 모듈화 할 수 있는 라우터 미들웨어를 지원하고 있고, 많은 경우 업무별로 URL을 분리하여 별도의 라우터 파일에서 코드를 작성하곤 한다.

이 때, 자바스크립트 파일이 분리가 되는 경우 각각의 파일에서 ```require()```함수를 이용해서 모듈을 불러다 쓰는 경우가 있다. 특히 DB와 같이 별도의 시스템에 연결이 필요한 경우 각각의 파일에서 모듈을 불러왔을 때, 매 비지니스 로직 처리마다 연결을 맺고 끊는 동작이 반복적으로 발생할 수 있다. 

여기에서는 Redis의 인스턴스를 하나만 생성해서, Express의 라우터 모듈간 해당 인스턴스를 공유하는 방법에대해 적어본다. 물론 모듈별로 파일이 분리되어 있더라도 인트선스가 공유되어 단일 연결로 처리가 되도록 구성할 것이다.

# 문제의 시작

먼저 redis 서버를 설치하고, 아래와 같이 3개의 파일을 작성한다.
* ```app.js``` : 메인 애플리케이션 파일, ```/main``` URL의 라우터를 포함한다
* ```user.js``` : ```/user``` URL의 라우터가 정의되어 있는 파일
* ```redis_instance.js``` : Redis 연결을 정의한 모듈. 

Node로 ```app.js```를 실행하면 Express를 이용해서 8080 포트로 웹서비스를 하는 예시이다. ```/main```과 ```/user``` URL의 요청을 처리하며, user의 경우 별도의 파일에서 처리한다. 웹 요청시에는 해당 모듈의 이름과 요청시각을 Redis에 기록하는 동작을 처리하고 있다.


```javascript
// app.js
const redis = require('./redis_instance');
const client = redis.getConnection();

const http = require('http');
const express = require('express');

var app = express();

app.use('/user', require('./user.js'));
app.use('/main', (req, res) => {
    client.hset('master', 'main', new Date().toUTCString());
    console.log('MAIN '+new Date().toUTCString())
    res.end('MAIN');
});


const server = http.Server(app);
server.listen(8080, () =>{
    console.log('listening on localhost:8080')
});
```

```javascript
//user.js
var express = require('express');
var router = express.Router();

router.get('/', (req, res) => {
    client.hset('master', 'user', new Date().toUTCString());
    console.log('USER '+ new Date().toUTCString())
    res.end('USER');
});

module.exports = router;
````

```javascript
//redis_instance.js
var redis = require('redis');
class Redis {
    constructor() {
         this.host = process.env.REDIS_HOST || 'localhost'
        this.port = process.env.REDIS_PORT || '16379'
        this.connected = false
        this.client = null

    }
   getConnection() {
        if(this.connected) return this.client
        else {
            console.log('New redis instance');
           this.client =  redis.createClient({
                host: this.host,
                port: this.port
            })
            return this.client
        }

    }
}

module.exports = new Redis()
```

# 문제 상황
```app.js```를 실행하여 서버를 기동한 다음, main과 user를 번갈아 두번씩 호출 하면 콘솔에 아래와 같이 출력된다.
```
$ node app
New redis instance
New redis instance
listening on localhost:8080
MAIN Fri, 20 Dec 2019 07:44:44 GMT
USER Fri, 20 Dec 2019 07:44:49 GMT
MAIN Fri, 20 Dec 2019 07:44:54 GMT
USER Fri, 20 Dec 2019 07:44:58 GMT
```

redis 연결이 2개가 만들어지며 서버가 기동됫고, main과 user가 번갈아가며 두번씩 호출된 것을 확인 할 수 있다.

이 때, redis 모니터링 로그에는 아래와 같은 기록이 남는다.
```
$ redis-cli -h 127.0.0.1 -p 16379 monitor
OK
1576827871.979559 [0 172.17.0.1:53696] "info"
1576827871.980246 [0 172.17.0.1:53694] "info"
1576827884.242927 [0 172.17.0.1:53696] "hset" "master" "main" "Fri, 20 Dec 2019 07:44:44 GMT"
1576827889.585328 [0 172.17.0.1:53694] "hset" "master" "user" "Fri, 20 Dec 2019 07:44:49 GMT"
1576827894.417528 [0 172.17.0.1:53696] "hset" "master" "main" "Fri, 20 Dec 2019 07:44:54 GMT"
1576827898.051999 [0 172.17.0.1:53694] "hset" "master" "user" "Fri, 20 Dec 2019 07:44:58 GMT"
```

자세히 보면 main 으로 요청된 메세지는 **53696**포트에 연결된 클라이언트에서, user 로 요청된 메세지는 **53694**포트에 연결된 클라이언트로부터 명령을 수신한 것을 볼 수 있다. 

즉, app.js 와 user.js가 각각의 Redis연결을 생성해서 요청을 하고 있다는 것이다. 모듈이 더 많아질 경우 파일당 redis 연결을 무분별하게 생성 할 것이며, 이는 redis서버에 부하를 발생할 수 있는 소지가 될 수 있다.


# 문제의 해결
이 상황의 문제는 Redis를 여러 파일에서 직접 연결하는데 있다. Express를 초기화하는 파일에서 Redis연결을 1개만 생성하고, 연관된 router모듈에 해당 연결을 공유하는 것으로 해결할 수 있을 것이다.

이에 ```app.js```의 라우터를 정의하는 코드 앞에 다음과 같이 미들웨어를 하나 추가한다. 이 때, request를 담고있는 객체에 Redis 클라이언트 연결을 추가해서 다음 모듈로 전파하면 하위 모든 라우터 모듈에서 받아서 사용할 수 있을 것이다.

```javascript
// [수정] app.js
const redis = require('./redis_instance');
const client = redis.getConnection();

const http = require('http');
const express = require('express');

var app = express();

// [수정1] 미들웨어 추가
app.use((req, res, next) =>{
    req.client = client;
    next();
});

app.use('/user', require('./user.js'));
app.use('/main', (req, res) => {
    // [수정2] req 객체에서 Redis 연결을 받아와서 사용 
    req.client.hset('master', 'main', new Date().toUTCString());
    console.log('MAIN '+new Date().toUTCString())
    res.end('MAIN');
});


const server = http.Server(app);
server.listen(8080, () =>{
    console.log('listening on localhost:8080')
});
```

마찬가지로 ```user.js```파일오 수정해 준다


```javascript
// [수정] user.js
var express = require('express');
var router = express.Router();

// [수정3] Redis 모듈 require 제거

router.get('/', (req, res) => {
    // [수정4] req 객체에서 Redis 연결을 받아와서 사용 
    req.client.hset('master', 'user', new Date().toUTCString());
    console.log('USER '+ new Date().toUTCString())
    res.end('USER');
});

module.exports = router;
```

# 수정 결과

앞서 테스트한 것과 마찬가지로 ```app.js```를 실행하여 서버를 기동한 다음, main과 user를 번갈아 두번씩 호출 하면 콘솔에 아래와 같이 출력된다.
```
$ node app
New redis instance
listening on localhost:8080
MAIN Fri, 20 Dec 2019 07:46:59 GMT
USER Fri, 20 Dec 2019 07:47:03 GMT
MAIN Fri, 20 Dec 2019 07:47:07 GMT
USER Fri, 20 Dec 2019 07:47:10 GMT
```

출력 내용은 큰 차이가 없어 보이지만, **New redis instance**라는 메세지가 한번만 출력된 것으로 보아 연결은 1개만 생성됫다는 것을 확인 할 수 있다.

이 때, redis 모니터링 로그에는 아래와 같은 기록이 남는다.

```
$ redis-cli -h 127.0.0.1 -p 16379 monitor
OK
1576828013.840570 [0 172.17.0.1:53700] "info"
1576828019.801135 [0 172.17.0.1:53700] "hset" "master" "main" "Fri, 20 Dec 2019 07:46:59 GMT"
1576828023.985582 [0 172.17.0.1:53700] "hset" "master" "user" "Fri, 20 Dec 2019 07:47:03 GMT"
1576828027.191314 [0 172.17.0.1:53700] "hset" "master" "main" "Fri, 20 Dec 2019 07:47:07 GMT"
1576828030.288646 [0 172.17.0.1:53700] "hset" "master" "user" "Fri, 20 Dec 2019 07:47:10 GMT"
```

모든 연결에서부터 메세지의 전송은 **53700**포트에 연결된 하나의 클라이언트로부터 요청되는 것을 볼 수 있다.

정리하자면, Node.js의 ```require()```함수는 모듈을 캐싱하여 한번만 로딩해게 해 주기는 하나, 외부모듈과의 단일 연결을 보장하지는 않는다. 따라서 Express 프레임워크의 라우터 파일 각각에서 단일 리소스를 공유해서 쓰고자 하는 경우, 서버 기동시에 라우터 모듈에서 재사용할 리소스를 등록하는 미들웨어를 추가하는 방식으로 연결을 간소화 하는 방법을 고려해 볼 수 있을 것이다.

# 참고 링크

* redis_instance.js : https://stackoverflow.com/a/54301599/988667
