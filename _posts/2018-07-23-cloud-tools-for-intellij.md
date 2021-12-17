---
layout: post
title: IntelliJ에 Google Cloud Tools 연동하기
author: 하얀눈길
description: Cloud Tools for IntelliJ
tags: [app-engine, gcp, intellij, cloud-tools, java]
featuredImage: 
img: Google-App.jpg
categories: [java, app engine, gcp, intellij]
date: '2018-07-23'
extensions:
  preset: gfm

---


> Intellij에서 Google Cloud Tools를 사용하려하고 찾아보니 구글문서가 깔끔하게 잘되어 있었다. 잊어버리기 전에 기록~
> 헌데, 내가 가진 intellij가 2016버전이라 다소 삽질을 좀 하고.. 결국 2018버전으로 업그래이드 후 재구성.. :sob:
> 문서를 좀 믿자~


## 필요한 툴
* [IntelliJ IDEA, 2017 or higher!!!](https://www.jetbrains.com/idea/#chooseYourEdition) - ultimate edition은 모두 지원하지만 그 이하는 제한적임. 
* [Git](https://git-scm.com/)
* [Maven 3.1 or later](http://maven.apache.org/download.cgi) 

왠만하면 gradle을 사용하려 하는데, GCP는 아직 maven을 더 잘 지원하는 듯 하다. 스트레스 받기 싫으면 아직은 GCP에서 gradle 사용하지 마시길..

아래 내용은 IntelliJ 2018.1.6 Ultimate Edition 기준임

## Cloud Tools 설치 및 설정
1. Preferences > Plugins > Browse repositories  클릭
    ![enter image description here](http://www.irgroup.org/assets/img/intellij-Preferences-plugins.jpg)

2.  Google Cloud Tools 선택 및 설치
  ![enter image description here](https://www.irgroup.org/assets/img/install_google_cloud_tools_in_intellij.jpg) 
    
3. Google App Engine Integration 메세지에 대한 처리
	위 처럼 설치하다보면 "Google App Engine Integration"을 disable 하겠냐는 메세지 창이 뜬다. 사실 이 녀석은 deprecate 됐다(https://cloud.google.com/tools/intellij/docs/migrate) . 구글 측에서는 Cloud Tools로 통합하여 관리하기 위해 버린 듯 하다.
	이 녀석은 삭제도 안된다.
	아무튼 disable하고 넘어가면 된다.

4. IntelliJ restart
5. JDK 설정
  FIle > Project Structure (`⌘` + `;`) > Project Settings > Project에서 JDK 세팅이 제대로 됐는지 확인

## 앱엔진 테스트 및 배포 환경 세팅
### Google App Engine Standard Local Server 세팅
App Engine Standard를 기준입니다.

1. Run > Edit Configurations 클릭
2. Run/Debug Configurations Dialog가 나타나면 왼쪽 상단 `+` 클릭하여 Google App Engine Standard Local Server 추가
   ![enter image description here](https://www.irgroup.org/assets/img/Run_Debug_Configurations.jpg)
3.  Artifact to deploy 변경
	만일 드롭다운 박스에 해당 artifact가 나타나지 않은다면, 프로젝트 세팅(`⌘` + `;`) 에서 artifacts를 추가해 줘야 한다.
	
4. App Engine Host/Port 를 원하는 것으로 설정
5. 환경변수 세팅
	![
](https://www.irgroup.org//assets/img/Run_Debug_Configurations2.jpg)
	GOOGLE_CLOUD_PROJECT 변수에 현재 사용중인 프로젝트 ID를 입력한다
	GOOGLE_APPLICATION_CREDENTIALS 변수에 IAM에서 서비스 키 json 파일을 받고 그 경로를 입력한다
	
여기까지 잘 됐다면 로컬에서 실행 및 디버깅이 가능하게 된다. 

### Deploy 환경 세팅
1. 메인 메뉴에서 Tools > Google Cloud Tools > Deploy to App Engine 클릭하면 아래와 같은 Dialog 박스가 나타난다
	![enter image description here](https://www.irgroup.org/assets/img/Create_Deployment_Configuration.jpg)
2. `...` 버튼을 클릭하면 Google App Engine 팝업창이 나타나고, 여기에서 "Google App Engine" 서버를 추가하자.
	만일 서버 목록이 없다면 `+` 버튼을 클릭하여 추가하면 된다
3. 서버를 추가하면 아래 그림처럼 Dialog 화면이 바뀌게 된다.
	![enter image description here](https://www.irgroup.org/assets/img/Create_Deployment_Configuration2.jpg)
4. Deployment 필드에 원하는 내용을 선택하자
	Community Edition을 사용중이면 Maven이나 Gradle artifact만 사용할 수 있다고 하는데.. 별 의미가 없다. (Community 버전을 살껄.. :sob:)
5. Project 필드에서 원하는 프로젝트를 선택하자. 
	만일, IntelliJ에서 처음 세팅하는 것이라면, 구글 accounnt로 로그인하라는 창이 나타날 것이다.
	인증을 진행하면 내 계정에 묶여 있는 모든 프로젝트가 나타난다. (이건 참 편리하다)
	
6. 다른 필드들도 필요에 따라 수정하면 된다.
7. 이제 준비는 끝났다. Run을 클릭하면 앱엔진에 내 사이트가 나타난다~ 와~~

### 기타
Tools > Google Cloud Tools 메뉴에 보면 여러가지 부가 기능들이 들어 있다. 유용한 것이 좀 있으니 자주 활용하자!



## 참조
* 구글 공식 문서 : https://cloud.google.com/tools/intellij/docs/how-to


 




