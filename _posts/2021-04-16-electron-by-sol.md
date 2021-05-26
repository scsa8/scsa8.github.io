---
layout: post
title: 일렉트론(Electron)으로 데스크탑 어플리케이션을 만들어보자.
description: 웹의 시대, 그 중심에 있는 Javascript를 이용하여 데스크탑 어플리케이션을 만들어보자.
tags: 
    - Javascript
    - Electron
    - Dev. Sol
author: Dev. Sol
category: Javascript
---

오늘날 인기가 날로 치솟는 자바스크립트도 한때는 많은 개발자들이 javascript는 저질언어라고 폄하하며 언어의 축에도 껴주지 않던 시절이 있었다고 한다.<br>
현재는 전세계에서 가장 많은 개발자가 사용하는 프로그래밍 언어로 그만큼 영역의 제한도 없어지고 있다.<br>
오늘은 자바스크립트로 데스크톱 애플리케이션을 만들 수 있도록 도와주는 프레임워크, 일렉트론에 대해 알아보겠다.

# 일렉트론과의 조우
최근 현업에서 맞지않는 옷을 억지로 입히는 작업을 필요로 했다. 
> 간단하게 설명하면, 맡고 있는 솔루션은 전자제품향 오프라인 매장 솔루션인데 사업팀에서 F&B 매장에 우리 솔루션을 쓰고 싶단다.<br>
> 애초에 제품 성격이 다르기 때문에 계속해서 부정적인 의견을 전달했지만, 반드시 해야하는 사업이라니 어쩔 수 없이 지원해주게 되었다.ㅠㅠ<br>

어쨌든 성격이 다른 도메인의 제품을 가져와서 우리 쪽 제품으로 넣어야 했기에 변환하는 작업이 필요했다. 수동으로 한땀 한땀 넣기에는 너무 비효율적이고,
앞으로 사업팀에서 "나 없이" 계속 대응해주려면 뭔가가 필요할 것 같았고 간단한 쿼리 제너레이터를 만들어줘야겠다는 생각이 들었다.
그러려면 쓰기 편한 데스크톱 애플리케이션이 좋을 듯 한데, 파이썬으로 할까 하다가 다른 방법이 없는지 검색해보았다. 검색해보니 예전에 동기가 얘기했던
일렉트론이라는 프레임워크가 눈에 들어왔고 간단하게 만들기 좋을 것 같아 일렉트론으로 만들기로 정하였다.
결과적으로, 간단한 데스크톱 애플리케이션을 만들고자 한다면 최적의 프레임워크인 듯 하다.

# 일렉트론
## 개요
일렉트론은 Node.js를 기반으로 자바스크립트, HTML, CSS를 사용하여 데스크톱을 만드는 오픈소스 프레임워크이다.<br>
현재는 GitHub에 의해 개발되고 있으며 윈도우, 맥, 리눅스를 모두 지원한다.<br>
일렉트론으로 개발된 데스크톱 애플리케이션은 대표적으로 Visual Studio Code, Facebook Messenger, Twitch, Slack 등이 있다.

## 구조
일렉트론은 이해하기 어렵지 않은 단순한 구조를 가지고 있기 때문에 간단하게 설명하겠다.<br>
일렉트론은 두 가지 프로세스를 가지고 있다. 하나는 main 스크립트를 실행하는 **메인 프로세스**, 화면을 그려주는 **렌더러 프로세스**가 있다.<br>
일렉트론은 웹페이지를 보여주기 위해 Chromium을 사용하는데 바로 이 **렌더러 프로세스**가 일렉트론 안에서 보여지는 각각의 웹 화면을 그려준다.<br>
메인 프로세스에서는 이러한 렌더러 프로세스들을 관리하고, 각각의 렌더러 프로세스들은 서로 독립적으로 동작한다.<br>
그리고 렌더러 프로세스에서 네이티브 API는 직접 호출이 불가능하며 오로지 메인 프로세스에 요청하여 실행할 수 있다.
<img src='https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FboSdbW%2FbtqBg8ou4He%2FXdrW3m4fvLkcOnOYOf6ox0%2Fimg.png'>

## 장점
- 낮은 진입장벽
- 개발 속도 향상
- 크로스 플랫폼 지원
- 풍부한 OS 네이티브 API를 가진 런타임 제공
- 써드파티 지원
- 노드 접근 권한 관리
- 빌드 툴과 인스톨러 제공

## 단점
- 큰 설치 파일 용량
- 상대적으로 느린 속도
- 보안은 개발자의 책임
- 크로스 플랫폼 빌드

## 일렉트론 시작하기
그러면 실제 예제를 통해 한번 만들어보자

### 프로젝트 생성
Node.js 기반이기 때문에 Node.js는 반드시 설치되어 있어야한다. <br>
일단 npm 프로젝트를 만들어주고 일렉트론 모듈을 설치하자. 아래 커맨드들을 실행한다.
```sh
$ npm init
$ npm install -D electron
```

