# WebFilter, HandlerFilterFunction
Webflux는 Servlet 기반이 아닌 Reactive 기반의 Spring Framework이다.  Servlet이 아니라는 것은 javax.servlet.Filter를 사용할 수 없다.    
이를 대신해 Webflux는 WebFilter와 HandlerFilterFunction을 제공한다.

## Controller, Router
```java
@Configuration
public class IndexRouter {
  @Bean
  public RouterFunction<ServerResponse> route(IndexHandler indexHandler) {
     return RouterFunction
             .route(GET("/index")
             .and(accept(MediaType.TEXT_PLAIN)), indexHandler::index);
  }
  @Bean
  public RouterFunction<ServerResponse> filterFunction(UserHandler userHandler) {
     return RouterFunction
             .route(GET("/user/{name}")
             .and(accept(MediaType.APPLICATION_JSON)), userHandler::name);
  }
}

@Component
public class IndexHandler {
  public Mono<ServerResponse> index(ServerRequest request) {
     return ServerResponse.ok().contentType(MediaType.TEXT_PLAIN)
             .body(BodyInserters.fromObject("Index Homepage!!!"));
  }
}

@Component
public class UserHandler {
   public Mono<ServerResponse> name(ServerRequest request) {
      return ServerResponse.ok().contentType(MediaType.TEXT_PLAIN)
              .body(BodyInserters.fromObject(request.pathVariable("name")));
   }
}
```
