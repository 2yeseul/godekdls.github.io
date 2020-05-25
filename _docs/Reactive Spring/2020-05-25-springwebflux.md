---
title: Spring WebFlux
category: Reactive Spring
order: 2
permalink: /Reactive%20Spring/springwebflux/
---

> [리액티브 스프링 공식 reference](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux)를 한글로 번역한 문서입니다.
>
> 전체 목차는 [여기](https://godekdls.github.io/Reactive%20Spring/contents/)에 있습니다.

### 목차

- [1.1. Overview](#11-overview)
  + [1.1.1. Define “Reactive”](#111-define-reactive)
  + [1.1.2. Reactive API](#112-reactive-api)
  + [1.1.3. Programming Models](#113-programming-models)
  + [1.1.4. Applicability](#114-applicability)
  + [1.1.5. Servers](#115-servers)
  + [1.1.6. Performance](#116-performance)
  + [1.1.7. Concurrency Model](#117-concurrency-model)
- [1.2. Reactive Core](#12-reactive-core)
  + [1.2.1. HttpHandler](#121-httphandler)
  + [1.2.2. WebHandler API](#122-webhandler-api)
  + [1.2.3. Filters](#123-filters)
  + [1.2.4. Exceptions](#124-exceptions)
  + [1.2.5. Codecs](#125-codecs)
  + [1.2.6. Logging](#126-logging)
- [1.3. DispatcherHandler](#13-dispatcherhandler)
  + [1.3.1. Special Bean Types](#131-special-bean-types)
  + [1.3.2. WebFlux Config](#132-webflux-config)
  + [1.3.3. Processing](#133-processing)
  + [1.3.4. Result Handling](#134-result-handling)
  + [1.3.5. Exceptions](#135-exceptions)
  + [1.3.6. View Resolution](#136-view-resolution)
- [1.4. Annotated Controllers](#14-annotated-controllers)
  + [1.4.1. @Controller](#141-controller)
  + [1.4.2. Request Mapping](#142-request-mapping)
  + [1.4.3. Handler Methods](#143-handler-methods)
  + [1.4.4. Model](#144-model)
  + [1.4.5. DataBinder](#145-databinder)
  + [1.4.6. Managing Exceptions](#146-managing-exceptions)
  + [1.4.7. Controller Advice](#147-controller-advice)
- [1.5. Functional Endpoints](#15-functional-endpoints)
  + [1.5.1. Overview](#151-overview)
  + [1.5.2. HandlerFunction](#152-handlerfunction)
  + [1.5.3. RouterFunction](#153-routerfunction)
  + [1.5.4. Running a Server](#154-running-a-server)
  + [1.5.5. Filtering Handler Functions](#155-filtering-handler-functions)
- [1.6. URI Links](#16-uri-links)
  + [1.6.1. UriComponents](#161-uricomponents)
  + [1.6.2. UriBuilder](#162-uribuilder)
  + [1.6.3. URI Encoding](#163-uri-encoding)
- [1.7. CORS](#17-cors)
  + [1.7.1. Introduction](#171-introduction)
  + [1.7.2. Processing](#172-processing)
  + [1.7.3. @CrossOrigin](#173-crossorigin)
  + [1.7.4. Global Configuration](#174-global-configuration)
  + [1.7.5. CORS WebFilter](#175-cors-webfilter)
- [1.8. Web Security](#18-web-security)
- [1.9. View Technologies](#19-view-technologies)
  + [1.9.1. Thymeleaf](#191-thymeleaf)
  + [1.9.2. FreeMarker](#192-freemarker)
  + [1.9.3. Script Views](#193-script-views)
  + [1.9.4. JSON and XML](#194-json-and-xml)
- [1.10. HTTP Caching](#110-http-caching)
  + [1.10.1. CacheControl](#1101-cachecontrol)
  + [1.10.2. Controllers](#1102-controllers)
  + [1.10.3. Static Resources](#1103-static-resources)
- [1.11. WebFlux Config](#111-webflux-config)
  + [1.11.1. Enabling WebFlux Config](#1111-enabling-webflux-config)
  + [1.11.2. WebFlux config API](#1112-webflux-config-api)
  + [1.11.3. Conversion, formatting](#1113-conversion-formatting)
  + [1.11.4. Validation](#1114-validation)
  + [1.11.5. Content Type Resolvers](#1115-content-type-resolvers)
  + [1.11.6. HTTP message codecs](#1116-http-message-codecs)
  + [1.11.7. View Resolvers](#1117-view-resolvers)
  + [1.11.8. Static Resources](#1118-static-resources)
  + [1.11.9. Path Matching](#1119-path-matching)
  + [1.11.10. Advanced Configuration Mode](#11110-advanced-configuration-mode)
- [1.12. HTTP/2](#112-http2)

스프링 프레임워크, 스프링 웹 MVC를 포함한 기존 웹 프레임워크는
서블릿 API와 서블릿 컨테이너를 위해 개발됐다.
리액티브 스택 웹 프레임워크인 스프링 웹플럭스는 5.0 버전에 추가됐다.
완전하게 논블로킹으로 동작하며, 
[Reactive Streams](https://www.reactive-streams.org/)
back pressure를 지원하고,
Netty, Undertow, 서블릿 3.1+ 컨테이너 서버에서 동작한다.

두 웹 프레임워크 모두 소스 모듈 이름과 동일하며
([spring-webmvc](https://github.com/spring-projects/spring-framework/tree/master/spring-webmvc),
[spring-webflux](https://github.com/spring-projects/spring-framework/tree/master/spring-webflux)),
스프림 프레임워크에 공존한다.
원하는 모듈을 선택하면 된다.
둘 중 하나를 사용해 어플리케이션을 개발할 수 있고,
둘 다 사용할 수도 있다(예를 들어, 리액티브 `WebClient`를 사용하는 스프링 MVC 컨트롤러). 

## 1.1. Overview

스프링 웹플럭스는 왜 만들었나?

웹플럭스가 탄생한 이유 중 하나는
적은 쓰레드로 동시 처리를 제어하고 적은 하드웨어 리소스로 확장하기 위해
논블로킹 웹 스택이 필요했기 때문이다.
이전에도 서블릿 3.1은 논블로킹 I/O를 위한 API를 제공했다.
하지만 서블릿으로 논블로킹을 구현하려면 다른
동기 처리나(`Filter`, `Servlet`) 블로킹 방식(`getParameter`, `getPart`)을 쓰는
API를 사용하기 어렵다.
이런 점때문에 어떤 논블로킹 런타임과도 잘 동작하는 새 공통 API를 만들게 됐다.
이미 비동기 논블로킹 환경에서 자리를 잡은 서버(e.g. Netty)때문에라도 새 API가 필요했다.

또 다른 이유는 함수형 프로그래밍이다.
자바 5의 애노테이션 등장으로 선택의 폭이 넓어진 것처럼
(애노테이션을 선언한 REST 컨트롤러나 유닛 테스트 등),
자바 8에서 추가된 람다 표현식덕분에 자바에서도 함수형 API를 작성할 수 있게 됐다.
이 기능은 논블로킹 어플리케이션을 만들 때도 요긴하게 쓰이며,
이제는 continuation-style API(`CompletableFuture`와 [ReactiveX](http://reactivex.io/)로 대중화된)로
비동기 로직을 선언적으로 작성할 수 있다.
프로그래밍 모델 관점에서 보면, 
애노테이션을 선언한 컨트롤러와 웹플럭스를 웹 엔드포인트로 사용할 수 있는 건 자바 8덕분이다.

### 1.1.1. Define “Reactive”

이제 "논블로킹"과 "함수형"이 뭔지 알게 되었는데, 그러면 리액티브는 무슨 뜻일까?

"리액티브"라는 용어는 변화에 반응하는 것을 중심에 두고 만든 프로그래밍 모델을 의미한다
(I/O 이벤트에 반응하는 네트워크 컴포넌트, 마우스 이벤트에 반응하는 UI 컨트롤러 등).
논블로킹은 작업을 기다리기보단 완료되거나 데이터를 사용할 수 있게 되면 반응하므로,
이 말대로면 논블로킹도 리액티브다.

스프링은 "리액티브"와 관련한 중요한 메커니즘이 하나 더 있는데, 논블로킹 back pressure다.
동기식 명령형(imperative) 코드에서 블로킹 호출은 호출자를 강제로 기다리게 하는 일종의 back pressure다.
논블로킹 코드에선, 
프로듀셔 속도가 컨슈머 속도를 압도하지 않도록 이벤트 속도를 제어한다.

리액티브 스트림은 back pressure를 통한 비동기 컴포넌트간의 상호작용을 정의한
[간단한 스펙](https://github.com/reactive-streams/reactive-streams-jvm/blob/master/README.md#specification)이다
(자바 9에서도 채택했다). 
예를 들어 데이터 레포지토리([Publisher](https://www.reactive-streams.org/reactive-streams-1.0.1-javadoc/org/reactivestreams/Publisher.html) 역할)가
데이터를 만들고, HTTP 서버([Subscriber](https://www.reactive-streams.org/reactive-streams-1.0.1-javadoc/org/reactivestreams/Subscriber.html) 역할)로
이 데이터로 요청을 처리할 수 있다.
리액티브 스트림을 쓰는 주목적은 subscriber가 publisher의 데이터 생산 속도를 제어하는 것이다.

> **자주 묻는 질문: publisher 속도를 늦출 수 없으면 어떻게 할까?**
>
> 리액티브 스트림의 목적은 메커니즘과 경계를 확립하는 것이다.
> publisher가 속도를 늦출 수 없다면 버퍼에 담을지, 데이터를 날릴지,
> 실패로 처리할지 결정해야 한다.

### 1.1.2. Reactive API

리액티브 스트림은 컴포넌트 상호 작용에서 중요한 역할을 한다.
하지만 이건 라이브러리와 기반 구조에 사용되는 컴포넌트엔 유용해도,
어플리케이션 API에서 다루기엔 너무 저수준이다.
어플리케이션은 비동기 로직을 만들기 위한 풍부한 고수준 함수형 API가 필요하다.
(자바 8 스트림 API와 비슷하지만 collection만을 위한게 아니다).
이게 바로 리액티브 라이브러리가 하는 일이다.

[Reactor](https://github.com/reactor/reactor)는
스프링 웹플럭스가 선택한 리액티브 라이브러리다.
리액터는 [`Mono`](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html)와
[`Flux`](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html)
API 타입을 제공한다.
ReactiveX [vocabulary of operators](http://reactivex.io/documentation/operators.html)에 정리된
풍부한 연산자를 사용해 데이터 시퀀스를 0~1개는 `Mono`, 0~N개는 `Flux`로 표현할 수 있다.
리액터는 리액티브 스트림 라이브러리이기 때문에
모든 연산자는 논블로킹 back pressure를 지원한다.
리액터는 특히 서버 사이드 자바에 초점을 두고 스프링과 긴밀히 협력해서 개발됐다.

웹플럭스는 리액터를 핵심 라이브러리로 사용하지만,
다른 리액티브 라이브러리를 써도 리액티브 스트림으로 상호작용 할 수 있다.
웹플럭스 API의 일반적인 룰은,
순수한 `Publisher`를 입력으로 받아 내부적으로 리액터 타입으로 맞추고,
이걸 사용해서 `Flux`나 `Mono`를 반환한다.
따라서 어떤 `Publisher`든 입력으로 전달하고 연산할 수 있지만,
다른 리액티브 라이브러리를 사용하려면 출력 형식을 맞춰줘야 한다.
웹플럭스는 가능만 하다면(e.g. 애노테이션을 선언한 컨트롤러)
투명한 방식으로 RxJava나 다른 리액티브 라이브러리에 맞게 바꿔준다. 
자세한 내용은 [Reactive Libraries](https://godekdls.github.io/Reactive%20Spring/reactivelibraries/)를 참고하라.

> 리액티브 API와는 별개로 
> WebFlux는 코틀린의 [코루틴](https://godekdls.github.io/Reactive%20Spring/coroutines/)
> API와도 사용할 수 있는데,
> 이를 사용하면 좀 더 명령적(imperative)인 프로그래밍이 가능하다.
> 아래 나오는 코틀린 코드 샘플은 코루틴 API를 사용할 것이다.

### 1.1.3. Programming Models

`spring-web` 모듈에 있는 웹플럭스는,
여러 서버를 지원하기위한 
HTTP 추상화와 리액티브 스트림 [아답터](#121-httphandler),
[코덱](#125-codecs),
Servlet API에 상응하는 코어 [`WebHandler` API](#122-webhandler-api)를
아우르는 개념이며, 이는 모두 논블로킹이다.

스프링 웹플럭스는 두 가지 프로그래밍 모델을 지원한다:

- [Annotated Controllers](#14-annotated-controllers):
스프링 MVC와 동일하며 `spring-web` 모듈에 있는 같은 애노테이션을 사용한다.
스프링 MVC와 웹플럭스 컨트롤러 모두 리액티브(Reactor, RxJava) 리턴 타입을
지원하기때문에 이 둘을 구분하기 어렵다.
한 가지 눈에 띄는 차이는 웹플럭스에선
`@RequestBody`로 리액티브 인자를 받을 수 있다는 것이다.
- [Functional Endpoints](#15-functional-endpoints):
경령화된 람다 기반 함수형 프로그래밍 모델.
요청을 라우팅해주는 조그만한 라이브러리나 유틸리티 모음이라고 생각하면 된다.
annotated controller와 다른 점은
애노테이션으로 의도를 선언해서 콜백받기보단
어플리케이션이 처음부터 끝까지 다 요청을 제어한다는 것이다.

### 1.1.4. Applicability

스프링 MVC냐 웹플럭스냐?

많이들 하는 질문이지만 이분법적 사고는 좋지 않다.
둘 모두 사용해서 선택의 폭을 넓힌다고 보는게 맞다.
이 둘은 지속성과 일관성을 위해 설계했으며,
함께 사용할 수 있고, 각자의 피드백이 서로에게 도움이 된다.
다음은 둘은 어떤 관련이 있는지,
공통점이 무엇이고 한쪽에서만 지원하는 게 무엇인지 나타낸 다이어그램이다:

![Spring MVC VS WebFlux](./../../images/reactivespring/spring-mvc-and-webflux-venn.png)

먼저 다음 제안을 고려해 보라:

- 이미 잘 동작하고 있는 스프링 MVC 어플리케이션이 있다면, 굳이 바꿀 필요 없다.
명령적(Imperative) 프로그래밍은 작성하기도, 이해하기도, 디버깅하기도 가장 쉽다.
지금까지 대부분이 블로킹 방식을 사용했기 때문에,
사용할 수 있는 라이브러리가 가장 풍부하다.
- 이미 논블로킹 웹 스택을 알아보고 있다면,
스프링 웹플럭스는 다른 웹 스택과 같은 실행 환경을 제공하면서도,
다양한 서버(Netty, Tomcat, Jetty, Undertow, 서블릿 3.1+ 컨테이너)와
여러 리액티브 라이브러리(리액터, JxJava 등)를 지원하며,
두 가지 프로그래밍 모델(애노테이션을 선언한 컨트롤러와 함수형 웹 엔드포인트)을 사용할 수 있다.
- 자바 8 람다나 코틀린으로 개발할 수 있는
경량의 함수형 웹 프레임워크를 찾고 있다면,
스프링 웹플럭스의 함수형 웹 엔드포인트를 사용하면 된다.
로직을 투명하게 제어할 수 있기 때문에
요구사항이 덜 복잡한 소규모 어플리케이션이나 마이크로서비스에서도
좋은 선택이 될 것이다.
- 마이크로 아키텍처에선 스프링 MVC로 만든 어플리케이션과,
스프링 웹플럭스 컨트롤러나 함수형 엔트포인트를 사용한
어플리케이션을 조합할 수 있다.
두 프레임워크 모두 애노테이션 기반 프로그래밍 모델을 지원하기 때문에
새로 학습할 필요 없이 각자에 맞는 툴을 선택할 수 있다.
- 간단하게는 어플리케이션 의존성(dependency)을 확인해봐도 좋다.
블로킹 방식의 영속성 API(JPA, JDBC)나 네트워크 API를 사용하고 있다면
스프링 MVC가 최소한 아키텍처를 통일할 수 있으므로 가장 좋은 선택이다.
리액터나 RxJava로도 각 쓰레드에서 블로킹 API를 호출할 수 있지만,
이렇게 하면 논블로킹 웹 스택을 거의 활용하기 어렵다.
- 스프링 MVC 어플리케이션에서 외부 서비스를 호출한다면
한번 리액티브 `WebClient`를 사용해봐라.
스프링 MVC 컨트롤러 메소드에서도
리액티브 타입(Reactor, RxJava나 [그 외](https://godekdls.github.io/Reactive%20Spring/reactivelibraries))을
반환할 수 있다.
서비스 호출에 지연이 있거나 여러 서비스가 엮여 있는 API라면
효과가 더 좋을 것이다.
물론 스프링 MVC 컨트롤러로 다른 리액티브 컴포넌트를 호출할 수도 있다.
- 팀 규모가 크다면 논블로킹, 함수형, 선언적 프로그래밍은
러닝커브가 높다는 점도 고려해야 한다.
한 번에 전환하지 않고 리액티브 `WebClient`부터 적용해보는 것도 좋은 방법이다.
작은 것부터 시작해서 변화가 있는지 확인해 봐라.
굳이 전환할 필요가 없는 경우도 많을 것이다.
어떤 변화를 확인해야 할지 감이 오지 않는다면,
논블로킹 I/O 동작 방식과 효과를 학습하는 것부터 시작해라
(예를 들어 싱글 쓰레드 기반 Node.js의 동시 처리).

### 1.1.5. Servers

스프링 웹플럭스는 톰캣, Jetty, 서블릿 3.1+ 컨테이너에서도,
서블릿 기반이 아닌 Netty나 Undertow에서도 잘 동작한다.
저수준 [공통 API](#121-httphandler)로 서버를 추상화하기 때문에
모든 서버에
고수준 [프로그래밍 모델](#113-programming-models)을 적용할 수 있다.

스프링 웹플럭스엔 서버 기동이나 중단을 위한 내장 기능은 없다.
하지만 스프링 설정과 [웹플럭스 구조](#111-webflux-config)를
[조립](#122-webhandler-api)해
적은 코드로 손쉽게 어플리케이션을 [실행](#121-httphandler)할 수 있다.

스프링 부트에선 웹플럭스 스타터가 이 단계를 자동화해준다.
스타터는 기본으로 Netty를 사용하지만,
메이븐이나 그래들 디펜던시만 수정하면 톰캣이나 Jetty, Undertow로 쉽게 교체할 수 있다.
스프링 부트가 Netty를 디폴트로 사용하는 이유는
보통 비동기 논블로킹에 많이 사용하기도 하고,
클라이언트와 서버가 리소스를 공유할 수 있어서다.

톰캣과 Jetty는 스프링 MVC, 웹플럭스 모두 사용할 수 있다.
하지만 동작 방식이 다르다는 점에 주의하라.
스프링 MVC는 서블릿의 블로킹 I/O를 사용하며,
어플리케이션에서 필요하면 서블릿 API를 직접 사용할 수 있다.
스프링 웹플럭스는 서블릿 3.1 논블로킹 I/O로 동작하며,
서블릿 API는 저수준 아답터에서 사용하기 때문에 노출돼 있지 않다.

스프링 웹플럭스에서 Undertow를 사용할 때는 서블릿 API가 아닌
Undertow API를 사용한다.

### 1.1.6. Performance

성능은 여러 의미로 해석할 수 있다.
리액티브랑 논블로킹을 사용한다고 해서 바로 어플리케이션이 빨라지는 건 아니다.
물론 빨라질 수도 있다
(예를 들어 WebClient를 사용해서 외부 서비스 호출을 병렬로 처리한다면).
전반적으로 보면 논블로킹 방식이 처리할 일이 더 많다보니
처리 시간이 약간 더 길어질 수 있다.

리액티브와 논블로킹의 주된 이점은
고정된 적은 쓰레드와 적은 메모리로도 확장할 수 있다는 것이다.
예측할 수 있는 방법으로 확장하기 때문에
부하 속에서도 어플리케이션 복원 능력은 더 좋아진다.
하지만 이를 확인하려면 약간의 대기 시간이 필요하다
(느리고 예측 불가능한 네트워크 I/O 시간을 포함해서).
바로 여기서 리액티브 스택이 강점을 드러내며, 그 차이는 엄청나다.

### 1.1.7. Concurrency Model

## 1.2. Reactive Core

### 1.2.1. HttpHandler
### 1.2.2. WebHandler API
### 1.2.3. Filters
### 1.2.4. Exceptions
### 1.2.5. Codecs
### 1.2.6. Logging

## 1.3. DispatcherHandler

### 1.3.1. Special Bean Types
### 1.3.2. WebFlux Config
### 1.3.3. Processing
### 1.3.4. Result Handling
### 1.3.5. Exceptions
### 1.3.6. View Resolution

## 1.4. Annotated Controllers

### 1.4.1. @Controller
### 1.4.2. Request Mapping
### 1.4.3. Handler Methods
### 1.4.4. Model
### 1.4.5. DataBinder
### 1.4.6. Managing Exceptions
### 1.4.7. Controller Advice

## 1.5. Functional Endpoints

### 1.5.1. Overview
### 1.5.2. HandlerFunction
### 1.5.3. RouterFunction
### 1.5.4. Running a Server
### 1.5.5. Filtering Handler Functions

## 1.6. URI Links

### 1.6.1. UriComponents
### 1.6.2. UriBuilder
### 1.6.3. URI Encoding

## 1.7. CORS

### 1.7.1. Introduction
### 1.7.2. Processing
### 1.7.3. @CrossOrigin
### 1.7.4. Global Configuration
### 1.7.5. CORS WebFilter

## 1.8. Web Security

## 1.9. View Technologies

### 1.9.1. Thymeleaf
### 1.9.2. FreeMarker
### 1.9.3. Script Views
### 1.9.4. JSON and XML

## 1.10. HTTP Caching

### 1.10.1. CacheControl
### 1.10.2. Controllers
### 1.10.3. Static Resources

## 1.11. WebFlux Config

### 1.11.1. Enabling WebFlux Config
### 1.11.2. WebFlux config API
### 1.11.3. Conversion, formatting
### 1.11.4. Validation
### 1.11.5. Content Type Resolvers
### 1.11.6. HTTP message codecs
### 1.11.7. View Resolvers
### 1.11.8. Static Resources
### 1.11.9. Path Matching
### 1.11.10. Advanced Configuration Mode

## 1.12. HTTP/2

> 전체 목차는 [여기](https://godekdls.github.io/Reactive%20Spring/contents/)에 있습니다.