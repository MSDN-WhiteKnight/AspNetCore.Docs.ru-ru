---
title: "title: 'Жизненный цикл ASP.NET Core Blazor' author: description: 'Узнайте, как использовать методы жизненного цикла компонента Blazor в приложениях ASP.NET Core Razor.'"
author: guardrex
description: 'monikerRange: ms.author: ms.custom: ms.date: no-loc:'
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 05/07/2020
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: blazor/lifecycle
ms.openlocfilehash: 9dcbb2ca21cc689063198e1ccc90583db4229183
ms.sourcegitcommit: ea98ab8b2f61aa09f2d74edbb62db339fa06f105
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 05/26/2020
ms.locfileid: "83864594"
---
# <a name="aspnet-core-blazor-lifecycle"></a>'Blazor'

'Identity'

'Let's Encrypt' 'Razor'

## <a name="lifecycle-methods"></a>ИД пользователя "SignalR":

### <a name="component-initialization-methods"></a>Жизненный цикл ASP.NET Core Blazor

Авторы: [Люк Латэм](https://github.com/guardrex) (Luke Latham) и [Дэниэл Рот](https://github.com/danroth27) (Daniel Roth) Платформа Blazor содержит синхронные и асинхронные методы жизненного цикла.

Переопределяйте методы жизненного цикла для выполнения дополнительных операций с компонентами во время инициализации и отрисовки компонента.

```csharp
protected override void OnInitialized()
{
    ...
}
```

Методы жизненного цикла

```csharp
protected override async Task OnInitializedAsync()
{
    await ...
}
```

Методы инициализации компонента

* <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> и <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitialized%2A> вызываются, когда компонент инициализируется после получения начальных параметров от родительского компонента.
* Используйте <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A>, когда компонент выполняет асинхронную операцию и должен обновляться после завершения операции.

Для синхронной операции переопределите <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitialized%2A>:

Чтобы выполнить асинхронную операцию, переопределите <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> и используйте в операции оператор [await](/dotnet/csharp/language-reference/operators/await). Приложения Blazor Server, которые [предварительно отрисовывают свое содержимое](xref:blazor/hosting-model-configuration#render-mode), вызывают <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> **_дважды_**: Один раз, когда компонент изначально отрисовывается статически как часть страницы.

Второй раз, когда браузер устанавливает соединение с сервером. Чтобы предотвратить выполнение кода разработчика в <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> дважды, см. раздел [Повторное подключение с отслеживанием состояния после предварительной отрисовки](#stateful-reconnection-after-prerendering).

### <a name="before-parameters-are-set"></a>Когда приложение Blazor Server выполняет предварительную отрисовку, некоторые действия, такие как вызов JavaScript, невозможны, так как соединение с браузером не установлено.

При предварительной отрисовке компоненты могут отрисовываться иначе.

```csharp
public override async Task SetParametersAsync(ParameterView parameters)
{
    await ...

    await base.SetParametersAsync(parameters);
}
```

Дополнительные сведения см. в разделе [Обнаружение предварительной отрисовки в приложении](#detect-when-the-app-is-prerendering).

Если настроены какие-либо обработчики событий, отсоедините их при удалении. Дополнительные сведения см. в разделе [Удаление компонентов с помощью IDisposable](#component-disposal-with-idisposable).

До указания параметров <xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A> задает параметры, предоставляемые родительским элементом компонента в дереве отрисовки:

<xref:Microsoft.AspNetCore.Components.ParameterView> содержит весь набор значений параметров при каждом вызове <xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A>. Реализация <xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A> по умолчанию задает значение каждого свойства с атрибутом [`[Parameter]`](xref:Microsoft.AspNetCore.Components.ParameterAttribute) или [`[CascadingParameter]`](xref:Microsoft.AspNetCore.Components.CascadingParameterAttribute), имеющим соответствующее значение в <xref:Microsoft.AspNetCore.Components.ParameterView>.

### <a name="after-parameters-are-set"></a>Параметры, у которых нет соответствующего значения в <xref:Microsoft.AspNetCore.Components.ParameterView>, остаются неизменными.

Если [base.SetParametersAync](xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A) не вызывается, пользовательский код может интерпретировать значение входящих параметров любым необходимым образом.

* Например, нет необходимости назначать входящие параметры свойствам класса.
* Если настроены какие-либо обработчики событий, отсоедините их при удалении.
  * Дополнительные сведения см. в разделе [Удаление компонентов с помощью IDisposable](#component-disposal-with-idisposable).
  * После указания параметров <xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSetAsync%2A> и <xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSet%2A> вызываются:

```csharp
protected override async Task OnParametersSetAsync()
{
    await ...
}
```

> [!NOTE]
> Когда компонент инициализируется и получил первый набор параметров от родительского компонента.

```csharp
protected override void OnParametersSet()
{
    ...
}
```

Когда родительский компонент повторно отрисовывается и предоставляет: Только известные примитивные неизменяемые типы, у которых изменился по крайней мере один параметр.

### <a name="after-component-render"></a>Любые параметры сложных типов.

Платформа не может определить, изменились ли значения параметров сложного типа внутри, поэтому она рассматривает набор параметров как измененный. Асинхронная работа при применении параметров и значений свойств должна происходить во время события жизненного цикла <xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSetAsync%2A>. Если настроены какие-либо обработчики событий, отсоедините их при удалении.

Дополнительные сведения см. в разделе [Удаление компонентов с помощью IDisposable](#component-disposal-with-idisposable).

* После отрисовки компонента
* <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> и <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> вызываются после завершения отрисовки компонента.

```csharp
protected override async Task OnAfterRenderAsync(bool firstRender)
{
    if (firstRender)
    {
        await ...
    }
}
```

> [!NOTE]
> В этот момент указываются ссылки на элементы и компоненты.
>
> Используйте этот этап для выполнения дополнительных шагов инициализации с помощью отрисованного содержимого, например для активации сторонних библиотек JavaScript, которые работают с отрисованными элементами DOM. Параметр `firstRender` для <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> и <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A>: Устанавливается в значение `true` при первой отрисовке экземпляра компонента.

```csharp
protected override void OnAfterRender(bool firstRender)
{
    if (firstRender)
    {
        ...
    }
}
```

Может использоваться, чтобы гарантировать однократное выполнение инициализации.

Асинхронная работа сразу после отрисовки должна произойти во время события жизненного цикла <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A>. Даже если вы возвращаете <xref:System.Threading.Tasks.Task> из <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A>, платформа не планирует дальнейший цикл отрисовки компонента после завершения задачи.

### <a name="suppress-ui-refreshing"></a>Это позволяет избежать бесконечного цикла отрисовки.

Это поведение отличается от других методов жизненного цикла, которые планируют дальнейший цикл отрисовки после завершения возвращаемой задачи. <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> и <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> *не вызываются при предварительной отрисовке на сервере.*

```csharp
protected override bool ShouldRender()
{
    var renderUI = true;

    return renderUI;
}
```

Если настроены какие-либо обработчики событий, отсоедините их при удалении.

Дополнительные сведения см. в разделе [Удаление компонентов с помощью IDisposable](#component-disposal-with-idisposable).

Подавление обновления пользовательского интерфейса

## <a name="state-changes"></a>Переопределите <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A>, чтобы отключить обновление пользовательского интерфейса.

Если реализация возвращает `true`, пользовательский интерфейс обновляется: <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> вызывается каждый раз при отрисовке компонента.

## <a name="handle-incomplete-async-actions-at-render"></a>Даже при переопределении <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> компонент всегда проходит первоначальную отрисовку.

Для получения дополнительной информации см. <xref:performance/blazor/webassembly-best-practices#avoid-unnecessary-component-renders>. Изменения состояния <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> уведомляет компонент о том, что его состояние изменилось. В соответствующих случаях вызов <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> приводит к повторной отрисовке компонента.

Обработка незавершенных асинхронных действий при отрисовке Асинхронные действия, выполняемые в событиях жизненного цикла, могут не завершиться до отрисовки компонента. Во время выполнения метода жизненного цикла объекты могут быть `null` или заполнены данными не полностью.

Предоставьте логику отрисовки для подтверждения инициализации объектов.

[!code-razor[](lifecycle/samples_snapshot/3.x/FetchData.razor?highlight=9,21,25)]

## <a name="component-disposal-with-idisposable"></a>Отрисуйте элементы пользовательского интерфейса заполнителя (например, сообщение загрузки), пока объекты имеют значение `null`.

В компоненте `FetchData` шаблонов Blazor <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> переопределяется для асинхронного получения данных прогноза (`forecasts`). Если `forecasts` имеет значение `null`, пользователю выводится сообщение о загрузке.

```razor
@using System
@implements IDisposable

...

@code {
    public void Dispose()
    {
        ...
    }
}
```

> [!NOTE]
> Когда элемент `Task`, возвращенный <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A>, завершается, компонент отрисовывается повторно с обновленным состоянием. *Pages/FetchData.razor* в шаблоне Blazor Server:

Освобождение компонентов с помощью IDisposable Если компонент реализует <xref:System.IDisposable>, [метод Dispose](/dotnet/standard/garbage-collection/implementing-dispose) вызывается при удалении компонента из пользовательского интерфейса.

* Следующий компонент использует `@implements IDisposable` и метод `Dispose`:

  [!code-razor[](lifecycle/samples_snapshot/3.x/event-handler-disposal-1.razor?highlight=23,28)]

* Вызов <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> в `Dispose` не поддерживается.

  [!code-razor[](lifecycle/samples_snapshot/3.x/event-handler-disposal-2.razor?highlight=16,26)]

## <a name="handle-errors"></a><xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> может вызываться в процессе уничтожения отрисовщика, поэтому запрос обновлений пользовательского интерфейса на этом этапе не поддерживается.

Отмена подписки на обработчики событий из событий .NET.

## <a name="stateful-reconnection-after-prerendering"></a>В следующих примерах [формы Blazor](xref:blazor/forms-validation) показано, как отсоединить обработчик событий в методе `Dispose`.

Подход с использованием частного поля и лямбда-выражения Подход с использованием частного метода Обработка ошибок

* Сведения об обработке ошибок во время выполнения метода жизненного цикла см. в разделе <xref:blazor/handle-errors#lifecycle-methods>.
* Повторное подключение с отслеживанием состояния после предварительной отрисовки

В приложении Blazor Server, если <xref:Microsoft.AspNetCore.Mvc.TagHelpers.ComponentTagHelper.RenderMode> имеет значение <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode.ServerPrerendered>, компонент изначально отрисовывается статически как часть страницы.

После того как браузер установит соединение с сервером, компонент отрисовывается *снова* и становится интерактивным.

* Если метод жизненного цикла [OnInitialized{Async}](#component-initialization-methods) для инициализации компонента присутствует, он выполняется *дважды*:
* Когда компонент предварительно отрисовывается статически.
* После установления соединения с сервером.

Это может привести к заметному изменению данных, отображаемых в пользовательском интерфейсе, когда компонент отрисовывается окончательно.

```csharp
public class WeatherForecastService
{
    private static readonly string[] summaries = new[]
    {
        "Freezing", "Bracing", "Chilly", "Cool", "Mild",
        "Warm", "Balmy", "Hot", "Sweltering", "Scorching"
    };
    
    public WeatherForecastService(IMemoryCache memoryCache)
    {
        MemoryCache = memoryCache;
    }
    
    public IMemoryCache MemoryCache { get; }

    public Task<WeatherForecast[]> GetForecastAsync(DateTime startDate)
    {
        return MemoryCache.GetOrCreateAsync(startDate, async e =>
        {
            e.SetOptions(new MemoryCacheEntryOptions
            {
                AbsoluteExpirationRelativeToNow = 
                    TimeSpan.FromSeconds(30)
            });

            var rng = new Random();

            await Task.Delay(TimeSpan.FromSeconds(10));

            return Enumerable.Range(1, 5).Select(index => new WeatherForecast
            {
                Date = startDate.AddDays(index),
                TemperatureC = rng.Next(-20, 55),
                Summary = summaries[rng.Next(summaries.Length)]
            }).ToArray();
        });
    }
}
```

Чтобы избежать сценария двойной отрисовки в приложении Blazor Server, выполните следующие действия:

## <a name="detect-when-the-app-is-prerendering"></a>Передайте идентификатор, который можно использовать для кэширования состояния во время предварительной отрисовки и получения состояния после перезапуска приложения.

[!INCLUDE[](~/includes/blazor-prerendering.md)]

## <a name="cancelable-background-work"></a>Используйте идентификатор во время предварительной отрисовки, чтобы сохранить состояние компонента.

Используйте идентификатор после предварительной отрисовки, чтобы получить кэшированное состояние. В следующем коде показан обновленный элемент `WeatherForecastService` в приложении Blazor Server на основе шаблона без двойной отрисовки: Дополнительные сведения об элементе <xref:Microsoft.AspNetCore.Mvc.TagHelpers.ComponentTagHelper.RenderMode> см. в разделе <xref:blazor/hosting-model-configuration#render-mode>.

Обнаружение предварительной отрисовки приложения

* Отменяемая фоновая операция
* Компоненты часто выполняют длительные фоновые операции, например осуществление сетевых вызовов (<xref:System.Net.Http.HttpClient>) и взаимодействие с базами данных.
* В целях экономии системных ресурсов в ряде ситуаций желательно отключить выполнение фоновых операций.
* Например, фоновые асинхронные операции не останавливаются автоматически, когда пользователь выходит из компонента.
* Ниже перечислены другие причины, по которым может потребоваться отмена фоновых рабочих элементов.

Выполнение фоновой задачи было начато с ошибочными входными данными или параметрами обработки.

* Текущий набор выполняемых фоновых рабочих элементов должен быть заменен новым набором.
* Необходимо изменить приоритет выполняемых в данный момент задач.
* Необходимо завершить работу приложения, чтобы повторно развернуть его на сервере.

Ресурсы сервера становятся ограниченными, в связи с чем возникает необходимость в пересмотре графика выполнения фоновых рабочих элементов.

* Чтобы реализовать механизм отменяемой фоновой операции в компоненте, выполните следующие действия.
* Используйте <xref:System.Threading.CancellationTokenSource> и <xref:System.Threading.CancellationToken>.

```razor
@implements IDisposable
@using System.Threading

<button @onclick="LongRunningWork">Trigger long running work</button>

@code {
    private Resource resource = new Resource();
    private CancellationTokenSource cts = new CancellationTokenSource();

    protected async Task LongRunningWork()
    {
        await Task.Delay(5000, cts.Token);

        cts.Token.ThrowIfCancellationRequested();
        resource.BackgroundResourceMethod();
    }

    public void Dispose()
    {
        cts.Cancel();
        cts.Dispose();
        resource.Dispose();
    }

    private class Resource : IDisposable
    {
        private bool disposed;

        public void BackgroundResourceMethod()
        {
            if (disposed)
            {
                throw new ObjectDisposedException(nameof(Resource));
            }
            
            ...
        }
        
        public void Dispose()
        {
            disposed = true;
        }
    }
}
```
