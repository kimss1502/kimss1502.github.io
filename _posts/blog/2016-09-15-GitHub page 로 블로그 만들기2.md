---
layout: single
categories:
  - 블로깅
title: "GitHub pages로 블로그 만들기2"
toc: true
---

## 1. 저장소(Repository) 생성 및 설정
 Git repository를 이용하는 것인만큼 Git에 새로운 Repository를 만들어야 한다. <br>
 주의할 점은 이 때 Repository 이름에 따라서 블로그의 도메인 주소가 바뀐다. <br/>
 `https://id.github.io` 형태로 만들고 싶다면 Repository name은 `id.github.io` 형태로 만들어야 한다. 만약 다른 이름(예를 들어 ProjectName) 으로 만들었다면 도메인은 `id.github.io/ProjectName` 이 된다. 

 그리고 생성할때는 Public 으로 만들어야 한다. (Private으로 만들려면 유료 요금제 필요)

 ![aa](./_attach/gitblog_new_repository.png)
> 위에서 나오는 경고는 이미 같은 이름의 Repository가 존재해서 나오는 것.


## 2. 원하는 Jekyll 테마 선택하기
 [https://github.com/topics/jekyll-theme](https://github.com/topics/jekyll-theme), 또는 [http://jekyllthemes.org/ ](http://jekyllthemes.org/ ) 를 참고하여 내가 원하는 테마를 선택하면 된다. <br>
 
 난 이중 [minimal-mistakes 테마](https://github.com/mmistakes/minimal-mistakes) 를 선택하였다.
 

## 3. 블로그 설치
> [https://github.com/mmistakes/minimal-mistakes#installation](https://github.com/mmistakes/minimal-mistakes#installation)  참고

 minimal-mistakes 테마의 경우 3가지 설치방법을 제공한다. 
 
 1. Gem 기반 방법
 2. Remote theme 방법
 3. minimal-mistakes repository를 fork 하거나 직접 다운로드하여 모든 파일을 내 프로젝트에 넣는 방법

설치 방법과 별개로 공통적으로 블로그 설정을 위한 `_config.yml` 파일의 수정이 필요하다. <br>
이곳에서 사이트 제목, 작성자 정보, 기타 설정을 변경할 수 있는데, 설정이 많으므로 자세한 내용은 **[공식 가이드](https://mmistakes.github.io/minimal-mistakes/docs/configuration/)** 를 참고하면 된다. 
 
 설치 방법과 관련된 `_config.yml` 파일 수정은 아래에서 다룬다.

### 3.1. Gem 기반 방법 (minimal-mistakes 테마는 github 지원 x)
> Gem에 대해서는 []() 포스팅을 참고할 것. <br>
 
 Gem은 Ruby 프로젝트의 라이브러리이다. <br>
 minimal-mistakes 테마는 gem 으로도 배포가 되어 있어서 이를 통해 블로그를 간단히 구성할 수 있는 방법을 제공하였다.
 
 Gem 기반으로 설치하는 방법의 경우 [공식 사이트](https://jekyllrb.com/docs/themes/#understanding-gem-based-themes) 설명 내용을 읽어보면 좀 더 이해에 도움이 된다.
 
  간단히 설명하면 jeykyll 의 테마 gem에는 `assets`, `_layouts`, `_includes`, `_sass` 디렉토리의 파일들이 포함되어 있다고 한다. 테마에 필요한 파일을 가지고 있다보니 내가 블로그를 구축할때는 이 파일들이 없어도 되는 장점이 있다. (아니면 위에서 열거한 저런 테마 관련 파일들을 직접 가지고 있어야 함.)
  
  라이브러리 형태인 gem으로 배포되다보니 테마가 업데이트 되는것을 반영하는것이 간단하고, 다른 테마로 바꾸는 것도 쉽다. <br>

#### 3.1.1. 설치 과정
 local에서 테스트해본 경우 아래와 같이 할 수 있었다. 
 
 1. minimal-mistakes 테마 repository를 내 컴퓨터에 다운받는다.

 2. 내가 만들 블로그 폴더를 생성하고,  다운받은 minimal-mistakes 파일에서 일부 가져온다. <br>

    ![gitblog_gem_base_files](./_attach/gitblog_gem_base_files.png)
    > _posts는 내가 포스팅할 내용들이 담기는 폴더로 처음에는 empty 폴더로 두면 된다.
     
 3. `_config.yml` 파일 수정 <br>
     theme 주석을 푼다. remote_theme은 주석 처리를 유지한다.
     
     ```
     theme                             : "minimal-mistakes-jekyll"
     # remote_theme             : "mmistakes/minimal-mistakes"
     ```
  
 4. `Gemfile` 파일 수정 <br>
     
     ```
     source "https://rubygems.org"
     gem "minimal-mistakes-jekyll"
     ```
     
 5. 콘솔에서 `bundle` 명령 실행을 통해 필요한 gem 을 설치한다. <br>
     실행되면 `Gemfile.lock` 파일이 생성된다.
          
 6. 콘솔에서 `bundle exec jekyll serve` 로 블로그 실행 <br>
     빌드 과정에서 `_site` 폴더가 만들어지면서 블로그 파일들이 자동으로 생성된다. <br>
     
     ![gitblog_gem_base_files2](./_attach/gitblog_gem_base_files2.png)
  > _posts 폴더는 테스트를 위해 넣어둔것.
 
 7. `http://localhost:4000/` 로 확인하기 


#### 3.1.2. 주의 사항
 minimal-mistakes 테마의 경우 gem 기반 방식은 github에서 지원하지 않았다.(이것때문에 삽질을...) <br>
 테마 README 파일에는 되는것처럼 되어 있었는데 이건 jeykll이 github 만을 위한것은 아니기 때문에 명시한 것인듯하다.
 
 이렇게 구성하고 내 repository에 commit한 경우 이메일로 아래 빌드 warning 메일이 날라왔다.
 
 You are attempting to use a Jekyll theme, "minimal-mistakes-jekyll", which is not supported by GitHub Pages. Please visit [https://pages.github.com/themes/](https://pages.github.com/themes/) for a list of supported themes. If you are using the "theme" configuration variable for something other than a Jekyll theme, we recommend you rename this variable throughout your site. For more information, see https://docs.github.com/github/working-with-github-pages/adding-a-theme-to-your-github-pages-site-using-jekyll.

 지원 가능한 테마의 경우 [https://pages.github.com/themes/](https://pages.github.com/themes/) 에서 확인할 수 있었는데 수가 많지 않았다.
 

### 3.2. Remote theme 방법 (minimal-mistakes 테마는 github 지원 O) 
 Remote theme 방식은 Gem 기반 방식과 유사하게 직접 테마 파일을 가지지 않는 방법이다. Github pages 에서 minimal-mistakes 테마의 Remote theme 방식을 지원하므로 이 방법을 사용할 수 있다.

#### 3.2.1. 설치 과정

 1. minimal-mistakes 테마 repository를 내 컴퓨터에 다운받는다.

 2. 내가 만들 블로그 폴더를 생성하고,  다운받은 minimal-mistakes 파일에서 일부 가져온다. <br>
    (Gem 기반 방식과 동일하게 `_config.yml`, `Gemfile`, `index.html` 을 가져온다.)
 
 3. `_config.yml` 파일 수정 <br>
     remote_theme 주석을 푼다. theme은 주석 처리를 유지한다.
     
     ```
     # theme             : "minimal-mistakes-jekyll"
     remote_theme   : "mmistakes/minimal-mistakes"
     ```
    
     plugins 에 `jekyll-include-cache` 가 있는지 확인한다. 
     
     ```
     plugins:
        ....
        - jekyll-include-cache
     ```
     
 4. Gemfile을 아래와 같이 수정한다.  <br>
      
    ```
    source "https://rubygems.org"

    gem "github-pages", group: :jekyll_plugins
    gem "jekyll-include-cache" 
    ``` 
 
 5. 콘솔에서 `bundle` 명령 실행을 통해 필요한 gem 을 설치한다. <br>
     실행되면 `Gemfile.lock` 파일이 생성된다.
          
 6. 콘솔에서 `bundle exec jekyll serve` 로 블로그 실행 <br>
     빌드 과정에서 `_site` 폴더가 만들어지면서 블로그 파일들이 자동으로 생성된다. <br>

     ![gitblog_gem_base_files3](./_attach/gitblog_gem_base_files3.png)
     > 생성되는 파일은 gem 기반 방식과 동일했다.
     
 7. `http://localhost:4000/` 로 확인하기 

#### 3.2.2. Github page에 올리는 방법
 Remote theme 방식을 지원하기 때문에 현재 파일들을 그대로 내 repository에 commit, push 하면 된다. <br>
 약간의 빌드시간 후 `https://id.github.io` 로 접속해보면 로컬에서 봤던 페이지 그대로 보이는것을 알 수 있다.
 
  
### 3.3. minimal-mistakes 소스 전체 받아서 하는 방법

#### 3.3.1. 설치 과정

 1. minimal-mistakes 테마 repository 전체를 내 Repository에 복사한다. <br>
     두가지 방법이 있을것이다. <br>
     
       A. minimal-mistakes repository를 fork한 후 이름바꿔서 쓰기 <br>
       B. minimal-mistakes 소스를 다운로드 받고, 새로 생성한 내 repository에 push 하기

 2. 전체 소스를 내 컴퓨터에 가져온다. <br>
      fork한 경우 fork한 내 repository를 가져오면 될 것이고, 다운로드 후 내 repository에 push한 경우 이미 로컬에 테마의 모든 파일이 있을 것이다.
      
 3. 필요없는 파일 삭제하기. <br>
      minimal-mistakes 테마 repository에는 샘플용으로 들어있는 불필요한 파일 및 repository의 README 파일 등과 같은것이 있으므로 이들은 삭제해야 한다.
      
     ```
     .editorconfig
     .gitattributes
     .github
     /docs
     /test
     CHANGELOG.md
     README.md
     minimal-mistakes-jekyll.gemspec
     screenshot-layouts.png
     screenshot.png
     ```

 4. `_config.yml` 파일 수정 <br>
     theme, remote_theme 주석을 모두 유지한다.
     
     ```
     # theme             : "minimal-mistakes-jekyll"
     # remote_theme   : "mmistakes/minimal-mistakes"
     ```

 5. 콘솔에서 `bundle` 명령 실행을 통해 필요한 gem 을 설치한다. <br>

 6. 콘솔에서 `bundle exec jekyll serve` 로 블로그 실행 <br>
  
 7. `http://localhost:4000/` 로 확인하기 

#### 3.3.2. Github page에 올리는 방법
 수정한 모든 파일들을 그대로 내 repository에 commit, push 하면 된다. <br>
 약간의 빌드시간 후 `https://id.github.io` 로 접속해보면 로컬에서 봤던 페이지 그대로 보이는것을 알 수 있다. 


## 4. 포스팅하기
> [minimal-mistakes 공식 가이드](https://mmistakes.github.io/minimal-mistakes/docs/posts/)
> [Jeykll 공식 가이드](https://jekyllrb.com/docs/posts/)

 포스팅을 하고 싶은 경우 `_post` 디렉토리 밑에 markdown으로 포스팅 내용 작성 후 repository에 commit, push 하면 된다.
 
 단, 이 때 파일명에 대해서 아래 규칙을 지켜줘야 한다. 
 
 ```
 YEAR-MONTH-DAY-title.MARKUP
 
 ex) 
   2011-12-31-new-years-eve-is-awesome.md
   2012-09-12-how-to-write-a-blog.md
 ```

그리고 포스팅하는 글 본문의 첫 시작을 `YAML front Matter` 형식으로 기술하면 포스팅되는 글에 대한 설정을 할 수 있다.

아래는 예이다. 자세한 설명은 [Jekyll 가이드](https://jekyllrb.com/docs/front-matter/)를 참고한다.

```
---
layout: single
categories: 
  - 내 카테고리
tags:
  - 태그1
  - 태그2
title:  "안드로이드에 대해서"
toc: true 
---

## 안드로이드에 대해서
 - 프레임워크와 라이브러리 차이를 설명할 수 있다.
 - `new Activity()` 코드가 왜 없는지에 대해 설명할 수 있다.
 - Lifecycle을 누가 호출하는지에 대해서 설명할 수 있다.
```

특정 설정이 매번 필요하다면 공통적인 부분에 대해서 `_config.yml` 파일에서 default 설정을 할 수 있다.

```
defaults:
  # _posts
  - scope:
      path: ""
      type: posts
    values:
      layout: single
      author_profile: true
      read_time: true
      comments: true
      share: true
      related: true
``` 

위 처럼  `single` 레이아웃에 대해서 default 설정을 해 둘 수 있다. default 설정을 하되 특정 포스트에서는 변경하고 싶다면 동일한 속성을 override 할 수 있다.




---

> **[참고 문서]**
> 
> https://flik.tistory.com/3
> https://www.ruby-lang.org/ko/libraries/
> https://www.railsguidebook.com/contents/walkthrough/gemfile.html
> https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/
> https://jekyllrb.com/docs