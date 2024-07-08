---
title: "[프로젝트] Uploadcontroller 리팩터링"
layout: post
date: 2024-07-08 18:07
image: 
headerImage: false
tag:
- 자바
- 프로젝트
- 스프링 부트 
- RestController
- Controller
category: blog
author: Chaerin
description: 프로젝트 코드 리팩토링
---

## @RestController VS @Controller

프로젝트 진행 중 컨트롤러 애노테이션의 올바른 사용방법에 의문을 느껴 검색하고 공부한 내용입니다.

### @Controller

이 애노테이션이 클래스가 "컨트롤러" 임을 나타냅니다. (예: 웹 컨트롤러) \
이 애노테이션은 클래스 경로 탐색을 통해 자동으로 탐지되도록 허용함으로써 @Component의 특별성을 제공합니다.\
보통 @RequestMapping 이 달린 메서드와 함께 사용됩니다.\
또한 내부에 Optional Element로서 String value 만을 가집니다.\
이 value는 logical component name을 가리킨다고 합니다.\
즉, Controller 클래스가 Bean으로 등록될 때의 이름을 변경하고 싶을 때 사용하는 요소입니다.

### @RestController

@RestController는 그 자체로 @Controller와 @ResponseBody가 함께 있는 편리한 애노테이션입니다.\
이 애노테이션이 포함된 타입은 @RequestMapping가 달린 메서드에 기본적으로 @ResponseBody도 함께 의미합니다. \
해당 컨트롤러 내 모든 메서드들은 반환 시, 웹 응답 body부에 값을 넣어 반환한다. \
즉, 사용자의 요청에 따라 데이터의 형식을 다르게 반환하고 싶을 때, 이 둘을 구분해서 사용해야 합니다.

### 리팩토링 전

```java
package com.WalkiePaw.presentation.domain.upload;

import lombok.RequiredArgsConstructor;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.multipart.MultipartFile;

@Controller
@RequiredArgsConstructor
@RequestMapping("/api/v1/uploads")
public class UploadController {

    private final UploadService uploadService;

    @PostMapping
    public ResponseEntity<String> uploadImage(@RequestParam MultipartFile file) {
        String uploadURL = uploadService.upload(file);
        return ResponseEntity.ok(uploadURL);
    }
}

```

또한 해당 메서드의 반환 타입을 String으로 할 때, JSON 파싱이 안되는 문제 발견.
그래서 UploadUrlResponse를 생성, 객체로 반환 처리하였습니다.

해당 기능은 스프링 내부의 HTTP 메시지 컨버터 인터페이스가 적용되었습니다. (객체 -> JSON) 

[관련 링크](https://velog.io/@woo00oo/HTTP-%EB%A9%94%EC%8B%9C%EC%A7%80-%EC%BB%A8%EB%B2%84%ED%84%B0)

### 리팩토링 후 코드 
```java

package com.WalkiePaw.presentation.domain.upload;

import lombok.Data;

@Data
public class UploadUrlResponse {
    private final String url;
}

````

```java

package com.WalkiePaw.presentation.domain.upload;

import lombok.RequiredArgsConstructor;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;

@RestController
@RequiredArgsConstructor
@RequestMapping("/api/v1/uploads")
public class UploadController {

    private final UploadService uploadService;

    @PostMapping
    public ResponseEntity<UploadUrlResponse> uploadImage(@RequestParam MultipartFile file) {
        String uploadURL = uploadService.upload(file);
        return ResponseEntity.ok(new UploadUrlResponse(uploadURL));
    }
}

```