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

## Note