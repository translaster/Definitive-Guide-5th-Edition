# Глава 8. Голосовая почта

> _Просто оставьте сообщение, может быть, я перезвоню._
>
> -- Джо Уолш

До того как почта и мгновенные сообщения стали широко распространены, голосовая почта была очень популярна. Даже теперь, когда большинство людей предпочитает обмениваться текстовыми сообщениями, голосовая почта является важным компонентом любой телефонной станции.

В Asterisk есть достаточно гибкая система голосовой почты называемая Comedian Mail \[^1\]. В диалплане она реализуется посредством модуля app\_voicemail.so.

#### Предупреждение о модуле голосовой почты в Asterisk

Модуль app\_voicemail один из старейших в ASterisk, как следствие он имеет много ограничений, особенно если сравнивать его с другими, постоянно усовершенствуемыми модулями. Код модуля настолько устарел, что ни у кого не возникает желания с ним разбираться, и поэтому очень маловероятно появление в нем новых функций. Вы должны понимать что app\_voicemail не просто предоставляет элемент диалплана; для работы голосовой почты должно произойти много событий, например хранение и управление файлами, взаимодействие с почтовой системой операционной системы, распознавание временных зон, работы с форматами файлов, вопросы безопасности и еще куча вещей. И хотя app\_voicemail все это делает, в итоге получается довольно неуклюжая подсистема \(стоит заметить, что в традиционных АТС  для голосовой почты выделяется отдельная машина\).

 Было предпринято множество попыток реинжинировать голосовую почту, но все они оказались неудачными. Причина проста: объем работ \(а следовательно и стоимость\), необходимых для перепроектирования модуля \(таким образом, чтобы удовлетворить потребности сообщества\), в сочетании с отсутствием интереса к технологии голосовой почты в целом, быстро убивали любую инициативу.

Тем не менее важно отметить, что голосовая почта в Asterisk работает, и работает хорошо. Возможно, она даже удовлетворит ваши потребности. В ином случае сообщество будет более чем благодарно, если вы попытаетесь перепроектировать её.

Вот некоторые функции, которые включает в себя модуль голосовой почты:

* Неограниченное количество защищенных паролем ящиков голосовой почты, каждый из которых содержит подкаталоги для сортировки голосовой почты
* Различные  приветствия для различных статусов, таких как "недоступен" или "занят"
* Наличие  предустановленных приветствий и возможность создания собственных
* Возможность ассоциирования телефонов с несколькими  почтовыми ящиками и почтового ящика с несколькими телефонами
* Уведомление о голосовом сообщении на электронную почту,  опционально с прикрепленным  аудио файлом
* Широковещательная голосовая почта и перенаправление голосовой почты
* Индикатор ожидания сообщения  \(мигающий светодиод или специальный сигнал\) поддерживаемый на многих типах телефонов
* Справочник сотрудников на основе голосовой почты

Теперь давайте познакомимся с основными частями конфигурационного файла голосовой почты, включая настройки в разделе Общие, различными возможными региональными настройками, интеграцией голосовой почты в ваш dialplan и проведем краткий обзор того, как Asterisk хранит голосовую почту в файловой системе Linux.

## Файл конфигурации voicemail.conf

Ранее в базе данных MySQL мы установили таблицу, необходимую для голосовой почты, и поэтому сейчас мы  можем создавать в ней почтовые ящики без какой-либо другой конфигурации. Однако, также можно создавать почтовые ящики в  конфигурационном файле /etc/asterisk/voicemail.conf \(в том числе в этом файле можно изменять различные настройки по умолчанию\). Мы продолжим использовать базу данных для создания пользователей и управления ими, поскольку она гораздо лучше подходит для этой задачи, но также  исследуем конфигурационный файл,  чтобы вы могли почувствовать гибкость настройки голосовой почты Asterisk.

Файл voicemail.conf содержит несколько секций, в которых могут быть переопределены различные предустановленные параметры. В большинстве случаев вам не понадобится их менять; однако  вы можете  посмотреть в файл примера  ~/src/asterisk-1.15.&lt;your version&gt;/configs/samples/voicemail.conf.sample. Он содержит полезную информацию о различных настройках.

Далее мы рассмотрим простейший voicemail.conf файл. Если у вас  возникнет желание доработать базовую конфигурацию, просто добавьте или измените соответствующие опции.

### Исходный файл voicemail.conf

Мы рекомендуем использовать следующий пример кофигурации как базовый. Вы можете ознакомиться с файлом ~/asterisk-complete/asterisk/11/configs/voicemail.conf.sample для детализации различных настроек.

Разместите следующий код в файле /etc/asterisk/voicemail.conf:

```text
; Voicemail Configuration
[general]
format=wav49|wav
serveremail=voicemail@shifteight.org
attach=yes
skipms=3000
maxsilence=10
silencethreshold=128
maxlogins=3
emaildateformat=%A, %B %d, %Y at %r
pagerdateformat=%A, %B %d, %Y at %r
sendvoicemail=yes ; Allow the user to compose and send a voicemail while inside
[zonemessages]
eastern=America/New_York|'vm-received' Q 'digits/at' IMp
central=America/Chicago|'vm-received' Q 'digits/at' IMp
central24=America/Chicago|'vm-received' q 'digits/at' H N 'hours'
military=Zulu|'vm-received' q 'digits/at' H N 'hours' 'phonetic/z_p'
european=Europe/Copenhagen|'vm-received' a d b 'digits/at' HM
```

