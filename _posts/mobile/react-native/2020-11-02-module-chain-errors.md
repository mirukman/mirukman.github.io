---
title: 리액트 네이티브 네비게이션 모듈 인식불가 에러 해결방법
author: mirukman
date: 2020-11-02 22:00:00 +0800
categories: [MOBILE, REACT-NATIVE]
tags: [react-native,리액트,네이티브,네비게이션,모듈,에러,cannot,resolve]
---

리액트 네이티브 네비게이션에 대해 공부하기 위해 아래 모듈들을 설치했다.

~~~ bash
$ npm install --save react-native-gesture-handler react-native-reanimated
$ npm install --save uuid
$ npm install --save react-navigation react-navigation-stack react-navigation-tabs
~~~

이후 예제를 작성하고 실행시켰는데 bundle 실패 에러가 발생했다.

![에러화면](https://mirukman.github.io/images/mobile/react-native/module-chain-errors/error-view.png){: width="100%" height="100%"}

읽다보면 중간에 아래와 같은 문구가 있다.

<span style="color:red">Unable to resolve module 'react-native-screens' ...</span>

모듈이 없어 생기는 문제이므로 프로젝트 루트 경로에서 아래 명령어로 모듈을 설치해준다.

~~~ bash
$ npm install --save react-native-screens
~~~

그리고 다시 실행했다.

이번에는 다른 에러다.

<span style="color:red">Unable to resolve module 'react-native-safe-area-context' ...</span>

또 설치했다.

~~~ bash
$ npm install --save react-native-safe-area-context
~~~

다시 실행했다.

또 에러가 떴다.

<span style="color:red">Unable to resolve module '@react-native-community/masked-view' ...</span>

이것 역시 설치했다.

~~~ bash
$ npm install --save @react-native-community/masked-view
~~~

잘 돌아간다.

해결하고 보니 간단하다. 에러메시지는 모듈 의존성 문제라고 알려주므로 해당 모듈만 설치하면 해결되는 문제였다. 그런데 사진을 보면 아래쪽에 'try these steps...' 라면서 해당 내용들을 시도해 볼것을 권유한다. 그것도 해보고 안돼서 stack overflow도 뒤져가며 찾았다. 운이 없게도 하필 같은 증상에 대한 잘못된 솔루션들을 시도하다가 삽질로 시간만 날렸다.

여기서 중요한 점은 아래와 같다.

대부분의 경우에는 에러 메시지에 힌트가 있다.
