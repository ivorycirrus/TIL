---
layout: post
title: "Git의 줄바꿈 문자 처리"
tags: [git, line ending]
comments: true
---

MacOS, Linux 와 Windows 환경 모두에서 개발시 줄바꿈 문자 때문에 영향이 없는 코드 까지 dirty check에 걸리는 경우가 있다. 이런 경우 commit diff를 보는데 정작 변경점이 있는 부분이 눈에 확 띄지 않아 불편함이 이만저만이 아니다. 대부분의 IDE는 Line Ending 처리를 설정 할 수 있으므로 프로젝트 환경에 맞게 통일해서 쓸 수 있지만, git 설정으로도 이이 대해 대응 할 수 있는 방법이 있어 정리 해 둔다.

# Line Ending

MaxOS 와 Linux 는 줄바꿈 문자로 ```LF``` 만을 사용하는 반면 Windows 에서는 ```CR``` 과 ```LF``` 를 모두 사용하고 있다. 대부분의 IDE는 CR과 LF를 모두 인지하므로 하나만 사용해도 개발에 지장이 없는 경우가 대부분이다. 

따라서 git commit 시점에 코드에 있는 줄바꿈 문자를 자동으로 LF만 사용하도록 변경해 주면 commit diff 에 줄바꿈 문자가 영향을 주는 일이 사라질 것이다.

# Git 설정방법

먼저 자동으로 줄바꿈 문자를 처리하도록 전역설정을 바꾸자

```bash
git config --global core.autocrlf true
```

그리고 줄바꿈 문자로 ```LF```만 사용하도록 설정하자

```bash
$  git config --global core.eol lf
```

# 또 다른 방법이 있다면..?

폴더와 파일 단위로 ```.gitattributes```를 설정 할 수도 있다.

이 때, 소스코드에 해당하는 파일 확장자만을 골라서 줄바꿈 처리를 지정하면 보다 세세하게 조정 할 수도 있을 것이다.

# 참고 링크

* https://www.lesstif.com/pages/viewpage.action?pageId=20776404
* https://git-scm.com/book/ko/v1/Git%EB%A7%9E%EC%B6%A4-Git-%EC%84%A4%EC%A0%95%ED%95%98%EA%B8%B0
* https://git-scm.com/book/ko/v1/Git%EB%A7%9E%EC%B6%A4-Git-Attribute
