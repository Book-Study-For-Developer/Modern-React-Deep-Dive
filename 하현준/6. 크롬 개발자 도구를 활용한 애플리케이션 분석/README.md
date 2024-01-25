# 6장 크롬 개발자 도구를 활용한 애플리케이션 분석

## 1. 크롬 개발자 도구란

크롬 개발자 도구란 크롬에서 제공하는 개발자용 도구로, 웹페이지에서 일어나는 거의 모든 일을 확인할 수 있는 강력한 개발 도구다.

## 2. 요소 탭

### 요소 화면

현재 웹페이지를 구성하는 HTML을 나타낸다. 요소의 중단점을 통해 요소의 변경을 일으킨 소스파일을 확인할 수 있다.

요소 정보

- 스타일(Styles): 요소와 관련된 스타일 정보를 나타낸다.
- 계산됨(Computed): 해당 요소의 크기, 패딩, 보더, 마진과 각종 CSS 적용 결괏 값을 알 수 있는 탭이다.
- 레이아웃(Layout): CSS 그리드나 레이아웃과 관련된 정보 확인
- 이벤트 리스너(Event Listeners): 현재 요소에 부착된 각종 이벤트 리스너를 확인
- DOM 중단점: 중단점이 있는지 알려주는 탭
- 속성(Properties): 해당 요소가 가지고 있는 모든 속성값을 나타낸다. 해당 요소가 가지고 있는 모든 값이 나옴(**.attributes**는 직접 할당된 값만 나옴)
- 접근성(Accessibility): 웹 이용에 어려움을 겪는 장애인, 노약자를 위한 스크린리더기 등이 활용하는 값을 말

## 3. 소스 탭

웹 애플리케이션을 불러오기 위해 실행하거나 참조된 모든 파일을 확인할 수 있다. (JS, CSS, HTML, Font 등)

중단점을 설정해 두면 빌드된 자바스크립트가 어떻게 실행되는지 확인이 가능하다. 스코프, 호출 스택 등 현재 자바스크립트가 실행되고 있는 구조도 확인이 가능

## 4. 네트워크 탭

웹 페이지를 접속하는 순간부터 발생하는 모든 네트워크 관련 작동이 기록.

네트워크 탭을 통해 집중적으로 확인해 봐야 하는 점은 다음과 같다.

- 불필요한 요청 또는 중복되는 요청이 없는지
- 웹페이지 구성에 필요한 리소스 크기가 너무 크지 않은지
- 리소스를 불러오는 속도는 적절한지 또는 너무 속도가 오래 걸리는 리소스는 없는지
- 리소스가 올바른 우선순위로 다운로드되어 페이지를 자연스럽게 만들어가는지

## 5. 메모리 탭

현재 웹페이지가 차지하고 있는 메모리 관련 정보를 확인할 수 있다.

