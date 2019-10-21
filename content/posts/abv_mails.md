+++
title = "ABV Emails Retrieval"
author = ["Stanislav Arnaudov"]
description = "A short walkthrough of how I downloaded all of my emails from an obscure email provider."
date = 2019-05-11T00:00:00+02:00
keywords = ["email", "unix-tools"]
lastmod = 2019-10-21T22:18:09+02:00
categories = ["other"]
draft = false
weight = 100
+++

## Abstract {#abstract}

So for several years now, I've been fed up with the email provider that I've been using. I choose it - or there it was chosen for me by my dad - in my most early internet days. Because of its early inception, the email has accumulated an enormous amount of emails over the years. Currently, I have around 7000 emails. Yes, the majority of them are useless spam but I _really_ don't want to go over all of them and clean them up. I also have sort of a sentimental attachment to all of the junk and I do have my hopes to go over everything someday and discover some "gems". I am a giant hoarder is what I am trying to say.

<br />

The email provider in question is [ABV](https://www.abv.bg/). It's a Bulgarian thing and even in Bulgaria everyone makes fun of it but somehow I am still using it. The problem with _ABV_ (or one of its problems) is the lack of [IMAP](https://en.wikipedia.org/wiki/Internet%5FMessage%5FAccess%5FProtocol) support. POP3 is the only offered protocol for access to your email. This sucks as it means I can't fully control my email through separate clients. I must always log through the web interface if I want to do even basic management. Archiving my emails is also impossible as well as keeping by emails and boxes offline. [POP3](https://bg.wikipedia.org/wiki/Post%5FOffice%5FProtocol) offers only rudimentary read-only access to your unread emails which is quite limiting.

<br />

All of this makes working with _ABV_ incredibly annoying. I want to make a switch, but my emails for the past 10 years are there and I can't have them elsewhere.

<br />

Needless to say, I had to do some "hacking" to solve my problem. So let me take you on a journey.

<br />

_Note:_ I am using GNU\Linux and the post assumes you are at least a little bit familiar with working on the terminal (in Bourne shell) in Unix like systems.


## Abusing the Firefox's network analysis tools {#abusing-the-firefox-s-network-analysis-tools}


### An observation {#an-observation}

As already said, _ABV_ does not support _IMAP_. There is, however, a button in the web interface to download an individual email. It shows up only when you open a separate email, so I can't "select & download all" them. Still, the ability to download a single email opens... some opportunities to say at least. I did some inspection with the [network tools of Firefox](https://developer.mozilla.org/en-US/docs/Tools/Network%5FMonitor) and I've found out that the download button makes an HTTP GET request to an URL of the following form:

```url
https://nm40.abv.bg/email/msgdownload?mid=15493841873&fid=10
```

Interesting! I assumed that the `mid` parameter is probably the message-id and the `fid` is "folder id" (maybe?). So OK, it turns out that _ABV_ has an exposed API for downloading emails. If I somehow get the ids of all my emails, maybe I would be able to abuse the endpoint in some automatic way. <br /> <br /> A thing to note is that the downloaded file is in _.eml_ format. I wasn't familiar with the format but a [quick google search](https://fileinfo.com/extension/eml) told me that this is standard for emails and the whole information about the email is there. Attachments included! This is quite convenient as if I had all of my emails in _.eml_ format, I would essentially have all of the information about the emails that are currently on the _ABV_ servers.


### Getting the information {#getting-the-information}

The tasks then became finding the ids of all of my emails. _How on earth was I supposed to do that?_ <br /> <br /> I did some playing around with the network's tool while more or less doing random things in the web interface of the email provider. At one point I deleted an email and I saw some post requests being made to somewhere (I don't care about their damn server). I looked at the request payload and it was something like:

```sh
sh7|2|14|https://nm40.abv.bg/mail/|A9281AB8DC456492EBCF569B4D467239|com.google.gwt.user.client.rpc.XsrfToken/4254043109|92B3E106F93030263CA9346A878A5C38|bg.abv.mail.sg.client.service.InboxService|moveMessages|J|java.lang.String/2004016611|I|[Ljava.lang.String;/2600011424|msg_id|desc||15486542791|1|2|3|4|5|6|8|7|7|8|8|8|9|10|7|K|Ba|11|12|13|0|10|1|14|ObqbBk|
```

