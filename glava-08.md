---
description: Голосовая почта
---

# Глава 8. Голосовая почта

> _Просто оставьте сообщение, может быть, я перезвоню._ 
>
> -- Джо Уолш

До того как почта и мгновенные сообщения стали широко распространены, голосовая почта была очень популярна. Даже теперь, когда большинство людей предпочитает обмениваться текстовыми сообщениями, голосовая почта является важным компонентом любой телефонной станции.

В Астериск есть достаточно гибкая система голосовой почты называемая Comedian Mail \[^1\]. В диалплане она реализуется посредством модуля app\_voicemail.so.

#### Предупреждение о модуле голосовой почты в Астериск

Модуль app\_voicemail один из старейших в Астериск, как следствие он имеет много ограничений, особенно если сравнивать его с другими, постоянно усовершенствуемыми модулями. Код модуля настолько устарел, что ни у кого не возникает желания с ним разбираться, и поэтому очень маловероятно появление в нем новых функций. Вы должны понимать что app\_voicemail не просто предоставляет элемент диалплана; для работы голосовой почты должно произойти много событий, например хранение и управление файлами, взаимодействие с почтовой системой операционной системы, распознавание временных зон, работы с форматами файлов, вопросы безопасности и еще куча вещей. И хотя app\_voicemail все это делает, в итоге получается довольно неуклюжая подсистема \(стоит заметить, что в традиционных АТС  для голосовой почты выделяется отдельная машина\).

 Было предпринято множество попыток реинжинировать голосовую почту, но все они оказались неудачными. Причина проста: объем работ \(а следовательно и стоимость\), необходимых для перепроектирования модуля \(таким образом, чтобы удовлетворить потребности сообщества\), в сочетании с отсутствием интереса к технологии голосовой почты в целом, быстро убивали любую инициативу.

Тем не менее важно отметить, что голосовая почта в Астериск работает, и работает хорошо. Возможно, она даже удовлетворит ваши потребности. В ином случае сообщество будет более чем благодарно, если вы попытаетесь перепроектировать её.

Вот некоторые функции, которые включает в себя модуль голосовой почты:

* Неограниченное количество защищенных паролем ящиков голосовой почты, каждый из которых содержит подпапки для сортировки голосовой почты
* Различные  приветствия для различных статусов, таких как "недоступен" или "занят"
* Наличие  предустановленных приветствий и возможность создания собственных
* Возможность ассоциирования телефонов с несколькими  почтовыми ящиками и почтового ящика с несколькими телефонами
* Уведомление о голсовом сообщении на электронную почту,  опционально с прикрепленным  аудио файлом
* Широковещательная голосовая почта и перенаправление голосовой почты
* Индикатор ожидания сообщения  \(мигающий светодиод или специальный сигнал\) поддерживаемый на многих типах телефонов
* Справочник сотрудников на основе голосовой почты

Теперь давайте познакомимся с основными частями конфигурационного файла голосовой почты, включая настройки в разделе Общие, различными возможными региональными настройками, интеграцией голосовой почты в ваш dialplan и проведем краткий обзор того, как Asterisk хранит голосовую почту в файловой системе Linux.

## Файл конфигурации voicemail.conf

Ранее в базе данных MySQL мы установили таблицу, необходимую для голосовой почты, и поэтому сейчас мы  можем создавать в ней почтовые ящики без какой-либо другой конфигурации. Однако, также можно создавать почтовые ящики в  конфигурационном файле /etc/asterisk/voicemail.conf \(в том числе в этом файле можно изменять различные настройки по умолчанию\). Мы продолжим использовать базу данных для создания пользователей и управления ими, поскольку она гораздо лучше подходит для этой задачи, но также  исследуем конфигурационный файл,  чтобы вы могли почувствовать гибкость настройки голосовой почты Asterisk.

Файл voicemail.conf содержит несколько секций, в которых могут быть переопределены различные предустановленные параметры. В большинстве случаев вам не понадобится их менять; однако  вы можете  посмотреть в файл примера  ~/src/asterisk-1.15.&lt;your version&gt;/configs/samples/voicemail.conf.sample. Он содержит полезную информацию о различных настройках.

Далее мы рассмотрим простейший voicemail.conf файл. Если у вас  возникнет желание доработать базовую конфигурацию, просто добавьте или измените соответствующие опции.

