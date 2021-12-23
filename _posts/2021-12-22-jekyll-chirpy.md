---
layout: post
title: Jekyll Chirpy 테마 사용하여 블로그 만들기
author: 하얀눈길
description: jekyll로 구성되었던 블로그를 요즘 인기 있는 chirpy 테마로 변경하면서 알게된 내용들을 기록함
featuredImage: null
img: null
tags: jekyll chirpy github-blog github blog
categories: jekyll chirpy
date: '2021-12-22 11:17:00 +0900'

---

> 오래전 도메인을 살려두기 위해 jekyll로 블로그를 만들어 두었었는데, 항상 UI가 엉성해서 맘에 안들었었습니다.\
> 근래에 chirpy 테마를 알게되서 적용해 보니, 예전보다 만드는 방법이 많이 간결해 지고 테마도 상당히 이쁘네요.(나중에 테마 순위 정리나 한번...)\
> 기본적인 세팅부터 간단한 커스터마이징까지 정리해 보려합니다.(다른 분들이 너무 잘 정리해 놨지만, 내 환경에 대한 내용을 추가하기 위해 정리합니다)\
> 본 글은 github 계정 생성이나 다른 부가적인 내용들은 기록하지 않습니다.\
> 오직 chirpy 테마로 블로그 만드는 것에만 집중하여 정리합니다.\
> MacOS 기준으로 작성합니다

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


# 로컬 환경 설정 하기
---
ruby가 필수이기 때문에 아래 문서를 참고해서 ruby를 설치합니다.   
나중에 로컬에서 jekyll을 실행시켜서 환경 설정이나 포스팅 결과를 미리 확인하기 위한 용도입니다.   

* 참고 : <https://jekyllrb.com/docs/installation/>{:target="_blank"}
```bash
brew install ruby
```


