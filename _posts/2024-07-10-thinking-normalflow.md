---
created: 2024-07-10
title: "[CSS] 노말 플로우로 사고하기"
description: 노말플로우를 통해 브라우저에서 레이아웃이 배치되는 원리를 이해하고 BFC, IFC에 대해서 알아보자
author: "Jiheon"
categories: [Nomal-Flow, BFC, IFC, margin-collapse]
tags: [Nomal-Flow, BFC, IFC, margin-collapse]
layout: post
image:
  path: /assets/images/2024-07-10-thinking-normalflow/thumbnail.webp
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Responsive rendering of Chirpy theme on multiple devices.
---


### 💡 시작에 앞서 

예전에 `CSS`를 공부하다가 마진병합 현상에 대해서 알게되었는데 그 과정에서 `normal flow`, `BFC` 등 생소한 내용이 등장하면서 혼란스러웠던 적이 있었다 

다소 추상적일 수 있는 내용이라 쉽지 않지만 `HTML`문서에서 요소들의 레이아웃 배치와 정렬, 상호작용 등을 설명하는 이러한 노말 플로우의 동작을 이해하고 정리하려고 한다. 😓


---
### 💡 노말 플로우 (Normal Flow)

노말 플로우는 브라우저가 레이아웃을 배치하는 가장 기본적인 배치 방식을 의미한다. 즉, 요소들을 화면에 어떻게 배치할지 결정하는 기본 방식인 것 

- `div` 태그는 블록 요소이기 때문에 부모의 너비를 상속받고 화면을 꽉채운다 
- `span` 태그는 인라인 요소이기 때문에 자신의 크기만큼의 너비를 갖는다

뭔가 노말 플로우라는 단어 자체가 낯설게 느껴지지만 위의 `div`, `span` 태그의 기본 동작처럼 우리가 평상시에 자연스럽게 받아들이는 레이아웃의 기본 규칙을 의미하는데 `Normal Flow` 라는 말 그대로 자연스러운 흐름 이라고 생각하면 된다.   

---
### 💡 Formatting Context (*FC) 

`Formatting Context`란 요소들이 어떻게 배치되고 정렬되는지를 결정하는 규칙과 메커니즘이 적용되는 영역을 의미하는데 블록 레벨의 요소라면 `BFC`에 의해서, 인라인 레벨의 요소라면 `IFC`에 의해서 배치된다. 


