---
layout: post
title: "CSS pre-loader 없이 변수 만들어 활용하기"
author: bluestragglr
date: 2020-1-2 15:45
tags: [html, css]
image: 'https://t1.daumcdn.net/cfile/tistory/995E674F5AC89CBF39'
---


> CSS pre-loader 없이 변수 만들어 활용하기 by 신성환(github.com/blueStragglr)

---



SCSS와 같은 CSS Pre-loader를 사용하게 되면 변수를 선언하고 스타일을 관리하는데 유용하게 사용할 수 있습니다. primary-color 등을 선언해 두고, 글로벌 단위에서의 색상 변경을 변수 하나의 변화만으로 관리하는 작업 등이 가능해 지는 것입니다. 하지만 CSS에서도 전역변수의 형태로 값을 관리할 수 있는 방법이 있었습니다!



### 변수 사용, SCSS에서만 되는게 아니었다!

알고 보니 변수의 선언은 SCSS의 전유물이 아니었습니다. 가상 선택자 `:root` 에 접근하여 CSS에서도 전역 변수 등을 생성할 수 있습니다.

우선 변수 선언을 위하여, 아래와 같이 HTML 파일을 작성하였습니다. 각자 다른 4개의 클래스가 존재합니다.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <link rel="stylesheet" href="rootCSS.css">
  <title>rootCSS</title>
</head>
<body>
  <div class="class-1">Colored by `--main-color` in  `:root`</div><br/>
  <div class="class-2">Background-colored by `--main-color` in  `:root`</div><br/>
  <div class="class-3">Border solid by `--main-color` in  `:root`</div><br/>
  <div class="class-4">Box-shadow by `--main-color` in  `:root`</div>
</body>
</html>
```

이제 각 클래스에 대하여 스타일을 적용하기 위하여, `:root`에 `--main-color: #123456` 을 작성하고 실행 해 보았습니다.

```css
:root{
  --main-color: #123456
}
div{
  display: inline-block;
  height: 30px;
  margin: 10px;
}

.class-1{
  color: var(--main-color);
}
.class-2{
  background-color: var(--main-color);
}
.class-3{
  border: 1px solid var(--main-color);
}
.class-4{
  box-shadow: 2px 2px 2px 2px var(--main-color);
}
```

그 결과, 다음과 같이 색상이 변수로써 작용하여 #123456 으로 각 요소들이 렌더링 되었습니다.

![css blue example](/assets/posts/blueStragglr/css-root/css_blue.png)

다음으로, 차이를 확인하기 위하여 `:root` 에 대해 작성된 CSS를 변경하였습니다.

```css
/* #123456 -> #00aa00 */

:root {
  /*--main-color: #123456*/
  --main-color: #00aa00;
}
```

의도했던 바와 같이, 전체 색상 테마를 바꾸는 느낌으로 동작할 수 있게 되었습니다.

![css green example](/assets/posts/blueStragglr/css-root/css_green.png)

### References

[CSS3 :root 변수를 이용한 HTML 스타일 적용 방법](https://rgy0409.tistory.com/3074)

[[css3\] :root 가상 클래스](https://aboooks.tistory.com/315)