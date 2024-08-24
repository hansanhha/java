spring boot 3.3.2 기준

[Spring Data Project](#spring-data-project)
- [주요 기능](#주요-기능)
- [메인 모듈](#메인-모듈)
- [Common 모듈 분석](#common-모듈-분석)
  - [Repository 추상화](#repository-abstraction)
  - [Domain](#domain)
  - [Util](#util)


## Spring Data Project

[Spring Data](https://spring.io/projects/spring-data)

스프링 데이터는 기본 데이터 저장소의 특수성을 유지하면서 데이터 접근을 위한 일관성 있는 스프링 기반 프로그래밍 모델을 제공함

관계형, 비관계형 데이터베이스, 데이터 접근 기술, 클라우드 기반 데이터 서비스, 맵 리듀스 프레임워크를 쉽게 사용할 수 있음

스프링 데이터 프로젝트는 특정 데이터베이스에 특화된 하위 프로젝트를 포함하는 포괄적인 프로젝트임

### 주요 기능

#### 리포지토리 패턴, 커스텀 객체 매핑 추상화
- 기본적인 CRUD 작업을 쉽게 처리할 수 있도록 추상화 제공
- 커스텀 객체 매핑도 지원
- ```java
  public interface UserRepository extends JpaRepository<User, Long> {}
  ```

#### 리포지토리 메서드 이름에서 동적 쿼리 파생
- 인터페이스의 메서드 이름을 기반으로 쿼리를 자동으로 생성함
- SQL을 직접 작성할 필요없이 쉽게 데이터 작업 가능
- ```java
  public interface UserRepository extends JpaRepository<User, Long> {
    List<User> findByEmailAndName(String email, String name);
  }
  ```

#### 기본 속성을 제공하는 구현 도메인 베이스 클래스
- 기본적인 속성을 제공하는 베이스 클래스를 통해 도메인 클래스의 반복 코드를 줄여줌
- ```java
  // 기본 속성 제공 베이스 클래스
  @MappedSuperclass
  @EntityListeners(AuditingEntityListener.class)
  public abstract class BaseEntity {
  
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @CreatedDate
    private LocalDateTime createdDate;

    @LastModifiedDate
    private LocalDateTime lastModifiedDate;
  }

  // 상속을 통해 기본 속성을 제공받음  
  @Entity
  public class User extends BaseEntity {
    private String name;
    private String email;
  }
  ```

#### auditing 지원(created, last changed 등)
- 엔티티 생성 및 수정 시간 등을 자동으로 기록하는 감사(auditing) 기능 제공
- ```java
  @Configuration
  @EnableJpaAuditing
  public class AuditConfig {
    Timestamp createdDate;
    Timestamp lastModifiedDate;
  }
  ```

#### 커스텀 리포지토리 코드 통합
- 기본 리포지토리 기능 외에도, 커스텀 리포지토리를 만들어 기존 리포지토리에 쉽게 통합할 수 있음
- ```java
  public interface UserRepositoryCustom {
    List<User> customLogic();
  }
  
  public class UserRepositoryCustomImpl implements UserRepositoryCustom {
    
    @PersistenceContext
    private EntityManager entityManager;
  
    @Override
    public List<User> customLogic() {
       // ...
    }
  }
  
  public interface UserRepository extends JpaRepository<User, Long>, UserRepositoryCustomImpl {
  }
  ```

#### JavaConfig 및 커스텀 XML 네임스페이스를 통한 스프링 통합
- JavaConfig를 통해 DataSource, 트랜잭션 매니저 등을 설정할 수 있음

#### Spring 컨트롤러와의 통합

### 메인 모듈

#### Spring Data Commons

모든 스프링 데이터 모듈의 기반이 되는 핵심 스프링 개념

#### Spring Data JDBC

JDBC에 대한 스프링 데이터 리포지토리

#### Spring Data JPA

JPA에 대한 스프링 데이터 리포지토리

#### Spring Data Redis

스프링 애플리케이션에서 간단하게 설정하여 레디스에 접근할 수 있는 모듈

## Common 모듈 분석

### Annotation

#### Auditing

`@CreatedBy`: 엔티티가 처음 생성될 때, 해당 엔티티를 생성한 사용자를 기록하기 위해 사용됨

`@LastModifiedBy`: 엔티티가 마지막으로 수정될 때, 해당 엔티티를 수정한 사용자를 기록하기 위해 사용됨

`@CreatedDate`: 엔티티가 처음 저장될 때, 생성 시간을 자동으로 기록하기 위해 사용됨

`LastModifiedDate`: 엔티티가 마지막으로 수정된 날짜와 시간을 자동으로 기록하기 위해 사용됨

#### Reference

`@Reference` 어노테이션은 데이터 저장소 간의 참조 관계를 나타내기 위해 사용됨

주로 Sprin Data REST 모듈에서 사용되며, 한 엔티티가 다른 엔티티를 참조할 때 해당 참조를 명시적으로 나타냄

#### Transient

`@Transient` 어노테이션은 특정 필드가 데이터베이스에 저장되지 않도록 표시함

엔티티에 포함되어 있어도, 데이터베이스 테이블에 매핑되지 않음

### Repository Abstraction

org.springframework.data.repository 패키지

#### Repository<T, ID>

@Indexed
- `@Indexed`는 spring 5.0에 추가된 어노테이션으로 컴포넌트 스캔을 최적화하는 용도로 사용됨
- 스프링이 클래스패스에서 컴포넌트를 검색할 때 인덱스를 생성하여, 컴포넌트 스캔의 성능을 향상시켜 애플리케이션 컨텍스트 초기화를 빠르게 할 수 있음
- `@Component`와 달리 빈 등록 여부를 결정하진 않음
- 스프링 데이터의 최상위 인터페이스인 Repository에 `@Indexed`를 적용시켜, 컴포넌트 스캔 시 인덱스 정보를 활용할 수 있도록 함

Repository 인터페이스는 관리할 도메인 타입과 ID 타입을 캡처하는 마커 인터페이스임

```java
@Indexed
public interface Repository<T, ID> {
}
```

#### CrudRepository<T, ID>

@NoRepositoryBean
- CrudRepository나 JpaRepository 같이 공통적인 메서드를 제공하는 리포지토리를 기반 리포지토리(base repository)라고 하는데,
- `@NoRepositoryBean` 어노테이션은 기반 리포지토리에 선언해서, 해당 인터페이스 자체가 스프링 빈으로 등록되지 않게 함
- 이를 구현한 구체적인 리포지토리 인터페이스만 빈으로 등록하게 됨

CrudRepository 인터페이스는 특정 타입에 대한 일반적인 CRUD 연산을 추상화함

```java
@NoRepositoryBean
public interface CrudRepository<T, ID> extends Repository<T, ID> {
    
    // T의 하위 타입 허용
    <S extends T> S save(S entity);

    // T의 하위 타입 허용
    <S extends T> Iterable<S> saveAll(Iterable<S> entities);
    
    Optional<T> findById(ID id);
    
    boolean existsById(ID id);
    
    Iterable<T> findAll();
    
    Iterable<T> findAllById(Iterable<ID> ids);
    
    long count();
    
    void deleteById(ID id);
    
    void delete(T entity);
    
    void deleteAllById(Iterable<? extends ID> ids);
    
    void deleteAll(Iterable<? extends T> entities);
    
    void deleteAll();
}
```

#### ListCrudRepository<T, ID>

CrudRepository를 확장한 인터페이스로, Iterable 대신 List로 반환하는 메서드를 정의함

빈 등록 방지를 위해 `@NoRepositoryBean`을 적용함

```java
@NoRepositoryBean
public interface ListCrudRepository<T, ID> extends CrudRepository<T, ID> {
    
    <S extends T> List<S> saveAll(Iterable<S> entities);

    List<T> findAll();

    List<T> findAllById(Iterable<ID> ids);
}
```

#### PagingAndSortingRepository<T, ID>

페이징과 정렬을 추상화한 리포지토리

PagingAndSortingRepository 인터페이스는 Repository를 확장하므로

이를 확장할 인터페이스나 구현체는 기본적인 Crud 연산을 위해 CrudRepository도 포함해야 됨

빈 등록 방지를 위해 `@NoRepositoryBean`을 적용함

```java
@NoRepositoryBean
public interface PagingAndSortingRepository<T, ID> extends Repository<T, ID> {
    
    // 주어진 정렬 조건에 따른 엔티티 반환
	Iterable<T> findAll(Sort sort);
    
    // 주어진 페이징 조건에 따른 엔티티 반환
	Page<T> findAll(Pageable pageable);
}
```

#### ListPagingAndSortingRepository<T, ID>

ListCrudRepository처럼 Iterable 대신 List를 반환하는 PagingAndSortingRepository 확장 인터페이스

빈 등록 방지를 위해 `@NoRepositoryBean`을 적용함

```java
@NoRepositoryBean
public interface ListPagingAndSortingRepository<T, ID> extends PagingAndSortingRepository<T, ID> {

	List<T> findAll(Sort sort);
}
```

#### @RepositoryDefinition

스프링 데이터 표준 리포지토리를 상속받지 않고, 특정한 리포지토리 기능을 직접 정의할 수 있도록 하는 어노테이션

Repository처럼 @Indexed가 포함되어 있으며, T, ID를 어노테이션 속성으로 지정함 

```java
@Indexed
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Inherited
public @interface RepositoryDefinition {

	Class<?> domainClass();

	Class<?> idClass();
}
```

아래처럼 표준 리포지토리 인터페이스를 상속받지 않고, 인터페이스 메서드를 직접 정의할 수 있음

```java
@RepositoryDefinition(domainClass = User.class, idClass = Long.class)
public interface UserRepository {
    User findByEmail(String email);
    List<User> findAll();
}
```

### Domain

org.springframework.data.domain 패키지

#### Slice

데이터의 일부분을 페이지네이션하기 위한 용도로 사용되는 인터페이스

Page 인터페이스와 유사하지만 Page와 달리 총 데이터 개수나 전체 페이지 수에 대한 정보를 제공하지 않음

데이터의 전체 목록을 메모리에 로드하지 않기 때문에 Page에 비해 효율적으로 메모리를 사용함

주요 특징
- 부분 데이터 제공
- 다음 페이지 존재 여부

```java
public interface Slice<T> extends Streamable<T> {
    
    // 현재 Slice의 번호
    int getNumber();
    
    // Slice 크기
    int getSize();
    
    // 현재 Slice의 요소 개수
    int getNumberOfElements();
    
    // 현재 Slice에 포함된 데이터 반환
    List<T> getContent();
    
    boolean hasContent();
    
    Sort getSort();
    
    boolean isFirst();
    
    boolean isLast();
    
    boolean hasNext();
    
    boolean hasPrevious();
    
    // 현재 Slice를 요청하는 데 사용된 Pageable 반환
    default Pageable getPageable() {
        return PageRequest.of(getNumber(), getSize(), getSort());
    }
    
    // 다음 Slice를 요청하기 위한 Pageable  
    Pageable nextPageable();
    
    // 이전 Slice를 요청하기 위한 Pageable
    Pageable previousPageable();
    
    <U> Slice<U> map(Function<? super T, ? extends U> converter);

    // 현재 Slice가 마지막이라면 현재 Pageable, 아니라면 nextPageable() 반환
    default Pageable nextOrLastPageable() {
        return hasNext() ? nextPageable() : getPageable();
    }
    // 현재 Slice가 처음이라면 현재 Pageable, 아니라면 previousPageable() 반환
    default Pageable previousOrFirstPageable() {
        return hasPrevious() ? previousPageable() : getPageable();
    }
}
```

#### Page

다음 페이지나 이전 페이지의 존재의 정보만을 제공하는 Slice와 달리, 전체 페이지 수와 전체 데이터 개수를 제공하는 Slice 확장 인터페이스

```java
public interface Page<T> extends Slice<T> {
    
    // 빈 Page 생성
    static <T> Page<T> empty() {
        return empty(Pageable.unpaged());
    }
    
    // 주어진 pageable에 따른 빈 Page 생성
    static <T> Page<T> empty(Pageable pageable) {
        return new PageImpl<>(Collections.emptyList(), pageable, 0);
    }
    
    // 전체 페이지 개수
    int getTotalPage();
    
    // 전체 데이터 개수
    long getTotalElements();

    <U> Page<U> map(Function<? super T, ? extends U> converter);
}
```

#### Sort

쿼리의 정렬 정보를 다루는 클래스

데이터베이스나 다른 저장소에서 데이터를 쿼리할 때 정렬 기준을 정의함

##### 중첩 클래스
- Direction
  - 정렬 방향을 나타내는 enum 
  - ```java
    public enum Direction {
        ASC, DESC;
    }
    ```
- NullHandling
  - 정렬 시 null 값을 어떻게 처리할 지 정의하는 enum
  - ```java
    public enum NullHandling {
        /*
            DataSource의 기본 null 처리 방식을 따름
            대부분의 데이터베이스에서 null 값은 마지막에 정렬됨
        */
        NATIVE,
        // null 값을 가장 먼저 정렬
        NULLS_FIRST,
        // null 값을 가장 나중에 정렬
        NULLS_LAST;
    }
    ``` 
- Order
  - 특정 필드에 대해 정렬 방향과 null 처리 방식을 포함한 정렬 정보를 정의하는 클래스
  - Sort 객체는 여러 개의 Order 객체를 포함할 수 있음
  - ```java
    public static class Order implements Serializable {
    
        private static final boolean DEFAULT_IGNORE_CASE = false;
		private static final NullHandling DEFAULT_NULL_HANDLING = NullHandling.NATIVE;
    
        private final Direction direction;
		private final String property;
        // 대소문자 구분, 기본값은 대소문자 구분 안함
		private final boolean ignoreCase;
        // null처리, 기본값은 데이터베이스 처리 방식을 따름
		private final NullHandling nullHandling;
    
        public 
    
        // 특정 속성을 ASC 정렬
        public static Order by(String property) {
            return new Order(DEFAULT_DIRECTION, properpty);
        }
    
        // 특정 속성을 ASC 정렬
        public static Order asc(String property) {
            return new Order(Direction.ASC, property);
        }
    
        // 특정 속성을 DESC 정렬
        public static Order desc(String property) {
            return new Order(Direction.DESC, property);
        }
    
        // 기존 속성의 정렬 방향 수정
        public Order with(Direction direction) {
			return new Order(direction, this.property, this.ignoreCase, this.nullHandling);
		}
    
        // 기존 속성의 정렬 방향 반대로 수정
        public Order reverse() {
			return with(this.direction == Direction.ASC ? Direction.DESC : Direction.ASC);
		}
            
        // null 처리 수정 등의 메서드 ...
    }
    ```
- TypedSort
  - 정렬이 적용된 특정 타입의 속성을 정의하는 데 사용되는 클래스로, Sort 클래스로부터 상속받음
  - 타입 안전성을 보장함
  - ```java
    Sort.TypedSort<Book> bookSort = Sort.by(Book.class);
    Sort sortByPrice = bookSort.and(Sort.by(Sort.Order.asc("price"))); 
    ```

##### Sort 클래스
- 여러 개의 Order 객체를 가진 클래스로, 각 속성에 대한 정렬 방향과 null 처리 방식에 대한 정보를 가지고 있음
- by(): 정렬 조건 지정
- reverse(): 역정렬
- ascending(): 현재 Order들에 대한 오름차순 정렬 수행
- descending(): 현재 Order들에 대한 내림차순 정렬 지정

#### Example

##### QBE (Query by Example)

개발자가 직접 쿼리를 직접 작성하지 않고 동적으로 쿼리를 생성할 수 있는 기술임

데이터베이스 쿼리를 생성할 때 엔티티 인스턴스를 기반으로 쿼리를 생성하며, 필터링할 조건을 객체 자체로 표현하고

해당 객체의 필드 값을 기반으로 동적으로 쿼리를 생성함

QBE 구성 요소
- Probe: 쿼리할 도메인 객체
- ExampleMatcher: 쿼리 조건을 정의한 객체 (여러 Example에서 사용될 수 있음)
- Example: 쿼리를 생성하는 객체, Probe와 ExampleMatcher로 구성됨
- FetchableFluentQuery: Example에서 파생된 쿼리를 추가로 커스텀할 수 있는 API 제공

QBE 유즈케이스
- 정적 또는 동적 제약 조건 집합으로 data source 쿼리
- 기존 쿼리 수정없이 도메인 객체 리팩토링
- data source API로부터 독립적으로 작성

QBE 한계
- `firstname = ?0 or (firstname = ?1 and lastname = ?2)`와 같이 중첩되거나 그룹화된 속성 제약 조건 지원 X
  - 단일 레벨의 조건만 사용할 수 있음
  - 복잡한 논리 연산(AND, OR, NOT 조합 등)을 지원하지 않음
- 데이터베이스에 의존적인 문자열 매칭 지원함 
  - QBE의 문자열 매칭 기능은 데이터베이스의 구현에 따라 다를 수 있음
  - 특정 기능이 모든 데이터베이스에서 지원되지 않을 수 있음
- 다른 데이터 타입에 대한 정확한 일치만 지원함
  - 문자열 외의 다른 데이터 타입에 대해서는 정확한 일치만 가능함
  - 범위 조건과 같은 복잡한 검색 조건을 지원하지 않음

### Querydsl


## Spring Data JPA