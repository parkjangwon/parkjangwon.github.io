---
layout: post
title:  "HttpServletRequest의 모든 파라미터를 출력하는 방법"
date:   2020-08-10 00:34:00 +0900
categories: java servlet
sitemap :
  changefreq : daily
  priority : 1.0
---

서블릿 기반의 웹어플리케이션을 개발할 때, 특정 컨트롤러에 요청한 request 파라미터에 대해 디버깅을 할 때가 있다.

break point를 걸어 variables view에서 값을 하나 하나 볼 수 있지만... **가끔 너무 귀찮을 때가 있다.**

아래 구문을 메모를 해두고, 디버깅 조차하기도 너무나 귀찮을 때 종종 꺼내서 쓰면 편리하다.

**HttpServletRequest**를 파라미터로 받는 메서드 (컨트롤러) 첫 줄에 아래 코드를 copy & paste를 한다.

# 리퀘스트에 담긴 모든 파라미터 출력하기

```java
Enumeration params = request.getParameterNames();
System.out.println("----------------------------");
while (params.hasMoreElements()){
    String name = (String)params.nextElement();
    System.out.println(name + " : " +request.getParameter(name));
}
System.out.println("----------------------------");
```