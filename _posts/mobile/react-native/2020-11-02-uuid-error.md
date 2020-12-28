---
title: uuid crypto.getRandomValues() 에러 해결방법
author: mirukman
date: 2020-11-02 23:00:00 +0800
categories: [MOBILE, REACT-NATIVE]
tags: [react-native,리액트,네이티브,네비게이션,uuid,getrandomvalues,에러]
---

네비게이션 예제에서 객체에 고유 식별자를 부여하기 위해 uuid 모듈을 사용했다.

~~~ bash
npm install --save uuid
~~~

이후 프로젝트에 아래와 같이 모듈을 import 하고 함수를 사용했다.

~~~ javascript
import uuidv4 from 'uuid/v4';

...

const obj = {
  id: uuidv4(),
  ...
}
~~~

실행을 하니 에러가 발생했다.

![에러화면](https://mirukman.github.io/images/mobile/react-native/uuid-error/error-view-1.png){: width="100%" height="100%"}

중간을 보면 아래와 같은 에러 메시지가 보인다.

<span style="color:red">Unable to resolve module 'uuid/v4'</span>

구글에서 검색을 해봤다. uuid 모듈이 업데이트되고 사용법이 달라졌다는 것이다.

[https://www.npmjs.com/package/uuid](https://www.npmjs.com/package/uuid) 에서 uuid 사용법을 확인하고 프로젝트에 적용했다.

바뀐 사용법은 아래와 같았다.

~~~ javascript
import {v4 as uuidv4} from 'uuid';

...

const obj = {
  id: uuidv4(),
  ...
}
~~~

그러니 이번에는 런타임 에러가 발생했다.

![에러화면](https://mirukman.github.io/images/mobile/react-native/uuid-error/error-view-2.png){: width="100%" height="100%"}

중요한 에러 메시지는 아래와 같다.

<span style="color:red">crypto.getRandomValues() not supported</span>

stack-overflow에서 같은 문제를 가진 사람의 질문을 찾았고 'react-native-get-random-values' 모듈을 설치하면 된다는 것을 알았다.

~~~ bash
npm install --save react-native-get-random-values
~~~

설치 후 uuid를 import하기 전에 'react-navigation-get-random-values'를 먼저 import 해준다.

~~~ javascript
import 'react-native-get-random-values';
import {v4 as uuidv4} from 'uuid';

...

const obj = {
  id: uuidv4(),
  ...
}
~~~

이후 실행을 했더니 문제없이 동작했다.

