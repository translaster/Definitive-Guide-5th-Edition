# Глава 21. Системный мониторинг и журналирование

Chaos is inherent in all compounded things. Strive on with diligence.

The Buddha

Asterisk comes with several subsystems that allow you to obtain detailed information about the workings of your system. Whether for troubleshooting or for tracking usage for billing or staffing purposes, Asterisk’s various monitoring modules can help you keep tabs on the inner workings of your system.

## logger.conf

When troubleshooting issues in your Asterisk system, you will find it very helpful to refer to some sort of historical record of what was going on in the system at the time the reported issue occurred. The parameters for the storing of this information are defined in /etc/asterisk/logger.conf.

Ideally, you might want the system to store a record of each and every thing it does. However, there is a cost to doing this. On a busy system, with full debug logging enabled, a large amount of data will be generated. Although storage is far cheaper today than it was when Asterisk was young, it may still be necessary to achieve a balance between detail and storage requirements.

The /etc/asterisk/logger.conf file allows you to define all sorts of different levels of logging, to multiple files if desired. This flexibility is excellent, but it can also be confusing.

The format of an entry in the logger.conf file is as follows:

filename =&gt; type\[,type\[,type\[,...\]\]\]

We have already been working with the logger.conf file, so you will already have entries in it similar to the following:

\[general\]

exec\_after\_rotate=gzip -9 ${filename}.2;

\[logfiles\]

;debug =&gt; debug

;console =&gt; notice,warning,error,verbose

console =&gt; notice,warning,error,debug

messages =&gt; notice,warning,error

full =&gt; notice,warning,error,debug,verbose,dtmf,fax

;full-json =&gt; \[json\]debug,verbose,notice,warning,error,dtmf,fax

;syslog keyword : This special keyword logs to syslog facility

;syslog.local0 =&gt; notice,warning,error

If you make any changes to this file, you will need to reload the logger by issuing the following command from the shell:

$ sudo touch full messages

$ chown asterisk:asterisk /var/log/asterisk/\*

$ asterisk -rx 'logger reload'

or from the Asterisk CLI:

\*CLI&gt; logger reload

**Verbose Logging: Useful but Dangerous**

We struggled with whether to recommend adding the following line to your logger.conf file:

verbose =&gt; notice,warning,error,verbose

This is quite possibly one of the most useful debugging tools you have when building and troubleshooting a dialplan, and therefore it is highly recommended. The danger comes from the fact that if you forget to disable this when you are done with your debugging, you will have left a ticking time bomb in your Asterisk system, which will slowly fill up the hard drive and kill your system one day, several months or years from now, when you are least expecting it.

Use it. It’s fantastic. Just remember that you will need to manage your storage to ensure your logfiles don’t fill up your drive!

You can specify any filename you want, but the special filename console will in fact print the output to the Asterisk CLI, and not to any file on the hard drive. All other filenames will be stored in the filesystem in the directory /var/log/asterisk. The logger.conf types are outlined in [Table 21-1](21.%20System%20Monitoring%20and%20Logging%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22Monitoring_id269148).

Table 21-1. logger.conf types

