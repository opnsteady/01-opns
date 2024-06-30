# Panda - generator
* CSS로 빌드해주는 부분
* 각 환경별로 나누어져있음

## 기능별 구현

* 코드를 동적으로 선택해 사용하는 방법으로 작성됨

```ts
export function generateReactJsxPattern(ctx: Context, filters?: ArtifactFilters) {
  const { typeName, factoryName, styleProps: jsxStyleProps } = ctx.jsx

  const details = ctx.patterns.filterDetails(filters)

  return details.map((pattern) => {
    const { upperName, styleFnName, dashName, jsxName, props, blocklistType } = pattern
    const { description, jsxElement = 'div', deprecated } = pattern.config

    return {
      name: dashName,
      js: outdent`
      import { createElement, forwardRef } from 'react'
      ${jsxStyleProps === 'minimal' ? ctx.file.import('mergeCss', '../css/css') : ''}
      ${ctx.file.import('splitProps', '../helpers')}
      ${ctx.file.import(styleFnName, `../patterns/${dashName}`)}
      ${ctx.file.import(factoryName, './factory')}

      export const ${jsxName} = /* @__PURE__ */ forwardRef(function ${jsxName}(props, ref) {
        ${match(jsxStyleProps)
          .with(
            'none',
            () => outdent`
          const [patternProps, restProps] = splitProps(props, ${JSON.stringify(props)})
      `,
      // 축약
      `,
    }
  })
}
```

### outdent ?

* https://www.npmjs.com/package/outdent
* 읽기 쉬운 코드를 위해 들여 쓰기(indent)를 사용하는데, 이때 결과물에서는 내어쓰기(outdent)해주는 라이브러리

## 환경 구분

* 각 문법(jsx, pattern)별로 나누어져 있음
* 같은 문법을 사용하는 환경끼리 객체를 만들고, 타입을 추론해 생성시에 사용

```ts
const factoryMap = {
  react: generateReactJsxFactory,
  solid: generateSolidJsxFactory,
  preact: generatePreactJsxFactory,
  vue: generateVueJsxFactory,
  qwik: generateQwikJsxFactory,
}

const isKnownFramework = (framework: string): framework is JsxFramework => Boolean((typesMap as any)[framework])

export function generateJsxFactory(ctx: Context) {
  if (!ctx.jsx.framework) return
  if (!isKnownFramework(ctx.jsx.framework)) return
  const factory = ctx.isTemplateLiteralSyntax
    ? factoryStringLiteralMap[ctx.jsx.framework]
    : factoryMap[ctx.jsx.framework]
  return factory?.(ctx)
}
```
