# Глава 18. AGI

> _Кофеин. Шлюз к наркотикам._
>
> -- Эдди Веддер

Диалплан Asterisk превратился в простой, но мощный программный интерфейс для обработки вызовов. Однако многие люди, особенно с опытом программирования, предпочитают реализовывать обработку вызовов на традиционном языке программирования. Asterisk Gateway Interface (AGI) позволяет разрабатывать управление вызовами от первого лица на выбранном вами языке программирования.

## Быстрый старт

В этом разделе приведен краткий пример использования AGI.

Во-первых, давайте создадим скрипт, который мы собираемся запустить. Скрипты AGI как правило помещаются в _/var/lib/asterisk/agi-bin_.

```text
$ cd /var/lib/asterisk/agi-bin
$ vim hello-world.sh
#!/bin/bash
# Consume all variables sent by Asterisk
while read VAR && [ -n ${VAR} ] ; do : ; done
# Answer the call.
echo "ANSWER"
read RESPONSE
# Say the letters of "Hello World"
echo 'SAY ALPHA "Hello World" ""'
read RESPONSE
exit 0
$ chown asterisk:asterisk hello-world.sh
$ chmod 700 hello-world.sh
```

Теперь добавьте следующую строку в _/etc/asterisk/extensions.conf_ в контекст `[sets]`:

```text
exten => 237,1,AGI(hello-world.sh)
```

Сохраните и перезагрузите свой диалплан и теперь, когда вы звоните на номер 237, то должны услышать, как Эллисон произносит “Hello World.”

## Варианты AGI

Существует несколько вариантов AGI, которые отличаются в первую очередь методом, используемым для связи с Asterisk. Полезно быть в курсе всех вариантов для совершения лучшего выбора, основанного на потребностях вашего приложения.

### Process-Based AGI

Process-based AGI (AGI на основе процесса) является простейшим вариантом AGI. Пример быстрого запуска в начале этой главы является примером сценария Process-based AGI. Скрипт вызывается с помощью приложения `AGI()` из диалплана Asterisk. Запускаемое приложение указывается в качестве первого аргумента функции `AGI()`. Если не указан полный путь - приложение должно находиться в каталоге _/var/lib/asterisk/agi-bin_. Аргументы, передаваемые приложению AGI, могут быть указаны в качестве дополнительных аргументов приложения `AGI()` в диалплане Asterisk. Синтаксис такой:

```text
AGI(command[,arg1[,arg2[,...]]])
```

<table border="1" width="100%" cellpadding="5">
  <tr>
    <td>
    <p><img src="pics/note.png" height="100" align="left">Убедитесь, что приложение имеет соответствующие разрешения на исполнение пользователем Asterisk. В противном случае функция <code>AGI()</code> завершится ошибкой.</p>
    </td>
  </tr>
</table>

---

