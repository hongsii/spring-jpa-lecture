# 백기선 - 스프링 데이터 JPA 강의 정리 

# 관계형데이터베이스(PostgreSQL) 설치

도커를 사용해 실습용 데이터베이스 설치

* docker run : 컨테이너 실행 명령어
    * -p : 호스트의 포트와 컨테이너의 포트 매핑 [host-port:container-port]
    * -e : 리눅스 환경변수 설정(key=value)
    * --name : 컨테이너명 설정
    * -d : 컨테이너를 실행할 이미지명 설정
* docker exec : 컨테이너 접속
    * -i : 표준 입력 유지
    * -t : 접속할 컨테이너명


``` bash
$ docker run -p 5432:5432 -e POSTGRES_DB=springdata -e POSTGRES_USER=hong -e POSTGRES_PASSWORD=hong123 --name postgres_jpa_lecture -d postgres

$ docker exec -i -t postgres_jpa_lecture bash
```

PostgreSQL 접속 확인

``` bash
$ su postgres
$ psql springdata -U hong
```

# JDBC

Java와 데이터베이스의 연결고리

## 단점

* SQL을 실행하는 비용이 비싸다.
* 벤더마다 SQL이 다르다.
* 동일한 내용이 반복되어 중복된 코드가 많다.

-> **이러한 JDBC의 단점을 해결하기 위해 ORM이 생기게 됨**

# ORM (Object-Relation Mapping)

Domain 모델을 기반으로 코딩을 하며, 해당 객체를 데이터베이스에 저장하는 형태

``` java
Accounet account = new Account("hong", "hong123");
accountRepository.save(account);
```

JDBC 보다 Domain 모델을 사용하려는 이유

* Domain 모델을 사용하기 때문에 객체지향적으로 코딩이 가능하다.
* 객체지향적으로 코딩을 하기 때문에 각종 디자인 패턴 및 코드 재사용이 가능하다.
* 비즈니스 로직 구현 및 테스트가 쉽다.

장점

* 도메인 코드를 사용하기 때문에 코드가 간결해져 생산성이 높아짐
* 유지보수가 쉬움
* 성능이 좋음
** 하나의 쿼리의 경우 직접 작성한 쿼리보다 느릴 수 있음
** 객체와 데이터베이스 사이에 캐시가 존재해 같은 트랜잭션 내의 연관 관계에 따라 캐시를 통해 데이터베이스를 조회하지 않기 때문에 성능이 더 좋음
* 벤더에 종속적이지 않음
** RDB에 따라 언어가 조금씩 다른 경우가 존재하지만, 하이버네이트가 RDB에 맞춰 쿼리를 생성해줌(dialect - 방언) 

단점

* 학습 비용이 매우 높음
** 하이버네이트가 쿼리를 만들어준다고 해도 쿼리를 잘 알아야 잘 쓸수 있음
** 학습이 적절하게 되지 않는다면 성능 문제로 인해 ORM을 포기하게될 수도 있음



# ORM 패러다임 불일치

밀도 문제

* 객체는 다양한 크기의 객체를 만들 수 있고 커스텀 타입(클래스)을 만들기 쉬움
* 릴레이션은 테이블, 데이터 타입 두 가지가 존재 (UDT도 있지만, 잘 사용되지 않음)

서브타입 문제

* 객체는 상속 구조를 만들기 쉬움 (다형성으로 참조)
* 릴레이션은 테이블 상속이 없음 (다형성을 표현하기 어려움)

식별성 문제

* 릴레이션
    * 기본키(Primary key)

* 객체
    * 기본키와 같은 개념이 없음
    * 레퍼런스 동일성 (==)
    * 인스턴스 동일성 (equals() 메소드)

관계 문제

* 릴레이션
    * 외래키(Foreign key)로 관계 표현
    * "방향"이 없음 -> Join을 통해 아무렇게나 묶을 수 있음
    * 다대다 관계를 만들 수 없고, 조인 테이블 또는 링크 테이블을 사용해 두 개의 1대다 관계로 해결해야함
* 객체
    * 객체 레퍼런스로 관계 표현
    * 근본적으로 "방향"이 존재 (일대일, 일대다, 다대다)

데이터 네비게이션 문제

* 릴레이션
    * Connection을 만드는 비용이 비싸기 때문에 데이터베이스를 자주 접근하면 비효율적
    * 한 번에 데이터를 가져오기 위해 Join을 많이 하는 것도 쓸모 없는 데이터를 많이 메모리에 적재하기 때문에 좋지 않음

* 객체
    * 레퍼런스를 통해 다른 객체로 이동 가능
    * getOwner().getMyStudyList().stream().forEach(s -> s.getOwner());
    * Lazy loading을 통해 필요할 때마다 select를 하게 되면 (n+1 select)로 


# 엔티티(Entity) 관련 어노테이션 정리

패키지 import시 javax.persistence

* `@Entity`
   - 도메인 객체로 설정
   - 보통 클래스와 같은 명칭으로 설정 (변경하고 싶다면 `name` 속성으로 변경 가능)
   - 엔티티명은 JQL에서 사용
   
