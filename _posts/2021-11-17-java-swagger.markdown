---
title: "Java - Swagger 로 API 공개하기"
excerpt: "Spring boot 프로젝트에서 Swagger 를 통해 API 를 공개하는 법을 알아봅니다."

header:
  overlay_image: https://images.unsplash.com/photo-1501785888041-af3ef285b470?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1350&q=80
  overlay_filter: 0.5

tags:
  - Java
  - Spring
  - Swagger

toc: true
toc_label: "table of content"
toc_icon: "bars"
toc_sticky: true
---

## Swagger 어노테이션 정리

| Annotation         | Description                               |
| ------------------ | ----------------------------------------- |
| @Api               | 클래스를 스웨거의 리소스로 표시           |
| @ApiOperation      | 특정ㅈ 경로의 오퍼레이션 HTTP 메소드 설명 |
| @ApiParam          | 오퍼레이션 파라미터에 메타 데이터 설명    |
| @ApiResponse       | 오퍼레이션의 응답 지정                    |
| @ApiModelProperty  | 모델의 속성 데이터를 설명                 |
| @ApiImplicitParam  | 메소드 단위의 오퍼레이션 파라미터를 설명  |
| @ApiImplicitParams | -                                         |