{% hint style="info" %}
Настройка Linux сервера для отправки почтовых сообщений  администратору выходит за рамки данной книги. Вы должны будете протестировать вашу службу голосовой почты чтобы убедиться что она корректно обрабатывается почтовым агентом\[^2\] и что нижеследующие спам-фильтры не отклоняют эти сообщения \(одна из причин почему это может происходить — использование сервером ASterisk в теле письма имени хоста которое не может быть разрешено\)
{% endhint %}

Вы можете создать массивный и сложный файл voicemail.conf \(и даже хранить в нем почтовые ящики пользователей\), но для упрощения задачи мы сосредоточимся на нескольких примерах.

### Секция \[general\]

В первой секции файла voicemail.conf, \[general\], определяются глобальные настройки. Многие из этих настроек могут быть переопределены в настройках каждого конкретного ящика. В таблице 8-1 мы перечислили  некоторые опции, которые, как мы считаем, наиболее важно рассмотреть.

| Опция | Значение/пример | Примечание |
| :--- | :--- | :--- |
| format | wav49\|gsm\|wav | Для каждого перечисленного формата, Asterisk создает  отдельную запись в этом формате, каждый раз когда остается сообщение. Преимущество этого механизма в  экономии ресурсов на транскодировании, которое не надо выполнять, если для записи используется тот же самый кодек что и для канала. Мы любим WAV за высокое качество, и WAV49 потому что  он хорошо сжимается и легок для передачи по почте. Мы не любим GSM за шумы в записи, но он пользуется  некторой популярностью\[^a\]. |
| serveremail | user@domain | Адрес в заголовке письма FROM, отображаемый в отправленном с Asterisk письме\[^b\]. |
| attach | yes,no | Если для ящика голосовой почты указан адрес электронной почты, эта опция определяет, будет ли прикреплено записанное сообщение к письму \(если нет, то будет отправлено простое уведомление, и пользователю нужно будет позвонить на голосовую почту чтобы получить свое сообщение\). |
| maxmsg | 9999 | По умолчанию, Asterisk разрешает хранить  максимум 100 сообщений на пользователя. Для пользователей удаляющих прослушанные сообщения это не является проблемой. Для пользователей которые предпочитают сохранять свои сообщения, этот лимит будет достигнут очень быстро. С размерами жестких дисков  в наши дни, вы можете легко хранить тысячи сообщений для каждого пользователя. Поэтому, по нашему мнению, можно выставить эту опцию в максимальное значение и позволить пользователям самим управлять этими данными. Имейте в виду, что после нескольких лет хранения, старые сообщения голосовой почты в больших системах могут занимать много места на жестком диске. |
| maxsecs | 600 | Эта установка может быть полезной, когда большая система голосовой почты имеет хранилище размером в 40МБ\[^c\]\: В данном случае необходимо ограничение длины сообщения, иначе система легко использует весь объем хранилища. Этот параметр может раздражать вызывающих абонентов (хотя он заставляет их переходить к сути сообщения, поэтому некоторым людям это нравится). В настоящее время с терабайтными дисками, нет никаких технических причин для ограничения длины сообщения. Но есть два соображения: 1) Если канал завис, то хорошо бы иметь какое-либо ограничение, чтобы система не записывала бесконечное пустое сообщение 2)Если абонент использует свой почтовый ящик как голосовую записную книжку, он не обрадуется если вы его отключите через три минуты. Вероятно, будет правильным установить значение где-то между 600 секундами (10 минут) и 3600 секундами (1 час). |
| emailsubject |\[PBX\]: New message ${VM_MSGNUM} in mailbox ${VM_MAILBOX}| Этой настройкой вы можете  задать вид темы письма которое отсылает Asterisk. Подробное описание смотрите в файле примера voicemail.conf.sample. |
| emailbody |Dear ${VM_NAME}:\n\n\t you have a ${VM_DUR} long message (number ${VM_MSGNUM})\n in mailbox ${VM_MAILBOX} \n\n\t\t\t\t --Asterisk\n| Этой настройкой вы определяете как будет выглядеть тело письма. Подробное описание смотрите в файле примера voicemail.conf.sample. |
| emaildateformat | %A, %d %B %Y at %H:%M:%S | Эта опция позволяет определить формат даты в письме. Используется тот же синтаксис, что и в функции STRFTIME языка C. |
| pollmailboxes | no, yes | Если содержимое почтового ящика меняется чем-нибудь кроме app_voicemail (Например внешним приложением или другой системой Asterisk), эта настройка выставленная в "yes" указывает периодически опрашивать  почтовые ящики на предмет изменений и выставляет правильную индикацию ожидающих сообщений (MWI). |
| pollfreq | 30 | Используется в сочетании с параметром pollmailboxes и в секундах задает частоту опроса почтового ящика. |

_Таблица 8-1. Настройки секции \[General\] файла voicemail.conf_

\[^a\]: Разделителем для каждого формата служит знак вертикальной черты, pipe (|).<br>
\[^b\]: Рассылка писем от Asterisk требует аккуратной настройки, так как многие спам-фильтры находят сообщения от Asterisk очень подозрительными и просто игнорируют их.<br>
\[^c\]: Да, вы все верно прочитали. Мегабайт.



