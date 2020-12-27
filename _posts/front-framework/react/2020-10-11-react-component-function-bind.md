---
title: 리액트 컴포넌트 함수 바인딩과 this의 범위
author: mirukman
date: 2020-10-11 20:00:00 +0800
categories: [FRONT-END, REACT]
tags: [react, component, function, binding, 리액트, 컴포넌트, 함수, 바인딩, this]
---

리액트에서 컴포넌트를 다루다보면 this의 scope 문제로 자바스크립트 엔진이 함수를 인식하지 못하는 일이 종종 발생한다.

자바스크립트 ES6부터 지원하는 class에서는 함수를 정의할 때 아래와 같이 생성자에서  this를 항상 바인드 해주어야한다다.

~~~ javascript
class Counter extends Component {
   constructor(props) {
      super(props);
      this.state = {
         count: 0,
      };

      this.increaseCount = this.increaseCount.bind(this);
      this.resetCount = this.resetCount.bind(this);
   }

   increaseCount() {
      this.setState({
         count: this.state.count + 1,
      });
   }

   resetCount() {
      this.setState({
         count: 0,
      });
   }

   render() {
      return (
         <div>
            현재 카운트: {this.state.count}
            <button onClick={this.increaseCount} onMouseOut={this.resetCount}>
               카운트 증가
            </button>
            버튼 밖으로 커서가 움직이면 0으로 초기화 됩니다.
         </div>
      );
   }
}
~~~

constructor 메소드 내부에서 자체 함수를 객체에 바인딩 해주었다. 이 설정을 하지 않으면 아래와 같은 에러 메시지가 뜬다.

<span style="color:red">Uncaught TypeError: Cannot read property 'setState' of undefined</span>

<br>
이런일이 발생하는 이유는 함수가 호출되는 환경에 있다.

상위 컴포넌트에서 `increaseCount()`가 호출되면 코드 블럭 내부의 `this.setState({count: this.state.count + 1})`이 실행되는데 이 시점에서의 this는 **window** 객체가 된다.

따라서 생성자에서 함수의 this가 가르키는 객체를 컴포넌트 자신으로 설정해주어야 문제가 사라진다.

<br/>

하지만 이보다 간단한 해결법이 존재한다.

바로 화살표 함수를 사용하는 방법이다.

애초에 `increaseCount()`의 정의를 아래와 같이 화살표 함수를 사용해서 정의하면 된다.

~~~ javascript
increaseCount = () => {
   this.setState({
      count: this.state.count + 1
   });
}
~~~

이유는 화살표 함수로 정의된 함수는 자동으로 this가 자신에게 바인드되기 때문이다.
