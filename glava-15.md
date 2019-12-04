# Глава 15. Интеграция реляционной базы данных

>Few things are harder to put up with than the annoyance of a good example.
>
> _--Марк Твен_

In this chapter, we are going to explore integrating some Asterisk features and functions into a database. There are several databases available for Linux, and Asterisk supports the most popular of them through its ODBC connector. While this chapter will demonstrate examples using the ODBC connector with a MySQL database, you will find that most of the concepts will apply to any database supported by unixODBC.

Integrating Asterisk with databases is one of the fundamental aspects of building a large clustered or distributed system. The power of the database will enable you to use dynamically changing data in your dialplans, for tasks like sharing information across an array of Asterisk systems or integrating with web-based services. Our favorite dialplan function, which we will cover later in this chapter, is func_odbc. We’ll also take a look at the Asterisk Realtime Architecture (ARA), call detail records (CDR), and logging details from any ACD queues you might have.

While not all Asterisk deployments will require relational databases, understanding how to harness them opens a treasure chest full of new ways to design your telecom solution.

## Your Choice of Database

In [Главе 3](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch03.html%22%20/l%20%22asterisk-Install), we installed and configured MySQL, plus the ODBC connector to it, and we’ve been using the tables that Asterisk provides to allow various configuration options to be stored in the database.

We chose MySQL primarily because it is still the most popular open source database engine, and rather than bouncing around, duplicating trivial commands on various different engines, we left implementing other types of databases to the skill set of the reader. If you want to use a different database such as MariaDB, PostGreSQL, Microsoft SQL, or in fact dozens \(perhaps hundreds\) of other databases supported by unixODBC, it’s quite likely that Asterisk will work with it.

Asterisk also offers native connectors to several databases; however, ODBC works so well we’ve never found any obvious reason to do things any other way. We’re going to both recommend ODBC, and also focus exclusively on it. If you have a preference for something else, this chapter should still provide you with the fundamentals, as well as some working examples, and from there you are of course free to branch out into other methodologies.

Note that regardless of the database you choose, this book cannot teach you about databases. We have tried as best as we can to provide examples that do not require too much expertise in database administration (DBA), but the simple fact is that basic DBA skills are a prerequisite for being able to fully harness the power of any database, including any you might wish to integrate with your Asterisk system. Database skills are essential to nearly all system administrative disciplines these days, so we felt it was appropriate to assume at least a basic level of familiarity with database concepts.

## Managing Databases

While it isn’t within the scope of this book to teach you how to manage databases, it is at least worth noting briefly some of the applications you could use to help with database management. There are many options, some of which are local client applications running from your computer and connecting to the database, and others being web-based applications that could be served from the same computer running the database itself, thereby allowing you to connect remotely.

Some of the ones we’ve used include:

* phpMyAdmin
* MySQL Workbench
* Navicat \(commercial\)

In our examples we will be using the MySQL command line, not because it is superior, but simply because it’s ubiquitous on any system with MySQL, so you’ve already got it and have been using it in this book.

For more heavy-duty database design, the command line is probably not as powerful as a well-designed GUI would be. Grab a copy of MySQL Workbench at least and give it a whirl.

### Troubleshooting Database Issues

When working with ODBC database connections and Asterisk, it is important to remember that the ODBC connection abstracts some of the information passed between Asterisk and the database. In cases where things are not working as expected, you may need to enable logging on your database platform to see what Asterisk is sending to the database \(e.g., which SELECT, INSERT, or UPDATE statements are being triggered from Asterisk\), what the database is seeing, and why the database may be rejecting the statements.

For example, one of the most common problems found with ODBC database integration is an incorrectly defined table or a missing column that Asterisk expects to exist. While great strides have been made in the form of adaptive modules, not all parts of Asterisk are adaptive. In the case of ODBC voicemail storage, you may have missed a column such as flag, which is a new column not found in versions of Asterisk prior to 11.[1](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch15.html%22%20/l%20%22idm46178405309816) As noted, in order to debug why your data is not being written to the database as expected, you should enable statement logging on the database side, and then determine what statement is being executed and why the database is rejecting it.

### SQL Injection

Security is always a consideration when you are building networked applications, and database security is no exception.