<table>
  <thead>
    <tr>
      <th style="text-align:left">Type</th>
      <th style="text-align:left">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">notice</td>
      <td style="text-align:left">You will see a lot of these during a reload, but they will also happen
        during normal call flow. A notice is simply any event that Asterisk wishes
        to inform you of.</td>
    </tr>
    <tr>
      <td style="text-align:left">warning</td>
      <td style="text-align:left">A warning represents a problem that could be severe enough to affect a
        call (including disconnecting a call because call flow cannot continue).
        Warnings need to be addressed.</td>
    </tr>
    <tr>
      <td style="text-align:left">error</td>
      <td style="text-align:left">Errors represent significant problems in the system that must be addressed
        immediately.</td>
    </tr>
    <tr>
      <td style="text-align:left">debug</td>
      <td style="text-align:left">Debugging is only useful if you are troubleshooting a problem with the
        Asterisk code itself. You would not use debug to troubleshoot your dialplan,
        but you would use it if the Asterisk developers asked you to provide logs
        for a problem you were reporting. Do not use debug in production, as the
        amount of detail stored can fill up a hard drive in a matter of days.
        <a
        href="https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch21.html%22%20/l%20%22idm46178396457896">a</a>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">verbose</td>
      <td style="text-align:left">This is one of the most useful of the logging types, but it is also one
        of the more risky to leave unattended, due to the possibility of the output
        filling your hard drive.<a href="https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch21.html%22%20/l%20%22idm46178396456264">b</a>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">dtmf</td>
      <td style="text-align:left">Logging DTMF can be helpful if you are getting complaints that calls are
        not routing from the automated attendant correctly.</td>
    </tr>
    <tr>
      <td style="text-align:left">fax</td>
      <td style="text-align:left">This type of logging causes fax-related messages from the fax technology
        backend (res_fax_spandsp or res_fax_digium) to be logged to the fax logger.</td>
    </tr>
    <tr>
      <td style="text-align:left">*</td>
      <td style="text-align:left">This will log everything (and we mean everything). Do not use this unless
        you understand the implications of storing this amount of data. It will
        not end well.</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><a href="https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch21.html%22%20/l%20%22idm46178396457896-marker">a</a> This
          is not theory. It has happened to us. It was not fun.</p>
        <p><a href="https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch21.html%22%20/l%20%22idm46178396456264-marker">b</a> It&#x2019;s
          not as risky as debug, since it&#x2019;ll take months to fill the hard
          drive, but the danger is that it will happen, say, a year later when you&#x2019;re
          on summer vacation, and it will not immediately be obvious what the problem
          is. Not fun.</p>
      </td>
      <td style="text-align:left"></td>
    </tr>
  </tbody>
</table>**Warning**

There is a peculiarity in Asterisk’s logging system that will cause you some consternation if you are unaware of it. The level of logging for the verbose and debug logging types is tied to the verbosity as set in the console. This means that if you are logging to a file with the verbose or debug type, and somebody logs into the CLI and issues the command core set verbose 0, or core set debug 0, the logging of those details to your logfile will stop.

### Reviewing Asterisk Logs

Searching through logfiles can be a challenge. The trick is to be able to filter what you are seeing so that you are only presented with information that is relevant to what you are searching for.

To start with, you will need to have an approximate idea of when the trouble you are looking for occurred. Once you are oriented to the approximate time, you will need to find clues that will help you to identify the call in question. Obviously, the more information you have about the call, the faster you will be able to pin it down.

Asterisk 11 introduced a logging feature that helps with debugging a specific call. Log entries associated with a call now include a call ID. This call ID can be used with grep to find all log entries associated with that call. In the following example log entry, the call ID is C-00000004:

\[Dec 4 08:22:32\] WARNING\[14199\]\[C-00000004\]: app\_voicemail.c:6286

leave\_voicemail: No entry in voicemail config file for '234123452'

In earlier versions of Asterisk, there is another trick you can use. If, for example, you are doing verbose logging, you should note that each distinct call has a thread identifier, which, when used with grep, can often help you to filter out everything that does not relate to the call you are trying to debug. For example, in the following verbose log, we have more than one call in the log, and since the calls are happening at the same time, it can be very confusing to trace one call:

$ tail -1000 verbose

\[Mar 11 …\] VERBOSE\[31362\] logger.c: -- IAX2/shifteight-4 answered Zap/1-1

\[Mar 11 …\] VERBOSE\[2973\] logger.c: -- Starting simple switch on 'Zap/1-1'

\[Mar 11 …\] VERBOSE\[31362\] logger.c: == Spawn extension \(shifteight, s, 1\)

exited non-zero on 'Zap/1-1'

\[Mar 11 …\] VERBOSE\[2973\] logger.c: -- Hungup 'Zap/1-1'