![image](https://github.com/Book-Study-For-Developer/Modern-React-Deep-Dive/assets/51049245/3b564c35-5525-4c1f-b5d5-a5acf6f82136)

- 힙 스냅샷: 현재 메모리 상황을 사진 찍듯이 촬영한다.
- 타임라인 할당 계측: 현재 시점의 메모리 상황이 아닌, 시간의 흐름에 따라 메모리의 변화를 살펴볼 수 있다.
- 할당 샘플링: 메모리 공간을 차지하고 있는 자바스크립트 함수

### 자바스크립트 인스턴스 VM 선택

자바스크립트 인스턴스 선택에서 현재 **실행중인 자바스크립트 VM 인스턴스**를 확인하고, 크기만큼 사용자의 브라우저에 부담을 준다.

### 힙 스냅샷

현재 페이지의 메모리 상태를 확인해 볼 수 있는 메모리 프로파일 도구다.

![image](https://github.com/Book-Study-For-Developer/Modern-React-Deep-Dive/assets/51049245/bf34f996-18c4-4c5f-bf89-e3a2973e54d8)


<aside>
✅ **얕은 크기(Shallow Size)**와 **유지된 크기(Retained Size)**의 차이점은 무엇인가요?

메모리 누수를 찾을 때는 얕은 크기(객체 자체의 크기)는 작으나 유지된 크기(객체가 참조하고 있는 모든 객체들의 크기)가 큰 객체를 찾아야 한다.

\*\*두 크기의 차이가 큰 객체는 다수의 다른 객체를 참조하고 있다는 뜻, 이는 해당 객체가 복잡한 참조 관계를 가지고 있다는 뜻이다.

이러한 객체가 오랜 시간 메모리에 남아있다면 그로 인해 많은 메모리를 점유하고 있을 수 있다.\*\*

</aside>

메모리 누수 정보를 확인하기 위해서는 메모리 누수가 발생하는 것으로 예상되거나 위험이 존재할 것 같은 스크립트 전후로 촬영해 비교하고, 기명함수를 통해 확인도 가능하다.

### 타입라인 할당 계측

시간의 흐름에 따라 메모리 변화를 확인할 수 있는 기능이다.

![image](https://github.com/Book-Study-For-Developer/Modern-React-Deep-Dive/assets/51049245/cc43b8ba-879e-483d-a493-b207def5e88d)

![image](https://github.com/Book-Study-For-Developer/Modern-React-Deep-Dive/assets/51049245/932122a7-b06b-4a39-b2b6-8033888e9787)


특정 변수를 클릭해 **전역 변수로 저장(Store as global variable)**을 클릭해 확인이 가능하다.

### 할당 샘플링

시간의 흐름에 따라 발생하는 메모리 점유를 확인할 수 있다는 점에서 할당 계측과 비슷하지만 **자바스크립트 실행 스택별로 분석할 수 있고, 함수 단위로 한다는 차이점이 있다.**

## 6. Next.js 환경 디버깅하기

사용자의 기기의 성능과 스펙에 따라 같은 메모리 누수라도 다른 결과를 낳게 될 것이다. 그러나 만약 **서버 사이드 렌더링을 수행하는 자바스크립트 환경에서 메모리가 누수가 발생**하면 서버 자체에 부담이 발생할 것이고 **모든 사용자가 사용할 수 없는 심각한 상황을 초래**하게 될 것이다.

### Next.js 프로젝트를 디버그 모드로 실행하기

- 다음과 같이 명령어를 설정한 뒤 `next dev` 를 실행하면 디버거가 활성된다.

```tsx
"dev": NODE_OPTIONS='--inspect' next dev
```

- chrome://inspect/#devices 에 접속하여 Open dedicated DevTools for Node로 새로운 개발자 도구 실행

### Next.js 서버에 트래픽 유입시키기

새로고침이나 다른 컴퓨터를 이용해서 접속하는 것보다 ab 라이브러리를 통해 트래픽을 발생시킬 수 있다. [ab](https://httpd.apache.org/docs/current/ko/programs/ab.html)는 아파치 재단에서 제공하는 웹서버 성능 검사 도구이다.

```tsx
ab -k -c 50 -n 10000 "http://127.0.0.1:3000/"
```

“http://127.0.0.1:3000/”(로컬에서는 IP주소를 입력해야 한다.)를 향해 한 번에 50개의 요청을 총 10,000회 시도하는 명령어이다.

ab를 사용하면 요청으로부터 응답받는 데 걸린시간, 바이트 크기 등 다양한 정보를 확인할 수 있다.

### Next.js의 메모리 누수 지점 확인하기

```tsx
import type { GetServerSidePropsContext, NextPage } from "next";

const access_users = [];

function Home({ currentDateTime }: { currentDateTime: number }) {
  return <>{currentDateTime}</>;
}

export const getServerSideProps = (ctx: GetServerSidePropsContext) => {
  const currentDateTime = new Date().getTime();

  access_users.push({
    user: `user-${Math.round(Math.random() * 100000)}`,
    currentDateTime,
  });

  return {
    props: {
      currentDateTime,
    },
  };
};
```

이 코드는 `getServerSideProps` 의 다수 실행과 메모리 누수가 발생한다.

페이지 접근 요청이 있을 때마다 실행되는 함수이므로 **최대한 부수 효과가 없는 순수함수로 만들어야 한다.**
