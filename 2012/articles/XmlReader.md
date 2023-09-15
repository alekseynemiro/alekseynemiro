# Обработка больших XML при помощи XmlReader

> В **.NET Framework** есть несколько основных классов для работы с **XML**.
> Большинство классов высокоуровневые и позволяют довольно легко обрабатывать XML-данные.
> Однако, если данных много, в приложении могут возникнуть проблемы с потреблением оперативной памяти.
> В этой статье рассмотрен метод чтения большого XML-файла при помощи класса `XmlReader` на языке **C#**.

## Введение

Для чтения **XML** чаще всего используются высокоуровневые классы `XmlDocument` и `XPathDocument`.

Что не удивительно, ведь ими довольно легко пользоваться.

Данные представлены в логичном, понятном виде, можно в любой момент получить любой набор данных в нужно формате.

Достигается это за счет того, что xml-данные загружаются в память. Это некритично для сравнительно небольших xml-файлов.

Однако когда размер xml-данных доходит до нескольких сотен мегабайт, а то и гигабайт, классы `XmlDocument` или `XPathDocument` использовать просто неприлично, а в некоторых случаях и вовсе невозможно, ведь приложение будет требовать объем оперативной памяти равный размеру данных.

Именно для таких случаев и предназначен класс `XmlReader`.

В отличие от своих собратьев по пространству имен, класс `XmlReader` не хранит данные в памяти.

Доступ к данным осуществляется последовательно.

Но из-за последовательности становится невозможным узнать, что содержит тот или иной xml-элемент, до того, как данные будут прочитаны.

Этот недостаток делает класс `XmlReader` неудобным в использовании и даже может отпугнуть неопытных программистов.

Но все не так страшно, как может показаться на первый взгляд.

## Пример чтения XML

Чтобы было понятно, о чем идет речь, приведу простой пример.

Предположим, что у нас есть xml-файл с информацией о пользователях **Kbyte.Ru**:

```xml
<kbyte>
  <members>
    <member kuid="1">
      <nickname>Алексей</nickname>
      <firstName>Алексей</firstName>
      <lastName>Немиро</lastName>
    </member>
    <member kuid="288">
      <nickname>Игорь Голов</nickname>
      <firstName>Игорь</firstName>
      <lastName>Голов</lastName>
    </member>
    <member kuid="1858">
      <nickname>[i]Pro</nickname>
      <firstName>Артем</firstName>
      <lastName>Донцов</lastName>
    </member>
    <member kuid="2575">
      <nickname>Shark1</nickname>
      <firstName>Виталий</firstName>
      <lastName />
    </member>
  </members>
</kbyte>
```

Данные отдельного пользователя находятся в элементе `member` и содержат: идентификатор пользователя (атрибут `kuid`), псевдоним (элемент `nickname`), имя (`firstName`) и фамилию (`lastName`).

При помощи класса `XmlDocument` данные каждого пользователя можно получить следующим образом:

```c#
XmlDocument xml = new XmlDocument();

xml.Load("XMLFile1.xml");

foreach (XmlNode n in xml.SelectNodes("/kbyte/members/member"))
{
  Console.WriteLine("KUID: {0}", n.Attributes["kuid"].Value);
  Console.WriteLine("Псевдоним: {0}", n.SelectSingleNode("nickname").InnerText);
  Console.WriteLine("Имя: {0}", n.SelectSingleNode("firstName").InnerText);
  Console.WriteLine("Фамилия: {0}", n.SelectSingleNode("lastName").InnerText);
  Console.WriteLine("------------------------------------------");
}
```

> [!NOTE]
> Классы `XmlDocument` и `XmlNode` принадлежат пространству имен `System.Xml`.
> Для их использования необходимо импортировать это пространство в проект.

> [!NOTE]
> Предполагается наличие файла с именем `XMLFile1.xml` в корневом каталоге приложения, который содержит приведенные выше xml-данные.

При использовании класса `XPathDocument` принципиальных отличий в коде не будет.

