
- [들어가며](#%EB%93%A4%EC%96%B4%EA%B0%80%EB%A9%B0)
- [본문](#%EB%B3%B8%EB%AC%B8)
	- [상황](#%EC%83%81%ED%99%A9)
	- [원인](#%EC%9B%90%EC%9D%B8)
		- [접두어 is 를 빼면?](#%EC%A0%91%EB%91%90%EC%96%B4%20is%20%EB%A5%BC%20%EB%B9%BC%EB%A9%B4?)
	- [방법 1: @JsonProperty (비추천)](#%EB%B0%A9%EB%B2%95%201:%20@JsonProperty%20(%EB%B9%84%EC%B6%94%EC%B2%9C))
	- [방법 2: boolean 대신 Boolean 사용하기 (비추천)](#%EB%B0%A9%EB%B2%95%202:%20boolean%20%EB%8C%80%EC%8B%A0%20%08Boolean%20%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0%20(%EB%B9%84%EC%B6%94%EC%B2%9C))
	- [방법 3: Enum 사용하기 (강력 추천)](#%EB%B0%A9%EB%B2%95%203:%20Enum%20%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0%20(%EA%B0%95%EB%A0%A5%20%EC%B6%94%EC%B2%9C))
- [마치며](#%EB%A7%88%EC%B9%98%EB%A9%B0)
		- [추가적으로 공부해볼 것](#%EC%B6%94%EA%B0%80%EC%A0%81%EC%9C%BC%EB%A1%9C%20%EA%B3%B5%EB%B6%80%ED%95%B4%EB%B3%BC%20%EA%B2%83)
- [참고](#%EC%B0%B8%EA%B3%A0)


#json #직렬화 #역직렬화 #RequestBody

# 들어가며 

API 스펙을 정의할 때 JSON 필드로 boolean 타입을 사용했다. 문제는 필드이름은 `isXXX` 형태로 작성했는데, `@RequestBody` 를 사용하는 컨트롤러 단계에서 제대로 역직렬화가 이루어지지 않아 필드값을 제대로 받지 못했다(`true` 보내는데 계속 `false` 로 남아있는 등). 문제를 해결하면서 결국에는 JSON 필드로 boolean 타입이 적절치 않다고 생각하게 되었는데, 왜 그런 생각이 들었는지 오류의 원인과 해결과정을 기록한다.


# 본문

## 상황

먼저 간단히 상황을 재연해보자.

```java
@Slf4j  
@RestController  
public class FooController {  
  
    @PostMapping("/api/members")  
    public ResponseEntity<Void> createMember(  
            @RequestBody MemberCreateRequest memberCreateRequest  
    ) {  
        log.info(memberCreateRequest.toString());  
  
        return ResponseEntity.noContent()  
                .build();  
    }  
  
}
```
```java
@Getter  
@ToString  
public class MemberCreateRequest {  
  
    private boolean isAdmin;  
    private String username;  
  
}
```

RestController 에서 `@RequestBody` 를 사용하여 Request DTO 를 받아오고 싶다.

![](attachments/Pasted%20image%2020231022184708.png)

그리고 API 를 위처럼 세팅하고 호출해보자. ([Intellij 의 .http 파일을 만들면 Postman 없이 편하게 API 호출이 가능하다.](https://jojoldu.tistory.com/266))

![](attachments/Pasted%20image%2020231022184836.png)

역직렬화된 `memberCreateRequest` 의 필드값을 보면 `isAdmin` 값이 `true` 가 아닌 `false`  로 되어있는 것을 볼 수 있다. 대체 뭐가 문제인 걸까.



## 원인

```java
public class MemberCreateRequest {  
    private boolean isAdmin;  
    private String username;  
  
    public MemberCreateRequest() {  
    }  
  
    public boolean isAdmin() {  
        return this.isAdmin;  
    }  
  
    public String getUsername() {  
        return this.username;  
    }  
  
    public String toString() {  
        boolean var10000 = this.isAdmin();  
        return "MemberCreateRequest(isAdmin=" + var10000 + ", username=" + this.getUsername() + ")";  
    }  
}
```

실제 컴파일된 `MemberCreateRequest` 의 바이트코드를 살펴보자(디컴파일은 Intellij 의 도움을 받았다).
lombok 의 `@Getter` 를 사용하면 `boolean` 필드의 getter 를 만들 때 `getXXX` 처럼 만드는게 아니라 `isXXX` 처럼 메서드명을 짓는다. 따라서 정상적인 `getXXX`  형태가 아니기 때문에 제대로 역직렬화가 되지 않는 것이다.

### 접두어 is 를 빼면?

여기서 잠깐, boolean 필드의 접두어 `is` 를 빼면 어떻게 될까?

```java
@Getter  
@ToString  
public class MemberCreateRequest {  
  
    private boolean admin;  
    private String username;  
  
}
```

`isAdmin` 이 아니라 `admin` 으로  바꾸고 다시 컴파일한 결과는!!

```java
public class MemberCreateRequest {  
    private boolean admin;  
    private String username;  
  
    public MemberCreateRequest() {  
    }  
  
    public boolean isAdmin() {  
        return this.admin;  
    }  
  
    public String getUsername() {  
        return this.username;  
    }  
  
    public String toString() {  
        boolean var10000 = this.isAdmin();  
        return "MemberCreateRequest(admin=" + var10000 + ", username=" + this.getUsername() + ")";  
    }  
}
```

역시나 `admin` 이 boolean 타입이기 때문에 getter가 `isXXX` 처럼 생긴다.

이 경우는

![](attachments/Pasted%20image%2020231022190152.png)
![](attachments/Pasted%20image%2020231022190212.png)

보다시피 역직렬화가 잘 되긴하지만, `admin` 필드명만 보고 boolean 타입인지 알아차리기는 쉽지 않을 것이다. 

그럼 다시 본론으로 돌아와서, 이 문제를 어떻게 해결하면 좋을까?


## 방법 1: @JsonProperty (비추천)

```java
@Getter  
@ToString  
public class MemberCreateRequest {  
  
    @JsonProperty("isAdmin")  
    private boolean isAdmin;  
    private String username;  
  
}
```

첫 번째 방법은 `@JsonProperty` 를 사용하는 것이다. 역직렬화도 잘되고, 직관적이기도 하다. 
그런데, 이 방법은 추천하지 않는다. 지금은 Request DTO 를 역직렬화를 하는 상황이라서 상관없지만, Response DTO 에서 `@JsonProperty` 를 사용할 때 문제가 된다.

```java
@Getter  
@ToString  
public class MemberCreateResponse {  
  
    @JsonProperty("isAdmin")  
    private boolean isAdmin;  
    private String username;  
  
    public MemberCreateResponse(boolean isAdmin, String username) {  
        this.isAdmin = isAdmin;  
        this.username = username;  
    }  
  
}
```

위 예제 처럼 ResponseBody 로 사용될 DTO에  `@JsonProperty` 를 사용하게 되면 다음의 결과를 얻는다.

![](attachments/Pasted%20image%2020231022191239.png)

`admin`, `isAdmin` 필드가 둘 다 리턴되며, 역시 롬복의 `@Getter` 로 인해 생긴 문제이다. 
일단 RequestBody 상황에서 `@JsonProperty` 를 사용해서 문제해결이 되었으니 괜찮지 않느냐라고 할 수도 있겠지만, 일관적이지 않은 사용은 후에 써야할 지 말아야할 지 고민하게 되는 추가적인 리소스가 생길 수 있다. 이미 API 스펙이 모두 공유되고 바꿀 수 없는 상황에서 응급 조치를 위해 `@JsonProperty` 를 사용할 수 있겠지만, 아직 설계와 구현 단계라면 위 같은 이유 때문에 필자는 권장하지 않는 바이다.



## 방법 2: boolean 대신 Boolean 사용하기 (비추천)

boolean 대신 래퍼 클래스인 Boolean 을 사용하면 어떻게 될까?

```java
@Getter  
@ToString  
public class MemberCreateRequest {  
  
    private Boolean isAdmin; // boolean -> Boolean
    private String username;  
  
}
```

먼저 기존의 boolean 필드를 `Boolean` 으로 바꾸고, 컴파일 해본다.

```java
public class MemberCreateRequest {  
    private Boolean isAdmin;  
    private String username;  
  
    public MemberCreateRequest() {  
    }  
  
    public Boolean getIsAdmin() {  
        return this.isAdmin;  
    }  
  
    public String getUsername() {  
        return this.username;  
    }  
  
    public String toString() {  
        Boolean var10000 = this.getIsAdmin();  
        return "MemberCreateRequest(isAdmin=" + var10000 + ", username=" + this.getUsername() + ")";  
    }  
}
```

바이트 코드를 까보면, 이번에는 롬복이 `isAdmin()` 이 아닌 `getIsAdmin()` 을 만들어준 것을 확인할 수 있다. API 를 호출해보면 정상적으로 역직렬화도 이루어진다. 

그런데 이 방법도 추천하지 않는다. 래퍼 클래스라면 null 일 수 있기 때문이다.

![](attachments/Pasted%20image%2020231022192410.png)

실수로 보내는 쪽에서 필드를 빼먹고 보내면, 

![](attachments/Pasted%20image%2020231022192441.png)

`isAdmin` 필드는 null 이 된다. 물론, 컨트롤러에서 `isAdmin` 이 null 이면 false 를 할당해주는 등의 코드를 작성해줘서 잠재적 NPE를 막을 수도 있다. 그런데 과연 이게 좋은 방법일까? Boolean 타입 필드가 있는 `Request DTO` 를 사용하는 모든 컨트롤러 메서드 마다 전부 그렇게 검증해준다는 것은 여간 귀찮은 일이 아니다.


## 방법 3: Enum 사용하기 (강력 추천)

사실 이거 추천하려고 글을 썼다. `boolean` 대신 `Enum` 을 쓸 것을 고려해보자.

```java
@Getter  
@ToString  
public class MemberCreateRequest {  
  
    private MemberType memberType;  
    private String username;  
  
    public enum MemberType {  
        ADMIN, NON_ADMIN  
    }  
  
}
```

`boolean` 값이 요구되는 비즈니스 로직의 경우 일반적으로 `enum` 으로 바꾸는 것이 그리 부자연스럽지는 않았다. 또, 이렇게 변경하면 단순 `true/false` 로만 표현할 수 없는 상황에 대해 더 유연하게 대응할 수도 있다. *물론, boolean 의 역직렬화 오류를 피하자고 이렇게 까지 하는 것은 너무 하는 것 아닌가?* 라고 생각할 수도 있겠다. 그런데 아주 약간의 공수만 들이면 더 안전한 설계가 가능하고, 그렇게 트레이드 오프를 따진다면 이쪽이 꽤 남는 장사라고 생각한다. 얼마전 [향로님](https://jojoldu.tistory.com/720)도 같은 맥락으로 위같은 방식을 추천해주셨으니 해당 포스팅 역시 같이 참고하면 좋을 것 같다.


# 마치며

REST API에서 직렬화/역직렬화의 오류의 경우 대개 `기본 생성자`, `@Getter` 등의 키워드로 인해 문제가 발생하는 경우가 많다. 그리고 이 경우 `@RequestBody` 와 `ObjectMapper`의 동작방식을 훤히 꿰고 있다면 조금 더 쉽게 문제를 해결할 수 있을 것이다. 물론 나 역시 위 키워드들에 대한 이해도가 부족한 상태라, 조만간 이 친구들을 파헤쳐 볼 생각이다. 마침.

### 추가적으로 공부해볼 것

- `@RequestBody`
- `ObjectMapper`, `Jackson`

# 참고

- [json object 상호 변환시 lombok으로 인한 boolean 기본형 오류](https://eclipse4j.tistory.com/343)
- [@RequestBody에 왜 기본 생성자는 필요하고, Setter는 필요 없을까? #1](https://velog.io/@conatuseus/RequestBody에-기본-생성자는-왜-필요한가)
- [boolean 타입 'is' 생략 없이 json 직렬화하기 (with Lombok)](https://colour-my-memories-blue.tistory.com/17)
- [좋은 API Response Body 만들기](https://jojoldu.tistory.com/720) 