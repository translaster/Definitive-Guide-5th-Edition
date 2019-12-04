# Глава 16. Введение в интерактивное голосовое меню

One day Alice came to a fork in the road and saw a Cheshire cat in a tree. “Which road do I take?” she asked.

“Where do you want to go?” was his response.

“I don’t know,” Alice answered.

“Then,” said the cat, “it doesn’t matter.”

Lewis Carroll

The term Interactive Voice Response \(IVR\) is often misused to refer to an automated voice attendant, but the two are very different things. The purpose of an IVR system is to take input from a caller, perform an action based on that input \(commonly, looking up data in an external system such as a database\), and speak a result to the caller. The purpose of an automated attendant \(which we covered in [Chapter 14](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch14.html%22%20/l%20%22asterisk-AA)\) is to route calls. Originally, an IVR didn’t even need to be a telephone system. Anything that took input from a human and spoke back a result fell with the realm of an IVR. Traditionally, IVR systems have been complex, expensive, and annoying to implement. Asterisk changes all that.

## Components of an IVR

The most basic elements of an IVR are quite similar to those of an automated attendant, though the goal is different. We need at least one prompt to tell the caller what the IVR expects, a method of receiving input from the caller, logic to verify that the caller’s response is valid input, logic to determine what the next step of the IVR should be, and finally, a storage mechanism for the responses, if applicable. We might think of an IVR as a decision tree, although it need not have any branches. For example, a survey may present exactly the same set of prompts to each caller, regardless of what choices the callers make, and the only routing logic involved is whether the responses given are valid for the questions.

From the caller’s perspective, every IVR needs to start with a prompt. This initial prompt will tell the caller what the IVR is for and ask the caller to provide the first input. We discussed prompts in the automated attendant in [Chapter 14](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch14.html%22%20/l%20%22asterisk-AA). Later, we’ll create a dialplan that will allow you to better manage multiple voice prompts.

The second component of an IVR is a method for receiving input from the caller. Recall that in [Chapter 14](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch14.html%22%20/l%20%22asterisk-AA) we discussed the Background\(\) and WaitExten\(\) applications for receiving a new extension. While you could create an IVR using Background\(\) and WaitExten\(\), it is generally easier and more practical to use the Read\(\) application, which handles both the prompt and the capture of the response. The Read\(\) application was designed specifically for use with IVR systems. Its syntax is as follows:

Read\(variable\[,filename\[&filename2...\]\]\[,maxdigits\]\[,option\]\[,attempts\]\[,timeout\]\)

The arguments are described in [Table 16-1](16.%20Introduction%20to%20Interactive%20Voice%20Response%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22IVR_id245719).

Table 16-1. The Read\(\) application

<table>
  <thead>
    <tr>
      <th style="text-align:left">Argument</th>
      <th style="text-align:left">Purpose</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">variable</td>
      <td style="text-align:left">The variable into which the caller&#x2019;s response is stored. It is
        best practice to give each variable in your IVR a name that is similar
        to the prompt associated with that variable. This will help later if, for
        business reasons or ease of use, you need to reorder the steps of the IVR.
        Naming your variables var1, var2, etc., may seem easy in the short term,
        but later in your life cycle it will make fixing bugs more difficult.</td>
    </tr>
    <tr>
      <td style="text-align:left">prompt</td>
      <td style="text-align:left">A file (or list of files, joined together with the &amp; character) to
        play for the caller, requesting input. Remember to omit the format extension
        on the end of each filename.</td>
    </tr>
    <tr>
      <td style="text-align:left">maxdigits</td>
      <td style="text-align:left">The maximum number of characters to allow as input. In the case of yes/no
        and multiple-choice questions, it&#x2019;s best practice to limit this
        value to 1. In the case of longer lengths, the caller may always terminate
        input by pressing the # key.</td>
    </tr>
    <tr>
      <td style="text-align:left">options</td>
      <td style="text-align:left">
        <p>s (skip)</p>
        <p>Exit immediately if the channel has not been answered.</p>
        <p>i (indication)</p>
        <p>Rather than playing a prompt, play an indication tone of some sort (such
          as the dialtone).</p>
        <p>n (no answer)</p>
        <p>Read digits from the caller, even if the line is not yet answered.</p>
        <p>attempts</p>
        <p>The number of times to play the prompt. If the caller fails to enter anything,
          the Read() application can automatically reprompt the user. The default
          is one attempt.</p>
        <p>timeout</p>
        <p>The number of seconds the caller has to enter his input. The default value
          in Asterisk is 10 seconds, although it can be altered for a single prompt
          using this option, or for the entire session by assigning a value using
          the dialplan function TIMEOUT(response).</p>
      </td>
    </tr>
  </tbody>
