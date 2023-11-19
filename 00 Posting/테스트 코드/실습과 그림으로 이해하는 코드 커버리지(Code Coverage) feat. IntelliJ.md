
- [코드 커버리지(Code Coverage)](#%EC%BD%94%EB%93%9C%20%EC%BB%A4%EB%B2%84%EB%A6%AC%EC%A7%80(Code%20Coverage))
- [커버리지 측정의 기준](#%EC%BB%A4%EB%B2%84%EB%A6%AC%EC%A7%80%20%EC%B8%A1%EC%A0%95%EC%9D%98%20%EA%B8%B0%EC%A4%80)
	- [함수 커버리지(Function Coverage)](#%ED%95%A8%EC%88%98%20%EC%BB%A4%EB%B2%84%EB%A6%AC%EC%A7%80(Function%20Coverage))
	- [구문 커버리지(Statement Coverage)](#%EA%B5%AC%EB%AC%B8%20%EC%BB%A4%EB%B2%84%EB%A6%AC%EC%A7%80(Statement%20Coverage))
	- [조건 커버리지(Condition Coverage)](#%EC%A1%B0%EA%B1%B4%20%EC%BB%A4%EB%B2%84%EB%A6%AC%EC%A7%80(Condition%20Coverage))
	- [브랜치 커버리지(Branch Coverage)](#%EB%B8%8C%EB%9E%9C%EC%B9%98%20%EC%BB%A4%EB%B2%84%EB%A6%AC%EC%A7%80(Branch%20Coverage))
		- [IntelliJ Branch Coverage 활성화](#IntelliJ%20Branch%20Coverage%20%ED%99%9C%EC%84%B1%ED%99%94)
- [구문 커버리지와 브랜치 커버리지의 관계](#%EA%B5%AC%EB%AC%B8%20%EC%BB%A4%EB%B2%84%EB%A6%AC%EC%A7%80%EC%99%80%20%EB%B8%8C%EB%9E%9C%EC%B9%98%20%EC%BB%A4%EB%B2%84%EB%A6%AC%EC%A7%80%EC%9D%98%20%EA%B4%80%EA%B3%84)
- [어떤 기준을 사용할 것인가?](#%EC%96%B4%EB%96%A4%20%EA%B8%B0%EC%A4%80%EC%9D%84%20%EC%82%AC%EC%9A%A9%ED%95%A0%20%EA%B2%83%EC%9D%B8%EA%B0%80?)
		- [Reference](#Reference)



![](attachments/Pasted%20image%2020231119221515.png)
*(IntelliJ 의 코드 커버리지 기능)*

# 코드 커버리지(Code Coverage)
---

`코드 커버리지(Code Coverage)`란, 작성한 프로덕션 코드가 테스트에 의해 얼마만큼 실행되었는지를 나타내는 백분율로 나타내는 지표이다. 테스트 역시 비용이라서, 이 커버리지를 그 기준점으로 활용할 수 있다. 



# 커버리지 측정의 기준
---

커버리지를 측정할 수 있는 기준은 다양하다. 함수가 실행되었는지 여부를 기준으로 할 수도 있고, 실제 작성한 Statement 개수를 기준으로 할 수도 있다. 여기서는 자주 사용되는 4가지 기준에 대해 소개한다. 

- Function Coverage
- Statment(=Line) Coverage
- Condition Coverage
- Branch(=Decision) Coverage


## 함수 커버리지(Function Coverage)

>  함수 커버리지 = 실행된 함수 개수 / 총 함수 개수 * 100

함수 커버리지는, 정의된 함수 중 몇 개나 호출되었는가를 나타내는 지표이다.
아래 예시를 보자.

![](attachments/Pasted%20image%2020231119182931.png)

위와 같이 `Calculator` 클래스가 있다. 생성자를 제외한 함수는 a, b, c, d 이렇게 4가지이다.
이 4개 중에, a와 b 두 가지만 테스트된다면 정의된 함수 4개 중에 2개만 실행되었으므로 함수 커버리지는 50%이다.

![](attachments/Pasted%20image%2020231119183344.png)

위와 같은 테스트를 작성하고(아무것도 검증하고 있지 않기 때문에 테스트라 말하기는 어렵지만, 개념 설명을 위해 많은 것을 생략한 것으로 이해해주시면 감사하겠습니다.), 커버리지를 측정해보자.

![](attachments/Pasted%20image%2020231119183500.png)

IntelliJ 에서 실행할 테스트 클래스를 우클릭하여, `More Run/Debug > Run TestClass with Coverage` 를 통해 커버리지를 측정할 수 있다.

![](attachments/Pasted%20image%2020231119183632.png)

Method 항목을 살펴보면, 정의된 함수 4개 중에 2개만 실행되었으므로 함수 커버리지는 50% 임을 알 수 있다. (private 생성자의 경우 카운팅 되지 않는다.)


## 구문 커버리지(Statement Coverage)

> 구문 커버리지 = 실행된 구문 개수 / 총 구문 개수 * 100

라인(Line) 커버리지라고도 한다. 작성한 Statement 중에 얼마나 많은 Statement가 히트되었는지를 나타낸다. (Statement와 Expression에 대한 차이는 [여기](https://www.oreilly.com/library/view/learning-java/1565927184/ch04s04.html)를 참고.)

![](attachments/Pasted%20image%2020231119191802.png)

설명을 위해, 예제 클래스를 하나의 함수를 갖도록 수정했다. `test()` 에는 4개의 statement 가 존재한다. 
x=1, y=1 을 인자로 주고 테스트를 한다면 `(3)번 구문`까지 실행될 것이고 구문 커버리지는 `3/4 * 100 = 75%`가 된다. 

![](attachments/Pasted%20image%2020231119195232.png)
*(Line Coverage = Statement Coverage 가 75% 인 것을 확인할 수 있다.)*


구문 커버리지를 `100%`로 만들기 위해서는, 모든 구문을 실행하도록 테스트를 작성하면 된다. 
예컨대, 위의 경우 `x=-1, y=-1` 을 입력으로 사용하면 `(4)번 구문` 까지 실행되므로 커버리지는
`4/4 * 100 = 100%`가 된다. 

![](attachments/Pasted%20image%2020231119195205.png)


## 조건 커버리지(Condition Coverage)

> 조건 커버리지 = 실행된 조건 개수 / 총 조건 개수 * 100

여기서 말하는 '조건'이란, 조건식 내부의 모든 조건을 말한다.
조건 커버리지는 각 조건이 *true/false*를 만족시키는지를 확인한다.

```java
public static boolean test(int x, int y) {  
    System.out.println(x + ", " + y); // (1)  
  
    if (x > 0 && y > 0) { // (2)  
        return true; // (3)  
    }  
  
    return false; // (4)  
}
```

위 예제에서, 조건은 `x > 0`, `y > 0` 이 두 가지가 존재한다.

- `x=1, y=-1` 을 입력하면, 
	- `x > 0`: true
	- `y > 0`: false
- `x=-1, y=1` 을 입력하면, 
	- `x > 0`: false
	- `y > 0`: true

이 되므로, 조건 `x > 0` 과 `y > 0` 입장에서는 각각 *true/false* 를 만족하는 테스트가 수행되었다. 
그러나, `if 문` 입장에서 보면 항상 false 인 테스트만 수행되었으므로 `(4)번 구문`은 수행되지 않았다. 
즉, 조건 커버리지를 만족하더라도 구문 커버리지는 만족 못하는 경우가 생길 수 있다는 것이다. 


## 브랜치 커버리지(Branch Coverage)

> 브랜치 커버리지 = 실행된 브랜치 개수 / 총 브랜치 개수 * 100

결정(Decision) 커버리지라고도 한다. 프로그램 내의 모든 조건식이 각각 true/false 의 결과를 갖게되면 만족한다.

브랜치 커버리지를 이해하기 위해서는, 먼저 이 '브랜치'가 무엇인지를 정확히 이해해야한다. 

```java
public static void printSum(int a, int b) {  
    int result = a + b; // (1)  
  
    if (result > 0) { // (2)  
        System.out.println("result is positive."); // (3)  
    } else if (result < 0){ // (4)  
        System.out.println("result is negative."); // (5)  
    }  
  
    // do nothing  
}
```

위 코드에서 브랜치가 몇 개일까? `if` 문에 해당하는 경우 한 개, `if-else` 에 해당하는 경우 한 개. 이렇게 해서 총 2개일까? 아니다. 위 코드의 브랜치는 총 4개이다.

여기서 조건식은 `result > 0`, `result < 0` 이렇게 두 개이며 `result > 0` 이 true/false 를 갖는 경우의 분기(브랜치) 2개, `result < 0` 이 true/false 를 갖는 경우의 분기 2개 해서 총 4개가 되는 것이다.
 
아래 그림을 통해 조금 더 쉽게 이해해보자.

![](attachments/Pasted%20image%2020231119210107.png)

어렵게 생각하지 말자. 말 그대로 분기의 개수를 세는 것이다. 조건문마다 2개의 분기가 이루어지고, 현재 조건문은 2개이므로 브랜치는 2 * 2 = 4개 이다. 


IntelliJ 에서도 브랜치가 몇 개인지 직접 확인해볼 수 있다.

### IntelliJ Branch Coverage 활성화하기

![](attachments/Pasted%20image%2020231119210841.png)

1. shift 를 연속으로 두 번 눌러 `edit configuration` 검색 및 실행한다.

![](attachments/스크린샷%202023-11-19%20오후%209.04.56.png)

2. 좌측 하단의 `Edit configuration templates` 를 선택 > `JUnit` 선택 > `Modify options` 선택 

![](attachments/스크린샷%202023-11-19%20오후%209.06.41.png)

3. `Enable branch coverage and test tracking` 를 적용 후 `Run with Coverage` 를 통해 테스트를 실행한다.

![](attachments/스크린샷%202023-11-19%20오후%209.13.04.png)

### 브랜치 개수 세보기

그럼 방금 전 설명과 마찬가지로, 브랜치 개수가 4개인 것을 확인할 수 있다.

아직 브랜치 개수에 대해 헷갈릴 수도 있으니, 예제를 더 살펴보자.

```java
public static void printSum(int a, int b) {  
    int result = a + b; // (1)  
  
    if (result > 0) { // (2)  
        System.out.println("result is positive."); // (3)  
    } else if (result < 0){ // (4)  
        System.out.println("result is negative."); // (5)  
    } else if (result == 0) { // (6)  
        System.out.println("result is Zero"); // (7)  
    }  
  
    // do nothing  
}
```

위 브랜치의 개수는 몇 개일까? 

![](attachments/Pasted%20image%2020231119211604.png)

각 조건문 마다 2개 씩 분기되니, 브랜치는 6개이다. 


![](attachments/스크린샷%202023-11-19%20오후%209.17.02.png)

테스트를 다시 수행해보면, IntelliJ 역시 브랜치를 6개로 표시하고 있음을 확인할 수 있다. 


### 브랜치 커버리지 만족시키기

```java
public static void printSum(int a, int b) {  
    int result = a + b; // (1)  
  
    if (result > 0) { // (2)  
        System.out.println("result is positive."); // (3)  
    } else if (result < 0){ // (4)  
        System.out.println("result is negative."); // (5)  
    }  
  
    // do nothing  
}
```

다시 원래의 예제로 돌아오자. 위 예제의 브랜치는 4개라고 했다. 
그럼, 브랜치가 4개니까 이를 만족시키려면 4개의 테스트가 필요한 것일까? 

![](attachments/스크린샷%202023-11-19%20오후%209.30.48.png)

아니, 그렇지 않다. 실제로 3개의 테스트만 가지고도 브랜치 커버리지를 만족시킬 수 있다. 
직관적으로 생각하면 4가지 분기가 있으니까 4개의 테스트가 있어야할 것 같은데, 실제로는 그렇지 않다.

![](attachments/Pasted%20image%2020231119220523.png)

첫 번째 조건식 다음, 바로 두 번째 조건식이 실행되므로, 3개의 입력만으로도 모든 브랜치를 확인할 수 있다.
각 조건식은 여전히 true/false 를 만족시키는 테스트 케이스를 모두 갖게 되므로, 브랜치 커버리지를 만족시키게 되는 것이다. 



# 구문 커버리지와 브랜치 커버리지의 관계
---

```java
public static void printSum(int a, int b) {  
    int result = a + b; // (1)  
  
    if (result > 0) { // (2)  
        System.out.println("result is positive."); // (3)  
    } else if (result < 0){ // (4)  
        System.out.println("result is negative."); // (5)  
    }  
  
    // do nothing  
}
```

위 예제를 통해 구문 커버리지와 브랜치 커버리지의 관계를 몇 가지 유추해볼 수 있다.

## 구문 커버리지보다 브랜치 커버리지가 더 강력한 규약

브랜치 커버리지는 만족시키지 못하지만, 구문 커버리지는 만족시킬 수 있다. 

![](attachments/스크린샷%202023-11-19%20오후%2010.28.45.png)

각 조건식을 만족시키는 테스트 케이스만 작성해도 Statement 는 언제든지 실행시킬 수 있다.

![](attachments/Pasted%20image%2020231119223206.png)

즉, ***브랜치 커버리지를 만족시키는 경우, 100% 구문 커버리지를 만족시킬 수 있다.*** (물론 브랜치가 존재하는 코드에 한해서다.) 브랜치 커버리지가 구문 커버리지보다 더 강력한 규약이기 때문이다.

그렇다면, 브랜치 커버리지를 구문 커버리지보다 우선순위에 두고 테스트를 작성해야하는 걸까? 어떤 기준을 우선시할 것인지에 대해 다음 섹션에서 살펴보자.



# 어떤 기준을 사용할 것인가?
---

일단, 함수 커버리지와 조건 커버리지는 후순위이다. 

- 함수 커버리지의 경우, 내 프로덕션 코드를 제대로 검증받을 수 없다. 호출되기만 하면 커버리지가 채워지기 때문이다. 
- 조건 커버리지도 마찬가지다. 조건 커버리지는 만족시켜도 구문 커버리지는 만족시키지 못한다는 것은 역시 내 프로덕션 코드를 제대로 검증할 수 없다는 것을 의미한다.

## 브랜치 커버리지 100%는 조금 힘들 수도

남은 것은 브랜치와 구문이다. 다만 현실적으로 브랜치 커버리지를 사용하기 어려운 몇 가지 이유가 있다.

- *테스트가 불필요한 브랜치도 있을 수 있다.*

예를 들어 내 경우 `equals & hashcode` 를 재정의하고 IDE에 의해 자동완성 시켜놓은 코드까지 테스트하는 것은 테스트 효율성을 떨어뜨린다고 생각해서 이에 대한 테스트는 진행하지 않는다. 이로 인해 브랜치 커버리지는 떨어지게 된다. 

- *조건문이 존재하지 않는 코드도 많다.*

앞서, '브랜치 커버리지를 만족시키는 경우 100% 구문 커버리지를 만족시킨다.'라고 했지만 이것은 언제까지나 브랜치가 존재할 경우에 한해서다. 실제 프로덕션 코드에는 브랜치(분기)가 없는 코드가 더 많으므로 이런 코드들은 브랜치 커버리지의 대상에 포함조차 되질 않는다. 


![](attachments/Pasted%20image%2020231119221515.png)
*(브랜치 커버리지보다 높은 구문(Statement, Line) 커버리지)*

실제로 위 같은 이유들로 인해, 진행 중이던 토이 프로젝트의 커버리지를 측정해보니 사진과 같은 결과가 나오는 것을 확인할 수 있었다.


## 일단 구문 커버리지

위 브랜치 커버리지를 현실적으로 '무조건' 우선순위로 두기에는 어렵다는 것은 알았으니, 남은 것은 구문 커버리지다. 구문 커버리지를 만족시킨다는 것은 적어도 작성한 코드들이 테스트에 의해 검증되었다는 것은 보장하므로, 우선은 이 편을 사용할 것 같다. ('장애'가 없다는 것까지 보장하지는 못한다.)

물론 브랜치 커버리지와 구문 커버리지를 모두 봐야하는 단위 테스트가 존재할 경우에는 브랜치 커버리지를 높이기 위해 테스트를 조금 더 추가하고. 더 나은 방식이 떠오를 때까지 당분간 이 방법을 써보려고한다.

<br>

*마침.*



### Reference

- [Udacity - Branch Coverage - Georgia Tech - Software Development Process](https://www.youtube.com/watch?v=JkJFxPy08rk) 
- https://www.baeldung.com/jacoco
- https://www.baeldung.com/cs/code-coverage
- https://www.atlassian.com/continuous-delivery/software-testing/code-coverage
- https://tecoble.techcourse.co.kr/post/2020-10-24-code-coverage/
- [ERRORCODE7 - 테스트 커버리지(Test Coverage)](https://err0rcode7.github.io/backend/2021/05/11/%ED%85%8C%EC%8A%A4%ED%8A%B8%EC%BB%A4%EB%B2%84%EB%A6%AC%EC%A7%80.html)
- [hudi - 코드 커버리지](https://hudi.blog/code-coverage/) 
- https://medium.com/uplusdevu/code-coverage-c252e271df60
- [IntelliJ 코드 커버리지 확인하기](https://amaran-th.github.io/%EC%86%8C%ED%94%84%ED%8A%B8%EC%9B%A8%EC%96%B4%20%EC%84%A4%EA%B3%84/[IntelliJ]%20%EC%BD%94%EB%93%9C%20%EC%BB%A4%EB%B2%84%EB%A6%AC%EC%A7%80%20%ED%99%95%EC%9D%B8%ED%95%98%EA%B8%B0/)
- https://velog.io/@pgmjun/Java-테스트-커버리지-확인법


> 잘못된 정보가 있다면 언제든지 말씀 부탁드립니다 :)