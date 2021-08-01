# Querydsl

## 5. 실무 활용 - 순수 JPA와 Querydsl

* 순수 JPA 리포지토리와 Querydsl
* 동적쿼리 Builder 적용
* 동적쿼리 Where 적용
* 조회 API 컨트롤러 개발

### 5-1. 순수 JPA 리포지토리와 Querydsl

#### MemberJpaRepository.java - 순수 JPA 리포지토리

* `src/main/java/study/querydsl/repository/MemberJpaRepository.java`

```java
package study.querydsl.repository;

import com.querydsl.jpa.impl.JPAQueryFactory;
import org.springframework.stereotype.Repository;
import study.querydsl.entity.Member;

import javax.persistence.EntityManager;
import java.util.List;
import java.util.Optional;

import static study.querydsl.entity.QMember.member;

@Repository
public class MemberJpaRepository {

    private final EntityManager em;
    private final JPAQueryFactory queryFactory;

    public MemberJpaRepository(EntityManager em) {
        this.em = em;
        this.queryFactory = new JPAQueryFactory(em);
    }

    public void save(Member member) {
        em.persist(member);
    }

    public Optional<Member> findById(Long id) {
        Member findMember = em.find(Member.class, id);
        return Optional.ofNullable(findMember);
    }

    public List<Member> findAll() {
        return em.createQuery("select m from Member m", Member.class)
                .getResultList();
    }

    public List<Member> findAll_Qeurydsl() {
        return queryFactory
                .selectFrom(member)
                .fetch();
    }

    public List<Member> findByUsername(String username) {
        return em.createQuery("select m from Member m where m.username = :username", Member.class)
                .setParameter("username", username)
                .getResultList();
    }

    public List<Member> findByUsername_Querydsl(String username) {
        return queryFactory
                .selectFrom(member)
                .where(member.username.eq(username))
                .fetch();
    }
}

```

#### JPQQueryFactory 스피링 빈 등록

다음과 같이 `JPAQueryFactory`를 스프링 빈으로 등록해서 주입받아 사용해도 된다.

```
@Bean
JPAQueryFactory jpaQueryFactory(EntityManager em) {
        return new JPAQueryFactgory(em);
}
```

> 참고: 동시성 문제는 걱정하지 않아도 된다. 왜냐하면 여기서 스프링이 주입해주는 엔티티 매니저는 실제 동작 시점에 진짜 엔티티 매니저를 찾아주는 프록시용 가짜 엔티티 매니저이다.       
> 이 가짜 엔티티 매니저는 실제 사용 시점에 트랜잭션 단위로 실제 엔티티 매니져(영속성 컨텍스트)를 할당해준다.

### 5-2. 동적 쿼리와 성능 최적화 조회 - Builder 사용

#### MemberTeamDto.java - 조회 최적화용 DTO 추가

* `src/main/java/study/querydsl/dto/MemberTeamDto.java`

```java
package study.querydsl.dto;

import com.querydsl.core.annotations.QueryProjection;
import lombok.Data;

@Data
public class MemberTeamDto {

    private Long memberId;
    private String username;
    private int age;
    private Long teamId;
    private String teamName;

    @QueryProjection
    public MemberTeamDto(Long memberId, String username, int age, Long teamId, String teamName) {
        this.memberId = memberId;
        this.username = username;
        this.age = age;
        this.teamId = teamId;
        this.teamName = teamName;
    }
}

```

* `@QueryProjection`을 추가했다. `QMemberTeamDto`를 생성하기 위해 `./gradlew compileQuerydsl`을 한번 실행하자.

> 참고: `@QueryProjection`을 사용하면 해당 DTO가 Querydsl을 의존하게 된다.       
> 이런 의존이 싫으면, 해당 에노테이션을 제거하고, `Projection.bean(), fields(), constructor()`을 사용하면 된다.

#### MemberSearchCondition.java - 회원 검색 조건

* `src/main/java/study/querydsl/dto/MemberSearchCondition.java`

