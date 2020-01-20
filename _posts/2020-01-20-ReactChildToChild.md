---
layout: single
title:  "[중간정리][Context] Child -> Child 로 데이터 전달하는방법"
tag : 
    - React
    - Context
    - React Hook API
---

## 들어가며

송이공판현황 웹 페이지의 raw data 조회 기능을 개발하는 와중에 겪은 문제와 이를 해결하기 위한 방법을 고민하는 과정, 실제로 적용한 방법에 대해 정리한다.


## 문제-형제관계(sibling)인 컴포넌트에서 데이터 전달방법에 대한 고민

(그림)


> 결론은 Context API를 사용.


raw data를 조회하기 위한 기능 구현중 유저가 지정한 필터 옵션을 전달하여 화면에 출력하는 부분에서 문제가 생겼다. 컴포넌트 구조는 위 그림과 같은데, Child1 컴포넌트의 state 데이터를 child2로 전달하는 방법에 대한것이다. 일반적인 Parent -> Child 혹은 Child -> Parent 이라면 이야기가 쉬운데 형제 관계에 있는 컴포넌트는 어떻게 하면 좋을지 쉽게 감이 오지 않았다.
그래서 현재 문제를 해결할 수 있는 방법들에 대해 몇시간동안 구글링한 결과 아래의 방법을 찾을 수 있었다.

1. `Redux`를 사용한다.
2. `Context API` 를 사용한다.


### 1. `Redux`를 사용한다.

