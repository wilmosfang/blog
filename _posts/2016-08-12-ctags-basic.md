---
layout:  post
title:  CTAGS 基础
author:  wilmosfang
tags:   ctags 
categories:  ctags  script
wc:  906  2137 23377 
excerpt: ctags 的安装，支持的语言，语言的映射，可识别的语法和对象，生成tags文件，定位，跳转，相同标签跳转，tag补全，tags历史记录 
comments: true
---


# 前言


长期的运维工作中难免会遇到需要查看脚本或工具源码的情况，这时单纯地使用文本编辑器来检索与跳转就很不方便了，如果有方法可以对代码进行索引就能很明显提升定位效率，减少垃圾时间，将注意力更多分配到有价值的事情上

**[ctags][ctags]** 正是用来应对此种需求的

**[ctags][ctags]** 可以在源码的基础上生成一份索引文件(标记体系)，然后提供给其它编辑器使用，以简单快速地定位这些被索引的对象和条目

**[ctags][ctags]** 目前可以支持多种语言，可以参考 **[programming languages][languages]** ，也可以支持多种工具和编辑器，可以参考 **[Editors and Tools Supporting CTAGS][tools]**

这里分享一下 **[ctags][ctags]** 相关基础，详细可以参考 **[官方文档][ctags]**
 
 
> **Tip:** 当前最新版本为 **Version 5.8**  发布于 09 July 2009，**http://ctags.sourceforge.net/** 可能需要翻墙才能访问


---


# 概要

* TOC
{:toc}



---

## 环境

~~~
[root@h102 ~]# cat /etc/issue
CentOS release 6.6 (Final)
Kernel \r on an \m

[root@h102 ~]# uname -a 
Linux h102.temp 2.6.32-504.el6.x86_64 #1 SMP Wed Oct 15 04:27:16 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
[root@h102 ~]#
~~~


---


## 安装

由于 ctags 太好用，正常情况下都集成到了各 Linux 发行版本的基础库中

如果没有安装，这里直接使用 yum 进行安装

~~~
[root@h102 ~]# yum install ctags
Loaded plugins: dellsysid, fastestmirror, refresh-packagekit, security
Setting up Install Process
Loading mirror speeds from cached hostfile
 * base: mirrors.aliyun.com
 * epel: mirrors.ustc.edu.cn
 * extras: mirror.bit.edu.cn
 * updates: mirrors.aliyun.com
Resolving Dependencies
--> Running transaction check
---> Package ctags.x86_64 0:5.8-2.el6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

======================================================================================================================================
 Package                       Arch                           Version                              Repository                    Size
======================================================================================================================================
Installing:
 ctags                         x86_64                         5.8-2.el6                            base                         147 k

Transaction Summary
======================================================================================================================================
Install       1 Package(s)

Total download size: 147 k
Installed size: 339 k
Is this ok [y/N]: y
Downloading Packages:
ctags-5.8-2.el6.x86_64.rpm                                                                                     | 147 kB     00:00     
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
  Installing : ctags-5.8-2.el6.x86_64                                                                                             1/1 
  Verifying  : ctags-5.8-2.el6.x86_64                                                                                             1/1 

Installed:
  ctags.x86_64 0:5.8-2.el6                                                                                                            

Complete!
[root@h102 ~]#
~~~

查看版本

~~~
[root@h102 ~]# ctags --version
Exuberant Ctags 5.8, Copyright (C) 1996-2009 Darren Hiebert
  Compiled: Nov 11 2010, 03:54:52
  Addresses: <dhiebert@users.sourceforge.net>, http://ctags.sourceforge.net
  Optional compiled features: +wildcards, +regex
[root@h102 ~]# 
~~~


---

## 支持的语言

~~~
[root@h102 ~]# ctags --list-languages
Ant
Asm
Asp
Awk
Basic
BETA
C
C++
C#
Cobol
DosBatch
Eiffel
Erlang
Flex
Fortran
HTML
Java
JavaScript
Lisp
Lua
Make
MatLab
OCaml
Pascal
Perl
PHP
Python
REXX
Ruby
Scheme
Sh
SLang
SML
SQL
Tcl
Tex
Vera
Verilog
VHDL
Vim
YACC
[root@h102 ~]#
~~~

