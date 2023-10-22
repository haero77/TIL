

#Querydsl #BooleanExpression #BooleanBuilder #동적쿼리

# 들어가며 

동적 쿼리를 구현하기 위해 Querydsl을 사용할 때, 조건식 조합의 편리함 등을 근거로 `BooleanBuilder` 보다 `BooleanExpression` 을 주로 사용해왔습니다. 그러나 정작 `BooleanExpression`을 이용하여 여러 조건들을 조합할 때 `and` 등을 이용 시 `NullPointerException`을 마주치게 되어 **_과연 `BooleanExpression`을 이용한 조건식 조합이 편리한 것인가?_** 라는 것까지 생각이 미치게 되었습니다. 본 포스팅에서는 NPE를 마주친 경위와 해결 방법, 그리고 같은 문제를 마주쳤을 때 `BooleanBuilder`와 `BooleanExpression` 중 어떤 것을 선택할지에 대해서도 간단히 의견을 남겨보겠습니다.

<br>
# 본문

## 예제 설명

예제는 영한님의 강의에서 많이 다뤄지는 회원과 팀 예제를 통해 설명하겠습니다.
(예제 코드는 [여기](https://github.com/haero77/labs/blob/main/src/main/java/com/labs/domain/member/repository/MemberRepositoryImpl.java)를 참고해주세요.)

<br>

### 엔티티

```java
@Entity
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "member_id")
    private Long id;

    private String username;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "team_id")
    private Team team;

    public Member(String username, Team team) {
        this.username = username;
        this.team = team;
    }

}
```
```java
@Entity
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Team {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "team_id")
    private Long id;

    private String name;

    public Team(String name) {
        this.name = name;
    }

}
```

먼저 엔티티입니다.

-  회원과 팀은 각각 id 외에 username 과 name을 필드로 갖습니다.
- 회원과 팀은 다대일 관계이며, `회원 -> 팀` 방향으로 단방향 매핑되어있습니다.


### Repository 구현체

```java
public interface MemberRepository extends JpaRepository<Member, Long>, MemberRepositoryCustom {
}
```
```java
@RequiredArgsConstructor
public class MemberRepositoryImpl implements MemberRepositoryCustom {

    private final JPAQueryFactory queryFactory;

    @Override
    public List<MemberTeamDto> searchBy(MemberSearchCondition condition) {
        return queryFactory
                .select(new QMemberTeamDto(
                        member.id.as("memberId"),
                        member.username,
                        team.id.as("teamId"),
                        team.name.as("teamName")))
                .from(member)
                .leftJoin(member.team, team)
                .where(
                        member.username.eq(condition.getUsername()),
                        team.name.eq(condition.getTeamName())
                )
                .fetch();
    }

}
```

다음은 동적쿼리를 구현하는 Repository 구현체입니다.

- `MemberRepository`는 `JpaRepository`와 `MemberRepositoryCustom`을 상속합니다.
- `MemberRepositoryCustom`의 구현체 `MemberRepositoryImpl`에 동적 쿼리를 사용하는 `searchBy()`를 구현했습니다.

<br>

### 조건식 DTO

```java
@Getter
public class MemberSearchCondition {

    private final String username;
    private final String teamName;

    public MemberSearchCondition(String username, String teamName) {
        this.username = username;
        this.teamName = teamName;
    }

}
```

동적쿼리의 파라미터로 사용되는 DTO입니다. 필드 `username`에 `null`을 명시적으로 나타내며 문제 상황을 시연해보겠습니다. 이것으로 필요한 예제 설명은 마쳤습니다. 

<br>

## 그냥 eq 를 사용하기 

먼저 아래처럼 where 절에 `eq` 를 사용할 때 인자로 `null` 이 주어질 때 어떻게 되는지 보겠습니다.

```java
...
        
.where(
        member.username.eq(condition.getUsername()),
        team.name.eq(condition.getTeamName())
)

...
```

```java
@Test
@DisplayName("회원 이름 또는 팀 이름과 일치하는 회원-팀 정보를 조회할 수 있다.")
void searchByCondition() {
    // given
    Team teamA = new Team("teamA");
    Team teamB = new Team("teamB");
    teamRepository.saveAll(List.of(teamA, teamB));

    Member member1 = new Member("member1", teamA);
    Member member2 = new Member("member2", teamA);
    Member member3 = new Member("member3", teamB);
    Member member4 = new Member("member4", teamB);
    memberRepository.saveAll(List.of(member1, member2, member3, member4));

    MemberSearchCondition condition = new MemberSearchCondition(null, "teamA");

    // when
    List<MemberTeamDto> memberTeamDtos = memberRepository.searchBy(condition);

    // then
    assertThat(memberTeamDtos).hasSize(2);
}
```

위는 앞으로 사용하게 될 테스트 코드입니다. 이 테스트를 실행하면 다음과 같이 `IllegalArgumentException`이 발생합니다.

```
org.springframework.dao.InvalidDataAccessApiUsageException: 
eq(null) is not allowed. Use isNull() instead; nested exception is java.lang.IllegalArgumentException: eq(null) is not allowed. Use isNull() instead
```

![image](https://github.com/haero77/Today-I-Learned/assets/65555299/103a07a5-32a1-4533-9b0b-12e1da7b6173)

`eq()` 메서드는 파라미터가 null 이면 `IllegalArgumentException`을 던지도록 구현되어있기 때문에, 이는 당영한 결과입니다. 그럼 이 문제를 어떻게 해결할까요? 인자가 null 이어서 생긴 문제니까, 인자가 null 인지 확인하도록 하면 될 것 같습니다.

<br>

## BooleanExpression을 리턴하는 null-safe 메서드

```java
// MembmerRepositoryImpl
@Override
public List<MemberTeamDto> searchBy(MemberSearchCondition condition) {
    return queryFactory
            .select(new QMemberTeamDto(
                    member.id.as("memberId"),
                    member.username,
                    team.id.as("teamId"),
                    team.name.as("teamName")))
            .from(member)
            .leftJoin(member.team, team)
            .where(
                    usernameEquals(condition.getUsername()),
                    teamNameEquals(condition.getTeamName())
            )
            .fetch();
}

public BooleanExpression usernameEquals(String username) {
    return username != null ? member.username.eq(username) : null;
}

public BooleanExpression teamNameEquals(String teamName) {
    return teamName != null ? team.name.eq(teamName) : null;
}
```

문제가 되었던 `member.username.eq(username)`을 호출하기 전에 먼저 `username`을 체크하는 메서드 `usernameEquals()` 만들었습니다. 테스트를 다시 실행해보면 이전과 같은 `IllegalArgumentException`은 발생하지 않습니다. `username`이 null 이어서, `usernameEquals()`이 null 을 리턴한다고 해도, **where 절에서 null 이 인자로 들어왔을 때 무시하기 때문에 문제될 것이 없습니다.** 그리고 이 때문에 동적쿼리가 가능해지는 것이죠. `BooleanExpression`을 리턴하는 메서드로 조건식을 분리하면 또 다른 장점도 있습니다. 바로 조합이 가능하다는거죠.

<br>

## BooleanExpression 조합하기

where 절에서 아래와 같이 모든 조건 메서드를 호출하는 것은 아무래도 조금 번거롭습니다. 추후에 조건이 더 추가될 수도 있는 것이고, 아예 모든 조건을 동적으로 처리하는 메서드로 새롭게 분리해지고 싶어졌습니다.

```
.where(
        usernameEquals(condition.getUsername()),
        teamNameEquals(condition.getTeamName())
)
```

이 두 개의 메서드를, 

```java
@Override
public List<MemberTeamDto> searchBy(MemberSearchCondition condition) {
    return queryFactory
            .select(new QMemberTeamDto(
                    member.id.as("memberId"),
                    member.username,
                    team.id.as("teamId"),
                    team.name.as("teamName")))
            .from(member)
            .leftJoin(member.team, team)
            .where(
                    searchConditionEquals(condition)
            )
            .fetch();
}

public BooleanExpression searchConditionEquals(MemberSearchCondition condition) {
    return usernameEquals(condition.getUsername())
            .and(teamNameEquals(condition.getTeamName()));
}

public BooleanExpression usernameEquals(String username) {
    return username != null ? member.username.eq(username) : null;
}

public BooleanExpression teamNameEquals(String teamName) {
    return teamName != null ? team.name.eq(teamName) : null;
}
```

`searchConditionEquals()` 라는 메서드에서 호출하도록 리팩토링 했습니다. 이제 조건이 더 추가되어도 해당 메서드만 조금씩 수정하면 되겠죠.

**그런데 문제는, null 체크를 해주는 메서드들을 위처럼 조합할 때 생깁니다. 바로 조합된 메서드(`searchConditionEquals()`)에서 NPE가 발생할 위험이 있기 때문입니다.**  

<br>

### BooleanExpression 조합 시 발생하는 NPE

먼저 NPE가 어느 상황에서 발생하는지 알아보겠습니다. 기존에 사용하던 테스트를 실행시켜봅시다.

```java
@Test
@DisplayName("회원 이름 또는 팀 이름과 일치하는 회원-팀 정보를 조회할 수 있다.")
void searchByCondition() {
    // given
    Team teamA = new Team("teamA");
    Team teamB = new Team("teamB");
    teamRepository.saveAll(List.of(teamA, teamB));

    Member member1 = new Member("member1", teamA);
    Member member2 = new Member("member2", teamA);
    Member member3 = new Member("member3", teamB);
    Member member4 = new Member("member4", teamB);
    memberRepository.saveAll(List.of(member1, member2, member3, member4));

    MemberSearchCondition condition = new MemberSearchCondition(null, "teamA");

    // when
    List<MemberTeamDto> memberTeamDtos = memberRepository.searchBy(condition);

    // then
    assertThat(memberTeamDtos).hasSize(2);
}
```

![image](https://github.com/haero77/Today-I-Learned/assets/65555299/e0cba88c-ffb3-4ad9-9b20-5782895dd425)


그럼 이렇게 NPE가 발생합니다. 이상합니다. 분명 where 절에서는 null 을 인자로 넘겨도 무시된다고 했는데 말이죠. 원인을 찾기 위해 디버깅 합니다.

<br>

<img width="828" alt="image" src="https://github.com/haero77/Today-I-Learned/assets/65555299/07da6f39-f562-48e1-ad46-dca516f214f8">

`MemberSearchCondition`의 필드 `username`이 `null`이므로, `userNameEquals()`의 결과로 `null`이 리턴 됩니다. 결국 `searchConditionEquals()` 에서  `null.and(teamNameEquals())`가 호출되고, `and()`를 호출하는 시점에서 NPE가 발생한 거죠. 

**_그럼 NPE를 방지하면서, 조건식 조합 역시 편리하게 이용하려면 어떻게 해야할까요?_**

일단 지금 방식인 `BooleanExpression`을 사용해서는 어려울 것 같습니다. `and()` 등을 이용해 조건을 조합할 때 NPE가 발생하니까요. 그렇습니다. 다시 `BooleanBuilder`를 살펴볼 때가 된거죠.

<br>

## NPE를 해결하는 null-safe BooleanBuilder

[영한님의 답변](https://www.inflearn.com/questions/94056/%EA%B0%95%EC%82%AC%EB%8B%98-where-%EB%8B%A4%EC%A4%91-%ED%8C%8C%EB%9D%BC%EB%AF%B8%ED%84%B0%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EB%8F%99%EC%A0%81-%EC%BF%BC%EB%A6%AC-%EC%82%AC%EC%9A%A9%EC%97%90-%EB%8C%80%ED%95%9C-%EC%A7%88%EB%AC%B8%EC%9E%85%EB%8B%88%EB%8B%A4)에서 위 문제에대한 해결법을 찾을 수 있었습니다. 바로 코드로 살펴보겠습니다. 

```java
@Override
public List<MemberTeamDto> searchBy(MemberSearchCondition condition) {
    return queryFactory
            .select(new QMemberTeamDto(
                    member.id.as("memberId"),
                    member.username,
                    team.id.as("teamId"),
                    team.name.as("teamName")))
            .from(member)
            .leftJoin(member.team, team)
            .where(
                    searchConditionEquals(condition)
            )
            .fetch();
}

public BooleanBuilder searchConditionEquals(MemberSearchCondition condition) {
    return usernameEquals(condition.getUsername())
            .and(teamNameEquals(condition.getTeamName()));
}

public BooleanBuilder usernameEquals(String username) {
    return nullSafeBooleanBuilder(() -> member.username.eq(username));
}

public BooleanBuilder teamNameEquals(String teamName) {
    return nullSafeBooleanBuilder(() -> team.name.eq(teamName));
}

private BooleanBuilder nullSafeBooleanBuilder(Supplier<BooleanExpression> supplier) {
    try {
        return new BooleanBuilder(supplier.get());
    } catch (IllegalArgumentException e) {
        return new BooleanBuilder();
    }
}
```

중요한 내용부터 하나씩 살펴보겠습니다. 

<br>

```java
private BooleanBuilder nullSafeBooleanBuilder(Supplier<BooleanExpression> supplier) {
    try {
        return new BooleanBuilder(supplier.get());
    } catch (IllegalArgumentException e) {
        return new BooleanBuilder();
    }
}
```

BooleanExpression의 Supplier를 파라미터로 받아서, 정의된 람다식을 실행하고, `IllegalArgumentException`이 발생하면 빈 BooleanBuilder 객체를 만들어 리턴합니다. 

파라미터로 `() -> member.username.eq(username)` 같은 람다식이 주어지는 경우를 예로 들어보죠.

![image](https://github.com/haero77/Today-I-Learned/assets/65555299/2223b132-eebb-417c-ba35-7a4d33349dbd)

`supplier.get()` 을 할 때, `member.username.eq(username)` 이 실행되고, `eq()` 에서는 위 사진에서 보다시피 `right` 가 `null` 이니까 `IllegalArgumentException` 을 발생시킵니다. try-catch 문에서 예외를 잡고 있으므로 빈 `BooleanBuilder()` 객체를 리턴해주게 됩니다. 

```java
public BooleanBuilder searchConditionEquals(MemberSearchCondition condition) {
    return usernameEquals(condition.getUsername())
            .and(teamNameEquals(condition.getTeamName()));
}
```

결국 `usernameEquals()` 의 결과로 **null** 이 아닌 `BooleanBuilder` 객체가 리턴되므로, `BooleanBuilder.and()` 처럼 동작하여 변경 전에 발생했던 `NPE`를 방지할 수 있게 됩니다. 



# 마치며

위에 소개해드린 `nullSafeBooleanBuilder()` 메서드의 경우, 여러 Repository 구현체에서 재활용하기 좋은 메서드입니다. 따라서 별도의 유틸 클래스 등을 만들어 적절하게 재사용해보면 좋을 것 같습니다. 

사실 영한님의 Querydsl 강의를 들을 때만 해도 `BooleanBuilder` 보다 `BooleanExpression` 이 낫구나 정도로만 이해했는데, 질답 게시판과 스스로 여러 테스트 케이스를 짜보며 위같은 `NPE` 문제 때문에 꼭 그런 것만도 아닌 것을 알게 되었습니다. 항상 뭐가 더 낫다라고 미리 판단하기 보다는 모두 사용해보며 장단점을 확실히 파악해둬야겠다는 생각이 드는 경험이었습니다. 이상으로 본 포스팅을 마칩니다.

_마침._

<br>



### Reference


- [WHERE 다중 파라미터 NULL 체크하기](https://www.inflearn.com/questions/94056/%EA%B0%95%EC%82%AC%EB%8B%98-where-%EB%8B%A4%EC%A4%91-%ED%8C%8C%EB%9D%BC%EB%AF%B8%ED%84%B0%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EB%8F%99%EC%A0%81-%EC%BF%BC%EB%A6%AC-%EC%82%AC%EC%9A%A9%EC%97%90-%EB%8C%80%ED%95%9C-%EC%A7%88%EB%AC%B8%EC%9E%85%EB%8B%88%EB%8B%A4)
- https://www.inflearn.com/questions/139577