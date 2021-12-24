---
title: 마크다운(Markdown) 문법 테스트 페이지
date: 2021-12-20 12:15:33 +0900
categories: [jekyll, markdown]
tags: [마크다운, markdown, 마크다운-문법]
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

# 기본 문단

테스트라네~~~~  

## 기본 문단에서 줄바꿈

줄바꿈은 마지막에 공백 2개 이거는 공백 3개   
줄바꿈 이거는 공백 2개  
줄바꿈 이거는 공백 많이          
테이블안에서 줄바꿈은 `<br>`을 입력하면 됨    


## 블럭 인용구에서 줄바꿈

> afasdfa\
> a sdfasfdaf\
> bbbb 

## 코드
인라인 코드는 `하하` 이렇게~
블럭 코드는..

```java
# 이렇게
class a {
	static string s = '하하';
}
```

## 링크
> 기본 형식 : ``[링크텍스트](링크)``\
> 새창으로 열기 : `[링크텍스트](링크){:target="_blank"}`\
> 참고 : `{}` 사이에 html attribute를 마음대로 넣을 수 있다\
> 예시 : `[favicon 만들기](https://www.favicon-generator.org/){:target="_blank"}`\
> 결과 : [favicon 만들기](https://www.favicon-generator.org/){:target="_blank"}

## 이미지
> 기본 형식 : `![설명](링크)`\
> 테두리 만들기 : `![설명](링크){:style="border:1px solid #eaeaea; border-radius: 7px; padding: 0px;" }`\
> 참고 : `{}` 사이에 html attribute를 마음대로 넣을 수 있다\
> 예시 : `![깃헙 커밋 액션](/assets/img/github-commit-action.jpg){:style="border:1px solid #eaeaea; border-radius: 7px; padding: 0px;" }`\
> 결과 : ![깃헙 커밋 액션](/assets/img/github-commit-action.jpg){:style="border:1px solid #eaeaea; border-radius: 7px; padding: 0px;" }





