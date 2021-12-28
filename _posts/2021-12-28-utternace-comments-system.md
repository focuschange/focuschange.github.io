---
title: Jekyll 테마에 utterances 댓글 연동하기
date: 2021-12-26 14:15:33 +0900
categories: [jekyll, chirpy]
tags: [jekyll, chirpy, comment, comments, utterances, 댓글, 댓글시스템, git, github, github기반 댓글, disqus]
description: jekyll, chirpy, comment, utterances, 댓글, 댓글시스템, git, github, github기반-댓글, 하얀눈길
toc: true
toc_sticky: true
toc_label: 목차
math: true
mermaid: true
image:
  src: /assets/img/utterance-comment.jpg
  width: 1000   # in pixels
  height: 400   # in pixels
  alt: image alternative text
---

> Chirpy는 [Disqus](https://disqus.com/){:target="_blank"}를 기본으로 지원합니다. 계정정보만 연동하면 잘 나옵니다.\
> 전에 Disqus를 사용했었는데, 무료로 쓰던 기능들이 유료화 하면서 커스터마이징에 어려움이 있습니다. 그리고, UI도 좀 그러네요..
> 그래서 다른 것을 알아보다가 [Utterances](https://utteranc.es/){:target="_blank"}를 알게 되었습니다.\
> github의 이슈를 댓글로 표시하는 것으로 무료이면서 설치도 단순하고 ui도 깔끔합니다.

> 관련글 : 
* [Jekyll Chirpy 테마 사용하여 블로그 만들기](https://www.irgroup.org/posts/jekyll-chirpy/){:target="_blank"}
* [Chirpy 테마 커스터마이징](https://www.irgroup.org/posts/Chirpy-%ED%85%8C%EB%A7%88-%EC%BB%A4%EC%8A%A4%ED%84%B0%EB%A7%88%EC%9D%B4%EC%A7%95/){:target="_blank"}


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

## 작동 원리
---
Utterances가 로드되면 github의 [이슈 검색 API](https://developer.github.com/v3/search/#search-issues){:target="_blank"}를 사용하여 url 또는 Pathname에 기반하여 페이지와 관련된 이슈를 찾습니다. 제목과 일치하는 페이지의 이슈를 찾을 수 없는 경우에도 큰 문제가 없습니다. [Utterance-bot](https://github.com/utterances-bot){:target="_blank"}이 누군가 처음 댓글을 달 때 자동으로 이슈를 생성합니다.

<br>
## 사전 준비
---
우선 github 계정이 필요합니다. Jekyll을 사용하니 당연히 있겠지요? 😅   
github 블로그가 아니라면 우선 github 계정을 만들고, 프로젝트를 하나 생성해 둡니다.  
저는 [test project](https://github.com/focuschange-test/focuschange-test.github.io){:target="_blank"}를 사용하겠습니다.  
저장소는 public으로 만들어야 합니다. 블로그용 저장소가 있을테니, 그 저장소를 그대로 사용해도 됩니다.  
우선, github에 로그인을 해 둡니다. 다음 단계에서 자동으로 나의 저장소를 Utterances가 찾아 줄 겁니다.  

<br>
## 내 저장소에 설치하기
---
[Utterances](https://utteranc.es/){:target="_blank"} 홈페이지의 중간에 보면, [utterances app](https://github.com/apps/utterances){:target="_blank"} 링크가 있습니다.  
이 안으로 들어가면 아래 그림처럼 나옵니다.  

![Utterances 앱 Install #1](/assets/img/GitHub_Apps_utterances.jpg){:style="border:1px solid #eaeaea; border-radius: 7px; padding: 0px;"}

오른쪽 위에 초록색 `Install` 버튼이 나타납니다.  
이미 설치 했다면, 회색으로 `Configuration` 버튼으로 변경되어 보일 겁니다.  

`Install` 버튼을 클릭합니다. 그러면 아래 그림처럼 나옵니다.   

![Utterances 앱 Install  #2](/assets/img/Installing_utterances.jpg){:style="border:1px solid #eaeaea; border-radius: 7px; padding: 0px;"}

여기에서 `Only select repositoryies`를 선택합니다.
그러면 미리 만들어 둔 나의 저장소가 아래에 뜨게 되고, 이것을 클릭하면 `Install` 버튼이 활성화 됩니다.   

![Utterances 앱 Install  #3](/assets/img/utterances_select_repo.jpg){:style="border:1px solid #eaeaea; border-radius: 7px; padding: 0px;"}

설치가 정상적으로 되고 나면 Utterances 홈페이지로 다시 돌아오게 됩니다.  

## Utterances Configuration
---
Utterances  홈페이지를 아래로 내려보면, 중간에 `Configuration` 섹션이 나옵니다.   
여기에 나의 저장소를 입력하는 부분이 있습니다. 입력 형식은 `[저장소ID]/[저장소명]` 입니다.   
저는 `focuschange-test/focuschange-test.github.io` 입력하였습니다.   

![Utterances configuration #1](/assets/img/utterances_configuration_1.jpg){:style="border:1px solid #eaeaea; border-radius: 7px; padding: 0px;"}

다른 몇몇 설정 값들이 있으나 별로 중요하지 않아서, 변경하지 않고 화면을 내려 보면, `Enable Utterances` 섹션이 나옵니다.   
여기에 자바스크립트 코드가 생성되어 있는데, 이를 복사합니다.
자바스크립트 2번째 줄에 `repo=...`이 설정되어 있는데, 위에서 입력한 저장소명이 그대로 나와 있으면 설정이 끝난 것입니다.  

![Utterances configuration #2](/assets/img/utterances_configuration_2.jpg){:style="border:1px solid #eaeaea; border-radius: 7px; padding: 0px;"}


## Chirpy에 스크립트 심기
---
Jekyll 기반 블로그는 아마도 아래 방법이 비슷할 것입니다.  
`_layouts/post.html` 파일의 끝에 위에서 복사한 스크립트를 그대로 넣어 줍니다. [여기](https://github.com/focuschange-test/focuschange-test.github.io/blob/master/_layouts/post.html){:target="_blank"}를 참고하세요~  

```html
<!-- _layouts/post.html 하단에 삽입 -->

...
{% raw %}
      {% endif %}
    </div>

    {% include post-sharing.html %}
{% endraw %}
  </div><!-- .post-tail-bottom -->

</div><!-- div.post-tail-wrapper -->

<!-- Utterances 댓글 스니핏 삽입 -->
<script src="https://utteranc.es/client.js"
        repo="focuschange-test/focuschange-test.github.io"
        issue-term="pathname"
        theme="github-light"
        crossorigin="anonymous"
        async>
</script>
```

<br>
## 결과 확인
---
자~ 이제 댓글 시스템이 잘 붙었는지 확인해 봅시다.   
로컬에서 `jekyll serve` 명령을 통해 확인해 보겠습니다.  
아래 처럼 잘 붙어서 나오네요.   

![Utterances configuration #3](/assets/img/utterances_configuration_3.jpg){:style="border:1px solid #eaeaea; border-radius: 7px; padding: 0px;"}

댓글을 쓰려면 github 로그인을 해야 합니다. `Sign In with GitHub` 버튼을 클릭해서 로그인 합시다.  
내용을 입력해 봤습니다. 잘 붙어나오네요. Preview도 지원하고, 마크다운도 쓸 수 있습니다.      
아래는 최종 결과입니다.

![Utterances configuration #4](/assets/img/utterances_configuration_4.jpg){:style="border:1px solid #eaeaea; border-radius: 7px; padding: 0px;"}

댓글이 어디에 저장되어 있는지 확인해 봅시다.   
github 저장소에 들어가 보면 상단 탭에 `Issues`가 있습니다. 이곳을 들어가 보면, 댓글이 달린 포스트 리스트가 보입니다.  

![Utterances configuration #5](/assets/img/utterances_configuration_5.jpg){:style="border:1px solid #eaeaea; border-radius: 7px; padding: 0px;"}


위에서 작성한 댓글이 `posts/welcome/`에 들어 있군요. 제 테스트 페이지가 welcome 이었습니다.  
이것을 클릭해 보면 아래와 같이 나옵니다.

![Utterances configuration #6](/assets/img/utterances_configuration_6.jpg){:style="border:1px solid #eaeaea; border-radius: 7px; padding: 0px;"}

저장된 댓글을 여기서 확인할 수 있습니다. 또한 댓글도 달 수 있습니다. 블로그내에서 다는 것이 더 편합니다만, 이곳에서 통합관리할 수 있으니 편하지요.

<br>
## 문제 해결
---
* 댓글 화면은 나오는데 저장이 되지 않습니다.  
  > 저의 경우는, Chirpy 설치 시 fork를 받아서 설치했었는데요. 이런게 하면 Issues 탭이 생기지 않더군요. 아예 zip 파일을 다운 받아서 설치하는 것이 더 편한 것 같습니다. 아니면, clone 받은 후 저장소를 삭제하고 다시 만들어서 소스를 올리세요. 그러면 Issues 탭이 생기면서 정상 동작합니다.


<br>
## 끝맺으며..
---
댓글 다는 시간은 10분 내외였습니다. 그런데 글을 쓰는 것은 하루 종일 걸리네요.   
그래도 좋은 댓글 시스템을 찾은 것 같아 좋네요~   
많이 활용하시기 바랍니다~



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