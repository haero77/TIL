
### 문제

- 나는 맥북에 개인 github과 회사 github 계정을 모두 사용한다.
- 문제는, 회사 깃헙으로 작성되었어야할 Code Author가 개인 계정인 `haero77` 으로 작성되는 문제가 있었다.
- 따라서, 프로젝트 별로 **Code Author** 와 **Author Email** 을 별도로 분리해야할 필요를 느꼈다.


### 해결

- `git config --list`를 통해서 user 와 email을 확인 가능
  - 보통 global 설정이 되어있기 때문에 여러 프로젝트에서 별도로 author 지정하지 않고 사용가능하다.

```java
git config --local user.name "haero77"
git config --local user.email "shineecard5@gmail.com"
```

- 이처럼 `--local` 옵션을 주면 프로젝트 별로 author를 지정할 수 있다. 


### Reference

- https://github.com/HomoEfficio/dev-tips/blob/master/IntelliJ%20%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8%20%EB%B3%84%EB%A1%9C%20author%20%EB%8B%A4%EB%A5%B4%EA%B2%8C%20%EC%84%A4%EC%A0%95%ED%95%98%EA%B8%B0.md
- https://stackoverflow.com/questions/15566025/intellij-idea-with-git-remember-author
