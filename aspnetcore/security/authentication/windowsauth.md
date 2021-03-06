---
title: Настройка проверки подлинности Windows в ASP.NET Core
author: scottaddie
description: Узнайте, как настроить проверку подлинности Windows в ASP.NET Core для IIS и HTTP.sys.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc, seodec18
ms.date: 07/01/2019
uid: security/authentication/windowsauth
ms.openlocfilehash: 30f1f554a29412ed6b84115d457d2da1aba91c17
ms.sourcegitcommit: eb3e51d58dd713eefc242148f45bd9486be3a78a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/02/2019
ms.locfileid: "67500501"
---
# <a name="configure-windows-authentication-in-aspnet-core"></a>Настройка проверки подлинности Windows в ASP.NET Core

По [Scott Addie](https://twitter.com/Scott_Addie) и [Люк Лэтем](https://github.com/guardrex)

::: moniker range=">= aspnetcore-3.0"

Проверка подлинности Windows (также называется Negotiate, Kerberos или NTLM проверка подлинности) можно настроить для приложений ASP.NET Core, размещенных с [IIS](xref:host-and-deploy/iis/index), [Kestrel](xref:fundamentals/servers/kestrel), или [HTTP.sys](xref:fundamentals/servers/httpsys) .

::: moniker-end

::: moniker range="< aspnetcore-3.0"

Проверка подлинности Windows (также называется Negotiate, Kerberos или NTLM проверка подлинности) можно настроить для приложений ASP.NET Core, размещенных с [IIS](xref:host-and-deploy/iis/index) или [HTTP.sys](xref:fundamentals/servers/httpsys).

::: moniker-end

Проверка подлинности Windows зависит от операционной системы для проверки подлинности пользователей, приложений ASP.NET Core. Можно использовать проверку подлинности Windows, когда сервер работает в корпоративной сети с помощью удостоверения домена Active Directory или учетных записей Windows для идентификации пользователей. Проверка подлинности Windows является наилучшим образом подходит для среды интрасети, где пользователи, клиентские приложения и веб-серверы принадлежат к тому же домену Windows.

> [!NOTE]
> Проверка подлинности Windows не поддерживается с HTTP/2. Проблемы проверки подлинности могут отправляться на ответы HTTP/2, но клиент необходимо понизить HTTP/1.1 до проверки подлинности.

## <a name="iisiis-express"></a>IIS/IIS Express

Добавить службы проверки подлинности путем вызова <xref:Microsoft.Extensions.DependencyInjection.AuthenticationServiceCollectionExtensions.AddAuthentication*> (<xref:Microsoft.AspNetCore.Server.IISIntegration?displayProperty=fullName> пространства имен) в `Startup.ConfigureServices`:

```csharp
services.AddAuthentication(IISDefaults.AuthenticationScheme);
```

### <a name="launch-settings-debugger"></a>Параметры (отладчик) запуска

Конфигурация параметров запуска влияет только на *Properties/launchSettings.json* файл для IIS Express и не настраивает IIS для Windows проверки подлинности. Конфигурация сервера описан в [IIS](#iis) раздел.

**Веб-приложение** шаблон, доступный через Visual Studio или .NET Core CLI можно настроить для поддержки проверки подлинности Windows, которая обновляет *Properties/launchSettings.json* файла автоматически.

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

**новый проект**

1. Создайте новый проект.
1. Выберите **Новое веб-приложение ASP.NET Core**. Выберите **Далее**.
1. Введите имя в **имя_проекта** поля. Подтвердите **расположение** запись правильно, или укажите расположение для проекта. Выберите **Создать**.
1. Выберите **изменение** под **проверки подлинности**.
1. В **изменить способ проверки подлинности** выберите **проверки подлинности Windows**. Нажмите кнопку **ОК**.
1. Выберите **Веб-приложение**.
1. Выберите **Создать**.

Запустите приложение. Имя пользователя отображается в готовом для просмотра приложения пользовательского интерфейса.

**Существующий проект**

Свойства проекта, включите проверку подлинности Windows и отключить анонимную проверку подлинности:

1. В **обозревателе решений** щелкните проект правой кнопкой мыши и выберите пункт **Свойства**.
1. Выберите вкладку **Отладка**.
1. Снимите флажок для **Включение анонимной проверки подлинности**.
1. Установите флажок для **включить проверку подлинности Windows**.
1. Сохранить и закрыть страницу его свойств.

Кроме того, можно настроить свойства в `iisSettings` узел *launchSettings.json* файла:

[!code-json[](windowsauth/sample_snapshot/launchSettings.json?highlight=2-3)]

# <a name="visual-studio-code--net-core-clitabvisual-studio-codenetcore-cli"></a>[Visual Studio Code или .NET Core CLI](#tab/visual-studio-code+netcore-cli)

**новый проект**

Выполнение [команды dotnet new](/dotnet/core/tools/dotnet-new) с `webapp` аргумент (веб-приложения ASP.NET Core) и `--auth Windows` переключения:

```console
dotnet new webapp --auth Windows
```

**Существующий проект**

Обновление `iisSettings` узел *launchSettings.json* файла:

[!code-json[](windowsauth/sample_snapshot/launchSettings.json?highlight=2-3)]

---

При изменении существующего проекта, убедитесь, что файл проекта содержит ссылку на пакет для [метапакет Microsoft.AspNetCore.App](xref:fundamentals/metapackage-app) **или** [ Microsoft.AspNetCore.Authentication](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication/) пакет NuGet.

### <a name="iis"></a>IIS

Службы IIS используют [модуля ASP.NET Core](xref:host-and-deploy/aspnet-core-module) для размещения приложений ASP.NET Core. Проверка подлинности Windows настроена для IIS с помощью *web.config* файл. В следующих разделах показано как:

* Укажите локальный *web.config* файл, который активирует проверку подлинности Windows на сервере, при развертывании приложения.
* Использование диспетчера служб IIS для настройки *web.config* файл приложения ASP.NET Core, которое уже развернуто на сервер.

Если вы еще не сделали, включении служб IIS для размещения приложений ASP.NET Core. Дополнительные сведения см. в разделе <xref:host-and-deploy/iis/index>.

Включение службы роли IIS для проверки подлинности Windows. Дополнительные сведения см. в разделе [включить проверку подлинности Windows в службы роли IIS (см. шаг 2)](xref:host-and-deploy/iis/index#iis-configuration).

[По промежуточного слоя для интеграции IIS](xref:host-and-deploy/iis/index#enable-the-iisintegration-components) настраивается для автоматической проверки подлинности запросов по умолчанию. Дополнительные сведения см. в разделе [размещение ASP.NET Core в Windows со службами IIS: Параметры служб IIS (AutomaticAuthentication)](xref:host-and-deploy/iis/index#iis-options).

Модуль ASP.NET Core настроен на пересылку токена проверки подлинности Windows в приложение по умолчанию. Дополнительные сведения см. в разделе [Справочник по конфигурации модуля ASP.NET Core: Атрибуты элемента aspNetCore](xref:host-and-deploy/aspnet-core-module#attributes-of-the-aspnetcore-element).

Используйте **либо** из следующих подходов:

* **Перед публикацией и развертывание проекта,** добавьте следующий *web.config* файл в корневую папку проекта:

  [!code-xml[](windowsauth/sample_snapshot/web_2.config)]

  При публикации проекта с помощью пакета SDK .NET Core (без `<IsTransformWebConfigDisabled>` свойство значение `true` в файле проекта), опубликованной *web.config* файл включает `<location><system.webServer><security><authentication>` раздел. Дополнительные сведения о `<IsTransformWebConfigDisabled>` свойство, см. в разделе <xref:host-and-deploy/iis/index#webconfig-file>.

* **После публикации и развертывании проекта,** выполнения конфигурации на стороне сервера с помощью диспетчера IIS:

  1. В диспетчере IIS выберите сайт IIS, в разделе **сайты** узел **подключений** боковой панели.
  1. Дважды щелкните **проверки подлинности** в **IIS** области.
  1. Выберите **анонимную проверку подлинности**. Выберите **отключить** в **действия** боковой панели.
  1. Выберите **проверки подлинности Windows**. Выберите **включить** в **действия** боковой панели.

  При этих действий, IIS Manager вносит изменения в приложения *web.config* файл. Объект `<system.webServer><security><authentication>` добавляется узел с обновленными параметрами для `anonymousAuthentication` и `windowsAuthentication`:

  [!code-xml[](windowsauth/sample_snapshot/web_1.config?highlight=4-5)]

  `<system.webServer>` Добавлен в раздел *web.config* файл диспетчером IIS находится за пределами приложения `<location>` добавлен с помощью пакета SDK .NET Core, когда приложение публикуется раздел. Поскольку раздел добавляется вне `<location>` узел, параметры, наследуются [дочерние приложения](xref:host-and-deploy/iis/index#sub-applications) с текущим приложением. Чтобы запретить наследование, переместите добавленного `<security>` разделе внутри `<location><system.webServer>` раздел, в котором указан пакет SDK для .NET Core.

  Когда диспетчер IIS используется для добавления конфигурации IIS, оно влияет только на приложения *web.config* файл на сервере. Последующие развертывания приложения может перезаписать параметры на сервере, если копия сервера *web.config* заменяется проекта *web.config* файл. Используйте **либо** из следующих методов для управления параметрами:

  * Используйте диспетчер служб IIS, чтобы присвоить параметрам в *web.config* файл после файл перезаписывается при развертывании.
  * Добавить *файл web.config* приложение локально с параметрами.

::: moniker range=">= aspnetcore-3.0"

## <a name="kestrel"></a>Kestrel

 [Microsoft.AspNetCore.Authentication.Negotiate](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.Negotiate) пакет NuGet может использоваться с [Kestrel](xref:fundamentals/servers/kestrel) для поддержки проверки подлинности Windows, с помощью Negotiate, Kerberos и NTLM в Windows, Linux и macOS.

> [!WARNING]
> Учетные данные могут сохраняться среди запросов на подключение. *Согласование проверки подлинности не должны использоваться с прокси-серверы, если прокси-сервер поддерживает сходство соединения 1:1 (постоянное подключение), с Kestrel.*

> [!NOTE]
> Negotiate обработчик определяет, если базовый сервер имеет встроенную поддержку проверки подлинности Windows, и если он включен. Если сервер поддерживает проверку подлинности Windows, но он отключен, сообщение об ошибке вопросом, нужно включить в реализации сервера. Если на сервере включена проверка подлинности Windows, обработчик Negotiate прозрачно перенаправляет к нему.

 Добавить службы проверки подлинности путем вызова <xref:Microsoft.Extensions.DependencyInjection.AuthenticationServiceCollectionExtensions.AddAuthentication*> (`Microsoft.AspNetCore.Authentication.Negotiate` пространства имен) и `AddNegotitate` (`Microsoft.AspNetCore.Authentication.Negotiate` пространства имен) в `Startup.ConfigureServices`:

 ```csharp
services.AddAuthentication(NegotiateDefaults.AuthenticationScheme)
    .AddNegotiate();
```

Добавьте по промежуточного слоя проверки подлинности путем вызова <xref:Microsoft.AspNetCore.Builder.AuthAppBuilderExtensions.UseAuthentication*> в `Startup.Configure`:

 ```csharp
app.UseAuthentication();

app.UseMvc();
```

Дополнительные сведения о по промежуточного слоя, см. в разделе <xref:fundamentals/middleware/index>.

Разрешены анонимные запросы. Используйте [авторизации ASP.NET Core](xref:security/authorization/introduction) бросить вызов анонимные запросы для проверки подлинности.

### <a name="windows-environment-configuration"></a>Конфигурация среды Windows

[Microsoft.AspNetCore.Authentication.Negotiate](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.Negotiate) компонент выполняет проверку подлинности в режиме пользователя. Необходимо добавить имена участника-службы (SPN) с учетной записью пользователя, под управлением службы, а не учетной записи компьютера. Выполнение `setspn -S HTTP/mysrevername.mydomain.com myuser` в административной командной оболочке.

### <a name="linux-and-macos-environment-configuration"></a>Конфигурации среды Linux и macOS

Инструкции по присоединению к домену Windows машину Linux или macOS [подключение к серверу SQL Server, использование проверки подлинности Windows — Kerberos Studio данных Azure](/sql/azure-data-studio/enable-kerberos?view=sql-server-2017#join-your-os-to-the-active-directory-domain-controller) статьи. Инструкции создания учетной записи компьютера для компьютера Linux в домене. Имена участников-служб необходимо добавить в эту учетную запись компьютера.

> [!NOTE]
> Если следовать инструкциям в [подключение к серверу SQL Server, использование проверки подлинности Windows — Kerberos Studio данных Azure](/sql/azure-data-studio/enable-kerberos?view=sql-server-2017#join-your-os-to-the-active-directory-domain-controller) статьи, замените `python-software-properties` с `python3-software-properties` при необходимости.

Как только Linux или macOS компьютер присоединен к домену, необходимы дополнительные действия для предоставления [файла keytab](https://blogs.technet.microsoft.com/pie/2018/01/03/all-you-need-to-know-about-keytab-files/) с имена SPN:

* На контроллере домена добавьте учетную запись компьютера новой веб-службы SPN:
  * `setspn -S HTTP/mywebservice.mydomain.com mymachine`
  * `setspn -S HTTP/mywebservice@MYDOMAIN.COM mymachine`
* Используйте [ktpass](/windows-server/administration/windows-commands/ktpass) для создания файла keytab:
  * `ktpass -princ HTTP/mywebservice.mydomain.com@MYDOMAIN.COM -pass myKeyTabFilePassword -mapuser MYDOMAIN\mymachine$ -pType KRB5_NT_PRINCIPAL -out c:\temp\mymachine.HTTP.keytab -crypto AES256-SHA1`
  * Некоторые поля должно быть указано в верхний регистр, как указано.
* Скопируйте файл keytab на компьютер Linux или macOS.
* Выберите файл keytab через переменную среды: `export KRB5_KTNAME=/tmp/mymachine.HTTP.keytab`
* Вызвать `klist` Показать имена SPN, доступные в настоящий момент.

> [!NOTE]
> Файла keytab содержит учетные данные домена для доступа и должен быть защищен соответствующим образом.

::: moniker-end

## <a name="httpsys"></a>HTTP.sys

[HTTP.sys](xref:fundamentals/servers/httpsys) поддерживает проверку подлинности Windows режима ядра с помощью Negotiate, NTLM или обычную проверку подлинности.

Добавить службы проверки подлинности путем вызова <xref:Microsoft.Extensions.DependencyInjection.AuthenticationServiceCollectionExtensions.AddAuthentication*> (<xref:Microsoft.AspNetCore.Server.HttpSys?displayProperty=fullName> пространства имен) в `Startup.ConfigureServices`:

```csharp
services.AddAuthentication(HttpSysDefaults.AuthenticationScheme);
```

Настроить узел веб-приложения для использования HTTP.sys с проверкой подлинности Windows (*Program.cs*). <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderHttpSysExtensions.UseHttpSys*> находится в пространстве имен <xref:Microsoft.AspNetCore.Server.HttpSys?displayProperty=fullName>.

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](windowsauth/sample_snapshot/Program_GenericHost.cs?highlight=13-19)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!code-csharp[](windowsauth/sample_snapshot/Program_WebHost.cs?highlight=9-15)]

::: moniker-end

> [!NOTE]
> HTTP.sys делегирует задачи в проверку подлинности в режиме ядра с помощью протокола проверки подлинности Kerberos. Проверка подлинности в режиме пользователя не поддерживается с Kerberos и HTTP.sys. Необходимо использовать учетную запись компьютера для расшифровки маркера/билета Kerberos, полученного из Active Directory и переадресованного клиентом на сервер для проверки подлинности пользователя. Зарегистрируйте имя субъекта-службы (SPN) для узла, а не пользователя приложения.

> [!NOTE]
> HTTP.sys не поддерживается на сервере Nano Server версии 1709 или более поздней версии. Чтобы использовать проверку подлинности Windows и HTTP.sys с сервером Nano Server, используйте [контейнера Server Core (microsoft/windowsservercore)](https://hub.docker.com/r/microsoft/windowsservercore/). Дополнительные сведения о Server Core см. в разделе [Какова вариант установки Server Core в Windows Server?](/windows-server/administration/server-core/what-is-server-core).

## <a name="authorize-users"></a>Авторизация пользователей

Состояние конфигурации анонимный доступ определяет способ, которым `[Authorize]` и `[AllowAnonymous]` атрибуты используются в приложении. Следующих двух разделах объясняется, как обрабатывать запрещенные и разрешенные конфигурации состояния анонимный доступ.

### <a name="disallow-anonymous-access"></a>Запретить анонимный доступ

Если включена проверка подлинности Windows и анонимный доступ отключен, `[Authorize]` и `[AllowAnonymous]` атрибуты никак не влияют. Если узел IIS настроен на запрет анонимного доступа, никогда не достигнут приложения. По этой причине `[AllowAnonymous]` атрибут не применяется.

### <a name="allow-anonymous-access"></a>Разрешить анонимный доступ

Если включены как проверка подлинности Windows, так и анонимный доступ, используйте `[Authorize]` и `[AllowAnonymous]` атрибуты. `[Authorize]` Атрибут позволяет защищать конечные точки приложения, которые требуют проверки подлинности. `[AllowAnonymous]` Атрибут переопределения `[Authorize]` атрибут в приложениях с возможностью анонимного доступа. Дополнительные сведения об использовании атрибута см. в разделе <xref:security/authorization/simple>.

> [!NOTE]
> По умолчанию пользователи, у которых отсутствует разрешение на доступ к странице, откроется пустой ответ HTTP 403. [StatusCodePages по промежуточного слоя](xref:fundamentals/error-handling#usestatuscodepages) можно настроить для предоставления пользователям более комфортные «Отказано в доступе».

## <a name="impersonation"></a>Олицетворение

ASP.NET Core не реализует олицетворения. Приложения запускаются с удостоверение приложения для всех запросов, с помощью удостоверения пула или процесс приложения. Если приложение должно выполнить действие от имени пользователя, используйте [WindowsIdentity.RunImpersonated](xref:System.Security.Principal.WindowsIdentity.RunImpersonated*) в [терминалов встроенного ПО промежуточного слоя](xref:fundamentals/middleware/index#create-a-middleware-pipeline-with-iapplicationbuilder) в `Startup.Configure`. Запустить одно действие в этом контексте, а затем закройте контекста.

[!code-csharp[](windowsauth/sample_snapshot/Startup.cs?highlight=10-19)]

`RunImpersonated` не поддерживает асинхронные операции и не должны использоваться для сложных сценариев. Например упаковки всего запросов или по промежуточного слоя цепочек не поддерживается и не рекомендуется.

::: moniker range=">= aspnetcore-3.0"

Хотя [Microsoft.AspNetCore.Authentication.Negotiate](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.Negotiate) пакет включает проверку подлинности в Windows, Linux и macOS, олицетворение поддерживается только в Windows.

::: moniker-end

## <a name="claims-transformations"></a>Преобразования утверждений

::: moniker range=">= aspnetcore-3.0"

При размещении в службах IIS, <xref:Microsoft.AspNetCore.Authentication.AuthenticationService.AuthenticateAsync*> не вызывается внутри для инициализации пользователя. Таким образом, реализация <xref:Microsoft.AspNetCore.Authentication.IClaimsTransformation>, используемая для преобразования утверждений после каждой проверки подлинности, не активируется по умолчанию. Дополнительные сведения и пример кода, который активирует преобразования утверждений, см. в разделе <xref:host-and-deploy/aspnet-core-module#in-process-hosting-model>.

::: moniker-end

::: moniker range="< aspnetcore-3.0"

При размещении в режиме в процессе IIS <xref:Microsoft.AspNetCore.Authentication.AuthenticationService.AuthenticateAsync*> не вызывается внутри для инициализации пользователя. Таким образом, реализация <xref:Microsoft.AspNetCore.Authentication.IClaimsTransformation>, используемая для преобразования утверждений после каждой проверки подлинности, не активируется по умолчанию. Дополнительные сведения и пример кода, который активирует преобразования утверждений, при размещении в процессе, см. в разделе <xref:host-and-deploy/aspnet-core-module#in-process-hosting-model>.

::: moniker-end

## <a name="additional-resources"></a>Дополнительные ресурсы

* [dotnet publish](/dotnet/core/tools/dotnet-publish)
* <xref:host-and-deploy/iis/index>
* <xref:host-and-deploy/aspnet-core-module>
* <xref:host-and-deploy/visual-studio-publish-profiles>
