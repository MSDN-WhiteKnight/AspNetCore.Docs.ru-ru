---
title: Создание и использование компонентов Razor ASP.NET Core
author: guardrex
description: Сведения о том, как создавать и использовать компоненты Razor, включая способы привязки к данным, обработки событий и управления жизненным циклом компонентов.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 06/11/2020
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: blazor/components
ms.openlocfilehash: 2a6de1a39737f98cb151a0556f36c223d86f9752
ms.sourcegitcommit: d243fadeda20ad4f142ea60301ae5f5e0d41ed60
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 06/12/2020
ms.locfileid: "84723955"
---
# <a name="create-and-use-aspnet-core-razor-components"></a>Создание и использование компонентов Razor ASP.NET Core

Авторы: [Люк Латэм (Luke Latham)](https://github.com/guardrex), [Дэниэл Рот (Daniel Roth)](https://github.com/danroth27) и Тобиас Бартщ [(Tobias Bartsch)](https://www.aveo-solutions.com/)

[Просмотреть или скачать образец кода](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/common/samples/) ([как скачивать](xref:index#how-to-download-a-sample))

Приложения Blazor создаются с использованием *компонентов*. Компонент — это автономный блок пользовательского интерфейса, такой как страница, диалоговое окно или форма. Компонент включает разметку HTML и логику обработки, необходимую для внедрения данных или реагирования на события пользовательского интерфейса. Компоненты являются гибкими и облегченными. Их можно вкладывать, использовать повторно и сразу в нескольких проектах.

## <a name="component-classes"></a>Классы компонентов

Компоненты реализуются в файлах компонентов [Razor](xref:mvc/views/razor) ( *.razor*) с помощью комбинации разметки HTML и C#. Компонент в Blazor формально называется *компонентом Razor* .

Имя компонента должно начинаться с заглавной буквы. Например, *MyCoolComponent.razor* является допустимым, а *myCoolComponent.razor* нет.

Пользовательский интерфейс для компонента определяется с помощью HTML. Логика динамического отображения (например, выражения, циклы и условные выражения) добавляется с помощью встроенного синтаксиса C# под названием *Razor* . Во время компиляции приложения разметка HTML и логика отрисовки C# преобразуются в класс компонента. Имя создаваемого класса соответствует имени файла.

Элементы класса компонента определяются в блоке [`@code`][1]. В блоке [`@code`][1] указываются состояние (свойства, поля) компонента и методы для обработки событий или определения другой логики компонента. Допускается использование нескольких блоков [`@code`][1].

Элементы компонента можно использовать как часть логики отрисовки компонента с использованием выражений C#, начинающихся с `@`. Например, поле C# отрисовывается путем добавления `@` к имени поля. В следующем примере вычисляется и отрисовывается следующее:

* `headingFontStyle` в значении свойства CSS для `font-style`.
* `headingText` в содержимом элемента `<h1>`.

```razor
<h1 style="font-style:@headingFontStyle">@headingText</h1>

@code {
    private string headingFontStyle = "italic";
    private string headingText = "Put on your new Blazor!";
}
```

После первоначальной отрисовки компонента он повторно создает дерево отрисовки в ответ на события. Затем Blazor сравнивает новое и прежнее дерево отрисовки и применяет все изменения в модели DOM браузера.

Компоненты являются обычными классами C# и могут размещаться в любом месте внутри проекта. Компоненты, создающие веб-страницы, обычно находятся в папке *Pages*. Компоненты, не связанные со страницами, часто находятся в папке *Shared* или настраиваемой папке, добавленной в проект.

Как правило, пространство имен компонента является производным от корневого пространства имен приложения и расположения компонента (папки) в приложении. Если корневым пространством имен приложения является `BlazorApp`, а компонент `Counter` находится в папке *Pages*:

* Пространством имен компонента `Counter` является `BlazorApp.Pages`.
* Полным именем компонента является `BlazorApp.Pages.Counter`.

При использовании пользовательских папок, содержащих компоненты, добавьте инструкцию `using` в родительский компонент или в файл *_Imports.razor* приложения. В следующем примере становятся доступными компоненты в папке *Компоненты*:

```razor
@using BlazorApp.Components
```

Кроме того, на компонент можно ссылаться напрямую:

```razor
<BlazorApp.Components.MyCoolComponent />
```

Дополнительные сведения см. в разделе [Импорт компонентов](#import-components).

## <a name="razor-syntax"></a>Синтаксис Razor

В компонентах Razor приложений Blazor часто используется синтаксис Razor. Если вы не знакомы с языком разметки Razor, сначала прочтите статью <xref:mvc/views/razor>.

При доступе к содержимому с синтаксисом Razor обратите особое внимание на следующее:

* [Директивы](xref:mvc/views/razor#directives) — это зарезервированные ключевые слова с префиксом `@`, которые обычно меняют то, как разметка компонента анализируется и функционирует.
* [Атрибуты директивы](xref:mvc/views/razor#directive-attributes) — зарезервированные ключевые слова с префиксом `@`, которые обычно меняют то, как разметка компонента анализируется и функционирует.

## <a name="static-assets"></a>Статические ресурсы.

Blazor соответствует соглашению для приложений ASP.NET Core о размещении статических ресурсов в [корневом веб-каталоге (wwwroot)](xref:fundamentals/index#web-root) проекта.

Используйте базовый относительный путь (`/`) для ссылки на корневой веб-каталог статического ресурса. В следующем примере *logo.png* физически находится в папке *{PROJECT ROOT}/wwwroot/images*:

```razor
<img alt="Company logo" src="/images/logo.png" />
```

Компоненты Razor **не** поддерживают нотацию тильды с косой чертой (`~/`).

Сведения о настройке базового пути приложения см. в разделе <xref:host-and-deploy/blazor/index#app-base-path>.

## <a name="tag-helpers-arent-supported-in-components"></a>Вспомогательные функции тегов в компонентах не поддерживаются

[Вспомогательные функции тегов](xref:mvc/views/tag-helpers/intro) не поддерживаются в компонентах Razor (файлы *.razor*). Чтобы обеспечить функциональные возможности, аналогичные вспомогательным функциям тегов, в Blazor, создайте компонент с теми же функциональными возможностями, что и вспомогательная функция тега, и используйте его вместо нее.

## <a name="use-components"></a>Использование компонентов

Компоненты могут включать другие компоненты, объявляя их с помощью синтаксиса HTML-элементов. Разметка для использования компонента выглядит как тег HTML с именем, соответствующем типу компонента.

Следующая разметка в *Index.razor* визуализирует экземпляр `HeadingComponent`:

```razor
<HeadingComponent />
```

*Components/HeadingComponent.razor*:

[!code-razor[](common/samples/3.x/BlazorWebAssemblySample/Components/HeadingComponent.razor)]

Если компонент содержит HTML-элемент с первой заглавной буквой, который не соответствует имени компонента, выдается предупреждение о том, что элемент имеет непредвиденное имя. Добавление директивы [`@using`][2] для пространства имен компонента делает компонент доступным, что позволяет устранить это предупреждение.

## <a name="routing"></a>Маршрутизация

Маршрутизация в Blazor достигается путем предоставления шаблона маршрута каждому доступному компоненту в приложении.

При компиляции файла Razor с директивой [`@page`][9] созданному классу предоставляется атрибут <xref:Microsoft.AspNetCore.Mvc.RouteAttribute>, указывающий шаблон маршрута. Во время выполнения маршрутизатор ищет классы компонентов с <xref:Microsoft.AspNetCore.Mvc.RouteAttribute> и отображает любой компонент, шаблон маршрута которого соответствует запрошенному URL-адресу.

```razor
@page "/ParentComponent"

...
```

Для получения дополнительной информации см. <xref:blazor/routing>.

## <a name="parameters"></a>Параметры

### <a name="route-parameters"></a>Параметры маршрута

Компоненты могут принимать параметры маршрута из шаблона маршрута, указанного в директиве [`@page`][9]. Маршрутизатор использует параметры маршрута для заполнения соответствующих параметров компонента.

*Pages/RouteParameter.razor*:

[!code-razor[](components/samples_snapshot/RouteParameter.razor?highlight=2,7-8)]

Необязательные параметры не поддерживаются, поэтому в предыдущем примере применяются две директивы [`@page`][9]. Первая позволяет переходить к компоненту без параметра. Вторая директива [`@page`][9] принимает параметр маршрута `{text}` и присваивает значение свойству `Text`.

Синтаксис *универсального* параметра (`*`/`**`) **не** поддерживается в компонентах Razor ( *.razor*).

### <a name="component-parameters"></a>Параметры компонентов

Компоненты могут иметь *параметры*, определяемые с помощью открытых свойств в классе компонента с атрибутом [`[Parameter]`](xref:Microsoft.AspNetCore.Components.ParameterAttribute). Используйте атрибуты, чтобы указать аргументы для компонента в разметке.

*Components/ChildComponent.razor*:

[!code-razor[](common/samples/3.x/BlazorWebAssemblySample/Components/ChildComponent.razor?highlight=2,11-12)]

В следующем примере из примера приложения `ParentComponent` задает значение свойства `Title` для `ChildComponent`.

*Pages/ParentComponent.razor*:

[!code-razor[](components/samples_snapshot/ParentComponent.razor?highlight=5-6)]

> [!WARNING]
> Не создавайте компоненты, записывающие их в собственные *параметры компонентов*, — используйте вместо этого закрытое поле. Дополнительные сведения см. в разделе [Не создавать компоненты, которые записываются в собственные свойства параметров](#dont-create-components-that-write-to-their-own-parameter-properties).

## <a name="child-content"></a>Дочернее содержимое

Компоненты могут задавать содержимое другого компонента. Назначение компонента задает содержимое между тегами, указывающими принимающий компонент.

В следующем примере `ChildComponent` имеет свойство `ChildContent`, представляющее <xref:Microsoft.AspNetCore.Components.RenderFragment>, который представляет сегмент пользовательского интерфейса для отрисовки. Значение `ChildContent` находится в том месте разметки компонента, где должно быть визуализировано содержимое. Значение `ChildContent` принимается от родительского компонента и визуализируется внутри `panel-body` панели начальной загрузки.

*Components/ChildComponent.razor*:

[!code-razor[](common/samples/3.x/BlazorWebAssemblySample/Components/ChildComponent.razor?highlight=3,14-15)]

> [!NOTE]
> Свойству, принимающему содержимое <xref:Microsoft.AspNetCore.Components.RenderFragment>, по соглашению необходимо присвоить имя `ChildContent`.

`ParentComponent` в примере приложения может предоставить содержимое для отрисовки `ChildComponent`, заключив содержимое в теги `<ChildComponent>`.

*Pages/ParentComponent.razor*:

[!code-razor[](components/samples_snapshot/ParentComponent.razor?highlight=7-8)]

## <a name="attribute-splatting-and-arbitrary-parameters"></a>Сплаттинг атрибутов и произвольные параметры

Компоненты могут записывать и визуализировать дополнительные атрибуты в дополнение к объявленным параметрам компонента. Можно записать дополнительные атрибуты в словарь, а затем выполнить *сплаттинг* для элемента при подготовке отрисовки компонента с помощью директивы [`@attributes`][3] Razor. Этот сценарий полезен при определении компонента, который создает элемент разметки, поддерживающий разнообразные настройки. Например, может оказаться утомительным по отдельности определять атрибуты для `<input>`, поддерживающего много параметров.

В следующем примере первый элемент `<input>` (`id="useIndividualParams"`) использует отдельные параметры компонента, а второй элемент `<input>` (`id="useAttributesDict"`) использует сплаттинг атрибутов:

```razor
<input id="useIndividualParams"
       maxlength="@Maxlength"
       placeholder="@Placeholder"
       required="@Required"
       size="@Size" />

<input id="useAttributesDict"
       @attributes="InputAttributes" />

@code {
    [Parameter]
    public string Maxlength { get; set; } = "10";

    [Parameter]
    public string Placeholder { get; set; } = "Input placeholder text";

    [Parameter]
    public string Required { get; set; } = "required";

    [Parameter]
    public string Size { get; set; } = "50";

    [Parameter]
    public Dictionary<string, object> InputAttributes { get; set; } =
        new Dictionary<string, object>()
        {
            { "maxlength", "10" },
            { "placeholder", "Input placeholder text" },
            { "required", "required" },
            { "size", "50" }
        };
}
```

Тип параметра должен реализовывать `IEnumerable<KeyValuePair<string, object>>` с ключами строки. В этом сценарии также можно использовать `IReadOnlyDictionary<string, object>`.

При использовании обоих подходов получаются идентичные отрисованные элементы `<input>`:

```html
<input id="useIndividualParams"
       maxlength="10"
       placeholder="Input placeholder text"
       required="required"
       size="50">

<input id="useAttributesDict"
       maxlength="10"
       placeholder="Input placeholder text"
       required="required"
       size="50">
```

Чтобы принять произвольные атрибуты, определите параметр компонента с помощью атрибута [`[Parameter]`](xref:Microsoft.AspNetCore.Components.ParameterAttribute) со свойством <xref:Microsoft.AspNetCore.Components.ParameterAttribute.CaptureUnmatchedValues>, имеющим значение `true`.

```razor
@code {
    [Parameter(CaptureUnmatchedValues = true)]
    public Dictionary<string, object> InputAttributes { get; set; }
}
```

Свойство <xref:Microsoft.AspNetCore.Components.ParameterAttribute.CaptureUnmatchedValues> в [`[Parameter]`](xref:Microsoft.AspNetCore.Components.ParameterAttribute) позволяет параметру соответствовать всем атрибутам, которые не соответствуют никакому другому параметру. Компонент может определять только один параметр с <xref:Microsoft.AspNetCore.Components.ParameterAttribute.CaptureUnmatchedValues>. Тип свойства, используемый с <xref:Microsoft.AspNetCore.Components.ParameterAttribute.CaptureUnmatchedValues>, должен быть назначаемым из `Dictionary<string, object>` с ключами строки. В этом сценарии также можно использовать `IEnumerable<KeyValuePair<string, object>>` или `IReadOnlyDictionary<string, object>`.

Расположение [`@attributes`][3] относительно положения атрибутов элемента имеет значение. Когда выполняется сплаттинг [`@attributes`][3] для элемента, атрибуты обрабатываются справа налево (от последнего к первому). Рассмотрим следующий пример компонента, использующего компонент `Child`:

*ParentComponent.razor*:

```razor
<ChildComponent extra="10" />
```

*ChildComponent.razor*:

```razor
<div @attributes="AdditionalAttributes" extra="5" />

[Parameter(CaptureUnmatchedValues = true)]
public IDictionary<string, object> AdditionalAttributes { get; set; }
```

Атрибут `extra` компонента `Child` стоит справа от [`@attributes`][3]. `<div>`, визуализируемый компонентом `Parent`, содержит `extra="5"` при передаче через дополнительный атрибут, так как атрибуты обрабатываются справа налево (от последнего к первому):

```html
<div extra="5" />
```

В следующем примере порядок `extra` и [`@attributes`][3] в `<div>` компонента `Child` изменен на противоположный:

*ParentComponent.razor*:

```razor
<ChildComponent extra="10" />
```

*ChildComponent.razor*:

```razor
<div extra="5" @attributes="AdditionalAttributes" />

[Parameter(CaptureUnmatchedValues = true)]
public IDictionary<string, object> AdditionalAttributes { get; set; }
```

Визуализируемый `<div>` в компоненте `Parent` содержит `extra="10"` при передаче через дополнительный атрибут:

```html
<div extra="10" />
```

## <a name="capture-references-to-components"></a>Запись ссылок на компоненты

Ссылки на компоненты позволяют ссылаться на экземпляр компонента, чтобы можно было выполнять команды для этого экземпляра, например `Show` или `Reset`. Чтобы записать ссылку на компонент, сделайте следующее:

* Добавьте к дочернему компоненту атрибут [`@ref`][4].
* Определите поле с тем же типом, что и у дочернего компонента.

```razor
<MyLoginDialog @ref="loginDialog" ... />

@code {
    private MyLoginDialog loginDialog;

    private void OnSomething()
    {
        loginDialog.Show();
    }
}
```

При отрисовке компонента поле `loginDialog` заполняется экземпляром дочернего компонента `MyLoginDialog`. Затем можно вызывать методы .NET в экземпляре компонента.

> [!IMPORTANT]
> Переменная `loginDialog` заполняется только после отрисовки компонента, а ее выходные данные включают элемент `MyLoginDialog`. До этого момента ссылаться не на что. Для управления ссылками на компоненты после завершения отрисовки компонента используйте [методы OnAfterRenderAsync или OnAfterRender](xref:blazor/lifecycle#after-component-render).

Сведения о ссылках на компоненты в цикле см. в разделе [Получение ссылок на несколько схожих дочерних компонентов (dotnet/aspnetcore #13358)](https://github.com/dotnet/aspnetcore/issues/13358).

При записи ссылок на компоненты используется аналогичный синтаксис для [записи ссылок на элементы](xref:blazor/call-javascript-from-dotnet#capture-references-to-elements), это не функция взаимодействия JavaScript. Ссылки на компоненты не передаются в код JavaScript. Они используются только в коде .NET.

> [!NOTE]
> **Не** используйте ссылки на компоненты для изменения состояния дочерних компонентов. Вместо этого используйте обычные декларативные параметры для передачи данных дочерним компонентам. Использование обычных декларативных параметров приводит к тому, что дочерние компоненты автоматически визуализируются в нужное время.

## <a name="invoke-component-methods-externally-to-update-state"></a>Внешний вызов методов компонента для изменения состояния

Blazor использует контекст синхронизации (<xref:System.Threading.SynchronizationContext>) для принудительного использования одного логического потока выполнения. [Методы жизненного цикла ](xref:blazor/lifecycle) компонента и все обратные вызовы событий, сделанные Blazor, выполняются в этом контексте синхронизации.

Контекст синхронизации Blazor Server пытается эмулировать однопоточную среду таким образом, чтобы она точно соответствовала модели WebAssembly в браузере, которая является однопоточной. В любой момент времени работа выполняется только в одном потоке, что создает впечатление единого логического потока. Две операции не могут выполняться одновременно.

Если компонент нужно изменить на основе внешнего события, такого как таймер или другие уведомления, используйте метод `InvokeAsync`, который выполняет отправку в контекст синхронизации Blazor. Например, рассмотрим *службу уведомителя*, которая может уведомлять любой компонент, ожидающий передачи данных, об измененном состоянии:

```csharp
public class NotifierService
{
    // Can be called from anywhere
    public async Task Update(string key, int value)
    {
        if (Notify != null)
        {
            await Notify.Invoke(key, value);
        }
    }

    public event Func<string, int, Task> Notify;
}
```

Зарегистрируйте службу `NotifierService` как одноэлементную:

* В Blazor WebAssembly зарегистрируйте службу в `Program.Main`:

  ```csharp
  builder.Services.AddSingleton<NotifierService>();
  ```

* В Blazor Server зарегистрируйте службу в `Startup.ConfigureServices`:

  ```csharp
  services.AddScoped<NotifierService>();
  ```

Используйте `NotifierService` для изменения компонента:

```razor
@page "/"
@inject NotifierService Notifier
@implements IDisposable

<p>Last update: @lastNotification.key = @lastNotification.value</p>

@code {
    private (string key, int value) lastNotification;

    protected override void OnInitialized()
    {
        Notifier.Notify += OnNotify;
    }

    public async Task OnNotify(string key, int value)
    {
        await InvokeAsync(() =>
        {
            lastNotification = (key, value);
            StateHasChanged();
        });
    }

    public void Dispose()
    {
        Notifier.Notify -= OnNotify;
    }
}
```

В предыдущем примере `NotifierService` вызывает метод `OnNotify` компонента вне контекста синхронизации Blazor. `InvokeAsync` используется для переключения на подходящий контекст и постановки отрисовки в очередь.

## <a name="use-key-to-control-the-preservation-of-elements-and-components"></a>Использование \@key для управления сохранением элементов и компонентов

При отрисовке списка элементов или компонентов и последующем изменении элементов или компонентов алгоритм сравнения Blazor должен решить, какие из предыдущих элементов или компонентов можно оставить и как следует сопоставить с ними объекты модели. Обычно этот процесс выполняется автоматически, и его можно игнорировать, но в некоторых случаях может потребоваться управлять данным процессом.

Рассмотрим следующий пример.

```csharp
@foreach (var person in People)
{
    <DetailsEditor Details="@person.Details" />
}

@code {
    [Parameter]
    public IEnumerable<Person> People { get; set; }
}
```

Содержимое коллекции `People` может изменяться при вставке, удалении или повторном упорядочении записей. Когда компонент отрисовывается повторно, компонент `<DetailsEditor>` может измениться на получение других значений параметра `Details`. Это может усложнить повторную отрисовку. В некоторых случаях повторная отрисовка может привести к появлению заметных различий в поведении, таких как потеря фокуса элемента.

Процесс сопоставления можно контролировать с помощью атрибута директивы [`@key`][5]. [`@key`][5] гарантирует, что алгоритм сравнения сохранит элементы или компоненты на основе значения ключа:

```csharp
@foreach (var person in People)
{
    <DetailsEditor @key="person" Details="@person.Details" />
}

@code {
    [Parameter]
    public IEnumerable<Person> People { get; set; }
}
```

При изменении коллекции `People` алгоритм сравнения сохраняет связь между экземплярами `<DetailsEditor>` и экземплярами `person`:

* Если `Person` удаляется из списка `People`, то из пользовательского интерфейса удаляется только соответствующий экземпляр `<DetailsEditor>`. Другие экземпляры остаются без изменений.
* Если в какой-либо позиции в списке вставляется `Person`, то в соответствующей позиции вставляется один новый экземпляр `<DetailsEditor>`. Другие экземпляры остаются без изменений.
* При переупорядочении записей `Person` соответствующие экземпляры `<DetailsEditor>` сохраняются и переупорядочиваются в пользовательском интерфейсе.

В некоторых сценариях использование [`@key`][5] уменьшает сложность повторной отрисовки и предотвращает потенциальные проблемы с изменяющимися элементами модели DOM с отслеживанием состояния, например положением фокуса.

> [!IMPORTANT]
> Ключи являются локальными для каждого компонента или элемента контейнера. Ключи не сравниваются глобально по всему документу.

### <a name="when-to-use-key"></a>Условия для использования \@key

Как правило, [`@key`][5] имеет смысл использовать при отрисовке списка (например, в блоке [foreach](/dotnet/csharp/language-reference/keywords/foreach-in)) и при наличии подходящего значения для определения [`@key`][5].

Можно также использовать [`@key`][5], чтобы запретить Blazor сохранять поддерево элементов или компонентов при изменении объекта:

```razor
<div @key="currentPerson">
    ... content that depends on currentPerson ...
</div>
```

Если `@currentPerson` меняется, директива атрибута [`@key`][5] принуждает Blazor отменить весь блок элемента `<div>` с потомками и перестроить поддерево в пользовательском интерфейсе с использованием новых элементов и компонентов. Это может быть полезно, если нужно гарантировать, что при изменении `@currentPerson` состояние пользовательского интерфейса не сохраняется.

### <a name="when-not-to-use-key"></a>Условия для отказа от использования \@key

Сравнение с использованием [`@key`][5] подразумевает определенное снижение производительности. Это снижение производительности незначительно, но указывать [`@key`][5] следует только в том случае, если управление правилами сохранения элементов или компонентов выгодно для приложения.

Даже если [`@key`][5] не используется, Blazor сохраняет экземпляры дочерних элементов и компонентов в максимально возможной степени. Единственным преимуществом использования [`@key`][5] является контроль над тем, *как* экземпляры модели сопоставляются с сохраненными экземплярами компонентов, вместо выбора сопоставления с помощью алгоритма сравнения.

### <a name="what-values-to-use-for-key"></a>Значения, которые следует использовать для \@key

Как правило, для [`@key`][5] имеет смысл указать значение одного из следующих видов:

* Экземпляры объектов модели (например, экземпляр `Person`, как в предыдущем примере). Это гарантирует сохранение на основе равенства ссылок на объекты.
* Уникальные идентификаторы (например, значения первичного ключа типа `int`, `string` или `Guid`).

Убедитесь, что значения, используемые для [`@key`][5], не конфликтуют. Если в одном родительском элементе обнаруживаются конфликтующие значения, Blazor выдает исключение, поскольку не может детерминированно сопоставлять старые элементы или компоненты с новыми. Используйте только уникальные значения, такие как экземпляры объекта или значения первичного ключа.

## <a name="dont-create-components-that-write-to-their-own-parameter-properties"></a>Не создавать компоненты, которые записываются в собственные свойства параметров

Параметры перезаписываются при следующих условиях.

* Содержимое дочернего компонента подготавливается к просмотру с помощью <xref:Microsoft.AspNetCore.Components.RenderFragment>.
* <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> вызывается в родительском компоненте.

Параметры сбрасываются, так как родительский компонент перерисовывается при вызове <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A>, а новые значения параметров передаются дочернему компоненту.

Рассмотрим следующий компонент `Expander`, который:

* преобразует дочернее содержимое;
* переключает отображение дочернего содержимого с помощью параметра компонента.

```razor
<div @onclick="@Toggle">
    Toggle (Expanded = @Expanded)

    @if (Expanded)
    {
        @ChildContent
    }
</div>

@code {
    [Parameter]
    public bool Expanded { get; set; }

    [Parameter]
    public RenderFragment ChildContent { get; set; }

    private void Toggle()
    {
        Expanded = !Expanded;
    }
}
```

Компонент `Expander` добавляется в родительский компонент, который может вызывать <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A>:

```razor
<Expander Expanded="true">
    <h1>Hello, world!</h1>
</Expander>

<Expander Expanded="true" />

<button @onclick="@(() => StateHasChanged())">
    Call StateHasChanged
</button>
```

Изначально компоненты `Expander` работают независимо, когда их свойства `Expanded` переключаются. Дочерние компоненты сохраняют свои состояния, как и ожидалось. Когда в родительском элементе вызывается <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A>, параметр `Expanded` первого дочернего компонента сбрасывается обратно к первоначальному значению (`true`). Значение `Expanded` второго компонента `Expander` не сбрасывается, так как во втором компоненте не отображается дочернее содержимое.

Чтобы сохранить состояние в предыдущем сценарии, используйте *закрытое поле* в компоненте `Expander`, чтобы сохранить состояние переключения.

Приведенный ниже компонент `Expander` делает следующее.

* Принимает значение параметра компонента `Expanded` из родительского элемента.
* Присваивает значение параметра компонента *закрытому полю* (`expanded`) при [событии OnInitialized](xref:blazor/lifecycle#component-initialization-methods).
* Использует закрытое поле для поддержания внутреннего состояния переключения.

```razor
<div @onclick="@Toggle">
    Toggle (Expanded = @expanded)

    @if (expanded)
    {
        @ChildContent
    }
</div>

@code {
    [Parameter]
    public bool Expanded { get; set; }

    [Parameter]
    public RenderFragment ChildContent { get; set; }

    private bool expanded;

    protected override void OnInitialized()
    {
        expanded = Expanded;
    }

    private void Toggle()
    {
        expanded = !expanded;
    }
}
```

## <a name="partial-class-support"></a>Поддержка разделяемых классов

Компоненты Razor создаются как разделяемые классы. Создавать компоненты Razor можно одним из следующих способов.

* Код C# определяется в блоке [`@code`][1] с разметкой HTML и кодом Razor в одном файле. Шаблоны Blazor определяют свои компоненты Razor с помощью этого подхода.
* Код C# помещается в файл кода программной части, определенный как разделяемый класс.

В следующем примере показан компонент `Counter` по умолчанию с блоком [`@code`][1] в приложении, созданном из шаблона Blazor. Разметка HTML, код Razor и код C# находятся в одном файле.

*Counter.razor*:

```razor
@page "/counter"

<h1>Counter</h1>

<p>Current count: @currentCount</p>

<button class="btn btn-primary" @onclick="IncrementCount">Click me</button>

@code {
    private int currentCount = 0;

    void IncrementCount()
    {
        currentCount++;
    }
}
```

Компонент `Counter` также можно создать, используя файл кода программной части с разделяемым классом:

*Counter.razor*:

```razor
@page "/counter"

<h1>Counter</h1>

<p>Current count: @currentCount</p>

<button class="btn btn-primary" @onclick="IncrementCount">Click me</button>
```

*Counter.razor.cs*:

```csharp
namespace BlazorApp.Pages
{
    public partial class Counter
    {
        private int currentCount = 0;

        void IncrementCount()
        {
            currentCount++;
        }
    }
}
```

При необходимости добавьте в файл разделяемого класса нужные пространства имен. К типичным пространствам имен, используемым компонентами Razor, относятся следующие.

```csharp
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Components;
using Microsoft.AspNetCore.Components.Authorization;
using Microsoft.AspNetCore.Components.Forms;
using Microsoft.AspNetCore.Components.Routing;
using Microsoft.AspNetCore.Components.Web;
```

## <a name="specify-a-base-class"></a>Указание базового класса

Директиву [`@inherits`][6] можно использовать для указания базового класса для компонента. В следующем примере показано, как компонент может наследовать базовый класс `BlazorRocksBase`, чтобы предоставить свойства и методы компонента. Базовый класс должен быть производным от <xref:Microsoft.AspNetCore.Components.ComponentBase>.

*Pages/BlazorRocks.razor*:

```razor
@page "/BlazorRocks"
@inherits BlazorRocksBase

<h1>@BlazorRocksText</h1>
```

*BlazorRocksBase.cs*:

```csharp
using Microsoft.AspNetCore.Components;

namespace BlazorSample
{
    public class BlazorRocksBase : ComponentBase
    {
        public string BlazorRocksText { get; set; } = 
            "Blazor rocks the browser!";
    }
}
```

## <a name="specify-an-attribute"></a>Указание атрибута

Атрибуты можно указать в компонентах Razor с помощью директивы [`@attribute`][7]. В следующем примере атрибут [`[Authorize]`](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute) применяется к классу компонентов:

```razor
@page "/"
@attribute [Authorize]
```

## <a name="import-components"></a>Импорт компонентов

Пространство имен компонента, созданного с помощью Razor, основано на следующем (в порядке приоритета).

* Назначение [`@namespace`][8] в разметке файла Razor ( *.razor*) (`@namespace BlazorSample.MyNamespace`).
* `RootNamespace` проекта в файле проекта (`<RootNamespace>BlazorSample</RootNamespace>`).
* Имя проекта, полученное из имени файла проекта (*CSPROJ*), и путь из корневого каталога проекта к компоненту. Например, платформа разрешает *{PROJECT ROOT}/Pages/Index.razor* (*BlazorSample.csproj*) в пространство имен `BlazorSample.Pages`. Компоненты соответствуют правилам привязки имен C#. Для компонента `Index` в этом примере компонентами в области действия являются все компоненты:
  * в той же папке *Pages*;
  * в корневой папке проекта, которая не задает другое пространство имен явным образом.

Компоненты, определенные в другом пространстве имен, передаются в область действия с помощью директивы [`@using`][2] Razor.

Если в папке *BlazorSample/Shared/* существует другой компонент (`NavMenu.razor`), его можно использовать в `Index.razor` с помощью инструкции [`@using`][2].

```razor
@using BlazorSample.Shared

This is the Index page.

<NavMenu></NavMenu>
```

На компоненты также можно ссылаться с помощью полных имен, для чего не требуется директива [`@using`][2]:

```razor
This is the Index page.

<BlazorSample.Shared.NavMenu></BlazorSample.Shared.NavMenu>
```

> [!NOTE]
> Квалификация `global::` не поддерживается.
>
> Импорт компонентов с инструкциями [using](/dotnet/csharp/language-reference/keywords/using-statement) с псевдонимами (например, `@using Foo = Bar`) не поддерживается.
>
> Частично определенные имена не поддерживаются. Например, добавление `@using BlazorSample` и ссылка на компонент `NavMenu` (`NavMenu.razor`) с помощью `<Shared.NavMenu></Shared.NavMenu>` не поддерживаются.

## <a name="conditional-html-element-attributes"></a>Условные атрибуты элемента HTML

Атрибуты элемента HTML условно визуализируются в зависимости от значения .NET. Если значение равно `false` или `null`, то атрибут не визуализируется. Если значение равно `true`, атрибут визуализируется в свернутом виде.

В следующем примере `IsCompleted` определяет, визуализируется ли `checked` в разметке элемента:

```razor
<input type="checkbox" checked="@IsCompleted" />

@code {
    [Parameter]
    public bool IsCompleted { get; set; }
}
```

Если `IsCompleted` имеет значение `true`, флажок визуализируется следующим образом:

```html
<input type="checkbox" checked />
```

Если `IsCompleted` имеет значение `false`, флажок визуализируется следующим образом:

```html
<input type="checkbox" />
```

Для получения дополнительной информации см. <xref:mvc/views/razor>.

> [!WARNING]
> Некоторые атрибуты HTML, такие как [, aria-pressed](https://developer.mozilla.org/docs/Web/Accessibility/ARIA/Roles/button_role#Toggle_buttons), работают неправильно, если типом .NET является `bool`. В этих случаях используйте тип `string` вместо `bool`.

## <a name="raw-html"></a>Необработанный HTML-код

Строки обычно визуализируются с помощью текстовых узлов модели DOM, что означает, что любая разметка, которую они могут содержать, игнорируется и обрабатывается как текстовый литерал. Для отрисовки необработанного HTML-кода заключите HTML-содержимое в значение `MarkupString`. Это значение анализируется как HTML или SVG и вставляется в модель DOM.

> [!WARNING]
> Отрисовка необработанного HTML-кода, созданного из любого ненадежного источника, является **угрозой безопасности**, и ее следует избегать!

В следующем примере показано использование типа `MarkupString` для добавления блока статического HTML-содержимого в визуализируемые выходные данные компонента:

```html
@((MarkupString)myMarkup)

@code {
    private string myMarkup = 
        "<p class='markup'>This is a <em>markup string</em>.</p>";
}
```

## <a name="cascading-values-and-parameters"></a>Каскадные значения и параметры

В некоторых сценариях неудобно передавать данные из компонента-предка в компонент-потомок, используя [параметры компонента](#component-parameters), особенно если имеется несколько уровней компонентов. Каскадные значения и параметры позволяют решить эту проблему, предоставляя компоненту-предку удобный способ передать значение всем его компонентам-потомкам. Каскадные значения и параметры также предоставляют подход для координации компонентов.

### <a name="theme-example"></a>Пример темы

В следующем примере из примера приложения класс `ThemeInfo` указывает сведения о теме для передачи вниз по иерархии компонентов, чтобы все кнопки в пределах заданной части приложения использовали одинаковый стиль.

*UIThemeClasses/ThemeInfo.cs*:

```csharp
public class ThemeInfo
{
    public string ButtonClass { get; set; }
}
```

Компонент-предок может предоставлять каскадное значение с помощью компонента каскадного значения. Компонент <xref:Microsoft.AspNetCore.Components.CascadingValue%601> упаковывает поддерево иерархии компонентов и предоставляет одно значение всем компонентам в этом поддереве.

Например, пример приложения указывает сведения о теме (`ThemeInfo`) в одном из макетов приложения в виде каскадного параметра для всех компонентов, составляющих основной текст макета свойства `@Body`. `ButtonClass` присваивается значение `btn-success` в компоненте макета. Любой компонент-потомок может использовать это свойство через каскадный объект `ThemeInfo`.

Компонент `CascadingValuesParametersLayout`:

```razor
@inherits LayoutComponentBase
@using BlazorSample.UIThemeClasses

<div class="container-fluid">
    <div class="row">
        <div class="col-sm-3">
            <NavMenu />
        </div>
        <div class="col-sm-9">
            <CascadingValue Value="theme">
                <div class="content px-4">
                    @Body
                </div>
            </CascadingValue>
        </div>
    </div>
</div>

@code {
    private ThemeInfo theme = new ThemeInfo { ButtonClass = "btn-success" };
}
```

Чтобы использовать каскадные значения, компоненты объявляют каскадные параметры с помощью атрибута [`[CascadingParameter]`](xref:Microsoft.AspNetCore.Components.CascadingParameterAttribute). Каскадные значения привязаны к каскадным параметрам по типу.

В примере приложения компонент `CascadingValuesParametersTheme` привязывает каскадное значение `ThemeInfo` к каскадному параметру. Параметр используется для задания класса CSS для одной из кнопок, отображаемых компонентом.

Компонент `CascadingValuesParametersTheme`:

```razor
@page "/cascadingvaluesparameterstheme"
@layout CascadingValuesParametersLayout
@using BlazorSample.UIThemeClasses

<h1>Cascading Values & Parameters</h1>

<p>Current count: @currentCount</p>

<p>
    <button class="btn" @onclick="IncrementCount">
        Increment Counter (Unthemed)
    </button>
</p>

<p>
    <button class="btn @ThemeInfo.ButtonClass" @onclick="IncrementCount">
        Increment Counter (Themed)
    </button>
</p>

@code {
    private int currentCount = 0;

    [CascadingParameter]
    protected ThemeInfo ThemeInfo { get; set; }

    private void IncrementCount()
    {
        currentCount++;
    }
}
```

Чтобы каскадировать несколько значений одного типа в одном поддереве, укажите уникальную строку <xref:Microsoft.AspNetCore.Components.CascadingValue%601.Name%2A> для каждого компонента <xref:Microsoft.AspNetCore.Components.CascadingValue%601> и его соответствующего атрибута [`[CascadingParameter]`](xref:Microsoft.AspNetCore.Components.CascadingParameterAttribute). В следующем примере два компонента <xref:Microsoft.AspNetCore.Components.CascadingValue%601> каскадируют разные экземпляры `MyCascadingType` по имени:

```razor
<CascadingValue Value=@parentCascadeParameter1 Name="CascadeParam1">
    <CascadingValue Value=@ParentCascadeParameter2 Name="CascadeParam2">
        ...
    </CascadingValue>
</CascadingValue>

@code {
    private MyCascadingType parentCascadeParameter1;

    [Parameter]
    public MyCascadingType ParentCascadeParameter2 { get; set; }

    ...
}
```

В компоненте-потомке каскадные параметры получают значения из соответствующих каскадных значений в компоненте-предке по имени:

```razor
...

@code {
    [CascadingParameter(Name = "CascadeParam1")]
    protected MyCascadingType ChildCascadeParameter1 { get; set; }
    
    [CascadingParameter(Name = "CascadeParam2")]
    protected MyCascadingType ChildCascadeParameter2 { get; set; }
}
```

### <a name="tabset-example"></a>Пример TabSet

Каскадные параметры также позволяют компонентам взаимодействовать в рамках иерархии компонентов. Например, рассмотрим следующий пример *TabSet* в примере приложения.

Пример приложения содержит интерфейс `ITab`, реализуемый вкладками:

[!code-csharp[](common/samples/3.x/BlazorWebAssemblySample/UIInterfaces/ITab.cs)]

Компонент `CascadingValuesParametersTabSet` использует компонент `TabSet`, содержащий несколько компонентов `Tab`:

```razor
<TabSet>
    <Tab Title="First tab">
        <h4>Greetings from the first tab!</h4>

        <label>
            <input type="checkbox" @bind="showThirdTab" />
            Toggle third tab
        </label>
    </Tab>
    <Tab Title="Second tab">
        <h4>The second tab says Hello World!</h4>
    </Tab>

    @if (showThirdTab)
    {
        <Tab Title="Third tab">
            <h4>Welcome to the disappearing third tab!</h4>
            <p>Toggle this tab from the first tab.</p>
        </Tab>
    }
</TabSet>
```

Дочерние компоненты `Tab` не передаются в `TabSet` в качестве параметров явным образом. Вместо этого дочерние компоненты `Tab` являются частью дочернего содержимого `TabSet`. Однако `TabSet` по-прежнему необходимо знать о каждом компоненте `Tab`, чтобы визуализировать заголовки и активную вкладку. Чтобы обеспечить такую координацию без дополнительного кода, компонент `TabSet` *может предоставить себя в качестве каскадного значения*, которое затем используется компонентами-потомками `Tab`.

Компонент `TabSet`:

[!code-razor[](common/samples/3.x/BlazorWebAssemblySample/Components/TabSet.razor)]

Компоненты-потомки `Tab` захватывают содержащий `TabSet` в качестве каскадного параметра, поэтому компоненты `Tab` добавляются в `TabSet` и координируют, какая вкладка активна.

Компонент `Tab`:

[!code-razor[](common/samples/3.x/BlazorWebAssemblySample/Components/Tab.razor)]

## <a name="razor-templates"></a>Шаблоны Razor

Фрагменты отрисовки можно определить с помощью синтаксиса шаблонов Razor. Шаблоны Razor позволяют определить фрагмент кода пользовательского интерфейса и подразумевают следующий формат.

```razor
@<{HTML tag}>...</{HTML tag}>
```

В следующем примере показано, как указать значения <xref:Microsoft.AspNetCore.Components.RenderFragment> и <xref:Microsoft.AspNetCore.Components.RenderFragment%601> и визуализировать шаблоны непосредственно в компоненте. Фрагменты отрисовки также могут передаваться в качестве аргументов в [шаблонные компоненты](xref:blazor/templated-components).

```razor
@timeTemplate

@petTemplate(new Pet { Name = "Rex" })

@code {
    private RenderFragment timeTemplate = @<p>The time is @DateTime.Now.</p>;
    private RenderFragment<Pet> petTemplate = (pet) => @<p>Pet: @pet.Name</p>;

    private class Pet
    {
        public string Name { get; set; }
    }
}
```

Визуализированные выходные данные предыдущего кода:

```html
<p>The time is 10/04/2018 01:26:52.</p>

<p>Pet: Rex</p>
```

## <a name="scalable-vector-graphics-svg-images"></a>Изображения SVG

Так как Blazor отображает визуализирует HTML-код, поддерживаемые браузером изображения, включая изображения *SVG*, поддерживаются с помощью тега `<img>`:

```html
<img alt="Example image" src="some-image.svg" />
```

Аналогичным образом изображения SVG поддерживаются в правилах CSS файла таблицы стилей (*CSS*):

```css
.my-element {
    background-image: url("some-image.svg");
}
```

Однако встроенная разметка SVG не поддерживается во всех сценариях. Если поместить тег `<svg>` непосредственно в файл компонента (*RAZOR*), то базовая отрисовка изображений доступна, но многие расширенные сценарии пока не поддерживаются. Например, теги `<use>` сейчас не учитываются, а с некоторыми тегами SVG невозможно использовать [`@bind`][10]. Дополнительные сведения см. в [справке по SVG в Blazor (dotnet/aspnetcore#18271)](https://github.com/dotnet/aspnetcore/issues/18271).

## <a name="additional-resources"></a>Дополнительные ресурсы

* <xref:security/blazor/server/threat-mitigation>. Содержит рекомендации по созданию приложений Blazor Server, которые должны соперничать в условиях нехватки ресурсов.

<!--Reference links in article-->
[1]: <xref:mvc/views/razor#code>
[2]: <xref:mvc/views/razor#using>
[3]: <xref:mvc/views/razor#attributes>
[4]: <xref:mvc/views/razor#ref>
[5]: <xref:mvc/views/razor#key>
[6]: <xref:mvc/views/razor#inherits>
[7]: <xref:mvc/views/razor#attribute>
[8]: <xref:mvc/views/razor#namespace>
[9]: <xref:mvc/views/razor#page>
[10]: <xref:mvc/views/razor#bind>
