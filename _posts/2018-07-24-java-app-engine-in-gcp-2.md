---
layout: post
title: 자바 어플리케이션을 gcp app engine으로 구성하기 2 - CloudSQL 연동
author: 하얀눈길
description: develop java application in app engine in gcp
tags: [app-engine, cloudSql, gcp, java]
featuredImage: 
img: Google-App.jpg
categories: [java, app engine, gcp, CloudSQL]
date: '2018-07-24'
extensions:
  preset: gfm

---

#### 관련 포스팅
1. [IntelliJ에 Google Cloud Tools 연동하기](http://www.irgroup.org/cloud-tools-for-intellij/)
2. [자바 어플리케이션을 gcp app engine으로 구성하기 1 - 예제 실행해 보기](http://www.irgroup.org/java-app-engine-in-gcp-1/)
3. [자바 어플리케이션을 gcp app engine으로 구성하기 2 - CloudSQL 연동](http://www.irgroup.org/java-app-engine-in-gcp-2/) - 이 문서


## 목표 
* 앱엔진에서 CloudSQL 연동하기

## Cloud SQL instance 세팅
1. Cloud SQL 인스턴스는 Second Generation으로 만들자
	[Create a Second Generation Cloud SQL instance](https://cloud.google.com/sql/docs/mysql/create-instance#create-2nd-gen)
	*이 포스팅은 sql instance가 이미 생성되어 있고, 데이터베이스, 테이블, 사용자 등이 모두 생성되어 있다는 가정하에 작성됐다.*
	
2. Cloud SDK를 사용하여 connection name 가져오기
	아래 명령을 사용하면 instance1에 대한 상세한 정보가 출력된다
	
	```
	$ gcloud sql instances describe instance1
	``` 
	
	출력되는 내용중에 `connectionName` 필드에 대한 값을 가져오면 된다.
	형태는 `project:zone:instance` 형식이다.

3. Connection Url
	Java7과 Java8의 connection url 구성이 다르다. 구글에서 java7 버전이   deprecate 됐기 때문에 이젠 java8로만 작성하자.
	
	**java 8 Conntection  URL 구성**
	```
	  jdbc:mysql://google/<DATABASE_NAME>?cloudSqlInstance=<INSTANCE_CONNECTION_NAME>&socketFactory=com.google.cloud.sql.mysql.SocketFactory&user=<MYSQL_USER_NAME>&password=<MYSQL_USER_PASSWORD>&useSSL=false
	```
	**Java 7 Connection URL 구성(deprecated)**
	```
	Deploy : jdbc:google:mysql://<INSTANCE_CONNECTION_NAME>/<DATABASE_NAME>?user=<MYSQL_USER_NAME>&password=<MYSQL_USER_PASSWORD>
	Local Testing : jdbc:mysql://google/<DATABASE_NAME>?cloudSqlInstance=<INSTANCE_CONNECTION_NAME>&socketFactory=com.google.cloud.sql.mysql.SocketFactory&user=<MYSQL_USER_NAME>&password=<MYSQL_USER_PASSWORD>&useSSL=false
	```
	*이하 Java7에 대한 설명은 배제한다*
	
4. POM에 환경변수 설정하기
	환경변수 설정은 gradle 프로젝트로 만들어도 가능할 것으로 보인다. 그러나 Maven으로 하는 것이 정신건강에 이로울 것이니, Cloud SQL연동하려면 무조건 Maven으로 하자.
	 
	 아래는 pom.xml의 일부 예시이다.
	 
	 ```XML
	  <properties>  
	  <!--  
    INSTANCE_CONNECTION_NAME from Cloud Console > SQL > Instance Details > Properties  
    or gcloud sql instances describe <instance>  
    project:region:instance for Cloud SQL 2nd Generation or  
    project:instance        for Cloud SQL 1st Generation  
    -->
	    <INSTANCE_CONNECTION_NAME>project:region:instance</INSTANCE_CONNECTION_NAME>
	    <user>root</user>  
	    <password>myPassword</password> 
	    <database>myDatabase</database>  
	    <!-- ...  -->
    </properties>
	 ```
	 
	 properties 태그에 필요한 변수들을 설정하자. 필드명은 원하는 것으로 바꿔도 상관 없다.
	 
5. POM에 플러그인 설정하기
	Cloud Tools에 대한 maven plugin 설정이 필요하다. 아래는 pom.xml의 snippet이다

	```XML
	<plugin>  
		<groupId>org.apache.maven.plugins</groupId>
		<artifactId>maven-war-plugin</artifactId>  
		<version>3.0.0</version>  
		<configuration>  
			<webResources>  
			<!-- in order to interpolate version from pom into appengine-web.xml -->  
				<resource>  
					<directory>${basedir}/src/main/webapp/WEB-INF</directory>  
					<filtering>true</filtering>  
					<targetPath>WEB-INF</targetPath>  
				</resource>  
			</webResources> 
		 </configuration>  
	 </plugin>  
	 
	 <plugin>  
		 <groupId>com.google.cloud.tools</groupId>  
		 <artifactId>appengine-maven-plugin</artifactId>  
		 <version>1.3.1</version>  
		 <configuration>  
			 <deploy.promote>true</deploy.promote>
			 <deploy.stopPreviousVersion>true</deploy.stopPreviousVersion>  
		</configuration>  
	</plugin>

	```
	
	
6.  POM에 JDBC 라이브러리 추가
	production 환경에서는 아래 의존성이 사실 필요 없다. 하지만, local에서 테스트를 위해서는 필요하니 미리 추가해 두자
	
	```XML
	<dependency>  <!-- Only used locally --> 
		<groupId>mysql</groupId>  
		<artifactId>mysql-connector-java</artifactId>  
		<version>5.1.42</version>  
	</dependency>  
	<dependency>  
		<groupId>com.google.cloud.sql</groupId>  
		<artifactId>mysql-socket-factory</artifactId>  
		<version>1.0.8</version>  
	</dependency>
	```
7.  만일 앱엔진 프로젝트와 Cloud SQL 프로젝트가 서로 다르다면 아래 문서를 참조하여 추가 설정해야 한다.
	https://cloud.google.com/appengine/docs/standard/java/cloud-sql/using-cloud-sql-mysql#granting-access
	
8. appengine-web.xml 설정
	pom.xml의 properties는 컴파일 시점에 참조가 될 것이다. 그러면, property의 내용을 프로그램에서 참조하는 곳은 어디인가? *(단순한 것인데 이부분을 이해 못해 한참 헤멨다. :sob:)*
	 바로 appengine-web.xml에서 pom.xml의 properties를 참조하여 시스템 변수를 생성해 줄 수 있다. 
	 이러한 흐름이 있기 때문에, local test 환경과 production 환경에 대한 세팅이 간결해지고 관리도 쉬워진다.

	**appengine-web.xml 예시**
	```XML
	  <appengine-web-app  xmlns="http://appengine.google.com/ns/1.0">  
		  <threadsafe>true</threadsafe>  
		  <runtime>java8</runtime>  
		  
		  <service>cloudsql</service>  
		  
		  <system-properties>  
			  <property  name="cloudsql"  
			  value="jdbc:mysql://google/${database}?useSSL=false&amp;cloudSqlInstance=${INSTANCE_CONNECTION_NAME}&amp;socketFactory=com.google.cloud.sql.mysql.SocketFactory&amp;user=${user}&amp;password=${password}"  
			  />  
		  </system-properties>  
	  </appengine-web-app>
	```
	system-properties는 구동되는 프로그램(앱엔진)의 시스템 환경변수로 등록된다. 그리고, 이 property에서 `${}`로 작성되는 변수들은 pom.xml의 property 변수명을 참조하게 된다.

9. 코드 샘플 
	```java
	  @SuppressWarnings("serial")  
	  // With @WebServlet annotation the webapp/WEB-INF/web.xml is no longer required.  
	  @WebServlet(name =  "CloudSQL", description =  "CloudSQL: Write timestamps of visitors to Cloud SQL", urlPatterns =  "/cloudsql")  
	  public  class  CloudSqlServlet  extends  HttpServlet  { 
	  Connection conn;  
	  @Override  public  void doGet(HttpServletRequest req,  HttpServletResponse resp)  throws  IOException,  ServletException  
	  {  
		  final  String createTableSql =  "CREATE TABLE IF NOT EXISTS visits ( "  
			  +  "visit_id SERIAL NOT NULL, ts timestamp NOT NULL, "  
			  +  "PRIMARY KEY (visit_id) );";  
		  final  String createVisitSql =  "INSERT INTO visits (ts) VALUES (?);";  
		  final  String selectSql =  "SELECT ts FROM visits ORDER BY ts DESC "  +  "LIMIT 10;";  
		  String path = req.getRequestURI();  
		  if  (path.startsWith("/favicon.ico"))  
		  {  
			  return;  // ignore the request for favicon.ico  
		  }  
			  PrintWriter out = resp.getWriter(); 	
			  resp.setContentType("text/plain");  
			  
			  Stopwatch stopwatch =  Stopwatch.createStarted();  
			  try  (PreparedStatement statementCreateVisit = conn.prepareStatement(createVisitSql))  { 
				  conn.createStatement().executeUpdate(createTableSql); 
				  statementCreateVisit.setTimestamp(1,  new  Timestamp(new  Date().getTime())); 
				  statementCreateVisit.executeUpdate(); 
				   
				  try  (ResultSet rs = conn.prepareStatement(selectSql).executeQuery())  { 
					  stopwatch.stop(); 
					  out.print("Last 10 visits:\n");  
					  while  (rs.next())  {  
						  String timeStamp = rs.getString("ts"); 
						  out.println("Visited at time: "  + timeStamp);  
					  }  
				  }  
			  }  catch  (SQLException e)  {  
				  throw  new  ServletException("SQL error", e);  
			  } 
			  out.println("Query time (ms):"  + stopwatch.elapsed(TimeUnit.MILLISECONDS));  
		  } 
	   @Override  public  void init()  throws  ServletException  {  
		   String url =  System.getProperty("cloudsql"); 
		   log("connecting to: "  + url);  
		   try  { 
			   conn =  DriverManager.getConnection(url);  
		   }  catch  (SQLException e)  {  
			   throw  new  ServletException("Unable to connect to Cloud SQL", e);  
		   }  
	   }  
	}
	```
	 
	 
10. 실행하기
	**개발환경에서 테스트하기**
	 
	```
	  mvn appengine:run
	```

	**Deploy 하기**

	```
	mvn appengine:deploy
	```
	
	[IntelliJ에서 Google Cloud Tool 연동하기](http://www.irgroup.org/cloud-tools-for-intellij/) 를 잘 따라 했다면, IDEA에서 바로 run을 실행시켜도 된다. 
	
### Test 및 deploy 환경 추가 설정
여기까지 잘 따라 왔다면 deploy하여 앱엔진에서 Cloud SQL로 잘 접속이 되는 것을 확인할 수 있을 것이다.
헌데, 문제가 있다. local testing환경에서 Cloud SQL로 직접 접속하여 테스트 할 수 없다는 것이다.
그래서, 아래와 같은 설정을 추가한다.

**pom.xml에 profiles 추가**

```xml
<profiles>  
  <profile> 
	 <id>local</id>  
	 <properties> 
		 <mysql_url><![CDATA[jdbc:mysql://your.ip:port/${database}?user=${user}&amp;password=${password}]]></mysql_url>  
	 </properties> 
  </profile> 
  <profile> 
	 <id>deploy</id>  
	 <properties> 
		 <mysql_url><![CDATA[jdbc:mysql://google/${database}?useSSL=false&amp;cloudSqlInstance=${INSTANCE_CONNECTION_NAME}&amp;socketFactory=com.google.cloud.sql.mysql.SocketFactory&amp;user=${user}&amp;password=${password}]]>
		 </mysql_url>  
	 </properties> 
   </profile>
 </profiles>
```

로컬에서는 Cloud SQL의 connection url이 먹히지 않는다. 일반적이 mysql connection url을 그대로 사용하면 된다. 당연히, cloud sql instance는 방화벽이 열려 있어야 할 것이다.

> 주의 : xml에서 xml을 참조하기 때문에 escape char 처리가 문제가 된다. 위 예시처럼 그냥 CDATA를 사용하면 편하다. 그래도, pom.xml에서 &와 같은 특수기호는 escape 처리를 해 줘야 한다.



**appengine-web.xml 수정**

```xml
<system-properties>  
	<property name="cloudsql" value="${mysql_url}" />
</system-properties>
```

이제 설정이 마무리 됐다. profile 기본설정을 local 로 해 두고 맘 편히 개발하자. 

끝!!

## 참조
* 구글 공식 문서 : https://cloud.google.com/appengine/docs/standard/java/


 




