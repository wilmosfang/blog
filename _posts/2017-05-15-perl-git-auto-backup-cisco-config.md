---
layout:  post
title:  Cisco Catalyst 3750 配置自动备份
author:  wilmosfang
tags:   network cisco perl tftp expect git
categories:  perl
wc: 2037 6460 86164
excerpt:  使用 perl 的 expect 模块结合 git 和 crontab 构建一套对思科设备配置信息的自动备份方案
comments: true
---


# 前言

生产环境中常常需要对关键信息进行备份

交换机的配置信息十分关键，如果可以对这类信息进行自动备份并且进行版本控制就可以有效降低生产风险

~~~
Switch#copy running-config ?
  flash2:         Copy to flash2: file system
  flash:          Copy to flash: file system
  ftp:            Copy to ftp: file system
  http:           Copy to http: file system
  https:          Copy to https: file system
  null:           Copy to null: file system
  nvram:          Copy to nvram: file system
  rcp:            Copy to rcp: file system
  running-config  Update (merge with) current system configuration
  scp:            Copy to scp: file system
  startup-config  Copy to startup configuration
  system:         Copy to system: file system
  tftp:           Copy to tftp: file system
  vb:             Copy to vb: file system

Switch#copy running-config
~~~

然而也许是出于安全的考虑，思科设备 (这里指3750) 只允许进行认证之后，从设备里面将信息往外拷贝，而不提供直接从外部抽取数据的接口 (比如认证后的 **scp** 和 **rsync** )

这样就多出了很多人肉操作的成本

这里使用 **perl** 的 **expect** 模块结合 **git** 和 **crontab** 构建一套对思科设备配置信息的自动备份方案



---

# 概要

* TOC
{:toc}


---


## 系统环境


~~~
[root@h102 ~]# cat /etc/issue
CentOS release 6.6 (Final)
Kernel \r on an \m

[root@h102 ~]# uname -a 
Linux h102.temp 2.6.32-504.el6.x86_64 #1 SMP Wed Oct 15 04:27:16 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
[root@h102 ~]#
~~~

---

## 安装 (perl)Expect



~~~
[root@h102 ~]# cpan
Sorry, we have to rerun the configuration dialog for CPAN.pm due to
some missing parameters...



The following questions are intended to help you with the
configuration. The CPAN module needs a directory of its own to cache
important index files and maybe keep a temporary mirror of CPAN files.
This may be a site-wide or a personal directory.



I see you already have a  directory
    /root/.cpan
Shall we use it as the general CPAN build and cache directory?

 <cpan_home>
CPAN build and cache directory? [/root/.cpan] 

Unless you are accessing the CPAN on your filesystem via a file: URL,
CPAN.pm needs to keep the source files it downloads somewhere. Please
supply a directory where the downloaded files are to be kept.

 <keep_source_where>
Download target directory? [/root/.cpan/sources] 

 <build_dir>
Directory where the build process takes place? [/root/.cpan/build] 

Normally CPAN.pm keeps config variables in memory and changes need to
be saved in a separate 'o conf commit' command to make them permanent
between sessions. If you set the 'auto_commit' option to true, changes
to a config variable are always automatically committed to disk.

 <auto_commit>
Always commit changes to config variables to disk? [no] 

CPAN.pm can limit the size of the disk area for keeping the build
directories with all the intermediate files.

 <build_cache>
Cache size for build directory (in MB)? [100] 

The CPAN indexes are usually rebuilt once or twice per hour, but the
typical CPAN mirror mirrors only once or twice per day. Depending on
the quality of your mirror and your desire to be on the bleeding edge,
you may want to set the following value to more or less than one day
(which is the default). It determines after how many days CPAN.pm
downloads new indexes.

 <index_expire>
Let the index expire after how many days? [1] 

By default, each time the CPAN module is started, cache scanning is
performed to keep the cache size in sync. To prevent this, answer
'never'.

 <scan_cache>
Perform cache scanning (atstart or never)? [atstart] 

To considerably speed up the initial CPAN shell startup, it is
possible to use Storable to create a cache of metadata. If Storable is
not available, the normal index mechanism will be used.

Note: this mechanism is not used when use_sqlite is on and SQLLite is
running.

 <cache_metadata>
Cache metadata (yes/no)? [yes] 

The CPAN module can detect when a module which you are trying to build
depends on prerequisites. If this happens, it can build the
prerequisites for you automatically ('follow'), ask you for
confirmation ('ask'), or just ignore them ('ignore'). Please set your
policy to one of the three values.

 <prerequisites_policy>
Policy on building prerequisites (follow, ask or ignore)? [ask] 

Every Makefile.PL is run by perl in a separate process. Likewise we
run 'make' and 'make install' in separate processes. If you have
any parameters (e.g. PREFIX, UNINST or the like) you want to
pass to the calls, please specify them here.

If you don't understand this question, just press ENTER.

Typical frequently used settings:

    PREFIX=~/perl    # non-root users (please see manual for more hints)

 <makepl_arg>
Parameters for the 'perl Makefile.PL' command? [INSTALLDIRS=site] 

Parameters for the 'make' command? Typical frequently used setting:

    -j3              # dual processor system (on GNU make)

 <make_arg>
Your choice: [] 

Parameters for the 'make install' command?
Typical frequently used setting:

    UNINST=1         # to always uninstall potentially conflicting files

 <make_install_arg>
Your choice: [] 

A Build.PL is run by perl in a separate process. Likewise we run
'./Build' and './Build install' in separate processes. If you have any
parameters you want to pass to the calls, please specify them here.

Typical frequently used settings:

    --install_base /home/xxx             # different installation directory

 <mbuildpl_arg>
Parameters for the 'perl Build.PL' command? [--installdirs site] 

Parameters for the './Build' command? Setting might be:

    --extra_linker_flags -L/usr/foo/lib  # non-standard library location

 <mbuild_arg>
Your choice: [] 

Do you want to use a different command for './Build install'? Sudo
users will probably prefer:

    su root -c ./Build
 or
    sudo ./Build
 or
    /path1/to/sudo -u admin_account ./Build

 <mbuild_install_build_command>
or some such. Your choice: [./Build] 

Parameters for the './Build install' command? Typical frequently used
setting:

    --uninst 1                           # uninstall conflicting files

 <mbuild_install_arg>
Your choice: [] 



If you're accessing the net via proxies, you can specify them in the
CPAN configuration or via environment variables. The variable in
the $CPAN::Config takes precedence.

 <ftp_proxy>
Your ftp_proxy? [] 

 <http_proxy>
Your http_proxy? [] 

 <no_proxy>
Your no_proxy? [] 

CPAN needs access to at least one CPAN mirror.

As you did not allow me to connect to the internet you need to supply
a valid CPAN URL now.

Please enter the URL of your CPAN mirror  http://mirrors.163.com/cpan/
Configuration does not allow connecting to the internet.
Current set of CPAN URLs:
  http://mirrors.163.com/cpan/
Enter another URL or RETURN to quit: [] http://mirrors.sohu.com/CPAN/
Enter another URL or RETURN to quit: [] http://mirrors.hust.edu.cn/CPAN/
Enter another URL or RETURN to quit: [] ftp://mirrors.ustc.edu.cn/CPAN/
Enter another URL or RETURN to quit: [] http://mirrors.ustc.edu.cn/CPAN/
Enter another URL or RETURN to quit: [] rsync://mirrors.ustc.edu.cn/CPAN/
Enter another URL or RETURN to quit: [] ftp://mirrors.xmu.edu.cn/CPAN/
Enter another URL or RETURN to quit: [] http://mirrors.xmu.edu.cn/CPAN/
Enter another URL or RETURN to quit: [] rsync://mirrors.xmu.edu.cn/CPAN/
Enter another URL or RETURN to quit: [] http://mirror.lzu.edu.cn/CPAN/
Enter another URL or RETURN to quit: [] http://mirrors.neusoft.edu.cn/cpan/
Enter another URL or RETURN to quit: [] http://mirrors.zju.edu.cn/CPAN/
Enter another URL or RETURN to quit: [] 
New urllist
  http://mirrors.163.com/cpan/
  http://mirrors.sohu.com/CPAN/
  http://mirrors.hust.edu.cn/CPAN/
  ftp://mirrors.ustc.edu.cn/CPAN/
  http://mirrors.ustc.edu.cn/CPAN/
  rsync://mirrors.ustc.edu.cn/CPAN/
  ftp://mirrors.xmu.edu.cn/CPAN/
  http://mirrors.xmu.edu.cn/CPAN/
  rsync://mirrors.xmu.edu.cn/CPAN/
  http://mirror.lzu.edu.cn/CPAN/
  http://mirrors.neusoft.edu.cn/cpan/
  http://mirrors.zju.edu.cn/CPAN/


Please remember to call 'o conf commit' to make the config permanent!


cpan shell -- CPAN exploration and modules installation (v1.9402)
Enter 'h' for help.

cpan[1]> install Expect
CPAN: Storable loaded ok (v2.20)
CPAN: LWP::UserAgent loaded ok (v5.833)
CPAN: Time::HiRes loaded ok (v1.9721)
Fetching with LWP:
  http://mirrors.163.com/cpan/authors/01mailrc.txt.gz
Going to read '/root/.cpan/sources/authors/01mailrc.txt.gz'
............................................................................DONE
Fetching with LWP:
  http://mirrors.163.com/cpan/modules/02packages.details.txt.gz
Going to read '/root/.cpan/sources/modules/02packages.details.txt.gz'
  Database was generated on Mon, 15 May 2017 01:53:49 GMT
.............
  New CPAN.pm version (v2.16) available.
  [Currently running version is v1.9402]
  You might want to try
    install CPAN
    reload cpan
  to both upgrade CPAN.pm and run the new version without leaving
  the current session.


...............................................................DONE
Fetching with LWP:
  http://mirrors.163.com/cpan/modules/03modlist.data.gz
Going to read '/root/.cpan/sources/modules/03modlist.data.gz'
DONE
Going to write /root/.cpan/Metadata
Running install for module 'Expect'
CPAN: Data::Dumper loaded ok (v2.124)
'YAML' not installed, falling back to Data::Dumper and Storable to read prefs '/root/.cpan/prefs'
Running make for J/JA/JACOBY/Expect-1.33.tar.gz
Fetching with LWP:
  http://mirrors.163.com/cpan/authors/id/J/JA/JACOBY/Expect-1.33.tar.gz
CPAN: Digest::SHA loaded ok (v5.47)
Fetching with LWP:
  http://mirrors.163.com/cpan/authors/id/J/JA/JACOBY/CHECKSUMS
Checksum for /root/.cpan/sources/authors/id/J/JA/JACOBY/Expect-1.33.tar.gz ok
Scanning cache /root/.cpan/build for sizes
DONE
CPAN: Archive::Tar loaded ok (v1.58)
expect.pm-Expect-1.33/
expect.pm-Expect-1.33/.gitignore
expect.pm-Expect-1.33/.perltidyrc
expect.pm-Expect-1.33/.travis.yml
expect.pm-Expect-1.33/Changes
expect.pm-Expect-1.33/LICENSE
expect.pm-Expect-1.33/MANIFEST.SKIP
expect.pm-Expect-1.33/Makefile.PL
expect.pm-Expect-1.33/README.md
expect.pm-Expect-1.33/examples/
expect.pm-Expect-1.33/examples/calc.pl
expect.pm-Expect-1.33/examples/expect_calc.pl
expect.pm-Expect-1.33/examples/kibitz/
expect.pm-Expect-1.33/examples/kibitz/Changelog
expect.pm-Expect-1.33/examples/kibitz/README
expect.pm-Expect-1.33/examples/kibitz/kibitz
expect.pm-Expect-1.33/examples/kibitz/kibitz.man
expect.pm-Expect-1.33/examples/ssh.pl
expect.pm-Expect-1.33/lib/
expect.pm-Expect-1.33/lib/Expect.pm
expect.pm-Expect-1.33/t/
expect.pm-Expect-1.33/t/01-test.t
expect.pm-Expect-1.33/t/02-bc.t
expect.pm-Expect-1.33/t/03-log.t
expect.pm-Expect-1.33/t/04-multiline.t
expect.pm-Expect-1.33/t/10-internal.t
expect.pm-Expect-1.33/t/11-calc.t
expect.pm-Expect-1.33/tutorial/
expect.pm-Expect-1.33/tutorial/1.A.Intro
expect.pm-Expect-1.33/tutorial/2.A.ftp
expect.pm-Expect-1.33/tutorial/2.B.rlogin
expect.pm-Expect-1.33/tutorial/3.A.debugging
expect.pm-Expect-1.33/tutorial/4.A.top
expect.pm-Expect-1.33/tutorial/5.A.top
expect.pm-Expect-1.33/tutorial/5.B.top
expect.pm-Expect-1.33/tutorial/6.A.smtp-verify
expect.pm-Expect-1.33/tutorial/6.B.modem-init
expect.pm-Expect-1.33/tutorial/README
CPAN: File::Temp loaded ok (v0.22)

  CPAN.pm: Going to build J/JA/JACOBY/Expect-1.33.tar.gz

Warning: prerequisite IO::Pty 1.11 not found.
Warning: prerequisite IO::Tty 1.11 not found.
Warning: prerequisite Test::More 1.00 not found. We have 0.92.
Writing Makefile for Expect
---- Unsatisfied dependencies detected during ----
----         JACOBY/Expect-1.33.tar.gz        ----
    IO::Tty [requires]
    Test::More [requires]
    IO::Pty [requires]
