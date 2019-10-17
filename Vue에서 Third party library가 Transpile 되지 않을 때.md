# Vue에서 Third party library가 Transpile 되지 않을 때

#### 대상 환경
- vue 3 (vue-cli 3)
- babel 7.x

## 본문
Vue로 구성된 프로젝트 진행 중 서비스 배포 후 IE11에서 SCRIPT1002: Syntax error로 인해  
로그인 화면부터 모든 페이지가 정상적으로 구동되지 않는 문제가 발생했다.  

해당 파일은 chunk-vendors.js 파일이었는데 단순 Polyfill 문제로 생각하여 es6 shim을 추가하였지만 해결되지 않았다.   

Local 환경에서 해당 문제를 재현하고자 확인하였으나 동일 문제가 재현되지는 않았다  

확인을 위해 배포된 곳에 접속하여 개발자도구의 Source Browser를 통해 따라가보니   
최근에 추가한 Third party에 포함 된 파일에서 발생하고 있었다.  

해당 소스에선 ES6의 문법 중 하나인 class를 사용하고 있었고 Syntax error로 간주되는 것으로 보아  
IE11에선 해당 문법을 지원하지 않는것으로 파악하였다. <b><a href="https://kangax.github.io/compat-table/es6/">(ES6 브라우저, Polyfills 지원 상황)</a></b>

현재 우리 프로젝트에선 class는 사용하지는 않지만 => {}와 같은 화살표 함수도 사용하고 있지만 문제는 없었다.  
추측하기로 프로젝트 외부에 있는 의존성, 즉 node_modules 하위에 있는 대상에 대해서는 Babel이 Transfile 하지 않는것 같았다.  

vue 3 ie 11 등과 같은 키워드로 구글에 검색을 해보면 다양한 자료들은 많았다  
- <a href="https://stackoverflow.com/questions/52056358/vue-cli-3-project-not-working-on-ie-11">https://stackoverflow.com/questions/52056358/vue-cli-3-project-not-working-on-ie-11</a>    
- <a href="https://forum.vuejs.org/t/vue-cli-3-and-ie-11-not-working/38717">https://forum.vuejs.org/t/vue-cli-3-and-ie-11-not-working/38717</a>    
- <a href="https://stackoverflow.com/questions/56259759/ie11-with-vue-cli-project-script1002-syntax-error/56262293">https://stackoverflow.com/questions/56259759/ie11-with-vue-cli-project-script1002-syntax-error/56262293</a>  
- <a href="https://cereme.dev/blog/vue-cli3-ie-polyfill/">https://cereme.dev/blog/vue-cli3-ie-polyfill/</a>  
- <a href="https://jsdev.kr/t/ie-vue/4257">https://jsdev.kr/t/ie-vue/4257</a>  

위의 해결 방법 외에도 다양한 방법들을 시도해보았으나 효과는 없었다.  
  
힌트는 다음의 스택오버플로우에서 찾을 수 있었다  
<a href="https://stackoverflow.com/questions/55132564/babel-not-transpiling-chunk-vendors-for-ie11-in-vue-cli-project">https://stackoverflow.com/questions/55132564/babel-not-transpiling-chunk-vendors-for-ie11-in-vue-cli-project</a>

chunk=vendors는 third party 라이브러리들을 묶어서 관리하는 것으로 webpack이 담당한다  
따라서 Webpack이 번들링하기 전에 Babel이 third part를 제대로 변환을 할 수 있다면 문제가 해결 될 것이라고 판단하였고  
vue.config.js에 아래 설정을 추가할 수 잇다는 것을 확인했다 

```javascript
transpileDependencies: ['/node_modules/myproblematicmodule/']
```

위 설정에 문제가 되는 모듈들을 명시하고 빌드 후 배포하였으나 동작하지는 않았다.  
혹시 vue.config.js가 제대로 동작하지 않나 싶어서 테스트 용도로 다른 옵션 중 하나인 outputDir을 설정해보았다 
```javascript
module.exports = {
    transpileDependencies: [
        '...'
    ],
    outputDir: 'www'
}
```
해당 옵션은 빌드 후 번들된 파일들의 디렉토리인데 Default는 dist였다.  
현재 Nginx에 배포 시 Document root는 www이어서 배포 편의를 위해 설정을 해보았고 정상적으로 동작 했다.  

