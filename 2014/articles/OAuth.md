# Авторизация по протоколу OAuth в проектах .NET Framework

## Введение

Крупные порталы и социальные сети могут предоставлять сторонним проектам ограниченный доступ к профилю и материалам своих пользователей. Безусловно, с разрешения последних. В большинстве своем это реализуется при помощи протокола **OAuth**.

Протокол **OAuth** позволяет клиентским приложениям получать необходимые разрешения к информации идентифицируемого пользователя, или даже права на выполнение определенных действий от имени пользователя на сайте-сервере **OAuth**.

Это может быть удобно, поскольку конечному пользователю не требуется регистрировать новые учетные записи на сайтах, а также пользователь может избавиться от необходимости вручную синхронизировать информацию между различными ресурсами.

При этом, клиентское приложение не получает доступ к учетным данным пользователя (имя пользователя и пароль), что делает данный механизм не только удобным, но и безопасным в использовании.

Для **.NET Framework** существует множество библиотек, которые позволяют реализовать авторизацию по протоколу **OAuth**.

В этой статье речь пойдет об одной из таких библиотек, которая называется **Nemiro.OAuth**.

Библиотека классов **Nemiro.OAuth** является клиентом и поддерживает работу с протоколами **OAuth 1.0** и **OAuth 2.0**, и, что не мало важно, содержит готовые классы для взаимодействия с популярными ресурсами, в том числе: **Facebook**, **Twitter**, **ВКонтакте**, **Mail.Ru**, **Одноклассники**, **Яндекс**, **Google** и многими другими.

> [!NOTE]
> Клиент - т.е. выступает в роли клиента, а не сервера **OAuth**.
> Позволяет проводить авторизацию пользователей на других ресурсах-серверах **OAuth**.

Исходный код библиотеки открыт и распространяется по лицензии **Apache 2.0**.

Вы можете найти проект и подробную документацию на сайте: http://oauth.nemiro.net.

## Подключение библиотеки к проекту

Наиболее простым способом подключения библиотеки **Nemiro.OAuth** к проекту является использование консоли диспетчера пакетов (**Package Manager Console**).

Чтобы открыть окно консоли диспетчера пакетов (не путать с обычной консолью), выберите меню **Сервис** (**Service**) -> **Диспетчер пакетов библиотек** (**Library Package Manager**) -> **Консоль диспетчера пакетов** (**Package Manager Console**).

В появившемся окошке введите команду:

```
Install-Package Nemiro.OAuth
```

Если все прошло нормально, то в проект автоматически загрузится и подключится библиотека **Nemiro.OAuth**.

> [!NOTE]
> Таким способом вы можете подключать практически любые внешние компоненты, а не только **Nemiro.OAuth**.
> Более подробную информацию по этому вопросу можно найти по следующей ссылке: http://msdn.microsoft.com/ru-ru/magazine/hh547106.aspx.

Если по каким-то причинам установка через диспетчер пакетов не сработала, то вы можете скачать файлы библиотеки с сервера **GitHub** и добавить ссылку на сборку **Nemiro.OAuth** вручную: меню **Проект** (**Project**) -> **Добавить ссылку** (**Add Reference**) -> вкладка **Обзор** (**Browse**).

Если вы всерьез занимаетесь программированием, то наверняка вам уже доводилась делать это с другими библиотеками и компонентами.

> [!NOTE]
> На сервере **GitHub**, помимо самой библиотеке **Nemiro.OAuth**, также можно найти файлы примеров использования библиотеки в различных типах проектов.

В проектах **Windows Forms** может потребоваться изменить целевую платформу **.NET Framework**.

Необходимо использовать полноценный **.NET Framework**, а не **Client Profile**.

Изменить целевую платформу можно в свойствах проекта: меню **Проект** (**Project**) -> **Свойства...** (**Properties**) -> вкладка **Приложение** (**Application**) -> параметр **Требуемая версия .NET Framework** (**Target framework**). Выберите полноценный **.NET Framework**.

> [!NOTE]
> В **Visual Studio 2010** в проектах **Visual Basic .NET** изменение целевой версии **.NET Framework** производится на влкадке **Компиляция** (**Build**) в окне **Дополнительные параметры компиляции** (**Advanced build options**).

## Как пользоваться библиотекой?

Использовать библиотеку очень просто. Для удобства, рекомендуется импортировать пространства имен `Nemiro.OAuth` и `Nemiro.OAuth.Clients` в проект.

**C#:**
```c#
using Nemiro.OAuth;
using Nemiro.OAuth.Clients;
```