In the case of Asterisk, you have to think about what input you are accepting from users \(typically what they are able to submit to the dialplan\), and work to sanitize that input to ensure you are only allowing characters that are valid to your application. As an example, a typical telephone call would only allow digits as input \(and possibly the \* and \# characters\), so there would be no reason to accept any other characters. Bear in mind that the SIP protocol allows more than just numbers as part of an address, so don’t assume that somebody attempting to compromise your system is limited to just digits.

A little extra time spent sanitizing your allowed input will improve the security of your application.

## Powering Your Dialplan with func\_odbc

The func\_odbc dialplan function module allows you to define and use relatively simple functions in your dialplan that will retrieve information from databases as calls are being processed. There are all kinds of ways in which this might be used, such as managing users or allowing the sharing of dynamic information within a clustered set of Asterisk machines. We won’t claim that this will make designing and writing dialplan code easier, but we will promise that it will allow you to add a whole new level of power to your dialplans, especially if you are comfortable working with databases. We don’t know anybody in the Asterisk community who does not love func\_odbc.

The way func\_odbc works is by allowing you to define SQL queries, to which you assign function names. The func\_odbc.conf file is where you specify the relationships between the functions you create and the SQL statements you wish them to perform. You use the named functions you have created in your dialplan to retrieve and update values in the database.

In order to get you into the right frame of mind for what follows, we want you to picture a Dagwood sandwich.[2](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch15.html%22%20/l%20%22idm46178405291112)

Can you relay the total experience of such a thing by showing someone a picture of a tomato, or by waving a slice of cheese about? Hardly. That is the conundrum we faced when trying to give useful examples of why func\_odbc is so powerful. So, we decided to build the whole sandwich for you. It’s quite a mouthful, but after a few bites of this, peanut butter and jelly is never going to be the same.

## ODBC Configuration File Relationships

Several files must all line up in order for Asterisk to be able to use ODBC from the dialplan. [Figure 15-1](15.%20Relational%20Database%20Integration%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22asterisk-DB-FIG-1) attempts to convey this visually. You will probably find this diagram more helpful once you have worked through the examples in the following sections.

![](.gitbook/assets/0%20%286%29.png)

**Figure 15-1. Relationships between func\_odbc.conf, res\_odbc.conf, /etc/odbc.ini \(unixODBC\), and the database connection**

## A Gentle Introduction to func\_odbc

Before we dive into func\_odbc, we feel a wee bit of history is in order.

The very first use of func\_odbc, which occurred while its author was still writing it, is also a good introduction to its use. A customer of one of the module’s authors noted that some people calling into his switch had figured out a way to make free calls with his system. While his eventual intent was to change his dialplan to avoid those problems, he needed to blacklist certain caller IDs in the meantime, and the database he wanted to use for this was a Microsoft SQL Server database.

With a few exceptions, this is the actual dialplan:

\[span3pri\]

exten =&gt; \_50054XX,1,NoOp\(\)

 same =&gt; n,Set\(CDR\(accountcode\)=pricall\)

 ; Does this callerID appear in the database?

 same =&gt; n,GotoIf\($\[${ODBC\_ANIBLOCK\(${CALLERID\(number\)}\)}\]?busy\)

 same =&gt; n\(dial\),Dial\(DAHDI/G1/${EXTEN}\)

 same =&gt; n\(busy\),Busy\(10\) ; Yes, you are on the blacklist.

 same =&gt; n,Hangup

This dialplan, in a nutshell, passes all calls to another system for routing purposes, except those calls whose caller IDs are in a blacklist. The calls coming into this system used a block of 100 seven-digit DIDs. You will note a dialplan function is being used that you won’t find listed in any of the functions that ship with Asterisk: ODBC\_ANIBLOCK\(\). This function was instead defined in another configuration file, func\_odbc.conf:

\[ANIBLOCK\]

dsn=telesys

readsql=SELECT IF\(COUNT\(1\)&gt;0, 1, 0\) FROM Aniblock WHERE NUMBER='${ARG1}'

So, your ODBC\_ANIBLOCK\(\)[3](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch15.html%22%20/l%20%22idm46178405269208) function connects to a data source in res\_odbc.conf named telesys and selects a count of records that have the NUMBER specified by the argument, which is \(referring to the preceding dialplan\) the caller ID. Nominally, this function should return either a 1 \(indicating the caller ID exists in the Aniblock table\) or a 0 \(if it does not\). This value also evaluates directly to true or false, which means we don’t need to use an expression in our dialplan to complicate the logic.

And that, in a nutshell, is what func\_odbc is all about: writing custom dialplan functions that return a result from a database. Next up, a more detailed example of how one might use func\_odbc.

## Getting Funky with func\_odbc: Hot-Desking

OK, back to the Dagwood sandwich we promised.

We believe the value of func\_odbc will become very clear to you if you work through the following example, which will produce a new feature on your Asterisk system that depends heavily on func\_odbc.

Picture a small company with a sales force of five people who have to share two desks. This is not as cruel as it seems, because these folks spend most of their time on the road, and they are each only in the office for at most one day each week.

Still, when they do get into the office, they’d like the system to know which desk they are sitting at, so that their calls can be directed there. Also, the boss wants to be able to track when they are in the office and control calling privileges from those phones when no one is there.

This need is typically solved by what is called a hot-desking feature. We have built one for you in order to show you the power of func\_odbc.

Let’s start with the easy stuff, and create two new phone credentials in our database.

First, the endpoints table:

MySQL&gt; INSERT INTO asterisk.ps\_endpoints \(id,transport,aors,auth,context,disallow,allow, \

direct\_media,callerid\)

VALUES

\('HOTDESK\_1','transport-tls','HOTDESK\_1','HOTDESK\_1','hotdesk','all','ulaw','no', \

'HOTDESK\_1'\),

\('HOTDESK\_2','transport-tls','HOTDESK\_2','HOTDESK\_2','hotdesk','all','ulaw','no', \

'HOTDESK\_2'\);

Then, the auths:

MySQL&gt; INSERT INTO asterisk.ps\_auths \(id,auth\_type,password,username\)

VALUES

\('HOTDESK\_1','userpass','notsohot1','HOTDESK\_1'\),

\('HOTDESK\_2','userpass','notsohot2','HOTDESK\_2'\);

Finally, the aors:

MySQL&gt; INSERT INTO asterisk.ps\_aors

\(id,max\_contacts\)

VALUES

\('HOTDESK\_1',1\),

\('HOTDESK\_2',1\);

Notice that we’ve told these two endpoints to enter the dialplan at a context named \[hotdesk\]. We’ll define that shortly.

That’s all for our endpoint configuration. We’ve got a few slices of bread, which is hardly a sandwich yet.

Now let’s get the custom database built that we’re going to use for this.

Connect to your MySQL console as root:

$ mysql -u root -p

First we want a new schema to put all this in. It’s technically possible to put this in the asterisk schema, but we prefer to leave that schema alone, reserved only for whatever Asterisk’s Alembic scripts do with it during upgrades.

MySQL&gt; CREATE SCHEMA pbx;

MySQL&gt; GRANT SELECT,INSERT,UPDATE,DELETE,EXECUTE,SHOW VIEW ON pbx.\* TO 'asterisk'@'::1';

MySQL&gt; GRANT SELECT,INSERT,UPDATE,DELETE,EXECUTE,SHOW VIEW ON pbx.\* TO \

'asterisk'@'127.0.0.1';

MySQL&gt; GRANT SELECT,INSERT,UPDATE,DELETE,EXECUTE,SHOW VIEW ON pbx.\* TO \

'asterisk'@'localhost';

MySQL&gt; GRANT SELECT,INSERT,UPDATE,DELETE,EXECUTE,SHOW VIEW ON pbx.\* TO \

'asterisk'@'localhost.localdomain';

MySQL&gt; FLUSH PRIVILEGES;

Then create the table with the following bit of SQL:

CREATE TABLE pbx.ast\_hotdesk

\(

 id serial NOT NULL,

 extension text,

 first\_name text,

 last\_name text,

 cid\_name text,

 cid\_number varchar\(10\),

 pin int,

 status bool DEFAULT false,

 endpoint text,

 CONSTRAINT ast\_hotdesk\_id\_pk PRIMARY KEY \(id\)

\);

After that, populate the database with the following information \(some of the values that you see actually will change only after the dialplan work is done, but we include it here by way of example\).

At the MySQL console, run the following command:

MySQL&gt; INSERT INTO pbx.ast\_hotdesk

\(extension, first\_name, last\_name, cid\_name, cid\_number, pin, status\)

VALUES

\('1101','Herb','Tarlek','WKRP','1101','110111',0\)

\('1102','Al','Bundy','Garys','1102','110222',0\),

\('1103','Willy','Loman','','1103','110333',0\),

\('1104','Jerry','Lundegaard','Gustafson','1104','110444',0\),

\('1105','Moira','Brown','Craterside','1105','110555',0\);

Repeat these commands, changing the VALUES as needed, for all entries you wish to have in the database.[4](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch15.html%22%20/l%20%22idm46178405232168) After you’ve input your sample data, you can view the data in the ast\_hotdesk table by running a simple SELECT statement from the database console:

MySQL&gt; SELECT \* FROM pbx.ast\_hotdesk;

Which might give you something like the following output:

+--+---------+----------+----------+----------+----------+------+------+--------+

\|id\|extension\|first\_name\|last\_name \|cid\_name \|cid\_number\|pin \|status\|endpoint\|

+--+---------+----------+----------+----------+----------+------+------+--------+

\| 1\|1101 \|Herb \|Tarlek \|WKRP \|1101 \|110111\| 0\|NULL \|

\| 2\|1102 \|Al \|Bundy \|Garys \|1102 \|110222\| 0\|NULL \|

\| 3\|1103 \|Willy \|Loman \| \|1103 \|110333\| 0\|NULL \|

\| 4\|1104 \|Jerry \|Lundegaard\|Gustafson \|1104 \|110444\| 0\|NULL \|

\| 5\|1105 \|Moira \|Brown \|Craterside\|1105 \|110555\| 0\|NULL \|

+--+---------+----------+----------+----------+----------+------+------+--------+

We’ve got the condiments now, so let’s get to our dialplan. This is where the magic is going to happen.

Somewhere in extensions.conf we are going to create the \[hotdesk\] context. To start, let’s define a pattern-match extension that will allow the users to log in:

\[hotdesk\]

include =&gt; sets

exten =&gt; \_\*99110\[1-5\],1,Noop\(Hotdesk login\)

 same =&gt; n,Set\(HotExten=${EXTEN:3}\) ; strip off the leading \*99

 same =&gt; n,Noop\(Hotdesk Extension ${HotExten} is changing status\) ; for the log

 same =&gt; n,Set\(${HotExten}\_STATUS=${HOTDESK\_INFO\(status,${HotExten}\)}\)

 same =&gt; n,Set\(${HotExten}\_PIN=${HOTDESK\_INFO\(pin,${HotExten}\)}\)

 same =&gt; n,Noop\(${HotExten}\_PIN is now ${${HotExten}\_PIN}\)

 same =&gt; n,Noop\(${HotExten}\_STATUS is ${${HotExten}\_STATUS}\)}\)

We’re not done writing this extension yet, but we need to digress for a few pages to discuss where we’re at so far.

When a sales agent sits down at a desk, they log in by dialing \*99 plus their extension number. In this case we have allowed the 1101 through 1105 extensions to log in with our pattern match of \_99110\[1-5\]. You could just as easily make this less restrictive by using \_9911XX \(allowing 1100 through 1199\). This extension uses func\_odbc to perform a lookup with the HOTDESK\_INFO\(\) dialplan function. This custom function \(which we will define in the func\_odbc.conf file\) performs an SQL statement and returns whatever is retrieved from the database.

So, let’s create the /etc/asterisk/func\_odbc.conf file, and within that define the new function HOTDESK\_INFO\(\):

$ sudo -u asterisk vim /etc/asterisk/func\_odbc.conf

\[INFO\]

prefix=HOTDESK

dsn=asterisk

