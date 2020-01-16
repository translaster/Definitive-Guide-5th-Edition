# Глава 14. Автосекретарь

> _Я не отвечаю на звонки. У меня всегда такое чувство, что на другом конце провода кто-то есть._
>
>  -- Фред Коуплз

Во многих УАТС, как правило, есть система меню для автоматического ответа на входящие вызовы, которая позволяет абонентам направлять себя на различные расширения и ресурсы в системе с помощью выбора меню. Эта система известна в телекоммуникационной отрасли как _автосекретарь_ (АС). АС, как правило, предоставляет следующие возможности:

* Перевод на добавочный номер
* Перевод на голосовую почту
* Переход в очередь
* Воспроизведение сообщения (например “наш адрес…”)
* Подключение к подменю (например “для получения списка наших отделов...”)
* Подключение к приемной
* Повтор выбора

Для всего остального, особенно если требуется внешняя интеграция, например поиск по базе данных, обычно требуется интерактивное голосовое меню (сокращенно на английском - IVR).

## АС это не IVR

В open sorce телекоммуникационном сообществе вы часто услышите термин IVR, используемый для описания автосекретаря.<sup><a href="#sn1">1</a></sup> Однако в телекоммуникационной отрасли в течение многих десятилетий до появления VoIP или УАТС с открытым исходным кодом IVR отличался от AС. По этой причине, когда вы говорите с кем-то, имеющим многолетний опыт работы в области телекоммуникаций, о любом виде телекоммуникационного меню, вы должны убедиться, что говорите об одном и том же. Для специалиста по телекоммуникациям термин _IVR_ подразумевает относительно комплексную и сложную в разработке (и последующе затратную), в то время как AС - это простая и недорогая вещь, которая является общей для большинства АТС.

В этой главе мы поговорим о создании автосекретаря. IVR мы обсудим в [Главе 16](glava-16.md).<sup><a href="#sn2">2</a></sup>

## Проектирование вашего АС

Самая распространенная ошибка новичков при проектировании АС - излишняя сложность. В то время как может быть много радости и чувства выполненного долга в создании многоуровневого AС с десятками отличных вариантов и кучами действительно интересных подсказок, ваши абоненты имеют другую повестку дня. Люди звонят по телефону в первую очередь потому, что хотят с кем-то поговорить. В то время как люди привыкли к реальности автосекретарей (и в некоторых случаях они могут ускорить процесс), по большей части люди предпочитают говорить с кем-то живым. Это означает, что есть два основных правила, которых должен придерживаться каждый АС:

* Быть проще.
* Убедитесь, что вы всегда делаете обработчик для людей, которые собираются нажимать 0, когда слышат меню. Если вы не хотите иметь опцию 0, имейте в виду, что многие люди будут оскорблены этим, они повесят трубку и не перезвонят. В бизнесе это вообще плохо.

Прежде чем вы начнете кодировать свой AС, разумно его спроектировать. Вам нужно будет определить поток вызовов и необходимо будет указать подсказки, которые будут воспроизводиться на каждом шаге. Программные инструменты для построения диаграмм могут быть полезны для этого, но нет необходимости фантазировать. Таблица 14-1 предоставляет хороший шаблон для базового AС, который будет делать то, что вам нужно.

_Таблица 14-1. Базовый автосекретарь_

