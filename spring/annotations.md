
@Component, @Bean 비교

- **순수 내 코드 컴포넌트**(서비스, 리포지토리, 컨트롤러 등)는 대부분 `@Component`(또는 `@Service` 등)로 간단히 등록.
    
- **외부 라이브러리 객체**, **초기화 로직이 복잡한 객체**, **동적으로 조립해야 하는 객체**는 `@Bean` 방식으로 등록.

@Configuration

@Autowired
타입 매칭을 시도하고, 이때 여러 빈이 있으면 필드 이름, 파라미터 이름으로 빈 이름을 추가 매칭한다.

---

## 1. 메타데이터상의 클래스 ≠ 실제 인스턴스 클래스

스프링에 빈을 정의할 때 `<bean class="…">` 혹은 `@Bean` 메소드 시그니처에 지정한 클래스는  
“초기 참조용”이지, 꼭 그 클래스의 인스턴스가 그대로 반환된다는 보장은 없습니다.

---

## 2. 타입이 바뀔 수 있는 주요 경우

1. **팩토리 메서드(static or instance) 사용**
    
    - `factory-method="createXxx"` 혹은 `factory-bean`을 통해 호출된 메서드가  
        실제로는 지정 클래스의 서브클래스 혹은 전혀 다른 클래스의 인스턴스를 반환할 수 있습니다.
        
2. **FactoryBean 구현체**
    
    - `FactoryBean<T>` 인터페이스를 구현한 클래스는, `getObject()`가 반환하는 `T`가  
        빈 이름으로 조회될 때 실제 객체가 됩니다. 이 경우 메타데이터상의 FactoryBean 클래스가 아닌,  
        그 내부에서 생성한 `T` 타입이 런타임 빈 타입이 됩니다.
        
3. **인스턴스 팩토리 메서드**
    
    - XML 설정에서 `<bean id="fooFactory" class="…"/>` 다음에  
        `<bean id="foo" factory-bean="fooFactory" factory-method="create"/>`처럼 쓰면,  
        `<bean class>` 속성 자체가 없고 팩토리 빈 이름만 지정된 경우도 있으므로,  
        메타데이터만 보고는 빈 타입을 바로 알 수 없습니다.
        
4. **AOP 프록시**
    
    - `@Transactional`, `@Cacheable` 등 AOP 기능이 붙으면  
        대상 객체를 프록시로 래핑합니다.
        
    - 인터페이스 기반 프록시는 실제 객체의 클래스가 아닌 “프록시 클래스”를 드러내기도 합니다.

---
DI 순환 참조는 어떻게 피하거나 해결될지?


todo 찾아볼 것
- Spring Bean LifeCycle 전체 흐름 (PostProcessor → Proxy → Bean 생성 순서)
    
- AOP가 메서드를 어떻게 가로채는지 (CGLIB 바이트코드 구조까지)
    
- Reflection의 성능/오버헤드는 어느 정도인지
    
- Spring Boot 자동 구성(@EnableAutoConfiguration)이 어떻게 동작하는지
    
- @Transactional 이 rollback 되는 내부 구조
    
- DI가 어떻게 순환참조를 해결하는지
- - @Transactional(propagation = …) 전파 레벨의 진짜 내부 동작
    
- CGLIB 프록시 구조 자체(프록시 바이트코드 분석)
    
- readOnly = true가 실제로 어떤 최적화 하는지
    
- 트랜잭션 경계에서 Hibernate 1차 캐시(i.e., 영속성 컨텍스트)가 어떻게 움직이는지
JDK Proxy의 실제 호출 구조 (InvocationHandler.invoke)

@Transactional 의 전파(Propagation) 정확한 내부 동작



Spring Bean LifeCycle + Proxy 생성 순서 (PostProcessor 레벨까지)

AOP Pointcut 매칭 과정(AspectJ Expression Parser 내부)

실제 @Transactional 프록시 바이트코드를 IntelliJ로 확인하는 법?