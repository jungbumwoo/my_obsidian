
```
%% jb: Spring Bean Lifecycle Sequence Diagram
%% 관련 코드 위치:
%%   AbstractApplicationContext.java    - refresh(), finishBeanFactoryInitialization(), finishRefresh(), doClose()
%%   PostProcessorRegistrationDelegate.java - invokeBeanFactoryPostProcessors(), registerBeanPostProcessors()
%%   DefaultListableBeanFactory.java    - preInstantiateSingletons()
%%   DefaultSingletonBeanRegistry.java  - getSingleton(), addSingleton(), destroySingletons()
%%   AbstractAutowireCapableBeanFactory.java - createBean(), doCreateBean(), populateBean(), initializeBean()
%%   DisposableBeanAdapter.java         - destroy()

sequenceDiagram
    autonumber

    participant Client   as Client
    participant AC       as AbstractApplicationContext
    participant PPRD     as PostProcessorRegistrationDelegate
    participant BF       as DefaultListableBeanFactory
    participant DSR      as DefaultSingletonBeanRegistry
    participant AACBF    as AbstractAutowireCapableBeanFactory
    participant BPP      as BeanPostProcessor
    participant Bean     as Bean Instance
    participant DBA      as DisposableBeanAdapter

    %% ====================================================
    %% ① ApplicationContext 생성 & BeanDefinition 로딩
    %% ====================================================
    Note over Client,AC: ① ApplicationContext 생성 & BeanDefinition 로딩
    Client->>+AC: new AnnotationConfigApplicationContext(Config.class)
    Note right of AC: register() / scan() 으로 BeanDefinition 등록<br/>ClassPathBeanDefinitionScanner.scan()<br/>→ DefaultListableBeanFactory.registerBeanDefinition()
    AC->>BF: registerBeanDefinition(beanName, beanDefinition)

    AC->>AC: refresh()

    %% ====================================================
    %% ② BeanFactory 준비
    %% ====================================================
    Note over AC,BF: ② BeanFactory 준비 - prepareBeanFactory()
    AC->>BF: setBeanClassLoader() / setBeanExpressionResolver()
    AC->>BF: addBeanPostProcessor(ApplicationContextAwareProcessor)
    Note right of BF: ApplicationContextAware 등<br/>Aware 계열 인터페이스 처리용 BPP 등록
    AC->>BF: registerResolvableDependency(BeanFactory, ApplicationContext ...)
    Note right of BF: @Autowired ApplicationContext 주입 가능하도록 등록

    %% ====================================================
    %% ③ BeanFactoryPostProcessor 실행
    %% ====================================================
    Note over AC,PPRD: ③ BeanFactoryPostProcessor 실행 - invokeBeanFactoryPostProcessors()
    AC->>PPRD: invokeBeanFactoryPostProcessors(beanFactory)
    Note right of PPRD: BeanDefinitionRegistryPostProcessor → BeanFactoryPostProcessor 순서로 실행
    PPRD->>BF: ConfigurationClassPostProcessor.postProcessBeanDefinitionRegistry()
    Note right of BF: @Configuration / @ComponentScan / @Import / @Bean 처리<br/>BeanDefinition 추가 등록
    PPRD->>BF: BeanFactoryPostProcessor.postProcessBeanFactory()
    Note right of BF: PropertySourcesPlaceholderConfigurer:<br/>${...} 플레이스홀더 치환

    %% ====================================================
    %% ④ BeanPostProcessor 등록
    %% ====================================================
    Note over AC,PPRD: ④ BeanPostProcessor 등록 - registerBeanPostProcessors()
    AC->>PPRD: registerBeanPostProcessors(beanFactory)
    Note right of PPRD: PriorityOrdered → Ordered → 일반 순서로 등록
    PPRD->>BF: addBeanPostProcessor(AutowiredAnnotationBeanPostProcessor)
    Note right of BF: @Autowired / @Value 처리 담당
    PPRD->>BF: addBeanPostProcessor(CommonAnnotationBeanPostProcessor)
    Note right of BF: @PostConstruct / @PreDestroy / @Resource 처리 담당
    PPRD->>BF: addBeanPostProcessor(AbstractAutoProxyCreator)
    Note right of BF: AOP 프록시 생성 담당<br/>등록만 함 - 실행은 각 Bean 생성 시점

    %% ====================================================
    %% ⑤ 싱글톤 Bean 생성 (finishBeanFactoryInitialization)
    %% ====================================================
    Note over AC,DSR: ⑤ 싱글톤 Bean 생성 - finishBeanFactoryInitialization()
    AC->>BF: finishBeanFactoryInitialization(beanFactory)
    BF->>DSR: preInstantiateSingletons()

    loop 각 non-lazy 싱글톤 Bean마다

        DSR->>DSR: getSingleton(beanName, singletonFactory)
        Note right of DSR: L1 캐시(singletonObjects) 확인<br/>없으면 생성 진행

        DSR->>DSR: beforeSingletonCreation(beanName)
        Note right of DSR: singletonsCurrentlyInCreation 에 추가<br/>(순환참조 감지에 사용)

        %% AACBF 활성화 (+)
        DSR->>+AACBF: singletonFactory.getObject()<br/>→ createBean(beanName, mbd, args)

        %% --- Bean 인스턴스 생성 ---
        Note over AACBF: [Bean 인스턴스 생성] createBeanInstance()
        AACBF->>BPP: InstantiationAwareBPP.postProcessBeforeInstantiation()
        Note right of BPP: null 반환 시 일반 인스턴스 생성 진행<br/>non-null 반환 시 생성 숏서킷 (프록시 즉시 반환)
        AACBF->>Bean: 생성자 리플렉션 호출
        Note right of Bean: instantiateBean() - 기본 생성자<br/>autowireConstructor() - 생성자 주입

        AACBF->>DSR: addSingletonFactory(beanName, earlyBeanReference)
        Note right of DSR: L3 캐시 등록 (순환참조 대비 early exposure)<br/>다른 Bean이 이 Bean을 먼저 요청하면<br/>미완성 참조를 L3→L2 캐시를 통해 제공

        %% --- 의존성 주입 ---
        Note over AACBF,BPP: [의존성 주입] populateBean()
        AACBF->>BPP: InstantiationAwareBPP.postProcessAfterInstantiation()
        Note right of BPP: false 반환 시 프로퍼티 주입 전체 건너뜀
        AACBF->>BPP: InstantiationAwareBPP.postProcessProperties()
        BPP->>Bean: @Autowired 필드 / 메서드 주입
        Note right of Bean: AutowiredAnnotationBeanPostProcessor 처리
        BPP->>Bean: @Resource 주입
        Note right of Bean: CommonAnnotationBeanPostProcessor 처리
        AACBF->>Bean: applyPropertyValues()
        Note right of Bean: "XML <property> 명시적 값 적용"

        %% --- 초기화 콜백 ---
        Note over AACBF,Bean: [초기화 콜백] initializeBean()

        AACBF->>Bean: invokeAwareMethods()
        Note right of Bean: BeanNameAware.setBeanName()<br/>BeanClassLoaderAware.setBeanClassLoader()<br/>BeanFactoryAware.setBeanFactory()

        AACBF->>BPP: BPP.postProcessBeforeInitialization()
        BPP->>Bean: @PostConstruct 메서드 실행
        Note right of BPP: CommonAnnotationBeanPostProcessor 처리<br/>ApplicationContextAwareProcessor 도 여기서<br/>ApplicationContextAware.setApplicationContext() 호출

        AACBF->>Bean: InitializingBean.afterPropertiesSet()
        Note right of Bean: InitializingBean 인터페이스 구현 방식

        AACBF->>Bean: custom init-method()
        Note right of Bean: @Bean(initMethod="...") 또는<br/>XML init-method="..." 방식

        %% --- AOP 프록시 생성 ---
        Note over AACBF,BPP: [AOP 프록시 적용] BPP.postProcessAfterInitialization()
        AACBF->>BPP: BPP.postProcessAfterInitialization()
        BPP-->>AACBF: AOP Proxy 객체 반환 (원본 Bean 대체)
        Note right of BPP: AbstractAutoProxyCreator:<br/>@Transactional / @Aspect 등 어드바이스가<br/>적용된 CGLIB/JDK 프록시 생성<br/>반환된 프록시가 이후 컨테이너에서 Bean으로 사용됨

        %% --- 소멸 콜백 등록 ---
        AACBF->>DSR: registerDisposableBeanIfNecessary()
        Note right of DSR: @PreDestroy / DisposableBean / destroy-method<br/>있는 경우 DisposableBeanAdapter 로 래핑하여 등록

        %% AACBF 비활성화 (-) 수정 완료
        AACBF-->>-DSR: Bean 생성 완료

        DSR->>DSR: afterSingletonCreation(beanName)
        DSR->>DSR: addSingleton(beanName, singletonBean)
        Note right of DSR: L1 캐시(singletonObjects)에 등록<br/>L2 / L3 캐시에서 제거 (초기화 완료)

    end

    %% ====================================================
    %% ⑥ Context Refresh 완료
    %% ====================================================
    Note over AC: ⑥ Context Refresh 완료 - finishRefresh()
    AC->>AC: lifecycleProcessor.onRefresh()
    Note right of AC: SmartLifecycle 구현체의 start() 호출
    AC->>Client: publishEvent(ContextRefreshedEvent)
    Note right of Client: ApplicationContext 사용 가능 상태<br/>이후 getBean() 호출로 Bean 사용

    AC-->>-Client: ApplicationContext 준비 완료

    %% ====================================================
    %% ⑦ 스프링 종료
    %% ====================================================
    Note over Client,DBA: ⑦ 스프링 종료

    alt 명시적 종료
        Client->>AC: applicationContext.close()
    else JVM 종료 시 자동 실행
        Note over AC: JVM shutdown hook 발동<br/>registerShutdownHook()으로 등록된<br/>SpringContextShutdownHook Thread
        AC->>AC: doClose()
    end

    AC->>Client: publishEvent(ContextClosedEvent)
    Note right of Client: ApplicationListener에게 종료 통보

    AC->>AC: lifecycleProcessor.onClose()
    Note right of AC: SmartLifecycle.stop() 호출

    AC->>BF: destroySingletons()
    BF->>DSR: destroySingletons()

    loop 등록 역순으로 각 Bean 소멸 (의존성 역순 보장)

        DSR->>DSR: destroyBean(beanName, disposableBean)
        Note right of DSR: 이 Bean에 의존하는 Bean들을 먼저 재귀 소멸<br/>(dependentBeanMap 역방향 순회)

        DSR->>DBA: disposableBean.destroy()
        Note right of DBA: DisposableBeanAdapter.destroy()

        DBA->>BPP: DestructionAwareBPP.postProcessBeforeDestruction()
        BPP->>Bean: @PreDestroy 메서드 실행
        Note right of BPP: CommonAnnotationBeanPostProcessor 처리

        DBA->>Bean: DisposableBean.destroy()
        Note right of Bean: DisposableBean 인터페이스 구현 방식

        DBA->>Bean: AutoCloseable.close()
        Note right of Bean: AutoCloseable 구현 방식

        DBA->>Bean: custom destroy-method()
        Note right of Bean: @Bean(destroyMethod="...") 또는 XML destroy-method<br/>destroyMethod 미지정 시 close() / shutdown() 자동 추론

    end
```