본래 [Redux](https://redux.js.org/)는 React를 위해 나온것은 아니고 자바스크립트 오픈소스 기반 상태관리 라이브러리였으나 지금은 React의 상태관리하면 떠오르는 라이브러리중 하나가 되었다. 현재 직면한 문제를 해결하는데에 도움이 될 것 같았다.

**예상되는 장단점**

- 장점 
    - 상태관리가 편리해지고 현재 당면한 문제를 확실히 해결할 수 있다.
- 단점 
    - Redux를 새로 익혀야한다.

문제를 해결할 수는 있지만 그에따른 단점이 극명하게 드러난다. 지금 상태에서 Redux를 새로 배우고 적용하기위해 소스코드를 다 뜯어고치는건 빈대 잡으려고 초가삼간 다 태우는격이다. 따라서 Redux를 도입하는건 불필요하다고 판단하였다.

### 2. 'Context API'를 사용한다.

Context API는 React에서 제공하는 API로 `Context`를 통해 여러 컴포넌트가 Context를 통해 전역(Global)수준의 데이터를 공유할 수 있다. 

**예상되는 장단점**
- 장점
    - React 내장 API이기 때문에 별도 라이브러리 설치없이 사용이 가능하다.
    - 내장 API를 사용하는데 큰 어려움이 없다.
- 단점
    - 코드 재사용성이 떨어진다.

Context API는 Redux에 비해서 쉽게 사용이 가능하다. 지금 코드에서 큰 수정이 필요하지 않아 적용이 쉽다. 하지만 코드 재사용성이 떨어지게 된다. 

Redux 와 Context API 둘다 문제를 해결할 수 있기때문에 각각 타당성이 존재한다. 고민이 되던 찰나 결정을 내리는데 도움이 된 좋은 글을 찾을 수 있었다.

[Context API가 Redux를 대체할 수 있을까요?](https://medium.com/lunit-engineering/context-api%EA%B0%80-redux%EB%A5%BC-%EB%8C%80%EC%B2%B4%ED%95%A0-%EC%88%98-%EC%9E%88%EC%9D%84%EA%B9%8C%EC%9A%94-76a6209b369b)

Context API와 Redux를 닭잡는 칼과 소잡는 칼로 비유하여 설명한 글인데 지금 개발하고있는 어플리케이션은 Redux를 사용할만큼 복잡하지 않고 Context API면 충분하다는 결론을 얻었다. 결국 `Context API`를 사용하였다.

다만 Context API 바로 사용한것은 아니다. 문제가 되는 컴포넌트가 Hook API를 사용하고 있어서 Hook API에서 Context API를 사용하기 위해 `useContext`를 사용했고 Hook의 변경이 일어나는것에 대한 대응을 위해 `useReducer`를 사용하였다


## 코드 적용





ReactDataSearchPanel.js(Parent)에 Context를 생성한다.
```js
//ReactDataSearchPanel.js(Parent)

// context object 생성
export const AppContext = React.createContext();

// 초기상태 설정
const initialState = {

  saleDate : null,
  area : null,
  union : null,
  condition1 : null,
  condition2 : null,
  searchValue : ''
};

//reducer 정의
function reducer(state, action) {
  switch (action.type) {
    case 'UPDATE_FILTER_OPT':
      return action.data;
    default:
      return initialState;
  }
}

```

useReducer를 사용하기 위해 import 한다.
```js
//ReactDataSearchPanel.js(Parent)
import React, { useReducer } from 'react';

```



state를 처리할 hook을 useReducer통해 생성한다. useState를 대체하는 함수이며 child2의 state가 child1의 state 변경에 의존적이기 때문에 useState대신 useReducer를 사용하였다.

그리고 AppContext.Provider 로 자식 컴포넌트를 감싼다. 이렇게 해야 해당 context를 구독하는 컴포넌트에게 context의 변화를 알릴 수 있다.
```js
//ReactDataSearchPanel.js(Parent)
export default function RawDataSearchPanel() {
  
  const classes = useStyles();

  //state에 대한 hook 생성
  const [state, dispatch] = useReducer(reducer, initialState);

  const [open, setOpen] = React.useState(false);

 return (
 
     //..

      <AppContext.Provider value={{ state, dispatch }}>
        <RawDataFilterPanel></RawDataFilterPanel>
        <RawDataTable></RawDataTable>
      </AppContext.Provider>
  
    </div>
  );

```


Context가 정의된 부모의 컴포넌트를 자식 컴포넌트에서 import한다.

```js
//RawDataFilterPanel.js(Child1)
import { AppContext } from './RawDataSearchPanel';
```


useContext를 통해 Parent에서 생성한 Contetxt를 구독한다. 그리고 dispatch될 객체와 액션 타입을 넣는다.
```js
//RawDataFilterPanel.js(Child1)
export default function RawDataFilterPanel() {
    const classes = useStyles();

    const {state, dispatch} = useContext(AppContext);

    //...


    
    const handleSearchButton = () =>{
        dispatch({
            type: 'UPDATE_FILTER_OPT',
            data: {
                saleDate: saleDate,
                area: area,
                union: union,
                condition1: condition1,
                condition2: condition2,
                searchValue: searchValue
            }
        });
    }

```


Child1에서 변경된 state를 구독할 RawDataTable.js(Child2)에도 Context를 import한다.
```js
//RawDataTable.js (Child2)
import { AppContext } from './RawDataSearchPanel';

```



마찬가지로 useContext로 context를 구독한다.
```js
//RawDataTable.js (Child2)
export default function RawDataTable() {
  const classes = useStyles();
  const {state, dispatch} = useContext(AppContext);


```


전달받은 state를 처리하는 코드를 작성한다.
```js
//RawDataTable.js (Child2)
React.useEffect(() => {
    
    let {saleDate, area, union, condition1, condition2, searchValue} = state;
   
    //...

    let requestURL = "/GetAllData" + parms;
    
    fetch(requestURL)
      .then((response) => {
        //response.json()이 이미 Promise 객체를 리턴함
        response.json().then((data) => {
          setDataSet(data[0].totalData);
          
          if(data[0].totalCount.length > 0){
            setTotalCount(data[0].totalCount[0].count);
          } else {
            setTotalCount(0);
          }
          
        });
      })
      .catch((error) => {
        console.log(error);
      });
  }, [page, rowsPerPage, state]);
```


이렇게 Context API와 Hook API를 사용하여 문제를 해결하였다.




## 마무리

문제 해결 방법을 찾는 도중에 Redux와 Context API 말고도 다른 방법들을 찾았다. React.createRef() 라던지.. 이렇게 찾는것은 비교적 쉬운일이다.

다만 여러 방법중 어떤 방법이 가장 적절한지 찾는것은 어려운 일이라고 생각한다. 이런 일련의 과정을 잘 해낼수 있는 개발자가 실력있는 개발자가 아닐까 생각해본다.


### 참고한 링크
 
- [https://itnext.io/passing-data-between-sibling-components-in-react-using-context-api-and-react-hooks-fce60f12629a](https://itnext.io/passing-data-between-sibling-components-in-react-using-context-api-and-react-hooks-fce60f12629a)


- [Context API](https://ko.reactjs.org/docs/context.html)

- [Context를 사용하기전 참고사항](https://ko.reactjs.org/docs/context.html#before-you-use-context)

- [리액트 16.3 에 소개된 새로워진 Context API 파헤치기(VELOPERT.LOG)](https://velopert.com/3606)


- [Hooks API Reference #useReducer](https://ko.reactjs.org/docs/hooks-reference.html#usereducer)


- [State 끌어올리기](https://reactjs-kr.firebaseapp.com/docs/lifting-state-up.html)

