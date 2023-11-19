
## MVC는 왜 생겨난걸까?

- 코드 유지보수가 너무 불편했다.
- MVC => 유지보수가 편해지는 코드 구성 방식

## MVC 구조

![](attachments/Pasted%20image%2020231028203557.png)

- Model
	- 데이터와 관련된 부분
- View 
	- 사용자한테 보여지는 부분
- Controller 
	- Model과 View 를 이어주는 부분

## MVC 를 지키면서 코딩하는 방법

### 1. Model은 Controller와 View에 의존하지 않아야 한다.

### 2. View는 Model에만 의존해야하고, Controller에는 의존하면 안 된다.

### 3. View가 Model로부터 데이터를 받을 때는, **사용자마다 다르게 보여주어야하는 데이터에 대해서만** 받아야한다. 

![500](attachments/Pasted%20image%2020231028203942.png)

- 뷰는 사용자마다 달라지는 부분(빨간 박스 부분)만 모델로부터 받아야한다.

![500](attachments/Pasted%20image%2020231028204100.png)

- View = UI + Model
- UI 에 해당하는 `배달정보` 같은 것은 모델로부터 받으면 안 되고 **뷰가 자체적으로 가져야하는 정보들**
	- (프린트 출력부분도 뷰가 자체적으로 가져야할 것 같다.)


### 4. Controller는 Model과 View에 의존해도 된다.

- 왜냐하면, 컨틀롤러는 모델과 뷰의 중개자 역할을 하면서 전체 로직을 구성하기 때문

### 5. View가 Model로부터 데이터를 받을 때, 반드시 Controller에서 받아야한다.

![500](attachments/Pasted%20image%2020231028204623.png)