\[Mar 11 …\] VERBOSE\[3680\] logger.c: -- Starting simple switch on 'Zap/1-1'

\[Mar 11 …\] VERBOSE\[31362\] logger.c: -- Hungup 'Zap/1-1'

To filter on one call specifically, we could grep on the thread ID. For example:

$ grep 31362 verbose

would give us:

\[Mar 11 …\] VERBOSE\[31362\] logger.c: -- IAX2/shifteight-4 answered Zap/1-1

\[Mar 11 …\] VERBOSE\[31362\] logger.c: == Spawn extension \(shifteight, s, 1\)

exited non-zero on 'Zap/1-1'

\[Mar 11 …\] VERBOSE\[31362\] logger.c: -- Hungup 'Zap/1-1'

This method does not guarantee that you will see everything relating to one call, since a call could in theory spawn additional threads, but for basic dialplan debugging we find this approach to be very useful when the call IDs from Asterisk 11 are not available.

### Logging to the Linux syslog Daemon

Linux contains a very powerful logging engine, which Asterisk can take advantage of. While a discussion of all the various flavors of syslog and all the possible ways to handle Asterisk logging would be beyond the scope of this book, suffice it to say that if you want to have Asterisk send logs to the syslog daemon, you simply need to specify the following in your /etc/asterisk/logger.conf file:

syslog.local0 =&gt; notice,warning,error ; or whatever type\(s\) you want to log

You will need a designation in your syslog configuration file[1](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch21.html%22%20/l%20%22idm46178396419208) named local0, which should look something like:

local0.\* /var/log/asterisk/syslog

**Note**

You can use local0 through local7 for this, but check your syslog.conf file to ensure that nothing else is using one of those syslog channels.

The use of syslog[2](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch21.html%22%20/l%20%22idm46178396410888) allows for much more powerful logging, but it also requires more knowledge than simply allowing Asterisk to log to files. It’s mostly going to be useful if you’re already collecting other logs on the system into some centralized syslog server.

### Verifying Logging

You can view the status of all your logger.conf settings through the Asterisk CLI by issuing the command:

\*CLI&gt; logger show channels

You should see output similar to:

Channel Type Status Configuration

------- ---- ------ -------------

syslog.local0 Syslog Enabled - NOTICE WARNING ERROR VERBOSE

/var/log/asterisk/verbose File Enabled - NOTICE WARNING ERROR VERBOSE

/var/log/asterisk/messages File Enabled - NOTICE WARNING ERROR

 Console Enabled - NOTICE WARNING ERROR DTMF=

### Log Rotation

There is some log rotation support built into Asterisk. Log rotation will be done in the following cases:

* If you run the logger rotate Asterisk CLI command:

\*CLI&gt; logger rotate

* During a configuration reload if any existing logfiles are greater than 1 GB in size
* If Asterisk receives the SIGXFSZ signal, indicating that a file it was writing to is too large

## Call Detail Records

The CDR system in Asterisk is used to log the history of calls in the system. In some deployments, these records are used for billing purposes. In others, call records are used for analyzing call volumes over time. They can also be used as a debugging tool by Asterisk administrators.

### CDR Contents

A CDR has a number of fields that are included by default. [Table 21-2](21.%20System%20Monitoring%20and%20Logging%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22default_cdr_fields) lists them.

Table 21-2. Default CDR fields