### An Initial voicemail.conf File

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
Настройка Linux сервера для отправки почтовых сообщений  администратору выходит за рамки данной книги. Вы должны будете протестировать вашу службу голосовой почты чтобы убедиться что она корректно обрабатывается почтовым агентом\[^2\] и что нижеследующие спам-фильтры не отклоняют эти сообщения \(одна из причин почему это может происходить — использование сервером Астериск в теле письма имени хоста которое он не может быть разрешено\)
{% endhint %}

Вы можете создать массивный и сложный файл voicemail.conf \(и даже хранить в нем почтовые ящики пользователей\), но для упрощения задачи мы сосредоточимся на нескольких примерах.

### Секция \[general\]

В первой секции файла voicemail.conf, \[general\], определяются глобальные настройки. Многие из этих настроек могут быть переопределены в настройках каждого конкретного ящика. В таблице 8-1 мы перечислили  некоторые опции, которые, как мы считаем, наиболее важно рассмотреть.

The first section of the voicemail.conf file, \[general\], allows you to define global settings. Many of these settings can be assigned on a per-mailbox setting. We’ve listed in [Table 8-1](8.%20Voicemail%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22Voicemail_id272012) a few settings that we feel are the most important to consider.

| Опция | Значение | Примечание |
| :--- | :--- | :--- |
| format | wav49\|gsm\|wav | Для каждого перечисленного формата, Астериск создает  отдельную запись в этом формате, каждый раз когда остается сообщение. Преимущество этого механизма в  экономии ресурсов на транскодировании, которое не надо выполнять, если для записи используется тот же самый кодек что и для канала. Мы любим WAV за высокое качество, и WAV49 потому что  он хорошо сжимается и легок для передачи по почте. Мы не любим GSM за шумы в записи, но он пользуется  некторой популярностью. |
| serveremail | user@domain |  |
| attach | yes,no |  |
|  |  |  |

Table 8-1. \[general\] section options for voicemail.conf

