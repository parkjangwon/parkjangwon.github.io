---
layout: post
title:  "NGINX 웹서버에 LetsEncrypt SSL 인증서를 적용해보자"
date:   2021-02-10 01:00:00 +0900
categories: nginx ssl letsencrypt
sitemap :
  changefreq : daily
  priority : 1.0
---

**웹서버를 구축 후 무료 SSL인증서 발급 및 HTTPS 적용을 하는 과정을 작성하겠다.**

**웹 보안의 가장 기본이 되는 것은 HTTPS 적용이다. 아직 HTTPS가 뭔지 모르는 사람은 아래 포스트를 간단히 훝어 보고 넘어가는 것을 추천한다.**

**[HTTPS는 어떻게 동작하는거지?](https://parkjangwon.github.io/https/2020/08/01/HTTPS는-어떻게-동작하는거지.html){:target="_blank"}**

**HTTPS 적용을 위한 SSL 인증서를 발급에 비용이 발생되지만, letsencrypt라는 기관에서 간단한 과정으로 무료로 SSL 인증서를 발급해준다.**

**와일드카드 인증서 혹은 멀티 도메인 인증서도 발급 가능하다! 단, 인증서의 만료기간은 3개월이며 사용자가 직접 갱신 요청을 해야한다.**

**인증서를 발급하는 과정에서는 본인 소유의 도메인(도메인의 DNS 레코드를 관리 할 수 있는 권한도 포함)과 서버가 준비되어야 한다. (웹 UI에서 작업하는 환경이 아니다.)**

---

# 작업 환경

**운영체제는 CentOS 7.9, 웹서버는 nginx를 사용할 것이다.**

**개인 서버가 없는 경우, 클라우드 컴퓨팅을 제공하는 무료 티어의 VM을 사용해보는 것을 추천한다.**

**작성자는 개인 개발 서버가 있지만, 이번 실습을 위하여 이전에 생성하였던 오라클 클라우드 서버를 활용하겠다.**

**필요한 경우 아래 포스트를 참고하여 무료로 클라우드 서버를 구축하여서 실습하여도 좋다.**

**[오라클 클라우드 서버 무료로 사용하기](https://parkjangwon.github.io/cloud/server/2020/09/26/%EC%98%A4%EB%9D%BC%ED%81%B4-%ED%81%B4%EB%9D%BC%EC%9A%B0%EB%93%9C-%EC%84%9C%EB%B2%84-%EB%AC%B4%EB%A3%8C%EB%A1%9C-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0.html){:target="_blank"}**
 
**또한, 도메인 호스팅의 경우, GoDaddy를 사용하고 있다.**

**당연한 얘기지만, 도메인을 다른 업체에서 호스팅을 받아도 작업 과정에는 전혀 달라지는것이 없다.**

**본인 소유의 도메인이 없을 시, 인증서를 발급 받을 수 없는 점 참고하길 바란다.**

---

# NGINX, Let's Encrypt 설치

**바이너리 설치 or 소스 컴파일 방식의 설치가 아닌 yum을 통한 설치를 할 것이다.**

```bash
# nginx를 설치 하기 앞서 epel repository를 설치 한다.
yum -y install epel-release

# nginx, letsencrypt, nginx plguin을 설치한다.
# nginx plguin는 필수 설치는 아니지만, 인증서 발급, 갱신 과정의 편의성을 위하여 설치한다.
yum -y install letsencrypt nginx python2-certbot-nginx
```
---

# NGINX 구동 테스트 및 간단한 커맨드

**앞서 언급하였듯이, 인증서를 발급하려면 웹서버가 필요하다.**

**먼저 nginx를 구동 후, 브라우저를 통해 접속이 되도록 작업을 하자.**

```bash
# nginx 서버 구동 전, 서버에 리슨하고 있는 포트 확인
netstat -tnlp | grep LISTEN
```
**아직 nginx를 구동하지 않았기 때문에, 80포트(http)는 아직 LISTEN 하지 않았다.**
![img](/assets/20210210/1.png)

```bash
# nginx 구동 명령어
nginx
```
**nginx를 구동하는 명령어는 'nginx'**

**참고로 1024 이하의 Well Known 포트를 바인딩하는 서비스는 반드시 root 권한으로 구동해야 한다.**

```bash
# nginx 서버 구동 후, 서버에 리슨하고 있는 포트 확인
netstat -tnlp | grep LISTEN
```
**nginx를 구동하여, 80포트(http)가 LISTEN 되었다.**
![img](/assets/20210210/2.png)

**자주 사용하는 nginx 명령어**

```bash
# nginx 구동
nginx

# nginx 중지
nginx -s stop

# nginx 설정 파일 문법 체크
nginx -t

# nginx 설정파일 반영
# 서비스를 중단하지 않고 설정파일을 반영 할 수 있다. 
# 만약 설정파일 중 문법 오류가 발생할 경우, 변경한 설정은 반영되지 않고, 원래 반영되어 있는 설정이 유지 된다.
nginx -s reload
```

---
# 80, 443 포트 접근 방화벽 정책 추가

**일반적으로 클라우드 서버를 구축하면, 서버 접속을 위한 SSH 기본 포트인 22포트를 제외한 나머지 포트에 대한 접근은 막혀있다.**

**클라우드 콘솔에서 80 포트를 허용해 줘야한다. 이번 포스팅의 주제는 https를 적용하는것이기 때문에,**

**외부에서 80(http 기본 포트), 443(https 기본 포트)포트를 접근할 수 있도록 방화벽 정책을 추가하겠다.**

**여기서는 오라클 클라우드 기준으로 설명하겠다.**

**오라클 클라우드 접속 > 좌측 세줄 버튼 > 컴퓨트 > 인스턴스 클릭**

![img](/assets/20210210/4.png)

**인스턴스 리스트 > 웹서버로 사용 중인 인스턴스 클릭**

![img](/assets/20210210/5.png)

**인스턴스 정보 > 인스턴스 세부정보 > 가상 클라우드 네트워크 클릭**

![img](/assets/20210210/6.png)

**보안 목록 > Default Security List for vcn-xxx 클릭**

![img](/assets/20210210/7.png)

**수신 규칙 > 수신규칙 추가**

**스크린샷을 참고하여 HOST는 ANY(0.0.0.0/0), 포트는 80을 입력 후 추가한다. 동일하게 443 포트도 추가.**

![img](/assets/20210210/8.png)

**서버에 SSH 접속 후, firewall-cmd(OS 방화벽)에 80,443 포트를 허용시켜준다.**

```bash
# 80 포트 접근 영구(--permanent) 허용
firewall-cmd --permanent  --add-port=80/tcp

# 443 포트 접근 영구(--permanent) 허용
firewall-cmd --permanent  --add-port=443/tcp

# 방화벽 정책 적용
firewall-cmd --reload

# 방화벽 정책 리스트 확인
firewall-cmd --permanent --list-all
```

**서버의 공인 IP로 접속해보자.**

**접속은 잘 되었다. 근데.... 보통 NGINX의 기본 html이 나와야 하는데, 작성자의 경우, Centos의 웰컴 페이지가 출력되었다.**

![img](/assets/20210210/9.png)

**이건 별로 중요한건 아니지만, 궁금해서 nginx.conf를 확인하였다. "/"으로 접속하였을 때 출력하는 설정파일이 저 페이지로 되어있었다 -_-;;**

![img](/assets/20210210/10.png)

![img](/assets/20210210/11.png)

---
# 도메인에 웹서버의 공인 IP 매칭 (A 레코드 등록)

**웹서버의 IP와 도메인을 매핑 시켜준다.**

**테스트를 할 도메인은 web.devj.in이다. 해당 도메인에 웹서버의 공인 IP를 매칭 시켜준다. (A 레코드)**

![img](/assets/20210210/3.png)

**아이피가 아닌 도메인(web.devj.in)으로 접속해보자.**

![img](/assets/20210210/12.png)

---
# Let's Encrypt의 SSL 인증서 발급 및 갱신 세팅

**단일 도메인 뿐만아니라 와일드카드(*.domain.com), 멀티도메인(abc.domain.com, zxc.domain.com, asd.domain2.net)을 지원한다.**

**여기서는 편의상 단일 도메인에 대해서만 발급하겠다..**

**인증서 이름(cert-name)을 지정하지 않는 경우, 첫번째 입력한 domainName이 인증서의 이름으로 설정 된다.**

**인증서의 이름을 입력하지 않는 경우**

**(여기서 입력하는 이메일주소로 발급하는 인증서의 만료 안내 메일이 발송된다.)**

```bash
letsencrypt --nginx certonly
```

![img](/assets/20210210/13.png)

**인증서를 삭제할 때는 아래와 같이 명령어를 입력한다.**

```bash
letsencrypt delete -cert-name web.devj.in
```

![img](/assets/20210210/14.png)

**인증서의 이름을 지정해서 다시 발급해보자.**

```bash
letsencrypt --nginx certonly --cert-name devj-chain
```

![img](/assets/20210210/15.png)

**인증서를 갱신하는 경우**

```bash
letsencrypt --nginx renew
```

**조금 더 편리한 운영을 위하여 인증서 갱신 스크립트를 crontab에 등록하자.**

```bash
crontab -e

[crontab]

# 매월 1일 01시에 갱신을 시도한다.

################### letsencrypt renew ############################
0 1 1 * * letsencrypt --nginx renew > /dev/null 2>&1
##################################################################
```

![img](/assets/20210210/16.png)

**인증서에 다른 도메인을 추가하는 경우**

```bash
letsencrypt --nginx certonly --cert-name devj-chain -d domain.com -d domain2.com
```

![img](/assets/20210210/17.png)

**인증서에 특정 도메인을 제거하는 경우**

```bash
letsencrypt --nginx certonly --cert-name devj-chain -d domain.com
```

![img](/assets/20210210/18.png)

**눈치챘겠지만, 인증서에 도메인을 추가하거나, 삭제하는 것은 그냥 업데이트 개념이라고 보면 된다.**

**따로 add domain, remove domain와 같은 명령어는 존재하지 않고,** 

**명령어의 인자 중 -d 기준으로 입력되는 domainName 리스트와,**

**인증서에 저장되어 있는 doaminName을 판단하여 letsencrypt에서 알아서 삭제, 추가를 한다.**

**인증서 발급, 갱신 시, 인증서에 들어가는 도메인이 현재 웹서버를 바라보고 있는지 DNS 룩업을 한다.**

**그 과정에서 실패를 할 경우 인증서 발급/갱신이 거부가 된다.**

**반복적으로 거부가 되는 경우, letsencrypt 측에서 일정 시간동안 발급/갱신을 막는다.**

**이러한 상황을 방지하기 위해 명령어에 --dry-run 옵션을 추가하면 실제 수행되지 않고, 정상적으로 발급/갱신 요청이 되는지 테스트를 할 수 있다.**

```bash
letsencrypt --dry-run --nginx renew --cert-name devj-chain

letsencrypt --dry-run --nginx certonly --cert-name devj-chain -d domain.com
```

---
# NGINX HTTPS 적용

**https 적용 및 약간의 nginx 설정 변경도 같이 진행하겠다.**

```bash
cd /etc/nginx/conf

# 항상 설정 파일을 변경할 때 백업을 생활화 하자!
cp nginx.conf nginx.conf_org
```

![img](/assets/20210210/19.png)

```bash
vi nginx.conf
```

![img](/assets/20210210/20.png)

**몇 가지 보안 취약 항목에 대한 조치를 하였고...**

```bash
# 디렉토리 검색 비활성화
autoindex   off;

# 서버 정보 노출되지 않도록
server_tokens off;
```

**nginx.conf 하단을 보면 아래와 같은 설정이 있다.**

```bash
include /etc/nginx/conf.d/*.conf;
```

**말 그대로 conf.d 디렉토리 하위에 있는 .conf 파일의 설정을 가져온다고 보면된다.**

**nginx.conf에 모든 설정 값을 넣기 보다, 관련 설정을 모아서 파일명으로 분류하여 관리 할 수 있다는 뜻이다.**

**작성자는 최초 nginx.conf에 있는 server 설정을 지운 후,**

**"/etc/nginx/conf.d/default.conf", "/etc/nginx/conf.d/ssl.conf"을 생성하였다.**

```bash
vi /etc/nginx/conf.d/default.conf
```

![img](/assets/20210210/21.png)

**아래는 http 접속 시 https로 자동 리다이렉트 하는 옵션이다.**

```bash
return 301 https://$server_name$request_uri;
```

```bash
vi /etc/nginx/conf.d/ssl.conf
```

![img](/assets/20210210/22.png)

**여기서 중요한 부분은 letsencrypt로 발급한 인증서의 경로를 잡아주는 부분이다.**

```bash
# SSL 인증서 설정
ssl_certificate /etc/letsencrypt/live/devj-chain/fullchain.pem;
ssl_certificate_key /etc/letsencrypt/live/devj-chain/privkey.pem;
```

참고로, 아래 설정은 nginx와 tomcat을 연동할 때 (reverse proxy 방식. 궁금하면 구글링하시길..) 사용하는 옵션이다.

```bash
[/etc/nginx/conf.d/ssl.conf]

location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        limit_except GET POST OPTIONS {
                deny all;
        }
}

[/home/tomcat/conf/server.xml]

<Connector port="8080"
            proxyPort="443"
            scheme="https"
```

**이제 다 끝났다! 변경한 nginx 설정을 반영해보자.**

```bash
# 설정 파일 문법 체크
nginx -t
# nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
# nginx: configuration file /etc/nginx/nginx.conf test is successful

# 설정 반영
nginx -s reload
```

![img](/assets/20210210/23.png)

**주소창에 본인의 웹서버를 접속할 수 있는 url을 입력해보자.**

![img](/assets/20210210/24.png)

**http://url로 접속하여도, https로 정상적으로 리다이렉트가 된다.**

**따라오느라 고생 많았다!**

---
# 마치면서....

**정적 컨텐츠만 제공하는 웹사이트의 경우, WAS를 사용하지 않고, WEB 서버로만 운영해도 무리가 없다.**

**요즘 운영되는 웹사이트의 경우 그런 경우보다, 정적 컨텐츠 뿐만 아니라 사용자의 요청에 따른 동적 컨텐츠를 제공하는 경우가 많다.**

**최근 웹 서비스 개발을 할 경우, 규모가 작으면, 따로 웹서버를 사용하지 않고 WAS(주로 Tomcat을 사용)로만 운영한다.**

**현재 대부분의 WAS는 동적 컨텐츠 뿐만 아니라, 정적 컨텐츠 제공도 가능하기 때문이다.**

**WAS 단독으로 운영해도 무방하지만, WEB 서버와 WAS 서버를 연동하여 사용하는 경우 많은 이점을 얻을 수 있다.**

# 1. 부하 분산

**설정에 따라 정적 컨텐츠는 WEB서버에서(자원 배포 뿐만아니라, web cash 기능을 포함), 동적 컨텐츠는 WAS에서 처리 하게끔 구성 가능.**

# 2. 리버스 프록시 기능을 통한 보안성 강화

**앞단에서 사용자의 요청을 받는 주체(end-point)는 WEB서버 (Apache, NGINX, IIS, ETC...)**

**실제 요청에 대하여 수행을 하는 주체(back-end)는 WAS(Tomcat, Webloginc, wildfly, ETC...)**

**이렇게 구성 후, WEB 서버 설정에서 용도에 맞게 사용자의 요청을 뒷 단에 WAS에 넘겨 줄 수 있다.**

**이러한 인프라를 구축하면 백엔드 서버를 굳이 사용자에게 노출 시킬 필요가 없다.**

**또한, WAS 다중화 구성도 용이하게 할 수 있다.**

...

**설명하지 못한 이점도 많이 있지만, 그 중 체감이 큰 건 SSL인증서 관리이다.**

**앞단에 WEB 서버를 둠으로써, SSL 인증서 세팅을 WAS 별로 다 세팅하지 않고 WEB서버에만 설정해도 적용이 된다는 점이다.** 

~~(WAS 별로 인증서 설정을 하게 되면 굉장히 귀찮다 .. -_-;;;)~~

**또.. 다른 개념이긴하지만 MSA(Micro Service Architecture)를 구성하는 경우,**

**API GATEWAY라는 프록시를 두게되면(NGINX 지원!),**

**API를 호출하는 클라이언트 입장에서도 엔드 포인트가 하나로 좁혀지기 때문에, API에 대한 관리가 편리해진다..**

**....**

**모쪼록 몇 몇 사람들에게 해당 포스트가 도움이 되었으면 좋겠다..**