Как только Asterisk выполнит ваше приложение AGI - связь между Asterisk и вашим приложением будет осуществляться через `stdin` и `stdout`. Более подробно об этом сообщении будет рассказано в разделе [“Обзор коммуникаций AGI”](glava-18.md#обзор-коммуникаций-agi). Для получения более подробной информации о вызове `AGI()` из диалплана проверьте документацию, встроенную в Asterisk:

```
*CLI> core show application AGI
```

**Плюсы Process-Based AGI (на основе процессов)**

Это самая простая форма реализации аги.

**Минусы Process-Based AGI**

Это наименее эффективная форма AGI с точки зрения потребления ресурсов. Вместо этого системы с высокой нагрузкой должны рассматривать FastAGI, описанный в разделе ["FastAGI - AGI через TCP"](glava-18.md#fastagi-agi-через-tcp).

#### EAGI

EAGI (Enhanced AGI - расширенный AGI) является легким вариантом `AGI()`. Он вызывается в диалплане Asterisk как `EAGI()`. Разница в том, что в дополнение к связи через `stdin` и `stdout` - Asterisk также обеспечивает однонаправленный аудиопоток, поступающий из канала на файловый дескриптор 3. Для получения более подробной информации о том, как вызвать `EAGI()` из диалплана Asterisk, проверьте документацию, встроенную в Asterisk:

```
*CLI> core show application EAGI
```

**Плюсы расширенного AGI**

Он проще Process-based AGI, включая канал аудиопотока только для чтения. Это единственный вариант, предлагающий эту функцию.

**Минусы расширенного AGI**

Поскольку для запуска приложения для каждого вызова необходимо создать новый процессь - он имеет те же проблемы эффективности, что и обычный, Process-Based AGI.

<table border="1" width="100%" cellpadding="5">
  <tr>
    <td>
      <p><img src="pics/note.png" height="100" align="left">Для альтернативного способа получения доступа к аудио вне Asterisk - рассмотрите возможность использования <a href="https://jackaudio.org">JACK</a>. Asterisk имеет модуль для интеграции JACK, называемый <code>app_jack</code>. Он предоставляет приложение <code>Jack()</code> и функцию диалплана <code>JACK_HOOK()</code>.</p>
    </td>
  </tr>
</table>

### FastAGI—AGI через TCP

_FastAGI_ - это термин, используемый для управления вызовами AGI через TCP-соединение. При использовании AGI на основе процессов экземпляр приложения AGI выполняется в системе для каждого вызова и связь с этим приложением осуществляется через `stdin` и `stdout`. С помощью FastAGI осуществляется TCP-соединение с сервером FastAGI. Управление вызовами осуществляется с использованием того же протокола AGI, но связь осуществляется через TCP-соединение и не требует запуска нового процесса для каждого вызова. Протокол AGI более подробно рассматривается в разделе ["Обзор коммуникаций AGI"](обзор-коммуникаций-agi). Использование FastAGI гораздо более масштабируемо, чем Process-Based AGI, хотя и более сложно в реализации.

Чтобы использовать FastAGI, вы вызываете приложение `AGI()` в диалплане Asterisk, но вместо имени приложения, которое нужно выполнить, вы предоставляете URL-адрес `agi://`. Например:

```
exten => 238,1,AGI(agi://127.0.0.1)
```

Номер порта по умолчанию для соединения FastAGI - `4573`. После двоеточия к URL-адресу можно добавить другой номер порта. Например:

```
exten => 238,1,AGI(agi://127.0.0.1:4574)
```

Так же, как и в случае AGI на основе процессов, в приложение FastAGI могут передаваться аргументы. Для этого добавьте их в качестве дополнительных аргументов в приложение `AGI()`, разделенных запятыми:

```
exten => 238,1,AGI(agi://192.168.1.199,arg1,arg2,arg3)
```

FastAGI также поддерживает использование записей DNS SRV, если вы предоставляете URL в виде `hagi://`. Используя записи SRV, DNS-серверы могут возвращать несколько узлов, к которым Asterisk может попытаться подключиться. Это может быть использовано для обеспечения высокой доступности и балансировки нагрузки. В следующем примере, для нахождения доступного для подключения сервера FastAGI, Asterisk выполнит поиск DNS для `_agi._tcp.shifteight.org`:

```
exten => 238,1,AGI(hagi://shifteight.org)
```

В этом примере DNS-сервера для домена `shifteight.org` потребуется хотя бы одна SRV-запись, настроенная для `_agi._tcp.shifteight.org`.

**Плюсы FastAGI**

Более эффективен чем process-based AGI. Вместо того, чтобы порождать новый процесс на вызов, можно построить сервер FastAGI для обработки нескольких вызовов.

DNS может использоваться для достижения высокой доступности и балансировки нагрузки между серверами FastAGI для дальнейшего повышения масштабируемости.

**Минусы FastAGI**

Он является более сложным при реализации сервера FastAGI, чем реализация приложения AGI на основе процессов.

### Async AGI - АМИ-контролируемый AGI

Async AGI позволяет приложению, использующему интерфейс AMI, асинхронно ставить команды AGI в очередь для выполнения на канале. Это может быть особенно полезно, если вы уже широко используете AMI и хотите улучшить свое приложение для обработки управления вызовами, а не писать подробный диалплан Asterisk или разрабатывать отдельный сервер FastAGI.

<table border="1" width="100%" cellpadding="5">
  <tr>
    <td>
      <p><img src="pics/note.png" height="100" align="left">Дополнительную информацию об Asterisk Manager Interface можно найти в <a href="glava-17.html#глава-17-ami-и-файлы-вызовов">Главе 17</a>.</p>
    </td>
  </tr>
</table>

Async AGI вызывается приложением `AGI()` в диалплане Asterisk. Аргумент для `AGI()` должен быть `agi:async`, как показано в следующем примере:

```
exten => 239,AGI(agi:async)
```

Дополнительную информацию о том, как использовать Async AGI через AMI, можно найти в следующем разделе.

**Плюсы Async AGI**

Существующее приложение AMI можно использовать для управления вызовами с помощью команд AGI.

**Минусы Async аги**

Это самый сложный способ реализации AGI.

<table border="1" width="100%" cellpadding="5">
  <tr>
    <td>
      <p align="center"><b>Настройка /etc/asterisk/manager.conf для Async AGI</b></p>
      <p>Чтобы использовать Async AGI, учетная запись AMI должна иметь разрешение <code>agi</code> как на <code>read</code>, так и на <code>write</code>. Например, следующий пользователь определенный в <i>manager.conf</i> будет иметь возможность как а) выполнять действия менеджера AGI, так и б) получать события AGI:</p>
      <p><pre><code>
; Определите пользователя с именем "hello" и паролем "world".
; Предоставьте этому пользователю разрешения на чтение/запись для AGI.
;
[hello]
secret = world
read = agi
write = agi
      </code></pre></p>
    </td>
  </tr>
</table>

## Обзор коммуникаций AGI

В предыдущем разделе были рассмотрены возможные варианты использования AGI. Этот раздел содержит более подробные сведения о том, как ваше пользовательское приложение AGI взаимодействует с Asterisk после вызова функции `AGI()`.

### Настройка сеанса AGI

После вызова `AGI()` или `EAGI()` из диалплана Asterisk, в приложение AGI передается некоторая информация для настройки сеанса. В этом разделе рассматривается, какие шаги предпринимаются в начале сеанса AGI для различных вариантов.

#### AGI на основе процессов/FastAGI

Для приложения AGI на основе процессов или подключения к серверу FastAGI переменные, перечисленные в Таблице 18-1, будут первыми фрагментами информации, отправленными из Asterisk в ваше приложение. Каждая переменная будет находиться на своей собственной строке, в виде:

```
переменная_agi: значение
```

_Таблица 18-1. Переменные среды AGI_