# Chirpy 테마 설치하기
---
chirpy 테마는 아래 두가지 방법으로 설치할 수 있습니다.
1. [chirpy starter](https://github.com/cotes2020/chirpy-starter/generate){:target="_blank"}를 통해 직접 설치  
  아주 단순합니다. 처음에 이렇게 쉽게 만들 수 있단 말인가 싶을 정도로 금방 설치가 되었습니다.  
  그러나, 커스터마이징 할 때 문제가 계속 발생합니다. 물론 모두 해결할 수 있지만 오히려 귀찮습니다. 

1. [github](https://github.com/cotes2020/jekyll-theme-chirpy){:target="_blank"}에서 소스를 fork 받아서 만들기  
  몇 가지 커스터마이징을 해 주어야 합니다. Jekyll를 조금이라도 사용해 본 적이 있는 분들에게 좀 더 유용할 듯 싶습니다.  
  그런데, 블로그를 커스터마이징 하겠다면 이 방법을 사용하는 것이 좋습니다. 원하는 대로 쉽게 커스터마이징을 할 수 있습니다.  

본 글에서는 2번째 방법을 사용하여 설치하는 법을 정리합니다. ui 커스터마이징이나 google analytics, adsense 등을 붙이기 위해서는 두번째 방법으로 설치하면 훨씬 편하게 작업할 수 있습니다.  

아래 단계를 따라가면서 설치해 보겠습니다.  


## Chirpy 테마 fork
[Fork Chirpy](https://github.com/cotes2020/jekyll-theme-chirpy/fork){:target="_blank"} 를 사용하여 소스를 내가 생성한 저장소로 fork 받습니다.  
화면에서 나의 계정을 선택하면 바로 fork 되어 아래 화면 처럼 jekyll-theme-chirpy 저장소가 자동 생성됩니다.  

![fork 결과](/assets/img/jekyll-theme-chirpy-fork-result.jpg)

## Repository 이름 바꾸기
개인 도메인이 있다면 Repository명을 임의로 줘도 상관 없으나, github의 기본 도메인을 사용할 때는 `<github 아이디>.github.io` 형식으로 만들어야 합니다.  
이름을 잘못 정했다고 너무 걱정하지 않아도 됩니다. Repository의 Settings에서 언제든 바꿀 수 있습니다. 
저는 개인 도메인이 있으나 테스트로 github 도메인을 사용하기 위해 `focuschange-test.github.io`로 저장소를 만들겠습니다.  
나중에 설치가 끝나고 나면 `https://focuschange-test.github.io`로 블로그에 접속할 수 있을 겁니다.  

  * github > fork 받은 저장소 > Settings 화면으로 이동
  * Repository name을 `<github 아이디>.github.io` 형식으로 변경
  * rename 버튼 클릭 
  * 저장소 Code 화면으로 돌아오게 되며, 상단에 보면 방금 바뀐 이름으로 저장소 명이 보이게 될 겁니다

## 소스 클론 받기
아래 명령을 통해 소스를 받습니다.

```bash
git clone git@github.com-kim:focuschange-test/focuschange-test.github.io.git
```
> git에 대한 기초 지식이 없으신 분은 우선 [여기](https://evan-moon.github.io/2019/07/25/git-tutorial/){:target="_blank"}를 참고하세요~ 

저는 멀티계정을 사용하고 있기 때문에 clone url이 좀 다릅니다만, 기본적인 clone 방식으로 받으시면 됩니다.  
정상적으로 로컬로 소스를 받으셨다면, 기본적인 준비가 끝났습니다.

## chirpy 초기화 하기
소스 홈에서 아래 명령을 사용하여 chirpy를 초기화 합니다.

```bash
$ tools/init.sh
[INFO] Initialization successful!  <-- 이런 메세지가 나오면 성공입니다.
```

위의 명령을 수행하면 다음 파일들이 삭제됩니다.
  * .travis.yml
  * _posts 폴더 하위의 파일들
  * docs 폴더

fork 받았기 때문에 chirpy 자체 개발을 위한 파일들을 미리 삭제해 주는 것으로 보입니다.

## 로컬에서 실행해 보기
우선 jekyll을 로컬에서 실행시키기 위해 터미널에서 아래 명령을 사용하여 의존성이 있는 모듈을 모두 설치합니다.  
이미 chirpy에 기본설정이 되어 있기 때문에 아래 명령만으로 필요한 모든 것이 설치 됩니다.

```bash
$ bundle
```

많은 내용들이 설치되는 것을 볼 수 있습니다.

터미널에서 다음 명령을 사용하여 jekyll을 실행시킵니다.

```bash
$ jekyll serve
```

정상적으로 수행됐다면 아래와 같이 출력됩니다.
![jekyll serve 결과](/assets/img/jekyll_serve_running.jpg)

내용에서 보시면, `Server address: http://127.0.0.1:4000/` 라는 것이 있군요.  
브라우저를 열어서 해당 주소를 입력해 봅시다.  
그러면 아래와 같이 기본 블로그 화면이 나타납니다.
![chirpy 최초 실행](/assets/img/first-meet-chirpy.jpg)


위 화면이 잘 나오면 로컬에서 테스트는 성공입니다.

### 문제 해결
위와 같이 진행했을 때 몇가지 문제가 발생할 수 있습니다. 제가 경험했던 것을 기록해 봅니다.  
  1. `command not found: jekyll`   
     - 로컬 환경에 jekyll이 설치되지 않아서 그렇습니다. 
     - [맥OS에 Jekyll 설치](https://jekyllrb-ko.github.io/docs/installation/macos/){:target="_blank"} 여기를 참조하여 Jekyll을 설치해 줍니다.
  1. `jekyll serve` 명령을 실행하면 오류 메세지가 나옵니다.    
     - 여러가지 오류가 발생할 수 있습니다.   
     - 우선, Gemfile.lock을 삭제한 후 `bundle` 명령을 다시 실행해 보세요. 그래도 안된다면.. 구글링 😅
     - 간혹 퍼미션 오류를 내면서 실행이 안되는 경우가 있습니다. 그런 경우는 해당하는 디렉토리와 파일을 모두 권한 변경을 해 주세요.   
     - 제 경우는 `chmod -R 0644 *` 이렇게 처리해 버렸습니다. 😱 (다른 오류를 만들 수 있습니다..)
  1. _config.yml을 수정했는데 반영되지 않습니다.
     - `jekyll serve`로 실행해 둔 것을 `⌃` + `C` 키를 사용하여 중지 시킨 후 재 시작하면 됩니다



# Chirpy 환경 설정
---
## 소스 둘러보기
소스 목록을 보면 아래와 같이 보일 겁니다.
![chirpy 최초 실행](/assets/img/chirpy-file-list.jpg)

사실 모든 것을 알 필요는 없지만, 블로그 환경 설정과 커스터마이징을 위해 몇가지는 알아둘 필요가 있습니다.  
본 글에서는 환경 세팅만 할 예정이기에 깊게 살펴보지는 않겠습니다만, 몇몇개 내용을 기억해 둡시다.

* _config.yml : 블로그의 기본 설정 파일입니다. 기본 환경세팅은 모두 여기에서 합니다.  
* _data : 왼쪽 사이드바와 포스트 하단의 공유하기 버튼등의 구성을 변경할 수 있습니다. 언어 설정에 따라 기본적으로 화면에 나오는 단어들을 변경할 수 있습니다.
* _include : 사이드바, toc, 구글애널리틱스, footer, 댓글 등의 대부분의 모듈형으로 삽입되는 UI를 변경할 수 있습니다
* _layout : 블로그 전역에 적용되는 기본 형식, 카테고리, 포스트 등에 적용되는 형식등을 변경할 수 있습니다.
* _posts : 내가 작성한 블로그 글을 저장해 두는 곳입니다.
* _sass : css 파일을 커스터마이징 할 수 있습니다.
* _site : 로컬에서 실행할 때, 화면 UI를 구성하는 모든 내용이 들어 있습니다. 이곳의 내용을 변경하면 로컬에는 잘 반영되지만, git에는 올라가지 않습니다. 또한 다른 기본 디렉토리의 내용을 변경하고 빌드하게 되면 이곳의 내용이 바뀌게 됩니다.
* _tabs : 왼쪽 사이드바의 기본 탭메뉴들에 대한 랜딩페이지가 들어 있습니다.
* assets : css, img등이 있습니다. 포스팅에 들어갈 이미지는 assets/img 아래에 넣으면 됩니다.
* tools : github에서 자동 배포를 위한 코드가 들어 있습니다. 이곳은 아예 건들지 맙시다



## _config.yml 수정
이제 나만의 환경을 위한 블로그 세팅에 들어가 봅시다.
수정할 내용은 아래와 같습니다.

| 항목 | 값 | 설명 |
|---|---|---|
| lang | ko | 언어를 한글로 설정합니다. 기본값은 en입니다.<br> ko에 대한 언어셋이 없기 때문에 사실상 효과 없습니다.<br> 그러나 나중에 커스터마이징을 위해 해 둡시다. |
| timezone | Asia/Seoul | 서울 표준시로 설정합니다.(만든분이 중국인인 것 같습니다 😁) |
| title | 아무거나~ | 블로그 제목을 넣어 줍니다. 아바타 바로 아래 큰 글씨로 표시됩니다 |
| tagline | 아무거나~ | title 아래에 작은 글씨로 부연설명을 넣을 수 있습니다 |
| description | 아무거나~ | [SEO](https://searchengineland.com/guide/what-is-seo){:target="_blank"}를 위한 키워드들을 입력합니다.<br>SEO에 대해 골치아프니, 쉽게 생각하자면 구글 검색에 어떤 키워드로 내 블로그를 검색하게 할 것인가를 결정해서 넣으면 됩니다. |
| url | <https://focuschange-test.github.io> | 내 블로그로 실제 접속할 url을 입력합니다 |
| github | github id | 본인의 github 아이디를 입력합니다 |
| twitter.username | twitter id | 트위터를 사용한다면 아이디를 입력합니다 |
| social.name | 이름 | 포스트 등에 표시할 나의 이름을 입력합니다 |
| social.email | 이메일 | 나의 이메일 계정을 입력합니다 |
| social.links | 소셜 링크들 | 트위터, 페이스등 내가 사용하고 있는 소셜 서비스의 나의 홈 url을 입력합니다 |
| theme_mode | light or dark | 원하는 테마 스킨을 선택합니다. 기본은 light입니다 |
| avatar | 이미지 경로 | 블로그 왼쪽 상단에 표시될 나의 아바타 이미지를 설정합니다 |
| toc | true | 포스팅된 글의 오른쪽에 목차를 표시합니다 |
| paginate | 10 | 한 목록에 몇개의 글을 표시해 줄 것인지 선택합니다 |

<br>
제가 설정한 [_config.yml](https://github.com/focuschange-test/focuschange-test.github.io/blob/master/_config.yml){:target="_blank"}을 참조하시면 됩니다. _config.yml을 수정하면 항상 jekyll을 재구동 해 줍니다.  
<br>


## 첫번째 포스팅 해보기
jekyll을 마크다운 문법으로 글을 작성할 수 있습니다. 다양한 툴들이 있으니 이들을 사용하면 되겠네요.  
마크다운 문법에 대해 간단히 알아보시려면 [여기](https://heropy.blog/2017/09/30/markdown/){:target="_blank"}를 확인해 보세요.   
편집기는 무엇을 써도 상관 없습니다.  
jekyll에서 제공하는 [jekyll 어드민](https://github.com/jekyll/jekyll-admin)을 사용하면 초보자들도 쉽게 마크다운 편집을 할 수 있습니다.  
배포나 관리가 아주 편리한데... chirpy에 설치해 보려니 계속 충돌이 일어납니다. (누가 좀 알려주세요~~ 😭)  
<br>

제가 주로 쓰는 툴은, 웹편집기로 [https://stackedit.io/](https://stackedit.io/){:target="_blank"}를 사용합니다.  
로컬에서는 [sublime text](https://www.sublimetext.com/){:target="_blank"}를 사용합니다.  
둘 다 장단점이 있긴 하지만 훌륭한 툴들입니다.  
<br>

글을 작성한 후 파일명은 반드시 아래 규칙을 지켜야 합니다.
* 형식 : `yyyy-mm-dd-제목.md`
* 년 4자리, 월 2자리, 일 2자리와 제목을 넣어줍니다.  
* 확장자는 `.md` 또는 `.markdown`으로 합니다. `.md`가 간편하겠지요~   
* 중간에 공백을 넣지 않습니다. 공백이 필요하면 `-`로 바꿉니다.  
* 작성한 파일은 _posts 디렉토리에 넣습니다.

<br>

이제 파일을 만들어 봅시다. 저는 간단히 "안녕하세요~"라고만 넣어 봤습니다.   
파일명은 [2021-12-23-welcome.md](https://github.com/focuschange-test/focuschange-test.github.io/blob/master/_posts/2021-12-23-welcome.md){:target="_blank"} 이렇게 정하고 `_posts` 디렉토리에 넣었습니다.   
브라우저에서 [http://127.0.0.1:4000/](http://127.0.0.1:4000/){:target="_blank"} 로 접속하면 새로 작성한 글의 목록이 보이게 됩니다.




# 배포
---
## 소스 올리기
로컬에서 모든 테스트가 끝났습니다. 수정한 내용을 github에 올릴 차례입니다.   
우선, `.gitignore` 파일을 열어서 아래 내용을 한줄 추가 합니다.   

```
Gemfile.lock
```

이 파일이 git에 올라가면 빌드/배포가 에러를 내는 경우가 많습니다. 그래서 올라가지 않도록 막는 것입니다.  


아래 git 명령을 사용하여 수정된 내용을 모두 github에 올립니다.(git에 대한 자세한 사용법은 구글링... )   

```bash
$ git add -A
...
$ git commit -m "first commit"  <-- -m 뒤에는 원하는 내용으로~
...
$ git push
```
<br>

## github에서 무슨 일이?
push를 하게되면 github은 자동으로 블로그 페이지를 만들어 줍니다. 시간이 좀 걸리는데요.. 잠시 살펴 봅시다.   
github의 저장소에 가보면, `Actions`라는 탭이 상단에 있습니다. 이를 클릭해 보면 아래와 같이 나타납니다.   
![깃헙 커밋 액션](/assets/img/github-commit-action.jpg)
<br>
위에서 commit할 때, `first commit`이라고 메세지를 넣어 두었습니다.  
위 그림에서 보이는 `first commit`이 그 내용입니다.  
정상적으로 종료가 된다면 왼쪽에 초록색 체크 아이콘이 붙게 될 겁니다. 위 그림은 진행중이네요. 제 경우 완료하는데 3분 25초가 걸렸습니다.  
<br>

소스를 push하게 되면, github은 자동으로 page build와 deployment를 수행합니다.   
이때, 중요한 것이, github이 `gh-pages` 라는 브랜치를 자동으로 생성해 줍니다.(사실, 이와 관련된 세팅이 이미 chirpy theme에 모두 들어 있습니다)
![gh-pages](/assets/img/build-gh-pages.jpg)

블로그 사이트는 이 브랜치에서 실제 내용이 구성되게 됩니다.   
우리가 push한 것은 master 브랜치이고, 여기에는 사이트에 대한 기본 구조가 없습니다.   
그래서, **한 가지 더 작업을 해 줘야 합니다.!!!**
<br>

## 서비스용 브랜치 변경하기
저장소의 상단 탭에 `Settings`가 있습니다. 이곳에서 서비스 페이지의 브랜치를 `gh-pages`로 변경해 줘야 합니다.
아래 그림처럼 순서대로 클릭하면 브랜치 리스트가 보입니다.
이중에 `gh-pages`를 선택하고 `Save` 버튼을 클릭합니다.
![change gh-pages](/assets/img/change-branch.jpg)
<br>
브랜치를 바꾸면 또다시 자동으로 build & deployment가 일어납니다. Actions 탭에 가 보면 적용중인 것이 보일 겁니다.   


## 블로그 확인하기
자동 배포가 완료되면 드디어 블로그 세팅이 끝난 것입니다. 나의 사이트에 들어가서 내용을 확인해 봅시다.   
저의 경우는 [https://focuschange-test.github.io/](https://focuschange-test.github.io/){:target="_blank"}가 이렇게 나오네요~  
성공입니다~  
![completed install](/assets/img/Chirpy_install_completed.jpg)


## 뒷정리
이제 불필요한 것들을 정리해 봅시다.
1. git branch 삭제 : master, gh-pages 브랜치를 제외하고 모두 삭제합니다.
1. 🤔 더 삭제할 것이... 없군요... 😝
<br>

# 마무리하며
chirpy는 정말 설치하기도 편하고 디자인도 단순하면서 깔끔합니다. 원했던 검색기능이나 카테고리 기능도 손댈 필요가 없을 정도로 너무 좋네요.  
front 수정은 상당히 난해(?)해서 최대한 손대지 않는 테마를 찾았었는데, 제게 딱 맞는 것을 찾은 것 같습니다.  
개발자에게 경의를~   
다음은 간단한 커스터마이징하는 것을 포스팅하겠습니다. 

<br>


# 참고
---
* 깃헙 블로그 만들기 : <http://blog.ju-ing.com/posts/Github-blog-chirpy-theme/>{:target="_blank"}
* chirpy theme github : <https://github.com/cotes2020/jekyll-theme-chirpy>{:target="_blank"}
* 마크다운 사용법 총정리 : <https://heropy.blog/2017/09/30/markdown/>{:target="_blank"}
* github 저장소 생성하기 : <https://www.lainyzine.com/ko/article/how-to-create-a-new-remote-git-repository-on-github/>{:target="_blank"} 
* git 튜토리얼 : <https://evan-moon.github.io/2019/07/25/git-tutorial/>{:target="_blank"}  


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