#### Проверка паролей голосовой почты внешними средствами

По умолчанию, Asterisk не проверяет  пароли пользователей на устойчивость к взлому. Любой кто разрабатывает системы голосовой почты скажет вам, что подавляющее число пользователей устанавливает такие пароли к своим ящикам, которые легко запомнить, например 1234 или 1111. Хотя обычно Фрод-боты не предназначены для баловства, наличие слабых паролей представляет собой дыру в системе безопасности голосовой почты.

Поскольку модуль app\_voicemail.so не имеет встроенной возможности проверки паролей, настройки externpass, externpassnotify, и externpasscheck позволяют проверять их с помощью внешней программы. Asterisk вызовет приложение находящееся по указанному вами пути и передаст следующие аргументы:

mailbox context oldpass newpass

Затем скрипт будет оценивать аргументы основываясь на правилах, которые вы в нем определили, и, соответственно, он должен вернуть в Asterisk значение VALID в случае успеха или INVALID в случае неуспеха \(На самом деле возвращаемое значение для пароля не прошедшего проверку может быть любое, кроме слов VALID и FAILURE\). Это значение выводится в stdout -- на стандартный вывод. Если сценарий вернул значение INVALID, Asterisk будет воспроизводить запись ошибочного пароля и пользователю будет нужно попробовать набрать что-то иное.

Возможно вам стоит реализовать следующие правила:

* Минимальная длина пароля должна составлять 6 символов
* Пароль не должен представлять собой строку повторяющихся цифр \(т.к. 111111\)
* Пароль не должен представлять собой последовательность цифр \(т.к. 123456 или 456789\)

Asterisk поставляется с простейшим сценарием, значительно усовершенствующим безопасность вашей системы голосовой почты. Он расположен в каталоге исходного кода: /contrib/scripts/voicemailpwcheck.py.

Мы настоятельно рекомендуем вам скопировать его в ваш каталог /usr/local/bin \(или туда где вы держите подобные вещи\), и затем раскомментировать опцию externpasscheck= в вашем файле voicemail.conf.

Часть секции \[general\], это раздел дополнительных опций \(Они упоминаются в конфигурационном файле как расширенные опции, хотя ничего особенного в них нет\). Эти опции \(перечисленные в таблице 8-2\) определены также как и другие в секции \[general\], но выделяет их то, что они могут быть переопределены для каждого отдельного почтового ящика, что перезапишет свойства установленные в этой области секции \[general\]. Другими словами нижеследующие опции могут быть установлены в базе данных когда вы создаете новый почтовый ящик.

| Опция | Значение/пример | Примечание |
| :--- | :--- | :--- |
| tz | eastern, european, etc. | Указывает имя временной зоны, также определенной в разделе \[zonemessages\] \(мы будем говорить об этом в следующей части главы\). |
| locale | de\_DE.utf8, es\_US.utf8, etc. | Используется для определения того, как Asterisk задает формат строки с данными даты и времени в различных локалях. Для определения локали корректной для вашей Linux системы, выполните в консоли операционной системы команду locale -a. |
| attach | yes, no | Если для вашего голосового почтового ящика определен адрес электронной почты, эта опция определяет, будут ли сообщения прикреплены к почтовым уведомлениям \(В ином случае будет отправлено простое уведомление\). |
| attachfmt | wav49, wav, etc. | Если опция attach включена и сообщения хранятся в различных форматах, данная опция определяет формат в котором будут отправлены записанные сообщения в почтовых уведомлениях. Обычно wav49 является хорошим выбором, так как использует лучший алгоритм сжатия, а следовательно использует меньшую полосу пропускания, и в тоже время не так жутко звучит как формат gsm. |
| exitcontext | context | Эта опция позволяет звонящим абонентам выйти из системы голосовой почты, когда они находятся в процессе записи сообщений \(для примера, нажатие на 0 переключает на оператора\). По умолчанию, контекст с которого пришел вызов будет использоваться как выходной. По желанию этот параметр может быть определен в иной контекст. |
| review | yes, no | Этот параметр почти всегда должен быть установлен в yes \(Хотя его значение по умолчанию no\). Люди расстраиваются, если ваша система голосовой почты не позволяет им прослушать свое сообщение перед отправкой. |
| operator | yes, no | В соответствии с лучшими практиками вы должны разрешить своим абонентам в любой момент прекратить запись, если они передумали оставлять голосовое сообщение. Обратите внимание, что для обработки этих вызовов контексте exitcontext, требуется расширение o \(не \«ноль\», а буква \«о\»\). |
| delete | no, yes | После того как почтовое уведомление \(которое может включать само голосовое сообщение\) будет отправлено, оно будет удалено. Эта опция рискованна, так как факт отправки сообщения не гарантирует его получения \(Спам фильтры любят удалять сообщения голосовой почты Asterisk\). На новых системах оставьте эту опцию в значении \"no\", до тех пор пока не убедитесь, что сообщения не теряются из-за спам-фильтров. |
| nextaftercmd | yes, no | Эта удобная маленькая опция сэкономит вам немного времени, так как переключает вас на следующее сообщение сразу после завершения работы с предыдущим. |
| passwordlocation | spooldir | Если вы захотите, то можете хранить пароли в собственных spool каталогах каждого почтового ящика\[\^a\]. Одно из преимуществ использования  опции spooldir в том, что она позволяет вам определять операторы \#include в файле voicemail.conf \(Имеется в виду что вы можете хранить ссылки на почтовые ящики в нескольких файлах, как, например, с кодом диалплана\). По другому так сделать невозможно, потому что в обычном случае app\_voicemail записывает изменение пароля в файловую систему и не может обновлять пароль почтового ящика  хранящийся за пределами voicemail.conf или spool/. Если вы не используете опцию passwordlocation, вы не сможете определять почтовые ящики снаружи voicemail.conf так как пароль не будет обновлен. Хранение паролей в файле в специальном spool каталоге решает эту проблему. |

