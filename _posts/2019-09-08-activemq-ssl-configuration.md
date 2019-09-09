---
layout: post
title: "ActiveMQ SSL 설정방법"
tags: [activemq, ssl]
comments: true
---

ActiveMQ는 큐나 토픽 방식 모두를 지원하며 다양한 프로토콜을 통해 이용할 수 있는 오픈소스 솔루션이다. 이 때 메세지 전달과정에 보안을 고려하고자 할 경우 SSL을 이용한 암호화를 설정 할 수 있을 것이다. SSL 설정은 Http 프로토콜 뿐 아니라 MQTT와 같은 경량 메세지 프로토콜에도 적용이 가능했다.

# 준비하기

버전이 꼭 맞을 필요는 없지만, 그래도 대략적인 준비 항목을 적어본다.

* ActiveMQ 5.15.x 버전 이상
* Java(JDK) 1.8 이상
* OpenSSL 1.1.x 이상

# SSL 인증서 생성

메세지를 암호화할 인증서 파일을 생성한다. 여기에서는 테스트를 위해 사설인증서 생성방법을 적어두지만, 이미 공인인증서가 준비되어 있다면 이 단꼐는 건너뛰어도 된다.

Java가 설치된 경로의 bin 폴더에 보면 ```keytool```파일이 있다. 이 경로에서 콘솔에 다음 명령을 실행한다.

```bash
## Create a keystore for the broker SERVER
$ keytool -genkey -alias amq-server -keyalg RSA -keysize 2048 -validity 90 -keystore amq-server.ks

## Export the broker SERVER certificate from the keystore
$ keytool -export -alias amq-server -keystore amq-server.ks -file amq-server_cert

## Create the CLIENT keystore
$ keytool -genkey -alias amq-client -keyalg RSA -keysize 2048 -validity 90 -keystore amq-client.ks

## Import the previous exported broker's certificate into a CLIENT truststore
$ keytool -import -alias amq-server -keystore amq-client.ts -file amq-server_cert

## If you want to make trusted also the client, you must export the client's certificate from the keystore
$ keytool -export -alias amq-client -keystore amq-client.ks -file amq-client_cert

## Import the client's exported certificate into a broker SERVER truststore
$ keytool -import -alias amq-client -keystore amq-server.ts -file amq-client_cert
```

* ```-validity 90``` : 인증서의 유효기간이다. 오늘기준 90일 유효함을 의미하며 9999 와 같이 큰 값으로 주면 넉넉히 사용할 수 있다.
* ```amq-server.ks``` , ```amq-server.ts``` : ActiveMQ에서 SSL인증에 사용할 인증서이다.

# 클라이언트 인증서 설정 (옵션)

클라이언트에서 루트인증서가 아닌 서버용인증서와 쌍으로 만든 인증서를 이용하고자 할 경우 다음과 같이 추가로 설정할 수 있다.

아래와 같이 인증서를 이용한 서비스 포트 등을 지정해 줄 경우, 해당 인증서는 지정된 포트로만 서비스가 가능하다. 61617 포트 이외의 임의의 포트로 서비스시 연결에 문제가 있는 경우 아래 옵션을 설정했는지 확인 해 보자.

```bash
## Other useful commands
$ openssl s_client -connect activemq-host:61617
$ keytool -list -keystore amq-server.ks
$ keytool -printcert -v -file amq-server_cert
$ keytool -storepasswd -keystore amq-server.ks
```

# activemq.xml 파일 수정

먼저 앞에서 생성한  ```amq-server.ks``` , ```amq-server.ts``` 파일을 ActiveMQ가 설치된 경로 아래의 conf폴더 안에 복사한다.

그리고 ```activemq.xml``` 파일에 다음과 같이 설정을 추가한다.

```bash
<broker xmlns="http://activemq.apache.org/schema/core" brokerName="fnmamq01" dataDirectory="/mnt/amq_data/amq-5.9">
 <!-- blablablabla -->

 <transportConnectors>
     <transportConnector name="ssl" uri="ssl://0.0.0.0:61617?trace=true&needClientAuth=true" />
 </transportConnectors>

  <!-- SSL Configuration Context -->
  <sslContext>
     <sslContext keyStore="file:${activemq.base}/conf/amq-server.ks"
                 keyStorePassword="123456"
                 trustStore="file:${activemq.base}/conf/amq-server.ts"
                 trustStorePassword="123456" />
  </sslContext>

</broker>
```

# 참고 링크

* http://www.giuseppeurso.eu/en/activemq-and-the-ssl-transport/
* https://stackoverflow.com/questions/55575078/mqttnet-client-cant-connect-server-certificate
* http://activemq.apache.org/how-do-i-use-ssl
* http://activemq.apache.org/mqtt.html





