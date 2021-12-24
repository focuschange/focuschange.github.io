---
title: jekyll에서 마크다운(Markdown) 사용하기
date: 2021-12-24 12:15:33 +0900
categories: [jekyll, markdown]
tags: [마크다운, markdown, 마크다운-문법, kramdown, chirpy]
description: jekyll, 마크다운, markdown, 마크다운-문법, kramdown, chirpy, 하얀눈길
toc: true
toc_sticky: true
toc_label: 목차
math: true
mermaid: true
image:
  src: https://upload.wikimedia.org/wikipedia/commons/thumb/4/48/Markdown-mark.svg/1200px-Markdown-mark.svg.png
  width: 1000   # in pixels
  height: 400   # in pixels
  alt: image alternative text
---

> 한번에 모두 정리하기에는 양이 너무 많고...\
> 자주 쓰는데 자꾸 잊어버리는 것만 우선 기록~ 😅

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

# 기본 사항
---
* 일반 텍스트를 마음대로 입력할 수 있다
* html tag를 마음대로 입력할 수 있다
* 처음 접하는 어려움이 줄바꿈이 안된다는 것이다.

# 줄바꿈
---
## 마크다운 어디서 사용할 수 있는 줄바꿈
* 형식 : `<BR>` 입력
* <font color=red>테이블내에 줄바꿈 할 때 유용하다</font>
* 예시
  ```
  첫번째<br>
  두번째
  세번째. 앞줄에 붙어서 나옴 
  ```
* 결과  
  ```
  첫번째  
  두번째. 세번째. 앞줄에 붙어서 나옴 
  ```

## 일반 텍스트에서 줄바꿈

* 형식 : 문장 끝에 공백 2개 이상 입력  
* 예시
  ```
  첫번째. 여기는 문장 끝에 공백 2개  
  두번째. 여기는 공백 없음
  세번째. 앞줄에 붙어서 나옴 
  ```
* 결과  
  ```
  첫번째. 여기는 공백 2개  
  두번째. 여기는 공백 없음 세번째. 앞줄에 붙어서 나옴 
  ```
  
## 블럭 인용구에서 줄바꿈

* 형식 : 인용구 끝에 `\` 입력. `\`뒤에는 아무것도 입력하지 않아야 한다\
* 예시
  ```
  > 첫번째 줄\
  > 두번째 줄\
  > 세번째 줄.
  > 네번째 줄. 앞줄과 붙음
  ```
* 결과
  > 첫번째 줄\
  > 두번째 줄\
  > 세번째 줄.
  > 네번째 줄. 앞줄과 붙음

# 블럭 인용구
---
* 형식 : `>`를 앞에 붙인다. 이중 인용구를 사용할 때는 `>>`를 입력한다
* 예시
  ```
  > 첫번째
  >> 두번째
  >>> 세번째
  > 네번째
  ```
* 결과
  > 첫번째
  >> 두번째
  >>> 세번째
  > 네번째


# 코드
---
## 인라인 코드
* 형식 : `` ` `` 사이에 내용 입력. `` ` `` 자체를 인라인 시키고 싶으면 두개를 입력하여 사용하면 escape 됨  
* 예시
  ```
  인라인 코드는 `하하` 이렇게~  
  인라인 코드 자체를 인라인 시키고자 할 때는 `` ` `` 이렇게 입력
  ```
* 결과

  인라인 코드는 `하하` 이렇게~  
  인라인 코드 자체를 인라인 시키고자 할 때는 `` ` `` 이렇게 입력

## 코드 블럭
* 형식 : `` ``` ``<사용할 언어>⏎코드⏎`` ``` ``
* 사용할 언어는... 음... 목록을 알아서..
* `` ` `` 대신에 ``~``를 사용할 수 있다 

* 예시
~~~
```java
  # 이렇게
  class a {
  	static string s = '하하';
  }
```
~~~

* 결과
```java
  # 이렇게
  class a {
    static string s = '하하';
  }
```

# 링크
---
* 기본 형식 : `[링크텍스트](링크)`
* 새창으로 열기 : `[링크텍스트](링크){:target="_blank"}`
* 참고 : `{}` 사이에 html attribute를 마음대로 넣을 수 있다
* 예시

  ```
    - 기본 링크 : [홈으로](https://www.irgroup.org/)
    - 새창으로 : [홈으로 새창](https://www.irgroup.org/){:target="_blank"}
  ```

* 결과
  - 기본 링크 : [홈으로](https://www.irgroup.org/)
  - 새창으로 : [홈으로 새창](https://www.irgroup.org/){:target="_blank"}

# 이미지
---
* 기본 형식 : `![설명](링크)`
* 테두리 만들기 : `![설명](링크){:style="border:1px solid #eaeaea; border-radius: 7px; padding: 0px;" }`
* 참고 : `{}` 사이에 html attribute를 마음대로 넣을 수 있다
* 예시

  ```
    ![깃헙 커밋 액션](/assets/img/github-commit-action.jpg){:style="border:1px solid #eaeaea; border-radius: 7px; padding: 0px;" }`
  ```
* 결과
  ![깃헙 커밋 액션](/assets/img/github-commit-action.jpg){:style="border:1px solid #eaeaea; border-radius: 7px; padding: 0px;" }



# 참조
* [kramdown syntax](https://kramdown.gettalong.org/syntax.html){:target="_blank"}
* [kramdown 사용법](http://gjchoi.github.io/env/Kramdown%28%EB%A7%88%ED%81%AC%EB%8B%A4%EC%9A%B4%29-%EC%82%AC%EC%9A%A9%EB%B2%95/){:target="_blank"}
* [Front Matter](https://jekyllrb.com/docs/front-matter/){:target="_blank"}

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
