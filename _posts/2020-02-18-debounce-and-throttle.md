---
layout: post
title: "Debounce & Throttle: 효율적인 이벤트 핸들링"
author: bluestragglr
date: 2020-02-18 15:39
tags: [html, css, javascript]
image: 'https://blog.avarteq.com/wp-content/uploads/2016/02/Lodash.jpg'
---


> Debounce & Throttle: 효율적인 이벤트 핸들링 by 신성환(github.com/blueStragglr)

---

Debounce와 Throttle은 자바스크립트 이벤트를 (양적인 측면에서) 효율적으로 처리하기 위한 방법입니다.

이러한 컨트롤이 필요한 경우의 예시로 만들었던 페이지를 가져와 보겠습니다. 

![scroll-interaction-example](/assets/posts/blueStragglr/debounce-throttle/vertical-scroll.gif)

(퇴사 전에는 좀 더 잘 만들었었는데 애석하게도 유지보수가 잘 되고 있지 않네요.)

해당 페이지에서는 가로 혹은 세로 스크롤을 모두 가로 스크롤 값으로 변환하며, 가로 스크롤 값에 따라 각 사각형 컴포넌트의 위치와 각도를 조절합니다. 

이를 위해서 `window` DOM에 scroll eventListner가 달려 있으며, 아래와 같은 다양한 메소드가 구현되어 있었습니다.

1. 스크롤의 norm값을 계산하여 가로 스크롤 컨테이너의 가로 스크롤값을 업데이트
2. 업데이트할 norm값을 바탕으로 네 개의 오브젝트의 transform을 컨트롤
3. 스크롤의 최종값을 바탕으로 현재 페이지에 대한 표시 업데이트
4. 뒷 배경으로 나올 유성을 가리고 있던 컴포넌트의 상태 컨트롤

해당 페이지를 트랙패드를 이용해 스크롤하게 되면 페이지 전체를 탐색하는 동안 약 300번 가량의 이벤트가 호출되며, 결과적으로 위의 1~4를 300번 수행하게 됩니다. 하지만 2번은 전체 스크롤의 1/6에서만, 3번과 4번은 각 분기점에서만 호출되면 되는 메소드입니다. 대부분의 영역에서 불필요한 이벤트를 호출함으로써 리소스를 낭비하고 있는 것입니다. 실제로, 맥북 프로가 아닌 환경에서 가끔 버벅이는 현상을 관찰하기도 하였습니다. 

Debounce와 Throttle은 이러한 이벤트를 "그룹"으로 묶어 리소스의 사용을 줄일 수 있는 방법입니다. 

### Debounce

Debounce는 연속적으로 발생하는 이벤트를 그룹으로 묶어 fire할 수 있도록 해 주는 메소드입니다. 발생한 이벤트 시퀀스가 종료되었다고 판단되는 threshold 이후에 이벤트를 fire하도록 할 수 있습니다. 

Debounce를 이용하면 크게 두 가지 측면에서 이점이 발생합니다.

1. 클라이언트 리소스 절약
   클라이언트에서 처리할 메소드의 수가 확실하게 줄어듦으로, 유저에게 더 나은 서비스 경험을 선사할 수 있습니다. 같은 자원 대비 훨씬 부드러운 인터랙션을 가능하게 만듭니다.
2. 서버 리소스 및 비용 절감
   입력 등 연속적인 인터랙션에 대해 모든 순간에 서버와 통신하게 된다면 서버 입장에서도 불필요한 중간값을 돌려주기 위해 자원을 소모하게 됩니다. 만약 해당 중간값들을 만드는 데 유료 API 등을 사용해야 하는 경우라면 (Debounce를 적용하지 않는 경우와 대비하여) 비용적으로 큰 손해를 볼 수도 있습니다.

### Throttle

