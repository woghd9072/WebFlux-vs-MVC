# WebFlux-vs-MVC
개인적으로 생각하는 WebFlux와 MVC 차이점을 적어놓은 저장소입니다.

### 차이점
  - 일반적으로 사용하는 MVC는 동기 방식이고, WebFlux는 비동기-논블로킹 방식입니다.
  - WebFlux는 기존의 어노테이션(@) 방식도 허용하지만 함수형 모델(RouterFunction, HandlerFunction)을 이용한 방식도 제공합니다.
  - WebFlux는 Servlet 기반이 아니기 때문에 HttpServletRequest, HttpServletResponse를 사용하지 않고 이를 추상화한 ServerRequest, ServerResponse를 사용합니다.
  - WebFlux는 동기 방식의 Servlet을 사용하지 않지만 3.1 버전 이상의 비동기-논블로킹 처리 방식을 이용하고, 다른 컨테이너로는 Undertow, Netty가 있습니다.
  
  - MVC 예제 코드
  ~~~ java
  @RestController
  @RequestMapping("/api/posts")
  @RequiredArgsConstructor
  public class PostController {

    private final PostService postService;

    @GetMapping(produces = MediaType.APPLICATION_JSON_VALUE)
    public ResponseEntity<?> getPosts(){
        return ResponseEntity.ok(postService.getPosts());
    }

    @PostMapping
    public ResponseEntity<?> registerPost(@RequestBody PostRegisterDto postRegisterDto) {
        return new ResponseEntity<>(postService.save(postRegisterDto), HttpStatus.CREATED);
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
  
### WebFlux 사용 용도
  - 비동기-논블로킹 리액티브 개발에 사용
  - 효율적으로 동작하는 고성능 웹 애플리케이션 개발
  - 서비스간 호출이 많은 마이크로서비스 아키텍처에 적합
  
### WebFlux가 성능이 좋으니 무조건 사용해야 하나?
  ![img](./사진/img.png)
  - 많은 요청에서는 WebFlux가 성능이 우수하나, 비교적 적은 요청에서는 MVC, WebFlux 둘다 성능이 비슷비슷합니다. 그러므로 무조건 사용해야 하는 것이 아니라 상황에 맞게 선택해야합니다.

### 레퍼런스
  - https://www.youtube.com/watch?v=2E_1yb8iLKk&t=455s
  - https://spring.io/guides/gs/reactive-rest-service/
  - https://dzone.com/articles/raw-performance-numbers-spring-boot-2-webflux-vs-s
