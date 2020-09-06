---
title: "react 공부 - Life Cycle"
layout: single
author_profile: true
categories: 
- react
---

# 리액트의 라이프 사이클

리액트에는 리액트 컴포넌트의 라이프 사이클이 존재한다. 컴포넌트가 생성되고, 업데이트 되고, 종료될 때 각각 호출되는 함수가 있는데 이를 라이프 사이클이라고 하며 이것을 적절히 이용하여 Back-end와 원활한 통신을 할 줄 알아야 한다.

## Mounting 과정

React 컴포넌트 객체가 DOM에 삽입되기의 과정을 뜻하며 그 과정은 다음과 같다.

1. constructor()
2. componentWillMount()
3. render()
4. componentDidMount()

위와 같은 과정을 거쳐 Mounting이 이루어 지며 앞선 예제에서도 살펴 보았듯이 어떤 조건에 의해 컴포넌트의 값을 변경해야 할 경우에는 componentDidMount 함수를 호출하여 해결한다. Back-end의 Api를 호출하는 작업도 보통 여기서 실행한다.



## Update(데이터 변경)

컴포넌트의 데이터(state)가 변경되면 다음과 같은 순서로 함수가 호출되게 되며 대개의 경우 리액트에서 자동으로 처리해주기 때문에 이 함수를 호출할 일은 적다.

1. shouldComponentUpdate()
2. componentWillUpdate()
3. render()
4. componentDidUpdate()

컴포넌트의 상태에 변경이 감지되면 shouldComponentUpdate가 실행되게 되며 그 아래의 함수를 자동으로 실행하게 된다.

## Api 호출 예제

여기서는 api를 호출하여 그 결과를 화면에 띄워주는 코드를 작성해 보았고, 테스트용 api를 제공해주는 jsonplaceholder 에서 데이터를 받아왔다. 

### 컴포넌트 작성

```jsx
class ApiExample extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      data: ''
    }
  }
  
  callApi = () => {
    fetch("https://jsonplaceholder.typicode.com/todos/1")
    .then(res => res.json())
    .then(json => {
      this.setState({
        data: json.title
      })
    })
  }
 
  componentDidMount() {
    this.callApi();
  }

  render() {
    return (
      <h3>
        {this.state.data ? this.state.data : "데이터를 불러오는 중입니다."}
      </h3>
    );
  }
}

ReactDOM.render(<ApiExample />, document.getElementById('root'));
```

constructor를 이용해 data라는 state를 초기화 해 주는데, 라이프 사이클 호출 순서상 componentDidMount가 실행 되기 이전에 render가 실행 되므로 맨 처음에는 아무런 값도 존재하지 않는 상태에서 렌더링을 하게 된다. 이 경우를 고려하여 삼항연산자를 통해 data가 비었을 경우를 처리해 준다.

callApi 함수에서는 api를 호출하여 데이터를 받아오고 해당 데이터를 json으로 변환한 뒤에 setState를 이용하여 컴포넌트의 data 값을 업데이트 해 준다. componentDidMount에서 callApi를 호출하면 state.data가 변경되면서 화면에 그 값이 렌더링 된다.

![image-20200906203151742](../../post_images/20200906/image-20200906203151742.png)