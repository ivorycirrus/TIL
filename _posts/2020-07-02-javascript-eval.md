---
layout: post
title: "javascript의 전역 객체, eval과 간접호출"
tags: [javascript, eval, execution scope, comma]
comments: true
---

자바스크립트의 전역객체 선언에 대해 알아보던 중 재미있는 코드를 봤다. this가 굉장히 어색한 방식으로 사용되고 있었으며, 왠만하면 가급적 사용하지 말아야할 명령어라고 말하는 eval 까지 섞여 있었다. 그 코드는 다음과 같다.

```javascript
var global = (function () {
    return this || (1, eval)('this');
}());
```

# 뭐하는 녀석인고 하니..
클로저 함수를 하나 선언해서, 해당 함수가 실행할 때의 this 객체를 반환하는 코드이다. 그런데, this를 반환하는 코드 뒤에 수상하게 eval이 붙어있다. 이 때 return 안에 있는 코드가 어떻게 동작하는지 뜯어보자.

### ```return this```의 의미
보통은 이런방식으로 함수 안에서 this를 반환 할 경우 전역컨텍스트의 this를 가리킨다. 보통 웹 브라우저의 전역에서 실행하는 함수인 경우,  window 객체를 반환하는 것이 보통이다.

그런데 ```strict mode```에서는 클로저 함수 안에서 이 this 값이 ```undefined```가 된다. 따라서, window객체를 반환 할 수 있는 상황이면 전역객체로 window객체를 반환하고, 그렇지 않은 경우 뒤의 코드를 실행하게 된다.

### ```(1, eval)('this')``` 의 의미
eval을 직접 호출할 경우, 해당 클로저 안의 this를 반환하게 된다. ```(1, eval)('this')```는 eval을 간접 호출해서 사용하는 방식이며, 이는 eval이 전역에서 실행될 수 있게하여 전역 컨텍스트의 this를 찾아 올 수 있게 한다.

### 종합해 보면..
strict mode가 선언되어 있건 아니건, 항상 global변수는 전역 객체를 참조 할 수 있게 되는 트릭이다. 콤마 연산자(,)를 이용해서 eval을 간접 호출로 실행 시키고, 그 때의 this객체를 잡아오는 것이 구현의 묘 라 생각하다.


# References
* [MDN document :: eval() function](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/eval)
* [stackoverflow :: (1, eval)('this') vs eval('this') in JavaScript?](https://stackoverflow.com/questions/9107240/1-evalthis-vs-evalthis-in-javascript)
* [blog :: Global eval. What are the options?](http://perfectionkills.com/global-eval-what-are-the-options/#evaling_in_global_scope)
* [blog :: The JavaScript Comma Operator](https://javascriptweblog.wordpress.com/2011/04/04/the-javascript-comma-operator/)