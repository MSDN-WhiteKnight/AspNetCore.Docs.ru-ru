---
title: Авторизация на основе политик в ASP.NET Core
author: rick-anderson
description: Узнайте, как создавать и использовать обработчики политик авторизации для применения требований к авторизации в приложении ASP.NET Core.
ms.author: riande
ms.custom: mvc
ms.date: 04/15/2020
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: security/authorization/policies
ms.openlocfilehash: 3b6fcef91355bf22e5aa185652d9489a44998db0
ms.sourcegitcommit: 70e5f982c218db82aa54aa8b8d96b377cfc7283f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 05/04/2020
ms.locfileid: "82777505"
---
# <a name="policy-based-authorization-in-aspnet-core"></a>Авторизация на основе политик в ASP.NET Core

::: moniker range=">= aspnetcore-3.0"

В процессе [авторизации на основе ролей](xref:security/authorization/roles) и [авторизации на основе утверждений](xref:security/authorization/claims) используются требования, обработчик требований и предварительно настроенная политика. Эти стандартные блоки поддерживают выражение оценки авторизации в коде. Результатом является расширенная, повторно используемая, тестируемая структура авторизации.

Политика авторизации состоит из одного или нескольких требований. Он регистрируется как часть конфигурации службы авторизации в `Startup.ConfigureServices` методе:

[!code-csharp[](policies/samples/3.0PoliciesAuthApp1/Startup.cs?range=31-32,39-40,42-45, 53, 58)]

В предыдущем примере создается политика "AtLeast21". Он имеет одно требование&mdash;с минимальным возрастом, которое предоставляется в качестве параметра для требования.

## <a name="iauthorizationservice"></a>IAuthorizationService 

Основная служба, которая определяет, успешно ли выполнена авторизация <xref:Microsoft.AspNetCore.Authorization.IAuthorizationService>:

[!code-csharp[](policies/samples/stubs/copy_of_IAuthorizationService.cs?highlight=24-25,48-49&name=snippet)]

