---
layout: post
title: curl을 사용하여 HTTP Request 시간 구하기
author: 하얀눈길
description: null
tags: [curl, http]
featuredImage: 
img: 
categories: http
date: '2018-07-04'
extensions:
  preset: gfm

---



> curl 로 타이밍 체크하는 방법을 모르는 분들이 꽤 많습니다. 물론 저도 몰랐구요
> 헌데 찾아보니 나오네요. 잊어버리기 전에 기록합니다.  :sweat_smile:


## curl을 사용하여 HTTP Request 시간 구하기

아래와 같은 내용을 파일로 저장합니다. 저는 curl-format.txt로 했습니다.


        time_namelookup:  %{time_namelookup}\n
           time_connect:  %{time_connect}\n
        time_appconnect:  %{time_appconnect}\n
       time_pretransfer:  %{time_pretransfer}\n
          time_redirect:  %{time_redirect}\n
     time_starttransfer:  %{time_starttransfer}\n
                        ----------\n
             time_total:  %{time_total}\n

명령어를 다음과 같이 입력합니다.

    curl -w "@curl-format.txt" --compress -o /dev/null -s http://www.naver.com

결과는 다음과 같네요

        time_namelookup:  0.012691
           time_connect:  0.015969
        time_appconnect:  0.000000
       time_pretransfer:  0.016212
          time_redirect:  0.000000
     time_starttransfer:  0.018516
                        ----------
             time_total:  0.018667

단위는 밀리세컨드입니다. 네이버 정말 빠르네요.. :scream:

시간상 자세한 내용은 아래 참고를 참고~

## 참고
- https://overloaded.io/timing-http-requests-curl
- https://curl.haxx.se/docs/manual.html