---

## 文件后缀与语言的映射

~~~
[root@h102 ~]# ctags --list-maps
Ant      *.build.xml
Asm      *.asm *.ASM *.s *.S *.A51 *.29[kK] *.[68][68][kKsSxX] *.[xX][68][68]
Asp      *.asp *.asa
Awk      *.awk *.gawk *.mawk
Basic    *.bas *.bi *.bb *.pb
BETA     *.bet
C        *.c
C++      *.c++ *.cc *.cp *.cpp *.cxx *.h *.h++ *.hh *.hp *.hpp *.hxx *.C *.H
C#       *.cs
Cobol    *.cbl *.cob *.CBL *.COB
DosBatch *.bat *.cmd
Eiffel   *.e
Erlang   *.erl *.ERL *.hrl *.HRL
Flex     *.as *.mxml
Fortran  *.f *.for *.ftn *.f77 *.f90 *.f95 *.F *.FOR *.FTN *.F77 *.F90 *.F95
HTML     *.htm *.html
Java     *.java
JavaScript *.js
Lisp     *.cl *.clisp *.el *.l *.lisp *.lsp
Lua      *.lua
Make     *.mak *.mk [Mm]akefile GNUmakefile
MatLab   *.m
OCaml    *.ml *.mli
Pascal   *.p *.pas
Perl     *.pl *.pm *.plx *.perl
PHP      *.php *.php3 *.phtml
Python   *.py *.pyx *.pxd *.pxi *.scons
REXX     *.cmd *.rexx *.rx
Ruby     *.rb *.ruby
Scheme   *.SCM *.SM *.sch *.scheme *.scm *.sm
Sh       *.sh *.SH *.bsh *.bash *.ksh *.zsh
SLang    *.sl
SML      *.sml *.sig
SQL      *.sql
Tcl      *.tcl *.tk *.wish *.itcl
Tex      *.tex
Vera     *.vr *.vri *.vrh
Verilog  *.v
VHDL     *.vhdl *.vhd
Vim      *.vim
YACC     *.y
[root@h102 ~]#
~~~

> **Tip:** 不过这个映射可以使用 **`--langmap`** 进行修改

如果不使用 **`−−language−force`** 进行语言指定，ctags 会根据默认的映射来解析带后缀的源文件，如果此源文件后缀没有包含在映射列表里，就会读取文件的第一行，包含 **`#!`** 的内容来判定语言


---

## 可识别的语法或对象


**`ctags --list-kinds`** 可以查看每种语言的哪些语法可以被识别

~~~
[root@h102 ~]# ctags --list-kinds
Ant
    p  projects 
    t  targets 
Asm
    d  defines
    l  labels
    m  macros
    t  types (structs and records)
Asp
    d  constants
    c  classes
    f  functions
    s  subroutines
    v  variables
Awk
    f  functions
Basic
    c  constants
    f  functions
    l  labels
    t  types
    v  variables
    g  enumerations
BETA
    f  fragment definitions
    p  all patterns [off]
    s  slots (fragment uses)
    v  patterns (virtual or rebound)
C
    c  classes
    d  macro definitions
    e  enumerators (values inside an enumeration)
    f  function definitions
    g  enumeration names
    l  local variables [off]
    m  class, struct, and union members
    n  namespaces
    p  function prototypes [off]
    s  structure names
    t  typedefs
    u  union names
    v  variable definitions
    x  external and forward variable declarations [off]
C++
    c  classes
    d  macro definitions
    e  enumerators (values inside an enumeration)
    f  function definitions
    g  enumeration names
    l  local variables [off]
    m  class, struct, and union members
    n  namespaces
    p  function prototypes [off]
    s  structure names
    t  typedefs
    u  union names
    v  variable definitions
    x  external and forward variable declarations [off]
C#
    c  classes
    d  macro definitions
    e  enumerators (values inside an enumeration)
    E  events
    f  fields
    g  enumeration names
    i  interfaces
    l  local variables [off]
    m  methods
    n  namespaces
    p  properties
    s  structure names
    t  typedefs
Cobol
    d  data items 
    f  file descriptions (FD, SD, RD) 
    g  group items 
    p  paragraphs 
    P  program ids 
    s  sections 