| Option | Value/example | Notes |
| :--- | :--- | :--- |
| accountcode | 12345 | An account ID. This field is user-defined and is empty by default. |
| src | 12565551212 | The calling party’s caller ID number. It is set automatically and is read-only. |
| dst | 102 | The destination extension for the call. This field is set automatically and is read-only. |
| dcontext | PublicExtensions | The destination context for the call. This field is set automatically and is read-only. |
| clid | "Big Bird" &lt;12565551212&gt; | The full caller ID, including the name, of the calling party. This field is set automatically and is read-only. |
| channel | SIP/0004F2040808-a1bc23ef | The calling party’s channel. This field is set automatically and is read-only. |
| dstchannel | SIP/0004F2046969-9786b0b0 | The called party’s channel. This field is set automatically and is read-only. |
| lastapp | Dial | The last dialplan application that was executed. This field is set automatically and is read-only. |
| lastdata | SIP/0004F2046969,30,tT | The arguments passed to the lastapp. This field is set automatically and is read-only. |
| start | 2010-10-26 12:00:00 | The start time of the call. This field is set automatically and is read-only. |
| answer | 2010-10-26 12:00:15 | The answered time of the call. This field is set automatically and is read-only. |
| end | 2010-10-26 12:03:15 | The end time of the call. This field is set automatically and is read-only. |
| duration | 195 | The number of seconds between the start and end times for the call. This field is set automatically and is read-only. |
| billsec | 180 | The number of seconds between the answer and end times for the call. This field is set automatically and is read-only. |
| disposition | ANSWERED | An indication of what happened to the call. This may be NO ANSWER, FAILED, BUSY, ANSWERED, or UNKNOWN. |
| amaflags | DOCUMENTATION | The Automatic Message Accounting \(AMA\) flag associated with this call. This may be one of the following: OMIT, BILLING, DOCUMENTATION, or Unknown. |
| userfield | PerMinuteCharge:0.02 | A general-purpose user field. This field is empty by default and can be set to a user-defined string.[a](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch21.html%22%20/l%20%22idm46178396348760) |
| uniqueid | 1288112400.1 | The unique ID for the src channel. This field is set automatically and is read-only. |
| [a](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch21.html%22%20/l%20%22idm46178396348760-marker) The userfield is not as relevant now as it used to be. Custom CDR variables are a more flexible way to get custom data into CDRs. |  |  |

You can access all fields of the CDR record in the Asterisk dialplan by using the CDR\(\) function. The CDR\(\) function is also used to set the fields of the CDR that are user-defined:

exten =&gt; 115,1,Verbose\(Call start time: ${CDR\(start\)}\)

 same =&gt; n,Set\(CDR\(userfield\)=zombie pancakes\)

In addition to the fields that are always included in a CDR, it is possible to add custom fields. You do this in the dialplan by using the Set\(\) application with the CDR\(\) function:

exten =&gt; 115,1,NoOp\(\)

 same =&gt; n,Set\(CDR\(mycustomfield\)=coffee\)

 same =&gt; n,Verbose\(I need some more ${CDR\(mycustomfield\)}\)

**Note**

If you choose to use custom CDR variables, make sure that the CDR backend that you choose is capable of logging them.

To view the built-in documentation for the CDR\(\) function, run the following command at the Asterisk console:

\*CLI&gt; core show function CDR

In addition to the CDR\(\) function, some dialplan applications may be used to influence CDR records. We’ll look at these next.

### Dialplan Applications

A few dialplan applications can be used to influence CDRs for the current call. To get a list of the CDR applications that are loaded into the current version of Asterisk, we can use the following CLI command:

\*CLI&gt; core show applications like CDR

 -= Matching Asterisk Applications =-

 ForkCDR: Forks the Call Data Record.

 NoCDR: Tell Asterisk to not maintain a CDR for the current call

 ResetCDR: Resets the Call Data Record.

 -= 3 Applications Matching =-

Each application has documentation built into the Asterisk application, which can be viewed using the following command:

\*CLI&gt; core show application &lt;application name&gt;

### cdr.conf

The cdr.conf file has a \[general\] section that contains options that apply to the entire CDR system. Additional optional sections may exist in this file that apply to specific CDR logging backend modules. [Table 21-3](21.%20System%20Monitoring%20and%20Logging%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22Monitoring_id250739) lists the options available in the \[general\] section.

Table 21-3. cdr.conf \[general\] section

