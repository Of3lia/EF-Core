[← EF Core](/README.md)  

# Razor Pages с EF Core

## [Создание проекта веб-приложения](https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-rp/intro?view=aspnetcore-3.1&tabs=visual-studio)
<p align="center">
   <a  href="https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-rp/intro?view=aspnetcore-3.1&tabs=visual-studio" target="_blank" >
  <img src="https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-rp/intro/_static/data-model-diagram.png?view=aspnetcore-3.1" width="400" alt="">
   </a>
</p>

* Модель данных. Создание базы данных. Заполнение базы данных.
Метод `EnsureCreated` создает пустую базу данных, поэтому он размещается в методе инициализации БД `DbInitializer.Initialize(context)`.  

* [EF Core CRUD](https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-rp/crud?view=aspnetcore-3.1)  
CREATE: обработка уязвимости [Чрезмерная передача данных](https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-rp/crud?view=aspnetcore-3.1#overposting), с помощью метода `TryUpdateModel` или модели представления (ViewModel) и метода ` entry.CurrentValues.SetValues(StudentVM);`  
EDIT: Если включать связанные данные не требуется, более эффективным будет метод `FindAsync`. [Состояния сущностей](https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-rp/crud?view=aspnetcore-3.1#entity-states).  
DELETE: [Обработка сбоя](https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-rp/crud?view=aspnetcore-3.1#update-the-delete-page) - операция удаления может завершиться сбоем из-за временных проблем с сетью.

* [Сортировка, фильтрация, разбиение на страницы](https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-rp/sort-filter-page?view=aspnetcore-3.1)  
Передача параметра сортировки в URL. [Добавление фильтрации](https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-rp/sort-filter-page?view=aspnetcore-3.1#add-filtering) - производительность IQueryable и IEnumerable.  
[Разбиения по страницам](https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-rp/sort-filter-page?view=aspnetcore-3.1#add-paging). Создание класса PaginatedList.  
[Добавление группирования](https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-rp/sort-filter-page?view=aspnetcore-3.1#add-grouping)  

## [Миграции](https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-rp/migrations?view=aspnetcore-3.1&tabs=visual-studio)
Ранее в методе DbInitializer.Initialize мы сначала использовали метод  EnsureCreated, который создает новую базу, каждый раз, когда меняется модель, затем мы добавляли данные в новую БД. Этот подход к обеспечению синхронизации базы данных с моделью данных хорошо работает до развертывания приложения в рабочей среде.  
Приложение, **выполняющееся в рабочей среде**, обычно содержит данные. Приложение не может запускаться с тестовой базой данных каждый раз при внесении изменений (например, при добавлении столбца). Функция миграций EF Core решает эту проблему, позволяя EF Core обновить схему базы данных вместо создания базы данных. 

⚠ Не вызывайте EnsureCreated() перед миграцией. EnsureCreated() обходит миграции, чтобы создать схему, что приводит к сбою мигграции.
Для этого в примере, первоначальная база удаляется, а метод EnsureCreated() комментируется в `DbInitializer.Initialize`

❶ После того, как вы определите начальную модель, можно переходить к созданию базы данных. Чтобы добавить первоначальную миграцию, используется команда PM `Add-Migration InitialCreate`.  
• После создания кода миграции проверьте, правильно ли он создан. Если потребуется, добавьте, удалите или измените любые операции для правильного применения изменений.

❷ Команда PM `Update-Database` создаст новую **пустую БД** (это займет некоторе время). При этом в БД будет создана [Таблица журнала миграции](https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-rp/migrations?view=aspnetcore-3.1&tabs=visual-studio#the-migrations-history-table)
 Таблица `__EFMigrationsHistory` используется для отслеживания миграций, которые были применены к базе данных, и она обязательна.  
 Кроме файлов миграции, будет создан файл [Моментальный снимок модели данных](https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-rp/migrations?view=aspnetcore-3.1&tabs=visual-studio#the-data-model-snapshot) `Migrations/SchoolContextModelSnapshot.cs`. При добавлении миграции EF определяет, что именно изменилось, сравнивая текущую модель данных с файлом моментального снимка.

❸ Запустите приложение и убедитесь в том, что база данных заполняется. Это обеспечивает вызов метода `DbInitializer.Initialize` при старте приложения, в файле Program.cs

Дополнительныe сведения:  
📘 [Управление схемами баз данных: Миграции](https://docs.microsoft.com/ru-ru/ef/core/managing-schemas/migrations/?tabs=dotnet-core-cli)

## [Создание сложной модели данных](https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-rp/complex-data-model?view=aspnetcore-3.1&tabs=visual-studio)

<p align="center">
   <a  href="https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-rp/complex-data-model?view=aspnetcore-3.1&tabs=visual-studio" target="_blank" >
  <img src="https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-rp/complex-data-model/_static/diagram.png?view=aspnetcore-3.1" width="400" alt="">
   </a>
</p>

Использование атрибутов данных, требуются классы `using System.ComponentModel.DataAnnotations` и
`using System.ComponentModel.DataAnnotations.Schema`.  
Атрибут `Column` - сопоставляются с базой данных, а атриут `Display` с отображением на странице. Запуск миграции после присвоения атрибутов.  
[Атрибут DatabaseGenerated](https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-rp/complex-data-model?view=aspnetcore-3.1&tabs=visual-studio#the-databasegenerated-attribute) - указывает, что первичный ключ предоставляется приложением, а не создается базой данных.  
🟡 По соглашению EF Core разрешает **каскадное удаление** для внешних ключей, не допускающих значение null, и связей "многие ко многим". Это [поведение по умолчанию](https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-rp/complex-data-model?view=aspnetcore-3.1&tabs=visual-studio#foreign-key-and-navigation-properties) может привести к циклическим правилам каскадного удаления. Такие правила вызывают исключение при добавлении миграции.

🔵 [Составной ключ](https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-rp/complex-data-model?view=aspnetcore-3.1&tabs=visual-studio#composite-key). Единственным способом указать составные первичные ключи для EF Core является [текучий API](https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-rp/complex-data-model?view=aspnetcore-3.1&tabs=visual-studio#fluent-api-alternative-to-attributes), в методе SchoolContext.OnModelCreating:
```
modelBuilder.Entity<CourseAssignment>()
                .HasKey(c => new { c.CourseID, c.InstructorID });
```
После изменения модели и добавления новых сущностей примените миграцию `Add-Migration ComplexDataModel`. Если сразу выполнить команду `Update-Database` то возникнет исключение:
```
The ALTER TABLE statement conflicted with the FOREIGN KEY constraint "FK_Course_Department_DepartmentID". The conflict occurred in database "SchoolContext", table "dbo.Department", column 'DepartmentID'.
```
Это возникает из-за несогласованности текущих данных в БД с новой схемой данных. Нам надо изменить файл текущей миграции `{метка_времени}_ComplexDataModel.cs` и добавить код, вставляющий временные данные, для согласования связей.
[Устранение ограничений внешнего ключа](https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-rp/complex-data-model?view=aspnetcore-3.1&tabs=visual-studio#apply-the-migration)  
Теперь мы вновь делаем `Update-Database`, запускается миграция, которая уже добавляет временные данные. Осталось заполнить данными вновь созданные сущности. Для данного примера пишется новая версия метода заполнителя, а база удаляется. Тогда при запуске приложения база будет вновь создана и заполнена новыми данными.


## [Чтение и обновление связанных данных](https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-rp/read-related-data?view=aspnetcore-3.1&tabs=visual-studio)
[Безотложная загрузка](https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-rp/read-related-data?view=aspnetcore-3.1&tabs=visual-studio#eager-explicit-and-lazy-loading). Безотложной является загрузка, когда запрос для одного типа сущности также загружает связанные сущности с помощью методов Include и ThenInclude.
 <img src="https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-rp/read-related-data/_static/eager-loading.png?view=aspnetcore-3.1" width="600" alt="">
 
**Явная загрузка**. При первом чтении сущности связанные данные не извлекаются. Нужно написать код, извлекающий связанные данные, когда они необходимы. Явная загрузка с отдельными запросами приводит к отправке нескольких запросов к базе данных. При явной загрузке код указывает, какие свойства навигации нужно загрузить. Для выполнения явной загрузки используется метод Load. Например:
<img src="https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-rp/read-related-data/_static/explicit-loading.png?view=aspnetcore-3.1" width="600" alt="">

Дополнительныe сведения:  
📘 [Загрузка связанных данных](https://docs.microsoft.com/ru-ru/ef/core/querying/related-data)

⚠ При формировании шаблонов сущностей, обратите внимание на пространство имен для шаблонов. Должно быть включено пространство верхнего уровня `ContosoUniversity.Pages.{имя_шаблона}`, например для Преподавателей должно быть `namespace ContosoUniversity.Pages.Instructors`. 

## Конфликты параллелизма
⚗ продолжение следует