</table>Once the input is received, it must be validated. If you do not validate the input, you are more likely to find your callers complaining of an unstable application. It is not enough to handle the inputs you are expecting; you also need to handle inputs you do not expect. For example, callers may get frustrated and dial 0 when in your IVR; if you’ve done a good job, you will handle this gracefully and connect them to somebody who can help them, or provide a useful alternative. A well-designed IVR \(just like any program\) will try to anticipate every possible input and provide mechanisms to gracefully handle that input.

Once the input is validated, you can submit it to an external resource for processing. This could be done via a database query, a submission to a URI, an AGI program, or many other things. This external application should produce a result, which you will want to relay back to the caller. This could be a detailed result, such as “Your account balance is…” or a simple confirmation, such as “Your account has been updated.” We can’t think of any real-world case where some sort of result returned to the caller is not required.

Sometimes the IVR may have multiple steps, and therefore a result might include a request for more information from the caller in order to move to the next step of the IVR application.

It is possible to design very complex IVR systems, with dozens or even hundreds of possible paths. We’ve said it before and we’ll say it again: people don’t like talking to your phone system, regardless of how clever it is. Keep your IVR simple for your callers, and they are much more likely to get some benefit from it.

#### A Perfectly Tasty IVR

An excellent example of an IVR that people love to use is one that many pizza delivery outfits use: when you call to place your order, an IVR looks up your caller ID and says “If you would like the exact same order as last time, press 1.”

That’s all it does, and it’s perfect.

Obviously, these companies could design massively complex IVRs that would allow you to select each and every detail of your pie \(“for seven-grain crust, press 7”\), but how many inebriated, starving customers could successfully navigate something like that at 3 A.M.?

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
* Bother building an IVR if you can’t take numeric or spoken input. Nobody wants to have to spell their name on the dialpad of a phone.[1](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch16.html%22%20/l%20%22idm46178404791016)
* Force your callers to listen to advertising. Remember that they can hang up at any moment they wish.

## Asterisk Modules for Building IVRs

The “frontend” of the IVR \(the parts that interact with the callers\) can be handled in the dialplan. It is possible to build an IVR system using the dialplan alone \(perhaps with the astdb to store and retrieve data\); however, you will typically need to communicate with something external to Asterisk \(the “backend” of the IVR\).

### CURL\(\)

The CURL\(\) dialplan function in Asterisk allows you to span entire web applications with a single line of dialplan code. We’ll use it in our sample IVR later in this chapter.

While you’ll find CURL\(\) itself to be quite simple to use, the creation of the web application will require experience with web development.

### func\_odbc

Using func\_odbc, it is possible to develop extremely complex applications in Asterisk using nothing more than dialplan code and database lookups. If you are not a strong programmer but are very adept with Asterisk dialplans and databases, you’ll love func\_odbc just as much as we do. Check it out in [Chapter 15](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch15.html%22%20/l%20%22asterisk-DB).

### AGI

The Asterisk Gateway Interface is such an important part of integrating external applications with Asterisk that we gave it its own chapter. You can find more information in [Chapter 18](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch18.html%22%20/l%20%22AGI).

### AMI

The Asterisk Manager Interface is a socket interface that you can use to get configuration and status information, request actions to be performed, and be notified about things happening to calls. We’ve written an entire chapter on AMI as well. You can find more information in [Chapter 17](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch17.html%22%20/l%20%22asterisk-AMI).

### ARI

Asterisk’s REST interface builds on knowledge gained over the years about how to integrate Asterisk with current-generation web-centric applications. It is so important, that yes, once again, there is a complete chapter dedicated to it. If you’re looking to build complex IVRs using Asterisk, take a closer look at ARI in [Chapter 19](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch19.html%22%20/l%20%22asteriskRESTch).

## A Simple IVR Using CURL\(\)

Before we go running off writing an external program to handle something, we always give some careful thought about whether there’s a way to do the work in the dialplan. One powerful way that Asterisk can interact with external data is through a URL, which the GNU/Linux program cURL does very well. In Asterisk, CURL\(\) is a dialplan function.