| Option | Value/example | Notes |
| :--- | :--- | :--- |
| enable | yes | Enable CDR logging. The default is yes. |
| unanswered | no | Log unanswered calls. Normally, only answered calls result in a CDR. Logging all call attempts can result in a large number of extra call records that most people do not care about. The default value is no. |
| end before hexten | no | Close out CDRs before running the h extension in the Asterisk dialplan. Normally, CDRs are not closed until the dialplan is completely finished running. The default value is no. |
| initiated seconds | no | When calculating the billsec field, always round up. For example, if the difference between when the call was answered and when the call ended is 1 second and 1 microsecond, billsec will be set to 2 seconds. This helps ensure that Asterisk’s CDRs match the behavior used by telcos. The default value is no. |
| batch | no | Queue up CDRs to be logged in batches instead of logging synchronously at the end of every call. This prevents CDR logging from blocking the completion of the call teardown process within Asterisk. Using batch mode can be incredibly useful when working with a database that may be slow to process requests. The default value is no, but we recommend turning it on.[a](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch21.html%22%20/l%20%22idm46178396300104) |
| size | 100 | Set the number of CDRs to queue up before they are logged during batch mode. The default value is 100. |
| time | 300 | Set the maximum number of seconds that CDRs will wait in the batch queue before being logged. The CDR batch-logging process will run at the end of this time period, even if size has not been reached. The default value is 300 seconds. |
| scheduler only | no | Set whether CDR batch processing should be done by spawning a new thread, or within the context of the CDR batch scheduler. The default value is no, and we recommend not changing it. |
| safe shutdown | yes | Block Asterisk shutdown to ensure that all queued CDR records are logged. The default is yes, and we recommend leaving it that way, as this option prevents important data loss. |
| [a](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch21.html%22%20/l%20%22idm46178396300104-marker) The disadvantage of enabling this option is that if Asterisk were to crash or die for some reason, the CDR records would be lost, as they are only stored in memory while the Asterisk process exists. See safeshutdown for more information. |  |  |

### Backends

Asterisk CDR backend modules provide a way to log CDRs. Most CDR backends require specific configuration to get them going.

#### cdr\_adaptive\_odbc

As the name suggests, the cdr\_adaptive\_odbc module allows CDRs to be stored in a database through ODBC. The “adaptive” part of the name refers to the fact that it works to adapt to the table structure: there is no static table structure that must be used with this module. When the module is loaded \(or reloaded\), it reads the table structure. When logging CDRs, it looks for a CDR variable that matches each column name. This applies to both the built-in CDR variables and custom variables. If you want to log the built-in channel CDR variable, just create a column called channel.

Adding custom CDR content is as simple as setting it in the dialplan. For example, if we wanted to log the User-Agent that is provided by a SIP device, we could add that as a custom CDR variable:

exten =&gt; 105,n,Set\(CDR\(useragent\)=${CHANNEL\(useragent\)}\)

To have this custom CDR variable inserted into the database by cdr\_adaptive\_odbc, all we have to do is create a column called useragent.

Multiple tables may be configured in the cdr\_adaptive\_odbc configuration file. Each goes into its own configuration section. The name of the section can be anything; the module does not use it. Here is an example of a simple table configuration:

\[mytable\]

connection = asterisk

table = asterisk\_cdr

A more detailed example of setting up a database for logging CDRs can be found in [“Storing Call Detail Records”](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch15.html%22%20/l%20%22database_storing-cdr).

[Table 21-4](21.%20System%20Monitoring%20and%20Logging%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22Monitoring_id263326) lists the options that can be specified in a table configuration section in the cdr\_adaptive\_odbc.conf file.

Table 21-4. cdr\_adaptive\_odbc.conf table configuration options

| Option | Value/example | Notes |
| :--- | :--- | :--- |
| connection | pgsql1 | The database connection to be used. This is a reference to the configured connection in res\_odbc.conf. This field is required. |
| table | asterisk\_cdr | The table name. This field is required. |
| usegmtime | no | Indicates whether to log timestamps using GMT instead of local time. The default value for this option is no. |

