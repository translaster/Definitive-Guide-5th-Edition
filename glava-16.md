# Глава 16. Введение в интерактивное голосовое меню

> _Однажды Алиса подошла к развилке и увидела на дереве Чеширского кота. - По какой дороге мне пойти?- спросила она._
> _“Куда ты хочешь пойти?- был его ответ._
> _“Не знаю, - ответила Алиса._
> _“Тогда, - сказал кот, - это не имеет значения.”_
> - Льюис Кэролл

Термин интерактивое голосовое меню (на самом деле ответ) (IVR) часто неправильно используется для обозначения автосекретаря, но это очень разные вещи. Цель системы IVR состоит в том, чтобы принять входные данные от абонента, выполнить действие, основанное на этих входных данных (обычно, поиск данных во внешней системе, такой как база данных), и сообщить результат абоненту. Назначение автосекретаря (о котором мы говорили в [Главе 14](glava-14.md)) - маршрутизация вызовов. Первоначально IVR даже не должен был быть телефонной системой. Все, что принимало информацию от человека и выдавало на запрос результат, падало вместе с областью IVR. Традиционно системы IVR были сложными, дорогими и раздражающими в реализации. Asterisk все это изменил.

## Компоненты IVR

Самые основные элементы IVR очень похожи на элементы автосекретаря, хотя цель и отличается. Нам нужно по крайней мере одно приветствие чтобы сообщить вызывающему, что ожидает IVR, метод получения входных данных от вызывающего, логику для проверки что ответ вызывающего является допустимым вводом, логику определения следующего шага IVR, и, наконец, механизм хранения ответов, если это применимо. Мы могли бы думать об IVR как о дереве решений, хотя оно не должно иметь никаких ветвей. Например, опрос может представлять точно такой же набор подсказок для каждого вызывающего абонента, независимо от того, какой выбор делают абоненты и единственная логика маршрутизации, включенная в опрос, заключается в том, являются ли полученные ответы допустимыми для вопросов.

С точки зрения вызывающего абонента, каждый IVR должен начинаться с подсказки. Этот первоначальный запрос будет сообщать абоненту что он попал на IVR и попросит собеседника ввести первые данные. Мы обсуждали подсказки в автосекретаре в Главе 14. Позже мы создадим диалплан, который позволит вам лучше управлять несколькими голосовыми подсказками.

Второй компонент IVR - это метод получения входных данных от вызывающего абонента. Напомним, что в Главе 14 мы обсуждали `Background()` и `WaitExten()` как метод получения нового расширения. Хотя вы можете создать IVR с помощью `Background()` и `WaitExten()` обычно проще и практичнее использовать приложение `Read()`, которое обрабатывает как приглашение, так и захват ответа. Приложение `Read()` было разработано специально для использования с системами IVR. Его синтаксис выглядит следующим образом:

```
Read(variable[,filename[&filename2...]][,maxdigits][,option][,attempts][,timeout])
```

Аргументы описаны в Таблице 16-1.

_Таблица 16-1. Приложение Read()_