В приведенном выше коде показаны два метода [IAuthorizationService](https://github.com/dotnet/AspNetCore/blob/v2.2.4/src/Security/Authorization/Core/src/IAuthorizationService.cs).

<xref:Microsoft.AspNetCore.Authorization.IAuthorizationRequirement>— это служба маркеров без методов, а также механизм отслеживания успешности авторизации.

Каждый <xref:Microsoft.AspNetCore.Authorization.IAuthorizationHandler> из них отвечает за проверку соблюдения требований:
<!--The following code is a copy/paste from 
https://github.com/dotnet/AspNetCore/blob/v2.2.4/src/Security/Authorization/Core/src/IAuthorizationHandler.cs -->

```csharp
/// <summary>
/// Classes implementing this interface are able to make a decision if authorization
/// is allowed.
/// </summary>
public interface IAuthorizationHandler
{
    /// <summary>
    /// Makes a decision if authorization is allowed.
    /// </summary>
    /// <param name="context">The authorization information.</param>
    Task HandleAsync(AuthorizationHandlerContext context);
}
```

Этот <xref:Microsoft.AspNetCore.Authorization.AuthorizationHandlerContext> класс используется обработчиком для обозначения того, выполнены ли требования.

```csharp
 context.Succeed(requirement)
```

В следующем коде показаны упрощенная реализация службы авторизации по умолчанию (и заметки с комментариями):

```csharp
public async Task<AuthorizationResult> AuthorizeAsync(ClaimsPrincipal user, 
             object resource, IEnumerable<IAuthorizationRequirement> requirements)
{
    // Create a tracking context from the authorization inputs.
    var authContext = _contextFactory.CreateContext(requirements, user, resource);

    // By default this returns an IEnumerable<IAuthorizationHandlers> from DI.
    var handlers = await _handlers.GetHandlersAsync(authContext);

    // Invoke all handlers.
    foreach (var handler in handlers)
    {
        await handler.HandleAsync(authContext);
    }

    // Check the context, by default success is when all requirements have been met.
    return _evaluator.Evaluate(authContext);
}
```

В следующем коде показана типичная `ConfigureServices`:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Add all of your handlers to DI.
    services.AddSingleton<IAuthorizationHandler, MyHandler1>();
    // MyHandler2, ...

    services.AddSingleton<IAuthorizationHandler, MyHandlerN>();

    // Configure your policies
    services.AddAuthorization(options =>
          options.AddPolicy("Something",
          policy => policy.RequireClaim("Permission", "CanViewPage", "CanViewAnything")));


    services.AddControllersWithViews();
    services.AddRazorPages();
}
```

Используйте <xref:Microsoft.AspNetCore.Authorization.IAuthorizationService> или `[Authorize(Policy = "Something")]` для авторизации.

## <a name="applying-policies-to-mvc-controllers"></a>Применение политик к контроллерам MVC

Если вы используете Razor страницы, см. раздел [применение политик Razor к страницам](#applying-policies-to-razor-pages) этого документа.

Политики применяются к контроллерам с помощью `[Authorize]` атрибута с именем политики. Например:

[!code-csharp[](policies/samples/PoliciesAuthApp1/Controllers/AlcoholPurchaseController.cs?name=snippet_AlcoholPurchaseControllerClass&highlight=4)]

## <a name="applying-policies-to-razor-pages"></a>Применение политик к Razor страницам

Политики применяются к Razor страницам с помощью `[Authorize]` атрибута с именем политики. Например:

[!code-csharp[](policies/samples/PoliciesAuthApp2/Pages/AlcoholPurchase.cshtml.cs?name=snippet_AlcoholPurchaseModelClass&highlight=4)]

Политики также можно применять к Razor страницам с помощью соглашения об [авторизации](xref:security/authorization/razor-pages-authorization).

## <a name="requirements"></a>Требования

Требование авторизации — это коллекция параметров данных, которую политика может использовать для вычисления текущего субъекта-пользователя. В нашей политике "AtLeast21" требование — это единственный параметр&mdash;с минимальным возрастом. Требование реализует [иаусоризатионрекуиремент](/dotnet/api/microsoft.aspnetcore.authorization.iauthorizationrequirement), который является пустым интерфейсом маркера. Требование к параметризованному минимальному возрасту можно реализовать следующим образом:

[!code-csharp[](policies/samples/PoliciesAuthApp1/Services/Requirements/MinimumAgeRequirement.cs?name=snippet_MinimumAgeRequirementClass)]

Если политика авторизации содержит несколько требований к авторизации, все требования должны пройти, чтобы оценка политики прошла успешно. Иными словами, несколько требований к авторизации, добавленных в одну политику авторизации, обрабатываются **и** на основе.

> [!NOTE]
> Требование не обязательно должно иметь данные или свойства.

<a name="security-authorization-policies-based-authorization-handler"></a>

## <a name="authorization-handlers"></a>Обработчики авторизации

Обработчик авторизации отвечает за оценку свойств требования. Обработчик авторизации оценивает требования к предоставленному [аусоризатионхандлерконтекст](/dotnet/api/microsoft.aspnetcore.authorization.authorizationhandlercontext) , чтобы определить, разрешен ли доступ.

Требование может иметь [несколько обработчиков](#security-authorization-policies-based-multiple-handlers). Обработчик может наследовать [AuthorizationHandler\<трекуиремент>](/dotnet/api/microsoft.aspnetcore.authorization.authorizationhandler-1), где `TRequirement` является требованием для обработки. Кроме того, обработчик может реализовать [иаусоризатионхандлер](/dotnet/api/microsoft.aspnetcore.authorization.iauthorizationhandler) для обработки более чем одного типа требований.

### <a name="use-a-handler-for-one-requirement"></a>Использование обработчика для одного требования

<a name="security-authorization-handler-example"></a>

Ниже приведен пример связи «один к одному», в которой обработчик минимального возраста использует одно требование:

[!code-csharp[](policies/samples/PoliciesAuthApp1/Services/Handlers/MinimumAgeHandler.cs?name=snippet_MinimumAgeHandlerClass)]

Приведенный выше код определяет, имеет ли текущий участник пользователя дату утверждения о рождении, выданного известным и надежным издателем. Авторизация не может произойти, если утверждение отсутствует, в этом случае возвращается завершенная задача. При наличии утверждения возраст пользователя вычисляется. Если пользователь соответствует минимальному возрасту, заданному требованием, авторизация считается успешной. Когда авторизация прошла `context.Succeed` успешно, вызывается с удовлетворенным требованием в качестве единственного параметра.

### <a name="use-a-handler-for-multiple-requirements"></a>Использование обработчика для нескольких требований

Ниже приведен пример связи «один ко многим», в которой обработчик разрешений может управлять тремя различными типами требований:

[!code-csharp[](policies/samples/PoliciesAuthApp1/Services/Handlers/PermissionHandler.cs?name=snippet_PermissionHandlerClass)]

Предыдущий код проходит по [пендингрекуирементс](/dotnet/api/microsoft.aspnetcore.authorization.authorizationhandlercontext.pendingrequirements#Microsoft_AspNetCore_Authorization_AuthorizationHandlerContext_PendingRequirements)&mdash;свойству, содержащему требования, не помеченные как неудачные. Для получения `ReadPermission` требования пользователь должен быть владельцем или спонсором для доступа к запрошенному ресурсу. В случае требования `EditPermission` или `DeletePermission` ему необходимо быть владельцем для доступа к запрошенному ресурсу.

<a name="security-authorization-policies-based-handler-registration"></a>

### <a name="handler-registration"></a>Регистрация обработчика

Обработчики регистрируются в коллекции служб во время настройки. Например:

[!code-csharp[](policies/samples/3.0PoliciesAuthApp1/Startup.cs?range=31-32,39-40,42-45, 53-55, 58)]

Предыдущий код регистрируется `MinimumAgeHandler` как одноэлементный, вызывая `services.AddSingleton<IAuthorizationHandler, MinimumAgeHandler>();`. Обработчики можно регистрировать с помощью любого встроенного [времени существования службы](xref:fundamentals/dependency-injection#service-lifetimes).

## <a name="what-should-a-handler-return"></a>Что должен возвращать обработчик?

Обратите внимание `Handle` , что метод в [примере обработчика](#security-authorization-handler-example) не возвращает значения. Как отображается состояние «успешно» или «сбой»?

* Обработчик указывает на успешное выполнение `context.Succeed(IAuthorizationRequirement requirement)`путем вызова, передавая требование, которое прошло проверку.

* Обработчику обычно не требуется обработка ошибок, так как другие обработчики для такого же требования могут быть выполнены удачно.

* Чтобы гарантировать сбой, даже если другие обработчики требований выполняются успешно `context.Fail`, вызовите.

Если обработчик вызывает `context.Succeed` или `context.Fail`, все остальные обработчики все еще вызываются. Это позволяет создавать побочные эффекты, например ведение журнала, которое выполняется, даже если другой обработчик прошел проверку или не прошел требование. Если задано `false`значение, свойство [инвокехандлерсафтерфаилуре](/dotnet/api/microsoft.aspnetcore.authorization.authorizationoptions.invokehandlersafterfailure#Microsoft_AspNetCore_Authorization_AuthorizationOptions_InvokeHandlersAfterFailure) (доступно в ASP.NET Core 1,1 и более поздних версиях) сокращено при `context.Fail` вызове обработчиков. `InvokeHandlersAfterFailure`по умолчанию `true`имеет значение, в этом случае вызываются все обработчики.

> [!NOTE]
> Обработчики авторизации вызываются даже в случае сбоя проверки подлинности.

<a name="security-authorization-policies-based-multiple-handlers"></a>

## <a name="why-would-i-want-multiple-handlers-for-a-requirement"></a>Зачем требуется несколько обработчиков для требования?

В случаях, когда требуется, чтобы оценка была в **или** на основе, реализуйте несколько обработчиков для одного требования. Например, в корпорации Майкрософт есть дверцы, которые открываются только с карточками ключей. Если вы оставите свой ключ дома, секретарь выводит временную наклейку и открывает дверцу. В этом сценарии у вас будет одно требование, *буилдинжентри*, но несколько обработчиков, каждый из которых проанализировать одно требование.

*BuildingEntryRequirement.cs*

[!code-csharp[](policies/samples/PoliciesAuthApp1/Services/Requirements/BuildingEntryRequirement.cs?name=snippet_BuildingEntryRequirementClass)]

*BadgeEntryHandler.cs*

[!code-csharp[](policies/samples/PoliciesAuthApp1/Services/Handlers/BadgeEntryHandler.cs?name=snippet_BadgeEntryHandlerClass)]

*TemporaryStickerHandler.cs*

[!code-csharp[](policies/samples/PoliciesAuthApp1/Services/Handlers/TemporaryStickerHandler.cs?name=snippet_TemporaryStickerHandlerClass)]

Убедитесь, что оба обработчика [зарегистрированы](xref:security/authorization/policies#security-authorization-policies-based-handler-registration). Если любой из обработчиков будет `BuildingEntryRequirement`выполнен, когда политика оценивает, то Вычисление политики будет завершено.

## <a name="using-a-func-to-fulfill-a-policy"></a>Использование функции func для выполнения политики

Могут возникнуть ситуации, в которых выполнение политики может быть простым для выражения в коде. `Func<AuthorizationHandlerContext, bool>` При настройке политики с помощью построителя `RequireAssertion` политик можно указать.

Например, Предыдущая `BadgeEntryHandler` возможность может быть переписана следующим образом:

[!code-csharp[](policies/samples/3.0PoliciesAuthApp1/Startup.cs?range=42-43,47-53)]

## <a name="accessing-mvc-request-context-in-handlers"></a>Доступ к контексту запроса MVC в обработчиках

`HandleRequirementAsync` Метод, реализуемый в обработчике авторизации, имеет два параметра: `AuthorizationHandlerContext` и `TRequirement` обрабатываемый объект. Платформы, такие как MVC или Жаббр, могут добавлять любой объект в `Resource` свойство в `AuthorizationHandlerContext` для передачи дополнительной информации.

Например, MVC передает экземпляр [аусоризатионфилтерконтекст](/dotnet/api/?term=AuthorizationFilterContext) в `Resource` свойство. Это свойство предоставляет доступ к `HttpContext`, `RouteData`и всем остальным, предоставляемым MVC Razor и страницами.

Использование `Resource` свойства зависит от платформы. Использование сведений в `Resource` свойстве ограничивает политики авторизации определенными платформами. Необходимо привести `Resource` свойство с помощью `is` ключевого слова, а затем подтвердить успешное приведение, чтобы гарантировать, что код не будет `InvalidCastException` завершаться с ошибкой при выполнении в других платформах:

```csharp
// Requires the following import:
//     using Microsoft.AspNetCore.Mvc.Filters;
if (context.Resource is AuthorizationFilterContext mvcContext)
{
    // Examine MVC-specific things like routing data.
}
```

::: moniker-end


::: moniker range="< aspnetcore-3.0"

В процессе [авторизации на основе ролей](xref:security/authorization/roles) и [авторизации на основе утверждений](xref:security/authorization/claims) используются требования, обработчик требований и предварительно настроенная политика. Эти стандартные блоки поддерживают выражение оценки авторизации в коде. Результатом является расширенная, повторно используемая, тестируемая структура авторизации.

Политика авторизации состоит из одного или нескольких требований. Он регистрируется как часть конфигурации службы авторизации в `Startup.ConfigureServices` методе:

[!code-csharp[](policies/samples/PoliciesAuthApp1/Startup.cs?range=32-33,48-53,61,66)]

В предыдущем примере создается политика "AtLeast21". Он имеет одно требование&mdash;с минимальным возрастом, которое предоставляется в качестве параметра для требования.

## <a name="iauthorizationservice"></a>IAuthorizationService 

Основная служба, которая определяет, успешно ли выполнена авторизация <xref:Microsoft.AspNetCore.Authorization.IAuthorizationService>:

[!code-csharp[](policies/samples/stubs/copy_of_IAuthorizationService.cs?highlight=24-25,48-49&name=snippet)]

В приведенном выше коде показаны два метода [IAuthorizationService](https://github.com/dotnet/AspNetCore/blob/v2.2.4/src/Security/Authorization/Core/src/IAuthorizationService.cs).

<xref:Microsoft.AspNetCore.Authorization.IAuthorizationRequirement>— это служба маркеров без методов, а также механизм отслеживания успешности авторизации.

Каждый <xref:Microsoft.AspNetCore.Authorization.IAuthorizationHandler> из них отвечает за проверку соблюдения требований:
<!--The following code is a copy/paste from 
https://github.com/dotnet/AspNetCore/blob/v2.2.4/src/Security/Authorization/Core/src/IAuthorizationHandler.cs -->

```csharp
/// <summary>
/// Classes implementing this interface are able to make a decision if authorization
/// is allowed.
/// </summary>
public interface IAuthorizationHandler
{
    /// <summary>
    /// Makes a decision if authorization is allowed.
    /// </summary>
    /// <param name="context">The authorization information.</param>
    Task HandleAsync(AuthorizationHandlerContext context);
}
```

Этот <xref:Microsoft.AspNetCore.Authorization.AuthorizationHandlerContext> класс используется обработчиком для обозначения того, выполнены ли требования.

```csharp
 context.Succeed(requirement)
