# Vercel

## 겪은 문제

```bash
package.json
apps
  - web
    - package.json
```

위와 같은 모노레포 구조였음

```json
// root/package.json
{
  "resolutions": {
    "next": "14.2.3" 
  }
}
```

```json
// root/apps/web/package.json
{
  "dependencies": {
    "next": "*"
  }
}
```

다음과 같은 형태로 root/package.json에 resolutions을 이용해 버전을 고정하도록 작성하였음

* 그러나 이때 vercel을 이용해 배포하면 빌드까지 성공하지만 404를 반환함

## resolutions?

https://classic.yarnpkg.com/lang/en/docs/selective-version-resolutions/

* yarn에서 제공하는 기능
* 의존성 내의 패키지 버전이나 범위를 고정할 수 있음

```
만약 foo라는 라이브러리를 사용한다면?
foo의 의존성이 react 16.7.0 이라고 하면...

foo 1.0.0
  - react 16.7.0

요롷게 설치될 것

그치만 resolutions에 다음과 같이 작성하면

{
  "resolutions": {
    "react": "16.8.0"
  }
}

foo 1.0.0
  - react 16.8.0

이렇게 설치되도록 의존성을 고정할 수 있음
```

* 이를 응용해 모노레포 내에서 공통된 의존성을 고정하도록 한 것

* pnpm에서는 `overrides`라는 이름으로 기능을 제공하며 `resolutions`라는 이름으로도 제공함
  + 이는 마이그레이션에 이점을 만들기 위한 것이라고 문서에 기술되어 있음
  + https://pnpm.io/package_json#resolutions
  + 우선순위는 `overrides` > `resolutions`로 작동함

## 해결 방법

* `apps/web/package.json`에 버전을 명시하면 정상적으로 배포됨
  + `root/package.json`에 resolution도 기입해, 오기입으로 인한 중복 버전 설치를 막았음 

## 추정 원인

```ts
// https://github.com/vercel/vercel/blob/545f1174879e51cc8116125d87e430cde67210ee/packages/next/src/index.ts#L141-L162

/**
 * Get the installed Next version.
 */
function getRealNextVersion(entryPath: string): string | false {
  try {
    // First try to resolve the `next` dependency and get the real version from its
    // package.json. This allows the builder to be used with frameworks like Blitz that
    // bundle Next but where Next isn't in the project root's package.json

    const resolved = require_.resolve('next/package.json', {
      paths: [entryPath],
    });
    const nextVersion: string = require_(resolved).version;
    console.log(`Detected Next.js version: ${nextVersion}`);
    return nextVersion;
  } catch (_ignored) {
    console.log(
      `Warning: Could not identify Next.js version, ensure it is defined as a project dependency.`
    );
    return false;
  }
}
```

`vercel` 패키지 내에서 빌드 과정에서 호출되는 함수인데, package.json에 기입된 버전을 가지고 오도록 기술되어 있어 발생하나 싶은 추정

## 부록 ..

resolutions으로 모노레포의 의존성 버전을 고정하는 게 권장되는 방법일까?

- 개발 의도와는 다른 방향이라 생각되어 의문을 품게 됨
- 그래서 해당 기능 개발자를 찾아 메일을 보내볼까 ... 하다가 PR을 찾게 됨
  - https://github.com/yarnpkg/berry/pull/829
  - 작성자분은 2017년부터 지금까지 yarn의 리드 메인테이너로 활동하신듯 https://github.com/arcanis
  - pnp 아키텍처를 디자인하고 구현하셨다고 함