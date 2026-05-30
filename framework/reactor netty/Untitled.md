
WebFlux는 **Netty 기반(Reactor Netty)** 의 **non-blocking, reactive pipeline** 위에 동작하고 있어서,  
메시지 프레이밍(framing), partial buffer, 여러 chunk를 이어붙이는 작업을 **Netty + Reactor가 자동으로 처리**합니다.

따라서 WebFlux 개발자는 TCP 소켓을 직접 read/write 하는 환경과 달리:

> “바이트 조각이 여러 번으로 나눠서 도착했는데, 내가 버퍼링해야 하나?”  
> “메시지가 부분적으로 들어왔을 때 저장했다가 이어붙여야 하나?”

이런 걸 직접 구현할 필요가 없음

---

### WebFlux에서 “Partial Message”가 어떻게 처리되는가?

#### 1) **Netty의 ByteBuf → frame decoder가 fragment를 조립**

WebFlux의 기본 런타임은 Netty입니다.

Netty는 TCP 스트림에서 다음을 자동으로 처리합니다:

- partial packet 조립
    
- chunked request body merge
    
- HTTP 프레임 단위로 분리
    
- WebSocket 프레임 조립
    
- backpressure 적용
    

즉 Netty가 raw TCP 바이트를 **HTTP Request 단위로 자동 복원(assemble)** 해줍니다.

#### 2) **Reactor Netty가 ByteBuf를 Flux<DataBuffer> 로 변환**

WebFlux는 메시지 바디를:

`ServerRequest.bodyToFlux(String.class)`

또는

`serverRequest.bodyToFlux(DataBuffer.class)`

같이 읽게 되는데,

이때 내부적으로는:

`TCP chunk1  → Netty ByteBuf → DataBuffer #1 TCP chunk2  → Netty ByteBuf → DataBuffer #2 ... Flux<DataBuffer> 로 스트림 처리`

하지만 WebFlux는 “문자 단위 또는 객체 단위”로 파싱할 때 이걸 **자동으로 이어붙여줌**.

예: JSON body가 3번 나눠서 도착해도

`{"name":"jun"}`

가 완성될 때까지 내부가 buffer merge → JSON decode → Mono<Pojo> 로 변환.

개발자는 조각을 합칠 필요가 없음.


#### 3) MessageReader/MessageDecoder 역할을 하는 계층이 있음

WebFlux에는 직접 MessageReader를 구현하지 않아도 되는 이유는:

### WebFlux 내부 스택

`(Netty ByteBuf)       ↓ ReactiveNetty HttpDecoder       ↓ Reactor Netty Codec       ↓ HttpMessageReader (WebFlux)       ↓ BodyExtractors → Mono<T> / Flux<T>`

여기서 `HttpMessageReader<T>`가  
**프레이밍・조립・파싱 모두 자동 처리**합니다.


---

```shell
[TCP 소켓 바이트]
   ↓ (Netty)
[ByteBuf 스트림]
   ↓ (Netty HTTP codec)
[HTTP Request + HTTP Body(ByteBuf...)]
   ↓ (Reactor Netty HttpServer)
[Flux<ByteBuf>]
   ↓ (Spring DataBuffer 추상화)
[Flux<DataBuffer>]
   ↓ (HttpMessageReader + Decoder)
[Mono<T> / Flux<T>]
   ↓
@RestController / HandlerFunction
```


#### ByteBuf가 어떻게 Flux<DataBuffer> / Mono<T>까지 올라오는가?

##### Netty 레벨: TCP 바이트 → ByteBuf → HTTP 프레임

- Netty의 채널 파이프라인에서, 소켓에서 읽은 raw 바이트는 `ByteBuf` 라는 타입으로 관리
    
- Netty의 HTTP codec(`HttpRequestDecoder` 등)이 이 ByteBuf들을 읽어 HTTP Request / Content 로 파싱

##### Spring 추상화: ByteBuf → DataBuffer

