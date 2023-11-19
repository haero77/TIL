
#테스트코드 #private


![](attachments/private_메서드를_테스트하고_싶을_때.png)


> private 메서드를 테스트하고 싶다는 것은, 잘못된 설계가 보내는 신호일 수 있다.


# 들어가며
---

개발을 하다보면, 문득 `private` 메서드를 테스트 하고 싶어지는 경우가 있다. 대개 한 개의 public 메서드에서 여러 private 메서드를 연쇄적으로 호출하는 경우, public 메서드의 테스트로는 충분히 private 을 커버하지 못하는 느낌을 받아 private 을 별도로 테스트해야 안심이 될 것 같아서이다.

단적으로 [private 메서드를 테스트하지 말라](https://shoulditestprivatemethods.com/)는 사이트가 존재하는 걸로 보아, 다른 선배 개발자들도 이 문제에 대해 꽤나 고심했던 것 같다. 그래도, private 메서드를 하고 싶은 걸 어떡하나. 다른 개발자들은 어떤 고민을 했고, 그 결과 *private 메서드를 테스트 하지 마라* 라는 결론에 이르기 되었는지 그 과정이 궁금하다.

그래서 이번 포스팅에서는 private 메서드를 테스트를 하고 싶어질 때 어떻게 하면 그 마음을 잠재울 수 있을지 몇 가지 방법에 대해 찾아보고자 한다. 또, 어떤 방법을 선택하여 사용할지 나의 기준 또한 말해보려 한다.



# 본문 
---

## public 메서드로 private 메서드의 동작을 충분히 확인하자


일단, private 메서드(이하 private)가  public 메서드(이하 public)에 의해 테스트가 이루어지는 것은 사실이다. public을 호출하면, 내부에서 연관된 private을 쓱 훑고 나올테니까. 

public을 테스트했음에도, private이 충분히 검증 받지 못한다고 느낀다면 **테스트가 부족한지는 않은지 의심**해볼 필요가 있다.

예컨대, 다음과 같은 public과 private이 있다고 하자. (리턴 타입이나 파라미터 등은 생략했다.)

```java
class Foo {

	public publicMethod() {
		validateSomething();
		...
	}

	private validateSomething() {
		if (something is invalid) {
			then throw some exception.
		}
	}
	
}
```

public에서, 검증하는 메서드인 private을 호출한다. 위 public 에 대한 테스트를 작성할 때, `정상 케이스`는 작성했지만 `예외 케이스` 를 빠뜨리고 작성했다고 하자. 이 경우, public을 제대로 테스트했다고 보기는 어려울 것이다. 그리고 이 순간, *'public을 테스트했음에도, private이 충분히 검증 받지 못한다'* 고 느끼게 된다.

자 그럼 이 불안감을 해소하기 위해, 다음 두 가지 선택지를 고려해볼 수 있다.

1. private의 정상, 예외 테스트 케이스 작성하기
2. public의 예외 테스트 케이스 추가하기

둘 중에 무엇을 선택하면 좋을까?


### private을 테스트하기

먼저 `1. private 의 정상, 예외 테스트 케이스 작성하기` 을 고려해보자. 일단 private이 제대로 작동하는지 확인하는 것이니, 심리적 안정감 하나는 기똥찰 것이다. 

private은 인스턴스를 만들어 테스트할 수는 없다보니, [Reflection 등을 이용해 테스트를 진행한다.](https://www.baeldung.com/java-unit-test-private-methods#how-to-test-private-methods-in-java)
앞으로 모든 private이 생길 때마다 위 같은 방식으로 테스트를 하면 되니 사실상 애플리케이션 내의 모든 메서드를 테스트할 수 있을 것이다. 더 이상 내가 구현한 코드가 비정상적으로 동작할지에 대한 걱정은 할 필요가 없어진다.

자 그럼 이제 테스트도 해봤으니, 정말로 이게 '좋은' 방법인지 고민해 볼 때다.

일단 접근 제어자 `private` 을 사용했다는 것은, 외부에 구현을 드러내지 않게 하기 위해서라는 건데, 이걸 억지로 끄집어 내어서 테스트를 하는 것 자체가 썩 내키진 않는다. 이럴거면 속 편히 `public`으로 접근 제한을 늘리고 주석으로 `'// 절대 외부에서 쓰지 말아주세요.'`라고 작성하는 편이 나을 것 같다. 기껏 고민해서 캡슐화를 한 이유 역시 사라지는 것 같기도 하다.

아무튼 썩 내키지 않는 것은 그렇다치고, 실질적인 문제도 있다. 
public 같은 경우, 해당 클래스만 놓고 보면 외부에서 사용하는 API이다. 즉, 이미 설계가 끝나 여기저기서 사용하고 있는 녀석을 바꾸기란 쉽지 않다(어떤 사이드 이펙트가 있을지 모른다.). 반면에 private의 경우 수정이 좀 더 자유로운 편이다. 즉, private을 테스트하게 된다면 private을 수정할 때마다 관련된 테스트 역시 모두 수정해줘야하니 유지보수 비용이 만만치 않을 것이다. 어디 테스트 무서워서 쉽사리 수정이야 하겠는가.


### public의 테스트를 추가하기

이번에는 `2. public의 예외 테스트 케이스 추가하기` 를 선택했다고 하자. private의 예외케이스는 이를 호출하는 public 입장에서도 같은 예외케이스에 해당한다. 즉, ***public 의 테스트가 충분하다면, private 역시 제대로 검증받을 수 있다.*** 

이제 public의 `예외 케이스` 테스트를 추가해보자. 기존에 작성된 `정상 케이스` 테스트가 있으니 추가로 작성하는 것은 그리 어렵지 않을 것이다. 그렇게 부족한 테스트를 채워가며 허기졌던 private 테스트 욕구를 점차 채워나가면 된다.



## 객체 분리의 신호탄

> ["private method를 테스트하고 싶다는 생각이 드는 것 자체가 **객체 분리의 신호탄**입니다.*"](https://www.inflearn.com/questions/1050402) - by 우빈님.

public의 테스트를 추가해도, private이 충분히 테스트받고 있지 않다고 느낄 때는 어떻게 해야할까? 아마 이 경우는 private의 몸집이 크거나 복잡해서 별도로 테스트가 필요한 경우에 해당할 것이다. 즉, 애초에 `private`이 아닌 `public`으로 설계했어야 했던 메서드라는 것이다. 

그런데 이미 private으로 설계했는데, 어떻게하면 안전하게 설계를 변경할 수 있을까?
생각보다 간단하다. 새 객체를 만들어 해당 책임을 옮기고, 접근 제어자를 `private`에서 `public`으로 변경한다([메서드를 메서드 오브젝트로 변경하는 방법](https://refactoring.com/catalog/replaceFunctionWithCommand.html)). 이제 손쉽게 테스트를 작성할 수 있다.

물론 처음에 public을 public으로, private을 private으로 잘 설계하면 위 같은 수고를 덜겠지만, 그게 어디 좀 어려운가. 코드야 계속 추가되고, 변경되고 그러다보면 private이 private이 아닐 수 있게되고 뭐 얼마든지 그럴 수 있다고 생각한다. 

내가 생각하기에 핵심은 이런 상황을 마주했을 때 `객체로 분리해야하는 상황`인지, 아니면 `테스트를 더 추가해 private을 충분히 검증`하게 할 것인지에 대해 자신만의 기준을 잘 세우는 것이라고 생각한다. 그리고 일단 내가 세운 기준은 다음과 같다.

- ***public 테스트를 충분히 진행했음에도, private에 대한 검증이 부족하다고 느끼면 그 때 객체로 분리하는 것을 고려한다.***



## 그래서 앞으로는 어떻게 할 것인가

눈치챘겠지만, 나 역시 ***private은 테스트 안 할 생각이다.*** 단, 이는 public으로 이미 private 역시 충분히 검증하고 있다는 전제가 놓여있다. 그리고 나서 위의 `기준`에 해당할 경우 객체로 분리하여 '테스트 가능하게' 만들어 테스트를 진행할 것이다.



# 마치며 
---

이번 포스팅을 하기 전까지는 사실 'private을 public으로 만드려면 객체 분리밖에 답이 없겠구나'하면서 느낌적(?)으로 객체로 분리하여 테스트를 해오고 있었다. 그런데 자료조사를 하면서 객체 분리가 많이 추천되는 것을 보고, 그동안 객체지향 공부를 하면서 얻은 지식을 나름 잘 녹여내고 있구나라는 모종의 안도감을 얻을 수 있었다. 특히, [평소에 즐겨보던 테스트 강의](https://inf.run/YLRXA)의 지식공유자이신 우빈님 역시 비슷한 의견을 말씀하시는 것을 보니 괜히 기분 좋고 그렇달까. 덕분에 이번 포스팅은 꽤나 재밌는 학습 경험이었다. 이제 private 테스트에 대한 기준도 세워봤으니, 테스트 자체를 잘 작성하기 위한 공부도 다시 이어나가보려 한다. 공부하며 얻은 인사이트는 역시 꾸준히 포스팅으로 남길 생각이다. 이상으로 본 포스팅은 여기서 마친다. *마침.*


## 더 공부해 볼 것 

- TDD
- Test 가능한 설계

## Reference

- [(인프런) 우빈님 - private 메서드 테스트에 대한 질답](https://www.inflearn.com/questions/1050402)
- [프라이빗 메소드들은 테스트 하지 마라](https://justinchronicles.net/ko/2014/07/14/dont-do-testing-private-methods/)
- https://stackoverflow.com/questions/105007/should-i-test-private-methods-or-only-public-ones#105021
- [Baeldung - Unit Test Private Methods in Java](https://www.baeldung.com/java-unit-test-private-methods#why-we-shouldnt-test-private-methods)
- https://yearnlune.github.io/java/java-private-method-test/#
- https://mntdev.tistory.com/91?category=1403630