DosBatch
    l  labels 
    v  variables 
Eiffel
    c  classes
    f  features
    l  local entities [off]
Erlang
    d  macro definitions
    f  functions
    m  modules
    r  record definitions
Flex
    f  functions
    c  classes
    m  methods
    p  properties
    v  global variables
    x  mxtags
Fortran
    b  block data
    c  common blocks
    e  entry points
    f  functions
    i  interface contents, generic names, and operators [off]
    k  type and structure components
    l  labels
    L  local, common block, and namelist variables [off]
    m  modules
    n  namelists
    p  programs
    s  subroutines
    t  derived types and structures
    v  program (global) and module variables
HTML
    a  named anchors 
    f  JavaScript functions 
Java
    c  classes
    e  enum constants
    f  fields
    g  enum types
    i  interfaces
    l  local variables [off]
    m  methods
    p  packages
JavaScript
    f  functions
    c  classes
    m  methods
    p  properties
    v  global variables
Lisp
    f  functions
Lua
    f  functions
Make
    m  macros
MatLab
    f  function 
    f  function 
    f  function 
OCaml
    c  classes
    m  Object's method
    M  Module or functor
    v  Global variable
    t  Type name
    f  A function
    C  A constructor
    r  A 'structure' field
    e  An exception
Pascal
    f  functions
    p  procedures
Perl
    c  constants
    f  formats
    l  labels
    p  packages
    s  subroutines
    d  subroutine declarations [off]
PHP
    c  classes 
    i  interfaces 
    d  constant definitions 
    f  functions 
    v  variables 
    v  variables 
    j  javascript functions 
    j  javascript functions 
    j  javascript functions 
Python
    c  classes
    f  functions
    m  class members
    v  variables
    i  imports
REXX
    s  subroutines 
Ruby
    c  classes
    f  methods
    m  modules
    F  singleton methods
Scheme
    f  functions
    s  sets
Sh
    f  functions
SLang
    f  functions 
    n  namespaces 
SML
    e  exception declarations
    f  function definitions
    c  functor definitions
    s  signature declarations
    r  structure declarations
    t  type definitions
    v  value bindings
SQL
    c  cursors
    d  prototypes [off]
    f  functions
    F  record fields
    l  local variables [off]
    L  block label
    P  packages
    p  procedures
    r  records [off]
    s  subtypes
    t  tables
    T  triggers
    v  variables
    i  indexes
    e  events
    U  publications
    R  services
    D  domains
    V  views
    n  synonyms
    x  MobiLink Table Scripts
    y  MobiLink Conn Scripts
Tcl
    c  classes
    m  methods
    p  procedures
Tex
    c  chapters
    s  sections
    u  subsections
    b  subsubsections
    p  parts
    P  paragraphs
    G  subparagraphs
Vera
    c  classes
    d  macro definitions
    e  enumerators (values inside an enumeration)
    f  function definitions
    g  enumeration names
    l  local variables [off]
    m  class, struct, and union members
    p  programs
    P  function prototypes [off]
    t  tasks
    T  typedefs
    v  variable definitions
    x  external variable declarations [off]
Verilog
    c  constants (define, parameter, specparam)
    e  events
    f  functions
    m  modules
    n  net data types
    p  ports
    r  register data types
    t  tasks
VHDL
    c  constant declarations
    t  type definitions
    T  subtype definitions
    r  record names
    e  entity declarations
    C  component declarations [off]
    d  prototypes [off]
    f  function prototypes and declarations
    p  procedure prototypes and declarations
    P  package definitions
    l  local definitions [off]
Vim
    a  autocommand groups
    c  user-defined commands
    f  function definitions
    m  maps
    v  variable definitions
YACC
    l  labels 
[root@h102 ~]#
~~~


---

## 生成 tags 文件


使用 **`-R/--recurse`** 在源文件根下执行，就会生成 tags 文件



