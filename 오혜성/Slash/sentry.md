# Slash - sentry

## 해결하는 문제점

* sentry는 제공하는 인터페이스는 같으나 사용하는 곳 (브라우저, 서버)에 따라 임포트를 `@sentry/browser`와 `@sentry/node`로 나눠야 함

## 해결하는 방법

* 브라우저, 서버 환경 모두 `@toss/sentry`를 사용하게 함
  + Node 21버전의 import condition을 이용해 사용하는 곳에 따라 임포트되게 함
  + https://nodejs.org/api/packages.html#community-conditions-definitions