만약 프록시를 타는 환경이라면 electron을 설치하는 중에 timeout 오류가 날 수 있다. 당황하지말고 아래 커맨드에 프록시 주소를 설정해주고 실행해 주자.
```sh
npx cross-env ELECTRON_GET_USE_PROXY=true GLOBAL_AGENT_HTTPS_PROXY=http://username:password@host:8080 npm install -D electron
```

그리고 package.json을 확인하여 script 필드에 아래처럼 일렉트론 실행 스크립트를 추가해준다.
```json
{
  "name": "electron-project",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "start": "electron .",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "electron": "^12.0.4"
  }
}
```

위 package.json의 main이 엔트리 포인트이다. index.js를 생성해주고 일렉트론에 사용할 모듈을 넣어주자.

### 프로젝트 구현

index.js
```javascript
const {app, BrowserWindow, ipcMain} = require('electron');

app.on('ready', () => {
    const win = new BrowserWindow({
        width: 800,
        height: 600,
        webPreferences: {
            nodeIntegration: true,
        }
    });
    win.setMenu(null);

    win.loadFile('index.html');
});
```
위 코드를 설명하자면 electron 노드 모듈에서 app, BrowserWindow, ipcMain을 가져온다. 각각의 내용은 다음과 같다.

> - app : 애플리케이션의 이벤트 사이클을 관리한다.(Main Process)
> - BrowserWindow : 브라우저 창을 생성하고 관리해준다.(Main Process)
>   - webPreference : 웹페이지들의 feature 설정 값
>       - nodeIntegration : node를 통합할지의 여부를 선택하는 옵션이며, 기본값은 false이다. true로 변경해줌으로써 클라이언트 웹페이지에서 node를 사용할 수 있다.
> - ipcMain : 메인 프로세스와 렌더러 프로세스 간의 비동기적인 통신을 담당한다.
> - win.setMenu(null)의 의미는 메뉴를 쓰지 않겠다는 의미이다. 해당 코드를 빼면 메뉴가 활성화된다.
> - win.loadFile('index.html')은 브라우저 창에 그릴 html파일을 로드한다.

index.js를 만들었다면 로드할 index.html을 간단하게 만들어보자.

index.html
```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Product&Category SQL Generator for F&B</title>
</head>
<body>
    <div>
        <div style="display: flex;align-items: center;">
            <p style="width: 200px;margin: 10px;">Select File</p>
            <input id='fileSelector' type="file" accept=".xlsx"/>
        </div>
        <div style="display: flex;align-items: center;">
            <p style="width: 200px;margin: 10px;">MGT_SHOP_ID</p>
            <input id="mgtShopId" type="text" style="height: 20px;"/>
        </div>
        <div style="display: flex;align-items: center;">
            <p style="width: 200px;margin: 10px;">LEAF_SHOP_ID</p>
            <input id="shopId" type="text" style="height: 20px;"/>
        </div>
        <div style="display: flex;align-items: center;">
            <p style="width: 200px;margin: 10px;">Key for generating ID</p>
            <input id="key" type="text" style="height: 20px;"/>
        </div>
        <div style="display: flex;align-items: center;">
            <p style="width: 200px;margin: 10px;">SQL 파일 저장 경로</p>
            <input id="savePath" type="text" style="height: 20px;"/>
        </div>
        <div style="display: flex;align-items: center;margin:10px;">
            <button class="snip1535" id="generateBtn">SQL 생성</button>
        </div>
    </div>
    <hr>
    <div class='description'>
        <h4>사용 설명</h4>
        <p>* MGT_SHOP_ID는 Management 매장 아이디를 의미합니다.</p>
        <p>* SHOP_ID는 Leaf 매장 아이디를 의미합니다.</p>
        <p>* Key는 Unique한 카테고리ID, 제품ID 생성을 위한 키입니다.</p>
        <p>* 파일 저장경로를 입력하지 않으면 입력한 파일의 경로로 기본 저장됩니다.</p>
        <p>* 엑셀 포맷은 아래 내용을 확인하십시오.</p>
        <p>아래 시트들 반드시 포함(이름 같아야함)</p>
        <div style="display: flex;">
            <button id="ALL_MENU">ALL_MENU</button>
            <button id="AVAILABLE_STORE_MENU">AVAILABLE_STORE_MENU</button>
            <button id="ALL_ADDON_MENU_MAP">ALL_ADDON_MENU_MAP</button>
            <button id="AVAILABLE_ADDON_MAP">AVAILABLE_ADDON_MAP</button>
        </div>
    </div>
    <script src="./renderer.js"></script>
</body>
<style type="text/css">
</style>
</html>
```
위 html 코드는 단순 html 코드이다. 여기까지 했다면 애플리케이션을 구동해보자.
아래 커맨드로 실행할 수 있다.
```sh
$ npm start
```

실행하면 윈도우 창과 함께 html 화면이 그려질 것이다.