```c#
XPathDocument xml = new XPathDocument("XMLFile1.xml");
XPathNavigator nav = xml.CreateNavigator();

foreach (XPathNavigator n in nav.Select("/kbyte/members/member"))
{
  Console.WriteLine("KUID: {0}", n.GetAttribute("kuid", ""));
  Console.WriteLine("Псевдоним: {0}", n.SelectSingleNode("nickname").Value);
  Console.WriteLine("Имя: {0}", n.SelectSingleNode("firstName").Value);
  Console.WriteLine("Фамилия: {0}", n.SelectSingleNode("lastName").Value);
  Console.WriteLine("------------------------------------------");
}
```

> [!NOTE]
> Классы `XPathDocument` и `XPathNavigator` принадлежат пространству имен `System.Xml.XPath`, которое необходимо импортировать в проект.

Класс `XPathDocument` отличается от `XmlDocument` в основном тем, что позволяет использовать механизмы языка **XPath** (**XML Path Language**).

Язык **XPath** содержит множество встроенных функций для обработки информации, но в рамках данной статьи эта тема рассматриваться не будет.

Как видино из вышеприведенных примеров, при использовании классов `XmlDocument` и `XPathDocument` доступ к данным осуществляется через коллекцию узлов (`node`), каждый из которых имеет функции и методы управления содержимым документа.

В случае с классом `XmlReader` все будет менее очевидно, т.к. доступ к данным осуществляется на более низком уровне.

```c#
string lastNodeName = "";
using (XmlReader xml = XmlReader.Create("XMLFile1.xml"))
{
  while (xml.Read())
  {
    switch (xml.NodeType)
    {
      case XmlNodeType.Element:
        // нашли элемент member
        if (xml.Name == "member") 
        {
          if (xml.HasAttributes)
          {
            // поиск атрибута kuid
            while (xml.MoveToNextAttribute())
            {
              if (xml.Name == "kuid")
              {
                Console.WriteLine("KUID: {0}", xml.Value);
                break;
              }
            }
          }
        }

        // запоминаем имя найденного элемента
        lastNodeName = xml.Name;

        break;

      case XmlNodeType.Text:
        // нашли текст, смотрим по имени элемента, что это за текст
        if (lastNodeName == "nickname")
        {
          Console.WriteLine("Псевдоним: {0}", xml.Value);
        }
        else if (lastNodeName == "firstName")
        {
           Console.WriteLine("Имя: {0}", xml.Value);
        }
        else if (lastNodeName == "lastName")
        {
          Console.WriteLine("Фамилия: {0}", xml.Value);
        }
        break;

      case XmlNodeType.EndElement:
        // закрывающий элемент
        if (xml.Name == "member")
        {
          Console.WriteLine("------------------------------------------");
        }
        break;
    }
  }
}
```

`XmlReader` читает данные последовательно из потока.

Свойство `NodeType` содержит информацию о типе прочитанных данных.

При последовательном чтении данных, невозможно сразу перейти в узел `/kbyte/members/member`.
До него должен дойти указатель. Но и после того, как узел будет найден, невозможно узнать, какие данные он содержит, пока указатель не дойдет до этих данных.

Код получается сложным для восприятия. И это притом, что в представленном примере структура xml-документа простая.

Обработка документов со сложной структурой потребует написания более мудреного кода.

Хотя, если хорошенько подумать, даже в самых тяжелых случаях можно упростить задачу. Об этом пойдет речь далее.

## Работа с XmlReader

Класс `XmlReader` не имеет конструктора.

Создать новый экземпляр класса `XmlReader` можно при помощи метода `Create`, который принимает либо физический путь или **url** к xml-документу, либо поток (`Stream`).

```c#
XmlReader xml = XmlReader.Create("XmlFile1.xml");
```

Для перехода от одного узла к другому предназначена функция `Read`, которая возвращает либо `true`, если удалось осуществить переход к узлу, либо, по достижению конца документа, `false`.

Как правило, чтение всего документа производится циклом.

```c#
while (xml.Read())
{
  // ...
}
```

В зависимости от типа узла (`NodeType`), экземпляр класса `XmlReader` может содержать дополнительно имя узла (`Name`) и его текстовое значение (`Value`).

```c#
Console.WriteLine("Тип узла: {0}", xml.NodeType);
Console.WriteLine("Имя узла: {0}", xml.Name);
Console.WriteLine("Значение узла: {0}", xml.Value);
```

Для проверки наличия данных, предназначены вспомогательные свойства:

* `HasValue` – указывает, имеет узел текстовое значение (`Value`) или нет
* `HasAttributes` – указывает, имеет узел атрибуты или нет
* `IsEmptyElement` – содержит `true`, если элемент не имеет никаких данных

Если текущий узел содержит атрибуты (`HasAttributes = true`), то получить их можно при помощи функции `MoveToNextAttribute`, которая, по аналогии с `Read`, будет возвращать `true`, до достижения конца определения элемента.

Имя атрибута и значение также будут находиться в свойствах `Name` и `Value`.

```c#
if (xml.HasAttributes)
{
  while (xml.MoveToNextAttribute())
  {
   Console.WriteLine("Атрибут: {0}", xml.Name);
   Console.WriteLine("Значение: {0}", xml.Value);
  }
}
```

После считывания атрибутов может потребоваться вернуться назад к элементу, для этого существует метод `MoveToElement`.

```c#
if(xml.MoveToNextAttribute())
{
  Console.WriteLine("Атрибут: {0}", xml.Name);
  xml.MoveToElement();
  Console.WriteLine("Элемент: {0}", xml.Name);
}
```

Для упрощения обработки документа есть несколько полезных функций.

Методы `ReadOuterXml` и `ReadInnerXml` позволяют получить фрагменты **xml** в виде текста, как есть.

Так, например, найдя в документе элемент `member`, при помощи функции `ReadOuterXml` можно получить его целиком и загрузить в `XmlDocument`, т.е. обработать данные на более высоком уровне, а не собирать их по крупинкам, как в продемонстрированном ранее примере.

```c#
if(xml.NodeType == XmlNodeType.Element)
{
  if (xml.Name == "member")
  {
    XmlDocument xmlDoc = new XmlDocument();
    xmlDoc.LoadXml(xml.ReadOuterXml());

    XmlNode n = xmlDoc.SelectSingleNode("member");

    Console.WriteLine("KUID: {0}", n.Attributes["kuid"].Value);
    Console.WriteLine("Псевдоним: {0}", n.SelectSingleNode("nickname").InnerText);
    Console.WriteLine("Имя: {0}", n.SelectSingleNode("firstName").InnerText);
    Console.WriteLine("Фамилия: {0}", n.SelectSingleNode("lastName").InnerText);
    Console.WriteLine("------------------------------------------");
  }
}
```

А при желании, можно вообще сделать класс `Member` и создать его экземпляр на основе полученной из **xml** информации при помощи механизмов десериализации **.NET Framework**.

```c#
[XmlRoot("member")]
public class Member
{

  [XmlAttribute("kuid")]
  public int KUID { get; set; }

  [XmlElement("nickname")]
  public string Nickname { get; set; }

  [XmlElement("firstName")]
  public string FirstName { get; set; }

  [XmlElement("lastName")]
  public string LastName { get; set; }

  public Member() {}

  public Member(string xml)
  {
    LoadXml(xml);
  }

  public void LoadXml(string source)
  {
    XmlSerializer mySerializer = new XmlSerializer(this.GetType());

    using (MemoryStream ms = new MemoryStream(Encoding.UTF8.GetBytes(source)))
    {
      object obj = mySerializer.Deserialize(ms);

      foreach (PropertyInfo p in obj.GetType().GetProperties())
      {
        PropertyInfo p2 = this.GetType().GetProperty(p.Name);

        if (p2 != null && p2.CanWrite)
        {
          p2.SetValue(this, p.GetValue(obj, null), null);
        }
      }
    }
  }
}
```

> [!NOTE]
> Для использования классов xml-сериализации необходимо импортировать в проект пространство имен `System.Xml.Serialization`.
> Помимо этого, для работы функции `LoadXml` может потребоваться импортировать пространства имен `System.IO` и `System.Reflection`.

Таким образом, обработка данных выходит на еще более высокий уровень.