Spring은 서버 종류(Netty, Jetty, Undertow…)가 달라도 같은 코드로 처리하고 싶기 때문에,  
`DataBuffer` 라는 공용 바이트 버퍼 추상화를 둡니다.[Spring Docs+1](https://docs.spring.io/spring-framework/reference/core/databuffer-codec.html?utm_source=chatgpt.com)

- Netty일 때는 `NettyDataBuffer`가 내부에 `ByteBuf`를 감싸고 있고,
    
- Jetty면 Jetty 버퍼를 감싸는 식.
    

공식 문서 표현 그대로 요약하면:

> DataBuffer는 WebFlux에서 byte buffer를 표현하는 타입이고,  
> 내부적으로는 Netty의 ByteBuf 같은 구현체를 감싸는 추상화 레이어.[Spring Docs+1](https://docs.spring.io/spring-framework/docs/5.3.6/reference/html/web-reactive.html?utm_source=chatgpt.com)

그래서 Reactor Netty가 넘겨주는 `Flux<ByteBuf>` 는 WebFlux 내부에서 곧바로:

`Flux<ByteBuf>  --(DataBufferFactory)-->  Flux<DataBuffer>`


이걸 노출하는 게 `ServerHttpRequest` 의 `getBody()`이고, 타입은 `Flux<DataBuffer>` [](https://docs.spring.io/spring-framework/docs/5.3.6/reference/html/web-reactive.html)

####  3. WebFlux 계층: DataBuffer → T (HttpMessageReader/Decoder)

이후는 완전 Spring WebFlux 레이어

##### 3-1. HttpMessageReader의 역할

`HttpMessageReader<T>` 의 책임:

1. `Flux<DataBuffer>` 를 받아서
    
2. 프로토콜/컨텐츠 타입에 따라 파싱해서
    
3. `Mono<T>` 또는 `Flux<T>` 로 바꿔준다.
    

예:

- `Jackson2JsonDecoder` 를 감싼 `DecoderHttpMessageReader` 는  
    JSON 바디(`application/json`) 를 `MyDto` 로 역직렬화.
    
- `StringDecoder` 를 감싼 `StringHttpMessageReader` 는  
    텍스트 바디를 `String` 으로 변환.
    

이 Reader/Decoder 조합들이 **바로 앞에서 너가 읽은 “Message Reader” 역할**을 하는 애들입니다.  
“partial message + full message 처리”를, HTTP/JSON/바이너리 포맷에 맞게 내부에서 처리.[Spring Docs+1](https://docs.spring.io/spring-framework/docs/5.3.6/reference/html/web-reactive.html)

##### 3-2. 컨트롤러 시그니처가 어떤 Reader를 쓸지 결정

예를 들어:

`@PostMapping("/users") public Mono<User> createUser(@RequestBody Mono<User> body) { ... }`

- 여기서 `@RequestBody Mono<User>` 를 보고,
    
- WebFlux는 “body를 User로 읽어야겠구나 → JSON이면 Jackson2JsonDecoder 쓰자” 라고 결정.
    
- 내부적으로 해당 `HttpMessageReader<User>` 를 골라서:
    

`Flux<DataBuffer> bodyBuffers = serverHttpRequest.getBody(); Mono<User> userMono = httpMessageReader.readMono(User.class, request, hints);`

같은 식으로 호출합니다. (실제 코드는 복잡하지만 개념은 이렇습니다)

그래서 **사용자 코드에서는 `bodyToMono(User.class)` 같은 한 줄로 끝나지만**,  
안쪽에서는 `Flux<DataBuffer>` 조각들을 계속 이어 붙여가며,  
full JSON이 모이는 시점에 Jackson으로 decode 해서 `User` 객체를 만들어내는 거죠.[Spring Docs+1](https://docs.spring.io/spring-framework/docs/5.3.6/reference/html/web-reactive.html?utm_source=chatgpt.com)

---

## 4. Reactor Netty ↔ WebFlux 어댑터: ReactorHttpHandlerAdapter

“Netty HTTP 서버 ↔ Spring WebFlux `HttpHandler`” 를 붙여주는 접착제 역할을 하는 게  
**`ReactorHttpHandlerAdapter`** 입니다.[Spring Docs+1](https://docs.spring.io/projectreactor/reactor-netty/docs/1.2.0-M3/reference/html/http-server.html?utm_source=chatgpt.com)

구조를 개념적인 의사 코드로 쓰면 대략:

`// Reactor Netty 쪽 HttpServer.create()     .handle((reactorRequest, reactorResponse) -> {         ServerHttpRequest springRequest =             new ReactorServerHttpRequest(reactorRequest, dataBufferFactory);         ServerHttpResponse springResponse =             new ReactorServerHttpResponse(reactorResponse, dataBufferFactory);          // WebFlux HttpHandler에게 넘김         return httpHandler.handle(springRequest, springResponse);     })     .bindNow();`

여기서:

- `reactorRequest` 가 내부적으로 Netty의 `HttpServerRequest` 를 감싸고 있고,
    
- `springRequest.getBody()` 를 부르면 `Flux<DataBuffer>` 가 흘러나옵니다.
    
- 이 DataBuffer는 내부에서 `ByteBuf` 에서 온 것.
    

즉 “Netty → Reactor Netty → DataBuffer → HttpMessageReader → Mono/Flux<T>” 까지를  
`ReactorHttpHandlerAdapter + ServerHttpRequest/Response + Codec/Reader` 들이 처리합니다.[Spring Docs+1](https://docs.spring.io/spring-framework/docs/5.3.6/reference/html/web-reactive.html?utm_source=chatgpt.com)

---

## 5. 앞에서 말한 “Message Reader”와 WebFlux 비교

우리가 처음 읽었던 설명에서는:

- **MessageReader**
    
    - Channel당 하나
        
    - 바이트 버퍼 accumulate
        
    - full message 탐지 + partial message 버퍼링
        
    - 프로토콜에 따라 구현 교체 가능 (factory 주입)
        

WebFlux에서는 이 역할이 쪼개져 있습니다:

|예전 설명의 개념|WebFlux/Netty에서 실제 클래스|
|---|---|
|Channel 당 Reader|Netty의 ChannelPipeline + Reactor Netty inbound|
|ByteBuf 버퍼링/조립|Netty + Reactor Netty가 책임|
|Message Reader|`HttpMessageReader<T>` + `Decoder<T>` 구현들|
|프로토콜 별 구현 교체|`CodecConfigurer` / 커스텀 `HttpMessageReader`, `Decoder` 등록|

그래서 WebFlux에서 “Message Reader를 직접 짜야 하나?” →  
일반적인 HTTP/JSON/웹소켓/폼/multipart 상황에서는 **이미 다 구현돼 있어서 거의 안 짠다**가 답입니다.

진짜로 특수한 바이너리 프로토콜을 WebFlux 위에서 처리하고 싶을 때만  
직접 `Decoder` / `HttpMessageReader` 를 구현해서 `WebFluxConfigurer#configureHttpMessageCodecs` 로 등록하는 식으로 “플러그인” 합니다.[Spring Docs+1](https://docs.spring.io/spring-framework/docs/5.3.6/reference/html/web-reactive.html?utm_source=chatgpt.com)

---

## 6. 한 줄 요약

- Netty가 **ByteBuf** 조각들을 모아서 HTTP 레벨로 끌어올리고,
    
- Spring이 그걸 **DataBuffer** 로 추상화해서 `Flux<DataBuffer>` 로 노출하고,
    
- WebFlux의 **HttpMessageReader + Decoder** 가 그걸 **Mono/Flux<T>** 로 바꿔줘서,
    
- 컨트롤러에서는 그냥 `Mono<User>` / `Flux<User>` 만 받으면 된다.
    

---

혹시 이걸 **직접 눈으로 보고 싶은 욕구**가 들면,  
다음 순서로 소스를 타고 들어가 보면 재미있습니다:

1. `ReactorHttpHandlerAdapter` (Spring WebFlux)
    
2. `ReactorServerHttpRequest` / `ReactorServerHttpResponse`
    
3. `AbstractMessageReaderArgumentResolver` (메서드 인자 바인딩)
    
4. `CodecConfigurer` + `Jackson2JsonDecoder` / `DecoderHttpMessageReader`
    

원하면, “bodyToMono(User.class)” 한 줄이 실제로 어떤 call stack을 타는지  
IDE에서 타고 들어가는 순서대로 정리해서 보여줄 수도 있어요.