```

В следующем коде показаны упрощенная реализация службы авторизации по умолчанию (и заметки с комментариями):

```csharp
public async Task<AuthorizationResult> AuthorizeAsync(ClaimsPrincipal user, 
             object resource, IEnumerable<IAuthorizationRequirement> requirements)
{
    // Create a tracking context from the authorization inputs.
    var authContext = _contextFactory.CreateContext(requirements, user, resource);

    // By default this returns an IEnumerable<IAuthorizationHandlers> from DI.
    var handlers = await _handlers.GetHandlersAsync(authContext);

    // Invoke all handlers.
    foreach (var handler in handlers)
    {
        await handler.HandleAsync(authContext);
    }

    // Check the context, by default success is when all requirements have been met.
    return _evaluator.Evaluate(authContext);
}
```

В следующем коде показана типичная `ConfigureServices`:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Add all of your handlers to DI.
    services.AddSingleton<IAuthorizationHandler, MyHandler1>();
    // MyHandler2, ...

    services.AddSingleton<IAuthorizationHandler, MyHandlerN>();

    // Configure your policies
    services.AddAuthorization(options =>
          options.AddPolicy("Something",
          policy => policy.RequireClaim("Permission", "CanViewPage", "CanViewAnything")));


    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_2);
}
```

Используйте <xref:Microsoft.AspNetCore.Authorization.IAuthorizationService> или `[Authorize(Policy = "Something")]` для авторизации.

