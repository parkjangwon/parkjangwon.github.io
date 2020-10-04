---
layout: post
title:  "원격 개발 환경 구축하기(Spring Boot CLI)"
date:   2020-10-01 18:00:00 +0900
categories: springboot java linux cloud
sitemap :
  changefreq : daily
  priority : 1.0
---

가끔 "아이패드로 코딩할 수 있나요?" 라는 질문을 받아 본 적이 있다.

**코딩 자체는 AppStore에서 업로드 된 IDE(단순 에디터라도 코딩 자체는 가능은 하다.) 앱을 사용해서 할 수 있다.**

Java의 경우 단순한 Java 컴파일러 앱은 있지만, JAVA EE를 온전하게 개발 할 수 있는 IDE(ex: Eclipse, Intelli J)앱은 찾지 못하였다.

**그래서 잠깐 "아이패드로 Spring Framework를 온전하게 개발할 수 있을까?"라는 주제로 구글링을 하였다.**

몇 시간동안 구글링을 한 결과, IDE 앱을 찾기 보단, **원격지 리눅스 서버에 Spring Boot CLI를 설치하고,**

**SSH Client 앱을 통하여 접속 후 CLI 기반의 Text 에디터인 "vim"으로 개발을 하면 가능하겠다라고 생각했다.**

여기서 문제가 되는 부분은, "vim"의 경우 숙련된 사람이라면 그나마 수월하겠지만, 그렇지 않은 사람이 더 많다는 것이다.

**"Spacevim"**이라는 에디터를 설치를 하여, 보다 편하게 개발을 할 수 있도록 환경을 구성할 것이다.

**["Spacevim"으로 작업 중인 Spring Boot 어플리케이션 : 아이패드에서 캡쳐를 했다.]**

![img](/assets/20201001/0.jpeg)

**캡쳐에서 볼 수 있다시피, 이번에는 원격 리눅스 서버에 Spring Boot 개발환경을 구성한 후**

**"Spacevim" 에디터를 통하여 터미널에서 직접 개발을 하는 환경을 세팅 할 것이다.**

**이렇게 구성하면 스마트폰이나 태블릿 PC, 데스크탑 상관없이 모두 SSH 연결을 통하여 터미널기반에서 개발 할 수 있다.**

**즉, 아이패드로 Spring Boot Framework 기반의 어플리케이션을 개발 할 수 있다!**

