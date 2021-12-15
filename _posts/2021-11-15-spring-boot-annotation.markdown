---
title: "Spring boot 주요 어노테이션 정리"
excerpt: "Spring Boot 에서 사용하는 주요 어노테이션을 표로 정리."

header:
  overlay_image: https://images.unsplash.com/photo-1501785888041-af3ef285b470?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1350&q=80
  overlay_filter: 0.5

tags:
  - Java
  - Spring

toc: true
toc_label: "table of content"
toc_icon: "bars"
toc_sticky: true
---

<br/>

---

## Spring Boot Annotations 정리

| Annotations            | Description                                          |
| ---------------------- | ---------------------------------------------------- |
| @SpringBootApplication | Spring boot application 으로 설정                    |
| @Controller            | View 를 제공하는 controller 로 설정                  |
| @RestController        | REST API 를 제공하는 controller 로 설정              |
| @RequestMapping        | URL 주소를 매핑                                      |
| @GetMapping            | Http GetMethod URL 주소 맵핑                         |
| @PostMapping           | Http PostMethod URL 주소 맵핑                        |
| @PutMapping            | Http PutMethod URL 주소 맵핑                         |
| @DeleteMapping         | Http DeleteMethod URL 주소 맵핑                      |
| @RequestParam          | URL Query Parameter 맵핑                             |
| @RequestBody           | Http Body 를 Parsing 맵핑                            |
| @Valid                 | POJO Java class 의 검증                              |
| @Configuration         | 1개 이상의 bean 을 등록할 때 설정                    |
| @Component             | 1개의 class 단위로 등록할 때 사용                    |
| @Bean                  | 1개의 외부 library 로부터 생성한 객체를 등록 시 사용 |
| @Autowired             | DI를 위한 곳에 사용                                  |
| @Qualifier             | @Autowired 사용 시 bean이 2개 이상일 때 명시적 사용  |
| @Resource              | @Autowired + @Qualifier 의 개념으로 이해             |
| @Aspect                | AOP 적용시 사용                                      |
| @Before                | AOP 메소드 이전 호출 지정                            |
| @After                 | AOP 메소드 호출 이후 지정 예외 발생 포함             |
| @Around                | AOP 이전/이후 모두 포함 예외 발생 포함               |
| @AfterReturning        | AOP 메소드의 호출이 정상일 때 실행                   |
| @AfterThrowing         | AOP시 해당 메소드가 예외 발생시 지정                 |
