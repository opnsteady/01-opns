# Slash - ky

## 해결하는 문제점

* ky는 CJS를 지원하지 않음
* SSR 시에는 `ky-universal`을 사용해야 함

## 해결하는 방법

* `@toss/ky`에서 CJS와 SSR 대응을 하도록 같이 제공

## 새롭게 안 것들

* ky-universal이 CJS를 지원못하는 이유?
  + top level await를 사용하기 때문
    - https://www.w3schools.io/javascript/es13-top-level-await/
    - ES2022에서 지원하는 기능

* abort-controller의 폴리필을 사용하고 있음
  + https://github.com/mysticatea/abort-controller?tab=readme-ov-file

* 새롭게 ky와 ky-universal repo에 가보니, ky `1.0.0`부터 node 환경을 정식 지원한다 함
  + https://github.com/sindresorhus/ky-universal
  - Slash에서 사용하고 있는 ky 버전은 `^0.31.1`

