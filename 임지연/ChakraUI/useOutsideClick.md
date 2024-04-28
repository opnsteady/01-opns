# useOutsideClick

- 모달이나 드롭다운과 같은 UI 구성 요소가 열려 있을 때 밖을 클릭하면 해당 구성 요소를 닫기 위해 사용 ( [예시](https://v1.chakra-ui.com/docs/styled-system/utility-hooks/use-outside-click#usage) )
- 예시 코드

  ```jsx
  const Example = () => {
    const ref = React.useRef();
    const [isModalOpen, setIsModalOpen] = React.useState(false);
    useOutsideClick({
      ref: ref,
      handler: () => setIsModalOpen(false),
    });

    return (
      <>
        {isModalOpen ? (
          <div ref={ref}>
            👋 Hey, I'm a modal. Click anywhere outside of me to close.
          </div>
        ) : (
          <button onClick={() => setIsModalOpen(true)}>Open Modal</button>
        )}
      </>
    );
  };
  ```

# 인터페이스

- `enabled?: boolean` : 훅 활성화 여부
- `ref: RefObject<HTMLElement>` : 연결된 DOM ref
- `handler?: (e: Event) => void` : outside를 눌렀을 때, 호출될 콜백함수

# 구현

- `useRef`와 `useEffect` 이용해서 state 관리
  - useRef
    - `isPointerDown` : 이벤트가 `down` 되었는지 여부
    - `ignoreEmulatedMouseEvents` : 터치 이후에 발생하는 마우스 이벤트 무시할지 여부 결정 (for 터치스크린)
      - **ignoreEmulatedMouseEvents**
        - 웹에서 터치스크린 사용 시, 터치이벤트를 처리하고 이와 동시에 유사한 마우스이벤트도 생성
        - 포인터이벤트와 마우스이벤트 사이의 중복을 방지하기 위한 역할
        - ex) 사용자가 터치스크린 사용 시 브라우저는 `touchstart` 와 같은 터치이벤트를 발생한 후바로 `mousedown` 이벤트 발생시킴
        - `ignoreEmulatedMouseEvents` 를 사용하여 터치이벤트가 처리된 후 발생하는 마우스이벤트 무시
  - useEffect
    - `ref` (클릭이 감지될 요소) 에 이벤트 등록
    - 클린업 함수로 이벤트 해제
- **savedHandler**
  - 호출될 콜백함수를 useRef를 통해 관리
- **onPointerDown**
  - 마우스 클릭하는 순간, 터치스크린에 손가락으로 터치하는 순간
  - 포인터가 요소 내부에서 아래로 내려갔는지 확인 후, 상태 업데이트
  ```jsx
  const onPointerDown: any = (e: PointerEvent) => {
    if (isValidEvent(e, ref)) {
      state.isPointerDown = true;
    }
  };
  ```
- **onMouseUp**

  - 마우스 클릭 끝나는 순간
  - 사용자가 클릭한 위치가 참조된 요소의 외부인지 확인 후 `savedHandler` 호출하여 콜백함수 `setState(false)` 실행

  ```jsx
  const onMouseUp: any = (event: MouseEvent) => {
    if (state.ignoreEmulatedMouseEvents) {
      state.ignoreEmulatedMouseEvents = false;
      return;
    }

    if (state.isPointerDown && handler && isValidEvent(event, ref)) {
      state.isPointerDown = false;
      savedHandler(event);
    }
  };
  ```

- **onTouchEnd**
  - 터치스크린에서 손가락 떼는 순간 실행되는 함수
  - 사용자가 클릭한 위치가 참조된 요소의 외부인지 확인 후 `savedHandler` 호출하여 콜백함수 실행
  ```jsx
  const onTouchEnd = (event: TouchEvent) => {
    state.ignoreEmulatedMouseEvents = true;
    if (handler && state.isPointerDown && isValidEvent(event, ref)) {
      state.isPointerDown = false;
      savedHandler(event);
    }
  };
  ```

### 유효한 outsideClick인지 확인

- **isValidEvent**

  - 마우스 좌클릭만 허용 : `if (event.button > 0)`
  - 이벤트가 발생한 곳이 doc 내부인지 확인 : `if (!doc.contains(target))`
  - 이벤트가 발생한 곳이 ref의 외부인지 확인 : `!ref.current?.contains(target)`

  ```jsx
  function isValidEvent(event: any, ref: React.RefObject<HTMLElement>) {
    const target = event.target as HTMLElement
    if (event.button > 0) return false
    if (target) {
      const doc = getOwnerDocument(target)
      if (!doc.contains(target)) return false
    }

    return !ref.current?.contains(target)
  }
  ```
