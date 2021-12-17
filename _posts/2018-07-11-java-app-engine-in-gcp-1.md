---
layout: post
title: 자바 어플리케이션을 gcp app engine으로 구성하기 1 - 예제 실행해 보기
author: 하얀눈길
description: develop java application in app engine in gcp
tags: [app-engine, gcp, java]
featuredImage: 
img: Google-App.jpg
categories: [java, app engine]
date: '2018-07-11'
extensions:
  preset: gfm

---

> GCP에서 서비스 구축할 일이 있어서 이것저것 공부중입니다. 생각보다 쉬운데.. 문서를 이해하는 시간이 더 오래 걸리는군요.
> 이 시리즈는 아래 3개로 구성하였습니다. intelliJ를 사용하려고 이것저것 테스트하다보니 포스팅 시간이 오래걸리네요.

#### 관련 포스팅
1. [IntelliJ에 Google Cloud Tools 연동하기](http://www.irgroup.org/cloud-tools-for-intellij/)
2. [자바 어플리케이션을 gcp app engine으로 구성하기 1 - 예제 실행해 보기](http://www.irgroup.org/java-app-engine-in-gcp-1/) - 이 문서
3. [자바 어플리케이션을 gcp app engine으로 구성하기 2 - CloudSQL 연동](http://www.irgroup.org/java-app-engine-in-gcp-2/)


## 목표
* Hello World 샘플 프로그램을 통해 앱엔진 맛보기


## 예제로 앱 엔진 구성하기
standard environment 에서 앱엔진 구성

### 필요한 툴
[Java SE 8 Development Kit(JDK)](http://www.oracle.com/technetwork/java/javase/downloads/index.html) - 앱엔진은 7, 8만 지원
[Git](https://git-scm.com/)
[Maven](http://maven.apache.org/download.cgi)

### 선행 작업
1. [Google Cloud SDK 다운로드 및 설치](https://cloud.google.com/sdk/docs/)
2. `gcloud` 명령으로 command-line 환경 설정
	```
	gcloud init  
    gcloud auth application-default login
    ```
3. 앱엔진 컴포넌트 설치
	```
	gcloud components install app-engine-java
	```

4. 최신 버전으로 업데이트
	```
	gcloud components update
	```

## 샘플 실행해보기
### Hellow Wordl app 다운로드
아래 명령을 사용하여 git에서 샘플 프로그램 다운로드하면 GCP의 getting started 샘플 프로그램을 모두 받을 수 있다.

```
git clone https://github.com/GoogleCloudPlatform/getting-started-java.git`
```

샘플 코드가 들어 있는 디렉토리로 이동
```
cd getting-started-java/appengine-standard-java8/helloworld`
```

### 로컬에서 실행해 보기
1. Jetty Maven plugin을 통해 로컬에서 Jetty server 실행
	```
	mvn appengine:run
	```
2. 브라우저에서 확인
	```
	http://localhost:8080
	```
3. 브라우저에서 `Hellow World` 메세지가 보이면 정상 
4. 터미널에서 `Ctrl`+`C` 로 web server 종료


### 앱엔진에 배포하고 실행해 보기
1. `getting-started-java/appengine-standard-java8/hellowworld` 디렉토리에서 아래 명령을 통해 배포
	```
	mvn appengine:deploy
	```
2. 아래 명령을 통해 브라우저 실행하면 `http://my_project_id.appspot.com`으로 접속하여 결과를 확인 할 수 있음
	```
	gcloud app browse
	```
3. 결과 확인
브라우저가 실행되면서 아래와 같이 화면이 나타나면 성공~

![실행 결과 화면](http://www.irgroup.org/assets/img/Hello_App_Engine_Standard_Java_8.jpg)


## 코드 리뷰
Hellow World는 가장 단순하게 만들어진 앱엔진 앱이다. 따라서 기본적인 구조를 파악하기 위해서는 아주 제격이다.

### HelloAppEngine.java
소스는 이 파일 하나로 구성되어 있다. 단일 버전, 다인 서비스로 구성되어 있는 단순한 구조이기 때문에 이 파일 하나로도 충분하다.

```java
import com.google.appengine.api.utils.SystemProperty;  
  
import java.io.IOException;  
import java.util.Properties;  
  
import javax.servlet.annotation.WebServlet;  
import javax.servlet.http.HttpServlet;  
import javax.servlet.http.HttpServletRequest;  
import javax.servlet.http.HttpServletResponse;  
  
// With @WebServlet annotation the webapp/WEB-INF/web.xml is no longer required.  
@WebServlet(name =  "HelloAppEngine", value =  "/hello")  
public  class  HelloAppEngine  extends  HttpServlet  
{  
	@Override  
	public  void doGet(HttpServletRequest request,  HttpServletResponse response)  throws  IOException  
	{  
		Properties properties =  System.getProperties(); 
		response.setContentType("text/plain"); 
		response.getWriter().println("Hello App Engine - Standard using "  
		+  SystemProperty.version.get()  
		+  " Java "
		+ properties.get("java.specification.version"));  
	}  

	public  static  String getInfo()  
	{  
		return  "Version: "  +  
			System.getProperty("java.version")  +  " OS: "  +  
			System.getProperty("os.name")  +  " User: "  +  System.getProperty("user.name");  
	}  
}
```

소스는 사실 별게 없다. 서블릿 프로그램을 많이 해본 분들이라면 그냥 봐도 알만한 내용..
이쪽 경험이 별로 없는 필자는 `@WebServlet` annotation을 처음 본지라, `web.xml` 설정하기 위한 번거로움이 필요 없다는 것을 새로 안 정도~

브라우저에서 `hello` 진입점으로 들어오게 되면 `doGet` 메소드가 실행되고 결과를 출력한다. `getInfo`메소드는 index.jsp에서 호출하는데, 처음 페이지를 만들때 사용된다(쉽네~)

### pom.xml
maven plugin을 사용하기 위해 아래 내용이 추가되어 있다. 

```xml
<plugin>
  <groupId>com.google.cloud.tools</groupId>
  <artifactId>appengine-maven-plugin</artifactId>  
  <version>1.3.1</version>  
</plugin>
```

### appengine-web.xml
runtime을 위한 java8과 같이 꼭 필요한 설정이 들어 있다. 
이 파일에 대한 설명은 [appengine-web.xml Reference](https://cloud.google.com/appengine/docs/standard/java/config/appref) 를 참조하자

```xml
<appengine-web-app  xmlns="http://appengine.google.com/ns/1.0">  
	<runtime>java8</runtime>
	<threadsafe>true</threadsafe>  
</appengine-web-app>
```


## Gradle로 실행하기
소스를 보니 gradle 환경설정이 되어 있다. 그래서 간단히 실행해보니 동일하게 잘 동작한다.

### 로컬에서 실행

```bash
./gradlew appengineRun
```

### Deploy
```
./gradlew appengineDeploy
```

### 앱엔진 관련 명령
솔직히 다 모르겠다~ :-(
```
'appengineDeploy' 
'appengineDeployCron'
'appengineDeployDispatch'
'appengineDeployDos'
'appengineDeployIndex'
'appengineDeployQueue'
'appengineRun'
'appengineShowConfiguration'
'appengineStage'
'appengineStart'
'appengineStop'
```



## 참조
* 구글 공식 문서 : https://cloud.google.com/appengine/docs/standard/java/
* 앱엔진 개발 환경 세팅 : https://cloud.google.com/appengine/docs/standard/java/building-app/environment-setup

 