In addition to the key/value pair fields that are shown in the previous table, cdr\_adaptive\_odbc.conf allows for a few other configuration items. The first is a column alias. Normally, CDR variables are logged to columns of the same name. An alias allows the variable name to be mapped to a column with a different name. The syntax is:

alias CDR variable =&gt; column name

Here is an example column mapping using the alias option:

alias src =&gt; source

It is also possible to specify a content filter. This allows you to specify criteria that must match for records to be inserted into the table. The syntax is:

filter CDR variable =&gt; content

Here is an example content filter:

filter accountcode =&gt; 123

Finally, cdr\_adaptive\_odbc.conf allows static content for a column to be defined. This can be useful when combined with a set of filters. This static content can help differentiate records that were inserted into the same table by different configuration sections. The syntax for static content is:

static "Static Content Goes Here" =&gt; column name

Here is an example of specifying static content to be inserted with CDRs:

static "My Content" =&gt; my\_identifier

#### cdr\_csv

The cdr\_csv module is a very simple CDR backend that logs CDRs into a CSV \(comma-separated values\) file. The file is /var/log/asterisk/cdr-csv/Master.csv. As long as CDR logging is enabled in cdr.conf and this module has been loaded, CDRs will be logged to the Master.csv file. We recommend that regardless of any other CDR backend you choose to configure, you leave this configured as well, as it will serve as an excellent backup should you lose other CDR data due to network or related issues.

While no options are required to get this module working, there are some options that customize its behavior. These options, listed in [Table 21-5](21.%20System%20Monitoring%20and%20Logging%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22Monitoring_id263534), are placed in the \[csv\] section of cdr.conf.

Table 21-5. cdr.conf \[csv\] section options

| Option | Value/example | Notes |
| :--- | :--- | :--- |
| usegmtime | no | Log timestamps using GMT instead of local time. The default is no. |
| loguniqueid | no | Log the uniqueid CDR variable. The default is no. |
| loguserfield | no | Log the userfield CDR variable. The default is no. |
| accountlogs | yes | Create a separate CSV file for each different value of the accountcode CDR variable. The default is yes. |

The order of CDR variables in CSV files created by the cdr\_csv module is:

&lt;accountcode&gt;,&lt;src&gt;,&lt;dst&gt;,&lt;dcontext&gt;,&lt;clid&gt;,&lt;channel&gt;,&lt;dstchannel&gt;,&lt;lastapp&gt;, \

 &lt;lastadata&gt;,&lt;start&gt;,&lt;answer&gt;,&lt;end&gt;,&lt;duration&gt;,&lt;billsec&gt;,&lt;disposition&gt;, \

 &lt;amaflags&gt;\[,&lt;uniqueid&gt;\]\[,&lt;userfield&gt;\]

Place the following lines into /etc/asterisk/cdr.conf:

\[general\]

enable=yes

\[csv\]

usegmtime=yes ; log date/time in GMT. Default is "no"

loguniqueid=yes ; log uniqueid. Default is "no"

loguserfield=yes ; log user field. Default is "no"

accountlogs=yes ; create separate log file for each account code. Default is "yes"

;newcdrcolumns=yes ; Enable logging of post-1.8 CDR columns \(peeraccount,linkedid,sequence\)

 ; Default is "no".

Save it, chown it, and reload the CDR module.

$ chown asterisk:asterisk /etc/asterisk/cdr.conf

$ sudo asterisk -rx 'module reload cdr'

#### cdr\_custom

This CDR backend allows for custom formatting of CDR records in a logfile. This module is most commonly used for customized CSV output. The configuration file used for this module is /etc/asterisk/cdr\_custom.conf. A single section called \[mappings\] should exist in this file. The \[mappings\] section contains mappings between a filename and the custom template for a CDR. The template is specified using Asterisk dialplan functions.

