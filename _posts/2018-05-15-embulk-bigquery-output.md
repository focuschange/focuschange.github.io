---
layout: post
title: embulk를 사용한 vertica to bigquery 방법
date: '2018-05-11 10:59:10 +0900'
description: 'embulk 설치부터 vertica input, bigquery out 방법 및 트러블 슈팅 기록'
img: embulk-architecture.png
tags: [embulk,bigquery,vertica,etl]

---



> 다른 플랫폼간에 데이터 전송이나 ETL을 위해서 여러가지 툴이 존재한다고 합니다. 
> 그 중 embulk를 소개받게 되어 사용성 테스트를 해 봤습니다. 
> 물론 많은 툴을 써본 경험은 없으나, embulk는 물건인 듯 싶습니다. 
> 설치부터 데이터 전송까지 단계적으로 기록해 보려 합니다.

``` macOS High Sierra 기준입니다. ```


# 1. embulk 설치하고 실행하기
embulk를 설치하는 것은 아주 간단합니다.
아래와 같이 입력하면 설치가 끝납니다.

    curl --create-dirs -o ~/.embulk/bin/embulk -L "https://dl.embulk.org/embulk-latest.jar"
    chmod +x ~/.embulk/bin/embulk
    echo 'export PATH="$HOME/.embulk/bin:$PATH"' >> ~/.bash_profile
    
# 2. embulk 예제 실행
"embulk example" 명령은 샘플용 csv 파일을 생성해 줍니다

    embulk example ./try1
    embulk guess   ./try1/seed.yml -o config.yml
    embulk preview config.yml
    embulk run     config.yml

위 내용에서 "embulk example ./try1"을 실행하면 샘플용 데이터를 ./try1으로 생성해 줍니다.
csv파일로 만들어주지만, 실제는 gz으로 압축되어 있습니다.
두번째 줄은 seed.yml파일을 사용하여 config.yml을 만들어 줍니다.
embulk를 사용하면서 좋았던 것이 guess 명령입니다.
간단한 것은 guess명령으로 config.yml을 생성해 주니 생각보다 편리합니다.
그러나, 항상 잘 만들어주지 못합니다.(아니, 거의 못만듭니다. ㅠㅠ)

preview는 말 그대로 preview이고, run 명령을 통해 config.yml의 설정들을 실행 시킵니다.

