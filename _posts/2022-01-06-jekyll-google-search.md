---
title: Jekyll 테마에서 구글 검색 노출 등록하기
date: 2022-01-13 15:11:33 +0900
categories: [jekyll]
tags: [jekyll, chirpy, google, google search, search engine, 검색노출, 구글검색, sitemap.xml, sitemap, 사이트맵]
description: jekyll, chirpy, google, google search, search engine, 검색노출, 구글검색, sitemap.xml, sitemap, 사이트맵
toc: true
math: true
mermaid: true
image:
  src: /assets/img/Google-search.jpg
  width: 1000   # in pixels
  height: 400   # in pixels
  alt: 구글 검색엔진
---


> 구글에 검색을 통해 사이트로 인입되는 비율은 엄청납니다. \
> 우리나라는 naver의 영향력이 막강하지만, 외부의 블로그나 사이트들은 그래도 구글을 통한 인입이 강력한 편입니다. \
> jekyll 테마에서 구글 검색엔진에 포스트 글들이 검색되도록 설정해 보겠습니다.\
> chirpy 기준이기는 하나 기본이 같기 때문에 쉽게 적용할 수 있을 겁니다.

> [참고한 사이트](https://imweb.me/faq?mode=view&category=29&category2=35&idx=15573){:target="_blank"}가 너무 잘 정리되어 있으나, 끝부분이 저와 환경이 달라 재정리 합니다.\
> 앞부분은 거의 똑같습니다.(카피라고 뭐라한다면... 음.. 내리겠습니다... 😭)

<!-- 상단 광고 -->
<br>
<div class="card">
<script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js?client=ca-pub-8993100314477491"
     crossorigin="anonymous"></script>
<ins class="adsbygoogle"
     style="display:block; text-align:center;"
     data-ad-layout="in-article"
     data-ad-format="fluid"
     data-ad-client="ca-pub-8993100314477491"
     data-ad-slot="6115278830"></ins>
<script>
     (adsbygoogle = window.adsbygoogle || []).push({});
</script>
</div>
<br>

<!-- start post -->
# 준비하기
---
우선 구글 계정이 필요합니다. 없다면 아래 링크를 통해 구글 계정을 새로 만듭니다.
> [구글 계정](https://accounts.google.com/){:target="_blank"}


# 구글 웹마스터 도구 등록하기
---
## 시작하기

[구글 웹마스터 도구(Search Console)](https://accounts.google.com/){:target="_blank"}에 접속합니다. 그러면 아래와 같이 화면이 나타나고, `시작하기` 버튼을 클릭합니다.

<br>
![구글검색 등록하기 #1](/assets/img/jekyll_google_search_1.jpg){:style="border:1px solid #eaeaea; border-radius: 7px; padding: 0px;" }
<br>
<br>

## URL 입력하기

속성유형 선택하는 화면에서 URL접두어를 사용하여 사이트 주소를 입력하고 `계속` 버튼을 클릭합니다. 

<br>
![구글검색 등록하기 #2](/assets/img/jekyll_google_search_2.jpg){:style="border:1px solid #eaeaea; border-radius: 7px; padding: 0px;" }
<br>

## 다른 확인 방법 선택하기

소유권 확인하기 위한 창이 뜨게 되면 `다른 확인 방법`에서 `HTML 태그` 항목을 선택합니다. jekyll에서 header부분을 세팅할 수 있기 때문에 이 방법이 유용하네요.

<br>
![구글검색 등록하기 #3](/assets/img/jekyll_google_search_3.jpg){:style="border:1px solid #eaeaea; border-radius: 7px; padding: 0px;" }
<br>

## 메타태그 복사하기

메타태그를 `복사`를 합니다. 아직 `확인`을 누르지 마세요. 지금 단계여서 확인을 하게 되면 `소유권 확인 실패` 경고창이 뜨게 될 것입니다.   

<br>
![구글검색 등록하기 #4](/assets/img/jekyll_google_search_4.jpg){:style="border:1px solid #eaeaea; border-radius: 7px; padding: 0px;" }
<br>

복사가 됐으면 브라우저 창을 닫지 말고 내용을 먼저 확인합니다.   
복사한 내용을 텍스트 편집기에서 붙여넣어 보면 다음과 같은 형태로 나옵니다.

```html
<meta name="google-site-verification" content="1gDBG.........................................._stqA" />
```

여기서 content 부분이 소유권 확인용 코드입니다.   
보통 jekyll 블로그에서는 header 부분에 내용 전체를 넣어두면 되는데요. Chirpy에서는 `_config.yml`에 설정하는 항목이 있습니다.  


## `_config.yml`에 등록하기 
`_config.yml`에 `google_site_verification` 항목에 위에서 복사한 메타태그에서 content 부분을 넣어 줍니다.
```yml
...
google_site_verification: 1gDBG.........................................._stqA
...
``` 

git push를 한 후 deploy가 끝날 때까지 잠시 기다립니다.

## 소유권 확인하기 
다시 메타태그를 복사했던 브라우저 창으로 돌아와서 `확인` 버튼을 클릭합니다.   
구글에서 인증을 시도하게 되고, 아래와 같은 창이 뜨게 되면 성공한 것입니다.   

<br>
![구글검색 등록하기 #5](/assets/img/jekyll_google_search_5.jpg){:style="border:1px solid #eaeaea; border-radius: 7px; padding: 0px;" }
<br>


# sitemap.xml 등록하기 
블로그가 만들어지면 github이 자동으로 sitemap.xml을 만들어 줍니다. 그러니 다른 작업은 필요없습니다.   
그러나 수정을 하고 싶다면, [플러그인 없이 Jekyll Sitemap 만들기](http://dveamer.github.io/homepage/Sitemap.html){:target="_blank"}를 참고해서 sitemap.xml 파일을 만들어 주면 됩니다.  
기본 구성요소는 [sitemap xml 형식](http://superkts.pe.kr/upload/helper/file1/siteMapXML.htm){:target="_blank"}을 참조하면 되겠네요.  
준비가 되면 Google Search Console > Sitemaps > 새 사이트맵 추가 부분에 `sitemap.xml`을 입력하고 `제출`버튼을 클릭합니다.

<br>
![구글검색 등록하기 #7](/assets/img/jekyll_google_search_7.jpg){:style="border:1px solid #eaeaea; border-radius: 7px; padding: 0px;" }
<br>

아래와 같이 나오게 되면 정상적으로 등록 완료된 것입니다.

<br>
![구글검색 등록하기 #8](/assets/img/jekyll_google_search_8.jpg){:style="border:1px solid #eaeaea; border-radius: 7px; padding: 0px;" }
<br>

# 문제 해결
* 구글 Search Console에서 sitemap.xml을 가져오지 못할 때
  <br>
  ![구글검색 등록하기 #6](/assets/img/jekyll_google_search_6.jpg){:style="border:1px solid #eaeaea; border-radius: 7px; padding: 0px;" }
  <br>

  > 미안하지만, 버그입니다. 아직도 수정이 안된 것 같습니다. 다른 이유는 모르겠고, 해결이 안됩니다. 😱\
  > 참고 : [Sitemap could not be read in new GSC](https://support.google.com/webmasters/thread/3105916/sitemap-could-not-be-read-in-new-gsc?hl=en){:target="_blank"}


# 참조
---
* [Jekyll sitemap plugin](https://github.com/jekyll/jekyll-sitemap){:target="_blank"}
* [구글 사이트 등록하기(Google 검색 노출)](https://imweb.me/faq?mode=view&category=29&category2=35&idx=15573){:target="_blank"}
* [구글 Search Console](https://search.google.com/search-console){:target="_blank"}
* [sitemap xml 형식](http://superkts.pe.kr/upload/helper/file1/siteMapXML.htm){:target="_blank"}
* [플러그인 없이 Jekyll Sitemap 만들기](http://dveamer.github.io/homepage/Sitemap.html){:target="_blank"}


<!-- end post -->

<!-- 상단 광고 -->
<br>
<div class="card">
<script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js?client=ca-pub-8993100314477491"
     crossorigin="anonymous"></script>
<!-- 디스플레이광고-수평형 -->
<ins class="adsbygoogle"
     style="display:block"
     data-ad-client="ca-pub-8993100314477491"
     data-ad-slot="9549119208"
     data-ad-format="auto"
     data-full-width-responsive="true"></ins>
<script>
     (adsbygoogle = window.adsbygoogle || []).push({});
</script>
</div>
<br>