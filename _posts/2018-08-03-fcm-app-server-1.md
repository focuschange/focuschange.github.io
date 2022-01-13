---
layout: post
title: FCM push notification을 위한 앱서버 구현하기 1 - 기본개념 및 core
author: 하얀눈길
description: 
tags: [app-engine, fcm, push, push-notification, gcp, java]
featuredImage: 
img: 
categories: [java, app engine, gcp, fcm, push]
date: 2018-08-03 12:15:33 +0900
lastmod: 2022-01-13 12:15:33 +0900
extensions:
  preset: gfm

---

> 본 문서는 java로 FCM app server를 구축하기 위한 방법을 설명합니다.
> push만 대상으로 하기 때문에 굳이 XMPP를 사용할 필요가 없어서 HTTP 방식을 사용할 예정이고, 최신 api spec인 -   [FCM HTTP v1 API](https://firebase.google.com/docs/reference/fcm/rest/v1/projects.messages?hl=ko)을 사용합니다.




## 필요한 서버 환경	

* 클라이언트와 통신할 수 있습니다.
* FCM 서버에 올바르게 형식이 지정된 메시지 요청을 보낼 수 있습니다.
* [지수 백오프](https://developers.google.com/api-client-library/java/google-http-java-client/backoff?hl=ko)를 사용하여 요청을 처리하고 다시 보낼 수 있습니다.
* 서버 키와 클라이언트 등록 토큰을 안전하게 저장할 수 있습니다. 참고: 클라이언트 코드에  **절대로**  서버 키를 포함해서는 안 됩니다.
* XMPP의 경우 전송하는 각 메시지를 고유하게 구별하기 위해 서버에서 메시지 ID를 생성할 수 있어야 합니다. FCM HTTP 연결 서버는 메시지 ID를 생성하여 응답에서 반환합니다. XMPP 메시지 ID는 발신자 ID별로 고유해야 합니다.


## FCM 서버와 상호작용 방법
* [Admin SDK](https://firebase.google.com/docs/cloud-messaging/admin/?hl=ko)
* 원시 프로토콜
	* [FCM HTTP v1 API](https://firebase.google.com/docs/reference/fcm/rest/v1/projects.messages?hl=ko)는 가장 최신 프로토콜로서 보다 안전한 인증과 유연한 교차 플랫폼 메시징 기능을 제공
	* 이전의 [HTTP](https://firebase.google.com/docs/cloud-messaging/server?hl=ko#implementing-http-connection-server-protocol) 및 [XMPP](https://firebase.google.com/docs/cloud-messaging/server?hl=ko#implementing-the-xmpp-server-protocol) 서버 프로토콜 영역도 사용 가능
		>  클라이언트 애플리케이션에서 업스트림 메시징을 사용하려면 XMPP를 사용해야 함
		> node.js로 앱서버 구축시는 [Admin Node.js SDK의 FCM API](https://firebase.google.com/docs/cloud-messaging/admin/?hl=ko)를 사용하여 메시지를 보내야 함

XMPP 메시징은 다음 방식에 있어서 HTTP 메시징과 다릅니다.

-   업스트림/다운스트림 메시지
    -   HTTP: 클라우드에서 기기로 전송하는 다운스트림 전용
    -   XMPP: 기기에서 클라우드로 전송하는 업스트림 및 클라우드에서 기기로 전송하는 다운스트림
-   메시징(동기 또는 비동기)
    -   HTTP: 동기 메시징입니다. 앱 서버가 HTTP POST 요청으로 메시지를 보내고 응답을 기다립니다. 이 방식은 동기 메시징이며 응답이 수신될 때까지 발신자가 다른 메시지를 보내지 못하도록 차단합니다.
    -   XMPP: 비동기 메시징입니다. 앱 서버가 영구 XMPP 연결을 통해 최대 회선 속도로 모든 기기에서 양방향으로 메시지를 주고받습니다. XMPP 연결 서버가 확인 또는 실패 알림을 비동기 방식으로 보냅니다. 확인 형식은 JSON으로 인코딩된 특수한 ACK 및 NACK XMPP 메시지입니다.
-   JSON
    -   HTTP: JSON 메시지는 HTTP POST로 전송됩니다.
    -   XMPP: JSON 메시지는 XMPP 메시지에 캡슐화됩니다.
-   일반 텍스트
    -   HTTP: 일반 텍스트 메시지는 HTTP POST로 전송됩니다.
    -   XMPP: 지원되지 않습니다.
-   여러 등록 토큰에 전송되는 멀티캐스트 다운스트림
    -   HTTP: JSON 메시지 형식으로 지원됩니다.
    -   XMPP: 지원되지 않습니다.


## HTTP 서버 프로토콜 구현

메시지를 보내려면 앱 서버에서 JSON 키-값 쌍으로 구성된 HTTP 헤더와 HTTP 본문을 포함하는 POST 요청을 만듭니다. 헤더 및 본문의 옵션에 관한 자세한 내용은  [앱 서버 보내기 요청 작성](https://firebase.google.com/docs/cloud-messaging/send-message?hl=ko)을 참조

### HTTP v1의 장점
- **액세스 토큰을 통한 보안 향상**: HTTP v1 API는 OAuth2 보안 모델에 따라 수명이 짧은 액세스 토큰을 사용합니다. 액세스 토큰이 공개되는 경우에도 만료되기 전에 1시간 정도만 악의적으로 사용될 수 있습니다. 새로고침 토큰이 이전 API에서 사용하는 보안 키만큼 자주 전송되지 않으므로 캡처될 가능성이 매우 낮습니다.
    
-   **여러 플랫폼에서 보다 효율적인 메시지 맞춤설정**: 메시지 본문의 경우 HTTP v1 API에 모든 대상 인스턴스에 전달되는 공용 키는 물론 여러 플랫폼의 메시지를 맞춤설정할 수 있는 플랫폼별 키가 있습니다. 이러한 키를 사용하면 메시지 하나로 여러 클라이언트 플랫폼에 약간 다른 페이로드를 전송하는 '재정의'를 만들 수 있습니다.
    
-   **새 클라이언트 플랫폼 버전을 위한 확장성 강화 및 미래 경쟁력 확보**: HTTP v1 API는 iOS, Android, 웹에 제공되는 메시지 옵션을 완전히 지원합니다. 각 플랫폼마다 JSON 페이로드에 자체 정의된 블록이 있으므로 FCM에서 필요에 따라 새 버전과 새 플랫폼으로 API를 확장할 수 있습니다.

### HTTP v1 단점
- 기기 그룹 메시징이나 멀티캐스트 메시징을 사용하는 앱은 다음 버전의 API를 기다리는 것이 좋습니다. HTTP v1은 이전 API의 해당 기능을 지원하지 않습니다.

이제부터는 기존 프로토콜에 대한 언급은 피하겠습니다.
필요하다면 https://firebase.google.com/docs/cloud-messaging/server?hl=ko 참고하시길~

### 보내기 요청 승인
서비스 계정을 인증하고 Firebase 서비스에 액세스하도록 승인하려면 JSON 형식의 비공개 키 파일을 생성하고 이 키를 사용하여 수명이 짧은 OAuth 2.0 토큰을 발급받아야 합니다. 유효한 토큰을 확보했으면 원격 구성, FCM 등 다양한 Firebase 서비스의 요구사항에 따라 서버 요청에 토큰을 추가할 수 있습니다.
다른 서비스 계정을 사용하는 경우 편집자 또는 소유자 권한이 있어야 합니다.


**서비스 계정에 대한 비공개 키 파일을 생성하는 방법**

1.  Firebase 콘솔에서  **설정>  [서비스 계정](https://console.firebase.google.com/project/_/settings/serviceaccounts/adminsdk?hl=ko)**을 열기.
2.  **새 비공개 키 생성**을 클릭하고  **키 생성**을 클릭하여 확인.
3.  키가 들어있는 JSON 파일을 안전하게 저장. 

엑세스 토큰을 발급받기 위해서 비공개 키 Json은 필수입니다.

**액세스 토큰을 발급받는 방법**

토큰을 발급받으려면 사용할 언어에 대한  [Google API 클라이언트 라이브러리](https://developers.google.com/api-client-library/?hl=ko)를 사용하여 다음과 같이 비공개 키 JSON 파일을 참조합니다.

```java
private  static  String getAccessToken()  throws  IOException  
{  
	GoogleCredential googleCredential =  GoogleCredential  
		.fromStream(new  FileInputStream("service-account.json"))  
		.createScoped(Arrays.asList(SCOPES));
		
	googleCredential.refreshToken();  
	return googleCredential.getAccessToken();  
}
```

`service--account.json`에 Firebase console에서 받은 json 파일을 입력합니다. 
꽤나 헤매던 부분인데, SCOPES에 대한 언급을 찾을 수 없었습니다.
답은 아래 url을 참고하시면 됩니다.
https://stackoverflow.com/questions/47325150/where-to-get-scopes-dependencies-for-java-for-new-firebase-cloud-message-api 

 저는 아래 3개 url로 세팅했습니다.
 ```html
 https://www.googleapis.com/auth/firebase
 https://www.googleapis.com/auth/cloud-platform
 https://www.googleapis.com/auth/firebase.readonly
 ```

AccessToken은 보안을 위해 주기적으로 바뀌는데, 위 소스를 적용하면 토큰이 만료되면 토큰 새로고침 메소드가 자동으로 호출되어 업데이트된 토큰이 발급됩니다. 



**HTTP 요청 헤더에 액세스 토큰을 추가하는 방법**

`Authorization`  헤더의 값으로 토큰을  `Authorization: Bearer <access_token>`  형식으로 추가합니다.

```java
URL url =  new URL(FCM_SEND_ENDPOINT);  
HttpURLConnection httpURLConnection =  (HttpURLConnection) url.openConnection();  
httpURLConnection.setRequestProperty("Authorization",  "Bearer "  + getAccessToken());  
httpURLConnection.setRequestProperty("Content-Type",  "application/json; UTF-8");  
return httpURLConnection;
```

여기까지 잘 됐다면 보낼 준비가 끝난 상태입니다. 
아직 프로그램을 구동시켜보지 못하겠지만, 다음 단계에서 curl을 이용하여 아래와 같이 테스트 해 볼 수 있습니다.

#### 앱 서버 보내기 요청 작성 


**보내기 요청의 전송 유형**
-   주제 이름
-   조건
-   기기 등록 토큰
-   기기 그룹 이름(기존 프로토콜만 해당)

자세한 내용은 [메시지 유형](https://firebase.google.com/docs/cloud-messaging/concept-options?hl=ko#notifications_and_data_messages)을 참조 바랍니다.


send endpoint는 다음과 같이 구성됩니다.
```
POST https://fcm.googleapis.com/v1/projects/[myproject-name]/messages:send
```
자신의 endpoint url은 Firevase 콘솔의 [일반 프로젝트 설정](https://console.firebase.google.com/project/_/settings/general/?hl=ko) 탭에서 확인할 수 있습니다. 단지, 프로젝트 ID만 알고 있으면 됩니다.


**특정 기기에 메시지 전송**
메세지 전송은 https POST로 전송합니다.

```bash
curl -X POST -H "Authorization: Bearer ya29.c.El7uBYyvqs1..." -H "Content-Type: application/json" -d '{
"message":{
  "notification": {
    "title": "FCM Message",
    "body": "This is an FCM Message",
  },
  "token": "fh1Ego6mhk0:APA91bF..."
  }
}' "https://fcm.googleapis.com/v1/projects/my-fcm-project/messages:send"
```

`Authorization`에 access token을 넣어줍니다. `message.token` 은 클라이언트 사용자의 등록 토큰입니다.
만일 클라이언트가 준비되어 있다면 거기서 생성된 등록 토큰을 넣어주면 됩니다.

정상적으로 전송되면 아래와 같은 응답이 오게 됩니다.

```javascript
{  
  "name":  "projects/myproject-b5ae1/messages/0:1500415314455276%31bd1c9631bd1c96"  
}
```

요청 본문에 대해 보다 자세한 사항은 [FCM 메시지 정보](https://firebase.google.com/docs/cloud-messaging/concept-options?hl=ko) 또는 [API 참조](https://firebase.google.com/docs/reference/fcm/rest/v1/projects.messages?hl=ko)를 확인하세요



## 코드 작성
FCM Server Key를 받았다면 Access Token을 다음과 같이 받아올 수 있습니다.
헌데, 이 작업은 [구글 API 클라이언트 라이브러리](https://developers.google.com/api-client-library/java/?hl=ko)를 사용합니다.

따라서 maven 의존성을 아래와 같이 설정해 줍니다.

```xml
<project>
  <dependencies>
    <dependency>
      <groupId>com.google.api-client</groupId>
      <artifactId>google-api-client</artifactId>  
      <version>1.23.0</version>  
    </dependency>  
  </dependencies>  
</project>
```

access token 을 받아오기 위한 클래스를 아래와 같이 정의합니다.
```java
package com.mysite.fcm.manager;  
  
import com.google.api.client.googleapis.auth.oauth2.GoogleCredential;  
  
import java.io.IOException;  
import java.io.InputStream;  
import java.util.List;  

public class AccessToken  
{  
  
   public static String getAccessToken(String keyFile, List<String> scopes) throws IOException  
   {  
  
      InputStream resourceAsStream = AccessToken.class  
  .getResourceAsStream("/" + keyFile);  
  
	  GoogleCredential googleCredential = GoogleCredential  
            .fromStream(resourceAsStream)  
            .createScoped(scopes);  
  
	  googleCredential.refreshToken();  
  
	 return googleCredential.getAccessToken();  
  }  
}
```

> Firebase 예제들은 dependency, import를 명시적으로 알려주지 않더군요.
> 이런거 찾다가 시간 다 보낸다는.... :sob: 

메세지를 보내기 위한 클래스를 작성합니다.
*지저분한 코드는 나중에 정리.. 안되어 있으면 알아서 보시길~*

```java
package com.mysite.fcm.manager;  
  
import java.io.BufferedReader;  
import java.io.IOException;  
import java.io.InputStreamReader;  
import java.io.OutputStream;  
import java.net.HttpURLConnection;  
import java.net.URL;  
import java.util.ArrayList;  
import java.util.Iterator;  
import java.util.List;  
import java.util.Map;  
  
public class Push  
{  
	String            endpoint;  
	List<String>      scope;  
	String            keyFile;  
	String            accessToken;  
	HttpURLConnection http;  
	StringBuffer      responseBody;  
  
	public Push() throws IOException  
	{  
	    this.endpoint = System.getProperty("endpoint");  
		this.keyFile = System.getProperty("fcm_key");  
	  
		String tmp[] = System.getProperty("scope").split(",");  
		this.scope = new ArrayList<>();  
		for (String s : tmp)  
		{  
			this.scope.add(s);  
	    }  
	  
		this.accessToken = AccessToken.getAccessToken(keyFile, scope);
		responseBody = new StringBuffer();  
	}  

	public String getEndpoint()  
	{  
		return endpoint;  
	}  
  
    public void setEndpoint(String endpoint)  
    {  
		this.endpoint = endpoint;  
	}  
  
	public String getAccessToken()  
	{  
		return accessToken;  
	}  
  
	public void setAccessToken(String accessToken)  
	{  
		this.accessToken = accessToken;  
	}  
  
	public StringBuffer getResponseBody()  
	{  
		return responseBody;  
	}  
  
	public String send(String userToken, String message) throws IOException  
	{  
		URL url = new URL(endpoint);  
		http = (HttpURLConnection) url.openConnection();  
  
		http.setRequestMethod("POST");  
		http.setDoInput(true);  
		http.setRequestProperty("Authorization", "Bearer " + accessToken);  
		http.setRequestProperty("Content-Type", "application/json; UTF-8");  
  
		http.setDoOutput(true);  
		OutputStream os = http.getOutputStream();  

		String body =  
            "{\n" + "\"message\":{\n" + " \"notification\": {\n" + " \"title\": \"FCM Message\",\n"  
			+ " \"body\": \"" + message + "\",\n"  
			+ "  },\n" + " \"token\": \"" + userToken + "\"\n" + "  }\n" + "}\n";  
  
		  System.out.println(body);  
  
		os.write(body.getBytes());  
		os.flush();  
  
		os.close();  
  
		System.out.println("* CODE : " + http.getResponseCode());  
		System.out.println("* MSG  : " + http.getResponseMessage());  
  
		if(http.getResponseCode() == 200)  
		{  
			BufferedReader br = new BufferedReader(new InputStreamReader(http.getInputStream(), "UTF8"));  
  
			String line;  
			while ((line = br.readLine()) != null)  
			{  
				responseBody.append(line);  
			}  
  
			br.close();  
		}  
  
		 http.disconnect();  
  
		return http.getResponseMessage();  
	}  
}

```

이제 push main만 만들면 된다.

```java
public static void main(String[] args) throws IOException 
{  
	Push push = new Push();  
	try 
	{  
		push.send("fPmC2-5G8dU:APA...(client 등록 토큰 입력)", "잘 갑니다~~");  
  } catch (IOException e) {  
      e.printStackTrace();  
  }  
}
```

실행해 보니 잘 가네요.

## 문제 해결
처음 보낼 때, 403 에러가 뜨면서 아래와 같이 리턴되는 경우가 있다.

```javascript
{
	"error": {
	"code": 403,
	"message": "Firebase Cloud Messaging API has not been used in project 42759??????? before or it is disabled. Enable it by visiting  https://console.developers.google.com/apis/api/fcm.googleapis.com/overview?project=42759??????? then retry. If you enabled this API recently, wait a few minutes for the action to propagate to our systems and retry.",
	"status": "PERMISSION_DENIED",
	"details": [
		{
			"@type": "type.googleapis.com/google.rpc.Help",
			"links": [
			{
				"description": "Google developers console API activation",
				"url": "https://console.developers.google.com/apis/api/fcm.googleapis.com/overview?project=42759???????"
			}
			]
		}
		]
	}
}
```

아마도 FCM이 구글과 통합되면서 Google Api 인증을 받아야만 하는 것으로 보인다.
Google API에서는 각 api별로 사용유무를 설정할 수 있는데, FCM은 기본값이 사용하지 않도록 되어 있어서 그런 것으로 보인다. 해당 링크로 들어가서 사용 버튼을 클릭해 주면 몇분 이내로 에러없이 보낼 수 있다.

> 필자는 계정을 여러개 쓰고 있는데, 해당 프로젝트를 못찾아서 한참을 헤메고 있었다. Firebase 계정을 로그인 된 상태에서 해당 Url을 클릭하면 프로젝트 설정화면이 나타난다.
> 이런 뻘짓좀 않했으면.. :sob:

## 참고
https://firebase.google.com/docs/cloud-messaging/?hl=ko