~~~
[root@h102 forklift_etl-1.2.2]# pwd
/usr/local/rvm/gems/ruby-2.3.0/gems/forklift_etl-1.2.2
[root@h102 forklift_etl-1.2.2]# ls
bin  example  forklift_etl.gemspec  forklift.jpg  Gemfile  Gemfile.lock  lib  LICENSE.txt  Rakefile  readme.md  spec  template
[root@h102 forklift_etl-1.2.2]# ctags -R *
[root@h102 forklift_etl-1.2.2]# ls
bin  example  forklift_etl.gemspec  forklift.jpg  Gemfile  Gemfile.lock  lib  LICENSE.txt  Rakefile  readme.md  spec  tags  template
[root@h102 forklift_etl-1.2.2]# file tags 
tags: Exuberant Ctags tag file text
[root@h102 forklift_etl-1.2.2]# wc tags 
  165  1206 16481 tags
[root@h102 forklift_etl-1.2.2]# head -n 20 tags 
!_TAG_FILE_FORMAT	2	/extended format; --format=1 will not append ;" to lines/
!_TAG_FILE_SORTED	1	/0=unsorted, 1=sorted, 2=foldcase/
!_TAG_PROGRAM_AUTHOR	Darren Hiebert	/dhiebert@users.sourceforge.net/
!_TAG_PROGRAM_NAME	Exuberant Ctags	//
!_TAG_PROGRAM_URL	http://ctags.sourceforge.net	/official site/
!_TAG_PROGRAM_VERSION	5.8	//
Base	lib/forklift/base/connection.rb	/^  module Base$/;"	m	class:Forklift
Base	lib/forklift/base/logger.rb	/^  module Base$/;"	m	class:Forklift
Base	lib/forklift/base/mailer.rb	/^  module Base$/;"	m	class:Forklift
Base	lib/forklift/base/pid.rb	/^  module Base$/;"	m	class:Forklift
Base	lib/forklift/base/utils.rb	/^  module Base$/;"	m	class:Forklift
Connection	lib/forklift/base/connection.rb	/^    class Connection$/;"	c	class:Forklift.Base
Connection	lib/forklift/transports/csv.rb	/^  module Connection$/;"	m	class:Forklift
Connection	lib/forklift/transports/elasticsearch.rb	/^  module Connection$/;"	m	class:Forklift
Connection	lib/forklift/transports/mysql.rb	/^  module Connection$/;"	m	class:Forklift
Csv	lib/forklift/transports/csv.rb	/^    class Csv < Forklift::Base::Connection$/;"	c	class:Forklift.Connection
ERBBinding	lib/forklift/base/mailer.rb	/^      class ERBBinding$/;"	c	class:Forklift.Base.Mailer
Elasticsearch	lib/forklift/patterns/elasticsearch_patterns.rb	/^    class Elasticsearch$/;"	c	class:Forklift.Patterns
Elasticsearch	lib/forklift/transports/elasticsearch.rb	/^    class Elasticsearch < Forklift::Base::Connection$/;"	c    class:Forklift.Connection
EmailSuffix	example/transformations/email_suffix.rb	/^class EmailSuffix$/;"	c
[root@h102 forklift_etl-1.2.2]#
~~~

> **Tip:** 在包含有tags文件的目录中，就可以直接使用 vim 进行定位了，也可以将 tags 文件写到 vimrc 中，这样就不用局限于当前路径

---

## 直接定位

使用 **`vim -t <tag>`** 来定位标记

~~~
[root@h102 forklift_etl-1.2.2]# vim -t forklift
...
...
...
~~~

会获得所有此标记的定义

~~~

  # pri kind tag               file
  1 F   f    forklift          lib/forklift/base/logger.rb
               class:Forklift.Base.Logger
               def forklift
  2 F   f    forklift          lib/forklift/base/mailer.rb
               class:Forklift.Base.Mailer
               def forklift
  3 F   f    forklift          lib/forklift/base/pid.rb
               class:Forklift.Base.Pid
               def forklift
  4 F   f    forklift          lib/forklift/transports/csv.rb
               class:Forklift.Connection.Csv
               def forklift
  5 F   f    forklift          lib/forklift/transports/elasticsearch.rb
               class:Forklift.Connection.Elasticsearch
               def forklift
  6 F   f    forklift          lib/forklift/transports/mysql.rb
               class:Forklift.Connection.Mysql
               def forklift
Type number and <Enter> (empty cancels): 
~~~

只用选择一个序号，就直接跳转过去了

