---
layout: post
title: Centos6 下 logrotate 探源
author: wilmosfang
tags:  linux log
categories:  log
wc: 749 3635 28148
excerpt: logrotate 执行时间的探源
comments: true
---

# 前言

有一天发现自动生成的文件归档和cron job中的时间不一致，于是想弄清原因，最后找到了原因，是由系统下面logrotate来负责生成的，下面对logrotate的执行时间进行探源。


---

# 概要

* TOC
{:toc}


---

## Centos5 与Centos6的区别

这是Centos5下面的crontab文件

~~~
[test@Test ~]$ cat /etc/crontab 
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root
HOME=/

# run-parts
01 * * * * root run-parts /etc/cron.hourly
02 4 * * * root run-parts /etc/cron.daily
22 4 * * 0 root run-parts /etc/cron.weekly
42 4 1 * * root run-parts /etc/cron.monthly
[test@Test ~]$ 

~~~

这是Centos6下面的crontab文件

~~~
[root@Test-slave ~]# cat /etc/crontab  
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root
HOME=/

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed

[root@Test-slave ~]# 
~~~

可知对于Centos6每日的计划任务不在crontab里定义

实际定义位置是anacrontab

~~~
[root@Test-slave ~]# cat /etc/anacrontab 
# /etc/anacrontab: configuration file for anacron

# See anacron(8) and anacrontab(5) for details.

SHELL=/bin/sh
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root
# the maximal random delay added to the base delay of the jobs
RANDOM_DELAY=45
# the jobs will be started during the following hours only
START_HOURS_RANGE=3-22

#period in days   delay in minutes   job-identifier   command
1	5	cron.daily		nice run-parts /etc/cron.daily
7	25	cron.weekly		nice run-parts /etc/cron.weekly
@monthly 45	cron.monthly		nice run-parts /etc/cron.monthly
[root@Test-slave ~]# 
~~~
它的意义是:每一天的3点到22点期间随机延时45分钟+5分钟，执行/etc/cron.daily中的任务，通知给root
(此处只对daily work进行解释，其它类推)

然后我们看看/etc/cron.daily下的内容

~~~
[root@Test-slave ~]# ll /etc/cron.daily
total 28
-rwx------. 1 root root  118 Oct 15 23:33 cups
-rwxr-xr-x. 1 root root  196 Jul 18  2013 logrotate
-rwxr-xr-x. 1 root root  905 Feb 22  2013 makewhatis.cron
-rwxr-xr-x. 1 root root  174 Sep 24  2012 mlocate.cron
-rwxr-xr-x. 1 root root 2126 Jul 19  2013 prelink
-rwxr-xr-x. 1 root root  563 Nov 23  2013 readahead.cron
-rwxr-xr-x. 1 root root  365 Oct 16  2009 tmpwatch
[root@Test-slave ~]# 
~~~
我们发现其中就有logrotate

打开看看里面什么内容

~~~
[root@Test-slave ~]# cat  /etc/cron.daily/logrotate 
#!/bin/sh

/usr/sbin/logrotate /etc/logrotate.conf >/dev/null 2>&1
EXITVALUE=$?
if [ $EXITVALUE != 0 ]; then
    /usr/bin/logger -t logrotate "ALERT exited abnormally with [$EXITVALUE]"
fi
exit 0
[root@Test-slave ~]# 
~~~

继续打开配置文件/etc/logrotate.conf

~~~
[root@Test-slave ~]# cat /etc/logrotate.conf 
# see "man logrotate" for details
# rotate log files weekly
weekly

# keep 4 weeks worth of backlogs
rotate 4

# create new (empty) log files after rotating old ones
create

# use date as a suffix of the rotated file
dateext

# uncomment this if you want your log files compressed
#compress

# RPM packages drop log rotation information into this directory
include /etc/logrotate.d

# no packages own wtmp and btmp -- we'll rotate them here
/var/log/wtmp {
    monthly
    create 0664 root utmp
	minsize 1M
    rotate 1
}

/var/log/btmp {
    missingok
    monthly
    create 0600 root utmp
    rotate 1
}

# system-specific logs may be also be configured here.
[root@Test-slave ~]# 
~~~

里面还有一个配置文件目录/etc/logrotate.d ,打开它，看看究竟有些什么 

