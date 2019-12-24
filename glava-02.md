# Глава 2. Архитектура Asterisk

> _Прежде всего, но не обязательно в таком порядке._
>
> -- Доктор Кто

Asterisk очень отличается от других, более традиционных УАТС тем, что диалплан в Asterisk обрабатывает все входящие каналы по существу одинаково, а не разделяет их на станции, транки, периферийные модули и т.д.

В традиционной АТС существует логическое различие между станциями \(телефонными аппаратами\) и транками \(магистралями - ресурсами, которые подключаются к внешнему миру\). Это ограничение делает творческую маршрутизацию в традиционных УАТС очень сложной или невозможной.

Asterisk, с другой стороны, не имеет внутреннего понятия транков или станций. В Asterisk все, что входит или выходит из системы, проходит через какой-то канал. Существует множество различных типов каналов; однако диалплан Asterisk обрабатывает все каналы аналогичным образом, что означает, например, внутренний пользователь может существовать на конце внешнего транка \(например, сотовый телефон\) и обрабатываться диалпланом точно так же, как если бы пользователь был на внутреннем номере. Если вы не работали с традиционной АТС, то может быть не сразу очевидно насколько это является мощным и освобождающим. Рисунок 2-1 иллюстрирует различия между этими двумя архитектурами.

![&#x420;&#x438;&#x441;&#x443;&#x43D;&#x43E;&#x43A; 2-1. &#x410;&#x440;&#x445;&#x438;&#x442;&#x435;&#x43A;&#x442;&#x443;&#x440;&#x430; Asterisk &#x43F;&#x440;&#x43E;&#x442;&#x438;&#x432; &#x423;&#x410;&#x422;&#x421;](.gitbook/assets/0%20%288%29.png)

### Модули

Asterisk построен на модулях. Модуль - это загружаемый компонент, который обеспечивает определенную функциональность, такую как драйвер канала \(например, `chan_pjsip.so`\) или ресурс, который позволяет подключиться к внешней технологии \(например `func_odbc.so`\). Модули Asterisk загружаются на основе параметров, определенных в файле _/etc/asterisk/modules.conf_. Мы обсудим использование многих модулей в этой книге, но на этом этапе мы просто хотим представить концепцию модулей и дать вам представление о типах модулей, которые доступны.

На самом деле можно запустить Asterisk вообще без каких-либо модулей, хотя в этом состоянии он ничего не сможет сделать. Это бывает полезно, чтобы понять модульную природу Asterisk для оценки архитектуры.

---

**Примечание**

Вы можете запустить Asterisk без модулей, загружаемых по умолчанию и загружать каждый нужный модуль вручную из консоли, но это не то, что вы хотели бы запустить в продакшен; это было бы полезно только в том случае, если бы вы настраивали производительность системы, в которой хотели убрать все, что не требуется вашим конкретным приложением Asterisk.

---

Типы модулей в Asterisk включают в себя следующие:

* Приложения - рабочие лошадки диалплана, такие как `Dial()`, `Voicemail()`, `Playback()`, `Queue()` и т.д.
* Модули соединения - механизмы, которые соединяют каналы \(вызовы\) друг с другом
* Модули записи деталей вызовов \(CDR\)
* Модули регистрации событий канала \(CEL\)
* Драйверы каналов — различные соединения с системой и из системы; SIP \(Session Initiation Protocol\) использует канальный драйвер PJSIP
* Трансляторы кодеков — преобразовывают различные кодеки как G729, G711, G722, Speex и так далее
* Интерпретаторы форматов — как указано выше, но относящиеся к файлам, хранящимся в файловой системе
* Функции диалплана — расширенные возможности диалплана
* Модули УАТС
* Модули ресурсов
* Дополнительные модули
* Тестовые модули

Существует официальный список типов статуса поддержки, включенных в `menuselect`.[2](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch02.html%22%20/l%20%22idm46178409833272)

### Приложения

Приложения диалплана используются в _extensions.conf_ для определения различных действий, применимых к вызову. Например, приложение `Dial()` отвечает за создание исходящих соединений с внешними ресурсами и, возможно, является самым важным приложением диалплана. Доступные приложения перечислены в Таблице 2-1.

Таблица 2-1. Популярные приложения диалплана

| Имя | Purpose |
| :--- | :--- |
| app\_authenticate | Сравнивает сигналы набора DTMF с указанной строкой \(паролем\) |
| app\_cdr | Записывает данные в CDR |
| app\_chanspy | Allows a channel to listen to audio on another channel |
| app\_confbridge | Provides conferencing |
| app\_dial | Used to connect channels together \(i.e., make phone calls\) |
| app\_directed\_pickup | Answers a call that’s ringing at another extension |
| app\_directory | Presents the list of names from voicemail.conf |
| app\_dumpchan | Dumps channel variables to Asterisk command-line interface \(CLI\) |
| app\_echo | Echos received audio back to source channel \(can be helpful in demonstrating latency\) |
| app\_exec | Contains Exec\(\), TryExec\(\), and ExecIf\(\): executes a dialplan application conditionally |
| app\_mixmonitor | Records both sides of a call \(transmit and receive\) and mixes them together into a single file |
| app\_originate | Allows dialplan logic to originate a call \(as opposed to a call coming in on a channel\) |
| app\_page | Creates multiple audio connections to specified devices for public address \(paging\) |
| app\_parkandannounce | Enables automated announcing of parked calls |
| app\_playback | Plays a file to the channel \(does not accept input\) |
| app\_playtones | Plays pairs of tones of specified frequencies \(DTMF mostly\) |
| app\_queue | Provides Automatic Call Distribution \(ACD\) |
| app\_read | Requests input of digits from callers and assigns input to a variable |
| app\_readexten | Requests input of digits from callers and passes call to a designated extension and context |
| app\_record | Records received audio to a file |
| app\_senddtmf | Transmits DTMF to calling party |
| app\_stack | Provides GoSub\(\), GoSubIf\(\), Return\(\), StackPop\(\), LOCAL\(\), and LOCAL\_PEEK\(\) |
| app\_stasis | Passes call control to an ARI application—many Asterisk developers use this one application, and from there handle all the rest of their development outside of the Asterisk dialplan |
| app\_system | Executes commands in a Linux shell |
| app\_transfer | Performs a transfer on the current channel |
| app\_voicemail | Provides voicemail |
| app\_while | Includes While\(\), EndWhile\(\), ExitWhile\(\), and ContinueWhile\(\); provides while loop functionality in the dialplan |

### Модули соединений

Модули соединений выполняют фактическое соединение каналов. Эти модули, перечисленные в Таблице 2-2, в настоящее время используются только для \(и необходимы\) _app\_confbridge_.

Таблица 2-2. Модули соединений

| Имя | Описание |
| :--- | :--- |
| bridge\_builtin\_features | Performs bridging when utilizing built-in user features \(such as those found in features.conf\). |
| bridge\_multiplexed | Performs complex multiplexing, as would be required in a large conference room \(multiple participants\). Currently only used by app\_confbridge. |
| bridge\_simple | Performs simple channel-to-channel bridging. |
| bridge\_softmix | Performs simple multiplexing, as would be required in a large conference room \(multiple participants\). Currently only used by app\_confbridge. |

В следующих разделах мы рассмотрели список модулей, которые, по нашему мнению, достаточно важны для обсуждения в этой книге. Вы найдете много других модулей в загрузке Asterisk, но многие старые модули либо устарели, либо имеют небольшую поддержку или не поддерживаются, и поэтому не рекомендуются для производства, если у вас нет доступа к разработчикам, которые могут поддерживать их для вас.

### Модули записи деталей вызова \(CDR\)

Модули CDR, перечисленные в Таблице 2-3, предназначены для обеспечения как можно большего числа методов хранения записей сведений о вызовах. CDR можно хранить в файле \(по умолчанию\), базе данных, RADIUS или _syslog_.

---

**Примечание**

Записи деталей вызовов не предназначены для использования в сложных приложениях биллинга. Если вам требуется больше контроля над биллингом и отчетностью о вызовах - обратите внимание на журнал событий канала (CEL), обсуждаемый далее. Преимущество CDR заключается в том, что он просто работает.

---

Таблица 2-3. Общие модули записи деталей вызова

| Name | Purpose |
| :--- | :--- |
| cdr_adaptive_odbc | Allows writing of CDRs through ODBC framework with ability to add custom fields |
| cdr_csv | Writes CDRs to disk as a comma-separated values (CSV) file |
| cdr_custom | Writes CDRs to a CSV file, but allows addition of custom fields |
| cdr_odbc | Writes CDRs through ODBC framework |
| cdr_syslog | Writes CDRs to syslog |

### Модули логирования событий канала

Channel event logging (CEL) provides much more powerful control over reporting of call activity. By the same token, it requires more careful planning of your dialplan, and by no means will it work automatically. Asterisk’s CEL modules are listed in [Table 2-4](Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition/2.%20Asterisk%20Architecture%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22Architecture_id292113).

Table 2-4. Channel event logging modules

| Name | Purpose |
| :--- | :--- |
| cel\_custom | CEL to disk/file |
| cel\_manager | CEL to AMI |
| cel\_odbc | CEL to ODBC |

### Драйверы каналов

Без драйверов каналов у Asterisk не было бы возможности совершать или принимать вызовы. Каждый драйвер канала специфичен для протокола или типа канала, который он поддерживает \(SIP, ISDN и т.д.\). Модуль канала действует как шлюз к ядру Asterisk. Некоторые из наиболее популярных драйверов каналов Asterisk перечислены в Таблице 2-5.

Table 2-5. Популярные драйверы каналов

| Имя | Назначение |
| :--- | :--- |
| chan\_bridge | Используется внутри приложения ConfBridge\(\); не должен использоваться напрямую |
| chan\_dahdi | Обеспечивает подключение к картам ТфОП, использующим драйверы каналов DAHDI |
| chan\_local | Предоставляет механизм для обработки части диалплана как канала |
| chan\_motif | Реализует протокол Jingle, включая возможность подключения к Google Talk и Google Voice; представлен в Asterisk 11 |
| chan\_multicast\_rtp | Обеспечивает подключение к потокам многоадресного Realtime Transport Protocol \(RTP\) |
| chan\_pjsip | Драйвер канала Session Initiation Protocol \(SIP\) |

### Трансляторы кодеков

Трансляторы кодеков[3](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch02.html%22%20/l%20%22idm46178409283960) \(часто называемые транскодерами\) позволяют Asterisk конвертировать форматы аудиопотоков между вызовами. Поэтому, если вызов поступает по каналу PRI \(используя G.711\) и должен быть передан в сжатый канал SIP \(например, используя G.729, один из многих кодеков, которые может обрабатывать SIP\), соответствующий транслятор кодека выполнит преобразование.

Кодеки-это сложные алгоритмы, которые обрабатывают преобразование аналоговой информации \(в данном случае звука, но также может быть и видео\) в цифровой формат. Многие кодеки также обеспечивают сжатие и исправление ошибок, но это не является обязательным требованием.

---

**Примечание**

Если кодек (например G.729) использует сложный алгоритм кодирования, интенсивное использование транскодинга может создать огромную нагрузку на процессор. Специализированное оборудование для декодирования/кодирования G.729 доступно от производителей оборудования, таких как Sangoma и Digium (и, вероятно, других).

---

Asterisk делает довольно хорошую работу по поддержке кодеков, но в основном сосредоточен на кодеках, обычно используемых телефонными приложениями (в отличие от кодеков, используемых, скажем, для музыки или видео, таких как MP3 или MP4). Они перечислены в [Таблице 2-6](Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition/2.%20Asterisk%20Architecture%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22Architecture_id292600).

Таблица 2-6. Общие трансляторы кодеков

| Имя | Назначение |
| :--- | :--- |
| codec_alaw | Кодек PCM A-law используется во всем мире на ТфОП (кроме Канады/США). Этот кодек (вместе с ulaw) должен быть включен на всех ваших каналах. |
| codec_g729 | До недавнего времени это был запатентованный кодек, но теперь он является бесплатным. На момент написания этой статьи он по-прежнему продается Digium в качестве дополнения, но его также можно найти в виде бесплатного пакета. Это очень популярный кодек, если требуется сжатие \(и использование процессора не является проблемой\), но он накладывает нагрузку на процессор, добавляет задержку к вызовам, немного снижает качество и никоим образом не уменьшает накладные расходы. |
| codec\_a\_mu | Прямой конвертер A-law в mu-law. |
| codec\_g722 | Широкополосный аудиокодек. |
| codec\_gsm | Кодек Global System for Mobile Communications \(GSM\). Очень низкое качество звука. |
| codec\_ilbc | Интернет-кодек с низким битрейтом \(iLBC\). |
| codec\_lpc10 | Линейный предсказательный кодирующий вокодер \(чрезвычайно низкая пропускная способность\). |
| codec\_opus | Предназначен для замены speex \(и vorbis\). |
| codec\_resample | Пересемлирование между 8-ми и 16-тибитными линейными сигналами. |
| codec\_speex | Кодек Speex. |
| codec\_ulaw | Кодек PCM Mu-law, используемый на ТфОП в Канаде/США. Это более формально написано как μ-закон, но не у многих людей есть греческая буква μ на клавиатуре, поэтому обычно пишется как ulaw.a Часто является кодеком по умолчанию, и должен быть включен на всех ваших каналах. |
| [a](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch02.html%22%20/l%20%22idm46178403326376-marker) Произносится как  “мью-лоу,” но так же вы часто будете слышать как "ю-лоу". |  |

---

**Совет**

Digium предоставляет некоторые дополнительные полезные модули кодеков: codec\_g729, codec\_silk, codec\_siren7 и codec\_siren14. Эти модули кодеков не являются open source по различным причинам. Вы должны приобрести лицензию на использование codec\_g729, но остальные являются бесплатными. Вы можете найти их на [сайте Digium](http://downloads.digium.com/pub/telephony/).

---

### Интерпретаторы формата

Format interpreters ([Table 2-7](Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition/2.%20Asterisk%20Architecture%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22Architecture_id292825)) perform a similar function as codec translators, but they do their work on files rather than channels, and handle more than just audio. If you have a recording on a menu that has been stored as GSM, you would need to use a format interpreter to play that recording to any channels not using the GSM codec.[4](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch02.html%22%20/l%20%22idm46178403315272)

If you store a recording in several formats simultaneously \(such as WAV, GSM, etc.\), Asterisk will determine the least costly format[5](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch02.html%22%20/l%20%22idm46178403313864) to use when a channel needs to play that recording.

Таблица 2-7. Format interpreters

| Name | Plays files stored in |
| :--- | :--- |
| format\_g729 | G.729: .g729 |
| format\_gsm | RPE-LTP \(original GSM codec\): .gsm |
| format\_h264 | H.264 video: .h264 |
| format\_ilbc | Internet Low Bitrate Codec: .ilbc |
| format\_jpeg | Graphic file: .jpeg, .jpg |
| format \_ogg\_ vorbis | Ogg container: .ogg |
| format\_pcm | Various Pulse-Coded Modulation formats: .alaw, .al, .alw, .pcm, .ulaw, .ul, .mu, .ulw, .g722, .au |
| format\_siren14 | G.722.1 Annex C \(14 kHz\): .siren14 |
| format\_siren7 | G.722.1 \(7 kHz\): .siren7 |
| format\_sln | 8-bit signed linear: .sln, .raw |
| format\_vox | .vox |
| format\_wav | .wav |
| format\_wav\_gsm | GSM audio in a WAV container: .wav, .wav49 |

### Dialplan Functions

Dialplan functions, listed in [Table 2-8](Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition/2.%20Asterisk%20Architecture%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22Architecture_id293094), complement the dialplan applications \(see [“Applications”](Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition/2.%20Asterisk%20Architecture%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22Architecture_id241422)\). They provide many useful enhancements to things like string handling, time and date wrangling, and ODBC connectivity.

Table 2-8. A curated list of useful dialplan functions

| Name | Purpose |
| :--- | :--- |
| func\_audiohook​in⁠herit | Allows calls to be recorded after transfer |
| func\_blacklist | Writes/reads blacklist in astdb |
| func\_callcompletion | Gets/sets call-completion configuration parameters for the channel |
| func\_callerid | Gets/sets caller ID |
| func\_cdr | Gets/sets CDR variable |
| func\_channel | Gets/sets channel information |
| func\_config | Includes AST\_CONFIG\(\); reads variables from config file |
| func\_curl | Uses cURL to obtain data from a URI |
| func\_cut | Slices and dices strings |
| func\_db | Provides astdb functions |
| func\_devstate | Gets state of device |
| func\_dialgroup | Creates a group for simultaneous dialing |
| func\_dialplan | Validates that designated target exists in dialplan |
| func\_env | Includes FILE\(\), STAT\(\), and ENV\(\); performs operating system actions |
| func\_global | Gets/sets global variables |
| func\_groupcount | Gets/sets channel count for members of a group |
| func\_hangupcause | Gets/sets hangupcause information from the channel |
| func\_logic | Includes ISNULL\(\), SET\(\), EXISTS\(\), IF\(\), IFTIME\(\), and IMPORT\(\); performs various logical functions |
| func\_math | Includes MATH\(\), INC\(\), and DEC\(\); performs mathematical functions |
| func\_odbc | Allows dialplan integration with ODBC resources |
| func\_rand | Returns a random number within a given range |
| func\_realtime | Performs lookups within the Asterisk Realtime Architecture \(ARA\) |
| func\_redirecting | Provides access to information about where this call was redirected from |
| func\_shell | Performs Linux shell operations and returns results |
| func\_sprintf | Performs string format functions similar to C function of same name |
| func\_srv | Performs SRV lookups in the dialplan |
| func\_strings | Includes over a dozen string manipulation functions |
| func\_timeout | Gets/sets timeouts on channel |
| func\_uri | Converts strings to URI-safe encoding |
| func\_vmcount | Returns count of messages in a voicemail folder for a particular user |

### PBX Modules

The PBX modules are peripheral modules that provide enhanced control and configuration mechanisms. For example, pbx\_config is the module that loads the traditional Asterisk dialplan. The currently available PBX modules are listed in [Table 2-9](Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition/2.%20Asterisk%20Architecture%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22Architecture_id293725).

Table 2-9. PBX modules

| Name | Purpose |
| :--- | :--- |
| pbx\_config | This module provides the traditional, and most popular, dialplan language for Asterisk. Without this module, Asterisk cannot read extensions.conf. |
| pbx\_dundi | Performs data lookups on remote Asterisk systems. |
| pbx\_realtime | Provides functionality related to the Asterisk Realtime Architecture. |
| pbx\_spool | Provides outgoing spool support relating to Asterisk call files. |

### Resource Modules

Resource modules integrate Asterisk with external resources. This group of modules has effectively turned into a catch-all for things that do not fit in other categories. We will break them into some subgroups of modules that are related.

#### Configuration backends

Asterisk is configured using text files in /etc/asterisk by default. These modules, listed in [Table 2-10](Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition/2.%20Asterisk%20Architecture%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22Architecture_id2938860), offer alternative configuration methods. See [Chapter 15](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch15.html%22%20/l%20%22asterisk-DB) for detailed documentation on setting up database-backed configuration.

Table 2-10. Configuration backend modules

| Name | Purpose |
| :--- | :--- |
| res\_config\_curl | Pulls configuration information using cURL |
| res\_config\_ldap | Pulls configuration information using LDAP |
| res\_config\_odbc | Pulls configuration information using ODBC |

#### Calendar integration

Asterisk includes some integration with calendar systems. You can read and write calendar information from the dialplan. You can also have calls originated based on calendar entries. The core calendar integration is provided by the res\_calendar module. The rest of the modules provide the ability to connect to specific types of calendar servers. [Table 2-11](Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition/2.%20Asterisk%20Architecture%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22Architecture_id2938862) lists the calendar integration modules.

Table 2-11. Calendar integration modules

| Name | Purpose |
| :--- | :--- |
| res\_calendar | Enables base integration to calendaring systems |
| res\_calendar\_caldav | Allows features provided by res\_calendar to connect to calendars via CalDAV |
| res\_calendar\_exchange | Allows features provided by res\_calendar to connect to MS Exchange |
| res\_calendar\_icalendar | Allows features provided by res\_calendar to connect to Apple/Google iCalendar |

#### Other resource modules

[Table 2-12](Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition/2.%20Asterisk%20Architecture%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22Architecture_id293886) includes the rest of the resource modules that did not fit into one of the subgroups we defined earlier in this section.

Table 2-12. Resource modules

<table>
  <thead>
    <tr>
      <th style="text-align:left">Name</th>
      <th style="text-align:left">Purpose</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">res_adsi</td>
      <td style="text-align:left">Provides ADSI<a href="https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch02.html%22%20/l%20%22idm46178409429112">a</a>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">res_agi</td>
      <td style="text-align:left">Provides the Asterisk Gateway Interface (see <a href="https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch18.html%22%20/l%20%22AGI">Chapter 18</a>)</td>
    </tr>
    <tr>
      <td style="text-align:left">res_corosync</td>
      <td style="text-align:left">Provides distributed message waiting indication (MWI) and device state
        notifications via the Corosync Cluster Engine</td>
    </tr>
    <tr>
      <td style="text-align:left">res_crypto</td>
      <td style="text-align:left">Provides cryptographic capabilities</td>
    </tr>
    <tr>
      <td style="text-align:left">res_curl</td>
      <td style="text-align:left">Provides common subroutines for other cURL modules</td>
    </tr>
    <tr>
      <td style="text-align:left">res_fax</td>
      <td style="text-align:left">Provides common subroutines for other fax modules</td>
    </tr>
    <tr>
      <td style="text-align:left">res_fax_spandsp</td>
      <td style="text-align:left">Plug-in for fax using the spandsp package</td>
    </tr>
    <tr>
      <td style="text-align:left">res_http_post</td>
      <td style="text-align:left">Provides POST upload support for the Asterisk HTTP server</td>
    </tr>
    <tr>
      <td style="text-align:left">res_http_websocket</td>
      <td style="text-align:left">Provides WebSocket support for the Asterisk internal HTTP server (required
        by WebRTC)</td>
    </tr>
    <tr>
      <td style="text-align:left">res_monitor</td>
      <td style="text-align:left">Provides call-recording resources</td>
    </tr>
    <tr>
      <td style="text-align:left">res_musiconhold</td>
      <td style="text-align:left">Provides music on hold (MOH) resources</td>
    </tr>
    <tr>
      <td style="text-align:left">res_mutestream</td>
      <td style="text-align:left">Allows muting/unmuting of audio streams</td>
    </tr>
    <tr>
      <td style="text-align:left">res_odbc</td>
      <td style="text-align:left">Provides common subroutines for other ODBC modules</td>
    </tr>
    <tr>
      <td style="text-align:left">res_phoneprov</td>
      <td style="text-align:left">Provisions phones from Asterisk HTTP server</td>
    </tr>
    <tr>
      <td style="text-align:left">res_pktccops</td>
      <td style="text-align:left">Provides PacketCable COPS resources</td>
    </tr>
    <tr>
      <td style="text-align:left">res_security_log</td>
      <td style="text-align:left">Enables logging of security events generated by other parts of Asterisk</td>
    </tr>
    <tr>
      <td style="text-align:left">res_snmp</td>
      <td style="text-align:left">Provides system status information to an SNMP-managed network</td>
    </tr>
    <tr>
      <td style="text-align:left">res_speech</td>
      <td style="text-align:left">Generic speech recognition API<a href="https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch02.html%22%20/l%20%22idm46178409412232">b</a>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">res_stasis</td>
      <td style="text-align:left">Ties together the various components of the Stasis application infrastructure</td>
    </tr>
    <tr>
      <td style="text-align:left">res_xmpp</td>
      <td style="text-align:left">Provides XMPP resources (FKA Jabber)</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><a href="https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch02.html%22%20/l%20%22idm46178409429112-marker">a</a> While
          most of the ADSI functionality in Asterisk is never used, the voicemail
          application uses this resource.</p>
        <p><a href="https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch02.html%22%20/l%20%22idm46178409412232-marker">b</a> Requires
          a separately licensed product in order to be used.</p>
      </td>
      <td style="text-align:left"></td>
    </tr>
  </tbody>
</table>### Add-on Modules

Add-on modules are community-developed modules with different usage or distribution rights from those of the main code \([Table 2-13](Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition/2.%20Asterisk%20Architecture%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22Architecture_id294470)\). They are kept in a separate directory and are not compiled and installed by default. To enable these modules, use the menuselect build configuration utility.

Table 2-13. Add-on modules

| Name | Purpose | Popularity/status |
| :--- | :--- | :--- |
| chan\_ooh323 | Enables making and receiving VoIP calls using the H.323 protocol | Usable |
| format\_mp3 | Allows Asterisk to play MP3 files | Usable |
| res\_config\_mysql | Uses a MySQL database as a real-time configuration backend | Useful |

### Test Modules

Test modules are used by the Asterisk development team to validate new code. They are constantly changing and being added to, and are not useful unless you are developing Asterisk software.

If you are an Asterisk developer, however, the Asterisk Test Suite may be of interest to you, as you can build automated tests for Asterisk and submit those back to the project, which runs on several different operating systems and types of machines. By expanding the number of tests constantly, the Asterisk project avoids the creation of regressions in code. By submitting your own tests to the project, you can feel more confident in future upgrades.

More information about building tests is available in [the “Asterisk Test Suite” document](http://bit.ly/14SLEqs), or you can join the \#asterisk-testing channel on the Freenode IRC network.

## File Structure

Asterisk is a complex system, composed of many resources. These resources make use of the filesystem in several ways. Since Linux is so flexible in this regard, it is helpful to understand what data is being stored, so that you can understand where you are likely to find a particular bit of stored data \(such as voicemail messages or logfiles\).

### Configuration Files

The Asterisk configuration files include extensions.conf, pjsip.conf, modules.conf, and dozens of other files that define parameters for the various channels, resources, modules, and functions that may be in use.

These files will normally be found in /etc/asterisk. You will be working in this folder a lot as you configure and administer your Asterisk system.

### Modules

Asterisk modules are usually installed to the /usr/lib/asterisk/modules folder. You will not normally have to interact with this folder; however, it will be occasionally useful to know where the modules are located. For example, if you upgrade Asterisk and select different modules during the menuselect phase of the install, the old \(incompatible\) modules from the previous Asterisk version will not be deleted, and you will get a warning from the install script. Those old files will need to be deleted from the modules folder. This can be done either manually or with the “uninstall” make \(make uninstall\) target.

### The Resource Library

There are several resources that require external data sources. For example, music on hold \(MOH\) can’t happen unless you have some music to play. System prompts also need to be stored somewhere on the hard drive. The /var/lib/asterisk folder is where system prompts, AGI scripts, music on hold, and other resource files are stored.

### The Spool

The spool is where applications store files on a Linux system that are going to change frequently, or that will be processed by other processes at a later time. For example, Linux print jobs and pending emails are normally written to the spool until they are processed.

In Asterisk, the spool is used to store transient items such as voice messages, call recordings,[6](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch02.html%22%20/l%20%22idm46178409226312) call files, and so forth.

The Asterisk spool will be found under the /var/spool/asterisk directory.

### Logging

Asterisk is capable of generating several different kinds of logfiles. The /var/log/asterisk folder is where call detail records \(CDRs\), channel events from CEL, debug logs, queue logs, messages, errors, and other output are written.

This folder will be extremely important for any troubleshooting efforts you undertake. We will talk more about how to make use of Asterisk logs in [Chapter 21](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch21.html%22%20/l%20%22asterisk-Monitoring).

## The Dialplan

The dialplan is the heart of Asterisk. All channels that arrive in the system will be passed through the dialplan, which contains the call-flow scripts that determine how incoming calls are handled.

Dialplan is typically written using Asterisk’s own dialplan syntax, which is stored in a file named /etc/asterisk/extensions.conf. There are other ways to control call flow, and we will explore them later, but no matter which method you eventually employ, you will find that a basic understanding of the traditional dialplan will be immensely helpful. That is what we will focus on for most of the first two-thirds of this book.

Later, we will explore handling call flow outside of the dialplan, using technologies such as AMI, AGI, and ARI.

## Hardware

Asterisk is capable of communicating with a vast number of different technologies. In general, these connections are made across a TCP/IP network connection \(usually using SIP\). However, connections to more traditional telecom circuits, such as PRI \(T1, E1, etc.\), BRI \(EuroISDN\) SS7 \(mostly T1 and E1\), and analog \(everything from a few FXO and FXS ports up to large channel banks fed through T1/E1 CAS/RBS connections\), can also be achieved using physical cards installed in a server.

Many companies produce this hardware, such as Digium \(the sponsor, owner, and primary developer of Asterisk\), Sangoma \(who recently purchased Digium\), Dialogic \(also a Sangoma company\), OpenVox, Pika, Voicetronix, beroNet, and many others. All of these companies have been involved with Asterisk for many years.

The most popular hardware for Asterisk is generally designed to work through the Digium Asterisk Hardware Device Interface \(known as DAHDI\). This is a complex architecture, and is out of the scope of this book. Server-based telephony cards will all have installation requirements unique to the manufacturer, and will require you to have strong skills in both Linux hardware installation as well as traditional PSTN circuit troubleshooting and provisioning.

If you need to interface with traditional PSTN circuits using Asterisk, we recommend that you keep Asterisk as a SIP-only platform, and interface using a third-party gateway of some sort. Be warned: this is not entry-level stuff, and if you are just starting out with Asterisk, you are strongly advised to keep your initial solutions to SIP-only.

## Asterisk Versioning

The Asterisk release methodology has gone through several styles over time. This has led to some confusion in the past, but these days the versioning is fairly straightforward, and relatively easy to understand. Digium has maintained an excellent reference at the [Asterisk wiki](http://bit.ly/2XTb5Wl), and we encourage you to go there for the latest details on Asterisk versions.

This book was written and tested using version 16, but you will find that the fundamental concepts we explore will be relevant to most Asterisk versions. The conceptual structure of Asterisk has not changed for quite some time, and as of this writing there are no known plans to change that going forward. Future versions will deliver more powerful multimedia and conferencing capabilities, to be sure, but they are likely to be implemented within the existing structure.

## Conclusion

Asterisk is composed of many different technologies, most of which are complicated in their own right. As a result, understanding Asterisk architecture can be overwhelming. Still, the reality is that Asterisk is well designed for what it does and, in our opinion, has achieved a remarkable balance between flexibility and complexity.

[1](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch02.html%22%20/l%20%22idm46178408963912-marker) A good indicator that you’ve worked with traditional PBXs is the presence of a large callus on your forehead, obtained from smashing your head against a brick wall too many times to count.

[2](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch02.html%22%20/l%20%22idm46178409833272-marker) This is a command that is available as part of the installation process. We will discuss the use of menuselect in the installation chapter.

[3](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch02.html%22%20/l%20%22idm46178409283960-marker) The term codec is short for “coder decoder.”

[4](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch02.html%22%20/l%20%22idm46178403315272-marker) It is partly for this reason that we do not recommend the default GSM format for system recordings. WAV recordings will sound better and use fewer CPU cycles.

[5](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch02.html%22%20/l%20%22idm46178403313864-marker) Some codecs will impose a significant load on the CPU, so much so that a system that might support several hundred channels without transcoding will only handle a few dozen when transcoding is in use.

[6](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch02.html%22%20/l%20%22idm46178409226312-marker) Not call detail records \(CDRs\), but rather audio recordings of calls generated by the MixMonitor\(\) and related applications.

[Глава 1. Революция в телефонии](glava-01.md) | [Содержание](SUMMARY.md)  | [Глава 3. Установка Asterisk](glava-03.md)
