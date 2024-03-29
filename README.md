Introduction
~~~~~~~~~~~~

Oinkmaster is written by Andreas �stling <andreaso@it.su.se>.
The homepage is at http://oinkmaster.sourceforge.net/

Oinkmaster is simple Perl script released under the BSD license that 
helps you keep your Snort rules current with little or no user 
interaction. It has quite a few useful features regarding rules 
management, such as ability to enable, disable and modify specified 
rules after each update. It will tell you the exact changes from your 
previous rules, so you have total control of what's going on.
It may be useful in conjunction with any program that can use Snort
rules, like Snort (doh!) or Prelude-NIDS.

Oinkmaster is most often used to grab the latest official rules tarball 
from www.snort.org and apply a set of modifications to them (such as 
disabling unwanted ones), but it can just as well be used to manage 
your local rules and also third party rules and distribute them to 
multiple sensors with ability to fine-tune the rules on each sensor or 
group of sensors. Oinkmaster is designed to integrate well with other 
scripts and you can easily setup a very powerful rules management system.
See the FAQ for hints and suggestions.

Use Oinkmaster with care and at your own risk. Check the INSTALL file 
for quick installation instructions. You may also find the FAQ useful, 
and README.gui if you want to use the graphical front-end. If you're on 
Windows, have a look at README.win32. Information about the available
command line options can be found in the manual page (oinkmaster.1).
If it's not installed on your system you don't want to install it, you
can read it directly with "groff -man -Tascii oinkmaster.1 | less".

Check out http://www.snort.org/ for more information about Snort and 
its rules.



Requirements
~~~~~~~~~~~~

It should work on most Unix-like systems with Perl 5.6.1 or later.
You also need either the binaries tar, gzip and wget, or the Perl 
modules Archive::Tar, IO::Zlib and LWP::UserAgent. See oinkmaster.conf 
for more information about this. So far it has been successfully tested 
on OpenBSD, FreeBSD, NetBSD, Linux, AIX, Solaris, Mac OS X, QNX and 
Microsoft Windows.



What problem does Oinkmaster solve?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To have any use of your NIDS, you must keep your signatures (or "rules") 
current. On www.snort.org for example, there is a rules tarball that is
updated with new and improved rules on a regular basis. Very soon you
will realize that this tarball contains rules that are not suitable for
your environment, so you delete (or comment out) them from your local 
rules files. There may also be rules that you have to tweak in different 
ways to suit you better. Everything is fine until it's time to update the 
rules since you will have to apply all these modifications again (and you 
must also keep track of these modifications so you can repeat them after 
each update). This clearly becomes a boring and difficult task if you do 
it manually, especially if you have many sensors and make lots of local 
fine-tuning of the rules.

This is where Oinkmaster comes in - it will automatically do those 
boring modifications to the rules that you would otherwise have to do 
manually after each update.



How it works
~~~~~~~~~~~~

It's actually really simple. Here follows a step by step list of how a 
typical rules update with Oinkmaster works. Optional features may of 
course activate additional procedures, but this is what the basics look 
like.


1) It fetches the Snort rules archive from the specified location
   and puts it in a temporary directory.

2) It unpacks this archive and looks for a directory called "rules" in 
   it. All rules files in here (i.e. files matching the "rules_update" 
   regexp specified in oinkmaster.conf) are processed as specified in 
   oinkmaster.conf. It basically opens each rules file, cleans some 
   leading/trailing whitespaces from it, enables/disables/modifies 
   rules as requested, and writes the result back to the same file 
   again.

3) These new files are now compared to the ones in your output 
   directory (the one specified with "-o"), file by file. All lines are 
   compared, not just the actual rules, although a more complicated 
   comparison is performed on the rules so that the we can later 
   present a result that is easier to parse for a human. Files that are 
   not identical are copied from the temporary directory to the output 
   directory. Detailed result of the comparison is then printed.



Usage information
~~~~~~~~~~~~~~~~~

Try ./oinkmaster.pl -h for more usage information and available options.
Full descriptions of the options can be found in the Oinkmaster manual
page. There are also many useful notes and examples in the default 
oinkmaster.conf.

As a first test, you can create an empty rules directory, for example
/tmp/rules/. Then try executing "oinkmaster.pl -o /tmp/rules". Since 
your /tmp/rules/ directory is empty, all rules files in the downloaded 
archive will be regarded as added, and copied to the output directory. 
Then try "oinkmaster.pl -o /tmp/rules" one more time. This time the 
files in the downloaded archive will be compared to the ones in 
/tmp/rules/ and Oinkmaster will tell you if something has been updated. 

If you already have several rules commented out (or removed) in your 
current local rules, you need to add the SIDs (Snort rule ID) of those 
to oinkmaster.conf manually (or check out the contrib directory for a 
help script) before running Oinkmaster, or they will be re-enabled 
after each update. 

