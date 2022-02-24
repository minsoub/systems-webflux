# WebFilter, HandlerFilterFunction
Webflux는 Servlet 기반이 아닌 Reactive 기반의 Spring Framework이다.  Servlet이 아니라는 것은 javax.servlet.Filter를 사용할 수 없다.    
이를 대신해 Webflux는 WebFilter와 HandlerFilterFunction을 제공한다.

# WebFilter Example
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

## WebFilter
```java
public interface WebFilter {
   Mono<Void> filter(ServerWebExchange exchange, WebFilterChain chain);
}
```
WebFilter interface를 implement하고, @Component 등록한다.
필터를 통해 header에 값을 넣는다.
```java
@Component
public class SystemsWebFilter implements WebFilter {
   @Override
   public Mono<Void> filter(ServerWebExchange serverWebExchange, WebFilterChain webFilterChain) {
       serverWebExchange.getResponse()
          .getHeaders().add("web-filter", "web-filter-test");
       return webFilterChain.filter(serverWebExchange);
   }
}
```
## Test code
```java
@WebFluxTest(value = {IndexHandler.class, IndexRouter.class, SystemsWebFilter.class, UserHandler.class})
class SystemsWebFilterTest {
   @Autowired
   WebTestClient webTestClient;
   
   @DisplayName("webFilter Test")
   @Test
   void webfilter_test() {
      webTestClient.get()
         .uri("/index")
         .exchange()
         .expectStatus().isOk()
         .expectBody(String.class)
         .isEqualTo("Index Homepage!!!")
         .consumeWith(response->
             assertEquals(response
                           .getResponseHeaders()
                           .getFirst("web-filter"), "web-filter-test")
         );
   }
}
```
Webfilter는 모든 request에 filter를 걸리게 한다. 만약 특정한 경우에 하고 싶으면 url pattern matching을 사용해서 분기해야 한다.

# HandlerFilterFunction example
