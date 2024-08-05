---
# permalink: /about/
title: "재조정 (Reconciliation)"
categories: [React Native]
tag: [Mobile]
sidebar:
    nav: "counts"
---
**Reconciliation**은 React에서 가장 중요한 개념이라고 볼 수 있다. React를 개발하다 보면, 개발이 너무 쉽고 단순 명료해지는것을 해가 지날수록 느낄수 있었다. 하지만 재조정 개념을 정확히 이해하지 못하면 웹뷰만도 못한 앱들이 나오게 된다. React중요한 개념이라.. 가상Dom,  State/Props 와 같은 다양한 내용들이 있겠지만, 재조정 만큼은 중요하지 않다고 생각한다!  아래 내용은 react 공식문서를 요약해놓은 글을 미리 알려드립니다.

https://ko.legacy.reactjs.org/docs/reconciliation.html#recursing-on-children



React는 **선언적** API를 제공하기 때문에 갱신이 될 때마다 매번 무엇이 바뀌었는지를 걱정할 필요가 없습니다. 이는 애플리케이션 작성을 무척 쉽게 만들어주지만, React 내부에서 어떤 일이 일어나고 있는지는 명확히 눈에 보이지 않습니다. 이 글에서는 우리가 React의 “비교 (diffing)” 알고리즘을 만들 때 어떤 선택을 했는지를 소개합니다. 이 비교 알고리즘 덕분에 컴포넌트의 갱신이 예측 가능해지면서도 고성능 앱이라고 불러도 손색없을 만큼 충분히 빠른 앱을 만들 수 있습니다.



![img](https://html-eslint.org/icon-incorrect.6e4fb3c4.svg)리스트의 맨 앞에 엘리먼트를 추가하는 경우 성능이 좋지 않습니다. 예를 들어, 아래의 두 트리 변환은 형편없이 작동합니다. React는 `<li>Duke</li>`와 `<li>Villanova</li>` 종속 트리를 그대로 유지하는 대신 모든 자식을 변경합니다. 이러한 비효율은 문제가 될 수 있습니다.

```
<ul>
  <li>Duke</li>
  <li>Villanova</li>
</ul>

<ul>
  <li>Connecticut</li>
  <li>Duke</li>
  <li>Villanova</li>
</ul>
```

![img](https://html-eslint.org/icon-correct.6ef01b36.svg) React는 `key` 속성을 지원합니다. 자식들이 key를 가지고 있다면, React는 key를 통해 기존 트리와 이후 트리의 자식들이 일치하는지 확인합니다. 예를 들어, 위 비효율적인 예시에 `key`를 추가하여 트리의 변환 작업이 효율적으로 수행되도록 수정할 수 있습니다.

```
<ul>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
</ul>

<ul>
  <li key="2014">Connecticut</li>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
</ul>
```

---

## Key

Key는 React가 어떤 항목을 변경, 추가 또는 삭제할지 식별하는 것을 돕습니다. key는 엘리먼트에 안정적인 고유성을 부여하기 위해 배열 내부의 엘리먼트에 지정해야 합니다.



![img](https://html-eslint.org/icon-incorrect.6e4fb3c4.svg)**예시: 잘못된 Key 사용법**

```
function ListItem(props) {
  const value = props.value;
  return (
    // 틀렸습니다! 여기에는 key를 지정할 필요가 없습니다.
    <li key={value.toString()}>
      {value}
    </li>
  );
}

function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    // 틀렸습니다! 여기에 key를 지정해야 합니다.
    <ListItem value={number} />
  );
  return (
    <ul>
      {listItems}
    </ul>
  );
}
```

![img](https://html-eslint.org/icon-correct.6ef01b36.svg) **예시: 올바른 Key 사용법**

```
function ListItem(props) {
  // 맞습니다! 여기에는 key를 지정할 필요가 없습니다.
  return <li>{props.value}</li>;
}

function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    // 맞습니다! 배열 안에 key를 지정해야 합니다.
    <ListItem key={number.toString()} value={number} />
  );
  return (
    <ul>
      {listItems}
    </ul>
  );
}
```
