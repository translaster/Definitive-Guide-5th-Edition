# Глава 12. Автоматические очереди распределения вызовов

## Глава 12. Очереди автоматического распределения вызовов

>Англичанин, даже если он один, формирует упорядоченную очередь из одного человека.
>
>Джордж Майкс

Автоматическое распределение вызовов (ACD), или организация очереди вызовов, позволяет УАТС ставить в очередь входящие вызовы от группы пользователей. Он объединяет несколько вызовов в шаблон удержания, присваивает каждому вызову ранг и определяет порядок, в котором этот вызов должен быть доставлен доступному оператору (как правило, сначала в порядке очереди). Когда агент становится доступным, вызывающий абонент с самым высоким рейтингом в очереди доставляется этому агенту, а все остальные повышаются в рейтинге.

Если вы когда-либо звонили в организацию и слышали, что «все наши представители заняты», вы испытали ACD. Преимущество ACD для вызывающих абонентов в том, что им не нужно продолжать набирать номер в попытке связаться с кем-то, а преимущества для организаций заключаются в том, что они могут лучше обслуживать своих клиентов и временно обрабатывать ситуации, когда есть больше звонящие, чем есть агенты.<sup><a href="#sn1">1</a></sup>

**Примечание**

Существует два типа колл-центров: входящие и исходящие. ACD относится к технологии, которая обрабатывает центры обработки входящих вызовов, тогда как термин _Dialer_ (или _Predictive Dialer_) относится к технологии, которая обрабатывает центры обработки исходящих вызовов. В этой книге мы прежде всего сосредоточимся на входящих звонках.

Мы все были разочарованы плохо спроектированными и управляемыми очередями: длительное удержание музыки с радио, которое не соответствует мелодии, ошеломляющее время ожидания и бессмысленные сообщения, которые каждые 20 секунд сообщают вам, насколько важен ваш звонок, несмотря на этот факт. что вы ждали 30 минут и слышали это сообщение так много раз, что можете процитировать его по памяти. С точки зрения обслуживания клиентов, дизайн очереди может быть одним из наиболее важных аспектов вашей телефонной системы. Как и в случае с автосекретарем, прежде всего следует помнить, что _ваши абоненты не заинтересованы в том, чтобы стоять в очереди_. Они позвонили, потому что _хотят с тобой поговорить_. Все ваши дизайнерские решения должны помнить об этом важном факте: люди хотят общаться с другими людьми, а не с вашей телефонной системой.<sup><a href="#sn2">2</a></sup>

Цель этой главы - научить вас, как создавать и проектировать очереди, которые доставляют абонентов по назначению максимально быстро и безболезненно.

**Примечание**

В этой главе мы можем переключаться между использованием терминов _queue members_ и _agents_. Так как мы не собираемся тратить много времени на модуль Asterisk с именем `chan_agent` (используя `AgentLogin()`), нам нужно прояснить, что в этой книге, когда мы используем термин _agent_, имеется в виду конечный пользователь - человек, а не канальная технология в Asterisk с именем `chan_agent`. Читайте дальше, и это должно иметь больше смысла.

## Создание простой очереди ACD

Для начала мы собираемся создать простую очередь ACD. Он будет принимать звонящих и пытаться доставить их участнику очереди.

**Примечание**

В Asterisk термин _member_ относится к каналу (обычно одноранговому узлу SIP), назначенному очереди, которую можно набрать, например, `SIP/0000FFFF0001`. _agent_ технически относится к каналу агента, также используемому для набора конечных точек. К сожалению, канал агента является устаревшей технологией в Asterisk, так как он ограничен в гибкости и может вызвать непредвиденные проблемы, которые трудно диагностировать и разрешать. Мы не будем покрывать использование `chan_agent`, поэтому помните, что мы обычно будем использовать термин _member_ (_Участник_) для обозначения телефонного устройства и _agent_ (_Агент_) для обозначения лица, которое выполняет вызов. Так как один не эффективен без другого, любой термин может относиться к обоим.

Мы создадим очередь(и) в файле _queues.conf_ и добавим в нее членов очереди через консоль Asterisk. В разделе “Queue Members” мы рассмотрим, как создать абонентскую группу, которая позволяет нам динамически добавлять и удалять участников очереди (а также приостанавливать и отменять их).

Первым шагом является создание пустого файла _agents.conf_ в вашем каталоге конфигурации _/etc/asterisk_. Мы не будем использовать или редактировать этот файл, но модуль `app_queue` ожидает его нахождения и не будет загружаться, если он не существует:

```text
$ cd /etc/asterisk
$ sudo -u asterisk touch agents.conf
```
Поскольку мы еще не сделали этого, мы также собираемся настроить базовую музыку в режиме ожидания, используя файл примера:
```text
$ sudo cp ~/src/asterisk-16.*/configs/samples/musiconhold.conf.sample /etc/asterisk/musiconhold.conf
```
```text
$ sudo chown asterisk:asterisk /etc/asterisk/musiconhold.conf
```
Затем вам нужно создать файл _queues.conf_, который мы не собираемся редактировать, потому что мы будем создавать наши очереди в базе данных (он просто должен быть там):
```text
$ sudo touch -u asterisk queues.conf
```
Далее мы собираемся создать несколько очередей в нашей базе данных:
```text
MySQL>; INSERT INTO `asterisk`.`queues`

(name,strategy,joinempty,leavewhenempty,ringinuse,autofill,musiconhold, \
monitor_format,monitor_type)

VALUES
'sales','rrmemory','unavailable,invalid,unknown','unavailable,invalid,unknown','no','yes',\
'default','wav','MixMonitor'),
('support','rrmemory','unavailable,invalid,unknown','unavailable,invalid,unknown','no',\
'yes','default','wav','MixMonitor') ;
```
Это даст нам две очереди, названные `sales` и `support`. Вы можете называть их как угодно, но мы будем использовать эти имена позже в этой книге, поэтому, если вы используете имена очередей, отличные от тех, которые мы рекомендовали здесь, запишите ваш выбор для дальнейшего использования.

Мы также определили параметры, изложенные в Таблице 12-1.

Таблица 12-1. Пример параметров очереди

| Parameter | Purpose |
| :--- | :--- |
| strategy=rrmemory | Use the round robin with memory strategy |
| joinempty=unavailable,invalid,unknown | Не присоединятся к очереди, когда нет доступных участников |
| leavewhenempty=unavailable,invalid,unknown | Покинуть очередь, когда нет доступных участников |
| ringinuse=no | Не звонить участникам, когда они уже используются (предотвращает многократные звонки участникам)|
| autofill=yes | Распределить всех ожидающих абонентов среди доступных участников |
| musiconhold=default | Воспроизведение музыки из класса `[default]` (см. <code>musiconhold.conf</code>) |

`Strategy`, которую мы будем использовать, это `rrmemory`, что означает «круговой прием с памятью». Стратегия `rrmemory` работает, чередуя агентов в очереди в последовательном порядке, отслеживая, какой агент получил последний вызов, и представляя следующий вызов следующему агенту. Когда он попадает к последнему агенту, он возвращается к началу (при входе агентов они добавляются в конец списка).

**Несколько примечаний по  Strategies**

`ringall`

Звонит всем доступным участникам (по умолчанию). Эта стратегия распределения на самом деле не считается ACD. В традиционных терминах телефонии это называется группой звонков (_ring group_).

`leastrecent`

Rings the interface that least recently received a call. In a queue where there are many calls of roughly the same duration, this can work. It doesn’t work as well if an agent has been on a call for an hour, and their colleagues all got their last call 30 minutes ago, because the agent who just finished the 60-minute call will get the next one.

`fewestcalls`

Rings the interface that has completed the fewest calls in this queue. This can be unfair if calls are not always of the same duration. An agent could handle three calls of 15 minutes each and her colleague had four 5-second calls; the agent who handled three calls will get the next one.

`random`

Rings a random interface. This actually can work very well and end up being very fair in terms of evenly distributing calls among agents.

`rrmemory`

Rings members in a round-robin fashion, remembering where it left off last for the next caller. This can also work out to be very fair, but not as much as random.

`linear`

Rings members in the order specified, always starting at the beginning of the list. This works if you have a team where there are some agents who are supposed to handle most calls, and other agents who should only get calls if the primary agents are busy.

`wrandom`

Rings a random member, but uses the members’ penalties as a weight. Worth considering in a larger queue with complex weighting among the agents.

We’ve set joinempty to no since it is generally bad form to put callers into a queue where there are no agents available to take their calls.

**Примечание**

You could set this to yes for ease of testing, but we would not recommend putting it into production unless you are using the queue for some function that is not about getting your callers to your agents. Nobody wants to wait in a line that is not going anywhere.

The leavewhenempty option is used to control whether callers should fall out of the Queue\(\) application and continue on in the dialplan if no members are available to take their calls. We’ve set this to yes because you won’t normally want callers waiting in a queue with no logged-in agents.

**Note**

