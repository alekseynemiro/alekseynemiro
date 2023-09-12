# Автоматизация поиска SQL Injection

**SQL Injection** – это довольно распространенная уязвимость веб-сайтов, позволяющая злоумышленнику выполнять свой SQL-код путем подстановки его в ожидаемые параметры страницы.
Таким образом, злоумышленник может запросто получить доступ к секретным данным, добавить, изменить или удалить данные.

В данной статье я расскажу и покажу на примерах языков **PHP** и **C#**, как написать программку, которая будет автоматически сканировать заданные сайты на предмет наличия в них уязвимости типа **SQL Injection**.
Естественно, все это делается в целях устранения таких уязвимостей.

## Постановка задачи

Первым делом необходимо определиться с задачами.

Программа должна уметь:

1. Выполнять простейшие HTTP-запросы.
2. Получать список всех внутренних ссылок сайта.
3. Добавлять свой SQL-код в параметры адресов страниц.
4. Анализировать полученный результат.
5. Выполнить простейший HTTP-запрос несложно и в рамках данной статьи, дальше GET-запросов нам ходить не придется.

Список ссылок можно получить при помощи регулярных выражений.
Нас будут интересовать только ссылки с параметрами, поскольку именно через параметры будет производиться поиск уязвимостей.
Конечно же, поиск уязвимостей не ограничивается параметризированными адресами, уязвимость можно найти и при передаче данных через форму.
Особенно это касается элементов формы, которые пользователь якобы не может изменять, таких, как списки (`select`) или скрытые поля формы (`hidden`).
Но в рамках данной статьи я ограничусь лишь поиском уязвимостей в параметрах страницы.

В качестве SQL-кода, который будет добавляться в параметры страницы, можно использовать обычную одинарную кавычку (`'`), этого вполне достаточно, чтобы при наличии кривых рук у web-программистов сервер выдал какую-нибудь ошибку.
Довольно часто веб-программисты делают ошибки именно с кавычками, особенно часто подобные ошибки допускаются в типах данных, когда параметр вроде бы является числом, но в тоже время, за счет особенностей используемых технологий, может содержать строковые данные.

Подобными ошибками частенько страдают сайты, написанные на классическом **ASP** и **PHP**, обычно по причине неопытности или банальной лени программистов.
Стоит заметить, что в **ASP.NET** вероятность возникновения **SQL Injection** равна практически нулю, и чтобы создать подобную уязвимость, руки у программиста действительно должны расти совсем не оттуда, откуда обычно растут.
Учитывая это, пытаться найти **SQL Injection** на сайте, написанном на ASP.NET, особого смысла нет.

## Реализация

Итак, задача поставлена, приступим к реализации.

Первым делом напишем функцию, которая будет выполнять обычный GET-запрос.

**PHP:**
```php
function ExecuteHTTPGetRequest($url, $deleteAllTags)
{
  $url = parse_url($url); // разбираем url
  $host = $url["host"];
  $path = $url["path"];
  $req = "GET $path HTTP/1.1\r\nUser-Agent: Bulbulator\r\n"
       . "Host: $host\r\nConnection: Close\r\n\r\n";
  $fp = fsockopen($host, 80); // открываем сокетif (!$fp)
  {  // ошибкаreturn"Error.";
  }
  else
  {
    // отправляем запрос
    fputs($fp, $req); 
    // считываем полученные данные в переменную кусками по 256 байт
    $data = "";
    while (!feof($fp)) 
    {
      $data .= @fgetss($fp, 256, $deleteAllTags === true ? "<a>" : NULL);
    }
    fclose($fp); // закрываем сокетreturn $data; // возвращает полученные данные
  }
}
```

