# Предисловие

Это книга для всех, кто использует Asterisk.

Asterisk - это платформа конвергентной телефонии с открытым исходным кодом, которая предназначена в первую очередь для работы на Linux. Asterisk объединяет более чем 100-летние знания в области телефонии в надежный набор тесно интегрированных телекоммуникационных приложений. Сила Asterisk заключается в его настраиваемой природе, дополненной непревзойденным соответствием стандартам. Ни одна другая управленческая автоматизированная телефонная станция \(УАТС\) не может быть развернута таким множеством творческих способов.

Такие приложения, как голосовая почта, конференции, очереди вызовов и агенты, музыка на удержании и парковка вызовов - все это стандартные функции, встроенные прямо в программное обеспечение. Кроме того, Asterisk может интегрироваться с другими бизнес-технологиями таким образом, о котором закрытые, проприетарные УАТС вряд ли могут мечтать.

Asterisk может показаться довольно пугающим и сложным для нового пользователя, поэтому документация так важна для его роста. Документация снижает барьер для входа и помогает людям созерцать возможности.

Выпущенный при щедрой поддержке O'Reilly Media, [Asterisk: Окончательное руководство](http://shop.oreilly.com/product/0636920025894.do) - это пятое издание того, что ранее называлось [Asterisk: Будущее телефонии](http://shop.oreilly.com/product/9780596510480.do).

Эта книга была написана для участников сообщества Asterisk.

## АудиторияAudience

Эта книга предназначена для того, чтобы быть нежной к тем, кто новичок в Asterisk, но мы предполагаем, что вы знакомы с базовым администрированием Linux, сетью и другими ИТ-дисциплинами. Если нет, мы рекомендуем вам изучить обширную и замечательную библиотеку книг, которые O'Reilly публикует по этим темам. Мы также предполагаем, что вы пока новичок в телекоммуникациях \(как традиционная коммутируемая Телефония, так и новый мир Voice over IP\).

Однако эта книга будет полезна и более опытному администратору Asterisk. Мы сами используем книгу в качестве ссылки на функции, которые мы не использовали в течение некоторого времени.

## Программное обеспечение

Эта книга сосредоточена на документировании версии 16 Asterisk; однако многие соглашения и большая часть информации в этой книге являются агностическими версиями. Linux - это операционная система, в которой мы запускали и тестировали Asterisk, и мы задокументировали инструкции по установке для CentOS \(Red Hat Enterprise Linux или RHEL\).

## Условные обозначения используемые в книге

В этой книге используются следующие типографские условные обозначения: 

_Курсив_

Обозначает новые термины, URL-адреса, адреса электронной почты, имена файлов, расширения файлов, пути, каталоги и имена пакетов, а также утилиты Unix, команды, модули, параметры и аргументы.

`Моноширинный`

Используется для отображения примеров кода, содержимого файлов, взаимодействий командной строки, команд базы данных, имен библиотек и параметров.

**`Моноширинный полужирный`**

Обозначает команды или другой текст, который должен быть набран буквально пользователем. Также используется для акцентирования в коде.

_`Моноширинный курсив`_

Показывает текст, который должен быть заменен пользовательскими значениями.

_`[ Keywords and other stuff ]`_

Указывает необязательные ключевые слова и аргументы.

_`{ вариант-1 | вариант-2 }`_

Означает _`вариант-1`_ или _`вариант-2`_.

{% hint style="info" %}
**Подсказка**

Этот элемент означает подсказку или предложение.
{% endhint %}

{% hint style="success" %}
**Примечания**

Этот элемент обозначает общую заметку.
{% endhint %}

{% hint style="warning" %}
**Предупреждение**

Этот элемент указывает на предупреждение или предостережение.
{% endhint %}

## O’Reilly Online Learning

**Note**

For almost 40 years, [O’Reilly Media](http://oreilly.com/) has provided technology and business training, knowledge, and insight to help companies succeed.

 Our unique network of experts and innovators share their knowledge and expertise through books, articles, conferences, and our online learning platform. O’Reilly’s online learning platform gives you on-demand access to live training courses, in-depth learning paths, interactive coding environments, and a vast collection of text and video from O’Reilly and 200+ other publishers. For more information, please visit [_http://oreilly.com_](http://www.oreilly.com/).

## How to Contact Us

Please address comments and questions concerning this book to the publisher:

* O’Reilly Media, Inc.
* 1005 Gravenstein Highway North
* Sebastopol, CA 95472
* 800-998-9938 \(in the United States or Canada\)
* 707-829-0515 \(international or local\)
* 707-829-0104 \(fax\)

We have a web page for this book, where we list errata, examples, and any additional information. You can access this page at [_https://oreil.ly/asterisk\_tdg\_5E_](https://oreil.ly/asterisk_tdg_5E).

To comment or ask technical questions about this book, send email to [_bookquestions@oreilly.com_](mailto:bookquestions@oreilly.com).

For more information about our books, courses, conferences, and news, see our website at [_http://www.oreilly.com_](http://www.oreilly.com/).

Find us on Facebook: [_http://facebook.com/oreilly_](http://facebook.com/oreilly)

Follow us on Twitter: [_http://twitter.com/oreillymedia_](http://twitter.com/oreillymedia)

Watch us on YouTube: [_http://www.youtube.com/oreillymedia_](http://www.youtube.com/oreillymedia)

## Acknowledgments from Jim Van Meggelen

To David Duffett, thanks for the chapter on internationalization, which properly looks at this technology from a more global perspective.

Thanks to Leif Madsen, Jared Smith, and Russell Bryant, for your contributions to the previous editions of this book. It was fun flying solo, but I can’t deny I missed you guys!

Specific thanks to Matt Fredrickson and Matt Jordan of Digium, who generously shared their time and knowledge with me, and without whom I would have been lost. Thanks guys!

Thanks to my editor, Jeff Bleiel, for keeping me on track and helping me make important decisions about the content and pacing of the book.

Also thanks to the rest of the unsung heroes in O’Reilly’s production department. These are the folks that take a book and make it an _O’Reilly book_.

Thanks especially to Joyce Wilmot and Dan Jenkins, my technical review team, for taking the time to work through the book and provide essential feedback.

Thomas Cameron of RedHat generously shared his knowledge of Selinux with me, and helped to demystify a Linux component that is too often left disabled.

Everyone in the Asterisk community also needs to thank the late Jim Dixon for creating the first open source telephony hardware interfaces, starting the revolution, and giving his creations to the community at large.

Finally, and most importantly, thanks go to Mark Spencer, the original author of Asterisk and founder of Digium, for Asterisk, for [Pidgin](http://www.pidgin.im/), and for contributing his creations to the open source community. Asterisk is your legacy!

