# Querydsl

## 2. 예제 도메인 모델

### 2-1. 예제 도메인 모델과 동작 확인

![](https://i.ibb.co/K9fY8mQ/bandicam-2021-07-27-19-05-56-370.jpg)

#### Member.java - Member 엔티티

* `src/main/java/study/querydsl/entity/Member.java`

```java
package study.querydsl.entity;

import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;
import lombok.ToString;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.JoinColumn;
import javax.persistence.ManyToOne;

import static javax.persistence.FetchType.LAZY;
import static lombok.AccessLevel.PROTECTED;

@Entity
@Getter @Setter
@NoArgsConstructor(access = PROTECTED)
@ToString(of = {"id", "username", "age"})
public class Member {

    @Id @GeneratedValue
    @Column(name = "member_id")
    private Long id;
    private String username;
    private int age;

    @ManyToOne(fetch = LAZY)
    @JoinColumn(name = "team_id")
    private Team team;

    public Member(String username) {
        this(username, 0);
    }

    public Member(String username, int age) {
        this(username, age, null);
    }

    public Member(String username, int age, Team team) {
        this.username = username;
        this.age = age;
        if (team != null) {
            changeTeam(team);
        }
    }

    public void changeTeam(Team team) {
        this.team = team;
        team.getMembers().add(this);
    }
}

```

* 롬복 설명
    * `@Setter`: 실무에서 가급적 Setter는 사용하지 않기
    * `@NoArgsConstructor(access = AccessLevel.PROTECTED)`: 기본 생성자 막고 싶은데, JPA 스팩상 PROTECTED로 열어두어야 함
    * `@ToString`은 가급적 내부 필드만(연관관계 없는 필드만)
* `changeTeam()`으로 양방향 연관관계 한번에 처리(연관관계 메소드)

#### Team.java - Team 엔티티

* `src/main/java/study/querydsl/entity/Team.java`

```java
package study.querydsl.entity;

import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;
import lombok.ToString;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.OneToMany;
import java.util.ArrayList;
import java.util.List;

import static lombok.AccessLevel.PROTECTED;

@Entity
@Getter @Setter
@NoArgsConstructor(access = PROTECTED)
@ToString(of = {"id", "name"})
public class Team {

    @Id @GeneratedValue
    private Long id;
    private String name;

    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>();

    public Team(String name) {
        this.name = name;
    }
}

```

* Member와 Team은 양방향 연관관계, `Member.team`이 연관관계 주인, `Team.members`는 연관관계의 주인이 아님, 따라서 `Member.team`이 데이터베이스 외래키 값을 변경,
  반대편은 읽기만 가능

#### MembetTest.java - 데이터 확인 테스트

* `src/test/java/study/querydsl/entity/MemberTest.java`

```java
package study.querydsl.entity;

import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.transaction.annotation.Transactional;

import javax.persistence.EntityManager;
import java.util.List;

@SpringBootTest
@Transactional
class MemberTest {

    @Autowired EntityManager em;

    @Test
    @DisplayName("test")
    public void testEntity() {
        // given
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

        em.flush();
        em.clear();

        // when
        List<Member> members = em.createQuery("select m from Member m", Member.class)
                .getResultList();

        // then
        for (Member member : members) {
            System.out.print("member = " + member);
            System.out.println(" -> member.Team = " + member.getTeam());
        }
    }
}
```

* 가급적 순수 JPA로 동작 확인(뒤에서 변경)
* db 테이블 결과 확인
* 지연 로딩 동작 확인

## Note