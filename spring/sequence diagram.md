


---

```
sequenceDiagram
      autonumber

      participant SA    as SpringApplication
      participant AACX  as AbstractApplicationContext
      participant CCPP  as ConfigurationClassPostProcessor
      participant CCP   as ConfigurationClassParser
      participant CPBDS as ClassPathBeanDefinitionScanner
      participant CCBDR as ConfigurationClassBeanDefinitionReader
      participant DLBF  as DefaultListableBeanFactory
      participant ABD   as AbstractBeanDefinition
      participant DSBR  as DefaultSingletonBeanRegistry

      SA->>AACX: refresh()
      AACX->>CCPP: postProcessBeanDefinitionRegistry(registry)
      CCPP->>CCPP: processConfigBeanDefinitions(registry)

      rect rgb(255, 250, 220)
          Note over CCPP,CCP: 파싱 단계 — BeanDefinition 등록 전 메타데이터 수집
          CCPP->>CCP: parse(configCandidates)

          alt @ComponentScan
              CCP->>CPBDS: doScan(basePackages)
              Note over CPBDS: classpath 탐색 → @Component 후보 필터링<br/>어노테이션 속성 적용 후 checkCandidate()
              CPBDS->>DLBF: registerBeanDefinition(beanName, beanDefinition)
          end

          alt @Bean / @Import / @ImportResource
              Note over CCP: 메타데이터만 수집<br/>실제 등록은 loadBeanDefinitions 단계로 지연
          end
      end

      rect rgb(245, 230, 255)
          Note over CCPP,CCBDR: 등록 단계 — @Bean / @Import / Registrar
          CCPP->>CCBDR: loadBeanDefinitions(configClasses)
          loop 각 ConfigurationClass
              CCBDR->>DLBF: registerBeanDefinition(beanName, beanDefinition)
              Note over CCBDR,DLBF: @Bean 메서드 / @ImportResource / ImportBeanDefinitionRegistrar
          end
      end

      rect rgb(215, 238, 255)
          Note over DLBF,DSBR: DefaultListableBeanFactory.registerBeanDefinition() 핵심 로직

          DLBF->>ABD: validate()
          Note over ABD: lookup-method / replaced-method 충돌 검증

          DLBF->>DLBF: existingDefinition = beanDefinitionMap.get(beanName)

          alt 기존 BeanDefinition 존재 (override)
              alt allowBeanDefinitionOverriding == false
                  DLBF-->>CCBDR: throw BeanDefinitionOverrideException
              else override 허용
                  Note over DLBF: ROLE 비교 → WARN / INFO / DEBUG 로그 후 덮어쓰기
                  DLBF->>DLBF: beanDefinitionMap.put(beanName, beanDefinition)
              end

          else 신규 등록
              alt hasBeanCreationStarted() == true
                  Note over DLBF: refresh 이후 런타임 동적 등록<br/>synchronized — copy-on-write로 beanDefinitionNames 교체
                  DLBF->>DLBF: beanDefinitionMap.put(beanName, beanDefinition)
                  DLBF->>DLBF: beanDefinitionNames = new ArrayList(기존) + beanName
              else hasBeanCreationStarted() == false
                  Note over DLBF: BeanFactoryPostProcessor 실행 중 (일반 경로)<br/>동기화 없이 직접 업데이트
                  DLBF->>DLBF: beanDefinitionMap.put(beanName, beanDefinition)
                  DLBF->>DLBF: beanDefinitionNames.add(beanName)
              end
              DLBF->>DLBF: removeManualSingletonName(beanName)
              DLBF->>DLBF: frozenBeanDefinitionNames = null
          end

          alt existingDefinition != null OR containsSingleton(beanName)
              DLBF->>DLBF: resetBeanDefinition(beanName)
              DLBF->>DSBR: destroySingleton(beanName)
              Note over DLBF,DSBR: 타입 캐시 클리어 — allBeanNamesByType / singletonBeanNamesByType<br/>mergedBeanDefinitions 제거 → 자식 BeanDefinition 재귀 처리
          end
      end
```

---

- AOP 호출 example

