# ImpressionArea

Intersection Observer API를 **효율적으로** 사용하는 component

## ImpressionArea의 이점

- 기존에 threshold만 받던 것을 timeThreshold, areaThreshold로 구분하여 인자로 받고 timeThreshold 만큼 디바운스 적용 (areaThreshold = 기존 threshold)
- document.visibilityState를 사용하여 화면이 백그라운드, 혹은 포그라운드에 있는지 또한 고려
- 전달된 콜백 함수와 렌더링과 상관없는 isIntersectioning 값(뷰 안에 있는지)을 ref로 감싸 최적화
- 내부에서 usePreservedCallback을 사용하여 리렌더링시에도 참조 안정성 보장

## 알게된 점

### document.visibilityState와 테스트

- document.visibilityState:
  - 사용자가 화면을 보고 있는지 확인할 수 있는 속성 ('visible' | 'hidden')
  - hidden: 다른 탭을 보고 있거나 화면을 최소화 했을 때, 컴퓨터가 잠자기 모드에 들어갔을 때 등
  - 대체 속성으로 document.hidden 속성이 있음 (반환값은 boolean)
- 테스트: mockDocumentVisibilityState
  - 이런 전역 객체를 모의 객체로 대체할 때, 1) setup 시 모의 객체로 대체 2) clear 시 기존 객체로 복구
  - 기존 document 객체에 Proxy 객체 사용 -> **다른 메소드들은 보존하되 visibilityState만 원하는 값으로 설정**
  - dispatchEvent를 통해 바로 이벤트를 발생시킴
  - +) 이벤트 객체를 만들 때 click, keyboardDown 등과 같은 내장 이벤트들은 별도의 내장이벤트 객체로 생성해야한다. 이벤트별로 특수한 속성이 있기 때문에
    ```js
    let event = new MouseEvent('click', {
      bubbles: true,
      cancelable: true,
      clientX: 100,
      clientY: 100,
    }) // ✅
    let event = new Event('click', {
      bubbles: true, // Event 생성자에선
      cancelable: true, // bubbles와 cancelable 프로퍼티만 동작합니다.
      clientX: 100,
      clientY: 100,
    }) // ❌
    ```

## 더 알아보기

- [as ... as ... 타입](https://f-lab.kr/blog/polymorphism-with-type-safe)
- [custom error message](https://niksa-fantela.com/posts/typescript-custom-error-messages-with-conditional-types/)
- intersection observer api
