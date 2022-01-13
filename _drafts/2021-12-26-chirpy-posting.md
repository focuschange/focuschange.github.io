---
title: chirpy 테마 포스팅하기
date: 2021-12-24 12:15:33 +0900
categories: jekyll chirpy
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


## liquid 문법은 소스블럭에 표시하기
{% assign endmark = "{" % endraw % }" %}
소스블럭 내에서 `{{ "%7B%25+raw+%25%7D" | url_decode}}` `{{ "%7B%25+endraw+%25%7D" | url_decode}}`를 사용한다.    

```liquid
...
{% raw %}
{% include lang.html %}
{% endraw %}
{{ "%7B%25+raw+%25%7D" | url_decode}}
{% raw %}
{% if page.image.src %}
  {% capture bg %}
    {% unless page.image.no_bg %}{{ 'bg' }}{% endunless %}
  {% endcapture %}
{% endif %}
{% endraw %}
{{ "%7B%25+endraw+%25%7D" | url_decode}}

...

```

# 문제 해결
* 포스팅이 중복되어 목록에 나타난다.
  > front matter에서 date가 동일한 포스트가 있으면 전체 목록보기 화면에서 제목이 중복되어 나타날 수 있다. 실제 포스팅 내용은 원래 문서와 같다.\
  > front matter의 타이틀을 복붙했는지 확인해 보자\
  > 


