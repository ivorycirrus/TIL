---
layout: post
title: "node.js 프로젝트에 jshint적용하기"
tags: [node, jshint]
comments: true
---

node.js 프로젝트에 javascript 코드 검사도구로 [JSHint](https://jshint.com/about/)를 적용 할 때 경고메세지에 따른 환경설정 조정방법에 대해 적어본다.

# 문제의 시작
배포전에 javascript 코드를 깔끔하게 작성해 보자는 생각에 visual studio code에 jshint 플러그인을 추가했다. 그런데 예상치 못한 곳에서 코드에 경고를 출력하는 부분이 있었다.

주로 문제가 되었던 부분은 다음과 같다.

* js파일의 첫 부분에 선언한 ```'use strict'```의 가장 맨 앞 따옴표에 lint 경고 발생
* ```process```, ```require` 등 node.js의 전역 객체및 함수에 lint 경고 발생
* ```const```키워드로 상수 선언시 lint 경고 발생
* ```async```, ```await``` 키워드 사용시 lint 경고 발생

# 문제의 해결

이 경고문구는 jshint가 node.js환경을 인식하기 못하고, javascript 버전에 대한 제한을 명시하지 않아서 발생하는 문제이다.

먼저, 네개의 문제중 첫 두개는 node.js환경임을 명시 해서 해결 할 수 있다. 그리고 세번째 문제는 javascript 버전이 ES5 이상이라고 선언해 주면 되며, 마지막 문제는 javascript 버전이 ES8 이상이라고 명시 해 주면해결 할 수 있다.

그래서 이를 설정 해 주고자 프로젝트의 루트에 ```.jshintrc``` 파일을 생성하고 다음 내용을 기입 후 개발툴을 다시 시작하면 경고가 사라진 것을 볼 수 있다.

```javascript
{
    "node": true,
    "esversion": 8
}
```

# 참고 링크

* https://jshint.com/about/
* https://blog.outsider.ne.kr/1007