synopsis=Select value of field in ARG1, where 'extension' matches ARG2

description=Allow dialplan to extract data from any field in pbx.ast\_hotdesk table.

readsql=SELECT ${ARG1} FROM pbx.ast\_hotdesk WHERE extension = '${ARG2}'

That’s a lot of stuff in just a few lines. Let’s quickly cover them before we move on.

**Note**

You should be able to reload your dialplan \(dialplan reload\) and func\_odbc \(module reload func\_odbc.so\), and test the dialplan out thus far \(dial 991101 from one of the sets you’ve assigned to this context\). Make sure your console verbosity is set to at least 3 \(\*CLI&gt; core set verbose 3\), as you will only be able to see this dialplan working by the console \(a call to this dialplan will return a fast busy even if it runs successfully\). For the rest of this section, we strongly recommend you test everything after each change. If you don’t, you’ll have a whale of a time trying to find the bugs. It’s critical that you code with a phone registered and the Asterisk console open, so you can reload and test changes within seconds of writing them.

First of all, the prefix is optional \(default prefix is 'ODBC'\). This means that if you don’t define a prefix, Asterisk adds 'ODBC' to the function name \(in this case, INFO\), which means this function would become ODBC\_INFO\(\). This is not very descriptive of what the function is doing, so it can be helpful to assign a prefix that helps to relate your ODBC functions to the tasks they are performing. We chose 'HOTDESK', which means that this custom function will be named HOTDESK\_INFO\(\) in the dialplan.

**Note**

The reason why prefix is separate is that the author of the module wanted to reduce possible collisions with existing dialplan functions. The intent of prefix was to allow multiple copies of the same function, connected to different databases, for multitenant Asterisk systems. We as authors have been a bit more liberal in our use of prefix than the developer originally intended.

The dsn attribute tells Asterisk which connection to use from res\_odbc.conf. Since several database connections could be configured in res\_odbc.conf, we specify which one to use here. In [Figure 15-1](15.%20Relational%20Database%20Integration%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22asterisk-DB-FIG-1), we show the relationship between the various file configurations and how they reference down the chain to connect to the database.

**Tip**

The func\_odbc.conf.sample file in the Asterisk source contains additional information about how to handle multiple databases and control the reading and writing of information to different DSN connections. Specifically, the readhandle, writehandle, readsql, and writesql arguments will provide you with great flexibility for database integration and control.

Finally, we define our SQL statement with the readsql attribute. Dialplan functions can be called with two different formats: one for retrieving information, and one for setting information. The readsql attribute is used when we call the HOTDESK\_INFO\(\) function with the retrieve format \(we could execute a separate SQL statement with the writesql attribute; we’ll discuss the format for that attribute a little bit later in this chapter\).

Reading values from this function would take this format in the dialplan:

exten =&gt; s,n,Set\(RETURNED\_VALUE=${HOTDESK\_INFO\(status,1101\)}\)

This would return the value located in the database within the status column where the extension column equals 1101. The status and 1101 we pass to the HOTDESK\_INFO\(\) function are then placed into the SQL statement we assigned to the readsql attribute, available as ${ARG1} and ${ARG2}, respectively. If we had passed a third option, this would have been available as ${ARG3}.

After the SQL statement is executed, the value returned \(if any\) is assigned to the RETURNED\_VALUE channel variable.

#### Using the ARRAY\(\) Function

In our example, we are utilizing two separate database calls and assigning those values to a pair of channel variables: ${HotdeskExtension}\_STATUS and ${HotdeskExtension}\_PIN. This was done to simplify the example. We’re going to shorten the names of the variables here because the printed format can’t handle such long lines, so in the following examples, you’ll see “HE” in place of “HotdeskExtension.” If you’re going to code this example, please replace HE with HotdeskExtension:

 same =&gt; n,Set\(${HE}\_STATUS=${HOTDESK\_INFO\(status,${HE}\)}\)

 same =&gt; n,Set\(${HE}\_PIN=${HOTDESK\_INFO\(pin,${HE}\)}\)

As an alternative, we could have returned multiple columns and saved them to separate variables utilizing the ARRAY\(\) dialplan function. If we had defined our SQL statement in an func\_odbc.conf function like so:

readsql=SELECT pin,status FROM ast\_hotdesk WHERE extension = '${HE}'

we could have used the ARRAY\(\) function to save each column of information for the row to its own variable with a single call to the database \(note that we’re using an example function called HOTDESK\_INFO\(\), which we haven’t created\):

 same =&gt; n,Set\(ARRAY\(${HE}\_PIN,${HE}\_STATUS\)=${HOTDESK\_INFO\(${HE}\)}\)

Using ARRAY\(\) is handy any time you might get comma-separated values back and want to assign the values to separate variables, such as with CURL\(\). However, it can also make your code more complicated to read, debug, and maintain.

So, in the first two lines of the following block of code, we are passing the value status and the value contained in the ${HotdeskExtension} variable \(e.g., 1101\) to the HOTDESK\_INFO\(\) function. The two values are then replaced in the SQL statement with ${ARG1} and ${ARG2}, respectively, and the SQL statement is executed. Finally, the value returned is assigned to the ${HotdeskExtension}\_STATUS channel variable.

Let’s finish writing the pattern-match extension now:

exten =&gt; \_\*99110\[1-5\],1,Noop\(Hotdesk login\)

 same =&gt; n,Set\(HotdeskExtension=${EXTEN:3}\) ; strip off the leading \*99

 same =&gt; n,Noop\(Hotdesk Extension ${HotdeskExtension} is changing status\) ; for the log

 same =&gt; n,Set\(${HotdeskExtension}\_STATUS=${HOTDESK\_INFO\(status,${HotdeskExtension}\)}\)

 same =&gt; n,Set\(${HotdeskExtension}\_PIN=${HOTDESK\_INFO\(pin,${HotdeskExtension}\)}\)

 same =&gt; n,Noop\(${HotdeskExtension}\_PIN is now ${${HotdeskExtension}\_PIN}\)

 same =&gt; n,Noop\(${HotdeskExtension}\_STATUS is ${${HotdeskExtension}\_STATUS}\)}\)

 same =&gt; n,GotoIf\($\["${${HotdeskExtension}\_PIN}" = ""\]?invalid\_user\)

 same =&gt; n,GotoIf\($\[${ODBCROWS} &lt; 0\]?invalid\_user\)

 same =&gt; n,GotoIf\($\[${${HotdeskExtension}\_STATUS} = 1\]?logout:login,1\)

We’ll be writing some labels to handle invalid\_user and logout a bit later, so don’t worry if it seems something is missing.

**Note**

You may have noticed that in some of the Goto/GotoIf examples, there might be a ,1 in the directive. This might seem confusing unless you recall that the target only needs the difference between the current context,extension,priority/label. So, if you send something to a label, such as logout, that is in the same extension, you don’t need to specify the context and extension, whereas if you are sending the call to the extension named login \(still in the same context\), you need to specify that you wish the call to go to label/priority 1. In the previous example, we could write our directive as follows:

... = 1\] ? hotdesk,${EXTEN},logout : hotdesk,login,1

 ^same ^same ^diff ^same ^diff ^diff

In other words, true goes to context \[hotdesk\], extension 99110\[1-5\], label logout; and false goes to context \[hotdesk\], extension login, and label/priority 1.

We only wrote what’s different.

If you want, for clarity, you can always write context,extension,priority for all your directives. It’s your call.

After assigning the value of the status column to the ${HotdeskExtension}\_STATUS variable \(if the user identifies themself as extension 1101, the variable name will be 1101\_STATUS\), we check if we’ve received a value back from the database using the ${ODBCROWS} channel variable.

The last row in the block checks the status of the phone and, if the agent is currently logged in, logs them off. If the agent is not already logged in, it will go to the login extension.

At the login extension the dialplan runs some initial checks to verify the PIN code entered by the agent. \(Additionally, we’ve used the FILTER\(\) function to make sure only numbers were entered to help avoid some SQL injection issues.\) We allow them three tries to enter the correct PIN, and if all tries are invalid we’ll hang up:

exten =&gt; login,1,NoOp\(\) ; set initial counter values

 same =&gt; n,Set\(PIN\_TRIES=1\) ; pin tries counter

 same =&gt; n,Set\(MAX\_PIN\_TRIES=3\) ; set max number of login attempts

 same =&gt; n,Playback\(silence/1\) ; play back some silence so first prompt is

 ; not cut off

 same =&gt; n\(get\_pin\),NoOp\(\)

 same =&gt; n,Set\(PIN\_TRIES=$\[${PIN\_TRIES} + 1\]\) ; increase pin try counter

 same =&gt; n,Read\(PIN\_ENTERED,enter-password,${LEN\(${${HotdeskExtension}\_PIN}\)}\)

 same =&gt; n,Set\(PIN\_ENTERED=${FILTER\(0-9,${PIN\_ENTERED}\)}\)

 same =&gt; n,GotoIf\($\["${PIN\_ENTERED}" = "${${HotdeskExtension}\_PIN}"\]?valid:invalid\)

 same =&gt; n,Hangup\(\)

 same =&gt; n\(invalid\),Playback\(vm-invalidpassword\)

 same =&gt; n,GotoIf\($\[${PIN\_TRIES} &lt;= ${MAX\_PIN\_TRIES}\]?get\_pin\)

 same =&gt; n,Playback\(goodbye\)

 same =&gt; n,Hangup\(\)

 same =&gt; n\(valid\),Noop\(Valid PIN\)

If the PIN entered matches, we continue with the login process through the \(valid\) label. First we utilize the CHANNEL variable to figure out which phone device the agent is calling from. The CHANNEL variable is usually populated with something similar to PJSIP/HOTDESK\_1-ab4034c, so we make use of the CUT\(\) function to first strip off the PJSIP/ component of the string. We then strip off the -ab4034c part of the string, and what remains is what we want \(HOTDESK\_1\):[5](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch15.html%22%20/l%20%22idm46178405135064)

 same =&gt; n\(valid\),Noop\(Valid PIN\)

; CUT off the channel technology and assign it to the LOCATION variable

 same =&gt; n,Set\(LOCATION=${CUT\(CHANNEL,/,2\)}\)

; CUT off the unique identifier and save the remainder to the LOCATION variable

 same =&gt; n,Set\(LOCATION=${CUT\(LOCATION,-,1\)}\)

; we'll come back to this shortly

We’re going to create and use some more functions in the func\_odbc.conf file: HOTDESK\_CHECK\_SET\(\), which will determine if other users are already assigned to this phone; HOTDESK\_STATUS\(\), which will assign the phone to this agent; and HOTDESK\_CLEAR\_SET\(\), which will clear any other users currently assigned to this phone \(who perhaps forgot to log out\).

In our func\_odbc.conf file we’ll need to create the following functions:

; func\_odbc.conf

\[CHECK\_SET\]

prefix=HOTDESK

dsn=asterisk

synopsis=Check if this set is already assigned to somebody.

readsql=SELECT COUNT\(status\) FROM pbx.ast\_hotdesk WHERE status = '1'

readsql+= AND endpoint = '${ARG1}'

\[STATUS\]

prefix=HOTDESK

dsn=asterisk

synopsis=Assign hotdesk extension to this endpoint/set.

writesql=UPDATE pbx.ast\_hotdesk SET status = '${SQL\_ESC\(${VAL1}\)}',

writesql+= endpoint = '${SQL\_ESC\(${VAL2}\)}'

writesql+= WHERE extension = '${SQL\_ESC\(${ARG1}\)}'

\[CLEAR\_SET\]

prefix=HOTDESK

dsn=asterisk

synopsis=Clear all instances of this endpoint

writesql= UPDATE pbx.ast\_hotdesk SET status=0,endpoint=NULL

writesql+= WHERE endpoint='${SQL\_ESC\(${VAL1}\)}'

**Tip**

Due to line-length limitations in the book, we’ve broken the readsql and writesql commands into multiple lines using the += syntax, which tells Asterisk to append the contents after readsql+= to the most recently defined readsql= value \(or writesql and writesql+\). The usage of += is applicable not only to the readsql option, but can be used in other places in other .conf files within Asterisk.

In our dialplan, we’ll need to call the function we just created, and pass call flow to the forcelogout label if somebody is already logged into this set:

 same =&gt; n\(valid\),Noop\(Valid PIN\)

 same =&gt; n,Set\(LOCATION=${CUT\(CHANNEL,/,2\)}\)

 same =&gt; n,Set\(LOCATION=${CUT\(LOCATION,-,1\)}\)

; We'll come back to this shortly ; you can remove this comment/line

 same =&gt; n\(checkset\),Set\(SET\_USED=${HOTDESK\_CHECK\_SET\(${LOCATION}\)}\)

 same =&gt; n,GotoIf\($\[${SET\_USED} &gt; 0\]?forcelogout\)

; Set status for agent to '1' and update the location/endpoint

 same =&gt; n\(set\_login\_status\),Set\(HOTDESK\_STATUS\(${HotdeskExtension}\)=1,${LOCATION}\)

 same =&gt; n,Noop\(ODBCROWS is ${ODBCROWS}\)

 same =&gt; n,GotoIf\($\[${ODBCROWS} &lt; 1\]?error,1\)

 same =&gt; n,Playback\(agent-loginok\)

 same =&gt; n,Hangup\(\)

 same =&gt; n\(forcelogout\),NoOp\(\)

; set all currently logged-in users on this device to logged-out status

 same =&gt; n,Set\(HOTDESK\_CLEAR\_SET\(\)=${LOCATION}\)

 same =&gt; n,Goto\(checkset\) ; return to logging in

There are some potentially new concepts we’ve just introduced in the examples. Specifically, the syntax in the HOTDESK\_STATUS\(\) function has a few new tricks you might have noticed. We now have both ${VALx} and ${ARGx} variables in our SQL statement.

**Note**

We’ve wrapped the ${VALx} and ${ARGx} values in the SQL\_ESC\(\) function as well, which will escape characters such as backticks that could be used in an SQL injection attack.

These contain the information we pass to the function from the dialplan. In this case, we have two VAL variables and a single ARG variable that were set from the dialplan via this statement:

same =&gt; n\(set\_login\_status\),Set\(HOTDESK\_STATUS\(${HotdeskExtension}\)=1,${LOCATION}\)

Notice the syntax is slightly different from that of the read-style function. This signals to Asterisk that you want to perform a write \(this is the same structural syntax as that used for other dialplan functions\).

We are including the value of the ${HotdeskExtension} variable in our call to the HOTDESK\_STATUS\(\) function \(which then becomes the ${ARG1} variable for that function in func\_odbc.conf\). However, we are also passing two values, '1' and ${LOCATION}. These will be associated in the function by the ${VAL1} and ${VAL2} variables, respectively.

#### Using SQL Directly in Your Dialplan

Some people would prefer to write their SQL statements in the dialplan directly, as opposed to crafting a custom function for each type of database transaction they might want to perform.

In theory, you could create just one function in func\_odbc.conf like this:

\[SQL\]

prefix=GENERIC

dsn=asterisk

readsql=${SQL\_ESC\(${ARG1}\)}

writesql=${SQL\_ESC\(${VALUE}\)} ; Whole value, un-parsed

Then, in your dialplan you could write pretty much any sort of SQL you wanted \(provided the ODBC connector could handle it, which has nothing to do with Asterisk\). That one function above would then submit whatever string you specified directly to the ODBC connection to your database.[6](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch15.html%22%20/l%20%22idm46178405097368)

Some would argue this makes for more confusion in your dialplan; others will insist that the benefit of having a much simpler func\_odbc.conf file is worth it:

 same =&gt; n,Set\(result=${GENERIC\_SQL\(SELECT col FROM table WHERE ...\)}\)

 same =&gt; n,Verbose\(1,${result}\)

 same =&gt; n,Set\(GENERIC\_SQL\(\)=UPDATE table SET field="VAL" WHERE ...\)

 same =&gt; n,Verbose\(1,ODBC\_RESULT is ${OBDBC\_RESULT}\)

We believe it’s generally better to build specific functions in func\_odbc.conf to handle queries from your dialplan; however, there’s no denying the temptation to use one function to handle all SQL queries.

