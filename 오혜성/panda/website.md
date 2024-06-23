# Panda - website

## Nextra 사용중

https://github.com/shuding/nextra

* Vercel 개발자가 만든 Next.js 기반 정적 사이트 생성기
* Nextra는 아직 app router를 지원하지 않음
  + https://github.com/shuding/nextra/issues/2023
  + 그래서 pages, app 라우터를 둘 다 사용하는 모습

## seo.config.ts

```ts
// seo.config.ts
import type { Metadata } from 'next'

const defineMetadata = <T extends Metadata>(metadata: T) => metadata

export const seoConfig = defineMetadata({
  metadataBase: new URL('https://panda-css.com'),
  title: {
    template: '%s - Panda CSS',
    default:
      'Panda CSS - Build modern websites using build time and type-safe CSS-in-JS'
  },
  themeColor: '#F6E458',
}
```

```tsx
// app/layout.tsx
const { themeColor, ...metadata } = seoConfig
export { metadata }
```

## 기본 생성 레이어에 추가하는 방법

```css
@layer reset,
base,
tokens,
recipes,
utilities;
```

* 기본적으로 위와 같은 형태로 css 파일을 만들어 사용 방법을 소개함

```css
@layer reset,
base,
tokens,
recipes,
utilities;

@layer reset {
    body {
        line-height: inherit;
    }
```

- 위처럼 레이어에 추가적인 스타일을 넣을 수 있음
