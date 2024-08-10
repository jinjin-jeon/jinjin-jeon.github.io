---
# permalink: /about/
title: "react-native story-book (2)"
categories: [React Native, StoryBook]
tag: [Mobile,StoryBook]
sidebar:
    nav: "counts"

---

 `forwardRef`는 렌더링 함수를 인자로 받습니다. React는 `props` 및 `ref`와 함께 이 함수를 호출합니다:

```
const MyInput = forwardRef(function MyInput(props, ref) {

  return (
    <label>
      {props.label}
      <input ref={ref} />
    </label>

  );

});
```

#### **Parameters**매개변수 

- `props`: 부모 컴포넌트가 전달한 props입니다.
- `ref`: 부모 컴포넌트가 전달한 `ref` 속성입니다. `ref`는 객체나 함수일 수 있습니다. 부모 컴포넌트가 ref를 전달하지 않은 경우 `null`이 됩니다. 받은 `ref`를 다른 컴포넌트에 전달하거나 [`useImperativeHandle`](https://ko.react.dev/reference/react/useImperativeHandle)에 전달해야 합니다.

#### **Returns**반환값 

`forwardRef`는 JSX에서 렌더링할 수 있는 React 컴포넌트를 반환합니다. 일반 함수로 정의된 React 컴포넌트와 달리, `forwardRef`가 반환하는 컴포넌트는 `ref` prop을 받을 수 있습니다.

------

## 사용법 

### 부모 컴포넌트에 DOM 노드 노출하기 

```
import { forwardRef } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  const { label, ...otherProps } = props;
  return (
    <label>
      {label}
      <input {...otherProps} ref={ref} />
    </label>

  );

});
```

이렇게 하면 부모 `Form` 컴포넌트가 `MyInput`에 의해 노출된 `<input>` DOM 노드에 접근할 수 있습니다:

```
function Form() {

  const ref = useRef(null);

  function handleClick() {
    ref.current.focus();
  }

  return (
    <form>
      <MyInput label="Enter your name:" ref={ref} />
      <button type="button" onClick={handleClick}>
        Edit
      </button>
    </form>

  );

}
```

이 `Form` 컴포넌트는 `MyInput`에 대한 [ref를 전달](https://ko.react.dev/reference/react/useRef#manipulating-the-dom-with-a-ref)합니다. `MyInput` 컴포넌트는 해당 ref를 `<input>` 브라우저 태그에 전달합니다. 그 결과 `Form` 컴포넌트는 해당 `<input>` DOM 노드에 접근하여 이 노드에서 [`focus()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/focus)를 호출할 수 있습니다.

컴포넌트 내부의 DOM 노드에 대한 ref를 노출하면 나중에 컴포넌트의 내부를 변경하기가 더 어려워진다는 점에 유의하세요. 일반적으로 버튼이나 텍스트 input과 같이 재사용 가능한 저수준 컴포넌트에서 DOM 노드를 노출하지만, 아바타나 댓글 같은 애플리케이션 레벨의 컴포넌트에서는 노출하고 싶지 않을 것입니다.



------

### DOM 노드 대신 명령형 핸들 노출하기 

Instead of exposing an entire DOM node, you can expose a custom object, called an *imperative handle,* with a more constrained set of methods. To do this, you’d need to define a separate ref to hold the DOM node: 전체 DOM 노드를 노출하는 대신 보다 제한된 메서드 집합을 사용하여 *명령형 핸들*이라고 하는 사용자 정의 객체를 노출할 수 있습니다. 이렇게 하려면 DOM 노드를 보유할 별도의 ref를 정의해야 합니다:

```
const MyInput = forwardRef(function MyInput(props, ref) {

  const inputRef = useRef(null);



  return <input {...props} ref={inputRef} />;

});
```

Pass the `ref` you received to [`useImperativeHandle`](https://ko.react.dev/reference/react/useImperativeHandle) and specify the value you want to expose to the `ref`: 받은 `ref`를 [`useImperativeHandle`](https://ko.react.dev/reference/react/useImperativeHandle)에 전달하고 ref에 노출할 값을 지정합니다:

```
import { forwardRef, useRef, useImperativeHandle } from 'react';



const MyInput = forwardRef(function MyInput(props, ref) {

  const inputRef = useRef(null);



  useImperativeHandle(ref, () => {

    return {

      focus() {

        inputRef.current.focus();

      },

      scrollIntoView() {

        inputRef.current.scrollIntoView();

      },

    };

  }, []);



  return <input {...props} ref={inputRef} />;

});
```

If some component gets a ref to `MyInput`, it will only receive your `{ focus, scrollIntoView }` object instead of the DOM node. This lets you limit the information you expose about your DOM node to the minimum. 일부 컴포넌트가 `MyInput`에 대한 ref를 받으면 DOM 노드 대신 `{ focus, scrollIntoView }` 객체만 받습니다. 이를 통해 DOM 노드에 대해 노출하는 정보를 최소한으로 제한할 수 있습니다.

### 함정

**ref를 과도하게 사용하지 마세요**. 노드로 스크롤하기, 노드에 초점 맞추기, 애니메이션 트리거하기, 텍스트 선택하기 등 prop으로 표현할 수 없는 필수적인 동작에만 ref를 사용해야 합니다.

 **prop으로 무언가를 표현할 수 있다면 ref를 사용해서는 안 됩니다.** 예를 들어, `Modal` 컴포넌트에서 `{ open, close }`와 같은 명령형 핸들을 노출하는 대신 `<Modal isOpen={isOpen} />`과 같이 prop `isOpen`을 사용하는 것이 더 좋습니다. [Effect](https://ko.react.dev/learn/synchronizing-with-effects)는 props를 통해 명령형 동작을 노출하는 데 도움이 될 수 있습니다.

------

## **Troubleshooting**문제 해결 

### 컴포넌트가 `forwardRef`로 감싸져 있지만, 컴포넌트의 `ref`는 항상 `null`입니다. 

이는 일반적으로 받은 `ref` 를 실제로 사용하는 것을 잊어버렸음을 의미합니다.

ref`: 예를 들어, 이 컴포넌트는 `ref`로 아무 작업도 하지 않습니다:

```
const MyInput = forwardRef(function MyInput({ label }, ref) {
  return (
    <label>
      {label}
      <input />
    </label>
  );
});
```

이 문제를 해결하려면 `ref` 를 DOM 노드나 ref를 받을 수 있는 다른 컴포넌트로 전달하세요:

```
const MyInput = forwardRef(function MyInput({ label }, ref) {
  return (
    <label>
      {label}
      <input ref={ref} />
    </label>
  );
});
```

일부 로직이 조건부인 경우 `MyInput`에 대한 `ref`가 `null`일 수도 있습니다:

```
const MyInput = forwardRef(function MyInput({ label, showInput }, ref) {
  return (
    <label>
      {label}
      {showInput && <input ref={ref} />}
    </label>
  );
});
```

 `showInput`이 `false`이면 ref가 어떤 노드로도 전달되지 않으며 `MyInput`에 대한 ref는 비어 있게 됩니다. 이 예제의 `Panel`처럼 조건이 다른 컴포넌트 안에 숨겨져 있는 경우 특히 이 점을 놓치기 쉽습니다:

```
const MyInput = forwardRef(function MyInput({ label, showInput }, ref) {
  return (
    <label>
      {label}
      <Panel isExpanded={showInput}>
        <input ref={ref} />
      </Panel>
    </label>
  );
});
```

###  😪 타입 스크립트

```
interface Props {
  index: string[];
  onChange: (index: number) => void;
}

export interface IndexScrollerRef {
  onScrollEvent(isStart: boolean): void;
}

const IndexScroller = forwardRef(props: Props, ref: React.Ref<IndexScrollerRef>) => {
 
  const onScrollEvent = () =>{
    
  }

  useImperativeHandle(ref, () => ({
    onScrollEvent: onScrollEvent,
  }));
```