**Visual Basic .NET:**
```vb
Imports Nemiro.OAuth
Imports Nemiro.OAuth.Clients
```

Пространство `Nemiro.OAuth` содержит основные классы, которые необходимы для работы с протоколами **OAuth** первой и второй версии.

В `Nemiro.OAuth.Clients` содержатся готовые реализации клиентов для популярных ресурсов.

Классы-клиенты можно использовать как есть, создавая новые экземпляры и вызывая определенные методы. Но лучше всего работать через два вспомогательных класса: `OAuthManager` и `OAuthWeb`.

Статичный класс `OAuthManager` позволяет регистрировать в приложении клиентов для различных серверов **OAuth**.

Например, если требуется реализовать авторизацию через **Facebook**, то необходимо зарегистрировать в приложении клиента `FacebookClient`. Для авторизации через сервер **ВКонтакте**, в приложении следует зарегистрировать клиента `VkontakteClient`. И так далее.

> [!NOTE]
> В одном приложении можно зарегистрировать один клиентский класс только один раз.
> Например, если в приложении уже зарегистрирован клиент `TwitterClient`, то добавить его повторно не получится.
> Как правило, использовать в рамках одного приложения несколько одинаковых клиентов нет необходимости.

Регистрировать клиенты **OAuth** следует в момент инициализации приложения.

В веб-проектах это можно осуществить в обработчике события `Application_Start` файла `Global.asax`.

В проектах **Windows Forms** проводить регистрацию клиентов в приложении удобно в момент загрузки формы.

При регистрации клиента нужно указывать идентификатор приложения и секретный ключ, которые можно найти на сайте-поставщике **OAuth**.

**C#:**
```c#
OAuthManager.RegisterClient
(
  new FacebookClient
  (
    "1435890426686808", 
    "c6057dfae399beee9e8dc46a4182e8fd"
  )
);
```

**Visual Basic .NET:**
```vb
OAuthManager.RegisterClient _
(
  New FacebookClient _
  (
    "1435890426686808", 
    "c6057dfae399beee9e8dc46a4182e8fd"
  )
)
```

Статичный класс `OAuthWeb` позволяет получить конечный адрес для аутентификации и авторизации пользователя на сервере **OAuth**. А также проводит проверку результатов авторизации, получение маркера доступа (**access token**) и детальной информации из профиля пользователя на сервере **OAuth**.

Как видно из названия, класс `OAuthWeb` предназначен преимущественно для веб-проектов, однако его вполне можно использовать и в проектах **Windows Forms**.

Процедура авторизации делится на два этапа.

На первом этапе пользователя необходимо перенаправить на сервер **OAuth** для прохождения аутентификации и получение подтверждения доступа к данным профиля пользователя.

Для получения адреса можно использовать метод `OAuthWeb.GetAuthorizationUrl()`, который принимает имя сервера **OAuth** в качестве основного параметра.

Адрес формируется автоматически и будет уникальным для каждого случая.

**C#:**
```c#
string url = OAuthWeb.GetAuthorizationUrl("facebook");
```

**Visual Basic .NET:**
```vb
Dim url As String = OAuthWeb.GetAuthorizationUrl("facebook")
```

> [!NOTE]
> В **ASP.NET WebForms** можно использовать метод `OAuthWeb.RedirectToAuthorization()`, который автоматически производит перенаправление текущего пользователя на сервер **OAuth**.

Имена серверов **OAuth** есть в документации. В таблице ниже представлен список имен всех серверов, для которых есть готовые клиенты в библиотеке **Nemiro.OAuth**.

| Уникальное имя поставщика | Описание |
| ------------------------- | -------- |
| Amazon | Клиент для Amazon |
| Facebook | Клиент для Facebook |
| GitHub | Клиент для GitHub |
| Google | Клиент для Google |
| Live | Клиент для Microsoft Live |
| Mail.Ru | Клиент для Mail.Ru |
| Odnoklassniki | Клиент для Одноклассники |
| Twitter | Клиент для Twitter |
| VK | Клиент для ВКонтакте |
| Yandex | Клиент для Яндекс |

На втором этапе проводится проверка результатов авторизации по адресу, на который сервер **OAuth** возвращает пользователя. Как правило, обратный адрес указывается в настройках приложения на сервере **OAuth**.

> [!NOTE]
> Обратный адрес также можно указать вторым параметром, при получении адреса аутентификации, в методе `OAuthWeb.GetAuthorizationUrl()`.
> Однако не все поставщики поддерживают данную возможность в полной мере.