The following example shows a sample configuration for cdr\_custom that enables a single CDR logfile, Master.csv. This file will be created as /var/log/asterisk/cdr-custom/Master.csv. The template that has been defined uses both the CDR\(\) and CSV\_QUOTE\(\) dialplan functions. The CDR\(\) function retrieves values from the CDR being logged. The CSV\_QUOTE\(\) function ensures that the values are properly escaped for the CSV file format:

\[mappings\]

Master.csv =&gt; ${CSV\_QUOTE\(${CDR\(clid\)}\)},${CSV\_QUOTE\(${CDR\(src\)}\)},

 ${CSV\_QUOTE\(${CDR\(dst\)}\)},${CSV\_QUOTE\(${CDR\(dcontext\)}\)},

 ${CSV\_QUOTE\(${CDR\(channel\)}\)},${CSV\_QUOTE\(${CDR\(dstchannel\)}\)},

 ${CSV\_QUOTE\(${CDR\(lastapp\)}\)},${CSV\_QUOTE\(${CDR\(lastdata\)}\)},

 ${CSV\_QUOTE\(${CDR\(start\)}\)},${CSV\_QUOTE\(${CDR\(answer\)}\)},

 ${CSV\_QUOTE\(${CDR\(end\)}\)},${CSV\_QUOTE\(${CDR\(duration\)}\)},

 ${CSV\_QUOTE\(${CDR\(billsec\)}\)},${CSV\_QUOTE\(${CDR\(disposition\)}\)},

 ${CSV\_QUOTE\(${CDR\(amaflags\)}\)},${CSV\_QUOTE\(${CDR\(accountcode\)}\)},

 ${CSV\_QUOTE\(${CDR\(uniqueid\)}\)},${CSV\_QUOTE\(${CDR\(userfield\)}\)}

**Note**

In the actual configuration file, the value in the Master.csv mapping should be on a single line.

#### cdr\_manager

The cdr\_manager backend emits CDRs as events on the Asterisk Manager Interface \(AMI\), which we discussed in detail in [Chapter 17](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch17.html%22%20/l%20%22asterisk-AMI). This module is configured in the /etc/asterisk/cdr\_manager.conf file. The first section in this file is the \[general\] section, which contains a single option to enable this module \(the default value is no\):

\[general\]

enabled = yes

The other section in cdr\_manager.conf is the \[mappings\] section. This allows for adding custom CDR variables to the manager event. The syntax is:

CDR variable =&gt; Header name

Here is an example of adding two custom CDR variables:

\[mappings\]

rate =&gt; Rate

carrier =&gt; Carrier

With this configuration in place, CDR records will appear as events on the manager interface. To generate an example manager event, we will use the following dialplan example:

exten =&gt; 110,1,Answer\(\)

 same =&gt; n,Set\(CDR\(rate\)=0.02\)

 same =&gt; n,Set\(CDR\(carrier\)=BS&S\)

 same =&gt; n,Hangup\(\)

This is the command used to execute this extension and generate a sample manager event:

\*CLI&gt; console dial 110@testing

Finally, this is an example manager event produced as a result of this test call:

Event: Cdr

Privilege: cdr,all

AccountCode:

Source:

Destination: 110

DestinationContext: testing

CallerID:

Channel: Console/dsp

DestinationChannel:

LastApplication: Hangup

LastData:

StartTime: 2010-08-23 08:27:21

AnswerTime: 2010-08-23 08:27:21

EndTime: 2010-08-23 08:27:21

Duration: 0

BillableSeconds: 0

Disposition: ANSWERED

AMAFlags: DOCUMENTATION

UniqueID: 1282570041.3

UserField:

Rate: 0.02

Carrier: BS&S

#### cdr\_odbc

This module enables the legacy ODBC interface for CDR logging. New installations should use cdr\_adaptive\_odbc instead.

#### cdr\_sqlite

This module allows posting of CDRs to an SQLite database using SQLite version 2. Unless you have a specific need for SQLite version 2 as opposed to version 3, we recommend that all new installations use cdr\_sqlite3\_custom.

