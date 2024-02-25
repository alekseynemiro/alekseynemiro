# Реализация IDisposable в моделях представлений в проектах MAUI

В процессе разработки приложения **MAUI 7** (**.NET Multi-Platform App UI + dotnet 7**) с использованием шаблона проектирования **MVVM** (**Model–view–ViewModel**) у меня возникла необходимость очищать ресурсы занимаемые моделями представления (**View Model**). Проблема вроде бы несложная, достаточно реализовать интерфейс `IDisposable` в моделях. Но все оказалось не так просто. Все модели внедряются на страницы с помощью стандартного механизма **Dependency Injection**. При использовании временных зависимостей (`Transient`) нет четкого понимания, когда ресурсы будут освобождены. В итоге, в программе создаются новые экземпляры моделей для каждого запроса, но старые продолжают висеть в памяти и занимать ресурсы.

В моем случае, ресурсы моделей должны освобождаться вместе с завершением жизненного цикла страницы (`ContentPage`), в которую внедряются модели. Для решения проблемы, я решил использовать обработчик события `Unloaded`, который вызывается после выгрузки страницы (компонента).

В итоге, модель представления реализует `IDisposable`, при инициализации экземпляра страницы, модель помещается в `BindingContext` и к странице добавляется обработчик `Unloaded`, в котором вызывается метод `IDisposable.Dispose`:

```c#
public SomePage(ISomePageViewModel model)
{
  BindingContext = model;

  InitializeComponent();

  Unloaded += (object sender, EventArgs e) =>
  {
    (((ContentPage)sender).BindingContext as IDisposable)?.Dispose();
  };
}
```

Приведение `BindingContext` в `IDisposable` с помощью ключевого слова as позволяет получить значение `null`, если объект в `BindingContext` не реализует интерфейс `IDisposable`. А использование условного оператора `?.` позволяет избежать ошибок, если значение будет `null`. Таким образом, метод `Dispose` будет вызван только если `BindingContext` реализует `IDisposable`.

Чтобы не писать одинаковый код на каждой странице, я сделал небольшой метод расширения `IServiceCollection`, который создает экземпляр страницы и добавляет к нему обработчик события `Unloaded`:

```c#
internal static class ServiceCollectionExtensions
{
  public static void AddPage<T>(this IServiceCollection serviceCollection) where T : ContentPage
  {
    serviceCollection.AddTransient(PageWithDisposableContext<T>);
  }

  private static T PageWithDisposableContext<T>(IServiceProvider serviceProvider) where T : ContentPage
  {
    var page = ActivatorUtilities.CreateInstance<T>(serviceProvider);

    page.Unloaded += (object sender, EventArgs e) =>
    {
      (((ContentPage)sender).BindingContext as IDisposable)?.Dispose();
    };

    return page;
  }
}
```

Если для получения экземпляра страницы использовать метод `IServiceProvider.GetRequiredService` или `IServiceProvider.GetService`, то это может привести к циклическим вызовам метода создания экземпляра страницы. Для решения этой проблемы, можно использовать метод `ActivatorUtilities.CreateInstance`.

В результате, добавление страниц в контейнер выглядит следующим образом:

```c#
builder.Services.AddPage<MainPage>();
builder.Services.AddPage<SomePage>();
builder.Services.AddPage<EtcPage>();
```

Пример проекта можно найти по следующей ссылке:  
https://github.com/alekseynemiro/knowledgebase/tree/master/csharp/maui/MauiViewModelDisposable

---
Aleksey Nemiro  
2023-04-23

https://habr.com/ru/articles/731060/
