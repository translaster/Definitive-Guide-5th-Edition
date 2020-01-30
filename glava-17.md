# Глава 17. AMI и файлы вызовов

> Джон Малкович: я видел мир, который не должен видеть ни один человек!
> Крейг Шварц: Правда? Потому что для большинства людей это довольно приятный опыт.
> -- Быть Джоном Малковичем

Интерфейс Asterisk Manager (Asterisk Manager Interface - AMI) - это интерфейс мониторинга и управления системой, предоставляемый Asterisk. Он позволяет в реальном времени отслеживать события, происходящие в системе, а также позволяет запрашивать Asterisk выполнение некоторых действий. Доступные действия имеют широкий диапазон и включают такие вещи, как возврат информации о состоянии или инициирование новых вызовов. На Asterisk было разработано много интересных приложений, использующих AMI в качестве основного интерфейса для Asterisk.

Эта глава также включает документацию по использованию файлов вызовов. Файлы вызовов Asterisk - это простой способ инициировать несколько вызовов. Как только объем исходящих вызовов увеличивается или ваши потребности становятся более сложными, вы можете перейти к использованию AMI. На самом деле, мы находим файлы вызовов достаточно полезными, так что сначала поговорим о них.

## Файлы вызовов

Обычно для инициализации вызовов используется AMI, но во многих ситуациях проще использовать файлы вызовов. Файл вызова - это простой текстовый файл, описывающий вызов, который вы хотите совершить через Asterisk. Когда файл вызова помещается в каталог _/var/spool/asterisk/outgoing_, Asterisk немедленно обнаружит, что файл был помещен туда, и обработает вызов.

Asterisk поставляется с образцом файла вызова, который вы найдете в _~/src/asterisk-15.\<TAB\>/sample.call_ (или там, где находится корневой каталог исходников Asterisk).

### Ваш первый файл вызова

Для вашего первого файла вызова давайте создадим вызов между двумя вашими телефонами. Убедитесь, что хотя бы два ваших телефона зарегистрированы и работают. Для этого примера мы будем использовать `SOFTPHONE_A` и `SOFTPHONE_B`.

Создайте в домашнем каталоге следующий файл:

```
$ vim ~/call-file

Channel: PJSIP/SOFTPHONE_A
Extension: 103
Context: sets
```

Сделайте копию этого файла (так что вам не придется заново создавать его каждый раз, когда захотите запустить его):

```
$ cp ~/call-file docall
```

Измените владельца файла docall на `asterisk`:

```
$ chown asterisk:asterisk docall
```

Переместите файл _docall_ в каталог _outgoing_ Asterisk.

```
$ sudo mv docall /var/spool/asterisk/outgoing
```

Иногда самый простой способ - лучший способ.

<table border="1" width="100%" cellpadding="5">
  <tr>
    <td>
      <p>Вы, вероятно, обнаружите, что делаете несколько правок в исходном файле вызова. Вы можете просто переместить созданный файл, а не делать его копию, но тогда вам придется заново создавать его каждый раз, когда вы его редактируете, и это раздражает. Весь этот набор можно сохранить как однострочный и запустить следующим образом:</p>
<p><pre><code>$ cp ~/call-file docall \
sudo chown asterisk:asterisk docall \
sudo mv docall /var/spool/asterisk/outgoing/</code></pre></p>
      <p>Попробуйте, и вы увидите, насколько это проще, чем каждый раз создавать и перемещать новый файл вызова.</p>
    </td>
  </tr>
</table>

<table border="1" width="100%" cellpadding="5">
  <tr>
    <td>
      <p align="left"><b>Предупреждение</b></p>
      <p>Использование <code>mv</code> вместо <code>cp</code> здесь важно. Asterisk следит за тем, чтобы содержимое отображалось в каталоге <i>spool</i>. Если вы используете копирование - Asterisk может попытаться прочитать новый файл до того, как содержимое будет скопировано в него. Создание файла, а затем его перемещение позволяет избежать этой проблемы.</p>
    </td>
  </tr>
</table>

Освойтесь с использованием файлов вызовов и вы обнаружите что они решают проблемы, которые в противном случае вам пришлось бы решать гораздо большим объемом работ.

### Заметки о файлах вызова