<table border="1" width="100%" cellpadding="5">
  <tr>
    <td><p align="left"><b>Аргумент</b></p></td>
    <td><p align="left"><b>Цель</b></p></td>
  </tr>
  <tr>
    <td><code>variable</code></td>
    <td>Переменная, в которой хранится ответ абонента. Рекомендуется присвоить каждой переменной в IVR имя, аналогичное приглашению, связанному с этой переменной. Это поможет позже, если по деловым соображениям или простоте использования вам потребуется изменить порядок шагов IVR. Присвоение имен переменным <code>var1</code>, <code>var2</code> и т.д. может показаться более простым в краткосрочной перспективе, но позже в вашем жизненном цикле это сделает исправление ошибок более трудным.</td>
  </tr>
  <tr>
    <td><code>prompt</code></td>
    <td>Файл (или список файлов, соединенных вместе с символом <code>&</code>) для воспроизведения вызывающему абоненту, запрашивающий ввод. Не забудьте опустить расширение в конце каждого имени файла.</td>
  </tr>
  <tr>
    <td><code>maxdigits</code></td>
    <td>Максимальное количество символов, которые можно использовать в качестве входных данных. В случае вопросов "Да/нет" и "множественный выбор" рекомендуется ограничить это значение <doe>1</code>. В случае более длинных значений вызывающий абонент всегда может прервать ввод, нажав клавишу <code>#</code>.</td>
  </tr>
  <tr>
    <td><code>options</code></td>
    <td><p><code>s</code>(skip)</p>
    <p>Немедленно выйти, если канал не отвечает.</p>
    <p><code>i</code>(indication)</p>
    <p>Вместо того чтобы воспроизводить подсказку, воспроизведите какой-либо сигнал индикации (например, сигнал набора номера).</p>
    <p><code>n</code>(no answer)</p>
    <p>Считывание цифр от абонента, даже если на линию еще не ответили.</p>
    <p><code>attempts</code></p>
    <p>Количество раз для воспроизведения подсказки. Если вызывающий не вводит ничего, приложение <code>Read()</code> может автоматически запросить пользователя. По умолчанию используется одна попытка.</p>
    <p><code>timeout</code></p>
    <p>Количество секунд, в течение которых вызывающий должен совершить свой ввод. Значение по умолчанию в Asterisk равно 10 секундам, хотя его можно изменить для одного приглашения с помощью этой опции или для всего сеанса, назначив значение с помощью функции диалплана <code>TIMEOUT(response)</code>.</p></td>
  </tr>
</table>

Как только входные данные получены, они должны быть проверены. Если вы не проверите входные данные, то с большей вероятностью обнаружите, что ваши абоненты жалуются на нестабильное приложение. Недостаточно обрабатывать ожидаемые входные данные; вам также нужно обрабатывать входные данные, которых вы не ожидаете. Например, абоненты могут быть разочарованы и набрать 0 в вашем IVR; если вы сделали хорошую работу, вы будете обращаться с этим деликатно и соедините их с кем-то, кто может помочь им или предоставить полезную альтернативу. Хорошо спроектированный IVR (как и любая программа) будет пытаться предвидеть все возможные входные данные и предоставлять механизмы для изящной их обработки.

После проверки входных данных вы можете отправить их на внешний ресурс для обработки. Это может быть сделано с помощью запроса в базу данных, отправки в URI, программы AGI или многих других вещей. Это внешнее приложение должно выдать результат, который вы сможете передать обратно абоненту. Это может быть подробный результат такой как "Баланс вашего счета..." или простое подтверждение такое как "ваш счет был обновлен". Мы не можем придумать ни одного реального случая, когда не будет требоваться какой-то результат, возвращаемый звонящему.

Иногда IVR может иметь несколько шагов, и поэтому результат может включать запрос дополнительной информации от вызывающего абонента для перехода к следующему шагу приложения IVR.

Можно проектировать очень сложные системы IVR с десятками или даже сотнями возможных путей. Мы уже говорили об этом раньше и повторим еще раз: люди не любят разговаривать с вашей телефонной системой независимо от того насколько она умна. Держите ваше IVR простым для ваших абонентов и они гораздо более вероятно получат некоторую выгоду от него.

#### A Perfectly Tasty IVR

An excellent example of an IVR that people love to use is one that many pizza delivery outfits use: when you call to place your order, an IVR looks up your caller ID and says “If you would like the exact same order as last time, press 1.”

That’s all it does, and it’s perfect.

Obviously, these companies could design massively complex IVRs that would allow you to select each and every detail of your pie (“for seven-grain crust, press 7”), but how many inebriated, starving customers could successfully navigate something like that at 3 A.M.?

The best IVRs are the ones that require the least input from the caller. Mash that 1 button and your ’za is on its way! Woo hoo!

## IVR Design Considerations