<table>
  <thead>
    <tr>
      <th style="text-align:left">Option</th>
      <th style="text-align:left">Value/example</th>
      <th style="text-align:left">Notes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">format</td>
      <td style="text-align:left">wav49|gsm|wav</td>
      <td style="text-align:left">For each format listed, Asterisk creates a separate recording in that
        format whenever a message is left. The benefit is that some transcoding
        steps may be saved if the stored format is the same as the codec used on
        the channel. We like WAV because it is the highest quality, and WAV49 because
        it is nicely compressed and easy to email. We don&#x2019;t like GSM due
        to its scratchy sound, but it enjoys some popularity.<a href="https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch08.html%22%20/l%20%22idm46178407560904">a</a>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">serveremail</td>
      <td style="text-align:left">user@domain</td>
      <td style="text-align:left">When an email is sent from Asterisk, this is the email address that it
        will appear to come from.<a href="https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch08.html%22%20/l%20%22idm46178407558120">b</a>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">attach</td>
      <td style="text-align:left">yes,no</td>
      <td style="text-align:left">If an email address is specified for a mailbox, this determines whether
        the message is attached to the email (if not, a simple message notification
        is sent, and the user will need to call into voicemail to retrieve their
        messages).</td>
    </tr>
    <tr>
      <td style="text-align:left">maxmsg</td>
      <td style="text-align:left">9999</td>
      <td style="text-align:left">By default, Asterisk only allows a maximum of 100 messages to be stored
        per user. For users who delete messages, this is no problem. For people
        who like to save their messages, this space can get eaten up quickly. With
        the size of hard drives these days, you could easily store thousands of
        messages for each user, so our current thinking is to set this to the maximum
        and let the users manage things from there. Be aware that old voicemail
        messages on a large system can waste a lot of hard drive space, after a
        few years of storing every message.</td>
    </tr>
    <tr>
      <td style="text-align:left">maxsecs</td>
      <td style="text-align:left">600</td>
      <td style="text-align:left">This type of setting was useful back when a large voicemail system might
        have only 40 MB<a href="https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch08.html%22%20/l%20%22idm46178407551304">c</a> of
        storage: it was necessary to limit the system because it was easy to fill
        up the hard drive. This setting can be annoying to callers (although it
        does force them to get to the point, so some people like it). Nowadays,
        with terabyte drives common, there is no technical reason to limit the
        length of a message. Two considerations are: 1) if a channel gets hung
        in a mailbox, it&#x2019;s good to set some sort of value so it doesn&#x2019;t
        mindlessly record an endless, empty voice message, but 2) if a user wants
        to use her mailbox to record notes to herself, she won&#x2019;t appreciate
        it if you cut her off after 3 minutes. A setting somewhere between 600
        seconds (10 minutes) and 3600 seconds (1 hour) will probably be about right.</td>
    </tr>
    <tr>
      <td style="text-align:left">emailsubject</td>
      <td style="text-align:left">[PBX]: New message ${VM_MSGNUM} in mailbox ${VM_MAILBOX}</td>
      <td style="text-align:left">When Asterisk sends an email, you can use this setting to define what
        the Subject: line of the email will look like. See the voicemail.conf.sample
        file for more details.</td>
    </tr>
    <tr>
      <td style="text-align:left">emailbody</td>
      <td style="text-align:left">Dear ${VM_NAME}:\n\n\tyou have a ${VM_DUR} long message (number ${VM_MSGNUM})\nin
        mailbox ${VM_MAILBOX} \n\n\t\t\t\t--Asterisk\n</td>
      <td style="text-align:left">When Asterisk sends an email, you can use this setting to define what
        the body of the email will look like. See the voicemail.conf.sample file
        for more details.</td>
    </tr>
    <tr>
      <td style="text-align:left">emaildateformat</td>
      <td style="text-align:left">%A, %d %B %Y at %H:%M:%S</td>
      <td style="text-align:left">This option allows you to specify the date format in emails. Uses the
        same rules as the C function STRFTIME.</td>
    </tr>
    <tr>
      <td style="text-align:left">pollmailboxes</td>
      <td style="text-align:left">no, yes</td>
      <td style="text-align:left">If the contents of mailboxes are changed by anything other than app_voicemail
        (such as external applications or another Asterisk system), setting this
        to yes will cause app_voicemail to poll all the mailboxes for changes,
        which will trigger proper message waiting indication (MWI) updates.</td>
    </tr>
    <tr>
      <td style="text-align:left">pollfreq</td>
      <td style="text-align:left">30</td>
      <td style="text-align:left">Used in concert with pollmailboxes, this option specifies the number of
        seconds to wait between mailbox polls.</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><a href="https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch08.html%22%20/l%20%22idm46178407560904-marker">a</a> The
          separator that is used for each format option must be the pipe (|) character.</p>
        <p><a href="https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch08.html%22%20/l%20%22idm46178407558120-marker">b</a> Sending
          email from Asterisk can require some careful configuration, because many
          spam filters will find Asterisk messages suspicious and will simply ignore
          them. We talk more about how to set email for Asterisk in <a href="8.%20Voicemail%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22voicemail_to_email">&#x201C;Voicemail to Email&#x201D;</a>.</p>
        <p><a href="https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch08.html%22%20/l%20%22idm46178407551304-marker">c</a> Yes,
          you read that correctly: megabytes.</p>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
  </tbody>
</table>#### External Validation of Voicemail Passwords

By default, Asterisk does not validate user passwords to ensure they are at least somewhat secure. Anyone who maintains voicemail systems will tell you that a large percentage of mailbox users set their passwords to something like 1234 or 1111, or some other string that’s easy to guess. Although fraud bots aren’t typically interested in making mischief, having lousy passwords does represent a security hole in the voicemail system.

Since the app\_voicemail.so module does not have the built-in ability to validate passwords, the settings externpass, externpassnotify, and externpasscheck allow you to validate them using an external program. Asterisk will call the program based on the path you specify, and pass it the following arguments:

mailbox context oldpass newpass

The script will then evaluate the arguments based on rules that you defined in the external script, and, accordingly, it should return to Asterisk a value of VALID for success or INVALID for failure \(actually, the return value for a failed password can be anything except the words VALID or FAILURE\). This value is typically printed to stdout. If the script returns INVALID, Asterisk will play an invalid-password prompt and the user will need to attempt something different.

Ideally, you would want to implement rules such as the following:

* Passwords must be a minimum of six digits in length
* Passwords must not be strings of repeated digits \(e.g., 111111\)
* Passwords must not be strings of contiguous digits \(e.g., 123456 or 987654\)

Asterisk comes with a simple script that will greatly improve the security of your voicemail system. It is located in the source code under the folder: /contrib/scripts/voicemailpwcheck.py.

