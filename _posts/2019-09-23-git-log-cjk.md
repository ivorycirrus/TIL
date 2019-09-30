---
layout: post
title: "Git Log의 한글 표시 문제"
tags: [git, encoding]
comments: true
---

Windows의 콘솔에서 ```git log``` 명령으로 과거 커밋이력을 조회 할 때, 커밋메시지에 한글이 표시되어 있는 경우 메세지가 깨져서 나오는 문제가 있다. 이는 로케일 설정을 통해 해결 할 수 있다.

# 사건의 시작

Windows에서 ```cmd```명령으로 콘솔 실행 후 한글이 포함된 git log를 출력하면 다음과 같이 16진수 바이트로 표시되는 현상이 있다.

```
c:\Data\Temp\study-excersize-codes>git log -n 3
commit 7f774eef1e010419455e2dbfbc7cfdb4f109b382 (HEAD -> master, origin/master, origin/HEAD)
Merge: ef61566 938f9a3
Author: ivorycirrus <ivorycirrus@gmail.com>
Date:   Thu Aug 3 19:48:09 2017 +0900

Merge branch 'master' of https://github.com/ivorycirrus/study

commit ef615662398b330ff766de420d22f7704ba7ef36
Author: ivorycirrus <ivorycirrus@gmail.com>
Date:   Thu Aug 3 19:47:33 2017 +0900

4<EC><9E><A5> <EC><8B><A0><EA><B2><BD><EB><A7><9D><ED><95><99><EC><8A><B5> <EB><81><9D><EA><B9><8C><EC><A7><80> <EC><A0><95><EB><A6><AC>

commit 938f9a3b92262beef9b0de2b664d95a31bdf1019
Author: Pilmo Kang <ivorycirrus@gmail.com>
Date:   Thu Aug 3 15:15:57 2017 +0900

Update README.md                                                                                                                           
c:\Data\Temp\study-excersize-codes>     
```


# 문제의 해결

이는 기본 로케일 설정이 되어 있지 않아 발생한 문제이다.

다음과 같이 로케일을 설정한 다음 다시 로그를 조회해 보면 한글이 정상적으로 보이는 것을 볼 수 있다.

```bash
c:\> set LC_ALL=ko_KR.UTF-8
```

```
c:\Data\Temp\study-excersize-codes>set LC_ALL=ko_KR.UTF-8

c:\Data\Temp\study-excersize-codes>git log -n 3
commit 7f774eef1e010419455e2dbfbc7cfdb4f109b382 (HEAD -> master, origin/master, origin/HEAD)
Merge: ef61566 938f9a3
Author: ivorycirrus <ivorycirrus@gmail.com>
Date:   Thu Aug 3 19:48:09 2017 +0900

Merge branch 'master' of https://github.com/ivorycirrus/study 

commit ef615662398b330ff766de420d22f7704ba7ef36
Author: ivorycirrus <ivorycirrus@gmail.com>
Date:   Thu Aug 3 19:47:33 2017 +0900

4장 신경망학습 끝까지 정리

commit 938f9a3b92262beef9b0de2b664d95a31bdf1019
Author: Pilmo Kang <ivorycirrus@gmail.com>
Date:   Thu Aug 3 15:15:57 2017 +0900

Update README.md

c:\Data\Temp\study-excersize-codes>
```

기본 코드페이지 설정을 콘솔 실행시마다 하기 번거로운 경우 시스템의 환경변수에 등록 해 두고 사용하면 될 것이다.


# 그럼 다른 것은?

물론 코드페이지 설정을 해보기는 했다.
Windows의 한글 인코딩은 CP949 기반으므로 콘솔에서도 코드페이지가 949로 설정되어 있다.

이것을 UTF-8 코드페이지은 65001로 바꿔 보았다

```
c:> chcp 65001
```

하지만.. 한글이 깨져나오는 것은 여전하더라..


# 참고 링크

* https://orashelter.tistory.com/60
* https://docs.oracle.com/cd/E26925_01/html/E27145/glmbx.html