**C#:**
```c#
string ExecuteHTTPGetRequest(string url)
{
  try  // делаем запрос и возвращаем ответ
  { 
    HttpWebRequest req = (HttpWebRequest)HttpWebRequest.Create(url);
    HttpWebResponse rsp = (HttpWebResponse)req.GetResponse();
    using(
StreamReader sr = 
        new StreamReader(rsp.GetResponseStream(), 
Encoding.GetEncoding(1251)))
    {
      return sr.ReadToEnd();
    }
  }
  catch (WebException ex)
  {
    if (ex.Status == WebExceptionStatus.ProtocolError 
&& ex.Message.Contains("500"))
    { // если ошибка сервера, получаем ее описаниеusing (
StreamReader sr = new StreamReader(
          ex.Response.GetResponseStream(), Encoding.GetEncoding(1251)))
      {
        return sr.ReadToEnd();
      }
    }
  }
  returnstring.Empty; // другая ошибка
}
```

В **PHP** функция `ExecuteHTTPGetRequest` принимает два параметра.

Первый – адрес странички, которую нужно получить, а второй – булевский параметр, указывающий, нужно ли получить всю страницу, или только тэги `<a></a>`.

В **C#** функция принимает только адрес странички.

В отличие от **PHP**, в **C#** прописаны обработчики ошибок.

Кроме того, в **C#** (да и вообще в **.NET Framework**) может возникнуть ошибка типа `The server committed a protocol violation. Section=ResponseHeader Detail=CR must be followed by LF`, происходящая на почве неразберихи в стандартах.
Для её предотвращения необходимо в файл конфигурации проекта (`app.config` или `web.config`) добавить следующие строки:

```xml
<system.net>
  <settings>
    <httpWebRequest useUnsafeHeaderParsing="true" />
  </settings>
</system.net>
```

Далее, нам понадобится функция, которая будет искать и фильтровать адреса страниц. Для хранения адресов предусмотрены две переменные. Первая будет содержать список страниц, которые нужно просканировать, а вторая – список страниц с параметрами, которые мы будет проверять на наличие уязвимости.

**PHP:**
```php
var $taskUrls = array(); // очередь адресов
var $analysUrls = array(); // адреса для анализа
```

**C#:**
```c#
List<string> _taskUrls = new List<string>();
List<string> _analysUrls = new List<string>();
```

Как я уже ранее говорил, поиск ссылок будет производиться при помощи регулярных выражений.

Следует учесть, что ссылки могут содержать относительный путь, поэтому такие ссылки нужно будет преобразовать в обычный вид.

Также необходимо предусмотреть, что значения параметров страницы могут меняться, и дубликаты таких страниц нужно исключить из анализа.

**PHP:**
```php
function SearchUrls($host, $source)
{
  global $taskUrls, $analysUrls;
  // шаблон регулярного выражения для поиска URL
  $pattern = "#href\s*=\s*(\"|\\')?(.*?)(\"|\\'|\s|\>)#si";
  preg_match_all($pattern, $source, $mtchs);
  if (count($mtchs) < 3) return; // ничего не найдено, выходим
  foreach ($mtchs[2] as $k => $v)
  {
    // если это E-Mail или JavaScript, то пропускаем егоif (strpos(strtolower($v), "mailto:") === false 
      && strpos(strtolower($v), "javascript:") === false)
    { 
      $newUrl = $v; $isNew = true;
      if (ereg("[a-zA-Z]+:\/\/(.*)", $newUrl) === false)
      { // если адрес начинается не с http://, анализируем и добавляем http://        // меняем слеши, на всякий случай :)
        $newUrl = strtr(trim($newUrl), "\\", "/"); 
        $arrPath = explode("/", $newUrl); // разбиваем путь на части для анализа
        $newUrl = "";
        foreach($arrPath as $kk => $vv) 
        {
          if ($vv != ".") 
          {
            if (!$kk && (strlen($vv) > 1 && $vv[1] === ":" || $vv === "")) 
            {
              $newUrl = $vv;
            } 
            elseif ($vv === "..") 
            {
              if (strlen($newUrl) > 1 && $newUrl[1] === ":") 
              {
               continue;
              }
              $p = dirname($newUrl);
              if ($p === "/" || $p === "\\" || $p === ".") 
              {
                $newUrl = "";
              } 
              else 
              {
                $newUrl = $p;
              }
            } 
            elseif ($vv !== "") 
            {
              $newUrl .= "/$vv";
            }
          }
        }
        $newUrl = $newUrl !== "" ? $newUrl : "/";
        $newUrl = "http://".$host.$newUrl;
      }
      // если url не относится к текущему домену, то добавлять его не надо
      // если данная фишка нужна, уберите комментарий
      //$nu = parse_url($newUrl); 
      //$isNew = ($nu["</str>host"] === $host);
      // смотрим, может у нас такой адрес уже стоит в очереди
if ($isNew && $taskUrls != NULL) $isNew = 
!in_array($newUrl, $taskUrls);
      // добавляем в список заданий
if ($isNew) $taskUrls[] = $newUrl;
      // если у адреса есть параметры, 
      // добавляем в список для поиска SQL Injection
if (strpos($newUrl, "?") && !in_array($newUrl, $analysUrls))
      { // смотрим, чтобы в списка заданий не было такого же адреса        $isNew = true;
        foreach ($analysUrls as $k => $v)
        {
          if (strtolower(strrev(stristr(strrev($v), "?")))
 === strtolower(strrev(stristr(strrev($newUrl), "?")))
          { // такой уже есть, выходим
            $isNew = false;
            break;
          }
        }
        if ($isNew) $analysUrls[] = $newUrl;
      }
    }
  }
}
```

