---
title: \[스프링\] MVC 컨트롤러에서 요청 파라미터 바인딩을 하는 여러가지 방법들
author: mirukman
date: 2020-12-26 16:00:00 +0800
categories: [BACKEND, SPRING]
tags: [spring,파라미터,스프링,parameter,initbinder,dateformat,requestparam,modelattribute]
---

## 1. 개별 파라미터 ##
---

~~~ java
@Controller
@Slf4j
public class IndexController {
    
    @GetMapping("/index")
    public String index(@RequestParam("name") String name, @RequestParam("age") int age) {
        log.info("name: " + name);
        log.info("age: " + age);
        return "index";
    }
}
~~~

@RequestParam("name")은 "name" 파라미터를 할당하기 위한 것이고 @RequestParam("age")는 "age" 파라미터를 할당하기 위한 것이다.

<br>

아래와 같은 url을 통해 컨트롤러가 호출되면 입력해준 파라미터가 변수에 성공적으로 저장되는 것을 볼 수 있다.

~~~ txt
http://localhost:8080/spring-mvc/index?name=john&age=21
~~~

~~~ txt
[apache-tomcat-9.0.41-5]: [INFO ] 2020-12-26 16:02:25.495 [http-nio-8080-exec-10] IndexController - name: john
[apache-tomcat-9.0.41-5]: [INFO ] 2020-12-26 16:02:25.495 [http-nio-8080-exec-10] IndexController - age: 21
~~~

이러한 일이 가능한 이유는 스프링의 DataBinder가 자동으로 파라미터를 resolve하여 매칭되는 변수를 찾을 수 있기 때문이다. 심지어 String이 아닌 변수타입에 대해서도 자동으로 타입 변환이 적용된다.

<br>

@RequestParam 어노테이션은 생략이 가능하다. 단 생략 후 스프링이 파라미터값을 변수에 올바르게 바인딩하기 위해서는 파라미터명과 변수명이 같아야한다. 따라서 위 컨트롤러 코드를 단순히 아래와 같이 변경할 수 있다.

~~~ java
@Controller
@Slf4j
public class IndexController {
    
    @GetMapping("/index")
    public String index(String name, int age) {
        log.info("name: " + name);
        log.info("age: " + age);
        return "index";
    }
}
~~~

<br>

이번 예시에서는 컨트롤러를 GET 요청으로 제한했지만 @RequestParam 어노테이션이 딱히 GET 방식 요청에서만 사용할 수 있는것은 아니다.

<br>

## 2. List 파라미터 ##
---

동일한 이름의 파라미터가 여러개 전달될 경우 리스트나 배열을 통해 처리할 수 있다. 이와같은 일이 가능한 이유는 스프링이 내부적으로 파라미터 타입에 맞게 자동으로 객체를 생성해주는 기능이 있기 때문이다.

~~~ java
@Controller
@Slf4j
public class IndexController {
    
    @GetMapping("/index")
    public String index(@RequestParam("ids") ArrayList<Integer> ids) {
        for (int id : ids) {
            log.info(String.valueOf(id));
        }
        return "index";
    }
}
~~~

주의할 점은 List같은 인터페이스 타입이 아닌 ArrayList와 같은 구현체를 사용해야 한다는 것이다.

아래와 같이 이름이 같은 여러 파라미터를 통해 URL을 호출할 경우 리스트에 할당이 된다.

~~~ txt
http://localhost:8080/spring-mvc/index?ids=1&ids=2&ids=3
~~~

~~~ txt
[apache-tomcat-9.0.41-5]: [INFO ] 2020-12-26 16:06:14.406 [http-nio-8080-exec-6] IndexController - 1
[apache-tomcat-9.0.41-5]: [INFO ] 2020-12-26 16:06:14.406 [http-nio-8080-exec-6] IndexController - 2
[apache-tomcat-9.0.41-5]: [INFO ] 2020-12-26 16:06:14.406 [http-nio-8080-exec-6] IndexController - 3
~~~