В обратный адрес сервер добавляет техническую информацию, по которой можно получить маркер доступа и основные данные профиля пользователя. Для проверки результатов авторизации используется функция `OAuthWeb.VerifyAuthorization()`.

**C#:**
```c#
var result = OAuthWeb.VerifyAuthorization("http://обратный адрес");
```

**Visual Basic .NET:**
```vb
Dim result = OAuthWeb.VerifyAuthorization("http://обратный адрес")
```

Если авторизация прошла успешно, то в полученном результате можно будет найти `AccessToken` и `UserInfo`.

`AccessToken` - это маркер доступа, ключ с ограниченным сроком действия, при помощи которого можно работать с **API** сервера-поставщика **OAuth**.

Класс `UserInfo` предоставляет базовую информацию о пользователе и может содержать различные данные, в зависимости от поставщика и уровня доступа приложения.

Например, сервер может вернуть: идентификатор пользователя, email, номер телефона, имя, фамилию, псевдоним, адрес фотографии и ссылку на сайт пользователя.

Важным и, пожалуй, самым сложным является правильная конфигурация приложений на серверах **OAuth**. В документации к библиотеке **Nemiro.OAuth** этот процесс достаточно подробно описан.

Далее будет рассмотрен процесс реализации авторизации по протоколу **OAuth** в различных типах проектов **.NET Framework** с использованием языков **C#** и **Visual Basic .NET**.

## Авторизация по протоколу OAuth в проектах ASP .NET WebForms

Практическая реализация авторизации по протоколу **OAuth** в проектах **ASP.NET WebForms** займет всего пять простых шага!

1. Добавьте в проект файл `Global.asax`, если такового еще нет.

2. В обработчике события `Application_Start` укажите список клиентов, через которые будет осуществляться авторизация. Например, если требуется осуществлять авторизацию через **Facebook** и **Mail.Ru**, то нужно зарегистрировать клиенты: `FacebookClient` и `MailRuClient`.

**C#:**
```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using System.Web.Security;
using System.Web.SessionState;
using Nemiro.OAuth;
using Nemiro.OAuth.Clients;

namespace Test.CSharp.AspWebForms
{
  public class Global : System.Web.HttpApplication
  {
    protected void Application_Start(object sender, EventArgs e)
    {
      OAuthManager.RegisterClient
      (
        new FacebookClient
        (
          "1435890426686808",
          "c6057dfae399beee9e8dc46a4182e8fd"
        )
      );
      OAuthManager.RegisterClient
      (
        new MailRuClient
        (
          "722701",
          "d0622d3d9c9efc69e4ca42aa173b938a"
        )
      );
    }
  }
}
```

**Visual Basic .NET:**
```vb
Imports Nemiro.OAuth
Imports Nemiro.OAuth.Clients

Public Class Global_asax
  Inherits System.Web.HttpApplication

  Sub Application_Start(ByVal sender As Object, ByVal e As EventArgs)
    OAuthManager.RegisterClient _
    (
      New FacebookClient _
      (
        "1435890426686808",
        "c6057dfae399beee9e8dc46a4182e8fd"
      )
    )
    OAuthManager.RegisterClient _
    (
      New MailRuClient _
      (
        "722701",
        "d0622d3d9c9efc69e4ca42aa173b938a"
      )
    )
  End Sub

End Class
````

> [!NOTE]
> Используйте идентификатор и секретный ключ для своих приложение, которые вам выдал сервер-поставщик **OAuth**.
> Работоспособность представленных в статье идентификаторов не гарантируется.

3. Далее, на сайте нужно вывести кнопки, нажатие на которые будет перенаправлять пользователя на сервер **OAuth** для авторизации.

> [!WARNING]
> Используйте кнопки с `PostBack` (обратным вызовом), такие как `LinkButton` или `Button`.
> Ни в коем случае не используйте ссылки! Использование ссылок может привести к утечке ресурсов, т.к. для каждой процедуры авторизации в памяти компьютера будет создаваться уникальная сессия.
> Процедура авторизации считается запущенной с момента получения ссылки на авторизацию.

Кнопки могут иметь один обработчик нажатия. Чтобы определить, на какую именно кнопку нажал пользователь, можно добавить атрибут `data-provider`, с указанием имени клиента **OAuth**.

```asp
<asp:LinkButton ID="lnkFacebook" runat="server" data-provider="facebook" Text="Войти через Facebook" onclick="RedirectToLogin_Click" />
<br />
<asp:LinkButton ID="lnkMailRu" runat="server" data-provider="mail.ru" Text="Войти через Mail.Ru" onclick="RedirectToLogin_Click" />
```

В обработчике нажатия на кнопку, необходимо выполнить перенаправление на сервер авторизации.

**C#:**
```c#
protected void RedirectToLogin_Click(object sender, EventArgs e)
{
  // получаем имя поставщика из атрибута data-provider
  string provider = ((LinkButton)sender).Attributes["data-provider"];

  // формируем обратный адрес (поддерживают не все поставщики)
  string returnUrl = new Uri(Request.Url, "ExternalLoginResult.aspx").AbsoluteUri;

  // перенаправляем пользователя на авторизацию
  OAuthWeb.RedirectToAuthorization(provider, returnUrl);
}
```

**Visual Basic .NET:**
```vb
Protected Sub RedirectToLogin_Click(sender As Object, e As System.EventArgs)
  ' получаем имя поставщика из атрибута data-provider
  Dim provider As String = CType(sender, LinkButton).Attributes("data-provider")

  ' формируем обратный адрес (поддерживают не все поставщики)
  Dim returnUrl As String = New Uri(Request.Url, "ExternalLoginResult.aspx").AbsoluteUri

  ' перенаправляем пользователя на авторизацию
  OAuthWeb.RedirectToAuthorization(provider, returnUrl)
