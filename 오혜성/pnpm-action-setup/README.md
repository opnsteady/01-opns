# pnpm/action-setup

## 겪은 문제

* github action에서 pnpm은 기본적으로 지원하지 않음

> 정확히는 actions/setup-node 액션이 지원하지 않음
> > npm, yarn은 지원

* 그렇기에 pnpm/setup-action을 사용해야함

* 공식문서에 의존성 cache 예제가 작성되어 있는데, 이때 actions/setup-node가 먼저 실행됨

* 그치만 이렇게 하면 에러를 뱉음 ..

## 해결 방법

* 검색을 통해 다음 actions/setup-node 문서를 찾을 수 있었음

> https://github.com/actions/setup-node/blob/main/docs/advanced-usage.md#caching-packages-data

* pnpm/action-setup을 먼저 사용해야함
