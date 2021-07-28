# Querydsl

## 3. 기본 문법

### 3-1. 시작 - JPQL vs Querydsl

#### QueryDslBasicTest.java - 테스트 기본 코드

* `src/test/java/study/querydsl/QuerydslBasicTest.java`

```java
package study.querydsl;

import com.querydsl.jpa.impl.JPAQueryFactory;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import study.querydsl.entity.Member;
import study.querydsl.entity.QMember;
import study.querydsl.entity.Team;

import javax.persistence.EntityManager;
import javax.transaction.Transactional;

import static org.assertj.core.api.Assertions.assertThat;

@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    @Autowired EntityManager em;

    JPAQueryFactory queryFactory;

    @BeforeEach
    public void before() {
        queryFactory = new JPAQueryFactory(em);
        Team teamA = new Team("teamA");
        Team teamB = new Team("teamB");
        em.persist(teamA);
        em.persist(teamB);

        Member member1 = new Member("member1", 10, teamA);
        Member member2 = new Member("member2", 20, teamA);

        Member member3 = new Member("member3", 30, teamB);
        Member member4 = new Member("member4", 40, teamB);

        em.persist(member1);
        em.persist(member2);
        em.persist(member3);
        em.persist(member4);
    }

    @Test
    public void startJPQL() {
        // member1을 찾아라
        String qlString = "select m from Member m " +
                "where m.username = :username";
        Member findMember = em.createQuery(qlString, Member.class)
                .setParameter("username", "member1")
                .getSingleResult();

        assertThat(findMember.getUsername()).isEqualTo("member1");
    }

    @Test
    @DisplayName("querydsl")
    public void startQuerydsl() {
        QMember m = new QMember("m");

        Member findMember = queryFactory
                .select(m)
                .from(m)
                .where(m.username.eq("member1"))
                .fetchOne();

        assertThat(findMember.getUsername()).isEqualTo("member1");
    }
}

```

* `EntityManager`로 `JPAQueryFactory`생성
* Querydsl은 JPQL 빌더
* JPQL: 문자(실행 시점 오류), Querydsl: 코드(컴파일 시점 오류)
* JPQL: 파라미터 바인딩 직접, Querydsl: 파라미터 바인딩 자동 처리
* JPAQueryFactory를 필드로 제공하면 동시성 문제는 어떻게 될까?
    * 동시성 문제는 JPAQueryFactory를 생성할 때 제동하는 `EntityManager(em)`에 달려있다.
    * 스프링 프레임워크는 여러 쓰레드에서 동시에 같은 EntityManager에 접근해도, 트랜젝션 마다 별도의 영속성 컨텍스트를 제공하기 때문에, 동시성 문제는 걱정하지 않아도 된다.

### 3-2. 기본 Q-Type 활용

#### Q클래스 인스턴스를 사용하는 2가지 방법

```
QMember qMember = new QMember("m");
Qmember qMember = QMember.member;
```

#### QuerydslBasicTest.java (추가) - 기본 인스턴스를 static import와 함께 사용

```java
package study.querydsl;

import com.querydsl.jpa.impl.JPAQueryFactory;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import study.querydsl.entity.Member;
import study.querydsl.entity.QMember;
import study.querydsl.entity.Team;

import javax.persistence.EntityManager;
import javax.transaction.Transactional;

import static org.assertj.core.api.Assertions.assertThat;
import static study.querydsl.entity.QMember.*;

@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    @Autowired EntityManager em;

    JPAQueryFactory queryFactory;

    // ...

    @Test
    @DisplayName("querydsl 코드 간편화")
    public void startQuerydsl2() {
        Member findMember = queryFactory
                .select(member)
                .from(member)
                .where(member.username.eq("member1"))
                .fetchOne();
        assertThat(findMember.getUsername()).isEqualTo("member1");
    }
}

```

다음 설정을 추가하면 실행되는 JPQL을 볼 수 있다.

```yaml
spring.jpa.properties.hibernate.use_sql_comments: true
```

> 참고: 같은 테이블을 조인해야 하는 경우가 아니면 기본 인스턴스를 사용하자.

### 3-3. 검색 조건 쿼리

#### QuerydslBasicTest.java - 기본 검색 쿼리

```java
public class QuerydslBasicTest {

    // ...

    @Test
    @DisplayName("검색 조건 querydsl")
    public void search() {
        Member findMember = queryFactory
                .selectFrom(member)
                .where(member.username.eq("member1")
                        .and(member.age.eq(10)))
                .fetchOne();

        assertThat(findMember.getUsername()).isEqualTo("member1");
    }
}
```

* 검색 조건은 `.and()`, `.or()`를 메서드 체인으로 연결할 수 있다.
* `select`, `from`을 `selectFrom`으로 합칠 수 있음

#### JPQL이 제공하는 모든 검색 조건 제공

```
member.username.eq("member1"); // username = 'member1'
member.username.ne("member1"); // username != 'member1'
member.username.eq("member1").not() // username != 'member1'

member.username.isNotNull() // 이름이 is not null

member.age.in(10, 20); // age in (10, 20)
member.age.notIn(10, 20); // age not in (10, 20)
member.age.between(10, 30); // between 10, 30

member.age.goe(30); // age >= 30
member.age.gt(30); // age > 30
member.age.loe(30); // age <= 30
member.age.lt(30); // age < 30

member.username.like("member%"); // like 검색
member.username.contains("member"); // like '%member%' 검색
member.username.startsWith("member"); // like 'member%' 검색
```

#### AND 조건을 파라미터로 처리

```java
public class QuerydslBasicTest {

    // ...

    @Test
    @DisplayName("검색 조건 querydsl")
    public void search() {
        Member findMember = queryFactory
                .selectFrom(member)
                .where(member.username.eq("member1")
                        .and(member.age.eq(10)))
                .fetchOne();

        Member findMemberWithoutAnd = queryFactory
                .selectFrom(member)
                .where(member.username.eq("member1"),
                        member.age.eq(10))
                .fetchOne();

        assertThat(findMember.getUsername()).isEqualTo("member1");
        assertThat(findMember).isEqualTo(findMemberWithoutAnd);
    }
}
```

* `where()`에 파라미터로 검색조건을 추가하면 `AND`조건이 추가됨
* 이 경우 `null`값은 무시 -> 메서드 추출을 활용해서 동적 쿼리를 깔끔하게 만들 수 있음

## Note