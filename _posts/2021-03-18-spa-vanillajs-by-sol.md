---
layout: post
title: "[Front-end] SPA란 무엇이고 어떻게 구현할까"
description: "SPA의 개념 및 배경, 그리고 Vanilla JS로 구현해보자."
categories: 
  - Front-end
tags: ["Dev. Sol", "Javascript", "Vanilla JS", "FE", "Front-end"]
author: "Dev. Sol"
---


# SPA(Single Page Application)

## SPA의 등장 배경
모바일 시대가 대두됨에 따라 렌더링 방식은 새로운 국면을 맞이했다. 전통적인 서버사이드 렌더링(SSR) 방식(ASP, PHP, JSP)에서 클라이언트사이드 렌더링으로 렌더링 방식의 변화를 맞이한 것이다.

기존에는 이미 고정된 페이지를 리턴하므로 모바일 페이지를 만들기 위해서는 모바일 사이트를 별도로 만들어야 했다. 즉, 웹용 사이트, 모바일용 사이트 이렇게 두 개를 만들어야 했다. 이 형태도 아이폰과 안드로이드에서 웹앱 형태로는 개발할 수 있지만 네이티브 형태로는 만들기가 불가능해졌다. 그래서 서버는 REST api만 제공해주고 클라이언트가 json 데이터를 렌더링해서 만드는 방식이 사용되는 SPA(Single Page Application)가 새로운 트렌드가 된다(개념은 오래전부터 있었던 것으로 보임).

SPA는 마크업과 데이터를 요청하고, 브라우저에 페이지를 바로 그린다. 이런 것들을 가능하게 해주는 대표적인 JavaScript Framework 에는 Angular, React, Vue.js가 있다.

## SPA란?

Single Page Application의 약자로 말 그대로 단일 페이지로 구성된 웹 어플리케이션이다.<br>
일반 페이지 같은 경우, Client가 Server에게 html, css, javascript 등의 화면을 보여줄 수 있는 구셩요소들을 모두 요청하고<br>
서버는 요청된 파일을 Client에게 전달해주는 형태이다.

SPA의 경우 Client가 Javascript를 이용하여 필요한 데이터만 서버로부터 JSON으로 전달 받아 동적으로 렌더링한다.<br>
기존 어플리케이션은 화면이동 시에 화면 이동에 필요한 HTML을 서버사이드에서 받아서 처음부터 다시 로딩하기 때문에 시간이 걸린다. 반면, SPA에서는 화면 구성에 필요한 모든 HTML을 클라이언트가 갖고 있고 서버사이드에는 필요한 데이터를 요청하고 JSON으로 받기 때문에 기존의 어플리케이션에 비해 화면을 구성하는 속도가 빠르다.

<img src='/assets/images/sol/20210318/best-single-page-applications.jpg'>
출처 : <a href='https://www.excellentwebworld.com/what-is-a-single-page-application'>https://www.excellentwebworld.com/what-is-a-single-page-application</a>

### 장점

> - 하나하나 화면 전체를 렌더링할 필요가 없기 때문에 화면 이동이 빠르며 성능이 좋다
> - Server의 탬플릿 연산을 Client로 분산하여 성능을 높일 수 있다.
> - 자연스러운 사용자 경험(UX)을 느낄 수 있다. 특히 모바일에서는 네이티브 앱과 유사한 경험을 할 수 있다.

### 단점

> - JavaScript 파일을 번들링해서 한 번에 받기 때문에 초기 구동 속도 느림(webpack 의 code splitting으로 해결 가능)
> - 검색엔진최적화(SEO)가 어려움 (SSR 로 해결 가능)
> - 보안 이슈 (프론트엔드에 비즈니스 로직 최소화해야함)

## SPA의 핵심, 라우팅
### 라우팅이란?
라우팅이란 출발지에서 목적지까지의 경로를 결정하는 기능이다. 사용자가 A 라는 화면에서 B 라는 화면으로 넘어가는 네비게이션을 관리하기 위한 기능을 의미한다.

