---
layout: post
title: "Node.JS의 이벤트 리스너에 타임아웃 적용하기"
tags: [node, event, timeout]
comments: true
---

Node.js에서는 기본으로 event라는 패키지를 제공하며, 그 안의 ```EventEmitter```클래스를 이용해서 이벤트를 전파하고 수신 할 수 있다. 이 때, 이벤트리스너는 이벤트가 도착하기만을 무한히 대기하게 되는데, 이를 지정한 시간만큼만 기다리도록 하는 방법을 고민해 봤다.

# 이벤트의 발생과 수신
Node.js의 이벤트는 다음과 같이 발생시키고  수신할 수 있다.

이벤트의 발생은 ```emit```함수를 호출하며, 이벤트의 수신은 ```on```과 ```once```함수를 이용해 이벤트 리스너를 등록 할 수 있다. 이 때 ```on```은 이벤트가 발생할 때마다 리스너를 동작시키며, ```once```는 리스너 등록 직후 최초 발생한 1회의 이벤트에 한해 리스너를 동작시킨다.

```javascript
const EventEmitter = require('events');
const EventBus = new EventEmitter();

EventBus.on('call', (msg) => {
	console.log(`Event : ${msg}`);
});
EventBus.once('call', (msg) =>{
	console.log(`First call : ${msg}`);
});

EventBus.emit('call', 'first message');
EventBus.emit('call', 'second message');
```

# 이벤트의 대기 및 타임아웃

시간제한있는 이벤트리스너를 구현하기 위해 먼저 ```EventEmitter```를 상속하는 클래스를 하나 작성했다.

그리고 그 안에 Promise 객체를 반환하는 시간제한 기능이 구현된 이벤트 리스너 함수를 작성했다. 제한시간만큼의 타이머를 동작시키고, 그 안에 해당 이벤트가 발생하지 않으면 reject 시키는 방식이다. Promise의 reject가 호출되면 async-await로 호출한 부분에서 예외가 발생하게 되며, 이 예외를 try-catch로 잡아서 타임아웃 처리를 해주는 방식이다.

이를 조금 더 편리하게 작성하기 위해 Node.js모듈로 작성하고 ```timeout_event.js```파일에 저장했다.

```javascript
// timeout_event.js
const EventEmitter = require('events');

class TimeoutEventEmitter extends EventEmitter {
	timeoutListener (eventName, timeout) {
		return new Promise((resolve, reject) =>{
			let timer, listener;
			listener = (ev) => { 
				clearTimeout(timer);
				resolve(ev); 
			};
			timer = setTimeout(()=>{
				this.removeListener(eventName, listener);
				reject(`Timeout on ${eventName} by ${timeout/1000} seconds.`);
			}, timeout);
			this.once(eventName, listener);
		});
	}

	async onceByTimeout (eventName, timeout, callback, onTimeout){
		try{
			let result = await this.timeoutListener(eventName, timeout);
			if(callback) callback(result);
		} catch(e) {
			if(onTimeout) onTimeout(e);
		}
	}
}

module.exports = TimeoutEventEmitter;
```

# 테스트 동작

앞서 작성한 파일을 다음 코드를 통해 실행해 보았다.

* 1.5초를 대기하는 이벤트 리스너를 두개 작성한다. 
* 1초와 2초가 지난 시점에 각각의 이벤트를 발생시킨다.
* 3초와 4초가 지난 시점에 이 이벤트를 다시 발생시켜본다.

이 경우 1초에 발생하는 이벤트는 정상콜백이, 2초에 발생한이벤트는 타임아웃 콜백이 발생해야 한다. 그리고 3초와 4초에 발생한 이벤트에는 리스너의 동작이 없을 것이다.

```javascript
const EventEmitter = require('./timeout_event.js');

let eventbus = new EventEmitter();

eventbus.onceByTimeout(
	'call_1', // event name
	1500,     // waiting for response
	res => {console.log(res);}, // if response received
	err => {console.log(`Error : ${err}`);} // if timeout occured
);

eventbus.onceByTimeout(
	'call_2', // event name
	1500,     // waiting for response
	res => {console.log(res);}, // if response received
	err => {console.log(`Error : ${err}`);} // if timeout occured
);

setTimeout(()=>{eventbus.emit('call_1', 'Hello There ~ 1')}, 1000);
setTimeout(()=>{eventbus.emit('call_2', 'Hello There ~ 2')}, 2000);
setTimeout(()=>{eventbus.emit('call_1', 'Hello There ~ 3')}, 3000);
setTimeout(()=>{eventbus.emit('call_2', 'Hello There ~ 4')}, 4000);
```

실행결과는 다음과 같이 call_1 은 한번 정상 호출되고, call_2는 타임아웃이 발생했다.
그리고, 정상호출이든 타임아웃이든 한번 완료된 이벤트는 다시 발생해도 리스너가 호출되지 않는다.

```
Hello There ~ 1
Error : Timeout on call_2 by 1.5 seconds.
```

# 여담

이 이벤트 모듈은 Node.js로 작성하는 데이터 연계시스템에 활용할 수 있을 것 같다. 하나의 시스템에서 들어온 요청을 또 다른 여러 시스템의 호출결과를 종합하여 응답을 줘야 할 때, 응답이 반드시 이루어 져야 한다고 할 경우 응답에 시간제한을 둘 수 밖에 없을 것이다. 이 때 다른 여러시스템의 호출결과를 쥐합하는 동작에 시간제한이 있는 이벤트를 활용해서 시간내 완료가 안된경우 실패응답을 보내는 방향으로 모듈을 작성해 볼 수 있을 것 같다.

그래서 언젠가 써먹을까 싶어 npm에도 배포 해 둔다.

* https://www.npmjs.com/package/timeout_eventemitter