You disable rules by adding "disablesid <sid>" to oinkmaster.conf, where 
<sid> is the SID of the rule in question. So if you want the rule with 
SID 12345 to be commented out after each update, you add the line 
"disablesid 12345" to oinkmaster.conf. When you update the rules the 
next time, this rule will become commented out and Oinkmaster will notify 
you of that change.

You can also add entire files to be totally ignored by adding 
"skipfile filename" options to oinkmaster.conf where "filename" is a 
file in the distribution archive you don't care about at all. These 
files will not be checked for changes and they will not be added or 
updated. For example if you don't include the file icmp-info.rules from 
your snort.conf and don't care about keeping it up to date and don't 
want to be notified of changes in it, you can to add the line
"skipfile icmp-info.rules" to oinkmaster.conf. Although it may be a good 
idea to track changes even for rules files you don't use (i.e. that you 
don't include from your snort.conf). Who knows, you might find something 
interesting some day.

The skilled/brave/stupid people can also use the "modifysid" keyword in
oinkmaster.conf to make arbitrary changes to certain rules after each 
download. This is an excellent way to screw up your rules, so don't use 
it unless you really have to.

Oinkmaster can be run manually or automatically as a cron job. But of 
course, be very careful when doing the latter. Things could easily get 
messed up from one update to another because of different layout in the 
rules archive, typo in a rule, bugs in the script, and so on. At least 
run snort -T on the new rules before restarting your Snort process. 
And as always when updating the Snort rules there may be some changes 
that you really don't like (or at least want to know about before 
using).

Remember that after switching to Oinkmaster for performing the rules 
updating, permanent modifications to the rules files must be done by 
editing oinkmaster.conf, not by editing your rules files directly as 
such modifications will be lost after an update.



Usage examples
~~~~~~~~~~~~~~

To automatically update the rules every night at 02:30 and make a 
backup of the old ones if there were any changes and send difference 
notification to syslog, you could use something like this in your 
crontab:

30 2 * * * oinkmaster.pl -o /etc/snort/rules/ -b /etc/snort/backup 2>&1 |logger -t oinkmaster

If you want the output to be sent to you  in a mail instead, you could 
use something like:
oinkmaster.pl ... 2>&1 | mail -s "subject" you@example.com

If you just want to check for changes in the rules but not actually 
update your existing rules, you can use the -c flag for "careful" mode.
To silently check for updates as a cron job and not sending any e-mail 
unless there were updates available, you could use something like this 
(wrapped for readability):

30 2 * * * TMP=`mktemp /tmp/oinkmaster.XXXXXXXX` &&
(oinkmaster.pl -o /etc/snort/rules/ -q -c > $TMP 2>&1;
if [ -s $TMP ]; then mail -s "subject" you@example.com < $TMP; fi; rm $TMP)

This example only works on systems with the mktemp command but could 
easily be rewritten for others. You get the point. The important thing 
here is to use the -q argument (quiet mode) so Oinkmaster doesn't give 
any output unless there were updates available or if there were 
warning/error messages. You should of course change directories and 
subject/e-mail addresses in the examples above as appropriate. More 
usage examples can be found in the Oinkmaster manual page.



Oinkmaster output
~~~~~~~~~~~~~~~~~

If there were any changes, Oinkmaster will tell you about them.
Here is what the different changes in the rules mean:

o Added:
  - New rule (the SID did not exist in the old rules file).

o Enabled:
  - The rule was commented out in the old rules file, but is
    now activated (perhaps because the SID was removed from oinkmaster.conf).

o Enabled and modified:
  -  The rule was commented out in the old rules file, but
     is now activated. The actual rule had also been modified.

o Removed:
  - The rule no longer exists in the new rules file.

o Disabled:
  - The rule still exists but has now been commented out (perhaps because 
    a disablesid statement with this SID was added to oinkmaster.conf).

o Disabled and modified:
  - The rule still exists but has now been commented out.
    The actual rule had also been modified.

o Modified active:
  - The rule has been modified and is an active rule.

o Modified inactive:
  - The rule has been modified but is currently an inactive (commented 
    out) rule.


Non-rule changes will also be printed. Non-rule lines are simply lines 
that are not actual rules (or rules that for some reason could not 
be successfully parsed), and Oinkmaster will keep track of those as well.

Here comes some typical example output. We see that an active rule in 
exploit.rules has been modified and that a rule in web-frontpage.rules has 
been disabled.

[***] Results from Oinkmaster started Thu Dec 11 09:39:06 2003 [***]