```
› 아래와 같이 main 함수 실행 시 어떻게 ApplicationContext 생성이 호출되는지도 궁금한데 관련 코드도 확인이 가능한가
  요? 아니면 관련 코드는 여기가 아닌 spring-projects/spring-boot 에서 코드를 다운받아봐야할까요?
  @SpringBootApplication
  @ComponentScan(basePackages = {"io.jungbum", "io.jungbum.service"})
  public class BillionaireApiApplication {

      public static void main(String[] args) {
          SpringApplication.run(BillionaireApiApplication.class, args);
      }
  }


• 지금 워크스페이스에 spring-boot 코드가 있는지 먼저 확인하고, 있으면 바로 진입 경로를 짚고 없으면 정확히 어느 저장
  소를 봐야 하는지 정리하겠습니다.

• Explored
  └ List ls

• Ran rg --files . | rg "SpringApplication\.java$|SpringApplicationRunListeners|ServletWebServerApplicationContext|
  │ AnnotationConfigServletWebServerApplicationContext|SpringBootApplication\.java$|
  │ AutoConfigurationImportSelector\.java$"
  └ (no output)

───────────────────────────────────────────────────────────────────────────────────────────────────────────────────

• 네. 지금 워크스페이스는 spring-framework만 있고 spring-boot 코드는 없습니다.
  SpringApplication.run(BillionaireApiApplication.class, args)에서 ApplicationContext를 만들고 refresh()까지 밀어
  넣는 진입점은 spring-framework가 아니라 spring-boot 쪽을 봐야 합니다.

  지금 코드 기준으로는 여기까지가 경계입니다.

  Spring Boot
  main() -> SpringApplication.run(...) -> ApplicationContext 구현체 선택/생성 -> context.refresh()

  Spring Framework
  AbstractApplicationContext.refresh()부터 실제 Bean lifecycle 시작

  봐야 할 대표 위치는 spring-boot 저장소의 아래 클래스들입니다.

  1. spring-boot-project/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java
  2. spring-boot-project/spring-boot/src/main/java/org/springframework/boot/DefaultApplicationContextFactory.java
  3. spring-boot-project/spring-boot/src/main/java/org/springframework/boot/web/servlet/context/
     AnnotationConfigServletWebServerApplicationContext.java
  4. spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/
     SpringBootApplication.java

  실제 따라갈 때는 보통 이 흐름으로 보면 됩니다.

  1. BillionaireApiApplication.main()
  2. SpringApplication.run(BillionaireApiApplication.class, args)
  3. new SpringApplication(primarySources)
  4. SpringApplication.run(args)
  5. createApplicationContext()
  6. prepareContext(...)
  7. refreshContext(context)
  8. 여기서 context.refresh() 호출
  9. 이후부터는 지금 보신 AbstractApplicationContext.refresh() 흐름

  중요한 포인트는 두 가지입니다.

  - @SpringBootApplication은 Boot가 해석합니다.
  - @ComponentScan 자체의 실제 스캔 동작은 결국 Framework의 ClassPathBeanDefinitionScanner로 내려갑니다.

  즉, 질문하신 “main 함수 실행 시 어떻게 ApplicationContext 생성이 호출되는가”는 spring-boot를 봐야 맞고,
  그 이후 “BeanDefinition 로딩 이후 refresh 안에서 무슨 일이 일어나는가”는 지금 보고 계신 spring-framework가 맞습니
  다.

  원하시면 다음 단계로는 제가 spring-boot 기준으로도 같은 방식으로

  - main -> SpringApplication.run
  - ApplicationContext 생성
  - refresh() 호출
  - @SpringBootApplication / 자동설정 진입

  까지 추적용 체크리스트를 바로 정리해드리겠습니다.

───────────────────────────────────────────────────────────────────────────────────────────────────────────────────


› 현재 경로가 아닌 ../spring-boot 에 관련 코드가 있을 겁니다. 관련 코드에 주석을 추가하여 확인할 수 있도록 해주세
  요.


• ../spring-boot의 실제 소스 위치를 확인하고, main -> SpringApplication.run -> ApplicationContext 생성 -> refresh()
  경로에 주석을 넣겠습니다. 먼저 저장소 구조와 수정 가능 여부부터 확인합니다.

• Explored
  └ List spring-boot

• Ran git -C ../spring-boot status --short
  └ (no output)

───────────────────────────────────────────────────────────────────────────────────────────────────────────────────

• 저장소는 접근 가능합니다. 이제 Boot 쪽에서 실제 ApplicationContext 생성과 refresh() 호출 지점을 확인한 뒤, 최소
  범위로 주석을 추가하겠습니다.

• Ran rg --files ../spring-boot | rg "SpringApplication\.java$|DefaultApplicationContextFactory\.java$|
  │ AnnotationConfigServletWebServerApplicationContext\.java$|SpringApplicationRunListeners\.java$|
  │ SpringBootApplication\.java$|ServletWebServerApplicationContext\.java$|
  │ … +1 lines
  └ ../spring-boot/core/spring-boot/src/main/java/org/springframework/boot/SpringApplicationRunListeners.java
    ../spring-boot/core/spring-boot/src/main/java/org/springframework/boot/DefaultApplicationContextFactory.java
    … +16 lines (ctrl + t to view transcript)
    ../spring-boot/module/spring-boot-micrometer-metrics-test/src/test/java/org/springframework/boot/micrometer/
    metrics/test/autoconfigure/AutoConfigureMetricsSpringBootApplication.java

• Explored
  └ Search run\(|createApplicationContext\(|refreshContext\(|prepareContext\(|ApplicationContextFactory|
           AnnotationConfigServletWebServerApplicationContext|SpringBootApplication in java

───────────────────────────────────────────────────────────────────────────────────────────────────────────────────

• Boot 쪽 포인트는 잡혔습니다. SpringApplication에서 context를 만들고 refreshContext에서 framework refresh()로 넘기
  는 구조인지 확인 중입니다. web app이면 AnnotationConfigServletWebServerApplicationContext가 기본 후보입니다.

• Explored
  └ Read SpringApplication.java, AnnotationConfigServletWebServerApplicationContext.java,
         DefaultApplicationContextFactory.java, SpringBootApplication.java, BeanDefinitionLoader.java
    Search createApplicationContext\(|load\(|BeanDefinitionLoader|sources in SpringApplication.java

───────────────────────────────────────────────────────────────────────────────────────────────────────────────────

• 수정 대상은 ../spring-boot라 현재 쓰기 허용 범위 밖일 가능성이 큽니다. 우선 최소 범위의 파일 4개에만 주석을 넣는
  패치 시도를 하고, 권한 문제면 바로 승인 요청으로 전환하겠습니다.

───────────────────────────────────────────────────────────────────────────────────────────────────────────────────

• 패치 자체는 가능해 보이고, 일부 문맥만 어긋났습니다. 정확한 위치를 다시 맞춰 넣겠습니다.

• Explored
  └ Read SpringApplication.java
    Search refreshContext\(|createApplicationContext\(|prepareContext\(|load\(ApplicationContext context in
           SpringApplication.java
    Read SpringApplication.java

───────────────────────────────────────────────────────────────────────────────────────────────────────────────────

• 주석 범위는 SpringApplication, DefaultApplicationContextFactory,
  AnnotationConfigServletWebServerApplicationContext, SpringBootApplication, 그리고 필요하면 BeanDefinitionLoader까
  지로 충분합니다. 지금부터 실제 패치를 넣겠습니다.

• Edited 5 files (+22 -0)
  └ ~/Documents/lab/spring-boot/core/spring-boot/src/main/java/org/springframework/boot/BeanDefinitionLoader.java
  (+4 -0)
    128         void load() {
    129 +               // jb: SpringApplication이 넘긴 source들을 순회하면서                                      
    130 +               // class / package / xml / groovy 등을 BeanDefinition으로 등록한다.                        
    131                 for (Object source : this.sources) {
        ⋮
    163                 if (isEligible(source)) {
    164 +                       // jb: BillionaireApiApplication.class 같은 annotated class는 여기서 register 된다.
    165                         this.annotatedReader.register(source);
        ⋮
    183         private void load(Package source) {
    184 +               // jb: package source가 넘어오면 component scan을 수행한다.                                
    185                 this.scanner.scan(source.getName());

  └ ~/Documents/lab/spring-boot/core/spring-boot/src/main/java/org/springframework/boot/
  DefaultApplicationContextFactory.java (+2 -0)
    59                  try {
    60 +                        // jb: SpringApplication.createApplicationContext()가 위임하는 실제 context 선택 로직.                                                                                                                
    61                          return getFromSpringFactories(webApplicationType, ApplicationContextFactory::create,                                                                                                                  
       ⋮
    71                  if (!AotDetector.useGeneratedArtifacts()) {
    72 +                        // jb: 웹 애플리케이션이 아닌 기본값은 framework의 AnnotationConfigApplicationContext.                                                                                                                
    73                          return new AnnotationConfigApplicationContext();

  └ ~/Documents/lab/spring-boot/core/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java (+10
  -0)
    317                         Banner printedBanner = printBanner(environment);
    318 +                       // jb: main()의 SpringApplication.run(...)이 실제로 ApplicationContext 인스턴스를  생성하는 지                                                                                                        
         점.                                                                                                       
    319                         context = createApplicationContext();
    320                         context.setApplicationStartup(this.applicationStartup);
    321 +                       // jb: refresh 전에 Environment, Initializer, Boot 전용 BeanFactoryPostProcessor,  
    322 +                       // 그리고 primary source(@SpringBootApplication가 붙은 main class 포함)를 context에 적재한다.                                                                                                         
    323                         prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printe                                                                                                        
         dBanner);
    324 +                       // jb: 여기서부터는 spring-framework의 AbstractApplicationContext.refresh()로 내려 간다.                                                                                                              
    325                         refreshContext(context);
        ⋮
    416                 if (!AotDetector.useGeneratedArtifacts()) {
    417 +                       // jb: 여기서 main class, 추가 source, package 등의 입력이 BeanDefinitionLoader로  전달된다.                                                                                                          
    418                         // Load the sources
        ⋮
    449                 }
    450 +               // jb: Boot는 이 메서드에서 framework refresh lifecycle로 제어를 넘긴다.                   
    451                 refresh(context);
        ⋮
    585         protected ConfigurableApplicationContext createApplicationContext() {
    586 +               // jb: WebApplicationType(SERVLET/REACTIVE/NONE)에 따라 적절한 ApplicationContext 구현체를 선택.                                                                                                              
    587 +               // 일반적인 Spring MVC 앱이면 AnnotationConfigServletWebServerApplicationContext가 선택된다.                                                                                                                  
    588                 ConfigurableApplicationContext context = this.applicationContextFactory
        ⋮
    694                 }
    695 +               // jb: primary source로 전달된 BillionaireApiApplication.class 같은 항목을                 
    696 +               // BeanDefinitionLoader를 통해 BeanDefinitionRegistry에 등록한다.                          
    697                 BeanDefinitionLoader loader = createBeanDefinitionLoader(getBeanDefinitionRegistry(context), sourc                                                                                                            
         es);

  └ ~/Documents/lab/spring-boot/core/spring-boot-autoconfigure/src/main/java/org/springframework/boot/
  autoconfigure/SpringBootApplication.java (+2 -0)
    55  @EnableAutoConfiguration
    56 +// jb: @SpringBootApplication 자체가 @ComponentScan을 포함한다.                                            
    57 +// 사용자가 별도로 @ComponentScan(basePackages=...)를 붙이면 그 설정도 함께 해석된다.                      
    58  @ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),

  └ ~/Documents/lab/spring-boot/module/spring-boot-web-server/src/main/java/org/springframework/boot/web/server/
  servlet/context/AnnotationConfigServletWebServerApplicationContext.java (+4 -0)
     73         public AnnotationConfigServletWebServerApplicationContext() {
     74 +               // jb: servlet 기반 Spring Boot 앱에서 자주 선택되는 ApplicationContext 구현체.            
     75 +               // 내부적으로 AnnotatedBeanDefinitionReader + ClassPathBeanDefinitionScanner를 보유한다.   
     76                 this.reader = new AnnotatedBeanDefinitionReader(this);
        ⋮
    203                 super.postProcessBeanFactory(beanFactory);
    204 +               // jb: refresh() 과정에서 Boot가 보관해 두었던 basePackages를 여기서 실제 scan 한다.       
    205                 if (this.basePackages != null && this.basePackages.length > 0) {
        ⋮
    207                 }
    208 +               // jb: main application class(@SpringBootApplication) 같은 annotated class도 여기서 register 된다.                                                                                                            
    209                 if (!this.annotatedClasses.isEmpty()) {

───────────────────────────────────────────────────────────────────────────────────────────────────────────────────

• 주석 추가는 끝났습니다. 마지막으로 위치를 검토하고, main -> run -> context 생성 -> source 로드 -> refresh 흐름을
  파일/라인으로 정리하겠습니다.

• Ran git -C ../spring-boot diff -- core/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java
  │ core/spring-boot/src/main/java/org/springframework/boot/DefaultApplicationContextFactory.java module/
  │ spring-boot-web-server/src/main/java/org/springframework/boot/web/server/servlet/context/
  │ … +3 lines
  └ diff --git a/core/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/
    SpringBootApplication.java b/core/spring-boot-autoconfigure/src/main/java/org/springframework/boot/
    … +139 lines (ctrl + t to view transcript)
                 this.reader.register(ClassUtils.toClassArray(this.annotatedClasses));
             }

• Explored
  └ Read SpringApplication.java, DefaultApplicationContextFactory.java,
         AnnotationConfigServletWebServerApplicationContext.java, SpringBootApplication.java,
         BeanDefinitionLoader.java

───────────────────────────────────────────────────────────────────────────────────────────────────────────────────

• ../spring-boot 쪽에도 주석 추가해 두었습니다. 이제 main -> SpringApplication.run -> ApplicationContext 생성 ->
  source 로딩 -> refresh() 흐름을 코드에서 바로 따라갈 수 있습니다.

  핵심 진입점은 /Users/jungbumwoo/Documents/lab/spring-boot/core/spring-boot/src/main/java/org/springframework/
  boot/SpringApplication.java:318 입니다. 여기서 createApplicationContext()로 컨텍스트 인스턴스를 만들고, /Users/
  jungbumwoo/Documents/lab/spring-boot/core/spring-boot/src/main/java/org/springframework/boot/
  SpringApplication.java:321에서 BillionaireApiApplication.class 같은 primary source를 적재한 뒤, /Users/
  jungbumwoo/Documents/lab/spring-boot/core/spring-boot/src/main/java/org/springframework/boot/
  SpringApplication.java:446에서 framework의 applicationContext.refresh()로 넘깁니다. 컨텍스트 구현체 선택은 /
  Users/jungbumwoo/Documents/lab/spring-boot/core/spring-boot/src/main/java/org/springframework/boot/
  DefaultApplicationContextFactory.java:58이고, servlet 앱에서 자주 선택되는 구현체는 /Users/jungbumwoo/Documents/
  lab/spring-boot/module/spring-boot-web-server/src/main/java/org/springframework/boot/web/server/servlet/context/
  AnnotationConfigServletWebServerApplicationContext.java:73 입니다.

  @SpringBootApplication 자체가 @ComponentScan을 포함한다는 점도 /Users/jungbumwoo/Documents/lab/spring-boot/core/
  spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/SpringBootApplication.java:54에 표
  시해 두었고, 실제 primary source 등록은 /Users/jungbumwoo/Documents/lab/spring-boot/core/spring-boot/src/main/
  java/org/springframework/boot/BeanDefinitionLoader.java:128와 /Users/jungbumwoo/Documents/lab/spring-boot/core/
  spring-boot/src/main/java/org/springframework/boot/BeanDefinitionLoader.java:157에서 보시면 됩니다. Boot가 보관해
  둔 base package / annotated class를 refresh 도중 실제 스캔/등록하는 지점은 /Users/jungbumwoo/Documents/lab/
  spring-boot/module/spring-boot-web-server/src/main/java/org/springframework/boot/web/server/servlet/context/
  AnnotationConfigServletWebServerApplicationContext.java:202 입니다.

  빌드나 테스트는 실행하지 않았습니다. 원하시면 다음에는 이 흐름에 이어서 @EnableAutoConfiguration이 어디서 후보를
  읽고 import 하는지도 Boot 쪽에서 바로 이어서 추적해드리겠습니다.
```