Shall I follow them and prepend them to the queue
of modules we are processing right now? [yes] yes
Running make test
  Delayed until after prerequisites
Running make install
  Delayed until after prerequisites
Running install for module 'IO::Tty'
'YAML' not installed, falling back to Data::Dumper and Storable to read prefs '/root/.cpan/prefs'
Running make for T/TO/TODDR/IO-Tty-1.12.tar.gz
Fetching with LWP:
  http://mirrors.163.com/cpan/authors/id/T/TO/TODDR/IO-Tty-1.12.tar.gz
Fetching with LWP:
  http://mirrors.163.com/cpan/authors/id/T/TO/TODDR/CHECKSUMS
Checksum for /root/.cpan/sources/authors/id/T/TO/TODDR/IO-Tty-1.12.tar.gz ok
IO-Tty-1.12/
IO-Tty-1.12/ChangeLog
IO-Tty-1.12/Makefile.PL
IO-Tty-1.12/MANIFEST
IO-Tty-1.12/META.json
IO-Tty-1.12/META.yml
IO-Tty-1.12/Pty.pm
IO-Tty-1.12/README
IO-Tty-1.12/t/
IO-Tty-1.12/try
IO-Tty-1.12/Tty.pm
IO-Tty-1.12/Tty.xs
IO-Tty-1.12/t/test.t

  CPAN.pm: Going to build T/TO/TODDR/IO-Tty-1.12.tar.gz

Now let's see what we can find out about your system
(logfiles of failing tests are available in the conf/ dir)...
Looking for _getpty()...... not found.
Looking for getpt()........ FOUND.
Looking for grantpt()...... FOUND.
Looking for openpty()...... FOUND.
Looking for posix_openpt(). FOUND.
Looking for ptsname()...... FOUND.
Looking for ptsname_r().... FOUND.
Looking for sigaction().... FOUND.
Looking for strlcpy()...... not found.
Looking for ttyname()...... FOUND.
Looking for unlockpt()..... FOUND.
Looking for libutil.h...... not found.
Looking for pty.h.......... FOUND.
Looking for sys/pty.h...... not found.
Looking for sys/ptyio.h.... not found.
Looking for sys/stropts.h.. not found.
Looking for termio.h....... FOUND.
Looking for termios.h...... FOUND.
Looking for util.h......... not found.
Checking which symbols compile OK...
(sorry for the tedious check, but some systems have not too clean
 header files, to say the least;  '+' means OK, '-' means not defined
 and '*' has compile problems...)
+B0 +B110 +B115200 +B1200 +B134 +B150 -B153600 +B1800 +B19200 +B200 +B230400 +B2400 +B300 -B307200 +B38400 +B460800 +B4800 +B50 +B57600 +B600 +B75 -B76800 +B9600 +BRKINT +BS0 +BS1 +BSDLY +CBAUD -CBAUDEXT +CBRK -CCTS_OFLOW -CDEL +CDSUSP +CEOF +CEOL -CEOL2 +CEOT +CERASE -CESC +CFLUSH +CIBAUD -CIBAUDEXT +CINTR +CKILL +CLNEXT +CLOCAL -CNSWTCH -CNUL +CQUIT +CR0 +CR1 +CR2 +CR3 +CRDLY +CREAD +CRPRNT +CRTSCTS -CRTSXOFF -CRTS_IFLOW +CS5 +CS6 +CS7 +CS8 +CSIZE +CSTART +CSTOP +CSTOPB +CSUSP -CSWTCH +CWERASE -DEFECHO -DIOC -DIOCGETP -DIOCSETP -DOSMODE +ECHO +ECHOCTL +ECHOE +ECHOK +ECHOKE +ECHONL +ECHOPRT +EXTA +EXTB +FF0 +FF1 +FFDLY -FIORDCHK +FLUSHO +HUPCL +ICANON +ICRNL +IEXTEN +IGNBRK +IGNCR +IGNPAR +IMAXBEL +INLCR +INPCK +ISIG +ISTRIP +IUCLC +IXANY +IXOFF +IXON -KBENABLED -LDCHG -LDCLOSE -LDDMAP -LDEMAP -LDGETT -LDGMAP -LDIOC -LDNMAP -LDOPEN -LDSETT -LDSMAP -LOBLK +NCCS +NL0 +NL1 +NLDLY +NOFLSH +OCRNL +OFDEL +OFILL +OLCUC +ONLCR +ONLRET +ONOCR +OPOST -PAGEOUT +PARENB -PAREXT +PARMRK +PARODD +PENDIN -RCV1EN -RTS_TOG +TAB0 +TAB1 +TAB2 +TAB3 +TABDLY -TCDSET +TCFLSH +TCGETA +TCGETS +TCIFLUSH +TCIOFF +TCIOFLUSH +TCION +TCOFLUSH +TCOOFF +TCOON +TCSADRAIN +TCSAFLUSH +TCSANOW +TCSBRK +TCSETA +TCSETAF +TCSETAW -TCSETCTTY +TCSETS +TCSETSF +TCSETSW +TCXONC -TERM_D40 -TERM_D42 -TERM_H45 -TERM_NONE -TERM_TEC -TERM_TEX -TERM_V10 -TERM_V61 +TIOCCBRK -TIOCCDTR +TIOCCONS +TIOCEXCL -TIOCFLUSH -TIOCGETC +TIOCGETD -TIOCGETP -TIOCGLTC +TIOCGPGRP +TIOCGSID +TIOCGSOFTCAR +TIOCGWINSZ -TIOCHPCL -TIOCKBOF -TIOCKBON -TIOCLBIC -TIOCLBIS -TIOCLGET -TIOCLSET +TIOCMBIC +TIOCMBIS +TIOCMGET +TIOCMSET +TIOCM_CAR +TIOCM_CD +TIOCM_CTS +TIOCM_DSR +TIOCM_DTR +TIOCM_LE +TIOCM_RI +TIOCM_RNG +TIOCM_RTS +TIOCM_SR +TIOCM_ST +TIOCNOTTY +TIOCNXCL +TIOCOUTQ -TIOCREMOTE +TIOCSBRK +TIOCSCTTY -TIOCSDTR -TIOCSETC +TIOCSETD -TIOCSETN -TIOCSETP -TIOCSIGNAL -TIOCSLTC +TIOCSPGRP -TIOCSSID +TIOCSSOFTCAR -TIOCSTART +TIOCSTI -TIOCSTOP +TIOCSWINSZ -TM_ANL -TM_CECHO -TM_CINVIS -TM_LCF -TM_NONE -TM_SET -TM_SNL +TOSTOP -VCEOF -VCEOL +VDISCARD -VDSUSP +VEOF +VEOL +VEOL2 +VERASE +VINTR +VKILL +VLNEXT +VMIN +VQUIT +VREPRINT +VSTART +VSTOP +VSUSP -VSWTCH +VT0 +VT1 +VTDLY +VTIME +VWERASE -WRAP +XCASE -XCLUDE -XMT1EN +XTABS 

>>> Configuration looks good! <<<

Writing IO::Tty::Constant.pm...
DEFINE = -DHAVE_DEV_PTMX -DHAVE_GETPT -DHAVE_GRANTPT -DHAVE_OPENPTY -DHAVE_POSIX_OPENPT -DHAVE_PTSNAME -DHAVE_PTSNAME_R -DHAVE_PTY_H -DHAVE_SIGACTION -DHAVE_TERMIOS_H -DHAVE_TERMIO_H -DHAVE_TTYNAME -DHAVE_UNLOCKPT
Checking if your kit is complete...
Looks good
Writing Makefile for IO::Tty
Could not read '/root/.cpan/build/IO-Tty-1.12-xRYZKq/META.yml'. Falling back to other methods to determine prerequisites
cp Tty.pm blib/lib/IO/Tty.pm
cp Tty/Constant.pm blib/lib/IO/Tty/Constant.pm
cp Pty.pm blib/lib/IO/Pty.pm
/usr/bin/perl /usr/share/perl5/ExtUtils/xsubpp  -typemap /usr/share/perl5/ExtUtils/typemap  Tty.xs > Tty.xsc && mv Tty.xsc Tty.c
gcc -c   -D_REENTRANT -D_GNU_SOURCE -fno-strict-aliasing -pipe -fstack-protector -I/usr/local/include -D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64 -O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector --param=ssp-buffer-size=4 -m64 -mtune=generic   -DVERSION=\"1.12\" -DXS_VERSION=\"1.12\" -fPIC "-I/usr/lib64/perl5/CORE"  -DHAVE_DEV_PTMX -DHAVE_GETPT -DHAVE_GRANTPT -DHAVE_OPENPTY -DHAVE_POSIX_OPENPT -DHAVE_PTSNAME -DHAVE_PTSNAME_R -DHAVE_PTY_H -DHAVE_SIGACTION -DHAVE_TERMIOS_H -DHAVE_TERMIO_H -DHAVE_TTYNAME -DHAVE_UNLOCKPT Tty.c
Running Mkbootstrap for IO::Tty ()
chmod 644 Tty.bs
rm -f blib/arch/auto/IO/Tty/Tty.so
gcc  -shared -O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector --param=ssp-buffer-size=4 -m64 -mtune=generic Tty.o  -o blib/arch/auto/IO/Tty/Tty.so 	\
	   -lutil  	\
	  
chmod 755 blib/arch/auto/IO/Tty/Tty.so
cp Tty.bs blib/arch/auto/IO/Tty/Tty.bs
chmod 644 blib/arch/auto/IO/Tty/Tty.bs
Manifying blib/man3/IO::Tty::Constant.3pm
Manifying blib/man3/IO::Tty.3pm
Manifying blib/man3/IO::Pty.3pm
  TODDR/IO-Tty-1.12.tar.gz
  make -- OK