```c#
using (XmlReader xml = XmlReader.Create("XmlFile1.xml"))
{
  while (xml.Read())
  {
    switch (xml.NodeType)
    {
      case XmlNodeType.Element:
        // нашли элемент member
        if (xml.Name == "member")
        {
          // передаем данные в класс Member
          Member m = new Member(xml.ReadOuterXml());
          Console.WriteLine("KUID: {0}", m.KUID);
          Console.WriteLine("Псевдоним: {0}", m.Nickname);
          Console.WriteLine("Имя: {0}", m.FirstName);
          Console.WriteLine("Фамилия: {0}", m.LastName);
          Console.WriteLine("------------------------------------------");
        }
        break;
    }
  }
}
```

При этом уровень обработки данных повышается не для всего содержимого **xml**, объем которого может достигать многих гигабайт, а лишь для его небольшого фрагмента.

Потребление памяти, да и в целом нагрузка на компьютер, незначительные.

А вот что делать, если элемент member будет содержать большие объемы вложенной информации.

Например, все сообщения пользователя с форумов **Kbyte.Ru**?

```xml
<kbyte>
  <members>
    <member kuid="1">
      <nickname>Алексей</nickname>
      <firstName>Алексей</firstName>
      <lastName>Немиро</lastName>
      <messages>
        <message id="1">
          <subject>Привет мир!</subject>
          <text>Это тестовое сообщение.</text>
        </message>
        <message id="2">
          <subject>Как работать с XmlReader?</subject>
          <text>Это текст о том, как работать с XmlReader.</text>
        </message>
      </messages>
    </member>
    <member kuid="288">
      <nickname>Игорь Голов</nickname>
      <firstName>Игорь</firstName>
      <lastName>Голов</lastName>
    </member>
    <member kuid="1858">
      <nickname>[i]Pro</nickname>
      <firstName>Артем</firstName>
      <lastName>Донцов</lastName>
      <messages>
        <message id="3">
          <subject>Еще одно сообщение</subject>
          <text>Это текст еще одного тестового сообщения.</text>
        </message>
      </messages>
    </member>
    <member kuid="2575">
      <nickname>Shark1</nickname>
      <firstName>Виталий</firstName>
      <lastName />
      <messages />
    </member>
  </members>
</kbyte>
```

В этом документе, каждый элемент `member` содержит один вложенный элемент `messages`, который в свою очередь может состоять из неограниченного числа элементов `message`.

Если получать все содержимое узла `member`, то потребление оперативной памяти может быть высоким.

Поэтому использование функции `ReadOuterXml` для этих целей не годится.

В подобных случаях нужно разделить логику обработки элементов `member` и `messages`.

Таким образом, чтобы элемент member содержал только основные данные о пользователе, как и в первых примерах, исключив из него элемент `messages`.

Но и список сообщений никуда не пропадает, он просто будет обработан отдельно, причем ничто не запрещает использовать для этого метод `ReadOuterXml`.

Чтобы исключить объемные фрагменты **xml**, необходимо записывать в память только то, что нужно.

Для этого потребуется создать экземпляр класса `XmlWriter`.

Запись данных проще всего производить в `StringBuilder`, хотя можно и в поток (`Stream`).

Также потребуется две дополнительных булевых (логических) переменных, одна из которых будет разрешать запись данных в экземпляр `XmlWriter`, а вторая – информировать об исключении **xml** элемента.

```c#
StringBuilder sb = null;
XmlWriter w = null;
XmlWriterSettings ws = new XmlWriterSettings() { Encoding = Encoding.UTF8 };
bool isMember = false, isSkiped = false;
```

Пропустить ненужный фрагмент **xml** можно при помощи метода `Skip`.