vue.config.js는 정상적으로 동작 함을 알앗으니 transpileDependencies 옵션의 정확한 사용법만 찾으면 될 듯 했다.  

참조) <a href="https://cli.vuejs.org/config/#configuration-reference">Vue Cli Configuration Reference</a>  

아래의 스택오버플로우 글에서 설정 예를 확인할 수 있었는데 
<a href="https://stackoverflow.com/questions/52615179/vue-cli-3-does-not-convert-vendors-to-es5">https://stackoverflow.com/questions/52615179/vue-cli-3-does-not-convert-vendors-to-es5</a>  
댓글에 힌트가 있었다.
```text
It works! But one little problem: you should list (in babel.config.js) every dependency you want to transpile:
 transpileDependencies: ['dateformat', 'query-string', 'strict-uri-encode', 'split-on-first'] – NikitaK Jun 7 at 14:59
```

대충 설명으로 보아 경로를 적는 것이 아니라 node module의 이름을 적는 것이 아닐까 추측하였고
관련 모듈들의 path를 이름으로 바꾸어 빌드후 배포해보았다.  
```javascript
module.exports = {
    transpileDependencies: [
        'operation-retrier',
        'twilio-sync',
        'twilio-chat',
        'twilio-mcs-client',
        'twilio-notifications',
        'twilsock',
    ],
    outputDir: 'www'
}
```

테스트 한 결과 정상적으로 동작 함을 확인 했다.  

## 결론
위의 해결 과정으로 파악하건데 현재 환경에서 Babel이 node_modules 하위의 모든 의존성에 대해서 변환은 하지 않는 것 같다.  
빌드 시간에 대해 최적화로 생각할 수 있었는데 이전에 Angular 프로젝트를 진행할 때 angular-cli를 통해서 빌드할 때에는 없었던 문제라  
생각보다 트러블 슈팅이 힘들었다. (Angular는 자동으로 변환을 해주기에 빌드 시간이 길었을까...)  

그런데 왜 Local 환경에선 문제가 없었을까?  
Local일 때는 모든 모듈에 대해서 변환을 하는 것인가?  

정답은 Local 환경에서도 변환은 일어나지 않는다. 그럼에도 정상적으로 돌아간다고 생각했던 것은 개발 환경과 빌드 된 배포 환경의   
코드 스플리팅 차이였다.  

문제가 되었던 라이브러리를 사용하는 화면은 require로 es6 문법을 사용하는 모듈을 로드하고 있었다.  
따라서 Local 환경 테스트 시 다른 페이지는 영향이 없었고 해당 페이지에서만 문제가 있었지만 제대로 테스트를 하지 않아  
정상적으로 동작하고 있다고 단정했던 것이다.  

빌드 된 배포 환경에선 chunk-vendor에 관련된 Third party 들을 모두 번들링 하여 로드하기 때문에  
문제가 로그인 화면부터 시작하여 모든 페이지에서 나타난 것이었다.  

문제 처리가 완료됐다고 생각하여 쓸데 없이 설정 한 Babel polyfill 설정을 제거하고 소스를 정리 후 배포했는데 해당 페이지에서 문제가 또 생겼다.  
이번엔 'exports not defined'이었다. Babel polyfill 설정을 다시 추가하여 해결이 되었는데 빌드 환경에 대해 테스트가 제대로  
되어있지 않았던 것이 아닌가 싶었다...

Polyfill 등록은 아래와 같이 했다.  
```javascript
//main.js
import 'babel-polyfill';  
```

```javascript
//babel.config.js
process.env.VUE_CLI_BABEL_TRANSPILE_MODULES = true;
module.exports = {
    presets: [
        [
            '@babel/preset-env'
        ]
    ]
}
```

```javascript
//index.js
configureWebpack: {
        entry: ["babel-polyfill", "./src/main.js"]
    }
```

FE frame work는 간편함과 편리함을 바탕한 생산성을 무기로 두고 있기는 하나 항상 브라우저 대응을 위해 시간이 더욱 소비되는 것 같다.  
지난 Angular에서도 버전별로 설정 파일의 위치가 다르거나 지원하는 설정 파일이 달랐는데 Vue도 마찬가지 였다.
  
