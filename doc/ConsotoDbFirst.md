[← EF Core](/README.md)  

# EF Core Database First

В данном примере рассматривается использование расширение Visual Studio для создания классов модели из существующнй БД.  
Вы можете попробывать и стандартный подход - [Реконструирование](https://docs.microsoft.com/ru-ru/ef/core/managing-schemas/scaffolding) спомощью команды Диспечера пакетов (PMC) `Scaffold-DbContext`, не забудьте при этом указать также ключ `-DataAnnotations`, для формирования анатаций к данным. Подробнее можно посмотреть в видео ▶ [Working with an Existing Database [2 of 5]](https://channel9.msdn.com/Series/Entity-Framework-Core-101/Working-with-an-Existing-Database).  
В моём случае, это не сработало - возникала ошибка при подключении к БД. Предлагаемый ниже способ дал требуемый результат.  
Этот подход можно использовать для любых типов веб-приложений: Razor Pages, MVC, Консольное приложение.  
Как начать работать с консольным приложением, смотрите в этом видео: ▶ [Getting Started with Entity Framework Core [1 of 5]](https://channel9.msdn.com/Series/Entity-Framework-Core-101/Getting-Started-with-Entity-Framework-Core) (есть субтитры на русском)    

## Подготовка
* Установите расширение <a href="https://marketplace.visualstudio.com/items?itemName=ErikEJ.EFCorePowerTools">EF Core Power Tools <img src="https://erikej.gallerycdn.vsassets.io/extensions/erikej/efcorepowertools/2.4.0/1581168364918/Microsoft.VisualStudio.Services.Icons.Default" width="32" alt=""></a>  
* Установите компонент **редактор DGML**. Для этого запустите стандартный установщик Visual Stiudio, перейдите на вкладку **Отдельные компоненты**, и в строке поиска наберите `dgml`

<p align="center">
     <img src="/Images/dgml.jpg" width="600" alt="">  
</p>

## Создание проекта
* Создание копии БД.  
Для этого будем использовать [ранее созданныей проект ConsotoUniversity](EF-Core-Razor-Pages.md). В нем, в файле _appsettings.json_ в строке подключение изменим имя базы с на ConsotoDbFirst:
```
  "ConnectionStrings": {
    "SchoolContext": "Server=(localdb)\\mssqllocaldb;Database=ConsotoDbFirst;Trusted_Connection=True;MultipleActiveResultSets=true"
  }
```
и выполним в консоле диспечера пакетов (PMC) команду `Update-Database`. В созданнной БД удалим таблицу отслеживания миграций `__EFMigrationsHistory`. 

<p align="center">
     <img src="/Images/db-first.jpg" width="294" alt="">
</p>

* Создайте проект Razor Pages с названием `ConsotoDbFirst`  
Добавьте в проект необходимые пакеты NuGet:
```
Microsoft.EntityFrameworkCore.SqlServer
Microsoft.EntityFrameworkCore.Design
Microsoft.EntityFrameworkCore.Tools
```

## Реконструирование 
Ранее мы установили расширение EF Core Power Tools, теперь пора им воспользоваться. Шёлкаем правой кнопкой мыши на проекте, и в открывшемся списке выбираем: `EF Core Power Tools` затем `Reverse Engineer`:

<p align="center">
     <img src="/Images/ef-core-power-tools.jpg" width="700" alt="">  
</p>

В маленьком окошке нам предлагается добавить соединение с БД, нажимаем кнопку `Add...`

<p align="center">
     <img src="/Images/db-connection.jpg" width="552" alt="">  
</p>

Это стандартное окно подключения к БД, выбираем нашу БД, и `OK`. Возвращаемся первоначальному окошку, и ставим галочку `Use Ef Core 3.x`:
<p align="center">
     <img src="/Images/add-model.jpg" width="404" alt="">  
</p>

Далее появится окошко, где можно выбрать все таблицы (или некоторые), снова `OK` и открывается окно параметров создания модели из БД:

<p align="center">
     <img src="/Images/generate-model.jpg" width="407" alt="">  
</p>

Здесь задаем необходимые каталога Model и Data, ставим галочку для анотации данных. После нажатия `OK` происходит реконструирование данных.

## Отображение модели из БД
Для отображения модели нам надо прописать строки подключения для проекта, сначала добавим строку подключения в _appsettings.json_ 
```
  "ConnectionStrings": {
    "SchoolContext": "Server=(localdb)\\mssqllocaldb;Database=ConsotoDbFirst;Trusted_Connection=True;MultipleActiveResultSets=true"
  }
```
затем в файле _Startup.cs_ добавим в метод ConfigureServices строку подключения и необходимые пространства имен:
```
using Microsoft.EntityFrameworkCore;
using ConsotoDbFirst.Data;
------------------------

 public void ConfigureServices(IServiceCollection services)
        {
            services.AddRazorPages();
            //строка подключения:
            services.AddDbContext<ConsotodbfirstContext>(options =>
                   options.UseSqlServer(Configuration.GetConnectionString("SchoolContext")));
        }
```
Теперь мы можем отобразить схему модели, для этого щёлкните правой кнопкой мыши на проект, в выпадающем списке выберите **EF Core Power Tools** затем **Add DbContext Model Diagramm**

<p align="center">
     <img src="/Images/db-dgml.jpg" width="1210" alt="">  
</p>

В проекте появится xml файл который хранит схему отображения данных _ConsotodbfirstContext.dgml_

## Отображение модели из классов
Мы также можем отоброзить схему из классов модели, без соединения с БД. Рассмотрим данный случай для консольного приложения.  
В каталоге _Data_ находится файл контекста (наследует от : DbContext), этом файле замените метод `OnConfiguring` на следующий код:
```
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
   if (!optionsBuilder.IsConfigured)
   {
       optionsBuilder.UseSqlServer("Data Source=base"); //заменитель строки подключения
       base.OnConfiguring(optionsBuilder);
   }
}
```
Теперь запустите построитель модели, как в предыдущем примере. Схема будет создана на основе классов в модели приложения.  
(слово base в коде, может быть заменено на любое.)  

## Обновление БД

В процессе разработки, вам может понадобится изменить какие то таблицы, добавить новые, изменить связи. Моё мнение, не стоит для этого применять миграции, кто даст гаранитии, что не будет сбоев, и ваши драгоценные данные не будут потеряны? Лучше использовать максимально безопасный метод. Для этого сделайте изменения прям в БД, а потом снова реконструируйте измененные таблицы, можно для этого использовать другую папу модели, а потом согласовать исходный код.

<br /><br />
<p align="center">
  Практические консультации вы можете получить на наших <a  href="http://creativcode.ru/learn" target="_blank" >веб курсах в Сочи, Адлер</a>:<br /><br />
   <a  href="http://creativcode.ru/learn/webnet" target="_blank" >
  <img src="http://creativcode.ru/img/learn/net-backend.jpg" width="400" alt="">
   </a>
</p>
