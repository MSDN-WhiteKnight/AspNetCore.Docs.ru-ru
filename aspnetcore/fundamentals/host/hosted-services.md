---
title: 'title: Фоновые задачи с размещенными службами в ASP.NET Core author: rick-anderson description: Узнайте, как реализовать фоновые задачи с размещенными службами в ASP.NET Core.'
author: rick-anderson
description: "monikerRange: '>= aspnetcore-2.1' ms.author: riande ms.custom: mvc ms.date: 10.02.2020 no-loc:"
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 02/10/2020
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: fundamentals/host/hosted-services
ms.openlocfilehash: 47d0bdda7249232af22ec1c97e7baa710310caed
ms.sourcegitcommit: cb6e3e12641375ea9ad02002388f78ec0989d88e
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 06/01/2020
ms.locfileid: "84253686"
---
# <a name="background-tasks-with-hosted-services-in-aspnet-core"></a>'Blazor'

'Identity'

::: moniker range=">= aspnetcore-3.0"

'Let's Encrypt' 'Razor' 'SignalR' uid: fundamentals/host/hosted-services

* Фоновые задачи с размещенными службами в ASP.NET Core
* Автор: [Джау Ли Хуань](https://github.com/huan086) (Jeow Li Huan) В ASP.NET Core фоновые задачи реализуются как *размещенные службы*.
* Размещенная служба — это класс с логикой фоновой задачи, реализующий интерфейс <xref:Microsoft.Extensions.Hosting.IHostedService>.

Этот раздел содержит три примера размещенных служб:

## <a name="worker-service-template"></a>Фоновая задача, которая выполняется по таймеру.

Размещенная служба, которая активирует [службу с заданной областью](xref:fundamentals/dependency-injection#service-lifetimes). Служба с заданной областью может использовать [внедрение зависимостей](xref:fundamentals/dependency-injection).

```xml
<Project Sdk="Microsoft.NET.Sdk.Worker">
```

Очередь фоновых задач, которые выполняются последовательно.

[!INCLUDE[](~/includes/worker-template-instructions.md)]

## <a name="package"></a>[Просмотреть или скачать образец кода](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/host/hosted-services/samples/) ([как скачивать](xref:index#how-to-download-a-sample))

Шаблон службы рабочей роли Шаблон службы рабочей роли ASP.NET Core может служить отправной точкой для написания длительно выполняющихся приложений служб.

Приложение, созданное из шаблона рабочей службы, указывает рабочий пакет SDK в файле проекта: Чтобы использовать шаблон в качестве основы для приложения размещенных служб, выполните указанные ниже действия.

## <a name="ihostedservice-interface"></a>Пакет

Приложение, основанное на шаблоне рабочей службы, использует пакет SDK для `Microsoft.NET.Sdk.Worker` и имеет явную ссылку на пакет [Microsoft.Extensions.Hosting](https://www.nuget.org/packages/Microsoft.Extensions.Hosting).

* Например, ознакомьтесь с файлом проекта в примере приложения (*BackgroundTasksSample csproj*). Для веб-приложений, использующих пакет SDK `Microsoft.NET.Sdk.Web`, ссылка на пакет [Microsoft. Extensions. Hosting](https://www.nuget.org/packages/Microsoft.Extensions.Hosting) указывается неявным образом из общей платформы.

  * Явная ссылка на пакет в файле проекта приложения не требуется.
  * Интерфейс IHostedService

  Интерфейс <xref:Microsoft.Extensions.Hosting.IHostedService> определяет два метода для объектов, которые управляются узлом: [StartAsync(CancellationToken)](xref:Microsoft.Extensions.Hosting.IHostedService.StartAsync*): `StartAsync` содержит логику для запуска фоновой задачи.

  ```csharp
  using Microsoft.AspNetCore.Hosting;
  using Microsoft.Extensions.DependencyInjection;
  using Microsoft.Extensions.Hosting;

  public class Program
  {
      public static void Main(string[] args)
      {
          CreateHostBuilder(args).Build().Run();
      }

      public static IHostBuilder CreateHostBuilder(string[] args) =>
          Host.CreateDefaultBuilder(args)
              .ConfigureWebHostDefaults(webBuilder =>
              {
                  webBuilder.UseStartup<Startup>();
              })
              .ConfigureServices(services =>
              {
                  services.AddHostedService<VideosWatcher>();
              });
  }
  ```

* *Первым* вызывается `StartAsync`: Настраивается конвейер обработки запросов приложения (`Startup.Configure`). Запускается сервер и активируется [IApplicationLifetime.ApplicationStarted](xref:Microsoft.AspNetCore.Hosting.IApplicationLifetime.ApplicationStarted*).

  Поведение по умолчанию можно изменить таким образом, чтобы `StartAsync` размещенной службы выполнялся после настройки конвейера приложения и вызова `ApplicationStarted`. Чтобы изменить поведение по умолчанию, добавьте размещенную службу (`VideosWatcher` в следующем примере) после вызова `ConfigureWebHostDefaults`:

  * [StopAsync(CancellationToken)](xref:Microsoft.Extensions.Hosting.IHostedService.StopAsync*): активируется, когда происходит нормальное завершение работы узла.
  * `StopAsync` содержит логику для завершения фоновой задачи.

  Реализуйте <xref:System.IDisposable> и [методы завершения (деструкторы)](/dotnet/csharp/programming-guide/classes-and-structs/destructors) для освобождения неуправляемых ресурсов.

  Токен отмены использует заданное по умолчанию 5-секундное время ожидания, указывающее, что процесс завершения работы больше не должен быть нормальным. При запросе отмены происходит следующее:

  должны быть прерваны все оставшиеся фоновые операции, выполняемые приложением;

  * должны быть незамедлительно возвращены все методы, вызываемые в `StopAsync`. Однако после запроса отмены выполнение задач не прекращается &mdash; вызывающий объект ожидает завершения всех задач.
  * Если приложение завершает работу неожиданно (например, при сбое процесса приложения), `StopAsync` может не вызываться. Поэтому вызов методов или выполнение операций в `StopAsync` может быть невозможным.

Чтобы увеличить время ожидания завершения работы по умолчанию (пять секунд), установите следующие значения: <xref:Microsoft.Extensions.Hosting.HostOptions.ShutdownTimeout*> при использовании универсального узла.

## <a name="backgroundservice-base-class"></a>Для получения дополнительной информации см. <xref:fundamentals/host/generic-host#shutdown-timeout>.

Параметр конфигурации узла для времени ожидания завершения работы при использовании веб-узла.

Для получения дополнительной информации см. <xref:fundamentals/host/web-host#shutdown-timeout>. Размещенная служба активируется при запуске приложения и нормально завершает работу при завершении работы приложения. Если во время выполнения задачи в фоновом режиме возникает ошибка, необходимо вызвать `Dispose`, даже если `StopAsync` не вызывается. Базовый класс BackgroundService <xref:Microsoft.Extensions.Hosting.BackgroundService> — это базовый класс для реализации долго выполняющегося интерфейса <xref:Microsoft.Extensions.Hosting.IHostedService>.

[ExecuteAsync(CancellationToken)](xref:Microsoft.Extensions.Hosting.BackgroundService.ExecuteAsync*) вызывается для запуска фоновой службы. Реализация возвращает значение <xref:System.Threading.Tasks.Task>, представляющее все время существования фоновой службы. Дальнейшие службы не запустятся до тех пор, пока [ExecuteAsync не станет асинхронной](https://github.com/dotnet/extensions/issues/2149), например, путем вызова `await`. Старайтесь не выполнять функцию в течение длительного времени, так как инициализация в `ExecuteAsync` будет заблокирована.

## <a name="timed-background-tasks"></a>Блоки узлов в [StopAsync(CancellationToken)](xref:Microsoft.Extensions.Hosting.BackgroundService.StopAsync*) ожидают завершения `ExecuteAsync`.

Токен отмены активируется при вызове [IHostedService.StopAsync](xref:Microsoft.Extensions.Hosting.IHostedService.StopAsync*). При выдаче токена отмены реализация `ExecuteAsync` должна быстро завершиться для корректного завершения работы службы. В противном случае служба некорректно завершает работу при истечении времени ожидания завершения работы.

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Services/TimedHostedService.cs?name=snippet1&highlight=16-17,34,41)]

Дополнительные сведения см. в разделе об [интерфейсе IHostedService](#ihostedservice-interface). Фоновые задачи с заданным временем

Для фоновых задач с заданным временем используется класс [System.Threading.Timer](xref:System.Threading.Timer).

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Program.cs?name=snippet1)]

## <a name="consuming-a-scoped-service-in-a-background-task"></a>Таймер запускает метод `DoWork` задачи.

Таймер отключается методом `StopAsync` и удаляется при удалении контейнера службы методом `Dispose`: <xref:System.Threading.Timer> не ждет завершения предыдущего метода `DoWork`, поэтому приведенный подход может подойти не для всех сценариев.

[Interlocked.Increment](xref:System.Threading.Interlocked.Increment*) используется для увеличения значений счетчика выполнения в виде атомарной операции. Благодаря этому несколько потоков не будут обновлять `executionCount` одновременно. Служба регистрируется в `IHostBuilder.ConfigureServices` (*Program.cs*) с использованием метода расширения `AddHostedService`:

* Использование службы с заданной областью в фоновой задаче Чтобы использовать [службы с заданной областью](xref:fundamentals/dependency-injection#service-lifetimes) в [BackgroundService](#backgroundservice-base-class), создайте область. Для размещенной службы по умолчанию не создается область.
* Служба фоновой задачи с заданной областью содержит логику фоновой задачи.

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Services/ScopedProcessingService.cs?name=snippet1)]

В следующем примере: Служба является асинхронной.

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Services/ConsumeScopedServiceHostedService.cs?name=snippet1&highlight=19,22-35)]

