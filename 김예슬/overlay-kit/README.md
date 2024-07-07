memo: 지금까지 파악한 구성

- action
    - 말그대로 액션
    - 타입은 open, add, close, remove 로 구성
    - overlay 객체의 open, close, unmount 메소드는 모두 위의 액션을 일으키는 함수 (= 액션을 dispatch에 전달)
- dispatch
    - 액션이 일어나는 시점에 상태(overlays)와 액션을 reducer에 전달
    - 리스너 호출
- reducer
    - 액션에 따라 상태 변경
    - *overlay.open은 open이벤트가 아닌 add 이벤트를 발생시킴
- overlays (state)
    
    ```tsx
    let overlays: OverlayData = {
        current: null,
        overlayOrderList: [],
        overlayData: {},
    };
    
    type OverlayItem = {
        id: OverlayId;
        isOpen: boolean;
        controller: OverlayControllerComponent;
    };
    
    type OverlayData = {
        current: OverlayId | null;
        overlayOrderList: OverlayId[];
        overlayData: Record<OverlayId, OverlayItem>;
    };
    ```
    
    - current: 현재 focuse된 overlay id
    - overlayOrderList: mount된 overlay의 id 리스트
    - overlayItem: 실제 overlay를 화면에 렌더하기 위한 데이터
- 이벤트(?) 리스너

느낀점

- 너무 좋다!
    - overlay.close가 overlayId를 받는다 → 원하는 시점에, 원하는 위치에서 overlay를 다시 close할 수 잇따
    - ?? 이건 어떻게 구현되는 건가-
    - getbyId로 가져와서 그 애의 상태를 변경하나?

소소한 TIL

- tsup 라이브러리: 굉장히 작고, 설정 필요없는 (간편한) 번들링 도구
- changeset
    - 하위 모듈에 push가 발생하면 이를 감지하여 자동 배포해주는 도구
    - 깃헙 액션과 함께 CI/CD 가능
    - 배포 시 semver 규칙에 따라 버전 관리 가능
- [`remainingOverlays.at](http://remainingoverlays.at/)(-1)`: 바로 마지막 요소 접근 가넝!