We’re going to use CURL\(\) as an example of what an extremely simple IVR can look like. We’re going to request our external IP address from [https://ipinfo.io/ip](https://ipinfo.io/ip).[2](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch16.html%22%20/l%20%22idm46178404755736)

**Note**

In reality, most IVR applications are going to be far more complex. Even most uses of CURL\(\) will tend to be complex, since a URI can return a massive and highly variable amount of data, the vast majority of which will be incomprehensible to Asterisk. The point being that an IVR is not just about the dialplan; it is also very much about the external applications that are triggered by the dialplan, which are doing the real work of the IVR.

The CURL\(\) module was installed during our installation process several chapters ago.

### The Dialplan

The dialplan for our example IVR is very simple. The CURL\(\) function will retrieve our IP address from [https://ipinfo.io/ip](https://ipinfo.io/ip), and then SayAlpha\(\) will speak the results to the caller:

exten =&gt; \*764,1,Verbose\(2, Run CURL to get IP address from whatismyip.org\)

 same =&gt; n,Answer\(\)

 same =&gt; n,Set\(MyIPAddressIs=${CURL\(https://ipinfo.io/ip\)}\)

 same =&gt; n,SayAlpha\(${MyIPAddressIs}\)

 same =&gt; n,Hangup\(\)

The simplicity of this is impossibly cool. In a traditional IVR system, this sort of thing could take days to program, assuming it would be possible at all.

## A Prompt-Recording IVR Function

In [Chapter 14](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch14.html%22%20/l%20%22asterisk-AA), we created a simple bit of dialplan to record prompts. It was fairly limited in that it only recorded one filename, and thus for each prompt a separate extension was needed. Here, we expand upon that to create a complete menu for recording prompts. Since this is a complex bit of dialplan, but it’s not a subroutine or a local channel, we’re going to create a new section of dialplan for various features, and put things like this there:

;FEATURES

\[prompts\]

exten =&gt; s,1,Answer

exten =&gt; s,n,Set\(step1count=0\) ; Initialize counters

; If we get no response after 3 times, we stop asking

 same =&gt; n\(beginning\),GotoIf\($\[${step1count} &gt; 2\]?end\)

 same =&gt; n,Read\(which,prompt-instructions,3\)

 same =&gt; n,Set\(step1count=$\[${step1count} + 1\]\)

; All prompts must be 3 digits in length

 same =&gt; n,GotoIf\($\[${LEN\(${which}\)} != 3\]?beginning\)

 same =&gt; n,Set\(step1count=0\) ; Successful response; reset counters

 same =&gt; n,Set\(step2count=0\)

 same =&gt; n\(step2\),Set\(step2count=$\[${step2count} + 1\]\)

 same =&gt; n,GotoIf\($\[${step2count} &gt; 2\]?beginning\) ; No response after 3 tries

; If the file doesn't exist, then don't ask whether to play it

 same =&gt; n,GotoIf\($\[${STAT\(f,/var/lib/asterisk/sounds/${which}.wav\)} = 0\]?recordonly\)

 same =&gt; n,Background\(prompt-tolisten\)

 same =&gt; n\(recordonly\),Background\(prompt-torecord\)

 same =&gt; n,WaitExten\(10\) ; Wait 10 seconds for a response

 same =&gt; n,Goto\(step2\)

 same =&gt; n\(end\),Playback\(goodbye\)

 same =&gt; n,Hangup\(\)

exten =&gt; 1,1,Set\(step2count=0\)

 same =&gt; n,Background\(/var/lib/asterisk/sounds/${which}\)

 same =&gt; n,Goto\(s,step2\)

exten =&gt; 2,1,Set\(step2count=0\)

 same =&gt; n,Playback\(prompt-waitforbeep\)

 same =&gt; n,Record\(${CHANNEL\(uniqueid\)}.wav\)

 same =&gt; n\(listen\),Playback\(${CHANNEL\(uniqueid\)}\)

 same =&gt; n,Set\(step3count=0\)

 same =&gt; n,Read\(saveornot,prompt-1tolisten-2tosave-3todiscard,1,,2,3\)

 same =&gt; n,GotoIf\($\["${saveornot}" = "1"\]?listen\)

 same =&gt; n,GotoIf\($\["${saveornot}" = "2"\]?saveit\)

 same =&gt; n,GotoIf\($\["${saveornot}" = "3"\]?tossit\)

 same =&gt; n,Goto\(listen\)

 same =&gt; n\(tossit\),System\(rm -f /var/lib/asterisk/sounds/${CHANNEL\(uniqueid\)}.wav\)

 same =&gt; n,Goto\(s,beginning\)

 same =&gt; n\(saveit\),Noop\('Set' app used to shorten example\)

 same =&gt; n,Set\(PromptToSave=/var/lib/asterisk/sounds/${CHANNEL\(uniqueid\)}.wav

 same =&gt; n,Set\(WhereToSave=/var/lib/asterisk/sounds/${which}.wav

 same =&gt; n,System\(mv -f ${PromptToSave} ${WhereToSave}\)

 same =&gt; n,Playback\(prompt-saved\)

 same =&gt; n,Goto\(s,beginning\)

In this system, the name of the prompt is no longer descriptive; instead, it is a number. This means that you can record a far greater variety of prompts using the same mechanism, but the trade-off is that your prompts will no longer have descriptive names.

If you want to test this, you’ll need to record the prompts this IVR function uses \(it’s kinda meta, but yup, our prompt creator needs prompts\).

Drop this into your dialplan:

exten =&gt; 510,1,GoSub\(subRecordPrompt,${EXTEN},1\(prompt-tolisten\)\) ; press 1

exten =&gt; 511,1,GoSub\(subRecordPrompt,${EXTEN},1\(prompt-torecord\)\) ; press 2

exten =&gt; 512,1,GoSub\(subRecordPrompt,${EXTEN},1\(prompt-instructions\)\) ;3-digit \(000 to 999\)

exten =&gt; 513,1,GoSub\(subRecordPrompt,${EXTEN},1\(prompt-waitforbeep\)\) ; wait for beep

exten =&gt; 514,1,GoSub\(subRecordPrompt,${EXTEN},1\(prompt-1tolisten-2tosave-3todiscard\)\)

exten =&gt; 515,1,GoSub\(subRecordPrompt,${EXTEN},1\(prompt-saved\)\)

Then phone them one by one and record as required.

Once you’ve recorded the prompts your prompt recorder needs, you should be able to test it out.

exten =&gt; \*742,1,Noop\(Prompts\)

 same =&gt; n,Goto\(prompts,s,1\)

 same =&gt; n,Hangup\(\)

From this point forward, you can record prompts using just a numeric identifier. You’ll need a way to keep track of what prompt says what, but from a recording perspective you shouldn’t need to write more dialplan every time you need a prompt.

## Speech Recognition and Text-to-Speech

Although traditionally and still in most cases today, an IVR system presents prerecorded prompts to the caller and accepts input by way of the dialpad, it is also possible to: a\) generate prompts artificially, popularly known as text-to-speech; and b\) accept verbal inputs through a speech recognition engine.

While the concept of being able to have an intelligent conversation with a machine is something sci-fi authors have been promising us for many long years, the actual science of this remains complex and error-prone. Despite their amazing capabilities, computers are ill-suited to the task of appreciating the subtle nuances of human speech.

Having said that, it should be noted that companies such as Google have achieved amazing advances in both text-to-speech and speech recognition. There are now APIs available that can do a remarkable job of making sense out of what is being said to them. Google of course benefits from having a massive backend that can perform near-miraculous feats of processing; something your IVR might not be able to fully harness.

### Text-to-Speech

Text-to-speech \(also known as speech synthesis\) requires that a system be able to artificially construct speech from stored data. While it would be nice if we could simply assign a sound to a letter and have the computer produce each sound as it reads the letters, written language is often not phonetic, and seldom reflects the nuances of speech \(English is arguably one of the worst languages in this regard\).

There are now excellent APIs available from Google \(and others\), that will do a very good job of reading back what has been written. As of this writing, it’s still very obvious that it’s a computer speaking, but it is nevertheless possible to generate system prompts on the fly from text, rather than having to record all prompts in advance. The usefulness of this is difficult to evaluate, since humans are still not interested in talking to your machines; they phoned because they want to talk to you.

### Speech Recognition

Since we’ve managed to convince our computers to talk to us, we naturally want to be able to talk to them as well.[3](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch16.html%22%20/l%20%22idm46178404720856)

Speech recognition was previously complex and expensive, but Google has recently released an API that allows the enormous power of their speech recognition capabilities to be available to external applications.

## Conclusion

Asterisk is an excellent IVR platform. This entire book, in many ways, is teaching you skills that can be applied to IVR development. While the mainstream media only really pays attention to Asterisk as a “free PBX,” the reality is that Asterisk is at its most potent when used as an IVR. Within any respectable-sized organization, it is very likely that the Linux system administrators are using Asterisk to solve telecom problems that previously were either unsolvable or impossibly expensive to solve. This is a stealthy revolution, but no less significant for its relative obscurity.

If you are in the IVR business, you need to get to know Asterisk.

[1](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch16.html%22%20/l%20%22idm46178404791016-marker) Especially if it’s something like Van Meggelen.

[2](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch16.html%22%20/l%20%22idm46178404755736-marker) These free IP lookup websites seem to get bought out all the time, and turned into advertising gateways, so what might have worked at this writing may no longer work. What you need is a website that will return your IP address, and nothing else. Today, that seems to be [https://ipinfo.io/ip](https://ipinfo.io/ip). By the time you read this, it may be something else.

[3](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch16.html%22%20/l%20%22idm46178404720856-marker) Actually, most of us talk to our computers, but this is seldom polite.
