---
layout: post
title: "Vue.js에서 Anime.js 사용하기"
author: bluestragglr
date: 2019-01-28 18:06
tags: [html, css, javascript]
image: 'https://user-images.githubusercontent.com/44422495/73249696-b73acb80-41f8-11ea-813f-a9c02338245b.png'
---


> Vue.js에서 Anime.js 사용하기  by 신성환(github.com/blueStragglr)

---



Vue.js 프로젝트에서 애니메이션의 손쉬운 구현을 위해  Anime.js 라이브러리를 사용하는 방법에 대해서 알아봅니다. 하나의 DOM 객체에 대해 다루는 기능을 중점적으로 서술하였습니다. 애니메이션 종료 후에 실행되는 콜백이나 SVG를 컨트롤하는 방법에 대해서는 다음에 다루도록 하겠습니다! 



## Installation & Import

npm을 이용해서 쉽게 설치할 수 있습니다.

```shell
$ npm install animejs --save
```

이후, 애니메이션을 사용할 컴포넌트에서 다음과 같이 import합니다.

```shell
import anime from 'animejs/lib/anime.es.js'

// use as `anime(...)`
```

애니메이션을 사용하는 컴포넌트가 많다면 `src/main.js`에서 다음과 같이 전역 변수로 import해 줍시다. 이 경우, 모든 컴포넌트에서 `this.$anime` 를 통해 anime 모듈을 호출할 수 있습니다.

```javascript
import anime from 'animejs/lib/anime.es.js'

Vue.prototype.$anime = anime;

// use as `this.$anime(...)`
```

## How to use

사용 방법은 직관적이고 간단합니다. anime 모듈은 css queryselector와 같은 방법으로 DOM 객체를 가르킬 수 있으며, 해당 객체에 애니메이션을 부여할 수 있습니다.

따라서, Vue.js 프로젝트에서 anime 모듈을 사용하기 위해서는 라이프사이클상 DOM이 생성된 후인 mounted() 단계에서 애니메이션을 선언해 주면 됩니다. 만일 애니메이션 트리거가 별도로 존재하는 경우(버튼을 눌렀을 때 애니메이션 시작 등)에는 해당 interaction 단계에서 anime 모듈을 바인딩합니다.

```php+HTML
<template>
  <div class="mobile-mockup__container">
    <img class="eye-tracker" src="../../assets/2x/eyetracker.png">
  </div>
</template>

<script>

  export default {
    name: "MobileMockup",
    mounted() {
      this.$anime({
        targets: '.eye-tracker',
        translateX: '250px',
        duration: 8000
      })

    }
  }
</script>
```

위 코드에서는 `class='.eye-tracker'` 인 DOM(img)에 anime 모듈이 바인딩되어 8000ms동안 250px의 translateX를 수행합니다.

## Properties

