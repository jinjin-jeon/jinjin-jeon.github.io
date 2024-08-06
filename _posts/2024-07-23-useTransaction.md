---
# permalink: /about/
title: "useTransaction"
categories: [React Native]
tag: [Mobile,Flutter]
sidebar:
    nav: "counts"
---
[React18업데이트][https://react.dev/blog/2022/03/29/react-v18]에서는 성능적으로 많이 향상되었지만 UX적으로 개발자 및 사용자에게 도움을 줄 수 있는 기능들이 몇개 추가 되었다. 추가된 디양한 기능 중 개발자라면 배워야하는 필수 기능이 useTransaction, Suspends라고 생각한다.(실전에서 가장 많이 사용합니다.) 여기에서는 *useTransaction*에 대해서 다뤄보도록 하겠습니다.

`useTransition`은 UI를 차단하지 않고 state를 업데이트할 수 있는 React 훅입니다.

컴포넌트의 최상위 레벨에서 `useTransition`을 호출하여 일부 state 업데이트를 트랜지션으로 표시합니다.

```javscript
import { useTransition } from 'react';

function TabContainer() {
  const [isPending, startTransition] = useTransition();
  function selectTab(nextTab) {
    startTransition(() => {
      setTab(nextTab);
    });
  }
  // ...
}
```

###  startTransition

`useTransition`이 반환하는 `startTransition` 함수를 사용하면 state 업데이트를 트랜지션으로 표시할 수 있습니다.

- scope`: 하나 이상의 [`set` 함수를 호출하여 일부 state를 업데이트하는 함수.](https://react-ko.dev/reference/react/useState#setstate) React는 매개변수 없이 `scope`를 즉시 호출하고 `scope` 함수 호출 중에 동기적으로 예약된 모든 state 업데이트를 트랜지션으로 표시합니다. 이는 [논블로킹](https://react-ko.dev/reference/react/useTransition#marking-a-state-update-as-non-blocking-transition)이고, [원치 않는 로딩을 표시하지 않을 것입니다.](https://react-ko.dev/reference/react/useTransition#preventing-unwanted-loading-indicators)

### isPending

-  보류 중인 트랜지션 이 있는지 여부를 알려주는  플래그를 선택합니다.



---

### 트랜지션에서 상위 컴포넌트 업데이트하기 

 `useTransition` 호출에서도 부모 컴포넌트의 state를 업데이트할 수 있습니다. 예를 들어, 이 `TabButton` 컴포넌트는 `onClick` 로직을 트랜지션으로 감쌉니다:

부모 컴포넌트가 `onClick` 이벤트 핸들러 내에서 state를 업데이트하기 때문에 해당 state 업데이트는 트랜지션으로 표시됩니다. 그렇기 때문에 앞의 예시처럼 ‘Posts’을 클릭한 다음 바로 ‘Contact’를 클릭할 수 있습니다. 선택한 탭을 업데이트하는 것은 트랜지션으로 표시되므로 사용자 상호작용을 차단하지 않습니다.

```
export default function TabButton({ children, isActive, onClick }) {
  const [isPending, startTransition] = useTransition();
  if (isActive) {
    return <b>{children}</b>
  }
  return (
    <button onClick={() => {
      startTransition(() => {
        onClick();
      });
    }}>
      {children}
    </button>
  );
```

### 트랜지션 중에 ‘보류중’ state 표시하기 

`useTransition`이 반환하는 `isPending` boolean 값을 사용하여 트랜지션이 진행 중임을 사용자에게 표시할 수 있습니다. 예를 들어, 탭 버튼은 특별한 ‘pending’ state를 가질 수 있습니다:

```
import { useTransition } from 'react';

export default function TabButton({ children, isActive, onClick }) {
  const [isPending, startTransition] = useTransition();
  if (isActive) {
    return <b>{children}</b>
  }
  if (isPending) {
    return <b className="pending">{children}</b>;
  }
  return (
    <button onClick={() => {
      startTransition(() => {
        onClick();
      });
    }}>
      {children}
    </button>
  );
}

```

### 원치 않는 로딩 표시 방지하기 

`PostsTab` 컴포넌트는 [Suspense가 도입된](https://react-ko.dev/reference/react/Suspense) 데이터 소스를 사용하여 일부 데이터를 가져옵니다. “Posts” 탭을 클릭하면 `PostsTab` 컴포넌트가 *중단*되어 가장 가까운 로딩 폴백이 나타납니다:

```
export default function TabContainer() {
  const [tab, setTab] = useState('about');
  return (
    <Suspense fallback={<h1>🌀 Loading...</h1>}>
      <TabButton
        isActive={tab === 'about'}
        onClick={() => setTab('about')}
      >
        About
      </TabButton>
      <TabButton
        isActive={tab === 'posts'}
        onClick={() => setTab('posts')}
      >
        Posts
      </TabButton>
      <TabButton
        isActive={tab === 'contact'}
        onClick={() => setTab('contact')}
      >
        Contact
      </TabButton>
      <hr />
      {tab === 'about' && <AboutTab />}
      {tab === 'posts' && <PostsTab />}
      {tab === 'contact' && <ContactTab />}
    </Suspense>
  );
}
```



### 에러 바운더리로 사용자에게 오류 표시하기 

`startTransition`에 전달된 함수가 에러를 발생시키면, [에러 바운더리](https://react-ko.dev/reference/react/Component#catching-rendering-errors-with-error-boundary)를 사용하여 사용자에게 에러를 표시할 수 있습니다. 에러 바운더리를 사용하려면 `useTransition`을 호출하는 컴포넌트를 에러 바운더리로 감싸면 됩니다. `startTransition`에 전달된 함수가 에러를 발생시키면 에러 바운더리에 대한 폴백이 표시됩니다.

```
import { useTransition } from "react";
import { ErrorBoundary } from "react-error-boundary";

export function AddCommentContainer() {
  return (
    <ErrorBoundary fallback={<p>⚠️Something went wrong</p>}>
      <AddCommentButton />
    </ErrorBoundary>
  );
}

function addComment(comment) {
  // For demonstration purposes to show Error Boundary
  if (comment == null) {
    throw new Error("Example Error: An error thrown to trigger error boundary");
  }
}

function AddCommentButton() {
  const [pending, startTransition] = useTransition();

  return (
    <button
      disabled={pending}
      onClick={() => {
        startTransition(() => {
          // Intentionally not passing a comment
          // so error gets thrown
          addComment();
        });
      }}
    >
      Add comment
    </button>
  );
}

```



### `startTransition`에 전달한 함수는 즉시 실행됩니다 

**1, 2, 3을 출력할 것으로 예상됩니다.** `startTransition`에 전달한 함수는 지연되지 않습니다. 브라우저의 `setTimeout`과 달리 나중에 콜백을 실행하지 않습니다. React는 함수를 즉시 실행하지만, 함수가 실행되는 동안 예약된 모든 state 업데이트는 트랜지션으로 표시됩니다. 이렇게 작동한다고 상상하면 됩니다:

```
console.log(1);
startTransition(() => {
  console.log(2);
  setPage('/about');
});
console.log(3);
```