From a business perspective, you should be telling your agents to clear all calls out of the queue before logging off for the day. If you find that there are a lot of calls queued up at the end of the day, you might want to consider extending someone’s shift to deal with them. Otherwise, they’ll just add to your stress when they call back the next day, in a worse mood.

You can use GotoIfTime\(\) near the end of the day to redirect callers to voicemail, or some other appropriate location in your dialplan, while your agents clear out any remaining calls in the queue.

We’ll want ringinuse to be no, which tells Asterisk not to ring members when their devices are already ringing. The purpose of setting ringinuse to no is to avoid multiple calls to the same member from one or more queues.

**Note**

It should be mentioned that joinempty and leavewhenempty are looking for either no members logged into the queue, or all members unavailable. Agents that are Ringing or InUse are not considered unavailable, so will not block callers from joining the queue or cause them to be kicked out when joinempty=no and/or leavewhenempty=yes.

The autofill option tells the queue to distribute all waiting callers to all available members immediately. Previous versions of Asterisk would only distribute one caller at a time, which meant that while Asterisk was signaling an agent, all other calls were held \(even if other agents were available\) until the first caller in line had been connected to an agent \(which obviously led to bottlenecks in older versions of Asterisk where large, busy queues were being used\). Unless you have a particular need for backward compatibility, this option should always be set to yes.

Verify that your /etc/asterisk/extconfig file contains the following lines:

queues =&gt; odbc,asterisk,queues

queue\_members =&gt; odbc,asterisk,queue\_members

Save and reload your queue configuration from the Asterisk CLI:

\*CLI&gt; queues reload

Verify that your queues were loaded into memory \(don’t forget to ensure an empty agents.conf file exists\):

localhost\*CLI&gt; queue show

support has 0 calls \(max unlimited\) in 'rrmemory' strategy

\(0s holdtime, 0s talktime\), W:0, C:0, A:0, SL:0.0% within 0s

 No Members

 No Callers

sales has 0 calls \(max unlimited\) in 'rrmemory' strategy

\(0s holdtime, 0s talktime\), W:0, C:0, A:0, SL:0.0% within 0s

 No Members

 No Callers

The output of queue show provides various pieces of information, including those parts detailed in [Table 12-2](Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition/12.%20Automatic%20Call%20Distribution%20Queues%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22queue_show_output_001).

Table 12-2. Output of queue show CLI command

| Field | Description |
| :--- | :--- |
| W: | Queue weight |
| C: | Number of calls presented to this queue |
| A: | Number of calls that have been answered by a member |
| SL: | Service level |

Now that you’ve created the queues, you need to configure your dialplan to allow calls to enter the queue.

Add the following dialplan logic to the extensions.conf file \(somewhere in the \[sets\] context\):

exten =&gt; 610,1,Noop\(\)

 same =&gt; n,Progress\(\)

 same =&gt; n,Queue\(sales\)

 same =&gt; n,Hangup\(\)

exten =&gt; 611,1,Noop\(\)

 same =&gt; n,Progress\(\)

 same =&gt; n,Queue\(support\)

 same =&gt; n,Hangup\(\)

Save the changes to your extensions.conf file, and reload the dialplan with the dialplan reload CLI command.

If you dial extension 610 or 611 at this point, you will end up with output like the following:

 == Setting global variable 'SIPDOMAIN' to '172.29.1.178'

 -- Executing \[610@sets:1\] NoOp\("PJSIP/SOFTPHONE\_A-00000004", ""\) in new stack

 -- Executing \[610@sets:2\] Progress\("PJSIP/SOFTPHONE\_A-00000004", ""\) in new stack

 -- Executing \[610@sets:3\] Queue\("PJSIP/SOFTPHONE\_A-00000004", "test"\) in new stack

 &gt; 0x7facc801ed60 -- Strict RTP learning after remote set to: 172.29.1.166:4022

 -- Started music on hold, class 'testmoh', on channel 'PJSIP/SOFTPHONE\_A-00000004'

 &gt; 0x7facc801ed60 -- Strict RTP switching to RTP target 172.29.1.166:4022 as source

 &gt; 0x7facc801ed60 -- Strict RTP learning complete - Locking on 172.29.1.166:4022

 -- Stopped music on hold on PJSIP/SOFTPHONE\_A-00000004

== Spawn extension \(sets, 610, 3\) exited non-zero on 'PJSIP/SOFTPHONE\_A-00000004'

Note that you won’t join the queue at this point because there are no agents in the queue to answer calls. We have joinempty=no and leavewhenempty=yes configured, so callers will not be placed into the queue. \(This would be a good opportunity to experiment with the joinempty and leavewhenempty options in queues.conf to better understand their impact on queues.\)

In the next section, we’ll demonstrate how to add members to your queue \(as well as other member interactions with the queue, such as pause/unpause\).

## Queue Members

Queues aren’t very useful without someone to answer the calls that come into them, so we need a method for allowing agents to be logged into the queues to answer calls. There are various ways of going about this, so we’ll show you how to add members to the queue both manually \(as an administrator, via either the CLI or hardcoded in the queue\_members table\) and dynamically \(as the agent, through an extension defined in the dialplan\). We’ll start with the Asterisk CLI method, which allows you to easily add members to the queue for testing, with minimal dialplan changes. Next we’ll show how you can define members in the queue\_members table. Finally, we’ll show you how to add dialplan logic that allows agents to log themselves into and out of the queues and to pause and unpause themselves in queues they are logged into \(this is likely the best method for production\).

### Controlling Queue Members via the CLI

We can add queue members to any available queue through the Asterisk CLI command queue add. The format of the queue add command is \(all on one line\):

\*CLI&gt; queue add member channel to queue \[\[\[penalty penalty\] as

membername\] state\_interface interface\]

The channel is the channel we want to add to the queue, such as SIP/0000FFFF0003, and the queue name will be something like support or sales—any queue name that exists in /etc/asterisk/queues.conf. For now we’ll ignore the penalty option, but we’ll discuss it in [“Advanced Queues”](Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition/12.%20Automatic%20Call%20Distribution%20Queues%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22ACD_id288932) \(penalty is used to control the rank of a member within a queue, which can be important for agents who are logged into multiple queues, or have differing skills\). We can define the membername to provide details to the queue-logging engine.

The state\_interface option informs the queue of the device state to be monitored for this agent. The details of how to work with device states are discussed in [Chapter 13](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch13.html%22%20/l%20%22asterisk-DeviceStates). Go ahead and work through that chapter, and then come back here and continue on. Don’t worry, we’ll wait.

Now that you’ve added callcounter=yes to sip.conf \(we’ll be using SIP channels throughout the rest of our examples\), let’s see how to add members to our queues from the Asterisk CLI.

Adding a queue member to the support queue can be done with the queue add member command:

\*CLI&gt; queue add member PJSIP/SOFTPHONE\_B to support

Added interface 'PJSIP/SOFTPHONE\_B' to queue 'support'

A query of the queue will verify that our new member has been added:

\*CLI&gt; queue show support

support has 0 calls \(max unlimited\) in 'rrmemory' strategy \(0s holdtime, 0s talktime\),

W:0, C:0, A:0, SL:0.0%, SL2:0.0% within 0s

 Members:

 PJSIP/SOFTPHONE\_B \(ringinuse disabled\) \(dynamic\) \(Not in use\) has taken no calls yet

 No Callers

To remove a queue member, you would use the queue remove member command:

\*CLI&gt; queue remove member PJSIP/SOFTPHONE\_B from support

Removed interface PJSIP/SOFTPHONE\_B from queue 'support'

Of course, you can use the queue show command again to verify that your member has been removed from the queue:

\*CLI&gt; queue show support

support has 0 calls \(max unlimited\) in 'rrmemory' strategy \(0s holdtime, 0s talktime\),

W:0, C:0, A:0, SL:0.0%, SL2:0.0% within 0s

 Members:

 PJSIP/SOFTPHONE\_B \(ringinuse disabled\) \(dynamic\) \(Not in use\) has taken no calls yet

 No Callers

We can also pause and unpause members in a queue from the Asterisk console, with the queue pause member and queue unpause member commands. They take a similar format to the previous commands we’ve been using:

\*CLI&gt; queue pause member PJSIP/SOFTPHONE\_B queue support reason Callbacks

paused interface 'PJSIP/SOFTPHONE\_B' in queue 'support' for reason 'Callbacks'

\*CLI&gt; queue show support

support has 0 calls \(max unlimited\) in 'rrmemory' strategy

\(0s holdtime, 0s talktime\), W:0, C:0, A:0, SL:0.0% within 0s

 Members:

 SIP/0000FFFF0001 \(dynamic\) \(paused\) \(Not in use\) has taken no calls yet

 No Callers

\*CLI&gt; queue show support

support has 0 calls \(max unlimited\) in 'rrmemory' strategy \(0s holdtime, 0s talktime\),

 W:0, C:0, A:0, SL:0.0%, SL2:0.0% within 0s

 Members:

 PJSIP/SOFTPHONE\_B \(ringinuse disabled\) \(dynamic\) \(paused:Callbacks\) \(Not in use\)