Компонент `Channel` файла вызова является обязательным. Обычно вызов, поступающий в Asterisk, инициируется конечной точкой (например, вы делаете вызов со своего телефона). В файле вызова это соединение должно происходить наоборот - Asterisk обращается к конечной точке, и только когда она отвечает, вызов может начаться. Планируйте соответственно.

Вы также должны указать `Context`, в котором вызов начнется, как только первоначальный канал ответит. Это может быть полезно, так как это означает, что вы можете подключить вызов через контекст, который обычно недоступен для этого канала, но на практике мы бы предложили вам просто использовать тот же контекст, через который канал вошел бы в диалплан, если бы он инициировал вызов как обычно.

Расширение, конечно, также должно быть указано. Обычно это номер телефона, по которому нужно позвонить, но, конечно, это может быть любой допустимый добавочный номер в `Context`.

Остальные параметры файла вызова являются необязательными и подробно описаны в файле _~/src/asterisk-15.\<TAB\>/sample.call_ и на веб-сайте Asterisk wiki.

## AMI Быстрый старт

Этот раздел предназначен для того, чтобы как можно быстрее испачкать руки с помощью AMI. Во-первых, поместите следующую конфигурацию в _/etc/asterisk/manager.conf_:

```
; Включить AMI и указать ему принимать соединения только от localhost.
[general]
enabled = yes
webenabled = yes
bindaddr = 127.0.0.1

; Создайть аккаунт с именем "hello" и паролем "world"
[hello]
secret=world
read=all     ; Получать все типы событий
write=all    ; Разрешить этому пользователю выполнять все действия
```

<table border="1" width="100%" cellpadding="5">
  <tr>
    <td>
      <p align="left"><b>Примечание</b></p>
      <p>Этот пример конфигурации настроен так, чтобы разрешить только локальные подключения к AMI. Если вы собираетесь сделать этот интерфейс доступным по сети, настоятельно рекомендуется использовать только протокол TLS. Использование TLS более подробно рассматривается далее в этой главе.</p>
    </td>
  </tr>
</table>

Как только конфигурация AMI готова, включите встроенный HTTP-сервер, поместив следующее содержимое в _/etc/asterisk/http.conf_:

```
; Включить встроенный HTTP-сервер и слушайть только соединения на localhost.
[general]
enabled = yes
bindaddr = 127.0.0.1
```

Перезагрузите диспетчер и http-серверы из Asterisk CLI:

```
*CLI> manager reload

*CLI> module reload http
```

### AMI через TCP

Существует несколько способов подключения к AMI, но наиболее распространенным является TCP-сокет. Мы будем использовать `telnet` для демонстрации подключения AMI. Для этого нам нужно будет установить `telnet`:

```
$ sudo yum -y install telnet
```

В этом примере показаны следующие шаги:
* Подключение к AMI через TCP-сокет на порту 5038.
* Вход в систему, используя действие `Login`.
* Выполнение действия `Ping`.
* Выход из системы с помощью действия `Logoff`.

Вот как это сделать с помощью `telnet`:

```
$ telnet localhost 5038

Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
Asterisk Call Manager/4.0.3
```

Вы подключились, но он будет висеть на вас, если вы не подтвердите свою подлинность. Вставьте в окно `telnet` следующее:

```
Action: Login
Username: hello
Secret: world
```

Обратите внимание, что после команд должна быть пустая строка (нажмите Enter после вставки всего, если ничего не происходит).

```
Response: Success
Message: Authentication accepted
```

Ладно, мы ему нравимся. Давайте выполним простую команду, чтобы убедиться, что он действительно говорит с нами:

```
Action: Ping
```

```
Response: Success
Ping: Pong
```

Все идет нормально. Мы просто уберемся и выйдем сейчас.

```
Action: Logoff
```

```
Response: Goodbye
Message: Thanks for all the fish.
Connection closed by foreign host.
```

Вы убедились что AMI принимает соединения через TCP-соединение.

### AMI через HTTP