| Шаг или выбор |	Образец приглашения |	Примечание | Имя файла<sup><a href="#sn1a">a</a></sup> |
| :------------ | :------------------ | :--------- | :----------------------------------- |
| Приветствие - рабочее время | Спасибо,что позвонили в компанию ABC. | Дневное приветствие. Воспроизводится сразу после того, как система отвечает на вызов. | _daygreeting.wav_ |
| Приветствие - нерабочеее время | Спасибо,что позвонили в компанию ABC. Наш офис сейчас закрыт. | Ночное приветствие. Как и выше, но играет в нерабочее время. | _nightgreeting.wav_ |
| Главное меню | Если вы знаете внутренний номер оператора, с которым хотите связаться, пожалуйста, введите его сейчас. Для свзи с отделом продаж, пожалуйста, нажмите 1; для отдела обслуживания, нажмите 2; для нашего каталога компании, нажмите #. Для получения информации о нашем адресе и факсе нажмите кнопку 3. Чтобы повторить это сообщение - нажмите 9, или оставайтесь на линии, или нажмите 0, чтобы связаться с операторому. | Подсказка главного меню. Играет сразу после приветствия. Для вызывающего абонента приветствие и главное меню звучат как единое приглашение; однако в системе полезно держать эти подсказки раздельно. | _mainmenu.wav_ |
| 1 | Пожалуйста, подождите пока мы переключим ваш звонок. | Перевод на очередь sales. | _holdwhileweconnect.wav_ |
| 2 |   |   |   |
| # |   |   |   |
| 3 |   |   |   |
| 0 |   |   |   |
| 9 |   |   |   |
| t |   |   |   |
| i |   |   |   |
| _XXX<sup><a href="#sn1b">b</a></sup> |   |   |   |

<!--
<table>
  <thead>
    <tr>
      <th style="text-align:left">Шаг или выбор</th>
      <th style="text-align:left">Образец приглашения</th>
      <th style="text-align:left">Примечание</th>
      <th style="text-align:left">Имя файла<sup><a href="#sn1a">a</a></sup></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">Приветствие - рабочее время</td>
      <td style="text-align:left">Спасибо,что позвонили в компанию ABC.</td>
      <td style="text-align:left">Дневное приветствие. Воспроизводится сразу после того, как система отвечает на вызов.</td>
      <td style="text-align:left"><em>daygreeting.wav</em></td>
    </tr>
    <tr>
      <td style="text-align:left">Приветствие - нерабочеее время</td>
      <td style="text-align:left">Спасибо,что позвонили в компанию ABC. Наш офис сейчас закрыт.</td>
      <td style="text-align:left">Ночное приветствие. Как и выше, но играет в нерабочее время.</td>
      <td style="text-align:left"><em>nightgreeting.wav</em></td>
    </tr>
    <tr>
      <td style="text-align:left">Главное меню</td>
      <td style="text-align:left">Если вы знаете внутренний номер оператора, с которым хотите связаться, пожалуйста, введите его сейчас. Для свзи с отделом продаж, пожалуйста, нажмите 1; для отдела обслуживания, нажмите 2; для нашего каталога компании, нажмите #. Для получения информации о нашем адресе и факсе нажмите кнопку 3. Чтобы повторить это сообщение - нажмите 9, или оставайтесь на линии, или нажмите 0, чтобы связаться с операторому.</td>
      <td style="text-align:left">Подсказка главного меню. Играет сразу после приветствия. Для вызывающего абонента приветствие и главное меню звучат как единое приглашение; однако в системе полезно держать эти подсказки раздельно.</td>
      <td style="text-align:left"><em>mainmenu.wav</em></td>
    </tr>
    <tr>
      <td style="text-align:left">1</td>
      <td style="text-align:left">Пожалуйста, подождите пока мы переключим ваш звонок.</td>
      <td style="text-align:left">Перевод на очередь sales.</td>
      <td style="text-align:left"><em>holdwhileweconnect.wav</em>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">2</td>
      <td style="text-align:left">Пожалуйста, подождите пока мы переключим ваш звонок.</td>
      <td style="text-align:left">Перевод на очередь support.</td>
      <td style="text-align:left"><em>holdwhileweconnect.wav</em>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">#</td>
      <td style="text-align:left">n/a</td>
      <td style="text-align:left">Запуск приложения <code>Directory()<code></td>
      <td style="text-align:left">n/a</td>
    </tr>
    <tr>
      <td style="text-align:left">3</td>
      <td style="text-align:left">Наш адрес [address]. Наш номер факса [fax number], и тд.</td>
      <td
      style="text-align:left">Воспроизведение записи, содержащей информацию об адресе и факсе. Возврат вызывающего абонента к подсказке меню после завершения.</td>
      <td style="text-align:left"><em>faxandaddress.wav</em></td>
    </tr>
    <tr>
      <td style="text-align:left">0</td>
      <td style="text-align:left">Соединяем со специалистом. Пожалуйста, подождите.</td>
      <td style="text-align:left">Перевод на приемную/оператора.</td>
      <td style="text-align:left"><em>transfertoreception.wav</em></td>
    </tr>
    <tr>
      <td style="text-align:left">9</td>
      <td style="text-align:left">n/a</td>
      <td style="text-align:left">Повтор. Воспроизведение подсказки меню (но не приветствие).</td>
      <td style="text-align:left">n/a</td>
    </tr>
    <tr>
      <td style="text-align:left">t</td>
      <td style="text-align:left">n/a</td>
      <td style="text-align:left">Тайм-аут. Если абонент не сделал выбора - считайте, что он набрал 0 (или в некоторых случаях повторите подсказку).</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">i</td>
      <td style="text-align:left">Вы сделали неверный выбор. Пожалуйста, попробуйте еще раз.</td>
      <td style="text-align:left">Абонент нажал неверную цифру: воспроизведение подсказки меню (но не приветствия).</td>
      <td style="text-align:left"><em>invalid.wav</em></td>
    </tr>
    <tr>
      <td style="text-align:left">_XXX <sup><a href="#sn1b">b</a></sup></td>
      <td style="text-align:left">n/a</td>
      <td style="text-align:left">Перевод вызова на вызываемый номер.</td>
      <td style="text-align:left"><em>holdwhileweconnect.wav</em></td>
    </tr>
  </tbody>
