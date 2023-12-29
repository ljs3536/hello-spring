/23-12-21 spring.io -> projects -> spring boot -> learn -> reference.doc thymeleaf.org

spring boot는 Tomcat을 내장하고 있고 웹 브라우저에서 내장 톰켓 서버에 요청을 하고 스프링컨테이너에 있는 컨트롤러에서 해당 요청을 받아 해당 컨트롤러의 로직을 진행한 후 컨트롤러가 리턴값으로 문자를 반환하면 뷰 리졸버(vieResolcer)가 화면을 찾아서 처리한다.

스트링 부트 템플릿엔진 기본 viewName 매핑
resources:templates/ +(ViewName) + .html
참고. spring-boot-devtools 라이브러리를 추가하면, html 파일을 컴파일만 해주면 서버 재시작 없이 View파일 변경이 가능하다.

/23-12-22 ./gradlew build 오류 발생 해결 하기 위한 방법은?

/23-12-23 ./gradlew build에서 오류가 난 원인은 PC에서 사용중인 JAVA_HOME의 jdk버전은 1.8이었지만 구성한 project의 java version은 17이었기 때문에 JAVA_HOME 경로를 해당 17버전의 jdk로 변경해 주었다.

스프링 웹개발 기초
정적 컨텐츠

MVC와 템플릿 엔진

API

정적 컨텐츠 EX) /static 폴더 안에 존재하는 html 정적 컨텐츠는 웹브라우저 요청 (/Hello-static.html) 을 내장 톰켓서버에서 받으면 스프링부트에서는 스프링 컨테이너에서 요청에 관한 컨트롤러가 있는지 확인한다. 허나 없으면, 내부 리소시스 안에있는 해당 요청파일이 존재하는지 찾아서 반환해준다.

/23-12-24 MVC와 템플릿 엔진 MVC:Model, VIew, Controller View : 화면을 그리는데 집중 Controller, Model : 비즈니스 로직, 내부적인 처리하는데 집중

thymeleaf 사용 시 ${}는 model안에 있는 값을 치환해서 나타낸다

웹브라우저에서 localhost:8080/hello-mvc 요청을 보내면 내장 톰켓서버에서 받고 톰켓서버는 이를 스프링한테 보낸다 스프링은 helloController의 메서드에 맵핑이 돼있네 해서 해당 메서드를 호출하고 해당 메서드에서 리턴을 hello-template라고 하고 모델안에 name이라는 이름과 값을 넣어 스프링한테 주면 스프링의 viewResolver(화면과 관련된 해결자:뷰를 찾아주고 템플릿을 연결)가 동작해 return의 스트링네임과 똑같은걸 찾아 thymeleaf 템플릿 엔진에서 처리해달라고 넘긴다 템플릿 엔진이 렌더링을 해서 변환을 한 HTML을 웹브라우저에 반환을 해서 화면을 보여주게 된다.

API Controller에 @ResponseBody를 쓰게 되면, 응답 바디부에 이 내용을 직접 넣어주겠다는 의미 이전 템플릿 엔진과의 차이는 View가 없이 문자가 그대로 내려간다. 템플릿엔진은 템플릿에 있는 뷰를 조작하지만 위 방식은 리턴한 데이터를 그대로 화면에 나타낸다

@ResponseBody를 사용하고 return을 객체를 해주게 되면 JSON형태의 데이터자체를 화면에 반환한다.

웹브라우저에서 /hello-api 요청을 보내면 내장 톰켓에서 해당 요청을 스프링에 알리고 스프링컨테이너에서 해당 컨트롤러를 찾아서 @ResponseBody가 있으면 이전에 ViewResolver에 던졌던 것과는 다르게 HttpMessageConverter가 동작을 하게 된다. 단순 문자면 StringConverter, 객체면 JsonConverter가 동작해 반환한다.

@ResponseBody를 사용하면

HTTP의 BODY에 문자 내용을 직접 반환
viewResolver 대신에 HttpMessageConberter가 동작
기본 문자처리 : StringHttpMessageConverter
기본 객체처리 : MappingJackson2HttpMessageConverter
Byte 처리 등등 기타 여러 HttpMessageConverter가 기본으로 등록되어 있음
Json 변환라이브러리는 Gson(구글), Jackson 2개가 있다

-참고 : 클라이언트의 HTTP Accept 헤더와 서버의 컨트롤러 반환 타입 정보 둘의 조합해서 HttpMessageConverter가 선택된다.

/23-12-25 회원관리 예제- 백엔드 개발

비즈니스 요구사항 정리

회원 도메인과 리포지토리 만들기

회원 리포지토리 테스트 케이스 작성

회원 서비스 개발

회원 서비스 테스트

비즈니스 요구사항 정리

데이터 : 회원 ID, 이름
기능 : 회원 등록, 조회
아직 데이터 저장소가 선정되지 않음
일반적인 웹 애플리케이션 계층 구조

컨트롤러 : 웹 MVC의 컨트롤러 역할
서비스 : 핵심 비즈니스 로직 구현
리포지토리 : 데이터베이스에 접근, 도메인 객체를 DB에 저장하고 관리
도메인 : 비즈니스 도메인 객체, 예) 회원, 주문, 쿠폰 등등 주로 데이터베이스에 저장하고 관리됨
클래스 의존관계 MemberService -> MemberRepository(Interface) <--- Memory MemberRepository

