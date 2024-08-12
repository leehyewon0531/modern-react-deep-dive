# [6장] 리액트 개발 도구로 디버깅하기

# 6-1. 리액트 개발 도구란?

→ **React-dev-tools -** 리액트, 리액트 네이티브 등 다양한 플랫폼에서 사용할 수 있는 디버깅 도구
![image](https://github.com/user-attachments/assets/9afb37c0-9e58-48fd-b4a4-c9939d27b015)

# 6-2. 리액트 개발 도구 설치

https://chromewebstore.google.com/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi

# 6-3. 리액트 개발 도구 활용하기

## **6.3.1 컴포넌트**

- 현재 리액트 애플리케이션의 컴포넌트 트리 구조로 나타난다.
- props와 내부 hooks

### **컴포넌트 트리**

- 기명 함수로 선언되면 해당 컴포넌트명을, 익명함수라면 Anonymous라고 보여준다.

```tsx
import React, { ComponentType, memo } from "react";
import AnonymousDefaultComponent from "./Component3";

// 함수 선언문
const Component1 = () => {
  return <>Component1</>;
};

// 함수 표현식
const Component2 = () => {
  return <>Component2</>;
};

// 메모된 컴포넌트
const MemoizedComponent = memo(() => <>MemoizedComponent</>);

const withHOC = (Component: ComponentType) => {
  return function () {
    return <Component />;
  };
};

// 고차 컴포넌트로 감싼 컴포넌트
const HOCComponent = withHOC(() => <>HOCComponent</>);

export default function App() {
  return (
    <div>
      <Component1 />
      <Component2 />
      <AnonymousDefaultComponent />
      <MemoizedComponent />
      <HOCComponent />
    </div>
  );
}

```

- 컴포넌트를 기명 함수로 선언하는 것은 디버깅에 많은 도움이 된다.
- 기명 함수로 바꾸기 어렵다면 함수에 `displayName`을 추가하여 개발 모드에서 참고할 수 있다.
    
    ```tsx
    // 메모된 기명 함수 컴포넌트
    const MemoizedComponent = memo(function MemoizedComponent() {
      return <>MemoizedComponent</>;
    });
    
    // displayName 설정
    MomoizedComponent.displayName = "메모된 컴포넌트입니다";
    
    const withHOC = (Component: ComponentType) => {
      return function withHOC() {
        return <Component />;
      };
    };
    
    // 고차 컴포넌트로 감싼 기명 함수 컴포넌트
    const HOCComponent = withHOC(function HOCComponent() {
      return <>HOCComponent</>;
    });
    
    ```
    

### **컴포넌트명과 props**

### 컴포넌트명과 Key

- 빨간색 경고 이모티콘: 애플리케이션이 strict mode로 렌더링 되지 않았다

### 컴포넌트 도구

- 눈 아이콘: 해당 컴포넌트가 HTML의 어디에서 렌더링됐는지 확인
- 벌레 아이콘: 콘솔 탭에 해당 컴포넌트의 props, hooks, nodes에 대한 정보 기록
- 소스코드 아이콘: 해당 컴포넌트의 소스코드 확인

→ 컴포넌트 `props`: 해당 컴포넌트가 받은 props

→ 컴포넌트 `hooks`: 컴포넌트에서 사용 중인 훅 정보

- 훅에 기명 함수를 넘겨주면 디버깅이 쉽다
    
    ```tsx
    useEffect(function effectOnlyMount() {
      console.log("useEffect");
    });
    
    ```
    
- **rendered by**: 컴포넌트를 렌더링한 주체

## **6.3.2 프로파일러**

- 리액트가 렌더링하는 과정에서 발생하는 상황을 개발 모드에서 확인하기 위한 도구
    
    → 어떤 컴포넌트가 몇 번 렌더링 됐는지, 어떤 작업이 오래 걸렸는지 확인
    

### Flamegraph

- 렌더 커밋별로 어떠한 작업이 일어났는지 나타낸다
- 노란색에 가까울수록 렌더링이 오래 걸린 컴포넌트, 녹색에 가까울수록 빠르게 렌더링된 컴포넌트, 회색으로 표시하면 렌더링되지 않은 컴포넌트
- 이를 통해 메모이제이션이나 렌더링이 의도한 대로 제한되어 있는지 확인 가능

### Timeline

- 리액트 18 이상 환경에서 확인 가능
- 시간의 흐름에 따라 리액트가 작동하는 내용을 추적
- 시간 단위로 프로파일링 동안 무슨 일이 있었는지, 무엇이 어느 시점에 렌더링됐는지, 유휴 시간은 어느 정도였는지 확인 가능

### 프로파일링을 통해 개선하기

- state 변경을 최소 컴포넌트 단위로 분리하는 게 좋다.
- 즉, input의 text 상태는 Input 컴포넌트에서 관리해 앱 전체가 렌더되지 않도록 해야 한다.
- onSubmit과 같은 함수를 props로 넘겨줄 때는 useCallback을 사용해 매 렌더링마다 메모이제이션 된 함수를 넘겨주어 불필요한 리렌더링 방지

```tsx
// App.tsx
import React, { useState, ChangeEvent, useCallback } from "react";
import InputText from "./InputText";

export default function App() {
  const [number, setNumber] = useState(0);
  const [list, setList] = useState([
    { name: "apple", amount: 5000 },
    { name: "orange", amount: 1000 },
    { name: "watermelon", amount: 1500 },
    { name: "pineapple", amount: 500 },
  ]);

  function handleNumberChange(e: ChangeEvent<HTMLInputElement>) {
    setNumber(e.target.valueAsNumber);
  }

  function onSubmit(text: string) {
    setList((prev) => [...prev, { name: text, amount: number }]);
  }

  // 메모이제이션
  const memoedSubmit = useCallback(onSubmit, [number]);

  return (
    <div>
      <InputText onSubmit={memoedSubmit} />
      <input type="number" value={number} onChange={handleNumberChange} />

      <ul>
        {list.map((value) => (
          <li key={value.name}>
            {value.name} {value.amount}원
          </li>
        ))}
      </ul>
    </div>
  );
}

```

```tsx
import React, { ChangeEvent, useState } from "react";

const InputText = ({ onSubmit }: { onSubmit: (text: string) => void }) => {
  // 상태는 최소 컴포넌트에서 관리
  const [text, setText] = useState("");

  function handleTextChange(e: ChangeEvent<HTMLInputElement>) {
    setText(e.target.value);
  }

  function handleSubmit() {
    onSubmit(text);
  }

  return (
    <>
      <input type="text" value={text} onChange={handleTextChange} />
      <button type="submit" onClick={handleSubmit}>
        추가
      </button>
    </>
  );
};

export default InputText;

```

## **정리**

- 리액트 개발 도구를 사용하여 정적으로 생성된 컴포넌트 트리, 시간에 따른 동작,
불필요한 리렌더링 발생 여부 확인 가능
- 리액트 컴포넌트 구조가 복잡해지기 전에 틈틈이 리액트 개발 도구를 활용해 디버깅을 수행하자.
- 컴포넌트 트리와 프로파일링을 실행해 원하는 대로 렌더링 되고 있는지,
메모이제이션을 활용한 최적화는 잘 되고 있는지 확인할 것.
- **꾸준히 리액트 개발 도구를 가까이에 두고 디버깅을 손에 익히고 사전에 버그를 방지하고 꾸준히 성능을 개선**하려고 노력하자.
