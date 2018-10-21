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