When designing your own IVR, there are some important things to keep in mind. We’ve put together this list of things to do and things not to do in your IVR.

Do

* Keep it simple.
* Have an option to dial 0 to reach a live person.
* Handle errors gracefully.

Don’t

* Think that an IVR can completely replace people.
* Use your IVR to show people how clever you are.
* Try to replicate your website with an IVR.
* Bother building an IVR if you can’t take numeric or spoken input. Nobody wants to have to spell their name on the dialpad of a phone.<sup><a href="#sn1">1</a></sup>
* Force your callers to listen to advertising. Remember that they can hang up at any moment they wish.

## Asterisk Modules for Building IVRs

The “frontend” of the IVR (the parts that interact with the callers) can be handled in the dialplan. It is possible to build an IVR system using the dialplan alone \(perhaps with the astdb to store and retrieve data\); however, you will typically need to communicate with something external to Asterisk \(the “backend” of the IVR).

### CURL()

The CURL\(\) dialplan function in Asterisk allows you to span entire web applications with a single line of dialplan code. We’ll use it in our sample IVR later in this chapter.

While you’ll find CURL\(\) itself to be quite simple to use, the creation of the web application will require experience with web development.

### func_odbc

Using func\_odbc, it is possible to develop extremely complex applications in Asterisk using nothing more than dialplan code and database lookups. If you are not a strong programmer but are very adept with Asterisk dialplans and databases, you’ll love func\_odbc just as much as we do. Check it out in [Chapter 15](glava-15.md).

### AGI

The Asterisk Gateway Interface is such an important part of integrating external applications with Asterisk that we gave it its own chapter. You can find more information in [Chapter 18](glava-18.md).

### AMI

The Asterisk Manager Interface is a socket interface that you can use to get configuration and status information, request actions to be performed, and be notified about things happening to calls. We’ve written an entire chapter on AMI as well. You can find more information in [Chapter 17](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch17.html%22%20/l%20%22asterisk-AMI).

### ARI

Asterisk’s REST interface builds on knowledge gained over the years about how to integrate Asterisk with current-generation web-centric applications. It is so important, that yes, once again, there is a complete chapter dedicated to it. If you’re looking to build complex IVRs using Asterisk, take a closer look at ARI in [Chapter 19](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch19.html%22%20/l%20%22asteriskRESTch).

## A Simple IVR Using CURL()

Before we go running off writing an external program to handle something, we always give some careful thought about whether there’s a way to do the work in the dialplan. One powerful way that Asterisk can interact with external data is through a URL, which the GNU/Linux program cURL does very well. In Asterisk, CURL() is a dialplan function.