~~~
[root@Test-slave ~]# ll  /etc/logrotate.d 
total 44
-rw-r--r--. 1 root root  71 Oct 15 23:33 cups
-rw-r--r--. 1 root root 103 Oct 16 05:49 dracut
-rw-r--r--. 1 root root 185 Aug 15  2014 httpd
-rw-r--r--. 1 root root 844 Mar  3 01:43 mysql
-rw-r--r--. 1 root root 136 Aug 23  2010 ppp
-rw-r--r--. 1 root root 329 Jul 17  2012 psacct
-rw-r--r--. 1 root root 210 Aug 15  2013 syslog
-rw-r--r--. 1 root root 100 Feb  4  2013 wpa_supplicant
-rw-r--r--. 1 root root  87 Oct 16 23:15 yum
-rw-r--r--. 1 root root 125 Jan 29  2014 zabbix-agent
-rw-r--r--. 1 root root 140 Jan 29  2014 zabbix-server
[root@Test-slave ~]# 
~~~

里面有对不同服务的定制配置，这是一种方便管理的方法，在apache，nginx等里面都是这样的, 我们随便打开一个如mysql,看看里面有些什么 

~~~
[root@Test-slave ~]# cat /etc/logrotate.d/mysql 
# The log file name and location can be set in
# /etc/my.cnf by setting the "log-error" option
# in either [mysqld] or [mysqld_safe] section as
# follows:
#
# [mysqld]
# log-error=/var/lib/mysql/mysqld.log
#
# In case the root user has a password, then you
# have to create a /root/.my.cnf configuration file
# with the following content:
#
# [mysqladmin]
# password = <secret> 
# user= root
#
# where "<secret>" is the password. 
#
# ATTENTION: The /root/.my.cnf file should be readable
# _ONLY_ by root !

/var/lib/mysql/mysqld.log {
        # create 600 mysql mysql
        notifempty
        daily
        rotate 5
        missingok
        compress
    postrotate
	# just if mysqld is really running
	if test -x /usr/bin/mysqladmin && \
	   /usr/bin/mysqladmin ping &>/dev/null
	then
	   /usr/bin/mysqladmin flush-logs
	fi
    endscript
}
[root@Test-slave ~]# 
~~~
这么一条龙下来，已经对logrotate的执行时间点有了一个直观的感受

# 附:

下面是logrotate的man page,上面的各种参数就可以扫盲了

~~~
LOGROTATE(8)             System Administrator’s Manual            LOGROTATE(8)



NAME
       logrotate - rotates, compresses, and mails system logs

SYNOPSIS
       logrotate [-dv] [-f|--force] [-s|--state file] config_file ..

DESCRIPTION
       logrotate  is  designed to ease administration of systems that generate
       large numbers of log files.  It allows automatic rotation, compression,
       removal, and mailing of log files.  Each log file may be handled daily,
       weekly, monthly, or when it grows too large.

       Normally, logrotate is run as a daily cron job.  It will not  modify  a
       log  multiple  times  in  one  day unless the criterion for that log is
       based on the log’s size and logrotate is being run multiple times  each
       day, or unless the -f or --force option is used.

       Any number of config files may be given on the command line. Later con-
       fig files may override the options given in earlier files, so the order
       in which the logrotate config files are listed is important.  Normally,
       a single config file which includes any other config  files  which  are
       needed  should  be  used.  See below for more information on how to use
       the include directive to accomplish this.  If a directory is  given  on
       the  command  line,  every  file  in that directory is used as a config
       file.

       If no command line arguments are given, logrotate  will  print  version
       and  copyright  information,  along with a short usage summary.  If any
       errors occur while rotating logs, logrotate  will  exit  with  non-zero
       status.


OPTIONS
       -d, --debug
              Turns  on  debug mode and implies -v.  In debug mode, no changes
              will be made to the logs or to the logrotate state file.


       -f, --force
              Tells logrotate to force the rotation, even if it doesn’t  think
              this  is  necessary.   Sometimes this is useful after adding new
              entries to a logrotate config file, or if  old  log  files  have
              been removed by hand, as the new files will be created, and log-
              ging will continue correctly.


       -m, --mail <command>
              Tells logrotate which command to use  when  mailing  logs.  This
              command  should accept two arguments: 1) the subject of the mes-
              sage, and 2) the recipient. The command must then read a message
              on standard input and mail it to the recipient. The default mail
              command is /bin/mail -s.


       -s, --state <statefile>
              Tells logrotate to use an alternate state file.  This is  useful
              if  logrotate  is being run as a different user for various sets
              of log files.  The default state file is /var/lib/logrotate.sta-
              tus.


       --usage
              Prints a short usage message.


       --?, --help
              -Prints help message.


       -v, --verbose
              Turns on verbose mode.


