---
title: [스프링] @ControllerAdvice를 통한 컨트롤러 exception 처리
author: mirukman
date: 2020-12-27 16:00:00 +0800
categories: [BACKEND, SPRING]
tags: [spring,파라미터,스프링,parameter,initbinder,dateformat,requestparam,modelattribute]
---

@ControllerAdvice는 스프링 프로젝트 내의 컨트롤러들에 대해 공통적으로 적용하기 위한 전역 코드를 작성할 때 사용하는 어노테이션이다. 거의 대부분은 @ExceptionHandler 어노테이션과 같이 사용해서 컨트롤러의 전역 예외처리를 위해서 사용한다.

여기서는 @ControllerAdvice \+ @ExceptionHandler 두 어노테이션을 조합하여 스프링 컨트롤러의 전역 처리기를 만드는 방법을 정리한다.

~~~ java
@ControllerAdvice
@Slf4j
public class CommonExceptionAdvice {
    
	@ExceptionHandler(NoHandlerFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public String handle404(NoHandlerFoundException ex) {

        return "error_404";
    }
	
    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public String exception(Exception ex, Model model) {
        log.error("Exception......" + ex.getMessage());
        model.addAttribute("exception", ex);
        log.error(model.toString());

        return "error_page";
    }
}
~~~

모든 컨트롤러에 대해 적용을 하기 위해서 클래스에 @ControllerAdvice를 적용한다. 이제 모든 컨트롤러에 대해 클래스 코드가 적용된다. 또한 위 클래스를 빈으로 등록해주어야한다. servlet-context.xml이나 Java confg에 추가해주면된다.

<br>

위 예에서는 404 에러와 나머지 에러로 분리해서 다뤘다. 나머지 에러들은 핸들러 매핑이 된 후 컨트롤러가 작동하는 도중 발생하므로 처리하기가 쉽다.

그러나 404에러는 URL, 경로 오류로 핸들러를 찾지 못했을 경우 발생하므로 따로 처리해주었다. 그리고 web.xml이나 Java 설정파일에 아래와 같은 설정이 필요하다.

web.xml
~~~ xml
<servlet>
	<servlet-name>appServlet</servlet-name>
	<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
	
	...
	
	<init-param>
		<param-name>throwExceptionIfNoHandlerFound</param-name>
		<param-value>true</param-value>
	</init-param>
</servlet>
~~~

Java 설정
~~~ java
public class WebConfig  extends AbstractAnnotationConfigDispatcherServletInitializer {

	...
	
	@Override
	protected void customizeRegistration(ServletRegistration.Dynamic registration) {
		registration.setInitParameter("throwExceptionIfNoHandlerFound", "true");
	}
    
}
~~~

이 설정은 DispatcherServlet으로 하여금 주소,경로를 찾지 못해 핸들러 매핑을 하지못하면 예외를 발생시키도록 한 설정이다.

<br>

설정이 끝났다. 이제는 컨트롤러에서 예외가 발생하면 위에서 미리 설정해둔 page를 화면에 보여준다.