End Sub
```

4. Добавьте новую страницу с именем `ExternalLoginResult.aspx`. Эта страница будет обрабатывать результат авторизации. В обработчике загрузки страницы можно выполните проверку текущего адреса методом `VerifyAuthorization()` класса `OAuthWeb`.

**C#:**
```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using System.Web.UI;
using System.Web.UI.WebControls;
using Nemiro.OAuth;

namespace Test.CSharp.AspWebForms
{
  public partial class ExternalLoginResult : System.Web.UI.Page
  {
    protected void Page_Load(object sender, EventArgs e)
    {
      var result = OAuthWeb.VerifyAuthorization();
      Response.Write(String.Format("Provider: {0}<br />", result.ProviderName));
      if (result.IsSuccessfully)
      {
         // успешно авторизованный
        var user = result.UserInfo;
        Response.Write(String.Format("User ID:  {0}<br />", user.UserId));
        Response.Write(String.Format("Name:     {0}<br />", user.DisplayName));
        Response.Write(String.Format("Email:    {0}", user.Email));
      }
      else
      {
        // ошибка
        Response.Write(result.ErrorInfo.Message);
      }
    }
  }
}
```

**Visual Basic .NET:**
```vb
Imports Nemiro.OAuth

Public Class ExternalLoginResult
  Inherits System.Web.UI.Page

  Protected Sub Page_Load(ByVal sender As Object, ByVal e As System.EventArgs) Handles Me.Load
    Dim result As AuthorizationResult = OAuthWeb.VerifyAuthorization()
    Response.Write(String.Format("Provider: {0}<br />", result.ProviderName))
    If result.IsSuccessfully Then
      ' успешно авторизованный
      Dim user As UserInfo = result.UserInfo
      Response.Write(String.Format("User ID:  {0}<br />", user.UserId))
      Response.Write(String.Format("Name:     {0}<br />", user.DisplayName))
      Response.Write(String.Format("Email:    {0}", user.Email))
    Else
      ' ошибка
      Response.Write(result.ErrorInfo.Message)
    End If
  End Sub

End Class
```

5. Пользуйтесь!

## Авторизация по протоколу OAuth в проектах ASP.NET MVC

Процедура авторизации по протоколу **OAuth** в проектах **ASP.NET MVC** не сложнее, чем в проектах **Web Forms**.

Первые два шага полностью идентичны.

3. Создайте в контроллере `HomeController` обработчик действия `ExternalLogin`, который будет перенаправлять пользователя на сервер авторизации.

**C#:**
```c#
public ActionResult ExternalLogin(string provider)
{
  // формируем обратный адрес (поддерживают не все поставщики)
  string returnUrl = Url.Action("ExternalLoginResult", "Home", null, null, Request.Url.Host);
  // перенаправляем пользователя на авторизацию
  return Redirect(OAuthWeb.GetAuthorizationUrl(provider, returnUrl));
}
```

**Visual Basic .NET:**
```vb
Public Function ExternalLogin(provider As String) As ActionResult
  ' формируем обратный адрес (поддерживают не все поставщики)
  Dim returnUrl As String = Url.Action("ExternalLoginResult", "Home", Nothing, Nothing, Request.Url.Host)
  ' перенаправляем пользователя на авторизацию
  Return Redirect(OAuthWeb.GetAuthorizationUrl(provider, returnUrl))
