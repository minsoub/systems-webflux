# systems-webflux

## WebFlux 개념
Spring Framework5에서 새롭게 추가된 모듈이며 webflus는 client, server에서 reactive 스타일의 어플리케이션 개발을 도와주는 모듈이다. 

### Web on Reactive Stack
스프링 프레임워크는 Servlet API와 Servlet 컨테이너로 이루어졌는데 5버전에서 WebFlux가 추가 되었다.   
WebFlux는 reactive-stack web framework이며 non-blocking에 Reactive streem을 지원한다. webmvc나 webflux 둘다 사용이 가능하다.    
WebFlux가 생긴 이유는,    
1) 적은 양의 스레드와 최소한의 하드웨어 자원으로 동시성을 핸듶링하기 위해 만들어졌다. 서블릿 3.1이 논블로킹을 지원하지만, 일부분이다.   
   이는 새로운 공통 API가 생긴 이유가 되었으며, netty와 같은 잘 만들어진 async, non-blocking 서버를 사용한다.
2) 함수형 프로그래밍 때문이다. Java5에서 Rest controllers나 unit test가 만들어지고, Java8에서는 함수형 API를 위한 Lambda 표현식이 추가되었다. 이는 논블로킹    
   어플리케이션의 API가 기초가 되었다.    
   non-blocking이나 functional이란 단어는 잘 와닿지만, reactive라는 단어를 잘 와닿지 않는다. reactive라는 건 무엇을 의미할까?
   "reactive"라는 건 변환에 반응하게 만들어진 프로그래밍 모델이다. I/O 이벤트에 따라 네트워크 컴포넌트가 반응하고 마우스 이벤트나 다른 것들에 UI 컨트롤러가    
   반응하는 것과 같은. 즉, non-blocking은 reactive한 것이다. 블럭킹을 하는 대신에 오퍼레이션이 완료되거나 데이터가 유효하다는 것에 따른 noti에 반응을 하는 것이다.    
      
   동기성에서는, 블로킹 콜은 콜러를 기다리게 하는 자연스로운 형태이다. 논블러킹 코드에서는 빠른 생산자가 목적지를 앞지르지 않게 이벤트 속도를 제어하는 것이 중요하다.    
   reactive stream은 (java9에 포함된) 비동기성 컴포넌트간의 상호작용에 정의된 작은 스펙이다. 예를 들어, publisher가 subscriber가 응답에 쓸 데이터를 생산한다고 하자.   
   reactive stream의 주 목적은 subscriber가 publisher가 생산하는 데이터의 속도를 제어하는 것이다.   
   결국 spring이 가지지 못했던 비동기 프로그래밍을 보완하기 위한 Spring 버전이라는 의미인것 같다. 그 버전이 Spring5이고 webflux 모듈을 사용한 것이다.    

#### MVC와 WebFlux 동작흐름
- 두개의 동작흐름은 거의 같다.
- MVC : request - Dispatcher Servlet - Handler Mapper - Controller - B/L - Controller - ViewResolver ...
- WebFlux : request - HttpHandler - WebHandler - Handler Mapper / Handler Adapter - Controller - B/L - Controller - ViewResolver ...

#### WebFlux 사용용도 
- 비동기 : 논블록킹 리액티브 개발에 사용
- 효율적으로 동작하는 고성능 웹어플리케이션 개발
- 서비스간 호출이 많은 마이크로서비스 아키텍처에 적합 

### WebFlux는 2가지 모델을 지원
- 기존의 @어노테이션 방식 : Spring MVC 기반과 동일하며 Spring MVC에서 제공하는 Annotation을 그대로 이용 가능핟.
- 새로운 함수형 모델 방식 : Java8 람다식 routing과 handling 방식이다. 가벼운 라우팅기능과 request 처리 라이브러리라고 생각하면 된다.   
  Callback 형태로써 요청이 있을때만 호출된다는 점이 Annotation Controller와의 차이점이다. 
  
#### 어노테이션 방식 (기존 MVC와 거의 동일)
```java
@RestController
@RequiredArgsConstructor
public class TestController {
   private final Service service;
   
   @GetMapping("/user/{name}")
   public Mono<User> hello(@PathVariable("name") String name) {
      return Mono.just(Service.findByName(name));
   }
}
```
#### 함수형 모델 방식
```java
@Configuration
@EnableWebFlux
public class TestFuncConfig {
   @Bean
   public RouterFunction<ServerResponse> routes(TestHandler handler) {
      return RouterFunctions.route(GET("/userfunc"), handler::findByName);
   }
}

@Component
@RequiredArgsConstructor
public class TestHandler {
   private final Service service;
   
   public Mono<ServerResponse> findByName(ServerRequest req) {
      Mono<User> user = Mono.just(service.findByName(req.getQueryParam("name")));
      return ServerResponse.ok().body(user, User.class);
   }
}
```
### 적은 리소스로 많은 트래픽 감당
이에 대한 답은 I/O를 Non Blocking을 이용하여 잘 사용하는 것과 Request를 Event-Driver을 통해서 효율적으로 처리하기 때문에 가능하다.   

### Reference
http://www.devkuma.com/pages/1514