We’re going to use CURL() as an example of what an extremely simple IVR can look like. We’re going to request our external IP address from [https://ipinfo.io/ip](https://ipinfo.io/ip).<sup><a href="#sn2">2</a></sup>

**Note**

In reality, most IVR applications are going to be far more complex. Even most uses of CURL() will tend to be complex, since a URI can return a massive and highly variable amount of data, the vast majority of which will be incomprehensible to Asterisk. The point being that an IVR is not just about the dialplan; it is also very much about the external applications that are triggered by the dialplan, which are doing the real work of the IVR.

The CURL() module was installed during our installation process several chapters ago.

### The Dialplan

The dialplan for our example IVR is very simple. The CURL() function will retrieve our IP address from [https://ipinfo.io/ip](https://ipinfo.io/ip), and then SayAlpha() will speak the results to the caller:

```
exten => *764,1,Verbose(2, Run CURL to get IP address from whatismyip.org)
    same => n,Answer()
    same => n,Set(MyIPAddressIs=${CURL(https://ipinfo.io/ip)})
    same => n,SayAlpha(${MyIPAddressIs})
    same => n,Hangup()
```

The simplicity of this is impossibly cool. In a traditional IVR system, this sort of thing could take days to program, assuming it would be possible at all.

## A Prompt-Recording IVR Function

In [Chapter 14](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch14.html%22%20/l%20%22asterisk-AA), we created a simple bit of dialplan to record prompts. It was fairly limited in that it only recorded one filename, and thus for each prompt a separate extension was needed. Here, we expand upon that to create a complete menu for recording prompts. Since this is a complex bit of dialplan, but it’s not a subroutine or a local channel, we’re going to create a new section of dialplan for various features, and put things like this there:

```
;FEATURES
[prompts]
exten => s,1,Answer
exten => s,n,Set(step1count=0) ; Initialize counters

; If we get no response after 3 times, we stop asking
   same => n(beginning),GotoIf($[${step1count} > 2]?end)
   same => n,Read(which,prompt-instructions,3)
   same => n,Set(step1count=$[${step1count} + 1])

; All prompts must be 3 digits in length
   same => n,GotoIf($[${LEN(${which})} != 3]?beginning)
   same => n,Set(step1count=0) ; Successful response; reset counters
   same => n,Set(step2count=0)

   same => n(step2),Set(step2count=$[${step2count} + 1])
   same => n,GotoIf($[${step2count} > 2]?beginning) ; No response after 3 tries

; If the file doesn't exist, then don't ask whether to play it
   same => n,GotoIf($[${STAT(f,/var/lib/asterisk/sounds/${which}.wav)} = 0]?recordonly)
   same => n,Background(prompt-tolisten)

   same => n(recordonly),Background(prompt-torecord)
   same => n,WaitExten(10) ; Wait 10 seconds for a response
   same => n,Goto(step2)

   same => n(end),Playback(goodbye)
   same => n,Hangup()

exten => 1,1,Set(step2count=0)
   same => n,Background(/var/lib/asterisk/sounds/${which})
   same => n,Goto(s,step2)

exten => 2,1,Set(step2count=0)
   same => n,Playback(prompt-waitforbeep)
   same => n,Record(${CHANNEL(uniqueid)}.wav)

   same => n(listen),Playback(${CHANNEL(uniqueid)})
   same => n,Set(step3count=0)
   same => n,Read(saveornot,prompt-1tolisten-2tosave-3todiscard,1,,2,3)
   same => n,GotoIf($["${saveornot}" = "1"]?listen)
   same => n,GotoIf($["${saveornot}" = "2"]?saveit)
   same => n,GotoIf($["${saveornot}" = "3"]?tossit)
   same => n,Goto(listen)

   same => n(tossit),System(rm -f /var/lib/asterisk/sounds/${CHANNEL(uniqueid)}.wav)
   same => n,Goto(s,beginning)

   same => n(saveit),Noop('Set' app used to shorten example)
   same => n,Set(PromptToSave=/var/lib/asterisk/sounds/${CHANNEL(uniqueid)}.wav
   same => n,Set(WhereToSave=/var/lib/asterisk/sounds/${which}.wav
   same => n,System(mv -f ${PromptToSave} ${WhereToSave})
   same => n,Playback(prompt-saved)
   same => n,Goto(s,beginning)
```

In this system, the name of the prompt is no longer descriptive; instead, it is a number. This means that you can record a far greater variety of prompts using the same mechanism, but the trade-off is that your prompts will no longer have descriptive names.

If you want to test this, you’ll need to record the prompts this IVR function uses \(it’s kinda meta, but yup, our prompt creator needs prompts\).

Drop this into your dialplan:

```
exten => 510,1,GoSub(subRecordPrompt,${EXTEN},1(prompt-tolisten))     ; press 1
exten => 511,1,GoSub(subRecordPrompt,${EXTEN},1(prompt-torecord))     ; press 2
exten => 512,1,GoSub(subRecordPrompt,${EXTEN},1(prompt-instructions)) ;3-digit (000 to 999)
exten => 513,1,GoSub(subRecordPrompt,${EXTEN},1(prompt-waitforbeep))  ; wait for beep
exten => 514,1,GoSub(subRecordPrompt,${EXTEN},1(prompt-1tolisten-2tosave-3todiscard))
exten => 515,1,GoSub(subRecordPrompt,${EXTEN},1(prompt-saved))
```

Then phone them one by one and record as required.

Once you’ve recorded the prompts your prompt recorder needs, you should be able to test it out.

```
exten => *742,1,Noop(Prompts)
    same => n,Goto(prompts,s,1)
    same => n,Hangup()
```

From this point forward, you can record prompts using just a numeric identifier. You’ll need a way to keep track of what prompt says what, but from a recording perspective you shouldn’t need to write more dialplan every time you need a prompt.

## Speech Recognition and Text-to-Speech

Although traditionally and still in most cases today, an IVR system presents prerecorded prompts to the caller and accepts input by way of the dialpad, it is also possible to: a) generate prompts artificially, popularly known as text-to-speech; and b\) accept verbal inputs through a speech recognition engine.