End Function
```

4. Добавьте в контроллер `HomeController` обработчик действия `ExternalLoginResult`.

**C#:**
```c#
public ActionResult ExternalLoginResult()
{
  string output = "";
  var result = OAuthWeb.VerifyAuthorization();
  output += String.Format("Provider: {0}\r\n", result.ProviderName);
  if (result.IsSuccessfully)
  {
    // успешно авторизованный
    var user = result.UserInfo;
    output += String.Format("User ID:  {0}\r\n", user.UserId);
    output += String.Format("Name:     {0}\r\n", user.DisplayName);
    output += String.Format("Email:    {0}", user.Email);
  }
  else
  {
    // ошибка
    output += result.ErrorInfo.Message;
  }
  return new ContentResult
  { 
    Content = output, 
    ContentType = "text/plain" 
  };
}
```

**Visual Basic .NET:**
```vb
Public Function ExternalLoginResult() As ActionResult
  Dim output As String = ""
  Dim result As AuthorizationResult = OAuthWeb.VerifyAuthorization()
  output &= String.Format("Provider: {0}", result.ProviderName)
  output &= vbCrLf
  If result.IsSuccessfully Then
    ' успешно авторизованный
    Dim user As UserInfo = result.UserInfo
    output &= String.Format("User ID:  {0}", user.UserId)
    output &= vbCrLf
    output &= String.Format("Name:     {0}", user.DisplayName)
    output &= vbCrLf
    output &= String.Format("Email:    {0}", user.Email)
  Else
    ' ошибка
    output &= result.ErrorInfo.Message
  End If
  Return New ContentResult() With _
  { 
    .Content = output, 
    .ContentType = "text/plain" 
  }
End Function
```

5. На странице разместите ссылки **JavaScript**, ведущие на действие `ExternalLogin`.

```html
<a href="#" onclick="window.location.href='@Url.Action("ExternalLogin", new { provider = "Facebook" });return false;'">Вход через Facebook</a>
<br />
<a href="#" onclick="window.location.href='@Url.Action("ExternalLogin", new { provider = "Mail.Ru" });return false;'">Вход через Mail.Ru</a>
```

> [!WARNING]
> Используйте ссылки **JavaScript**. Ни в коем случае не используйте прямые ссылки!
> Использование прямых ссылок может привести к утечке ресурсов, т.к. для каждой процедуры авторизации в памяти компьютера будет создаваться уникальная сессия.
> Процедура авторизации считается запущенной с момента получения ссылки на авторизацию.

6. Пользуйтесь!

## Примечание для веб-проектов

Страница `ExternalLoginResult.aspx` и обработчик `ExternalLoginResult` могут не содержать никакого кода **HTML**.

Если у вас на сайте есть локальная система аутентификации пользователей, то вы можете связать локального пользователя с пользователем внешнего ресурса.

Например, можно сделать в базе данных таблицу: `oauth_users`, которая может содержать поля: `local_user_id`, `provider_user_id, provider_name`, где:

* `local_user_id` – идентификатор пользователя в вашей базе данных;
* `provider_user_id` – идентификатор пользователя на сайте поставщика;
* `provider_name` – имя поставщика **OAuth**.

При этом, результат авторизации можно не показывать пользователю, по крайней мере успешный результат.

Можно сразу перенаправлять пользователя, например, в личный кабинет или другой раздел, ради которого пользователь проходил идентификацию.

Некоторые поставщики могут отказаться выдавать **email** пользователя. Это может быть связано с недостаточным уровнем доверия к приложению или отсутствием необходимых разрешений.

Внимательно ознакомьтесь с документацией поставщиков **OAuth** (на сайтах поставщиков).

В качестве решения этой проблемы, можно после авторизации показывать пользователю форму с запросом недостающих данных, чтобы пользоватьль указал их самостоятельно.

## Авторизация по протоколу OAuth в проектах Windows Forms

В проектах **Windows Forms** осуществить авторизацию по протоколу **OAuth** можно при помощи стандартного элемента `WebBrowser`.

1. В обработчике события главной формы, или при инициализации приложения, зарегистрируйте необходимые клиенты **OAuth**.

**C#:**
```c#
private void Form1_Load(object sender, EventArgs e)
{
  OAuthManager.RegisterClient
  (
    new VkontakteClient
    (
      "4457505",
      "wW5lFMVbsw0XwYFgCGG0"
    ) 
    { 
      Parameters = new NameValueCollection { { "display", "popup" } }
    }
  );
  OAuthManager.RegisterClient
  (
    new YandexClient
    (
      "0ee5f0bf2cd141a1b194a2b71b0332ce",
      "59d76f7c09b54ad38e6b15f792da7a9a"
    )
  );
}
```

**Visual Basic .NET:**
```vb
Private Sub Form1_Load(sender As System.Object, e As System.EventArgs) Handles MyBase.Load
  OAuthManager.RegisterClient _
  (
    New VkontakteClient _
    (
      "4457505",
      "wW5lFMVbsw0XwYFgCGG0"
    ) _
    With
    {
      .Parameters = New NameValueCollection() From {{"display", "popup"}}
    }
  )
  OAuthManager.RegisterClient _
  (
    New YandexClient _
    (
      "0ee5f0bf2cd141a1b194a2b71b0332ce",
      "59d76f7c09b54ad38e6b15f792da7a9a"
    )
  )
