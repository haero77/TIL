---

---
### Reference

- [QClass가 생성되지만 import가 안되는 문제에 관하여](https://www.inflearn.com/questions/736214)
- https://www.inflearn.com/questions/753251
- https://www.inflearn.com/questions/781873

---

# 선 결론


![](attachments/Pasted%20image%2020231216190409.png)


- IntelliJ Annotation Processing 결과fh 생기는 src/main/generated 파일이 생긴다.
	- 해당 파일을 우클릭하여 Source 폴더로 등록한다.
	- 다시 import 시도


# 상황

- **빌드된 QClass 파일이 제대로 import 되지 않는 문제가 발생.**

# 원인 분석

## IntelliJ build 와 Gradle build 결과물 파일 위치가 다르다.

### IntelliJ build

- Intellij build 의 빌드 파일 위치

![](attachments/Pasted%20image%2020231213211051.png)

- `/project/out` 패키지에 빌드된 바이트 코드(.class) 파일 존재

![](attachments/Pasted%20image%2020231213211252.png)

- `annotationProcessor` 결과는 `/project/src/main/generated` 패키지에 존재

![400](attachments/Pasted%20image%2020231216185516.png)

- annotation processing 에서 디렉토리를 `generated`로 만들어놔서 그럼.

### Gradle build 

![500](attachments/Pasted%20image%2020231213211411.png)

- `Build and run using`을 `Gradle`로 하고 애플리케이션 실행

![](attachments/Pasted%20image%2020231213211611.png)

- 빌드된 클래스는 `/project/build/classes` 에 존재
- `QClass`는 `/project/build/generated` 에 존재