Также можно использовать AMI через HTTP. Мы будем выполнять те же действия что и раньше, но через HTTP вместо собственного TCP-интерфейса к AMI. АMI через HTTP подробно описаны в [“AMI через HTTP”](#AMI-HTTP).

<table border="1" width="100%" cellpadding="5">
  <tr>
    <td>
      <p align="left"><b>Примечание</b></p>
      <p>Учетные записи, используемые для подключения к AMI через HTTP, являются теми же учетными записями, настроенными в файле <i>/etc/asterisk/manager.conf</i>.</p>
    </td>
  </tr>
</table>

В этом примере показано, как получить доступ к AMI по протоколу HTTP, войти в систему, выполнить действие `Ping` и выйти из системы:

```
$ curl "http://localhost:8088/rawman?action=login&username=hello&secret=world" \
-c /tmp/tempcookie

Response: Success
Message: Authentication accepted
```

```
$ curl "http://localhost:8088/rawman?action=ping" -b /tmp/tempcookie

Response: Success
Ping: Pong
Timestamp: 1538871944.474131
```

```
$ curl "http://localhost:8088/rawman?action=logoff" -b /tmp/tempcookie

Response: Goodbye
Message: Thanks for all the fish.
```

Интерфейс HTTP для AMI позволяет интегрировать управление вызовами Asterisk в веб-службу.

## Конфигурация

Раздел ["AMI быстрый старт"](glava-17.md#ami-быстрый-старт) показал очень простой набор конфигурационных файлов для начала работы. Существует много способов тонкой настройки конфигурации AMI.

### manager.conf

Основной конфигурационный файл для AMI - это _/etc/asterisk/manager.conf_. Раздел `[general]` содержит параметры, управляющие общей работой AMI. Любые другие разделы в _manager.conf_ определяют учетные записи для входа в систему и использования AMI. Пример файла содержит подробные объяснения различных параметров и может быть найден в  _~/src/asterisk-15\<TAB\>/configs/samples/manager.conf.sample_.

<table border="1" width="100%" cellpadding="5">
  <tr>
    <td>
      <p align="left"><b>Предупреждение</b></p>
      <p>Если вы собираетесь выставить свой AMI за пределы машины, на которой он работает, вам потребуется настроить подключение TLS.</p>
    </td>
  </tr>
</table>

Конфигурационный файл _manager.conf_ также содержит конфигурацию учетных записей пользователей AMI. Вы создаете учетную запись, добавляя раздел с именем пользователя в квадратных скобках. В каждом разделе `[username]` есть параметры, которые могут быть установлены, которые будут применяться только к этой учетной записи. Файл _~/src/asterisk-15\<TAB\>/configs/samples/manager.conf.sample_ также содержит подробные объяснения каждого из этих параметров. Наш пользователь по имени `[hello]`, имеет простейшую конфигурацию, которая позволяет все операции чтения и записи. Обычно следует создавать пользователей AMI, которые ограничены только действиями, необходимыми для их функционирования.

В разделе `[username]` параметры `read` и `write` определяют к каким действиям и событиям диспетчера имеет доступ конкретный пользователь. На данный момент есть 20 из них: `all`, `system`, `call`, `log`, `verbose`, `agent`, `user`, `config`, `command`, `dtmf`, `reporting`, `cdr`, `dialplan`, `originate`, `agi`, `cc`, `aoc`, `test`, `security` и `message`. Вы увидите что файл _manager.conf.sample_ содержит ссылку на каждый из них, относящийся к вашему выпуску (и, если какие-либо из них добавлены, которые не были перечислены здесь, они будут в файле примера).

<table border="1" width="100%" cellpadding="5">
  <tr>
    <td>
      <p align="left"><b>Предупреждение</b></p>
      <p>Обратите особое внимание на разрешения <code>system</code>, <code>command</code> и <code>originate</code>. Эти разрешения предоставляют значительные полномочия всем приложениям, которые имеют право их использовать. Предоставляйте эти разрешения только приложениям, над которыми у вас есть полный контроль (и в идеале они работают в одном окне).</p>
    </td>
  </tr>
</table>

### http.conf

Как мы уже видели, интерфейс Asterisk Manager может быть доступен как по протоколу HTTP, так и по протоколу TCP. Для этого в Asterisk встроен очень простой HTTP-сервер. Все параметры, относящиеся к AMI, находятся в разделе [general] файла _/etc/asterisk/http.conf_.

<table border="1" width="100%" cellpadding="5">
  <tr>
    <td>
      <p align="center"><b>Примечание</b></p>
      <p>Включение доступа к AMI по протоколу HTTP требует наличия <i>/etc/asterisk/manager.conf</i> и <i>/etc/asterisk/http.conf</i>. AMI должен быть включен в <i>manager.conf</i> с параметром <code>enabled</code>, установленным в <code>yes</code> и <code>webenabled</code> должен быть установлен в значение <code>yes</code> чтобы разрешить доступ по протоколу HTTP. Наконец, опция <code>enabled</code> в <i>http.conf</i> должна быть установлена в <code>yes</code> чтобы включить сам HTTP-сервер.</p>
    </td>
  </tr>
</table>

Доступные опции будут найдены в вашем файле _~/src/asterisk-15\<TAB\>/configs/samples/http.conf.sample_.

## Обзор протокола

В AMI есть два основных типа сообщений: события диспетчера и действия диспетчера.

_События диспетчера_ - это односторонние сообщения, посылаемые Asterisk клиентам AMI для сообщения о том, что произошло в системе (Рисунок 17-1).

![Рисунок 17-1. События диспетчера](/pics/pic17-1.png)

_Рисунок 17-1. События диспетчера_

_Действия диспетчера_ - это запросы от клиента к Asterisk для выполнения некоторого действия и возврата результата (Рисунок 17-2). Например, действие AMI инициирует запросы, чтобы Asterisk создал новый вызов, и, естественно, клиентскому приложению потребуются ответы от Asterisk, чтобы указать ход выполнения этого действия.

![Рисунок 17-2. Действия диспетчера](/pics/pic17-2.png)

_Рисунок 17-2. Действия диспетчера_

Другие действия менеджера - это запросы данных. Например, есть действие - получить список всех активных каналов в системе: сведения о каждом канале доставляются как событие. Когда список результатов будет завершен, будет отправлено окончательное сообщение о том, что цель достигнута. См. Рисунок 17-3 для графического представления клиента, отправляющего этот тип управляющего действия и получающего список ответов.

![Рисунок 17-3. Действия диспетчера возвращающие список данных](/pics/pic17-3.png)

_Рисунок 17-3. Действия диспетчера возвращающие список данных_

### Кодировка сообщений

Все сообщения AMI, включая события, действия и ответы на действия, кодируются одинаково. Сообщения являются текстовыми, со строками, заканчивающимися возвратом каретки и символом перевода строки. Сообщение завершается пустой строкой:

```
Header1: This is the first header<CR><LF>
Header2: This is the second header<CR><LF>
Header3: This is the last header of this message<CR><LF>
<CR><LF>
```
Если вы запускаете тесты из telnet-клиента - это означает, что после последней строки инструкций вам нужно будет дважды нажать клавишу Enter.


#### События

События всегда имеют заголовок `Event` и заголовок `Privilege`. В заголовке `Event` указывается имя события, а в заголовке `Privilege` - уровни разрешений, связанные с данным событием. Любые другие заголовки, включенные в событие, являются специфичными для данного типа события. Вот вам пример:

```
Event: Hangup
Privilege: call,all
Channel: SIP/0004F2060EB4-00000000
Uniqueid: 1283174108.0
CallerIDNum: 2565551212
CallerIDName: Russell Bryant
Cause: 16
Cause-txt: Normal Clearing
```

CLI Asterisk включает в себя `manager show events` и `manager show event <event>`. Выполните эти команды в CLI Asterisk, чтобы получить список событий или узнать подробности конкретного события.

Не забывайте, что отличным справочником для всех вещей Asterisk, включая AMI, является официальная [Asterisk wiki](https://wiki.asterisk.org/).

#### Действия

При выполнении действия _необходимо_ включить заголовок `Action`. Заголовок `Action` определяет, какое действие выполняется. Остальные заголовки являются аргументами для действия и могут потребоваться или не потребоваться в зависимости от действия.

Чтобы получить список заголовков, связанных с определенным действием, введите в CLI Asterisk команду `manager show command <Action>`. Чтобы получить полный список действий, поддерживаемых используемой версией Asterisk, введите `manager show commands`.

Окончательный ответ на действие обычно представляет собой сообщение, содержащее заголовок `Response`. Значение заголовка `Response` будет `Success`, если действие было выполнено успешно. Если действие не было успешно выполнено, то значение заголовка ответа будет `Error`. Например:

```
Action: Login
Username: hello
Secret: world

Response: Success
Message: Authentication accepted
```

### AMI через HTTP <a name="AMI-HTTP"></a>

Помимо собственного TCP-интерфейса, можно также получить доступ к AMI по протоколу HTTP. Программисты с имеющимся опытом написания приложений, использующие веб-API, скорее всего предпочтут его по сравнению с подключением TCP. В то время как интерфейс TCP предлагает только один тип структуры сообщений, AMI через HTTP предлагает несколько вариантов кодирования. Вы можете получать ответы в том же формате что и в TCP, в формате XML или в виде базовой HTML-страницы. Тип кодировки выбирается на основе поля в URL запросе. Варианты кодирования рассматриваются более подробно далее в этом разделе.

#### Аутентификация и обработка сессии

Существует два метода выполнения аутентификации против AMI через HTTP. Первый - это использование действия `Login`, аналогичного аутентификации с помощью собственного интерфейса TCP. Это метод, который использовался в Примере быстрого запуска, как показано в [AMI через HTTP](glava17.md#ami-через-http).

После успешной аутентификации Asterisk предоставит файл cookie, который идентифицирует аутентифицированный сеанс. Вот пример ответа на действие `Login`, которое включает в себя файл cookie сеанса от Asterisk:

```
$ curl -v "http://localhost:8088/rawman?action=login&username=hello&secret=world"
```

Второй вариант аутентификации - это HTTP-дайджест аутентификации. В этом примере запрошенный тип кодировки, основанный на URL-запросе, является `rawman`. Чтобы указать, что следует использовать дайджест аутентификацию HTTP, префикс типа кодировки в URL-адресе запроса должен содержать `a`:

```
$ curl -v --digest -u hello:world http://127.0.0.1:8088/arawman?action=ping
```

#### Кодирование /rawman (/arawman)

Тип кодирования `rawman` - это то, что до сих пор использовалось во всех примерах AMI через HTTP в этой главе. Ответы, полученные от запросов, использующих `rawman`, форматируются точно так же, как они были бы, если бы запросы были отправлены по прямому TCP-соединению к AMI.

```
curl -v "http://localhost:8088/rawman?action=login&username=hello&secret=world"

curl -v --digest -u hello:world http://127.0.0.1:8088/arawman?action=ping
```

#### /manager (/amanager) encoding

The manager encoding type provides a response in simple HTML form. This interface is primarily useful for experimenting with the AMI:

$ curl -v "http://localhost:8088/manager?action=login&username=hello&secret=world"

$ curl -v --digest -u hello:world http://localhost:8088/amanager?action=ping

#### /mxml (/amxml) encoding

The mxml encoding type provides responses to manager actions encoded in XML:

$ curl -v "http://localhost:8088/mxml?action=login&username=hello&secret=world"

$ curl -v --digest -u hello:world http://localhost:8088/amxml?action=ping

#### Manager events

When connected to the native TCP interface for the AMI, manager events are delivered asynchronously. When using the AMI over HTTP, you must retrieve events by polling for them. You retrieve events over HTTP by executing the WaitEvent manager action. The following example shows how events can be retrieved using the WaitEvent manager action. The steps are:

1. Start an HTTP AMI session using the Login action.
2. Register a SIP phone to Asterisk to generate a manager event.
3. Retrieve the manager event using the WaitEvent action.

The interaction looks like this:

$ wget --save-cookies cookies.txt \

&gt; "http://localhost:8088/mxml?action=login&username=hello&secret=world" -O -

&lt;ajax-response&gt;

&lt;response type='object' id='unknown'&gt;

 &lt;generic response='Success' message='Authentication accepted' /&gt;

&lt;/response&gt;

&lt;/ajax-response&gt;

$ wget --load-cookies cookies.txt \

&lt; "http://localhost:8088/mxml?action=waitevent" -O -

&lt;ajax-response&gt;

&lt;response type='object' id='unknown'&gt;

 &lt;generic response='Success' message='Waiting for Event completed.' /&gt;

&lt;/response&gt;

&lt;response type='object' id='unknown'&gt;

 &lt;generic event='PeerStatus' privilege='system,all'

 channeltype='SIP' peer='SIP/0000FFFF0004'

 peerstatus='Registered' address='172.16.0.160:5060' /&gt;

&lt;/response&gt;

&lt;response type='object' id='unknown'&gt;

 &lt;generic event='WaitEventComplete' /&gt;

&lt;/response&gt;

&lt;/ajax-response&gt;

You’ll need to develop mechanisms in your application to ensure that buffered events are frequently polled.

## Example Usage

Most of this chapter so far discussed the concepts and configuration related to the AMI. This section will provide some example usage.

### Originating a Call

The AMI has the Originate manager action that can be used to originate a call. Many of the accepted headers are the same as the options placed in call files. [Table 17-1](17.%20Asterisk%20Manager%20Interface%20and%20Call%20Files%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22AMI_originate_headers) lists the headers accepted by the Originate action.

Table 17-1. Headers for the Originate action

| Option | Example value | Description |
| :--- | :--- | :--- |
| ActionID | a3a58876-f7c9-4c28-aa97-50d8166f658d | This header is accepted by most AMI actions. It is used to provide a unique identifier that will also be included in all responses to the action. It gives you a way to identify which request a response is associated with. This is important since all actions, their responses, and events are all transmitted over the same connection \(unless using AMI over HTTP\). |
| Channel | SIP/myphone | This header is critical and must be specified. This describes the outbound call that will be originated. The value is the same syntax that would be used for the channel argument to the Dial\(\) application in the dialplan. |
| Context | default | This header is used to specify a location in the dialplan to start executing once the outbound call has answered. The Context, Exten, and Priority headers must be used together. When using these headers, the Application and Data headers should not be used. |
| Exten | s | See the documentation for the Context header. |
| Priority | 1 | See the documentation for the Context header. |
| Application | ConfBridge | The Application and Data headers can be used instead of the Context, Exten, and Priority headers. In this case, the outbound call is directly connected to a single application once the call has been answered. |
| Data | 500 | See the documentation for the Application header. |
| Timeout | 30000 | This header specifies how long to wait in milliseconds for an answer before giving up on an outbound call. The default is 30000 milliseconds \(30 seconds\). |
| CallerID | Matthew Jordan &lt;\(555\) 867-5309&gt; | This header can be used to specify the caller ID used for the outbound call. |
| Account | someaccount | This header sets the CDR account code for the outbound call. |
| Variable | VARIABLE=VALUE or FUNCTION\(arguments\)=VALUE | The Variable header can be used to set both channel variables or channel functions on the outbound channel. It can be specified multiple times. |
| Codecs | ulaw,alaw | This option can be used to limit which codecs are allowed for the outbound call. If not specified, the set of codecs configured in the channel driver configuration file will still be honored. |
| EarlyMedia | true | If this header is specified and set to true, the outbound call will get connected to the specified extension or application as soon as there is any early media. |
| Async | true | If this header is specified and set to true, this call will be originated asynchronously. This will allow you to continue executing other actions on the AMI connection while the call is being processed. |

The simplest example of using the Originate action is via telnet:

$ telnet localhost 5038

Trying 127.0.0.1...

Connected to localhost.

Escape character is '^\]'.

Asterisk Call Manager/4.0.3

Once the connection is established, you need to log in.

Action: Login

Username: hello

Secret: world

Response: Success

Message: Authentication accepted

Now you’re ready to originate your call. We’re doing essentially the same thing we did with the call file, only this time using the AMI:

Action: Originate

Channel: PJSIP/SOFTPHONE\_A

Context: sets

Exten: 103

Priority: 1

You should hear SOFTPHONE\_A ringing. As soon as you answer it, a call will be placed to SOFTPHONE\_B.

AMI is no longer involved in what’s going on. You can disconnect and the call will continue \(leave the call up for now, as we’re going to work with the in-progress call next\).

Action: Logoff

Response: Goodbye

Message: Thanks for all the fish.

Connection closed by foreign host.

If you’ve already hung up the call, that’s no problem. You’ll just need to re-establish the call, which of course you can do simply by calling one extension from the other \(101 to 103, or whatever you want\).

### Redirecting a Call

Redirecting \(or transferring\) a call from the AMI is another feature worth mentioning. The Redirect AMI action can be used to send one or two channels to any other extension in the Asterisk dialplan. If you need to redirect two channels that are bridged together, do them both at the same time. Otherwise, once one channel has been redirected, the other will be hung up.

An important thing to understand about Asterisk channels is that they don’t exist until a call is in progress. The name we all think of as the channel name \(e.g., SOFTPHONE\_A\) is not in fact the channel name, but merely a reference to data that is used to create a channel. The naming of a channel takes place when a call is originated \(which is when the channel is actually created\). What all this means is that you have to determine the full name of the channel before you can act on it.

Originate a call, and then review the Event: Newchannel and you will see the channel name under the Channel: header.

Action: Originate

Channel: PJSIP/SOFTPHONE\_A

Context: sets

Exten: 103

Priority: 1

Response: Success

Message: Originate successfully queued

Event: Newchannel

Privilege: call,all

Channel: PJSIP/SOFTPHONE\_A-00000013

ChannelState: 0

ChannelStateDesc: Down

CallerIDNum: &lt;unknown&gt;

CallerIDName: &lt;unknown&gt;

ConnectedLineNum: &lt;unknown&gt;

ConnectedLineName: &lt;unknown&gt;

Language: en

AccountCode:

Context: sets

Exten: s

Priority: 1

Uniqueid: 1538939479.29

Linkedid: 1538939479.29

The Newchannel event will provide the name of the created channel, which in this example is PJSIP/SOFTPHONE\_A-00000013.

You will need to keep track of these channel names if you wish to properly perform actions on calls in progress. Once the call ends, the channel is destroyed. A new call using the same endpoint will be assigned a different channel name. One channel definition can support multiple calls \(for example, multiple calls to a phone are possible\), and this is why the channel name is different from the channel definition.

You can redirect a single channel \(the other channel will be disconnected\):

Action: Redirect

Channel: PJSIP/SOFTPHONE\_A-00000013

Exten: 209

Context: sets

Priority: 1

Or you can redirect two channels:

Action: Redirect

Channel: PJSIP/SOFTPHONE\_A-00000015

Context: sets

Exten: 209

Priority: 1

ExtraChannel: PJSIP/SOFTPHONE\_B-00000016

ExtraContext: sets

ExtraExten: 209

ExtraPriority: 1

The redirect function allows you to create powerful external applications that can control calls in progress.

## Development Frameworks

Many application developers write code that directly interfaces with the AMI. However, there are a number of frameworks that have been created with the purpose of making AMI application development easier. If you search for Asterisk frameworks in the popular programming language of your choice, you are likely to find one. The onus is on you to determine the suitability of the framework you are interested in. Some things you should look for in a framework include:

Maturity

Has this project been around for a few years? A mature project is far less likely to have serious bugs in it.

Maintenance

Check the age of the latest update. If the project hasn’t been updated in five years, there’s a strong possibility it has been abandoned. It might still be usable, but you’ll be on your own. Similarly, what does the bug tracker look like? Are there a lot of important bugs being ignored? \(Be discerning here, since often the realities of maintaining a free project require disciplined triage—not everybody’s features are going to get added.\)

Quality of the code

Is this a well-written framework? If it was not engineered well, you should be aware of that when deciding whether to trust your project to it.

Community

Is there an active community of developers using this project? It’s likely you’ll need help; will it be available when you need it?

Documentation

The code should be well commented, but ideally, a wiki or other official documentation to support the library is essential.

[Table 17-2](17.%20Asterisk%20Manager%20Interface%20and%20Call%20Files%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22AMI-frameworks) lists some frameworks we have found that, as of this writing, met the preceding criteria. There may be others out there.

Table 17-2. AMI development frameworks

| Framework | Language |
| :--- | :--- |
| Adhearsion | Ruby |
| StarPy | Python |
| Asterisk-Java | Java |
| AsterNET | .NET |
| ami-io | Node.js |
| panoramisk | Python |

## Conclusion

The Asterisk Manager Interface provides an API for monitoring events from an Asterisk system, as well as requesting that Asterisk perform a wide range of actions. An HTTP interface has been provided, and a number of frameworks have been developed, that make it easier to develop applications.

[Глава 16. Введение в интерактивное голосовое меню](glava-16.md) | [Содержание](SUMMARY.md) | [Глава 18. AGI](glava-18.md)