End Sub
```

2. Разместите на форме две кнопки и добавьте общий обработчик нажатия (`btn_Click`), который будет открывать окно авторизации. В свойстве `Tag` кнопок необходимо указать имя поставщика **OAuth**.

**C#:**
```c#
private void btn_Click(object sender, EventArgs e)
{
  var frm = new frmLogin(((Button)sender).Tag.ToString());
  frm.ShowDialog();
}
```

**Visual Basic .NET:**
```vb
Private Sub btn_Click(sender As System.Object, e As System.EventArgs)
  Call New Form2(CType(sender, Button).Text).ShowDialog()
End Sub
```

3. Добавьте новую форму (далее `frmLogin`) и разместите на ней элемент `WebBrowser`.

4. Создайте для формы `frmLogin` конструктор, принимающий один строковой параметр. В этот параметр будет передаваться имя зарегистрированного в программе поставщика **OAuth**, по которому будет строиться адрес для авторизации.

**C#:**
```c#
public frmLogin(string providerName)
{
  InitializeComponent();
  webBrowser1.Navigate(OAuthWeb.GetAuthorizationUrl(providerName));
}
```

**Visual Basic .NET:**
```vb
Public Sub New(providerName As String)
  InitializeComponent()
  WebBrowser1.Navigate(OAuthWeb.GetAuthorizationUrl(providerName))
End Sub
```

5. В обработчик события `DocumentCompleted` элемента `WebBrowser` поместите код, который будет искать и обрабатывать результаты авторизации.

**C#:**
```c#
private void webBrowser1_DocumentCompleted(object sender, WebBrowserDocumentCompletedEventArgs e)
{
  // ожидаем, когда появится адрес с результатами авторизации
  if (e.Url.Query.IndexOf("code=") != -1 || e.Url.Fragment.IndexOf("code=") != -1 || e.Url.Query.IndexOf("oauth_verifier=") != -1)
  {
    // проверяем адрес
    var result = OAuthWeb.VerifyAuthorization(e.Url.ToString());
    if (result.IsSuccessfully)
    {
      // показываем данные пользователя
      MessageBox.Show
      (
        String.Format
        (
          "User ID: {0}\r\nUsername: {1}\r\nDisplay Name: {2}\r\nE-Mail: {3}", 
          result.UserInfo.UserId,
          result.UserInfo.UserName,
          result.UserInfo.DisplayName ?? result.UserInfo.FullName,
          result.UserInfo.Email
        ), 
        "Successfully", 
        MessageBoxButtons.OK, 
        MessageBoxIcon.Information
      );
    }
    else
    {
      // ошибка
      MessageBox.Show(result.ErrorInfo.Message, "Error", MessageBoxButtons.OK, MessageBoxIcon.Error);
    }
    this.Close();
  }
}
```

**Visual Basic .NET:**
```vb
Private Sub WebBrowser1_DocumentCompleted(sender As System.Object, e As System.Windows.Forms.WebBrowserDocumentCompletedEventArgs) Handles WebBrowser1.DocumentCompleted
  ' ожидаем, когда появится адрес с результатами авторизации
  If Not e.Url.Query.IndexOf("code=") = -1 OrElse Not e.Url.Fragment.IndexOf("code=") = -1 OrElse Not e.Url.Query.IndexOf("oauth_verifier=") = -1 Then
    ' проверяем адрес
    Dim result = OAuthWeb.VerifyAuthorization(e.Url.ToString())
    If result.IsSuccessfully Then
      ' показываем данные пользователя
      MessageBox.Show _
      (
        String.Format _
        (
          "User ID: {0}{4}Username: {1}{4}Display Name: {2}{4}E-Mail: {3}",
          result.UserInfo.UserId,
          result.UserInfo.UserName,
          If(Not String.IsNullOrEmpty(result.UserInfo.DisplayName), result.UserInfo.DisplayName, result.UserInfo.FullName),
          result.UserInfo.Email,
          vbNewLine
        ),
        "Successfully",
        MessageBoxButtons.OK,
        MessageBoxIcon.Information
      )
    Else
      ' ошибка
      MessageBox.Show(result.ErrorInfo.Message, "Error", MessageBoxButtons.OK, MessageBoxIcon.Error)
    End If
    Me.Close()
  End If
