---
layout: post
title: "AWS health check"
tags: [aws, target group, health check]
comments: true
---

AWS의 ALB를 설정하면서 대상그룹(Target group)에 node.js로 웹서비스를 하는 EC2를 연결했다. 그런데 서비스가 정상 적으로 동작하고 있음에도 대상 그룹의 상태가 unhealthy로 표시되는 문제가 있었다. 

# 1. 문제의 원인
이 서비스는 Root URL로 접근하는 경우 세션의 유무를 가지고, 유효한 세션이 살아있으면 메인페이지로 아닌경우 로그인페이지로 이동하도록 작성되어 있었다. 

AWS 대상그룹의 기본 상태체크 방식은 루트 URL에 HTTP메소드로 요청해서 응답이 정상적으로 도착하는지를 확인한다. 이 때 확인 하는 값은 다음과 같다.
* HTTP Header의 Content-Type이 ```text/plain``` 인지
* HTTP 응답코드가 200 인지

그런데 루트URL접속시 다른페이지로 이동하는 경우 HTTP 상태코드가 302반환하게 되서 상태체크 조건을 만족하지 못하는 문제가 있다.

# 2. 해결 방법
이 문제를 해결하는 방법은 두 가지가 있다.
* 대상그룹의 상태체크 설정의 Success code 값에 302 를 추가 해 준다.
* 상태체크를 위한 전용 API를 작성하고, 대상 그룹의 Path값에 추가 해 준다.

두 가지 방법 중 첫번째 방법은 메인페이지가 바뀌는 경우 유연하게 대응 할 수 없을 것 같다. 그리고 access log를 남기는 경우에도, 정상젃으로 메인페이지에 접속하는 경우와 상태체크용 heartbeat가 섞이게 되어 로그 분석시 불편한 상황이 예상된다. 

따라서 아래와 같이 ```/healthCheck``` 라는 GET 요청에 대해 응답하는 기능을 node.js 애플리케이션에 추가하고 대상그룹을 등록하는 것으로 해결했다.

```javascript
app.get('/healthCheck', function(req, res)
{
    res.writeHead(200, { "Content-Type": "text/html" });
    res.write("Health Check Page");
    res.end();
});
```