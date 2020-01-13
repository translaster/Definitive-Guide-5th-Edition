# Глава 18. AGI

> _Кофеин. Шлюз к наркотикам._
>
> -- Эдди Веддер

Диалплан Asterisk превратился в простой, но мощный программный интерфейс для обработки вызовов. Однако многие люди, особенно с опытом программирования, предпочитают реализовывать обработку вызовов на традиционном языке программирования. Asterisk Gateway Interface \(AGI\) позволяет разрабатывать управление вызовами от первого лица на выбранном вами языке программирования.

## Быстрый старт

В этом разделе приведен краткий пример использования AGI.

Во-первых, давайте создадим скрипт, который мы собираемся запустить. Скрипты AGI обычно помещаются в _/var/lib/asterisk/agi-bin_.

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

Сохраните и перезагрузите свой диалплан и теперь когда вы звоните по номеру 237, то должны услышать, как Эллисон произносит “Hello World.”

## Варианты AGI

Существует несколько вариантов AGI, которые отличаются в первую очередь методом, используемым для связи с Asterisk. Полезно быть в курсе всех вариантов чтобы сделать лучший выбор, основанный на потребностях вашего приложения.

### Process-Based AGI

Process-based AGI \(AGI на основе процесса\) является простейшим вариантом AGI. Пример быстрого запуска в начале этой главы является примером сценария Process-based AGI. Скрипт вызывается с помощью приложения `AGI()` из диалплана Asterisk. Запускаемое приложение указывается в качестве первого аргумента функции `AGI()`. Если не указан полный путь, приложение должно находиться в каталоге _/var/lib/asterisk/agi-bin_. Аргументы, передаваемые приложению AGI, могут быть указаны в качестве дополнительных аргументов приложения `AGI()` в диалплане Asterisk. Синтаксис такой:

```text
AGI(command[,arg1[,arg2[,...]]])
```

---

**Подсказка**

Убедитесь, что приложение имеет соответствующие разрешения, чтобы пользователь Asterisk имел разрешения на его выполнение. В противном случае функция `AGI()` завершится ошибкой.

---

Once Asterisk executes your AGI application, communication between Asterisk and your application will take place over stdin and stdout. More details about this communication will be covered in [“AGI Communication Overview”](18.%20Asterisk%20Gateway%20Interface%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22AGI-communication). For more details about invoking AGI\(\) from the dialplan, check the documentation built into Asterisk:

*CLI&gt; core show application AGI

Pros of process-based AGI

It is the simplest form of AGI to implement.

Cons of process-based AGI

It is the least efficient form of AGI with regard to resource consumption. Systems with high load should consider FastAGI, discussed in [“FastAGI—AGI over TCP”](18.%20Asterisk%20Gateway%20Interface%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22FastAGI), instead.

#### EAGI

EAGI \(Enhanced AGI\) is a slight variant on AGI\(\). It is invoked in the Asterisk dialplan as EAGI\(\). The difference is that in addition to the communication on stdin and stdout, Asterisk also provides a unidirectional stream of audio coming from the channel on file descriptor 3. For more details on how to invoke EAGI\(\) from the Asterisk dialplan, check the documentation built into Asterisk:

\*CLI&gt; core show application EAGI

Pros of Enhanced AGI

It has the simplicity of process-based AGI, with the addition of a simple read-only stream of the channel’s audio. This is the only variant that offers this feature.

Cons of Enhanced AGI

Since a new process must be spawned to run your application for every call, it has the same efficiency concerns as regular, process-based AGI.

**Tip**

