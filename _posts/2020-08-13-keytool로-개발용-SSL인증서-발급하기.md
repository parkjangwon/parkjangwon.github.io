---
layout: post
title:  "keytool로 개발용 SSL인증서(Java KeyStore) 발급하기"
date:   2020-08-13 22:27:00 +0900
categories: ssl
sitemap :
  changefreq : daily
  priority : 1.0
---

Java 웹 어플리케이션을 개발 시, HTTPS를 적용을 해야할 때 SSL인증서를 발급해야한다.

기본적으로 SSL인증서를 발급할 때, 비용이 들게 된다. 

물론 Let's Encrypt에서 발급할 땐 무료이다. 하지만 Let's Encrypt에서 인증서를 발급하려면 특정 도메인의 80포트로 접근 가능한 웹 서버가 필요하다.

(Let's Encrypt에서는 pem 형식으로 발급 되기 때문에, Java기반의 미들웨어에서 사용하려면 Java KeyStore 형식으로 변환 작업이 필요하다.)

상황이 여의치 않은 경우, CA의 인증을 받지 않은 말 그대로 개발용 인증서(Java KeyStore)를 JDK에서 제공하는 keytool을 통하여 발급 받는 방법이 있다.

# 1. JDK 설치 경로 이동 및 keytool 유무 확인

```bash
[root@parkjw ssl]# echo $JAVA_HOME
/usr/local/java
[root@parkjw ssl]# cd /usr/local/java/bin
[root@parkjw bin]# ls -l | grep keytool
-rwxr-xr-x 1 root root   8832 Mar 12 15:33 keytool
```

> \# java 환경 변수가 등록 되어있지 않는 경우, keytool을 사용할 때 jdk의 절대 경로를 입력해야하기 때문에 왠만하면 환경변수에 등록하자. 

# 2. keytool 커맨드로 인증서(keyStore) 발급

```bash
[root@parkjw ssl]# keytool -genkey -alias KeystoreAlias -keysize 2048 -keyalg RSA -keystore keystore.jks -dname "CN=mydomain.com, OU=myteam, O=mycompany, L=seoul, ST=seoul, C=kr"
```

> \# 어차피 개발용이니까 적당한 값으로 설정하자.


```bash
Enter keystore password:
Re-enter new password:
Enter key password for <KeystoreAlias>
	(RETURN if same as keystore password):
```

> \# keystore의 비밀번호를 입력 후 엔터를 누른다, 그리고 비밀번호 확인. 알리아스의 비밀번호를 따로 설정해줄 것이 아니면 그냥 엔터를 입력.

```bash
Warning:
The JKS keystore uses a proprietary format. It is recommended to migrate to PKCS12 which is an industry standard format using "keytool -importkeystore -srckeystore keystore.jks -destkeystore keystore.jks -deststoretype pkcs12".
[root@parkjw ssl]# ls -trl
total 4
-rw-r--r-- 1 root root 2234 Aug 13 22:24 keystore.jks
[root@parkjw ssl]# file keystore.jks
keystore.jks: Java KeyStore
```

> \# file 명령어로 발급된 인증서를 확인해보면 Java KeyStore라고 나온다. 

이렇게 발급한 해당 인증서를 Tomcat과 같은 Java Servlet Container의 관련 설정 파일에 설정해주면 된다.

개발용으로 발급된 인증서이긴 하지만 Was에서 SSL 관련 keystore 설정 할 때, 정상 인증서와 설정 값의 차이는 없다.

개발용 인증서이기 때문에, 범용 브라우저에서는 보안 인증서 관련 경고 화면이 나온다. 브라우저 내에서 예외 옵션을 넣어주고,

HTTPS 통신 시, 패킷이 정상적으로 암호화가 되는지 확인을 하면 된다.

\* Java Keystore 파일이기 때문에 Java Servlet Container가 아닌 다른 미들웨어, 예를 들면 아파치 웹서버
에서는 사용할 수 없다!