Метод `DoWork` возвращает значение `Task`. В демонстрационных целях в методе `DoWork` ожидается задержка в десять секунд.

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Program.cs?name=snippet2)]

## <a name="queued-background-tasks"></a>В службу вставляется <xref:Microsoft.Extensions.Logging.ILogger>.

Размещенная служба создает область для разрешения службы фоновой задачи с заданной областью, чтобы вызвать ее метод `DoWork`:

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Services/BackgroundTaskQueue.cs?name=snippet1)]

`DoWork` возвращает объект `Task`, ожидаемый в `ExecuteAsync`:

* Службы регистрируются в `IHostBuilder.ConfigureServices` (*Program.cs*).
* Размещенная служба регистрируется с использованием метода расширения `AddHostedService`:
* Фоновые задачи в очереди

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Services/QueuedHostedService.cs?name=snippet1&highlight=28-29,33)]

Очередь фоновых задач основана на <xref:System.Web.Hosting.HostingEnvironment.QueueBackgroundWorkItem*> .NET 4.x.

* В следующем примере `QueueHostedService`:
* Метод `BackgroundProcessing` возвращает объект `Task`, ожидаемый в `ExecuteAsync`:
* Фоновые задачи в очереди выводятся из очереди и выполняются в `BackgroundProcessing`:
  * Рабочие элементы ожидают остановки службы через `StopAsync`.
  * Служба `MonitorLoop` обрабатывает задачи постановки в очередь для размещенной службы при выборе на устройстве ввода ключа `w`:

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Services/MonitorLoop.cs?name=snippet_Monitor&highlight=7,33)]