---
```
1. AbstractApplicationContext.java

  - refresh() (line 582) — [Step 1~6] 전체 흐름 주석 추가
    - Step 1: obtainFreshBeanFactory() → BeanDefinition 로딩 (XML/어노테이션 경로 설명)
    - Step 2: prepareBeanFactory() → BeanFactory 준비
    - Step 3: invokeBeanFactoryPostProcessors() → ConfigurationClassPostProcessor, PropertySourcesPlaceholderConfigurer
    - Step 4: registerBeanPostProcessors() → AutowiredAnnotationBeanPostProcessor, AOP 프록시 생성기 등록
    - Step 5: finishBeanFactoryInitialization() → 싱글톤 Bean 생성 전체 체인 설명
    - Step 6: finishRefresh() → ContextRefreshedEvent 발행
  - registerShutdownHook() — JVM 셧다운 훅 등록 설명
  - doClose() — 종료 순서: ContextClosedEvent → Lifecycle 중지 → Bean 소멸 콜백 체인

  2. AbstractAutowireCapableBeanFactory.java

  - createBean() (line 488) — Bean 생성 진입점, 인스턴스화 전 BPP 숏서킷 설명
  - doCreateBean() (line 556) — 전체 6단계 요약 주석 + 각 단계별 인라인 주석
  - populateBean() (line 1390) — 의존성 주입 4단계 순서 주석
  - initializeBean() (line 1797) — 초기화 콜백 4단계 + AOP 프록시 생성 설명
  - invokeInitMethods() (line 1854) — afterPropertiesSet() vs custom init-method 순서, @PostConstruct와의 관계

  3. DefaultSingletonBeanRegistry.java

  - 필드 선언부 — L1/L2/L3 3단계 캐시 역할과 순환참조 해결 원리 설명
  - disposableBeans 필드 — 등록 역순 소멸 설명
  - getSingleton(name, allowEarlyRef) (line 208) — L1 → L2 → L3 탐색 흐름
  - getSingleton(name, factory) (line 255) — Bean 생성 위임 (createBean() 람다 호출) → L1 등록
  - destroySingletons() — 등록 역순 소멸 설명
  - destroyBean() — 의존 Bean 먼저 소멸 → DisposableBeanAdapter.destroy() 위임

  4. DisposableBeanAdapter.java

  - 클래스 선언부 — 소멸 콜백 어댑터 역할, 등록/호출 시점 설명
  - destroy() (line 197) — 소멸 콜백 4단계 순서:
    a. @PreDestroy (CommonAnnotationBeanPostProcessor)
    b. DisposableBean.destroy()
    c. AutoCloseable.close()
    d. custom destroy-method (자동 추론 설명 포함)
```

