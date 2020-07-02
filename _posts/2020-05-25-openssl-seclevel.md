---
layout: post
title: "Ubuntu 20.04 openssl의 key길이 문제(SECLEVEL)"
tags: [linux, openssl, https, node, security]
comments: true
---

ubuntu 20.04에는 openssl 1.1.1f가 기본으로 탑재되어 있으며, 암호문자에 대한 보안레벨의 기본값이 2로 설정되어 있다. 보안레벨 2 에서는 암호화 키의 길이가 최소 2048bit이상의 길이를 필요로 하는데, 이보다 짧은 경우 오류가 발생하게 된다.

# 사건의 발단
기존에 ubuntu 16.04에서 node.js를 이용해서 https로 서비스하던 코드를 새로설치한 ubuntu 20.04 개발서버 환경에서 기동했더니 다음과 같은 오류가 발생했다.
```
Error: error:140AB18F:SSL routines:SSL_CTX_use_certificate:ee key too small
```
이는 기존에 https를 서비스 할 때 사용하던 암호키의 길이가 1024bit이었는데, 보안레벨의 기준미달로 openssl에서 서비스를 거부해 발생한 문제이다.

# 보안레벨을 낮추자
개발서버를 운영서버와 동일하게 보안레벨을 1로 낮추면 문제가 해결된다.
보안레벨을 낮춘다고 openssl의 인증방식이 달라진다거나 하는 것은 아니고, 키 길이에 대한 경고만 해제하는 목적으로 변경했다.

```/etc/ssl/openssl.cnf``` 파일을 열어서 다음과 같이 SECLEVEL을 1로 설정 해 주자.
```
[ default_conf ]

ssl_conf = ssl_sect

[ssl_sect]

system_default = ssl_default_sect

[ssl_default_sect]
MinProtocol = TLSv1.2
CipherString = DEFAULT:@SECLEVEL=1
```

# 보안레벨에 따른 차이
보안레벨을 맞 낮춰도 되는걸까 궁금했다. 보안레벨별로 뭐가 다르기에..

찾아보니 보안레벨은 0~5까지 있는데 레벨별로 차이를 요약하면 다음과 같다.
* 레벨 0 : 모든것 허용. 이전버전 openssl과 호완성 유지
* 레벨 1 : 최소80비트 보안. RSA/DSA/DH키는 1024비트 미만은 허용안함. SSLv2 , MD5 허용안함.
* 레벨 2 : 최소 112비트 보안. RSA/DSA/DH키는 2048비트 미만은 허용안함. SSLv3 허용안함.
* 레벨 3 : 최소 128비트 보안. RSA/DSA/DH키는 3072비트 미만은 허용안함. TLS1.1, SessionTecket 허용안함.
* 레벨 4 : 최소 192비트 보안. RSA/DSA/DH키는 7680비트 미만은 허용안함. TLS1.2, SHA1 허용안함.
* 레벨 5 : 최소 256비트 보안. RSA/DSA/DH키는 15360비트 미만은 허용안함.

인증서의 키 길이 및 프로토콜 제한까지 있다. TLS1.1 이하 프로토콜은 어차피 2020년 상반기 이후 대부분 웹브라우저에서 지원중단되는 상황이니 보안레벨 1과2의 차이는 크게 의미가 없을 듯 하니, 개발서버의 1024bit 보안키를 사용하고자 하는 이유로 레벨1을 사용하는 것은 무리가 없어 보인다.

# 1024bit 키를 쓰면 문제가 되나?
SSL은 일반적으로 디퍼-헬만 키 교환방식을 사용하고 있다. 디퍼 헬만 키 교환방식은 충분히 큰 소수를 이용한 암호화 방식인데, 이 때 충분히 큰 숫자로 1024bit의 수를 사용한다고 보는것이다. 그런데 Logjam이라는 SSL취약점 소수 추정방식을 통해 키 길이를 512bit 수준으로 추정할 수 있으며, 512bit의 키는 충분한 수의 CPU 가 주어지면 사용한 소수를 해독 할 수 있는 수준이라고 한다. 

이런 문제는 openssl의 버전업그레이드로 해결할 수도 있지만, 타원곡선을 이용한 디퍼헬만 키교환방식(ECDHE)를 쓴다던지, 디퍼헬만의 키 값을 더 큰값(2048bit이상)으로 생성해서 사용한다던지 하는 방법으로 해결할 수 있을 것이다. 그래서 보안레벨이 올라갈 수록 더 긴 길이의 키를 요구한다고 볼 수 있겟다.

# 그래서...
일단 우선은 보안레벨을 낮춰서 테스트를 하고 있지만, 앞으로 SSL키 생성시 2048bit 이상으로 생성해서 보안레벨 2에서도 동작 할 수 있도록 구성하는 편이 좋지 않을까 한다. 

# Reference
* [Ubuntu 20.04 - how to set lower SSL security level?](https://askubuntu.com/questions/1233186/ubuntu-20-04-how-to-set-lower-ssl-security-level)
* [SSL_CTX_set_security_level](https://www.openssl.org/docs/man1.1.1/man3/SSL_CTX_set_security_level.html)
* [디피 헬만 키 (Diffie-Hellman Key) 를 2048 bit 로 바꿔야 하는 이유](https://rsec.kr/?p=242)
* [Weak Diffie-Hellman and the Logjam Attack](https://weakdh.org/)