```c#
using (XmlReader xml = XmlReader.Create("XmlFile1.xml"))
{
  while (xml.Read())
  {
    switch (xml.NodeType)
    {
      case XmlNodeType.Element:
        // нашли элемент member
        if (xml.Name == "member")
        {
          isMember = true;

          // в памяти есть данные пользователя
          if (sb != null)
          {
            w.Flush();
            w.Close();

            // обрабатываем их
            XmlDocument xmlDoc = new XmlDocument();
            xmlDoc.LoadXml(sb.ToString());

            XmlNode n = xmlDoc.SelectSingleNode("member");
            Console.WriteLine("KUID: {0}", n.Attributes["kuid"].Value);
            Console.WriteLine("Псевдоним: {0}", n.SelectSingleNode("nickname").InnerText);
            Console.WriteLine("Имя: {0}", n.SelectSingleNode("firstName").InnerText);
            Console.WriteLine("Фамилия: {0}", n.SelectSingleNode("lastName").InnerText);
            Console.WriteLine("------------------------------------------");
        }

        // создаем новый XmlWirter, для записи данных пользователя
        sb = new StringBuilder();
        w = XmlWriter.Create(sb, ws);
        w.WriteProcessingInstruction("xml", "version=\"1.0\" encoding=\"utf-8\"");
      }
      else if (xml.Name == "messages")
      {
        // пропускаем фрагмент с сообщениями
        xml.Skip();

        // ставим отметку, что фрагмент пропущен
        isSkiped = true;
      }

      // это данные пользователя и не пропущенный фрагмент
      if (isMember && !isSkiped)
      {
        w.WriteStartElement(xml.Name);

        // если есть атрибуты, записываем их
        if (xml.HasAttributes)
        {
          while (xml.MoveToNextAttribute())
          {
            w.WriteAttributeString(xml.Name, xml.Value);
          }
        }
      }

      // убираем отметку о пропущенном фрагменте
      isSkiped = false;
      break;

    case XmlNodeType.Text:
      if (isMember)
      {
        w.WriteString(xml.Value);
      }
      break;

    case XmlNodeType.EndElement:
      if (isMember)
      {
        w.WriteFullEndElement();
      }

      if (xml.Name == "member")
      {
        isMember = false;
      }
      break;
    }
  }
}
```

По функционалу приведенный листинг ничем не отличается от предыдущих.

На выходе также будет получен фрагмент документа, содержащий только элемент `member`, исключая вложенный элемент `messages`.

Что касается обработки `messages`, то для этого проще всего сделать отдельный `XmlReader` при помощи функции `ReadSubtree`.

```c#
else if (xml.Name == "messages")
{
  XmlReader xml2 = xml.ReadSubtree();
  while (xml2.Read())
  {
    switch (xml2.NodeType)
    {
      case XmlNodeType.Element:
        // нашли элемент message
        if (xml2.Name == "message")
        {
          XmlDocument xmlDoc2 = new XmlDocument();
          xmlDoc2.LoadXml(xml2.ReadOuterXml());
          XmlNode n = xmlDoc2.SelectSingleNode("message");

          Console.WriteLine("Сообщение # {0}", n.Attributes["id"].Value);
          Console.WriteLine("Тема: {0}", n.SelectSingleNode("subject").InnerText);
          Console.WriteLine("Текст: {0}", n.SelectSingleNode("text").InnerText);
          Console.WriteLine("------------------------------------------");
        }

        break;
    }
  }

  // пропускаем фрагмент с сообщениями
  xml.Skip();

  // ставим отметку, что фрагмент пропущен
  isSkiped = true;
}
```

Таким образом, у нас будет отдельно обработан каждый элемент `member` и каждый связанный с ним `message`.

Это практически никак не отразится на потреблении ресурсов, каким бы большим не был размер xml-файла.

По завершению работы с документом, всегда необходимо закрывать поток при помощи метода `Close`.

> [!NOTE]
> При использовании инструкции `using { }`, закрытие потока происходит автоматически.
> Отдельный вызов метода `Close` может привести к возникновению исключения.

## Послесловие

Логика работы с `XmlReader` заметно отличается от `XmlDocument` и `XPathDocument`.

И хотя код может показаться сложным, `XmlReader` имеет существенные преимущества при обработке xml-данных большого объема.

А благодаря последовательности чтения документов, `XmlReader` идеально подходит для работы с поврежденным потоком данных.

Правильно комбинируя `XmlReader` с другими инструментами работы с **XML** в **.NET Framework** можно значительно упростить процесс обработки xml-содержимого.

Ниже вы найдете файл, содержащий приведенные в статье примеры.

Проект выполнен в виде консольного приложения **C# 3.5**, хотя код без проблем должен работать, как в ранних версиях **.NET Framework**, так и в новых.

Если у вас возникнут какие-либо вопросы, обращайтесь на [форум Kbyte.Ru](http://kbyte.ru/ru/Forums?id=0).

**:floppy_disk: [Скачать XmlReadExample.zip (329 Кб)](assets/XmlReadExample.zip)**

---
Алексей Немиро  
2012-08-12
