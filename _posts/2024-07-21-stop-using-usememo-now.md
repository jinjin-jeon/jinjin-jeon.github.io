---
# permalink: /about/
title: "useMemo 알고 사용하자"
categories: [React Native]
tag: [Mobile]
sidebar:
    nav: "counts"
---
---
**[Medium](https://medium.com/)**을 훑어보던중 흥미로운 주제가 있어서 번역해보았다! ReactNative의 기본개념인 useMemo/useCallBack에 대한 내용인데 성능적인 이슈를 다룬 블로그 내용이었다. 쭉 훑어 보았지만 아직 크게 와닿지는 않는다. 단순 지표만 보았을때는 -우리가 속고 있엇구나?- 라고 느꼈지만 나는 언제나 그랬든 습관의 무서움을 잘알고 있다. 실전 프로그램을 작성할때 단순한 알고리즘(단편적인 로직)으로 작성한적이 결코 단한번도 없다. 얽히고 설키고 꼬인 로직을 푸는것보다 뭔가 명료한 컴퍼니룰(company-rule)이 더 좋지 않을까 싶다. 간단한 예로들면 enum이 javascript에서 성능적으로 좋지 않다고 해도 개발자들이 유지보수하기 쉬운 코드가 가장 잘짜놓은 프로그램이 라고 생각한다. (추후 해당 내용에 대해 정리하겠다.) 그리고 심지어 실전에서는 enum의 key값을 한글로 지정하기도한다. 아래 내용에 따르면 단순히 동등연산자(cost가 낮은계산)와 같은 로직에는 추가가 필요업지만, 실제 협업을 하다보면 hook dependency를 서로 잘못 참조 해줘 무한루프가 발생하거나 복잡도가 더많이 늘어나는 경우도 허다하다. 


원본글 : https://javascript.plainenglish.io/stop-using-usememo-now-e5d07d2bbf70


---

최적화를 지나치게 추구하는 것은 React 애플리케이션에서 지옥이 될 수 있습니다. 우리는 매일 코드 성능을 높이려는 수많은 불필요한 최적화에 직면합니다.(React에서 가장 중요한 작업) 그러나 이러한 최적화는 성능을 향상시키기는커녕 코드만 더 복잡하고 이해하기 어렵게 만듭니다. 어떤 개발자들은 `useMemo`와 `useCallback`을 기본 스타일 가이드에 포함시키기까지 하는데, 이는 코드 작성이 더 쉬워진다고 생각하기 때문입니다. 하지만 이 함정에 빠지지 마세요—이러한 훅을 과도하게 사용하면 오히려 애플리케이션 속도를 저하시킬 수 있습니다. 메모이제이션(memoization)은 공짜가 아니며, 그 자체로 비용이 따릅니다.

이 글에서는 많은 개발자들이 어떻게 `useMemo` 훅을 과용하고 있는지, 그리고 이러한 실수를 피할 수 있는 팁을 공유하려고 합니다. 저도 처음 이러한 실수를 깨달았을 때, 어리석었다고 느꼈지만, 그 경험을 통해 많은 것을 배웠습니다. 



### 우리는 왜 `useMemo`를 사용할까요?

`useMemo`는 컴포넌트가 다시 렌더링될 때 계산 결과를 캐시할 수 있도록 해주는 훅입니다. 이는 주로 성능 최적화를 위해 사용되며, 종종 `React.memo`, `useCallback`, 디바운싱(debouncing), 동시성 렌더링(concurrent rendering) 등과 함께 사용됩니다.

이 훅이 실제로 유용하고 중요한 역할을 하는 상황도 있지만, 많은 개발자들이 이를 잘못 사용하고 있습니다. 그들은 모든 변수를 `useMemo`로 감싸며 무작위 최적화가 성과를 낼 것이라고 기대합니다. 하지만 예상대로 이러한 접근 방식은 가독성을 떨어뜨리고 메모리 사용량을 증가시킬 뿐입니다.

> 엘론 머스크는 이렇게 말했습니다. “똑똑한 엔지니어가 저지르는 가장 흔한 실수는 존재해서는 안 될 것을 최적화하는 것이다.”

**또 한 가지 중요한 점**은 `useMemo`가 재렌더링 단계에서만 가치를 제공한다는 것입니다. 초기화 단계에서는 메모이제이션이 오히려 애플리케이션을 느리게 만들며, 이 효과는 시간이 지날수록 누적될 수 있습니다. 그래서  “메모이제이션은 공짜가 아니다”라고 말한 것입니다.



### 예제 1: `NavTabs`에서 `useMemo` 사용

```
export const NavTabs = ({ tabs, className, withExpander }) => {
  const currentMainPath = useMemo(() => {
    return pathname.split("/")[1];
  }, [pathname]);
  const isCurrentMainPath = useMemo(() => {
    return currentMainPath === pathname.substr(1);
  }, [pathname, currentMainPath]);

  return (
    <StyledWrapper>
      <Span fontSize={18}>
        {isCurrentMainPath ? (
          t(currentMainPath)
        ) : (
          <StyledLink to={`/${currentMainPath}`}>
            {t(currentMainPath)}
          </StyledLink>
        )}
      </Span>
    </StyledWrapper>
  );
};
```

이 코드에서 `useMemo`는 `currentMainPath`와 `isCurrentMainPath`에 사용되고 있습니다. 이 경우 `useMemo`가 불필요한 이유는 다음과 같습니다:

1. **기본값 처리**: `currentMainPath`와 `isCurrentMainPath`의 계산은 단순한 연산 (`split`과 `===`)이므로, 비용이 큰 연산이 아닙니다. `useMemo`를 사용하는 오버헤드가 성능 이득보다 클 가능성이 높습니다.
2. **JSX에서 직접 사용**: 이 메모이즈된 값들은 JSX에서 직접 사용되며, 자식 컴포넌트에 전달되지 않습니다. 즉, 이 값들이 다른 컴포넌트의 재렌더링에 영향을 미치지 않으므로 `useMemo`가 불필요합니다.
3. **비용이 큰 계산 없음**: 복잡한 계산이나 반복이 없으므로 메모이제이션이 필요하지 않습니다.

**결론**: 이 예제에서는 `useMemo`를 제거해도 괜찮습니다. 계산이 너무 간단하고, `useMemo`를 사용하는 것이 복잡성만 추가합니다.

### 예제 2: `Client`에서 `useMemo` 사용

```
javascript
코드 복사
export const Client = ({ clientId, ...otherProps }) => {
  const tabs = useMemo(
    () => [
      {
        label: t("client withdrawals"),
        path: `/clients/${clientId}/withdrawals`
      },
      ...
    ],
    [t, clientId]
  );
  
  return (
    <>
      ...
      <NavTabs tabs={tabs} />
    </>
  )
}

export const NavTabs = ({ tabs, className, withExpander }) => {
  return (
    <Wrapper className={className} withExpander={withExpander}>
      {tabs.map((tab) => (
        <Item
          key={tab.path}
          to={tab.path}
          withExpander={withExpander}
        >
          <StyledLabel>{tab.label}</StyledLabel>
        </Item>
      ))}
    </Wrapper>
  );
};
```

이 경우, `useMemo`는 `tabs` 배열을 메모이즈하기 위해 사용되고 있습니다. 여기에서 `useMemo`를 사용하는 주된 목적은 `NavTabs` 컴포넌트의 불필요한 재렌더링을 방지하는 것입니다.

다음은 `useMemo`가 어느 정도 효과적일 수 있지만 더 나은 최적화 방법이 있는 이유입니다:

1. **참조 유지**: `tabs`를 메모이즈함으로써 `tabs` 배열의 참조를 유지하고 있습니다. 이는 `NavTabs`에 전달될 때 유용할 수 있습니다.
2. **최적화 맥락**: `NavTabs`가 가벼운 컴포넌트라 하더라도, `Client`가 자주 재렌더링된다면 불필요한 재렌더링을 방지하는 것이 유용할 수 있습니다. 그러나 `useMemo`만으로는 `NavTabs`의 재렌더링을 완전히 막을 수 없습니다.<u> `NavTabs`를 `React.memo`로 감싸는 것</u>이 더 효과적입니다.
3. **React.memo**: `React.memo`를 사용하면, `NavTabs`는 `tabs`가 변경되지 않는 한 재렌더링되지 않습니다. 따라서 `tabs`가 실제로 변경될 때만 렌더링됩니다.

**결론**: `useMemo`는 `tabs`의 재생성을 방지하는 데 도움이 되지만, 성능 최적화를 완전히 이뤄내려면 `NavTabs`를 `React.memo`로 감싸는 것이 좋습니다.

### 일반적인 가이드라인

- **비용이 적은 계산에 `useMemo` 사용 피하기**: `useMemo`를 사용해도 성능 이득이 적다면, 오히려 성능에 악영향을 미칠 수 있습니다.
- **메모이제이션 필요 여부 판단**: 메모이제이션이 필요한지 확신이 없다면, 우선 메모이제이션 없이 시작하고 문제 발생 시 점진적으로 최적화하는 것이 좋습니다.
- **컴포넌트 트리 깊이 영향 없음**: 메모이즈된 값이 JSX에서만 사용되고 다른 컴포넌트로 전달되지 않는다면, 대부분 최적화가 필요 없습니다.
- **의존성 배열의 빈번한 변경**: 의존성 배열이 너무 자주 변경된다면 `useMemo`는 성능 이득을 제공하지 않을 수 있습니다.

이러한 최적화 기법들을 적용하면, `useMemo`와 다른 성능 최적화 기술을 더 효과적이고 효율적으로 활용할 수 있습니다.



### `useMemo`를 올바르게 사용하는 팁

React 컴포넌트는 매번 재렌더링될 때마다 컴포넌트의 본문이 실행됩니다. 이는 props나 state가 변경될 때 발생합니다. 보통, 렌더링 동안 비용이 많이 드는 계산을 피하려고 합니다. 이렇게 하지 않으면 컴포넌트의 성능이 저하될 수 있습니다.

하지만 대부분의 계산은 여전히 매우 빠르기 때문에 실제로 `useMemo`를 어디에 사용해야 할지 이해하기 어려울 수 있습니다. 최적화를 효과적이고 목적적으로 적용하려면 먼저 문제를 식별해야 합니다. 또한, `useCallback`, `React.memo`, 디바운싱, 코드 스플리팅, 지연 로딩 등의 다른 기법도 잊지 말고 고려해야 합니다.

다음은 간단한 예제입니다. `doCalculation`은 인위적으로 느리게 만든 함수로, 500ms가 걸려서 랜덤 숫자를 반환합니다. 이 코드에서 어떤 문제가 있을까요? 맞아요, 값이 업데이트될 때마다 무거운 함수를 재계산하게 되고, 이로 인해 입력을 사용하는 데 매우 어려움을 겪습니다.

```
function doCalculation() {
  const now = performance.now();
  while (performance.now() - now < 500) {
    // 500ms 동안 아무것도 하지 않음
  }

  return Math.random();
}

export default function App() {
  const [value, setValue] = useState(0);
  const calculation = doCalculation();

  return (
    <div>
      <input value={value} onChange={(e) => setValue(e.target.value)} />
      <h1>{`Calculation is ${calculation}`}</h1>
    </div>
  );
}
```

이 문제를 해결하려면, `doCalculation`을 `useMemo`로 감싸서 의존성 배열 없이 사용하는 것이 좋습니다. 이렇게 하면 `calculation`의 값을 재계산하지 않고, 컴포넌트가 재렌더링될 때 `doCalculation`이 반복적으로 호출되는 것을 방지할 수 있습니다.

```
import React, { useMemo, useState } from 'react';

function doCalculation() {
  const now = performance.now();
  while (performance.now() - now < 500) {
    // 500ms 동안 아무것도 하지 않음
  }

  return Math.random();
}

export default function App() {
  const [value, setValue] = useState(0);
  const calculation = useMemo(() => doCalculation(), []);

  return (
    <div>
      <input value={value} onChange={(e) => setValue(e.target.value)} />
      <h1>{`Calculation is ${calculation}`}</h1>
    </div>
  );
}
```

이렇게 하면, `doCalculation`은 컴포넌트의 첫 렌더링 시에만 실행되며, 이후의 렌더링에서는 메모이즈된 결과를 사용합니다. 이는 `calculation`의 값을 재계산하지 않고 성능을 개선하는 데 도움을 줄 수 있습니다.

(사실 [useTransaction](https://react-ko.dev/reference/react/useTransition)을 사용해서 개발하는게 UX적으로는 더 깔끔하다. 추후에 한번 다뤄 보겠다.)