```sequenceDiagram
    autonumber

    participant Main  as CoinInfoApiApplication
    participant SA    as SpringApplication
    participant SAL   as SpringApplicationRunListeners
    participant Env   as ConfigurableEnvironment
    participant AACX  as AnnotationConfigServletWebServerApplicationContext
    participant DLBF  as DefaultListableBeanFactory
    participant PPRD  as PostProcessorRegistrationDelegate
    participant CCPP  as ConfigurationClassPostProcessor
    participant AACABF as AbstractAutowireCapableBeanFactory
    participant TWSF  as TomcatServletWebServerFactory
    participant DLP   as DefaultLifecycleProcessor

    Main->>SA: SpringApplication.run(CoinInfoApiApplication.class, args)

    rect rgb(230, 242, 255)
        Note over SA: [1단계] SpringApplication 초기화
        SA->>SA: new SpringApplication(primarySources)
        SA->>SA: WebApplicationType.deduceFromClasspath() → SERVLET
        SA->>SA: setInitializers() — spring.factories에서 ApplicationContextInitializer 로드
        SA->>SA: setListeners() — spring.factories에서 ApplicationListener 로드
        SA->>SA: deduceMainApplicationClass()
    end

    rect rgb(230, 255, 235)
        Note over SA,Env: [2단계] Environment 준비
        SA->>SA: createBootstrapContext()
        SA->>SAL: getRunListeners() → EventPublishingRunListener
        SA->>SAL: starting(bootstrapContext, mainClass)
        SA->>Env: prepareEnvironment(listeners, bootstrapContext, applicationArguments)
        SA->>Env: createEnvironment() → ApplicationServletEnvironment
        SA->>Env: configureEnvironment() — propertySources, profiles 구성
        SA->>SAL: environmentPrepared(bootstrapContext, environment)
        Note over Env: EnvironmentPostProcessor 순차 실행<br/>application.yml / application-{profile}.yml 로드<br/>외부 설정 소스 (AWS Secrets Manager 등) 로드<br/>spring.config.import 처리
        SA->>SA: bindToSpringApplication(environment)
        SA->>SA: printBanner(environment)
    end

    rect rgb(255, 250, 220)
        Note over SA,AACX: [3단계] ApplicationContext 생성 및 준비
        SA->>AACX: createApplicationContext()
        Note over AACX: AnnotationConfigServletWebServerApplicationContext 생성
        AACX->>DLBF: new DefaultListableBeanFactory()
        AACX->>AACX: new AnnotatedBeanDefinitionReader(this)
        AACX->>AACX: new ClassPathBeanDefinitionScanner(this)

        SA->>AACX: prepareContext(bootstrapContext, context, environment, listeners, ...)
        AACX->>AACX: setEnvironment(environment)
        SA->>SA: postProcessApplicationContext(context)
        SA->>SA: applyInitializers(context)
        Note over SA: ApplicationContextInitializer.initialize() 순차 실행
        SA->>SAL: contextPrepared(context)
        SA->>SA: load(context, sources)
        Note over SA: AnnotatedBeanDefinitionReader.register(CoinInfoApiApplication.class)<br/>→ DefaultListableBeanFactory.registerBeanDefinition() — 메인 클래스 최초 등록
        SA->>SAL: contextLoaded(context)
    end

    rect rgb(255, 232, 232)
        Note over AACX,DLBF: [4단계] refresh() — BeanFactory 기반 준비
        SA->>AACX: refreshContext(context) → AbstractApplicationContext.refresh()
        AACX->>AACX: prepareRefresh()
        Note over AACX: earlyApplicationEvents 초기화<br/>PropertySource 활성화
        AACX->>DLBF: obtainFreshBeanFactory() → refreshBeanFactory()
        Note over DLBF: serializationId 설정
        AACX->>AACX: prepareBeanFactory(beanFactory)
        Note over AACX,DLBF: BeanExpressionResolver 등록 (SpEL)<br/>PropertyEditorRegistrar 등록<br/>ApplicationContextAwareProcessor 등록<br/>환경 빈 등록 (environment, systemProperties, systemEnvironment)
        AACX->>AACX: postProcessBeanFactory(beanFactory)
        Note over AACX: request / session Scope 등록<br/>ServletContextAwareProcessor 등록
    end

    rect rgb(240, 255, 230)
        Note over AACX,CCPP: [5단계] refresh() — BeanFactoryPostProcessor 실행 (BeanDefinition 등록)
        AACX->>PPRD: invokeBeanFactoryPostProcessors(beanFactory)
        PPRD->>CCPP: postProcessBeanDefinitionRegistry(registry)
        CCPP->>CCPP: processConfigBeanDefinitions(registry)
        Note over CCPP: @SpringBootApplication 파싱<br/>@ComponentScan → 패키지 전체 스캔<br/>@EnableAutoConfiguration → AutoConfigurationImportSelector<br/>→ META-INF/spring/AutoConfiguration.imports 로드<br/>@Bean 메서드 메타데이터 수집
        CCPP->>DLBF: registerBeanDefinition() × N
        Note over DLBF: 모든 BeanDefinition 등록 완료
        PPRD->>PPRD: invokeBeanFactoryPostProcessors() — 일반 BeanFactoryPostProcessor 실행
        Note over PPRD: PropertySourcesPlaceholderConfigurer — ${...} 치환<br/>@ConfigurationProperties 바인딩
    end

    rect rgb(235, 235, 255)
        Note over AACX,AACABF: [6단계] refresh() — BeanPostProcessor 등록
        AACX->>PPRD: registerBeanPostProcessors(beanFactory)
        Note over PPRD: AutowiredAnnotationBeanPostProcessor 등록 (@Autowired, @Value)<br/>CommonAnnotationBeanPostProcessor 등록 (@PostConstruct, @PreDestroy)<br/>PersistenceAnnotationBeanPostProcessor 등록 (@PersistenceContext)<br/>우선순위(PriorityOrdered → Ordered → 나머지) 순으로 등록
    end

    rect rgb(255, 248, 225)
        Note over AACX: [7단계] refresh() — 인프라 빈 초기화
        AACX->>AACX: initMessageSource()
        Note over AACX: MessageSource 빈 등록 (없으면 DelegatingMessageSource)
        AACX->>AACX: initApplicationEventMulticaster()
        Note over AACX: SimpleApplicationEventMulticaster 등록
    end

    rect rgb(220, 255, 245)
        Note over AACX,TWSF: [8단계] refresh() — 내장 웹 서버 생성 (onRefresh)
        AACX->>AACX: onRefresh()
        AACX->>TWSF: createWebServer()
        TWSF->>TWSF: getWebServer(getSelfInitializer())
        Note over TWSF: Tomcat 인스턴스 생성<br/>Connector 구성 (port: 8080, management: 9080)<br/>StandardContext 구성<br/>VirtualThreadConfig 적용 (Java 21)
        TWSF-->>AACX: TomcatWebServer (미기동 상태)
    end

    rect rgb(250, 235, 255)
        Note over AACX,AACABF: [9단계] refresh() — 싱글톤 Bean 일괄 초기화 (finishBeanFactoryInitialization)
        AACX->>AACX: registerListeners()
        Note over AACX: ApplicationListener 빈 등록<br/>earlyApplicationEvents 발행
        AACX->>AACX: finishBeanFactoryInitialization(beanFactory)
        AACX->>DLBF: preInstantiateSingletons()
        Note over DLBF: non-lazy 싱글톤 BeanDefinition 순회

        loop 각 BeanDefinition (non-lazy singleton)
            DLBF->>AACABF: getBean(beanName) → doGetBean() → createBean()
            AACABF->>AACABF: instantiateBean() — 생성자 호출
            AACABF->>AACABF: populateBean() — 의존성 주입 (@Autowired, @Value)
            AACABF->>AACABF: initializeBean()
            Note over AACABF: Aware 인터페이스 콜백 (ApplicationContextAware 등)<br/>BeanPostProcessor.postProcessBeforeInitialization()<br/>@PostConstruct 메서드 실행<br/>InitializingBean.afterPropertiesSet()<br/>BeanPostProcessor.postProcessAfterInitialization()
        end

        Note over DLBF: SmartInitializingSingleton.afterSingletonsInstantiated() 실행
    end

    rect rgb(235, 248, 255)
        Note over AACX,DLP: [10단계] refresh() — 완료 처리 (finishRefresh)
        AACX->>AACX: finishRefresh()
        AACX->>AACX: initLifecycleProcessor()
        AACX->>DLP: onRefresh()
        Note over DLP: SmartLifecycle 빈 phase 오름차순으로 start() 호출
        AACX->>TWSF: TomcatWebServer.start()
        Note over TWSF: Tomcat 실제 기동<br/>DispatcherServlet.init() 실행<br/>HandlerMapping / HandlerAdapter 초기화<br/>포트 바인딩 완료 (8080 / 9080)
        AACX->>AACX: publishEvent(new ContextRefreshedEvent(this))
    end

    rect rgb(240, 255, 240)
        Note over SA,DLP: [11단계] 후처리 — Runner 실행 및 기동 완료
        SA->>SA: afterRefresh(context, applicationArguments)
        SA->>SAL: started(context, timeTakenToStartup)
        SA->>SA: callRunners(context, applicationArguments)
        Note over SA: ApplicationRunner.run() 순차 실행<br/>CommandLineRunner.run() 순차 실행
        SA->>SAL: ready(context, timeTakenToReady)
        Note over SA: 기동 완료 — HTTP 요청 수신 가능 상태
    end
```

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