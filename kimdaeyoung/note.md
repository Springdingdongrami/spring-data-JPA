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

# 섹션 2. 예제 도메인 모델

## @GeneratedValue

- jpa와 mysql 사용시 `@GeneratedValue(strategy = GenerationType.IDENTITY)` 사용하기
    - default값은 `GenerationType.Auto` 이다
    

## 양방향 매핑시 외래키가 없는 쪽에 mappedBy 작성

- 외래키를 가지고 있는 쪽이 연관관계의 주인이고 이 경우 mappedBy를 사용하면 안된다.
- mappedBy를 통해 주인이 아님을 설정하고, 해당 속성 값은 연관관계 주인의 해당 속성 필드명과 일치해야 한다.
- mappedBy를 사용하지 않으면 다대일 관계의 경우 중간 테이블이 생성되고, 일대일 관계의 경우 각각의 테이블에 서로를 참조하는 FK가 설정된다.
- 데이터베이스에서 일대다, 다대일 관계에서 다쪽이 외래키를 가지고 있다.
    - 이유는 데이터베이스에서는 컬렉션을 담을 수 없기 때문에 일쪽에서 여러 외래키를 가질 수 없다.
- 연관관계의 주인 - 외래키 등록, 수정, 삭제 가능
- 연관관계의 주인 아님 - 외래 키 읽기만 가능

## 양방향 매핑시 toString을 주의하자!

- 양방향 매핑 엔티티에서 toString을 사용할 때, 매핑한 다른 엔티티를 toString에 쓸 경우 순환 참조가 일어난다.

<br><br><br><br>

# 섹션 3. 공통 인터페이스 기능

## `@EnableJpaRepositories(basePackages = “폴더 위치”)`

- 스프링 부트 사용 시 `@SpringBootApplication` 위치 해당 패키지와 하위 패키지는 자동으로 인식함 → 이외 위치에 사용해야 할 때 사용하는 어노테이션

 

## Jpa Interface 주요 메서드

- save() : 새로운 엔티티는 저장, 이미 있는 엔티티는 병합
- delete() : 엔티티 하나 삭제. 내부에서 EntityManager.remove() 호출
- findById() : 엔티티 하나 조회. 내부에서 EntityManager.find() 호출
- getOne() : 엔티티를 프록시로 조회. 내부에서 EntityManager.getReference() 호출
- findAll() : 모든 엔티티 조회. 정렬이나 페이징 조건을 파라미터로 제공할 수 있다.

<br><br><br><br>