# 3. embulk 플러그인 설치하기
example은 말그대로 예제일 뿐이고, 우리가 해 볼 것은 버티카에서 빅쿼리로 데이터를 전송하는 것이기 때문에, 해당하는 플러그인을 설치해야 합니다.
데이터를 뽑아낼 플랫폼은 input이고, 데이터를 출력할 플랫폼은 output입니다. 
다분히 embulk입장에서 input과 output을 정의한 것이고, 플랫폼들이 다 다르기 때문에, 플러그인으로 해당하는 in/out에 맞게 설치해 줘야 합니다.
input/output 플러그인 목록은 아래 url로 확인할 수 있습니다.
[embulk 플러그인 목록](http://www.embulk.org/plugins/)

다음 명령을 통해 플러그인을 간단히 설치할 수 있습니다.

    기본 명령 : embulk gem install <name>
    
input/output에 따라 name 값을 "embulk-input-command" 또는 "embulk-output-command"로 입력합니다.
"embulk gem" 명령은 ruby gem 명령을 wrapping한 것으로 보입니다. 
    
embulk 플러그인을 찾으려면 아래 명령으로 가능합니다.

    embulk gem search -rd embulk-output
 
설치된 플러그인 리스트를 보려면 아래 처럼 입력합니다.

    embulk gem list

embulk upgrade는 아래 명령으로 가능합니다.

    embulk selfupdate - 최신 버전으로 업그래이드
    embulk selfupdate x.y.z - x.y.z 버전으로 업그래이드.

가끔 embulk 버전과 플러그인 버전을 맞춰야 하는 경우가 있습니다.
이런 경우 위의 명령이 도움이 됩니다.

# 4. BigQuery output 플러그인 사용법
bigquery output plugin 설치는 아래와 같습니다.

    embulk gem install embulk-output-bigquery
    
bigquery를 사용하려면 당연한 얘기겠지만, 인증을 받아야 합니다.
GCP 서비스계정을 새로 만들고 Json 형태로 받습니다.(p12형태도 가능하지만 스킵하겠습니다)
gcp의 iam에서 서비스 계정을 만들 때, 키 유형을 반드시 json으로 선택합니다.
이후, 생성된 json을 로컬에 저장합니다.

기본적인 config는 아래와 같습니다.

    in:
      type: file
      path_prefix: /embulk/try1/csv/sample_
      decoders:
      - {type: gzip}
      parser:
        charset: UTF-8
        newline: LF
        type: csv
        delimiter: ','
        quote: '"'
        escape: '"'
        null_string: 'NULL'
        trim_if_not_quoted: false
        skip_header_lines: 1
        allow_extra_columns: false
        allow_optional_columns: false
      columns:
      - {name: id, type: long}
      - {name: account, type: long}
      - {name: time, type: timestamp, format: '%Y-%m-%d %H:%M:%S'}
      - {name: purchase, type: timestamp, format: '%Y%m%d'}
      - {name: comment, type: string}
    out:
    type: bigquery
    mode: append
    auth_method: json_key
    json_keyfile: /embulk/key/your-keyfile-name.json
    project: your-project-id
    dataset: tmp
    table: embulk_test
    auto_create_dataset: true
    auto_create_table: true
    column_options:
    - {name: id, type: INTEGER}
    - {name: account, type: FLOAT}
    - {name: time, type: STRING, timestamp_format: '%Y-%m-%d %H:%M:%S', timezone: "Asia/Tokyo"}

input으로는 샘플로 생성한 csv파일을 사용하였습니다.
output으로 bigquery로 직접 입력하도록 하였는데, 데이터셋이나 테이블 자동생성 옵션을 두었는데, 잘 되는 군요


# 5. vertica input 플러그인 사용법
vertica input plugin 설치명령은 아래와 같습니다. 

    embulk gem install embulk-input-vertica
    
기본적인 config는 다음과 같습니다.

    in:
    type: vertica
    host: 35.194.97.115
    user: dbadmin
    password: dbadmin
    database: vdb
    query: |
      SELECT
      employee_key
      employee_gender,
      courtesy_title,
      employee_first_name,
      employee_middle_initial,
      employee_last_name,
      employee_age,
      hire_date,
      employee_street_address,
      employee_city,
      employee_state,
      employee_region,
      job_title,
      reports_to,
      salaried_flag,
      annual_salary,
      hourly_rate,
      vacation_days
      from employee_dimension
    column_options:
      hire_date: {type: string, value_type: string, timestamp_format: "%Y-%m-%d", timezone: "Asia/Tokyo"}
    out:
      type: stdout

config.yml 구성 시 주의할 것은 date, timestamp type에 대한 형 변환입니다. 
column_options에 해당하는 필드를 정의해 주면 되구요.
아래와 같이 timestamp를 변형해 주니 bigquery에서도 동일한 type으로 잘 들어가는 군요

    reg_dt: {value_type: string, timestamp_format: "%Y-%m-%d %H:%M%S", timezone: "+0900"}

주의할 점은 vertica의 date, timestamp는 value_type을 반드시 string 타입으로 해 주어야 합니다.
embulk가 내부에서 string으로 처리하는 것 같습니다.

timestamp type은 ruby의 strftime format을 따릅니다. (참고 : https://docs.ruby-lang.org/en/2.4.0/Date.html#method-i-strftime)

# Troubleshooting 
**embulk-output-bigquery를 0.4.6에서 0.4.7로 업그래이드 할 때**

config.yml에 location 필드가 추가되야 합니다.(옵션이라면서 잘 안됩니다)
google-api-client도 0.20.1로 업그래이드 되야 합니다.

설치 방법

    embulk gem install google-api-client


**vertica에서 bigquery로 직접 전송할 때 csv 포맷인식이 잘 안되는 경우**

특별한 옵션을 주지 않는 한 중간 변환 및 전송을 위해서 embulk는 csv 파일을 사용합니다. 
보통 /tmp 디렉토리 아래에 임시파일을 생성하는데요. 
csv파일을 아무리 잘 만들어도 전송 시 오류가 발생하는 경우가 있습니다.
이때는, 다음 옵션을 사용합니다. 물론 embulk-output-bigquery config에 해당합니다.

    source_format: NEWLINE_DELIMITED_JSON
  
 bigquery에서는 그냥 필수로 사용하는 것이 좋습니다.
 헌데, csv에 비해 크기가 10배 가까이 늘어납니다. 
 그래서, 압축옵션을 다음과 같이 넣어 주는 것이 좋습니다.
 
    compression: GZIP

> 압축옵션 사용하지 않았다가 네트워크 대역폭 다 잡아먹었다고 욕먹었습니다. ㅠ_ㅠ

**input data가 커서 timeout이 발생하는 경우**

용량이 너무 크다보니 다음과 같은 에러가 발생하는 군요

    org.jruby.exceptions.RaiseException: (TransmissionError) execution expired

이럴 경우는 아래 옵션들을 추가합니다.

    open_timeout_sec : 7200
	send_timeout_sec : 7200
	read_timeout_sec : 7200

**vertica select 절 구분자가 빠졌을 경우**

이건 제가 vertica를 몰라서 생긴 문제였는데요. 
select 절에 구분자가 빠지면 word가 exact match되는 필드를 그냥 가져옵니다.(정확한 표현은 아닙니다)
이유는,  mysql의 'key as name" 이 버티카에서는 "key name"으로 사용됩니다.
콤마(,)가 빠져서 bigquery에 필드가 하나 없어진 원인을 찾는데 한참 삽질했네요.


# 참조

 - embulk plugins referrence : http://www.embulk.org/plugins/
 - embulk bult-in configuration : http://www.embulk.org/docs/built-in.html
 - bigquery output plugin : https://github.com/embulk/embulk-output-bigquery
 - vertica input plugin : https://github.com/sonots/embulk-input-vertica
 - https://medium.com/@jwlee98/embulk-%EC%9D%B4%EC%9A%A9%ED%95%B4%EC%84%9C-oracle-db-%EC%97%90%EC%84%9C-bigquery-%EB%A1%9C-%EB%8D%B0%EC%9D%B4%ED%84%B0-%EB%A7%88%EC%9D%B4%EA%B7%B8%EB%A0%88%EC%9D%B4%EC%85%98-%EC%82%BD%EC%A7%88%EA%B8%B0-141dc1d62b73
 - config 변수는 liquid template engine을 사용 : https://shopify.github.io/liquid/