_Таблица 8-2. Утвержденный список дополнительных опций для voicemail.conf_

\[a\]\: Обычно каталог spool находится по пути /var/spool/asterisk, и может быть переопределен в /etc/asterisk/asterisk.conf.

### Секция \[zonemessages\]

Следующей в файле voicemail.conf идет секция \[zonemessages\]. Цель этой секции, разрешить обработку сообщений в соответствии с часовым поясом, таким образом вы можете воспроизводить сообщения пользователей с правильными отметками времени. Вы можете установить имя зоны в нужное вам значение. Затем вы можете определить на какой часовой пояс будет ссылаться это имя зоны, а так же некоторые параметры которые определяют как воспроизводятся временные метки. Примеры синтаксиса вы можете посмотреть в файле ~/src/asterisk-16.&lt;TAB&gt;/configs/samples/voicemail.conf.sample. Asterisk  включает примеры показанные в таблице 8-3. Можно настроить любую временную зону, известную Linux-системе. Просто используйте для зон имена как в Linux и затем определите как вы хотите чтобы они обрабатывались.


| Имя зоны | Значение/пример | Примечание |
| :--- | :--- | :--- |
| eastern | America/New\_York\|'vm-received' Q 'digits/at' IMp | Это значение подойдет для восточных часовых поясов \(EST/EDT\). |
| central | America/Chicago\|'vm-received' Q 'digits/at' IMp | Это значение подойдет для центрального часового пояса \(CST/CDT\). |
| central24 | America/Chicago\|'vm-received' q 'digits/at' H N 'hours' | Это значение так же подойдет для CST/CDT, но будет отображать время в 24 часовом формате. |
| military | Zulu\|'vm-received' q 'digits/at' H N 'hours' 'phonetic/z\_p' | Это значение подойдет для UTC - Всемирное координированное время \(Zulu time, formerly GMT\). |
| european | Europe/Copenhagen\|'vm-received' a d b 'digits/at' HM | Это значение подойдет для Центральной Европы \(CEST\). |

_Table 8-3. Настройки секции \[zonemessages\] для voicemail.conf_

### Mailboxes

Вы можете настраивать почтовые ящики в конфигурационном файле voicemail.conf, но это не рекомендуемый путь. Для определения ваших ящиков мы будем использовать базу данных.
Первая вещь которую нам надо сделать -- это сказать Asterisk, что голосовая почта пользователей доступна в базе данных. Это можно сделать отредактировав файл /etc/asterisk/extconfig.conf:

```
$ sudo vim /etc/asterisk/extconfig.conf
\[settings\] ; older mechanism for connecting all other modules to the database
ps\_endpoints =&gt; odbc,asterisk
ps\_auths =&gt; odbc,asterisk
ps\_aors =&gt; odbc,asterisk
ps\_domain\_aliases =&gt; odbc,asterisk
ps\_endpoint\_id\_ips =&gt; odbc,asterisk
ps\_contacts =&gt; odbc,asterisk
voicemail =&gt; odbc,asterisk,voicemail
```
Вы должны перезапустить Asterisk для применения этих изменений \($ sudo service asterisk restart\).

В системе голосовой почты, почтовый ящик должен быть определен в контексте. Это не имеет отношения к какому-нибудь контексту диалплана; Этот контекст является специфичной меткой голосовой почты, которая определяет, какие почтовые ящики будут сгруппированы вместе, а также используется для именования папки в спуле, который содержит различные файлы связанные с этим почтовым ящиком \(Приветствия, сообщения, конверты, и т.д.\). Обычно вам не стоит волноваться об этом, так как все почтовые ящики окажутся в контексте по умолчанию. На самом деле вам нужно определить только контексты, настройки которых отличаются от остальных -- если вы используете сложную, мультитенантную систему, в которой возможно перекрытие расширений, или если вы не хотите, чтобы определенные группы пользователей были доступны другим группам пользователей.

Таблица \`asterisk\`.\`voicemail\` поддерживает много опций; Однако, для создания почтового ящика необходимо заполнить только три поля, плюс еще два рекомендовано. Требуются поля context, mailbox, и password, а fullname и email являются строго рекомендованными. Вот простой MySQL запрос позволяющий вам создать несколько почтовых ящиков:

```
INSERT INTO \`asterisk\`.\`voicemail\` \(context,mailbox,password,fullname,email\)
VALUES
\('default','100','486541','Russell Bryant', 'russell@shifteight.org'\),
\('default','101','957642','Leif Madsen', 'leif@shifteight.org'\),
\('default','102','656844','Jared Smith', 'jared@shifteight.org'\),
\('default','103','375416','Jim VanMeggelen', 'jim@shifteight.org'\);
```


