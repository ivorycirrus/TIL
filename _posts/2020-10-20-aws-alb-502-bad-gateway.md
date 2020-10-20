---
layout: post
title: "AWS 502 error on ALB"
tags: [aws, alb, 502, node.js]
comments: true
---

AWS의 ALB를 설정하면서 대상그룹(Target group)에 node.js로 웹서비스를 하는 EC2를 연결했다. 그런데 GET요청시 간헐적으로 ```502 Bad gateway``` 오류가 발생하는 현상이 있었다.

# 1. 문제의 원인
ALB의 대상그룹의 인스턴스 상태체크도 정상이고, 간헐적으로 안된다 뿐이지 대부분 정상적으로 node.js 서비스에 연결이 되는 것으로 보아 보안그룹과 같은 네트워크 설정은 아닌 것으로 판단했다. ALB에 연결된 EC2 인스턴스가 1개라서 다른 서비스에 연결하다 실패한 것도 아닐 것이고, ALB의 sticky session설정도 되어 있어서 세션관련 문제로 연결이 끊기는 것도 아닐것으로 생각한다.

정상접속일때와 502 오류가 발생했을때의 ALB의 access log를 보니 아래와 같았다. 정상 접속일때는 node.js가 서비스중인 EC2로부터 0.003초 만에 304코드로 응답을 받아서 서비스를 정상완료 한 것이 보인다. 그런데 502 오류가 나는 상황에서는 EC2에서 처리한 시간이 기록되지 않고, ALB 자체에서 502 오류를 발생시켜서 반환하는 것으로 보인다. 
```text
https 2020-10-20T05:44:51.555553Z <<ALB-ARN>> <<Client-IP>> <<node.js EC2 local ip>>:80 0.000 0.003 0.000 304 304 723 188 "GET <<Request URL>> HTTP/1.1" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.75 Safari/537.36" ECDHE-RSA-AES128-GCM-SHA256 TLSv1.2 <<ARN-Target Group>> <<이하 생략..>>

https 2020-10-20T05:44:56.555810Z <<ALB-ARN>> <<Client-IP>> <<node.js EC2 local ip>>:80 0.000 0.000 -1 502 - 723 679 "GET <<Request URL>> HTTP/1.1" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.75 Safari/537.36" ECDHE-RSA-AES128-GCM-SHA256 TLSv1.2 <<ARN-Target Group>> <<이하 생략..>>
```

이 상황에서 node.js 의 access log에는 502 오류가 발생한 시점인 05:44:56에 GET요청을 정상적으로 처리한 이력이 남아 있었다. 따라서 요청을 처리하고 ALB로 응답을 보내는 과정에서 문제가 있을 것이라 생각하고 유사 사례를 찾아보았다.

결론부터 말하면  AWS ALB의 기본 Idle timeout 값이 60초 인 반면 node.js의 http keepAliveTimeout기본값이 5초라서, 5초가 넘는 TCP 연결을 node.js에서 간헐적으로 끊어버리면서 발생하는 문제였다. 이와 비슷한 문제가 [ADAM CROWDER의 블로그](https://adamcrowder.net/posts/node-express-api-and-aws-alb-502/)에 정리가 되어 있었으며 문제의 분석과 해결 방법을 도움받아 해결 할 수 있었다.


# 2. 해결 방법
앞의 블로그를 참고하면, 이 문제에는 두 가지 요소가 원인으로 작용하고 있다.
* node.js 서버의 ```keepAliveTimeout``` 설정
* ALB의 Idle timeout 설정

ALB의 Idle timeout이 만료되기 전까지 TCP연결을 살려두면 될테니, node.js의 애플리케이션 코드에 keepAliveTimeout 값을 아래와 같이 추가 해 주었다. ALB Idle timeout은 60초 라서 keepAliveTimeout과 headersTimeout을 각각 65초와 66초로 설정 해 두었다. (설정값은 milisecond 단위 )

```javascript
const app = express();
var server = http.createServer(app);

// 아래 두 줄 추가
server.keepAliveTimeout = 65000;
server.headersTimeout = 66000;
```

# 3. 참고 링크
*  [ADAM CROWDER의 블로그](https://adamcrowder.net/posts/node-express-api-and-aws-alb-502/)