For an alternative way of gaining access to the audio outside Asterisk, consider using [JACK](http://jackaudio.org/). Asterisk has a module for JACK integration, called app\_jack. It provides the JACK\(\) dialplan application and the JACK\_HOOK\(\) dialplan function.

### FastAGI—AGI over TCP

FastAGI is the term used for AGI call control over a TCP connection. With process-based AGI, an instance of an AGI application is executed on the system for every call, and communication with that application is done over stdin and stdout. With FastAGI, a TCP connection is made to a FastAGI server. Call control is done using the same AGI protocol, but the communication is over the TCP connection and does not require a new process to be started for every call. The AGI protocol is discussed in more detail in [“AGI Communication Overview”](18.%20Asterisk%20Gateway%20Interface%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22AGI-communication). Using FastAGI is much more scalable than process-based AGI, though it is also more complex to implement.

To use FastAGI you invoke the AGI\(\) application in the Asterisk dialplan, but instead of providing the name of the application to execute, you provide an agi:// URL. For example:

exten =&gt; 238,1,AGI\(agi://127.0.0.1\)

The default port number for a FastAGI connection is 4573. A different port number can be appended to the URL after a colon. For example:

exten =&gt; 238,1,AGI\(agi://127.0.0.1:4574\)

Just as with process-based AGI, arguments can be passed to a FastAGI application. To do so, add them as additional arguments to the AGI\(\) application, delimited by commas:

exten =&gt; 238,1,AGI\(agi://192.168.1.199,arg1,arg2,arg3\)

FastAGI also supports the usage of DNS SRV records, if you provide a URL in the form of hagi://. By using SRV records, your DNS servers can return multiple hosts that Asterisk can attempt to connect to. This can be used for high availability and load balancing. In the following example, to find a FastAGI server to connect to, Asterisk will perform a DNS lookup for \_agi.\_tcp.shifteight.org:

exten =&gt; 238,1,AGI\(hagi://shifteight.org\)

In this example, the DNS servers for the shifteight.org domain would need at least one SRV record configured for \_agi.\_tcp.shifteight.org.

Pros of FastAGI

It’s more efficient than process-based AGI. Rather than spawning a process per call, a FastAGI server can be built to handle many calls.

DNS can be used to achieve high availability and load balancing among FastAGI servers to further enhance scalability.

Cons of FastAGI

It is more complex to implement a FastAGI server than to implement a process-based AGI application.

### Async AGI—AMI-Controlled AGI

Async AGI allows an application that uses the Asterisk Manager Interface \(AMI\) to asynchronously queue up AGI commands to be executed on a channel. This can be especially useful if you are already making extensive use of the AMI and would like to enhance your application to handle call control, rather than writing a detailed Asterisk dialplan or developing a separate FastAGI server.

**Tip**

More information on the Asterisk Manager Interface can be found in [Chapter 17](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch17.html%22%20/l%20%22asterisk-AMI).

Async AGI is invoked by the AGI\(\) application in the Asterisk dialplan. The argument to AGI\(\) should be agi:async, as shown in the following example:

exten =&gt; 239,AGI\(agi:async\)

Additional information on how to use async AGI over the AMI can be found in the next section.

Pros of async AGI

An existing AMI application can be used to control calls using AGI commands.

Cons of async AGI

It is the most complex way to implement AGI.

**Setting Up /etc/asterisk/manager.conf for Async AGI**

To make use of async AGI, an AMI account must have the agi permission for both read and write. For example, the following user defined in manager.conf would be able to both a\) execute AGI manager actions, and b\) receive AGI manager events:

; Define a user called 'hello', with a password of 'world'.

; Give this user read/write permissions for AGI.

;

\[hello\]

secret = world

read = agi

write = agi

## AGI Communication Overview

The preceding section discussed the variations of AGI that can be used. This section goes into more detail about how your custom AGI application communicates with Asterisk once AGI\(\) has been invoked.

### Setting Up an AGI Session

Once AGI\(\) or EAGI\(\) has been invoked from the Asterisk dialplan, some information is passed to the AGI application to set up the AGI session. This section discusses what steps are taken at the beginning of an AGI session for the different variants of AGI.

#### Process-based AGI/FastAGI

For a process-based AGI application or a connection to a FastAGI server, the variables listed in [Table 18-1](18.%20Asterisk%20Gateway%20Interface%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22AGI-variables) will be the first pieces of information sent from Asterisk to your application. Each variable will be on its own line, in the form:

agi\_variable: value

Table 18-1. AGI environment variables

| Variable | Value/example | Description |
| :--- | :--- | :--- |
| agi\_request | hello-world.sh | The first argument that was passed to the AGI\(\) or EAGI\(\) application. For process-based AGI, this is the name of the AGI application that has been executed. For FastAGI, this would be the URL that was used to reach the FastAGI server. |
| agi\_channel | SIP/0004F2060EB4-00000009 | The name of the channel that has executed the AGI\(\) or EAGI\(\) application. |
| agi\_language | en | The language set on agi\_channel. |
| agi\_type | SIP | The channel type for agi\_channel. |
| agi\_uniqueid | 1284382003.9 | The uniqueid of agi\_channel. |
| agi\_version | 1.8.0-beta4 | The Asterisk version in use. |
| agi\_callerid | 12565551212 | The full caller ID string that is set on agi\_channel. |
| agi\_calleridname | Russell Bryant | The caller ID name that is set on agi\_channel. |
| agi\_callingpres | 0 | The caller presentation associated with the caller ID set on agi\_channel. For more information, see the output of core show function CALLERPRES at the Asterisk CLI. |
| agi\_callingani2 | 0 | The caller ANI2 associated with agi\_channel. |
| agi\_callington | 0 | The caller ID TON \(Type of Number\) associated with agi\_channel. |
| agi\_callingtns | 0 | The dialed number TNS \(Transit Network Select\) associated with agi\_channel. |
| agi\_dnid | 7010 | The dialed number associated with agi\_channel. |
| agi\_rdnis | unknown | The redirecting number associated with agi\_channel. |
| agi\_context | phones | The context of the dialplan that agi\_channel was in when it executed the AGI\(\) or EAGI\(\) application. |
| agi\_extension | 500 | The extension in the dialplan that agi\_channel was executing when it ran the AGI\(\) or EAGI\(\) application. |
| agi\_priority | 1 | The priority of agi\_extension in agi\_context that executed AGI\(\) or EAGI\(\). |
| agi\_enhanced | 0.0 | An indication of whether AGI\(\) or EAGI\(\) was used from the dialplan. 0.0 indicates that AGI\(\) was used. 1.0 indicates that EAGI\(\) was used. |
| agi\_accountcode | myaccount | The accountcode associated with agi\_channel. |
| agi\_threadid | 140071216785168 | The threadid of the thread in Asterisk that is running the AGI\(\) or EAGI\(\) application. This may be useful for associating logs generated by the AGI application with logs generated by Asterisk, since the Asterisk logs contain thread IDs. |
| agi\_arg\_&lt;argument number&gt; | my argument | These variables provide the contents of the additional arguments provided to the AGI\(\) or EAGI\(\) application. |

For an example of the variables that might be sent to an AGI application, see the AGI communication debug output in [“Quick Start”](18.%20Asterisk%20Gateway%20Interface%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22AGI-quickstart). The end of the list of variables will be indicated by a blank line. The code handles these variables by reading lines of input in a loop until a blank line is received. At that point, the application continues and begins executing AGI commands.

#### Async AGI

When you use async AGI, Asterisk will send out a manager event called AsyncAGI to initiate the async AGI session. This event will allow applications listening to manager events to take over control of the call via the AGI manager action. Here is an example manager event sent out by Asterisk:

Event: AsyncAGI

Privilege: agi,all

SubEvent: Start

Channel: SIP/0000FFFF0001-00000000

Env: agi\_request%3A%20async%0Aagi\_channel%3A%20SIP%2F0000FFFF0001-00000000%0A \

 agi\_language%3A%20en%0Aagi\_type%3A%20SIP%0A \

 agi\_uniqueid%3A%201285219743.0%0A \

 agi\_version%3A%201.8.0-beta5%0Aagi\_callerid%3A%2012565551111%0A \

 agi\_calleridname%3A%20Julie%20Bryant%0Aagi\_callingpres%3A%200%0A \

 agi\_callingani2%3A%200%0Aagi\_callington%3A%200%0Aagi\_callingtns%3A%200%0A \

 agi\_dnid%3A%20111%0Aagi\_rdnis%3A%20unknown%0Aagi\_context%3A%20LocalSets%0A \

 agi\_extension%3A%20111%0Aagi\_priority%3A%201%0Aagi\_enhanced%3A%200.0%0A \

 agi\_accountcode%3A%20%0Aagi\_threadid%3A%20-1339524208%0A%0A

**Note**

The value of the Env header in this AsyncAGI manager event is all on one line. The long value of the Env header has been URI encoded.

### Commands and Responses

Once an AGI session has been set up, Asterisk begins performing call processing in response to commands sent from the AGI application. As soon as an AGI command has been issued to Asterisk, no further commands will be processed on that channel until the current command has been completed. When it finishes processing a command, Asterisk will respond with the result.

**Note**

The AGI processes commands in a serial manner. Once a command has been executed, no further commands can be executed until Asterisk has returned a response. Some commands can take a very long time to execute. For example, the EXEC AGI command executes an Asterisk application. If the command is EXEC Dial, AGI communication is blocked until the call is done. If your AGI application needs to interact further with Asterisk at this point, it can do so using the AMI, which is covered in [Chapter 17](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch17.html%22%20/l%20%22asterisk-AMI).

You can retrieve a full list of available AGI commands from the Asterisk console by running the command agi show commands. These commands are described in [Table 18-2](18.%20Asterisk%20Gateway%20Interface%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22AGI-commands-table). To get more detailed information on a specific AGI command, including syntax information for any arguments that a command expects, use agi show commands topic COMMAND. For example, to see the built-in documentation for the ANSWER AGI command, you would use agi show commands topic ANSWER.

Table 18-2. AGI commands

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

AGI commands are sent to Asterisk on a single line. The line must end with a single newline character. Once a command has been sent to Asterisk, no further commands will be processed until the last command has finished and a response has been sent back to the AGI application. Here is an example response to an AGI command:

200 result=0

**Tip**

The Asterisk console allows debugging the communications with an AGI application. To enable AGI communication debugging, run the agi set debug on command. To turn debugging off, use agi set debug off. While this debugging mode is on, all communication to and from an AGI application will be printed out to the Asterisk console. An example of this output can be found in [“Quick Start”](18.%20Asterisk%20Gateway%20Interface%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22AGI-quickstart).

#### Async AGI

When you’re using async AGI, you issue commands by using the AGI manager action. To see the built-in documentation for the AGI manager action, run manager show command AGI at the Asterisk CLI. A demonstration will help clarify how AGI commands are executed using the async AGI method. First, an extension is created in the dialplan that runs an async AGI session on a channel:

exten =&gt; 240,AGI\(agi:async\)

When the AGI dialplan application is executed, a manager event called AsyncAGI will be sent out with all the AGI environment variables. Details about this event are in [“Async AGI”](18.%20Asterisk%20Gateway%20Interface%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22AGI_id268960). After this, AGI manager actions can start to take place via AMI.

The following shows an example manager-action execution and the manager events that are emitted during async AGI processing. After the initial execution of the AGI manager action, there is an immediate response to indicate that the command has been queued up for execution. Later, there is a manager event that indicates that the queued command has been executed. The CommandID header can be used to associate the initial request with the event that indicates that the command has been executed:

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

The following output is what was seen on the Asterisk console during this async AGI session:

 -- Executing \[7011@phones:1\] AGI\("SIP/0004F2060EB4-00000013",

 "agi:async"\) in new stack

 agi:async: Puppies like cotton candy.

 == Spawn extension \(phones, 7011, 1\)

exited non-zero on 'SIP/0004F2060EB4-00000013'

### Ending an AGI Session

An AGI session ends when your AGI application is ready for it to end. The details about how this happens depend on whether your application is using process-based AGI, FastAGI, or async AGI.

#### Process-based AGI/FastAGI

Your AGI application may exit or close its connection at any time. As long as the channel has not hung up before your application ends, dialplan execution will continue.

If channel hangup occurs while your AGI session is still active, Asterisk will provide notification that this has occurred so that your application can adjust its operation as appropriate.

If a channel hangs up while your AGI application is still executing, a couple of things will happen. If an AGI command is in the middle of executing, you may receive a result code of -1. You should not depend on this, though, since not all AGI commands require channel interaction. If the command being executed does not require channel interaction, the result will not reflect the hangup.

The next thing that happens after a channel hangs up is that an explicit notification of the hangup is sent to your application. For process-based AGI, the signal SIGHUP will be sent to the process to notify it of the hangup. For a FastAGI connection, Asterisk will send a line containing the word HANGUP.

If you would like to disable having Asterisk send the SIGHUP signal to your process-based AGI application or the HANGUP string to your FastAGI server, you can do so by setting the AGISIGHUP channel variable, as demonstrated in this short example:

; no SIGHUP \(AGI\) or HANGUP \(FastAGI\)

exten =&gt; 237,1,Set\(AGISIGHUP=no\)

 same =&gt; n,AGI\(hello-world.sh\)

Once the hangup has happened, the only AGI commands that may be used are those that do not require channel interaction. The documentation for the AGI commands built into Asterisk includes an indication of whether or not each command can be used once the channel has been hung up.

#### Async AGI

When you’re using async AGI, the manager interface provides mechanisms to notify you about channel hangups. When you would like to end an async AGI session for a channel, you must execute the ASYNCAGI BREAK command. When the async AGI session ends, Asterisk will send an AsyncAGI manager event with a SubEvent of End. The following is an example of ending an async AGI session:

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

At this point, the channel returns to the next step in the Asterisk dialplan \(assuming it has not yet been hung up\).

## Example: Account Database Access

[Example 18-1](18.%20Asterisk%20Gateway%20Interface%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22agiexample1) is an example of an AGI script. To run this script you would first place it in the /var/lib/asterisk/agi-bin directory. Then you would execute it from the Asterisk dialplan like this:

exten =&gt; 241,1,AGI\(account-lookup.py\)

 same =&gt; n,Hangup\(\)

This example is written in Python and is very sparsely documented for brevity. It demonstrates how an AGI script interfaces with Asterisk using stdin and stdout.

The script prompts a user to enter an account number, and then plays back a value associated with that number. In the interest of brevity, we have hardcoded a few fake accounts into the script—this would obviously be something normally handled by a database connection.

The script is intentionally terse, since we are interested in briefly showing some AGI functions without filling this book with pages of code.

**Example 18-1. account-lookup.py**

\#!/usr/bin/env python

\# An example for AGI \(Asterisk Gateway Interface\).

import sys

def agi\_command\(cmd\):

 '''Write out the command and return the response'''

 print cmd

 sys.stdout.flush\(\) \#clear the buffer

 return sys.stdin.readline\(\).strip\(\) \# strip whitespace

asterisk\_env = {} \# read AGI env vars from Asterisk

while True:

 line = sys.stdin.readline\(\).strip\(\)

 if not len\(line\):

 break

 var\_name, var\_value = line.split\(':', 1\)

 asterisk\_env\[var\_name\] = var\_value

\# Fake "database" of accounts.

ACCOUNTS = {

 '12345678': {'balance': '50'},

 '11223344': {'balance': '10'},

 '87654321': {'balance': '100'},

}

response = agi\_command\('ANSWER'\)

\# three arguments: prompt, timeout, maxlength

response = agi\_command\('GET DATA enter\_account 3000 8'\)

if 'timeout' in response:

 response = agi\_command\('STREAM FILE goodbye ""'\)

 sys.exit\(0\)

\# The response will look like: 200 result=&lt;digits&gt;

\# Split on '=', we want index 1

account = response.split\('=', 1\)\[1\]

if account == '-1': \# digits if error

 response = agi\_command\('STREAM FILE astcc-account-number-invalid ""'\)

 response = agi\_command\('HANGUP'\)

 sys.exit\(0\)

if account not in ACCOUNTS: \# invalid

 response = agi\_command\('STREAM FILE astcc-account-number-invalid ""'\)

 sys.exit\(0\)

balance = ACCOUNTS\[account\]\['balance'\]

response = agi\_command\('STREAM FILE account-balance-is ""'\)

response = agi\_command\('SAY NUMBER %s ""' % \(balance\)\)

sys.exit\(0\)

## Development Frameworks

There have been a number of efforts to create frameworks or libraries that make AGI programming easier. You will notice that several of these frameworks also appeared in [Chapter 17](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch17.html%22%20/l%20%22asterisk-AMI). Just as with AMI, when evaluating a framework, we recommend you find one that meets the following criteria:

Maturity

Has this project been around for a few years? A mature project is far less likely to have serious bugs in it.

Maintenance

Check the age of the latest update. If the project hasn’t been updated in a few years, there’s a strong possibility it has been abandoned. It might still be usable, but you’ll be on your own. Similarly, what does the bug tracker look like? Are there a lot of important bugs being ignored? \(Be discerning here, since often the realities of maintaining a free project require disciplined triage—not everybody’s features are going to get added.\)

Quality of the code

Is this a well-written framework? If it was not engineered well, you should be aware of that when deciding whether to trust your project to it.

Community

Is there an active community of developers using this project? It’s likely you’ll need help; will it be available when you need it?

Documentation

The code should be well commented, but ideally, a wiki or other official documentation to support the library is essential.

The frameworks listed in [Table 18-3](18.%20Asterisk%20Gateway%20Interface%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22AGI-frameworks) met all or most of the preceding criteria at this writing. If you do not see a library listed here for your preferred programming language, it might be out there somewhere, but simply didn’t make our list.

Table 18-3. AGI development frameworks

| Framework | Language |
| :--- | :--- |
| Adhearsion | Ruby |
| Asterisk-Java | Java |
| AsterNET | .NET |
| ding-dong | Node.js |
| PAGI | PHP |
| Panoramisk | Python |
| StarPy | Python + Twisted |

## Conclusion

AGI provides a powerful interface to Asterisk that allows you to implement first-party call control in the programming language of your choice. You can take multiple approaches to implementing an AGI application. Some approaches can provide better performance, but at the cost of more complexity. AGI provides a programming environment that may make it easier to integrate Asterisk with other systems, or just provide a more comfortable call-control programming environment for the experienced programmer. In many cases, the use of a prebuilt framework will be the best approach, especially when evaluating or prototyping a complex project. For the ultimate performance, we still recommend you consider writing as much of your application as you can using the Asterisk dialplan.

[Глава 17. AMI и файлы вызовов](glava-17.md) | [Содержание](SUMMARY.md) | [Глава 19. Asterisk REST Interface](glava-19.md)