* `@Table`
   - 객체를 테이블로 맵핑
   - 기본적으로 클래스명으로 테이블이 생성
   - @Entity를 설정시 생략 가능 (추가 옵션이 필요한 경우에 명시)
   
* `@Id`
   - 기본키 설정
   - primitive 타입과 wrapper 타입으로 사용 가능
      - primitive 타입일 경우 초기화값이 존재하기 때문에 0이라는 id값이 존재할 경우 중복될 수 있기 때문에 주로 wrapper 타입을 사용
    
* `@GeneratedValue`
   - 기본키 생성 방식 설정
   - 생성 전략과 생성기를 설정
   - 기본 전략은 AUTO : 사용하는 DB에 따라 적절한 전략 선택
      - TABLE, SEQUENCE, IDENTITY 중 하나 선택
    
* `@Column`
   - 프로퍼티를 컬럼으로 맵핑 및 속성 설정
   - @Entity 설정시 속성들이 Column으로 인식되기 때문에 생략 가능
    
* `@Temporal`
   - Date, Calander만 지원
   - JPA 2.2부터는 LocalDateTime 등 Java8의 새로운 타입 사용 가능

* `@Transient`
    - 프로퍼티를 컬럼으로 맵핑하지 않고 객체 내에서만 사용하도록 설정
    

## JPA 옵션

* `spring.jpa.show-sql=true`
   - true(SQL 표시), false(SQL 미표시)
   - JPA로 실행되는 SQL을 콘솔에 출력
   - 실제 실행되는 SQL을 확인하고 싶다면 logger로 설정
   
* `spring.jpa.properties.hibernate.format_sql=true`
   - true(SQL 정렬), false(SQL 미정렬)
   - 콘솔에 출력되는 SQL을 보기 쉽게 출력
   
   
# Value 관련 어노테이션 정리

Entity에 종속적인 타입

* `@Embeddable`
   - 클래스에 어노테이션 선언
   - 해당 어노테이션이 있는 클래스를 Entity에 넣어줄 때 사용
   
* `@Embedded`
   - 프로퍼티에 어노테이션 선언
   - Embeddable 클래스를 Entity에 맵핑하고 싶을 때 해당 어노테이션 사용
   - Entity 내에 동일한 Embeddable 클래스의 프로퍼티를 두 가지 이상 선언하고 싶다면 `@AttributeOverrides` 사용
* `@AttributeOverrides`, `@AttributeOverride`
   - name, column 필수 선언
      - name : Embeddable 클래스의 프로퍼티 명칭
      - column : 테이블에 설정될 컬럼명
``` java
@Embeddable
public class Address {
   private String address;
   private String zipcode;
   
}

@Entity
public class Account {
   @Id @GeneratedValue
   private Long id;
   
   @Embedded
   @AttributeOverrides({
      @AttributeOverride(name = "street", column = @Column(name = "home_street"))
   })
   private Address address;
}
```

# 관계

두 개의 엔티티끼리 관계를 맺을 수 있다.

## 일대다

단방향

`@ManyToOne`

* 프로퍼티가 하나일 때, 해당 어노테이션을 선언
* 프로퍼티의 PK가 FK가 됨

`@OneToMany (단방향)`

* 프로퍼티가 Collection타입과 같이 여러 개를 가지는 경우 어노테이션 선언
* 관계를 저장하는 테이블이 생성됨


아래는 두 개의 엔티티가 서로 단방향으로만 관계를 맺음

``` java
@Entity
public class Owner {
    
    @Id @GeneratedValue
    private Long id;

    private String name;

    @OneToMany
    privat Set<Study> studies;
}

@Entity
public class Study {

    @Id @GeneratedValue
    private Long id;

    private String name;

    // owner 테이블의 PK(id)가 FK(owner_id)가 된다
    // 주인은 Study (주인 : 관계를 설정했을 때, 값이 반영되는 경우)
    @ManyToOne
    private Owner owner;
}
```

양방향

양방향의 관계를 맺으려면 주인 엔티티의 프로퍼티명을 지정해줘야함.
`@OneToMany(mappedBy = "관계의 주인테이블의 프로퍼티명")`

``` java
@Entity
public class Owner {
    
    @Id @GeneratedValue
    private Long id;

    private String name;

    @OneToMany(mappedBy = "owner")
    privat Set<Study> studies;
}

@Entity
public class Study {

    @Id @GeneratedValue
    private Long id;

    private String name;

    // owner 테이블의 PK(id)가 FK(owner_id)가 된다
    // 주인은 Study (주인 : 관계를 설정했을 때, 값이 반영되는 곳)
    @ManyToOne
    private Owner owner;
}
```

양방향이기 때문에 각각의 엔티티에 모두 반영해줘야한다.


```
Study study = new Study();
Owner owner = new Owner();
owner.getStudies().add(study);
study.setOwner(owner);
```

# 엔티티의 상태와 Cascade

Cascade : 관계 어노테이션의 속성

