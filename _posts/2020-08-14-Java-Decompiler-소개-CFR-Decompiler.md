---
layout: post
title:  "Java Decompiler 소개 : CFR Decompiler"
date:   2020-08-14 18:00:00 +0900
categories: java tool
sitemap :
  changefreq : daily
  priority : 1.0
---

가끔 컴파일 된 class 파일을 디컴파일을 해서 원 소스를 확인해야하는 상황이 발생한다.

유명한 디컴파일러로 **jd-gui**가 있다. 해당 디컴파일로의 단점으로는 **Java 8 이상의 버전으로 컴파일 된 소스를 디컴파일 할 수 없다는 점**이다.

따라서 최신 자바 버전을 지원하는 디컴파일러를 소개 하도록 하겠다.

---

# CFR Decompiler

![cfr](/assets/20200814/cfr.png)

CFR Decompiler는 Java 8이상의 최신 버전으로 컴파일된 class 파일을 문제 없이 java 파일로 디컴파일이 가능하다.

\* 사진에서 알 수 있다 시피, Java 6 이하는 지원하지 않는다.

GUI를 제공하지 않고 <U>CLI만 제공</U>한다.

이 점은 사람에 따라서 장점이 될 수 있고, 단점이 될 수 있다. 개인 적으로 CLI를 선호하는 편이라 이 부분은 필자에게 단점은 아니었다.

jar 파일로 배포 된다. 따라서 cfr을 사용하려면 사용할 <U>환경에 Jre or Jdk가 설치</U>되어 있어야 한다.

또한 **jar**형태이기 때문에, 사용자의 **OS환경과 무관**하게 java만 설치 되어있으면 사용 가능하다.

\* 즉, 서버에 Java만 설치 되어 있다면 FTP로 파일을 옮기지 않고 **바로 소스를 디컴파일 할 수 있다!**

---

# CFR Decompiler 다운 로드

[Download Url](https://www.benf.org/other/cfr/)

위 링크를 클릭하여 이동한 후, **Download cfr** 부분에 링크를 클릭하여 **cfr-version.jar** 다운을 한다. (다운로드 시점에서 가장 최신 버전을 받는 걸 추천한다.)

\* 그냥 다운로드 폴더에 저장하지말고, <U>본인 PC에 개발 관련 tool을 저장하는 디렉토리에 옮겨 놓는게</U> 추후 재사용시 혼동이 없을 것 같다.

```bash
➜  cfr pwd
/Users/pjw/Documents/dev/tools/decompiler/cfr
➜  cfr ls
cfr-0.150.jar class         java
```
**적당한 경로**에 다운 받은 **cfr-version.jar** 옮겨 놓는다.

---

# CFR Decompiler 실행 방법

```bash
➜  cfr java -jar cfr-0.150.jar A.class > A.java
```
위와 같이 **java -jar "디컴파일할 클래스 파일명" > "디컴파일한 java 파일명"** 으로 인자를 주면 <U>디컴파일된 소스를 파일로 저장</U> 할 수 있다.

```bash
➜  cfr java -jar cfr-0.150.jar A.class
```
만약 디컴파일한 java 소스를 **파일로 저장하지 않을 경우** 위와 같이 **2번 째 인자를 생략**한다.

```bash
➜  cfr ll
total 4192
-rwxr--r--@ 1 pjw  staff  1998700  8 14 17:51 cfr-0.150.jar
drwxr-xr-x  3 pjw  staff       96  8 14 17:55 class
drwxr-xr-x  3 pjw  staff       96  8 14 17:57 java
```
필자의 경우 디컴파일을 할 소스를 저장할 폴더 "class", 디컴파일이 된 java 파일을 저장할 폴더 "java" 폴더를 생성했다.

\* 그럴 일은 없겠지만, 원본 소스를 건드리고 싶지 않기 때문에, "class" 폴더에 소스를 copy 한 후 작업을 한다.

---

# CFR Decompiler 사용 예시

\* 참고로 예시에서 사용한 **IndexCtl.class**은 **Java 1.8**로 컴파일한 소스이다.

```bash
➜  cfr pwd
/Users/pjw/Documents/dev/tools/decompiler/cfr
➜  cfr ll
total 4192
-rwxr--r--@ 1 pjw  staff  1998700  8 14 17:51 cfr-0.150.jar
drwxr-xr-x  3 pjw  staff       96  8 14 17:55 class
drwxr-xr-x  3 pjw  staff       96  8 14 17:57 java
➜  cfr java -jar cfr-0.150.jar class/IndexCtl.class > java/IndexCtl.java
➜  cfr ls -Rl class java
class:
total 8
-rw-r--r--  1 pjw  staff  575  8 14 17:55 IndexCtl.class

java:
total 8
-rw-r--r--  1 pjw  staff  456  8 14 18:32 IndexCtl.java
➜  cfr cat java/IndexCtl.java
/*
 * Decompiled with CFR 0.150.
 *
 * Could not load the following classes:
 *  org.springframework.stereotype.Controller
 *  org.springframework.web.bind.annotation.GetMapping
 */
package in.parkjw.jwtoolbox.ctl;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class IndexCtl {
    @GetMapping(value={"/"})
    public String main() {
        return "main/index";
    }
}
```

