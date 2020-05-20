---
layout: post
title: "Vue.js Mixin: 기능 캡슐화하기"
author: bluestragglr
date: 2020-05-20 12:18
tags: [vuejs, html, css, javascript]
image: 'https://user-images.githubusercontent.com/44422495/82424593-5e04e980-9ac0-11ea-83e0-2d4daaf47801.png'
---


> Vue.js Mixin: 기능 캡슐화하기 by 신성환(github.com/blueStragglr)

# Vue.js Mixin: 기능 캡슐화하기

Vue.js는 컴포넌트 단위로 움직이는 프레임워크입니다. 일반적으로는 부모자식 컴포넌트를 개발하고 컴포넌트 단위로 구조를 설계하게 되지만, 좀 더 작은 단위에서 기능을 반복하는 등의 경우가 있을 수 있습니다. 특정 라이프사이클 훅을 추가하거나, 함수를 추가할 때 Vue.js 프레임워크에 알맞는 방식으로 사용할 수 있는 믹스인을 소개합니다. 

믹스인은 뷰의 컴포넌트 라이프사이클에 대한 이해가 제법 필요합니다. 컴포넌트가 생성되고 사라지기까지의 과정이 아직 낯선 분이라면 믹스인의 활용은 조금 미루어 두는 것을 추천드립니다!

## 목차

1. 믹스인이 뭐에요?
2. 기본 예제
3. 실제 사용 예시
4. 무엇이 개선됐나요?

### 믹스인이 뭐에요?

믹스인은 이름이 나타내고 있는 것 처럼 컴포넌트에 무언가를 섞어 원하는 것을 구현하는 기능입니다. 믹스인을 구현하고 동작시키는 과정을 순서대로 쓰면 아래와 같이 정리할 수 있습니다.

1. 믹스인할 객체를 만들고
2. 컴포넌트에 객체를 믹스인하고
3. 나머지 부분을 구현해 완성한다

여기서 가장 중요한 '2. 컴포넌트에 객체를 믹스인하고' 과정에서는 아래와 같은 (그림에 시각화해둔) 원칙들이 적용됩니다.

![Vue mixin의 구조](/assets/posts/blueStragglr/vue-mixin/1.png)

a. HTML/CSS는 믹스인하지 않습니다. 즉, 믹스인 파일은 순수 자바스크립트로 작성된 객체입니다.

```jsx
// 가능하긴 하지만 권장되지 않음!

let mixin = {
	...
	Template: `<div> ...</div>`
	data {
		return {
			styleObj: { 'width' : 100 }
		}
	}
}
```

b. 믹스인에서 선언한 속성(e.g. data)를 컴포넌트에서 다시 선언할 수 있으며, 이 경우 컴포넌트에 선언된 값을 우선하여 병합됩니다.

```jsx
// 즉, 아래와 같이 mixin.js와 component.vue에 같은 속성이 중복 선언된 경우에
// mixin.js
...
data {
	return {
		age: 20
	}
}
...

// component.vue
...
data {
	return {
		age: 26
	}
}
...

// 컴포넌트의 선언값이 살아남습니다.
// result
<div>{{ age }}</div>
// => <div>26</div>
```

c. 믹스인에만 선언하고 컴포넌트에 선언하지 않는 경우에는 (당연하게도) 믹스인에 선언한 속성이 남습니다.

### 기본 예제

우선 Lifecycle Hook을 믹스하는 경우를 만들어 봅시다. 구체적인 작업은 나중에 해도 늦지 않으니까요!

첫 번째로 믹스인 파일(js)을 만들어 주세요.

```jsx
// @/mixin/MyMixin.js

let MyMixin = {
	mounted() {
		console.log('Mixed!')
	}
}
export default MyMixin

```

그 다음, 사용할 컴포넌트에 만든 믹스인을 불러와 주세요.

```jsx
// @/component. js
import MyMixin from 'mixins/MyMixin'
export default {
	mixins: [MyMixin] // mixin"s" 입니다! s에 주의하세요! 에러메세지 없이 동작하지 않습니다.
}
```

그 다음 브라우저로 돌아가면 component.vue의 mounted 훅에 정상적으로 콘솔에 메세지가 출력되는 것을 확인할 수 있습니다 🎉

### 실제 사용 예시

믹스인을 공부하고 포스트를 작성하게 된 계기이기도 한 케이스입니다.

아래 예시는 여러 개의 엘리먼트가 순차적으로 트랜지션하는 애니메이션을 가진 컴포넌트가 반복등장하는 경우에 믹스인을 활용해 애니메이션을 구현한 경우입니다. 

![믹스인을 이용한 스태거 애니메이션](/assets/posts/blueStragglr/vue-mixin/stagger.gif)

작성한 믹스인 객체는 아래와 같은 모양입니다.

```jsx
// ListStagger.js
let ListStagger = {
  data () {
    return {
      animationDelay: 200
    }
  },
  mounted () {
		// anime.js 라이브러리 사용
		// main.js에 전역으로 import하였음. (Vue.
    this.$anime({
      targets: '.stagger-list',
      translateY: [20, 0],
      opacity: [0, 1],
      delay: this.$anime.stagger(100, { start: this.animationDelay + 800 }) // increase delay by 100ms for each elements.
    });
  }
};
export default ListStagger
```

기본적인 vue 파일의 export default 뒤에 오는 객체와 같은 모양이지만, 변수 형태로 선언하고 해당 변수를 ES6+ 문법을 이용해 export하도록 작성하였습니다. 

불러온 컴포넌트에서 클래스명이 `stagger-list` 인 모든 컴포넌트를 감지하여, staggered animation을 줄 수 있도록 작성하였습니다.

이 때, mounted 대신에 created를 사용하면 해당 기능은 정상적으로 동작하지 않는다는 것에 주의하세요! 믹스인을 사용하는 경우, 라이프사이클 훅은 일반 컴포넌트의 라이프사이클 훅과 똑같이 동작합니다. 즉, 일반 컴포넌트에서도 마찬가지지만 위 경우에 created를 사용하게 되면 DOM트리가 생성되기 전에 CSS Query (class query)로 객체를 찾으려는 시도를 하는 셈이 되어 아무 일도 일어나지 않습니다. 

### 무엇이 개선되었나요?

믹스인을 통해 코드를 정리한 부분을 조금 더 시각화해 보면 아래와 같습니다. 우선, 믹스인을 사용하지 않는다고 하면 코드는 아래와 같은 모양이 됩니다. 

![기존의 코드 구조](/assets/posts/blueStragglr/vue-mixin/2.png)

이러한 구조를 믹스인을 활용함으로써 다음과 같이 변경했습니다. 

![수정된 구조](/assets/posts/blueStragglr/vue-mixin/3.png)

같은 코드를 반복하지 않게 되고, 특정 '기능'을 나타내는 파일을 캡슐화 할 수 있게 되었습니다. 단순히 코드의 전체 줄 수가 줄어든 것이 아니라, 유지보수 및 협업 측면에서 훌륭한 구조를 갖추게 된 것으로 볼 수 있습니다. 

> 직접적인 예시로써 애니메이션의 전체적인 시간을 조정하고 싶은 경우를 상상해 보세요. 모든 파일을 탐색하며 애니메이션의 수치를 변경하는 것 보다는 믹스인 파일 하나의 값을 조정하는게 훨씬 생산적입니다. 제3자가 이해하기도 좋구요!