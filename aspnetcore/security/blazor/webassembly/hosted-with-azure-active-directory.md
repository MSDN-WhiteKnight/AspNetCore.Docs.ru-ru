---
title: Обеспечение безопасности Blazor размещенного в ASP.NET Core приложения сборки Azure Active Directory
author: guardrex
description: ''
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
uid: security/blazor/webassembly/hosted-with-azure-active-directory
ms.openlocfilehash: 5e403d3503fb534d5c3db71a7d6831953c60ac28
ms.sourcegitcommit: b0062f29cba2e5c21b95cf89eaf435ba830d11a3
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 06/15/2020
ms.locfileid: "84776531"
---
# <a name="secure-an-aspnet-core-blazor-webassembly-hosted-app-with-azure-active-directory"></a>Обеспечение безопасности Blazor размещенного в ASP.NET Core приложения сборки Azure Active Directory

[Хавьер Калварро Воронков](https://github.com/javiercn) и [Люк ЛаСаМ](https://github.com/guardrex)

В этой статье описывается создание [ Blazor размещенного приложения сборки](xref:blazor/hosting-models#blazor-webassembly) , которое использует [Azure Active Directory (AAD)](https://azure.microsoft.com/services/active-directory/) для проверки подлинности.

## <a name="register-apps-in-aad-and-create-solution"></a>Регистрация приложений в AAD и создание решения

### <a name="create-a-tenant"></a>Создание клиента

Следуйте указаниям в [кратком руководстве по настройке клиента](/azure/active-directory/develop/quickstart-create-new-tenant) для создания клиента в AAD.

### <a name="register-a-server-api-app"></a>Регистрация приложения API сервера

Следуйте указаниям в [кратком руководстве: регистрация приложения с помощью платформы удостоверений Майкрософт](/azure/active-directory/develop/quickstart-register-app) и последующих разделов Azure AAD, чтобы зарегистрировать приложение AAD для *приложения API сервера* , а затем выполнить следующие действия.

1. В **Azure Active Directory**  >  **Регистрация приложений**выберите пункт **Новая регистрация**.
1. Укажите **имя** приложения (например, ** Blazor сервер AAD**).
1. Выберите **Поддерживаемые типы учетных записей**. Для этого интерфейса можно выбрать **учетные записи только в этом каталоге Организации** (один клиент).
1. В этом сценарии *приложению API сервера* не требуется **универсальный код ресурса (URI) перенаправления** , поэтому оставьте в раскрывающемся списке значение **Web** и не вводите URI перенаправления.
1. Отключите **разрешения**  >  **предоставление согласия администратора для OpenID Connect и offline_access разрешений** .
1. Выберите **Зарегистрировать**.

Запишите следующие сведения:

* *Приложение API сервера* Идентификатор приложения (идентификатор клиента) (например, `11111111-1111-1111-1111-111111111111` )
* Идентификатор каталога (идентификатор клиента) (например, `222222222-2222-2222-2222-222222222222` )
* Домен клиента AAD (например, `contoso.onmicrosoft.com` ). домен доступен в качестве **домена издателя** в колонке **фирменной символики** портал Azure для зарегистрированного приложения.

В окне **разрешения API**удалите разрешение **Microsoft Graph**  >  **пользователь. чтение** , так как приложению не требуется доступ для входа или профиля пользователя.

В **предоставление API**:

1. Нажмите **Добавить группу**.
1. Выберите **Сохранить и продолжить**.
1. Укажите **имя области** (например, `API.Access` ).
1. Укажите **Отображаемое имя согласия администратора** (например, `Access API` ).
1. Введите **Описание согласия администратора** (например, `Allows the app to access server app API endpoints.` ).
1. Убедитесь, что для **состояния** задано значение **включено**.
1. Выберите **Добавить область**.

Запишите следующие сведения:

* URI идентификатора приложения (например,, `https://contoso.onmicrosoft.com/11111111-1111-1111-1111-111111111111` `api://11111111-1111-1111-1111-111111111111` или предоставленное вами пользовательское значение)
* Область по умолчанию (например, `API.Access` )

### <a name="register-a-client-app"></a>Регистрация клиентского приложения

Следуйте указаниям в [кратком руководстве: регистрация приложения с помощью платформы удостоверений Майкрософт](/azure/active-directory/develop/quickstart-register-app) и последующих разделов Azure AAD, чтобы зарегистрировать приложение AAD для *клиентского приложения* , а затем выполнить следующие действия.

1. В **Azure Active Directory**  >  **Регистрация приложений**выберите пункт **Новая регистрация**.
1. Укажите **имя** приложения (например, ** Blazor AAD клиента**).
1. Выберите **Поддерживаемые типы учетных записей**. Для этого интерфейса можно выбрать **учетные записи только в этом каталоге Организации** (один клиент).
1. Оставьте в раскрывающемся списке **URI перенаправления** значение **веб-сайт** и укажите следующий URI перенаправления: `https://localhost:{PORT}/authentication/login-callback` . Порт по умолчанию для приложения, работающего на Kestrel, — 5001. Если приложение выполняется на другом порту Kestrel, используйте порт приложения. Для IIS Express созданный случайным образом порт для приложения можно найти в свойствах серверного приложения на панели **отладки** . Так как на этом этапе приложение не существует и порт IIS Express неизвестен, вернитесь к этому шагу после создания приложения и обновите URI перенаправления. Замечание появится в разделе [Создание приложения](#create-the-app) , чтобы напомнить IIS Express пользователям об обновлении URI перенаправления.
1. Отключите **разрешения**  >  **предоставление согласия администратора для OpenID Connect и offline_access разрешений** .
1. Выберите **Зарегистрировать**.

Запишите идентификатор приложения *клиентского приложения* (идентификатор клиента) (например, `33333333-3333-3333-3333-333333333333` ).

В конфигурации платформы **проверки подлинности**  >  **Platform configurations**  >  **веб-сайт**:

1. Убедитесь, что **URI перенаправления** `https://localhost:{PORT}/authentication/login-callback` имеется.
1. Для **неявного предоставления**установите флажки для **маркеров доступа** и **маркеров идентификации**.
1. Остальные значения по умолчанию для приложения приемлемы для этого интерфейса.
1. Нажмите кнопку **Сохранить**.

В **разрешениях API**:

1. Убедитесь, что приложение имеет **Microsoft Graph**  >  **пользователь. чтение** .
1. Выберите **Добавить разрешение** , а затем — **Мои API**.
1. Выберите *приложение API сервера* из столбца **имя** (например, ** Blazor Server AAD**).
1. Откройте список **API** .
1. Разрешение доступа к API (например, `API.Access` ).
1. Выберите **Добавить разрешения**.
1. Нажмите кнопку **предоставить содержимое администратора для {имя клиента}** . Выберите **Да** для подтверждения.

### <a name="create-the-app"></a>Создание приложения

Замените заполнители в следующей команде на записанные ранее сведения и выполните команду в командной оболочке:

```dotnetcli
dotnet new blazorwasm -au SingleOrg --api-client-id "{SERVER API APP CLIENT ID}" --app-id-uri "{SERVER API APP ID URI}" --client-id "{CLIENT APP CLIENT ID}" --default-scope "{DEFAULT SCOPE}" --domain "{TENANT DOMAIN}" -ho --tenant-id "{TENANT ID}"
```

Чтобы указать расположение выходных данных, которое создает папку проекта, если она не существует, включите параметр OUTPUT в команду с путем (например, `-o BlazorSample` ). Имя папки также станет частью имени проекта.

> [!NOTE]
> Передайте универсальный код ресурса (URI) идентификатора приложения в `app-id-uri` параметр, но обратите внимание, что в клиентском приложении может потребоваться изменение конфигурации, которое описано в разделе [области действия маркера доступа](#access-token-scopes) .

> [!NOTE]
> В портал Azure параметры платформы **проверки подлинности**для веб- *клиента клиентского приложения*  >  **Platform configurations**  >  **Web**  >  **Redirect URI** настроены для порта 5001 для приложений, работающих на сервере Kestrel с параметрами по умолчанию.
>
> Если *клиентское приложение* выполняется на случайном IIS Express порту, порт для приложения можно найти в свойствах *серверного приложения* на панели **отладки** .
>
> Если порт не был настроен ранее с помощью известного порта *клиентского приложения* , вернитесь к регистрации *клиентского приложения* в портал Azure и обновите URI перенаправления с правильным портом.

## <a name="server-app-configuration"></a>Конфигурация серверного приложения

*Этот раздел относится к **серверному** приложению решения.*

### <a name="authentication-package"></a>Пакет проверки подлинности

Поддержка проверки подлинности и авторизации вызовов ASP.NET Core веб-API предоставляется пакетом [Microsoft. AspNetCore. Authentication. AzureAD. UI](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.AzureAD.UI/) :

```xml
<PackageReference Include="Microsoft.AspNetCore.Authentication.AzureAD.UI" 
  Version="3.1.4" />
```

### <a name="authentication-service-support"></a>Поддержка службы проверки подлинности

Метод настраивает `AddAuthentication` службы проверки подлинности в приложении и настраивает обработчик носителя JWT в качестве метода проверки подлинности по умолчанию. <xref:Microsoft.AspNetCore.Authentication.AzureADAuthenticationBuilderExtensions.AddAzureADBearer%2A>Метод настраивает определенные параметры в обработчике носителя JWT, который требуется для проверки маркеров, созданных Azure Active Directory:

```csharp
services.AddAuthentication(AzureADDefaults.BearerAuthenticationScheme)
    .AddAzureADBearer(options => Configuration.Bind("AzureAd", options));
```

<xref:Microsoft.AspNetCore.Builder.AuthAppBuilderExtensions.UseAuthentication%2A>и <xref:Microsoft.AspNetCore.Builder.AuthorizationAppBuilderExtensions.UseAuthorization%2A> Убедитесь, что:

* Приложение пытается проанализировать и проверить маркеры в входящих запросах.
* Любой запрос, пытающийся получить доступ к защищенному ресурсу без соответствующих учетных данных, завершится ошибкой.

```csharp
app.UseAuthentication();
app.UseAuthorization();
```

### <a name="useridentityname"></a>Пользователь. Identity . Безымян

По умолчанию API серверного приложения заполняет `User.Identity.Name` значение значением из `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name` типа утверждения (например, `2d64b3da-d9d5-42c6-9352-53d8df33d770@contoso.onmicrosoft.com` ).

Чтобы настроить приложение для получения значения из `name` типа утверждения, настройте [TokenValidationParameters. намеклаимтипе](xref:Microsoft.IdentityModel.Tokens.TokenValidationParameters.NameClaimType) <xref:Microsoft.AspNetCore.Authentication.JwtBearer.JwtBearerOptions> в `Startup.ConfigureServices` :

```csharp
using Microsoft.AspNetCore.Authentication.JwtBearer;

...

services.Configure<JwtBearerOptions>(
    AzureADDefaults.JwtBearerAuthenticationScheme, options =>
    {
        options.TokenValidationParameters.NameClaimType = "name";
    });
```

### <a name="app-settings"></a>Параметры приложения

*appsettings.jsв* файле содержит параметры для настройки обработчика носителя JWT, используемого для проверки маркеров доступа:

```json
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "Domain": "{DOMAIN}",
    "TenantId": "{TENANT ID}",
    "ClientId": "{SERVER API APP CLIENT ID}",
  }
}
```

Пример

```json
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "Domain": "contoso.onmicrosoft.com",
    "TenantId": "e86c78e2-8bb4-4c41-aefd-918e0565a45e",
    "ClientId": "41451fa7-82d9-4673-8fa5-69eff5a761fd",
  }
}
```

### <a name="weatherforecast-controller"></a>Контроллер Веасерфорекаст

Контроллер Веасерфорекаст (*Controllers/веасерфорекастконтроллер. CS*) предоставляет защищенный API с [`[Authorize]`](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute) атрибутом, примененным к контроллеру. **Важно** понимать, что:

* [`[Authorize]`](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute)Атрибут в этом контроллере API является единственным, который защищает этот API от несанкционированного доступа.
* [`[Authorize]`](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute)Атрибут, используемый в Blazor приложении сборки, служит указанием для приложения, которое должно быть проверено для правильной работы приложения.

```csharp
[Authorize]
[ApiController]
[Route("[controller]")]
public class WeatherForecastController : ControllerBase
{
    [HttpGet]
    public IEnumerable<WeatherForecast> Get()
    {
        ...
    }
}
```

## <a name="client-app-configuration"></a>Конфигурация клиентского приложения

*Этот раздел относится к **клиентскому** приложению решения.*

### <a name="authentication-package"></a>Пакет проверки подлинности

При создании приложения для использования рабочих или учебных учетных записей `SingleOrg` приложение автоматически получает ссылку на пакет для [библиотеки проверки подлинности Майкрософт](/azure/active-directory/develop/msal-overview) ([Microsoft. Authentication. Msal](https://www.nuget.org/packages/Microsoft.Authentication.WebAssembly.Msal/)). Пакет предоставляет набор примитивов, которые помогают приложению проверять подлинность пользователей и получать маркеры для вызова защищенных интерфейсов API.

При добавлении проверки подлинности в приложение вручную добавьте пакет в файл проекта приложения:

```xml
<PackageReference Include="Microsoft.Authentication.WebAssembly.Msal" 
  Version="3.2.0" />
```

Пакет [Microsoft. Authentication. Msal](https://www.nuget.org/packages/Microsoft.Authentication.WebAssembly.Msal/) . в качестве транзитного пакета добавляет пакет [Microsoft. AspNetCore. Components. Assembly. Authentication](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication/) в приложение.

### <a name="authentication-service-support"></a>Поддержка службы проверки подлинности

Добавлена поддержка <xref:System.Net.Http.HttpClient> экземпляров, включающая маркеры доступа при выполнении запросов к серверному проекту.

*Program.cs*:

```csharp
builder.Services.AddHttpClient("{APP ASSEMBLY}.ServerAPI", client => 
        client.BaseAddress = new Uri(builder.HostEnvironment.BaseAddress))
    .AddHttpMessageHandler<BaseAddressAuthorizationMessageHandler>();

builder.Services.AddTransient(sp => sp.GetRequiredService<IHttpClientFactory>()
    .CreateClient("{APP ASSEMBLY}.ServerAPI"));
```

Заполнитель `{APP ASSEMBLY}` — это имя сборки приложения (например, `BlazorSample.ServerAPI` ).

Поддержка проверки подлинности пользователей регистрируется в контейнере службы с помощью <xref:Microsoft.Extensions.DependencyInjection.MsalWebAssemblyServiceCollectionExtensions.AddMsalAuthentication%2A> метода расширения, предоставленного пакетом [Microsoft. Authentication. WebService. Msal](https://www.nuget.org/packages/Microsoft.Authentication.WebAssembly.Msal/) . Этот метод настраивает службы, необходимые для взаимодействия приложения с Identity поставщиком (IP).

*Program.cs*:

```csharp
builder.Services.AddMsalAuthentication(options =>
{
    builder.Configuration.Bind("AzureAd", options.ProviderOptions.Authentication);
    options.ProviderOptions.DefaultAccessTokenScopes.Add("{SCOPE URI}");
});
```

<xref:Microsoft.Extensions.DependencyInjection.MsalWebAssemblyServiceCollectionExtensions.AddMsalAuthentication%2A>Метод принимает обратный вызов для настройки параметров, необходимых для проверки подлинности приложения. Значения, необходимые для настройки приложения, можно получить из конфигурации AAD на портале Azure при регистрации приложения.

Конфигурация предоставляется файлом *wwwroot/appsettings.jsв* файле:

```json
{
  "AzureAd": {
    "Authority": "https://login.microsoftonline.com/{TENANT ID}",
    "ClientId": "{CLIENT APP CLIENT ID}",
    "ValidateAuthority": true
  }
}
```

Пример

```json
{
  "AzureAd": {
    "Authority": "https://login.microsoftonline.com/e86c78e2-...-918e0565a45e",
    "ClientId": "4369008b-21fa-427c-abaa-9b53bf58e538",
    "ValidateAuthority": true
  }
}
```

### <a name="access-token-scopes"></a>Области токенов доступа

Области токена доступа по умолчанию представляют собой список областей токенов доступа.

* Включается по умолчанию в запросе на вход.
* Используется для предоставления маркера доступа сразу после проверки подлинности.

Все области должны принадлежать одному и тому же приложению для правил Azure Active Directory. При необходимости можно добавить дополнительные области для дополнительных приложений API:

```csharp
builder.Services.AddMsalAuthentication(options =>
{
    ...
    options.ProviderOptions.DefaultAccessTokenScopes.Add("{SCOPE URI}");
});
```

[!INCLUDE[](~/includes/blazor-security/azure-scope.md)]

Дополнительные сведения см. в следующих разделах статьи *Дополнительные сценарии* :

* [Запрос дополнительных маркеров доступа](xref:security/blazor/webassembly/additional-scenarios#request-additional-access-tokens)
* [Присоединение маркеров к исходящим запросам](xref:security/blazor/webassembly/additional-scenarios#attach-tokens-to-outgoing-requests)


### <a name="imports-file"></a>Файл импорта

[!INCLUDE[](~/includes/blazor-security/imports-file-hosted.md)]

### <a name="index-page"></a>Страница индексации

[!INCLUDE[](~/includes/blazor-security/index-page-msal.md)]

### <a name="app-component"></a>Компонент приложения

[!INCLUDE[](~/includes/blazor-security/app-component.md)]

### <a name="redirecttologin-component"></a>Компонент Редиректтологин

[!INCLUDE[](~/includes/blazor-security/redirecttologin-component.md)]

### <a name="logindisplay-component"></a>Компонент Логиндисплай

[!INCLUDE[](~/includes/blazor-security/logindisplay-component.md)]

### <a name="authentication-component"></a>Компонент проверки подлинности

[!INCLUDE[](~/includes/blazor-security/authentication-component.md)]

### <a name="fetchdata-component"></a>Компонент FetchData

[!INCLUDE[](~/includes/blazor-security/fetchdata-component.md)]

## <a name="run-the-app"></a>Запуск приложения

Запустите приложение из серверного проекта. При использовании Visual Studio выполните одно из следующих действий.

* Задайте раскрывающийся список **запускаемые проекты** на панели инструментов для *приложения API сервера* и нажмите кнопку **выполнить** .
* Выберите серверный проект в **Обозреватель решений** и нажмите кнопку **выполнить** на панели инструментов или запустите приложение из меню **Отладка** .

<!-- HOLD
[!INCLUDE[](~/includes/blazor-security/usermanager-signinmanager.md)]
-->

[!INCLUDE[](~/includes/blazor-security/troubleshoot.md)]

## <a name="additional-resources"></a>Дополнительные ресурсы

* <xref:security/blazor/webassembly/additional-scenarios>
* [Запросы, не прошедшие проверку подлинности или неавторизованные веб-API в приложении с защищенным клиентом по умолчанию](xref:security/blazor/webassembly/additional-scenarios#unauthenticated-or-unauthorized-web-api-requests-in-an-app-with-a-secure-default-client)
* <xref:security/blazor/webassembly/aad-groups-roles>
* <xref:security/authentication/azure-active-directory/index>
* [Документация по платформе удостоверений Майкрософт](/azure/active-directory/develop/)
