# setFocusTimeout

## 해결하는 문제

* 일정 시간의 딜레이 이후에 input.focus를 주고 싶은 경우
  + 모바일 사파리에서 async function ㅐ부에서 input.focus가 제대로 작동하지 않는 이슈가 있음
  + https://stackoverflow.com/questions/12204571/mobile-safari-javascript-focus-method-on-inputfield-only-works-with-click


## 해결하는 방법

- 가짜 인풋을 만든 후 포커스 시키고, 딜레이 이후 인수로 받은 함수를 실행함

## 새로 알게된 점

- `focus({ preventScroll: true })`

- `input.readOnly = true` focus 시 키보드가 나오지 않음