또한 단순 파라미터 전달과는 다르게 @RequestParam 어노테이션이 항상 존재해야한다.

<br>

## 3. 객체 파라미터 ##
---

~~~ java
@Data
public class Person {
    
    private String name;
    private int age;
}
~~~

~~~ java
@Controller
@Slf4j
public class IndexController {
    
    @GetMapping("/index")
    public String index(@ModelAttribute Person person) {
        log.info("name: " + person.getName());
        log.info("age: " + person.getAge());
        return "index";
    }
}
~~~

위와 같이 컨트롤러 메소드 인자로 객체를 전달하면 객체의 필드에 파라미터가 할당된다. 이와 같은 객체를 커맨드 객체라 한다.

스프링은 커맨드객체의 setter 메소드를 통해서 필드에 파라미터값을 저장하게 된다. 따라서 커맨드객체는 필수적으로 setter 메소드를 가져야한다. 더 정확하게는 커맨드 객체는 Java Beans 규약을 따르는 객체여야한다. 좁은 의미로는 getter/setter 메소드 및 빈 생성자를 가지는 객체이다.

그런데 Java Beans가 아닌 객체에 대해서도 컨트롤러 메소드에 특정 처리작업을 추가함으로써 setter가 없어도 필드에 직접 접근을 가능하도록하여 데이터 바인딩이 가능하게하는 방법이 존재한다고한다. 굳이 그럴 필요가 있을까 싶지만 말이다.

<br>

참고로 위 예제의 Person 객체에는 명시적으로 setter를 작성하진 않았지만 lombok 라이브러리의 @Data 어노테이션이 컴파일시 자동으로 setter/getter 와 생성자를 만들어준다.

<br>

아래와 같은 url을 통해 컨트롤러를 호출해보면 입력해준 파라미터가 자신과 이름이 같은 person 객체의 필드에 자동으로 할당된다.

~~~ txt
http://localhost:8080/spring-mvc/index?name=john&age=21
~~~

<br>

커맨드 객체는 스프링에 의해 자동으로 Model 객체에 저장된다. 따라서 View에서 커맨드 객체에 접근이 가능하다. 실제로 View에서는 요청으로 받은 파라미터를 사용하는 일이 흔하다.

@ModelAttribute가 없어도 커맨드 객체에 데이터 바인딩이 되고 Model 객체에 저장까지 된다. 왜냐하면 객체 파라미터가 있을경우 스프링은 @ModelAttribute 어노테이션이 생략된 것으로 보기 때문이다.

그럼 @ModelAttribute를 쓰는 이유는 무엇일까? 기본 자료형같은 경우는 커맨드 객체와 달리 Model에 저장되지 않는다. 이 때 @ModelAttribute를 사용하면 Model에 저장이되고 View에서 접근할 수 있게된다. 그러나 그러한 간단한 데이터들은 `model.setAttribute()`를 통해서도 저장할 수 있다.

따라서 내 생각에는 굳이 @ModelAttribute를 사용할 이유는 다음과 같다.

1) 파라미터명과 변수명을 어떠한 이유로 다르게 하고 싶어서 alias를 줄 때 
	
	=> `@ModelAttribute(name = "name")`

2) 기본 자료형을 Model 객체에 

	=> `public String index(@ModelAttribute(value = "age") int age)`

	
<br>

## 4. 객체 리스트 파라미터 ##
---

기본자료형 뿐만 아니라 커맨드 객체의 배열이나 리스트에도 파라미터를 수집해서 할당할 수 있다.

~~~ java
@Data
public class Person {
    
    private String name;
    private int age;
}
~~~

~~~ java
@Data
public class PersonList {
    
    private List<Person> list;

    public PersonList() {
        list = new ArrayList<>();
    }
}
~~~

Person 객체와 Person객체의 리스트를 가지고 있는 PersonList 객체를 작성했다.

