---
layout: post
title: "SVG 활용: HTML상에 벡터 이미지 삽입하기"
author: bluestragglr
date: 2019-11-11 00:00
tags: [svg, HTML/CSS]
image: 'https://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2017/12/1498798784svg101.svg'
---


> SVG 활용: HTML상에 벡터 이미지 삽입하기 by 신성환(github.com/blueStragglr)

---



HTML에서는 벡터 이미지를 렌더링 할 때 각 포인트에 대한 값 형식인 SVG 파일을 전달받아 렌더링을 수행할 수 있습니다. 일반 이미지 형식의 경우, 안티앨리어싱 등의 문제로 자칫 잘못하면 2000년대 양산형 홈페이지같은 느낌을 줄 수도 있으므로, 반응형으로 간단한 형태의 이미지를 활용하여야 하는 경우에는 SVG를 사용하는 것을 고려해볼 만 합니다. 

SVG파일을 직접 작성하기 위해서는 벡터 패스의 좌표값들과 캔버스 크기, 색깔 등의 속성을 지정해주어야 합니다. 하지만 모든 좌표값을 일일히 입력하는건 굉장히 비생산적인 일이고, 쉽지도 않습니다. 추천하는 방법은 SVG파일을 다운로드 받거나 일러스트레이터에서 직접 SVG를 export하여 사용하는 것입니다.  

본 문서에서는 SVG파일을 이용해서 hover 속성을 추가하는 예제를 다루어 보겠습니다. 

1. SVG 파일 구하기
   필요한 SVG파일을 다운로드 받거나, 어도비 일러스트레이터에 이미지를 그린 다음 export 탭에서 file format을 SVG로 선택한 후 내보냅니다. 
2. SVG 파일 열기
   사용하는 IDE를 이용해 SVG파일을 엽니다. 벡터 패스 등의 내용이 줄넘김 없이 한 줄로 작성되어 있어 자칫하면 코드인줄 모르고 넘어갈 수 있습니다. 
3. 코드 복사해 오기 및 정리하기
   원래의 이미지 태그가 있던 자리에 SVG 태그를 붙여넣기합니다. Auto formatter 등으로 보기 좋게 줄을 나누어도 문제가 발생하지 않습니다. 

```html
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 78.08 16">
    <defs>
        <style>
            .cls-1{fill:#fff;}
            .cls-2{fill:none;stroke:#514b4b;stroke-miterlimit:10;stroke-width:2px;}
            .cls-3{fill:#514b4b;}
        </style>
    </defs>
    <title>arrow_test</title>
    <g id="레이어_2" data-name="레이어 2">
        <g id="레이어_3" data-name="레이어 3">
            <g id="레이어_1-2" data-name="레이어 1">
                <rect class="cls-1" x="62.08" width="16" height="16"/>
                <line class="cls-2" y1="7.99" x2="51.09" y2="7.99"/>
                <polygon class="cls-3" points="54.08 7.89 41.24 0.48 41.24 15.3 54.08 7.89"/>
            </g>
        </g>
    </g>
</svg>
```

​	코드를 살펴보면 스타일에 대한 정의가 포함된 <defs> 영역과 패스 정보 등을 담고 있는 <g> 영역이 있습니다. 스타일 영역은
​	SVG 영역 바깥에서도 적용할 수 있습니다. 
​	이 때, 여러 개의 SVG이미지를 사용하는 경우 클래스 이름이 모두 cls-1, cls-2 등으로 겹칠 우려가 있습니다. 여러 이미지를 사용 
​	해야 하는 경우라면, 자동으로 생성된 클래스 이름을 변경하도록 합시다.



4. :hover 등의 클래스 적용하기
   이제 style 영역의 코드를 옮긴 뒤, :hover등의 css 선택자를 추가로 생성하거나 스타일을 변경할 수 있습니다. 

   ```css
   <style>
   .cls-1 {
     fill: #fff;
   }
   
   .cls-2 {
     fill: none;
     stroke: #514b4b;
     stroke-miterlimit: 10;
     stroke-width: 2px;
   }
   
   .cls-3 {
     fill: #514b4b;
   }
   
   /********* 추가된 스타일 ***********/
   .cls-1,
   .cls-2,
   .cls-3{
     transition: .5s ease;
   }
   .cls-1:hover,
   .cls-2:hover,
   .cls-3:hover {
     fill: #f3bcaa;
   }
   </style>
   ```

   

   이후 위 코드를 띄워 보면, 아래와 같은 결과를 얻을 수 있습니다.  화살표의 삼각형 부분과, 오른쪽에 있는 흰색 사각형에 마우스를hover하였을 때 자연스럽게 색상이 변하는 것을 확인할 수 있습니다. 

   ![svg-hover](/assets/images/svg-hover.gif)

   

   SVG를 이용하면 이미지 비율에 따른 안티앨리어싱 관련 문제를 해결할 수 있고, 자연스러운 방식으로 interaction을 구현할 수 있습니다. 예를 들어, 이미지 모양의 버튼이 있고 hover되었을 때 특정 영역의 색깔만 바뀌게 하고 싶다고 했을 때, 이미지 두 장을 이용해 구현하는 것은 쉽지 않습니다. 하지만 SVG를 활용하면 HTML 구조를 비틀지 않고(즉, CSS 트릭을 사용하지 않고) 자연스럽고 다양한 동작을 수행하도록 할 수 있습니다.

