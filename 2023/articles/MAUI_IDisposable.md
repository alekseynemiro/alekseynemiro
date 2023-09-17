# How to implement IDisposable in view models in MAUI projects

When developing a **MAUI 7** application (**.NET Multi-Platform App UI + dotnet 7**) using the **MVVM** (**Model-view-ViewModel**) design pattern, I had to clear the resources occupied by view models. I am using **Dependency Injection** for all view model in my project. All models are implemented in the `Transient` mode, i.e. creates a new instance of the model whenever needed. The implementation of the `IDisposable` interface in the models did not give any result. As a result, the program creates new instances of models for each request, but the old ones continue to hang in memory and take up resources.

In my case, model resources should be released when the life cycle of the page (`ContentPage`) on which the models are used ends. To solve the problem, I decided to use the `Unloaded` event handler, which is called after the page (component) is unloaded.

As a result, the view model implements `IDisposable`, when the page instance is initialized, the model is placed in the `BindingContext` and the `Unloaded` handler is added to the page, in which the `IDisposable.Dispose` method is called:

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

Casting the `BindingContext` to `IDisposable` using the `as` keyword returns null if the object in the `BindingContext` does not implement the `IDisposable` interface. Using the conditional operator `?.` avoids errors if the value is `null`. Thus, the `Dispose` method will only be called if the `BindingContext` implements `IDisposable`.

In order not to write the same code on every page, I made a small `IServiceCollection` extension method that instantiates the page and adds an `Unloaded` event handler:

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

I used the `ActivatorUtilities.CreateInstance` method instead of `IServiceProvider.GetRequiredService` or `IServiceProvider.GetService` to avoid looping.

As a result, adding pages to the container looks like this:

```c#
builder.Services.AddPage<MainPage>();
builder.Services.AddPage<SomePage>();
builder.Services.AddPage<EtcPage>();
```

The sample project can be found at the following link:  
https://github.com/samples-kit/maui-viewmodel-disposable

---
Aleksey Nemiro  
2023-04-23

https://medium.com/maui-stories/how-to-implement-idisposable-in-view-models-in-maui-projects-7a78c766f90c