Throttle은 특정 주기를 정해 한 반복주기 내에서는 이벤트가 또다시 트리거되지 않도록 처리하는 메소드입니다. 주기를 조절할 수 있으며, 트위터에서 스크롤 관련 버벅임이 발생했을 때 존 레식이 해결책으로써 제안한 방법론이기도 합니다.

Throttle을 이용하면 무한 스크롤 페이지를 구현할 때 로딩 여부를 footer까지 남은 스크롤 거리 threshold를 설정함으로써 로딩 시간을 유저가 (최대한) 기다리지 않도록 처리하는 구조 등을 설계할 수 있습니다. 

### Usage example: Scroll Debounce

Vue.js에서 사용하기 위해서 window에 eventListener를 달았습니다. 

Debounce를 사용하기 위해 LoDash.js 라이브러리를 인스톨하고, 전역으로 import하였습니다.

    # installation
    $ npm i lodash
    
    // import at src/main.js
    import _ from 'lodash'
    
    Vue.prototype.$_ = _
    
    mounted () {
        window.addEventListener('scroll', () => {
          console.log(window.scrollY)
        })
        window.addEventListener('scroll', this.$_.debounce(() => {
          console.log('DEBOUNCE TRIGGERED!')
        }, 50))
      }

window 객체에 두 개의 scroll eventListener를 붙이고 하나에는 일반 함수를, 하나에는 debounce 메소드로 감싼 함수를 두었습니다.

실행 결과, 아래와 같이 일반 함수는 모든 스크롤 이벤트에서 동작하고 debounce 함수는 동작 정지 후 50ms 이후에 동작하는 것을 확인할 수 있습니다. 

![scroll-debounce-example](/assets/posts/blueStragglr/debounce-throttle/scroll_debounce.gif)

### Usage Example:  Scroll Throttle

이 메소드들을 찾아봤던 이유는 스크롤 높이에 따라 네비게이션 바의 모양을 결정하는 동작의 리소스를 효율적으로 관리하기 위함이었습니다. 하지만 안타깝게도, Debounce 메소드는 이 인터랙션을 구현하는데 적합하지 않았습니다. 스크롤 하는 동작에 의해 네비게이션 바가 줄어드는 느낌을 주고자 하였는데, 스크롤 이후에 동작이 진행되어 스크롤과 병렬적으로 인지되지 않았기 때문입니다. 

이를 해결하기 위해서 보다 적합한 방법인 Throttle을 적용해 보았습니다. Throttle을 사용하면 이벤트가 반복해서 실행될 최소 시간을 설정할 수 있습니다. 

    mounted () {
        window.addEventListener('scroll', () => {
          console.log(this.getScrolledPosition())
        })
        window.addEventListener('scroll',
    				this.$_.throttle(this.checkScrolledPositionThreshold, 50))
      },

이번에도 마찬가지로 window 객체에 두 개의 scroll eventListener를 붙이고 하나에는 일반 함수를, 하나에는 Throttle 메소드로 감싼 함수를 두었습니다.

실행 결과, Throttle 메소드로 감싼 함수가 약 1/3정도의 횟수로 실행되는 것을 확인할 수 있습니다.

![scroll-interaction-example](/assets/posts/blueStragglr/debounce-throttle/scroll_throttle.gif)

시각적으로 화면이 자연스럽게 전환되는 것으로 느껴지기 위한 초당 프레임 수는 30fps 정도입니다. 따라서 약 30초에 한 번만 이벤트를 수집하면 사람의 눈은 자연스러운 이벤트를 인지할 수 있습니다. 이벤트가 스타일에 직접 바인딩되는 것이 아니라, 클래스 토글이나 기능 활성화같이 기능적인 면과 연결된 경우에 유저 경험을 해치지 않기 위한 프레임 요구량은 더욱 낮아집니다.

두 가지 방법을 적절히 활용하여 리소스를 과도하게 소모하지 않는 멋진 인터랙션을 구현해 봅시다!

### Reference

[디바운스(Debounce)와 스로틀(Throttle ) 그리고 차이점](https://webclub.tistory.com/607)