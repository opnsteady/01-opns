# useTransition 내부 구현 파악하기

- ReactSharedInternals.H가 리액트 모든 훅에서 사용하는 dispatcher
- dispatcher 안에 모든 리액트 훅이 담겨있음 (객체 형태)
- 하지만 ReactSharedInternals.H는 초기값이 null, dispatcher가 Null인지 아닌지를 통해 훅이 컴포넌트 내부에서 호출되었는지를 확인하는 것으로 보아 렌더 이후 ReactSharedInternals.H 값이 초기화되는 걸로 보임
- render 함수는 ReactDomRoot의 메소드. updateContainer를 통해 생성됨 (react)

Next

- updateContainer 함수를 통해 ReactSharedInternals.H 값 트래킹
- ReactSharedInternals 값을 외부에 오픈하지 않으려는 의도도 보임 -> preact 코드를 확인해봐도 좋을 듯

추가

- Suspense, transition관련 코드도 모두 react-reconciler 폴더 아래 위치한 걸로 보인다
  - ReactFiberSuspense..
  - ReactFiberTransition..
