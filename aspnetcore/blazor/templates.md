---
title: Шаблоны ASP.NET Core Blazor
author: guardrex
description: Сведения о шаблонах приложений Blazor и структуре проекта Blazor в ASP.NET Core.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 05/19/2020
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: blazor/templates
ms.openlocfilehash: f582e8201a3393b848cf3f2c21ce3a7df5554100
ms.sourcegitcommit: 6a71b560d897e13ad5b61d07afe4fcb57f8ef6dc
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 06/09/2020
ms.locfileid: "84105536"
---
# <a name="aspnet-core-blazor-templates"></a>Шаблоны ASP.NET Core Blazor

Авторы: [Дэниэл Рот (Daniel Roth)](https://github.com/danroth27) и [Люк Лэтем (Luke Latham)](https://github.com/guardrex)

Платформа Blazor предоставляет шаблоны разработки приложений для каждой из моделей размещения Blazor:

* Blazor WebAssembly (`blazorwasm`)
* Blazor Server (`blazorserver`)

Дополнительные сведения о моделях размещения Blazor см. в статье <xref:blazor/hosting-models>.

Пошаговые инструкции по созданию приложения Blazor на основе шаблона см. в статье <xref:blazor/get-started>.

Параметры шаблона доступны при передаче параметра `--help` в команду CLI [DotNet New](/dotnet/core/tools/dotnet-new):

```dotnetcli
dotnet new blazorwasm --help
dotnet new blazorserver --help
```

## <a name="blazor-project-structure"></a>Структура проекта Blazor

Приложение Blazor, создаваемое на основе шаблона Blazor, состоит из следующих файлов и папок:

* *Program.cs*: это точка входа в приложение, где настраиваются следующие элементы:

  * [узел](xref:fundamentals/host/generic-host) ASP.NET Core (Blazor Server);
  * узел WebAssembly (Blazor WebAssembly): код в этом файле используется только для приложений, созданных из шаблона Blazor WebAssembly (`blazorwasm`).
    * Компонент `App`, который является корневым компонентом приложения, указывается как элемент `app` модели DOM в методе `Add`.
    * Службы могут настраиваться с помощью метода `ConfigureServices` построителя узла (например, `builder.Services.AddSingleton<IMyDependency, MyDependency>();`).
    * Конфигурацию можно предоставить посредством построителя узла (`builder.Configuration`).

* *Startup.cs* (Blazor Server). Содержит логику для запуска приложения. В классе `Startup` определены два метода:

  * `ConfigureServices`. Настраивает службы [внедрения зависимостей](xref:fundamentals/dependency-injection) для приложения. В приложениях Blazor Server службы добавляются путем вызова метода <xref:Microsoft.Extensions.DependencyInjection.ComponentServiceCollectionExtensions.AddServerSideBlazor%2A>. В контейнер службы добавляется служба `WeatherForecastService`, которая используется примером компонента `FetchData`.
  * `Configure`. Настраивает конвейер обработки запросов для приложения.
    * <xref:Microsoft.AspNetCore.Builder.ComponentEndpointRouteBuilderExtensions.MapBlazorHub%2A> вызывается с целью настройки конечной точки для соединения в режиме реального времени с браузером. Соединение создается с помощью [SignalR](xref:signalr/introduction), платформы для добавления веб-функций реального времени в приложения.
    * [MapFallbackToPage("/_Host")](xref:Microsoft.AspNetCore.Builder.RazorPagesEndpointRouteBuilderExtensions.MapFallbackToPage*) вызывается для настройки корневой страницы приложения (*Pages/_Host.cshtml*) и обеспечения навигации.

* *wwwroot/index.html* (Blazor WebAssembly): Корневая страница приложения в формате HTML.
  * При первом запросе любой страницы приложения эта страница преобразовывается для просмотра и возвращается в ответе.
  * На странице указывается место отрисовки корневого компонента `App`. Компонент `App` (*App.razor*) указывается как элемент `app` модели DOM в методе `AddComponent` в `Startup.Configure`.
  * Загружается файл JavaScript `_framework/blazor.webassembly.js`, который выполняет следующие действия:
    * скачивает среду выполнения .NET, приложение и его зависимости;
    * инициализирует среду выполнения для запуска приложения.

* *App.razor*. Корневой компонент приложения, который настраивает маршрутизацию на стороне клиента с помощью компонента <xref:Microsoft.AspNetCore.Components.Routing.Router>. Компонент <xref:Microsoft.AspNetCore.Components.Routing.Router> перехватывает навигацию в браузере и отображает страницу, соответствующую запрошенному адресу.

* Папка *Pages*. Содержит маршрутизируемые компоненты и страницы ( *.razor*), которые входят в приложение Blazor, а также корневую страницу Razor приложения Blazor Server. Маршрут для каждой страницы указывается с помощью директивы [`@page`](xref:mvc/views/razor#page). Шаблон включает в себя следующее:
  * *_Host.cshtml* (Blazor Server). Корневая страница приложения, реализованная как страница Razor.
    * При первом запросе любой страницы приложения эта страница преобразовывается для просмотра и возвращается в ответе.
    * Загружается файл JavaScript `_framework/blazor.server.js`, который настраивает соединение SignalR в режиме реального времени между браузером и сервером.
    * На странице Host указывается место отрисовки корневого компонента `App` (*App.razor*).
  * `Counter` (*Counter.razor*). Реализует страницу счетчика.
  * `Error` (*Error.razor*, только для приложения Blazor Server). Отображается, когда в приложении происходит необработанное исключение.
  * `FetchData` (*FetchData.razor*). Реализует страницу получения данных.
  * `Index` (*Index.razor*). Реализует главную страницу приложения.

* Папка *Shared*. Содержит другие компоненты пользовательского интерфейса ( *.razor*), используемые приложением.
  * `MainLayout` (*MainLayout.razor*). Компонент макета приложения.
  * `NavMenu` (*NavMenu.razor*). Реализует боковую панель навигации. Включает в себя [компонент NavLink](xref:blazor/routing#navlink-component) (<xref:Microsoft.AspNetCore.Components.Routing.NavLink>), который служит для отрисовки навигационных ссылок на другие компоненты Razor. Компонент <xref:Microsoft.AspNetCore.Components.Routing.NavLink> автоматически указывает выбранное состояние при загрузке компонента, что помогает пользователю понять, какой компонент отображается в настоящее время.

* *_Imports.razor*. Содержит стандартные директивы Razor, включаемые в компонент приложения ( *.razor*), например директивы [`@using`](xref:mvc/views/razor#using) для пространств имен.

* Папка *Data* (Blazor Server). Содержит класс `WeatherForecast` и реализацию `WeatherForecastService`, которые предоставляют пример метеоданных для компонента `FetchData` приложения.

* *wwwroot*. Папка [корневого каталога документов](xref:fundamentals/index#web-root) для приложения, которая содержит общедоступные статические ресурсы.

* *appsettings.json* (Blazor Server). Параметры конфигурации для приложения.