```java
package study.querydsl.dto;

import lombok.Data;

@Data
public class MemberSearchCondition {
    //회원명, 팀명, 나이(ageGoe, ageLoe)
    //
    private String username;
    private String teamName;
    private Integer ageGoe;
    private Integer ageLoe;
}

```

* 이름이 너무 길면 `MemberCond`등으로 줄여 사용해도 된다.

#### MemberJpaRepository.java - 동적 쿼리(Builder 사용)

```java
package study.querydsl.repository;

import com.querydsl.core.BooleanBuilder;
import com.querydsl.jpa.impl.JPAQueryFactory;
import org.springframework.stereotype.Repository;
import study.querydsl.dto.MemberSearchCondition;
import study.querydsl.dto.MemberTeamDto;
import study.querydsl.dto.QMemberTeamDto;
import study.querydsl.entity.Member;

import javax.persistence.EntityManager;
import java.util.List;
import java.util.Optional;

import static org.springframework.util.StringUtils.hasText;
import static study.querydsl.entity.QMember.member;
import static study.querydsl.entity.QTeam.team;

@Repository
public class MemberJpaRepository {

    private final EntityManager em;
    private final JPAQueryFactory queryFactory;

    public MemberJpaRepository(EntityManager em) {
        this.em = em;
        this.queryFactory = new JPAQueryFactory(em);
    }

    // ...

    //Builder 사용
    //회원명, 팀명, 나이(ageGoe, ageLoe)
    public List<MemberTeamDto> searchByBuilder(MemberSearchCondition condition) {
        BooleanBuilder builder = new BooleanBuilder();
        if (hasText(condition.getUsername())) {
            builder.and(member.username.eq(condition.getUsername()));
        }
        if (hasText(condition.getTeamName())) {
            builder.and(team.name.eq(condition.getTeamName()));
        }
        if (condition.getAgeGoe() != null) {
            builder.and(member.age.goe(condition.getAgeGoe()));
        }
        if (condition.getAgeLoe() != null) {
            builder.and(member.age.loe(condition.getAgeLoe()));
        }
        return queryFactory
                .select(new QMemberTeamDto(
                        member.id,
                        member.username,
                        member.age,
                        team.id,
                        team.name))
                .from(member)
                .leftJoin(member.team, team)
                .where(builder)
                .fetch();
    }
}

```

#### MemberJpaRepositoryTest.java - 조회 예제 테스트

```java
package study.querydsl.repository;

import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import study.querydsl.dto.MemberSearchCondition;
import study.querydsl.dto.MemberTeamDto;
import study.querydsl.entity.Member;
import study.querydsl.entity.Team;

import javax.persistence.EntityManager;
import javax.transaction.Transactional;
import java.util.List;

import static org.assertj.core.api.Assertions.assertThat;

@SpringBootTest
@Transactional
class MemberJpaRepositoryTest {

    @Autowired EntityManager em;
    @Autowired MemberJpaRepository memberJpaRepository;

    // ...

    @Test
    @DisplayName("동적쿼리와 성능 최적화 조회 - Builder 사용")
    public void searchTest() {
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

        MemberSearchCondition condition = new MemberSearchCondition();
        condition.setAgeGoe(35);
        condition.setAgeLoe(40);
        condition.setTeamName("teamB");

        List<MemberTeamDto> result = memberJpaRepository.searchByBuilder(condition);

        assertThat(result).extracting("username").containsExactly("member4");
    }
}
```

## Note

### Issue

#### `Attempt to recreate a file for type...`

해당 에러가 발생하는 경우는 Q 객체를 생성해야 하는데 이미 폴더나 객체가 생성되어 있어 덮어쓸수 없을 때 발생

따라서 build 되었을 때 생성되는 파일, out으로 생성되는 파일을 지우고 다시 실행하면 정상 동작한다.

* `./gradlew clean`
* `./gradlew compileQuerydsl` 혹은 `./gradlew compileJava`