This module requires no configuration to work. If the module has been compiled and loaded into Asterisk, it will insert CDRs into a table called cdr in a database located at /var/log/asterisk/cdr.db.

#### cdr\_sqlite3\_custom

This CDR backend inserts CDRs into an SQLite database using SQLite version 3. The database created by this module lives at /var/log/asterisk/master.db. This module requires a configuration file, /etc/asterisk/cdr\_sqlite3\_custom.conf. The configuration file identifies the table name, as well as customizes which CDR variables will be inserted into the database.

#### cdr\_syslog

This module allows logging of CDRs using syslog. To enable this, first add an entry to the system’s syslog configuration file, /etc/syslog.conf. For example:

local4.\* /var/log/asterisk/asterisk-cdr.log

The Asterisk module has a configuration file as well. Add the following section to /etc/asterisk/cdr\_syslog.conf:

\[cdr\]

facility = local4

priority = info

template = "We received a call from ${CDR\(src\)}"

Here is an example syslog entry using this configuration:

$ cat /var/log/asterisk/asterisk-cdr.log

Aug 12 19:17:36 pbx cdr: "We received a call from 2565551212"

### Example Call Detail Records

We will use the cdr\_custom module to illustrate some example CDR records for different call scenarios. The configuration used for /etc/asterisk/cdr\_custom.conf is shown in [“cdr\_custom”](21.%20System%20Monitoring%20and%20Logging%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22cdr_custom).

#### Single-party call

In this example, we’ll show what a CDR looks like for a simple one-party call:

exten =&gt; 227,1,VoiceMailMain\(@${GLOBAL\(VOICEMAIL\_CONTEXT\)}\)

This is the CDR from /var/log/asterisk/cdr-custom/Master.csv that was created as a result of calling this extension:

"","SOFTPHONE\_A","227","sets","""101"" &lt;SOFTPHONE\_A&gt;","PJSIP/SOFTPHONE\_A-00000002",

"","Playback","hear-odd-noise",

"2019-03-04 02:31:39","2019-03-04 02:31:39","2019-03-04 02:31:42",

3,3,"ANSWERED","DOCUMENTATION","1551666699.4",""

Open it up in a spreadsheet and it’ll be lined up neatly.

### Caveats

The CDR system in Asterisk works very well for fairly simple call scenarios. However, as call scenarios get more complicated—involving calls to multiple parties, transfers, parking, and other such features—the CDR system starts to fall short. Many users report that the records do not show all the information that they expect. Many bug fixes have been made to address some of the issues, but the cost of regressions or changes in behavior when making changes in this area is very high, since these records are used for billing.

As a result, the Asterisk development team has become increasingly resistant to making additional changes to the CDR system. Instead, a new system, channel event logging \(CEL\), has been developed that is intended to help address logging of more complex call scenarios. Bear in mind that call detail records are simpler and easier to consume, though, so we still recommend using CDRs if they suit your needs.

## Channel Event Logging

Channel event logging \(CEL\) provides a more flexible means of logging the details of complex call scenarios. Instead of collapsing a call down to a single log entry, a series of events are logged for the call. This provides a more accurate picture of what has happened to the call, at the expense of a more complex log.

For more details on CEL, check out the [Asterisk wiki](https://wiki.asterisk.org/).

## Conclusion

Asterisk is very good at allowing you to keep track of many different facets of its operation, from simple call detail records to full debugging of the running code. Take a look in the source code directories, and you’ll find many more components than we’ve had space to cover here. These various mechanisms will help you in your efforts to manage your Asterisk PBX, and they represent one of the ways that Asterisk is vastly superior to most \(if not all\) traditional PBXs.

[1](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch21.html%22%20/l%20%22idm46178396419208-marker) Which will normally be found at /etc/syslog.conf.

[2](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch21.html%22%20/l%20%22idm46178396410888-marker) And rsyslog, syslog-ng, and what-all-else.
