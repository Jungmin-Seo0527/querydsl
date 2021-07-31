# Querydsl

## 4. 중급 문법

### 4-1. 프로젝션과 결과 반환 - 기본

프로젝션: select 대상 지정

#### 프로젝션 대상이 하나

```java
public class QuerydslBasicTest {

    @Test
    @DisplayName("프로젝션 한개")
    public void simpleProjection() {
        List<String> result = queryFactory
                .select(member.username)
                .from(member)
                .fetch();
        result.stream()
                .map(s -> "s = " + s)
                .forEach(System.out::println);
    }
}
```

* 프로젝션 대상이 하나면 타입을 명확하게 지정할 수 있음
* 프로젝션 대상이 둘 이상이면 튜플이나 DTO로 조회

#### 튜플 조회

* 프로젝션이 둘 이상을 때 사용

```java
public class QuerydslBasicTest {
    @Test
    @DisplayName("튜블로 조회")
    public void tupleProjection() {
        List<Tuple> result = queryFactory
                .select(member.username, member.age)
                .from(member)
                .fetch();

        result.forEach(tuple -> {
            System.out.println("username = " + tuple.get(member.username));
            System.out.println("age = " + tuple.get(member.age));
        });
    }
}
```

* `com.querydsl.core.Tuple`

> 참고        
> `Tuple`은 querydsl의 core 에 위치한다. 그러므로 다른 계층에서 Tuple을 알고 있는 것은 좋은 설계가 아니다.      
> 만약 querydsl이 아닌 다른 기술을 사용할 때 Tuple을 알고 있는 계층에서 문제가 발생할 수 있기 때문이다.

### 4-2. 프로젝션과 결과 반환 - DTO 조회

#### MemberDto.java

```java
package study.querydsl.dto;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class MemberDto {

    private String username;
    private int age;
}

```

#### 순수 JPA에서 DTO 조회

```java
public class QuerydslBasicTest {
    @Test
    @DisplayName("JPQL을 이용해서 DTO로 조회하기 (new)")
    public void findDtoByJPQL() {
        List<MemberDto> result = em.createQuery("select new study.querydsl.dto.MemberDto(m.username, m.age)" +
                        " from Member m", MemberDto.class)
                .getResultList();

        result.forEach(o -> System.out.println("memberDto = " + o));
    }
}
```

* 순수 JPA에서 DTO를 조회할 때는 new 명령어를 사용해야 함
* DTO의 package이름을 다 적어줘야해서 지저분함
* 생성자 방식만 지원함

#### Querydsl 빈 생성(Bean population)

결과를 DTO로 반환할 때 사용       
다음 3가지 방법 지원

* 프로퍼티 접근
* 필드 직접 접근
* 생성자 사용

#### 프로퍼티 접근 - setter

```java
public class QuerydslBasicTest {
    @Test
    @DisplayName("querydsl을 이용해서 DTO로 조회하기 - setter")
    public void findDtoBySetter() {
        List<MemberDto> result = queryFactory
                .select(Projections.bean(MemberDto.class,
                        member.username,
                        member.age))
                .from(member)
                .fetch();

        result.forEach(o -> System.out.println("memberDto = " + o));
    }
}
```

#### 필드 직접 접근

```java
public class QuerydslBasicTest {
    @Test
    @DisplayName("querydsl을 이용해서 DTO로 조회하기 - field")
    public void findDtoByField() {
        List<MemberDto> result = queryFactory
                .select(Projections.fields(MemberDto.class,
                        member.username,
                        member.age))
                .from(member)
                .fetch();

        result.forEach(o -> System.out.println("memberDto = " + o));
    }
}
```

#### 별칭이 다를 때

```java
package study.querydsl.dto;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class UserDto {
    private String name;
    private int age;
}

```

```java
public class QuerydslBasicTest {
    @Test
    @DisplayName("querydsl을 이용해서 DTO로 조회하기 - 별칭이 다를 때 & 서브쿼리")
    public void findUserDtoByConstructor3() {
        QMember memberSub = new QMember("memberSub");
        List<UserDto> result = queryFactory
                .select(Projections.fields(UserDto.class,
                        member.username.as("name"),
                        ExpressionUtils.as(JPAExpressions
                                .select(memberSub.age.max())
                                .from(memberSub), "age")
                ))
                .from(member)
                .fetch();

        result.forEach(o -> System.out.println("memberDto = " + o));
    }
}
```