</table>
-->

<p><sup><a name="sn1a">a</a></sup>Эти файлы пока нигде не существуют. Мы используем их в качестве примеров.</p>
<p><sup><a name="sn1b">b</a></sup>Это совпадение шаблонов должно соответствовать вашему диапазону расширений.</p>

---

Давайте рассмотрим различные компоненты этого шаблона. Затем мы покажем вам код диалплана, необходимый для его реализации, а также как создавать подсказки.

### Приветствие

Первое, что слышит абонент - это на самом деле две подсказки.

Первая подсказка - это приветствие. Единственное, что приветствие должно сделать - это поприветствовать абонента. Примером приветствия может быть “Спасибо, что позвонили Брайанту, Ван Меггелену и партнерам”, “Добро пожаловать в школу мудрости и дизайна футболок Лейфа" или "Вы позвонили в офисы адвокатов Дьюи, Читама и Хоуи.” Вот именно - выбор для звонящего придет позже. Это позволяет записывать различные приветствия без необходимости записывать все новое меню. Например, в течение нескольких недель каждый год вы можете делать, чтобы ваше приветствие говорило "приветствие сезона" или что-то еще, но ваше меню не нужно будет менять. Кроме того, если вы хотите воспроизвести другую запись в нерабочее время (“Спасибо за звонок. Наш офис сейчас закрыт.”), то можете использовать разные приветствия, но сердце меню останется прежним. Наконец, если вы хотите иметь возможность вернуть абонентов в меню из другой части системы, то не захотите, чтобы они снова услышали приветствие.

### Главное меню

Приглашение главного меню - это место, где вы сообщаете своим абонентам о доступных им вариантах выбора. Вы должны проговаривать его как можно быстрее (без спешки или глупостей).<sup><a href="#sn3">3</a></sup> Когда вы записываете выбор, всегда сообщайте пользователям о действии, которое будет предпринято, прежде чем дать им возможность выполнить это действие с помощью набора цифр. Поэтому не говорите "нажмите 1 для продаж“, а скорее скажите: "для продаж нажмите 1". Причина этого заключается в том, что большинство людей не обратят должного внимания на подсказку, пока не услышат интересующий их выбор. Как только они услышат свой выбор, то будут в полном внимании и им можно будет сообщить какую кнопку нажать, чтобы направить их туда, куда требуется.