#### 전통적인 링크 방식
```html
<a href="index.html">index</a>
```
링크태그를 이용하는 방식이다.

링크를 클릭하게 되면 리소스의 경로가 URL의 path에 저장이되고, 서버는 해당 html을 클라이언트에 응답한다.
이를 서버 사이드 렌더링(SSR)이라 한다.

장점
> - 페이지마다 URL이 존재하여 history 관리 및 SEO 대응에 문제가 없다.

단점
> - 전체페이지를 로딩하므로 변경이 불필요한 요소도 로딩된다.
> - 복잡한 웹페이지라면 중복된 html, css, javascript를 계속 다운로드해야하므로 속도가 저하된다.

#### AJAX 방식
```html
<a id="index">index</a>
```
AJAX(Asynchronous JavaScript and XML)는 전통적 링크방식에서 개선한 방식으로 자바스크립트를 이용해 비동기적으로 서버와 브라우저가 데이터를 교환하는 방식이다.

href 를 사용하지 않고, AJAX 를 통해 서버에 필요한 리소스를 요청하고 응답을 하면 html 내의 내용을 갈아치운다.

장점
> - 필요한 부분만 렌더링을 하므로 기존의 방식보다 속도가 빠르다.

단점
> - URL 의 변경이 없으므로 앞으로가기, 뒤로가기 등의 history 관리가 되지 않는다.
> - URL이 하나이므로 SEO 이슈에 대해서도 자유로울 수 없다.