Warning (usually harmless): 'YAML' not installed, will not store persistent state
Running make test
PERL_DL_NONLAZY=1 /usr/bin/perl "-MExtUtils::Command::MM" "-e" "test_harness(0, 'blib/lib', 'blib/arch')" t/*.t
t/test.t .. # Configuration: -DHAVE_DEV_PTMX -DHAVE_GETPT -DHAVE_GRANTPT -DHAVE_OPENPTY -DHAVE_POSIX_OPENPT -DHAVE_PTSNAME -DHAVE_PTSNAME_R -DHAVE_PTY_H -DHAVE_SIGACTION -DHAVE_TERMIOS_H -DHAVE_TERMIO_H -DHAVE_TTYNAME -DHAVE_UNLOCKPT
# Checking for appropriate ioctls:
# TIOCNOTTY
# TIOCSCTTY
trying posix_openpt()...
trying grantpt()...
trying unlockpt()...
trying ptsname_r()...
trying to open /dev/pts/4...
t/test.t .. 1/5 #  === Checking if child gets pty as controlling terminal
trying posix_openpt()...
trying grantpt()...
trying unlockpt()...
trying ptsname_r()...
trying to open /dev/pts/4...
t/test.t .. 3/5 
WARNING: when the client closes the slave pty, the master gets an error
(undef return value and $! eq "Input/output error")
instead of EOF (0 return value).  Please be sure to handle this 
in your application (Expect already does).

# Checking basic functionality and how your ptys handle large strings...
#   This test may hang on certain systems, even though it is protected
#   by alarm().  If the counter stops, try Ctrl-C, the test should continue.
trying posix_openpt()...
trying grantpt()...
trying unlockpt()...
trying ptsname_r()...
trying to open /dev/pts/4...
# isatty($master): YES
# isatty($slave): YES
# Child PID = 37649
# Good, your raw ptys can handle at least 530 bytes at once.
t/test.t .. 5/5 sysread(): Input/output error at t/test.t line 151.
Slave got EOF at line 530, byte 0.
t/test.t .. ok   
All tests successful.
Files=1, Tests=5,  5 wallclock secs ( 0.21 usr  0.18 sys +  0.29 cusr  0.56 csys =  1.24 CPU)
Result: PASS
  TODDR/IO-Tty-1.12.tar.gz
  make test -- OK
Warning (usually harmless): 'YAML' not installed, will not store persistent state
Running make install
Prepending /root/.cpan/build/IO-Tty-1.12-xRYZKq/blib/arch /root/.cpan/build/IO-Tty-1.12-xRYZKq/blib/lib to PERL5LIB for 'install'
Files found in blib/arch: installing files in blib/lib into architecture dependent library tree
Installing /usr/local/lib64/perl5/auto/IO/Tty/Tty.bs
Installing /usr/local/lib64/perl5/auto/IO/Tty/Tty.so
Installing /usr/local/lib64/perl5/IO/Pty.pm
Installing /usr/local/lib64/perl5/IO/Tty.pm
Installing /usr/local/lib64/perl5/IO/Tty/Constant.pm
Installing /usr/local/share/man/man3/IO::Pty.3pm
Installing /usr/local/share/man/man3/IO::Tty::Constant.3pm
Installing /usr/local/share/man/man3/IO::Tty.3pm
Appending installation info to /usr/lib64/perl5/perllocal.pod
  TODDR/IO-Tty-1.12.tar.gz
  make install  -- OK
Warning (usually harmless): 'YAML' not installed, will not store persistent state
Running install for module 'Test::More'
'YAML' not installed, falling back to Data::Dumper and Storable to read prefs '/root/.cpan/prefs'
Running make for E/EX/EXODIST/Test-Simple-1.302085.tar.gz
Fetching with LWP:
  http://mirrors.163.com/cpan/authors/id/E/EX/EXODIST/Test-Simple-1.302085.tar.gz
Fetching with LWP:
  http://mirrors.163.com/cpan/authors/id/E/EX/EXODIST/CHECKSUMS
Checksum for /root/.cpan/sources/authors/id/E/EX/EXODIST/Test-Simple-1.302085.tar.gz ok
Test-Simple-1.302085
Test-Simple-1.302085/README
Test-Simple-1.302085/LICENSE
Test-Simple-1.302085/Changes
Test-Simple-1.302085/MANIFEST
Test-Simple-1.302085/dist.ini
Test-Simple-1.302085/cpanfile
Test-Simple-1.302085/META.yml
Test-Simple-1.302085/lib
Test-Simple-1.302085/lib/ok.pm
Test-Simple-1.302085/META.json
Test-Simple-1.302085/README.md
Test-Simple-1.302085/Makefile.PL
Test-Simple-1.302085/appveyor.yml
Test-Simple-1.302085/lib/Test2.pm
Test-Simple-1.302085/t
Test-Simple-1.302085/t/00-report.t
Test-Simple-1.302085/t/00compile.t
Test-Simple-1.302085/t/lib
Test-Simple-1.302085/t/lib/Dummy.pm
Test-Simple-1.302085/t/lib/SigDie.pm
Test-Simple-1.302085/t/lib/TieOut.pm
Test-Simple-1.302085/t/lib/MyTest.pm
Test-Simple-1.302085/t/Legacy
Test-Simple-1.302085/t/Legacy/died.t
Test-Simple-1.302085/t/Legacy/utf8.t
Test-Simple-1.302085/t/Legacy/note.t
Test-Simple-1.302085/t/Legacy/skip.t
Test-Simple-1.302085/t/Legacy/plan.t
Test-Simple-1.302085/t/Legacy/diag.t
Test-Simple-1.302085/t/Legacy/fork.t
Test-Simple-1.302085/t/Legacy/exit.t
Test-Simple-1.302085/t/Legacy/fail.t
Test-Simple-1.302085/t/Legacy/todo.t
Test-Simple-1.302085/t/Legacy/More.t
Test-Simple-1.302085/t/Legacy/auto.t
Test-Simple-1.302085/lib/Test
Test-Simple-1.302085/lib/Test/More.pm
Test-Simple-1.302085/lib/Test2
Test-Simple-1.302085/lib/Test2/API.pm
Test-Simple-1.302085/lib/Test2/Hub.pm
Test-Simple-1.302085/lib/Test2/IPC.pm
Test-Simple-1.302085/examples
Test-Simple-1.302085/examples/tools.t
Test-Simple-1.302085/t/lib/SkipAll.pm
Test-Simple-1.302085/t/Legacy/depth.t
Test-Simple-1.302085/t/Legacy/undef.t
Test-Simple-1.302085/t/Legacy/extra.t
Test-Simple-1.302085/lib/Test2/Util.pm
Test-Simple-1.302085/examples/tools.pl
Test-Simple-1.302085/t/lib/Dev
Test-Simple-1.302085/t/lib/Dev/Null.pm
Test-Simple-1.302085/t/Legacy/simple.t
Test-Simple-1.302085/t/Legacy/use_ok.t
Test-Simple-1.302085/t/Legacy/import.t
Test-Simple-1.302085/t/Legacy/c_flag.t
Test-Simple-1.302085/t/Legacy/cmp_ok.t
Test-Simple-1.302085/t/Legacy/useing.t
Test-Simple-1.302085/t/Legacy/buffer.t
Test-Simple-1.302085/t/Legacy/new_ok.t
Test-Simple-1.302085/t/Legacy/eq_set.t
Test-Simple-1.302085/t/Legacy/strays.t
Test-Simple-1.302085/lib/Test/Tester.pm
Test-Simple-1.302085/lib/Test/Simple.pm
Test-Simple-1.302085/lib/Test/use
Test-Simple-1.302085/lib/Test/use/ok.pm
Test-Simple-1.302085/lib/Test2/Event.pm
Test-Simple-1.302085/examples/subtest.t
Test-Simple-1.302085/examples/indent.pl
Test-Simple-1.302085/t/lib/SmallTest.pm
Test-Simple-1.302085/t/Legacy/explain.t
Test-Simple-1.302085/t/Legacy/missing.t
Test-Simple-1.302085/t/Legacy/capture.t
Test-Simple-1.302085/t/Legacy/threads.t
Test-Simple-1.302085/t/Legacy/skipall.t
Test-Simple-1.302085/t/Legacy/no_plan.t
Test-Simple-1.302085/lib/Test/Builder.pm
Test-Simple-1.302085/t/lib/MyOverload.pm
Test-Simple-1.302085/t/lib/NoExporter.pm
Test-Simple-1.302085/t/Legacy/no_tests.t
Test-Simple-1.302085/t/Legacy/bad_plan.t
Test-Simple-1.302085/t/Legacy/01-basic.t
Test-Simple-1.302085/t/Legacy/fail_one.t
Test-Simple-1.302085/t/Legacy/bail_out.t
Test-Simple-1.302085/t/Legacy/overload.t
Test-Simple-1.302085/t/Legacy/plan_bad.t
Test-Simple-1.302085/t/Legacy/run_test.t
Test-Simple-1.302085/t/Legacy/versions.t
Test-Simple-1.302085/t/Legacy/Bugs
Test-Simple-1.302085/t/Legacy/Bugs/600.t
Test-Simple-1.302085/t/Legacy/Bugs/629.t
Test-Simple-1.302085/t/zzz-check-breaks.t
Test-Simple-1.302085/t/Legacy/fail-more.t
Test-Simple-1.302085/t/Legacy/fail-like.t
Test-Simple-1.302085/t/Legacy/extra_one.t
Test-Simple-1.302085/t/Test2/legacy
Test-Simple-1.302085/t/Test2/legacy/TAP.t
Test-Simple-1.302085/lib/Test/Tutorial.pod
Test-Simple-1.302085/lib/Test2/Event
Test-Simple-1.302085/lib/Test2/Event/Ok.pm
Test-Simple-1.302085/t/Legacy/require_ok.t
Test-Simple-1.302085/t/Legacy/subtest
Test-Simple-1.302085/t/Legacy/subtest/do.t
Test-Simple-1.302085/t/Test2/modules
Test-Simple-1.302085/t/Test2/modules/IPC.t
Test-Simple-1.302085/t/Test2/modules/API.t
Test-Simple-1.302085/t/Test2/modules/Hub.t
Test-Simple-1.302085/xt/author
Test-Simple-1.302085/xt/author/pod-spell.t
Test-Simple-1.302085/lib/Test2/Formatter.pm
Test-Simple-1.302085/lib/Test2/API
Test-Simple-1.302085/lib/Test2/API/Stack.pm
Test-Simple-1.302085/t/Legacy/filehandles.t
Test-Simple-1.302085/t/Legacy/check_tests.t
Test-Simple-1.302085/t/Legacy/Builder
Test-Simple-1.302085/t/Legacy/Builder/try.t
Test-Simple-1.302085/t/Legacy/subtest/die.t
Test-Simple-1.302085/t/Legacy/Simple
Test-Simple-1.302085/t/Legacy/Simple/load.t
Test-Simple-1.302085/t/Test2/modules/Util.t
Test-Simple-1.302085/xt/author/pod-syntax.t
Test-Simple-1.302085/lib/Test2/Event/Note.pm
Test-Simple-1.302085/lib/Test2/Event/Skip.pm
Test-Simple-1.302085/lib/Test2/Event/Diag.pm
Test-Simple-1.302085/lib/Test2/Event/Bail.pm
Test-Simple-1.302085/lib/Test2/Event/Plan.pm
Test-Simple-1.302085/lib/Test2/Util
Test-Simple-1.302085/lib/Test2/Util/Trace.pm
Test-Simple-1.302085/lib/Test2/IPC
Test-Simple-1.302085/lib/Test2/IPC/Driver.pm
Test-Simple-1.302085/lib/Test2/Tools
Test-Simple-1.302085/lib/Test2/Tools/Tiny.pm
Test-Simple-1.302085/t/Legacy/BEGIN_use_ok.t
Test-Simple-1.302085/t/Legacy/thread_taint.t
Test-Simple-1.302085/t/Legacy/plan_no_plan.t
Test-Simple-1.302085/t/Legacy/Builder/carp.t
Test-Simple-1.302085/t/Legacy/subtest/args.t
Test-Simple-1.302085/t/Legacy/subtest/plan.t
Test-Simple-1.302085/t/Legacy/subtest/fork.t
Test-Simple-1.302085/t/Legacy/subtest/todo.t
Test-Simple-1.302085/t/Test2/modules/Event.t
Test-Simple-1.302085/lib/Test2/Transition.pod
Test-Simple-1.302085/lib/Test2/Hub
Test-Simple-1.302085/lib/Test2/Hub/Subtest.pm
Test-Simple-1.302085/lib/Test2/API/Context.pm
Test-Simple-1.302085/t/Legacy/circular_data.t
Test-Simple-1.302085/t/Legacy/plan_skip_all.t
Test-Simple-1.302085/t/Legacy/Builder/reset.t
Test-Simple-1.302085/t/Legacy/Builder/is_fh.t
Test-Simple-1.302085/t/Legacy/subtest/wstat.t
Test-Simple-1.302085/t/Legacy/subtest/basic.t
Test-Simple-1.302085/t/Legacy/Test2
Test-Simple-1.302085/t/Legacy/Test2/Subtest.t
Test-Simple-1.302085/t/Test2/behavior
Test-Simple-1.302085/t/Test2/behavior/Taint.t
Test-Simple-1.302085/lib/Test2/API/Instance.pm
Test-Simple-1.302085/lib/Test2/API/Breakage.pm
Test-Simple-1.302085/t/Legacy/harness_active.t
Test-Simple-1.302085/t/Legacy/plan_is_noplan.t
Test-Simple-1.302085/t/Legacy/is_deeply_fail.t
Test-Simple-1.302085/t/Legacy/Builder/ok_obj.t
Test-Simple-1.302085/t/Legacy/Builder/create.t
Test-Simple-1.302085/t/Legacy/Builder/output.t
Test-Simple-1.302085/t/Legacy/subtest/events.t
Test-Simple-1.302085/t/Legacy/Regression
Test-Simple-1.302085/t/Legacy/Regression/637.t
Test-Simple-1.302085/lib/Test/Tester
Test-Simple-1.302085/lib/Test/Tester/Capture.pm
Test-Simple-1.302085/lib/Test/Builder
Test-Simple-1.302085/lib/Test/Builder/Tester.pm
Test-Simple-1.302085/lib/Test/Builder/Module.pm
Test-Simple-1.302085/lib/Test2/Event/Waiting.pm
Test-Simple-1.302085/lib/Test2/Event/Generic.pm
Test-Simple-1.302085/lib/Test2/Event/Subtest.pm
Test-Simple-1.302085/lib/Test2/Formatter
Test-Simple-1.302085/lib/Test2/Formatter/TAP.pm
Test-Simple-1.302085/lib/Test2/Util/HashBase.pm
Test-Simple-1.302085/t/lib/Test/Simple
Test-Simple-1.302085/t/lib/Test/Simple/Catch.pm
Test-Simple-1.302085/t/Legacy/478-cmp_ok_hash.t
Test-Simple-1.302085/t/Legacy/Tester
Test-Simple-1.302085/t/Legacy/Tester/tbt_09do.t
Test-Simple-1.302085/t/Legacy/Builder/details.t
Test-Simple-1.302085/t/Legacy/Builder/Builder.t
Test-Simple-1.302085/t/Legacy/Builder/no_diag.t
Test-Simple-1.302085/t/Legacy/subtest/threads.t
Test-Simple-1.302085/t/Test2/modules/Event
Test-Simple-1.302085/t/Test2/modules/Event/Ok.t
Test-Simple-1.302085/t/Test2/regression
Test-Simple-1.302085/t/Test2/regression/gh_16.t
Test-Simple-1.302085/t/Test2/behavior/err_var.t
Test-Simple-1.302085/lib/Test/Tester/Delegate.pm
Test-Simple-1.302085/lib/Test2/Event/Encoding.pm
Test-Simple-1.302085/t/Legacy/BEGIN_require_ok.t
Test-Simple-1.302085/t/Legacy/overload_threads.t
Test-Simple-1.302085/t/Legacy/explain_err_vars.t
Test-Simple-1.302085/t/Legacy/Tester/tbt_03die.t
Test-Simple-1.302085/t/Legacy/Builder/has_plan.t
Test-Simple-1.302085/t/Legacy/subtest/bail_out.t
Test-Simple-1.302085/t/Test2/modules/API
Test-Simple-1.302085/t/Test2/modules/API/Stack.t
Test-Simple-1.302085/lib/Test/Builder/TodoDiag.pm
Test-Simple-1.302085/lib/Test2/Event/Exception.pm
Test-Simple-1.302085/lib/Test2/Hub/Interceptor.pm
Test-Simple-1.302085/t/Legacy/is_deeply_dne_bug.t
Test-Simple-1.302085/t/Legacy/Tester/tbt_07args.t
Test-Simple-1.302085/t/Legacy/Builder/has_plan2.t
Test-Simple-1.302085/t/Legacy/Builder/no_ending.t
Test-Simple-1.302085/t/Legacy/Builder/no_header.t
Test-Simple-1.302085/t/Legacy/subtest/predicate.t
Test-Simple-1.302085/t/Legacy/subtest/singleton.t
Test-Simple-1.302085/t/Test2/modules/Event/Plan.t
Test-Simple-1.302085/t/Test2/modules/Event/Bail.t
Test-Simple-1.302085/t/Test2/modules/Event/Diag.t
Test-Simple-1.302085/t/Test2/modules/Event/Skip.t
Test-Simple-1.302085/t/Test2/modules/Event/Note.t
Test-Simple-1.302085/t/Test2/modules/Util
Test-Simple-1.302085/t/Test2/modules/Util/Trace.t
Test-Simple-1.302085/t/Test2/modules/IPC
Test-Simple-1.302085/t/Test2/modules/IPC/Driver.t
Test-Simple-1.302085/t/Test2/modules/Tools
Test-Simple-1.302085/t/Test2/modules/Tools/Tiny.t
Test-Simple-1.302085/t/Test2/behavior/Formatter.t
Test-Simple-1.302085/lib/Test/Builder/Formatter.pm
Test-Simple-1.302085/lib/Test/Builder/IO
Test-Simple-1.302085/lib/Test/Builder/IO/Scalar.pm
Test-Simple-1.302085/lib/Test2/IPC/Driver
Test-Simple-1.302085/lib/Test2/IPC/Driver/Files.pm
Test-Simple-1.302085/t/Legacy/Tester/tbt_01basic.t
Test-Simple-1.302085/t/Legacy/Builder/is_passing.t
Test-Simple-1.302085/t/Test2/modules/Hub
Test-Simple-1.302085/t/Test2/modules/Hub/Subtest.t
Test-Simple-1.302085/t/Test2/modules/API/Context.t
Test-Simple-1.302085/t/Test2/behavior/init_croak.t
Test-Simple-1.302085/lib/Test2/Event/TAP
Test-Simple-1.302085/lib/Test2/Event/TAP/Version.pm
Test-Simple-1.302085/lib/Test2/Util/ExternalMeta.pm
Test-Simple-1.302085/t/lib/Test/Builder
Test-Simple-1.302085/t/lib/Test/Builder/NoOutput.pm
Test-Simple-1.302085/t/Legacy/Builder/maybe_regex.t
Test-Simple-1.302085/t/Legacy/subtest/for_do_t.test
Test-Simple-1.302085/t/Legacy/Regression/6_cmp_ok.t
Test-Simple-1.302085/t/regression
Test-Simple-1.302085/t/regression/662-tbt-no-plan.t
Test-Simple-1.302085/t/Test2/modules/API/Breakage.t
Test-Simple-1.302085/t/Test2/modules/API/Instance.t
Test-Simple-1.302085/t/Test2/behavior/no_load_api.t
Test-Simple-1.302085/t/Legacy/00test_harness_check.t
Test-Simple-1.302085/t/Legacy/plan_shouldnt_import.t
Test-Simple-1.302085/t/Legacy/Tester/tbt_08subtest.t
Test-Simple-1.302085/t/Legacy/Builder/done_testing.t
Test-Simple-1.302085/t/Legacy/Builder/current_test.t
Test-Simple-1.302085/t/Legacy/subtest/line_numbers.t
Test-Simple-1.302085/t/Test2/modules/Event/Waiting.t
Test-Simple-1.302085/t/Test2/modules/Event/Subtest.t
Test-Simple-1.302085/t/Test2/modules/Event/Generic.t
Test-Simple-1.302085/t/Test2/modules/Formatter
Test-Simple-1.302085/t/Test2/modules/Formatter/TAP.t
Test-Simple-1.302085/t/Test2/modules/Util/HashBase.t
Test-Simple-1.302085/t/Test2/behavior/Subtest_todo.t
Test-Simple-1.302085/t/Test2/behavior/Subtest_plan.t
Test-Simple-1.302085/lib/Test/Tester/CaptureRunner.pm
Test-Simple-1.302085/lib/Test/Builder/Tester
Test-Simple-1.302085/lib/Test/Builder/Tester/Color.pm
Test-Simple-1.302085/t/Legacy/Tester/tbt_04line_num.t
Test-Simple-1.302085/t/Legacy/Tester/tbt_05faildiag.t
Test-Simple-1.302085/t/Legacy/Builder/reset_outputs.t
Test-Simple-1.302085/t/Legacy/subtest/implicit_done.t
Test-Simple-1.302085/t/Legacy/Regression/736_use_ok.t
Test-Simple-1.302085/t/Test2/acceptance
Test-Simple-1.302085/t/Test2/acceptance/try_it_fork.t
Test-Simple-1.302085/t/Test2/acceptance/try_it_plan.t
Test-Simple-1.302085/t/Test2/acceptance/try_it_skip.t
Test-Simple-1.302085/t/Test2/acceptance/try_it_todo.t
Test-Simple-1.302085/t/Test2/behavior/special_names.t
Test-Simple-1.302085/t/Legacy/is_deeply_with_threads.t
Test-Simple-1.302085/t/Legacy/Tester/tbt_06errormess.t
Test-Simple-1.302085/t/Legacy/Tester/tbt_02fhrestore.t
Test-Simple-1.302085/t/Legacy/Builder/no_plan_at_all.t
Test-Simple-1.302085/t/regression/no_name_in_subtest.t
Test-Simple-1.302085/t/regression/642_persistent_end.t
Test-Simple-1.302085/t/Test2/modules/Event/Exception.t
Test-Simple-1.302085/t/Test2/modules/Hub/Interceptor.t
Test-Simple-1.302085/t/Test2/behavior/Subtest_events.t
Test-Simple-1.302085/t/Legacy/Tester/tbt_09do_script.pl
Test-Simple-1.302085/t/Test2/modules/IPC/Driver
Test-Simple-1.302085/t/Test2/modules/IPC/Driver/Files.t
Test-Simple-1.302085/t/Test2/behavior/trace_signature.t
Test-Simple-1.302085/t/Test2/behavior/subtest_bailout.t
Test-Simple-1.302085/t/regression/757-reset_in_subtest.t
Test-Simple-1.302085/t/regression/684-nested_todo_diag.t
Test-Simple-1.302085/t/Test2/modules/Util/ExternalMeta.t
Test-Simple-1.302085/t/Test2/acceptance/try_it_threads.t
Test-Simple-1.302085/t/Test2/acceptance/try_it_no_plan.t
Test-Simple-1.302085/t/Test2/behavior/ipc_wait_timeout.t
Test-Simple-1.302085/t/Legacy_And_Test2
Test-Simple-1.302085/t/Legacy_And_Test2/hidden_warnings.t
Test-Simple-1.302085/t/Legacy/dont_overwrite_die_handler.t
Test-Simple-1.302085/t/Legacy/tbm_doesnt_set_exported_to.t
Test-Simple-1.302085/t/Legacy/Regression/683_thread_todo.t
Test-Simple-1.302085/t/regression/696-intercept_skip_all.t
Test-Simple-1.302085/t/Test2/regression/693_ipc_ordering.t
Test-Simple-1.302085/t/Legacy/Builder/done_testing_double.t
Test-Simple-1.302085/t/Test2/behavior/run_subtest_inherit.t
Test-Simple-1.302085/lib/Test2/Hub/Interceptor
Test-Simple-1.302085/lib/Test2/Hub/Interceptor/Terminator.pm
Test-Simple-1.302085/t/lib/Test/Simple/sample_tests
Test-Simple-1.302085/t/lib/Test/Simple/sample_tests/exit.plx
Test-Simple-1.302085/t/Legacy/Builder/fork_with_new_stdout.t
Test-Simple-1.302085/t/lib/Test/Simple/sample_tests/death.plx
Test-Simple-1.302085/t/Legacy_And_Test2/builder_loaded_late.t
Test-Simple-1.302085/t/Test2/acceptance/try_it_done_testing.t
Test-Simple-1.302085/t/Test2/regression/746-forking-subtest.t
Test-Simple-1.302085/t/lib/Test/Simple/sample_tests/extras.plx
Test-Simple-1.302085/t/Legacy/Builder/done_testing_with_plan.t
Test-Simple-1.302085/t/Test2/regression/ipc_files_abort_exit.t
Test-Simple-1.302085/t/lib/Test/Simple/sample_tests/success.plx
Test-Simple-1.302085/t/lib/Test/Simple/sample_tests/too_few.plx
Test-Simple-1.302085/t/lib/Test/Simple/sample_tests/require.plx
Test-Simple-1.302085/t/regression/721-nested-streamed-subtest.t
Test-Simple-1.302085/t/regression/694_note_diag_return_values.t
Test-Simple-1.302085/t/lib/Test/Simple/sample_tests/two_fail.plx
Test-Simple-1.302085/t/lib/Test/Simple/sample_tests/one_fail.plx
Test-Simple-1.302085/t/Legacy/Builder/done_testing_with_number.t
Test-Simple-1.302085/t/Test2/behavior/nested_context_exception.t
Test-Simple-1.302085/t/Test2/behavior/Subtest_buffer_formatter.t
Test-Simple-1.302085/t/lib/Test/Simple/sample_tests/five_fail.plx
Test-Simple-1.302085/t/Legacy/Builder/done_testing_with_no_plan.t
Test-Simple-1.302085/t/Legacy/Builder/current_test_without_plan.t
Test-Simple-1.302085/t/Test2/modules/Hub/Interceptor
Test-Simple-1.302085/t/Test2/modules/Hub/Interceptor/Terminator.t
Test-Simple-1.302085/t/Legacy/Builder/done_testing_plan_mismatch.t
Test-Simple-1.302085/t/lib/Test/Simple/sample_tests/too_few_fail.plx
Test-Simple-1.302085/t/lib/Test/Simple/sample_tests/death_in_eval.plx
Test-Simple-1.302085/t/lib/Test/Simple/sample_tests/pre_plan_death.plx
Test-Simple-1.302085/t/lib/Test/Simple/sample_tests/last_minute_death.plx
Test-Simple-1.302085/t/lib/Test/Simple/sample_tests/death_with_handler.plx
Test-Simple-1.302085/t/lib/Test/Simple/sample_tests/missing_done_testing.plx
Test-Simple-1.302085/t/lib/Test/Simple/sample_tests/one_fail_without_plan.plx

  CPAN.pm: Going to build E/EX/EXODIST/Test-Simple-1.302085.tar.gz

Checking if your kit is complete...
Looks good
Writing Makefile for Test::Simple
Could not read '/root/.cpan/build/Test-Simple-1.302085-5xr_TI/META.yml'. Falling back to other methods to determine prerequisites
cp lib/Test/Tester.pm blib/lib/Test/Tester.pm
cp lib/Test/Tester/Capture.pm blib/lib/Test/Tester/Capture.pm
cp lib/Test2/Event.pm blib/lib/Test2/Event.pm
cp lib/ok.pm blib/lib/ok.pm
cp lib/Test/Simple.pm blib/lib/Test/Simple.pm
cp lib/Test/Builder/TodoDiag.pm blib/lib/Test/Builder/TodoDiag.pm
cp lib/Test2/API/Instance.pm blib/lib/Test2/API/Instance.pm
cp lib/Test2/API.pm blib/lib/Test2/API.pm
cp lib/Test2/Event/Waiting.pm blib/lib/Test2/Event/Waiting.pm
cp lib/Test/Builder.pm blib/lib/Test/Builder.pm
cp lib/Test/use/ok.pm blib/lib/Test/use/ok.pm
cp lib/Test/Tester/Delegate.pm blib/lib/Test/Tester/Delegate.pm
cp lib/Test/More.pm blib/lib/Test/More.pm
cp lib/Test2/Event/Skip.pm blib/lib/Test2/Event/Skip.pm
cp lib/Test2/Event/TAP/Version.pm blib/lib/Test2/Event/TAP/Version.pm
cp lib/Test/Builder/Formatter.pm blib/lib/Test/Builder/Formatter.pm
cp lib/Test2/Formatter.pm blib/lib/Test2/Formatter.pm
cp lib/Test2/Event/Exception.pm blib/lib/Test2/Event/Exception.pm
cp lib/Test2/Hub/Subtest.pm blib/lib/Test2/Hub/Subtest.pm
cp lib/Test/Builder/Tester.pm blib/lib/Test/Builder/Tester.pm
cp lib/Test2/Event/Subtest.pm blib/lib/Test2/Event/Subtest.pm
cp lib/Test/Tutorial.pod blib/lib/Test/Tutorial.pod
cp lib/Test2/Transition.pod blib/lib/Test2/Transition.pod
cp lib/Test2/Util/ExternalMeta.pm blib/lib/Test2/Util/ExternalMeta.pm
cp lib/Test2/Util/HashBase.pm blib/lib/Test2/Util/HashBase.pm
cp lib/Test2/API/Context.pm blib/lib/Test2/API/Context.pm
cp lib/Test2/Util.pm blib/lib/Test2/Util.pm
cp lib/Test2/Formatter/TAP.pm blib/lib/Test2/Formatter/TAP.pm
cp lib/Test2/Event/Ok.pm blib/lib/Test2/Event/Ok.pm
cp lib/Test2/Event/Plan.pm blib/lib/Test2/Event/Plan.pm
cp lib/Test2/Hub/Interceptor/Terminator.pm blib/lib/Test2/Hub/Interceptor/Terminator.pm
cp lib/Test2/IPC.pm blib/lib/Test2/IPC.pm
cp lib/Test2/Event/Bail.pm blib/lib/Test2/Event/Bail.pm
cp lib/Test2/Util/Trace.pm blib/lib/Test2/Util/Trace.pm
cp lib/Test/Builder/Tester/Color.pm blib/lib/Test/Builder/Tester/Color.pm
cp lib/Test2/Event/Diag.pm blib/lib/Test2/Event/Diag.pm
cp lib/Test2/Hub.pm blib/lib/Test2/Hub.pm
cp lib/Test2/Event/Generic.pm blib/lib/Test2/Event/Generic.pm
cp lib/Test2/API/Breakage.pm blib/lib/Test2/API/Breakage.pm
cp lib/Test2/API/Stack.pm blib/lib/Test2/API/Stack.pm
cp lib/Test/Builder/Module.pm blib/lib/Test/Builder/Module.pm
cp lib/Test2/IPC/Driver/Files.pm blib/lib/Test2/IPC/Driver/Files.pm
cp lib/Test2/IPC/Driver.pm blib/lib/Test2/IPC/Driver.pm
cp lib/Test/Tester/CaptureRunner.pm blib/lib/Test/Tester/CaptureRunner.pm
cp lib/Test2/Hub/Interceptor.pm blib/lib/Test2/Hub/Interceptor.pm
cp lib/Test2/Event/Encoding.pm blib/lib/Test2/Event/Encoding.pm
cp lib/Test2/Tools/Tiny.pm blib/lib/Test2/Tools/Tiny.pm
cp lib/Test2.pm blib/lib/Test2.pm
cp lib/Test/Builder/IO/Scalar.pm blib/lib/Test/Builder/IO/Scalar.pm
cp lib/Test2/Event/Note.pm blib/lib/Test2/Event/Note.pm
Manifying blib/man3/Test2::Event.3pm
Manifying blib/man3/Test::Tester::Capture.3pm
Manifying blib/man3/Test::Tester.3pm
Manifying blib/man3/Test::Simple.3pm
Manifying blib/man3/ok.3pm
Manifying blib/man3/Test2::API::Instance.3pm
Manifying blib/man3/Test::Builder::TodoDiag.3pm
Manifying blib/man3/Test2::Event::Waiting.3pm
Manifying blib/man3/Test2::API.3pm
Manifying blib/man3/Test::use::ok.3pm
Manifying blib/man3/Test::Builder.3pm
Manifying blib/man3/Test::More.3pm
Manifying blib/man3/Test2::Event::TAP::Version.3pm
Manifying blib/man3/Test2::Event::Skip.3pm
Manifying blib/man3/Test::Builder::Formatter.3pm
Manifying blib/man3/Test2::Event::Exception.3pm
Manifying blib/man3/Test2::Formatter.3pm
Manifying blib/man3/Test2::Hub::Subtest.3pm
Manifying blib/man3/Test2::Event::Subtest.3pm
Manifying blib/man3/Test::Builder::Tester.3pm
Manifying blib/man3/Test::Tutorial.3pm
Manifying blib/man3/Test2::Transition.3pm
Manifying blib/man3/Test2::Util::HashBase.3pm
Manifying blib/man3/Test2::Util::ExternalMeta.3pm
Manifying blib/man3/Test2::API::Context.3pm
Manifying blib/man3/Test2::Event::Plan.3pm
Manifying blib/man3/Test2::Event::Ok.3pm
Manifying blib/man3/Test2::Formatter::TAP.3pm
Manifying blib/man3/Test2::Util.3pm
Manifying blib/man3/Test2::Hub::Interceptor::Terminator.3pm
Manifying blib/man3/Test2::Event::Bail.3pm
Manifying blib/man3/Test2::IPC.3pm
Manifying blib/man3/Test2::Util::Trace.3pm
Manifying blib/man3/Test::Builder::Tester::Color.3pm
Manifying blib/man3/Test2::Event::Diag.3pm
Manifying blib/man3/Test2::Event::Generic.3pm
Manifying blib/man3/Test2::Hub.3pm
Manifying blib/man3/Test::Builder::Module.3pm
Manifying blib/man3/Test2::API::Stack.3pm
Manifying blib/man3/Test2::API::Breakage.3pm
Manifying blib/man3/Test2::IPC::Driver.3pm
Manifying blib/man3/Test2::IPC::Driver::Files.3pm
Manifying blib/man3/Test::Tester::CaptureRunner.3pm
Manifying blib/man3/Test2::Hub::Interceptor.3pm
Manifying blib/man3/Test2::Event::Encoding.3pm
Manifying blib/man3/Test::Builder::IO::Scalar.3pm
Manifying blib/man3/Test2.3pm
Manifying blib/man3/Test2::Event::Note.3pm
Manifying blib/man3/Test2::Tools::Tiny.3pm
  EXODIST/Test-Simple-1.302085.tar.gz
  make -- OK
Warning (usually harmless): 'YAML' not installed, will not store persistent state
Running make test
PERL_DL_NONLAZY=1 /usr/bin/perl "-MExtUtils::Command::MM" "-e" "test_harness(0, 'blib/lib', 'blib/arch')" t/*.t t/Legacy/*.t t/Legacy/Bugs/*.t t/Legacy/Builder/*.t t/Legacy/Regression/*.t t/Legacy/Simple/*.t t/Legacy/Test2/*.t t/Legacy/Tester/*.t t/Legacy/subtest/*.t t/Legacy_And_Test2/*.t t/Test2/acceptance/*.t t/Test2/behavior/*.t t/Test2/legacy/*.t t/Test2/modules/*.t t/Test2/modules/API/*.t t/Test2/modules/Event/*.t t/Test2/modules/Formatter/*.t t/Test2/modules/Hub/*.t t/Test2/modules/Hub/Interceptor/*.t t/Test2/modules/IPC/*.t t/Test2/modules/IPC/Driver/*.t t/Test2/modules/Tools/*.t t/Test2/modules/Util/*.t t/Test2/regression/*.t t/regression/*.t
t/00-report.t .................................. 1/? 
# DIAGNOSTICS INFO IN CASE OF FAILURE:

# Perl: 5.010001

# CAPABILITIES:
# CAN_FORK         Yes
# CAN_REALLY_FORK  Yes
# CAN_THREAD       Yes

# DEPENDENCIES:
# Carp          1.11
# File::Spec    3.3
# File::Temp    0.22
# PerlIO        1.06
# Scalar::Util  1.21
# Storable      2.20
# Test2         1.302085
# overload      1.07
# threads       1.82
# utf8          1.07
t/00-report.t .................................. ok   
t/00compile.t .................................. ok     
t/Legacy/00test_harness_check.t ................ ok   
t/Legacy/01-basic.t ............................ ok   
t/Legacy/478-cmp_ok_hash.t ..................... ok   
t/Legacy/auto.t ................................ ok   
t/Legacy/bad_plan.t ............................ ok   
t/Legacy/bail_out.t ............................ ok   
t/Legacy/BEGIN_require_ok.t .................... ok   
t/Legacy/BEGIN_use_ok.t ........................ ok   
t/Legacy/buffer.t .............................. ok     
t/Legacy/Bugs/600.t ............................ ok   
t/Legacy/Bugs/629.t ............................ ok   
t/Legacy/Builder/Builder.t ..................... ok   
t/Legacy/Builder/carp.t ........................ ok   
t/Legacy/Builder/create.t ...................... ok   
t/Legacy/Builder/current_test.t ................ ok   
t/Legacy/Builder/current_test_without_plan.t ... ok   
t/Legacy/Builder/details.t ..................... ok   
t/Legacy/Builder/done_testing.t ................ ok   
t/Legacy/Builder/done_testing_double.t ......... ok   
t/Legacy/Builder/done_testing_plan_mismatch.t .. ok   
t/Legacy/Builder/done_testing_with_no_plan.t ... ok   
t/Legacy/Builder/done_testing_with_number.t .... ok   
t/Legacy/Builder/done_testing_with_plan.t ...... ok   
t/Legacy/Builder/fork_with_new_stdout.t ........ ok   
t/Legacy/Builder/has_plan.t .................... ok   
t/Legacy/Builder/has_plan2.t ................... ok   
t/Legacy/Builder/is_fh.t ....................... ok     
t/Legacy/Builder/is_passing.t .................. ok    
t/Legacy/Builder/maybe_regex.t ................. ok     
t/Legacy/Builder/no_diag.t ..................... ok   
t/Legacy/Builder/no_ending.t ................... ok   
t/Legacy/Builder/no_header.t ................... ok   
t/Legacy/Builder/no_plan_at_all.t .............. ok   
t/Legacy/Builder/ok_obj.t ...................... ok   
t/Legacy/Builder/output.t ...................... ok   
t/Legacy/Builder/reset.t ....................... ok    
t/Legacy/Builder/reset_outputs.t ............... ok   
t/Legacy/Builder/try.t ......................... ok   
t/Legacy/c_flag.t .............................. ok   
t/Legacy/capture.t ............................. ok   
t/Legacy/check_tests.t ......................... ok       
t/Legacy/circular_data.t ....................... ok     
t/Legacy/cmp_ok.t .............................. ok     
t/Legacy/depth.t ............................... ok   
t/Legacy/diag.t ................................ ok   
t/Legacy/died.t ................................ ok   
t/Legacy/dont_overwrite_die_handler.t .......... ok   
t/Legacy/eq_set.t .............................. ok   
t/Legacy/exit.t ................................ ok     
t/Legacy/explain.t ............................. ok   
t/Legacy/explain_err_vars.t .................... ok   
t/Legacy/extra.t ............................... ok   
t/Legacy/extra_one.t ........................... ok   
t/Legacy/fail-like.t ........................... ok   
t/Legacy/fail-more.t ........................... ok     
t/Legacy/fail.t ................................ ok   
t/Legacy/fail_one.t ............................ ok   
t/Legacy/filehandles.t ......................... ok   
t/Legacy/fork.t ................................ ok   
t/Legacy/harness_active.t ...................... ok   
t/Legacy/import.t .............................. ok   
t/Legacy/is_deeply_dne_bug.t ................... ok   
t/Legacy/is_deeply_fail.t ...................... ok       
t/Legacy/is_deeply_with_threads.t .............. skipped: many perls have broken threads.  Enable with AUTHOR_TESTING.
t/Legacy/missing.t ............................. ok   
t/Legacy/More.t ................................ ok     
t/Legacy/new_ok.t .............................. ok     
t/Legacy/no_plan.t ............................. ok   
t/Legacy/no_tests.t ............................ ok   
t/Legacy/note.t ................................ ok   
t/Legacy/overload.t ............................ ok     
t/Legacy/overload_threads.t .................... ok   
t/Legacy/plan.t ................................ ok   
t/Legacy/plan_bad.t ............................ ok     
t/Legacy/plan_is_noplan.t ...................... ok   
t/Legacy/plan_no_plan.t ........................ ok   
t/Legacy/plan_shouldnt_import.t ................ ok   
t/Legacy/plan_skip_all.t ....................... skipped: Just testing plan & skip_all
t/Legacy/Regression/637.t ...................... skipped: many perls have broken threads.  Enable with AUTHOR_TESTING.
t/Legacy/Regression/683_thread_todo.t .......... ok   
t/Legacy/Regression/6_cmp_ok.t ................. ok   
t/Legacy/Regression/736_use_ok.t ............... ok   
t/Legacy/require_ok.t .......................... ok   
t/Legacy/run_test.t ............................ ok     
t/Legacy/simple.t .............................. ok   
t/Legacy/Simple/load.t ......................... ok   
t/Legacy/skip.t ................................ ok     
t/Legacy/skipall.t ............................. ok   
t/Legacy/strays.t .............................. skipped: not completed
t/Legacy/subtest/args.t ........................ ok   
t/Legacy/subtest/bail_out.t .................... ok   
t/Legacy/subtest/basic.t ....................... ok     
t/Legacy/subtest/die.t ......................... ok   
t/Legacy/subtest/do.t .......................... ok   
t/Legacy/subtest/events.t ...................... ok   
t/Legacy/subtest/fork.t ........................ ok   
t/Legacy/subtest/implicit_done.t ............... ok   
t/Legacy/subtest/line_numbers.t ................ ok   
t/Legacy/subtest/plan.t ........................ ok   
t/Legacy/subtest/predicate.t ................... ok   
t/Legacy/subtest/singleton.t ................... ok   
t/Legacy/subtest/threads.t ..................... ok   
t/Legacy/subtest/todo.t ........................ ok     
t/Legacy/subtest/wstat.t ....................... ok   
t/Legacy/tbm_doesnt_set_exported_to.t .......... ok   
t/Legacy/Test2/Subtest.t ....................... ok    
t/Legacy/Tester/tbt_01basic.t .................. ok     
t/Legacy/Tester/tbt_02fhrestore.t .............. ok   
t/Legacy/Tester/tbt_03die.t .................... ok   
t/Legacy/Tester/tbt_04line_num.t ............... ok   
t/Legacy/Tester/tbt_05faildiag.t ............... ok   
t/Legacy/Tester/tbt_06errormess.t .............. ok   
t/Legacy/Tester/tbt_07args.t ................... ok     
t/Legacy/Tester/tbt_08subtest.t ................ ok   
t/Legacy/Tester/tbt_09do.t ..................... ok   
t/Legacy/thread_taint.t ........................ ok   
t/Legacy/threads.t ............................. ok   
t/Legacy/todo.t ................................ ok     
t/Legacy/undef.t ............................... ok     
t/Legacy/use_ok.t .............................. ok    
t/Legacy/useing.t .............................. ok   
t/Legacy/utf8.t ................................ ok   
t/Legacy/versions.t ............................ ok   
t/Legacy_And_Test2/builder_loaded_late.t ....... ok   
t/Legacy_And_Test2/hidden_warnings.t ........... ok   
t/regression/642_persistent_end.t .............. ok   
t/regression/662-tbt-no-plan.t ................. ok   
t/regression/684-nested_todo_diag.t ............ ok   
t/regression/694_note_diag_return_values.t ..... ok   
t/regression/696-intercept_skip_all.t .......... ok   
t/regression/721-nested-streamed-subtest.t ..... ok   
t/regression/757-reset_in_subtest.t ............ ok   
t/regression/no_name_in_subtest.t .............. ok   
t/Test2/acceptance/try_it_done_testing.t ....... ok   
t/Test2/acceptance/try_it_fork.t ............... ok   
t/Test2/acceptance/try_it_no_plan.t ............ ok   
t/Test2/acceptance/try_it_plan.t ............... ok   
t/Test2/acceptance/try_it_skip.t ............... skipped: testing skip all
t/Test2/acceptance/try_it_threads.t ............ ok   
t/Test2/acceptance/try_it_todo.t ............... ok   
t/Test2/behavior/err_var.t ..................... ok   
t/Test2/behavior/Formatter.t ................... ok   
t/Test2/behavior/init_croak.t .................. ok   
t/Test2/behavior/ipc_wait_timeout.t ............ ok   
t/Test2/behavior/nested_context_exception.t .... ok    
t/Test2/behavior/no_load_api.t ................. ok   
t/Test2/behavior/run_subtest_inherit.t ......... ok    
t/Test2/behavior/special_names.t ............... ok   
t/Test2/behavior/subtest_bailout.t ............. ok   
t/Test2/behavior/Subtest_buffer_formatter.t .... ok   
t/Test2/behavior/Subtest_events.t .............. ok   
t/Test2/behavior/Subtest_plan.t ................ ok   
t/Test2/behavior/Subtest_todo.t ................ ok   
t/Test2/behavior/Taint.t ....................... ok   
t/Test2/behavior/trace_signature.t ............. ok    
t/Test2/legacy/TAP.t ........................... ok   
t/Test2/modules/API.t .......................... ok    
t/Test2/modules/API/Breakage.t ................. 1/? Subroutine new redefined at /usr/lib64/perl5/Data/Dumper.pm line 63.
Subroutine Seen redefined at /usr/lib64/perl5/Data/Dumper.pm line 129.
Subroutine Values redefined at /usr/lib64/perl5/Data/Dumper.pm line 162.
Subroutine Names redefined at /usr/lib64/perl5/Data/Dumper.pm line 176.
Subroutine DESTROY redefined at /usr/lib64/perl5/Data/Dumper.pm line 187.
Subroutine Dump redefined at /usr/lib64/perl5/Data/Dumper.pm line 189.
Subroutine Dumpperl redefined at /usr/lib64/perl5/Data/Dumper.pm line 201.
Subroutine _quote redefined at /usr/lib64/perl5/Data/Dumper.pm line 252.
Subroutine _dump redefined at /usr/lib64/perl5/Data/Dumper.pm line 264.
Subroutine Dumper redefined at /usr/lib64/perl5/Data/Dumper.pm line 552.
Subroutine DumperX redefined at /usr/lib64/perl5/Data/Dumper.pm line 557.
Subroutine Dumpf redefined at /usr/lib64/perl5/Data/Dumper.pm line 561.
Subroutine Dumpp redefined at /usr/lib64/perl5/Data/Dumper.pm line 563.
Subroutine Reset redefined at /usr/lib64/perl5/Data/Dumper.pm line 568.
Subroutine Indent redefined at /usr/lib64/perl5/Data/Dumper.pm line 574.
Subroutine Pair redefined at /usr/lib64/perl5/Data/Dumper.pm line 593.
Subroutine Pad redefined at /usr/lib64/perl5/Data/Dumper.pm line 598.
Subroutine Varname redefined at /usr/lib64/perl5/Data/Dumper.pm line 603.
Subroutine Purity redefined at /usr/lib64/perl5/Data/Dumper.pm line 608.
Subroutine Useqq redefined at /usr/lib64/perl5/Data/Dumper.pm line 613.
Subroutine Terse redefined at /usr/lib64/perl5/Data/Dumper.pm line 618.
Subroutine Freezer redefined at /usr/lib64/perl5/Data/Dumper.pm line 623.
Subroutine Toaster redefined at /usr/lib64/perl5/Data/Dumper.pm line 628.
Subroutine Deepcopy redefined at /usr/lib64/perl5/Data/Dumper.pm line 633.
Subroutine Quotekeys redefined at /usr/lib64/perl5/Data/Dumper.pm line 638.
Subroutine Bless redefined at /usr/lib64/perl5/Data/Dumper.pm line 643.
Subroutine Maxdepth redefined at /usr/lib64/perl5/Data/Dumper.pm line 648.
Subroutine Useperl redefined at /usr/lib64/perl5/Data/Dumper.pm line 653.
Subroutine Sortkeys redefined at /usr/lib64/perl5/Data/Dumper.pm line 658.
Subroutine Deparse redefined at /usr/lib64/perl5/Data/Dumper.pm line 663.
Subroutine import redefined at /usr/share/perl5/bytes.pm line 7.
Subroutine unimport redefined at /usr/share/perl5/bytes.pm line 11.
Subroutine AUTOLOAD redefined at /usr/share/perl5/bytes.pm line 15.
Subroutine qquote redefined at /usr/lib64/perl5/Data/Dumper.pm line 680.
Subroutine _sortkeys redefined at /usr/lib64/perl5/Data/Dumper.pm line 720.
Subroutine Data::Dumper::init_refaddr_format redefined at /usr/lib64/perl5/Data/Dumper.pm line 106.
Subroutine Data::Dumper::format_refaddr redefined at /usr/lib64/perl5/Data/Dumper.pm line 111.
t/Test2/modules/API/Breakage.t ................. ok    
t/Test2/modules/API/Context.t .................. ok    
t/Test2/modules/API/Instance.t ................. ok    
t/Test2/modules/API/Stack.t .................... ok    
t/Test2/modules/Event.t ........................ ok    
t/Test2/modules/Event/Bail.t ................... ok   
t/Test2/modules/Event/Diag.t ................... ok   
t/Test2/modules/Event/Exception.t .............. ok   
t/Test2/modules/Event/Generic.t ................ ok    
t/Test2/modules/Event/Note.t ................... ok   
t/Test2/modules/Event/Ok.t ..................... ok   
t/Test2/modules/Event/Plan.t ................... ok    
t/Test2/modules/Event/Skip.t ................... ok   
t/Test2/modules/Event/Subtest.t ................ ok   
t/Test2/modules/Event/Waiting.t ................ ok   
t/Test2/modules/Formatter/TAP.t ................ ok    
t/Test2/modules/Hub.t .......................... ok   
t/Test2/modules/Hub/Interceptor.t .............. ok   
t/Test2/modules/Hub/Interceptor/Terminator.t ... ok   
t/Test2/modules/Hub/Subtest.t .................. ok    
t/Test2/modules/IPC.t .......................... ok   
t/Test2/modules/IPC/Driver.t ................... ok   
t/Test2/modules/IPC/Driver/Files.t ............. ok    
t/Test2/modules/Tools/Tiny.t ................... ok    
t/Test2/modules/Util.t ......................... ok    
t/Test2/modules/Util/ExternalMeta.t ............ ok    
t/Test2/modules/Util/HashBase.t ................ ok    
t/Test2/modules/Util/Trace.t ................... ok    
t/Test2/regression/693_ipc_ordering.t .......... ok   
t/Test2/regression/746-forking-subtest.t ....... ok   
t/Test2/regression/gh_16.t ..................... ok   
t/Test2/regression/ipc_files_abort_exit.t ...... ok   
t/zzz-check-breaks.t ........................... skipped: breakage test requires CPAN::Meta, CPAN::Meta::Requirements and Module::Metadata
All tests successful.
Files=192, Tests=2271, 49 wallclock secs ( 2.39 usr  1.14 sys + 29.51 cusr 12.47 csys = 45.51 CPU)
Result: PASS
  EXODIST/Test-Simple-1.302085.tar.gz
  make test -- OK
Warning (usually harmless): 'YAML' not installed, will not store persistent state
Running make install
Prepending /root/.cpan/build/Test-Simple-1.302085-5xr_TI/blib/arch /root/.cpan/build/Test-Simple-1.302085-5xr_TI/blib/lib to PERL5LIB for 'install'
Installing /usr/local/share/perl5/Test2.pm
Installing /usr/local/share/perl5/ok.pm
Installing /usr/local/share/perl5/Test/More.pm
Installing /usr/local/share/perl5/Test/Tutorial.pod
Installing /usr/local/share/perl5/Test/Builder.pm
Installing /usr/local/share/perl5/Test/Simple.pm
Installing /usr/local/share/perl5/Test/Tester.pm
Installing /usr/local/share/perl5/Test/Tester/Delegate.pm
Installing /usr/local/share/perl5/Test/Tester/CaptureRunner.pm
Installing /usr/local/share/perl5/Test/Tester/Capture.pm
Installing /usr/local/share/perl5/Test/use/ok.pm
Installing /usr/local/share/perl5/Test/Builder/Formatter.pm
Installing /usr/local/share/perl5/Test/Builder/Module.pm
Installing /usr/local/share/perl5/Test/Builder/Tester.pm
Installing /usr/local/share/perl5/Test/Builder/TodoDiag.pm
Installing /usr/local/share/perl5/Test/Builder/IO/Scalar.pm
Installing /usr/local/share/perl5/Test/Builder/Tester/Color.pm
Installing /usr/local/share/perl5/Test2/API.pm
Installing /usr/local/share/perl5/Test2/Formatter.pm
Installing /usr/local/share/perl5/Test2/Util.pm
Installing /usr/local/share/perl5/Test2/IPC.pm
Installing /usr/local/share/perl5/Test2/Hub.pm
Installing /usr/local/share/perl5/Test2/Transition.pod
Installing /usr/local/share/perl5/Test2/Event.pm
Installing /usr/local/share/perl5/Test2/API/Stack.pm
Installing /usr/local/share/perl5/Test2/API/Context.pm
Installing /usr/local/share/perl5/Test2/API/Breakage.pm
Installing /usr/local/share/perl5/Test2/API/Instance.pm
Installing /usr/local/share/perl5/Test2/Hub/Subtest.pm
Installing /usr/local/share/perl5/Test2/Hub/Interceptor.pm
Installing /usr/local/share/perl5/Test2/Hub/Interceptor/Terminator.pm
Installing /usr/local/share/perl5/Test2/IPC/Driver.pm
Installing /usr/local/share/perl5/Test2/IPC/Driver/Files.pm
Installing /usr/local/share/perl5/Test2/Formatter/TAP.pm
Installing /usr/local/share/perl5/Test2/Tools/Tiny.pm
Installing /usr/local/share/perl5/Test2/Util/ExternalMeta.pm
Installing /usr/local/share/perl5/Test2/Util/HashBase.pm
Installing /usr/local/share/perl5/Test2/Util/Trace.pm
Installing /usr/local/share/perl5/Test2/Event/Subtest.pm
Installing /usr/local/share/perl5/Test2/Event/Diag.pm
Installing /usr/local/share/perl5/Test2/Event/Encoding.pm
Installing /usr/local/share/perl5/Test2/Event/Ok.pm
Installing /usr/local/share/perl5/Test2/Event/Exception.pm
Installing /usr/local/share/perl5/Test2/Event/Bail.pm
Installing /usr/local/share/perl5/Test2/Event/Plan.pm
Installing /usr/local/share/perl5/Test2/Event/Note.pm
Installing /usr/local/share/perl5/Test2/Event/Waiting.pm
Installing /usr/local/share/perl5/Test2/Event/Generic.pm
Installing /usr/local/share/perl5/Test2/Event/Skip.pm
Installing /usr/local/share/perl5/Test2/Event/TAP/Version.pm
Installing /usr/local/share/man/man3/Test2::Util::HashBase.3pm
Installing /usr/local/share/man/man3/Test2::API::Stack.3pm
Installing /usr/local/share/man/man3/Test::Builder::TodoDiag.3pm
Installing /usr/local/share/man/man3/Test2::Event::TAP::Version.3pm
Installing /usr/local/share/man/man3/Test2::IPC::Driver::Files.3pm
Installing /usr/local/share/man/man3/Test::Tester.3pm
Installing /usr/local/share/man/man3/Test2::Hub::Subtest.3pm
Installing /usr/local/share/man/man3/Test::Builder::IO::Scalar.3pm
Installing /usr/local/share/man/man3/Test2::Hub::Interceptor::Terminator.3pm
Installing /usr/local/share/man/man3/Test2::Util::Trace.3pm
Installing /usr/local/share/man/man3/Test2::Event::Subtest.3pm
Installing /usr/local/share/man/man3/Test2.3pm
Installing /usr/local/share/man/man3/Test2::Event.3pm
Installing /usr/local/share/man/man3/Test::Tester::Capture.3pm
Installing /usr/local/share/man/man3/Test2::Event::Diag.3pm
Installing /usr/local/share/man/man3/Test2::Event::Exception.3pm
Installing /usr/local/share/man/man3/Test2::Event::Generic.3pm
Installing /usr/local/share/man/man3/Test2::Util.3pm
Installing /usr/local/share/man/man3/Test::Tutorial.3pm
Installing /usr/local/share/man/man3/Test::Builder.3pm
Installing /usr/local/share/man/man3/Test2::Event::Encoding.3pm
Installing /usr/local/share/man/man3/Test::Builder::Formatter.3pm
Installing /usr/local/share/man/man3/Test::More.3pm
Installing /usr/local/share/man/man3/Test2::Event::Waiting.3pm
Installing /usr/local/share/man/man3/Test::Builder::Tester.3pm
Installing /usr/local/share/man/man3/Test2::Event::Ok.3pm
Installing /usr/local/share/man/man3/Test2::API::Context.3pm
Installing /usr/local/share/man/man3/Test2::Transition.3pm
Installing /usr/local/share/man/man3/Test2::IPC.3pm
Installing /usr/local/share/man/man3/Test2::Util::ExternalMeta.3pm
Installing /usr/local/share/man/man3/Test2::Event::Bail.3pm
Installing /usr/local/share/man/man3/ok.3pm
Installing /usr/local/share/man/man3/Test::Simple.3pm
Installing /usr/local/share/man/man3/Test2::Event::Skip.3pm
Installing /usr/local/share/man/man3/Test::use::ok.3pm
Installing /usr/local/share/man/man3/Test2::API::Breakage.3pm
Installing /usr/local/share/man/man3/Test2::Formatter.3pm
Installing /usr/local/share/man/man3/Test::Tester::CaptureRunner.3pm
Installing /usr/local/share/man/man3/Test2::Event::Note.3pm
Installing /usr/local/share/man/man3/Test2::IPC::Driver.3pm
Installing /usr/local/share/man/man3/Test2::Hub::Interceptor.3pm
Installing /usr/local/share/man/man3/Test2::Hub.3pm
Installing /usr/local/share/man/man3/Test2::Tools::Tiny.3pm
Installing /usr/local/share/man/man3/Test2::API::Instance.3pm
Installing /usr/local/share/man/man3/Test2::Event::Plan.3pm
Installing /usr/local/share/man/man3/Test2::Formatter::TAP.3pm
Installing /usr/local/share/man/man3/Test2::API.3pm
Installing /usr/local/share/man/man3/Test::Builder::Tester::Color.3pm
Installing /usr/local/share/man/man3/Test::Builder::Module.3pm
Appending installation info to /usr/lib64/perl5/perllocal.pod
  EXODIST/Test-Simple-1.302085.tar.gz
  make install  -- OK
Warning (usually harmless): 'YAML' not installed, will not store persistent state
IO::Pty is up to date (1.12).
Running make for J/JA/JACOBY/Expect-1.33.tar.gz
  Has already been unwrapped into directory /root/.cpan/build/expect.pm-Expect-1.33-qZ9Ycv

  CPAN.pm: Going to build J/JA/JACOBY/Expect-1.33.tar.gz

cp lib/Expect.pm blib/lib/Expect.pm
Manifying blib/man3/Expect.3pm
  JACOBY/Expect-1.33.tar.gz
  make -- OK
Warning (usually harmless): 'YAML' not installed, will not store persistent state
Running make test
PERL_DL_NONLAZY=1 /usr/bin/perl "-MExtUtils::Command::MM" "-e" "test_harness(0, 'blib/lib', 'blib/arch')" t/*.t
t/01-test.t .......     # Basic tests...
t/01-test.t ....... 1/14     # Testing exec failure...
Cannot exec(Ignore_This_Error_Its_A_Test__efluna3w6868tn8): No such file or directory
    # Testing exp_continue...
    # number of timeout calls in 5 sec: 4
t/01-test.t ....... 4/14     # timeout shouldn't destroy accum contents
t/01-test.t ....... 5/14     # Testing -notransfer...
t/01-test.t ....... 6/14     # Testing raw reversing...
    # isatty($exp): YES
    # Called: 3
    # Elapsed time: 6  delay by expect: 4.5
    # Elapsed time: 6  delay by expect: 3.9
    # Elapsed time: 6  delay by expect: 4.3
    # Called: 3
t/01-test.t ....... 7/14     # Check if the raw pty can handle large chunks of text at once
    # ------------------------------------------------------------------------------
    #   The following tests check system-dependend behaviour, so even if some fail,
    #   Expect might still be perfectly usable for you!
    # ------------------------------------------------------------------------------
    # Length: 512
    # Good, your raw pty can handle lines of at least 512 bytes at a time.
    # Status: match
t/01-test.t ....... 8/14     # Check if the default pty can handle large chunks of text at once
    # ------------------------------------------------------------------------------
    #   The following tests check system-dependend behaviour, so even if some fail,
    #   Expect might still be perfectly usable for you!
    # ------------------------------------------------------------------------------
    # Length: 164
    # Good, your default pty can handle lines of at least 164 bytes at a time.
    # Status: match
    # Testing controlling terminal...
    # Checking if exit status is returned correctly...
    # soft_close: 0x2A00
t/01-test.t ....... 11/14     # Checking if signal exit status is returned correctly...
    # soft_close: 0x000F
t/01-test.t ....... 12/14 # 
# Checking if EOF on pty slave is correctly reported to master...
# (this fails on about 50% of the supported systems, so don't panic!
#  Expect will work anyway!)
# 
t/01-test.t ....... ok     
t/02-bc.t ......... skipped: See https://rt.cpan.org/Ticket/Display.html?id=98495
t/03-log.t ........ # Test created for https://rt.cpan.org/Ticket/Display.html?id=62359 related to clear_accum
t/03-log.t ........ ok     
t/04-multiline.t .. ok     
t/10-internal.t ... ok     
t/11-calc.t ....... 1/22 # SPACE
t/11-calc.t ....... ok     
All tests successful.
Files=6, Tests=97, 62 wallclock secs ( 0.16 usr  0.06 sys +  1.84 cusr  1.09 csys =  3.15 CPU)
Result: PASS
  JACOBY/Expect-1.33.tar.gz
  make test -- OK
Warning (usually harmless): 'YAML' not installed, will not store persistent state
Running make install
Prepending /root/.cpan/build/expect.pm-Expect-1.33-qZ9Ycv/blib/arch /root/.cpan/build/expect.pm-Expect-1.33-qZ9Ycv/blib/lib to PERL5LIB for 'install'
Installing /usr/local/share/perl5/Expect.pm
Installing /usr/local/share/man/man3/Expect.3pm
Appending installation info to /usr/lib64/perl5/perllocal.pod
  JACOBY/Expect-1.33.tar.gz
  make install  -- OK
Warning (usually harmless): 'YAML' not installed, will not store persistent state

cpan[2]> 
~~~

> **Tip:** perl CPAN 的镜像地址可以参考 **[CPAN][cpan]**

---

## 安装 tftp

### 安装 tftp 和 tftp-server

~~~
[root@h102 ~]# yum install tftp tftp-server
Loaded plugins: dellsysid, fastestmirror, refresh-packagekit, security
Setting up Install Process
Determining fastest mirrors
epel/metalink                                                                        | 5.6 kB     00:00     
 * base: mirrors.btte.net
 * epel: mirrors.ustc.edu.cn
 * extras: mirrors.btte.net
 * updates: mirrors.btte.net
base                                                                                 | 3.7 kB     00:00     
base/primary_db                                                                      | 4.7 MB     00:09     
epel                                                                                 | 4.3 kB     00:00     
epel/primary_db                                                                      | 5.9 MB     00:24     
extras                                                                               | 3.4 kB     00:00     
extras/primary_db                                                                    |  37 kB     00:00     
updates                                                                              | 3.4 kB     00:00     
updates/primary_db                                                                   | 821 kB     00:01     
Resolving Dependencies
--> Running transaction check
---> Package tftp.x86_64 0:0.49-8.el6 will be installed
---> Package tftp-server.x86_64 0:0.49-7.el6 will be updated
---> Package tftp-server.x86_64 0:0.49-8.el6 will be an update
--> Finished Dependency Resolution

Dependencies Resolved

============================================================================================================
 Package                     Arch                   Version                      Repository            Size
============================================================================================================
Installing:
 tftp                        x86_64                 0.49-8.el6                   base                  32 k
Updating:
 tftp-server                 x86_64                 0.49-8.el6                   base                  39 k

Transaction Summary
============================================================================================================
Install       1 Package(s)
Upgrade       1 Package(s)

Total download size: 71 k
Is this ok [y/N]: y
Downloading Packages:
(1/2): tftp-0.49-8.el6.x86_64.rpm                                                    |  32 kB     00:00     
(2/2): tftp-server-0.49-8.el6.x86_64.rpm                                             |  39 kB     00:00     
------------------------------------------------------------------------------------------------------------
Total                                                                       435 kB/s |  71 kB     00:00     
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
  Updating   : tftp-server-0.49-8.el6.x86_64                                                            1/3 
  Installing : tftp-0.49-8.el6.x86_64                                                                   2/3 
  Cleanup    : tftp-server-0.49-7.el6.x86_64                                                            3/3 
  Verifying  : tftp-0.49-8.el6.x86_64                                                                   1/3 
  Verifying  : tftp-server-0.49-8.el6.x86_64                                                            2/3 
  Verifying  : tftp-server-0.49-7.el6.x86_64                                                            3/3 

Installed:
  tftp.x86_64 0:0.49-8.el6                                                                                  

Updated:
  tftp-server.x86_64 0:0.49-8.el6                                                                           

Complete!
[root@h102 ~]#
~~~

### 安装 xinetd

~~~
[root@h102 ~]# yum install tftp tftp-server xinetd
Loaded plugins: dellsysid, fastestmirror, refresh-packagekit, security
Setting up Install Process
Loading mirror speeds from cached hostfile
 * base: mirrors.btte.net
 * epel: mirrors.ustc.edu.cn
 * extras: mirrors.btte.net
 * updates: mirrors.btte.net
Package tftp-0.49-8.el6.x86_64 already installed and latest version
Package tftp-server-0.49-8.el6.x86_64 already installed and latest version
Resolving Dependencies
--> Running transaction check
---> Package xinetd.x86_64 2:2.3.14-39.el6_4 will be updated
---> Package xinetd.x86_64 2:2.3.14-40.el6 will be an update
--> Finished Dependency Resolution

Dependencies Resolved

============================================================================================================
 Package                Arch                   Version                           Repository            Size
============================================================================================================
Updating:
 xinetd                 x86_64                 2:2.3.14-40.el6                   base                 122 k

Transaction Summary
============================================================================================================
Upgrade       1 Package(s)

Total download size: 122 k
Is this ok [y/N]: y
Downloading Packages:
xinetd-2.3.14-40.el6.x86_64.rpm                                                      | 122 kB     00:00     
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
  Updating   : 2:xinetd-2.3.14-40.el6.x86_64                                                            1/2 
  Cleanup    : 2:xinetd-2.3.14-39.el6_4.x86_64                                                          2/2 
  Verifying  : 2:xinetd-2.3.14-40.el6.x86_64                                                            1/2 
  Verifying  : 2:xinetd-2.3.14-39.el6_4.x86_64                                                          2/2 

Updated:
  xinetd.x86_64 2:2.3.14-40.el6                                                                             

Complete!
[root@h102 ~]# 
~~~

### 开启tftp

~~~
[root@h102 ~]# /etc/init.d/xinetd status
xinetd (pid  39467) is running...
[root@h102 ~]# chkconfig --list | grep tftp
	tftp:          	on
[root@h102 ~]# 
~~~

> **Tip:** 可以确认一下配置，再重启一下服务

~~~
[root@h102 ~]# cat /etc/xinetd.d/tftp 
# This file is being maintained by Puppet.
# DO NOT EDIT

service tftp
{
        port            = 69
        disable         = no
        socket_type     = dgram
        protocol        = udp
        wait            = yes
        user            = root
        group           = root
        groups          = yes
        server          = /usr/sbin/in.tftpd
        server_args     = -v -s /var/lib/tftpboot/ -m /etc/tftpd.map  -c
        per_source      = 11
        cps             = 100 2
        flags           = IPv4
}
[root@h102 ~]# 
~~~

**`server_args`** 中的参数非常重要 

* **`-c`** 代表可以创建文件，如果不加这个参数则会有如下报错

~~~
[root@h102 ~]# tftp 127.0.0.1 
tftp> put iotop.log 
Error code 1: File not found
tftp>
~~~

* **`-s`** 代表服务存取数据的目录，这个目录要具备相应的读写权限，否则会有如下报错

~~~
[root@h102 ~]# tftp 127.0.0.1 
tftp> put iotop.log 
Error code 0: Permission denied
tftp>
~~~

解决办法就是加上相应的权限

~~~
[root@h102 ~]# chmod o+w /var/lib/tftpboot/
[root@h102 ~]# ll /var/lib/tftpboot/ -d 
drwxr-xrwx. 2 root root 4096 May 15 14:53 /var/lib/tftpboot/
[root@h102 ~]#
~~~


重启一下服务

~~~
[root@h102 ~]# /etc/init.d/xinetd restart
Stopping xinetd:                                           [  OK  ]
Starting xinetd:                                           [  OK  ]
[root@h102 ~]# netstat -a | grep tftp
udp        0      0 *:tftp                      *:*                                     
[root@h102 ~]#
~~~


### 测试上传下载


~~~
[root@h102 ~]# tftp 127.0.0.1 
tftp> put sda2.log 
tftp> get menu.c32
tftp> 
-------
[root@h102 ~]# ll /var/lib/tftpboot/ 
total 640
-rw-r--r--. 1 root   root    20832 May 15  2015 chain.c32
-rw-r--r--. 1 root   root    26268 May 15  2015 memdisk
-rw-r--r--. 1 root   root    61796 May 15  2015 menu.c32
-rw-r--r--. 1 root   root    26759 May 15  2015 pxelinux.0
-rw-rw-rw-  1 nobody nobody 505802 May 15 15:51 sda2.log
[root@h102 ~]#
~~~

---

## 加强设备秘钥长度


~~~
[f:\~]$ ssh admin@192.168.90.1


Connecting to 192.168.90.1:22...
Connection established.
To escape to local shell, press 'Ctrl+Alt+]'.

WARNING! The remote SSH server rejected X11 forwarding request.

Switch#conf
Switch#configure 
Configuring from terminal, memory, or network [terminal]? terminal
Enter configuration commands, one per line.  End with CNTL/Z.
Switch(config)#crypto key generate rsa general-keys
% You already have RSA keys defined named Switch.MyDomain.com.
% Do you really want to replace them? [yes/no]: yes
Choose the size of the key modulus in the range of 360 to 2048 for your
  General Purpose Keys. Choosing a key modulus greater than 512 may take
  a few minutes.

How many bits in the modulus [512]: 1024
% Generating 1024 bit RSA keys ...[OK]

Switch(config)#
~~~

> **Tip:**  为什么要加长秘钥强度呢，因为默认是512太短不安全，如果使用linux服务器进行连接会报错

~~~
[root@h102 ~]# ssh   admin@192.168.90.1
The authenticity of host '192.168.90.1 (192.168.90.1)' can't be established.
RSA key fingerprint is 28:c3:8c:2e:6c:8b:30:ba:71:ad:7d:6c:11:64:62:ed.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.90.1' (RSA) to the list of known hosts.
ssh_rsa_verify: RSA modulus too small: 512 < minimum 768 bits
key_verify failed for server_host_key
[root@h102 ~]# 
~~~

长度加固到1024后就可以正常登录了



~~~
[root@h102 ~]# ssh   admin@192.168.90.1
The authenticity of host '192.168.90.1 (192.168.90.1)' can't be established.
RSA key fingerprint is 43:ba:d6:2a:4a:94:d0:4f:43:ac:79:d8:5d:4f:3e:07.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.90.1' (RSA) to the list of known hosts.
admin@192.168.90.1's password: 

Switch#show run
Switch#show running-config 
Building configuration...

Current configuration : 2475 bytes
!
version 12.2
no service pad
service timestamps debug uptime
service timestamps log uptime
no service password-encryption
!
hostname Switch
!
!
username admin privilege 15 password 0 admin123
aaa new-model
aaa authentication login default local
aaa authorization exec default local 
aaa authorization network default local 
!
aaa session-id common
switch 2 provision ws-c3750-48ts
system mtu routing 1500
 --More-- 
~~~

---

## 手动备份配置文件(运行时配置)

~~~
[root@h102 ~]# ssh   admin@192.168.90.1
admin@192.168.90.1's password: 

Switch#show flash

Directory of flash:/

    2  -rwx        1100   Mar 1 1993 01:51:16 +00:00  private-config.text
    3  -rwx        2475   Mar 1 1993 01:51:15 +00:00  config.text
  364  drwx         192  Apr 13 1993 07:53:17 +00:00  c3750-advipservicesk9-mz.122-35.SE
  363  -rwx         616   Mar 1 1993 00:15:24 +00:00  vlan.dat

15998976 bytes total (4786688 bytes free)
Switch#copy running-config tftp
Address or name of remote host []? 192.168.90.3
Destination filename [switch-confg]? 
!!
2475 bytes copied in 1.141 secs (2169 bytes/sec)
Switch#
~~~

检查一下备份内容

~~~
[root@h102 ~]# ll /var/lib/tftpboot/
total 644
-rw-r--r--. 1 root   root    20832 May 15  2015 chain.c32
-rw-r--r--. 1 root   root    26268 May 15  2015 memdisk
-rw-r--r--. 1 root   root    61796 May 15  2015 menu.c32
-rw-r--r--. 1 root   root    26759 May 15  2015 pxelinux.0
-rw-rw-rw-  1 nobody nobody 505802 May 15 15:51 sda2.log
-rw-rw-rw-  1 nobody nobody   2475 May 15 16:37 switch-confg
[root@h102 ~]# head /var/lib/tftpboot/switch-confg 

!
version 12.2
no service pad
service timestamps debug uptime
service timestamps log uptime
no service password-encryption
!
hostname Switch
!
[root@h102 ~]#
~~~

> **Tip:**  这个过程可能因为防火墙的问题导致没法上传成功，如果发现了类似 **`time out`** 的报错，可以尝试通过打开服务器防火墙来解决，防火墙的检查方法为 **`iptables -L -n`**  防火墙的配置文件为 **`/etc/sysconfig/iptables`**


---

## 编写自动备份脚本

~~~
[root@h102 backup_script]# vim conf_backup.pl 
[root@h102 backup_script]# cat conf_backup.pl 
#!/usr/bin/perl
#2017.05.15
#by wilmosfang
#used to save switch running-config 
use Expect;


#connection info
$username="admin"; 		#swith login name 
$password="admin123"; 		#switch login password
$switchip="192.168.90.1"; 	#switch ip 
$serverip="192.168.90.3"; 	#tftp server ip 
#save file
$configname="3750-switch-confg"; #config save name 
#set time out
$timeout=10;



$exp = Expect->spawn("ssh -l $username $switchip");
$exp->expect($timeout,
        [ qr/\(yes\/no\)/i,sub { my $self = shift;$self->send("yes\n");exp_continue;}],
        [ qr/password:/i,sub { my $self = shift;$self->send("$password\n");exp_continue;}],
	[ qr/sec\)/,sub { my $self = shift;$self->hard_close();}],
        [ qr/Switch#$/,sub { my $self = shift;$self->send("copy running-config tftp\n");exp_continue;}],
	[ qr/Address or name of remote host \[\]\?/,sub { my $self = shift;$self->send("$serverip\n");exp_continue;}],
	[ qr/Destination filename \[switch-confg\]\?/,sub { my $self = shift;$self->send("$configname\n");exp_continue;}],
        );
[root@h102 backup_script]# ll conf_backup.pl 
-rw-r--r-- 1 root root 1033 May 15 18:21 conf_backup.pl
[root@h102 backup_script]# chmod +x conf_backup.pl 
[root@h102 backup_script]# ll conf_backup.pl 
-rwxr-xr-x 1 root root 1033 May 15 18:21 conf_backup.pl
[root@h102 backup_script]#
~~~

运行脚本进行测试

~~~
[root@h102 backup_script]# ./conf_backup.pl 
admin@192.168.90.1's password: 

Switch#copy running-config tftp
Address or name of remote host []? 192.168.90.3
Destination filename [switch-confg]? 3750-switch-confg
!!
2475 bytes copied in 1.133 secs (2184 bytes/sec)
Switch#[root@h102 backup_script]#
------
[root@h102 tftpboot]# ls
3750-switch-confg  memdisk   pxelinux.0  switch-confg
chain.c32          menu.c32  sda2.log
[root@h102 tftpboot]# head 3750-switch-confg 

!
version 12.2
no service pad
service timestamps debug uptime
service timestamps log uptime
no service password-encryption
!
hostname Switch
!
[root@h102 tftpboot]#
~~~

---

## 创建 git 仓库

~~~
[root@h102 tftpboot]# ls
3750-switch-confg  switch-confg
[root@h102 tftpboot]# git init . 
Initialized empty Git repository in /var/lib/tftpboot/.git/
[root@h102 tftpboot]# git add . 
[root@h102 tftpboot]# git commit -m "abc"
[master (root-commit) ed36342] abc
 Committer: root <root@h102.temp>
Your name and email address were configured automatically based
on your username and hostname. Please check that they are accurate.
You can suppress this message by setting them explicitly:

    git config --global user.name "Your Name"
    git config --global user.email you@example.com

If the identity used for this commit is wrong, you can fix it with:

    git commit --amend --author='Your Name <you@example.com>'

 2 files changed, 320 insertions(+), 0 deletions(-)
 create mode 100644 3750-switch-confg
 create mode 100644 switch-confg
[root@h102 tftpboot]# git log --oneline
ed36342 abc
[root@h102 tftpboot]# pwd
/var/lib/tftpboot
[root@h102 tftpboot]#
~~~

---

## 构建定时备份加版本控制脚本

~~~
[root@h102 bin]# vim git_commit.bash 
[root@h102 bin]# cat git_commit.bash 
#!/bin/bash
#2017.5.15
#by wilmosfang
#used to add to git 

target_dir="/var/lib/tftpboot"
ts=`date +%Y%m%d%H%M%S`
sync_cmd="/root/perl_script/backup_script/conf_backup.pl"
git_cmd="/usr/bin/git"


$sync_cmd
cd $target_dir
$git_cmd add . 
$git_cmd commit -am "$ts" 
[root@h102 bin]# chmod +x git_commit.bash 
[root@h102 bin]# ./git_commit.bash 
admin@192.168.90.1's password: 

Switch#copy running-config tftp
Address or name of remote host []? 192.168.90.3
Destination filename [switch-confg]? 3750-switch-confg
!!
2475 bytes copied in 1.158 secs (2137 bytes/sec)
Switch## On branch master
nothing to commit (working directory clean)
[root@h102 bin]# ll git_commit.bash 
-rwxr-xr-x 1 root root 266 May 15 20:01 git_commit.bash
[root@h102 bin]# 
~~~

---

## 备份测试

先修改一下目录中的文件内容

~~~
[root@h102 tftpboot]# echo test123 > abc
[root@h102 tftpboot]# ls
3750-switch-confg  abc
[root@h102 tftpboot]#
~~~

执行脚本

~~~
[root@h102 bin]# ./git_commit.bash 
admin@192.168.90.1's password: 

Switch#copy running-config tftp
Address or name of remote host []? 192.168.90.3
Destination filename [switch-confg]? 3750-switch-confg
!!
2475 bytes copied in 1.124 secs (2202 bytes/sec)
Switch#[master 1da38fd] 20170515201127
 Committer: root <root@h102.temp>
Your name and email address were configured automatically based
on your username and hostname. Please check that they are accurate.
You can suppress this message by setting them explicitly:

    git config --global user.name "Your Name"
    git config --global user.email you@example.com

If the identity used for this commit is wrong, you can fix it with:

    git commit --amend --author='Your Name <you@example.com>'

 1 files changed, 1 insertions(+), 1 deletions(-)
[root@h102 bin]# 
~~~

检查一下版本日志

~~~
[root@h102 tftpboot]# git log --oneline
1da38fd 20170515201127
ff2d1fd 20170515193720
301e3d0 20170515193633
ed36342 abc
[root@h102 tftpboot]#
~~~

---

## 设置定时任务


~~~
[root@h102 bin]# crontab  -l 
*/1 * * * * /root/bin/git_commit.bash  2>&1  1> /dev/null 
[root@h102 bin]# 
~~~

