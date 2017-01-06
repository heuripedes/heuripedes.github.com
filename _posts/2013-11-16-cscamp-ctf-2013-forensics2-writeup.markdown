---
layout: post
title: CSCamp CTF 2013 Forensics2 Writeup
lang: en
---

>enygmata/fmul: Alright guys, first-off I want to say that this text does not
>really show everything I did to beat this challenge. It took me about four
>hours to figure everything out and I don't remember most of the things i
>tried.

Forensics 2
-----------
>We got a memory dump of a challenger accused of cheating, can you confirm that?

The challenge description provides a memory dump of a VMware machine (.vmem).
Googling around for memory analysers and related things I found the
[volatility](https://code.google.com/p/volatility/) project, which has a Python
script to analyse memory dumps and other stuff.

Before doing any real work, vol.py (the volatility script) needs a profile that
describes the kind of data we are going to handle. Because of the filename, I
already knew it was a Windows XP machine but since it was the first time using
vol.py I just followed the [instructions](https://code.google.com/p/volatility/wiki/VolatilityUsage22):

    $ vol.py -f Windows\ XP\ Professional-f67cc587.vmem imageinfo
    Volatile Systems Volatility Framework 2.2
    Determining profile based on KDBG search...

              Suggested Profile(s) : WinXPSP2x86, WinXPSP3x86 (Instantiated with WinXPSP2x86)
                         AS Layer1 : JKIA32PagedMemoryPae (Kernel AS)
                         AS Layer2 : FileAddressSpace (<edited>/Windows XP Professional-f67cc587.vmem)
                          PAE type : PAE
                               DTB : 0x34c000L
                              KDBG : 0x8054d2e0L
            Number of Processors : 2
         Image Type (Service Pack) : 3
                    KPCR for CPU 0 : 0xffdff000L
                    KPCR for CPU 1 : 0xf8942000L
                 KUSER_SHARED_DATA : 0xffdf0000L
               Image date and time : 2013-10-17 23:19:02 UTC+0000
         Image local date and time : 2013-10-18 00:19:02 +0100

Now we can take a look at the dump. Let's see which processes the guy was
running:

    $ vol.py -f Windows\ XP\ Professional-f67cc587.vmem pslist
	Volatile Systems Volatility Framework 2.2
	Offset(V)  Name                    PID   PPID   Thds     Hnds   Sess  Wow64 Start                Exit
	---------- -------------------- ------ ------ ------ -------- ------ ------ -------------------- --------------------
	0x823c6660 System                    4      0     61      281 ------      0
	0x81fcb020 smss.exe                548      4      3       19 ------      0 2013-06-30 09:22:16
	0x81f3e020 csrss.exe               672    548     12      370      0      0 2013-06-30 09:22:23
	0x820f3020 winlogon.exe            696    548     19      582      0      0 2013-06-30 09:22:23
	0x8205c020 services.exe            740    696     15      294      0      0 2013-06-30 09:22:23
	0x81fdc020 lsass.exe               752    696     20      359      0      0 2013-06-30 09:22:23
	0x820e97f0 vmacthlp.exe            940    740      1       25      0      0 2013-06-30 09:22:25
	0x8208d6e8 svchost.exe             956    740     16      207      0      0 2013-06-30 09:22:26
	0x81e5e020 svchost.exe            1004    740      8      296      0      0 2013-06-30 09:22:26
	0x82072da0 svchost.exe            1148    740     68     1388      0      0 2013-06-30 09:22:26
	0x820137a8 svchost.exe            1340    740      5       78      0      0 2013-06-30 09:22:26
	0x820e93c8 svchost.exe            1448    740     12      189      0      0 2013-06-30 09:22:28
	0x81ee2620 spoolsv.exe            1780    740     10      140      0      0 2013-06-30 09:22:29
	0x82164da0 explorer.exe           1828   1736     10      461      0      0 2013-06-30 09:22:29
	0x81f3d3b8 rundll32.exe            128   1828      4       74      0      0 2013-06-30 09:22:32
	0x81f3cc08 vmtoolsd.exe            140   1828      5      199      0      0 2013-06-30 09:22:32
	0x81e6cda0 svchost.exe             360    740      5      111      0      0 2013-06-30 09:22:44
	0x820f1c10 svchost.exe             396    740      5      105      0      0 2013-06-30 09:22:44
	0x81fcf4b8 vmtoolsd.exe            592    740      7      278      0      0 2013-06-30 09:22:45
	0x82230350 imapi.exe               516    740      4      118      0      0 2013-06-30 09:22:59
	0x821efc10 alg.exe                1428    740      6      110      0      0 2013-06-30 09:22:59
	0x821e7da0 wscntfy.exe            1572   1148      1       37      0      0 2013-06-30 09:23:00
	0x822c5980 wuauclt.exe            2140   1148      3      110      0      0 2013-06-30 09:24:03
	0x8215a648 rundll32.exe           3548   1828      0 --------      0      0 2013-10-17 21:58:10  2013-10-17 21:58:33
	0x820a5c10 ctfmon.exe             2176    732      1       88      0      0 2013-10-17 22:42:46
	0x820ac5a0 cmd.exe                2504   1828      1       33      0      0 2013-10-17 22:44:26
	0x82101318 xchat.exe              3348   1828      3       92      0      0 2013-10-17 22:45:34
	0x822bf4d8 decryptpastebin        3292   2504      1       90      0      0 2013-10-17 23:18:43


The last process seems suspicious. I dumped it using the [`procmemdump`](https://code.google.com/p/volatility/wiki/CommandReference22#procmemdump)
vol.py command and looking at it's strings I noticed Python and py2exe
references. The next step would be analysing this executable but py2exe
executables are just garbage mixed with internal Python commands so I had to
find a decompiler. I found [py2exe-extract](https://code.google.com/p/py2exe-extract/):

    $ py2exe-extract decryptpastebin
    [*] extracting C:\Python27\lib\site-packages\py2exe\boot_common.py
    [*] extracting decryptpastebin.py
    [*] done!

Unfortunately that output lies, py2exe-extract did not create a .py file. It
just extracted the compiled module (.pyc). Now it was time for some real
decompiling action, so I used [uncompyle2](https://github.com/wibiti/uncompyle2),
but it was made for Python 2.7 and `decryptpastebin.pyc` had 2.6 bytecode.
Luckily, uncompyle2 worked fine after I shut it's mouth by commenting the if
(lines 73 and 74 of uncompyle2/\_\_init\_\_.py) that checked for the bytecode
version.

    $ uncompyle2  decryptpastebin.pyc

    $ python2 decryptpastebin.py
    Welcome to pastebin decryptor v0.1beta
    ======================================
    ID:KEY=

Looking inside `decryptpastebin.py`, I saw that ID was a `pastebin.com` paste
id:

{% highlight python %}
def decryptbin(idkey):
    id = idkey.split(':')[0]
    key = idkey.split(':')[1]
    import httplib
    c = httplib.HTTPConnection('pastebin.com')
    c.request('GET', '/raw.php?i=' + id)
    response = c.getresponse()
    if response.status == 200:
        data = response.read()
        print AESdecrypt(key, data, True)
{% endhighlight %}

I looked at `decryptpastebin`'s strings again but there was no compromising
pastebin.com string. There was nothing interesting on the other processes'
strings, so I decided to check the vmem image's string. A couple HTTP requests
to pastebin.com came into view:

    GET /raw.php?i=V5BvcN64 HTTP/1.1
    Host: pastebin.com
    ...
    GET /raw.php?i=dpAtnvCB HTTP/1.1
    Host: pastebin.com
    ...

So, now I had two pastebin ids, `V5BvcN64` and `dpAtnvCB`, but I needed the
keys to proceed. Again, I didn't find a thing from the executable files and had
to check the .vmem file:

    $ strings Windows\ XP\ Professional-f67cc587.vmem | grep V5BvcN64 | sort -u
    GET /raw.php?i=V5BvcN64 HTTP/1.1
    :sp4nky!~sp4nky@41.227.151.248 PRIVMSG [DDS]ahmed :this one V5BvcN64:LtqK_tRU7YlF
    this one V5BvcN64:LtqK_tRU7YlF
            this one V5BvcN64:LtqK_tRU7YlF
    this one V5BvcN64:LtqK_tRU7YlF
    V5BvcN64:LtqK_tRU7YlF

The string `V5BvcN64:LtqK_tRU7YlF` matches `decryptpastebin.py`'s `KEY:ID`
format so let's try that:

    $ python2 decryptpastebin.py
    Welcome to pastebin decryptor v0.1beta
    ======================================
    ID:KEY= V5BvcN64:LtqK_tRU7YlF
    +====================================>
    The flag is: d99cbe01e4b0c0da0e4711e7c0114e79
    +====================================>

And there's the flag.