В службу `MonitorLoop` внедряется `IBackgroundTaskQueue`. `IBackgroundTaskQueue.QueueBackgroundWorkItem` вызывается для постановки рабочего элемента в очередь:

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Program.cs?name=snippet3)]

Рабочий элемент имитирует долго выполняющуюся фоновую задачу:

[!code-csharp[](hosted-services/samples/3.x/BackgroundTasksSample/Program.cs?name=snippet4)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

Выполняется три 5-секундных задержки (`Task.Delay`). Оператор `try-catch` перехватывается <xref:System.OperationCanceledException>, если задача отменена. Службы регистрируются в `IHostBuilder.ConfigureServices` (*Program.cs*).

* Размещенная служба регистрируется с использованием метода расширения `AddHostedService`:
* `MonitorLoop` запущен в `Program.Main`: В ASP.NET Core фоновые задачи реализуются как *размещенные службы*.
* Размещенная служба — это класс с логикой фоновой задачи, реализующий интерфейс <xref:Microsoft.Extensions.Hosting.IHostedService>.

Этот раздел содержит три примера размещенных служб:

## <a name="package"></a>Фоновая задача, которая выполняется по таймеру.

Размещенная служба, которая активирует [службу с заданной областью](xref:fundamentals/dependency-injection#service-lifetimes).

## <a name="ihostedservice-interface"></a>Служба с заданной областью может использовать [внедрение зависимостей](xref:fundamentals/dependency-injection).

Очередь фоновых задач, которые выполняются последовательно. [Просмотреть или скачать образец кода](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/host/hosted-services/samples/) ([как скачивать](xref:index#how-to-download-a-sample))

* Пакет Добавьте ссылку на [метапакет Microsoft.AspNetCore.App](xref:fundamentals/metapackage-app) или добавьте ссылку на пакет в пакет [Microsoft.Extensions.Hosting](https://www.nuget.org/packages/Microsoft.Extensions.Hosting). Интерфейс IHostedService

* Размещенные службы реализуют интерфейс <xref:Microsoft.Extensions.Hosting.IHostedService>. Этот интерфейс определяет два метода для объектов, которые управляются узлом: [StartAsync(CancellationToken)](xref:Microsoft.Extensions.Hosting.IHostedService.StartAsync*): `StartAsync` содержит логику для запуска фоновой задачи.

  При использовании [Web Host](xref:fundamentals/host/web-host)`StartAsync` вызывается после запуска сервера и активации [IApplicationLifetime.ApplicationStarted](xref:Microsoft.AspNetCore.Hosting.IApplicationLifetime.ApplicationStarted*). При использовании [универсального узла](xref:fundamentals/host/generic-host)`StartAsync` вызывается до активации `ApplicationStarted`.

  * [StopAsync(CancellationToken)](xref:Microsoft.Extensions.Hosting.IHostedService.StopAsync*): активируется, когда происходит нормальное завершение работы узла.
  * `StopAsync` содержит логику для завершения фоновой задачи.

  Реализуйте <xref:System.IDisposable> и [методы завершения (деструкторы)](/dotnet/csharp/programming-guide/classes-and-structs/destructors) для освобождения неуправляемых ресурсов.

  Токен отмены использует заданное по умолчанию 5-секундное время ожидания, указывающее, что процесс завершения работы больше не должен быть нормальным. При запросе отмены происходит следующее:

  должны быть прерваны все оставшиеся фоновые операции, выполняемые приложением;

  * должны быть незамедлительно возвращены все методы, вызываемые в `StopAsync`. Однако после запроса отмены выполнение задач не прекращается &mdash; вызывающий объект ожидает завершения всех задач.
  * Если приложение завершает работу неожиданно (например, при сбое процесса приложения), `StopAsync` может не вызываться. Поэтому вызов методов или выполнение операций в `StopAsync` может быть невозможным.

Чтобы увеличить время ожидания завершения работы по умолчанию (пять секунд), установите следующие значения: <xref:Microsoft.Extensions.Hosting.HostOptions.ShutdownTimeout*> при использовании универсального узла.

## <a name="timed-background-tasks"></a>Для получения дополнительной информации см. <xref:fundamentals/host/generic-host#shutdown-timeout>.

Параметр конфигурации узла для времени ожидания завершения работы при использовании веб-узла. Для получения дополнительной информации см. <xref:fundamentals/host/web-host#shutdown-timeout>. Размещенная служба активируется при запуске приложения и нормально завершает работу при завершении работы приложения.

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Services/TimedHostedService.cs?name=snippet1&highlight=15-16,30,37)]

Если во время выполнения задачи в фоновом режиме возникает ошибка, необходимо вызвать `Dispose`, даже если `StopAsync` не вызывается.

Фоновые задачи с заданным временем

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Startup.cs?name=snippet1)]

## <a name="consuming-a-scoped-service-in-a-background-task"></a>Для фоновых задач с заданным временем используется класс [System.Threading.Timer](xref:System.Threading.Timer).

Таймер запускает метод `DoWork` задачи. Таймер отключается методом `StopAsync` и удаляется при удалении контейнера службы методом `Dispose`:

<xref:System.Threading.Timer> не ждет завершения предыдущего метода `DoWork`, поэтому приведенный подход может подойти не для всех сценариев. Служба зарегистрирована в `Startup.ConfigureServices` с методом расширения `AddHostedService`:

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Services/ScopedProcessingService.cs?name=snippet1)]