아직 데이터 저장소가 선정되지 않아서, 우선 인터페이스로 구현 클래스를 변경할 수 있도록 설계
데이터 저장소는 RDB, NoSQL 등등 다양한 저장소를 고민중인 상황으로 가정
개발을 진행하기 위해서 초기 개발 단계에서는 구현체로 가벼운 메모리 기반의 데이터 저장소 사용
회원 도메인과 리포지토리 만들기

회원 레포지토리 테스트 케이스 작성
개발한 기능을 실행해서 테스트 할 때 자바의 main 메서드를 통해서 실행하거나, 웹 애플리케이션의 컨트롤러를 통해서 해당 기능을 실행한다.
이러한 방법은 준비하고 실행하는데 오래 걸리고, 반복 실행하기가 어렵고 여러 테스트를 한번에 실행하기 어렵다는 단점이 있다. 
자바는 JUnit이라는 프레임워크로 테스트를 실행해서 이러한 문제를 해결한다.

/23-12-26
회원 서비스 개발 
이전에 만들었던 Repository를 사용하여 Service클래스를 작성한다.
리팩토링 메뉴 열기 단축키 => Ctrl + T

회원 서비스 개발 테스트
테스트는 given, when, then을 구분해 작성하는것이 좋다.
Try Catch문을 사용한 테스트와 assertThrows 람다식을 사용한 테스트 
테스트 케이스 작성법 및 
Service를 생성하면서 Repository를 넣어주는
Dependency Injection 방법

스프링 빈과 의존관계
스프링 빈을 등록하고, 의존관계 설정하기
회원 컨트롤러가 회원서비스와 회원 리포지토리를 사용할 수 있게 의존관계를 준비하자
- 생성자에 @Autowired가 있으면 스프링이 연관된 객체를 스프링 컨테이너에서 찾아서 넣어준다. 이렇게 객체 의존관계를 외부에서 넣어주는 것을 DI(Dependency Injection), 의존성 주입이라 한다.
- 이전 테스트에서는 개발자가 직접 주입했고, 여기서는 @Autowired에 의해 스프링이 주입한다.

스프링 빈을 등록하는 2가지 방법
- 컴포넌트 스캔과 자동 의존관계 설정
- 자바 코드로 직접 스프링 빈 등록하기

컴포넌트 스캔과 자동 의존관계 설정
- @Component : 애노테이션이 있으면 스프링 빈으로 자동 등록된다.
- @Controller : 컨트롤러가 스프링 빈으로 자동 등록된 이유도 컴포넌트 스캔 때문이다.
- @Component를 포함하는 다음 애노테이션도 스프링 빈으로 자동 등록된다. @Controller , @Service , @Repository

참고 : 생성자에 @Autowired를 사용하면 객체 생성 시점에 스프링 컨테이너에서 해당 스프링 빈을 찾아서 주입한다. 
생성자가 1개만 있으면 @Autowired는 생략할 수 있다.
기본적으로 스프링이 인식하려면, @SpringBootApplication이 포함된 패키지 하위만 스캔을 한다.

참고 : 스프링은 스프링 컨테이너에 스프링 빈을 등록할 때, 기본적으로 싱글톤으로 등록한다.(유일하게 하ㄴ만 등록해서 공유한다)
따라서 같은 스프링 빈이면 모두 같은 인스턴스다. 설정으로 싱글톤이 아니게 설정할 수 있지만, 특별한 경우를 제외하면 대부분 싱글톤을 사용한다.

/23-12-27
자바 코드로 직접 스프링 빈 등록하기
Config클래스를 생성해 @Configuration 어노테이션을 사용하고 @Bean 메서드를 생성하면 Service와 Repository를 생성할 수 있다.

참고 : 과거에는 XML로 설정하는 방식도 있지만 최근에는 잘 사용하지 않는다.
참고 : DI에는 필드 주입, Setter 주입, 생성자 주입 이렇게 3가지 방법이 있다.
의존관계가 실행중에 동적으로 변하는 경우는 거의 없으므로 생성자 주입을 권장한다.
필드 주입의 단점은 변경할 수 있다는 점, Setter주입은 public하게 노출이 되어 누군가 임의로 변경이 가능해진다는 단점이 있음
참고 : 실무에서는 주로 정형화된 컨트롤러, 서비스, 리포지토리 같은 코드는 컴포넌트 스캔을 사용한다.
그리고 정형화 되지 않거나, 상황에 따라 구현 클래스를 변경해야 하면 설정을 통해 스프링 빈으로 등록한다.
주의 : @Autowired 를 통한 DI는 helloController, MemberService 등과 같이 스프링이 관리하는 객체에서만 동작한다.
스프링 빈으로 등록하지 않고 내가 직접 생성한 객체에서는 동작하지 않는다.

회원 웹 기능 - 홈화면 추가
localhost:8080요청이 오면 스프링 컨트롤러에서 해당 매핑이 있는지 확인하고 없으면 정적 컨트츠를 제공하기 때문에 매핑되는 페이지를 만들면 해당 부분을 반환한다.

/23-12-28
화면 웹 기능 - 등록
화면 웹 기능 - 조회

스프링 DB 접근 기술

스프링 데이터 엑세스
H2 데이터 베이스 설치
순수 Jdbc 
스프링 JdbcTemplate
JPA
스프링 데이터 JPA

h2database 설치 후 
JDBC URL을 jdbc:h2:tcp://localhost/~/test로 변경해야한다.