We strongly recommend that you copy it to your /usr/local/bin folder \(or wherever you prefer to put such things\), and then uncomment the externpasscheck= option in your voicemail.conf file. Your voicemail system will then enforce the password security rules you have established.

Part of the \[general\] section is an area of supplementary options \(referred to in the config file as advanced options, though there’s not really anything advanced about them\). These options \(listed in [Table 8-2](8.%20Voicemail%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22Voicemail_id291550)\) are defined in the same way as the other options in the \[general\] section, but what is significant about them is that they can also be defined on a per-mailbox basis, which would override whatever is defined under \[general\] for that particular setting. In other words, the following options can be set in the database when you create a new mailbox.

Table 8-2. A curated list of supplementary options for voicemail.conf

| Option | Value/example | Notes |
| :--- | :--- | :--- |
| tz | eastern, european, etc. | Specifies the zonemessages name, as defined under \[zonemessages\] \(discussed in the next section\). |
| locale | de\_DE.utf8, es\_US.utf8, etc. | Used to define how Asterisk generates date/time strings in different locales. To determine the locales that are valid on your Linux system, type locale -a at the shell. |
| attach | yes, no | If an email address is specified for a mailbox, this determines whether the messages are attached to the email notifications \(otherwise, a simple message notification is sent\). |
| attachfmt | wav49, wav, etc. | If attach is enabled and messages are stored in different formats, this defines which format is sent with the email notifications. Often wav49 is a good choice, as it uses a better compression algorithm and thus will use less bandwidth, but doesn’t sound crappy, as gsm does. |
| exitcontext | context | There are options that allow the callers to exit the voicemail system when they are in the process of leaving a message \(for example, pressing 0 to get an operator\). By default, the context the caller came from will be used as the exit context. If desired, this setting will define a different context for callers exiting the voicemail system. |
| review | yes, no | This should almost always be set to yes \(even though it defaults to no\). People get upset if your voicemail system does not allow them to review their messages prior to delivering them. |
| operator | yes, no | Best practice dictates that you should allow your callers to “zero out” from a mailbox, should they not wish to leave a message. Note that an o extension \(not “zero,” but “oh”\) is required in the exitcontext in order to handle these calls. |
| delete | no, yes | After an email message notification is sent \(which could include the message itself\), the message will be deleted. This option is risky, because the fact that a message was emailed is not a guarantee that it was received \(spam filters seem to love to delete Asterisk voicemail messages\). On a new system, leave this at no until you are certain that no messages are being lost due to spam filters. |
| nextaftercmd | yes, no | This handy little setting will save you some time, as it takes you directly to the next message once you’ve finished dealing with the current message. |
| passwordlocation | spooldir | If you want, you can have mailbox passwords stored in the spool folder for each mailbox.[a](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch08.html%22%20/l%20%22idm46178407478744) One of the advantages of using the spooldir option is that it will allow you to define file \#include statements in voicemail.conf \(meaning you can store mailbox references in multiple files, as you can with, for example, dialplan code\). This is not possible otherwise, because app\_voicemail normally writes password changes to the filesystem, and cannot update a mailbox password stored outside of either voicemail.conf or the spool. If you do not use passwordlocation, you will not be able to define mailboxes outside of voicemail.conf, since password updates will not happen. Storing passwords in a file in the specific mailbox folder in the spool solves this problem. |
| [a](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch08.html%22%20/l%20%22idm46178407478744-marker) Typically the spool folder is /var/spool/asterisk, and it can be defined in /etc/asterisk/asterisk.conf. |  |  |

### The \[zonemessages\] Section

The next section of the voicemail.conf file is the \[zonemessages\] section. The purpose of this section is to allow time zone–specific handling of messages, so you can play back to the user messages with the correct timestamps. You can set the name of the zone to whatever you need. Following the zone name, you can define which time zone you want the name to refer to, as well as some options that define how timestamps are played back. You can look at the ~//src/asterisk-16.&lt;TAB&gt;/configs/samples/voicemail.conf.sample file for syntax details. Asterisk includes the examples shown in [Table 8-3](8.%20Voicemail%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22Voicemail_id292277). Any valid time zone known to the Linux system should be configurable. Just use the Linux name for the zone, and then provide the details of how you want it handled.