이번 트러블 슈팅에서 건드린 파일만 해도 .babelrc, babel.config.js, package.json, index.js, main.js 등 파일이 있었고  
버전별로 미묘하게 설정 값 내용이 달랐다.  

Angular에서도 polyfill을 이것 저것 넣어서 어떤것이 제대로 동작하는지도 모르고 덕지 덕지 붙여넣었던 경험을 떠올리면  
React는 어떨까 생각이 들기도 하고 이번에 처리한 내용, 과거에 처리한 내용이 최적은 아닐 텐데... 하는 생각이 들기도 한다.  

조금 더 우아하게 처리할 수 있을 것 같은 아쉬움이 남지만 더 건드리기엔 의욕이 생기지는 않는다. 



## Modern build
Babel 사용 시 ES6 이후의 최신 기능을 사용할 수 있지만 Transpile, Polyfill이 반드시 필요하다  
하지만 이러한 것들은 원래 코드보다 더 클 것이고 브라우저가 구문분석하기에 오버헤드가 될 수 있다.  

하위 브라우저 커버를 위해서 최신 문법으로 작서한 코드를 이전의 문법으로 맞게 변환한 것을 무작정 실행하기에는  
최신 브라우저들이 최신 문법에 맞춰 최적화 된 것들이 너무 아쉽다.   

Vue CLI에선 이러한 문제를 처리하기 위해 Modern mode를 지원한다.  

최신 브라우저를 지원하는 번들과 그렇지 않은 번들을 나누어 배포하고 실행하는 것.  
관련자료 <a href="https://cli.vuejs.org/guide/browser-compatibility.html#modern-mode">Vue CLI Modern mode</a>  

번들링 된 결과는 아래와 같다.  
자세히 보면 -legacy postfix가 붙은 파일들이 있는데 이것들이 하위 브라우저에 맞게 Transpile 된 번들이다.
```html
<!DOCTYPE html>
<html lang=en>
<head>
    <meta charset=utf-8>
    <meta http-equiv=X-UA-Compatible content="IE=edge">
    <meta name=viewport content="width=device-width,initial-scale=1">
    <link rel=icon href=/appicon.png>
    <link href="https://fonts.googleapis.com/css?family=Roboto:100,300,400,500,700,900|Material+Icons" rel=stylesheet>
    <script src="https://cdn.polyfill.io/v2/polyfill.js?features=String.prototype.includes,Array.prototype.find,Promise,Array.from"></script>
    <link rel=stylesheet
          href="https://fonts.googleapis.com/css?family=Noto+Sans+JP:400,500,700|Noto+Sans+KR:400,500,700&subset=japanese,korean">
    <link rel=stylesheet href=https://unpkg.com/element-ui/lib/theme-chalk/index.css>
    <title>cubeSafetyAPP</title>
    <link href=/js/pdfjsWorker.cbe1949e.js rel=prefetch>
    <link href=/css/app.34982bed.css rel=preload as=style>
    <link href=/css/chunk-vendors.a30f3840.css rel=preload as=style>
    <link href=/js/app.bad75dc2.js rel=modulepreload as=script>
    <link href=/js/chunk-vendors.97b51f02.js rel=modulepreload as=script>
    <link href=/css/chunk-vendors.a30f3840.css rel=stylesheet>
    <link href=/css/app.34982bed.css rel=stylesheet>
</head>
<body>
<noscript><strong>We're sorry but cubesafetyapp-front doesn't work properly without JavaScript enabled. Please enable it
    to continue.</strong></noscript>
<div id=app></div>
<script type=module src=/js/chunk-vendors.97b51f02.js></script>
<script type=module src=/js/app.bad75dc2.js></script>
<script>!function () {
    var e = document, t = e.createElement("script");
    if (!("noModule" in t) && "onbeforeload" in t) {
        var n = !1;
        e.addEventListener("beforeload", function (e) {
            if (e.target === t) n = !0; else if (!e.target.hasAttribute("nomodule") || !n)return;
            e.preventDefault()
        }, !0), t.type = "module", t.src = ".", e.head.appendChild(t), t.remove()
    }
}();</script>
<script src=/js/chunk-vendors-legacy.97b51f02.js nomodule></script>
<script src=/js/app-legacy.9865cd4b.js nomodule></script>
</body>
</html>
```





