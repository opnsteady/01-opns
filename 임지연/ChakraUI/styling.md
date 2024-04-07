# Button 스타일링은 어떤 식으로 할까?

- `button.tsx` 코드
  ```jsx
  const styles = useStyleConfig("Button", { ...group, ...props });
  ```
- `use-style-config.ts` 코드
  ```jsx
  export function useStyleConfig(
    themeKey: string,
    props: ThemingProps & Dict = {},
  ) {
    return useStyleConfigImpl(themeKey, props) as SystemStyleObject
  }
  function useStyleConfigImpl(
    themeKey: string | null,
    props: ThemingProps & Dict = {},
  ) {
    const { styleConfig: styleConfigProp, ...rest } = props

    const { theme, colorMode } = useChakra()

    const themeStyleConfig = themeKey
      ? get(theme, `components.${themeKey}`)
      : undefined

    const styleConfig = styleConfigProp || themeStyleConfig

    const mergedProps = mergeWith(
      { theme, colorMode },
      styleConfig?.defaultProps ?? {},
      compact(omit(rest, ["children"])),
    )

    /**
     * Store the computed styles in a `ref` to avoid unneeded re-computation
     */
    const stylesRef = useRef<StylesRef>({})

    if (styleConfig) {
      const getStyles = resolveStyleConfig(styleConfig)
      const styles = getStyles(mergedProps)

      const isStyleEqual = isEqual(stylesRef.current, styles)

      if (!isStyleEqual) {
        stylesRef.current = styles
      }
    }

    return stylesRef.current
  }
  ```

```jsx
const { theme, colorMode } = useChakra();
```

`useChakra()`를 통해 라이브러리의 디폴트 `theme`와 `colorMode`를 받아온다.

```jsx
const { styleConfig: styleConfigProp, ...rest } = props;
const themeStyleConfig = themeKey
  ? get(theme, `components.${themeKey}`)
  : undefined;
const styleConfig = styleConfigProp || themeStyleConfig;
```

props로 받은 `styleConfig` 가 있으면 그거 쓰고, 없으면 파라미터 `themeKey`로 받은 컴포넌트명(`button`)의 디폴트 테마 스타일을 사용한다.

```jsx
const mergedProps = mergeWith(
  { theme, colorMode },
  styleConfig?.defaultProps ?? {},
  compact(omit(rest, ["children"]))
);
```

이렇게 생성한 스타일 관련 객체들을 합쳐서 하나로 만들어내는 과정

- `useChakra()`에서 받아온 `theme`과 `colorMode`
- styleConfig(prop로 받거나, 컴포넌트가 디폴트로 가지고 있는 컴포넌트 스타일)의 default값
- `rest` : style관련 props를 제외한 props
  - `omit` 을 사용하여, 지정된 키(`children`) 을 제외한 새로운 속성을 반환한다.
  - `compact` 를 사용하여 falsy값을 가진 키를 제거하여 의미있는 속성만 남긴다.

```jsx
/**
 * Store the computed styles in a `ref` to avoid unneeded re-computation
 */
const stylesRef = useRef < StylesRef > {};
```

`useRef` 를 이용해서 계산된 스타일 객체를 저장하여, 컴포넌트가 리렌더링되어도 초기화되지 않도록 한다.

```jsx
if (styleConfig) {
  const getStyles = resolveStyleConfig(styleConfig);
  const styles = getStyles(mergedProps);

  const isStyleEqual = isEqual(stylesRef.current, styles);

  if (!isStyleEqual) {
    stylesRef.current = styles;
  }
}
```

`if(styleConfig)` 로, 사용자 또는 테마에서 설정한 옵션이 있을 때에만 해당 로직을 탄다.

### 정리

1. 해당 라이브러리의 default 테마, 컬러값을 쌓는다.
2. props로 받거나 해당 컴포넌트가 가지고 있는 값으로 덮어씌운다.
3. 그리고 최종적으로 `styleConfig`props를 제외한 나머지 props를 합쳐서 최종 스타일을 만든다.
4. 스타일을 따로 `useRef` 에 저장하여 기억하고 있는다.
