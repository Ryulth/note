## React-typescript 101 with Woowa tech learning

### 1 주차

* create app

npm install -g create-react-app

npm create-react-app my-app --typescript

* 기본 폴더 구조

public dir ->

entry file 사실상 고칠 일이 없음 

src dir ->

index.tsx 첫진입점

ts -> typescript

tsx -> ui적인 요소를 가지고 있는 코드

폴더명 src로 고정되어서 컴파일된다.



안정적인 버젼js 5.0 너무 후져서 바벨 2 를 쓴다

상대 경로가 없는 애들은 npm 에서 다운받은 패키지들이다

개발자가 건드는 만드는건 상대경로 쓰자



@types -> 저장소

타입스크립트는 전부 타입이 있어야한다.  (객체화 같은 느낌인가)

타입스크립트로 리액트를 사용하는 거엿구나



내가 만든건 /App 라고 하고

```react
ReactDOM.render(<App />, document.getElementById("root"));
```



번들러를 사용하기떄문에 여러가지 사용가능

파일 같은것도 코드안에 넣을 수 있음

```react
import logo from "./logo.png";
```



리액트의 컴포넌트 생성

1. 일반적인 것
2. 컴포넌트가 해비하면 필요한것 만 쓰는 컴포넌트
3. 함수형 컴포넌트 (리턴이 리액트 UI 요소)



 타입스크립트 타입 지정방법

App 이라는 변수에 React.FC 타입을 넣는다

FC -> functional components

html 태그가 컴포넌트라고 불리운다 직접만드는 태그는 대문자시작으로 작성한다.

class -> className 으로 바꿔쓴다.

```react
const App: React.FC = () => {
  return (
    <div className="App">
      <header className="App-header">
        <img src={logo} className="App-logo" alt="logo" />
        <p>
          Edit <code>src/App.tsx</code> and save to reload.
        </p>
        <a
          className="App-link"
          href="https://reactjs.org"
          target="_blank"
          rel="noopener noreferrer"
        >
          Learn React
        </a>
      </header>
    </div>
  );
};
export default App;
```



export default App; 햇기떄문에 다른대서 App 으로 쓸 수 있다

```react
import { render } from "react-dom";
```

render 만 가져오기

마법의 타입 any 

한 tsx 에 한 개의 export 를 한다 . 자바 클래스 명 같구만?



컴포넌트 알아서 상태관리 state 이거 flutter 의 state 같은 거 같은데? 

함수형은 state 가 안됨 props  이게 더 좋아보인다

```react
interface PlayButtonProps {
  monitoring: boolean; // 꼭 필요함
  onPlay?: () => void;
  onPause?: () => void; // ?는 옵셔널 
}
export const PlayButton: React.FC<PlayButtonProps(여기다 인터페이스 타입)> = props
```

타입으로 사용해도되나 interface 를 선호한다. (상황에 따라 다름)

리액트 최신 기술 훅!



```react
bordered={false} // primitive type 는 {}
bodyStyle={{   // 객체는 {{}} 이렇게
        background: "#fff",
        padding: "24px"
}}

(value: number): string // string 반환할거야
```



어디에도 new 는 없어.. 인스턴스는 리액트가 알아서 만들어줌 new 이딴거 필요없다

퍼블릭이나 이딴거 접근할 일이 없다!

index.ts 를 폴더명에서 불러온다 (default)

types/index.ts 에서 한꺼번에 올려버린다 그룹핑 같은 느낌

그래서 한번에 types 를 불러오면 한방에 필요한거 import 가능 



* flutter 를 해봐서 그런지 뭔가 비슷하다 react 가 많은 걸 알아서 해준다. 