| Переменная    | Значение/пример  | Описание |
| :------------ | :--------------- | :------- |
| `agi_request` | `hello-world.sh` | Первый аргумент, который был передан в приложение `AGI()` или `EAGI()`. Для process-based AGI - это имя выполненного приложения AGI. Для FastAGI это будет URL-адрес, использованный для подключения к серверу FastAGI. |
| `agi_channel` | `SIP/0004F2060EB4-00000009` | Имя канала, выполнившего команду приложения `AGI()` или `EAGI()`. |
| `agi_language` | `en`            | Язык, установленный на `agi_channel`. |
| `agi_type`    | `SIP`            | Тип канала для `agi_channel`. |
| `agi_uniqueid`| `1284382003.9`   | uniqueid для agi_channel. |
| `agi_version` | `1.8.0-beta4`    | Используемая версия Asterisk. |
| `agi_callerid`| `12565551212`    | Полная строка callerID, установленная на `agi_channel`. |
| `agi_calleridname` | `Russell Bryant` | Имя caller ID, установленное на `agi_channel`. |
| `agi_callingpres` | `0`          | Представление вызывающего абонента, связанное с caller ID, установленное на `agi_channel`. Дополнительные сведения см. в выводе `core show function CALLERPRES` в CLI Asterisk. |
| `agi_callingani2` | `0`          | ANI2 абонента, связанный с `agi_channel`. |
| `agi_callington` | `0`           | ТН (тип номера) ID абонента, связанный с `agi_channel`. |
| `agi_callingtns` | `0`           | Набранный номер TNS (выбор транзитной сети), связанный с `agi_channel`. |
| `agi_dnid`    | `7010`           | Набранный номер, связанный с `agi_channel`. |
| `agi_rdnis`   | `unknown`        | Номер перенаправления, связанный с agi_channel. |
| `agi_context` | `phones`         | Контекст диалплана, в котором находился `agi_channel` при выполнении приложения `AGI()` или `EAGI()`. |
| `agi_extension` | `500`          | Расширение в диалплане, которое выполнялось `agi_channel` при запуске приложения `AGI()`` или `EAGI()``. |
| `agi_priority`| `1`              | Приоритет `agi_extension` в `agi_context`, в котором выполнилось `AGI()` или `EAGI()`. |
| `agi_enhanced`| `0.0`            | Указание на то, был ли использован `AGI()` или `EAGI()` из диалплана. `0.0` указывает на то, что был использован `AGI()`. `1.0` указывает на то, что был использован `EAGI()`.  |
| `agi_accountcode` | `myaccount`  | Код учетной записи, связанный с `agi_channel`. |
| `agi_threadid`| `140071216785168`| `threadid` потока в Asterisk, на котором выполняется приложение `AGI()` или `EAGI()`. Это может быть полезно для связывания журналов, созданных приложением AGI, с журналами, созданными Asterisk, поскольку журналы Asterisk содержат идентификаторы потоков. |
| `agi_arg_<argument number>` | `my argument` | Эти переменные предоставляют содержимое дополнительных аргументов, предоставленных приложению `AGI()` или `EAGI()`. |