**C#:**
```c#
void SearchUrls(string host, string source)
{
  Regex rgx = new Regex(@"href\s*=\s*([\x22\x27]{0,1})(?<link>[^\s\n\x22\x27\x3E]*)");
  MatchCollection mtchs = rgx.Matches(source);
  if (mtchs == null || mtchs.Count <= 0) return; // ничего не найдено, выходимforeach (Match m in mtchs)
  {
    string newUrl = m.Groups["link"].Value;
    // если это E-Mail или JavaScript, то пропускаем егоif (!newUrl.ToLower().StartsWith("mailto:") && !newUrl.ToLower().StartsWith("javascript:"))
    { 
      if (!new Regex(@"[a-zA-Z]+:\/\/").IsMatch(newUrl))
      { // это относительный путьstring p 
= newUrl.IndexOf("?") != -1 
              ? newUrl.Substring(0, newUrl.IndexOf("?")) : newUrl;
        string q = newUrl.LastIndexOf("?") != -1 
? newUrl.Substring(newUrl.LastIndexOf("?"), 
                        newUrl.Length - newUrl.LastIndexOf("?")) 
                    : string.Empty;
        newUrl = new UriBuilder("http", host, 80, p, q).Uri.ToString();
      }
      bool isNew = true;
      // проверка хоста/* если данная фишка нужна, уберите комментарий
       isNew = (new Uri(newUrl).Host.ToLower() == host);
      */// проверка, может такая ссылка уже есть в списке заданийif (isNew && _taskUrls.IndexOf(newUrl) == -1)
      { // добавляем адрес в список заданий
        _taskUrls.Add(newUrl);
      }
      // если у адреса есть параметры, 
      // добавляем в список для поиска SQL Injectionif (isNew && newUrl.IndexOf("?") != -1)
      { 
        isNew = true;
        foreach (string u in _analysUrls)
        {
          if (new Uri(u).LocalPath == new Uri(newUrl).LocalPath)
          { // такой уже есть, выходим
            isNew = false;
            break;
          }
        }
        if (isNew) _analysUrls.Add(newUrl);
      }
    }
  }
}
```

Функция `SearchUrls` принимает два параметра.

Первый, `host`, содержит адрес сервера, а второй, source, содержит данные для анализа.

Параметр `host` необходим для конвертирования относительных путей в обычные, а также для фильтрации ссылок на другие домены.

Фильтрация ссылок на другие доменов в коде закомментирована, поэтому если вы хотите, чтобы обрабатывались только ссылки с конкретного домена, необходимо убрать комментарий.