## <a name="applying-policies-to-mvc-controllers"></a>Применение политик к контроллерам MVC

Если вы используете Razor страницы, см. раздел [применение политик Razor к страницам](#applying-policies-to-razor-pages) этого документа.

Политики применяются к контроллерам с помощью `[Authorize]` атрибута с именем политики. Например:

[!code-csharp[](policies/samples/PoliciesAuthApp1/Controllers/AlcoholPurchaseController.cs?name=snippet_AlcoholPurchaseControllerClass&highlight=4)]

## <a name="applying-policies-to-razor-pages"></a>Применение политик к Razor страницам

Политики применяются к Razor страницам с помощью `[Authorize]` атрибута с именем политики. Например:

[!code-csharp[](policies/samples/PoliciesAuthApp2/Pages/AlcoholPurchase.cshtml.cs?name=snippet_AlcoholPurchaseModelClass&highlight=4)]

Политики также можно применять к Razor страницам с помощью соглашения об [авторизации](xref:security/authorization/razor-pages-authorization).

## <a name="requirements"></a>Требования

Требование авторизации — это коллекция параметров данных, которую политика может использовать для вычисления текущего субъекта-пользователя. В нашей политике "AtLeast21" требование — это единственный параметр&mdash;с минимальным возрастом. Требование реализует [иаусоризатионрекуиремент](/dotnet/api/microsoft.aspnetcore.authorization.iauthorizationrequirement), который является пустым интерфейсом маркера. Требование к параметризованному минимальному возрасту можно реализовать следующим образом:

[!code-csharp[](policies/samples/PoliciesAuthApp1/Services/Requirements/MinimumAgeRequirement.cs?name=snippet_MinimumAgeRequirementClass)]

Если политика авторизации содержит несколько требований к авторизации, все требования должны пройти, чтобы оценка политики прошла успешно. Иными словами, несколько требований к авторизации, добавленных в одну политику авторизации, обрабатываются **и** на основе.

> [!NOTE]
> Требование не обязательно должно иметь данные или свойства.

<a name="security-authorization-policies-based-authorization-handler"></a>

## <a name="authorization-handlers"></a>Обработчики авторизации

Обработчик авторизации отвечает за оценку свойств требования. Обработчик авторизации оценивает требования к предоставленному [аусоризатионхандлерконтекст](/dotnet/api/microsoft.aspnetcore.authorization.authorizationhandlercontext) , чтобы определить, разрешен ли доступ.

Требование может иметь [несколько обработчиков](#security-authorization-policies-based-multiple-handlers). Обработчик может наследовать [AuthorizationHandler\<трекуиремент>](/dotnet/api/microsoft.aspnetcore.authorization.authorizationhandler-1), где `TRequirement` является требованием для обработки. Кроме того, обработчик может реализовать [иаусоризатионхандлер](/dotnet/api/microsoft.aspnetcore.authorization.iauthorizationhandler) для обработки более чем одного типа требований.

### <a name="use-a-handler-for-one-requirement"></a>Использование обработчика для одного требования

<a name="security-authorization-handler-example"></a>

Ниже приведен пример связи «один к одному», в которой обработчик минимального возраста использует одно требование:

[!code-csharp[](policies/samples/PoliciesAuthApp1/Services/Handlers/MinimumAgeHandler.cs?name=snippet_MinimumAgeHandlerClass)]

Приведенный выше код определяет, имеет ли текущий участник пользователя дату утверждения о рождении, выданного известным и надежным издателем. Авторизация не может произойти, если утверждение отсутствует, в этом случае возвращается завершенная задача. При наличии утверждения возраст пользователя вычисляется. Если пользователь соответствует минимальному возрасту, заданному требованием, авторизация считается успешной. Когда авторизация прошла `context.Succeed` успешно, вызывается с удовлетворенным требованием в качестве единственного параметра.

### <a name="use-a-handler-for-multiple-requirements"></a>Использование обработчика для нескольких требований

Ниже приведен пример связи «один ко многим», в которой обработчик разрешений может управлять тремя различными типами требований:

[!code-csharp[](policies/samples/PoliciesAuthApp1/Services/Handlers/PermissionHandler.cs?name=snippet_PermissionHandlerClass)]

Предыдущий код проходит по [пендингрекуирементс](/dotnet/api/microsoft.aspnetcore.authorization.authorizationhandlercontext.pendingrequirements#Microsoft_AspNetCore_Authorization_AuthorizationHandlerContext_PendingRequirements)&mdash;свойству, содержащему требования, не помеченные как неудачные. Для получения `ReadPermission` требования пользователь должен быть владельцем или спонсором для доступа к запрошенному ресурсу. В случае требования `EditPermission` или `DeletePermission` ему необходимо быть владельцем для доступа к запрошенному ресурсу.

<a name="security-authorization-policies-based-handler-registration"></a>

### <a name="handler-registration"></a>Регистрация обработчика

Обработчики регистрируются в коллекции служб во время настройки. Например:

[!code-csharp[](policies/samples/PoliciesAuthApp1/Startup.cs?range=32-33,48-53,61,62-63,66)]

Предыдущий код регистрируется `MinimumAgeHandler` как одноэлементный, вызывая `services.AddSingleton<IAuthorizationHandler, MinimumAgeHandler>();`. Обработчики можно регистрировать с помощью любого встроенного [времени существования службы](xref:fundamentals/dependency-injection#service-lifetimes).

## <a name="what-should-a-handler-return"></a>Что должен возвращать обработчик?

Обратите внимание `Handle` , что метод в [примере обработчика](#security-authorization-handler-example) не возвращает значения. Как отображается состояние «успешно» или «сбой»?

* Обработчик указывает на успешное выполнение `context.Succeed(IAuthorizationRequirement requirement)`путем вызова, передавая требование, которое прошло проверку.

* Обработчику обычно не требуется обработка ошибок, так как другие обработчики для такого же требования могут быть выполнены удачно.

* Чтобы гарантировать сбой, даже если другие обработчики требований выполняются успешно `context.Fail`, вызовите.

Если обработчик вызывает `context.Succeed` или `context.Fail`, все остальные обработчики все еще вызываются. Это позволяет создавать побочные эффекты, например ведение журнала, которое выполняется, даже если другой обработчик прошел проверку или не прошел требование. Если задано `false`значение, свойство [инвокехандлерсафтерфаилуре](/dotnet/api/microsoft.aspnetcore.authorization.authorizationoptions.invokehandlersafterfailure#Microsoft_AspNetCore_Authorization_AuthorizationOptions_InvokeHandlersAfterFailure) (доступно в ASP.NET Core 1,1 и более поздних версиях) сокращено при `context.Fail` вызове обработчиков. `InvokeHandlersAfterFailure`по умолчанию `true`имеет значение, в этом случае вызываются все обработчики.

> [!NOTE]
> Обработчики авторизации вызываются даже в случае сбоя проверки подлинности.

<a name="security-authorization-policies-based-multiple-handlers"></a>

## <a name="why-would-i-want-multiple-handlers-for-a-requirement"></a>Зачем требуется несколько обработчиков для требования?

В случаях, когда требуется, чтобы оценка была в **или** на основе, реализуйте несколько обработчиков для одного требования. Например, в корпорации Майкрософт есть дверцы, которые открываются только с карточками ключей. Если вы оставите свой ключ дома, секретарь выводит временную наклейку и открывает дверцу. В этом сценарии у вас будет одно требование, *буилдинжентри*, но несколько обработчиков, каждый из которых проанализировать одно требование.

*BuildingEntryRequirement.cs*

[!code-csharp[](policies/samples/PoliciesAuthApp1/Services/Requirements/BuildingEntryRequirement.cs?name=snippet_BuildingEntryRequirementClass)]

*BadgeEntryHandler.cs*

[!code-csharp[](policies/samples/PoliciesAuthApp1/Services/Handlers/BadgeEntryHandler.cs?name=snippet_BadgeEntryHandlerClass)]

*TemporaryStickerHandler.cs*

[!code-csharp[](policies/samples/PoliciesAuthApp1/Services/Handlers/TemporaryStickerHandler.cs?name=snippet_TemporaryStickerHandlerClass)]

Убедитесь, что оба обработчика [зарегистрированы](xref:security/authorization/policies#security-authorization-policies-based-handler-registration). Если любой из обработчиков будет `BuildingEntryRequirement`выполнен, когда политика оценивает, то Вычисление политики будет завершено.

## <a name="using-a-func-to-fulfill-a-policy"></a>Использование функции func для выполнения политики

Могут возникнуть ситуации, в которых выполнение политики может быть простым для выражения в коде. `Func<AuthorizationHandlerContext, bool>` При настройке политики с помощью построителя `RequireAssertion` политик можно указать.

Например, Предыдущая `BadgeEntryHandler` возможность может быть переписана следующим образом:

[!code-csharp[](policies/samples/PoliciesAuthApp1/Startup.cs?range=50-51,55-61)]

## <a name="accessing-mvc-request-context-in-handlers"></a>Доступ к контексту запроса MVC в обработчиках

`HandleRequirementAsync` Метод, реализуемый в обработчике авторизации, имеет два параметра: `AuthorizationHandlerContext` и `TRequirement` обрабатываемый объект. Платформы, такие как MVC или SignalR , могут добавлять любой объект в `Resource` свойство в `AuthorizationHandlerContext` для передачи дополнительной информации.

При использовании маршрутизации конечных точек авторизация обычно обрабатывается по промежуточного слоя авторизации. В этом случае `Resource` свойство является экземпляром <xref:Microsoft.AspNetCore.Http.Endpoint>. Конечную точку можно использовать для проверки базового ресурса, к которому выполняется маршрутизация. Например:

```csharp
if (context.Resource is Endpoint endpoint)
{
   var actionDescriptor = endpoint.Metadata.GetMetadata<ControllerActionDescriptor>();
   ...
}
```

Если `Resource` используется традиционная маршрутизация или когда Авторизация происходит как часть фильтра авторизации MVC, значение является <xref:Microsoft.AspNetCore.Mvc.Filters.AuthorizationFilterContext> экземпляром. Это свойство предоставляет доступ к `HttpContext`, `RouteData`и всем остальным, предоставляемым MVC Razor и страницами.

Использование `Resource` свойства зависит от платформы. Использование сведений в `Resource` свойстве ограничивает политики авторизации определенными платформами. Необходимо привести `Resource` свойство с помощью `is` ключевого слова, а затем подтвердить успешное приведение, чтобы гарантировать, что код не будет `InvalidCastException` завершаться с ошибкой при выполнении в других платформах:

```csharp
// Requires the following import:
//     using Microsoft.AspNetCore.Mvc.Filters;
if (context.Resource is AuthorizationFilterContext mvcContext)
{
    // Examine MVC-specific things like routing data.
}
```

::: moniker-end
