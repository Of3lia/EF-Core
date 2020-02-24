[← EF Core](/README.md)  

# EF Core с MVC
[Учебник по работе с ASP.NET Core 2.0 MVC и EF Core](https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-mvc/?view=aspnetcore-3.1)

В проекте используется Net Core 3.1, что то приходится подправить, на основе руководства по EF Core с Razor Pages.  
Классы модели данных в точности такие же, как и для Razor Pages.

При добавлении класса SchoolContext требуется добавить в приложение несколько пакетов с помощью NuGet:
ссылку на общий пакет Microsoft.EntityFrameworkCore -- не добавлять.

Добавить те же пакеты, что и в приложении `EF Core с Razor Pages`

<p align="center"> 
  <img src="/ContosoUniversityMVC/wwwroot/img/prtsc/addNuget.png" width="400" alt="Установка пакетов Nuget для EF Core 3.1 с MVC">  
</p>

### [Реализация функциональности CRUD](https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-mvc/sort-filter-page?view=aspnetcore-3.1)
* Настройка страницы сведений: представление Details для отображения списка Enrollments;  
* Обновление страницы Create: настройка обработчика сбоев, защита от чрезмерной передачи данных;  
* Обновление страницы редактирования: настройка HttpPost Edit, защита от чрезмерной передачи данных, вариант с предварительным чтением данных. Альтернативный код метода HttpPost Edit: создание и подключение.  
* Обновление страницы удаления: обработка ошибок, изменение метода GET для отображение ошибки сохранения. Подход с предварительным чтением для метода HttpPost Delete.  
* Управление БД: закрытие подключений. Обработка транзакций: По умолчанию платформа Entity Framework реализует транзакции неявно. Если нужен дополнительный контроль, то см. в разделе [Транзакции](https://docs.microsoft.com/ru-ru/ef/core/saving/transactions).  
* Отключение отслеживания запросов: [метод `.AsNoTracking()`](https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-mvc/crud?view=aspnetcore-3.1#no-tracking-queries)

### [Cортировка, фильтрация и разбиение на страницы](https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-mvc/sort-filter-page?view=aspnetcore-3.1)
<p align="center">
   <a  href="https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-mvc/sort-filter-page?view=aspnetcore-3.1" target="_blank" >
  <img src="https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-mvc/sort-filter-page/_static/paging.png?view=aspnetcore-3.1" width="400" alt="">
   </a>
</p>

* Добавление ссылок для сортировки столбцов  
* Добавление поля поиска. [Особенности работы при наличии слоя репозитария](https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-mvc/sort-filter-page?view=aspnetcore-3.1#add-a-search-box). Особенности оптимизации SQL сервера при учете регистра строк.  
* Разбиение по страницам. Учет фильтрации и сортировки при разбиении по страницам  
* Создание страницы сведений. Группировка и расчеты.

### [Функции миграций](https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-mvc/migrations?view=aspnetcore-3.1)
* Создание первоначальной миграции. Чтобы избежать ошибок, воспользуйтесь командами PowerShell как в руководстве [Razor Pages EF](https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-rp/migrations?view=aspnetcore-3.1&tabs=visual-studio#create-an-initial-migration). 
Первая команда `Add-Migration InitialCreate` создаст файлы миграции, в вторая `Update-Database` создаст новую БД `SchoolContext2`  
* Далее вернитесь к исходному руководству [Обзор методов Up и Down](https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-mvc/migrations?view=aspnetcore-3.1#examine-up-and-down-methods). Параметр имени миграции (в примере это "InitialCreate") используется в качестве имени файла и может быть любым.  
* Из Razor Pages EF: [Таблица журнала миграции в базе SQL](https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-rp/migrations?view=aspnetcore-3.1&tabs=visual-studio#the-migrations-history-table), [Моментальный снимок модели данных](https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-rp/migrations?view=aspnetcore-3.1&tabs=visual-studio#the-data-model-snapshot)   
* Удаление миграции. Сначала надо откатить базу до предыдущей миграции, я сделал это в PowerShell командой [Update-Database -Migration 0](https://docs.microsoft.com/ru-ru/ef/core/miscellaneous/cli/powershell#update-database) -- таблицы, созданные предыдущей миграцией, удаены из БД. Для удаления файлов миграции из проекта я выполнил команду  [Remove-Migration](https://docs.microsoft.com/ru-ru/ef/core/miscellaneous/cli/powershell#remove-migration)

### [Создание сложной модели данных](https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-mvc/complex-data-model?view=aspnetcore-3.1)
<p align="center">
   <a  href="https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-mvc/complex-data-model?view=aspnetcore-3.1" target="_blank" >
  <img src="https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-mvc/complex-data-model/_static/diagram.png?view=aspnetcore-3.1" width="400" alt="">
   </a>
</p>
* Настройка модели данных с помощью атрибутов.

Если вы не делали миграции для БД, то сначала разберитесь с миграциями. Придется вернутся к первоначальным данным, которые задаются в классе DbInitializer (namespace ContosoUniversityMVC.Data).  
Итак: Обнулите все миграции `Update-Database -Migration 0`, откройте БД и удалите оставшиеся таблицы, сначала таблицу связей Enrollment потом Course и Student.  
Если вы уже редактировали атрибуты, то закоментируйте все атрибуты и создайте первоначальную миграцию `Add-Migration InitialCreate`, примените миграцию к БД `Update-Database` - будут созданы пустые таблицы. Запустите приложение, и таблицы заполнятся из класса DbInitializer.  
Добавьте атрубуты к классу Student и создайте миграцию `Add-Migration MaxLengthOnNames` -- убедитесь, что миграция нацелена на обновление таблиц. Обновите базу `Update-Database` и проверьте изменения в схеме таблицы Student.  
* [Связи "многие ко многим"](https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-mvc/complex-data-model?view=aspnetcore-3.1#many-to-many-relationships)  
Составной ключ - добавляется в классе SchoolContext (namespace ContosoUniversityMVC.Data).  
* [Добавление миграции](https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-mvc/complex-data-model?view=aspnetcore-3.1#add-a-migration)  Создание данных-заглушек в базу данных для соблюдения ограничений внешнего ключа. 
Поскольку создан новый файл первичных данных DbInitializer, то для тестирования используем новую БД.

### [Чтение связанных данных](https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-mvc/read-related-data?view=aspnetcore-3.1#prerequisites)
<p align="center">
   <a  href="https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-mvc/read-related-data?view=aspnetcore-3.1#prerequisites" target="_blank" >
  <img src="https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-mvc/read-related-data/_static/instructors-index.png?view=aspnetcore-3.1" width="400" alt="">
   </a>
</p>

* Загрузка связанных данных. Безотложная загрузка, Явная загрузка, Отложенная загрузка.  
На странице "Instructors" (Преподаватели) отображаются данные из трех различных таблиц, для этого создается отдельный класс модели отображения.

### [Обновление связанных данных](https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-mvc/update-related-data?view=aspnetcore-3.1)
<p align="center">
   <a  href="https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-mvc/update-related-data?view=aspnetcore-3.1" target="_blank" >
  <img src="https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-mvc/update-related-data/_static/instructor-edit-courses.png?view=aspnetcore-3.1" width="400" alt="">
   </a>
</p>

* Добавление списка выбора на основе SelectList  
* Добавление .AsNoTrackin в методы Details и Delete   
* Настройка страниц Курсов. Добавление страницы редактирования данных о преподавателях. Добавление курсов на страницу редактирования. Обновление страницы удаления.  
* Добавление расположения кабинета и курсов на страницу создания  

### [Обработка параллелизма](https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-mvc/concurrency?view=aspnetcore-3.1)

* Пессимистичный параллелизм (блокировка) - EF Core не поддерживает,т.к. требует много ресурсов, используется оптимистическая блокировка. Варианты: отслживание изменений записи разными пользователями; победа клиента - сохранение последнего внесенного изменения; победа хранилища - если запись изменена другим пользователем (два пользователя одновременно редактируют запись), то будет сообщение об ошибке.  
Данный метод гарантирует, что никакие изменения не перезаписываются без оповещения пользователя о случившемся - он и используется в примере.  
* Обнаружение конфликтов параллелизма - обработки исключений `DbConcurrencyException` - требует настройки БД и модели. В БД включаетя столбец отслеживания `rowversion`, он изменяется при каждом обновлении, и включается в запрос UPDATE или DELETE как часть предложения WHERE.  
* Создание представлений и контроллера кафедр. Обновление страницы удаления. Обновление представлений Details и Create.

### [Реализация наследования](https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-mvc/inheritance?view=aspnetcore-3.1)
<p align="center">
   <a  href="https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-mvc/inheritance?view=aspnetcore-3.1" target="_blank" >
  <img src="https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-mvc/inheritance/_static/inheritance.png?view=aspnetcore-3.1" width="400" alt="">
   </a>
</p>

Изменения вносятся в коде, а не на веб-страницах, и автоматически отражаются в базе данных. Платформа Entity Framework поддерживает только модель наследования "одна таблица на иерархию". Необходимо создать класс Person, изменить классы Instructor и Student так, чтобы они были производными от класса Person, добавить новый класс в DbContext, после чего создать миграцию.

⚠ ВНИМАНИЕ! [Опечатка в документации](https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-mvc/inheritance?view=aspnetcore-3.1#add-person-to-the-model):  
из класса SchoolContext в методе `OnModelCreating(ModelBuilder modelBuilder)` закомментируйте 2 строки с классами Student и Instructor, надо оставить только одну таблицу Person вместо двух прежних. При миграции будет предупреждение о потере данных. 

### [Сведения о сложных сценариях для ASP.NET MVC с EF Core](https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-mvc/advanced?view=aspnetcore-3.1)

* Выполнение прямых SQL-запросов  
⚠ ВНИМАНИЕ! **Вызов запроса для получения сущностей**, [устаревший метод](https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-mvc/advanced?view=aspnetcore-3.1#call-a-query-to-return-entities):  
Меод `.FromSql(query, id)` не поддерживается, надо использовать `.FromSqlRaw(query, id)`  
* Вызов запроса для получения других типов  
* [Вызов запроса на обновление](https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-mvc/advanced?view=aspnetcore-3.1#call-a-query-to-return-entities)  
⚠ ВНИМАНИЕ! [устаревший метод](https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-mvc/advanced?view=aspnetcore-3.1#call-a-query-to-return-entities):  
Меод `.ExecuteSqlCommandAsync` устарел, надо использовать `.ExecuteSqlRawAsync`   
⚠ ВНИМАНИЕ! При тестировании обновления, вы делаете обновление и переходите на страницу списка курсов, если вернуться назад, используя браузер, то можно ошибочно выполнить повторное обновление.  
* Изучение SQL-запросов -- возможность просмотреть фактические SQL-запросы, отправляемые в базу данных.  
* Создание уровня абстракции:  
✅ Платформа EF предусматривает функции для реализации разработки на основе тестирования, **не требующие написания кода репозитория**.  
* Автоматическое обнаружение изменений.  
✅ Если вы отслеживаете большое число сущностей и многократно вызываете один из методов DbContext.SaveChanges, DbContext.Entry, ChangeTracker.Entries  **в цикле** , вы сможете добиться заметного повышения производительности, отключив автоматическое обнаружение изменений с помощью свойства `ChangeTracker.AutoDetectChangesEnabled`.  
* [Использование динамических запросов LINQ для упрощения кода](https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-mvc/advanced?view=aspnetcore-3.1#use-dynamic-linq-to-simplify-code).  
✅ Метод `EF.Property` используется для указания **имени свойства** в виде строки


### Дополнительные ресурсы
📘 [документации по платформе Entity Framework Core.](https://docs.microsoft.com/ru-ru/ef/core/)  
<br /><br />
<p align="center">
  Практические консультации вы можете получить на наших <a  href="http://creativcode.ru/learn" target="_blank" >веб курсах в Сочи, Адлер</a>:<br /><br />
   <a  href="http://creativcode.ru/learn/webnet" target="_blank" >
  <img src="http://creativcode.ru/img/learn/net-backend.jpg" width="400" alt="">
   </a>
</p>