End Sub
```

6. Пользуйтесь!

## Маркер доступа

Маркер доступа, известный также как **access token** или просто «токен», необходим для работы с **API** сайтов, предоставляющих доступ к ресурсам своих пользователей по протоколу **OAuth**.

Каждый поставщик имеет свой уникальный **API**. Единой схемы работы с **API**, к сожалению, нет. Однако есть общие признаки, которые можно использовать при работе с различными интерфейсами.

В библиотеке **Nemiro.OAuth** базовая информация о пользователе получается при помощи **API**. Вы можете взять за основу методику работы с **API** и применить её для воплощения своих потребностей.

Не рекомендуется реализовывать работу с **API** непосредственно в библиотеке. Библиотека классов **Nemiro.OAuth** предназначена исключительно для работы с протоколом **OAuth**. Если вы хотите реализовать работу с **API**, лучше сделайте новый, узкопрофильный, проект, который будет принимать от **Nemiro.OAuth** маркер доступа и использовать его для запросов к методам **API**.

Следует отметить, что для работы с **API** могут потребоваться особые разрешения. Вы можете легко запрашивать любые разрешения, указав их в свойстве `Scope` при регистрации классов-клиентов **OAuth**.

Например, чтобы получить доступ к базой информации о пользователе и списку друзей пользователя с сервера **ВКонтакте**, необходимо запросить разрешения: `status` и `friends`.

**C#:**
```c#
OAuthManager.RegisterClient
(
  new VkontakteClient
  (
    "4457505",
    "wW5lFMVbsw0XwYFgCGG0"
  ) 
  { 
    Parameters = new NameValueCollection { { "display", "popup" } },
    Scope = "status,friends"
  }
);
```

**Visual Basic .NET:**
```vb
OAuthManager.RegisterClient _
(
  New VkontakteClient _
  (
    "4457505",
    "wW5lFMVbsw0XwYFgCGG0"
  ) _
  With
  {
    .Parameters = New NameValueCollection() From {{"display", "popup"}},
    .Scope = "status,friends"
  }
)
```

> [!NOTE]
> Если вы заметили, в примере также указано свойство `Parameters`.
> Это свойство содержит список параметров, которые будут переданы в адрес авторизации.
> В данном случае, будет передан параметр `display=popup`, что сообщит серверу **ВКонтатке**, что следует показывать модальное окно авторизации, а не стандартное.
> Для каждого сервера дополнительные параметры могут быть индивидуальными, информацию об этом вы можете найти на сайтах серверов **OAuth**.

Каждый поставщик **OAuth** имеет собственный набор прав. Информацию о доступных разрешениях вы можете найти в документации **API** на сайтах поставщиков **OAuth**.

Маркер доступа можно сохранять в приложении и использовать, не проводя повторную процедуру авторизации, до тех пор, пока не истечет его срок действия. Срок действия обычно зависит от требуемых прав. Чем меньше прав необходимо приложению, тем дольше может быть срок действия маркера доступа.

Маркер доступа находится в результатах проверки авторизации пользователя, в свойстве `AccessTokenValue`. Разумеется, только если результат проверки был успешным (`IsSuccessfully`).

**C#:**
```c#
var result = OAuthWeb.VerifyAuthorization(e.Url.ToString());

if (result.IsSuccessfully)
{
  Console.WriteLine("Маркер доступа: {0}",  result.AccessTokenValue);
}
```

**Visual Basic .NET:**
```vb
Dim result = OAuthWeb.VerifyAuthorization(e.Url.ToString())

If result.IsSuccessfully Then
  Console.WriteLine("Маркер доступа: {0}",  result.AccessTokenValue)