[///]     Modified active rules:     [///]

     -> Modified active in exploit.rules (1):
        old: alert tcp $EXTERNAL_NET any -> $HOME_NET 515 (msg:"EXPLOIT LPRng overflow"; 
flow:to_server; content: "|43 07 89 5B 08 8D 4B 08 89 43 0C B0 0B CD 80 31 C0 FE C0 CD 80 
E8 94 FF FF FF 2F 62 69 6E 2F 73 68 0A|"; reference:cve,CVE-2000-0917; 
reference:bugtraq,1712; classtype:attempted-admin; sid:301;  rev:4;)
        new: alert tcp $EXTERNAL_NET any -> $HOME_NET 515 (msg:"EXPLOIT LPRng overflow"; 
flow:to_server,established; content: "|43 07 89 5B 08 8D 4B 08 89 43 0C B0 0B CD 80 31 C0 
FE C0 CD 80 E8 94 FF FF FF 2F 62 69 6E 2F 73 68 0A|"; reference:cve,CVE-2000-0917; 
reference:bugtraq,1712; classtype:attempted-admin; sid:301;  rev:4;)

[---]         Disabled rules:        [---]

     -> Disabled in web-frontpage.rules (1):
        #alert tcp $EXTERNAL_NET any -> $HTTP_SERVERS $HTTP_PORTS (msg:"WEB-FRONTPAGE /_vti_bin/ 
access";flow:to_server,established; uricontent:"/_vti_bin/"; nocase; classtype:web-application-activity; sid:1288;  
rev:5;)

[*] Non-rule line modifications: [*]
    None.

[*] Added files: [*]
    None.


If you think it's too hard to see the actual difference between the old
and new rule, you could use the "-m" argument to remove common leading
and trailing parts from the rules (see the manual page for details).
The above modified rule would then instead look like this:

[///]     Modified active rules:     [///]

     -> Modified active in exploit.rules (1):
        old SID 301: ...low"; flow:to_server; content: "|43 07 8...
        new SID 301: ...low"; flow:to_server,established; content: "|43 07 8...


If there were any added rules files (i.e. files that have not been 
included in the rules archive before and that have now been added to 
your output directory), you should consider including them from your 
Snort configuration file by adding an "include <file>" statement for it.
If there were any files removed from the archive, you should probably 
consider excluding them from your Snort configuration file (you must use 
-r to check for removed files). Note that if a SID is moved from one file 
to another, it will be regarded as removed from the first one and added 
to the new one. This is good, because if a rule we like is moved from file 
A to file B and we only include file A from our snort.conf, we sure would 
like to be informed of that.



Misc important notes
~~~~~~~~~~~~~~~~~~~~

o It's a really good idea to separate your local rules files (if you have 
  any) from the official rules files by keeping them in separate 
  directories. This way you don't have to worry about filename collisions 
  and that some of your local file can become overwritten by files from 
  the downloaded rules archive.

  It's also a good idea to keep an extra (unused) copy of snort.conf in 
  the rules directory but specify another snort.conf (one that's not in 
  your output directory specified by -o to Oinkmaster) when starting 
  Snort. This way you can keep track of changes in the snort.conf file 
  from the rules archive while keeping your customized copy intact. You 
  do not want to update your production snort.conf automatically!. If 
  there are added/removed rules files in the archive or added/modified 
  variables in snort.conf, you will have to edit your production copy 
  of snort.conf manually and update as needed. Oinkmaster does not do 
  that kind of stuff. Yes, this means that if there is a new variable 
  added the distribution snort.conf and that is used in the new rules, 
  your new rules will not even load until you define that variable in 
  your snort.conf manually. See, Auto-updating with auto-restart is bad 
  unless you're careful.

  You can run with -U <file> though, which means that Oinkmaster will 
  search snort.conf in the downloaded rules archive(s) for all 
  variables (the "var varname something" lines) and then verify against 
  <file>, which probably is your local production copy of snort.conf. 
  Variables that exist in the distribution snort.conf but not in <file> 
  will be added to <file> directly after any other variables it may 
  contain. Variables that have just been modified are NOT updated, the 
  -U argument only cares about new ones.

o Oinkmaster never deletes any rules files from your system.
  So if a file called foo.rules is usually included in the rules 
  distribution archive but is one day removed from it, the old 
  foo.rules will still be left in your rules directory. Oinkmaster will 
  notify you of this but only if you use the "-r" argument, which means 
  that each rules file (*.rules, *.conf ...) in your output directory 
  will be checked to see if it also exists in the downloaded rules 
  archive. If it doesn't, it will be regarded as removed from the 
  archive and a message will be printed because you may want to edit 
  your Snort configuration file to exclude this file and possibly 
  remove it from your system as well.

  If you use a separate directory for your Snort rules from snort.org
  (i.e. you have possible local rules files in another directory) you
  probably want to use the "-r" argument. Otherwise it will be very
  misleading since your local rules files obviously don't exist in the
  rules distribution, and will therefore be regarded as removed every 
  time.

o Remember that there are many ways to stop alerts from firing, and
  disabling the rule is not always the best way. For example, if all
  you want is to stop the alert from/to a particular host, you should
  not disable the rule completely. Instead, check the Snort manual on
  how to use BPF filters, suppressing and pass rules, and choose the
  best method for the situation. Also check out the document entitled
  "How to stop Snort alerts from being generated / how to (not) ignore 
  traffic" on the Oinkmaster web site.

o Don't depend too much on this script for the rules updating process.
  It's just intended to be a small help and you should always keep an 
  eye on it. Manual reviewing of all your rules and configuration 
  should always be done.