**\* 원격 리눅스 서버와 로컬 PC에서 VS Code를 연동하는 부분은 지난 [포스트](https://parkjangwon.github.io/golang/linux/cloud/2020/09/28/%EC%9B%90%EA%B2%A9-%EA%B0%9C%EB%B0%9C-%ED%99%98%EA%B2%BD-%EA%B5%AC%EC%B6%95%ED%95%98%EA%B8%B0-Golang.html){:target="_blank"}를 참고하면 동일하게 세팅할 수 있다.**

---

# 설치 환경 및 순서

**\* 설치 환경은 Linux (CentOS7) & open JDK 8**

**\* 본인은 이전 [포스트](https://parkjangwon.github.io/cloud/server/2020/09/26/%EC%98%A4%EB%9D%BC%ED%81%B4-%ED%81%B4%EB%9D%BC%EC%9A%B0%EB%93%9C-%EC%84%9C%EB%B2%84-%EB%AC%B4%EB%A3%8C%EB%A1%9C-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0.html){:target="_blank"}에서 구축한 Oracle Cloud 서버를 사용할 것이다.**


1. JDK 설치 및 환경 변수 설정
2. Maven 설치 및 환경 변수 설정
3. Spring Boot CLI 설치 및 환경 변수 설정
4. 테스트 코드 작성 및 빌드 테스트
5. Spacevim 설치

---

# JDK 설치

![img](/assets/20201001/1.png)

**...**

**....**

**.....**

![img](/assets/20201001/2.png)

**패키지 매니저(yum)을 통하여 openJDK 8을 설치한다.**

```bash
sudo yum -y install java-1.8.0-openjdk java-1.8.0-openjdk-devel
```

![img](/assets/20201001/3.png)

**"java -version" 명령어로 설치가 되었는지 확인한다.** 

```bash
java -version
```

#  Java 환경 변수 설정

![img](/assets/20201001/4.png)

**패키지 매니저(yum)을 통하여 JDK를 설치 할 경우 위 경로에 설치가 된다. 이 경로를 JAVA_HOME으로 설정 할 것이다.**

![img](/assets/20201001/5.png)

```bash
vi ~/.bash_profile
```

![img](/assets/20201001/6.png)

**각자 환경에 따라 .bash_profile의 내용은 다를 수 있다.**

**여기서 중요한 부분은 JAVA_HOME, CLASSPATH이다.**

```bash
# 아래 내용 추가

JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.262.b10-0.el7_8.x86_64
CLASSPATH=$JAVA_HOME/jre/lib:$JAVA_HOME/lib/tools.jar
PATH=기존에 설정된 값:$JAVA_HOME/bin

export JAVA_HOME CLASSPATH PATH
```

![img](/assets/20201001/7.png)

**변경한 환경변수 값을 적용하기 위하여 source 명령어를 사용한다.**

**\* 그냥 ssh를 재접속하면 반영되긴 한다.**

```bash
source ~/.bash_profile # 변경한 환경변수 값 적용
echo $JAVA_HOME
echo $CLASSPATH
java -version
```

---

# Maven 설치

Spring Boot 어플리케이션을 빌드하기 위하여 Maven을 설치한다.

\* 만약 빌드 툴을 Maven이 아닌 Gradle을 사용할 것이라면 구글링을 통하여 설치를 한다. (참고로 설치 과정은 굉장히 심플하다.)

![img](/assets/20201001/8.png)

**웹 다운로드 링크를 통하여 바로 서버에서 다운받기 위하여 "wget" 패키지를 설치한다. (본인은 사전에 설치를 하였다.)**

**\* 디렉토리 관리를 위하여 /home/사용자/tools 디렉토리를 만들었다.**

**그 후, 아파치 홈페이지에서 제공하는 공식 다운로드 링크를 복사 한 다음 "wget 다운로드 링크"로 다운로드를 받는다.**

```bash
sudo yum -y install wget # 이미 설치했을 경우 Skip
mkdir tools # 하기 싫으면 Skip
cd tools # 하기 싫으면 Skip
wget http://apache.tt.co.kr/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz
```

![img](/assets/20201001/9.png)

**...**

**....**

**.....**

![img](/assets/20201001/10.png)

**다운로드 받은 ~.tar.gz 파일을 압축 해제 한 후,**

**~bin 디렉토리 아래에 있는 mvn 실행 파일을 실행 시켜 정상 동작하는지 확인한다.**

```bash
tar xvf apache-maven-3.6.3-bin.tar.gz
ls -al
cd apache-maven-3.6.3/bin/
./mvn -version
```
---

#  Maven 환경 변수 설정

![img](/assets/20201001/11.png)

**Maven Home이라고 출력된 경로를 복사한다.**

```bash
vi ~/.bash_profile
```

![img](/assets/20201001/12.png)

**Maven이 설치된 디렉토리를 MAVEN_HOME의 값으로 작성하고,**

**PATH값 마지막에 ":MAVEN_HOME/bin"을 추가해준다.**

```bash
MAVEN_HOME=/home/사용자이름/tools/apache-maven-3.6.3
PATH=기존에 설정된 값:$MAVEN_HOME/bin
export MAVEN_HOME # 추가
```

![img](/assets/20201001/13.png)

**변경한 환경변수 값을 적용하기 위하여 source 명령어를 사용한다.**

**"mvn -version" 커맨드를 통하여 정상 동작 확인을 한다.**

```bash
source ~/.bash_profile # 변경한 환경변수 값 적용
echo $MAVEN_HOME # 확인
mvn -version # 확인
```

---

# Spring Boot CLI 설치

![img](/assets/20201001/14.png)

**본인의 경우 디렉토리 관리를 위하여 core 디렉토리에 Spring Boot CLI, 프로젝트별 워크스페이스는 workspaces 디렉토리 아래에 관리를 할 것이다.**

```bash
cd ~ # 현재 사용자의 home 디렉토리로 이동한다.
mkdir spring-boot
cd spring-boot
mkdir core workspaces
```

![img](/assets/20201001/15.png)

**spring.io에서 공식으로 제공하는 Spring Boot CLI 다운로드 링크를 복사한 후 wget 패키지를 통하여 다운로드 받는다.**

```bash
wget https://repo.spring.io/release/org/springframework/boot/spring-boot-cli/2.3.4.RELEASE/spring-boot-cli-2.3.4.RELEASE-bin.tar.gz
```

![img](/assets/20201001/16.png)

**다운로드 받은 ~tar.gz 파일을 압축 해제 한다.**

```bash
tar xvf spring-boot-cli-2.3.4.RELEASE-bin.tar.gz
```

![img](/assets/20201001/17.png)

**~bin 디렉토리로 이동한 후 spring CLI가 정상 동작하는지 "./spring version" 커맨드를 통하여 확인한다.**

```bash
cd spring-2.3.4.RELEASE/
cd bin/
./spring version
```

---

# Spring Boot CLI 환경변수 설정

![img](/assets/20201001/18.png)

**"spring-boot-cli-2.3.4.RELEASE-bin.tar.gz"을 압축 해제 하였을 때 생성되는 디렉토리 "spring-2.3.4.RELEASE"가 SPRING_HOME 값이 된다.**

**해당 경로를 복사 한다.**

```bash
vi ~/.bash_profile
```

![img](/assets/20201001/19.png)

```bash
SPRING_HOME=/home/사용자이름/spring-boot/core/spring-2.3.4.RELEASE
PATH=기존에 설정된 값:$SPRING_HOME/bin
export SPRING_HOME # 추가
```

![img](/assets/20201001/20.png)

**마찬가지로 source 명령어로 변경한 환경변수 값을 적용 후 "spring version"명령어를 통하여 동작 여부를 확인한다.**

```bash
source ~/.bash_profile # 변경한 환경변수 값 적용
echo $SPRING_HOME # 확인
spring version # 확인
```

---

# Spring Boot CLI를 통한 프로젝트 생성

![img](/assets/20201001/21.png)

**프로젝트를 생성할 디렉토리(본인의 경우 ~workspaces)로 이동한 후,**

**"spring init 프로젝트명"을 입력하면 순식간에 기본 템플릿으로 프로젝트가 생성된다.**

**생성된 프로젝트를 자세히 보면 너~무 default template이다.**

**일반 PC로 작업을 한다면 별 문제가 되지 않겠지만, CLI의 터미널에서 작업할 경우 여간 귀찮은게 아니다.**

**당연히 Spring Boot CLI에서는 여러 옵션을 제공한다. 상세한 정보는 "spring init --list" 커맨드를 통하여 확인하거나 구글링을 해라.**

```bash
spring init 프로젝트명
```

![img](/assets/20201001/22.png)

**위 옵션들은 본인 생각에 이 정도면 적당한 초기 세팅 값이라고 생각한다.**

**여기서 jwapp은 임의로 지정한 값이기 때문에 각자 원하는 값을 넣으면 된다.**

```bash
spring init -d=web --groupId=in.parkjw --artifactId=jwapp --name="JwApp" --package-name=in.parkjw.jwapp --description="JwApp Demo" --build maven jwapp
```
---

# Spring Boot 어플리케이션 빌드 및 구동

**Spring Boot Application을 구동하는 순서는 소스 빌드(build) > 구동(run) 순이다.**

![img](/assets/20201001/23.png)

**먼저 pom.xml(Maven Build 기준)을 열어보자.**

```bash
vi pom.xml
```

![img](/assets/20201001/24.png)

**project/properties/java.version의 기본값은 11이다.**

**JAVA_HOME을 jdk11로 세팅하였으면 따로 수정 할 필요는 없지만, 본인처럼 JDK8로 설정했을 경우 캡쳐와 같이 8로 바꿔주자.**

**JAVA_HOME은 JDK8인데 java.version 값은 11일 경우 아래와 같은 에러가 발생할 것이다.**

**\* 혹은 JAVA_HOME이 JDK가 아닌 JRE일 경우도 발생한다.**

    Failed to execute goal org.apache.maven.plugins:maven-compiler-plugin:3.8.1:compile (default-compile) on project jwapp: Fatal error compiling

![img](/assets/20201001/25.png)

**...**

**....**

**.....**

![img](/assets/20201001/26.png)

**최초 build일 경우 maven repo에서 라이브러리들을 가져오기 때문에 다소 오래 걸릴 수 있다.**

```bash
mvn clean install
```

![img](/assets/20201001/27.png)

**target 디렉토리 아래에 빌드된 파일들이 생겼는지 확인 해보자.**

![img](/assets/20201001/28.png)

**...**

**....**

**.....**

![img](/assets/20201001/29.png)

**mvn spring-boot:run 명령어로 Application을 구동 시키자.**

**Log를 보았을 때 Application이 8080(Spring Boot 기본 port)port로 올라왔다는 것을 확인 할 수 있다.**

```bash
mvn spring-boot:run
```
**브라우저의 주소창에 서버IP:8080 으로 접속해보자.**

![img](/assets/20201001/30.png)

**너무 반가운 화면이 출력된다! 여기까지 따라오느라 고생이 많았다...**

---

**만약, Application Log에는 정상적으로 올라갔으나, 브라우저를 통해 접속 시 서버의 응답이 없는 경우**

**방화벽을 확인하자. 만약 inbound 정책이 없다면 추가를 하면 된다.**

**AWS, Oracle Cloud 등 클라우드 서버인 경우 콘솔에 접속해서 보안그룹(방화벽)설정에 Inbound 정책을 추가하고,**

**만약 공유기를 활용해서 서버를 구축하였다면 포트 포워딩 설정을 확인 하자.**

**그리고 위에서 언급한 "외부 방화벽 정책"을 허용해도 접속이 되지 않는다면,**

**OS 내부의 방화벽인 firewall(CentOS 7) or iptables(CentOS 6)에도 정책을 추가하면 된다.**

---

# Spacevim 설치

**이제 마지막 단계이다.**

**~~상개발자이면 그냥 여기서 그만하고 vim 에디터로 개발을 시작하면 된다.~~**

---

# Neovim 설치

**Spacevim은 vim 에디터에 설치를 할 수 있고, Neovim 에디터에도 설치할 수 있다.**

**vim에디터는 단순 문서를 볼 때도 사용하기 때문에, Neovim 에디터를 설치한 후 Neovim 에디터에 Spacevim을 설치하도록 하겠다.**

**\* Spacevim으로 문서를 열 때 순정 vim에디터 보다 무겁게 열리기 때문....**

![img](/assets/20201001/31.png)

```bash
sudo yum -y install epel-release
sudo curl -o /etc/yum.repos.d/dperson-neovim-epel-7.repo https://copr.fedorainfracloud.org/coprs/dperson/neovim/repo/epel-7/dperson-neovim-epel-7.repo 
sudo yum -y install neovim --enablerepo=epel
```
---

# (Neovim) Spacevim 설치

![img](/assets/20201001/32.png)

**기본 설치는 이 명령어 한줄만 입력하면 끝난다.**

```bash
curl -sLf https://spacevim.org/install.sh | bash -s -- --install neovim
```

![img](/assets/20201001/33.png)

**nvim을 입력한 후 엔터를 쳐보자.**

```bash
nvim
```

**~~아무리 생각해도 Spacevim 개발자는 정말 대단한것 같다.(심지어 Logo Text는 에디터를 열 때 마다 바뀐다..)~~**

---

# (Neovim) Spacevim Java Language Setting

**많은 설정들이 있겠지만, 해당 포스팅에서는 정말 기초 설정만 하고, 나머지는 각자 구글링을 하여서 설정을 하자.**

![img](/assets/20201001/34.png)

```bash
vi ~/.SpaceVim.d/init.toml
```

![img](/assets/20201001/35.png)

**설정 파일 맨 아래에 해당 값을 추가한 후 문서를 저장하고 나간다..**

```bash
[[layers]]
name = "lang#java"
```

---

# Spacevim 으로 Spring Boot 개발하기

**말은 거창하지만 가장 기본적인 내용만 설명하겠다.**

![img](/assets/20201001/36.png)

**1. 에디터 열기**

```bash
cd 프로젝트 워크스페이스 디렉토리
nvim
```

![img](/assets/20201001/37.png)

**2. 초기 화면 및 view 이동**

**자세히 보면 "1 startify", "2 vimfiler"라고 써있는데**

**중립상태(esc)에서 spacebar를 누르면 단축키 가이드가 나온다.**

![img](/assets/20201001/38.png)

**만약 파일 목록으로 포커스(커서)를 옮기고 싶다면 (터치, 마우스클릭도 가능) (esc) > spacebar > 2를 누르면 포커스가 옮겨진다.**

```bash
(esc) > spacebar > 원하는 view의 번호
```

**3. 터미널창 열기**

![img](/assets/20201001/39.png)

**spacebar > "을 누르게되면 보통 말하는 터미널창이 열린다.**

```bash
(esc) > spacebar > " 
```

**4. 코드 수정 후 컴파일(빌드)**

![img](/assets/20201001/40.png)

**...**

**....**

**.....**

![img](/assets/20201001/41.png)

**3번 항목에서 터미널을 연 후 Spring Boot App의 Root 디렉토리(보통 pom.xml이 위치한 디렉토리)로 이동 후,**

**mvn clean install 입력**

```bash
mvn claen install
```

**5. Application 구동**

![img](/assets/20201001/42.png)

**...**

**....**

**.....**

![img](/assets/20201001/43.png)

**3번 항목에서 터미널을 연 후 Spring Boot App의 Root 디렉토리(보통 pom.xml이 위치한 디렉토리)로 이동 후,**

**mvn spring-boot:run 입력**

**수정된 코드가 반영되는 것을 확인 할 수 있다.**

```bash
mvn spring-boot:run
```

---

# 이제 정말 끝났다!

**이렇게 구성한 개발서버에 스마트폰이나, 태블릿, 혹은 PC에서 SSH Client App등으로 해당 서버에 접속하면,**

**CLI 터미널 기반으로 Spring Boot 개발을 할 수 있다.**

**특히 Spacevim으로 개발을 할 경우 터미널 화면을 클릭이나, 터치가 동작하므로 무선키보드만 준비되어있다면 생각보다 편하게 개발을 할 수 있다.**

**~~이제 상개발자다운 환경에서 코딩을 즐길 수 있다.~~**