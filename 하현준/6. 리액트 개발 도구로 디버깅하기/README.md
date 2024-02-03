## 리액트 개발 도구 활용하기

리액트 개발 도구를 설치 한 뒤 Components와 Profiler 메뉴를 통해 디버깅을 제공한다.

### Components

Components 탭에서는 컴포넌트 트리, props, hooks등 컴포넌트에 대한 자세한 정보를 보여주게 된다.

이떄 hook의 경우에는 use가 제외된 형태로 나타나게 된다.

### Profiler

프로파일러는 리액트가 렌더링하는 과정에서 발생하는 상황을 확인하기 위한 도구이다.

여기서 컴포넌트가 렌더링될 때마다 강조 표시를 하고 싶다면 ‘Highlight update when components render’를 체크하면 표시가 나타난다.

Start Profiling을 통해 렌더링이 일어난 컴포넌트의 정보를 순차적으로 확인을 해볼 수 있다.

Flamegraph가 아닌 Ranked 메뉴를 통해 렌더링이 발생한 정보만 파악할 수 있다.