Еще один момент, который следует рассмотреть - это порядок, в котором следует разместить выбор. Типичный бизнес, например, захочет, чтобы продажи были первым пунктом меню и большинство абонентов также будут ожидать этого. Самое главное - думать о своих клиентах. Например, большинство людей не будут заинтересованы в адресной и факсимильной информации, поэтому не делайте их первыми пунктами.<sup><a href="#sn4">4</a></sup> Подумайте о том, чтобы как можно быстрее соединить абонентов с местом их назначения, когда вы проектируете свой выбор. Безжалостно режьте все, что не является абсолютно необходимым.

#### Выбор 1

Вариант 1 в нашем примере будет простым переводом. Обычно это ресурс, расположенный в другом контексте, и он, как правило, имеет внутренний добавочный номер, так что внутренние пользователи могут также передавать вызовы на него. В этом примере мы будем использовать эту опцию для отправки абонентов в очередь `sales`, созданную в [Главе 12](glava-12.md).

#### Выбор 2

Вариант 2 будет технически идентичен варианту 1. Только место назначения будет другим. Этот выбор переместит абонентов в очередь `support`.

#### Выбор `#`

Хорошо иметь возможность выбора справочника как можно ближе к началу записи. Многие люди будут использовать каталог, если знают что тот существует, но им не требуется прослушивать все меню, чтобы узнать о нем. Нетерпеливые люди будут нажимать 0, поэтому чем раньше вы расскажете им о справочнике, тем больше шансов, что они им воспользуются, и тем самым уменьшится нагрузка на вашего секретаря.

#### Выбор 3

Если у вас есть опция, которая ничего не делает, кроме воспроизведения записи абоненту (например, адрес и факс), вы можете оставить весь код для неё в том же контексте, что и меню, и просто вернуть абонента в главное меню в конце записи. В общем, такие варианты не очень полезны, как нам хотелось бы думать, поэтому в большинстве случаев вы, вероятно, захотите оставить её.

#### Выбор 9

Очень важно дать звонящему возможность услышать подсказку еще раз. Многие люди не будут обращать внимания на все меню, и если вы не дадите им возможность услышать меню снова, они, скорее всего, нажмут 0.

Обратите внимание, что вам не нужно воспроизводить приветствие снова, только подсказку главного меню.

#### Выбор 0

Как было сказано ранее, и нравится вам это или нет, это выбор, который предпочтут многие (возможно, большинство) ваши абоненты. Если вы действительно не хотите, чтобы кто-то отвечал на эти звонки, то можете отправлять вызовы в почтовый ящик, но мы не рекомендуем это делать. Если вы занимаетесь бизнесом, многие из ваших абонентов будут вашими клиентами. Вы же хотите, чтобы им было легко связаться с вами. Доверьтесь нам.

### Тайм-аут

Многие люди будут звонить по номеру и не обращать слишком много внимания на то, что происходит. Они знают, что если будут просто ждать на линии, то в конце концов их переведут на оператора. Или, возможно, они находятся в автомобиле, и на самом деле не могут нажимать кнопки своих телефонов. В любом случае, сделайте им одолжение. Если они не делают никакого выбора, не преследуйте их и не заставляйте их делать этого. Соедините их с оператором.

### Invalid (Неверно)

Люди совершают ошибки. Это нормально. Неверный обработчик сообщит им о том, что они выбрали, не является допустимым вариантом, и вернет их в приглашение меню, чтобы они могли повторить попытку выбора. Обратите внимание, что вы не должны воспроизводить приветствие снова, только приглашение главного меню.

### Вызов добавочного номера

Если кто-то звонит в вашу систему и знает добавочный номер, который хочет набрать, ваш автосекретарь должен иметь код для обработки этого.

<table border="1" width="100%" cellpadding="5">
  <tr>
    <td>
      <p align="left"><b>Примечание</b></p>
      <p>Хотя Asterisk может обрабатывать перекрытие между вариантами меню и добавочными номерами (например, у вас может быть выбор меню `1` и расширения от 100 до 199), лучше избегать этого перекрытия. В противном случае диалплану всегда придется ожидать межразрядный тайм-аут всякий раз, когда кто-то нажимает `1`, потому что не будет знать, планируют ли набрать добавочный номер 123. Межразрядный тайм-аут - это задержка, которую система допускает между цифрами, прежде чем предполагает, что было введено все число. Этот таймер гарантирует, что абоненты имеют достаточно времени для набора многоразрядного расширения, но он также вызывает задержку в обработке одноразрядных вводов.</p>
    </td>
  </tr>