CONFIGURATION FILE
       logrotate  reads  everything  about the log files it should be handling
       from the series of configuration files specified on the  command  line.
       Each configuration file can set global options (local definitions over-
       ride global ones, and later  definitions  override  earlier  ones)  and
       specify  logfiles  to  rotate.  A  simple configuration file looks like
       this:

       # sample logrotate configuration file
       compress

       /var/log/messages {
           rotate 5
           weekly
           postrotate
               /usr/bin/killall -HUP syslogd
           endscript
       }

       "/var/log/httpd/access.log" /var/log/httpd/error.log {
           rotate 5
           mail www@my.org
           size 100k
           sharedscripts
           postrotate
               /usr/bin/killall -HUP httpd
           endscript
       }

       /var/log/news/* {
           monthly
           rotate 2
           olddir /var/log/news/old
           missingok
           postrotate
               kill -HUP ‘cat /var/run/inn.pid‘
           endscript
           nocompress
       }


       The first few lines set global options; in the example, logs  are  com-
       pressed after they are rotated.  Note that comments may appear anywhere
       in the config file as long as the first non-whitespace character on the
       line is a #.

       The next section of the config files defined how to handle the log file
       /var/log/messages. The log will go through five weekly rotations before
       being  removed. After the log file has been rotated (but before the old
       version of the log has been compressed), the command /sbin/killall -HUP
       syslogd will be executed.

       The     next     section    defines    the    parameters    for    both
       /var/log/httpd/access.log  and  /var/log/httpd/error.log.    They   are
       rotated whenever it grows over 100k in size, and the old logs files are
       mailed (uncompressed) to www@my.org after going  through  5  rotations,
       rather  than being removed. The sharedscripts means that the postrotate
       script will only be run once (after the old logs have been compressed),
       not  once  for  each  log which is rotated. Note that the double quotes
       around the first filename at  the  beginning  of  this  section  allows
       logrotate  to rotate logs with spaces in the name. Normal shell quoting
       rules apply, with ’, ", and \ characters supported.

       The last section defines  the  parameters  for  all  of  the  files  in
       /var/log/news.  Each  file is rotated on a monthly basis.  This is con-
       sidered a single rotation directive and if errors occur for  more  than
       one file, the log files are not compressed.

       Please  use  wildcards  with caution.  If you specify *, logrotate will
       rotate all files, including previously rotated ones.  A way around this
       is  to  use  the  olddir  directive  or  a more exact wildcard (such as
       *.log).

       Here is more information on the directives which may be included  in  a
       logrotate configuration file:


       compress
              Old  versions  of  log  files  are  compressed  with  gzip(1) by
              default. See also nocompress.


       compresscmd
              Specifies which command to  use  to  compress  log  files.   The
              default is gzip.  See also compress.


       uncompresscmd
              Specifies  which  command  to  use to uncompress log files.  The
              default is gunzip.


       compressext
              Specifies which extension to use on compressed logfiles, if com-
              pression is enabled.  The default follows that of the configured
              compression command.


       compressoptions
              Command line options may be passed to the  compression  program,
              if  one  is  in use.  The default, for gzip(1), is "-9" (maximum
              compression).


       copy   Make a copy of the log file, but don’t change  the  original  at
              all.   This option can be used, for instance, to make a snapshot
              of the current log file, or when some  other  utility  needs  to
              truncate  or parse the file.  When this option is used, the cre-
              ate option will have no effect, as the old  log  file  stays  in
              place.


       copytruncate
              Truncate  the  original log file in place after creating a copy,
              instead of moving the old log file and optionally creating a new
              one.   It  can be used when some program cannot be told to close
              its logfile and thus might continue writing (appending)  to  the
              previous log file forever.  Note that there is a very small time
              slice between copying the file and truncating it, so  some  log-
              ging  data  might be lost.  When this option is used, the create
              option will have no effect, as the old log file stays in  place.


       create mode owner group, create owner group
              Immediately after rotation (before the postrotate script is run)
              the log file is created (with the same name as the log file just
              rotated).   mode  specifies  the  mode for the log file in octal
              (the same as chmod(2)), owner specifies the user name  who  will
              own  the  log  file,  and group specifies the group the log file
              will belong to. Any of the log file attributes may  be  omitted,
              in  which  case  those  attributes for the new file will use the
              same values as the original log file for the omitted attributes.
              This option can be disabled using the nocreate option.


       daily  Log files are rotated every day.


       dateext
              Archive  old versions of log files adding a daily extension like
              YYYYMMDD instead of simply adding a number. The extension may be
              configured using the dateformat option.


       dateformat format_string
              Specify  the extension for dateext using the notation similar to
              strftime(3) function. Only  %Y  %m  %d  and  %s  specifiers  are
              allowed.  The default value is -%Y%m%d. Note that also the char-
              acter separating log name from the  extension  is  part  of  the
              dateformat  string.  The  system  clock must be set past Sep 9th
              2001 for %s to work correctly.  Note that the datestamps  gener-
              ated  by this format must be lexically sortable (i.e., first the
              year, then the month then the day. e.g., 2001/12/01 is  ok,  but
              01/12/2001 is not, since 01/11/2002 would sort lower while it is
              later).  This is because when using the rotate option, logrotate
              sorts all rotated filenames to find out which logfiles are older
              and should be removed.


       delaycompress
              Postpone compression of the previous log file to the next  rota-
              tion  cycle.  This only has effect when used in combination with
              compress.  It can be used when some program cannot  be  told  to
              close  its logfile and thus might continue writing to the previ-
              ous log file for some time.


       extension ext
              Log files with ext extension can keep it after the rotation.  If
              compression  is  used,  the compression extension (normally .gz)
              appears  after  ext.  For  example  you  have  a  logfile  named
              mylog.foo  and  want  to  rotate it to mylog.1.foo.gz instead of
              mylog.foo.1.gz.


       ifempty
              Rotate the  log  file  even  if  it  is  empty,  overriding  the
              notifempty option (ifempty is the default).


       include file_or_directory
              Reads the file given as an argument as if it was included inline
              where the include directive appears. If a  directory  is  given,
              most of the files in that directory are read in alphabetic order
              before processing of the  including  file  continues.  The  only
              files  which  are  ignored are files which are not regular files
              (such as directories and named pipes) and files whose names  end
              with  one  of the taboo extensions, as specified by the tabooext
              directive.  The include directive may not appear  inside  a  log
              file definition.


       mail address
              When a log is rotated out-of-existence, it is mailed to address.
              If no mail should be generated by a particular log,  the  nomail
              directive may be used.


       mailfirst
              When using the mail command, mail the just-rotated file, instead
              of the about-to-expire file.


       maillast
              When using the mail  command,  mail  the  about-to-expire  file,
              instead of the just-rotated file (this is the default).


       maxage count
              Remove  rotated  logs  older  than <count> days. The age is only
              checked if the logfile is to be rotated. The files are mailed to
              the configured address if maillast and mail are configured.


       minsize size
              Log files are rotated when they grow bigger than size bytes, but
              not before the  additionally  specified  time  interval  (daily,
              weekly, monthly, or yearly).  The related size option is similar
              except that it is mutually  exclusive  with  the  time  interval
              options,  and  it  causes log files to be rotated without regard
              for the last rotation time.  When minsize is used, both the size
              and timestamp of a log file are considered.


       missingok
              If  the log file is missing, go on to the next one without issu-
              ing an error message. See also nomissingok.


       monthly
              Log files are rotated the first time logrotate is run in a month
              (this is normally on the first day of the month).


       nocompress
              Old versions of log files are not compressed. See also compress.


       nocopy Do not copy the original log file and leave it in place.   (this
              overrides the copy option).


       nocopytruncate
              Do  not truncate the original log file in place after creating a
              copy (this overrides the copytruncate option).


       nocreate
              New log  files  are  not  created  (this  overrides  the  create
              option).


       nodelaycompress
              Do not postpone compression of the previous log file to the next
              rotation cycle (this overrides the delaycompress option).


       nodateext
              Do not archive  old versions of log files  with  date  extension
              (this overrides the dateext option).


       nomail Don’t mail old log files to any address.


       nomissingok
              If  a  log  file  does  not  exist,  issue an error. This is the
              default.


       noolddir
              Logs are rotated in the same directory the log normally  resides
              in (this overrides the olddir option).


       nosharedscripts
              Run prerotate and postrotate scripts for every log file which is
              rotated (this is the default, and  overrides  the  sharedscripts
              option).  The  absolute  path to the log file is passed as first
              argument to the script. If the  scripts  exit  with  error,  the
              remaining  actions  will  not  be  executed for the affected log
              only.


       noshred
              Do not use shred when deleting old log files. See also shred.


       notifempty
              Do not rotate the log if it is empty (this overrides the ifempty
              option).


       olddir directory
              Logs  are  moved into directory for rotation. The directory must
              be on the same physical device as the log  file  being  rotated,
              and  is  assumed to be relative to the directory holding the log
              file unless an absolute path name is specified. When this option
              is  used  all old versions of the log end up in directory.  This
              option may be overridden by the noolddir option.


       postrotate/endscript
              The lines between postrotate and endscript (both of  which  must
              appear  on  lines  by  themselves)  are executed (using /bin/sh)
              after the log file is rotated. These directives may only  appear
              inside a log file definition. Normally, the absolute path to the
              log file is passed as first argument to the script.  If  shared-
              scripts  is  specified,  whole  pattern is passed to the script.
              See also prerotate. See sharedscripts  and  nosharedscripts  for
              error handling.


       prerotate/endscript
              The  lines  between  prerotate and endscript (both of which must
              appear on lines by  themselves)  are  executed  (using  /bin/sh)
              before the log file is rotated and only if the log will actually
              be rotated. These directives may only appear inside a  log  file
              definition.  Normally,  the  absolute  path  to  the log file is
              passed as first argument to the script.   If   sharedscripts  is
              specified,  whole  pattern  is  passed  to the script.  See also
              postrotate.  See sharedscripts  and  nosharedscripts  for  error
              handling.


       firstaction/endscript
              The  lines between firstaction and endscript (both of which must
              appear on lines by themselves) are executed (using /bin/sh) once
              before  all  log  files  that  match  the wildcarded pattern are
              rotated, before prerotate script is run and only if at least one
              log  will actually be rotated.  These directives may only appear
              inside a log file definition. Whole pattern  is  passed  to  the
              script  as  first  argument.  If the script exits with error, no
              further processing is done. See also lastaction.


       lastaction/endscript
              The lines between lastaction and endscript (both of  which  must
              appear on lines by themselves) are executed (using /bin/sh) once
              after all log  files  that  match  the  wildcarded  pattern  are
              rotated, after postrotate script is run and only if at least one
              log is rotated. These directives may only appear  inside  a  log
              file  definition. Whole pattern is passed to the script as first
              argument. If the script exits with error, just an error  message
              is shown (as this is the last action). See also firstaction.


       rotate count
              Log files are rotated count times before being removed or mailed
              to the address specified in a mail directive. If count is 0, old
              versions are removed rather than rotated.


       size size
              Log  files are rotated only if they grow bigger then size bytes.
              If size is followed by k, the size is assumed  to  be  in  kilo-
              bytes.   If the M is used, the size is in megabytes, and if G is
              used, the size is in gigabytes. So size  100,  size  100k,  size
              100M and size 100Gare all valid.


       sharedscripts
              Normally,  prerotate and postrotate scripts are run for each log
              which is rotated and the absolute path to the log file is passed
              as  first argument to the script. That means a single script may
              be run multiple times for log file entries which match  multiple
              files (such as the /var/log/news/* example). If sharedscripts is
              specified, the scripts are only run once,  no  matter  how  many
              logs  match  the wildcarded pattern, and whole pattern is passed
              to them.  However, if none of the logs in  the  pattern  require
              rotating,  the  scripts  will  not be run at all. If the scripts
              exit with error, the remaining actions will not be executed  for
              any  logs.  This option overrides the nosharedscripts option and
              implies create option.


       shred  Delete log files using  shred  -u  instead  of  unlink().   This
              should  ensure  that logs are not readable after their scheduled
              deletion; this is off by default.  See also noshred.


       shredcycles count
              Asks GNU shred(1) to overwite log files count times before dele-
              tion.  Without this option, shred’s default will be used.


       start count
              This is the number to use as the base for rotation. For example,
              if you specify 0, the logs will be created with a  .0  extension
              as they are rotated from the original log files.  If you specify
              9, log files will be created with a  .9,  skipping  0-8.   Files
              will  still  be  rotated  the number of times specified with the
              count directive.


       tabooext [+] list
              The current taboo extension list is  changed  (see  the  include
              directive  for information on the taboo extensions). If a + pre-
              cedes the list of extensions, the current taboo  extension  list
              is  augmented,  otherwise  it is replaced. At startup, the taboo
              extension list contains .rpmorig, .rpmsave, ,v,  .swp,  .rpmnew,
              ~, .cfsaved and .rhn-cfg-tmp-*.


       weekly Log  files  are  rotated if the current weekday is less than the
              weekday of the last rotation or if more than a week  has  passed
              since  the  last rotation. This is normally the same as rotating
              logs on the first day of the week, but it works better if logro-
              tate is not run every night.


       yearly Log files are rotated if the current year is not the same as the
              last rotation.


FILES
       /var/lib/logrotate.status  Default state file.
       /etc/logrotate.conf        Configuration options.

SEE ALSO
       gzip(1)

       <http://fedorahosted.org/logrotate/>

AUTHORS
       Erik Troan, Preston Brown, Jan Kaluza.

       <logrotate-owner@fedoraproject.org>




Linux                           Wed Nov 5 2002                    LOGROTATE(8)
~~~