Пример переменных, которые могут быть отправлены в приложение AGI, см. в разделе выходные данные отладки связи AGI в разделе [Быстрый старт](glava-18.md#быстрый-старт). Конец списка переменных будет обозначен пустой строкой. Код обрабатывает эти переменные путем считывания строк ввода в цикле, пока не будет получена пустая строка. В этот момент приложение продолжается и начинает выполнять команды AGI.

#### Async AGI

При использовании Async AGI, Asterisk будет отправлять событие диспетчера называемое `AsyncAGI` чтобы инициировать сеанс Async AGI. Это событие позволит приложениям, прослушивающим события диспетчера, взять на себя управление вызовом с помощью события менеджера AGI. Вот пример события менеджера, отправленного Asterisk:

```
Event: AsyncAGI
Privilege: agi,all
SubEvent: Start
Channel: SIP/0000FFFF0001-00000000
Env: agi_request%3A%20async%0Aagi_channel%3A%20SIP%2F0000FFFF0001-00000000%0A \
agi_language%3A%20en%0Aagi_type%3A%20SIP%0A \
agi_uniqueid%3A%201285219743.0%0A \
agi_version%3A%201.8.0-beta5%0Aagi_callerid%3A%2012565551111%0A \
agi_calleridname%3A%20Julie%20Bryant%0Aagi_callingpres%3A%200%0A \
agi_callingani2%3A%200%0Aagi_callington%3A%200%0Aagi_callingtns%3A%200%0A \
agi_dnid%3A%20111%0Aagi_rdnis%3A%20unknown%0Aagi_context%3A%20LocalSets%0A \
agi_extension%3A%20111%0Aagi_priority%3A%201%0Aagi_enhanced%3A%200.0%0A \
agi_accountcode%3A%20%0Aagi_threadid%3A%20-1339524208%0A%0A
```

<table border="1" width="100%" cellpadding="5">
  <tr>
    <td>
      <p><img src="pics/voron.png" height="100" align="left">Значение самого заголовка <code>Env</code> в этом событии менеджера AsyncAGI находится все на одной линии. Длинное значение заголовка <code>Env</code> было закодировано URL-адресом.</p>
    </td>
  </tr>
</table>

### Команды и ответы

После настройки сеанса AGI Asterisk начинает выполнять обработку вызовов в ответ на команды, отправленные из приложения AGI. Как только команда AGI будет выдана Asterisk - никакие другие команды не будут обработаны на этом канале, пока текущая не будет завершена. Когда он закончит обработку команды, Asterisk ответит с результатом.

<table border="1" width="100%" cellpadding="5">
  <tr>
    <td>
      <p><img src="pics/voron.png" height="100" align="left">AGI обрабатывает команды последовательно. После выполнения команды никакие другие команды не могут быть выполнены до тех пор, пока Asterisk не вернет ответ. Некоторые команды могут выполняться очень долго. Например, в команде <code>EXEC</code> AGI выполняет приложение Asterisk. Если есть команда <code>EXEC Dial</code> - cвязь с AGI блокируется до тех пор, пока вызов не будет выполнен. Если на данном этапе вашему приложению AGI необходимо продолжить взаимодействие с Asterisk - оно может сделать это с помощью AMI, который рассматривается в <a href="glava-17.md">Главе 17</a>.</p>
    </td>
  </tr>
</table>    

Вы можете получить полный список доступных команд AGI из консоли Asterisk, выполнив команду `agi show commands`. Эти команды описаны в Таблице 18-2. Для получения более подробной информации о конкретной команде AGI, включая сведения о синтаксисе для всех ожидаемых аргументов, используйте `agi show commands topic COMMAND`. Например, чтобы просмотреть встроенную документацию для команды AGI `ANSWER` - вы бы использовали `agi show commands topic ANSWER`.

*Таблица 18-2. Команды AGI*

| Команда AGI         | Описание                                                                  |
| :------------------ | :------------------------------------------------------------------------ |
| `ANSWER`            | Ответ на входящий вызов.                                                  |
| `ASYNCAGI BREAK`    | Завершение сеанса Async AGI и возврат канала в диалплан Asterisk.  |
| `CHANNEL STATUS`    | Получение статуса канала. Используется для получения текущего состояния канала, такого как up (ответ), down (трубка положена) или вызов. |
| `DATABASE DEL`      | Удаляет пару ключ/значение из встроенной базы данных AstDB.               |
| `DATABASE DELTREE`  | Удаляет дерево пар ключ/значение из встроенной базы данных AstDB.         |
| `DATABASE GET`      | Извлекает значение для ключа в базе данных AstDB.                         |
| `DATABASE PUT`      | Устанавливает значение для ключа в базе данных AstDB.                     |
| `EXEC`              | Выполняет приложение диалплана Asterisk на канале. Эта команда очень полезна в том, что между `EXEC` и `GET FULL VARIABLE`, вы можете сделать все что угодно с вызовом, который можете сделать из диалплана Asterisk. |
| `GET DATA`          | Считывание цифр от вызывающего абонента.                                  |
| `GET FULL VARIABLE` | Оценвает выражение диалплана Asterisk. Вы можете отправить строку, содержащую переменные и/или функции диалплана и Asterisk вернет результат после выполнения соответствующих подстановок. Эта команда очень полезна в том, что между `EXEC` и `FET FULL VARIABLE`, вы можете сделать все, что угодно с вызовом, который вы можете сделать из диалплана Asterisk. |
| `GET OPTION`        | Потоковая передача звукового файла во время ожидания цифры от вызывающего абонента. Это похоже на приложение диалплана `Background()`. |
| `GET VARIABLE`      | Извлечение значения переменной канала.                                    |
| `HANGUP`            | Заершение вызова. <sup><a href="#snat18">a</a></sup>                      |
| `NOOP`              | Ничего не делать. Вы получите результата запроса от этой команды, как и любой другой. Он может быть использован в качестве простого теста пути связи с Asterisk. |
| `RECEIVE CHAR`      | Получить один символ. Это работает только для типов каналов, которые его поддерживают, таких как iax2 использующий  фреймы `TEXT` или SIP использующий метод `MESSAGE`. |
| `RECEIVE TEXT`      | Получение текстового сообщения. Это работает только в тех же случаях, что и `RECEIVE CHAR`. |
| `RECORD FILE`       | Запись аудиопотока от вызывающего абонента в файл. Это блокирующая операция, аналогичная приложению диалплана `Record()`. Чтобы записать вызов в фоновом режиме во время выполнения других операций, используйте `EXEC Monitor` или `EXEC MixMonitor`. |
| `SAY ALFA`          | Озвучить строку символов. Вы можете найти пример этого в разделе "[Быстрый старт](glava-18#быстрый-старт)". Чтобы получить локализованную обработку этого и другой команды `SAY`, устанавливающие язык канала либо в файле конфигурации устройств (например, _sip.conf_ ) или в диалплане установкой функцией диалплана `CHANNEL(language)`.                   |
| `SAY DIGIT`         | Озвучить строку цифр. Например, 100 будет озвучено "один ноль ноль", если язык канала установлен на русский. |
| `SAY NUMBER`        | Озвучить номер телефона. Например, 100 будет сказано как "сто", если язык канала установлен на русский. |
| `SAY PHONETIC`      | Произнесите строку символов, но используя общее слово для каждой буквы (Альфа, Браво, Чарли...). |
| `SAY DATE`          | Произнести заданную дату                                                  |
| `SAY TIME`          | Произнести заданное ВРЕМЯ                                                 |
| `SAY DATETIME`      | Произнести заданную дату и время, используя указанный формат.             |
| `SEND IMAGE`        | Отправить изображение на канал. IAX2 поддерживает это, но нет никаких активно разработанных клиентов IAX2, о которых мы знаем, которые поддерживают это.  |
| `SEND TEXT`         | Отправить текст на канал, который его поддерживает. Это может быть использовано по крайней мере с каналами SIP и IAX2. |
| `SET AUTOHANGUP`    | Запланировать отключение канала в указанный момент времени в будущем.     |
| `SET CALLERID`      | Установить имя и номер идентификатора вызывающего абонента на канале.     |
| `SET CONTEXT`       | Установите текущий контекст диалплана на канале.                          |
| `SET EXTENSION`     | Установите текущее расширение диалплана на канале.                        |
| `SET MUSIC`         | Запуск или остановка музыки на удержание на канале.                       |
| `SET PRIORITY`      | Установите текущий приоритет диалплана на канале.                         |
| `SET VARIABLE`      | Задать для переменной канала указанное значение.                          |
| `STREAM FILE`       | Потоковая передача содержимого файла в канал.                             |
| `CONTROL STREAM FILE`  | Передать содержимое файла в канал, но также позволить каналу управлять потоком. Например, канал может приостановить, перемотать поток назад или вперед.  |
| `TDD MODE`          | Переключить режим TDD (Telecommunications Device for the Deaf - телекоммуникационное устройство для глухих) на канале. |
| `VERBOSE`           | Отправить на канал сообщение verbose logger. Подробные сообщения отображаются в консоли Asterisk, если значение параметра verbose достаточно высокое. Подробные сообщения также будут отправляться в любой файл журнала, настроенный для `verbose` канала регистратора в _/etc/asterisk/logger.conf_.  |
| `WAIT FOR DIGIT`    | Дождитесь, пока вызывающий абонент нажмет цифру.  |
| `SPEECH CREATE`     | Инициализировать распознавание речи. Это необходимо сделать перед использованием других речевых команд AGI. <sup><a href="#snbt18">b</a></sup> |
| `SPEECH SET`        | Установить настройку речевого движка. Доступные настройки относятся только к используемому механизму распознавания речи.  |
| `SPEECH DESTROY`    | Уничтожить ресурсы, выделенные для выполнения распознавания речи. Эта команда должна быть последней выполненной речевой командой.  |
| `SPEECH LOAD GRAMMAR`  | Загрузить грамматику                                                   |
| `SPEECH UNLOAD GRAMMAR` | Выгрузить грамматику                                                  |
| `SPEECH ACTIVATE GRAMMAR` | Активировать грамматику, которая была загружена.                    |
| `SPEECH DEACTIVATE GRAMMAR` | Деактивировать грамматику                                         |
| `SPEECH RECOGNIZE`  | Воспроизвести запрос и выполнить распознавание речи, а также ждать ввода цифр.  |
| `GOSUB`             | Выполнить функцию диалплана. Она будет выполняться так же, как и приложение диалплана `GoSub()`.  |

---

<sup><a name="snat18">a</a></sup>Когда используется команда AGI `HANGUP` канал отключается не сразу. Вместо этого канал помечается как нуждающийся в отключении. Ваше приложение AGI должно завершиться раьше, чем Asterisk продолжит и выполнит фактический процесс отключения.

<sup><a name="snbt18">b</a></sup>Хотя Asterisk включает в себя основной API для обработки распознавания речи, он не поставляется с модулем, обеспечивающим механизм распознавания речи. В настоящее время Digium предоставляет два коммерческих варианта распознавания речи: [Lumenvox](https://www.lumenvox.com/) и [Vestec](http://www.digium.com/en/products/software/vestec.php).

#### Process-based AGI/FastAGI

Команды AGI отправляются в Asterisk в одну строку. Строка должна заканчиваться символом новой строки. После отправки команды в Asterisk дальнейшие команды не будут обрабатываться до тех пор, пока последняя не будет завершена и ответ не будет отправлен обратно в приложение AGI. Вот пример ответа на команду AGI:

```
200 result=0
```

<table border="1" width="100%" cellpadding="5">
  <tr>
    <td>
      <p><img src="pics/note.png" height="100" align="left">Консоль Asterisk позволяет отлаживать взаимодействие с приложением AGI. Чтобы включить отладку связи AGI, выполните команду <code>agi set debug on</code>. Чтобы отключить отладку, используйте <code>agi set debug off</code>. Пока этот режим отладки включен - вся связь с приложением AGI и из него будет выводиться в консоли Asterisk. Пример такого вывода можно найти в разделе <a href="glava-18.md#быстрый-старт">Быстрый старт</a>.</p>
    </td>
  </tr>
</table>

#### Async AGI

При использовании Async AGI можно выполнять команды с помощью действия менеджера AGI. Чтобы просмотреть встроенную документацию для действия менеджера AGI - выполните `manager show command AGI` в CLI Asterisk. Демонстрация поможет выяснить, как выполняются команды AGI с помощью метода Async AGI. Во-первых, расширение создается в диалплане, который выполняет сеанс Async AGI в канале:

```
exten = > 240,AGI(agi:async)
```

При выполнении приложения AGI вызываемое событие менеджера `AsyncAGI` будет отправлено вместе со всеми переменными среды AGI. Подробная информация об этом событии находится в разделе "Async AGI” . После этого, действия менеджера AGI могут начать выполняться через AMI.

Ниже приведен пример выполнения действия диспетчера и его события, генерируемые во время обработки Async AGI. После первоначального выполнения действия менеджера AGI немедленно появляется ответ, указывающий на то, что команда была поставлена в очередь на выполнение. Позже появляется событие менеджера, указывающее что команда в очереди была выполнена. Заголовок идентификатора команды можно использовать для связывания начального запроса с событием, указывающим на то, что команда была выполнена:

```
Action: AGI
Channel: SIP/0004F2060EB4-00000013
ActionID: my-action-id
CommandID: my-command-id
Command: VERBOSE "Puppies like cotton candy." 1

Response: Success
ActionID: my-action-id
Message: Added AGI command to queue

Event: AsyncAGI
Privilege: agi,all
SubEvent: Exec
Channel: SIP/0004F2060EB4-00000013
CommandID: my-command-id
Result: 200%20result%3D1%0A
```

Следующие выходные данные - это то, что было замечено в консоли Asterisk во время этого сеансе Async AGI:

```
    -- Executing [7011@phones:1] AGI("SIP/0004F2060EB4-00000013",
       "agi:async") in new stack
 agi:async: Puppies like cotton candy.
  == Spawn extension (phones, 7011, 1)
exited non-zero on 'SIP/0004F2060EB4-00000013'
```

### Завершение сеанса AGI

Сеанс AGI завершается когда приложение AGI готово к его завершению. Подробности того как это происходит, зависят от того, использует ли ваше приложение process-based AGI, FastAGI или async AGI.

#### Process-based AGI/FastAGI

Ваше приложение AGI может выйти или закрыть свое соединение в любое время. Если канал не был отключен до завершения работы приложения - выполнение диалплана будет продолжено.

Если отключение канала происходит пока сеанс AGI все еще активен - Asterisk предоставит уведомление о том, что это произошло, чтобы ваше приложение могло соответствующим образом настроить свою работу.

Если канал завершается пока приложение AGI все еще выполняется - произойдет несколько вещей. Если команда AGI находится в середине выполнения - вы можете получить код результата `-1`. Однако вы не должны зависеть от этого, поскольку не все команды AGI требуют взаимодействия с каналом. Если выполняемая команда не требует взаимодействия с каналом, результат не будет отражать завершение.

Следующее что происходит после того, как канал завершается - это отправка уведомления о завершении в ваше приложение. Для process-based AGI, сигнал `SIGHUP` будет отправлен в процесс для уведомления о его завершении. Для быстрого подключения Asterisk отправит строку, содержащую слово `HANGUP`.

<table border="1" width="100%" cellpadding="5">
  <tr>
    <td><p>Если вы хотите отключить функцию отправки Asterisk сигнала <code>SIGHUP</code> для вашего приложения process-based AGI или строки <code>HANGUP</code> для вашего сервера FastAGI - вы можете сделать это, установив переменную канала <code>AGISIGHUP</code> как показано в этом коротком примере:</p>
<p><pre><code>
; нет SIGHUP (AGI) или HANGUP (FastAGI)
exten => 237,1,Set(AGISIGHUP=no)
    same => n,AGI(hello-world.sh)
</code></pre></p>
    </td>
  </tr>
</table>

После того, как завершение канала произошло - единственные команды AGI, которые могут использоваться, являются теми, которые не требуют взаимодействия с каналом. Документация для команд AGI, встроенных в Asterisk, включает указание на то, можно ли использовать каждую команду после того, как канал был отключен.

#### Async AGI

При использовании Async AGI интерфейс менеджера предоставляет механизмы для уведомления о завершении канала. Если вы хотите завершить сеанс Async AGI для канала - необходимо выполнить команду `ASYNCAGI BREAK`. Когда сеанс Async AGI закончится - Asterisk отправит событие менеджера `AsyncAGI` с `SubEvent` об `End`. Ниже приведен пример завершения сеанса Async AGI:

```
Action: AGI
Channel: SIP/0004F2060EB4-0000001b
ActionID: my-action-id
CommandID: my-command-id
Command: ASYNCAGI BREAK

Response: Success
ActionID: my-action-id
Message: Added AGI command to queue

Event: AsyncAGI
Privilege: agi,all
SubEvent: End
Channel: SIP/0004F2060EB4-0000001b
```

На этом этапе канал возвращается к следующему шагу в диалплане Asterisk (если он еще не был отключен).

## Пример: Доступ к базе данных учетной записи

Пример 18-1 - это пример сценария AGI. Чтобы запустить этот скрипт - сначала поместите его в каталог _/var/lib/asterisk/agi-bin_. Тогда вы бы выполнили его из диалплана Asterisk вот так:

```
exten = > 241,1, AGI(account-lookup.py)
    same => n,Hangup()
```

Этот пример написан на Python и очень скудно документирован для краткости. Он демонстрирует, как сценарий AGI взаимодействует с Asterisk с помощью `stdin` и `stdout`.

Сценарий предлагает пользователю ввести номер учетной записи, а затем воспроизводит значение, связанное с этим номером. В интересах краткости, мы жестко закодировали несколько поддельных учетных записей в скрипт — это, очевидно, будет что-то нормально обрабатываемое подключением к базе данных.

Сценарий намеренно лаконичен, так как мы заинтересованы в кратком показе некоторых функций AGI, не заполняя эту книгу страницами кода.

_Пример 18-1. account-lookup.py_

```
#!/usr/bin/env python
# Пример для AGI (Asterisk Gateway Interface).

import sys

def agi_command(cmd):
    '''Вписать команду и вернуть ответ'''
    print cmd
    sys.stdout.flush() #очистка буфера
    return sys.stdin.readline().strip() # строка пробелов

asterisk_env = {} # чтерие переменной среды AGI из Asterisk
while True:
    line = sys.stdin.readline().strip()
    if not len(line):
        break
    var_name, var_value = line.split(':', 1)
    asterisk_env[var_name] = var_value

# Поддельные "базы данных" учетных записей.
ACCOUNTS = {
    '12345678': {'balance': '50'},
    '11223344': {'balance': '10'},
    '87654321': {'balance': '100'},
}

response = agi_command('ANSWER')

# три аргумента: приглашение, тайм-аут, максимальная длина
response = agi_command('GET DATA enter_account 3000 8')

if 'timeout' in response:
    response = agi_command('STREAM FILE goodbye ""')
    sys.exit(0)

# Ответ будет выглядеть как: 200 result=<digits>
# С разделителем '=' мы получаем индекс 1
account = response.split('=', 1)[1]

if account == '-1': # ответ при ошибке
    response = agi_command('STREAM FILE astcc-account-number-invalid ""')
    response = agi_command('HANGUP')
    sys.exit(0)

if account not in ACCOUNTS: # неверный
    response = agi_command('STREAM FILE astcc-account-number-invalid ""')
    sys.exit(0)

balance = ACCOUNTS[account]['balance']

response = agi_command('STREAM FILE account-balance-is ""')
response = agi_command('SAY NUMBER %s ""' % (balance))
sys.exit(0)
```

## Разрабатываемые фреймворки

Был предпринят ряд усилий по созданию фреймворков или библиотек, облегчающих программирование AGI. Вы заметите, что некоторые из них уже упоминались в [Главе 17](glava-17.md). Так же, как и в случае с AMI, при оценке фреймворка мы рекомендуем вам найти тот, который соответствует следующим критериям:

_Зрелость_

Этот проект существует уже несколько лет? Зрелый проект гораздо менее вероятно будет иметь серьезные ошибки в нем.

_Поддержка_

Проверьте дату последнего обновления. Если проект не обновлялся в течении нескольких лет - есть большая вероятность, что он был заброшен. Он все еще может быть полезен, но вы будете представлены сами себе. Аналогично, как выглядит трекер ошибок? Много ли важных ошибок, которые игнорируются? (Будьте проницательны здесь, так как часто реалии поддержки свободного проекта требуют тщательного отбора — не все функции будут добавлены.)

_Качество кода_

Это хорошо написанная структура? Если он не был спроектирован хорошо - вы должны знать об этом, решая стоит ли доверять ему свой проект.

_Сообщество_

Есть ли активное сообщество разработчиков, поддерживающих этот проект? В случае если вам понадобится помощь - будет ли она доступна?

_Документация_

Код должен быть хорошо прокомментирован, но в идеале, вики или другая официальная документация для поддержки библиотеки имеет важное значение.

На момент подготовки настоящего документа фреймворки, перечисленные в Таблице 18-3, удовлетворяли всем или большинству из приведенных выше критериев. Если вы не видите здесь библиотеку для вашего предпочтительного языка программирования, она может быть где-то там, но просто не вошедшей в наш список.

_Таблица 18-3. Разрабатываемые фреймворки AGI_

| Фреймворк         | Язык |
| :---------------- | :--- |
| Adhearsion        | Ruby |
| **Asterisk-Java** | Java |
| **AsterNET**      | .NET |
| **ding-dong**     | Node.js |
| **PAGI**          | PHP |
| **Panoramisk**    | Python |
| StarPy            | Python + Twisted |

## Вывод

AGI предоставляет мощный интерфейс для Asterisk, который позволяет реализовать управление вызовами от первого лица на выбранном вами языке программирования. Вы можете использовать несколько подходов к реализации приложения AGI. Некоторые подходы могут обеспечить лучшую производительность, но ценой большей сложности. AGI предоставляет среду программирования, которая может облегчить интеграцию Asterisk с другими системами или просто обеспечить более удобную среду программирования управления вызовами для опытного программиста. Во многих случаях наилучшим подходом будет использование предварительно созданной структуры, особенно при оценке или прототипировании сложного проекта. Для максимальной производительности мы по-прежнему рекомендуем вам рассмотреть возможность написания как можно большего объема вашего приложения с помощью диалплана Asterisk.

[Глава 17. AMI и файлы вызовов](glava-17.md) | [Содержание](SUMMARY.md) | [Глава 19. Asterisk REST Interface](glava-19.md)

<!---
Таблица 18-2. Команды AGI

<table>
  <thead>
    <tr>
      <th style="text-align:left">AGI command</th>
      <th style="text-align:left">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">ANSWER</td>
      <td style="text-align:left">Answer the incoming call.</td>
    </tr>
    <tr>
      <td style="text-align:left">ASYNCAGI BREAK</td>
      <td style="text-align:left">End an async AGI session and have the channel return to the Asterisk dialplan.</td>
    </tr>
    <tr>
      <td style="text-align:left">CHANNEL STATUS</td>
      <td style="text-align:left">Retrieve the status of the channel. This is used to retrieve the current
        state of the channel, such as up (answered), down (hung up), or ringing.</td>
    </tr>
    <tr>
      <td style="text-align:left">DATABASE DEL</td>
      <td style="text-align:left">Delete a key/value pair from the built-in AstDB.</td>
    </tr>
    <tr>
      <td style="text-align:left">DATABASE DELTREE</td>
      <td style="text-align:left">Delete a tree of key/value pairs from the built-in AstDB.</td>
    </tr>
    <tr>
      <td style="text-align:left">DATABASE GET</td>
      <td style="text-align:left">Retrieve the value for a key in the AstDB.</td>
    </tr>
    <tr>
      <td style="text-align:left">DATABASE PUT</td>
      <td style="text-align:left">Set the value for a key in the AstDB.</td>
    </tr>
    <tr>
      <td style="text-align:left">EXEC</td>
      <td style="text-align:left">Execute an Asterisk dialplan application on the channel. This command
        is very powerful in that between EXEC and GET FULL VARIABLE, you can do
        anything with the call that you can do from the Asterisk dialplan.</td>
    </tr>
    <tr>
      <td style="text-align:left">GET DATA</td>
      <td style="text-align:left">Read digits from the caller.</td>
    </tr>
    <tr>
      <td style="text-align:left">GET FULL VARIABLE</td>
      <td style="text-align:left">Evaluate an Asterisk dialplan expression. You can send a string that contains
        variables and/or dialplan functions, and Asterisk will return the result
        after making the appropriate substitutions. This command is very powerful
        in that between EXEC and GET FULL VARIABLE, you can do anything with the
        call that you can do from the Asterisk dialplan.</td>
    </tr>
    <tr>
      <td style="text-align:left">GET OPTION</td>
      <td style="text-align:left">Stream a sound file while waiting for a digit from the caller. This is
        similar to the Background() dialplan application.</td>
    </tr>
    <tr>
      <td style="text-align:left">GET VARIABLE</td>
      <td style="text-align:left">Retrieve the value of a channel variable.</td>
    </tr>
    <tr>
      <td style="text-align:left">HANGUP</td>
      <td style="text-align:left">Hang up the channel.<a href="https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch18.html%22%20/l%20%22idm46178404238808">a</a>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">NOOP</td>
      <td style="text-align:left">Do nothing. You will get a result response from this command, just like
        any other. It can be used as a simple test of the communication path with
        Asterisk.</td>
    </tr>
    <tr>
      <td style="text-align:left">RECEIVE CHAR</td>
      <td style="text-align:left">Receive a single character. This only works for channel types that support
        it, such as IAX2 using TEXT frames or SIP using the MESSAGE method.</td>
    </tr>
    <tr>
      <td style="text-align:left">RECEIVE TEXT</td>
      <td style="text-align:left">Receive a text message. This only works in the same cases as RECEIVE CHAR.</td>
    </tr>
    <tr>
      <td style="text-align:left">RECORD FILE</td>
      <td style="text-align:left">Record the audio from the caller to a file. This is a blocking operation
        similar to the Record() dialplan application. To record a call in the background
        while you perform other operations, use EXEC Monitor or EXEC MixMonitor.</td>
    </tr>
    <tr>
      <td style="text-align:left">SAY ALPHA</td>
      <td style="text-align:left">Say a string of characters. You can find an example of this in <a href="18.%20Asterisk%20Gateway%20Interface%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22AGI-quickstart">&#x201C;Quick Start&#x201D;</a>.
        To get localized handling of this and the other SAY commands, set the channel
        language either in the device configuration file (e.g., sip.conf) or in
        the dialplan, by setting the CHANNEL(language) dialplan function.</td>
    </tr>
    <tr>
      <td style="text-align:left">SAY DIGITS</td>
      <td style="text-align:left">Say a string of digits. For example, 100 would be said as &#x201C;one
        zero zero&#x201D; if the channel&#x2019;s language is set to English.</td>
    </tr>
    <tr>
      <td style="text-align:left">SAY NUMBER</td>
      <td style="text-align:left">Say a number. For example, 100 would be said as &#x201C;one hundred&#x201D;
        if the channel&#x2019;s language is set to English.</td>
    </tr>
    <tr>
      <td style="text-align:left">SAY PHONETIC</td>
      <td style="text-align:left">Say a string of characters, but use a common word for each letter (Alpha,
        Bravo, Charlie&#x2026;).</td>
    </tr>
    <tr>
      <td style="text-align:left">SAY DATE</td>
      <td style="text-align:left">Say a given date.</td>
    </tr>
    <tr>
      <td style="text-align:left">SAY TIME</td>
      <td style="text-align:left">Say a given time.</td>
    </tr>
    <tr>
      <td style="text-align:left">SAY DATETIME</td>
      <td style="text-align:left">Say a given date and time using a specified format.</td>
    </tr>
    <tr>
      <td style="text-align:left">SEND IMAGE</td>
      <td style="text-align:left">Send an image to a channel. IAX2 supports this, but there are no actively
        developed IAX2 clients that support it that we know of.</td>
    </tr>
    <tr>
      <td style="text-align:left">SEND TEXT</td>
      <td style="text-align:left">Send text to a channel that supports it. This can be used with SIP and
        IAX2 channels, at least.</td>
    </tr>
    <tr>
      <td style="text-align:left">SET AUTOHANGUP</td>
      <td style="text-align:left">Schedule the channel to be hung up at a specified point in time in the
        future.</td>
    </tr>
    <tr>
      <td style="text-align:left">SET CALLERID</td>
      <td style="text-align:left">Set the caller ID name and number on the channel.</td>
    </tr>
    <tr>
      <td style="text-align:left">SET CONTEXT</td>
      <td style="text-align:left">Set the current dialplan context on the channel.</td>
    </tr>
    <tr>
      <td style="text-align:left">SET EXTENSION</td>
      <td style="text-align:left">Set the current dialplan extension on the channel.</td>
    </tr>
    <tr>
      <td style="text-align:left">SET MUSIC</td>
      <td style="text-align:left">Start or stop music on hold on the channel.</td>
    </tr>
    <tr>
      <td style="text-align:left">SET PRIORITY</td>
      <td style="text-align:left">Set the current dialplan priority on the channel.</td>
    </tr>
    <tr>
      <td style="text-align:left">SET VARIABLE</td>
      <td style="text-align:left">Set a channel variable to a given value.</td>
    </tr>
    <tr>
      <td style="text-align:left">STREAM FILE</td>
      <td style="text-align:left">Stream the contents of a file to a channel.</td>
    </tr>
    <tr>
      <td style="text-align:left">CONTROL STREAM FILE</td>
      <td style="text-align:left">Stream the contents of a file to a channel, but also allow the channel
        to control the stream. For example, the channel can pause, rewind, or fast-forward
        the stream.</td>
    </tr>
    <tr>
      <td style="text-align:left">TDD MODE</td>
      <td style="text-align:left">Toggle the TDD (Telecommunications Device for the Deaf) mode on the channel.</td>
    </tr>
    <tr>
      <td style="text-align:left">VERBOSE</td>
      <td style="text-align:left">Send a message to the verbose logger channel. Verbose messages show up
        on the Asterisk console if the verbose setting is high enough. Verbose
        messages will also go to any logfile that has been configured for the verbose
        logger channel in /etc/asterisk/logger.conf.</td>
    </tr>
    <tr>
      <td style="text-align:left">WAIT FOR DIGIT</td>
      <td style="text-align:left">Wait for the caller to press a digit.</td>
    </tr>
    <tr>
      <td style="text-align:left">SPEECH CREATE</td>
      <td style="text-align:left">Initialize speech recognition. This must be done before using other speech
        AGI commands.<a href="https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch18.html%22%20/l%20%22idm46178404204824">b</a>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">SPEECH SET</td>
      <td style="text-align:left">Set a speech engine setting. The settings that are available are specific
        to the speech recognition engine in use.</td>
    </tr>
    <tr>
      <td style="text-align:left">SPEECH DESTROY</td>
      <td style="text-align:left">Destroy resources that were allocated for doing speech recognition. This
        command should be the last speech command executed.</td>
    </tr>
    <tr>
      <td style="text-align:left">SPEECH LOAD GRAMMAR</td>
      <td style="text-align:left">Load a grammar.</td>
    </tr>
    <tr>
      <td style="text-align:left">SPEECH UNLOAD GRAMMAR</td>
      <td style="text-align:left">Unload a grammar.</td>
    </tr>
    <tr>
      <td style="text-align:left">SPEECH ACTIVATE GRAMMAR</td>
      <td style="text-align:left">Activate a grammar that has been loaded.</td>
    </tr>
    <tr>
      <td style="text-align:left">SPEECH DEACTIVATE GRAMMAR</td>
      <td style="text-align:left">Deactivate a grammar.</td>
    </tr>
    <tr>
      <td style="text-align:left">SPEECH RECOGNIZE</td>
      <td style="text-align:left">Play a prompt and perform speech recognition, as well as wait for digits
        to be pressed.</td>
    </tr>
    <tr>
      <td style="text-align:left">GOSUB</td>
      <td style="text-align:left">Execute a dialplan subroutine. This will perform in the same way as the
        GoSub() dialplan application.</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><a href="https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch18.html%22%20/l%20%22idm46178404238808-marker">a</a> When
          the HANGUP AGI command is used, the channel is not immediately hung up.
          Instead, the channel is marked as needing to be hung up. Your AGI application
          must exit first before Asterisk will continue and perform the actual hangup
          process.</p>
        <p><a href="https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch18.html%22%20/l%20%22idm46178404204824-marker">b</a> While
          Asterisk includes a core API for handling speech recognition, it does not
          come with a module that provides a speech recognition engine. Digium currently
          provides two commercial options for speech recognition: <a href="https://www.lumenvox.com/">Lumenvox</a> and
          <a
          href="http://www.digium.com/en/products/software/vestec.php">Vestec</a>.</p>
      </td>
      <td style="text-align:left"></td>
    </tr>
  </tbody>
</table>#### Process-based AGI/FastAGI

-->