</table>

## Создание вашего АС

После того, как вы разработали свой АС, есть три вещи, которые вам нужно сделать, чтобы заставить его работать должным образом:

* Запись подсказок.
* Создание абонентской группы для меню.
* Направление входящих каналов в контекст AС.

Начнем с того, что поговорим о записях.

### Запись подсказок

Recording prompts for a telephone system is a critical task. This is what your callers will hear when they interact with your system, and the quality and professionalism of these prompts will reflect on your organization.

Asterisk is very flexible in this regard and can work with many different audio formats. We have found that, in general, the most useful format to use is WAV. Files saved in this format can be of many different kinds, but only one type of WAV file will work with Asterisk: files must be encoded in 16-bit, 8,000 Hz, mono format.

**Recommended Prompt File Format**

The WAV file format we have recommended is useful for system prompts because it can easily be converted to any other format that your phones might use, without loss or distortion, and almost any computer can play it without any special software. Thus, not only can Asterisk handle the file easily, but it is also easy to work with it on a PC \(which can be useful\). Asterisk can handle other file formats as well, and in some cases these may be more suitable for your needs, but in general we find 16-bit 8 kHz WAV files to be the easiest to work with and, most of the time, the best possible quality.

There are essentially two ways to get prompts into a system. One is to record sound files in a studio or on a PC, and then move those files into the system. A second way is to record the prompts directly onto the system using a telephone set. We prefer the second method.

Our advice is this: don’t get hung up on the complexities of recording audio through a PC or in a studio.<sup><a href="#sn5">5</a></sup> It is generally not necessary. A telephone set will produce excellent-quality recordings, and the reasons are simple: the microphone and electronics in a telephone are carefully designed to capture the human voice in a format that is ideal for transmission on telephone networks, and therefore a phone set is also ideal for doing prompts. The set will capture the audio in the correct format, and will filter out background noise and normalize the decibel level.

**Note**

Yes, a properly produced studio prompt will be superior to a prompt recorded over a telephone, but if you don’t have the equipment or experience, take our advice and use a telephone to do your recordings, because a poorly produced studio prompt will be much worse than a prompt recorded through a phone set.

#### Using the dialplan to create recordings

The simplest method of recording prompts is to use the Record() application.

Add this new subroutine at the bottom of your extensions.conf file:

```
[subRecordPrompt]
exten => 500,1,Playback(vm-intro)
   same => n,Record(daygreeting.wav)
   same => n,Wait(2)
   same => n,Playback(daygreeting)
   same => n,Hangup

exten => 501,1,Playback(vm-intro)
   same => n,Record(mainmenu.wav)
   same => ... etc ... (create dialplan code for each prompt you need to record)
```

**Note**

In order to use this context, you will need to include it in the context where your sets enter the dialplan. So in your \[LocalSets\] context, you will want to add the line include=&gt;UserServices. In a production environment, you’ll probably want a password on this so that not just anybody can record prompts.

This subroutine plays a prompt, issues a beep, makes a recording, and plays that recording back.<sup><a href="#sn6">6</a></sup> It’s notable that the Record() application takes the entire filename as its argument, while the Playback() application excludes the filetype extension (.wav, .gsm, etc.). This is because the Record() application needs to know which format the recording should be made in, while the Playback() application does not. Instead, Playback() automatically selects the best audio format available, based upon the codec your handset is using and the formats available in the sounds folder \(for example, if you have a daygreeting.wav and a daygreeting.gsm file in your sounds folder, Playback\(daygreeting\) will select the one that requires the least CPU to play back to the caller\).

You’ll probably want a separate extension for recording each of the prompts, possibly hidden away from your normal set of extensions, to prevent a mistyped extension from wiping out any of your current menu prompts. If the number of prompts you have is large, repeating this extension with slight modifications for each will get tedious, but there are ways around that. We’ll show you how to make your prompt recording more intelligent in [Chapter 16](glava-16.md), but for now the method just described will serve our immediate needs.