Напоследок нужно написать самую главную функцию, которая будет «сканировать Интернет» и искать страницы, которые могут содержать уязвимость типа **SQL Injection**.

Процесс сканирования будет проходить в два этапа: первый – получение списка адресов, второй – поиск уязвимостей.

Конечно, все это дело можно объединить в один этап, но тогда код может получиться немного сложноватым для восприятия.

**PHP:**
```
function Process()
{
  global $taskUrls, $analysUrls;
  if (count($taskUrls) <= 0)
  {
    echo "Адреса для сканирования не найдены.";
    return;
  }
  // этап первый, сбор адресов  for ($i = 0; $i <= count($taskUrls); $i++)
  {
    $nu = parse_url($taskUrls[$i]);
    SearchUrls($nu["host"], ExecuteHTTPGetRequest($taskUrls[$i], true));
    if ($i > 49) break; // сканируем только 50 страниц
  }

  // этап второй, попытка найти уязвимости на страницах
  echo "Интернет просканирован. :) Найдено ссылок: ".count($analysUrls)
    ."<br />";
  // слова и сочетания слов, которые встречаются в сообщениях 
  // об ошибках при работы с БД
  $errorMessages = array( 
  "sql syntax", "sql error",
  "ole db error", "incorrect syntax", "unclosed quotation mark",
  "sql server error", "microsoft jet database engine error",
"'microsoft.jet.oledb.4.0' reported an error", "reported an error",
  "provider error", "oracle database error", "database error", 
  "db error", "syntax error in string in query expression",
  "ошибка синтаксиса", "синтаксическая ошибка", 
  "ошибка бд", "ошибочный запрос", "ошибка доступа к бд"
  );
  echo "<hr />";
  foreach ($analysUrls as $k => $v)
  {
    // особо фантазировать не будем, 
    // просто вляпаем одинарную кавычку сразу после знака "равно"
    echo "$k. Анализ ссылки $v ";
    $data = ExecuteHTTPGetRequest(str_replace("=", "='", $v), false);
    // с надеждой ищем сообщение об ошибке из словаря
    foreach ($errorMessages as $ek => $ev)
    {
      if (stripos($data, $ev) !== false)
      { // что-то найдено, ставим отметку
        echo " &nbsp;&nbsp; <span style='color:red'>
          <strong>SQL Injection!</strong></span>";
        break;
      }
    }
    echo "<br />";
  }
}
```

**C#:**
```c#
void Process()
{
  if (_taskUrls == null || _taskUrls.Count <= 0)
  {
    Console.WriteLine("Адреса для сканирования не найдены.");
    return;
  }
  // этап первый, сбор адресов  for (int i = 0; i <= _taskUrls.Count - 1; i++)
  {
    SearchUrls(new Uri(_taskUrls[i]).Host, 
      ExecuteHTTPGetRequest(_taskUrls[i]));
    if (i > 49) break; // сканируем только 50 страниц
  }
  // этап второй, попытка найти уязвимости на страницах
  Console.BackgroundColor = ConsoleColor.Black;
  Console.WriteLine(
    "Интернет просканирован. Найдено ссылок: {0}", _analysUrls.Count);
  // слова и сочетания слов, которые встречаются 
  // в сообщениях об ошибках при работе с БДstring[] errorMessages  = newstring[] 
{
    "sql syntax", "sql error",
    "ole db error", "incorrect syntax", "unclosed quotation mark",
    "sql server error", "microsoft jet database engine error",
"'microsoft.jet.oledb.4.0' reported an error", "reported an error",
    "provider error", "oracle database error", "database error", 
    "db error", "syntax error in string in query expression",
    "ошибка синтаксиса", "синтаксическая ошибка", 
    "ошибка бд", "ошибочный запрос", "ошибка доступа к бд"};
    foreach (string u in _analysUrls)
    {
      // особо фантазировать не будем, 
      // просто вляпаем одинарную кавычку сразу после знака "равно"string s = ExecuteHTTPGetRequest(u.Replace("=", "='"));
      // с надеждой ищем сообщение об ошибке из словаряbool sqlinj = false;
      foreach (string e in errorMessages)
      {
        // если что-то найдено, ставим отметкуif (s.IndexOf(e, StringComparison.OrdinalIgnoreCase) != -1)
        {
          sqlinj = true;
          break;
        }
      }
      if (sqlinj) 
        Console.BackgroundColor = ConsoleColor.Red; 
      else 
        Console.BackgroundColor = ConsoleColor.Black; 

      Console.WriteLine("Анализ ссылки {0}", u 
        + (sqlinj ? " >>>>> SQL Injection! <<<<<" : ""));
  }
}
```

