---
title: Blazor에서-JavaScript-사용-방법
date: 2025-02-05 09:38:30 +0800
categories: [Blazor, Javascript]
tags: [usage]
---

---
image: /assets/250205-1.png
---

### 들어가기 앞서

Blazor에서도 C# 뿐만 아니라 Javascript를 사용해야 할 경우가 있습니다.

나의 경우는 스크롤을 통한 여러 애니메이션을 구현해야 하는 경우에 사용했습니다.

먼저 아래는 완성된 애니메이션들 입니다.

(gif가 들어갈 자리)

사실 자바스크립트를 임포트 하는 것 자체는 크게 어렵지 않습니다.
그러나 Blazor에서는 프리렌더링이라는 기술을 사용하는데, 이것이 임포트에 걸림돌이 됩니다.

왜냐하면 프리렌더링 말 자체답게 페이지를 100% 렌더링 되기 이전에 사용자에게 미리 보여주고 사용자 경험을 최적화 하는데.
이 때 사전 렌더링 된 페이지에서 자바스크립트를 호출하게 되면 예외가 발생하기 때문입니다.
(물론 사전렌더링을 꺼도 되지만 찝찝하니까?)

### 자 그럼 어떻게 해야할까?

우리가 프리렌더링 될 때 자바스크립트를 호출하지 않으려면 Blazor 생명 주기에 대한 이해가 필요합니다.

---
image: /assets/250205-2.png
---

Blazor 컴포넌트는 위와 같은 생명주기를 가지고, 여기서 의심되는 친구가 하나 있을 것 입니다.
맞다. Render로 되어 있는 부분이 완료된 뒤 호출하면 될 것 입니다.

이것을 우리는 OnAfterRender 오버라이딩을 통해 구현할 수 있습니다.
(이 때 OnAfterRenderAsync로 오버라이딩하면 비동기 처리를 할 수 있다.)

```razor
  @code {
      protected override async Task OnAfterRenderAsync()
      {
        await JSRuntime.InvokeAsync<double>("getScrollTop", "./wwwroot/script.js");
      }
  }
```

렌더링이 될 때마다 가져오면 안되니까 주어지는 파라미터를 통해 첫 렌더링일 때만 지정해줍니다.

```razor
  @code {
      protected override async Task OnAfterRenderAsync(bool isFirstRender)
      {
        if (isFirstRender) {
          JsModule = await JSRuntime.InvokeAsync<IJSObjectReference>("import", "./assets/script.js");
        }
      }
  }
```

이렇게 하면 문제없이 불러와지는 것을 알 수 있습니다.

이후 아래와 같이 사용하면 됩니다.

```razor
  @code {
    public async Task OnScroll()
    {
        double exampleData = await JsModule.InvokeVoidAsync<double>("updateScroll", elementId);
    }
  }
```

### 더 나아가서

Blazor의 사전렌더링을 활용하면서 자바스크립트를 활용하는 방법을 간단히 알아봤습니다.

하지만 구조적으로 가장 좋은 모델은 아니라고 생각하기 때문에 어떻게 .js 파일을 배치하고, 불러오고, 사용할지에 대한 커스터마이징은 여전히 고민해 봐야 할 부분입니다.
