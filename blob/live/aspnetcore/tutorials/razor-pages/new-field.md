---
title: "Добавление нового поля на страницу Razor"
author: rick-anderson
description: "Демонстрирует, как добавить новое поле на страницу Razor с помощью Entity Framework Core"
keywords: "ASP.NET Core,Entity Framework Core,миграция"
ms.author: riande
manager: wpickett
ms.date: 08/07/2017
ms.topic: get-started-article
ms.technology: aspnet
ms.prod: aspnet-core
uid: tutorials/razor-pages/new-field
ms.openlocfilehash: 128b69513976a56104524bb803f2b8cb1daf1967
ms.sourcegitcommit: 9a9483aceb34591c97451997036a9120c3fe2baf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/10/2017
---
# <a name="adding-a-new-field-to-a-razor-page"></a><span data-ttu-id="855d6-104">Добавление нового поля на страницу Razor</span><span class="sxs-lookup"><span data-stu-id="855d6-104">Adding a new field to a Razor Page</span></span>

<span data-ttu-id="855d6-105">Автор: [Рик Андерсон](https://twitter.com/RickAndMSFT) (Rick Anderson)</span><span class="sxs-lookup"><span data-stu-id="855d6-105">By [Rick Anderson](https://twitter.com/RickAndMSFT)</span></span>

<span data-ttu-id="855d6-106">В этом разделе вы будете использовать [Entity Framework](https://docs.microsoft.com/ef/core/get-started/aspnetcore/new-db) Code First Migrations для добавления нового поля к модели и переноса этого изменения в базу данных.</span><span class="sxs-lookup"><span data-stu-id="855d6-106">In this section you'll use [Entity Framework](https://docs.microsoft.com/ef/core/get-started/aspnetcore/new-db) Code First Migrations to add a new field to the model and migrate that change to the database.</span></span>

<span data-ttu-id="855d6-107">Если вы используете EF Code First для автоматического создания базы данных, Code First добавляет в нее таблицу, которая позволяет отслеживать синхронизацию схемы базы данных с классами модели, на основе которой она была создана.</span><span class="sxs-lookup"><span data-stu-id="855d6-107">When you use EF Code First to automatically create a database, Code First adds a table to the database to help track whether the schema of the database is in sync with the model classes it was generated from.</span></span> <span data-ttu-id="855d6-108">Если синхронизация нарушена, EF вызывает исключение.</span><span class="sxs-lookup"><span data-stu-id="855d6-108">If they aren't in sync, EF throws an exception.</span></span> <span data-ttu-id="855d6-109">Это позволяет упростить поиск проблем с согласованностью между базой данных и кодом.</span><span class="sxs-lookup"><span data-stu-id="855d6-109">This makes it easier to find inconsistent database/code issues.</span></span>

## <a name="adding-a-rating-property-to-the-movie-model"></a><span data-ttu-id="855d6-110">Добавление свойства Rating в модель Movie</span><span class="sxs-lookup"><span data-stu-id="855d6-110">Adding a Rating Property to the Movie Model</span></span>

<span data-ttu-id="855d6-111">Откройте файл *Models/Movie.cs* и добавьте свойство `Rating`:</span><span class="sxs-lookup"><span data-stu-id="855d6-111">Open the *Models/Movie.cs* file and add a `Rating` property:</span></span>

[!code-csharp[Main](razor-pages-start/sample/RazorPagesMovie/Models/MovieDateRating.cs?highlight=11&range=7-18)]

<span data-ttu-id="855d6-112">Постройте приложение (CTRL+SHIFT+B).</span><span class="sxs-lookup"><span data-stu-id="855d6-112">Build the app (Ctrl+Shift+B).</span></span>

<span data-ttu-id="855d6-113">Измените файл *Pages/Movies/Index.cshtml* и добавьте в него поле `Rating`:</span><span class="sxs-lookup"><span data-stu-id="855d6-113">Edit *Pages/Movies/Index.cshtml*, and add a `Rating` field:</span></span>

[!code-cshtml[Main](razor-pages-start/sample/RazorPagesMovie/Pages/Movies/Index.cshtml?highlight=40-42,61-63)]

<span data-ttu-id="855d6-114">Добавьте поле `Rating` на страницы "Delete" (Удаление) и "Details" (Сведения).</span><span class="sxs-lookup"><span data-stu-id="855d6-114">Add the `Rating` field to the Delete and Details pages.</span></span>

<span data-ttu-id="855d6-115">Обновите файл *Create.cshtml*, добавив в него поле `Rating`.</span><span class="sxs-lookup"><span data-stu-id="855d6-115">Update *Create.cshtml* with a `Rating` field.</span></span> <span data-ttu-id="855d6-116">Вы можете скопировать и вставить предыдущий элемент `<div>` и дождаться автоматического обновления полей с помощью IntelliSense.</span><span class="sxs-lookup"><span data-stu-id="855d6-116">You can copy/paste the previous `<div>` element and let intelliSense help you update the fields.</span></span> <span data-ttu-id="855d6-117">IntelliSense работает со [вспомогательными функциями тегов](xref:mvc/views/tag-helpers/intro).</span><span class="sxs-lookup"><span data-stu-id="855d6-117">IntelliSense works with [Tag Helpers](xref:mvc/views/tag-helpers/intro).</span></span>

![Разработчик ввел букву R в качестве значения атрибута asp-for во втором элементе label представления.](new-field/_static/cr.png)

<span data-ttu-id="855d6-121">В следующем коде показан файл *Create.cshtml* с полем `Rating`:</span><span class="sxs-lookup"><span data-stu-id="855d6-121">The following code shows *Create.cshtml* with a `Rating` field:</span></span>

[!code-cshtml[Main](razor-pages-start/sample/RazorPagesMovie/Pages/Movies/Create.cshtml?highlight=36-40)]

<span data-ttu-id="855d6-122">Добавьте поле `Rating` на страницу "Edit" (Редактирование).</span><span class="sxs-lookup"><span data-stu-id="855d6-122">Add the `Rating` field to the Edit Page.</span></span>

<span data-ttu-id="855d6-123">Для работы приложения необходимо обновить базу данных, включив в нее новое поле.</span><span class="sxs-lookup"><span data-stu-id="855d6-123">The app won't work until the DB is updated to include the new field.</span></span> <span data-ttu-id="855d6-124">Если запустить приложение сейчас, возникнет исключение `SqlException`:</span><span class="sxs-lookup"><span data-stu-id="855d6-124">If run now, the app throws a `SqlException`:</span></span>

```
SqlException: Invalid column name 'Rating'.
```

<span data-ttu-id="855d6-125">Эта ошибка связана с тем, что обновленный класс модели Movie отличается от схемы таблицы Movie в базе данных.</span><span class="sxs-lookup"><span data-stu-id="855d6-125">This error is caused by the updated Movie model class being different than the schema of the Movie table of the database.</span></span> <span data-ttu-id="855d6-126">(В таблице базы данных отсутствует столбец `Rating`.)</span><span class="sxs-lookup"><span data-stu-id="855d6-126">(There's no `Rating` column in the database table.)</span></span>

<span data-ttu-id="855d6-127">Устранить эту ошибку можно несколькими способами:</span><span class="sxs-lookup"><span data-stu-id="855d6-127">There are a few approaches to resolving the error:</span></span>

1. <span data-ttu-id="855d6-128">Можно с помощью Entity Framework автоматически удалить и повторно создать базу данных на основе новой схемы класса модели.</span><span class="sxs-lookup"><span data-stu-id="855d6-128">Have the Entity Framework automatically drop and re-create the database using  the new model class schema.</span></span> <span data-ttu-id="855d6-129">Этот подход удобен на ранних стадиях цикла разработки. В этом случае развитие модели и схемы базы данных осуществляется одновременно.</span><span class="sxs-lookup"><span data-stu-id="855d6-129">This approach is convenient early in the development cycle; it allows you to quickly evolve the model and database schema together.</span></span> <span data-ttu-id="855d6-130">Недостатком такого подхода является потеря существующих данных в базе.</span><span class="sxs-lookup"><span data-stu-id="855d6-130">The downside is that you lose existing data in the database.</span></span> <span data-ttu-id="855d6-131">В рабочей базе данных применять этот подход невозможно.</span><span class="sxs-lookup"><span data-stu-id="855d6-131">You don't want to use this approach on a production database!</span></span> <span data-ttu-id="855d6-132">При разработке приложения часто выполняется удаление базы данных при изменении схемы, для чего используется инициализатор для автоматического заполнения базы тестовыми данными.</span><span class="sxs-lookup"><span data-stu-id="855d6-132">Dropping the DB on schema changes and using an initializer to automatically seed the database with test data is often a productive way to develop an app.</span></span>

2. <span data-ttu-id="855d6-133">Можно явно изменить схему существующей базы данных в соответствии с новыми классами модели.</span><span class="sxs-lookup"><span data-stu-id="855d6-133">Explicitly modify the schema of the existing database so that it matches the model classes.</span></span> <span data-ttu-id="855d6-134">Преимущество такого подхода состоит в том, что сохраняются все данные.</span><span class="sxs-lookup"><span data-stu-id="855d6-134">The advantage of this approach is that you keep your data.</span></span> <span data-ttu-id="855d6-135">Это изменение можно выполнить как вручную, так и с помощью соответствующего скрипта базы данных.</span><span class="sxs-lookup"><span data-stu-id="855d6-135">You can make this change either manually or by creating a database change script.</span></span>

3. <span data-ttu-id="855d6-136">Можно обновить схему базы данных с помощью Code First Migrations.</span><span class="sxs-lookup"><span data-stu-id="855d6-136">Use Code First Migrations to update the database schema.</span></span>

<span data-ttu-id="855d6-137">В этом руководстве используется Code First Migrations.</span><span class="sxs-lookup"><span data-stu-id="855d6-137">For this tutorial, use Code First Migrations.</span></span>

<span data-ttu-id="855d6-138">Обновите класс `SeedData` так, чтобы он предоставлял значение нового столбца.</span><span class="sxs-lookup"><span data-stu-id="855d6-138">Update the `SeedData` class so that it provides a value for the new column.</span></span> <span data-ttu-id="855d6-139">Ниже показан пример изменения, которое необходимо выполнить для каждого блока `new Movie`.</span><span class="sxs-lookup"><span data-stu-id="855d6-139">A sample change is shown below, but you'll want to make this change for each `new Movie` block.</span></span>

[!code-csharp[Main](razor-pages-start/sample/RazorPagesMovie/Models/SeedDataRating.cs?name=snippet1&highlight=8)]

<span data-ttu-id="855d6-140">См. [готовый файл SeedData.cs](https://github.com/aspnet/Docs/blob/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie/Models/SeedDataRating.cs).</span><span class="sxs-lookup"><span data-stu-id="855d6-140">See the [completed SeedData.cs file](https://github.com/aspnet/Docs/blob/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie/Models/SeedDataRating.cs).</span></span>

<span data-ttu-id="855d6-141">Постройте решение.</span><span class="sxs-lookup"><span data-stu-id="855d6-141">Build the solution.</span></span>

<a name="pmc"></a> <span data-ttu-id="855d6-142">В меню **Сервис** последовательно выберите пункты **Диспетчер пакетов NuGet > Консоль диспетчера пакетов**.</span><span class="sxs-lookup"><span data-stu-id="855d6-142">From the **Tools** menu, select **NuGet Package Manager > Package Manager Console**.</span></span>
<span data-ttu-id="855d6-143">В PMC введите следующие команды:</span><span class="sxs-lookup"><span data-stu-id="855d6-143">In the PMC, enter the following commands:</span></span>

```powershell
Add-Migration Rating
Update-Database
```

<span data-ttu-id="855d6-144">Команда `Add-Migration` задает следующие инструкции для платформы:</span><span class="sxs-lookup"><span data-stu-id="855d6-144">The `Add-Migration` command tells the framework to:</span></span>

* <span data-ttu-id="855d6-145">Сравните модель `Movie` со схемой базы данных `Movie`.</span><span class="sxs-lookup"><span data-stu-id="855d6-145">Compare the `Movie` model with the `Movie` DB schema.</span></span>
* <span data-ttu-id="855d6-146">Создайте код для переноса схемы базы данных в новую модель.</span><span class="sxs-lookup"><span data-stu-id="855d6-146">Create code to migrate the DB schema to the new model.</span></span>

<span data-ttu-id="855d6-147">В качестве имени файла переноса используется произвольное имя "Rating".</span><span class="sxs-lookup"><span data-stu-id="855d6-147">The name "Rating" is arbitrary and is used to name the migration file.</span></span> <span data-ttu-id="855d6-148">Рекомендуется присваивать этому файлу понятное имя.</span><span class="sxs-lookup"><span data-stu-id="855d6-148">It's helpful to use a meaningful name for the migration file.</span></span>

<a name="ssox"></a> <span data-ttu-id="855d6-149">Если удалить все записи из базы данных, при инициализации она будет заполнена значениями и в нее будет включено поле `Rating`.</span><span class="sxs-lookup"><span data-stu-id="855d6-149">If you delete all the records in the DB, the initializer will seed the DB and include the `Rating` field.</span></span> <span data-ttu-id="855d6-150">Это можно сделать с помощью ссылок удаления в браузере или из [обозревателя объектов SQL Server](xref:tutorials/razor-pages/sql#ssox) (SSOX).</span><span class="sxs-lookup"><span data-stu-id="855d6-150">You can do this with the delete links in the browser or from [Sql Server Object Explorer](xref:tutorials/razor-pages/sql#ssox) (SSOX).</span></span> <span data-ttu-id="855d6-151">Удаление базы данных из SSOX:</span><span class="sxs-lookup"><span data-stu-id="855d6-151">To delete the database from SSOX:</span></span>

* <span data-ttu-id="855d6-152">Выберите базу данных в SSOX.</span><span class="sxs-lookup"><span data-stu-id="855d6-152">Select the database in SSOX.</span></span>
* <span data-ttu-id="855d6-153">Щелкните базу данных правой кнопкой мыши и выберите *Удалить*.</span><span class="sxs-lookup"><span data-stu-id="855d6-153">Right click on the database, and select *Delete*.</span></span>
* <span data-ttu-id="855d6-154">Выберите **Закрыть существующие соединения**.</span><span class="sxs-lookup"><span data-stu-id="855d6-154">Check **Close existing connections**.</span></span>
* <span data-ttu-id="855d6-155">Нажмите кнопку **ОК**.</span><span class="sxs-lookup"><span data-stu-id="855d6-155">Select **OK**.</span></span>
* <span data-ttu-id="855d6-156">Обновите базу данных в [PMC](xref:tutorials/razor-pages/new-field#pmc).</span><span class="sxs-lookup"><span data-stu-id="855d6-156">In the [PMC](xref:tutorials/razor-pages/new-field#pmc), update the database:</span></span>

  ```powershell
  Update-Database
  ```

<span data-ttu-id="855d6-157">Запустите приложение и проверьте возможность создания, редактирования и отображения фильмов с использованием поля `Rating`.</span><span class="sxs-lookup"><span data-stu-id="855d6-157">Run the app and verify you can create/edit/display movies with a `Rating` field.</span></span> <span data-ttu-id="855d6-158">Если база данных не заполнена начальными значениями, остановите IIS Express и затем запустите приложение.</span><span class="sxs-lookup"><span data-stu-id="855d6-158">If the database is not seeded, stop IIS Express, and then run the app.</span></span>

>[!div class="step-by-step"]
<span data-ttu-id="855d6-159">[Предыдущая тема — "Добавление поиска"](xref:tutorials/razor-pages/search)
[Следующая тема — "Добавление проверки"](xref:tutorials/razor-pages/validation)</span><span class="sxs-lookup"><span data-stu-id="855d6-159">[Previous: Adding Search](xref:tutorials/razor-pages/search)
[Next: Adding Validation](xref:tutorials/razor-pages/validation)</span></span>