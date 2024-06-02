# Next CDN ECS 예제

- ECS에 업로드해 배포하는 예제가 구성되어 있음

## ECS?

https://aws.amazon.com/ko/ecs/

- AWS에서 제공하는 컨테이너 오케스트레이션 서비스

## 배포 방법

- github action에서 빌드 (Next.js standalone)
- 빌드한 결과물만 ECS에 업로드

### Next.js standalone?

```js
module.exports = {
  output: 'standalone'
}
```

의존성(노드 모듈)에서 배포에 필요한 파일만 복사해

독립 실행형 폴더를 생성해서 빌드하는 방법