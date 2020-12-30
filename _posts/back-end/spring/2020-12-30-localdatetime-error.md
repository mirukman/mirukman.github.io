---
title: LocalDateTime 사용시 초 부분이 00이면 분 까지만 출력이 되는 문제
author: mirukman
date: 2020-12-30 21:00:00 +0800
categories: [BACKEND, SPRING]
tags: [spring,스프링,LocalDateTime]
---

스프링 학습 중 게시물 아이템 클래스를 아래와 같이 정의했다. 

~~~ java
@Data
public class BoardVo {
    
    private Long bno;
    private String title;
    private String content;
    private String writer;
    private LocalDateTime regdate;
    private LocalDateTime updatedate;
}
~~~

<br>

View는 JSP를 사용했는데, 게시물의 시간 부분을 표현하기 위해 fmt 태그를 사용했다.

~~~ jsp
<td><fmt:parseDate type="both" pattern="yyyy-MM-dd'T'HH:mm:ss" value="${boardVo.regdate}" /></td>
<td><fmt:parseDate type="both" pattern="yyyy-MM-dd'T'HH:mm:ss" value="${boardVo.updatedate}" /></td>
~~~

<br>

그런데 아래와 같은 에러가 발생한 것이다.

~~~ txt
SEVERE: Servlet.service() for servlet [dispatcher] in context with path [] threw exception [javax.servlet.ServletException: javax.servlet.jsp.JspException: In &lt;parseDate&gt;, value attribute can not be parsed: "2020-12-30T23:21"] with root cause
java.text.ParseException: **Unparseable date**: "2020-12-30T23:21"
	at java.base/java.text.DateFormat.parse(DateFormat.java:395)
	at org.apache.taglibs.standard.tag.common.fmt.ParseDateSupport.doEndTag(ParseDateSupport.java:178)
	at org.apache.jsp.WEB_002dINF.views.board.list_jsp._jspx_meth_fmt_005fparseDate_005f0(list_jsp.java:890)
	at org.apache.jsp.WEB_002dINF.views.board.list_jsp._jspx_meth_c_005fforEach_005f0(list_jsp.java:745)
	at org.apache.jsp.WEB_002dINF.views.board.list_jsp._jspService(list_jsp.java:544)
	...(생략)
~~~

확인을 해본 결과 DB에서 읽어올때까지는 초 부분까지 존재한다. 그런데 LocalDateTime 타입의 필드에 바인딩이 되면서 초 부분이 00이면 사라지는 것이다.

LocalDateTime 클래스는 ISO 국제표준을 준수하는 클래스로 [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) 에는 아래와 같은 부분이 정의되어있다.

~~~ txt
The format used will be the shortest that outputs the full value of the time where the omitted parts are implied to be zero.
~~~

최대한 짧은 패턴을 사용하기위해 최소단위가 0이면 삭제한다는 내용이다.

실제로 LocalDateTime이 내부 멤버변수로 가지고 있는 LocalTime의 toString()은 아래와 같이 재정의 되어있다.

~~~ java
@Override
public String toString() {
	StringBuilder buf = new StringBuilder(18);
	int hourValue = hour;
	int minuteValue = minute;
	int secondValue = second;
	int nanoValue = nano;
	buf.append(hourValue < 10 ? "0" : "").append(hourValue)
		.append(minuteValue < 10 ? ":0" : ":").append(minuteValue);
	if (secondValue > 0 || nanoValue > 0) {
		buf.append(secondValue < 10 ? ":0" : ":").append(secondValue);
		if (nanoValue > 0) {
			buf.append('.');
			if (nanoValue % 1000_000 == 0) {
				buf.append(Integer.toString((nanoValue / 1000_000) + 1000).substring(1));
			} else if (nanoValue % 1000 == 0) {
				buf.append(Integer.toString((nanoValue / 1000) + 1000_000).substring(1));
			} else {
				buf.append(Integer.toString((nanoValue) + 1000_000_000).substring(1));
			}
		}
	}
	return buf.toString();
}
~~~

초 이하의 부분이 0인지 체크를 해서 출력버퍼에 포함시킬지를 정한다.

<br>
위와 같이 제대로 된 원인을 파악하기 전에 아래와 같은 시도들을 했었다.

1. @DateTimeFormat 어노테이션으로 pattern 강제지정
2. fmt 태그에서 type, timeStyle 속성을 이것저것 바꿔가며 시도

<br>

꽤 삽질하다가 LocalDateTime의 클래스를 들여다보니 위와같은 코드가 있었다.

결국 toString()이 재정의되었기 때문에 객체의 포맷지정같은 짓들은 아무리 해도 결국 최종 출력형태에서 초단위가 제거되는 것이기 때문에 결국은 view 레벨에서 해결하는게 좋아보였다.

fmt 태그는 자바 클래스를 사용하다가 에러가 발생할 위험이 있다. 그래서 그냥 아래와 같이 문자열 출력으로 바꿨다.

~~~ java
<td><c:out value="${boardVo.regdate.toString()}" /></td>
<td><c:out value="${boardVo.updatedate.toString()}" /></td>
~~~

이렇게 하면 초 부분이 00이면 분 단위까지만 출력이 되는데, 굳이 00이라도 초까지 출력하고싶다면 view에서 체크를 해서 초 부분이 없으면 00을 붙여주면 되겠다.