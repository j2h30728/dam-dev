---
sidebar_position: 2
---

# Funnel 컴포넌트로 추상화

여러 연속적인 화면을 효율적으로 구성할 수 있도록, **Funnel 방식**을 채택하여 페이지 흐름을 관리했습니다.<br/>
이를 기반으로 재사용 가능한 **커스텀 훅**을 개발하였으며, 페이지의 이름을 기준으로 즉시 렌더링할 수 있도록 설계되었습니다.

## useFunnel 커스텀훅 생성

- [feat: 강아지 등록 폼에 사용할 funnel 커스텀훅 생성](https://github.com/meong-story/meong-story-FE/pull/20/commits/3925108c4d05304463ecc4a5a2dfbaf297ca7533)

### 구현 내용

1. 제네릭 타입을 사용
   - 커스텀훅을 사용할 떄, 원하는 페이지의 스텝들을 리터럴 타입의 유니온 타입을 넣어 줍니다.
   - 리터럴 타입과 페이지 이름과 동일하게 두어 명시적으로 사용할 수 있게 합니다.
2. 현재 보여줄 스텝(페이지)을 찾는 방법
   - 페이지 스텝마다 제네릭 터입에 주입한 리터럴 타입과 동일한 name 을 지정합니다.
   - `useState`의 `setState`를 사용해 현재 상태와 페이지 스텝의 이름과 매칭되는 스텝을 렌더링 합니다.
3. 스텝은 퍼널 내부에서만 사용해야함을 명시하기 위해서 컴파운드 컴포넌트 패턴을 사용합니다.
   - 루트 컴포넌트를 퍼널로 두고 각 스텝을 하위 컴포넌트로 포함시킵니다.
   - 스텝은 퍼널 내부의 일부분으로 취급되게 하여, 스텝은 퍼널 내부에서만 사용하게 합니다.
   - `useState`로 현재 스텝을 관리하며 `setState`를 사용하여 외부에서 직접 상태 기반으로 스텝을 동적으로 변경 시킬수 있습니다.

```typescript
import { ReactElement, ReactNode, useState } from "react";

export interface Step {
  name: string;
  children: ReactNode;
}

export interface Funnel {
  children: Array<ReactElement<Step>>;
}

export const useFunnel = <T extends string>(defaultStep: T) => {
  const [step, setStep] = useState(defaultStep);

  const Step = (props: Step): ReactElement => {
    return <>{props.children}</>;
  };

  const FunnelRoot = ({ children }: Funnel) => {
    const targetStep = children.find((childStep) => childStep.props.name === step);
    return <>{targetStep}</>;
  };
  const Funnel = Object.assign(FunnelRoot, { Step });

  return [Funnel, setStep] as const;
};
```

### 사용 방법

1. `useFunnel` 사용과 동시에 퍼널의 스텝으로 사용할 페이지 이름(name)을 유니온 타입으로 지정합니다.
2. 초기값에 들어가는 이름은 퍼널 스텝의 첫 번쨰 페이지입니다.
3. 퍼널 훅 내에서 이름(name)을 사용해 현재 보여줄 스텝을 결정합니다.
4. 직접 `setStep`을 외부에서 넣어주어 어떤 행동이 어떤 스텝으로 갈지 결정합니다.

:::info
react-hook-form의 `FormProvider`,`useForm`, `useFormContext`를 결합하여 사용하면 같은 퍼널 내부에서 폼 상태를 공유할 수 있습니다.
:::

> 스텝(페이지) 이름의 문자열은 상수화 시켜 사용 가능합니다.
>
> ```ts
> const UPLOAD_STEP = {
>   인증성공: "인증 성공",
>   인증순간남기기: "인증 순간 남기기 폼",
>   인증순간남기기성공: "인증 순간 남기기 성공",
> } as const;
> ```

#### 적용한 부분

```ts
const RegisterPage = () => {
  const [Funnel, setStep] = useFunnel<"계정확인" | "펫확인" | "펫등록하기" | "등록및초대선택하기" | "초대링크입력">(
    "등록및초대선택하기"
  );

  return (
    <Funnel>
      <Funnel.Step name="등록및초대선택하기">
        <SelectPet onResister={() => setStep("펫등록하기")} onInvitationLink={() => setStep("초대링크입력")} />
      </Funnel.Step>
      <Funnel.Step name="초대링크입력">
        <InputInvitationLink onNext={() => setStep("펫확인")} onPrevious={() => setStep("등록및초대선택하기")} />
      </Funnel.Step>
      <Funnel.Step name="펫확인">
        <ConfirmPet onInCorrect={() => setStep("초대링크입력")} />
      </Funnel.Step>
      <Funnel.Step name="펫등록하기">
        <RegisterPet onPrevious={() => setStep("등록및초대선택하기")} />
      </Funnel.Step>
    </Funnel>
  );
};
```

- 각 스텝의 이름을 **리터럴 값의 유니온 타입**으로 설정하여, **휴먼 에러**를 방지하며 안정적인 페이지 흐름을 제공했습니다.

---

- [관련 풀리퀘스트](https://github.com/meong-story/meong-story-FE/pull/20)