PersonList 객체를 받을 수 있는 컨트롤러는 아래와 같이 작성한다.

~~~ java
@Controller
@Slf4j
public class IndexController {
    
    @GetMapping("/index")
    public String index(PersonList personList) {
        for (Person person : personList.getList()) {
            log.info("name: " + person.getName() + ", age: " + person.getAge());
        }
        return "index";
    }
}
~~~

이 때 주의해야 할 점은 PersonList 객체 내부의 List 필드명을 통해 호출해주어야 한단 것이다. 위에서는 리스트 필드명이 'list'이다. 따라서 호출 URL은 아래와 같아야한다.

~~~ txt
http://localhost:8080/spring-mvc/index?list[0].name=John&list[0].age=20&list[1].name=Adam&list[1].age=30
http://localhost:8080/spring-mvc/index?list%5B0%5D.name=John&list%5B0%5D.age=20&list%5B1%5D.name=Adam&list%5B1%5D.age=30
~~~

첫 번째 url을 URLEncoding한 것이 두 번째 url이다. JavaScript와 같은 클라이언트 언어에서는 `encodeURIComponent()`와 같은 메소드를 통해서 URL 특수문자를 escape 할 수 있다. 그러나 브라우저에서 직접 호출한다면 두 번째 url과 같이 URL 인코딩이 적용된 형식으로 직접 던져주어야한다.

<br>

## 5. @InitBinder ##
---

파라미터를 받아서 변수나 객체 필드에 매핑시키는 작업을 바인딩(binding)이라고 한다. @InitBinder는 데이터를 바인딩 하기 전 특정한 처리작업을 위한 어노테이션이다.

컨트롤러의 특정 메소드에 @InitBinder 어노테이션을 붙여서 매핑 전 해당 메소드가 먼저 실행되도록 함으로써 파라미터에 대한 전처리를 하는 것이다.

<br>

Person 객체가 아래와 같이 작성되었다.

~~~ java
@Data
public class Person {
    
    private String name;
    private Date birth;
}
~~~

지금까지의 방법으로는 java.util.Date와 같은 특별한 데이터타입을 받을 수 없다.

~~~ java
@Controller
@Slf4j
public class IndexController {
    
    @GetMapping("/index")
    public String index(Person person) {
        log.info("name: " + person.getName());
        log.info("birth: " + person.getBirth());
        return "index";
    }

    @InitBinder
    public void initBinder(WebDataBinder binder) {
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
        binder.registerCustomEditor(Date.class, new CustomDateEditor(dateFormat, false));
    }
}
~~~

`initBinder()` 메소드가 먼저 처리되면서 특정 파라미터가 지정한 포맷을 만족하면 java.util.Date 클래스로 변환시키도록 했다.

실행을 시켜보면 아래와 같은 로그가 출력되는데, Person 객체의 birth 필드에 데이터가 잘 bind 된 것을 볼 수 있다.

~~~ txt
[apache-tomcat-9.0.41-5]: [INFO ] 2020-12-26 19:57:42.706 [http-nio-8080-exec-1] IndexController - name: John
[apache-tomcat-9.0.41-5]: [INFO ] 2020-12-26 19:57:42.707 [http-nio-8080-exec-1] IndexController - birth: Mon Dec 11 00:00:00 KST 1995
~~~

<br>

## 6. @DateTimeFormat ##
---

날짜형식은 자주 쓰이는 데이터타입이다. 그래서 @InitBinder보다 더 간단하게 사용할 수 있는 애노테이션이 존재한다.

~~~ java
@Data
public class Person {
    
    private String name;

    @DateTimeFormat(pattern = "yyyy-MM-dd")
    private Date birth;
}

~~~

날짜 형식으로의 변환이 필요한 객체 필드에 @DateTimeFormat 어노테이션을 적용하고 입력으로 들어올 날짜 패턴을 정해주면 스프링이 자동으로 문자열을 날짜 클래스 타입으로 변환해준다.
