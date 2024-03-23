# HiddenHeading

## 설명 및 코드

접근성 목적을 위해 사용할 수 있는 헤딩 컴포넌트입니다.

토스앱의 일반 사용자에게는 보이지 않지만 스크린리더를 사용하는 사용자들에게만 텍스트가 보입니다.

- 유사한 컴포넌트로 `<ScreenReaderOnly />` 컴포넌트가 있습니다.
- 페이지 구역을 나눌때 보조기술은 HTML5 Outline과 다르게 작동하므로, 헤딩의 수준에 따라 'as'를 적절하게 사용하여 구역을 의미있게 나눠주시기 바랍니다.

```tsx
/** @jsxImportSource react */
import classnames from 'classnames';
import { ComponentType, HTMLAttributes, ReactNode } from 'react';
import Style, { generateClassNames } from '../../utils/Style';

type HeadingType = 'h1' | 'h2' | 'h3' | 'h4' | 'h5' | 'h6';

interface Props {
  /**
   * @default 'h1'
   */
  as?: HeadingType;
  className?: string;
  children?: ReactNode;
  id?: string;
}

/** @tossdocs-ignore */
export function HiddenHeading({ as = 'h1', id, className, children }: Props) {
  **const Heading = as as unknown as ComponentType<HTMLAttributes<HTMLHeadingElement>>**;

  return (
    <Style css={css}>
      <Heading id={id} className={classnames(className, CLASSNAMES.hiddenHeading)}>
        {children}
      </Heading>
    </Style>
  );
}

const CLASSNAMES = generateClassNames({
  hiddenHeading: 'hidden-heading',
});

const css = `
  .${CLASSNAMES.hiddenHeading} {
    position: absolute;
    width: 1px;
    height: 1px;
    margin: -1px;
    padding: 0;
    overflow: hidden;
    border: 0;
    clip: rect(0, 0, 0, 0);
  }
`;

```

사용 예시

```tsx
// 일반 사용자는 바텀시트의 디머 바깥에서 어둡게 보이는 모습을 통해 이 바텀시트가 '프로필 종류를 선택'하는 바텀시트임을 쉽게 알 수 있지만 스크린리더 사용자의 경우 어떤 바텀시트인지 알 수 없을 때
<BottomSheet>
  <HiddenHeading as="h3" id={id}>
    사용할 프로필의 종류를 선택해주세요
  </HiddenHeading>
  <List role="radiogroup" aria-labelledby={id}>
    // blahblah
  </List>
</BottomSheet>
```

### 내가 작성한 코드와 비교

```tsx
import { HTMLAttributes } from 'react'
import { colors } from '@/styles/colors'
import { css } from '@emotion/react'
import styled from '@emotion/styled'

type TypographyVariant = keyof typeof variants

interface TextProps extends HTMLAttributes<HTMLElement> {
  variant?: TypographyVariant
  color?: string
}

type TagType = {
  [key in TypographyVariant]: React.ElementType
}

const tags: TagType = {
  heading1: 'h1',
  heading2: 'h2',
  heading3: 'h3',
  body: 'p',
  bodyStrong: 'strong',
  detail: 'span',
  detailStrong: 'strong',
}

export function Text({
  variant = 'body',
  color = colors.gray900,
  ...props
}: TextProps) {
  return (
    <TextTag
      as={tags[variant]}
      css={[{ color }, variants[variant]]}
      {...props}
    />
  )
}

**const TextTag = styled.p``**

const variants = {
  heading1: css`
    font-size: 24px;
    font-weight: 700;
  `,
  // ... 이하 생략
}

```

- `HiddenHeading`에서는 `as` props에 사용할 태그를 직접 넣어 코드를 작성할 때 바로 어떤 태그인지 쉽게 알아볼 수 있음
- 따라서 타입 작성이 간결해짐

## 새롭게 안 내용

### `as unknown as`

타입을 전환하기 위한 방식

```tsx
export function HiddenHeading({ as = 'h1', id, className, children }: Props) {
  **const Heading = as as unknown as ComponentType<HTMLAttributes<HTMLHeadingElement>>**;

  return (
    <Style css={css}>
      <Heading id={id} className={classnames(className, CLASSNAMES.hiddenHeading)}>
        {children}
      </Heading>
    </Style>
  );
}
```

- `as`에 따라 Heading 컴포넌트가 동적으로 변화한다.(이를 다형성이라 한다.) 이 때문에 제대로 된 타입 추론이 되지 않아 타입 에러가 발생할 수 있다.
- 이를 해결하기 위해 일시적으로 타입을 `unknown`으로 지정, 해당 에러를 피할 수 있게 된다.
- 이 컴포넌트의 타입은 현재 알 수 없지만(`unknown`), as를 한 번 더 사용해 다시 필요한 타입으로 변환하게 된다.



참고

https://www.typescriptlang.org/ko/docs/handbook/2/everyday-types.html#%ED%83%80%EC%9E%85-%EB%8B%A8%EC%96%B8

https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-0.html#new-unknown-top-type

https://f-lab.kr/blog/polymorphism-with-type-safe