그러면 이제 필요한 값들을 화면 입력을 통해 받아서 Main Process로 전달하여 처리하는 로직을 넣어보겠다.
html 소스에 스크립트가 참조되어있는 것을 보았을 것이다. renderer.js를 아래와 같이 만들어보자.
엑셀파일을 파싱하는 작업을 하기 때문에 관련 모듈인 xlsx 노드 모듈을 설치해야한다. **npm install xlsx**를 통해 설치하자.

여기서 중요한 내용은 **ipcRenderer**이다. 'start'라는 이벤트를 통해 ipcMain으로 sendData를 보내는 코드를 확인하자.

renderer.js
```javascript
const {ipcRenderer} = require('electron');
const XLSX = require('xlsx');


const button = document.getElementById('generateBtn');

button.addEventListener('click', () => {
    const fObj = document.getElementById("fileSelector");
    if(fObj.value === '') {
        alert('Please Select File');
    } else {
        const mgtShopId = document.getElementById('mgtShopId').value;
        const shopId = document.getElementById('shopId').value;
        const key = document.getElementById('key').value;
        let savePath = document.getElementById('savePath').value;
        const selectedFile = fObj.files[0];
        if(savePath.length == 0) {
            savePath = selectedFile.path;
        }
        console.log('savePath', savePath);
        var reader = new FileReader();

        reader.onload = function(evt) {
            if(evt.target.readyState == FileReader.DONE) {
                var data = evt.target.result;
                data = new Uint8Array(data);

                // call 'xlsx' to read the file
                let workbook = XLSX.read(data, {type: 'array'});
                const sendData = {
                    mgtShopId,
                    shopId,
                    key,
                    workbook,
                    savePath
                }

                ipcRenderer.send('start', JSON.stringify(sendData));
            }
        };
        reader.readAsArrayBuffer(selectedFile);
    }
});

ipcRenderer.on('finish', () => {
    alert('SQL 생성이 완료되었습니다.');
});
```

렌더러 프로세스에서 메인 프로세스로 데이터를 보냈으니 메인 프로세스에서 렌더러 프로세스를 받아야한다.<br>
index.js에서 아래 코드를 추가하자. start로 발행된 이벤트를 받아서 처리해주는 내용이다. convert 함수에 비즈니스 로직이 들어있다고 보면 된다.<br>
비즈니스 로직이 끝나면 끝났다는 이벤트를 트리거해주자(event.sender.send('fininsh')). 그러면 위의 renderer.js에서 'finish' 이벤트를 받아 alert를 띄워줄 것이다.
```javascript
ipcMain.on('start', (event, arg) => {
    const data = JSON.parse(arg);
    const {mgtShopId, shopId, key, workbook, savePath} = data;
    // convert(mgtShopId, shopId, key, workbook, savePath);
    event.sender.send('finish');
});
```

자바스크립트로 간단하게 엑셀을 파싱해서 쿼리를 만들어주는 데스크톱 어플리케이션을 만들어 보았다. 
실제 비즈니스 로직 구현부는 fs 모듈을 활용하여 엑셀데이터를 파싱 후 insert 쿼리를 만들고 sql 파일을 하나 생성하도록 만들었다.
자세한 내용은 보안 이슈가 될 것 같아 올리진 않겠다.

### 프로젝트 패키징
electron 프로젝트를 패키징해주는 다양한 모듈들이 있는데 그 중에서 electron-builder를 사용하여 패키징해보겠다.<br>
다양한 옵션을 넣을 수 있지만 간단한 애플리케이션이기 때문에 커맨드를 통해 바로 패키징해보겠다.

일단 electron-builder를 글로벌로 설치해주자
```sh
$ npm install -g electron-builder
```

그리고 루트 경로에서 아래 커맨드를 실행해준다.
```sh
$ npx electron-builder build
```

그러면 해당 경로에 dist 폴더가 생성되고 윈도우 설치파일이 생성된다. 해당 파일을 설치하면 exe 파일로 윈도우 애플리케이션을 실행할 수 있다.

# 정리
일을 하다보면 간단한 데스크톱 애플리케이션이 필요할때가 종종 생긴다. 매크로를 만든다거나 위처럼 엑셀을 파싱한다거나 필요에 따라 API를 호출하여 사용하는 애플리케이션들이 필요하다.<br>
파이썬도 좋지만 자바스크립트와 Node.js의 개념만 있으면 일렉트론을 사용하여 데스크톱 애플리케이션(심지어 크로스플랫폼을 지원하는)을 손쉽게 만들 수 있다.<br>
심심하면 해보는 것을 추천한다.

# Reference
- 공식문서 <a href='https://www.electronjs.org/docs'>https://www.electronjs.org/docs</a>
- <a href='https://www.samsungsds.com/kr/insights/1239178_4627.html'>https://www.samsungsds.com/kr/insights/1239178_4627.html</a>
- <a href='https://cyberx.tistory.com/206'>https://cyberx.tistory.com/206</a>
