---
layout: post
title: "Win10에서 Nancy로 SelfHosting시 접속오류"
tags: [c#, nancy, netsh]
comments: true
---

C#으로 윈도우 프로그램을 만들때, 프로그램의 설정값을 원격지의 웹브라우저를 이용해서 변경하고자 하는 기능이 필요해 Nancy라는 .Net기반 임베디드 웹서비스 라이브러리를 도입했다. REST웹서비스를 처리 모듈을 개발하고 테스트 하는 도중 Win7환경에서는 잘 동작하던 것이 Win10환경에서는 ```AutomaticUrlReservationCreationFailureException```을 발생하며 접속이 되지 않는 문제가 있었다. 

# 문제의 해결

처음에는 OS의 방화벽 설정 문제가 아닐까 의심도 했었다. 문제의 원인은 Win10의 UrlAlc 관련 설정문제였으며, 서비스할 URL을 ```netsh``` 명령을 이용해서 URL에 등록해 주는 방법으로 해결했다.

```bash
netsh http add urlacl url=http://+:1234/ user=DOMAIN\username
```

이 때 설정 옵션은 다음과 같다.

* ```1234``` : 서비스할 포트 번호
* ```DOMAIN\username``` : 웹서비스 또는 응용프로그램을 실행할 윈도우 계정 및 도메인

# 참고 링크

* https://stackoverflow.com/questions/18368821/how-to-fix-a-automaticurlreservationcreationfailureexception-when-using-nancy-fx
* https://docs.microsoft.com/ko-kr/windows-server/networking/technologies/netsh/netsh-http#add-urlacl
* https://github.com/NancyFx/Nancy/wiki/Self-Hosting-Nancy