检查效果

~~~
[root@h102 tftpboot]# git log --oneline
1da38fd 20170515201127
ff2d1fd 20170515193720
301e3d0 20170515193633
ed36342 abc
[root@h102 tftpboot]# git log --oneline
3aa6ebb 20170515201901
1da38fd 20170515201127
ff2d1fd 20170515193720
301e3d0 20170515193633
ed36342 abc
[root@h102 tftpboot]#
[root@h102 tftpboot]# git diff 3aa6ebb 1da38fd
diff --git a/abc b/abc
index 0e4b0c7..5271a52 100644
--- a/abc
+++ b/abc
@@ -1 +1 @@
-abc123
+test123
[root@h102 tftpboot]#
~~~

过五分钟

~~~
[root@h102 tftpboot]# git log --oneline
3aa6ebb 20170515201901
1da38fd 20170515201127
ff2d1fd 20170515193720
301e3d0 20170515193633
ed36342 abc
[root@h102 tftpboot]#
~~~

可见没有变化

说明如果这个目录下的文件内容有变化，最长经过一分钟，就会生成一个以当前时间戳为标记的新版本，如果没有变化，则不会产生新的版本

期间如果交换机的运行配置发生了变化，最多经过一分钟也会导致一个新版本的产生

这个一分钟的间隙，可以根据具体生产环境而进行调整


---


[cpan]:http://www.cpan.org/SITES.html
