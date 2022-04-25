---
title: "Spring Boot - Swagger"
excerpt: "Swagger 로 API 문서 공개하기"

header:
  overlay_image: https://images.unsplash.com/photo-1501785888041-af3ef285b470?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1350&q=80
  overlay_filter: 0.5

tags:
  - Java
  - Spring Boot
  - Swagger

toc: true
toc_label: "table of content"
toc_icon: "bars"
toc_sticky: true
---

# Swagger

nodejs 로 Swagger 문서 만들 땐 상당히 귀찮았었던 기억이 있는데 spring boot 의 swagger 는 상당히 편리함. 메모용 포스트를 남겨보자~

<br/>

# 기본 설정

## build.gradle

```gradle
dependencies {
  implementation group: 'io.springfox', name: 'springfox-boot-starter', version: '3.0.0'
}
```

한 줄 넣어주고 빌드. 사실 이 것으로 swagger를 사용하기 위한 설정은 끝임. `http://{호스트}/swagger-ui/#/` 로 접속해보면 이미 페이지가 잘 나올 것임.

포스트 작성 기준으로 작업 중인 서버에 3개 controller가 작성되어 있는데, 예를 들어 아래와 같은 controller 가 있다고 치면 그 아래의 그림처럼 페이지에 작성한 controller 가 표시될 것임.

```java
@RestController
@RequestMapping("/api")
@RequiredArgsConstructor
public class GroupController {

  @PostMapping("/")
  public ResponseEntity<GroupRegisterResDto> groupRegister(
      @RequestBody @Valid GroupRegisterReqDto rawBody
    ) {
    return groupService.groupRegisterAction(rawBody);
  }
}
```

![시작 페이지](/images/2022-04-25-springboot-swagger/1.png)

<br/>

## Config 파일 생성 (Header trouble shooting)

`build.gradle` 에 한 줄 넣고 끝날 줄 알았으면 좋았겠지만, 이상한 문제 때문에 하루를 통째로 날려먹었다. 아래와 같은 API가 있다고 쳐보자.

```java
@RestController
@RequestMapping("/v2/group")
@RequiredArgsConstructor
public class GroupController {
  @CheckAuth
  @PutMapping("/modify")
  public ResponseEntity<Object> groupModify(
    @ApiParam(value = "access token", required = true) @RequestHeader(value = "Authorization") String authorization,
    @RequestBody @Valid GroupModifyReqDto rawBody
  ) {
    return groupService.groupModifyAction(rawBody);
  }
}
```

그럼 swagger-ui 에는 잘 나옴. 문제는 `Authorization` 으로 선언한 필드에 값을 집어넣어도 curl 명령에 설정한 헤더값이 제대로 들어가 있지 않는 것을 볼 수 있음.

![이거땜에하루날림](/images/2022-04-25-springboot-swagger/2.png)

그리고 왜 인지는 나는 자바 어린이이기 때문에 알 순 없지만 그냥 일반적인 `SwaggerConfig` 객체를 하나 생성하여 `@EnableSwagger2` 어노테이션과 함께 빈 등록 하면 해결되긴 하더라.

```java
@Configuration
@EnableSwagger2
public class SwaggerConfig {
  @Bean
  public Docket api() {
    return new Docket(DocumentationType.SWAGGER_2)
        .select()
        .apis(RequestHandlerSelectors.any())
        .paths(PathSelectors.any())
        .build().apiInfo(apiInfo());
  }

  private ApiInfo apiInfo() {
    return new ApiInfoBuilder().title("나의 작은 API서버")
        .description("나작API서버")
        .version("1.0")
        .build();
  }
}
```

그래서 Header 값을 따로 parameter 로 받지 않을 서버라면 굳이 config 파일을 생성할 필요 없겠지만,,,, 보통은.. 특히나 JWT를 이용하여 인증 서비스를 구축한 경우 header 에 access token 을 많이 담아 보내기 때문에 config 파일을 생성하자. <h6><s>어노테이션 이름이 @EnableSwagger3 이었으면 빨리 시도해봤을텐데 시ㅂ</s></h6>

<br/>

# Description

이미 이대로도 서버에 관계되어 있는 이해관계자들이 api 서버를 테스트 해 볼 수 있는 환경의 구성은 끝났다. 하지만 내가 지은 끔찍한 변수 이름 때문에 테스트 해보는 사람이 필드에 무슨 값을 넣어야 하는 지 모르고 swagger 문서는 있으나 마나 또 나한테 귀찮은 질문 세례가 쏟아질 것이다. 친절한 문서를 작성해보자.

<br/>

## Parameter Description

`@ApiModelProperty` 어노테이션을 사용해보자.

```java
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class GroupListAllReqDto {
  @ApiModelProperty(value = "Number 의 타입 ('resource' 혹은 'user')", example = "resource", required = true)
  @NotNull private String type;

  @ApiModelProperty(value = "resourceNo 혹은 userNo", example = "Rl3Tsjl", required = true)
  @NotNull private String number;
}
```

![난 너무 친절해](/images/2022-04-25-springboot-swagger/3.png)

이 정도면 꽤 친절하지 않을까.

Response DTO 에도 똑같이 친절하게 작성하면 된다.

<br/>

## API Description

`@ApiOperation` 어노테이션을 사용해보자.

```java
@RestController
@RequestMapping("/v2/group")
@RequiredArgsConstructor
public class GroupController {
  @ApiOperation(value = "그룹 내 모바일 사용자를 삭제.")
  @CheckAuth
  @DeleteMapping("/mo")
  public ResponseEntity<Object> groupMoDelete(
      @ApiParam(value = "인증 토큰 타입과 정보", required = true) @RequestHeader(value = "Authorization") String authorization,
      @RequestBody @Valid GroupMoDeleteReqDto rawBody
      ) {
    return groupService.groupMoDeleteAction(rawBody);
  }
}
```

사진은 아래거랑 같이 봅시다.

<br/>

## Response Description

`@ApiResponses` 와 `@ApiResponse` 를 사용해보자.

```java
@RestController
@RequestMapping("/v2/group")
@RequiredArgsConstructor
public class GroupController {
  @ApiResponses({
      @ApiResponse(code = 1, message = "뭐"),
      @ApiResponse(code = 2, message = "요")
  })
  @ApiOperation(value = "그룹 내 모바일 사용자를 삭제.")
  @CheckAuth
  @DeleteMapping("/mo")
  public ResponseEntity<Object> groupMoDelete(
      @ApiParam(value = "인증 토큰 타입과 정보", required = true) @RequestHeader(value = "Authorization") String authorization,
      @RequestBody @Valid GroupMoDeleteReqDto rawBody
      ) {
    return groupService.groupMoDeleteAction(rawBody);
  }
}
```

![](/images/2022-04-25-springboot-swagger/4.png)

네. 그렇습니다.

<br/>

# 갈무리