#### Hash 방식
```html
<a href="#index">index</a>
```
Hash 방식은 URI 의 fragment identifier (예: #index )의 고유기능인 앵커(anchor) 를 사용한다. fragment identifier는 URI 에서 # 으로 시작하는 문자열인데 hash 또는 hash mark 라고 불린다.

href 어트리뷰트에 hash 를 사용한다. index를 누르게 되면 hash 가 추가된 URI가 표시된다. hash 가 추가되면 URL 이 동일한 상태에서 URI 만 바뀌므로 서버에 어떤 요청도 하지않는다. 다시말하면, hash 가 변경되어도 요청이 보내지지않으므로 페이지의 새로고침이 발생하지 않는다.

hash 는 페이지가 새로고침이 되지않지만 논리적으로 페이지를 분리한다. 그렇기 때문에 히스토리 관리가 되는것이다.
hash 방식은 hashchange 이벤트를 통해 hash 값 변경을 통해 AJAX 요청을 수행한다.

장점
> - 기존의 AJAX 방식에서 히스토리 관리가 안됐다면 hash 방식은 히스토리 관리가 된다.

단점
> - 여전히 SEO 이슈이다. 크롤러는 javascript 를 실행시키지 않기 때문에 hash 방식으로 만들어진 콘텐츠를 수집할 수 없다.

#### PJAX 방식
```html
<a href="/index">index</a>
```
큰 단점인 SEO이슈를 보완한 방법이 PJAX(pushstate + ajax) 이다. window.location의 pushstate 와 popstate 를 사용한다. 하지만 두 메서드는 IE10 이상에서 작동한다.

hash 방식에서는 URL이 변경되지않아 SEO 가 문제가 되지만
PJAX 는 페이지마다 고유의 URL이 존재하기 때문에 아무 문제가 없다.

## SPA 구현해보기
위에서 언급한 SPA를 가능하게 해주는 대표적인 Framework들 또한 쉽게 라우팅을 해줄 수 있는 라우터 컴포넌트들을 제공하고 있다. 그러나 원리를 이해하기 위해 직접 Vanilla JS로 구현해볼 것이다.

라우팅 방식은 Hash 방식을 이용했다.

폴더구조는 아래와 같다.
<img src="/assets/images/sol/20210318/vanilla-js-router-folder-structure.PNG">

먼저 entry point인 index.html을 보자.
```html
<!DOCTYPE html>
<html>
  <head>
    <title>Canvas</title>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="stylesheet" type="text/css" href="style.css">
  </head>
  <body>
    <script type="module" src="index.js"></script>
  </body>
</html>
```
index.html에서는 css와 index.js만 사용한다.

다음은 index.js다.
```javascript
import {Router} from './js/router.js';
import Header from './js/layout/header.js';
class App {
    constructor() {
        // header
        new Header();

        // main
        let div = document.createElement('div');
        div.setAttribute('id', 'root');
        document.body.appendChild(div);

        // router
        new Router();
    }
}


window.onload = () => {
    new App();
}

```
Header를 만들고 main 페이지들이 들어갈 div영역을 만들어준다. 그리고 라우터를 생성해준다.

router.js는 아래와같다.
```javascript
import BallCanvas from './canvas/canvas1/ballCanvas.js';
import WaveCanvas from './canvas/canvas2/waveCanvas.js';
import GlowCanvas from './canvas/canvas3/glowCanvas.js';

export const routes = [
    {
        hash: 'canvas1',
        pageName : 'Ball Canvas',
        component: new BallCanvas()
    },
    {
        hash: 'canvas2',
        pageName : 'Wave Canvas',
        component: new WaveCanvas()
    },
    {
        hash: 'canvas3',
        pageName : 'Glow Canvas',
        component: new GlowCanvas()
    },
]

export class Router {
    constructor() {
        this.page = null;
        
        this.render.bind(this);

        this.render();
        window.addEventListener('hashchange', this.render);
        window.addEventListener('DOMContentLoaded', this.render);
    }

    async render() {
        try {
            const root = document.getElementById('root');
			const hash = location.hash.replace('#', '').split('?')[0];
            if (hash === '') {
                root.innerHTML = '';
                routes[0].component.render(root);
                return;        
            }
            const { pageName, component} = routes.find(route => route.hash === hash);
            
            if (!pageName) {
                return;
            }
            root.innerHTML = '';

            component.render(root);
			
		} catch (err) {
			console.error(err);
		}
    }
}
```

라우터의 핵심 코드는 이것이다.
```javascript
window.addEventListener('hashchange', this.render);
window.addEventListener('DOMContentLoaded', this.render);
```
hash가 change되거나 DOMContent가 로드되고 난 뒤 render 함수를 불러서 routes에서 hash와 일치하는 아이템을 찾아 main 페이지를 다시 렌더링하는 것이다.

Header에서는 아래와 같이 routes의 item에 따라 메뉴 리스트를 구성한다.
```javascript
import {routes} from '../router.js';

export default class Header {
    constructor() {
        document.body.innerHTML = `<div class='header-wrapper'><ul class='menu-wrapper'></ul><div>`;
        const ul = document.getElementsByClassName('menu-wrapper');

        routes.forEach(route => {
            const li = document.createElement('li');
            const a = document.createElement('a');
            a.setAttribute('href', `#${route.hash}`);
            a.innerText = route.pageName;
            li.appendChild(a);
            ul[0].appendChild(li);
        });
    }
}
```

참고로 각 컴포넌트들은 Interative Developer님의 Canvas Practice 클론 코딩을 했던 내용들이다.

위 소스는 <a href='https://github.com/hsolemio-lee/vanilla-js-router'>vanilla-js-router</a>에서 확인해볼 수 있으며,<br>
구현된 모습은 <a href='https://hsolemio-lee.github.io/vanilla-js-router'>여기</a>에서 확인할 수 있다.

구현된 소스를 보면 비록 서버에 요청하는 항목들은 없지만, 위와 같이 각 component들 안에서 각각이 필요한 리소스를 요청하여 필요한 부분만 업데이트할 수 있을 것이다.

## 정리하며
최근 웹은 대부분 SPA로 만들어지고 있으며 앞으로도 새로운 패러다임이 나오기 전까지는 계속해서 사용될 것으로 보인다. 그렇기 때문에 웹 개발자라면 FE 개발자가 아니더라도 SPA를 이해하고 있어야 심도있는 개발을 할 수 있을 것이라고 생각한다.<br>
그리고 FE 개발자라면 아마도 앞으로 SPA의 단점들을 커버할 수 있는 내용들에 대해 더욱 깊이있는 공부가 필요할 것이다.