```
sequenceDiagram
    autonumber

    participant DLBF  as DefaultListableBeanFactory
    participant AACABF as AbstractAutowireCapableBeanFactory
    participant APC   as AnnotationAwareAspectJAutoProxyCreator
    participant PF    as ProxyFactory
    participant CGLIB as CglibAopProxy
    participant Caller as Controller (호출자)
    participant Proxy as CoinSymbolAdminService$$SpringCGLIB$$0
    participant Chain as CglibMethodInvocation
    participant TI    as TransactionInterceptor
    participant TAS   as AnnotationTransactionAttributeSource
    participant TM    as JpaTransactionManager
    participant EM    as EntityManager / DataSource
    participant Target as CoinSymbolAdminService (실 인스턴스)
    participant Repo  as JpaRepository
    participant EventBus as ApplicationEventPublisher
    participant EL    as SymbolMappingLogListener

    rect rgb(230, 242, 255)
        Note over DLBF,APC: [1단계] AOP BeanPostProcessor 등록 — registerBeanPostProcessors()
        Note over APC: AopAutoConfiguration이 @EnableAspectJAutoProxy를 활성화<br/>→ AnnotationAwareAspectJAutoProxyCreator가<br/>  BeanPostProcessor로 등록됨 (Ordered.HIGHEST_PRECEDENCE에 준하는 우선순위)
        DLBF->>AACABF: getBean("internalAutoProxyCreator")
        AACABF-->>DLBF: AnnotationAwareAspectJAutoProxyCreator 인스턴스 반환
        DLBF->>DLBF: addBeanPostProcessor(AnnotationAwareAspectJAutoProxyCreator)
        Note over DLBF: 이후 모든 Bean 초기화 시<br/>postProcessAfterInitialization()이 자동 호출됨
    end

    rect rgb(230, 255, 235)
        Note over DLBF,CGLIB: [2단계] @Transactional Bean의 AOP Proxy 생성 — finishBeanFactoryInitialization()
        DLBF->>AACABF: getBean("coinSymbolAdminService")
        AACABF->>AACABF: instantiateBean()<br/>→ CoinSymbolAdminService 실 인스턴스 생성 (new)
        AACABF->>AACABF: populateBean()<br/>→ @Autowired 의존성 주입

        AACABF->>AACABF: initializeBean()
        Note over AACABF: BeanPostProcessor 순차 실행

        AACABF->>APC: postProcessAfterInitialization(bean, "coinSymbolAdminService")
        APC->>APC: wrapIfNecessary(bean, beanName)
        Note over APC: 등록된 Advisor 목록을 순회하며<br/>해당 Bean에 적용 가능한 Advisor 탐색<br/>→ BeanFactoryTransactionAttributeSourceAdvisor 매칭 확인<br/>  (CoinSymbolAdminService의 @Transactional 메서드 발견)

        APC->>PF: new ProxyFactory(target)
        PF->>PF: addAdvisor(BeanFactoryTransactionAttributeSourceAdvisor)
        Note over PF: Advisor 구성<br/>  Pointcut : @Transactional 어노테이션이 있는 메서드<br/>  Advice   : TransactionInterceptor (MethodInterceptor 구현체)

        PF->>CGLIB: getProxy(classLoader)
        Note over CGLIB: proxyTargetClass=true (Spring Boot 2.x 기본값)<br/>→ JDK Dynamic Proxy 아닌 CGLIB 서브클래스 생성<br/>→ CoinSymbolAdminService를 상속한<br/>  CoinSymbolAdminService$$SpringCGLIB$$0 클래스 생성

        CGLIB-->>PF: CGLIB Proxy 인스턴스 반환
        PF-->>APC: Proxy 반환
        APC-->>AACABF: Proxy 반환 (실 인스턴스 대신)
        AACABF-->>DLBF: Proxy를 singletonObjects 캐시에 저장
        Note over DLBF: 이후 getBean("coinSymbolAdminService") 호출 시<br/>실 인스턴스가 아닌 CGLIB Proxy가 반환됨<br/>실 인스턴스는 Proxy 내부의 target 필드에 보관
    end

    rect rgb(255, 250, 220)
        Note over Caller,Target: [3단계] 런타임 — @Transactional 메서드 호출 (Proxy 인터셉터 체인)
        Caller->>Proxy: updateCoinSymbol(request)
        Note over Proxy: CGLIB DynamicAdvisedInterceptor.intercept() 호출<br/>적용된 Advisor 목록으로 인터셉터 체인 구성

        Proxy->>Chain: new CglibMethodInvocation(proxy, target, method, args, advisors)
        Chain->>TI: invoke(MethodInvocation)
        Note over TI: TransactionInterceptor.invokeWithinTransaction() 진입

        TI->>TAS: getTransactionAttribute(method, targetClass)
        Note over TAS: 메서드 → 클래스 순으로 @Transactional 메타데이터 파싱<br/>속성 추출: propagation, isolation, readOnly, rollbackFor, timeout
        TAS-->>TI: RuleBasedTransactionAttribute 반환

        TI->>TM: getTransaction(transactionAttribute)
        Note over TM: TransactionSynchronizationManager 확인<br/>현재 스레드에 활성 트랜잭션 없음<br/>→ PROPAGATION_REQUIRED: 새 트랜잭션 시작

        TM->>EM: DataSource.getConnection()
        Note over EM: Connection 획득<br/>connection.setAutoCommit(false)<br/>EntityManager 생성 및 Connection 연결<br/>TransactionSynchronizationManager에 스레드 로컬로 리소스 바인딩
        EM-->>TM: Connection / EntityManager 반환

        TM-->>TI: DefaultTransactionStatus 반환

        TI->>Chain: proceed() — 다음 인터셉터 또는 타겟 메서드 호출
        Note over Chain: 인터셉터 체인 끝 → 실제 타겟 메서드 직접 호출

        Chain->>Target: updateCoinSymbol(request)
        Note over Target: 실제 비즈니스 로직 수행<br/>같은 스레드이므로 TransactionSynchronizationManager에서<br/>동일한 EntityManager / Connection 재사용

        Target->>Repo: coinSymbolRepository.save(entity)
        Note over Repo: EntityManager.persist() 또는 merge()<br/>SQL 생성 후 1차 캐시(Persistence Context)에 저장<br/>(아직 DB에 flush 되지 않음)
        Repo-->>Target: 저장된 entity 반환

        Target->>EventBus: publishEvent(new SymbolMappingChangedEvent(...))
        Note over EventBus: @TransactionalEventListener가 등록된 이벤트<br/>현재 트랜잭션이 활성 상태이므로<br/>이벤트를 큐에 저장 (즉시 실행 아님)
        EventBus-->>Target: 이벤트 등록 완료

        Target-->>Chain: 메서드 결과 반환
        Chain-->>TI: 결과 반환
    end

    rect rgb(220, 255, 245)
        Note over TI,EM: [4단계] 트랜잭션 커밋 또는 롤백

        alt 정상 완료
            TI->>TM: commitTransactionAfterReturning(txStatus)
            TM->>EM: EntityManager.flush()<br/>→ 1차 캐시의 변경 내용을 SQL로 변환하여 DB에 전송
            TM->>EM: connection.commit()<br/>→ DB에 영구 반영
            Note over TM: TransactionSynchronization.beforeCommit() 콜백<br/>TransactionSynchronization.afterCommit() 콜백 실행
            TM->>EM: EntityManager.close() / Connection 반환 (커넥션 풀)
            TM->>TM: TransactionSynchronizationManager 스레드 로컬 리소스 정리
            TM-->>TI: 커밋 완료
            TI-->>Proxy: 결과 반환
            Proxy-->>Caller: 결과 반환

        else 예외 발생 (rollbackFor 조건 해당)
            Target-->>Chain: RuntimeException / 지정된 예외 throw
            Chain-->>TI: 예외 전파
            TI->>TI: rollbackOn(exception) 확인
            Note over TI: @Transactional(rollbackFor=Exception.class) 이면<br/>Checked Exception도 롤백 대상
            TI->>TM: rollbackOnException(txStatus, exception)
            TM->>EM: connection.rollback()<br/>→ DB 변경 사항 전체 취소
            TM->>EM: EntityManager.close() / Connection 반환
            TM->>TM: TransactionSynchronizationManager 리소스 정리
            TM-->>TI: 롤백 완료
            TI-->>Proxy: 예외 재throw (UnexpectedRollbackException 가능)
            Proxy-->>Caller: 예외 전파
        end
    end

    rect rgb(255, 232, 232)
        Note over TM,EL: [5단계] @TransactionalEventListener — AFTER_COMMIT 처리
        Note over EL: SymbolMappingLogListener는<br/>@TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT) 사용<br/>커밋 성공 후에만 리스너 실행 보장

        TM->>TM: afterCommit() 콜백 — 등록된 TransactionSynchronization 순회
        TM->>EL: SymbolMappingLogListener.handleAfterCommit(event)
        Note over EL: 이 시점에서 트랜잭션은 이미 종료됨<br/>기본적으로 새 트랜잭션 없이 실행<br/>(새 트랜잭션이 필요하면 @Transactional(propagation=REQUIRES_NEW) 추가 필요)<br/>로그 저장 등 커밋 후속 처리 수행
        EL-->>TM: 처리 완료
    end

    rect rgb(240, 230, 255)
        Note over Caller,Target: [부록] readOnly=true 트랜잭션 동작 차이
        Note over TM: @Transactional(readOnly = true) 적용 시<br/>  • FlushMode.MANUAL 설정 → flush() 자동 호출 안됨<br/>  • 변경 감지(Dirty Checking) 수행하지 않아 성능 향상<br/>  • DB 드라이버/인프라에 따라 읽기 전용 힌트 전달 가능<br/>  • 쓰기 작업 시 TransientPropertyValueException 발생 가능<br/><br/>프로젝트 내 readOnly=true 사용처:<br/>  CoinSymbolAdminService.getXxx()<br/>  CoinSymbolService.getXxx()<br/>  ExchangeService (클래스 레벨)
    end

```