실제 스펙은 [W3C CSS2.1](https://www.w3.org/TR/CSS2/visuren.html#normal-flow)에서 정의 되어있는데 `float`, `overflow`, `마진병합`, `line box`등 어려운 얘기가 많이 나오고 복잡하지만 결국은 우리가 알고 있는 것 처럼 블록 요소는 수직으로 배치되고 텍스트나 인라인 요소들은 수평으로 배치된다는 얘기를 하고있다.  


![](https://velog.velcdn.com/images/heonys/post/287150d3-cb5f-4e63-b906-3e9d505addd3/image.png)

 따라서 지금까지 당연하다고 생각되던 요소들의 기본 동작들은 이러한 노말 플로우의 `BFC`, `IFC`에 의해서 레이아웃이 배치되고 있었던 것 

>앞서 말했듯이 노말 플로우는 실제 스펙상으로 훨씬 어렵고 복잡한 규칙인데 이러한 `BFC`, `IFC`에 대해서 조금 더 자세히 알아보도록 하자. 

---
### 💡 BFC (Block Formatting Context)

문서의 최상위 요소인 `HTML` 요소는 특수하게도 `BFC` 영역을 생성하는데 따라서 `body`요소를 비롯해서 내부의 자식 요소들은 기본적으로는 상단으로부터 수직으로 배치되어 웹페이지의 기본 레이아웃을 유지하고 하나의 독립된 컨텍스트에서 예측 가능하고 일관되게 배치된다. 

하지만 `BFC`는 `CSS` 속성을 통해서 의도적으로 생성하는게 가능한데 [MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_flow_layout/Introduction_to_formatting_contexts) 문서에 따르면 새로운 `BFC`가 생성되는 경우는 아래와 같다 

- 문서의 최상위 요소인 `HTML` 
- `float` 속성을 기본값인 `none`이 아닌 속성으로 플로팅 했을 때
- `position` 속성을 `absolute`, `fixed`로 해서 노말 플로우에서 벗어났을 때  
- `overflow` 속성이 `hidden` 또는 `scroll`일 때
- `display` 속성이 `inline-block` 또는 `flow-root` 일때 
- `...` 이외에도 더 많은 경우가 있음 

새로운 `BFC`를 만드는 방법은 이처럼 여러 가지가 있지만, 이 글에서는 예시 코드를 작성할 때 `display: flow-root` 속성을 사용할 것이다. 

```css
.selector  {
    display: flow-root /* 새로운 BFC 생성 */
 }
```
주로 `overflow: hidden` 속성도 많이 사용되던데 `overflow` 속성의 원래의 목적과 다른 사용일 뿐더러 사이드 이펙트가 발생할 수 있어 주의해야 할 필요가 있다. 그래서 상황에 맞게 적절한 방법을 선택하면 될 것 같고 `flow-root`는 다른 방법들과 다르게 사이드 이펙트 없이 단순히 `BFC`를 만들기 때문에 이 글에서는 `flow-root`를 선택했다

>`BFC`와 관련해서 레퍼런스를 찾아보고 공부 하면서도 뭔가 크게 와닿지 않는 느낌이 들어서 의문점을 정리해 보고 나름대로의 해답을 찾아보려고 했다. 


#### 🚩 내가 div 태그를 만들면 새로운 `BFC`를 생성 하는걸까? 
그건 아니다 `div` 태그는 블록 요소는 맞지만 단순히 `BFC` 안에서 배치될 뿐 위에서 언급한 것처럼 특정 상황에서만 새로운 `BFC`를 생성한다.

#### 🚩 새로운 `BFC`를 만든다는 것은 무슨 의미인데? 

모든 요소들은 최상위 요소인 `HTML` 요소에 의해서 만들어진 `BFC`의 영향을 받는다. 하지만 새로운 `BFC`를 만든다는 것은 해당 요소와 그 자식 요소들이 배치되는 독립적인 영역을 생성하는 것을 의미하는데 같은 `BFC`일지라도 서로 다른 `BFC` 영역으로 배치되어 `CSS`의 레이아웃 모델을 더욱 정교하게 제어할 수 있다. (대표적으로 플로팅 문제와 마진 병합으로 발생하는 문제를 해결 할 수 있다) 

#### 🚩 BFC는 중첩될 수 있을까? 
중첩될 수 있다. 하지만 결국은 독립적인 영역을 만들기 때문에 `BFC` 라는 같은 범주에 있지만 결국 외부와 서로 다른 레이아웃 영역이 된다. 

#### 🚩 그래서 BFC의 중요성은 무엇인데? 

`BFC`는 브라우저에서 블록 레벨의 요소들이 배치되는 방식을 규정하는 영역을 의미한다 따라서 `BFC`를 이해하고 적절히 활용하는 것은 레이아웃이 배치되는 원리를 이해하고 더욱 정교하며 예측 가능한 설계를 만드는 데 도움이 될 수 있다.

---
### 💡 플로팅 (floating) 

`CSS`의 `float` 속성은 [MDN float](https://developer.mozilla.org/ko/docs/Web/CSS/float) 스펙에 의하면 노말 플로우에서 벗어나 텍스트나 인라인 요소들이 자신을 감싼다고 표현하고 있다. 정리하자면 
>- 새로운 `BFC`를 만든다 (`BFC`의 생성 조건에서 확인 가능)
- 노말 플로우를 벗어나 위에 떠오른다 `(floating)`
- 텍스트와 인라인 요소에 대한 경계면을 만든다 

`float` 속성은 요즘에는 잘 사용하지 않지만 텍스트 래핑 등을 구현할 때 사용될 수 있는데 중요한 건 노말 플로우에서 벗어나 위에 떠버리기 때문에 일반적인 문서의 흐름에서 벗어나 주변 요소들과 상호작용이 되지 않는 문제가 발생한다. 

![](https://velog.velcdn.com/images/heonys/post/9e5fb516-12ca-4d18-b053-ef71aee0c702/image.webp)

```css
.container  {
    background-color: green;
 }
.container  div {
    float: left;
    margin: 10px;
    background-color: lightgreen;
 }
```

```html 
<div class="container">
  <div>Sibling</div>
  <div>Sibling</div>
</div>
```


위의 코드는 오른쪽 그림과 같은 결과를 기대했지만 `float`된 요소가 노말 플로우에서 벗어났고 부모는 명시적으로 높이를 지정해 주지 않았기 때문에 높이를 잃어버려서 왼쪽 그림과 같이 렌더링된다

이런 문제를 해결하기 위해 `BFC`를 사용할 수 있는데 `BFC`는 자신이 포함하고 있는 `float`된 요소들을 포함하여 다른 컨텐츠의 높이와 동일하게 만든다는 특징이있다 

```css
.container  {
    display: flow-root; /* 새로운 BFC 생성 */
    background-color: green;
 }
```
따라서 `display: flow-root`을 비롯한 새로운 `BFC`가 생성되도록 컨테이너에 속성을 추가하면 높이를 잃어버리지 않고 의도한 대로 렌더링된다 

>플로팅으로 인한 높이를 갖지않는 문제는 `clear`속성을 사용해서 플로팅된 요소가 다른 요소와 겹치지 않도록 해서 해결하는 방법도 있다. 

---
### 💡 마진병합 (margin collapse)

[마진병합](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_box_model/Mastering_margin_collapsing) 이란 인접한 블록요소 사이의 마진이 병합되어 더 큰 마진으로 합쳐지는 현상을 의미한다. 상식적으로라면 둘의 마진의 합 만큼 적용되어야 할 것 같지만 `CSS`는 의도적으로 이렇게 설계되어 문서의 안정성을 높이도록 동작한다. 

![](https://velog.velcdn.com/images/heonys/post/539c8e68-0eff-4859-9ebe-7f1364318c68/image.webp)


>이런 `CSS`의 기본동작 때문에 때로는 의도치 않은 레이아웃이 구현될 수 있는데 마진병합 현상을 막을 수 있는 방법을 알아보자 

#### [MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_box_model/Mastering_margin_collapsing) 문서에 따르면 마진병합의 발생 조건을 아래와 같이 정의 하고있다

- 인접한 형제 요소간 발생 
- 부모 자식 사이에 인라인 요소가 없고 부모에 `border`, `padding` 같은 스타일로 여백이 생기지 않으며, `BFC`가 생성되지 않은경우 부모 자식간 발생 



발생 조건에서 볼 수 있듯이 새로운 `BFC`생성이 없어야한다는 것은 결국 부모와 자식이 동일한 `BFC`내에 존재할 때 마진병합이 발생한다는 의미이고 따라서 마진병합이 일어나지 않도록 의도한다면 테두리, 패딩 등의 속성으로 인접한 요소 사이에 공간을 만들던지 아니면 새로운 `BFC`를 생성해서 인접한 블록을 서로 다른 `BFC`에 속하도록 하면 마진 병합을 막을 수 있다.


```css
 .parent {
   background-color: tomato;
   display: flow-root
 }

 .child {
   margin: 1rem;
   background-color: cornsilk;
 }
 
 .bfc {
   display: flow-root
 }
```

```html
 <div class="parent">
   <div class="child">child1</div>
   <div class="bfc"> 
     <div class="child">child2</div>
   </div>
 </div>
```
위의 코드에서 `parent`는 새로운 `BFC`의 생성으로 부모 자식 간의 마진병합이 발생하지 않으며 `.bfc` 클래스로 인해 새로운 `BFC`를 생성해서 `child1`과 `child2`는 각각 서로 다른 `BFC`에 속해있기 때문에 `child`간의 마진 병합 역시 발생하지 않는다. 

---
### 💡 IFC (Inline Formatting Context)

`IFC`는 인라인 레벨의 요소들이 배치되는 영역으로 텍스트나 인라인 요소들이 수평으로 배치되며 `BFC`와는 다르게 `a`태그나 `span` 태그와 같은 인라인 요소를 만들면 자동으로 새로운 `IFC`가 생성된다. 

#### ⭐ 라인박스 (Line Box)

[라인박스](https://drafts.csswg.org/css-inline/#line-box)는 여러 인라인 요소들이 수평으로 배치될 때, 이들이 결합된 한 줄의 박스 영역을 의미한다 그래서 라인박스의 높이는 라인박스에 포함된 가장 큰 인라인 요소의 높이에 맞춰지며 라인박스의 폭을 초과하면 새로운 라인박스가 만들어져서 수직으로 쌓인다

- `line-height` 속성은 라인박스의 최소 높이를 설정한다  
- `vertical-align` 속성은 라인박스의 범위를 기준으로 수직 정렬한다 

![](https://velog.velcdn.com/images/heonys/post/3af8c394-cd8f-4ade-8427-63546bb08b03/image.png)

>`IFC`랑 라인박스에 대한 관계를 내가 이해한 것을 바탕으로 직접 그림을 그려서 표현 해봤다. `IFC`는 인라인 요소들의 배치 영역을 정의하며, 인라인 요소들은 라인 박스 안에서 정렬된다. 

#### ⭐ IFC와 BFC의 중첩 


`BFC` 내부에는 블록 요소뿐만 아니라 인라인 요소도 포함될 수 있는데 이런 인라인 요소들은 자기들끼리 `IFC` 영역을 형성한다. 하지만 반대로 `IFC` 내부에서 블록 요소를 추가하게 되면 해당 `IFC`는 그대로 종료되어 닫히고 상위 `BFC`영역에 이어서 블록 요소가 배치된다. 따라서 `BFC`내부에는 `IFC`가 존재할 수 있지만 반대의 경우는 불가능하다

 
#### ⭐ 추가적으로 
`IFC`에 대해서 관련된 내용을 더 찾다 보면 `font metrics`, `content-area` 등 굉장히 어려운 내용들이 나오던데 기회가 된다면 공부해서 나중에 관련된 글을 써보고 싶다. 


---
### 💡 CSS: Position 
`CSS`에 요소를 배치하는 방식과 그 위치를 결정하는 `Position` 속성은 노말 플로우와 깊은 연관이있는데 노말 플로우는 `Position` 속성이 오직 `static`과 `relative`, `sticky` 일때만 적용된다. ([MDN - position](https://developer.mozilla.org/ko/docs/Web/CSS/position) 에서 확인 가능)

따라서 `absolute`, `fixed` 으로 포지션을 설정하면 노말 플로우에서 벗어나기 때문에 기존의 노말 플로우에서 벗어나 자유로운 배치를 할 수 있게되는 특징이 있다. 

>
- `static` : 노말 플로우에 의해 배치되는 기본값 
- `relative` : 노말 플로우에 의해 배치된 이후 상대적으로 위치 조정 
-  `absolute` : 노말 플로우에서 벗어나 `offset parent`로 부터 위치 조정
-  `fixed` : 항상 브라우저의 뷰포트를 기준으로 위치 고정  
- `sticky`: 노말 플로우에 의해 배치되며 특정 위치에 도착 후 고정 

`span` 태그와 같은 인라인 요소에 `absolute`로 포지셔닝 하면 `BFC` 생성 조건에 따라서 새로운 `BFC`를 만들게 되는데, 재밌는 건 `span`태그는 원래 인라인 요소지만 `display` 속성을 확인 해보면 `block`으로 속성이 바뀌었으며, 블록 요소임에도 불구하고 명시적으로 너비와 높이를 지정하지 않으면 마치 인라인 요소처럼 자신의 컨텐츠 만큼의 크기를 갖는다. 


#### 🚩 같지만 다른 BFC 


1. `position: absolute`는 기존의 노말 플로우에서 벗어남
2. `BFC` 생성조건에 의해 `absolute`는 새로운 `BFC`를 생성함 
3. 새로 생긴 `BFC`영역은 기존의 `BFC`의 규칙과는 다르게 동작함 

즉, `absolute`로 인해 만들어진 `BFC` 영역은 기존의 `BFC`의 규칙과는 다르게 동작하는데 그럼에도 불구하고 왜 똑같이 `BFC`일까? 


비단 `absolute`만의 문제는 아니고 `float`, `overflow`, `display`  속성 등으로 각각 `BFC`를 생성하면 마진병합이 사라지는 `BFC`만의 특성이 나타나면서도 각 속성이 적용되는 것을 보면, 같은 `BFC` 일지라도 `BFC`의 특성은 유지하면서 유연하게 레이아웃 규칙이 바뀔 수 있고 여전히 블록 레벨 요소의 레이아웃 컨텍스트라는 관점에서는 부합하니까 그대로 `BFC`라고 하는게 아닐까 싶다 


---
### 💡 마무리 

주제 자체가 어려운 주제라서 많은 문서를 참조 하면서 정확하게 내용을 전달하려고 노력했는데 내용전달이 잘 됐는지 모르겠다. (아직도 어렵다 😅) 또한 브라우저가 레이아웃을 계산할 때 `BFC`, `IFC`의 규칙에 따라서 요소들이 배치될 뿐 단순히 문서의 구조를 나타내는 `DOM`트리의 포함관계와 레이아웃은 무관하다는 것을 기억하면 좋을 것 같다. 


### 📕 참고 문서
>
[W3C - Visual formatting model](https://www.w3.org/TR/CSS2/visuren.html)
[MDN - Block and inline layout in normal flow](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_flow_layout/Block_and_inline_layout_in_normal_flow)
[MDN - Block formatting context](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_display/Block_formatting_context)
[MDN - Mastering margin collapsing](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_box_model/Mastering_margin_collapsing)
[Understanding Block Formatting Contexts in CSS](https://www.sitepoint.com/understanding-block-formatting-contexts-in-css/)
[카카오 기술 블로그 -  Line Box](https://devblog.kakaostyle.com/ko/2019-04-29-1-vertical-align-line-box/)