I haven't seen such format and by now I only expect [JSON](https://www.json.org/) strings in requests but I am not huge on web development so I have no idea what am I talking about. Anyway, when you pay a little bit of closer attention, you'll see that there is a number in the payload - `1548654279` - which looks suspiciously similar to the number of the `mid` parameter of the download email API. I restored the email from the trash and I tried calling the previous URL with this message id - `1548654279`. And boom! It prompted me to download the previously deleted and restored email. I was onto something! <br /> <br /> A little bit more playing and I had an idea. What if I create a temporary email box and I transfer all of the emails from the inbox there. The client must somehow tell the server to move the emails. Maybe I can then look at the request and the ids will be there. <br /> <br /> I proceeded with my plan and it almost went well. The thing was, probably because of the large number of emails that I have, the client was sending several post requests to the server. I assumed every request was moving some parts of the emails. I inspected the payload of one of the requests and this was there

{{< highlight sh "linenos=table, linenostart=1" >}}
7|2|108|https://nm40.abv.bg/mail/|A9281AB8DC456492EBCF569B4D467239|com.google.gwt.user.client.rpc.XsrfToken/4254043109|92B3E106F93030263CA9346A878A5C38|bg.abv.mail.sg.client.service.InboxService|moveM
essages|J|[Ljava.lang.String;/2600011424|15496357640|15496349933|15493841873|15491365378|15488910522|15488667774|15488660586|15488525888|15488053264|15487848360|15487912098|15486818166|15485039543|154
84640093|15484637666|15484550681|15483826724|15482043396|15481236007|15480216745|15479965022|15479943539|15479891587|15479382334|15478811147|15475565022|15475419857|15475370542|15475073151|15474052961
|15471770500|15471406374|15470918112|15468683208|15467229194|15467152209|15467142650|15466683573|15466504143|15466333339|15466281862|15466292312|15465558833|15465107279|15464506738|15464449485|1546431
6403|15463881116|15462160256|15460884410|15453412832|15453375839|15452559978|15452370377|15452274482|15452270888|15452249423|15451424878|15451258379|15451104402|15450711681|15435367283|15423865703|154
23864259|15423862778|15423090185|15421404008|15421305261|15421273217|15420181646|15420011093|15420009581|15419761356|15419761212|15418273846|15417350335|15417279186|15417206986|15415980677|15415503157
|15413569393|15413293846|15412493040|15412000702|15410110312|15409379406|15406671619|15405845918|15405662432|15404843590|15404532892|15404476121|15404311535|15402037809|15401837448|15401229152|1539713
3757|15396087361|15393747205|15393495764|1|2|3|4|5|6|3|7|7|8|K|SGNP|8|100|9|10|11|12|13|14|15|16|17|18|19|20|21|22|23|24|25|26|27|28|29|30|31|32|33|34|35|36|37|38|39|40|41|42|43|44|45|46|47|48|49|50|5
1|52|53|54|55|56|57|58|59|60|61|62|63|64|65|66|67|68|69|70|71|72|73|74|75|76|77|78|79|80|81|82|83|84|85|86|87|88|89|90|91|92|93|94|95|96|97|98|99|100|101|102|103|104|105|106|107|108|
{{< /highlight >}}

Pretty cryptic but the pattern should be clear by now. The numbers on the middle:

{{< highlight sh "linenos=table, linenostart=1" >}}
15496357640|15496349933|15493841873|15491365378|15488910522|15488667774|15488660586|15488525888|15488053264|15487848360|15487912098|15486818166|15485039543|15484640093|15484637666|15484550681|15483826
724|15482043396|15481236007|15480216745|15479965022|15479943539|15479891587|15479382334|15478811147|15475565022|15475419857|15475370542|15475073151|15474052961|15471770500|15471406374|15470918112|1546
8683208|15467229194|15467152209|15467142650|15466683573|15466504143|15466333339|15466281862|15466292312|15465558833|15465107279|15464506738|15464449485|15464316403|15463881116|15462160256|15460884410|
15453412832|15453375839|15452559978|15452370377|15452274482|15452270888|15452249423|15451424878|15451258379|15451104402|15450711681|15435367283|15423865703|15423864259|15423862778|15423090185|15421404
008|15421305261|15421273217|15420181646|15420011093|15420009581|15419761356|15419761212|15418273846|15417350335|15417279186|15417206986|15415980677|15415503157|15413569393|15413293846|15412493040|1541
2000702|15410110312|15409379406|15406671619|15405845918|15405662432|15404843590|15404532892|15404476121|15404311535|15402037809|15401837448|15401229152|15397133757|15396087361|15393747205|15393495764
{{< /highlight >}}

are clearly the email ids. The other requests contained similar payloads. <br /> <br /> OK, the information is there. _How do I get it quickly?_ I did some more random clicking in the network tools. I managed to figure out that there is something like "downloading multiple requests in .har archive file". I have no idea what a ".har" file is and was too lazy to read the [Wikipedia article](https://en.wikipedia.org/wiki/.har). As far as I understand it, it's some description of "what the browser" did and how it did it (cookies, headers, etc.). The idea is that the actions are described in fully reproducible by other browsers way. Or something like that. This ultimately doesn't matter. <br /> <br /> I selected all of the relevant POST requests and I saved a _.har_ file for them. I looked into it and it did contain lots of information. I stared it for some time and I figured that the information I needed was in JSON nodes of the following kind:

{{< highlight sh "linenos=table, linenostart=1" >}}
...
"text":
"7|2|108|https://nm40.abv.bg/mail/|A9281AB8DC456492EBCF569B4D467239|com.google.gwt.user.client.rpc.XsrfToken/4254043109|8BA318BA03057C85E63015BCC3E398A9|bg.abv.mail.sg.client.service.InboxService|move
Messages|J|[Ljava.lang.String;/2600011424|15493841873|15491365378|15488910522|15488667774|15488660586|15488525888|15488053264|15487848360|15487912098|15486818166|15486542791|15485914949|15485039543|15
484640093|15484637666|15484550681|15483826724|15482043396|15481236007|15480216745|15479965022|15479943539|15479891587|15479382334|15478811147|15475565022|15475419857|15475370542|15475073151|1547405296
1|15471770500|15471406374|15470918112|15468683208|15467229194|15467152209|15467142650|15466683573|15466504143|15466333339|15466281862|15466292312|15465558833|15465107279|15464506738|15464449485|154643
16403|15463881116|15462160256|15460884410|15453412832|15453375839|15452559978|15452370377|15452274482|15452270888|15452249423|15451424878|15451258379|15451104402|15450711681|15435367283|15423865703|15
423864259|15423862778|15423090185|15421404008|15421305261|15421273217|15420181646|15420011093|15420009581|15419761356|15419761212|15418273846|15417350335|15417279186|15417206986|15415980677|1541550315
7|15413569393|15413293846|15412493040|15412000702|15410110312|15409379406|15406671619|15405845918|15405662432|15404843590|15404532892|15404476121|15404311535|15402037809|15401837448|15401229152|153971
33757|15396087361|15393747205|15393495764|1|2|3|4|5|6|3|7|7|8|K|SGNP|8|100|9|10|11|12|13|14|15|16|17|18|19|20|21|22|23|24|25|26|27|28|29|30|31|32|33|34|35|36|37|38|39|40|41|42|43|44|45|46|47|48|49|50|
51|52|53|54|55|56|57|58|59|60|61|62|63|64|65|66|67|68|69|70|71|72|73|74|75|76|77|78|79|80|81|82|83|84|85|86|87|88|89|90|91|92|93|94|95|96|97|98|99|100|101|102|103|104|105|106|107|108|",
...
{{< /highlight >}}

Cool! Getting really close now. _How do I get every id out?_ With [grep](https://www.cyberciti.biz/faq/howto-use-grep-command-in-linux-unix/) of course! Some more staring and I notice that a consistent part of what I need was `moveMessages`. The whole string is on one line so if I call something like

```sh
grep "moveMessages.*" requests.har -o
```

This will filter a lot of the information down. In [regex](https://en.wikipedia.org/wiki/Regular%5Fexpression) land "moveMessages.\*" means "First match `moveMessages` and then anything till the end of the line" The `-o` is there so that _grep_ prints only the matching part of the lines that match. A problem I had then was that there was an inconsistent amount of junk between `moveMessages` and the numbers that I needed. At the end I used this:

```sh
grep "moveMessages.*" requests.har -o | grep "|[0-9]{5,}|" -E -o
```

The first _grep_ command is the same as before. The second matches a minimum of 5 digits between two "|"-symbols. The "-E" flag is there so that extended regular expressions can be used. The command as a whole produces a list of ids that I saved in a plain text file. <br /> <br /> _Note:_ There were, however, some numbers that obviously weren't like the others so I filtered the file with yet another _grep_:

```sh
grep "^26" msgs.txt -v > msgs_clean.txt
```

Luckily, all junk messages were starting with "26" so I matched those and inverted the output of _grep_ with the "-v". <br /> <br /> OK, at this point I had all the information I needed to start downloading. _So how exactly am I supposed to do that?_


### Doing GET requests "manually" {#doing-get-requests-manually}

There is a nifty utility called [curl](https://curl.haxx.se/) that can make HTTP requests at some URLs and dump the information on the standard output. When it comes to the task at hand, the intuitive thing to do is to call that email downloading URL with the ids from the previously generated file. This, however, won't work for obvious reasons. When I am logged in my email on my browser and I call the URL... well, I am logged in my email account. The browser probably sends some cookie so that the server on the other side can know that I am allowed to use the URL. If that wasn't the case, everyone could have downloaded my emails given they had the ids of the emails. <br /> <br /> _So how did I do it?_ Again, random clicking in the network tools of Firefox lead me to find out that one can save a made request as a "cURL". This means (I presumed) that a _curl_ command is copied to your clipboard and if executed, it will do _literally_ the same thing as the copied request. Cookies, headers, and payload - everything is in this command. I made the download email request manually in the browser, copied it as a cURL and it was something like:

```sh
curl 'https://nm40.abv.bg/email/msgdownload?mid=15496357640&fid=10' \
 -H 'User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:66.0) Gecko/20100101 Firefox/66.0' \
 -H 'Accept:text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8' \
 -H 'Accept-Language: en-US,en;q=0.5' \
 --compressed \
 -H 'Referer: https://nm40.abv.bg/Mail.html' \
 -H 'Connection: keep-alive' \
 -H 'Cookie: ...'\
 -H 'TE: Trailers'
```

(Here I've taken out the cookies of course. I wouldn't want you to steal my session 😉.) In the URL there is again that `mid` parameter. Cool! I wrote a very simple bash script that will just take its first argument and put it in the right place.

_get.sh:_

```sh
curl "https://nm40.abv.bg/email/msgdownload?mid=${1}&fid=10" \
 -H 'User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:66.0) Gecko/20100101 Firefox/66.0' \
 -H 'Accept:text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8' \
 -H 'Accept-Language: en-US,en;q=0.5' \
 --compressed \
 -H 'Referer: https://nm40.abv.bg/Mail.html' \
 -H 'Connection: keep-alive' \
 -H 'Cookie: ...'\
 -H 'TE: Trailers'
```

_Note:_ You have to use double quotations when using variables inside strings. It's because of how variables are resolved in bash. <br /> <br /> I can now call the script with an arbitrary email id and redirect the output to a file and have that email to be downloaded in the file:

```sh
./get.sh 1512783 > 1512783.txt
```

I did that and lo and behold - it worked! Now I just had to do that for every id in my ids file. Of course, this is a trivial job when you know your [GNU Core Utilities](https://www.gnu.org/software/coreutils/). The final command was:

```sh
cat msgs_clean.txt | xargs -I{} sh -c "./get.sh {} > {}.eml"
```

xargs takes strings from the standard input, substitutes them in the given command and runs the command. The "given command" is `./get.sh {} > mails/{}.eml` and the "{}" gets substituted with the read email id. <br /> <br /> I ran the command and after an hour or so, I had a directory full of _.eml_ files. **Success!**


## Further doing "whatever you want" {#further-doing-whatever-you-want}

At this point I have all of my emails but what the hell can I do with all of those _.eml_ files. I could open them in some email clients ([Thunderbird](https://www.thunderbird.net/en-US/) for examples) but bulk processing them probably won't be a fun experience.

<br />

It turns out that GNU\Linux provides yet another utility that does something handy - [Mu](https://www.djcbsoftware.nl/code/mu/). _Mu_ can do a lot and it is a really capable program. You can read about it on its website. Here I want to mention just a couple of things I've found so far. Even if I am not using _mu_ to its full extent, I am sure I can accomplish a lot when it comes to organizing my emails and making them "cleaner for reading".

<br />

First off with `mu view email.eml` one can nicely format a _.eml_ file. The output is directly on the standard out so I can do whatever I want with it. The command also displays the headers of the email so the information like sender, receiver, date, etc. is also displayed. This means that I can `mu view` all the files and _grep_ something specific. For example, with

```sh
mu view *.eml | grep "^Subject:"
```

I get a list of all the subjects. With

```sh
mu view *.eml | grep "^Date:"
```

a list of all dates. Only with this I can easily create a script that goes over the emails, formats them in plain text files, puts every email in a separate folder and groups the folders by month. Or like that. All those GNU utilities together with _mu_ make the organization pretty limitless.

<br />

Another feature of _mu_ is the ability to extract attached files from _.eml_ files. With

```sh
mu extract email.eml --save-all --target-dir ./
```

one can extract all of the attachments of a given _.eml_ file and put them in the current folder. With that, I can further bring order to my emails. An idea would be to have one folder per month. In each month-folder, we can then have several subfolders for different parts of the emails for this month - plain text, attachments, and metadata. As said, endless opportunities for a nicely structured email archive.


### Doing it properly {#doing-it-properly}

At this point I have to mention that the whole "organization" consideration can be vastly simpler when done properly. _Mu_ can be used together with [OfflineIMAP](http://www.offlineimap.org/). _OfflineIMAP_ allows you to download all of your emails on a server (that supports _IMAP_, of course) and save them in local [Maildirs](https://en.wikipedia.org/wiki/Maildir). _Mu_, on the other hand, when used correctly can be used to index and query emails to extract some useful information from them. One can further use some email client to have their emails visualized ([neomutt](https://neomutt.org/) or [Mu4e](https://www.djcbsoftware.nl/code/mu/mu4e.html)). It is possible to keep all of your emails locally on your computer and syncing them periodically with some remote server. <br /> With that being said, I am not exactly sure how predownloaded _.eml_ files fit this whole narrative. Can I have my old, archived mails in _.eml_ format along side my active emails on Gmail? I'll have to do some digging in order to be able to answer that.


## Conclusion {#conclusion}

Those were my two cents on emails so far. I am surprised that I succeeded in getting a hold of my data. Transitioning between email addresses is proving to be quite the challenge but now at least I have everything online so I won't be losing anything. <br /> I do understand that this blog post is highly specific and the vast majority of the readers cannot directly extract value out of it. Still, I found the whole process interesting in the way I "hacked" my way to the solution of the problem. With that, I think I can at least show you that a lot of things are possible when you know where to look and how to use the tools that you have at your disposal. <br /> A brief summary of the thing to remember would be:

-   Think about how you can use the APIs of different websites to do what you want.
-   Know your GNU Coreutils. They are useful and save time.
-   The network-analyze tools of Firefox are your friend. Use them!
-   _Curl_ can simulate sessions by sending the appropriate cookies while making requests.
