---
# permalink: /about/
title: "React.memo"
categories: [React Native]
tag: [Mobile,Flutter]
sidebar:
    nav: "counts"
---


React.memo에대한 잘 정리된 글을 이관했습니다. 

https://medium.com/wantedjobs/react-profiler%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EC%97%AC-%EC%84%B1%EB%8A%A5-%EC%B8%A1%EC%A0%95%ED%95%98%EA%B8%B0-5981dfb3d934

# React Developer Tools 익스텐션 설치하기

크롬 브라우저에서 Profiler 기능을 이용하기 위해서 React Developer Tools 익스텐션을 설치합니다.

- 설치하러 가기: https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi?hl=ko

![img](https://miro.medium.com/v2/resize:fit:1400/1*BrBbXxTemxhdQSC_03GfNw.png)

# Profiler로 성능 측정하기

React 컴포넌트 성능을 측정하기 위해서, 비교 가능한 2개의 컴포넌트를 만들어보겠습니다. 우리는 두 컴포넌트의 성능 차이를 쉽게 비교하기 위해서 다음과 같은 컴포넌트 2개를 생성합니다.

PhotoOne 컴포넌트는 하위 컴포넌트를 사용하지 않고 하나의 컴포넌트에서 모든 데이터를 렌더링하도록 구현합니다. 그다음 PhotoTwo 컴포넌트는 하위 컴포넌트를 적절하게 분리하여 각각의 하위 컴포넌트에서 데이터를 렌더링하도록 구현합니다.

```
// PhotoOne.js
const PhotoOne = ({ message = "", photos = [] }) => {
  return (
    <div>
      <h1>PhotoOne</h1>
      <p>{message}</p>
      <ul>
        {photos.map(photo => {
          return (
            <li key={photo.id}>
              <img src={photo.url} alt={photo.title} />
            </li>
          );
        })}
      </ul>
    </div>
  );
};
export default PhotoOne;

// PhotoTwo.js
const Message = ({ message }) => {
  return <p>{message}</p>;
};

const ListItem = ({ photo }) => {
  return (
    <li key={photo.id}>
      <img src={photo.url} alt={photo.title} />
    </li>
  );
};

const List = ({ photos }) => {
  return (
    <ul>
      {photos.map(photo => (
        <ListItem key={photo.id} photo={photo} />
      ))}
    </ul>
  );
};

const PhotoTwo = ({ message = "", photos = [] }) => {
  return (
    <div>
      <h1>PhotoTwo</h1>
      <Message message={message} />
      <List photos={photos} />
    </div>
  );
};
export default PhotoTwo;

// App.js
import PhotoOne from "./PhotoOne";
import PhotoTwo from "./PhotoTwo";

function App() {
  const [message, setMessage] = React.useState("");
  const [photos, setPhotos] = React.useState([]);
 
  return (
    <div>
      <input
        value={message}
        onChange={event => setMessage(event.target.value)}
      />
      <div className="photos">
        <PhotoOne photos={photos} message={message} />
        <PhotoTwo photos={photos} message={message} />
      </div>
    </div>
  );
}

```





이제 렌더링 성능을 측정하기 위해 화면에 출력할 많은 데이터가 필요합니다. 코드에 dummy 데이터를 직접 하드 코딩하여 사용할 수도 있습니다. 하지만 우리는 간단하게 무료로 제공하는 Open API를 사용하여 데이터를 가져와서 화면에 출력해보겠습니다.

다음과 같이 [jsonplaceholder](https://jsonplaceholder.typicode.com/) 서비스를 이용하여 Photo 데이터를 불러옵니다. Photo 데이터는 5천 건이므로 성능을 측정하기에 충분합니다.

```
React.useEffect(() => {     
  fetch("https://jsonplaceholder.typicode.com/photos")         
    .then(response => response.json())
    .then(setPhotos);   
}, [setPhotos]);
```

지금까지 구현한 App.js의 전체 코드는 다음과 같습니다.

```
import React from "react";
import "./App.css";

import PhotoOne from "./PhotoOne";
import PhotoTwo from "./PhotoTwo";

function App() {
  const [message, setMessage] = React.useState("");
  const [photos, setPhotos] = React.useState([]);

  React.useEffect(() => {
    fetch("https://jsonplaceholder.typicode.com/photos")
      .then(response => response.json())
      .then(setPhotos);
  }, [setPhotos]);
 
  return (
    <div>
      <input
        value={message}
        onChange={event => setMessage(event.target.value)}
      />
      <div className="photos">
        <PhotoOne photos={photos} message={message} />
        <PhotoTwo photos={photos} message={message} />
      </div>
    </div>
  );
}

export default App;
```



이제 React 앱을 실행하고 성능을 측정해보겠습니다. 크롬 개발자 도구를 열어서 Profiler 탭으로 이동합니다.

![img](https://miro.medium.com/v2/resize:fit:1400/1*P8zbBDCk9IXoMCe0_OoyuA.png)

Profiler 탭을 열어보면 위와 같이 빈화면으로 시작합니다. 성능 데이터를 기록하고 측정하기 위해서 우리는 프로파일링을 수행해야합니다.

레코드 버튼을 클릭하여 프로파일링을 시작합니다. 이제 Profiler는 컴포넌트가 재렌더링이 될 때마다 성능을 기록합니다. 프로파일링을 끝내기 위해서 다시 중지 버튼을 클릭합니다.

프로파일링이 끝나면 다음과 같이 성능 데이터가 나타납니다. 혹시 Profiler의 기능에 대해서 더 궁금하거나, 자세하게 알고 싶으면 [Reading performance data 문서](https://reactjs.org/blog/2018/09/10/introducing-the-react-profiler.html#reading-performance-data)를 참고하세요.

![img](https://miro.medium.com/v2/resize:fit:1400/1*PfTkYVgSKPkGSiyPEnUd4A.png)

위의 Flamegraph 차트를 보면 App 컴포넌트는 렌더링하는데 191.3ms가 걸렸습니다. 그리고 PhotoOne 컴포넌트는 73.5ms가 걸렸고, PhotoTwo 컴포넌트는 116.6ms가 걸렸습니다. PhotoOne 보다 PhotoTwo 컴포넌트의 렌더링 시간이 더 오래 걸렸다는 것을 알 수 있습니다.

![img](https://miro.medium.com/v2/resize:fit:1400/1*q2DG6X2-9779gQj4RBWNow.png)

그리고 Ranked 차트를 살펴보면, PhotoOne 컴포넌트의 렌더링 시간이 제일 오래걸렸다는 것도 알 수 있습니다. 여기까지만 보았을 때는 PhotoOne 컴포넌트의 성능이 더 좋아 보입니다.

# React.memo를 사용하여 성능 최적화하기

이제 컴포넌트 성능을 최적화 해보겠습니다.

React Developer Tools 설정 버튼(톱니바퀴 아이콘)을 클릭합니다. 그리고 Highlight updates when compoennts render 옵션을 켜줍니다. 이 옵션은 렌더링이 발생하는 컴포넌트를 브라우저 화면에 표시해줍니다.

![img](https://miro.medium.com/v2/resize:fit:1400/1*eRCfJwQ6RvieunOEJzxcOQ.png)

아래 GIF 이미지로 캡쳐한 화면을 보면 message 데이터가 변할 때마다 모든 컴포넌트의 렌더링이 발생하는 것을 알 수 있습니다.

![img](https://miro.medium.com/v2/resize:fit:1400/1*oYovZQ3s01I8noPcLvPKmg.gif)

메세지를 입력할 때마다, Message 컴포넌트만 재렌더링하면 되는데, 전체 컴포넌트가 재렌더링되고 있습니다. 우리는 어떤 부분이 성능 저하를 일으키는 원인인지 알 수 있습니다.

이제 컴포넌트의 렌더링 성능 최적화를 해줄 수 있는 React.memo 기능을 사용해보겠습니다. 다음과 같이 PhotoOne과 PhotoTwo 컴포넌트를 수정하여 리렌더링을 방지합니다.



```
// PhotoOne.js
const PhotoOne = ({ message = "", photos = [] }) => {
  return (
    <div>
      <h1>PhotoOne</h1>
      <p>{message}</p>
      <ul>
        {photos.map(photo => {
          return (
            <li key={photo.id}>
              <img src={photo.url} alt={photo.title} />
            </li>
          );
        })}
      </ul>
    </div>
  );
};

export default React.memo(PhotoOne);

// PhotoTwo.js
const Message = React.memo(({ message }) => {
  return <p>{message}</p>;
});

const ListItem = React.memo(({ photo }) => {
  return (
    <li key={photo.id}>
      <img src={photo.url} alt={photo.title} />
    </li>
  );
});

const List = React.memo(({ photos }) => {
  return (
    <ul>
      {photos.map(photo => (
        <ListItem key={photo.id} photo={photo} />
      ))}
    </ul>
  );
});

const PhotoTwo = ({ message = "", photos = [] }) => {
  return (
    <div>
      <h1>PhotoTwo</h1>
      <Message message={message} />
      <List photos={photos} />
    </div>
  );
};

export default React.memo(PhotoTwo);
```



그리고나서 Profiler에서 성능을 다시 측정합니다.

![img](https://miro.medium.com/v2/resize:fit:1400/1*IJoy70AZYrValXUoYyw9VA.png)

이제 App 컴포넌트는 렌더링하는데 59ms가 걸렸습니다. 이전보다 3배 이상 빨라졌습니다. 그리고 PhotoOne 컴포넌트는 59ms, PhotoTwo는 0.1ms가 걸렸습니다. 이전보다 PhotoTwo의 렌더링 속도가 매우 빨라진 것을 알수 있습니다.

PhotoTwo의 렌더링 속도가 빨라진 이유는 Ranked 차트를 보면 더 자세하게 알 수 있습니다. PhotoOne는 전체 컴포넌트를 재렌더링했지만, PhotoTwo는 Message 컴포넌트만 재렌더링했기 때문에 성능이 매우 좋아졌습니다.

![img](https://miro.medium.com/v2/resize:fit:1400/1*-h3flv4Vz6Q93AgrlIBb4w.png)

```
function MyComponent(props) {
  /* 컴포넌트 */
}

function areEqual(prevProps, nextProps) {
  /*
  return true  랜더링 패스
  return false 랜더링
  */
}

export default React.memo(MyComponent, areEqual);
```



# 글을 마치며…

렌더링 성능을 최적화 하기 위해서 React 컴포넌트를 분리하고, React.memo를 사용하면 성능이 향상된다는 것을 알 수 있었습니다.

참고로 React API 문서에는 React.memo가 props를 비교할때 [shallow 비교](https://github.com/facebook/react/blob/v16.13.1/packages/shared/shallowEqual.js)를 수행한다고 나와있습니다. 하지만 React.memo를 리렌더링 방지에 사용하면 버그를 유발할 수 있으며, 성능 최적화에만 사용하는 것을 권장하고 있습니다. 그러므로 React.memo를 사용하여 성능을 최적화 할때는 Profiling을 꼭 사용하시길 바랍니다.

React 성능을 최적화하기 위한 다른 방법들도 존재합니다. 하지만 이 글에서 소개한 2가지 방법만 사용해도 매우 좋은 결과를 얻을 수 있었습니다. 성능 최적화에 대한 다른 방법도 궁금하다면 [Optimizing Performance 문서](https://reactjs.org/docs/optimizing-performance.html)를 읽어보세요.



