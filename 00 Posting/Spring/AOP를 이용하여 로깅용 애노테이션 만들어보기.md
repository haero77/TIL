- 개요
- 애노테이션 구현
- 클래스 레벨에서 사용하기
- 마치며

>  예제 코드는 [여기](https://github.com/haero77/labs/blob/main/src/main/java/com/labs/hello/global/aop/log/LogRequestBodyAspect.java)서 확인하실 수 있습니다.


# 개요

현재 팀에서 서비스 중인 애플리케이션에서는 별도의 HTTP Request, Response 정보를 로깅하지 않고 있습니다. 프로덕션 코드에서 로깅을 하는 것이 아닌 데이터독을 통해 HTTP Request, Response 를 로깅하고 있습니다. 

문제는 `HTTP Method & Status code`, `URI`, `Path Parmeter`, `Query Parameter` 만 로깅하고, Request 와 Response 의 `Body` 는 로깅하고 있지 않아, 실제 에러가 나는 경우(주로 `POST` 요청) 원인을 추적하기 어려운 상태였습니다. 에러가 나지 않아도, 의도한대로 ResponseBody가 응답되는 것을 확인할 수 없어 운영 상에도 답답한 감이 있었습니다.

기존의 설계가 이렇게 된 경위를 파악해보니, 비용의 문제였습니다. 서비스 특성상 ResponseBody의 길이가 매우 길었고, 이를 데이터독에서 모두 관리할 경우 말 그대로 돈이 많이 들기 때문에 `Body` 를 로깅에서 제외해주게 된 것이었죠. 

운영상의 편의와 데이터독 지불 비용은 트레이드 오프 관계이지만, 현재로서 운영의 불편함을 겪는 것이 더 큰 비용이었습니다. 따라서 필요한 API에서 `Body` 정보만 추가적으로 로깅할 수 있도록 애노테이션을 개발할 니즈가 생겼고, AOP를 통해 구현하기로 했습니다. 


# 애노테이션 구현

## 설계

![](attachments/Pasted%20image%2020231202191112.png)

바로 코드로 구현하기 전에, 어느 시점에 어드바이스가 적용되어야할지 간단히 설계해봤습니다. 

RequestBody 의 경우 컨트롤러 메서드가 호출되기 전 로깅하도록 했습니다. 에러가 날 경우 RequsetBody 로그가 남는다면 에러 경위를 파악하기 쉬울 뿐더러, 본격적으로 비즈니스 로직이 시작되기 전에 요청이 어떻게 들어왔는지 보여주는 것이 자연스럽다고 생각했습니다. 

Response 로깅의 경우, 컨트롤러 메서드 이후 로직이 모두 완료된 후 로깅하도록 했습니다. 에러가 날 경우 에러 메시지는 기존에 잘 로깅되고 있었으므로, 정상적인 흐름에 대해서만 로깅하도록 했습니다.


<br>

## @LogRequestBody

### 애노테이션 선언

먼저 사용할 애노테이션을 선언해보겠습니다. 

```java
@Target(value = {ElementType.METHOD, ElementType.TYPE})  
@Retention(RetentionPolicy.RUNTIME)  
public @interface LogRequestBody {  
  
}
```

메서드와 클래스 레벨에서 모두 적용할 수 있도록 `@Target`의 value를 `value = {ElementType.METHOD, ElementType.TYPE}` 으로 선언했습니다. 

어노테이션 유지 정책(Retention)의 경우, 컴파일된 바이트코드에서 유지되고 실행 시 JVM에서 사용가능한  `RetentionPolicy.RUNTIME`를 사용했습니다. (일반적으로 많이 사용되는 방법이기도 합니다.)
<br>

### 애스펙트 선언

```java
@Slf4j  
@Component  
@Aspect  
public class LogRequestBodyAspect {  
  
    private final ObjectMapper objectMapper;  
  
    public LogRequestBodyAspect(ObjectMapper objectMapper) {  
        this.objectMapper = objectMapper;  
    }  
  
    @Pointcut("@within(com.labs.hello.global.aop.log.LogRequestBody) || "  
            + "@annotation(com.labs.hello.global.aop.log.LogRequestBody)")  
    public void hasLogRequestBody() {  
    }  
    
    @Before("hasLogRequestBody() && args(.., org.springframework.web.bind.annotation.RequestBody bodyParam)")  
    public void doLogRequestBody(JoinPoint joinPoint, Object bodyParam) throws JsonProcessingException {  
        log.info("[doLogRequestBody] {}" + System.lineSeparator() + "{}",  
                joinPoint.getSignature(),  
                JsonPrettyFormatter.format(this.objectMapper, bodyParam));  
    }  
  
}
```

`@LogRequestBody` 애노테이션을 이용하여 AOP를 적용하기 위한 애스펙트를 만들었습니다.

먼저 포인트컷 지시자를 이용하여 조인 포인트를 정합니다. 지시자 `@within`을 통해 `@LogRequestBody`가 붙여진 클래스의 각 메서드마다 AOP가 적용되게 하였고(클래스 단위), `@annotation`을 이용하여 `@LogRequestBody`가 붙여진 메서드마다 AOP가 적용되게 했습니다(메서드 단위). 

조인 포인트를 지정하는 것은 여기서 끝이 아닙니다. 조인 포인트를 더 좁힐 필요가 있습니다. 우리의 목적은 `@RequestBody`가 붙여진 파라미터를 로깅하는 것이므로, 인자로 `@RequestBody`를 갖는 메서드로 조인 포인트를 좁혀야 하죠.

- `args(.., org.springframework.web.bind.annotation.RequestBody bodyParam)`

`args`를 이용하여, `@RequestBody`를 가진 파라미터가 있는 메서드로 조인 포인트를 좁혔고, 동시에 매개변수를 받아오도록 했습니다(`bodyParam`). 이제 출력해야할 데이터가 주어졌으니, 이를 예쁘게 로그로 남겨봅시다.

<br>
### Json String Pretty 하게 출력하기

```java
@Before("hasLogRequestBody() && args(.., org.springframework.web.bind.annotation.RequestBody bodyParam)")  
public void doLogRequestBody(JoinPoint joinPoint, Object bodyParam) throws JsonProcessingException {  
	log.info("[doLogRequestBody] {}" + System.lineSeparator() + "{}",  
			joinPoint.getSignature(),  
			JsonPrettyFormatter.format(this.objectMapper, bodyParam));  
}  
```


조인 포인트가 정해졌으므로 본격적으로 어드바이스(로깅)을 구현해보겠습니다. 
먼저, `joinPoint.getSignature()` 를 로그로 남김으로써, 어느 메서드에서 로깅이 이뤄졌는지 확인합니다.
남은 것은 `@RequestBody`를 가진 파라미터인 `bodyParam` 을 예쁘게 출력하기만 하면되므로, 별도의 유틸 클래스를 만들어서 이 작업을 해봤습니다.

```java
@Slf4j  
public class JsonPrettyFormatter {  
  
    private JsonPrettyFormatter() {  
    }  
    
    public static String format(ObjectMapper objectMapper, Object source) throws JsonProcessingException {  
        try {  
            return objectMapper.writerWithDefaultPrettyPrinter().writeValueAsString(source);  
        } catch (JsonProcessingException e) {  
            log.info("Error occurred while converting {}", source.toString());  
            throw e;  
        }  
    }  
  
}
```

`objectMapper.writerWithDefaultPrettyPrinter()`를 이용하면 JSON을 Pretty하게 출력할 수 있습니다. 정적 유틸 클래스에서 자체적으로 `ObjectMapper`를 주입받기는 어려우므로, 외부에서 의존성을 넣어주는 방법을 택했습니다.

<br>
### @LogRequestBody 적용 결과

만든 애노테이션이 정상적으로 동작하는지 확인하기 위해서, 간단한 컨트롤러를 만들었습니다.

```java
@LogRequestBody  
@RestController  
public class HelloApi {  
  
    @GetMapping("/api/hello/requestDto")  
    public ResponseEntity<String> requestDto(@RequestBody HelloRequest request) {  
        return ResponseEntity.ok()  
                .body("String");  
    }  
  
    @GetMapping("/api/hello/map")  
    public ResponseEntity<Map<String, Integer>> map(@RequestBody Map<String, String> body) {  
        return ResponseEntity.ok()  
                .body(Map.of("Response", 123));  
    }  
  
  
    @Getter  
    public static class HelloRequest {  
  
        private int id;  
  
    }  
  
    @Getter  
    public static class HelloResponse {  
  
        private final int id;  
  
        private HelloResponse(int id) {  
            this.id = id;  
        }  
  
    }  
  
}
```

별도의 Request DTO 없이 `@RequestBody Map<String, String> body`처럼 파라미터를 구성한 것이 조금 의아하실 수도 있을 것 같습니다. 현재 레거시 코드에는 Request DTO 외에도, 위처럼 Map으로 받는 경우가 많으므로, 다양한 케이스에서 우리가 만든 애노테이션을 적용시킬 수 있다는 것을 보여드리기 위해 위처럼 예제를 구성했습니다. 

잠깐 샛길로 새서, Request DTO와 `Map` 사용에 대해 간단히 제 의견을 남겨보자면, Request DTO 클래스가 많아진다고 해도 `Map` 보다 나은 선택지라고 생각합니다. 이름이 없어서 의도를 유추를 해야하는 것과, 명시적으로 어떤 용도의 파라미터인지 타입 이름만 보고 알아차리는 것은 가독성에서 많이 차이나며, 이는 곧 유지보수 비용의 차이로 직결되기 때문입니다. 팀원들 역시 이 부분에 대하여 충분히 인지하고 공감하는 관계로, 약간의 여유가 있을 때면 위 같은 `Map` 을 Request DTO 클래스로 대체하며 개선해나가고 있습니다.

다시 본론으로 돌아와서, 만든 2개의 API를 호출해서 제대로 로깅이 찍히는지 확인해보겠습니다. API 호출은 [IntelliJ의 .http](https://jojoldu.tistory.com/266)를 이용하여 진행했습니다.

![](attachments/Pasted%20image%2020231205212458.png)

```java
2023-12-05 21:23:32.673  INFO 58351 --- [nio-8081-exec-5] c.l.h.g.aop.log.LogRequestBodyAspect     : [doLogRequestBody] ResponseEntity com.labs.hello.domain.hello.api.HelloApi.requestDto(HelloRequest)
{
  "id" : 1
}
2023-12-05 21:23:33.389  INFO 58351 --- [nio-8081-exec-7] c.l.h.g.aop.log.LogRequestBodyAspect     : [doLogRequestBody] ResponseEntity com.labs.hello.domain.hello.api.HelloApi.map(Map)
{
  "abc" : "123"
}
```

콘솔창에 남겨진 로그를 보면, 조인 포인트의 메서드 시그니처와 함께 Request Body로 넘어온 데이터가 파라미터의 타입에 관계없이 JSON 형태로 Pretty 하게 찍히는 것을 확인할 수 있습니다. 
<br>

## LogResponseBody

### 애노테이션 선언

```java
@Target(value={ElementType.METHOD, ElementType.TYPE})  
@Retention(RetentionPolicy.RUNTIME)  
public @interface LogResponseBody {  
  
}
```

`@LogResponseBody` 의 구현은 `@LogRequestBody` 보다 더 간단합니다. 애노테이션을 적용할 애스펙트를 만들어보죠.

<br>

### 애스펙트 선언

```java
@Slf4j  
@Component  
@Aspect  
public class LogResponseBodyAspect {  
  
    private final ObjectMapper objectMapper;  
  
    public LogResponseBodyAspect(ObjectMapper objectMapper) {  
        this.objectMapper = objectMapper;  
    }  
  
    @Pointcut("@within(com.labs.hello.global.aop.log.LogResponseBody) || "  
            + "@annotation(com.labs.hello.global.aop.log.LogResponseBody)")  
    public void hasLogResponseBody() {  
    }  
    
    @AfterReturning(value = "hasLogResponseBody()", returning = "responseBody")  
    public void doLogRequestBody(JoinPoint joinPoint, Object responseBody) throws JsonProcessingException {  
        log.info("[doLogResponseBody] {}" + System.lineSeparator() + "{}",  
                joinPoint.getSignature(),  
                JsonPrettyFormatter.format(this.objectMapper, responseBody));  
    }  
  
}
```

`LogRequestBodyAspect`처럼, `@within`과 `@annotation` 포인트컷 지시자를 이용하여 조인 포인트를 좁혀줍니다. 타겟은 `@LogResponseBody` 애노테이션을 가진 클래스에 속하는 메서드 또는 해당 애노테이션을 가진 메서드입니다. 

어드바이스 종류는 `@AfterReturning`를 택했습니다. 타겟이 정상적으로 호출되고 나서,



<br>
### 적용 결과

```java
@LogRequestBody  
@LogResponseBody // 추가
@RestController  
public class HelloApi {

	...
	
}
```


*(이 부분은 추가적으로 공부하고 보완해보겠습니다.)*


# 3. 마치며


### 참고 

- https://stackoverflow.com/questions/63008446/get-requestbody-using-aspectj
- [아무 관심 없던 @Retention 어노테이션 정리(RetentionPolicy SOURCE vs CLASS vs RUNTIME)](https://jeong-pro.tistory.com/234)