has taken no calls yet

 No Callers

By adding a reason for pausing the queue member, such as lunchtime, you ensure that your queue logs will contain some additional information that may be useful. Here’s how to unpause the member:

\*CLI&gt; queue unpause member PJSIP/SOFTPHONE\_B queue support reason FinishedCallBacks

unpaused interface 'PJSIP/SOFTPHONE\_B' in queue 'support' for reason 'FinishedCallbacks'

In a production environment, the CLI would not normally be the best way to control the state of agents in a queue. Instead, there are dialplan applications that allow agents to inform the queue as to their availability.

### Defining Queue Members in the queue\_members Table

If you define a queue member in the asterisk.queue\_members table of the database, that member will always be logged into the queue. This usually doesn’t work well if your members are human beings, since humans tend to get up and move about.

Within each queue definition, you simply define the members thus:

MySQL&gt; insert into \`asterisk\`.\`queue\_members\`

\(queue\_name,interface,penalty\)

VALUES

'hotline','PJSIP/SOME\_NON\_HUMAN','0'\);

In a typical queue \(one in which you have a group of people responsible for answering calls\), you will find that defining the members in the queue\_members table might not serve you well. Human agents usually need to be able to log in and out \(and not be automatically logged in whenever the queue is reloaded\). We do not recommend defining members in the queue\_members table unless they have some other purpose \(such as a bank of devices that answer calls, where you want to use the queue to load-balance calls to the device pool, or a ring group, where all phones ring for all calls all the time regardless of whether anyone is sitting near the phone\).

### Controlling Queue Members with Dialplan Logic

In a call center staffed by live agents, it is most common to have the agents themselves log in and log out at the start and end of their shifts \(or whenever they go for lunch, or to the bathroom, or are otherwise not available to the queue\).

To enable this, we will make use of the following dialplan applications:

* AddQueueMember\(\)
* RemoveQueueMember\(\)

While logged into a queue, it may be that an agent needs to put themself into a state where they are temporarily unavailable to take calls. The following applications will allow this:

* PauseQueueMember\(\)
* UnpauseQueueMember\(\)

The Add/Remove applications are used to log in and log out, and Pause/Unpause are used for short periods of agent unavailability. The difference is simply that Pause and Unpause set the member as unavailable/available without actually removing them from the queue. This is mostly useful for reporting purposes \(if a member is paused, the queue supervisor can see that they are logged into the queue, but simply not available to take calls at that moment\). If you’re not sure which one to use, we recommend that the agents use Add/Remove whenever they are not physically at their phone, and Pause/Unpause when they are at their desk, but temporarily not available.

If in doubt, it’s usually better to have your agents log out.

**Using Pause and Unpause**

In some environments, Pause and Unpause are used for all activities during the day that render an agent unavailable \(such as during the lunch hour and when performing work that is not queue-related\). In most call centers, however, if an agent is not beside the phone and ready to take a call at that moment, they should not be logged in at all, even if they are only going to be away from their desk for a few minutes \(such as for a bathroom break\).

Some supervisors like to use the Pause/Unpause settings as a sort of punch clock, so that they can track when their staff arrive for work and leave at the end of the day, and how long they spend at their desks and on breaks. This may not be a sound practice, since the purpose of these applications is to inform the queue as to agent availability, with activity tracking a secondary function.

An important thing to note here relates to the joinempty setting in the asterisk.queues table, which was discussed earlier. If an agent is paused, they are still logged into the queue. Let’s say it is near the end of the day, and one agent put themself into pause a few hours earlier to work on a project. All the other agents have logged out and gone home. A call comes in. The queue will note that an agent is logged into the queue, and will therefore queue the call, even though the reality is that there are no people actually staffing that queue at that time. This caller may end up holding in an unstaffed queue indefinitely.

In short, agents who are not sitting at their desks and planning to be available to take calls in the next few minutes should log out. Pause/Unpause should only be used for brief moments of unavailability \(if at all\). If you want to use your phone system as a punch clock, there are lots of great ways to do that using Asterisk, but the queue member applications are not the way we would recommend.

Let’s build some simple dialplan logic that will allow our agents to indicate their availability to the queue. We are going to use the CUT\(\) dialplan function to extract the name of our channel from our call to the system, so that the queue will know which channel to log into the queue.

We have built this dialplan to show a simple process for logging into and out of a queue, and changing the paused status of a member in a queue. We are doing this only for a single queue that we previously defined in the queues.conf file. The status channel variables that the AddQueueMember\(\), RemoveQueueMember\(\), PauseQueueMember\(\), and UnpauseQueueMember\(\) applications set might be used to Playback\(\) announcements to the queue members after they’ve performed certain functions to let them know whether they have successfully logged in/out or paused/unpaused:

exten =&gt; \*731,1,Page\(${PAGELIST},i,120\)

exten =&gt; \*732,1,Verbose\(2,Logging In Queue Member\)

 same =&gt; n,Set\(MemberChannel=${CHANNEL\(channeltype\)}/${CHANNEL\(endpoint\)}\)

 same =&gt; n,AddQueueMember\(support,${MemberChannel}\)

 same =&gt; n,Verbose\(1,${AQMSTATUS}\) ; ADDED, MEMBERALREADY, NOSUCHQUEUE

 same =&gt; n,Playback\(agent-loginok\)

 same =&gt; n,Hangup\(\)

exten =&gt; \*733,1,Verbose\(2,Logging Out Queue Member\)

 same =&gt; n,Set\(MemberChannel=${CHANNEL\(channeltype\)}/${CHANNEL\(endpoint\)}\)

 same =&gt; n,RemoveQueueMember\(support,${MemberChannel}\)

 same =&gt; n,Verbose\(1,${RQMSTATUS}\) ; REMOVED, NOTINQUEUE, NOSUCHQUEUE

 same =&gt; n,Playback\(agent-loggedoff\)

 same =&gt; n,Hangup\(\)

exten =&gt; \*734,1,Verbose\(2,Pause Queue Member\)

 same =&gt; n,Set\(MemberChannel=${CHANNEL\(channeltype\)}/${CHANNEL\(endpoint\)}\)

 same =&gt; n,PauseQueueMember\(support,${MemberChannel}\)

 same =&gt; n,Verbose\(1,${PQMSTATUS}\) ; PAUSED, NOTFOUND

 same =&gt; n,Playback\(dictate/paused\)

 same =&gt; n,Hangup\(\)

exten =&gt; \*735,1,Verbose\(2,Unpause Queue Member\)

 same =&gt; n,Set\(MemberChannel=${CHANNEL\(channeltype\)}/${CHANNEL\(endpoint\)}\)

 same =&gt; n,UnpauseQueueMember\(support,${MemberChannel}\)

 same =&gt; n,Verbose\(1,${UPQMSTATUS}\) ; UNPAUSED, NOTFOUND

 same =&gt; n,Playback\(agent-loginok\)

 same =&gt; n,Hangup\(\)

exten =&gt; \*98,1,NoOp\(Access voicemail retrieval.\)

### Automatically Logging Into and Out of Multiple Queues

It is quite common for an agent to be a member of more than one queue. Rather than having a separate extension for logging into each queue \(or demanding information from the agents about which queues they want to log into\), this code uses the Asterisk database \(astdb\) to store queue membership information for each agent, and then loops through each queue the agents are a member of, logging them into each one in turn.

In order for this code to work, an entry similar to the following will need to be added to the AstDB via the Asterisk CLI. For example, the following would store the member SOFTPHONE\_A as being in both the support and sales queues:[3](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch12.html%22%20/l%20%22idm46178405949096)

\*CLI&gt; database put queue\_agent SOFTPHONE\_A/available\_queues support^sales

You will need to do this once for each agent, regardless of how many queues they are members of.

If you then query the Asterisk database, you should get a result similar to the following:

pbx\*CLI&gt; database show queue\_agent

/queue\_agent/SOFTPHONE\_A/available\_queues : support^sales

The following dialplan code is an example of how to allow this queue member to be automatically added to both the support and sales queues. We’ve defined a subroutine that is used to set up three channel variables \(MemberChannel, MemberChanType, AvailableQueues\). These channel variables are then used by the login \(\*736\), logout \(\*737\), pause \(\*738\), and unpause \(\*739\) extensions. Each of the extensions uses the subSetupAvailableQueues subroutine to set these channel variables and to verify that the AstDB contains a list of one or more queues for the device the queue member is calling from.

Near the end of your extensions.conf file, where you’ve put your subroutines, add the following:

\[subSetupAvailableQueues\]

; This subroutine is used by the various login/logout/pausing/unpausing routines

; in our multiple queue login example.

;

exten =&gt; start,1,Verbose\(2,Checking for available queues\)

; Get the current channel's peer name

 same =&gt; n,Set\(MemberChannel=${CHANNEL\(endpoint\)}\)

; Get the current channel's technology type

 same =&gt; n,Set\(MemberChanType=${CHANNEL\(channeltype\)}\)

; Get the list of queues available for this agent

 same =&gt; n,Set\(AvailableQueues=${DB\(queue\_agent/${MemberChannel}/available\_queues\)}\)

; if there are no queues assigned to this agent we'll handle it in the

; no\_queues\_available extension

 same =&gt; n,GotoIf\($\[${ISNULL\(${AvailableQueues}\)}\]?no\_queues\_available,1\)

 same =&gt; n,Return\(\)

exten =&gt; no\_queues\_available,1,Verbose\(2,No queues available for agent ${MemberChannel}\)

; playback a message stating the channel has not yet been assigned

 same =&gt; n,Playback\(silence/1&channel&not-yet-assigned\)

 same =&gt; n,Hangup\(\)

Next, in your \[sets\] context, add the following:

; Logging into multiple queues via the AstDB system

exten =&gt; \*736,1,Verbose\(2,Logging into multiple queues per the database values\)

; get the available queues for this channel

 same =&gt; n,GoSub\(subSetupAvailableQueues,start,1\(\)\)

 same =&gt; n,Set\(QueueCounter=1\) ; setup a counter variable

; using CUT\(\), get the first listed queue returned from the AstDB

; Note that we've used '^' as our delimiter

 same =&gt; n,Set\(WorkingQueue=${CUT\(AvailableQueues,^,${QueueCounter}\)}\)

; While the WorkingQueue channel variable contains a value, loop

 same =&gt; n,While\($\[${EXISTS\(${WorkingQueue}\)}\]\)

; AddQueueMember\(queuename\[,interface\[,penalty\[,options\[,membername

; \[,stateinterface\]\]\]\]\]\)

; Add the channel to a queue, setting the interface for calling

; and the interface for monitoring of device state

; \*\*\* This should all be on a single line

 same =&gt; n,AddQueueMember\(

 ${WorkingQueue},${MemberChanType}/${MemberChannel},,,${MemberChanType}/${MemberChannel}\)

 same =&gt; n,Set\(QueueCounter=$\[${QueueCounter} + 1\]\) ; increase our counter

; get the next available queue; if it is null our loop will end

 same =&gt; n,Set\(WorkingQueue=${CUT\(AvailableQueues,^,${QueueCounter}\)}\)

 same =&gt; n,EndWhile\(\)

; let the agent know they were logged in okay

 same =&gt; n,Playback\(silence/1&agent-loginok\)

 same =&gt; n,Hangup\(\)

exten =&gt; no\_queues\_available,1,Verbose\(2,No queues available for ${MemberChannel}\)

 same =&gt; n,Playback\(silence/1&channel&not-yet-assigned\)

 same =&gt; n,Hangup\(\)

; Used for logging agents out of all configured queues per the AstDB

exten =&gt; \*737,1,Verbose\(2,Logging out of multiple queues\)

; Because we reused some code, we've placed the duplicate code into a subroutine

 same =&gt; n,GoSub\(subSetupAvailableQueues,start,1\(\)\)

 same =&gt; n,Set\(QueueCounter=1\)

 same =&gt; n,Set\(WorkingQueue=${CUT\(AvailableQueues,^,${QueueCounter}\)}\)

 same =&gt; n,While\($\[${EXISTS\(${WorkingQueue}\)}\]\)

 same =&gt; n,RemoveQueueMember\(${WorkingQueue},${MemberChanType}/${MemberChannel}\)

 same =&gt; n,Set\(QueueCounter=$\[${QueueCounter} + 1\]\)

 same =&gt; n,Set\(WorkingQueue=${CUT\(AvailableQueues,^,${QueueCounter}\)}\)

 same =&gt; n,EndWhile\(\)

 same =&gt; n,Playback\(silence/1&agent-loggedoff\)

 same =&gt; n,Hangup\(\)

; Used for pausing agents in all available queues

exten =&gt; \*738,1,Verbose\(2,Pausing member in all queues\)

 same =&gt; n,GoSub\(subSetupAvailableQueues,start,1\(\)\)

 ; if we don't define a queue, the member is paused in all queues

 same =&gt; n,PauseQueueMember\(,${MemberChanType}/${MemberChannel}\)

 same =&gt; n,GotoIf\($\[${PQMSTATUS} = PAUSED\]?agent\_paused,1:agent\_not\_found,1\)

exten =&gt; agent\_paused,1,Verbose\(2,Agent paused successfully\)

 same =&gt; n,Playback\(dictate/paused\)

 same =&gt; n,Hangup\(\)

; Used for unpausing agents in all available queues

exten =&gt; \*739,1,Verbose\(2,UnPausing member in all queues\)

 same =&gt; n,GoSub\(subSetupAvailableQueues,start,1\(\)\)

 ; if we don't define a queue, then the member is unpaused from all queues

 same =&gt; n,UnPauseQueueMember\(,${MemberChanType}/${MemberChannel}\)

 same =&gt; n,GotoIf\($\[${UPQMSTATUS} = UNPAUSED\]?agent\_unpaused,1:agent\_not\_found,1\)

exten =&gt; agent\_unpaused,1,Verbose\(2,Agent paused successfully\)

 same =&gt; n,Playback\(silence/1&available\)

; Used by both pausing and unpausing dialplan functionality

exten =&gt; agent\_not\_found,1,Verbose\(2,Agent was not found\)

 same =&gt; n,Playback\(silence/1&cannot-complete-as-dialed\)

You could further refine these login and logout routines to take into account that the AQMSTATUS and RQMSTATUS channel variables are set each time AddQueueMember\(\) and RemoveQueueMember\(\) are used. For example, you could set a flag that lets the queue member know they have not been added to a queue, or even add recordings or text-to-speech systems to play back the particular queue that is producing the problem. Or, if you’re monitoring this via the Asterisk Manager Interface, you could have a screen pop, or use JabberSend\(\) to inform the queue member via instant messaging, or…\(ain’t Asterisk fun?\).

## Advanced Queues

In this section we’ll take a look at some of the finer-grained queue controls, such as options for controlling announcements and when callers should be placed into \(or removed from\) the queue. We’ll also look at penalties and priorities, exploring how we can control the agents in our queue by giving preference to a pool of agents, and then increasing that pool dynamically based on the wait times in the queue. Finally, we’ll look at using local channels as queue members, which gives us the ability to perform dialplan tricks prior to connecting the caller to an agent.

### Priority Queue \(Queue Weighting\)

Sometimes you need to add people to a queue at a higher priority than that given to other callers. Perhaps the caller has already spent time waiting in a queue, and an agent has taken some information but realized the caller needed to be transferred to another queue. In this case, to minimize the caller’s overall wait time, it might be desirable to transfer the call to a priority queue that has a higher weight \(and thus a higher preference\), so it will be answered quickly.

Setting a higher priority on a queue is done with the weight option. If you have two queues with differing weights \(e.g., support and support-priority\), agents assigned to both queues will be passed calls from the higher-priority queue in preference to calls from the lower-priority queue. Those agents will not take any calls from the lower-priority queue until the higher-priority queue is cleared. \(Normally, there will be some agents who are assigned only to the lower-priority queue, to ensure that those calls are dealt with in a timely manner.\) For example, if we place queue member James Shaw into both the support and support-priority queues, callers in the support-priority queue will have a preferred standing with James over callers in the support queue.

Let’s take a look at how we could make this work. First, we need to create a new queue that’s similar to the support queue except for the weight option.

MySQL&gt; INSERT INTO \`asterisk\`.\`queues\`

\(name,strategy,joinempty,leavewhenempty,ringinuse,autofill,musiconhold,monitor\_format,

monitor\_type,weight\)

VALUES

\('support-priority','rrmemory','unavailable,invalid,unknown','unavailable,invalid,unknown',

'no','yes','default','wav','MixMonitor','10'\);

With our new queue configured, we can now create two extensions to transfer callers to. This can be done wherever you would normally place your dialplan logic to perform transfers. We’re going to use the LocalSets context, which we’ve previously enabled as the starting context for our devices:

exten =&gt; 611,1,Noop\(\)

 same =&gt; n,Progress\(\)

 same =&gt; n,Queue\(support\)

 same =&gt; n,Hangup\(\)

exten =&gt; 612,1,Noop\(\)

 same =&gt; n,Progress\(\)

 same =&gt; n,Queue\(support-priority\)

 same =&gt; n,Hangup\(\)

exten =&gt; \*724,1,Noop\(Page\)

The only other configuration left to do is to make sure some or all of your queue members are placed in both queues.

### Queue Member Priority

Within a queue, we can apply a penalty to members in order to lower their preference for being called when there are people waiting in a particular queue. For example, we may penalize queue members when we want them to be a member of a queue, but only receive calls when the queue gets full enough that all our preferred agents are unavailable. By defining different penalties for each member of the queue,[4](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch12.html%22%20/l%20%22idm46178405905624) we can help control the preference for where callers are delivered, but still ensure that other queue members will be available to answer calls if the preferred member is unavailable.

Penalties can also be defined using AddQueueMember\(\). We’ll modify our multiple queue login to provide the required penalties.

First, let’s update our AstDB to include penalties for a member:

\*CLI&gt; database put queue\_agent SOFTPHONE\_A/penalty 0^2

\*CLI&gt; database show queue agent

/queue\_agent/SOFTPHONE\_A/available\_queues : support^sales

/queue\_agent/SOFTPHONE\_A/penalty : 0^2

Next, a few tweaks to our dialplan.

The subroutine needs a new line \(some code has been removed for brevity, replaced with ;...\):

\[subSetupAvailableQueues\]

; ...

; Get the list of queues available for this agent

 same =&gt; n,Set\(AvailableQueues=${DB\(queue\_agent/${MemberChannel}/available\_queues\)}\)

 same =&gt; n,Set\(MemberPenalties=${DB\(queue\_agent/${MemberChannel}/penalty\)}\)

; if there are no queues assigned ...

The \[sets\] context requires a couple of new lines as well \(some code has been removed for brevity, replaced with ;...\). Only insert/change the code written in bold.

exten =&gt; \*736,1,Verbose\(2,Logging into multiple queues per the database values\)

; ...

 same =&gt; n,Set\(WorkingQueue=${CUT\(AvailableQueues,^,${QueueCounter}\)}\)

 same =&gt; n,Set\(WorkingPenalty=${CUT\(MemberPenalties,^,${QueueCounter}\)}\)

; While the WorkingQueue ...

; ...

 same =&gt; n,Set\(WorkingQueue=${CUT\(AvailableQueues,^,${QueueCounter}\)}\)

 same =&gt; n,Set\(WorkingPenalty=${CUT\(MemberPenalties,^,${QueueCounter}\)}\)

 same =&gt; n,EndWhile\(\)

; ...

These examples are probably not suitable for a production environment \(we’d use purpose-built MySQL tables for this sort of thing rather than AstDB\), but it gives you an idea of how the dialplan can be used to apply dynamic logic to more complex configuration scenarios.

### Changing Penalties Dynamically \(queuerules\)

Using the asterisk.queuerules table, it is possible to define rules that change the values of the QUEUE\_MIN\_PENALTY and QUEUE\_MAX\_PENALTY channel variables. The QUEUE\_MIN\_PENALTY and QUEUE\_MAX\_PENALTY channel variables are used to control which members of a queue are preferred for servicing callers. Let’s say we have a queue called support, and we have five queue members with various penalties ranging from 1 through 5. If, prior to a caller entering the queue, the QUEUE\_MIN\_PENALTY channel variable is set to a value of 2 and the QUEUE\_MAX\_PENALTY is set to a value of 4, only queue members whose penalties are set to values ranging from 2 through 4 will be considered available to answer that call:

 same =&gt; n,Set\(QUEUE\_MIN\_PENALTY=2\) ; set minimum member penalty

 same =&gt; n,Set\(QUEUE\_MAX\_PENALTY=4\) ; set maximum member penalty

 same =&gt; n,Queue\(support\) ; entering the queue with min and max

 ; member penalties to be used

What’s more, during the caller’s stay in the queue, we can dynamically change the values of QUEUE\_MIN\_PENALTY and QUEUE\_MAX\_PENALTY for that caller. This allows either more or a different set of queue members to be used, depending on how long the caller waits in the queue. For instance, in the previous example, we could modify the minimum penalty to 1 and the maximum penalty to 5 if the caller has to wait more than 60 seconds in the queue.

The sample file ~/src/asterisk-15.&lt;TAB&gt;/configs/samples/queuerules.conf.sample contains an excellent reference for how queue rules work.

The rules are defined using the asterisk.queuerules table. Multiple rules can be created in order to facilitate different penalty changes throughout the call. Let’s take a look at how we might choose to define a rule:

MySQL&gt; insert into \`asterisk\`.\`queue\_rules\`

\(rule\_name,time,min\_penalty,max\_penalty\)

VALUES

\('more\_members',60,5,1\);

**Note**

New rules will affect only new callers entering the queue, not existing callers already holding.

We’ve named the rule more\_members and defined the following values:

60

The number of seconds to wait before changing the penalty values.

5

The new QUEUE\_MAX\_PENALTY.

1

The new QUEUE\_MIN\_PENALTY.

We can now tell our queues to make use of it.

MySQL&gt; update \`asterisk\`.\`queues\`

set defaultrule='more\_members' where \`name\` in \('sales','support'\)

The queuerules.conf.sample file shows that these rules are quite flexible. If you want fine-grained control over call prioritization, some additional lab work may be worth your while.

### Announcement Control

Asterisk has the ability to play several announcements to callers waiting in the queue. For example, you might want to announce the caller’s position in the queue, announce the average wait time, or periodically thank your callers for waiting \(or whatever your audio files say\). It’s important to carefully tune the values that control when these announcements are played to the callers, because announcing their position, thanking them for waiting, and informing them of the average hold time too frequently is going to tend to annoy them, which is not the goal of these things.

**Playing Announcements Between Music on Hold Files**

Instead of handling the intricacies of announcements for each of your queues, you could alternatively \(or in conjunction\) utilize the announcement functionality defined in musiconhold.conf. Prior to playing a file for music, the announcement file will be played, and then played again between audio files. Let’s say you have a 5-minute loop of audio, but you want to play a “Thank you for waiting” message every 30 seconds. You could split the audio file into 30-second segments, set their filenames as starting with 00-, 01-, 02-, and so on \(to keep them playing in order\), and then define the announcement. The musiconhold.conf class might look something like this:

\[moh\_jazz\_queue\]

mode=files

sort=alpha

announcement=queue-thankyou

directory=moh\_jazz\_queue

There are several options in the queues table that you can use to fine-tune what and when announcements are played to your callers. The full list of queue options is available in the ~/src/asterisk-15.&lt;TAB&gt;/configs/samples/queues.conf.sample file. [Table 12-3](Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition/12.%20Automatic%20Call%20Distribution%20Queues%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22options_prompt_control_timing_id001) reviews a few of the more useful ones.

Table 12-3. Options related to prompt control timing within a queue

| Option | Available values | Description |
| :--- | :--- | :--- |
| announce-frequency | Value in seconds | Defines how often we should announce the caller’s position and/or estimated hold time in the queue. Set this value to zero to disable. |
| min-announce-frequency | Value in seconds | Indicates the minimum amount of time that must pass before we announce the caller’s position in the queue again. This is used when the caller’s position may change frequently, to prevent the caller hearing multiple updates in a short period of time. |
| periodic-announce-frequency | Value in seconds | Specifies how often to make periodic announcements to the caller. |
| random-periodic-announce | yes, no | If set to yes, will play the defined periodic announcements in a random order. See periodic-announce. |
| relative-periodic-announce | yes, no | If set to yes, the periodic-announce-frequency timer will start when the end of the file being played back is reached, instead of from the beginning. Defaults to no. |
| announce-holdtime | yes, no, once | Defines whether the estimated hold time should be played along with the periodic announcements. Can be set to yes, no, or only once. |
| announce-position | yes, no, limit, more | Defines whether the caller’s position in the queue should be announced to them. If set to no, the position will never be announced. If set to yes, the caller’s position will always be announced. If the value is set to limit, the caller will hear their position in the queue only if it is within the limit defined by announce-position-limit. If the value is set to more, the caller will hear their position only if it is beyond the number defined by announce-position-limit. |
| announce-position-limit | Number of zero or greater | Used if you’ve defined announce-position as either limit or more. |
| announce-round-seconds | Value in seconds | If this value is nonzero, the number of seconds is announced as well, and rounded to the value defined. |

[Table 12-4](Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition/12.%20Automatic%20Call%20Distribution%20Queues%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22options_controlling_playback_prompts) defines the files that will be used when announcements are played to the caller.

Table 12-4. Options for controlling the playback of prompts within a queue

| Option | Available values | Description |
| :--- | :--- | :--- |
| musicclass | Music class as defined by musiconhold.conf | Sets the music class to be used by a particular queue. You can also override this value with the CHANNEL\(musicclass\) channel variable. |
| queue-thankyou | Filename of prompt to play | If not defined, plays the default value \(“Thank you for your patience”\). If set to an empty value, prompt will not be played at all. |
| queue-youarenext | Filename of prompt to play | If not defined, plays the default value \(“You are now first in line”\). If set to an empty value, prompt will not be played at all. |
| queue-thereare | Filename of prompt to play | If not defined, plays the default value \(“There are”\). If set to an empty value, prompt will not be played at all. |
| queue-callswaiting | Filename of prompt to play | If not defined, plays the default value \(“calls waiting”\). If set to an empty value, prompt will not be played at all. |
| queue-holdtime | Filename of prompt to play | If not defined, plays the default value \(“The current estimated hold time is”\). If set to an empty value, prompt will not be played at all. |
| queue-minutes | Filename of prompt to play | If not defined, plays the default value \(“minutes”\). If set to an empty value, prompt will not be played at all. |
| queue-seconds | Filename of prompt to play | If not defined, plays the default value \(“seconds”\). If set to an empty value, prompt will not be played at all. |
| queue-reporthold | Filename of prompt to play | If not defined, plays the default value \(“hold time”\). If set to an empty value, prompt will not be played at all. |
| periodic-announce | A set of periodic announcements to be played, separated by commas | Prompts are played in the order they are defined. Defaults to queue-periodic-announce \(“All representatives are currently busy assisting other callers. Please wait for the next available representative”\). |

There’s a ton of flexibility possible when designing a caller’s experience while they’re waiting, but please don’t forget that your callers will never be happy to be waiting in the queue. Also, if you’ve found some half-decent hold music, and your callers are enjoying it, an interruption to play yet another message runs the risk of really setting their blood boiling. When they are finally answered, your poor agents will get the brunt of their anger, even though it is actually your fault.[5](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch12.html%22%20/l%20%22idm46178405806600)

So keep your on-hold tweaking simple. Callers know they’re waiting, and they aren’t going to be happy about it. Get them to an agent as quickly as possible, with the bare minimum amount of silliness while they’re holding, and don’t succumb to the temptation of making the queue more important to your callers than it actually is.

### Overflow

Unfortunately, your queue will not always get your callers to an agent in a timely manner. When various conditions cause the queue to reject incoming callers, we have an overflow situation. Overflowing out of the queue is done either with a timeout value or when no queue members are available \(as defined by joinempty or leavewhenempty\). In this section we’ll discuss how to control when overflow happens.

#### Controlling timeouts

The Queue\(\) application supports two kinds of timeout: one defines the maximum period of time a caller stays in the queue, and the other specifies how long to ring a device when attempting to connect a caller to a queue member. The two are unrelated but can affect each other. In this section we’ll be talking about the maximum period of time a caller stays in the Queue\(\) application before the call overflows to the next step in the dialplan, which could be something like VoiceMail\(\), or even another queue. Once the call has fallen out of the queue, it can go anywhere that a call could normally go when controlled by the dialplan.

The timeouts are specified in two locations. The timeout that indicates how long to ring queue members for is specified in the queues table. The absolute timeout \(how long the caller stays in the queue\) is controlled via the Queue\(\) application. To set a maximum amount of time for callers to stay in a queue, simply specify it after the queue name in the Queue\(\) application:

; Queue

exten =&gt; 610,1,Noop\(\)

 same =&gt; n,Progress\(\)

 same =&gt; n,Queue\(sales,120\)

 same =&gt; n,Voicemail\(${EXTEN}@queues,u\)

 same =&gt; n,Hangup\(\)

exten =&gt; 611,1,Noop\(\)

 same =&gt; n,Progress\(\)

 same =&gt; n,Queue\(support,120\)

 same =&gt; n,Voicemail\(${EXTEN}@queues,u\)

 same =&gt; n,Hangup\(\)

exten =&gt; 612,1,Noop\(\)

 same =&gt; n,Progress\(\)

 same =&gt; n,Queue\(support-priority,120\)

 same =&gt; n,Voicemail\(${EXTEN}@queues,u\)

 same =&gt; n,Hangup\(\)

Since we’re sending the calls to voicemail, we’ll need some mailboxes:

MySQL&gt; INSERT INTO \`asterisk\`.\`voicemail\`

\(context,mailbox,password,fullname,email\)

VALUES

\('queues','610','192837','Queue sales','name@shifteight.org'\),

\('queues','611','192837','Queue support','name@shifteight.org'\),

\('queues','612','192837','Queue support-priority','name@shifteight.org'\);

Of course, we could define a different destination, but the VoiceMail\(\) application is a common overflow destination for a queue. Obviously, sending callers to voicemail is not ideal \(they were hoping to speak to someone live\), so make sure someone checks it regularly and calls your customers back.

Now, let’s say we have set our absolute timeout to 10 seconds, our timeout value for ringing queue members to 5 seconds, and our retry timeout value to 4 seconds. In this scenario, we would ring the queue member for 5 seconds, then wait 4 seconds before attempting another queue member. That brings us up to 9 seconds of our absolute timeout of 10 seconds. At this point, should we ring the second queue member for 1 second and then exit the queue, or should we ring this member for the full 5 seconds before exiting?

We control which timeout value has priority with the timeoutpriority option in the queues table. The available values are app \(the default\) and conf. If we want the application timeout \(the absolute timeout\) to take priority, which would cause our caller to be kicked out after exactly 10 seconds \(even though it was just starting to ring an agent\), we should set the timeoutpriority value to app. If we want the configuration file timeout to take priority and finish ringing the queue member, which will cause the caller to stay in the queue a little longer, we should set timeoutpriority to conf. The default value is app \(which is the default behavior in previous versions of Asterisk\). Probably in most cases you’ll want to use conf \(especially if you want your caller experience to be as non-weird as possible\).

MySQL&gt; update \`asterisk\`.\`queues\` set timeoutpriority='conf'

 where name in \('sales','support','support-priority'\);

The goal is to get callers to agents, yes?

#### Controlling when to join and leave a queue

Asterisk provides two options that control when callers can join and are forced to leave queues, both based on the statuses of the queue members. The first option, joinempty, is used to control whether callers can enter a queue in the first place. The second option, leavewhenempty, is used to control events that will cause callers already in a queue to be removed from that queue \(i.e., if all of the queue members become unavailable\). Both options allow for a comma-separated list of values to control this behavior, as listed in [Table 12-5](Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition/12.%20Automatic%20Call%20Distribution%20Queues%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22options_joinempty).

Table 12-5. Options that can be set for joinempty or leavewhenempty

| Value | Description |
| :--- | :--- |
| paused | Members are considered unavailable if they are paused. |
| penalty | Members are considered unavailable if their penalties are less than QUEUE\_MAX\_PENALTY. |
| inuse | Members are considered unavailable if their device status is InUse. |
| ringing | Members are considered unavailable if their device status is Ringing. |
| unavailable | Applies primarily to agent channels; if the agent is not logged in but is a member of the queue, the channel is considered unavailable. |
| invalid | Members are considered unavailable if their device status is Invalid. This is typically an error condition. |
| unknown | Members are considered unavailable if device status is unknown. |
| wrapup | Members are considered unavailable if they are currently in the wrapup time after the completion of a call. |

For joinempty, prior to placing a caller into the queue, all the members are checked for availability using the factors you list as criteria. If all members are deemed to be unavailable, the caller will not be permitted to enter the queue, and dialplan execution will continue at the next priority.[6](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch12.html%22%20/l%20%22idm46178405761096) For the leavewhenempty option, the members’ statuses are checked periodically against the listed conditions; if it is determined that no members are available to take calls, the caller is removed from the queue, with dialplan execution continuing at the next priority.

An example use of joinempty could be:

joinempty=unavailable,invalid,unknown

With this configuration, prior to a caller entering the queue the statuses of all queue members will be checked, and the caller will not be permitted to enter the queue unless at least one queue member is found to have a status that is not unavailable, invalid, or unknown.

The leavewhenempty example could be something like:

leavewhenempty=unavailable,invalid,unknown

In this case, the queue members’ statuses will be checked periodically, and callers will be removed from the queue if no queue members can be found who do not have a status of unavailable, invalid, or unknown.

Previous versions of Asterisk used the values yes, no, strict, and loose as the available values to be assigned. The mapping of those values is shown in [Table 12-6](Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition/12.%20Automatic%20Call%20Distribution%20Queues%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22mapping_old_new_id001).

Table 12-6. Mapping between old and new values for controlling when callers join and leave queues

| Value | Mapping \(joinempty\) | Mapping \(leavewhenempty\) |
| :--- | :--- | :--- |
| yes | \(empty\) | penalty,paused,invalid |
| no | penalty,paused,invalid | \(empty\) |
| strict | penalty,paused,invalid,unavailable | penalty,paused,invalid,unavailable |
| loose | penalty,invalid | penalty,invalid |

### Using Local Channels

The use of local channels as queue members is a powerful way of executing dialplan code prior to dialing the actual agent’s device. When Queue\(\) decides to present a call to an agent, using local channels allows us to define custom channel variables, write to a logfile, set some limit on call length \(e.g., if it is a paid service\), send messages of all sorts all over the place, perform database transactions, and perform many of the other actions we might wish to do at that exact moment. Normally, we have no control over when the Queue\(\) application has decided to present a caller to a specific member, but with local channels, we get one final kick at the can, and can even return Congestion\(\), which will have the effect of returning the caller to the queue, since the queue will not consider this call to have been successfully delivered to an agent \(this can be very handy, since some external condition can be evaluated before the call is just fired off to an endpoint\).

When using local channels for queues, they are added just like any other channels, typically dynamically through the AddQueueMember\(\) dialplan application.

We’ll need to define the local channel where all the magic happens, and since local channels are typically used in a manner similar to subroutines, we like to name and locate them in the dialplan with the subroutines, with a context name starting with local \(akin to how subroutines start with sub\). If you’ve been building out your dialplan along with the book, you’ll notice you already have a local channel \[localDialDelay\]. Add this code somewhere in that part of the dialplan.

\[localMemberConnector\]

exten =&gt; \_\[A-Za-z0-9\].,1,Verbose\(2,Connect ${CALLERID\(all\)} to Agent at ${EXTEN}\)

 ; filter out any bad characters, allow alphanumeric chars and hyphen

 same =&gt; n,Set\(QueueMember=${FILTER\(A-Za-z0-9\-,${EXTEN}\)}\)

 ; assign the first field of QueueMember to Technology; hyphen as separator

 same =&gt; n,Set\(Technology=${CUT\(QueueMember,-,1\)}\)

 ; assign the second field of QueueMember to Device using the hyphen separator

 same =&gt; n,Set\(Device=${CUT\(QueueMember,-,2\)}\)

 ; dial the agent

 same =&gt; n,Dial\(${Technology}/${Device}\)

 same =&gt; n,Hangup\(\)

This code might not make total sense just yet, but what it’s doing is taking the ${EXTEN} \(which is a complex alphanumeric string at this point\), and slicing and dicing it to extract the actual channel to be called \(i.e., we pass as part of the local channel all the information needed to dial the actual channel\).

Let’s look at the AddQueueMember code and see if we can make more sense of this:

exten =&gt; \*740,1,Noop\(Logging in device ${CHANNEL\(endpoint\)} into the support queue\)

 same =&gt; n,Set\(MemberTech=${CHANNEL\(channeltype\)}\)

 same =&gt; n,Set\(MemberIdent=${CHANNEL\(endpoint\)}\)

 same =&gt; n,Set\(Interface=${MemberTech}/${MemberIdent}\)

 ;;; THE FOLLOWING SHOULD ALL BE ON ONE LINE

same =&gt; n,AddQueueMember\(support,Local/${MemberTech}-${MemberIdent}@localMemberConnector

,,,${IF\($\[${MemberTech} = PJSIP\]?${Interface}\)}\)

 same =&gt; n,Playback\(silence/1\)

 same =&gt; n,Playback\(${IF\($\[${AQMSTATUS} = ADDED\]?agent-loginok:agent-incorrect\)}\)

 same =&gt; n,Hangup\(\)

Once you’ve input all this and reloaded your dialplan, log into the queue by dialing \*740, and let’s see what we’ve got.

\*CLI&gt; queue show support

support has 0 calls \(max unlimited\) in 'rrmemory' strategy \(1s holdtime, 0s talktime\),

W:0, C:1, A:1, SL:0.0%, SL2:0.0% within 0s

 Members:

 PJSIP/SOFTPHONE\_A \(Local/PJSIP-SOFTPHONE\_A@localMemberConnector\)

\(ringinuse disabled\) \(dynamic\) \(Not in use\)

 No Callers

The member is now identified to the queue as a local channel named PJSIP-SOFTPHONE\_A in the \[localMemberConnector\] context. \(The PJSIP/SOFTPHONE\_A channel will be monitored for actual status of the endpoint.\) When Queue\(\) decides to send a call to the member, the call will end up in the \[localMemberConnector\] context, where the EXTEN \(PJSIP-SOFTPHONE\_A\) will be sliced and diced in order to yield our channel type and endpoint,[7](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch12.html%22%20/l%20%22idm46178405718440) which is what will actually be called.

At this point, the purpose of all this extra complexity is not immediately clear. So far we don’t get anything useful out of all this extra code.

So now that we can add devices to the queue using local channels, let’s look at how this might be useful.

Let’s say we have a customer who just can’t stand our best agent. They’re a good customer, so we don’t want to lose them, but it’s our best agent, so we’re not going to fire them.

To set this up, we’re going to assign a caller ID to SOFTPHONE\_B, so we have something to match against.

MySQL&gt; UPDATE \`asterisk\`.\`ps\_endpoints\` SET callerid='SOFTPHONE\_B &lt;103&gt;' \

WHERE id='SOFTPHONE\_B';

We’re going to build a little trick into our dialplan that will reject the call to the agent if the caller ID matches our sensitive customer.

\[localMemberConnector\]

exten =&gt; \_\[A-Za-z0-9\].,1,Verbose\(2,Connect ${CALLERID\(all\)} to Agent at ${EXTEN}\)

 same =&gt; n,Wait\(0.1\) ; Prevent loop from completely hogging CPU

 same =&gt; n,Set\(QueueMember=${FILTER\(A-Za-z0-9\-\_,${EXTEN}\)}\) ; allow alphanum, - , \_

 same =&gt; n,Set\(Technology=${CUT\(QueueMember,-,1\)}\) ; first field, hyphen is separator

 same =&gt; n,Set\(Device=${CUT\(QueueMember,-,2\)}\) ; second field, hypen separator

 ; is this our mismatched pair?

 same =&gt; n,DumpChan\(\)

 same =&gt; n,Noop\(${CALLERID\(all\)} : ${Device}\)

same=&gt;n,GotoIf\($\["${CALLERID\(num\)}"="103"&"${Device}"="SOFTPHONE\_A"\]?rejectcall:ringagent\)

 ; dial the agent

 same =&gt; n\(ringagent\),Dial\(${Technology}/${Device}\)

 same =&gt; n,Hangup\(\)

 ; send it back!

 same =&gt; n\(rejectcall\),Congestion\(\)

 same =&gt; n,Hangup\(\)

The passing back of Congestion\(\) will cause the caller to be returned to the queue \(while this is happening, the caller gets no indication that anything is amiss and keeps hearing music until their call is answered by a channel of some sort\).[8](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch12.html%22%20/l%20%22idm46178405708120) Ideally, your queue is programmed to try another agent; however, you need to keep in mind that if app\_queue determines that this member is still its first choice to present the call to, the call will simply be reconnected to the same agent \(and get congestion again, and thus potentially create a CPU-hogging logic loop\). To avoid this, you will need to ensure your queue is using a distribution strategy such as round\_robin, random, or any strategy that ensures the same member is not tried over and over. This is also why we toss a tiny little delay into our \[localMemberConnector\] context, so if a loop like this does happen, there’s at least a small throttle on it.

Let’s just sanity check our code. Set the caller ID number to something other than 103, and the call should go through.

MySQL&gt; UPDATE \`asterisk\`.\`ps\_endpoints\` SET callerid='SOFTPHONE\_B &lt;123&gt;' \

WHERE id='SOFTPHONE\_B';

The use of local channels for your member channels will not make queue design and debugging easier, but it does give you far more power over your queues than just using app\_queue on its own, so if you have a complex queue requirement, the use of local channels will give you a level of control you would not have otherwise.

## Queue Statistics: The queue\_log File

The queue\_log file \(commonly located in /var/log/asterisk\) contains cumulative event information for the queues defined in your system \(such as when a queue is reloaded, when queue members are added or removed, pause/unpause events, and so forth\) as well as some call details \(e.g., their status and which channels the callers were connected to\). The queue log is enabled by default, but it can be controlled via the /etc/asterisk/logger.conf file. There are three options related to the queue\_log file specifically:

queue\_log

Controls whether the queue log is enabled or not. Valid values are yes or no \(defaults to yes\).

queue\_log\_to\_file

Controls whether the queue log should be written to a file even when a real-time backend is present. Valid values are yes or no \(defaults to no\).

queue\_log\_name

Controls the name of the queue log. The default is queue\_log.

The queue log is a pipe-separated list of events. The fields in the queue\_log file are as follows:

* UNIX Epoch timestamp of the event
* Unique ID of the call
* Name of the queue
* Name of bridged channel
* Type of event
* Zero or more event parameters

The information contained in the event parameters depends on the type of event. A typical queue\_log file will look something like the following:

1530389309\|NONE\|NONE\|NONE\|QUEUESTART\|

1530409313\|CLI\|support\|PJSIP/SOFTPHONE\_B\|ADDMEMBER\|

1530409467\|CLI\|support\|PJSIP/SOFTPHONE\_B\|REMOVEMEMBER\|

1530409666\|NONE\|support\|PJSIP/SOFTPHONE\_B\|PAUSE\|Callbacks

1530411108\|NONE\|support\|PJSIP/SOFTPHONE\_B\|UNPAUSE\|FinishedCallbacks

1530440239\|1530440239.10\|support\|PJSIP/SOFTPHONE\_A\|ADDMEMBER\|

1530440303\|1530440303.16\|support\|PJSIP/SOFTPHONE\_A\|REMOVEMEMBER\|

1530497165\|1530497165.54\|support\|Local/PJSIP-SOFTPHONE\_A@MemberConnector\|ADDMEMBER\|

1530497388\|CLI\|support\|Local/PJSIP-SOFTPHONE\_A@MemberConnector\|REMOVEMEMBER\|

1530497408\|1530497408.60\|support\|Local/PJSIP-SOFTPHONE\_A@localMemberConnector\|ADDMEMBER\|

1530497506\|1530497506.71\|support\|NONE\|ENTERQUEUE\|\|SOFTPHONE\_B\|1

1530497511\|1530497506.71\|support\|PJSIP/SOFTPHONE\_A\|CONNECT\|5\|1530497506.72\|4

1530497517\|1530497506.71\|support\|PJSIP/SOFTPHONE\_A\|COMPLETEAGENT\|5\|6\|1

1530509861\|1530509861.134\|support\|NONE\|ENTERQUEUE\|\|SOFTPHONE\_B\|1

1530509864\|1530509861.134\|support\|PJSIP/SOFTPHONE\_A\|RINGCANCELED\|2224

1530509864\|1530509861.134\|support\|NONE\|ABANDON\|1\|1\|3

1530510503\|1530510503.156\|support\|NONE\|ENTERQUEUE\|\|103\|1

1530510503\|1530510503.156\|support\|PJSIP/SOFTPHONE\_A\|RINGNOANSWER\|0

1530510511\|1530510503.156\|support\|NONE\|ABANDON\|1\|1\|8

1530510738\|1530510738.163\|support\|NONE\|ENTERQUEUE\|\|123\|1

1530510742\|1530510738.163\|support\|PJSIP/SOFTPHONE\_A\|CONNECT\|4\|1530510738.164\|4

1530510752\|1530510738.163\|support\|PJSIP/SOFTPHONE\_A\|COMPLETECALLER\|4\|10\|1

As you can see from this example, there may not always be a unique ID for the event. External services, such as the Asterisk CLI, can perform actions on the queue, and in these cases you’ll see something like CLI in the Unique ID field.

The available events and the information they provide are described in [Table 12-7](Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition/12.%20Automatic%20Call%20Distribution%20Queues%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22ACD_id289031).

Table 12-7. Events in the Asterisk queue log

| Event | Information provided |
| :--- | :--- |
| ABANDON | Written when a caller in a queue hangs up before his call is answered by an agent. Three parameters are provided for ABANDON: the position of the caller at hangup, the original position of the caller when entering the queue, and the amount of time the caller waited prior to hanging up. |
| ADDMEMBER | Written when a member is added to the queue. The bridged channel name will be populated with the name of the channel added to the queue. |
| AGENTDUMP | Indicates that the agent hung up on the caller while the queue announcement was being played, prior to them being bridged together. |
| AGENTLOGIN | Recorded when an agent logs in. The bridged channel field will contain something like Agent/9994 if logging in with chan\_agent, and the first parameter field will contain the channel logging in \(e.g., SIP/0000FFFF0001\). |
| AGENTLOGOFF | Logged when an agent logs off, along with a parameter indicating how long the agent was logged in for. Note that since you will often use RemoveQueueMember\(\) for agent log off, this parameter may not be written. See the REMOVEMEMBER event instead. |
| COMPLETEAGENT | Recorded when a call is bridged to an agent and the agent hangs up, along with parameters indicating the amount of time the caller was held in the queue, the length of the call with the agent, and the original position at which the caller entered the queue. |
| COMPLETECALLER | Same as COMPLETEAGENT, except the caller hung up and not the agent. |
| CONFIGURELOAD | Indicates that the queue configuration was reloaded \(e.g., via module reload app\_queue.so\). |
| CONNECT | Written when the caller and the agent are bridged together. Three parameters are also written: the amount of time the caller waited in the queue, the unique ID of the queue member’s channel to which the caller was bridged, and the amount of time the queue member’s phone rang prior to being answered. |
| ENTERQUEUE | Written when a caller enters the queue. Two parameters are also written: the URL \(if specified\) and the caller ID of the caller. |
| EXITEMPTY | Written when the caller is removed from the queue due to a lack of agents available to answer the call \(as specified by the leavewhenempty parameter\). Three parameters are also written: the position of the caller in the queue, the original position at which the caller entered the queue, and the amount of time the caller was held in the queue. |
| EXITWITHKEY | Written when the caller exits the queue by pressing a single DTMF key on his phone to exit the queue and continue in the dialplan \(as enabled by the context parameter in queues.conf\). Four parameters are recorded: the key used to exit the queue, the position of the caller in the queue upon exit, the original position the caller entered the queue at, and the amount of time the caller was waiting in the queue. |
| EXITWITHTIMEOUT | Written when the caller is removed from the queue due to timeout, as specified by the timeout parameter to Queue\(\). Three parameters are also recorded: the position the caller was in when exiting the queue, the original position of the caller when entering the queue, and the amount of time the caller waited in the queue. |
| PAUSE | Written when a queue member is paused. |
| PAUSEALL | Written when all members of a queue are paused. |
| UNPAUSE | Written when a queue member is unpaused. |
| UNPAUSEALL | Written when all members of a queue are unpaused. |
| PENALTY | Written when a member’s penalty is modified. The penalty can be changed through several means, such as the QUEUE\_MEMBER\_PENALTY\(\) function, the Asterisk Manager Interface, or the Asterisk CLI commands. |
| REMOVEMEMBER | Written when a queue member is removed from the queue. The bridge channel field will contain the name of the member removed from the queue. |
| RINGNOANSWER | Logged when a queue member is rung for a period of time, and the timeout value for ringing the queue member is exceeded. A single parameter will also be written indicating the amount of time the member’s extension rang. |
| TRANSFER | Written when a caller is transferred to another extension. Additional parameters are also written, which include the extension and context the caller was transferred to, the hold time of the caller in the queue, the amount of time the caller was speaking to a member of the queue, and the original position of the caller when he entered the queue.[a](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch12.html%22%20/l%20%22idm46178405642568) |
| SYSCOMPAT | Recorded if an agent attempts to answer a call, but the call cannot be set up due to incompatibilities in the media setup. |
| [a](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch12.html%22%20/l%20%22idm46178405642568-marker) Please note that when the caller is transferred using SIP transfers \(rather than the built-in transfers triggered by DTMF and configured in features.conf\), the TRANSFER event may not be written. |  |

## Вывод

We started this chapter with a look at basic call queues, discussing what they are, how they work, and when you might want to use one. After building a simple queue, we explored how to control queue members through various means \(including the use of local channels, which provide the ability to perform some dialplan logic just prior to connecting to a queue member\). Of course, we need the ability to monitor what our queues are doing, so we had a quick look at the queue\_log file, and the various fields written as a result of events happening in our queues.

With the information provided in this chapter, you have most of the foundational knowledge required to implement queues in Asterisk.

##

<ol>
<li id="sn1"> Это распространенное заблуждение, что очередь может позволить вам обрабатывать больше вызовов. Это не совсем верно: ваши абоненты все равно захотят поговорить с живым человеком, и они будут только ждать так долго. Другими словами, если у вас мало сотрудников, ваша очередь может оказаться не более чем препятствием для ваших абонентов. Это то же самое, говорите ли вы по телефону или на кассе Walmart. Никто не любит ждать в очереди. Идеальная очередь невидима для звонящих, так как на их звонки отвечают сразу, без ожидания.</li>

<li id="sn2"> Существует несколько книг, в которых обсуждаются метрики колл-центра и доступные стратегии организации очередей, например, «Руководство по метрикам колл-центра» Джеймса Эббота (Роберт Хьюстон Смит).</li>

[3](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch12.html%22%20/l%20%22idm46178405949096-marker) We’re going to use the ^ character as a delimiter. You could probably use another character instead, just so long as it’s not one the Asterisk parser would see as a normal delimiter \(and thus get confused by\). So avoid commas, semicolons, and so forth.

[4](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch12.html%22%20/l%20%22idm46178405905624-marker) Similar to adding ballast to a jockey or racing car.

[5](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch12.html%22%20/l%20%22idm46178405806600-marker) Just sayin’.

[6](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch12.html%22%20/l%20%22idm46178405761096-marker) If the priority n+1 \(from where the Queue\(\) application was called\) is not defined, the call will be hung up. In other words, don’t use this functionality unless your dialplan does something useful at the step immediately following Queue\(\).

[7](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch12.html%22%20/l%20%22idm46178405718440-marker) Perhaps we could have used / instead of - as a delimiter, giving us Local/PJSIP/SOFTPHONE\_A@localMemberConnector, but we felt that would be more prone to strange syntax errors, and awkward to filter and parse, so we went with -.

[8](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch12.html%22%20/l%20%22idm46178405708120-marker) Obviously, don’t use any dialplan code in your local channel that will answer, such as Answer\(\), Playback\(\), and so forth.
</ol>

[Глава 11. Функции АТС, включая парковку, пейджинг и конференц-связь](glava-11.md) | [Содержание](SUMMARY.md) | [Глава 13. Состояния устройств](glava-13.md)