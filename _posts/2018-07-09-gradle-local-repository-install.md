---
layout: post
title: gradle로 jar파일 로컬 저장소에 설치하고 재사용하기
author: 하얀눈길
description: how to install jar to local repository by gradle
tags: [jar, gradle, gradle-install, java]
featuredImage: 
img: 
categories: java
date: '2018-07-09'
extensions:
  preset: gfm

---





## gradle로 jar파일 로컬 저장소에 설치하고 재사용하기

내가 만든 jar를 다른 프로젝트에서 사용하려는 경우는 자주 있습니다.
헌데, gradle로 해 보려니 이것도 많이 찾아봐야 하는 군요.(물론 구글링 잘하면 단 1분만에 끝낼 수 있습니다 :sob:

### install 하려는 프로젝트
build.gradle 파일에서 maven plugin을 허용하고, group, version을 세팅해 줍니다.

    apply plugin: "maven"
    
	group = "foo"
	version = "1.0"	

그런 다음,  `gradle install`  명령을 입력하면 됩니다.


### 사용하려는 프로젝트
build.gradle 파일에서 로컬저장소와 의존성을 설정해 줍니다.

    repositories {
	    mavenLocal()
	}
	
	dependencies {
		compile "foo:sdk:1.0"
	}

그런 다음,  `gradle build` 명령을 입력하면 됩니다.
 
## 참조
https://stackoverflow.com/questions/6122252/gradle-alternate-to-mvn-install


 