Как видите, сначала сканируется список адресов, причем этот список может увеличиваться, т.к. программа получает новые ссылки с обрабатываемых страниц.

Чтобы не состариться, пока программа будет «сканировать Интернет» (как-никак сеть большая, страниц много, а мы отнюдь не **Google**), необходимо поставить ограничение на количество страниц.

В данном случае сканируется не более 50 страниц.

Далее я подставляю одинарную кавычку в параметры найденных страниц, делаю запрос и анализирую содержимое этих страниц.

В переменной `errorMessages` содержится список наиболее частых слов и словосочетаний, встречающихся в сообщениях об ошибках баз данных, именно эти слова я и буду искать в ответах сервера.

К слову, сайты, работающие на популярных движках, при получении необычного значения параметра могут показать сообщение о том, что была предотвращена попытка взлома, либо просто будут отдавать пустую страницу.

Собственно, все готово, осталось провести тесты.

Для этого достаточно в переменную `taskUrls` добавить хотя бы один адрес и вызвать функцию `Process`.

Следует отметить, что в **PHP** потребуется увеличить время выполнения скрипта, для этого достаточно воспользоваться функцией `set_time_limit`.

**PHP:**
```php
set_time_limit(0); // ставим максимальное время выполнения скрипта
$taskUrls[] = "http://kbyte.ru"; // добавляем url в список заданий
Process(); // запускаем процесс анализа
```

**C#:**
```c#
_taskUrls.Add("http://kbyte.ru"); // добавляем url в список заданий
Process(); // запускаем процесс анализа
```

Через поисковик я нашел пару сайтов с **SQL Injection**, не скрою, что искать пришлось долговато, в основном попадались обычные ошибки БД, из которых ничего интересного получить не удалось.

Тем не менее, программка работает, и может упростить работу по анализу уязвимости собственных проектов.

## Заключение

Конечно, код не идеальный, но общая идея, думаю понятна.

Если говорить о том, что неплохо было бы реализовать, то это такие мелочи, как анализ заголовков, главным образом, обработка ответов с кодом `301` и `302`.

В **PHP** для этого придется разобрать HTTP-заголовки, а в **C#** это можно сделать автоматически, при помощи свойства `AllowAutoRedirect`.

Также можно проверить тип содержимого, которое возвращает сервер, чтобы, например, не пытаться найти ссылки в графическом файле.

Что касается POST-запросов, в которых также можно найти немало дыр, то тут необходимо получить содержимое из тэгов `form`, найти в этом содержимом элементы управления и создать из них `url`, а в параметры подставить хотя бы старую-добрую одинарную кавычку.

С технической стороны это реализовать несложно, хотя кому-то может показаться немного муторно, т.к. придется учесть много различных факторов.

Собственно, если возникнут вопросы, меня всегда можно найти на форуме Kbyte.Ru.

**[:floppy_disk: Скачать пример на PHP](assets/sqlinj.php.zip)**  
**[:floppy_disk: Скачать пример на C#](assets/sqlinj.cs.zip)**  
**[:floppy_disk: Скачать пример на VB .NET](assets/sqlinj.vb.zip)**

---
Алексей Немиро  
2008-05-12

https://rsdn.org/article/inet/SqlInjectionSearch.xml