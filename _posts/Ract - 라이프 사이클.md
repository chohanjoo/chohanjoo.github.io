## React - 라이프 사이클

라이프 사이클 메서드의 종류는 총 열 가지이다.
`Will` 접두사가 붙은 메서드는 어떤 작업을 작동하기 전에 실행되는 메서드고,
`Did` 접두사가 붙은 메서드는 어떤 작업을 작동한 후에 실행되는 메서드이다.



### 마운트

DOM이 생성되고 웹 브라우저상에 나타나는 것을 마운트(mount) 라고 한다.

~~~
컴포넌트 만들기 -> constructor -> getDerivedStateFromProps -> render -> componentDidMount
~~~

constructor : 컴포넌트를 새로 만들 때마다 호출되는 클래스 생성자 메서드

getDerivedStateFromProps : props에 있는 값을 state에 동기화하는 메서드

render : 우리가 준비한 UI를 렌더링하는 메서드

​	이 메서드 안에서 this.props 와 this.state에 접근할 수  있으며, 리액트 요소를 반환한다.

​	이 메서드 안에서는 절대로 state를 변형해서는 안 되며, 웹 브라우저에 접근해서도 안 된다.
​	DOM 정보를 가져오거나 변화를 줄 때는 componentDidMount에서 처리해야 한다.

componentDidMount : 컴포넌트가 웹 브라우저상에 나타난 후 호출되는 메서드

​	다른 자바스크립트 라이브러리 또는 프레임워크의 함수를 호출하거나 이벤트 등록, setTimeout, setInterval, 네트워크 요청 같은 비동기 작업을 처리하면 된다.

​    예를 들면 먼저 사용자 정보 목록이 실린 JSON 데이터를 가져올 수 있다. 그 후에 사용자 정보를 table을 이용하여 표로 그릴 수 있다.



### 업데이트

컴포넌트를 업데이트할 때는 네 가지

1. props가 바뀔 때
2. state가 바뀔 때
3. 부모 컴포넌트가 리렌더링될 때
4. this.forceUpdate가 강제로 렌더링을 트리거할 때



![update](C:\Users\USER\Desktop\Blog\update.png)



getDerivedStateFromProps : 이 메서드는 마운트 과정에도 호출하며, props가 바뀌어서 업데이트할 때에도 호출

shouldComponentUpdate : 컴포넌트가 리렌더링을 해야 할지 말아야 할지를 결정하는 메서드이다. 여기서 false를 반환하면 아래 메서드들을 호출하지 않는다.

​	반드시 true or false를 반환해야 한다.

​	이 메서드 안에서 현재 props와 state는 this.props와 this.state로 접근하고, 새로 설정될 props 또는 state는 nextProps와 nextState로 접근할 수 있다.

componentWillUpdate: 새로운 속성이나 상태를 받은 후 렌더링 직전에 호출된다. 이 메서드는 초기 렌더링 시에는 호출되지 않는다. 갱신 전에 필요한 준비 작업을 처리할 대 componentWillUpdate() 메서드를 사용하고, 이 메서드 내에서 this.setState() 를 사용하는 것을 피하는 것이 좋다. shouldComponentUpdate가 false를 반환하면, 실행되지 않는다.

getSnapshotBeforeUpdate : 컴포넌트 변화를 DOM에 반영하기 바로 직전에 호출하는 메서드

componentDidUpdate : 컴포넌트의 업데이트 작업이 끝난 후 호출하는 메서드
     컴포넌트가 갱신된 후에 DOM이나 그 외의 요소를 다루는 코드를 작성해야 할 때 유용하다. 왜냐하면 이 시점에는 DOM에 모든 변경 사항이 반영되어 있기 때문이다.



### 언마운트

마운트의 반대 과정, 컴포넌트를 DOM에서 제거하는 것

~~~
언마운트하기 -> componentWillUnmount
~~~

componentWillUnmount : 컴포넌트가 웹 브라우저상에서 사라지기 전에 호출하는 메서드