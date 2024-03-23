# react-query
* 내부적으로 V3 `react-query`를 사용하고 있음

> V4부터 `@tanstack/react-query`

> 현재 최신 버전은 `5.28.4`

## useQuery

### 해결하는 문제

* queryFn의 반환값이 undefined일 경우 타입 에러를 발생 시킴

### 해결하는 방법

* 함수 오버로딩
* queryFn의 반환값 타입인 `TQueryFnData` 타입에 컨디셔널 타입을 적용

## useSuspendedQuery

### 해결하는 문제

* suspense 옵션을 default로 하며, data가 non-nullable
  + error, isLoading, isError, isFetching이 존재하지 않음

### 해결하는 방법

* 함수 오버로딩
* react-query 타입을 기반으로 재작성한 타입들

## 알게된 것

* `ts-expect-error`를
  + 테스트 코드에 사용되고 있음
  + https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-9.html#-ts-expect-error-comments
    - 타입스크립트 3.9에서 등장
  
* 커스텀 타임 에러 메세지를 던지는 방법
  + 컨디셔널 타입과 리터럴 타입을 사용하여 구현 가능
  + https://niksa-fantela.com/posts/typescript-custom-error-messages-with-conditional-types/
  

```ts
type ErrorMsg = 'error msg';

const a = <T = number | string>(b: T extends string ? ErrorMsg : number) => {
  return b;
};

a('12'); // error
```

* react-query `networkMode`
  + default `online`
  + options `online` | `offlineFirst` | `always`
    - `online`
      + 온라인일 때만 fetch, 오프라인일 때 네트워크 커넥션 요청하지 않음
      + fetchStatus가 `paused`일 수 있음
    - `always`
      + 항상 fetch
      + 캐싱된 값을 바로 쓰는 경우나, Promise.resolve(5)와 같이 네트워크 커넥션이 필요 없는 경우에 사용
      + fetchStatus가 `paused`일 수 없음
    - `offlineFirst`
      + 네트워크 연결 여부 상관 없이 실행
      + fetchStatus가 `paused`일 수 있음
      + 캐싱된 값을 오프라인 시 사용할 수 있음
      + 위 두 옵션의 중간이라고 공식 문서에서 소개하는데 와닿지는 않음
      + `이는 오프라인 우선 PWA처럼 캐싱 요청을 가로채는 serviceWorker가 있거나 Cache-Control 헤더를 통해 HTTP 캐싱을 사용하는 경우 매우 유용합니다.`
    - 관련 블로그 아티클
      - 리액트 쿼리 개발자의 글: https://tkdodo.eu/blog/offline-react-query 
      - 번역 글: https://velog.io/@dosomething/Offline-React-Query
      