anime 라이브러리에는 translateX나 duration 이외에도 다양한 속성들이 존재합니다. 공식 문서상의 자세한 예시들은 **[여기](https://animejs.com/documentation/)**에서 확인할 수 있습니다. 라이브러리에 포함된 속성 중 중요하고 핵심적인 것들을 골라 정리해 보았습니다.

### targets

target 속성은 하나의 DOM을 가르킬 수도 있지만, 한번에 여러 DOM을 가르키는 것 또한 가능합니다. 아래와 같이, querySelectorAll등을 통해 다수의 DOM을 선택하고 같은 애니메이션을 일괄적으로 적용하는 것이 가능합니다.

```javascript
var elements = document.querySelectorAll('.dom-node-demo .el');

anime({
  targets: elements,
  translateX: 270
});
```

### CSS properties

(당연하게도) Keyframe을 통해 애니메이션을 만들던 것과 같이 CSS property에 직접 접근하여 값을 부여할 수 있습니다. JavaScript에서 CSS를 직접 핸들링할때와 같이 각 속성의 이름은 carmelCase로 변경하여 작성하며, 여러 개의 attribute가 들어가는 경우에는 string array로 묶어 삽입합니다.

```javascript
anime({
  targets: '.css-prop-demo .el',
  left: '240px',
  backgroundColor: '#FFF',
  borderRadius: ['0%', '50%'],
  easing: 'easeInOutQuad'
});
```

### Numerical value

CSS 속성이 아닌 일반 텍스트의 경우에도 숫자를 포함하고 있다면 애니메이션 효과를 줄 수 있습니다. 숫자가 아닌 문자를 무시하고 값이 순차적으로 변합니다.

round 속성은 수치의 변화를 얼마나 잘게 쪼갤지에 대한 속성으로, 값이 작을 수록 변화의 단위가 커집니다. 즉, round가 1이라면 수치는 1씩 변화하고, round가 0.1이라면 수치는 10씩 변합니다. 여러 개의 변수가 존재하여 수치 변화의 폭이 다르다고 하더라도, 한 번에 숫자가 변화하는 크기는 같게 고정되며 변화 사이의 시간만이 변수마다 다르게 조정됩니다.

```javascript
let myObject = {
  prop1: 0,
  prop2: '0'
};

this.$anime({
  targets: myObject,
  prop1: 50,
  prop2: '10000',
  easing: 'linear',
  round: 0.01,
  duration: 8000,
  update: function() {
    objPropLogEl.innerHTML = JSON.stringify(myObject);
  }
});
```

![round](https://user-images.githubusercontent.com/44422495/73249725-c28df700-41f8-11ea-9ffd-b5ea025556bd.gif)



### values

- unit

CSS 속성을 지정하는 경우, 단위를 입력하지 않아도 묵시적으로 기본 단위가 적용되는 속성들이 있습니다.

```javascript
anime({
  targets: '.unitless-values-demo .el',
  translateX: 250, // -> '250px'
  rotate: 540 // -> '540deg'
});
```

각 속성의 기본 단위에 대한 자세한 내용은 공식 문서(Reference 참조)에서 확인할 수 있습니다.

### Direction

애니메이션이 실행될 방향을 지정할 수 있습니다. 초기 상태에서 지정한 애니메이션 값으로 변화하는 normal, 지정 값에서 초기 값으로 돌아오는 reverse, 초기값과 지정값을 왕복하는 alternate 속성을 부여할 수 있습니다.

### Loop

애니메이션의 반복 횟수는 Loop property로 정할 수 있습니다. Integer 값을 주게 되면 해당 횟수만큼 반복하며, true를 주게 되면 무한 반복하게 됩니다.

### Object Parameter & Animation Chaining

여러 속성에 Object를 Parameter로 전달하여 하나의 객체에 복합적인 애니메이션을 생성할 수 있습니다.

```javascript
this.$anime({
	  targets: '.eye-tracker',
	  translateX: {
	    value: 250,
	    duration: 800
	  },
	  rotate: {
	    value: 360,
	    duration: 1800,
	    easing: 'easeInOutSine'
	  },
	  scale: {
	    value: 2,
	    duration: 1600,
	    delay: 800,
	    easing: 'easeInOutQuart'
	  },
	  delay: 250 // All properties except 'scale' inherit 250ms delay
	});
}
```

위와 같은 코드를 작성하게 되면 세 종류의 각기 다른 애니메이션이 한번에 혼합되어 삽입됩니다. delay와 같은 property를 object parameter에 설정하지 않고 외부에 작성한 경우, 각 property는 외부에 작성된 값을 상속합니다. 이를 응용하여 각 애니메이션 내의 delay를 수동으로 조절함으로써 원하는 순서대로 애니메이션이 실행되도록 할 수 있습니다.

### Timeline

여러 오브젝트의 애니메이션을 연계하려는 경우, Timeline을 이용할 수도 있습니다.

```javascript
// Create a timeline with default parameters
var tl = anime.timeline({
  easing: 'easeOutExpo',
  duration: 750
});

// Add children
tl
.add({
  targets: '.basic-timeline-demo .el.square',
  translateX: 250,
})
.add({
  targets: '.basic-timeline-demo .el.circle',
  translateX: 250,
})
.add({
  targets: '.basic-timeline-demo .el.triangle',
  translateX: 250,
});
```

각기 다른 DOM에 바인딩된 애니메이션들을 add 메소드를 통해 순서대로 실행되도록 할 수 있습니다.



### Reference

[anime.js](