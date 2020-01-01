---
layout: post
title: "클래스 이름, 막 써도 되나요?"
author: bluestragglr
date: 2019-12-31 17:45
tags: [svg, html, css]
image: 'https://blogeduonix-2f3a.kxcdn.com/wp-content/uploads/2018/08/bem.jpeg'
---


> 클래스 이름, 막 써도 되나요? by 신성환(github.com/blueStragglr)

---



프론트 개발을 하다 보면 HTML 요소에 클래스 이름을 붙이는 것을 상당히 어려워 하는 사람들을 많이 볼 수 있습니다. 무슨 이름을 붙였을 때 다른 사람 (혹은 미래의 자신)이 알아볼 수 있을지, 문제가 생기지는 않을지에 대해 고민하는 것이죠.

HTML의 클래스 이름을 붙일 때, 각 요소가 어떤 동작을 하는가에 따라 명료한 규칙을 가지고 클래스 이름을 붙이면 보다 생산적으로 개발할 수 있습니다. 이러한 규칙으로써 범용적으로 정의되고 사용되는 BEM에 대해서 간단히 소개해 볼까 합니다.

 

### BEM이 뭔가요?

BEM은 Block, Element, Modifier의 앞글자를 딴 약어입니다. HTML의 구조를 세 계층으로 구조화하여 각각을 구분할 수 있는 형태로 클래스의 이름을 짓는 것을 BEM 스타일이라고 부릅니다. 각각의 요소는 다음과 같은 의미를 가집니다. 



### Block

Block은 특정 기능을 하는 단위로 볼 수 있습니다. 그렇기에 각 블록의 이름은 수행하는 목적을 명료하게 표시해줄 수 있는 편이 권장됩니다. 이 때, Block은 최소 단위를 뜻하는 것이 아니며, 중첩될 수 있습니다.

에러를 나타내는 빨간색 요소가 있다고 할 때, 권장되는 예시와 권장되지 않는 예시는 아래와 같습니다.

```html
<!-- Recommended -->
<div class="error"> ... </div>

<!-- Not Recommended -->
<div class="big-red-text"> ... </div>
```

CSS적으로 블록은 독립적인 상태를 유지하는 것이 권장됩니다. `margin`이나 `position` 등으로 인하여 외부 요소의 위치를 변경시킬 수 있는 블록은 외부 요소에 의존성을 발생시킬 수 있습니다. 즉, 특정 블록이 사라짐으로 인하여 블록을 감싸고 있던 레이아웃이나 이웃 요소의 스타일이 방해받는 것은 바람직하지 않습니다.



### Element

Element는 기능을 이루는 하위 단위로써, 해당 객체가 무엇인지를 인지할 수 있는 단위입니다. Element의 예시로는 텍스트나 인풋, 버튼 등을 들 수 있습니다.

Element를 나타내는 방법은 Block name 뒤에 언더스코어 두 개 (`__`)를 붙인 뒤 Element name을 붙이는 것입니다. `search-form` 이라는 Block 내의 input과 button에 대해 권장되는 예시와 권장되지 않는 예시는 아래와 같습니다.

```html
<!-- Recommended -->
<form class="search-form">
    <div class="search-form__content">
        <input class="search-form__input">
        <button class="search-form__button">Search</button>
    </div>
</form>

<!-- Not Recommended -->
<form class="search-form">
    <div class="search-form__content">
        <input class="search-form__content__input">
        <button class="search-form__content__button">Search</button>
    </div>
</form>
```



### Modifier

Modifier는 Block이나 Element의 행동, 상태, 외양을 정의하는 속성입니다. Block과 Element와는 다르게, 조금 더 시각적인 부분의 속성을 표현하는 값으로 생각할 수 있습니다. 사이즈, 테마, 활성화 여부 등을 표시하는 데 사용됩니다.

Modifier를 나타내는 방법은 Element의 표현과 비슷합니다. 표현하고자 하는 객체의 클래스 이름 뒤에 언더스코어 한 개 (`_`)를 붙인 뒤 Modifier를 작성합니다.



### Anything Else?

BEM은 클래스 이름뿐만 아니라 디렉토리 구조, 파일 단위 등 여러 가지에 함께 적용될 수 있는 개념이며, 다양한 규칙이 더 존재합니다.

추가적인 규약이나 내용 등에 대해서는 레퍼런스에 잘 정리되어 있으니, 더 관심이 있으신 분들은 찾아보시면 도움이 될 것 같습니다.



### 꼭 써야 할까요?

BEM은 클래스와 스타일의 네이밍을 위한 규칙 중 하나입니다. 이외에도 다양한 규약이 있을 수 있고 자신만의, 혹은 팀만의 규약이 존재할 수도 있습니다. BEM은 여러가지의 규약들 중 제법 범용적이고 직관적인 규약이라고 볼 수 있습니다. 문서화가 잘 되어 있음은 물론, 많은 사람들이 이미 사용하고 있기 때문이죠. 따라서 새로운 팀을 만났을 때 좀 더 많은 사람들이 익숙할 가능성이 높은 규약이기에 적응이 빠를 수 있습니다. 혹시 전혀 감이 오지 않는 경우에 괜찮은 스탠다드로써 사용할 수 있겠지요.

문제를 발생시키기 위한 필수적인 선택은 아니지만 클래스 이름의 혼용에서 오는 문제를 미연에 방지하는 등 생산적인 개발을 위해서는 꽤 유효한 수단이 될 수 있겠습니다 😊



### Reference

[Methodology](https://en.bem.info/methodology/quick-start/#block)

[BEM( Block, Element, Modifier) Quick start](
