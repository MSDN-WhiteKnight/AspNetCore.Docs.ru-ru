---
title: Временные поставщики защиты данных в ASP.NET Core
author: rick-anderson
description: Сведения о реализации ASP.NET Core временных поставщиков защиты данных.
ms.author: riande
ms.date: 10/14/2016
no-loc:
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: security/data-protection/implementation/key-storage-ephemeral
ms.openlocfilehash: 22a332230e15256dc33fd1d06f2da3ea8d34d3bc
ms.sourcegitcommit: 70e5f982c218db82aa54aa8b8d96b377cfc7283f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 05/04/2020
ms.locfileid: "82776894"
---
# <a name="ephemeral-data-protection-providers-in-aspnet-core"></a>Временные поставщики защиты данных в ASP.NET Core

<a name="data-protection-implementation-key-storage-ephemeral"></a>

Существуют сценарии, в которых приложению требуется пустячных `IDataProtectionProvider`. Например, разработчик может просто поэкспериментировать в односторонних консольном приложении, или само приложение является временным (в скрипте или проекте модульного теста). Для поддержки этих сценариев пакет [Microsoft. AspNetCore. Protection](https://www.nuget.org/packages/Microsoft.AspNetCore.DataProtection/) включает тип `EphemeralDataProtectionProvider`. Этот тип предоставляет базовую реализацию `IDataProtectionProvider` , репозиторий ключей которого хранится исключительно в памяти и не записывается в резервное хранилище.

Каждый экземпляр `EphemeralDataProtectionProvider` использует собственный уникальный главный ключ. Таким образом, если `IDataProtector` в корне `EphemeralDataProtectionProvider` создается защищенная полезная нагрузка, то эти полезные данные могут быть незащищенными с помощью `IDataProtector` эквивалентной (с той же цепочкой [назначения](xref:security/data-protection/consumer-apis/purpose-strings#data-protection-consumer-apis-purposes) ), имеющей корень в том же `EphemeralDataProtectionProvider` экземпляре.

В следующем примере демонстрируется создание экземпляра `EphemeralDataProtectionProvider` и его использование для защиты и снятия защиты данных.

```csharp
using System;
using Microsoft.AspNetCore.DataProtection;

public class Program
{
    public static void Main(string[] args)
    {
        const string purpose = "Ephemeral.App.v1";

        // create an ephemeral provider and demonstrate that it can round-trip a payload
        var provider = new EphemeralDataProtectionProvider();
        var protector = provider.CreateProtector(purpose);
        Console.Write("Enter input: ");
        string input = Console.ReadLine();

        // protect the payload
        string protectedPayload = protector.Protect(input);
        Console.WriteLine($"Protect returned: {protectedPayload}");

        // unprotect the payload
        string unprotectedPayload = protector.Unprotect(protectedPayload);
        Console.WriteLine($"Unprotect returned: {unprotectedPayload}");

        // if I create a new ephemeral provider, it won't be able to unprotect existing
        // payloads, even if I specify the same purpose
        provider = new EphemeralDataProtectionProvider();
        protector = provider.CreateProtector(purpose);
        unprotectedPayload = protector.Unprotect(protectedPayload); // THROWS
    }
}

/*
* SAMPLE OUTPUT
*
* Enter input: Hello!
* Protect returned: CfDJ8AAAAAAAAAAAAAAAAAAAAA...uGoxWLjGKtm1SkNACQ
* Unprotect returned: Hello!
* << throws CryptographicException >>
*/
```