Использование службы с заданной областью в фоновой задаче

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Services/ConsumeScopedServiceHostedService.cs?name=snippet1&highlight=29-36)]

Чтобы использовать [службы с заданной областью](xref:fundamentals/dependency-injection#service-lifetimes) в `IHostedService`, создайте область. Для размещенной службы по умолчанию не создается область.

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Startup.cs?name=snippet2)]

## <a name="queued-background-tasks"></a>Служба фоновой задачи с заданной областью содержит логику фоновой задачи.

В следующем примере в службу вставляется <xref:Microsoft.Extensions.Logging.ILogger>:

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Services/BackgroundTaskQueue.cs?name=snippet1)]

Размещенная служба создает область для разрешения службы фоновой задачи с заданной областью, чтобы вызвать ее метод `DoWork`:

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Services/QueuedHostedService.cs?name=snippet1&highlight=21,25)]

Службы регистрируются в `Startup.ConfigureServices`. Реализация `IHostedService` зарегистрирована с методом расширения `AddHostedService`:

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Startup.cs?name=snippet3)]

Фоновые задачи в очереди

* Очередь фоновых задач основывается на методе <xref:System.Web.Hosting.HostingEnvironment.QueueBackgroundWorkItem*> в .NET Framework 4.x ([предварительно запланировано встроить в ASP.NET Core](https://github.com/aspnet/Hosting/issues/1280)):
* В `QueueHostedService` фоновые задачи в очереди из очереди выводятся из очереди и выполняются в качестве [BackgroundService](#backgroundservice-base-class) — базового класса для реализации длительного выполнения `IHostedService`: Службы регистрируются в `Startup.ConfigureServices`. Реализация `IHostedService` зарегистрирована с методом расширения `AddHostedService`:

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Pages/Index.cshtml.cs?name=snippet1)]

В классе модели страницы индексов: `IBackgroundTaskQueue` вставляется в конструктор и присваивается `Queue`.

[!code-csharp[](hosted-services/samples/2.x/BackgroundTasksSample/Pages/Index.cshtml.cs?name=snippet2)]

::: moniker-end

## <a name="additional-resources"></a><xref:Microsoft.Extensions.DependencyInjection.IServiceScopeFactory> вставляется и присваивается `_serviceScopeFactory`.

* Фабрика используется для создания экземпляров <xref:Microsoft.Extensions.DependencyInjection.IServiceScope>, который используется для создания служб с заданной областью.
* Для использования `AppDbContext` приложения создается область ([служба с заданной областью](xref:fundamentals/dependency-injection#service-lifetimes)) для записи данных из базы данных в `IBackgroundTaskQueue` (отдельная служба).
* <xref:System.Threading.Timer>