While the concept of being able to have an intelligent conversation with a machine is something sci-fi authors have been promising us for many long years, the actual science of this remains complex and error-prone. Despite their amazing capabilities, computers are ill-suited to the task of appreciating the subtle nuances of human speech.

Having said that, it should be noted that companies such as Google have achieved amazing advances in both text-to-speech and speech recognition. There are now APIs available that can do a remarkable job of making sense out of what is being said to them. Google of course benefits from having a massive backend that can perform near-miraculous feats of processing; something your IVR might not be able to fully harness.

### Text-to-Speech

Text-to-speech \(also known as speech synthesis\) requires that a system be able to artificially construct speech from stored data. While it would be nice if we could simply assign a sound to a letter and have the computer produce each sound as it reads the letters, written language is often not phonetic, and seldom reflects the nuances of speech \(English is arguably one of the worst languages in this regard\).

There are now excellent APIs available from Google \(and others\), that will do a very good job of reading back what has been written. As of this writing, it’s still very obvious that it’s a computer speaking, but it is nevertheless possible to generate system prompts on the fly from text, rather than having to record all prompts in advance. The usefulness of this is difficult to evaluate, since humans are still not interested in talking to your machines; they phoned because they want to talk to you.

### Speech Recognition

Since we’ve managed to convince our computers to talk to us, we naturally want to be able to talk to them as well.<sup><a href="#sn3">3</a></sup>

Speech recognition was previously complex and expensive, but Google has recently released an API that allows the enormous power of their speech recognition capabilities to be available to external applications.

## Вывод

Asterisk является отличной платформой IVR. Вся эта книга, во многом, учит вас навыкам, которые могут быть применены для развития IVR. В то время как основные СМИ действительно уделяют внимание Asterisk только как “свободной УАТС”, реальность такова, что Asterisk является наиболее мощной, когда используется в качестве IVR. В любой солидной организации очень вероятно, что системные администраторы Linux используют Asterisk для решения телекоммуникационных проблем, которые ранее были либо неразрешимыми, либо невероятно дорогими для решения. Это скрытая революция, но не менее значимая из-за своей относительной неизвестности.

Если вы занимаетесь IVR-бизнесом, вам обязательно нужно познакомиться с Asterisk.

<ol>
  <li id="sn1">Especially if it’s something like Van Meggelen.</li>
  <li id="sn2">These free IP lookup websites seem to get bought out all the time, and turned into advertising gateways, so what might have worked at this writing may no longer work. What you need is a website that will return your IP address, and nothing else. Today, that seems to be <a href="https://ipinfo.io/ip">https://ipinfo.io/ip</a>. By the time you read this, it may be something else.</li>
  <li id="sn3">Actually, most of us talk to our computers, but this is seldom polite.</li>
</ol>

[Глава 15. Интеграция реляционной базы данных](glava-15.md) | [Содержание](SUMMARY.md) | [Глава 17. AMI и файлы вызовов](glava-17.md)
