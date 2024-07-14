# cva

> Creating variants with the "traditional" CSS approach can become an arduous task; manually matching classes to props and manually adding types.
`cva` aims to take those pain points away, allowing you to focus on the more fun aspects of UI development.
*"전통적인" CSS 접근 방식으로 variants를 만드는 것은 class를 props에 수동으로 일치시키고 types를 수동으로 추가하는 등 힘든 작업이 될 수 있습니다. cva는 이러한 문제점을 없애고 UI 개발의 더 재미있는 측면에 집중할 수 있도록 하는 것을 목표로 합니다.*
> 

⇒ (Tailwind CSS나 기본 CSS에서) variants를 편리하게 부여하기 위한 기능

## Variants

### **variants**

### **compoundVariants**

다른 변형 조건이 충족될 때 적용되는 변형

다양한 조건의 조합이 가능

### **defaultVariants**

기본값, `null`로 설정해 제거 가능

## 구현 코드

- 전체 코드     [링크](https://github.com/joe-bell/cva/blob/main/packages/class-variance-authority/src/index.ts)
    
    ```tsx
    export const cva =
      <T>(base?: ClassValue, config?: Config<T>) =>
      (props?: Props<T>) => {
        if (config?.variants == null)
          return cx(base, props?.class, props?.className);
    
        const { variants, defaultVariants } = config;
    
        const getVariantClassNames = Object.keys(variants).map(
          (variant: keyof typeof variants) => {
            const variantProp = props?.[variant as keyof typeof props];
            const defaultVariantProp = defaultVariants?.[variant];
    
            if (variantProp === null) return null;
    
            const variantKey = (falsyToString(variantProp) ||
              falsyToString(
                defaultVariantProp,
              )) as keyof (typeof variants)[typeof variant];
    
            return variants[variant][variantKey];
          },
        );
    
        const propsWithoutUndefined =
          props &&
          Object.entries(props).reduce(
            (acc, [key, value]) => {
              if (value === undefined) {
                return acc;
              }
    
              acc[key] = value;
              return acc;
            },
            {} as Record<string, unknown>,
          );
    
        const getCompoundVariantClassNames = config?.compoundVariants?.reduce(
          (
            acc,
            { class: cvClass, className: cvClassName, ...compoundVariantOptions },
          ) =>
            Object.entries(compoundVariantOptions).every(([key, value]) =>
              Array.isArray(value)
                ? value.includes(
                    {
                      ...defaultVariants,
                      ...propsWithoutUndefined,
                    }[key],
                  )
                : {
                    ...defaultVariants,
                    ...propsWithoutUndefined,
                  }[key] === value,
            )
              ? [...acc, cvClass, cvClassName]
              : acc,
          [] as ClassValue[],
        );
    
        return cx(
          base,
          getVariantClassNames,
          getCompoundVariantClassNames,
          props?.class,
          props?.className,
        );
      };
    ```
    

### getVariantClassNames

```tsx
const getVariantClassNames = Object.keys(variants).map(
  (variant: keyof typeof variants) => {
    const variantProp = props?.[variant as keyof typeof props];
    const defaultVariantProp = defaultVariants?.[variant];

    if (variantProp === null) return null;

    const variantKey = (falsyToString(variantProp) ||
      falsyToString(
        defaultVariantProp,
      )) as keyof (typeof variants)[typeof variant];

    return variants[variant][variantKey];
  },
);
```

- `varints`에 정의된 `key`값을 가져오는 함수
    - `defaultVariants`의 값이 정의되었다면 `defaultVariants`의 `variant key`값을
    - 아니라면 `variants`의 `variant key`값을 반환한다
- `keyof typeof`
    - 객체의 키로 이루어진 **유니온 타입**을 의미
        - 객체의 키들 중 하나만 값으로 들어올 수 있게 되어 타입 안정성 보장
        
        ```tsx
        variants: {
            intent: {
              primary: [
                "bg-blue-500",
                "text-white",
                "border-transparent",
                "hover:bg-blue-600",
              ],
              // **or**
              // primary: "bg-blue-500 text-white border-transparent hover:bg-blue-600",
              secondary: [
                "bg-white",
                "text-gray-800",
                "border-gray-400",
                "hover:bg-gray-100",
              ],
            },
            size: {
              small: ["text-sm", "py-1", "px-2"],
              medium: ["text-base", "py-2", "px-4"],
            },
          },
        
        const variantKey = (falsyToString(variantProp) ||
          falsyToString(
            defaultVariantProp,
          )) as keyof (typeof variants)[typeof variant];
          
          
        typeof variants =
        {
          intent: {
            primary: string[];
            secondary: string[];
          };
          size: {
            small: string[];
            medium: string[];
          };
        }
        
        typeof variant = 'intent' | 'size'
        
        (typeof variants)[typeof variant] =
        { primary: string[]; secondary: string[] } | { small: string[]; medium: string[] }
        
        keyof (typeof variants)[typeof variant] = 'primary' | 'secondary' | 'small' | 'medium'
        ```
        
        - `variantKey`가 `variants`객체의 특정 `variant`에 유효한 `key`값인 것을 보장
        - `(typeof variants)`인 것은 우선 순위를 위한 것..!

### propsWithoutUndefined

```tsx
const propsWithoutUndefined =
  props &&
  Object.entries(props).reduce(
    (acc, [key, value]) => {
      if (value === undefined) {
        return acc;
      }

      acc[key] = value;
      return acc;
    },
    {} as Record<string, unknown>,
  );
```

- `props`의 `undefined`가 있다면 이를 제거한 새로운 객체를 반환하는 역할
    - `value`가 `undefined`가 아닌 경우 `acc`에 새로운 `key`, `value`을 추가하지만
    - `value`가 `undefined`인 경우 `acc`를 그대로 반환, 초기값은 빈 객체

### getCompoundVariantClassNames

```tsx
const getCompoundVariantClassNames = config?.compoundVariants?.reduce(
  (
    acc,
    { class: cvClass, className: cvClassName, ...compoundVariantOptions },
  ) =>
    Object.entries(compoundVariantOptions).every(([key, value]) =>
      Array.isArray(value)
        ? value.includes(
            {
              ...defaultVariants,
              ...propsWithoutUndefined,
            }[key],
          )
        : {
            ...defaultVariants,
            ...propsWithoutUndefined,
          }[key] === value,
    )
      ? [...acc, cvClass, cvClassName]
      : acc,
  [] as ClassValue[],
);
```

- `config?.compoundVariants?.reduce((acc, cur) ⇒ (acc + cur), int)`
    - 초기값: `[] as ClassValue[]`
- `Object.entries(compoundVariantOptions)`
    - `compoundVariantOptions`의 `key`, `value`값을 `[key, value]` 쌍의 배열로 반환
        
        ```tsx
        compoundVariants: [
          {
            intent: "primary",
            size: "medium",
            class: "uppercase",
          },
        ]
        
        compoundVariantsOptions = {
        	intent: "primary",
        	size: "medium",
        }
        
        Object.entires(compoundVariantsOptions) // ['intent': 'primary', 'size': 'medium']
        ```
        
- `Object.entries(compoundVariantOptions).every(([key, value]) ⇒ …`
    - `[key, value]` 쌍의 배열 조건을 검증
        
        ```tsx
        	Array.isArray(value)
          ? value.includes(
              {
                ...defaultVariants,
                ...propsWithoutUndefined,
              }[key],
            )
          : {
              ...defaultVariants,
              ...propsWithoutUndefined,
            }[key] === value,
        )
        ```
        
        - `value`가 배열인 경우, `value`값이 포함되는지 확인
        - 배열이 아닌 경우, `value`값과 일치하는지 확인
    - `every`에서 검증한 조건이 `true`인 경우, `acc`에 `cvClass`와 `cvClassName` 추가
    - `false`이 경우, `acc`반환