#### Multirow Functionality with func\_odbc

Asterisk has a multirow mode that allows it to handle multiple rows of data returned from the database. For example, if we were to create a dialplan function in func\_odbc.conf that returned all available extensions, we would need to enable multirow mode for the function. This would cause the function to work a little differently, returning an ID number that could then be passed to the ODBC\_FETCH\(\) function to return each row in turn.

A simple example follows. Suppose we have the following func\_odbc.conf:

\[AVAILABLE\_EXTENS\]

prefix=HOTDESK

dsn=asterisk

mode=multirow

readsql=SELECT extension FROM ast\_hotdesk WHERE status = '${ARG1}'

and a dialplan in extensions.conf that looks something like this:

exten =&gt; \*9997,1,Noop\(multirow\)

 same =&gt; n,Set\(ODBC\_ID=${HOTDESK\_AVAILABLE\_EXTENS\(\)}\)

 same =&gt; n,GotoIf\($\[${ODBCROWS} &lt; 1\]?no\_rows\)

 same =&gt; n,Answer\(\)

 same =&gt; n,Set\(COUNTER=1\)

 same =&gt; n,While\($\[${COUNTER} &lt;= ${ODBCROWS}\]\)

 same =&gt; n,Set\(AVAIL\_EXTEN\_${COUNTER}=${ODBC\_FETCH\(${ODBC\_ID}\)}\)

 same =&gt; n,SayDigits\(${AVAIL\_EXTEN\_${COUNTER}}\)

 same =&gt; n,Wait\(0.2\) ; Pause between speaking

 same =&gt; n,Set\(COUNTER=$\[${COUNTER} + 1\]\)

 same =&gt; n,EndWhile\(\)

 same =&gt; n\(norows\),ODBCFinish\(\)

 same =&gt; n,Hangup\(\)

Note that unless you have multiple endpoints to log in, this will never return more than one extension in your lab because only one device will be logged in at any time. You can add some dummy data to the table just to see how this works:

MySQL&gt; UPDATE pbx.ast\_hotdesk

 SET status='1',endpoint='HOTDESK\_2'

 WHERE id='3'

 ;

MySQL&gt; UPDATE pbx.ast\_hotdesk

 SET status='1',endpoint='HOTDESK\_3'

 WHERE id='5'

 ;

The ODBC\_FETCH\(\) function will essentially treat the information as a stack, and each call to it with the passed ODBC\_ID will pop the next row of information off the stack. We also have the option of using the ODBC\_FETCH\_STATUS channel variable, which is set once the ODBC\_FETCH\(\) function \(which returns SUCCESS if additional rows are available or FAILURE if no additional rows are available\) is called. This permits us to write a dialplan like the following, which does not use a counter, but still loops through the data. This may be useful if we’re looking for something specific and don’t need to look at all the data. Once we’re done, the ODBCFinish\(\) dialplan application should be called to clean up any remaining data.

Here’s another extensions.conf example:

\[multirow\_example\_2\]

exten =&gt; start,1,Verbose\(1,Looping example with break\)

 same =&gt; n,Set\(ODBC\_ID=${GET\_ALL\_AVAIL\_EXTENS\(1\)}\)

 same =&gt; n\(loop\_start\),NoOp\(\)

 same =&gt; n,Set\(ROW\_RESULT=${ODBC\_FETCH\(${ODBC\_ID}\)}\)

 same =&gt; n,GotoIf\($\["${ODBC\_FETCH\_STATUS}" = "FAILURE"\]?cleanup,1\)

 same =&gt; n,GotoIf\($\["${ROW\_RESULT}" = "1104"\]?good\_exten,1\)

 same =&gt; n,Goto\(loop\_start\)

exten =&gt; cleanup,1,Verbose\(1,Cleaning up after all iterations\)

 same =&gt; n,Verbose\(1,We did not find the extension we wanted\)

 same =&gt; n,ODBCFinish\(${ODBC\_ID}\)

 same =&gt; n,Hangup\(\)

exten =&gt; good\_exten,1,Verbose\(1,Extension we want is available\)

 same =&gt; n,ODBCFinish\(${ODBC\_ID}\)

 same =&gt; n,Verbose\(1,Perform some action we wanted\)

 same =&gt; n,Hangup\(\)

OK, we’ve digressed a bit. Let’s wrap up a few parts of the agent components that we haven’t handled yet.

In the \_\*99110\[1-5\] extension, we need the following labels:

 same =&gt; n,GotoIf\($\[${${HotdeskExtension}\_STATUS} = 1\]?logout:login,1\)

 same =&gt; n\(invalid\_user\),Noop\(Hot Desk extension ${HotdeskExtension} does not exist\)

 same =&gt; n,Playback\(silence/2&login-fail\)

 same =&gt; n,Hangup\(\)

 same =&gt; n\(logout\),Noop\(\)

 same =&gt; n,Set\(HOTDESK\_STATUS\(${HotdeskExtension}\)=0,\) ; Note VAL2 is empty

 same =&gt; n,GotoIf\($\[${ODBCROWS} &lt; 1\]?error,1\)

 same =&gt; n,Playback\(silence/1&agent-loggedoff\)

 same =&gt; n,Hangup\(\)

We also include the hotdesk\_outbound context, which will handle our outgoing calls after we have logged the agent into the system:

include =&gt; hotdesk\_outbound ; this line can go anywhere in the \[hotdesk\] context

The \[hotdesk\_outbound\] context utilizes many of the same principles already discussed. This context uses a pattern match to catch any numbers dialed from the hot-desk phones. We first set our LOCATION variable using the CHANNEL variable, then determine which extension \(agent\) is logged into the system and assign that value to the WHO variable. If this variable is NULL, we reject the outgoing call. If it is not NULL, then we get the agent information using the HOTDESK\_INFO\(\) function and assign it to several CHANNEL variables.

include =&gt; hotdesk\_outbound

; put this code right below your \[hotdesk\] context

\[hotdesk\_outbound\]

exten =&gt; \_NXXXXXX.,1,NoOp\(\)

 same =&gt; n,Set\(LOCATION=${CUT\(CHANNEL,/,2\)}\)

 same =&gt; n,Set\(LOCATION=${CUT\(LOCATION,-,1\)}\)

 same =&gt; n\(checkset\),Set\(VALID\_AGENT=${HOTDESK\_CHECK\_SET\(${LOCATION}\)}\)

 same =&gt; n,Noop\(VALID\_AGENT is ${VALID\_AGENT}\)

 same =&gt; n,Set\(${CALLERID\(name\)}=${HOTDESK\_INFO\(cid\_name,${VALID\_AGENT}\)}\)

 same =&gt; n,Set\(${CALLERID\(num\)}=${HOTDESK\_INFO\(cid\_number,${VALID\_AGENT}\)}\)

 same =&gt; n,GotoIf\($\[${VALID\_AGENT} = 0\]?notallowed\) ; Nobody logged in--calls not allowed

 same =&gt; n,Dial\(${LOCAL}/${EXTEN}\) ; See the Outside Connectivity chapter

 same =&gt; n,Hangup\(\)

 same =&gt; n\(notallowed\),Playback\(sorry-cant-let-you-do-that2\)

 same =&gt; n,Hangup\(\)

If you are not logged in, the call will fail with a message. If you are logged in, the call will be passed to the Dial\(\) application \(which might also fail if you don’t have a carrier configured, but that’s something covered in earlier chapters, so we’re going to leave it as this for this section\).

There’s one last bit of dialplan required. We have built this complex environment that lets our agents log in and out, but there isn’t actually any way of calling them!

We’re going to fix that now, by doing four things:

1. We’re going to include the \[sets\] context in the \[hotdesk\] context, so that our agents can use the other parts of our dialplan.
2. We’re going to give our agents mailboxes.
3. We’re going to create a new subroutine that will check the hotdesk for an agent, and a\) ring them if they’re there, or b\) fire the call off to voicemail if they’re not.
4. We’re going to build dialplan in the \[sets\] context so that everyone can call our agents.

Let’s get the mailboxes out of the way first:

