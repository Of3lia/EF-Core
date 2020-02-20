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

🔵 После того, как вы определите начальную модель, можно переходить к созданию базы данных. Чтобы добавить первоначальную миграцию, используется команда PM `Add-Migration InitialCreate`.  
• После создания кода миграции проверьте, правильно ли он создан. Если потребуется, добавьте, удалите или измените любые операции для правильного применения изменений.

🔵 Команда PM `Update-Database` создаст новую **пустую БД** (это займет некоторе время). При этом в БД будет создана [Таблица журнала миграции](https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-rp/migrations?view=aspnetcore-3.1&tabs=visual-studio#the-migrations-history-table)
 Таблица `__EFMigrationsHistory` используется для отслеживания миграций, которые были применены к базе данных, и она обязательна.  
 Кроме файлов миграции, будет создан файл [Моментальный снимок модели данных](https://docs.microsoft.com/ru-ru/aspnet/core/data/ef-rp/migrations?view=aspnetcore-3.1&tabs=visual-studio#the-data-model-snapshot) `Migrations/SchoolContextModelSnapshot.cs`. При добавлении миграции EF определяет, что именно изменилось, сравнивая текущую модель данных с файлом моментального снимка.

🔵 Запустите приложение и убедитесь в том, что база данных заполняется. Это обеспечивает вызов метода `DbInitializer.Initialize` при старте приложения, в файле Program.cs

Дополнительныe сведения:  
📘 [Управление схемами баз данных: Миграции](https://docs.microsoft.com/ru-ru/ef/core/managing-schemas/migrations/?tabs=dotnet-core-cli)