End If
```

В примере ниже, показано, как можно использовать маркер доступа для получения списка друзей авторизованного пользователя **ВКонтакте**.

**C#:**
```c#
private void GetVKontakteFriends(string accessToken)
{
  // параметры запроса
  var parameters = new NameValueCollection
  { 
    { "access_token", accessToken },
    { "count", "10" }, // только первые десять друзей
    { "fields", "nickname" }
  };

  // выполняем запрос к API
  var result = Nemiro.OAuth.Helpers.ExecuteRequest
  (
    "GET",
    "https://api.vk.com/method/friends.get",
    parameters,
    null
  );

  // выводим список друзей в окно консоли
  foreach (Dictionary<string, object> itm in (Array)result["response"])
  {
    string friendName = "";
    if (itm.ContainsKey("first_name") && itm.ContainsKey("last_name"))
    {
      friendName = String.Format("{0} {1}", itm["first_name"], itm["last_name"]);
    }
    else
    {
      friendName = itm["nickname"].ToString();
    }
    Console.WriteLine(friendName);
  }
}
```

**Visual Basic .NET:**
```vb
Public Sub GetVKontakteFriends(accessToken As String)
  ' параметры запроса
  Dim parameters As New NameValueCollection() From _
  {
    {"access_token", accessToken},
    {"count", "10"},
    {"fields", "nickname"}
  }

  ' выполняем запрос к API
  Dim result As RequestResult = Helpers.ExecuteRequest _
  (
    "GET",
    "https://api.vk.com/method/friends.get",
    parameters,
    Nothing
  )

  ' выводим список друзей в окно консоли
  For Each itm As Dictionary(Of String, Object) In CType(result("response"), Array)
    Dim friendName As String = ""
    If itm.ContainsKey("first_name") AndAlso itm.ContainsKey("last_name") Then
      friendName = String.Format("{0} {1}", itm("first_name"), itm("last_name"))
    Else
      friendName = itm("nickname").ToString()
    End If
    Console.WriteLine(friendName)
  Next
End Sub
```

## Послесловие

Реализация авторизации через внешние ресурсы является важной функцией, которая может существенно увеличить аудиторию вашего сайта или вашего приложения, повысить лояльность и активность пользователей.

Библиотека **Nemiro.OAuth** позволяет довольно просто реализовать авторизацию в проектах **.NET Framework** через любые серверы, поддерживающие работу по протоколу **OAuth**.

Благодаря открытому исходному коду, можно дополнять или изменять функционал библиотеки. Если у вас есть учетная запись на сервер **GitHub**, вы можете просто скопировать себе проект.

Пользуйтесь, и помните, чем меньше кода вы напишите, тем проще будет его поддерживать!

**[Скачать библиотеку Nemiro.OAuth и примеры использования](https://github.com/nemiro-net/nemiro.oauth/releases)**

---

### Примечания

* Статья написана для версии библиотеки **Nemiro.OAuth v.1.1**. Для более новых версий, представленная информация может быть неактуальной или отличаться. Тем не менее, базовый функционал должен в любом случае сохраниться без изменений, либо с полной совместимостью, во всех версиях 1.x.
* Представленные в статье учетные данные приложений со временем могут перестать работать. Следует это учитывать при проверке работы примеров. Новые учетные данные можно получить на серверах **OAuth**, зарегистрировав свои приложения.
* Ни в коем случае не используйте представленные в этой статье и в документации к библиотеке **Nemiro.OAuth** учетные данные приложений в реальных проектах.
* При написании примеров кодов, представленных в этой статье, использовалась среда **Microsoft Visual Studio 2010 Professional SP1**. Работоспособность представленных фрагментов кодов и примеров в более ранних или младших версиях **Visual Studio**, а также альтернативных средах разработки, не гарантируется. Представленные фрагменты кода и примеры должны без проблем работать в новых версиях **Visual Studio**, в том числе: **Visual Studio 2012** и **2013**.
* Для работы с протоколом **OAuth** требуется активное сетевое соединение. Убедитесь, что ваш брандмауэр не блокирует доступ вашему приложению в сеть.

### Полезные ссылки

* [Документация Nemiro.OAuth](http://oauth.nemiro.net)

### Регистрация и управление приложениями на сайтах поставщиков:

* [Amazon](http://login.amazon.com/manageApps)
* [Facebook](https://developers.facebook.com/)
* [GitHub](https://github.com/settings/applications/)
* [Google](https://console.developers.google.com/)
* [Microsoft Live](https://account.live.com/developers/applications/index)
* [Mail.Ru](http://api.mail.ru/sites/my/)
* [Одноклассники](http://odnoklassniki.ru/dk?st.cmd=appsInfoMyDevList&st._aid=Apps_Info_MyDev)
* [Twitter](https://apps.twitter.com/)
* [ВКонтакте](http://vk.com/dev)
* [Яндекс](https://oauth.yandex.ru/client/my)

---
Алексей Немиро  
2014-07-31
