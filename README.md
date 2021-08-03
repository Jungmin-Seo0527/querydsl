# Querydsl

## 6. 실무 활용 - 스프링 데이터 JPA와 Querydsl

### 6-1. 스프링 데이터 JPA 리포지토리로 변경

#### MemberRepository.java - 스프링 데이터 JPA

* `src/main/java/study/querydsl/repository/MemberRepository.java`

```java
package study.querydsl.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import study.querydsl.entity.Member;

import java.util.List;

public interface MemberRepository extends JpaRepository<Member, Long> {

    List<Member> findByUsername(String username);
}

```

#### MemberRepositoryTest.java - 스프링 데이터 JPA 테스트

* `src/test/java/study/querydsl/repository/MemberRepositoryTest.java`

```java
package study.querydsl.repository;

import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import study.querydsl.entity.Member;

import javax.persistence.EntityManager;
import javax.transaction.Transactional;
import java.util.List;

import static org.assertj.core.api.Assertions.assertThat;

@SpringBootTest
@Transactional
class MemberRepositoryTest {
    @Autowired EntityManager em;
    @Autowired MemberJpaRepository memberRepository;

    @Test
    @DisplayName("Spring Data JPA 적용 테스트")
    public void basicTest() {
        Member member = new Member("member1", 10);
        memberRepository.save(member);

        Member findMember = memberRepository.findById(member.getId()).orElseThrow();
        assertThat(member).isEqualTo(findMember);

        List<Member> result1 = memberRepository.findAll();
        assertThat(result1).containsExactly(member);

        List<Member> result2 = memberRepository.findByUsername("member1");
        assertThat(result2).containsExactly(member);
    }

}

```

### 6-2. 사용자 정의 리포지토리

* 사용자 정의 리포지토리 사용법
    1. 사용자 정의 인터페이스 작성
    2. 사용자 정의 인터페이스 구현
    3. 스프링 데이터 리포지토리에 사용자 정의 인터페이스 상속

#### 사용자 정의 리포지토리 구성

![](https://i.ibb.co/HHf2pkd/bandicam-2021-08-03-15-41-48-658.jpg)

#### MemberRepositoryCustom.java - 사용자 정의 인터페이스

* `src/main/java/study/querydsl/repository/MemberRepositoryCustom.java`

```java
package study.querydsl.repository;

import study.querydsl.dto.MemberSearchCondition;
import study.querydsl.dto.MemberTeamDto;

import java.util.List;

public interface MemberRepositoryCustom {
    List<MemberTeamDto> search(MemberSearchCondition condition);
}

```

#### 사용자 정의 인터페이스 구현

* `src/main/java/study/querydsl/repository/MemberRepositoryImpl.java`

```java
package study.querydsl.repository;

import com.querydsl.core.types.dsl.BooleanExpression;
import com.querydsl.jpa.impl.JPAQueryFactory;
import study.querydsl.dto.MemberSearchCondition;
import study.querydsl.dto.MemberTeamDto;
import study.querydsl.dto.QMemberTeamDto;

import javax.persistence.EntityManager;
import java.util.List;

import static org.springframework.util.StringUtils.hasText;
import static study.querydsl.entity.QMember.member;
import static study.querydsl.entity.QTeam.team;

public class MemberRepositoryImpl implements MemberRepositoryCustom {

    private final JPAQueryFactory queryFactory;

    public MemberRepositoryImpl(EntityManager em) {
        this.queryFactory = new JPAQueryFactory(em);
    }

    @Override
    public List<MemberTeamDto> search(MemberSearchCondition condition) {
        return queryFactory
                .select(new QMemberTeamDto(
                        member.id,
                        member.username,
                        member.age,
                        team.id,
                        team.name))
                .from(member)
                .leftJoin(member.team, team)
                .where(usernameEq(condition.getUsername()),
                        teamNameEq(condition.getTeamName()),
                        ageGoe(condition.getAgeGoe()),
                        ageLoe(condition.getAgeLoe()))
                .fetch();

    }

    private BooleanExpression usernameEq(String username) {
        return hasText(username) ? member.username.eq(username) : null;
    }

    private BooleanExpression teamNameEq(String teamName) {
        return hasText(teamName) ? team.name.eq(teamName) : null;
    }

    private BooleanExpression ageGoe(Integer ageGoe) {
        return ageGoe != null ? member.age.goe(ageGoe) : null;
    }

    private BooleanExpression ageLoe(Integer ageLoe) {
        return ageLoe != null ? member.age.loe(ageLoe) : null;
    }
}

```

#### MemberRepository.java - 스프링 데이터 리포지토리에 사용자 정의 인터페이스 상속

```java
package study.querydsl.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import study.querydsl.entity.Member;

import java.util.List;

public interface MemberRepository extends JpaRepository<Member, Long>, MemberRepositoryCustom {

    List<Member> findByUsername(String username);
}

```

## Note