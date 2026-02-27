## ❓Spring AOP에 대해 설명해보세요.
관점 지향 프로그래밍(Aspect-Oriented Programming)을 스프링에서 구현한 기능입니다. <br>
공통적으로 반복되는 부가 기능을 핵심 로직과 분리해서 관리하는 방법입니다. <br>
예를 들어, 모든 메서드마다 로그르 남겨야 한다면, 각 메서드 안에서 직접 작성하는 대신 AOP를 통해 공통 로직으로 분리할 수 있습니다. 

스프링이 Bean을 생성할 때, 실제 객체 대신 프록시 객체를 등록합니다. 
클라이언트가 메서드를 호출하면 실제 객체가 아니라 프록시가 먼저 실행되고, 부가 기능을 수행한 뒤 실제 메서드를 호출합니다.
이 과정을 위빙(weaving)이라고 합니다.

**➕ @Aspect는 어떻게 동작하나요?** <br>
@Aspect는 개발자가 직접 공통 로직을 정의하는 방식입니다. 
@Before, @AfterReturning 같은 어노테이션을 사용해 특정 패키지나 메서드에 적용할 수 있습니다.
스프링은 해당 Bean을 프록시로 감싸고, 메서드 호출 시 먼저 Aspect 로직을 실행한 뒤 실제 메서드를 실행합니다. 

**➕ @Transactional은 어떻게 동작하나요?** <br>
트랜잭션을 하나의 작업 단위로 묶어서 성공하면 commit, 실패하면 rollback을 자동으로 처리해주는 기능입니다.
@Transactional이 붙은 Bean을 프록시로 감싸고, 메서드 호출 시 프록시가 먼저 실행되어 트랜잭션을 시작 후 실제 메서드를 호출합니다.
이후 예외가 없으면 commit, 있으면 rollback을 수행합니다.

<br>

## ❓@Transactional(readOnly = true)는 어떤 기능인가요?
읽기 전용 트랜잭션으로, insert, update, delete 같은 쓰기 작업이 없는 경우 최적화를 위해 사용합니다.
내부적으로 JPA가 Dirty Checking을 수행하지 않기 때문에 불필요한 체크가 발생하지 않아 성능이 더 좋습니다.

**➕ Dirty Checking이란 무엇인가요?** <br>
JPA는 엔티티를 조회할 때 원본 상태를 스냅샷으로 저장해두고, 트랙잭션 커밋 시점에 현재값과 비교해서 변경되었으면 dirty로 판단해 자동으로 UPDATE 쿼리를 실행합니다. <br>
단, Dirty Checking이 동작하려면 트랙잭션 안에서 실행되어야 하고 영속성 컨텍스트가 관리하는 엔티티여야 합니다.

**➕ 그런데, 읽기에 트랜잭션을 걸 필요가 있나요? @Transactional을 안 붙이면 되는거 아닐까요?** <br>
읽기에도 트랜잭션이 필요한 경우가 있습니다.
1. 조회 중 다른 트랜잭션이 데이터를 변경하면 안 되는 경우, 예를 들어 재고 확인/잔액 조회 같은 경우에 데이터 일관성을 위해 사용합니다.
2. Lazy로 설정된 연관 데이터는 실제 접근하는 시점에 DB 조회가 발생하는데, 이때 영속성 컨텍스트가 열려 있어야 합니다. 영속성 컨텍스트는 트랜잭션 안에서 생성되고 유지되기 때문에 Lazy Loading을 사용하려면 트랜잭션이 필요합니다.

**➕ @Transactional에서 격리 수준을 설정하면 어떻게 동작하나요?** <br>
@Transactional(isolation = Isolation.XXX)를 통해 트랜잭션의 격리 수준을 지정할 수 있습니다.
- **READ_UNCOMMITTED** : 다른 트랜잭션에서 커밋 되지 않은 데이터도 읽을 수 있어 Ditry Read가 발생할 수 있음
- **READ_COMMITTED** : 커밋된 데이터만 읽지만, 같은 트랜잭션 안에서는 두 번 조회 시 값이 달라질 수 있음 (Non-repeatable Read)
- **REPEATABLE_READ** : 같은 트랜잭션 내에서는 동일한 조회 결과 보장하지만, 새로운 Row가 추가되는 경우 Phantom Read가 발생할 수 있음
- **SERIALIZABLE** : 모든 트랜잭션을 순차적으로 실행하여 일관성은 강하지만 성능이 가장 낮음

<br>

## ❓스프링 로컬 캐시가 아닌 redis를 쓰는 이유는 무엇일까요?
로컬 캐시는 애플리케이션 내부 메모리를 사용하기 때문에 단일 서버 환경에서는 가장 빠른 응답 속도를 제공합니다. 하지만 서버가 여러 대로 확장되면 여러 서버가 같은 캐시를 공유할 수 없어 데이터 불일치 문제가 생기고, 서버 재시작 시 데이터가 사라지는 문제도 발생합니다. <br>
Redis는 별도의 인메모리 서버를 사용하는 Remote Cache로,  위 문제를 해결하고 TTL, 분산 락, Pub/Sub 같은 기능을 제공해 다중 서버 환경이나 확장성을 고려한다면 Redis를 사용하는 것이 더 적합합니다.

<br>

## ❓스프링 시큐리티의 원리와 동작 방식 설명해주세요.
스프링 시큐리티는 스프링 기반 애플리케이션의 보안을 담당하는 스프링 하위 프레임워크입니다. <br>
핵심 구조는 **SecurityFilterChain** 기반의 필터 체인 방식으로, 사용자의 요청이 컨트롤러에 가기 전에 여러 보안 필터를 거쳐 인증과 인가를 처리합니다. <br>

<img width="450" height="350" alt="image" src="https://github.com/user-attachments/assets/153941c2-d894-47b8-a9bc-cd1306b1ed4f" />

1. 사용자가 로그인 폼을 통해 아이디와 비밀번호를 전송한다.
2. **UsernamePasswordAuthenticationFilter**가 HttpServletRequest에서 아이디와 비밀번호를 추출하고 UsernamePasswordAuthenticationToken을 만들어 AuthenticationManager에 전달한다.
3. **AuthenticationManager**는 실제 인증을 처리할 AuthenticationProvider에게 인증을 위임한다.
4. **AuthenticationProvider**는 **UserDetailsService**를 호출해 DB에서 사용자 정보를 조회하고, 이를 UserDetails 객체로 반환받는다.
5. AuthenticationProvider는 사용자가 입력한 비밀번호와 DB에 저장된 암호화 비밀번호를 비교해 인증을 수행한다.
6. 인증에 성공하면 인증 정보가 담긴 Authentication 객체를 **SecurityContextHolder**에 저장하고 AuthenticationSuccessHandler를 실행합니다. 실패하면 AuthenticationFailureHandler가 실행됩니다.