* 프로퍼티나, 필드 접근 생성 방식에서 이름이 다를 때 해결 방안
* `ExpressionUtils.as(source, alias)`: 필드나, 서브 쿼리에 별칭 적용
* `username.as("memberName")`: 필드에 별칭 적용

#### 생성자 사용

```java
public class QuerydslBasicTest {
    @Test
    @DisplayName("querydsl을 이용해서 DTO로 조회하기 - constructor")
    public void findDtoByConstructor() {
        List<MemberDto> result = queryFactory
                .select(Projections.constructor(MemberDto.class,
                        member.username,
                        member.age))
                .from(member)
                .fetch();

        result.forEach(o -> System.out.println("memberDto = " + o));
    }
}
```

### 4-3. 프로젝션과 결과 반환 - @QueryProjection

#### 생성자 + @QueryProjection

```java
package study.querydsl.dto;

import com.querydsl.core.annotations.QueryProjection;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
public class MemberDto {

    private String username;
    private int age;

    @QueryProjection
    public MemberDto(String username, int age) {
        this.username = username;
        this.age = age;
    }
}

```

* `./gradlew compileQuerydsl`
* `QMemberDto`생성 확인

#### @QueryProjection 활용

```java
public class QuerydslBasicTest {
    @Test
    @DisplayName("프로젝션과 결과 반환 - @QueryProjection")
    public void findDtoByQueryProjection() {
        List<MemberDto> result = queryFactory
                .select(new QMemberDto(member.username, member.age))
                .from(member)
                .fetch();

        result.forEach(o -> System.out.println("memberDto = " + o));
    }
}
```

이 방법은 컴파일러로 타입을 체크할 수 있으므로 가장 안전한 방법이다. 다만 DTO에 QueryDSL 어노테이션을 유지해야 하는 점과 DTO까지 Q파일을 생성해야 하는 단점이 있다.

#### distinct

```java
public class QuerydslBasicTest {
    public void distinct() {
        List<String> result = queryFactory
                .select(member.username).distinct()
                .from(member)
                .fetch();
    }
}
```

* distinct는 JPQL의 distinct와 같다.

### 4-4. 동적 쿼리 - BooleanBuilder 사용

* 동적 쿼리를 해결하는 두가지 방식
    * `BooleanBuilder`
    * Where 다중 파라미터 사용

```java
public class QuerydslBasicTest {
    @Test
    @DisplayName("동적 쿼리 - BooleanBuilder")
    public void dynamicQuery_BooleanBuilder() {
        String usernameParam = "member1";
        Integer ageParam = null;

        List<Member> result = searchMember1(usernameParam, ageParam);
        assertThat(result.size()).isEqualTo(1);
    }

    private List<Member> searchMember1(String usernameCond, Integer ageCond) {
        BooleanBuilder builder = new BooleanBuilder();
        if (usernameCond != null) {
            builder.and(member.username.eq(usernameCond));
        }

        if (ageCond != null) {
            builder.and(member.age.eq(ageCond));
        }

        return queryFactory
                .selectFrom(member)
                .where(builder)
                .fetch();
    }
}
```

### 4-5. 동적 쿼리 - Where 다중 파라미터 사용

```java
public class QuerydslBasicTest {
    @Test
    @DisplayName("동적 쿼리 - where 다중 파라미터")
    public void dynamicQuery_WhereParam() {
        String usernameParam = "member1";
        Integer ageParam = null;

        List<Member> result = searchMember2(usernameParam, ageParam);
        assertThat(result.size()).isEqualTo(1);
    }

    private List<Member> searchMember2(String usernameCond, Integer ageCond) {
        return queryFactory
                .selectFrom(member)
                .where(usernameEq(usernameCond), ageEq(ageCond))
                .fetch();
    }

    private BooleanExpression usernameEq(String usernameCond) {
        return usernameCond == null ? null : member.username.eq(usernameCond);
    }

    private BooleanExpression ageEq(Integer ageCond) {
        return ageCond == null ? null : member.age.eq(ageCond);
    }
}
```

* `where`조건에 `null`값은 무시된다.
* 메서드를 다른 쿼리에서도 재활용 할 수 있다.
* 쿼리 자체의 가독성이 높아진다.

#### 조합 가능

````
privagte BooleanExpression allEq(String usernameCond, Integer ageCond) {
    return usernameEq(usernameCond).and(ageEq(ageCond));
````

* `null`체크는 주의해서 처리해야 함

## Note