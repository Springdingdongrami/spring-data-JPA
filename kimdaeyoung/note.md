# 섹션 1. 프로젝트 환경설정

## 엔티티 생성 시 기본 생성자는 protected로 설정한다.

- JPA에서 프록시 기술을 쓸 경우에 private으로 설정할 경우 못할 수 있기 때문이다.

## 영속성 컨텍스트 - 1차 캐시

- MemberJpaRepositoryTest.java

```java
import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.annotation.Rollback;
import org.springframework.transaction.annotation.Transactional;
import study.datajpa.entity.Member;

@SpringBootTest
@Transactional
@Rollback(false)
class MemberJpaRepositoryTest {

    @Autowired
    MemberJpaRepository memberJpaRepository;

    @Test
    public void testMember() throws Exception {
        //given
        Member member = new Member("member1");
        Member member1 = memberJpaRepository.save(member);

        //when
        Member findMember = memberJpaRepository.find(member.getId());

        //then
        Assertions.assertThat(findMember.getId()).isEqualTo(member1.getId());
        Assertions.assertThat(findMember.getUsername()).isEqualTo(member1.getUsername());
        Assertions.assertThat(findMember).isEqualTo(member1);
    }

}
```

- Member.java

```java
import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.Id;
import lombok.Getter;

@Entity
@Getter
public class Member {

    @Id
    @GeneratedValue
    private Long id;

    private String username;

    protected Member() {
    }

    public Member(String username) {
        this.username = username;
    }

    public void changeUsername(String username) {
        this.username = username;
    }

```

- `MemberJpaRepositoryTest.java`의  `testMember()`에서`Assertions.assertThat(findMember).isEqualTo(member1);`를 보면 isEqualTo를 사용하였다. 이 메서드는 단순히 객체 두개가 같은지를 확인 하는 것이다. 즉 findMember == member1을 확인한다. 따라서 Member.class를 보면 hashCode override를 하고 있지 않아 다른 객체이므로 False가 나와야 한다고 예상된다. 하지만 True로 나온다. 이 이유는 `testMember()` 가 한 Transaction에 묶여 있는 경우, ORM의 1차 캐시에 의해 같은 객체로 인식하기 때문이다. 위 코드를 예로 설명하자면 먼저 member 객체를 생성하여 save()를 통해 저장하게 되면 persist()에 의해 1차 캐깃에 member 객체가 저장된다. 그 후 find()로 조회하게 되면 먼저 영속성 컨텍스트의 1차 캐시에 엔티티가 존재하는지 확인하는 과정을 거친다. 여기선 존재하므로 같은 객체가 반환되는 것이다.

<br><br><br><br>