Ниже части определений почтового ящика:

mailbox<br>
Это номер почтового ящика. Обычно он соответствует добавочному номеру абонента.

password<br>
Это числовой пароль с помощью которого владелец почтового ящика сможет получить доступ к своей голосовой почте. Если пользователь меняет пароль, система обновит это поле в базе данных.<br>
Если перед паролем стоит знак дефиса \(-\), пользователь не сможет изменить свой  пароль почтового ящика.


fullname \(FirstName LastName\)<br>
Это имя владельца почтового ящика. Каталог компании использует текст в этом поле для проверки имен пользователей. вы можете использовать только один пробел, чтобы отделить имя от фамилии, поэтому если ваша фамилия Ван Меггелен, вы должны записать ее как ВанМеггелен. Другие знаки пунктуации также могут вызвать проблемы. \(Мы смотрим на тебя, О'Рейли. \)

email address<br>
Это почтовый адрес владельца ящика. Asterisk может выслать запись голосовой почты на
определенный адрес электронной почты.

**Внимание**
Asterisk не может обрабатывать концепцию фамилии отличающуюся от простого слова. Это значит, что перед добавлением в voicemail.conf, во всех таких фамилиях как О'Рейли, Брайан-Мэдсен-Смитт, и, да, даже Ван Меггелен, должны быть удалены все пунктуационные символы и пробелы.

Есть довольно много других опций, которые вы можете определить для каждого пользователя. Маловероятно что вы будете использовать их все, но в таблице 8-4 содержится список тех из них которые могут быть вам полезны:

| Option | Description |
| :--- | :--- |
| delete | After Asterisk sends the voicemail via email, the voicemail is deleted from the server. This option is useful for users who only want to receive voicemail via email. Valid options are yes or no. Option can only be set per mailbox. |
| envelope | Turns on or off envelope playback prior to playback of the voicemail message. Valid options are yes or no. Default is yes. |
| exitcontext | The dialplan context to exit to when pressing \* or 0 from the Voicemail\(\) application. Works in conjunction with the operator option as well. Must have an extension a in the context for exiting with \*. Must have an extension o in the context for exiting with 0. You’ll need to do a bit of design work before your dialplan will be able to handle this well, so it’s best to leave it blank until you’ve had a chance to prototype everything you’ll need to handle. |
| forcegreeting | Forces the recording of a greeting for new mailboxes. A new mailbox is determined by the mailbox number and password matching. Valid values are yes or no. Default is no. |
| forcename | Forces the recording of the person’s name for new mailboxes. A new mailbox is determined by the mailbox number and password matching. Valid values are yes or no. Default is no. |
| hidefromdir | If set to yes, this mailbox will be hidden from the Directory\(\) application. Default is no. |
| locale | Allows you to set the locale for the mailbox in order to control formatting of the date/time strings. See voicemail.sample.conf for more information. |
| messagewrap | Allows the first and last messages to wrap around \(e.g., allow last message to wrap back to the first on the next message, or first message to wrap to the last message when going to the previous message\). Valid options are yes or no. Default is no. |
| minpassword | Sets the minimum password length. Argument should be a whole number. |
| nextaftercmd | Skips to the next message after the user presses the 7 key \(delete\) or 9 key \(save\). Valid values are yes or no. Default is yes. |
| operator | Will allow the sender of a voicemail to hit 0 before, during, or after recording of a voicemail. Will exit to the o extension in the same context, or the context defined by the exitcontext option. Valid options are yes or no. Default is no. There are security risks associated with this, so it’s best not to use it until you’re certain the exitcontext does not allow calls to leave the system \(i.e., end up making an expensive overseas call\). |
| passwordlocation | By default, the password for voicemail is stored in the voicemail.conf file, and modified by Asterisk whenever the password changes. This may not be desirable, especially if you want to parse the password from an external location \(or script\). The alternate option for passwordlocation is spooldir, which will place the password for the voicemail user in a file called secret.conf in the user’s voicemail spool directory. Valid options are voicemail.conf and spooldir. The default option is voicemail.conf. |
| review | When enabled, will allow the user recording a voicemail message to re-record their message. After pressing the \# key to save their voicemail, they’ll be prompted whether they wish to re-record or save the message. Valid options are yes or no. Default is no. |
| saycid | If enabled, and a prompt exists in /var/spool/asterisk/voicemail/recordings/callerids, then that file will be played prior to the message, playing the file instead of saying the digits of the caller ID number. Valid options are yes or no. Default is no. |
| sayduration | Determines whether to play the duration of the message prior to message playback. Valid options are yes or no. Default is yes. |
| saydurationm | Allows you to set the minimum duration to play \(in minutes\). For example, if you set the value to 2, you will not be informed of the message length for messages less than 2 minutes long. Valid values are whole numbers. Default is 2. |
| searchcontexts | For applications such as Voicemail\(\), VoicemailMain\(\), and Directory\(\), the voicemail context is an optional argument. If the voicemail context is not specified, then the default is to only search the default context. With this option enabled, all contexts will be searched. This comes with a caveat that, if enabled, the mailbox number must be unique across all contexts—otherwise there will be a collision, and the system will not understand which mailbox to use. Valid options are yes and no. Default is no. |
| sendvoicemail | Allows the user to compose and send a voicemail message from within the VoicemailMain\(\) application. Available as option 5 under the advanced menu. If this option is disabled, then option 5 in the advanced menu will not be prompted. Valid options are yes or no. Default is no. |
| tempgreetwarn | Enables a notice to the user when their temporary greeting is enabled. Valid options are yes or no. Default is no. |
| tz | Sets the time zone for a voicemail user \(or globally\). See /usr/share/timezone for different available time zones. Not applicable if envelope=no. |
| volgain | The volgain option allows you to set volume gain for voicemail messages. The value is in decibels \(dB\). The sox application must be installed for this to work. |

_Table 8-4. Mailbox options_

## Voicemail Dialplan Integration

There are two primary dialplan applications provided by the app\_voicemail.so module in Asterisk. The first, simply named VoiceMail\(\), does exactly what you would expect it to, which is to record a message in a mailbox. The second one, VoiceMailMain\(\), allows a user to log into a mailbox to retrieve messages.

### The VoiceMail\(\) Dialplan Application

When you want to pass a call to voicemail, you need to provide two arguments: the mailbox \(or mailboxes\) in which the message should be left, and any options relating to this, such as which greeting to play or whether to mark the message as urgent. The structure of the VoiceMail\(\) command is this:

VoiceMail\(mailbox\[@context\]\[&mailbox\[@context\]\[&...\]\]\[,options\]\)

The options you can pass to VoiceMail\(\) that provide a higher level of control are detailed in [Table 8-5](8.%20Voicemail%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22Voicemail_id292600).

Table 8-5. VoiceMail\(\) optional arguments

| Argument | Purpose |
| :--- | :--- |
| b | Instructs Asterisk to play the busy greeting for the mailbox \(if no busy greeting is found, the unavailable greeting will be played\). |
| d\(\[c\]\) | Accepts digits to be processed by context c. If the context is not specified, it will default to the current context. |
| g\(\#\) | Applies the specified amount of gain \(in decibels\) to the recording. Only works on DAHDI channels. |
| s | Suppresses playback of instructions to the callers after playing the greeting. |
| u | Instructs Asterisk to play the unavailable greeting for the mailbox \(this is the default behavior\). |
| U | Indicates that this message is to be marked as urgent. The most notable effect this has is when voicemail is stored on an IMAP server. In that case, the email will be marked as urgent. When the mailbox owner calls in to the Asterisk voicemail system, he should also be informed that the message is urgent. |
| P | Indicates that this message is to be marked as priority. |

The VoiceMail\(\) application sends the caller to the specified mailbox, so that they can leave a message. The mailbox should be specified as mailbox@context, where context is the name of the voicemail context \(not the dialplan context\). The option letters b or u can be added to request the type of greeting. If the letter b is used, the caller will hear the mailbox owner’s busy message \(if one exists\). If the letter u is used, the caller will hear the mailbox owner’s unavailable message \(also assuming one exists\). If no greeting exists, the system will generate a generic message: The person at extension &lt;mailbox&gt; is unavailable. Please leave a message at the tone.

In the dialplan we built in [Chapter 6](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html%22%20/l%20%22asterisk-DP-Basics), we created several extensions. Consider this simple example extension 102, which allows people to call UserB\_DeskPhone:

exten =&gt; 102,1,Dial\(${UserB\_DeskPhone},10\)

 same =&gt; n,Playback\(vm-nobodyavail\)

 same =&gt; n,Hangup\(\)

We faked a voicemail by playing a prompt that didn’t actually do anything. Let’s change that so the call goes to an actual mailbox instead. For now we’ll just let voicemail play a generic greeting that the caller will hear. Remember, the second argument to the Dial\(\) application is a timeout. If the call is not answered before the timeout expires, the call is sent to the next priority. We’ve got a 10-second timeout, and a new priority to send the caller to voicemail after the dial timeout:

exten =&gt; 102,1,Dial\(${UserB\_DeskPhone},10\)

 same =&gt; n,Voicemail\(${EXTEN}@default,u\)\)

 same =&gt; n,Hangup\(\)

We can do more if we wish, and change it so that if the user is busy \(on another call\), the caller will hear a busy message. To do this, we will make use of the ${DIALSTATUS} variable, which contains one of several status values \(type core show application Dial at the Asterisk console for a listing of all the possible values\):

exten =&gt; 102,1,Dial\(${UserA\_SoftPhone}\)

 same =&gt; n,GotoIf\($\["${DIALSTATUS}" = "BUSY"\]?busy:unavail\)

 same =&gt; n\(unavail\),VoiceMail\(101@default,u\)

 same =&gt; n,Hangup\(\)

 same =&gt; n\(busy\),VoiceMail\(101@default,b\)

 same =&gt; n,Hangup\(\)

Now callers will get voicemail \(with the appropriate greeting\) if the user is either busy or unavailable. An alternative syntax is to use the IF\(\) function to define which of the unavailable or busy messages to use:[3](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch08.html%22%20/l%20%22idm46178407326408)

exten =&gt; 103,1,Dial\(${UserB\_SoftPhone}\)

 same =&gt; n,Voicemail\(${EXTEN}@default,${IF\($\["${DIALSTATUS}" = "BUSY"\]?b:u\)}\)

 same =&gt; n,Hangup\(\)

A slight problem remains, however, in that our users have no way of retrieving their messages, nor setting their greetings or any other voicemail options. We will remedy that in the next section.

### The VoiceMailMain\(\) Dialplan Application

Users can retrieve their voicemail messages, change their voicemail options, and record their voicemail greetings using the VoiceMailMain\(\) application. VoiceMailMain\(\) accepts two arguments: the mailbox number \(and context if necessary\), plus a few options. Both arguments are optional.

The structure of the VoiceMailMain\(\) application looks like this:

VoiceMailMain\(\[mailbox\]\[@context\]\[,options\]\)

If you do not pass any arguments to VoiceMailMain\(\), it will play a prompt asking the caller to provide their mailbox number. The options that can be supplied are listed in [Table 8-6](8.%20Voicemail%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22Voicemail_id292757).

Table 8-6. VoiceMailMain\(\) optional arguments

<table>
  <thead>
    <tr>
      <th style="text-align:left">Argument</th>
      <th style="text-align:left">Purpose</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">p</td>
      <td style="text-align:left">Allows you to treat the mailbox parameter as a prefix to the mailbox number.</td>
    </tr>
    <tr>
      <td style="text-align:left">g(#)</td>
      <td style="text-align:left">Increases the gain by # decibels when playing back messages.</td>
    </tr>
    <tr>
      <td style="text-align:left">s</td>
      <td style="text-align:left">Skips the password check.</td>
    </tr>
    <tr>
      <td style="text-align:left">a(folder)</td>
      <td style="text-align:left">
        <p>Starts the session in one of the following voicemail folders (defaults
          to 0):</p>
        <ul>
          <li>0 - INBOX</li>
          <li>1 - Old</li>
          <li>2 - Work</li>
          <li>3 - Family</li>
          <li>4 - Friends</li>
          <li>5 - Cust1</li>
          <li>6 - Cust2</li>
          <li>7 - Cust3</li>
          <li>8 - Cust4</li>
          <li>9 - Cust5</li>
        </ul>
      </td>
    </tr>
  </tbody>
</table>To allow users to dial an extension to check their voicemail, you could add an extension to the dialplan like this:

exten =&gt; \*98,1,NoOp\(Access voicemail retrieval.\)

 same =&gt; n,VoiceMailMain\(\)

Any user whose device is assigned to the \[sets\] context can now dial \*98, and they’ll be able to log into their mailbox to listen to messages, record their name, set their greeting, and so forth.

### Standard Voicemail Keymap

[Figure 8-1](8.%20Voicemail%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22Voicemail_id293125_8-1) shows the standard keymap configuration for Asterisk Mail. Some options may be enabled or disabled based on the configuration of voicemail.conf \(e.g., envelope=no\). This can be given to users as a reference.

![](.gitbook/assets/0%20%287%29.png)

**Figure 8-1. Keymap configuration for Comedian Mail**

### Creating a Dial-by-Name Directory

One last feature of the Asterisk voicemail system that we should cover is the dial-by-name directory. This is created with the Directory\(\) application. This application uses the names defined in the mailboxes in voicemail.conf to present the caller with a dial-by-name directory of users.

Directory\(\) takes up to three arguments: the voicemail context from which to read the names, the optional dialplan context in which to dial the user, and an option string \(which is also optional\). By default, Directory\(\) searches for the user by last name, but passing the f option forces it to search by first name instead. Let’s add two dial-by-name directories to the TestMenu context of our sample dialplan, so that callers can search by either first or last name:

exten =&gt; 4,1,Dial\(${UserB\_SoftPhone},10\)

 same =&gt; n,Playback\(vm-nobodyavail\)

 same =&gt; n,Hangup\(\)

exten =&gt; 8,1,Directory\(default,sets,f\)

exten =&gt; 9,1,Directory\(default,sets\)

exten =&gt; i,1,Playback\(pbx-invalid\)

 same =&gt; n,Goto\(TestMenu,start,1\)

If you call 201, and then press 8, you’ll get a directory by first name. If you dial 9, you’ll get the directory by last name.

## Voicemail to Email

When Asterisk first came out, it did something very simple that was nevertheless revolutionary within the PBX market of the time. None of the major PBX brands could figure out how to effectively send voice messages to email \(which, put simply, is just sending an email with the message itself as a WAV file attachment\). Sure, some manufacturers offered the functionality, but it was needlessly complex, unreliable, and expensive. Asterisk cut through all that nonsense and just allowed a mailbox to have an assigned email address, and messages would simply be sent through the normal email mechanisms of Linux. This proved both simple and effective, and really showed how out-of-date and out-of-touch the traditional PBX manufacturers were.

Unfortunately, in every great story there’s always a bad guy, and in this case a whole epidemic of them: spammers nearly brought the internet to its knees. The simple SMTP relay could no longer be trusted, as any machine open to relaying email would quickly become a vector for spam.

So, email became far more complex. If you want to send email from your Asterisk system, you have three fundamental ways to do that, as shown in [Table 8-7](8.%20Voicemail%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22table0807).

Table 8-7. Overview of methods for transmitting voicemail to email

| Method | Cons | Pros |
| :--- | :--- | :--- |
| 1. Send email in the clear, directly to the SMTP port of the MX record the target domain returns. Almost guaranteed to fail. | Downstream spam filters will tend to discard suspicious traffic, and this traffic will look very suspicious. | No configuration required on the Asterisk server. |
| 2. Relay your email through a host that knows and trusts your system. Solid DNS and mail server skills are required by the team handling the relay server. | The downstream relay server will need to be configured to work correctly with this arrangement \(it will need to trust email relayed from your Asterisk server\). | Relatively simple to configure on the Asterisk server. |
| 3. Create a normal user account on an email server \(complete with a valid email address\), and send emails as an authenticated user through that platform. We recommend this method since it tends to work very well, and the requirements can be easily communicated to the team that maintains your email. | Slightly more complicated to set up on the Asterisk server. | Easy to set up an email account for Asterisk: you just have to create a user on your email system named “Company PBX” or some name that identifies it, and then use the credentials for this user to send all email through. |

Essentially, what you need to do is make sure the Mail Transport Agent \(MTA\)[4](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch08.html%22%20/l%20%22idm46178407263160) of your Asterisk server can send email from the asterisk shell/user account. The Asterisk voicemail engine will use the same mechanisms to send your voicemail to email.

For further information on the subject of MTAs, you’ll want to consult a Linux administration book such as UNIX and Linux System Administration Handbook, 5th Edition, or an MTA-specific title such as O’Reilly’s Postfix: The Definitive Guide.

## Voicemail Storage Backends

The storage of messages on traditional voicemail systems has always tended to be overly complicated.[5](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch08.html%22%20/l%20%22idm46178407258632) Asterisk not only provides you with a simple, logical filesystem-based storage mechanism, but also offers a few extra message storage options.

### Linux Filesystem

By default, Asterisk stores voice messages in the spool, at /var/spool/asterisk/voicemail/&lt;voicemailcontext&gt;/&lt;mailbox&gt;. The messages can be stored in multiple formats \(such as wav and wav49\), depending on what you specified as the format in the \[general\] section of your voicemail.conf file. Your greetings are also stored in this folder.

**Note**

Asterisk does not create a folder for any mailboxes that do not have any recordings yet \(as would be the case with a new mailbox\), so this folder cannot be used as a reliable method of determining which mailboxes exist on the system.

[Figure 8-2](8.%20Voicemail%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22Voicemail_id293125_8-2) shows an example of what might be in a mailbox folder. This mailbox has no new messages in the INBOX, has two saved messages in the Old folder, and has busy, unavailable and name \(greet\) greetings recorded.

![](.gitbook/assets/1%20%281%29.png)

**Figure 8-2. Sample mailbox folder**

**Note**

For each message, there is a matching msg\#\#\#\#.txt file, which contains the envelope information for the message. The msg\#\#\#\#.txt file is also critically important for message waiting indication \(MWI\), as this is the file that Asterisk looks for in the INBOX to determine whether the message light for a user should be on or off.

### IMAP

Some organizations prefer to manage voicemail as part of their email system. This has been called unified messaging by the telecom industry, and its implementation has traditionally been expensive and complex. Asterisk allows for a fairly simple integration between voicemail and email, either through its built-in voicemail-to-email handler, or through a relationship with an IMAP server. We don’t recommend IMAP integration simply because it’s a lot of work for very little gain, and it is out of scope for this book.

### Message Storage in a Database

It is possible to configure Asterisk voicemail to store messages as blobs within a database. This was originally seen as a simple way to allow synchronization of messages between systems. We’ve never been fans of the idea, since databases are not designed for bulk storage of binary data, and there are many other ways to synchronize files across systems.

## Conclusion

Asterisk’s voicemail system is a mature and capable module, and an essential part of any PBX. It’s not likely to be enhanced beyond what it does, but that’s not likely to be a problem, either.

[1](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch08.html%22%20/l%20%22idm46178407606744-marker) This name was a play on words, inspired in part by Nortel’s voicemail system Meridian Mail. Nortel \(and Meridian Mail\) are gone, but Comedian Mail soldiers on.

[2](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch08.html%22%20/l%20%22idm46178407573752-marker) Also sometimes called a Message Transfer Agent.

[3](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch08.html%22%20/l%20%22idm46178407326408-marker) We’ll dive into functions like IF\(\) in [Chapter 10](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch10.html%22%20/l%20%22asterisk-DP-Deeper).

[4](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch08.html%22%20/l%20%22idm46178407263160-marker) Popular MTAs these days are Postfix and Exim. The ubiquitous sendmail still exists as well, although its popularity has waned in the past few years. You’ll find Postfix on your RHEL/CentOS machines by default, and likely Exim on your Debian/Ubuntu platforms \(although Postfix is often recommended as the MTA there too\).

[5](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch08.html%22%20/l%20%22idm46178407258632-marker) Nortel used to store its messages in a sort of special partition, in a proprietary format, which made it pretty much impossible to extract messages from the system, or email them, or archive them, or really do anything with them. Ah, the good old days of closed, proprietary systems. We miss ... no ... wait ... we do not miss them!