Here’s the dialplan (in bold) that’ll create all our prompts. Place it wherever you wish in the `[sets]` context:

```
exten => _4XX,1,Noop(User Dialed ${EXTEN})
  same => n,Answer()
  same => n,SayDigits(${EXTEN})
  same => n,Hangup()

exten => 500,1,GoSub(subRecordPrompt,${EXTEN},1(daygreeting)
exten => 501,1,GoSub(subRecordPrompt,${EXTEN},1(nightgreeting)
exten => 502,1,GoSub(subRecordPrompt,${EXTEN},1(mainmenu)
exten => 503,1,GoSub(subRecordPrompt,${EXTEN},1(holdwhileweconnect)
exten => 504,1,GoSub(subRecordPrompt,${EXTEN},1(faxandaddress)
exten => 505,1,GoSub(subRecordPrompt,${EXTEN},1(transfertoreception)
exten => 506,1,GoSub(subRecordPrompt,${EXTEN},1(invalid)
exten => 507,1,GoSub(subRecordPrompt,${EXTEN},1(holdwhileweconnect)

exten => _555XXXX,1,Answer()
  same => n,SayDigits(${EXTEN})
exten => _55512XX,1,Answer()
  same => n,Playback(tt-monkeys)
```

The recordings (aka prompts) will be placed in the /var/lib/asterisk/sounds folder. You can put them elsewhere, so long as you specify the full path when recording and playing back (and ensure the directory where you put them is readable by the asterisk user). In a production system, you should put them elsewhere, so as to separate your custom prompts from the generic prompts. For now, we’ll keep things simple and put them in the same folder as the system prompts.

### The Dialplan

Here is the code required to create the AA that we designed earlier. We will often use blank lines before labels within an extension to make the dialplan easier to read, but note that just because there is a blank line does not mean there is a different extension.

You can place this code at the end of your `[TestMenu]` context, right before your subroutines:

```
[MainMenu]

exten => s,1,Verbose(1, Caller ${CALLERID(all)} has entered the auto attendant)
    same => n,Answer()

; this sets the inter-digit timer
    same => n,Set(TIMEOUT(digit)=2)

; wait one second to establish audio
    same => n,Wait(1)

; If Mon-Fri 9-5 goto label daygreeting
    same => n,GotoIfTime(9:00-17:00,mon-fri,*,*?daygreeting:afterhoursgreeting)

    same => n(afterhoursgreeting),Background(nightgreeting) ; AFTER HOURS GREETING
    same => n,Goto(menuprompt)

    same => n(daygreeting),Background(daygreeting)   ; DAY GREETING
    same => n,Goto(menuprompt)

    same => n(menuprompt),Background(mainmenu) ; MAIN MENU PROMPT
    same => n,WaitExten(4)                      ; more than 4 seconds is probably
                                                ; too much
    same => n,Goto(0,1)                         ; Treat as if caller has pressed '0'

exten => 1,1,Verbose(1, Caller ${CALLERID(all)} has entered the sales queue)
    same => n,Goto(sets,610,1)     ; Sales Queue - see Chapter 13 for details

exten => 2,1,Verbose(1, Caller ${CALLERID(all)} has entered the service queue)
    same => n,Goto(sets,611,1)     ; Service Queue - see Chapter 13 for details

exten => 3,1,Verbose(1, Caller ${CALLERID(all)} has requested address and fax info)
    same => n,Background(faxandaddress)            ; Address and fax info
    same => n,Goto(s,menuprompt)      ; Take caller back to main menu prompt

exten => #,1,Verbose(1, Caller ${CALLERID(all)} is entering the directory)
    same => n,Directory(default)   ; Send the caller to the directory.
                                   ; Use InternalSets as the dialing context

exten => 0,1,Verbose(1, Caller ${CALLERID(all)} is calling the operator)
    same => n,Goto(sets,611,1)     ; Service Queue - see Chapter 13 for details

exten => i,1,Verbose(1, Caller ${CALLERID(all)} has entered an invalid selection)
    same => n,Playback(invalid)
    same => n,Goto(s,menuprompt)

exten => t,1,Verbose(1, Caller ${CALLERID(all)} has timed out)
    same => n,Goto(0,1)

; You will want to have a pattern match for the various extensions
; that you'll allow external callers to dial
; BUT DON'T JUST INCLUDE THE LocalSets CONTEXT
; OR EXTERNAL CALLERS WILL BE ABLE TO MAKE CALLS OUT OF YOUR SYSTEM

; WHATEVER YOU DO HERE, TEST IT CAREFULLY TO ENSURE EXTERNAL CALLERS
; WILL NOT BE ABLE TO DO ANYTHING BUT DIAL INTERNAL EXTENSIONS

exten => _1XX,1,Verbose(1,Call to an extension starting with '1')
    same => n,Goto(sets,${EXTEN},1)
```

