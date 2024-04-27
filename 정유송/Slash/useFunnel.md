# useFunnel

## 먼저 퍼널이란?

[https://www.youtube.com/results?search_query=토스+퍼널+](https://www.youtube.com/results?search_query=%ED%86%A0%EC%8A%A4+%ED%8D%BC%EB%84%90+)

- 설문조사 패턴: 여러 페이지들을 통해 상태를 수집하고, 결과 페이지를 보여주는 형태
- → 이 형태를 퍼널이라고 부른다.

## 사용 예시

https://www.slash.page/ko/libraries/react/use-funnel/README.i18n

```tsx
const KyoboLifeFunnel = () => {
  const [Funnel, state, setStep] = useFunnel(['아파트여부', '지역선택', '완료'] as const).withState<{
    propertyType?: '빌라' | '아파트';
    address?: string;
  }>({});

  const 상담신청 = useLoanApplicationCallback();

  return (
    <Funnel>
      <Funnel.Step name="아파트여부">
        <아파트여부스텝 지역선택으로가기={() => setStep(prev => ({...prev, step: '지역선택', isApartment: true}))} />
      </Funnel.Step>
      <Funnel.Step name="지역선택">
        <지역선택스텝 지역선택완료={(지역정보) => setStep(prev => ({...prev, step: '완료', region: 지역정보}))} />
      </Funnel.Step>
      <Funnel.Step name="완료">
        <완료스텝 신청={() => 상담신청(state)} />
      </Funnel.Step>
    </Funnel>
  );
};
```

- `Funnel`: 어떤 스텝을 보여줄지 Conditional Rendering을 수행하는 역할
- `setStep`: 스텝을 변경할 수 있는 함수

- 퍼널의 상태 관리는 useFunnel을 사용하는 곳에서! → 흐름 파악 용이
    - `withState`: useFunnel과 useState를 함께 쓸 때 발생하는 상태 동기화 이슈 해결을 위한
- 스텝 전환은 useFunnel을 사용하는 곳에서 파악할 수 있도록
    - setState를 props로 컴포넌트에 주입해 스텝을 전환하는 로직은 지양

## Funnel

```tsx
/** @tossdocs-ignore */
import { assert } from '@toss/assert';
import { Children, isValidElement, ReactElement, ReactNode, useEffect } from 'react';
import { NonEmptyArray } from './models';

export interface FunnelProps<Steps extends NonEmptyArray<string>> {
  steps: Steps;
  step: Steps[number];
  children: Array<ReactElement<StepProps<Steps>>> | ReactElement<StepProps<Steps>>;
}

export const Funnel = <Steps extends NonEmptyArray<string>>({ steps, step, children }: FunnelProps<Steps>) => {
  const validChildren = Children.toArray(children)
    .filter(isValidElement)
    .filter(i => steps.includes((i.props as Partial<StepProps<Steps>>).name ?? '')) as Array<
    ReactElement<StepProps<Steps>>
  >;

  const targetStep = validChildren.find(child => child.props.name === step);

  assert(targetStep != null, `${step} 스텝 컴포넌트를 찾지 못했습니다.`);

  return <>{targetStep}</>;
};

export interface StepProps<Steps extends NonEmptyArray<string>> {
  name: Steps[number];
  onEnter?: () => void;
  children: ReactNode;
}

export const Step = <Steps extends NonEmptyArray<string>>({ onEnter, children }: StepProps<Steps>) => {
  useEffect(() => {
    onEnter?.();
  }, [onEnter]);

  return <>{children}</>;
};
```

- children으로 받는 자식 요소(컴포넌트)를 배열로 만들어 ReactElement인지 확인
    - 자식 컴포넌트의 props의 name이 steps 배열에 있는 요소인지를 확인
- Funnel의 step props에 따라 targetStep (=자식 요소)가 정해지고 이것이 반환됨
- 고차 컴포넌트를 활용해 name 부여