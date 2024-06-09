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


## useFunnel

**`useFunnel<Step[]>(steps, options)`**

```tsx
export const useFunnel = <Steps extends NonEmptyArray<string>>(
  steps: Steps,
  options?: {
    /**
     * 이 query key는 현재 스텝을 query string에 저장하기 위해 사용됩니다.
     * @default 'funnel-step'
     */
    stepQueryKey?: string;
    initialStep?: Steps[number];
    onStepChange?: (name: Steps[number]) => void;
  }
)
```

- `steps`
    - `Steps[]` 타입의 문자열 배열을 받아 이를 기반으로 단계를 관리
    - `Steps`는 `NonEmptyArray<string>`를 확장한 타입,
        - `type NonEmptyArray<T> = readonly [T, ...T[]];` : readonly 읽기 전용 타입
- `options`: optional
    - `stepQueryKey`
        - 현재 스텝을 query string에 저장하기 위해 사용, 기본값 `funnel-step`
    - `initialStep`
    - `onStepChange`
        - step이 변경될 때마다 전달된 콜백이 호출됨

```tsx
const FunnelComponent = useMemo(
  () =>
    Object.assign(
      function RouteFunnel(props: RouteFunnelProps<Steps>) {
        // eslint-disable-next-line react-hooks/rules-of-hooks
        const step = useQueryParam<Steps[number]>(stepQueryKey) ?? options?.initialStep;

        assert(
          step != null,
          `표시할 스텝을 ${stepQueryKey} 쿼리 파라미터에 지정해주세요. 쿼리 파라미터가 없을 때 초기 스텝을 렌더하려면 useFunnel의 두 번째 파라미터 options에 initialStep을 지정해주세요.`
        );

        return <Funnel<Steps> steps={steps} step={step} {...props} />;
      },
      {
        Step,
      }
    ),
  // eslint-disable-next-line react-hooks/exhaustive-deps
  []
);

```

- [`Object.assign(target, …sources)`](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)
    - source 객체를 target 객체에 병합한 결과를 반환해주는 메서드
    - 동일한 key를 갖는 경우 sources 객체의 값으로 덮어쓰게 됨
    
    ⇒ target과 source 객체 두 가지 요소를 포함한 새로운 객체가 생성됨
    
- `RouteFunnel`
    - useQueryParam을 활용해 현재 step을 가져오고 그에 따른 `Funnel`을 렌더링함
- `Step`
    - `Funnel` 컴포넌트에 선언된 `Step`
    - 특정 단계에 진입 했을 때 콜백 함수 실행 혹은 자식 컴포넌트 렌더링하는 역할

```tsx
const setStep = useCallback(
  (step: Steps[number], setStepOptions?: SetStepOptions) => {
    const { preserveQuery = true, query = {} } = setStepOptions ?? {};

    const url = `${QS.create({
      ...(preserveQuery ? router.query : undefined),
      ...query,
      [stepQueryKey]: step,
    })}`;

    options?.onStepChange?.(step);

    switch (setStepOptions?.stepChangeType) {
      case 'replace':
        router.replace(url, undefined, {
          shallow: true,
        });
        return;
      case 'push':
      default:
        router.push(url, undefined, {
          shallow: true,
        });
        return;
    }
  },
  // eslint-disable-next-line react-hooks/exhaustive-deps
  [options, router]
);
```

- `step`
    - 변경할 단계
- `setStepOptions`
    - 단계 변경에 대한 옵션: 쿼리 파라미터를 보존할지, 쿼리 파라미터 변경 등
- `preserveQuery`를 기반으로 변경된 url 생성
    - [QS.create](https://www.slash.page/ko/libraries/common/utils/src/querystring.i18n/#createquerystring-qscreate)를 활용해 쿼리 스트링을 제작
- step이 변경될 때마다 호출되는 함수 `onStepChange`가 호출됨
- `setStepOptions`의 `stepChangeType`에는 `push`와 `replace`가 있음
    
    → 해당 type에 따라 새로운 url로 이동하거나(`push`) 현재 페이지의 url을 새로운 url로 교체(`replace`)하는 동작 수행