* 연관된 엔티티로 변경된 엔티티의 상태 변화를 전파
* 4가자의 상태가 존재
    * Transient
        - JPA가 객체에 대해 전혀 모르는 상태 (객체의 저장여부를 모르는 상태)
    * Persistent
        - session의 save() 메소드가 호출되면 JPA에서 관리하는 객체가 됨
        - insert 쿼리가 바로 발생되지 않음
        - Persistent Context에 인스턴스가 추가되어 1차 캐시가 됨
        - Persistent 상태의 인스턴스를 조회하게 되면 DB에서 조회하지 않고 1차 캐시 상태의 인스턴스가 조회됨
        - 트랜잭션이 끝나면 insert 쿼리가 발생함
        - 객체의 변경사항이 모니터링되고 변경이 생기면 변경사항이 반영됨
            - Dirty checking : 변경사항을 계속 감지
            - Write behind : 객체의 상태 변화를 필요한 시점에 반영
    * Detached
        - 트랜잭션이 종료되고 난 뒤 변경되는 상태
        - JPA에 의해 관리되지 않음
        - 다시 관리가 필요하면 redetached 되어 persistent 상태가 된다
    * Removed


부모-자식간의 관계일 때, cascade를 지정하면 부모가 반영될 때, 부모의 상태가 자식에도 전파된다.

Cascade.REMOVED 옵션이 걸려있다면 부모 엔티티를 삭제시키면 REMOVED 상태가 전파되어 자식 엔티티도 같이 삭제가 된다.

# Fetch

연관 관계의 엔티티를 언제 가져올 지 결정하는 옵션

`@OneToMany(mappedBy = "owner", ... , fetch = FetchType.LAZY)` fetch 옵션이 생략되어 있다면 **기본적으로  LAZY**가 적용됨

반대로 `@ManyToOne`의 경우는 가져와야할 값이 하나밖에 없기 때문에 기본적으로 EAGER가 적용되어있음


# JPA

6-7년 전에는 아래와 같이 구현해서 사용

``` java
@Repository
@Transactional
public class PostRepository {
    
    @PersistenceContext
    EntityManager entityManager;

    public Post add(Post post) {
        return entityManager.persis(post);
    }

    public void delete(Post post) {
        entityManager.remove(post);
    }

    public List<Post> findAll() {
        return entityManager.createQuery("SELECT p FROM Post As p")
                .getResultList();
    }
}
```

* Spring Data JPA에서는 **인터페이스만 만들어도 되도록 개발**되어 옛날보다 훨씬 편하게 사용 가능
* `@Repository`를 선언하지 않아도 `JpaRepositoriesRegistrar`에서 빈으로 등록해줌
* 기본적인 CRUD 메소드를 제공하기 때문에 별다른 코드를 작성할 필요가 없음

``` java
// JpaRepository<Entity, id타입>
public interface PostRepository extends JpaRepository<Post, Long> {

}
```

# 2부 스프링 데이터 JPA 활용

[Spring Data](http://projects.spring.io/spring-data/는 여러 프로젝트로 구성되어있음.

* Spring Data Common : 여러 저장소 지원 프로젝트의 공통 기능 제공
* Spring Data REST : 저장소 데이터를 HTTP 리소스로 제공하는 프로젝트
* SPring Data JPA : Common의 확장판 (JPA 관련 기능 추가)

## Spring Data Common


`JpaRepository`는 JPA 프로젝트에 존재하며,  `PagingAndSortingRepository`를 구현한 인터페이스

아래 인터페이스는 Common에 속해있는 인터페이스

* Repository<T, ID>
* CrudRepository<T, ID> extends Repository<T, ID>
    - 기본적인 CRUD 메소드가 구현되어있음
* PagingAndSortingRepository extends CrudRepository<T, ID>

### 쿼리 생성 방법

`@EnableJpaRepositories(queryLookupStrategy = ...)`의 queryLookupStrategy로 쿼리 생성 방법을 지정할 수 있음

* Key.CREATE : 메소드명을 분석해서 쿼리를 만들어줌
* Key.USE_DECLARED_QUERY : `@Query` 어노테이션으로 JPQL 작성 - 구현체마다 다름
* Key.CREATE_IF_NOT_FOUND : 미리 정의한 @Query를 찾고 없으면 메소드명을 분석해 생성

#### 쿼리 만드는 방법

리턴타입 {접두어}{도입부}By{프로퍼티 표현식}{조건식}{And | Or}{프로퍼티 표현식}{조건식}{정렬 조건}{매개변수}

* 접두어 : Find, Get, Query, Count, ...
* 도입부 : Distinct, First(N), Top(N)
* 프로퍼티 표현식 : Person, Address, ZipCode => findPersonByAddress_ZipCode(...)
* 조건식 : IgnoreCase, Between, LessThan, GreaterThan, Like, Contains, ...
* 정렬 조건 : OrderBy{프로퍼티}Asc|Desc
* 리턴 타입 : E, Optional<E>, List<E>, Page<E>, Slice<E>, Stream<E>
* 매겨변수 : Pageable, Sort

#### 미리 선언된 쿼리 찾는 방법

* 메소드명으로 쿼리를 표현하기 힘든 경우에 `@Query` 사용
* 저장소 기술마다 구현 방법이 다름
* JPA : @Query @NamedQuery

