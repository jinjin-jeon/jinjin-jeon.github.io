---
# permalink: /about/
title: "react-native-root-siblings"
categories: [React Native, StoryBook]
tag: [Mobile,StoryBook]
sidebar:
    nav: "counts"

---

React Native를 개발하면서 가장 잘쓰고 애용하는 라이브러리를 소개 하고자 한다. RootSibing은 사용자가 작성한 컨포넌트를 root로 노출 시켜준다! 우리가 보통 알고 있는 [Portal](https://react.dev/reference/react-dom/createPortal)개념과 동일하다고 생각하시면 이해가 빠를거다! 다만 좋은점은 State에 의존적이지 않게 전역으로 작성할수 있고, 중첩이 되는게 가장 큰 장점이 아닌가 싶다. Toast와 간단한 Dialog에서 사용하며, 복잡한 모달의 경우 [gorhom/bototm-sheet](https://github.com/gorhom/react-native-bottom-sheet)을 사용하는걸 추천한다. 극단적인 상위 root(Provider가 있다.)로 올리기때문에 잘못 설정하면 SafeArea나 Gesture와 같은 기능이 동작 하지 않는다.(그래서 보통 sibling-component는 React 기본 애니메이션으로 구성했다.)

라이브러리 주소는 [다음](https://github.com/magicismight/react-native-root-siblings) 과 같다.



## react-native-root-siblings 사용하기

```
npm install react-native-root-siblings
or
yarn add react-native-root-siblings
```

### 1. Root Provider Portal 추가

```
import { RootSiblingParent } from 'react-native-root-siblings';

const App = () => {
  return (
    <RootSiblingParent>
      <AppStack />
    </RootSiblingParent>
  );
};
export default App;
```

### 2. 컨포넌트화 작성

```
import RootSiblings from 'react-native-root-siblings';

let _loginToastSibling: RootSiblings | null;
const show = (onClose?: any) => {
  _loginToastSibling = new RootSiblings(
    (
      <LoginBottomSheet
        onClose={() => {
          hide();
          onClose?.();
        }}
      />
    )
  );
  return _loginToastSibling;
};
const hide = () => {
  if (_loginToastSibling instanceof RootSiblings) {
    _loginToastSibling.destroy();
    _loginToastSibling = null;
  } else {
    console.warn(
      `Toast.hide expected a \`RootSiblings\` instance as argument.`
    );
  }
};
export default { show, hide };
```

### 3. 사용

```
import {show} from './compoents'

const App = ()=>{
 	useEffect(()=>{
 		show()
 	},[])
}
```





### 기억해야 할 사항

- 👹메모리 누수를 방지하기 위해 RootSibling 인스턴스를 꼭 해제 하세요.👹
