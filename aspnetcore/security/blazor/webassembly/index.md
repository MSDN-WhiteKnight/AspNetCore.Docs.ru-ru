---
title: Защита ASP.NET Core Blazor WebAssembly
author: guardrex
description: Узнайте о защите приложений Blazor WebAssemlby как одностраничных приложений (SPA).
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 03/12/2020
no-loc:
- Blazor
- SignalR
uid: security/blazor/webassembly/index
ms.openlocfilehash: 652d4c61110f786396d9d5af4f131b817c40e333
ms.sourcegitcommit: 91dc1dd3d055b4c7d7298420927b3fd161067c64
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2020
ms.locfileid: "80219250"
---
# <a name="secure-aspnet-core-opno-locblazor-webassembly"></a>Защита ASP.NET Core Blazor WebAssembly

Автор: [Javier Calvarro Nelson](https://github.com/javiercn) (Хавьер Кальварро Нельсон)

[!INCLUDE[](~/includes/blazorwasm-preview-notice.md)]

[!INCLUDE[](~/includes/blazorwasm-3.2-template-article-notice.md)]

Защита приложений Blazor WebAssembly обеспечивается аналогично защите одностраничных приложений (SPA). Существует несколько подходов к проверке подлинности пользователей в одностраничных приложениях, но наиболее распространенным и комплексным является использование реализации на основе [протокола oAuth 2.0](https://oauth.net/), например [Open ID Connect (OIDC)](https://openid.net/connect/).

## <a name="authentication-library"></a>Библиотека проверки подлинности

Blazor WebAssembly поддерживает проверку подлинности и авторизацию приложений с использованием OIDC с помощью библиотеки `Microsoft.AspNetCore.Components.WebAssembly.Authentication`. Библиотека предоставляет набор примитивов для простой проверки подлинности на серверных компонентах ASP.NET Core. Библиотека включает в себя ASP.NET Core Identity с поддержкой авторизации API на основе [сервера удостоверений](https://identityserver.io/). Библиотеку можно использовать для проверки подлинности любого стороннего поставщика удостоверений (IP), который поддерживает OIDC (их называют поставщиками OpenID (OP)).

В основе поддержки проверки подлинности в Blazor WebAssembly лежит библиотека *oidc-client.js*, которая используется для управления сведениями о базовом протоколе проверки подлинности.

Кроме того, доступны другие варианты проверки подлинности одностраничных приложений, например использование файлов cookie SameSite. Однако технический проект BlazorWebAssembly реализован на основе oAuth и OIDC и представляет собой наилучший вариант для проверки подлинности в приложениях Blazor WebAssembly. По соображениям безопасности и в силу функциональных причин вместо [проверки подлинности на основе файлов cookie](xref:security/anti-request-forgery#cookie-based-authentication) была выбрана [проверка подлинности на основе токенов](xref:security/anti-request-forgery#token-based-authentication) на базе [JSON Web Tokens (JWT)](https://self-issued.info/docs/draft-ietf-oauth-json-web-token.html).

* Использование протокола на основе токенов позволяет уменьшить контактную зону атак, так как токены отправляются не во всех запросах.
* Конечные точки сервера не нуждаются в защите от [подделки межсайтовых запросов (CSRF)](xref:security/anti-request-forgery), так как токены отправляются явным образом. Это позволяет размещать приложения Blazor WebAssembly параллельно с приложениями MVC или Razor Pages.
* Разрешения токенов более узкие, чем разрешения файлов cookie. Например, токены нельзя использовать для управления учетной записью пользователя или изменения пароля пользователя, если такие функции не реализованы явным образом.
* У токенов короткий срок жизни (по умолчанию — один час), что ограничивает распространение атаки. Токены можно отозвать в любое время.
* Автономные токены JWT гарантируют выполнение надлежащего процесса проверки подлинности на клиенте и сервере. Например, на клиенте есть средства для обнаружения и проверки допустимости получаемых токенов, а также подтверждения их выпуска в рамках данного процесса проверки подлинности. Если третья сторона пытается изменить токен в ходе процесса проверки подлинности, клиент может обнаружить измененный токен и не использовать его.
* Токены с oAuth и OIDC обеспечивают безопасность приложения вне зависимости от поведения агента пользователя.
* Протоколы на основе токенов, такие как oAuth и OIDC, позволяют выполнять проверку подлинности и авторизацию размещенных и автономных приложений с одинаковым набором характеристик безопасности.

## <a name="authentication-process-with-oidc"></a>Процесс проверки подлинности с использованием OIDC

Библиотека `Microsoft.AspNetCore.Components.WebAssembly.Authentication` предлагает несколько примитивов для реализации проверки подлинности и авторизации с помощью OIDC. В общих чертах проверка подлинности работает следующим образом.

* Когда анонимный пользователь нажимает кнопку входа или запрашивает страницу с примененным атрибутом [`[Authorize]`](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute), он перенаправляется на страницу входа приложения (`/authentication/login`).
* На странице входа библиотека проверки подлинности готовится к перенаправлению на конечную точку авторизации. Конечная точка авторизации находится вне приложения Blazor WebAssembly и может размещаться в отдельном источнике. Конечная точка определяет, прошел ли пользователь проверку подлинности, и выдает в ответ один или несколько токенов. Библиотека проверки подлинности предоставляет обратный вызов входа для получения ответа проверки подлинности.
  * Если пользователь не прошел проверку подлинности, он перенаправляется в базовую систему проверки подлинности. Обычно это ASP.NET Core Identity.
  * Если пользователь уже прошел проверку подлинности, конечная точка авторизации создает соответствующие токены и перенаправляет браузер назад в конечную точку обратного вызова входа (`/authentication/login-callback`).
* Когда приложение Blazor WebAssembly загружает конечную точку обратного вызова входа (`/authentication/login-callback`), происходит обработка ответа проверки подлинности.
  * Если процесс проверки подлинности завершается успешно, пользователь проходит проверку подлинности и при необходимости перенаправляется по запрошенному исходному защищенному URL-адресу.
  * Если по какой-либо причине проверка подлинности завершается сбоем, пользователь направляется на страницу ошибки входа (`/authentication/login-failed`), и выводится сообщение об ошибке.
  
## <a name="options-for-hosted-apps-and-third-party-login-providers"></a>Варианты для размещенных приложений и сторонних поставщиков входа

При проверке подлинности и авторизации размещенного приложения Blazor WebAssembly в стороннем поставщике доступно несколько вариантов проверки подлинности пользователя. Выбор варианта зависит от вашего сценария.

Для получения дополнительной информации см. <xref:security/authentication/social/additional-claims>.

### <a name="authenticate-users-to-only-call-protected-third-party-apis"></a>Проверка подлинности пользователей для вызова только защищенных сторонних API

Проверьте подлинность пользователя с помощью потока oAuth на стороне клиента в стороннем поставщике API:

 ```csharp
 builder.services.AddOidcAuthentication(options => { ... });
 ```
 
 В этом сценарии:

* Сервер, на котором размещено приложение, не имеет особого значения.
* Невозможно обеспечить защиту API на сервере.
* Приложение может вызывать только защищенные сторонние интерфейсы API.

### <a name="authenticate-users-with-a-third-party-provider-and-call-protected-apis-on-the-host-server-and-the-third-party"></a>Проверка подлинности пользователей в стороннем поставщике и вызов защищенных API на сервере узла и сторонних API

Настройте удостоверение с помощью стороннего поставщика входа. Получите токены, необходимые для доступа к сторонним API, и сохраните их.

При входе пользователя в систему удостоверение собирает токены доступа и обновления в рамках процесса проверки подлинности. На этом этапе существует несколько подходов для отправки вызовов API к сторонним API.

#### <a name="use-a-server-access-token-to-retrieve-the-third-party-access-token"></a>Использование токена доступа сервера для получения стороннего токена доступа

С помощью созданного на сервере токена доступа получите сторонний токен доступа из конечной точки API сервера. Затем воспользуйтесь сторонним токеном доступа для вызова ресурсов стороннего API непосредственно из удостоверения на клиенте.

Этот вариант использовать не рекомендуется. Здесь сторонний токен доступа необходимо рассматривать так, как если бы он был создан для общедоступного клиента. С точки зрения использования oAuth у общедоступного приложения нет секрета клиента, так как оно не может считаться доверенным и надежно хранить секреты, а токен доступа создается для конфиденциального клиента. Конфиденциальный клиент — это клиент, у которого есть секрет клиента. Кроме того, предполагается, что он способен надежно хранить секреты.

* Исходя из того, что третья сторона выдала токен более доверенному клиенту, сторонним токенам доступа могут быть предоставлены дополнительные области для выполнения конфиденциальных операций.
* Аналогичным образом, токены обновления не должны выдаваться недоверенному клиенту, так как в этом случае клиент получает неограниченный доступ до применения других ограничений.

#### <a name="make-api-calls-from-the-client-to-the-server-api-in-order-to-call-third-party-apis"></a>Отправка вызовов API с API клиента на API сервера для вызова сторонних API

Отправьте вызов API с API клиента на API сервера. На сервере получите токен доступа для ресурса стороннего API и осуществите необходимый вызов.

Несмотря на то, что в этом случае для вызова стороннего API требуется выполнить дополнительный сетевой прыжок через сервер, этот подход является более безопасным.

* Сервер может хранить токены обновления и гарантировать, что приложение не потеряет доступ к сторонним ресурсам.
* Приложение не может получать с сервера токены доступа, содержащие более конфиденциальные разрешения.