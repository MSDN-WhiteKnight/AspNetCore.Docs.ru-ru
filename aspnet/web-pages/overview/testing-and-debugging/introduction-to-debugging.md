---
uid: web-pages/overview/testing-and-debugging/introduction-to-debugging
title: "Общие сведения об отладке ASP.NET Web Pages (Razor) узлов | Документы Microsoft"
author: tfitzmac
description: "Отладка — это процесс поиска и исправления ошибок в коде страницы. В этой главе показано некоторые средства и методы, которые можно использовать для отладки и analyz..."
ms.author: aspnetcontent
manager: wpickett
ms.date: 02/20/2014
ms.topic: article
ms.assetid: 68de4326-7611-4b9b-b5f6-79b7adc3069f
ms.technology: dotnet-webpages
ms.prod: .net-framework
msc.legacyurl: /web-pages/overview/testing-and-debugging/introduction-to-debugging
msc.type: authoredcontent
ms.openlocfilehash: 2bc1f096540d17095ef760eed67b458fcd4e1372
ms.sourcegitcommit: 9a9483aceb34591c97451997036a9120c3fe2baf
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/10/2017
---
<a name="introduction-to-debugging-aspnet-web-pages-razor-sites"></a>Общие сведения об отладке ASP.NET Web Pages (Razor) узлов
====================
по [Tom FitzMacken](https://github.com/tfitzmac)

> В этой статье описываются различные способы отладки страницы на веб-сайт ASP.NET Web Pages (Razor). Отладка — это процесс поиска и исправления ошибок в коде страницы.
> 
> **Что вы узнаете следующее.** 
> 
> - Как отобразить сведения для анализа и отладки страниц.
> - Как использовать отладку средств в Visual Studio.
>   
> 
> Существуют следующие функции ASP.NET, представленные в статье:
> 
> - `ServerInfo` Вспомогательные.
> - `ObjectInfo`Вспомогательный метод.
>   
> 
> ## <a name="software-versions"></a>Версии программного обеспечения
> 
> 
> - Веб-страниц ASP.NET (Razor) 3
> - Visual Studio 2013
>   
> 
> Этот учебник также работает с веб-страницы ASP.NET 2. Можно использовать WebMatrix 3, но не поддерживается встроенный отладчик.


Во избежание их в первую очередь является важным аспектом Устранение ошибок и проблем в коде. Это можно сделать, поместив фрагменты кода, которые могут привести к ошибкам в `try/catch` блоков. Для получения дополнительной информации обратитесь к разделу об обработке ошибок в [введение в ASP.NET веб-программирование с использованием синтаксиса Razor](https://go.microsoft.com/fwlink/?LinkId=202890).

`ServerInfo` Помощник — это средство диагностики, содержатся обзорные сведения о среде веб-сервера, на котором размещается на странице. Он также содержит сведения о запросе HTTP, который отправляется, когда браузер запрашивает страницу. `ServerInfo` Вспомогательный объект отображает идентификатор текущего пользователя, тип браузера, который выполняет запрос, и т. д. Подобные сведения могут помочь устранить распространенные проблемы.

1. Создать новый веб-страницу с именем *ServerInfo.cshtml*.
2. В конце страницы сразу перед закрытием `</body>` , добавьте `@ServerInfo.GetHtml()`:

    [!code-cshtml[Main](introduction-to-debugging/samples/sample1.cshtml)]

    Можно добавить `ServerInfo` код на странице. Но его добавления в конце будет хранить свои выходные данные отдельно от вашей содержимое страницы, что делает его более удобным для чтения.

    > [!NOTE] 
    > 
    > **Важные** код диагностики следует удалить из веб-страниц, перед перемещением веб-страницы на рабочем сервере. Это относится к `ServerInfo` поддержки, а также других методах диагностики в этой статье, связанных с добавлением кода на страницу. Вы не хотите посетителей веб-сайта для отображения сведений об имени сервера, имена пользователей, пути на сервере, а подобной подробной информации, так как этот тип данных может быть полезно для людей с злоумышленников.
3. Сохраните страницу и запустите его в браузере.

    ![Отладка 1](introduction-to-debugging/_static/image1.jpg)

    `ServerInfo` Вспомогательный объект отображает четыре таблицы сведения на странице:

    - Конфигурация сервера. Этот раздел содержит сведения о веб-сервере, включая имя компьютера, версию ASP.NET, у вас, имя домена и время сервера.
    - Переменные сервера ASP.NET. Этот раздел содержит сведения о многих сведения протокола HTTP (переменных, вызванной HTTP) и значения, которые являются частью каждого запроса веб-страницы.
    - Сведения о времени выполнения HTTP. В этом разделе содержатся сведения об этом версию Microsoft .NET Framework, на которой запущена веб-страницы, путь, сведения о кэше и т. д. (Как вы узнали из [введение в ASP.NET веб-программирование с использованием синтаксиса Razor](https://go.microsoft.com/fwlink/?LinkId=202890), веб-страниц ASP.NET с помощью синтаксиса построены на технологию веб-сервера Microsoft ASP.NET, который сам лежит широко программного обеспечения является Razor библиотеки разработки вызывается .NET Framework).
    - Переменные среды. Этот раздел предоставляет список все локальные переменные среды и их значения на веб-сервере.

    Полное описание всех данных сервера и запроса выходит за рамки данной статьи, но можно видеть, что `ServerInfo` вспомогательный возвращает большой объем диагностических данных. Дополнительные сведения о значениях, `ServerInfo` возвращает, в разделе [распознается переменные среды](https://technet.microsoft.com/en-us/library/dd560744(WS.10).aspx) на веб-сайте Microsoft TechNet и [переменные сервера IIS](https://msdn.microsoft.com/en-us/library/ms524602(VS.90).aspx) на веб-сайте MSDN.

## <a name="embedding-output-expressions-to-display-page-values"></a>Внедренные выражения выходных данных для отображения значений страницы

Другой способ узнать, что происходит в коде является внедрение выражений выходные данные на странице. Как вы знаете, значение переменной может выводить напрямую путем добавления примерно `@myVariable` или `@(subTotal * 12)` на страницу. Для отладки, можно поместить эти выражения выходные данные в стратегически важных местах кода. Это дает возможность просмотра значения ключевых переменных или результат вычислений, при запуске страницы. После завершения отладки, можно удалить из выражений или комментарий их. Эта процедура показывает типичный способ использования внедренных выражений для отладки на странице.

1. Создайте новую страницу WebMatrix, которая называется *OutputExpression.cshtml*.
2. Замените содержимое следующим страницы:

    [!code-html[Main](introduction-to-debugging/samples/sample2.html)]

    В этом примере `switch` инструкцию, чтобы проверить значение `weekday` переменной и затем отображение, это сообщение разные выходные данные в зависимости от того, какой день недели. В примере `if` блок в первый блок кода произвольно изменения день недели, добавив один день к значению текущего дня недели. Это ошибка для наглядности.
3. Сохраните страницу и запустите его в браузере.

    На странице отображается сообщение для Неверный день недели. Какой бы день недели фактически это, вы увидите сообщение об ошибке за один день в более поздней версии. Несмотря на то, что в этом случае вы знаете, почему сообщение отключена (потому, что код намеренно устанавливает значение неправильное дня), на самом деле часто бывает сложно знать, где дела идут не так с кодом. Для отладки, необходимо узнать, выполняемых до значения ключевых объектов и переменных, такие как `weekday`.
4. Добавить выходные данные выражения, вставив `@weekday` как показано в двух местах, обозначенном в комментариях к коду. Эти выражения выходные данные отображаются значения переменной в этот момент выполнения кода.

    [!code-csharp[Main](introduction-to-debugging/samples/sample3.cs?highlight=2-3,15-16)]
5. Сохраните и запустите страницу в браузере.

    На странице отображаются реальные день недели, сначала, то обновленный день недели, полученный в результате добавления один день, а затем полученное в результате сообщение из `switch` инструкции. Выходные данные из двух выражений переменной (`@weekday`) не содержит пробелов между днями, так как не удалось добавить любой код HTML `<p>` тегов в выходных данных; содержатся выражения, только для тестирования.

    ![Отладка 2](introduction-to-debugging/_static/image2.jpg)

    Теперь можно увидеть, где — ошибка. При первом вызове `weekday` переменной в коде, он показывает правильный день. При отображении его во второй раз, после `if` блока в коде, отключен на один день. Чтобы знать, что-то произошло между первого и второго вида переменной дня недели. Если бы это был настоящую ошибку, такой подход помогает сузить расположение кода, который вызывает проблему.
6. Исправьте код на странице путем удаления два выходных выражений, которые вы добавили и удаление код, который изменяет день недели. Оставшиеся, завершения блока кода выглядит как в следующем примере:

    [!code-cshtml[Main](introduction-to-debugging/samples/sample4.cshtml)]
7. Запустите страницу в браузере. Это время вы видите правильное сообщение, отображаемое для конкретного дня недели.

## <a name="using-the-objectinfo-helper-to-display-object-values"></a>Использование вспомогательного метода ObjectInfo для отображения значений объектов

`ObjectInfo` Вспомогательный объект отображает тип и значение каждого объекта, передаваемого. Его можно использовать для просмотра значения переменных и объектов в коде (как это было сделано с выражениями выходные данные в предыдущем примере), а также можно увидеть информацию об объекте тип данных.

1. Откройте файл с именем *OutputExpression.cshtml* , созданного ранее.
2. Замените весь код на странице следующий блок кода:

    [!code-html[Main](introduction-to-debugging/samples/sample5.html)]
3. Сохраните и запустите страницу в браузере.

    ![Отладка 4](introduction-to-debugging/_static/image3.jpg)

    В этом примере `ObjectInfo` вспомогательный объект отображает два элемента:

    - Тип. Для первой переменной, тип — `DayOfWeek`. Второй переменной, тип — `String`.
    - Значение. Таким образом, так как значение переменной приветствие уже отображаются на странице, значение отображается снова при передаче переменной в `ObjectInfo`.

    Для более сложных объектов `ObjectInfo` вспомогательный можно отобразить дополнительные сведения о &#8212; по сути, он может отображать типы и значения всех свойств объекта.

## <a name="using-debugging-tools-in-visual-studio"></a>С помощью средства отладки в Visual Studio

Для более комплексную отладку, используйте Visual Studio 2013 или бесплатную [Visual Studio Express 2013 для Web](https://www.visualstudio.com/downloads/download-visual-studio-vs#d-2013-express). С помощью Visual Studio можно установить точку останова в коде, в строке, на который требуется проверить.

![Набор точек останова](introduction-to-debugging/_static/image1.png)

При тестировании веб-сайта, выполнение кода останавливается в точке останова.

![При достижении точки останова](introduction-to-debugging/_static/image2.png)

Можно просмотреть текущие значения переменных и пошаговое выполнение кода строке.

![просмотреть значения](introduction-to-debugging/_static/image3.png)

Сведения об использовании встроенный отладчик в Visual Studio для отладки страниц ASP.NET Razor см. в разделе [программирование веб-страниц ASP.NET (Razor) с помощью Visual Studio](https://go.microsoft.com/fwlink/?LinkId=205854).

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Программирование веб-страниц ASP.NET (Razor) с помощью Visual Studio](https://go.microsoft.com/fwlink/?LinkId=205854)
- [Переменные сервера IIS](https://msdn.microsoft.com/en-us/library/ms524602(VS.90).aspx) (MSDN)
- [Распознается переменные среды](https://technet.microsoft.com/en-us/library/dd560744(WS.10).aspx) (TechNet)