### Delivering Incoming Calls to the AA

Any call coming into the system will enter the dialplan in the context defined for whatever channel the call arrives on. In many cases this will be a context named `[incoming]`, or `[from-pstn]`, or something similar. The calls will arrive either with an extension \(as would be the case with a DID) or without one \(which would be the case with a traditional analog line\).

Whatever the name of the context, and whatever the name of the extension, you will want to send each incoming call to the menu.

```
[incoming] ; a DID coming in on a channel with
           ; context=incoming
exten => 4169671111,1,Goto(MainMenu,s,1)
```

Depending on how you configure your incoming channels, you will generally want to use the Goto\(\) application if you want to send the call to an AA. This is far neater than just coding your whole AA in the incoming context.

Since we don’t have any incoming circuits in our lab,<sup><a href="#sn7">7</a></sup> we’re going to create a simple extension that’ll deliver us to our fancy new AA:

```
exten => 613,1,Noop()
  same => n,Goto(MainMenu,s,1)
  same => n,Hangup()
```

And that’s it! A simple automated attendant that is easy to manage, and will handle the expectations of most callers.

### IVR

Мы рассмотрим интерактивное голосовое меню \(IVR\) более подробно в [Главе 16](glava-16.md), но прежде чем мы это сделаем поговорим о том, что важно для любого IVR. Интеграция баз данных является предметом следующей главы.

## Вывод

Автосекретарь может предоставить очень полезную услугу для абонентов. Однако, если он не разработан и хорошо реализован, то также может стать барьером для ваших абонентов, который может отпугнуть их. Потратьте время, чтобы тщательно спланировать свой АС и держать его простым.

<ol>
<li id="sn1">Это, скорее всего, потому, что "IVR“ гораздо легче сказать, чем "автосекретарь".</li>

<li id="sn2"> Следует отметить, что Asterisk - это отличный инструмент для создания IVR. Но также отлично подходит и для создания автосекретаря.</li>

<li id="sn3"> При необходимости вы можете использовать программу редактирования звука, такую как Audacity, чтобы удалить тишину и даже немного ускорить запись.</li>

<li id="sn4"> На самом деле, мы обычно не рекомендуем это в AС, потому что это добавляется к тому, что абонент должен слушать, и большинство людей все равно пойдет на сайт для получения такого рода информации.</li>

<li id="sn5"> Если вы не являетесь экспертом в этих областях, в таком случае пойдите на это!</li>

<li id="sn6"> Подсказка <i>vm-intro</i> не идеальна (она просит вас оставить сообщение), но она достаточно близка для наших целей. Инструкции по использованию по крайней мере верны: нажмите <b>#</b>, чтобы завершить запись. После того, как вы научились записывать подсказки можете вернуться назад, записать пользовательское приглашение и изменить приоритет 1, чтобы отразить более подходящие инструкции для записи собственных приглашений.</li>

<li id="sn7"> And if you do in yours, congratulations and please be careful swimming with the sharks.</li>
</ol>

[Глава 13. Состояния устройств](glava-13.md) | [Содержание](SUMMARY.md) | [Глава 15. Интеграция реляционной базы данных](glava-15.md)
