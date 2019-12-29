# 프로그라피 기술 블로그

프로그라피 기술 블로그입니다. 

- jekyll 로 만들어져있습니다. 작성 후 PR 을 날려주세용
- PR 전 로컬에서 확인하고 싶은경우 jekyll을 깔아주세요
    https://jekyllrb-ko.github.io/docs/installation/

## Getting Started

### 기본 설정 


1. https://github.com/prography-tech/prography-tech.github.io 포크 하기

2. 포크 저장소 계정(개인 계정) 선택

   ```bash
   $ git clone https://github.com/soomtopia/prography-tech.github.io.git
   $ cd kakao.github.io
   $ git remote add upstream git@github.com:prography-tech/prography-tech.github.io.git # 원격 저장소 등록
   ```

   리모트 확인

   ```bash
   $ git remote -v
   origin	https://github.com/user-name/prography-tech.github.io.git (fetch)
   origin	https://github.com/user-name/prography-tech.github.io.git (push)
   upstream	git@github.com:prography-tech/prography-tech.github.io.git (fetch)
   upstream	git@github.com:prography-tech/prography-tech.github.io.git(push)
   ```

   git fetch

   ```
   $ git fetch upstream
   $ git checkout master
   ```



### 글 등록

1. _posts 폴더에 자신의 글을 이동시키기 

    - 형식은 markdown 

    - 제목 형식은  `yyyy-mm-dd-<title>.md`  (년-월-일)

2. 파일 최 상단에 레이아웃 등록하기

    sample

    ```
    # 2019-03-18-testposts.md
    ---
    layout: post #require
    title: "원하는 제목 등록"
    author: "자신의 작성 이름 등록"
    date: 2019-10-11 00:00 # 작성 날짜 등록 
    tags: [algorithm, python, djnago] # 원하는 태그 등록 
    image: 'https://github.com/jangjichang/Today-I-Learn/raw/master/Algorithm/theory/stack.jpg?raw=true' # 원하는 이미지 url 등록 
    ---
    # 이 아래에 마크다운 형식의 포스트를 작성하세요요요
    ```


    - 이미지의 경우 주소 url 을 붙히면 좋다
    - 로컬에 있는 이미지를 올리고 싶은 경우 assets/images 폴더에 넣자
    -  `assets/images/<photo-name>.jpg` 와 같이 등록한다. 

    - __작성자와 태그가 authors , tags 에 등록되지 않았다면 등록 해주어야한다.__



### 작성자 등록하기

1. _authors 폴더에 `<user-name>.md` 이름으로 파일 등록하기

2. `<user-name>.md` 파일 최상단에 레이아웃 등록

   ```
   ---
   name: soomtopia
   title: 한수민 # 이름 또는 닉네임등록 
   image: https://avatars2.githubusercontent.com/u/50104236?s=460&v=4
   intro: django
   ---
   # 자기 소개를 적으면 된다 
   ```

   ###### sample

   ```markdown
   ---
   name: bluestragglr
   title: 신성환
   image: https://avatars1.githubusercontent.com/u/44422495?s=400&v=4
   intro: vue.js developer
   
   ---
   
   안녕하세요. 저는 갓성환입니다. 요리 코딩 겅부 포켓몬 다 잘하죠 훗 
   
   ####  github
   http://github.com/sin
   
   #### 주요언어 
   vue.js / html / css / javascript
   ```

   ###### result

   <https://prography-tech.github.io/authors/bluestragglr/>



### 태그 등록하기 

자신이 달고싶은 태그가 현재 _tags 에 없다면 등록해준다. 

- _tags 폴더에 `<tag_name>.md` 형식으로 파일을 등록해준다

- 파일 최 상단에 아래와 같이 등록

  ```markdown
  ---
  name: algorithm
  title: 'algorithm'#태그에 대한 설명
  image: #태그 관련 이미지. 옵션
  ---
  ```
