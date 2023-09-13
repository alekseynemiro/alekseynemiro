# Вывод картинок в ASP .NET MVC

Довольно часто при разработке веб-приложений требуется выдать пользователю данные в отличном от **html** формате.

Например, вывести уменьшенную копию изображения (**thumb image**), или же выдать защищенные данные из БД.

В **ASP.NET WebForms** для этих целей можно использовать **Handlers** (хэндлеры), либо обычные **ASPX**-страницы.

В **ASP.NET MVC** ситуация немного изменилась. Хотя конечно, по прежнему, ничто не запрещается использовать хэндлеры. Разве что, может потребоваться правило для `Routing`, типа: `routes.IgnoreRoute("{resource}.ashx")`.

В данном обзоре будет рассмотрен пример вывода изображения средствами **ASP.NET MVC**, однако описанный подход можно использовать для вывода данных абсолютно любого формата.

## ActionResult

Как известно (надеюсь, что известно), результатом выполнения любого действия (**action**) контроллера (**controller**) является `ActionResult`.

`ActionResult` – это базовый класс, от которого наследуются все классы, в конечном итоге, формирующие контент для вывода клиенту.

Например: `ViewResult`, `JsonResult`, `ContentResult`, `RedirectResult` и т.д.

Сам по себе, класс `ActionResult` использовать нельзя, от него можно только наследоваться, и за счет этого, список результатов выполнения действий контроллеров ограничивается лишь фантазией разработчиков.

Формирование данных для вывода происходит в методе `ExecuteResult`.

## ImageResult

Собственно, чтобы контроллер мог выдать клиенту изображение, нужно написать свой класс наследованный от `ActionResult`.

По логике, имя его должно быть `ImageResult`, но это уже на ваше усмотрение.

**VB**
```vb
Public Class ImageResult
  Inherits ActionResult
End Class
```

**C#**
```
public class ImageResult : ActionResult
{ }
```

Как я уже говорил, контент формируется в методе `ExecuteResult`, однако, прежде чем это делать, необходимо получить исходный материал.

Другими словами, передать в класс информацию, на основе которой будут формироваться конечные данные.

В случае с изображениями, наиболее универсально будет передавать в класс `Stream`.

Также потребуется передать информацию о типе контента (`Content-Type`), чтобы поисковики не пугались, и браузеры не гадали на кофейной гуще и, как следствие, не травмировали психику пользователя.

Для этого придется написать пару соответствующих свойств, а для полноты картины прописать конструктор.

**VB**
```vb
Private _ImageStream As Stream
Private _ContentType As String = ""

Public Property ImageStream() As Stream
  Get
    Return _ImageStream
  End Get
  Set(ByVal value As Stream)
    _ImageStream = value
  End Set
End Property

Public Property ContentType() As String
  Get
    Return _ContentType
  End Get
  Set(ByVal value As String)
    _ContentType = value
  End Set
End Property

Public Sub New(ByVal imageStream As Stream, ByVal contentType As String)
  If imageStream Is Nothing Then Throw New ArgumentNullException("imageStream")
  If String.IsNullOrEmpty(contentType) Then Throw New ArgumentNullException("contentType")

  _ImageStream = imageStream
  _ContentType = contentType
End Sub
```

**C#**
```C#
private static Stream _imageStream;
private static string _contentType = "";

public static Stream ImageStream
{
  get { return _imageStream; }
  set { _imageStream = value; }
}

public static string ContentType
{
  get { return _contentType; }
  set { _contentType = value; }
}

public ImageResult(Stream imageStream, string contentType)
{
  if (imageStream == null) throw new ArgumentNullException("imageStream");
  if (String.IsNullOrEmpty(contentType)) throw new ArgumentNullException("contentType")

  _imageStream = imageStream;
  _contentType = contentType;
}
```

В завершение этого, нужно написать код для метода `ExecuteResult`, в котором будет выводиться изображение на основе полученных данных.