Table 8-3. \[zonemessages\] section options for voicemail.conf

| Zone name | Value/example | Notes |
| :--- | :--- | :--- |
| eastern | America/New\_York\|'vm-received' Q 'digits/at' IMp | This value would be suitable for the Eastern time zone \(EST/EDT\). |
| central | America/Chicago\|'vm-received' Q 'digits/at' IMp | This value would be suitable for the Central time zone \(CST/CDT\). |
| central24 | America/Chicago\|'vm-received' q 'digits/at' H N 'hours' | This value would also be suitable for CST/CDT, but would play back the time in 24-hour format. |
| military | Zulu\|'vm-received' q 'digits/at' H N 'hours' 'phonetic/z\_p' | This value would be suitable for Universal Time Coordinated \(Zulu time, formerly GMT\). |
| european | Europe/Copenhagen\|'vm-received' a d b 'digits/at' HM | This value would be suitable for Central European time \(CEST\). |

### Mailboxes

You can configure mailboxes in the voicemail.conf file, but it’s not the recommended way. We’re going to use the database to define your mailboxes.

The first thing we need to do is tell Asterisk that voicemail users are available in the database. We do that by editing the /etc/asterisk/extconfig.conf file:

$ sudo vim /etc/asterisk/extconfig.conf

\[settings\] ; older mechanism for connecting all other modules to the database

ps\_endpoints =&gt; odbc,asterisk

ps\_auths =&gt; odbc,asterisk

ps\_aors =&gt; odbc,asterisk

ps\_domain\_aliases =&gt; odbc,asterisk

ps\_endpoint\_id\_ips =&gt; odbc,asterisk

ps\_contacts =&gt; odbc,asterisk

voicemail =&gt; odbc,asterisk,voicemail

You should restart Asterisk to ensure this change has been applied \($ sudo service asterisk restart\).

In the voicemail system, a mailbox must be defined with a context. This does not relate to any dialplan context; it’s a label specific to voicemail that will determine what mailboxes will be grouped together, and is also used to name the folder in the spool that contains the various files associated with this mailbox \(greeting, messages, envelopes, and so forth\). Normally, you don’t need to worry about this, as all mailboxes will end up in the default context. You really only need to define various contexts if you have a complex, multi-tenanted system, where there’s a potential for extension overlap, or where you don’t want certain groups of users exposed to other groups of users.

The \`asterisk\`.\`voicemail\` table offers many options; however, to create a mailbox there are only three fields that are required, plus two more that are recommended. The context, mailbox, and password fields are required, and fullname and email are strongly recommended. Here’s a simple MySQL INSERT that’ll create some mailboxes for you.

INSERT INTO \`asterisk\`.\`voicemail\` \(context,mailbox,password,fullname,email\)

VALUES

\('default','100','486541','Russell Bryant', 'russell@shifteight.org'\),

\('default','101','957642','Leif Madsen', 'leif@shifteight.org'\),

\('default','102','656844','Jared Smith', 'jared@shifteight.org'\),

\('default','103','375416','Jim VanMeggelen', 'jim@shifteight.org'\)

;

The parts of the mailbox definition are:

mailbox

This is the mailbox number. It is normal to ensure it corresponds with the extension number of the associated set.

password

This is the numeric password that the mailbox owner will use to access her voicemail. If the user changes her password, the system will update this field in the database.

If the password is preceded by the hyphen \(-\) character, the user cannot change their mailbox password.

fullname \(FirstName LastName\)

This is the name of the mailbox owner. The company directory uses the text in this field to allow callers to spell usernames. You only get one space, which is meant to delimit the first name from the last name, so if your last name is something like Van Meggelen, you’ll put that in as VanMeggelen. Other punctuation characters might also cause problems. \(We’re looking at you, O’Reilly.\)

email address

This is the email address of the mailbox owner. Asterisk can send the voicemail to the specified email box.

**Warning**

The Asterisk directory cannot handle the concept of a surname that is anything other than a simple word. This means that family names such as O’Reilly, Bryant-Madsen-Smith, and yes, even Van Meggelen must have any punctuation characters and spaces removed before being added to voicemail.conf.

There are quite a few other options you can define for each user. It’s unlikely you’ll use many of them, but [Table 8-4](8.%20Voicemail%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22mailbox_options) contains a curated list of some that may be of use to you.

Table 8-4. Mailbox options

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