---

## 跳转

当光标在 **generate** 时

~~~
...
...
if ['--generate', '-generate'].include?(ARGV[0])
  generate
else
  run_plan
end
...
...
~~~

使用 **`Ctrl + ]`** 可以瞬间跳转到光标所在的函数定义处

~~~
...
...
begin
  require 'forklift'
rescue LoadError
  require "#{File.expand_path(File.dirname(__FILE__))}/../lib/forklift.rb"
end

def generate
  p = Dir.pwd

  Dir.mkdir "#{p}/config"
  Dir.mkdir "#{p}/config/connections"
...
...
~~~

使用 **`Ctrl + T`** 又可以跳回来


---

## 文件里定位

在文件里也可以对标记定位

使用 **`:tag <tag>`**

比如: 对 **`forklift`** 标签进行定位

~~~
...
...
  Dir.mkdir "#{p}/template"
  Dir.mkdir "#{p}/transformations"
  Dir.mkdir "#{p}/transports"
:tag forklift  
~~~

~~~
...
...
  Dir.mkdir "#{p}/pid"
  Dir.mkdir "#{p}/template"
  Dir.mkdir "#{p}/transformations"
  Dir.mkdir "#{p}/transports"
  # pri kind tag               file                                                                                 
  1 F   f    forklift          lib/forklift/base/logger.rb
               class:Forklift.Base.Logger
               def forklift
  2 F   f    forklift          lib/forklift/base/mailer.rb
               class:Forklift.Base.Mailer
               def forklift
  3 F   f    forklift          lib/forklift/base/pid.rb
               class:Forklift.Base.Pid
               def forklift
  4 F   f    forklift          lib/forklift/transports/csv.rb
               class:Forklift.Connection.Csv
               def forklift
  5 F   f    forklift          lib/forklift/transports/elasticsearch.rb
               class:Forklift.Connection.Elasticsearch
               def forklift
  6 F   f    forklift          lib/forklift/transports/mysql.rb
               class:Forklift.Connection.Mysql
               def forklift
Type number and <Enter> (empty cancels): 
~~~

根据左边的数字，就可以快速跳转



---

## 相同标签快速跳转

**`ts tf tl tn tp`**

 
在  (ex command) 模式中
 
Command| Commant
-------- | ---
:ts|列出所有当前标签位置
:tf/tfirst|跳转到和当前标签相同的第一个标签位置
:tl/tlast|跳转到和当前标签相同的最后一个标签位置
:[count]tp/[count]tprevious|跳转到和当前标签相同的前N个标签位置
:[count]tn/tnext|跳转到和当前标签相同的后N个标签位置

对于 tp 和 tn 而言，前面加上数字后可以直接跳过指定个数的标签


**ts** 后

~~~
  # pri kind tag               file
> 1 F C f    forklift          lib/forklift/transports/mysql.rb
               class:Forklift.Connection.Mysql
               def forklift
  2 F   f    forklift          lib/forklift/base/logger.rb
               class:Forklift.Base.Logger
               def forklift
  3 F   f    forklift          lib/forklift/base/mailer.rb
               class:Forklift.Base.Mailer
               def forklift
  4 F   f    forklift          lib/forklift/base/pid.rb
               class:Forklift.Base.Pid
               def forklift
  5 F   f    forklift          lib/forklift/transports/csv.rb
               class:Forklift.Connection.Csv
               def forklift
  6 F   f    forklift          lib/forklift/transports/elasticsearch.rb
               class:Forklift.Connection.Elasticsearch
               def forklift
Type number and <Enter> (empty cancels): 
~~~

**3tn** 然后 **ts**

~~~
  # pri kind tag               file
  1 F C f    forklift          lib/forklift/transports/mysql.rb
               class:Forklift.Connection.Mysql
               def forklift
  2 F   f    forklift          lib/forklift/base/logger.rb
               class:Forklift.Base.Logger
               def forklift
  3 F   f    forklift          lib/forklift/base/mailer.rb
               class:Forklift.Base.Mailer
               def forklift