MySQL&gt; insert into \`asterisk\`.\`voicemail\`

\(mailbox,fullname,context,password\)

VALUES

\('1101','Herb Tarlek','default','110111'\),

\('1102','Al Bundy','default','110222'\),

\('1103','Willy Loman','default','110333'\),

\('1104','Jerry Lundegaard','default','110444'\),

\('1105','Moira Brown','default','110555'\);

All the rest of the work is in extensions.conf:

Way down at the bottom, let’s craft a subroutine that’ll handle things for us:

\[subDialHotdeskUser\]

exten =&gt; \_\[a-zA-Z0-9\].,1,Noop\(Call Hotdesk\)

 same =&gt; n,Set\(HOTDESK\_ENDPOINT=${HOTDESK\_INFO\(endpoint,${EXTEN}\)}\) ; Get assigned device

 same =&gt; n,GotoIf\($\["${HOTDESK\_ENDPOINT}" = ""\]?voicemail\) ; if blank, send to voicemail

 same =&gt; n\(ringhotdesk\),Dial\(PJSIP/${HOTDESK\_ENDPOINT},${ARG1}\)

 same =&gt; n\(voicemail\),Voicemail\(${EXTEN}\)

 same =&gt; n,Hangup\(\)

And somewhere far closer to the top, we’ll add our hotdesk users to the section of dialplan where our other users live:

exten =&gt; 110,1,Dial\(${UserA\_DeskPhone}&${UserA\_SoftPhone}&${UserB\_SoftPhone}\)

exten =&gt; 1101,1,GoSub\(subDialHotdeskUser,${EXTEN},1\(12\)\)

exten =&gt; 1102,1,GoSub\(subDialHotdeskUser,${EXTEN},1\(12\)\)

exten =&gt; 1103,1,GoSub\(subDialHotdeskUser,${EXTEN},1\(12\)\)

exten =&gt; 1104,1,GoSub\(subDialHotdeskUser,${EXTEN},1\(12\)\)

exten =&gt; 1105,1,GoSub\(subDialHotdeskUser,${EXTEN},1\(12\)\)

exten =&gt; 200,1,Answer\(\)

 same =&gt; n,Playback\(hello-world\)

 same =&gt; n,Hangup\(\)

And finally, back in our \[hotdesk\] context, we’re going to allow our agents to use the rest of the phone system:

\[hotdesk\]

include =&gt; sets

exten =&gt; \_\*99110\[1-5\],1,Noop\(Hotdesk login\)

Try a few scenarios:

1. Call from an agent internally.
2. Call from a normal user to a logged-in agent.
3. Call from a normal user to an unavailable agent.

Marvel at this technological terror you’ve constructed.

Now that we’ve implemented a fairly complex feature in the dialplan, using func\_odbc to retrieve and store data in a remote relational database, you can see that with a handful of fairly simple functions in the func\_odbc.conf file and a couple of tables in a database, you can create some powerful telephony applications.

OK, let’s move on to the Asterisk Realtime Architecture, which has in many cases been made obsolete by ODBC, but can still be useful.

## Using Realtime

The Asterisk Realtime Architecture \(ARA\) allows you to store all the parameters normally stored in your Asterisk configuration files \(commonly located in /etc/asterisk\) in a database. There are two types of realtime: static and dynamic.

The static version is similar to the traditional method of reading a configuration file \(information is only loaded when triggered from the CLI\), except that the data is read from the database instead.[7](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch15.html%22%20/l%20%22idm46178405034424)

The Dynamic Realtime method, which loads and updates the information as it is used by the live system, is commonly used for things such as SIP \(or IAX2, etc.\) user and peer objects, as well as voicemail boxes.

Making changes to static information requires a reload, just as if you had changed a text file on the system, but dynamic information is polled by Asterisk as needed, so no reload is required when changes are made to this data. Realtime is configured in the extconfig.conf file located in the /etc/asterisk directory. This file tells Asterisk what to load from the database and where to load it from, allowing certain files to be loaded from the database and other files to be loaded from the standard configuration files.

**Tip**

Another \(arguably older\) way to store Asterisk configuration was through an external script, which would interact with a database and generate the appropriate flat files \(or .conf files\), and then reload the appropriate module once the new file was written. There is an advantage to this \(if the database goes down, your system will continue to function; the script will simply not update any files until connectivity to the database is restored\), but it also has disadvantages. One major disadvantage is that any changes you make to a user will not be available until you run the update script. This is probably not a big issue on small systems, but on large systems, waiting for changes to take effect can cause issues, such as pausing a live call while a large file is loaded and parsed.

You can relieve some of this by utilizing a replicated database system. Asterisk provides the ability to fail over to another database system. This way, you can cluster the database backend utilizing a master-master relationship \(for PostgreSQL, [pgcluster](http://pgfoundry.org/projects/pgcluster/), or Postgre-R;[8](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch15.html%22%20/l%20%22idm46178405023496) for MySQL it’s native[9](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch15.html%22%20/l%20%22idm46178405022184)\), or a master-slave \(for PostgreSQL or [Slony-I](http://www.slony.info/); for MySQL it’s native\) replication system.

Our informal survey of such things suggests that using scripts to write flat files from databases is not as popular as querying a database in real time \(and ensuring the database has a proper amount of fault tolerance to handle the fact that a live telecom system is dependent on it\).

### Static Realtime

Static Realtime was one of the earliest ways that Asterisk configuration could be stored in a database. It is still somewhat useful for storing simple configuration files in a database \(which you might normally place in /etc/asterisk\). We don’t tend to use it much anymore because Dynamic Realtime is far better for larger sets of data, and the file-based configuration files are more than adequate for smaller configuration settings.

The same rules that apply to flat files on your system still apply when you’re using Static Realtime. For example, after making changes to the configuration you still have to run the module reload command for the relevant technology \(e.g., \*CLI&gt; module reload res\_musiconhold.so\).

When using Static Realtime, we tell Asterisk which files we want to load from the database using the following syntax in the extconfig.conf file:

; /etc/asterisk/extconfig.conf

\[settings\]

filename.conf =&gt; driver,database\[,table\]

**Note**

There is no configuration file called filename.conf. Instead, use the actual name of the configuration file you are storing in the database. If the table name is not specified, Asterisk will use the name of the file as the table name instead \(less the .conf part\). Also, all settings inside the extconfig.conf file should fall under the \[settings\] header. Be aware that you can’t load certain files from realtime at all, including asterisk.conf, extconfig.conf, and logger.conf.

The Static Realtime module uses a very specifically formatted table to allow Asterisk to read the various static files from the database. [Table 15-1](15.%20Relational%20Database%20Integration%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22Database_id243292) illustrates the columns as they must be defined in your database.

Table 15-1. Table layout and description of ast\_config

| Column name | Column type | Description |
| :--- | :--- | :--- |
| id | Serial, autoincrementing | An autoincrementing unique value for each row in the table. |
| cat\_metric | Integer | The weight of the category within the file. A lower metric means it appears higher in the file \(see the sidebar \). |
| var\_metric | Integer | The weight of an item within a category. A lower metric means it appears higher in the list \(see the sidebar \). This is useful for things like codec order in sip.conf, or iax.conf where you want disallow=all to appear first \(metric of 0\), followed by allow=ulaw \(metric of 1\), then allow=gsm \(metric of 2\). |
| filename | Varchar 128 | The filename the module would normally read from the hard drive of your system \(e.g., musiconhold.conf, sip.conf, iax.conf\). |
| category | Varchar 128 | The section name within the file, such as \[general\]. Do not include the square brackets around the name when saving to the database. |
| var\_name | Varchar 128 | The option on the left side of the equals sign \(e.g., disallow is the var\_name in disallow=all\). |
| var\_val | Varchar 128 | The value of an option on the right side of the equals sign \(e.g., all is the var\_val in disallow=all\). |
| commented | Integer | Any value other than 0 will evaluate as if it were prefixed with a semicolon in the flat file \(commented out\). |

#### A Word About Metrics

The metrics in Static Realtime are used to control the order in which objects are read into memory. Think of the cat\_metric and var\_metric as the original line numbers in the flat file. A higher cat\_metric is processed first, because Asterisk matches categories from bottom to top. Within a category, though, a lower var\_metric is processed first, because Asterisk processes the options top-down \(e.g., disallow=all should be set to a value lower than the allow’s value within a category to make sure it is processed first\).

There’s not much more to say about Static Realtime. It was very useful in the past, but has now been mostly superseded by Dynamic Realtime. If you want to read more about it, older versions of this book discuss it in more detail.

### Dynamic Realtime

The Dynamic Realtime system is used to load objects that may change often, such as PJSIP entities, queues and their members, and voicemail. Likewise, when new records are likely to be added on a regular basis, we can utilize the power of the database to let us load this information on an as-needed basis.

You have already worked extensively with Dynamic Realtime, since that is how we’ve been working for this entire book, both during installation, and in most of the examples we have worked through.

All of realtime is configured in the /etc/asterisk/extconfig.conf file; however, Dynamic Realtime has explicitly defined configuration names. All the predefined names should be configured under the \[settings\] header. For example, defining SIP peers is done using the following format:

; extconfig.conf

\[settings\]

sippeers =&gt; driver,database\[,table\]

The table name is optional. If it is omitted, Asterisk will use the predefined name \(i.e., sippeers\) to identify the table in which to look up the data.

The sample file ~/src/asterisk-15.&lt;TAB&gt;/configs/samples/extconfig.conf.sample contains excellent information about Dynamic Realtime.

## Storing Call Detail Records

Call detail records \(CDR\) contain information about calls that have passed through your Asterisk system. They are discussed further in [Chapter 21](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch21.html%22%20/l%20%22asterisk-Monitoring). Storing CDR is a popular use of databases in Asterisk, because it makes them easier to work with. Also, by placing records into a database you open up many possibilities, including building your own web interface for tracking statistics such as call usage and most-called locations, billing, or phone company invoice verification.

You should always implement CDR storage to a database on any production system \(you can always store CDR to a file as well, so there’s nothing lost\).

#### Setting the systemname for Globally Unique IDs

A CDR consists of a unique identifier and several fields of information about the call \(including the source and destination channel, length of call, last application executed, and so forth\). In a clustered set of Asterisk boxes, it is theoretically possible to have duplication among unique identifiers, since each Asterisk system considers only itself. To address this, we can automatically append a system identifier to the front of the unique IDs by adding an option to /etc/asterisk/asterisk.conf. For each of your boxes, set an identifier by adding something like:

\[options\]

systemname=toronto

The best way to store your call detail records is via the cdr\_adaptive\_odbc module. This module allows you to choose which columns of data built into Asterisk are stored in your table, and it permits you to add additional columns that can be populated with the CDR\(\) dialplan function. You can even store different parts of CDR data to different tables and databases, if that is required.

To create the table, we have Alembic. The process is almost identical to the one you performed during the system installation, except of course the .ini file is different.

$ cd ~/src/asterisk-15.&lt;TAB&gt;/contrib/ast-db-manage

$ cp cdr.ini.sample cdr.ini

$ egrep ^sqlalchemy config.ini

sqlalchemy.url = mysql://asterisk:YouNeedAReallyGoodPasswordHereToo@localhost/asterisk

The same credentials we used before will also work for CDR.

$ sudo vim cdr.ini

Add the line you just got back from grep to this file, and save.

$ alembic -c ./cdr.ini upgrade head

INFO \[alembic.runtime.setup\] Creating new alembic\_version\_cdr table.

INFO \[alembic.runtime.migration\] Running upgrade -&gt; 210693f3123d, Create CDR table.

INFO \[alembic.runtime.migration\] Running upgrade 210693f3123d -&gt; 54cde9847798

Alembic doesn’t do too much bragging, so the output is terse, but it appears to have completed successfully. Let’s check.

$ mysql -u asterisk -p

MySQL&gt; describe asterisk.cdr

You should get a list of all the fields in the table \(which means Alembic was successful\). If you get a message like Table 'asterisk.cdr' doesn't exist, that indicates Alembic didn’t complete the configuration, and you need to review the messages from the Alembic output to see what went wrong \(credentials is usually what causes grief here\).

Well, that wasn’t too hard, eh? The next step is to tell Asterisk to use this new table for CDR going forward.

$ sudo -u asterisk touch /etc/asterisk/cdr\_adaptive\_odbc.conf

$ sudo -u asterisk vim /etc/asterisk/cdr\_adaptive\_odbc.conf

Into this new file, paste the following:

\[adaptive\_connection\]

connection=asterisk

table=cdr

This is almost too easy, wouldn’t you say? Alrighty, now we just have to reload the ccdr\_adaptive\_odbc.so module in Asterisk:

$ sudo asterisk -rvvvvvvv

\*CLI&gt; module reload cdr\_adaptive\_odbc.so

You can verify that the Adaptive ODBC backend has been loaded by running the following:[10](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch15.html%22%20/l%20%22idm46178404930968)

\*CLI&gt; cdr show status

Call Detail Record \(CDR\) settings

----------------------------------

 Logging: Enabled

 Mode: Simple

 Log unanswered calls: No

 Log congestion: No

\* Registered Backends

 -------------------

 cdr-syslog

 Adaptive ODBC

 cdr-custom

 csv

 cdr\_manager

Now place a call that gets answered \(e.g., using Playback\(\), or Dial\(\)ing another channel and answering it\). You should get some CDRs stored into your database. You can check by running SELECT \* FROM CDR; from your database console.

With the basic CDR information stored in the database, you might want to add some additional information to the cdr table, such as the route rate. You can use the ALTER TABLE directive to add a column called route\_rate to the table:

sql&gt; ALTER TABLE cdr ADD COLUMN route\_rate varchar\(10\);

Now reload the cdr\_adaptive\_odbc.so module from the Asterisk console:

\*CLI&gt; module reload cdr\_adaptive\_odbc.so

and populate the new column from the Asterisk dialplan using the CDR\(\) function, like so:

exten =&gt; \_NXXNXXXXXX,1,Verbose\(1,Example of adaptive ODBC usage\)

 same =&gt; n,Set\(CDR\(route\_rate\)=0.01\)

 same =&gt; n,Dial\(SIP/my\_itsp/${EXTEN}\)

 same =&gt; n,Hangup\(\)

After the alteration to your database and dialplan, you can place a call and then look at your CDRs. You should see something like the following:

+--------------+----------+---------+------------+

\| src \| duration \| billsec \| route\_rate \|

+--------------+----------+---------+------------+

\| 0000FFFF0008 \| 37 \| 30 \| 0.01 \|

+--------------+----------+---------+------------+

In reality, storing rating in the call record might not be ideal \(CDR is typically used as a raw resource, and things such as rates are added downstream by billing software\). The ability to add custom fields to CDR is very useful, but be careful not to use your call records to replace a proper billing platform. Best to keep your CDR clean and do further processing downstream.

#### Additional Configuration Options for cdr\_adaptive\_odbc.conf

Some extra configuration options exist in the cdr\_adaptive\_odbc.conf file that may be useful. The first is that you can define multiple databases or tables to store information into, so if you have multiple databases that need the same information, you can simply define them in res\_odbc.conf, create tables in the databases, and then refer to them in separate sections of the configuration:

\[mysql\_connection\]

connection=asterisk\_mysql

table=cdr

\[mssql\_connection\]

connection=production\_mssql

table=call\_records

**Note**

If you specify multiple sections using the same connection and table, you will get duplicate records.

Beyond just configuring multiple connections and tables \(which of course may or may not contain the same information; the CDR module we’re using is adaptive to situations like that\), we can define aliases for the built-in variables, such as accountcode, src, dst, and billsec.

If we were to add aliases for column names for our MS SQL connection, we might alter our connection definition like so:

\[mssql\_connection\]

connection=production\_mssql

table=call\_records

alias src =&gt; Source

alias dst =&gt; Destination

alias accountcode =&gt; AccountCode

alias billsec =&gt; BillableTime

In some situations you may specify a connection where you only want to log calls from a specific source, or to a specific destination. We can do this with filters:

\[logging\_for\_device\_0000FFFF0008\]

connection=asterisk\_mysql

table=cdr\_for\_0000FFFF0008

filter src =&gt; 0000FFFF0008

If you need to populate a certain column with information based on a section name, you can set it statically with the static option, which you may utilize with the filter option:

\[mysql\_connection\]

connection=asterisk\_mysql

table=cdr

\[filtered\_mysql\_connection\]

connection=asterisk\_mysql

table=cdr

filter src =&gt; 0000FFFF0008

static "DoNotCharge" =&gt; accountcode

**Note**

In the preceding example, you will get duplicate records in the same table, but all the information will be the same except for the populated accountcode column, so you should be able to filter it out using SQL.

## Database Integration of ACD Queues

With a Call Center \(often referred to as ACD queues\), it can be very useful to be able to allow adjustment of queue parameters without having to edit and reload configuration files. Management of a call center can be a complex task, and allowing for simpler adjustment of parameters can make everyone’s life a whole lot easier.

The queues themselves we’ve already placed in the database in [Chapter 12](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch12.html%22%20/l%20%22asterisk-ACD). If, however, you also want to store dialplan parameters relating to your queues, the database can do that too.

### Storing Dialplan Parameters for a Queue in a Database

The dialplan application Queue\(\) allows for several parameters to be passed to it. The CLI command core show application Queue defines the following syntax:

\[Syntax\]

Queue\(queuename\[,options\[,URL\[,announceoverride\[,timeout\[,AGI\[,macro\[,gosub\[,

 rule\[,position\]\]\]\]\]\]\]\]\]\)

Since we’re storing our queue in a database, why not also store the parameters you wish to pass to the queue in a similar manner?[11](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch15.html%22%20/l%20%22idm46178404879016)

MySQL&gt; CREATE TABLE \`pbx\`.\`QueueDialplanParameters\` \(

 \`QueueDialplanParametersID\` mediumint\(8\) NOT NULL auto\_increment,

 \`Description\` varchar\(128\) NOT NULL,

 \`QueueID\` mediumint\(8\) unsigned NOT NULL COMMENT 'Pointer to asterisk.queues table',

 \`options\` varchar\(45\) default 'n',

 \`URL\` varchar\(256\) default NULL,

 \`announceoverride\` bit\(1\) default NULL,

 \`timeout\` varchar\(8\) default NULL,

 \`AGI\` varchar\(128\) default NULL,

 \`macro\` varchar\(128\) default NULL,

 \`gosub\` varchar\(128\) default NULL,

 \`rule\` varchar\(128\) default NULL,

 \`position\` tinyint\(4\) default NULL,

 \`queue\_tableName\` varchar\(128\) NOT NULL,

 PRIMARY KEY \(\`QueueDialplanParametersID\`\)

\);

Using func\_odbc, you can write a function that will return the dialplan parameters relevant to that queue:

\[QUEUE\_DETAILS\]

prefix=GET

dsn=asterisk

readsql=SELECT \* FROM pbx.QueueDialplanParameters

readsql+= WHERE QueueDialplanParametersID='${ARG1}'

Then pass those parameters to the Queue\(\) application as calls arrive:

exten =&gt; s,1,Verbose\(1,Call entering queue named ${SomeValidID\)

 same =&gt; n,Set\(QueueParameters=${GET\_QUEUE\_DETAILS\(SomeValidID\)}\)

 same =&gt; n,Queue\(${QueueParameters}\)

While somewhat more complicated to develop than just writing an appropriate dialplan, the advantage is that you will be able to manage a larger number of queues, with a wider variety of parameters, using dialplan that is flexible enough to handle any sort of parameters the queueing application in Asterisk accepts. For anything more than a very simple queue, we think you will find the use of a database for all this is well worth the effort.

### Writing queue\_log to Database

Finally, we can store our queue\_log to a database, which can make it easier for external applications to extract queue performance details from the system:

CREATE TABLE queue\_log \(

 id int\(10\) UNSIGNED NOT NULL AUTO\_INCREMENT,

 time char\(26\) default NULL,

 callid varchar\(32\) NOT NULL default '',

 queuename varchar\(32\) NOT NULL default '',

 agent varchar\(32\) NOT NULL default '',

 event varchar\(32\) NOT NULL default '',

 data1 varchar\(100\) NOT NULL default '',

 data2 varchar\(100\) NOT NULL default '',

 data3 varchar\(100\) NOT NULL default '',

 data4 varchar\(100\) NOT NULL default '',

 data5 varchar\(100\) NOT NULL default '',

 PRIMARY KEY \(\`id\`\)

\);

Edit your extconfig.conf file to refer to the queue\_log table:

\[settings\]

queue\_log =&gt; odbc,asterisk,queue\_log

A restart of Asterisk, and your queue will now log information to the database. As an example, logging an agent into the sales queue should produce something like this:

mysql&gt; select \* from queue\_log;

+----+----------------------------+----------------------+-----------+

\| id \| time \| callid \| queuename \|

+----+----------------------------+----------------------+-----------+

\| 1 \| 2013-01-22 15:07:49.772263 \| NONE \| NONE \|

\| 2 \| 2013-01-22 15:07:49.809028 \| toronto-1358885269.1 \| support \|

+----+----------------------------+----------------------+-----------+

+------------------+------------+-------+-------+-------+-------+-------+

\| agent \| event \| data1 \| data2 \| data3 \| data4 \| data5 \|

+------------------+------------+-------+-------+-------+-------+-------+

\| NONE \| QUEUESTART \| \| \| \| \| \|

\| SIP/0000FFFF0001 \| ADDMEMBER \| \| \| \| \| \|

+------------------+------------+-------+-------+-------+-------+-------+

If you’re developing any sort of external application that needs access to queue statistics, having the data stored in this manner will prove far superior to using the /var/log/asterisk/queue\_log file.

## Conclusion

In this chapter, you learned about several areas where Asterisk can integrate with a relational database. This is useful for systems where you need to start scaling by clustering multiple Asterisk boxes working with the same centralized information, or when you want to start building external applications to modify information without requiring a reload of the system \(i.e., not requiring the modification of flat files\).

[1](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch15.html%22%20/l%20%22idm46178405309816-marker) This was actually an issue one of the authors had while working on this book, and he found the flag column by looking at the statement logging during testing.

[2](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch15.html%22%20/l%20%22idm46178405291112-marker) And if you don’t know what a Dagwood is, that’s what Wikipedia is for. I am not that old.

[3](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch15.html%22%20/l%20%22idm46178405269208-marker) We’re using the IF\(\) SQL function to make sure we return a value of 0 or 1. This works on MySQL 5.1 or later. If it does not work on your SQL installation, you could also check the returned result in the dialplan using the IF\(\) function there.

[4](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch15.html%22%20/l%20%22idm46178405232168-marker) Note that in the first example user, we are assigning a status of 1 and a location, whereas for the second example user, we are not defining a value for these fields.

[5](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch15.html%22%20/l%20%22idm46178405135064-marker) Yes, you can nest functions within functions, and so do this all on one line. We didn’t do so as it’s more difficult to debug, and doesn’t affect performance.

[6](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch15.html%22%20/l%20%22idm46178405097368-marker) It could also pose a needless security risk.

[7](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch15.html%22%20/l%20%22idm46178405034424-marker) Yes, calling this “realtime” is somewhat misleading, as updates to the data will not affect anything happening in real time \(until a reload of the relevant module is performed\).

[8](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch15.html%22%20/l%20%22idm46178405023496-marker) pgcluster appears to be a dead project, and Postgres-R appears to be in its infancy, so there may currently be no good solution for master-master replication using PostgreSQL.

[9](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch15.html%22%20/l%20%22idm46178405022184-marker) There are several tutorials on the web describing how to set up replication with MySQL.

[10](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch15.html%22%20/l%20%22idm46178404930968-marker) You may see different backends registered, depending on what configuration you have done with other components of the various CDR modules.

[11](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch15.html%22%20/l%20%22idm46178404879016-marker) Note that we’re creating this table in our pbx schema, rather than the asterisk schema, and that is because this is not a table that comes with Asterisk, but instead one we’re creating ourselves. We recommend letting Asterisk and Alembic have exclusive control over the asterisk schema, and using a custom schema \(such as pbx\) for anything custom we might create.
