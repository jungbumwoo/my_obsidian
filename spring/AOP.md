Aspect-Oriented Programming. 
공통 기능을 하나로 모아서 관리하는 것, 횡단 관심사 프로그래밍

핵심 비지니스 로직과 부가 기능을 분리하는 프로그래밍 패러다임

OOP 한계 보완 - 여러 클래스에 걸쳐 반복되는 코드를 한 곳에 관리

ex) 로깅, 트랜잭션, 보완, 캐싱, 예외처리 등등


### 핵심 개념
Aspect: 관심사를 모듈화한 클래스. ex) 로깅 aspect, 보안 aspect. 횡단 관심사라고도 함.
Advice: 실제 실행되는 부가기능 코드
Join Point: Advice가 적용될 수 있는 지점 (spring에서는 메소드 실행 시점이 join point.)
Pointcut: Join Point를 선택하는 표현식. 어떤 메소드에 적용할건지?
Target: Advice가 적용되는 대상 객체
Proxy:  Target을 감싸서 Advice를 실행하는 객체
Weaving: Aspect를 Target에 적용하는 과정

### Spring AOP 동작원리 - Proxy

client가 method르 호출 시, Proxy 코드가 Advice를 실행
JDK Dynamoic Proxy - 인터페이스 기반
CGLIB Proxy  - 클래스 상속 기반 (Spring Boot default)



### Weaving 방식 비교

| **방식**           | **시점**   | **도구**      | **특징**       |
| ---------------- | -------- | ----------- | ------------ |
| **Compile-time** | 컴파일 시    | AspectJ     | 강력하지만 설정 복잡  |
| **Load-time**    | 클래스 로딩 시 | AspectJ LTW | 에이전트 필요      |
| **Runtime**      | 실행 시     | Spring AOP  | 간편, Proxy 기반 |
|                  |          |             |              |

Compile 시점에 적용하면, Proxy 객체를 한 번 더 거치지 않음.
Compile 시에는 .java -> .class 변환 시 끼워넣어줌
Load time의 경우, class를 memory에 로드 시점에 적용

AspectJ가 필요한 경우는? - 이런 경우가 있나 잘 모르겠음. 할거 다 하고 이것도 줄여보는건가 싶다.

1. self-invocation에도 AOP를 적용해야 하는 경우
2. private/protected/final method에도 advice가 필요할 때
3. 생성자 호출, field access 같은 더 넓은 join point가 필요할 때
4. 매우 빈번하게 호출되는 hot path에 AOP가 걸려 있고 오버헤드가 실제 병목일 때

### Advice 종류

| **Advice**          | **실행 시점**     | **용도**        |
| ------------------- | ------------- | ------------- |
| **@Before**         | 메서드 실행 전      | 권한 체크 및 입력 검증 |
| **@After**          | 메서드 실행 후 (항상) | 리소스 정리        |
| **@AfterReturning** | 정상 반환 후       | 결과 로깅 및 후처리   |
| **@AfterThrowing**  | 예외 발생 후       | 예외 로깅 및 알림    |
| **@Around**         | 메서드 전후 모두     | 성능 측정 및 트랜잭션  |

### Pointcut 표현식
```
// 기본 문법
execution(접근제어자? 반환타입 패키지.클래스.메서드(파라미터))

// 모든 public 메서드
execution(public * * (..))

// service 패키지의 모든 메서드
execution(* com.example.service.*.*(..))

// Order로 시작하는 메서드
execution(* com.example..*Service.order*(..))
```

자주 쓰는 지시자:
- `execution()` - 메서드 실행 매칭
- `@annotation()` - 특정 어노테이션이 붙은 메서드
- `within()` - 특정 클래스/패키지 내 모든 메서드
- `bean()` - 특정 Bean 이름으로 매칭
### 실전 예제

1. 실행시간 로깅
```java
@Aspect
@Component
public class ExecutionTimeAspect{
	@Around("execution(* com.example.service.*.*(..))")
	public Object measureTime(ProceedingJoinPoint joinPoint) throws Throwable {
	long start = System.currentTimeMillis();
	Object result = joinPoint.proceed();
	long elapsed = System.currentTimeMillis() - start;
	log.info("{} 실행시간: {} ms", joinPoint.getSignature().getName(), elaspsed);
	return result;
	}
}
```
- `@Around` + `ProceedingJoinPoint`로 메서드 전 후 제어
- `proceed()` 호출이 실제 Target method 실행 

2. 예제 2 - custom annotation + AOP

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Loggable{}
```

```java
@Aspect
@Component
public class LoggableAspect{
	@Before("annotation(Loggable)")
	public void logBefore(JoinPoint joinPoint) {
		log.info("[Log] {} 호출, args={}",
			joinPoint.getSignature().getName(),
			joinPoint.getArgs()
		);
	}
}
```

```java
@Service
public class OrderService {
	@Loggable
	public Order createOrder(OrderRequest req) { .. }
}
```

적용방식 명확, 정밀하게 제어 가능
JoinPoint 는 annotation이 붙어있는 메서드, target이다.



3. 실전 예제 3 - 권한 체크

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface RequireRole {
	String value();
}
```

```java
@Aspect
@Component
public class AuthAspect {
	@Before("@annotation(requireRole)")
	public void checkRole(RequireRole requireRole) {
		String current = SecurityContext.getCurrentRole();
		if (!current.equals(requireRole.value())) {
		throw new AccessDeniedException(
		"권한 부족");}
	}
}
```

```java
@RequireRole("ADMIN")
public void deleteUser(Long userId) { ... }
```

### 주의사항, 한계

- self-Invocation 시 적용안됨.
	- 별도 Bean으로 분리 or AopContext.currentProxy() 사용
- private 메서드에 AOP 적용 불가 (proxy 기반 한계)
	- Proxy AOP는 결국 “proxy가 받을 수 있고 override할 수 있는 호출”만 가로챌 수 있는데, private/final은 그 조건을 만족하지 못하고, protected는 CGLIB에서는 가능하지만 JDK Proxy나 self-invocation에서는 적용되지 않음
	- 상속이건 interface이건 private/final 되어있으면 적용이 불가능

- 
- pointcut 범위가 너무 넒으면 성능 저하 가능
- advice 순서 제어 - @order 어노테이션 활용

### 프록시 생성 되는 것

Service 객체 생성된걸 빈 후처리 과정에서 전달받고,
Advisor 들을 조회해서 전달된 빈이 Advisor - PointCut에서 적용된 대상, 지점인지 확인 후 프록시 생성 및 기존 객체를 대체해서 끼워넣음