> 4 F   f    forklift          lib/forklift/base/pid.rb
               class:Forklift.Base.Pid
               def forklift
  5 F   f    forklift          lib/forklift/transports/csv.rb
               class:Forklift.Connection.Csv
               def forklift
  6 F   f    forklift          lib/forklift/transports/elasticsearch.rb
               class:Forklift.Connection.Elasticsearch
               def forklift
Type number and <Enter> (empty cancels):
~~~

**tl**  然后 **ts**

~~~
  # pri kind tag               file
  1 F C f    forklift          lib/forklift/transports/mysql.rb
               class:Forklift.Connection.Mysql
               def forklift
  2 F   f    forklift          lib/forklift/base/logger.rb
               class:Forklift.Base.Logger
               def forklift
  3 F   f    forklift          lib/forklift/base/mailer.rb
               class:Forklift.Base.Mailer
               def forklift
  4 F   f    forklift          lib/forklift/base/pid.rb
               class:Forklift.Base.Pid
               def forklift
  5 F   f    forklift          lib/forklift/transports/csv.rb
               class:Forklift.Connection.Csv
               def forklift
> 6 F   f    forklift          lib/forklift/transports/elasticsearch.rb
               class:Forklift.Connection.Elasticsearch
               def forklift
Type number and <Enter> (empty cancels): 
~~~

**3tp** 然后 **ts**

~~~
  # pri kind tag               file
  1 F C f    forklift          lib/forklift/transports/mysql.rb
               class:Forklift.Connection.Mysql
               def forklift
  2 F   f    forklift          lib/forklift/base/logger.rb
               class:Forklift.Base.Logger
               def forklift
> 3 F   f    forklift          lib/forklift/base/mailer.rb
               class:Forklift.Base.Mailer
               def forklift
  4 F   f    forklift          lib/forklift/base/pid.rb
               class:Forklift.Base.Pid
               def forklift
  5 F   f    forklift          lib/forklift/transports/csv.rb
               class:Forklift.Connection.Csv
               def forklift
  6 F   f    forklift          lib/forklift/transports/elasticsearch.rb
               class:Forklift.Connection.Elasticsearch
               def forklift
Type number and <Enter> (empty cancels):
~~~

**tf**  然后 **ts**


~~~
  # pri kind tag               file
> 1 F C f    forklift          lib/forklift/transports/mysql.rb
               class:Forklift.Connection.Mysql
               def forklift
  2 F   f    forklift          lib/forklift/base/logger.rb
               class:Forklift.Base.Logger
               def forklift
  3 F   f    forklift          lib/forklift/base/mailer.rb
               class:Forklift.Base.Mailer
               def forklift
  4 F   f    forklift          lib/forklift/base/pid.rb
               class:Forklift.Base.Pid
               def forklift
  5 F   f    forklift          lib/forklift/transports/csv.rb
               class:Forklift.Connection.Csv
               def forklift
  6 F   f    forklift          lib/forklift/transports/elasticsearch.rb
               class:Forklift.Connection.Elasticsearch
               def forklift
Type number and <Enter> (empty cancels): 
~~~


---

## tag 补全

在命令模式下 **`:tag <tag>`**  后面的 **`<tag>`** 可以自动补全

使用 **tab** 键自动补全


---

## tags 历史记录

在命令模式下 **`:tags`** 可以看到过往的标签记录


~~~
...
...  
      def drop!(table, database=current_database)
:tags
  # TO tag         FROM line  in file/text
  1  6 forklift           12  bin/forklift
  2  1 forklift           25  def forklift
> 3  1 connect            16  lib/forklift/transports/elasticsearch.rb
Press ENTER or type command to continue
~~~



---

# 命令汇总


* **`yum install ctags`**
* **`ctags --version`**
* **`ctags --list-languages`**
* **`ctags --list-maps`**
* **`ctags --list-kinds`**
* **`ctags -R *`**
* **`file tags`**
* **`head -n 20 tags`**
* **`vim -t forklift`**
* **`Ctrl+]`**
* **`Ctrl+T`**
* **`:ta`**
* **`:ts`**
* **`:tf`**
* **`:tl`**
* **`:[n]tn`**
* **`:[p]tp`**



---

[ctags]:http://ctags.sourceforge.net/
[languages]:http://ctags.sourceforge.net/languages.html
[tools]:http://ctags.sourceforge.net/tools.html