**VB**
```vb
Public Override Sub ExecuteResult(ByVal context As System.Web.Mvc.ControllerContext)
  If context Is Nothing Then Throw ArgumentNullException("context")
  Dim Response As HttpResponseBase = context.HttpContext.Response
  Response.ContentType = _ContentType
  Dim buffer(1024) As Byte
  Do
    Dim read As Integer = _ImageStream.Read(buffer, 0, buffer.Lenght)
    If read = 0 Then Exit Do
    Response.OutputStream.Write(buffer, 0, read)
  Loop
  Response.End()
End Sub
```

**C#**
```cs
public static override void ExecuteResult(System.Web.Mvc.ControllerContext context)
{
  if (context == null) throw ArgumentNullException("context");
  HttpResponseBase response = context.HttpContext().Response;
  response.ContentType = _contentType;
  byte[] buffer = new byte[1024];
  do
  {
    int read = _imageStream.Read(buffer, 0, buffer.Lenght);
    if (read == 0) { break; }
    Response.OutputStream.Write(buffer, 0, read);
  }
  Response.End();
}
```

Теперь результатом действия любого контроллера может быть `ImageResult`.

**VB**
```vb
Return New ImageResult(System.IO.File.OpenRead("C:\kbyte.ru.gif"), "image/gif")
```

**C#**
```c#
return new ImageResult(System.IO.File.OpenRead("C:\\kbyte.ru.gif"), "image/gif");
```

## Extension

Для полного счастья, можно написать расширение для `System.Web.Mvc.Controller`.

Для этого, вбшники могут создать обычный модуль (`Module`), а сишники **static**-класс, с именем, например `ControllerExtensions`, и прописать в нем несколько удобных функций, которые будут возвращать `ImageResult`.

**Примечание.** Не забудьте импортировать пространство имен `System.Runtime.CompilerServices`.

**VB**
```vb
Public Module ControllerExtensions

  <Extension()> _
  Public Function Image(ByVal controller As System.Web.Mvc.Controller, ByVal imageStream As Stream, ByVal contentType As String) As ImageResult
    Return new ImageResult(imageStream, contentType)
  End Function

  <Extension()> _
  Public Function Image(ByVal controller As System.Web.Mvc.Controller, ByVal imageBytes() As Byte, ByVal contentType As String) As ImageResult
    Return new ImageResult(new MemoryStream(imageBytes), contentType)
  End Function

  <Extension()> _
  Public Function Image(ByVal controller As System.Web.Mvc.Controller, ByVal fileName As String, ByVal contentType As String) As ImageResult
    Return new ImageResult(Syste.IO.File.OpenRead(fileName), contentType)
  End Function

End Module
```

**C#**
```c#
public static class ControllerExtensions
{

  [Extension()]
  public static ImageResult Image(System.Web.Mvc.Controller controller, Stream imageStream, string contentType)
  {
    return new ImageResult(imageStream, contentType);
  }

  [Extension()]
  public static ImageResult Image(System.Web.Mvc.Controller controller, byte[] imageBytes, string contentType)
  {
    return new ImageResult(new MemoryStream(imageBytes), contentType);
  }

  [Extension()]
  public static ImageResult Image(System.Web.Mvc.Controller controller, string fileName, string contentType)
    return new ImageResult(Syste.IO.File.OpenRead(fileName), contentType);
  End Function

End Module
```

Теперь в контроллерах можно использовать прописанные функции, например:

**VB**
```vb
Return Image("C:\kbyte.ru.gif", "image/gif")
```

**C#**
```c#
return Image("C:\\kbyte.ru.gif", "image/gif");
```

## Послесловие

Наследуясь от класса `ActionResult` можно вывести данные абсолютно любого формата.

Какой функционал кода будет прописан в методе `ExecuteResult`, зависит полностью от вашей фантазии.

Что касается изображений, если вы будете выводить изображения из файлов, я бы рекомендовал предварительно считывать их (делать копию, например, в массив байт), чтобы случайно не заблокировать файлы, если в дальнейшем потребуется производить над ними (над файлами) какие-либо манипуляции (перемещение, удаление).

---
Алексей Немиро  
2009-06-29

https://habr.com/ru/articles/63149/
