---
title: Размещение образов ASP.NET Core с помощью DOCKER по протоколу HTTPS
author: rick-anderson
description: Сведения о размещении образов ASP.NET Core с помощью DOCKER по протоколу HTTPS
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 07/05/2019
uid: security/docker-https
ms.openlocfilehash: f17a3abe1b00b39b7b6787be5b20ce65771190b8
ms.sourcegitcommit: 257cc3fe8c1d61341aa3b07e5bc0fa3d1c1c1d1c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/19/2019
ms.locfileid: "69619699"
---
# <a name="hosting-aspnet-core-images-with-docker-over-https"></a>Размещение образов ASP.NET Core с помощью DOCKER по протоколу HTTPS

Автор: [Рик Андерсон](https://twitter.com/RickAndMSFT) (Rick Anderson)

По умолчанию ASP.NET Core использует [протокол HTTPS](/aspnet/core/security/enforcing-ssl). [Протокол HTTPS](https://en.wikipedia.org/wiki/HTTPS) использует [Сертификаты](https://en.wikipedia.org/wiki/Public_key_certificate) для доверия, удостоверений и шифрования.

В этом документе объясняется, как запускать предварительно созданные образы контейнеров с помощью протокола HTTPS.

Сценарии разработки см. [в статье Разработка приложений ASP.NET Core с помощью DOCKER по протоколу HTTPS](https://github.com/dotnet/dotnet-docker/blob/master/samples/aspnetapp/aspnetcore-docker-https-development.md) .

Для этого примера требуется [docker 17,06](https://docs.docker.com/release-notes/docker-ce) или более поздней версии [клиента DOCKER](https://www.docker.com/products/docker).

## <a name="prerequisites"></a>Предварительные требования

Для выполнения некоторых инструкций в этом документе требуется [пакет SDK для .NET Core 2,2](https://www.microsoft.com/net/download) или более поздней версии.

## <a name="certificates"></a>Сертификаты

Сертификат из [центра](https://en.wikipedia.org/wiki/Certificate_authority) сертификации необходим для [размещения в рабочей среде](https://blogs.msdn.microsoft.com/webdev/2017/11/29/configuring-https-in-asp-net-core-across-different-platforms/) для домена.  [Шифрование](https://letsencrypt.org/) — это центр сертификации, предлагающий бесплатные сертификаты.

В этом документе используются [самозаверяющие сертификаты разработки](https://en.wikipedia.org/wiki/Self-signed_certificate) для размещения предварительно созданных образов `localhost`. Инструкции аналогичны использованию рабочих сертификатов.

Для производственных сертификатов:

* `dotnet dev-certs` Средство не требуется.
* Сертификаты не обязательно хранить в расположении, используемом в инструкциях. Должно работать любое расположение, хотя не рекомендуется хранить сертификаты в каталоге сайта.

Инструкции по подключению сертификатов к контейнерам. Сертификаты можно добавить в образы контейнеров с помощью `COPY` команды в Dockerfile. Копирование сертификатов в образ не рекомендуется:

* Использование одного и того же образа для тестирования с помощью сертификатов разработчика затрудняется.
* Использование одного и того же образа для размещения с производственными сертификатами затрудняется.
* Существует значительный риск раскрытия сертификата.

## <a name="running-pre-built-container-images-with-https"></a>Запуск предварительно созданных образов контейнеров с помощью HTTPS

Используйте следующие инструкции для настройки операционной системы.

### <a name="windows-using-linux-containers"></a>Windows с контейнерами Linux

Создать сертификат и настроить локальный компьютер:

```console
dotnet dev-certs https -ep %USERPROFILE%\.aspnet\https\aspnetapp.pfx -p { password here }
dotnet dev-certs https --trust
```

В предыдущих командах замените `{ password here }` паролем.

Запустите образ контейнера с ASP.NET Core, настроенным для HTTPS:

```console
docker pull mcr.microsoft.com/dotnet/core/samples:aspnetapp
docker run --rm -it -p 8000:80 -p 8001:443 -e ASPNETCORE_URLS="https://+;http://+" -e ASPNETCORE_HTTPS_PORT=8001 -e ASPNETCORE_Kestrel__Certificates__Default__Password="password" -e ASPNETCORE_Kestrel__Certificates__Default__Path=/https/aspnetapp.pfx -v %USERPROFILE%\.aspnet\https:/https/ mcr.microsoft.com/dotnet/core/samples:aspnetapp
```

Пароль должен совпадать с паролем, используемым для сертификата.

### <a name="macos-or-linux"></a>macOS или Linux

Создать сертификат и настроить локальный компьютер:

```console
dotnet dev-certs https -ep ${HOME}/.aspnet/https/aspnetapp.pfx -p { password here }
dotnet dev-certs https --trust
```

`dotnet dev-certs https --trust`поддерживается только в macOS и Windows. Необходимо доверять сертификатам в Linux так, как это поддерживается вашим дистрибутив. Вероятно, вам нужно доверять сертификату в браузере.

В предыдущих командах замените `{ password here }` паролем.

Запустите образ контейнера с ASP.NET Core, настроенным для HTTPS:

```console
docker pull mcr.microsoft.com/dotnet/core/samples:aspnetapp
docker run --rm -it -p 8000:80 -p 8001:443 -e ASPNETCORE_URLS="https://+;http://+" -e ASPNETCORE_HTTPS_PORT=8001 -e ASPNETCORE_Kestrel__Certificates__Default__Password="password" -e ASPNETCORE_Kestrel__Certificates__Default__Path=/https/aspnetapp.pfx -v ${HOME}/.aspnet/https:/https/ mcr.microsoft.com/dotnet/core/samples:aspnetapp
```

Пароль должен совпадать с паролем, используемым для сертификата.

### <a name="windows-using-windows-containers"></a>Windows, использующих контейнеры Windows

Создать сертификат и настроить локальный компьютер:

```console
dotnet dev-certs https -ep %USERPROFILE%\.aspnet\https\aspnetapp.pfx -p { password here }
dotnet dev-certs https --trust
```

В предыдущих командах замените `{ password here }` паролем.

Запустите образ контейнера с ASP.NET Core, настроенным для HTTPS:

```console
docker pull mcr.microsoft.com/dotnet/core/samples:aspnetapp
docker run --rm -it -p 8000:80 -p 8001:443 -e ASPNETCORE_URLS="https://+;http://+" -e ASPNETCORE_HTTPS_PORT=8001 -e ASPNETCORE_Kestrel__Certificates__Default__Password="password" -e ASPNETCORE_Kestrel__Certificates__Default__Path=\https\aspnetapp.pfx -v %USERPROFILE%\.aspnet\https:C:\https\ mcr.microsoft.com/dotnet/core/samples:aspnetapp
```

Пароль должен совпадать с паролем, используемым для сертификата.