# WebFlux-vs-MVC
개인적으로 생각하는 WebFlux와 MVC 차이점을 적어놓은 저장소입니다.

### 차이점
  - WebFlux는 비동기-논블로킹 방식입니다.
  - WebFlux는 기존의 어노테이션(@) 방식도 허용하지만 함수형 모델(RouterFunction, HandlerFunction)을 이용한 방식도 제공합니다.
  - WebFlux는 Servlet 기반이 아니기 때문에 HttpServletRequest, HttpServletResponse를 사용하지 않고 이를 추상화한 ServerRequest, ServerResponse를 사용합니다.
  - WebFlux는 동기 방식의 Servlet을 사용하지 않지만 3.1 버전 이상의 비동기-논블로킹 처리 방식을 이용하고, 다른 컨테이너로는 Undertow, Netty가 있습니다.
  - WebFlux 도입과 동시에 Embedded Netty가 함께 제공되어, 기존에 Tomcat을 사용한 MVC에 비해 많은 요청에 효율적인 성능을 보입니다.(Tomcat - 1 Request : 1 Thread, Netty - Many Request : 1 Thread)
  
### WebFlux 사용 용도
  - 효율적으로 동작하는 고성능 웹 애플리케이션 개발에 사용합니다.
  - ex) 포털 사이트의 메인 페이지
  
### WebFlux가 성능이 좋으니 무조건 사용해야 하나?
  ![img](https://user-images.githubusercontent.com/37733264/82406011-7bc25680-9aa0-11ea-9189-b9ef34175c4e.png)
  - Boot1이 MVC이고 Boot2가 WebFlux입니다.
  - 위 표와 같이 많은 요청에서는 WebFlux가 성능이 우수하나, 비교적 적은 요청에서는 MVC, WebFlux 둘다 성능이 비슷비슷합니다. 그러므로 무조건 사용해야 하는 것이 아니라 상황에 맞게 선택해야합니다.

  - MVC 예제 코드
  ~~~ java
  @RestController
  @RequestMapping("/api/articles")
  @RequiredArgsConstructor
  public class ArticleController {

    private final ArticleService articleService;

    @GetMapping(produces = MediaType.APPLICATION_JSON_VALUE)
    public ResponseEntity<?> getArticles(){
        return ResponseEntity.ok(articleService.getArticles());
    }

    @PostMapping
    public ResponseEntity<?> registerArticle(@RequestBody ArticleRegisterDto articleRegisterDto) {
        return new ResponseEntity<>(articleService.save(articleRegisterDto), HttpStatus.CREATED);
    }
  }
  ~~~
  - RouterFunction 예제 코드
  ~~~ java
  @Configuration
  @RequiredArgsConstructor
  public class ArticleRouter {

    private final ArticleRepository articleRepository;

    @Bean
    public RouterFunction<ServerResponse> route() {
        ArticleHandler articleHandler = new ArticleHandler(articleRepository);
        return RouterFunctions.nest(path("/api/articles"),
                RouterFunctions.route(POST("/").and(contentType(MediaType.APPLICATION_JSON)), articleHandler::createArticle)
                .andRoute(GET("/").and(accept(MediaType.APPLICATION_JSON)), articleHandler::getAllArticle)
        );
    }
  }
  ~~~
  - HandlerFunction 예제 코드
  ~~~ java
  @Component
  @RequiredArgsConstructor
  public class ArticleHandler {

    private final ArticleRepository articleRepository;

    public Mono<ServerResponse> createArticle(ServerRequest request) {
        Mono<Article> articleMono = request.bodyToMono(Article.class)
                .flatMap(article -> articleRepository.save(article));
        return ServerResponse.ok().contentType(MediaType.APPLICATION_JSON).body(articleMono, Article.class);
    }

    public Mono<ServerResponse> getAllArticle(ServerRequest request) {
        Flux<Article> articleFlux = articleRepository.findAll();
        return ServerResponse.ok().contentType(MediaType.APPLICATION_JSON).body(articleFlux, Article.class);
    }
  }
  ~~~

### 함수형 WebFlux의 장점
  - 모든 웹 요청 처리 작업을 명시적인 코드로 작성 가능합니다.
  - 함수조작을 통한 편리한 구성/추상화에 유리합니다.
  - 테스트 작성이 편리합니다.(핸들러 로직, 요청 매핑, 리턴값까지 단위 테스트 가능)

### 함수형 WebFlux의 단점
  - 함수형 스타일의 코드 작성이 편하지 않으면 코드 작성과 이해 모두 어렵습니다.
  
### 함수형을 쓰는 이유
  - 간결하고 조합하기 편한 코드를 작성할 수 있습니다.
  - 데이터 흐름에 따른 다양한 오퍼레이터가 가능합니다.
  - 연산을 조합해서 만든 동시성 정보가 노출되지 않는 추상화된 코드를 작성할 수 있습니다.
  - 데이터 흐름의 속도를 제어할 수 있는 메커니즘을 제공합니다.

### 레퍼런스
  - https://www.youtube.com/watch?v=2E_1yb8iLKk&t=455s
  - https://spring.io/guides/gs/reactive-rest-service/
  - https://dzone.com/articles/raw-performance-numbers-spring-boot-2-webflux-vs-s
  - https://jojoldu.tistory.com/155
