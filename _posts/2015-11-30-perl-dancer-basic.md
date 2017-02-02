---
layout: post
title: perl dancer 基础
author: wilmosfang
tags:  perl dancer
categories:  perl
wc: 1577 6327 64750
excerpt: follow me
comments: true
---



# 前言

**[Dancer][perldancer]** 是一个perl的web框架，可以快速生成web server.

>Dancer is a simple but powerful web application framework for Perl.

下面对 **[Dancer][perldancer]** 的基础操作进行一下分享

> **Tip:** 当前的版本为 **Dancer2-0.163** ,下载地址:  **[Dancer2-0.163000][Dancer2-0.163000]** , CPAN: **[Dancer2_CPAN][Dancer2_cpan]** , git: **[Dancer2_git][Dancer2_git]** 

---



# 概要

* TOC
{:toc}



---

## 安装

使用 **`curl -L http://cpanmin.us | perl - --sudo Dancer2`** 进行安装

~~~
[root@dancer-test ~]# curl -L http://cpanmin.us | perl - --sudo Dancer2 
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  296k  100  296k    0     0   208k      0  0:00:01  0:00:01 --:--:--  208k
--> Working on Dancer2
Fetching http://www.cpan.org/authors/id/X/XS/XSAWYERX/Dancer2-0.163000.tar.gz ... OK
==> Found dependencies: ExtUtils::MakeMaker, File::ShareDir::Install
--> Working on ExtUtils::MakeMaker
Fetching http://www.cpan.org/authors/id/B/BI/BINGOS/ExtUtils-MakeMaker-7.10.tar.gz ... OK
Configuring ExtUtils-MakeMaker-7.10 ... OK
Building ExtUtils-MakeMaker-7.10 ... OK
Successfully installed ExtUtils-MakeMaker-7.10 (upgraded from 6.55_02)
--> Working on File::ShareDir::Install
Fetching http://www.cpan.org/authors/id/G/GW/GWYN/File-ShareDir-Install-0.10.tar.gz ... OK
Configuring File-ShareDir-Install-0.10 ... OK
Building File-ShareDir-Install-0.10 ... OK
Successfully installed File-ShareDir-Install-0.10
Configuring Dancer2-0.163000 ... OK
==> Found dependencies: HTTP::Headers::Fast, Capture::Tiny, YAML, Template::Tiny, App::Cmd::Setup, HTTP::Tiny, Return::MultiLevel, Config::Any, Plack::Middleware::RemoveRedundantBody, HTTP::Body, Plack::Middleware::FixMissingBodyInRedirect, Import::Into, Hash::Merge::Simple, MooX::Types::MooseLike, Role::Tiny, Test::Fatal, Moo::Role, Safe::Isa, Plack, Moo, Class::Load, Template, MIME::Base64, JSON
--> Working on HTTP::Headers::Fast
Fetching http://www.cpan.org/authors/id/T/TO/TOKUHIROM/HTTP-Headers-Fast-0.20.tar.gz ... OK
==> Found dependencies: Module::Build
--> Working on Module::Build
Fetching http://www.cpan.org/authors/id/L/LE/LEONT/Module-Build-0.4214.tar.gz ... OK
==> Found dependencies: Module::Metadata, version, CPAN::Meta, Perl::OSType
--> Working on Module::Metadata
Fetching http://www.cpan.org/authors/id/E/ET/ETHER/Module-Metadata-1.000027.tar.gz ... OK
Configuring Module-Metadata-1.000027 ... OK
==> Found dependencies: version
--> Working on version
Fetching http://www.cpan.org/authors/id/J/JP/JPEACOCK/version-0.9912.tar.gz ... OK
Configuring version-0.9912 ... OK
Building version-0.9912 ... OK
Successfully installed version-0.9912 (upgraded from 0.77)
Building Module-Metadata-1.000027 ... OK
Successfully installed Module-Metadata-1.000027
--> Working on CPAN::Meta
Fetching http://www.cpan.org/authors/id/D/DA/DAGOLDEN/CPAN-Meta-2.150005.tar.gz ... OK
Configuring CPAN-Meta-2.150005 ... OK
==> Found dependencies: Parse::CPAN::Meta
--> Working on Parse::CPAN::Meta
Fetching http://www.cpan.org/authors/id/D/DA/DAGOLDEN/Parse-CPAN-Meta-1.4417.tar.gz ... OK
Configuring Parse-CPAN-Meta-1.4417 ... OK
==> Found dependencies: CPAN::Meta::YAML
--> Working on CPAN::Meta::YAML
Fetching http://www.cpan.org/authors/id/D/DA/DAGOLDEN/CPAN-Meta-YAML-0.016.tar.gz ... OK
Configuring CPAN-Meta-YAML-0.016 ... OK
Building CPAN-Meta-YAML-0.016 ... OK
Successfully installed CPAN-Meta-YAML-0.016 (upgraded from 0.008)
Building Parse-CPAN-Meta-1.4417 ... OK
Successfully installed Parse-CPAN-Meta-1.4417 (upgraded from 1.4405)
Building CPAN-Meta-2.150005 ... OK
Successfully installed CPAN-Meta-2.150005 (upgraded from 2.120351)
--> Working on Perl::OSType
Fetching http://www.cpan.org/authors/id/D/DA/DAGOLDEN/Perl-OSType-1.009.tar.gz ... OK
Configuring Perl-OSType-1.009 ... OK
Building Perl-OSType-1.009 ... OK
Successfully installed Perl-OSType-1.009
Configuring Module-Build-0.4214 ... OK
Building Module-Build-0.4214 ... OK
Successfully installed Module-Build-0.4214 (upgraded from 0.35)
Configuring HTTP-Headers-Fast-0.20 ... OK
==> Found dependencies: Test::More, Test::Requires
--> Working on Test::More
Fetching http://www.cpan.org/authors/id/E/EX/EXODIST/Test-Simple-1.001014.tar.gz ... OK
Configuring Test-Simple-1.001014 ... OK
Building and testing Test-Simple-1.001014 ... OK
Successfully installed Test-Simple-1.001014 (upgraded from 0.92)
--> Working on Test::Requires
Fetching http://www.cpan.org/authors/id/T/TO/TOKUHIROM/Test-Requires-0.10.tar.gz ... OK
Configuring Test-Requires-0.10 ... OK
Building and testing Test-Requires-0.10 ... OK
Successfully installed Test-Requires-0.10
Building and testing HTTP-Headers-Fast-0.20 ... OK
Successfully installed HTTP-Headers-Fast-0.20
--> Working on Capture::Tiny
Fetching http://www.cpan.org/authors/id/D/DA/DAGOLDEN/Capture-Tiny-0.30.tar.gz ... OK
Configuring Capture-Tiny-0.30 ... OK
Building and testing Capture-Tiny-0.30 ... OK
Successfully installed Capture-Tiny-0.30
--> Working on YAML
Fetching http://www.cpan.org/authors/id/I/IN/INGY/YAML-1.15.tar.gz ... OK
Configuring YAML-1.15 ... OK
==> Found dependencies: Test::YAML
--> Working on Test::YAML
Fetching http://www.cpan.org/authors/id/I/IN/INGY/Test-YAML-1.06.tar.gz ... OK
Configuring Test-YAML-1.06 ... OK
==> Found dependencies: Test::Base
--> Working on Test::Base
Fetching http://www.cpan.org/authors/id/I/IN/INGY/Test-Base-0.88.tar.gz ... OK
Configuring Test-Base-0.88 ... OK
==> Found dependencies: Text::Diff, Spiffy, Algorithm::Diff
--> Working on Text::Diff
Fetching http://www.cpan.org/authors/id/N/NE/NEILB/Text-Diff-1.43.tar.gz ... OK
Configuring Text-Diff-1.43 ... OK
==> Found dependencies: Algorithm::Diff
--> Working on Algorithm::Diff
Fetching http://www.cpan.org/authors/id/T/TY/TYEMQ/Algorithm-Diff-1.1903.tar.gz ... OK
Configuring Algorithm-Diff-1.1903 ... OK
Building and testing Algorithm-Diff-1.1903 ... OK
Successfully installed Algorithm-Diff-1.1903
Building and testing Text-Diff-1.43 ... OK
Successfully installed Text-Diff-1.43
--> Working on Spiffy
Fetching http://www.cpan.org/authors/id/I/IN/INGY/Spiffy-0.46.tar.gz ... OK
Configuring Spiffy-0.46 ... OK
Building and testing Spiffy-0.46 ... OK
Successfully installed Spiffy-0.46
Building and testing Test-Base-0.88 ... OK
Successfully installed Test-Base-0.88
Building and testing Test-YAML-1.06 ... OK
Successfully installed Test-YAML-1.06
Building and testing YAML-1.15 ... OK
Successfully installed YAML-1.15
--> Working on Template::Tiny
Fetching http://www.cpan.org/authors/id/A/AD/ADAMK/Template-Tiny-1.12.tar.gz ... OK
Configuring Template-Tiny-1.12 ... OK
Building and testing Template-Tiny-1.12 ... OK
Successfully installed Template-Tiny-1.12
--> Working on App::Cmd::Setup
Fetching http://www.cpan.org/authors/id/R/RJ/RJBS/App-Cmd-0.330.tar.gz ... OK
Configuring App-Cmd-0.330 ... OK
==> Found dependencies: Pod::Usage, Test::Fatal, IO::TieCombine, String::RewritePrefix, Sub::Exporter, Getopt::Long::Descriptive, Getopt::Long, Class::Load, Data::OptList, Sub::Exporter::Util, Sub::Install
--> Working on Pod::Usage
Fetching http://www.cpan.org/authors/id/M/MA/MAREKR/Pod-Usage-1.67.tar.gz ... OK
Configuring Pod-Usage-1.67 ... OK
==> Found dependencies: Pod::Text
--> Working on Pod::Text
Fetching http://www.cpan.org/authors/id/R/RR/RRA/podlators-2.5.3.tar.gz ... OK
Configuring podlators-v2.5.3 ... OK
Building and testing podlators-v2.5.3 ... OK
Successfully installed podlators-v2.5.3 (upgraded from 3.13)
Building and testing Pod-Usage-1.67 ... OK
Successfully installed Pod-Usage-1.67 (upgraded from 1.36)
--> Working on Test::Fatal
Fetching http://www.cpan.org/authors/id/R/RJ/RJBS/Test-Fatal-0.014.tar.gz ... OK
Configuring Test-Fatal-0.014 ... OK
==> Found dependencies: Try::Tiny
--> Working on Try::Tiny
Fetching http://www.cpan.org/authors/id/D/DO/DOY/Try-Tiny-0.22.tar.gz ... OK
Configuring Try-Tiny-0.22 ... OK
Building and testing Try-Tiny-0.22 ... OK
Successfully installed Try-Tiny-0.22
Building and testing Test-Fatal-0.014 ... OK
Successfully installed Test-Fatal-0.014
--> Working on IO::TieCombine
Fetching http://www.cpan.org/authors/id/R/RJ/RJBS/IO-TieCombine-1.005.tar.gz ... OK
Configuring IO-TieCombine-1.005 ... OK
Building and testing IO-TieCombine-1.005 ... OK
Successfully installed IO-TieCombine-1.005
--> Working on String::RewritePrefix
Fetching http://www.cpan.org/authors/id/R/RJ/RJBS/String-RewritePrefix-0.007.tar.gz ... OK
Configuring String-RewritePrefix-0.007 ... OK
==> Found dependencies: Sub::Exporter
--> Working on Sub::Exporter
Fetching http://www.cpan.org/authors/id/R/RJ/RJBS/Sub-Exporter-0.987.tar.gz ... OK
Configuring Sub-Exporter-0.987 ... OK
==> Found dependencies: Data::OptList, Params::Util, Sub::Install
--> Working on Data::OptList
Fetching http://www.cpan.org/authors/id/R/RJ/RJBS/Data-OptList-0.109.tar.gz ... OK
Configuring Data-OptList-0.109 ... OK
==> Found dependencies: Params::Util, Sub::Install
--> Working on Params::Util
Fetching http://www.cpan.org/authors/id/A/AD/ADAMK/Params-Util-1.07.tar.gz ... OK
Configuring Params-Util-1.07 ... OK
Building and testing Params-Util-1.07 ... OK
Successfully installed Params-Util-1.07
--> Working on Sub::Install
Fetching http://www.cpan.org/authors/id/R/RJ/RJBS/Sub-Install-0.928.tar.gz ... OK
Configuring Sub-Install-0.928 ... OK
Building and testing Sub-Install-0.928 ... OK
Successfully installed Sub-Install-0.928
Building and testing Data-OptList-0.109 ... OK
Successfully installed Data-OptList-0.109
Building and testing Sub-Exporter-0.987 ... OK
Successfully installed Sub-Exporter-0.987
Building and testing String-RewritePrefix-0.007 ... OK
Successfully installed String-RewritePrefix-0.007
--> Working on Getopt::Long::Descriptive
Fetching http://www.cpan.org/authors/id/R/RJ/RJBS/Getopt-Long-Descriptive-0.099.tar.gz ... OK
Configuring Getopt-Long-Descriptive-0.099 ... OK
==> Found dependencies: Test::Warnings, Params::Validate
--> Working on Test::Warnings
Fetching http://www.cpan.org/authors/id/E/ET/ETHER/Test-Warnings-0.021.tar.gz ... OK
Configuring Test-Warnings-0.021 ... OK
==> Found dependencies: CPAN::Meta::Check
--> Working on CPAN::Meta::Check
Fetching http://www.cpan.org/authors/id/L/LE/LEONT/CPAN-Meta-Check-0.012.tar.gz ... OK
Configuring CPAN-Meta-Check-0.012 ... OK
==> Found dependencies: Test::Deep
--> Working on Test::Deep
Fetching http://www.cpan.org/authors/id/R/RJ/RJBS/Test-Deep-0.119.tar.gz ... OK
Configuring Test-Deep-0.119 ... OK
Building and testing Test-Deep-0.119 ... OK
Successfully installed Test-Deep-0.119
Building and testing CPAN-Meta-Check-0.012 ... OK
Successfully installed CPAN-Meta-Check-0.012
Building and testing Test-Warnings-0.021 ... OK
Successfully installed Test-Warnings-0.021
--> Working on Params::Validate
Fetching http://www.cpan.org/authors/id/D/DR/DROLSKY/Params-Validate-1.21.tar.gz ... OK
Configuring Params-Validate-1.21 ... OK
==> Found dependencies: Module::Implementation
--> Working on Module::Implementation
Fetching http://www.cpan.org/authors/id/D/DR/DROLSKY/Module-Implementation-0.09.tar.gz ... OK
Configuring Module-Implementation-0.09 ... OK
==> Found dependencies: Module::Runtime
--> Working on Module::Runtime
Fetching http://www.cpan.org/authors/id/Z/ZE/ZEFRAM/Module-Runtime-0.014.tar.gz ... OK
Configuring Module-Runtime-0.014 ... OK
Building and testing Module-Runtime-0.014 ... OK
Successfully installed Module-Runtime-0.014
Building and testing Module-Implementation-0.09 ... OK
Successfully installed Module-Implementation-0.09
Building and testing Params-Validate-1.21 ... OK
Successfully installed Params-Validate-1.21
Building and testing Getopt-Long-Descriptive-0.099 ... OK
Successfully installed Getopt-Long-Descriptive-0.099
--> Working on Getopt::Long
Fetching http://www.cpan.org/authors/id/J/JV/JV/Getopt-Long-2.48.tar.gz ... OK
Configuring Getopt-Long-2.48 ... OK
Building and testing Getopt-Long-2.48 ... OK
Successfully installed Getopt-Long-2.48 (upgraded from 2.38)
--> Working on Class::Load
Fetching http://www.cpan.org/authors/id/E/ET/ETHER/Class-Load-0.23.tar.gz ... OK
Configuring Class-Load-0.23 ... OK
==> Found dependencies: Package::Stash
--> Working on Package::Stash
Fetching http://www.cpan.org/authors/id/D/DO/DOY/Package-Stash-0.37.tar.gz ... OK
==> Found dependencies: Dist::CheckConflicts
--> Working on Dist::CheckConflicts
Fetching http://www.cpan.org/authors/id/D/DO/DOY/Dist-CheckConflicts-0.11.tar.gz ... OK
Configuring Dist-CheckConflicts-0.11 ... OK
Building Dist-CheckConflicts-0.11 ... OK
Successfully installed Dist-CheckConflicts-0.11
Configuring Package-Stash-0.37 ... OK
==> Found dependencies: Package::Stash::XS
--> Working on Package::Stash::XS
Fetching http://www.cpan.org/authors/id/D/DO/DOY/Package-Stash-XS-0.28.tar.gz ... OK
Configuring Package-Stash-XS-0.28 ... OK
Building and testing Package-Stash-XS-0.28 ... OK
Successfully installed Package-Stash-XS-0.28
Building and testing Package-Stash-0.37 ... OK
Successfully installed Package-Stash-0.37
Building and testing Class-Load-0.23 ... OK
Successfully installed Class-Load-0.23
Building and testing App-Cmd-0.330 ... OK
Successfully installed App-Cmd-0.330
--> Working on HTTP::Tiny
Fetching http://www.cpan.org/authors/id/D/DA/DAGOLDEN/HTTP-Tiny-0.056.tar.gz ... OK
Configuring HTTP-Tiny-0.056 ... OK
Building and testing HTTP-Tiny-0.056 ... OK
Successfully installed HTTP-Tiny-0.056
--> Working on Return::MultiLevel
Fetching http://www.cpan.org/authors/id/M/MA/MAUKE/Return-MultiLevel-0.04.tar.gz ... OK
Configuring Return-MultiLevel-0.04 ... OK
==> Found dependencies: Data::Munge
--> Working on Data::Munge
Fetching http://www.cpan.org/authors/id/M/MA/MAUKE/Data-Munge-0.095.tar.gz ... OK
Configuring Data-Munge-0.095 ... OK
Building and testing Data-Munge-0.095 ... OK
Successfully installed Data-Munge-0.095
Building and testing Return-MultiLevel-0.04 ... OK
Successfully installed Return-MultiLevel-0.04
--> Working on Config::Any
Fetching http://www.cpan.org/authors/id/B/BR/BRICAS/Config-Any-0.26.tar.gz ... OK
Configuring Config-Any-0.26 ... OK
Building and testing Config-Any-0.26 ... OK
Successfully installed Config-Any-0.26
--> Working on Plack::Middleware::RemoveRedundantBody
Fetching http://www.cpan.org/authors/id/S/SW/SWEETKID/Plack-Middleware-RemoveRedundantBody-0.05.tar.gz ... OK
Configuring Plack-Middleware-RemoveRedundantBody-0.05 ... OK
==> Found dependencies: Plack::Middleware, Plack::Util, Plack::Test, Plack::Builder
--> Working on Plack::Middleware
Fetching http://www.cpan.org/authors/id/M/MI/MIYAGAWA/Plack-1.0038.tar.gz ... OK
Configuring Plack-1.0038 ... OK
==> Found dependencies: Stream::Buffered, Test::TCP, File::ShareDir, Cookie::Baker, Hash::MultiValue, URI, Devel::StackTrace, Apache::LogFormat::Compiler, HTTP::Body, Filesys::Notify::Simple, Devel::StackTrace::AsHTML
--> Working on Stream::Buffered
Fetching http://www.cpan.org/authors/id/D/DO/DOY/Stream-Buffered-0.03.tar.gz ... OK
Configuring Stream-Buffered-0.03 ... OK
Building and testing Stream-Buffered-0.03 ... OK
Successfully installed Stream-Buffered-0.03
--> Working on Test::TCP
Fetching http://www.cpan.org/authors/id/T/TO/TOKUHIROM/Test-TCP-2.14.tar.gz ... OK
Configuring Test-TCP-2.14 ... OK
==> Found dependencies: IO::Socket::IP, Test::SharedFork
--> Working on IO::Socket::IP
Fetching http://www.cpan.org/authors/id/P/PE/PEVANS/IO-Socket-IP-0.37.tar.gz ... OK
Configuring IO-Socket-IP-0.37 ... OK
==> Found dependencies: Socket
--> Working on Socket
Fetching http://www.cpan.org/authors/id/P/PE/PEVANS/Socket-2.021.tar.gz ... OK
==> Found dependencies: ExtUtils::Constant
--> Working on ExtUtils::Constant
Fetching http://www.cpan.org/authors/id/N/NW/NWCLARK/ExtUtils-Constant-0.23.tar.gz ... OK
Configuring ExtUtils-Constant-0.16 ... OK
Building ExtUtils-Constant-0.23 ... OK
Successfully installed ExtUtils-Constant-0.23 (upgraded from 0.22)
Configuring Socket-2.021 ... OK
Building and testing Socket-2.021 ... OK
Successfully installed Socket-2.021 (upgraded from 1.82)
Building and testing IO-Socket-IP-0.37 ... FAIL
! Installing IO::Socket::IP failed. See /root/.cpanm/work/1448594784.19306/build.log for details. Retry with --force to force install it.
--> Working on Test::SharedFork
Fetching http://www.cpan.org/authors/id/E/EX/EXODIST/Test-SharedFork-0.34.tar.gz ... OK
Configuring Test-SharedFork-0.34 ... OK
Building and testing Test-SharedFork-0.34 ... OK
Successfully installed Test-SharedFork-0.34
! Installing the dependencies failed: Module 'IO::Socket::IP' is not installed
! Bailing out the installation for Test-TCP-2.14.
--> Working on File::ShareDir
Fetching http://www.cpan.org/authors/id/R/RE/REHSACK/File-ShareDir-1.102.tar.gz ... OK
Configuring File-ShareDir-1.102 ... OK
==> Found dependencies: Class::Inspector
--> Working on Class::Inspector
Fetching http://www.cpan.org/authors/id/A/AD/ADAMK/Class-Inspector-1.28.tar.gz ... OK
Configuring Class-Inspector-1.28 ... OK
Building and testing Class-Inspector-1.28 ... OK
Successfully installed Class-Inspector-1.28
Building and testing File-ShareDir-1.102 ... OK
Successfully installed File-ShareDir-1.102
--> Working on Cookie::Baker
Fetching http://www.cpan.org/authors/id/K/KA/KAZEBURO/Cookie-Baker-0.06.tar.gz ... OK
Configuring Cookie-Baker-0.06 ... OK
==> Found dependencies: Test::Time
--> Working on Test::Time
Fetching http://www.cpan.org/authors/id/S/SA/SATOH/Test-Time-0.04.tar.gz ... OK
Configuring Test-Time-0.04 ... OK
==> Found dependencies: Test::Name::FromLine
--> Working on Test::Name::FromLine
Fetching http://www.cpan.org/authors/id/S/SA/SATOH/Test-Name-FromLine-0.13.tar.gz ... OK
Configuring Test-Name-FromLine-0.13 ... OK
==> Found dependencies: Test::Differences, File::Slurp
--> Working on Test::Differences
Fetching http://www.cpan.org/authors/id/D/DC/DCANTRELL/Test-Differences-0.64.tar.gz ... OK
Configuring Test-Differences-0.64 ... OK
==> Found dependencies: Data::Dumper
--> Working on Data::Dumper
Fetching http://www.cpan.org/authors/id/S/SM/SMUELLER/Data-Dumper-2.154.tar.gz ... OK
Configuring Data-Dumper-2.154 ... OK
Building and testing Data-Dumper-2.154 ... OK
Successfully installed Data-Dumper-2.154 (upgraded from 2.124)
Building and testing Test-Differences-0.64 ... OK
Successfully installed Test-Differences-0.64
--> Working on File::Slurp
Fetching http://www.cpan.org/authors/id/U/UR/URI/File-Slurp-9999.19.tar.gz ... OK
Configuring File-Slurp-9999.19 ... OK
Building and testing File-Slurp-9999.19 ... OK
Successfully installed File-Slurp-9999.19
Building and testing Test-Name-FromLine-0.13 ... OK
Successfully installed Test-Name-FromLine-0.13
Building and testing Test-Time-0.04 ... OK
Successfully installed Test-Time-0.04
Building and testing Cookie-Baker-0.06 ... OK
Successfully installed Cookie-Baker-0.06
--> Working on Hash::MultiValue
Fetching http://www.cpan.org/authors/id/A/AR/ARISTOTLE/Hash-MultiValue-0.16.tar.gz ... OK
Configuring Hash-MultiValue-0.16 ... OK
Building and testing Hash-MultiValue-0.16 ... OK
Successfully installed Hash-MultiValue-0.16
--> Working on URI
Fetching http://www.cpan.org/authors/id/E/ET/ETHER/URI-1.69.tar.gz ... OK
Configuring URI-1.69 ... OK
Building and testing URI-1.69 ... OK
Successfully installed URI-1.69 (upgraded from 1.40)
--> Working on Devel::StackTrace
Fetching http://www.cpan.org/authors/id/D/DR/DROLSKY/Devel-StackTrace-2.00.tar.gz ... OK
Configuring Devel-StackTrace-2.00 ... OK
Building and testing Devel-StackTrace-2.00 ... OK
Successfully installed Devel-StackTrace-2.00
--> Working on Apache::LogFormat::Compiler
Fetching http://www.cpan.org/authors/id/K/KA/KAZEBURO/Apache-LogFormat-Compiler-0.32.tar.gz ... OK
Configuring Apache-LogFormat-Compiler-0.32 ... OK
==> Found dependencies: Test::MockTime, POSIX::strftime::Compiler
--> Working on Test::MockTime
Fetching http://www.cpan.org/authors/id/D/DD/DDICK/Test-MockTime-0.15.tar.gz ... OK
Configuring Test-MockTime-0.15 ... OK
Building and testing Test-MockTime-0.15 ... OK
Successfully installed Test-MockTime-0.15
--> Working on POSIX::strftime::Compiler
Fetching http://www.cpan.org/authors/id/K/KA/KAZEBURO/POSIX-strftime-Compiler-0.41.tar.gz ... OK
Configuring POSIX-strftime-Compiler-0.41 ... OK
Building and testing POSIX-strftime-Compiler-0.41 ... OK
Successfully installed POSIX-strftime-Compiler-0.41
Building and testing Apache-LogFormat-Compiler-0.32 ... OK
Successfully installed Apache-LogFormat-Compiler-0.32
--> Working on HTTP::Body
Fetching http://www.cpan.org/authors/id/G/GE/GETTY/HTTP-Body-1.22.tar.gz ... OK
Configuring HTTP-Body-1.22 ... OK
Building and testing HTTP-Body-1.22 ... OK
Successfully installed HTTP-Body-1.22
--> Working on Filesys::Notify::Simple
Fetching http://www.cpan.org/authors/id/M/MI/MIYAGAWA/Filesys-Notify-Simple-0.12.tar.gz ... OK
Configuring Filesys-Notify-Simple-0.12 ... OK
Building and testing Filesys-Notify-Simple-0.12 ... OK
Successfully installed Filesys-Notify-Simple-0.12
--> Working on Devel::StackTrace::AsHTML
Fetching http://www.cpan.org/authors/id/M/MI/MIYAGAWA/Devel-StackTrace-AsHTML-0.14.tar.gz ... OK
Configuring Devel-StackTrace-AsHTML-0.14 ... OK
Building and testing Devel-StackTrace-AsHTML-0.14 ... OK
Successfully installed Devel-StackTrace-AsHTML-0.14
! Installing the dependencies failed: Module 'Test::TCP' is not installed
! Bailing out the installation for Plack-1.0038.
! Installing the dependencies failed: Module 'Plack::Middleware' is not installed, Module 'Plack::Util' is not installed, Module 'Plack::Test' is not installed, Module 'Plack::Builder' is not installed
! Bailing out the installation for Plack-Middleware-RemoveRedundantBody-0.05.
--> Working on Plack::Middleware::FixMissingBodyInRedirect
Fetching http://www.cpan.org/authors/id/S/SW/SWEETKID/Plack-Middleware-FixMissingBodyInRedirect-0.12.tar.gz ... OK
Configuring Plack-Middleware-FixMissingBodyInRedirect-0.12 ... OK
==> Found dependencies: Plack::Util, Plack::Middleware, Plack::Test, Plack::Builder
! Installing the dependencies failed: Module 'Plack::Middleware' is not installed, Module 'Plack::Util' is not installed, Module 'Plack::Test' is not installed, Module 'Plack::Builder' is not installed
! Bailing out the installation for Plack-Middleware-FixMissingBodyInRedirect-0.12.
--> Working on Import::Into
Fetching http://www.cpan.org/authors/id/H/HA/HAARG/Import-Into-1.002005.tar.gz ... OK
Configuring Import-Into-1.002005 ... OK
Building and testing Import-Into-1.002005 ... OK
Successfully installed Import-Into-1.002005
--> Working on Hash::Merge::Simple
Fetching http://www.cpan.org/authors/id/R/RO/ROKR/Hash-Merge-Simple-0.051.tar.gz ... OK
Configuring Hash-Merge-Simple-0.051 ... OK
==> Found dependencies: Test::Most, Clone
--> Working on Test::Most
Fetching http://www.cpan.org/authors/id/O/OV/OVID/Test-Most-0.34.tar.gz ... OK
Configuring Test-Most-0.34 ... OK
==> Found dependencies: Exception::Class, Test::Harness, Test::Exception, Test::Warn
--> Working on Exception::Class
Fetching http://www.cpan.org/authors/id/D/DR/DROLSKY/Exception-Class-1.39.tar.gz ... OK
Configuring Exception-Class-1.39 ... OK
==> Found dependencies: Class::Data::Inheritable
--> Working on Class::Data::Inheritable
Fetching http://www.cpan.org/authors/id/T/TM/TMTM/Class-Data-Inheritable-0.08.tar.gz ... OK
Configuring Class-Data-Inheritable-0.08 ... OK
Building and testing Class-Data-Inheritable-0.08 ... OK
Successfully installed Class-Data-Inheritable-0.08
Building and testing Exception-Class-1.39 ... OK
Successfully installed Exception-Class-1.39
--> Working on Test::Harness
Fetching http://www.cpan.org/authors/id/L/LE/LEONT/Test-Harness-3.35.tar.gz ... OK
Configuring Test-Harness-3.35 ... OK
Building and testing Test-Harness-3.35 ... OK
Successfully installed Test-Harness-3.35 (upgraded from 3.17)
--> Working on Test::Exception
Fetching http://www.cpan.org/authors/id/E/EX/EXODIST/Test-Exception-0.40.tar.gz ... OK
Configuring Test-Exception-0.40 ... OK
==> Found dependencies: Sub::Uplevel
--> Working on Sub::Uplevel
Fetching http://www.cpan.org/authors/id/D/DA/DAGOLDEN/Sub-Uplevel-0.25.tar.gz ... OK
Configuring Sub-Uplevel-0.25 ... OK
Building and testing Sub-Uplevel-0.25 ... OK
Successfully installed Sub-Uplevel-0.25
Building and testing Test-Exception-0.40 ... OK
Successfully installed Test-Exception-0.40
--> Working on Test::Warn
Fetching http://www.cpan.org/authors/id/C/CH/CHORNY/Test-Warn-0.30.tar.gz ... OK
Configuring Test-Warn-0.30 ... OK
==> Found dependencies: Carp
--> Working on Carp
Fetching http://www.cpan.org/authors/id/R/RJ/RJBS/Carp-1.38.tar.gz ... OK
Configuring Carp-1.38 ... OK
Building and testing Carp-1.38 ... OK
Successfully installed Carp-1.38 (upgraded from 1.11)
Building and testing Test-Warn-0.30 ... OK
Successfully installed Test-Warn-0.30
Building and testing Test-Most-0.34 ... OK
Successfully installed Test-Most-0.34
--> Working on Clone
Fetching http://www.cpan.org/authors/id/G/GA/GARU/Clone-0.38.tar.gz ... OK
Configuring Clone-0.38 ... OK
Building and testing Clone-0.38 ... OK
Successfully installed Clone-0.38
Building and testing Hash-Merge-Simple-0.051 ... OK
Successfully installed Hash-Merge-Simple-0.051
--> Working on MooX::Types::MooseLike
Fetching http://www.cpan.org/authors/id/M/MA/MATEU/MooX-Types-MooseLike-0.29.tar.gz ... OK
Configuring MooX-Types-MooseLike-0.29 ... OK
==> Found dependencies: Moo
--> Working on Moo
Fetching http://www.cpan.org/authors/id/H/HA/HAARG/Moo-2.000002.tar.gz ... OK
Configuring Moo-2.000002 ... OK
==> Found dependencies: Role::Tiny, Class::Method::Modifiers, Devel::GlobalDestruction
--> Working on Role::Tiny
Fetching http://www.cpan.org/authors/id/H/HA/HAARG/Role-Tiny-2.000001.tar.gz ... OK
Configuring Role-Tiny-2.000001 ... OK
Building and testing Role-Tiny-2.000001 ... OK
Successfully installed Role-Tiny-2.000001
--> Working on Class::Method::Modifiers
Fetching http://www.cpan.org/authors/id/E/ET/ETHER/Class-Method-Modifiers-2.11.tar.gz ... OK
Configuring Class-Method-Modifiers-2.11 ... OK
Building and testing Class-Method-Modifiers-2.11 ... OK
Successfully installed Class-Method-Modifiers-2.11
--> Working on Devel::GlobalDestruction
Fetching http://www.cpan.org/authors/id/H/HA/HAARG/Devel-GlobalDestruction-0.13.tar.gz ... OK
Configuring Devel-GlobalDestruction-0.13 ... OK
==> Found dependencies: Sub::Exporter::Progressive, Devel::GlobalDestruction::XS
--> Working on Sub::Exporter::Progressive
Fetching http://www.cpan.org/authors/id/F/FR/FREW/Sub-Exporter-Progressive-0.001011.tar.gz ... OK
Configuring Sub-Exporter-Progressive-0.001011 ... OK
Building and testing Sub-Exporter-Progressive-0.001011 ... OK
Successfully installed Sub-Exporter-Progressive-0.001011
--> Working on Devel::GlobalDestruction::XS
Fetching http://www.cpan.org/authors/id/H/HA/HAARG/Devel-GlobalDestruction-XS-0.02.tar.gz ... OK
Configuring Devel-GlobalDestruction-XS-0.02 ... OK
Building and testing Devel-GlobalDestruction-XS-0.02 ... OK
Successfully installed Devel-GlobalDestruction-XS-0.02
Building and testing Devel-GlobalDestruction-0.13 ... OK
Successfully installed Devel-GlobalDestruction-0.13
Building and testing Moo-2.000002 ... OK
Successfully installed Moo-2.000002
Building and testing MooX-Types-MooseLike-0.29 ... OK
Successfully installed MooX-Types-MooseLike-0.29
--> Working on Safe::Isa
Fetching http://www.cpan.org/authors/id/E/ET/ETHER/Safe-Isa-1.000005.tar.gz ... OK
Configuring Safe-Isa-1.000005 ... OK
Building and testing Safe-Isa-1.000005 ... OK
Successfully installed Safe-Isa-1.000005
--> Working on Template
Fetching http://www.cpan.org/authors/id/A/AB/ABW/Template-Toolkit-2.26.tar.gz ... OK
Configuring Template-Toolkit-2.26 ... OK
==> Found dependencies: Test::LeakTrace, AppConfig
--> Working on Test::LeakTrace
Fetching http://www.cpan.org/authors/id/G/GF/GFUJI/Test-LeakTrace-0.15.tar.gz ... OK
Configuring Test-LeakTrace-0.15 ... OK
Building and testing Test-LeakTrace-0.15 ... OK
Successfully installed Test-LeakTrace-0.15
--> Working on AppConfig
Fetching http://www.cpan.org/authors/id/N/NE/NEILB/AppConfig-1.71.tar.gz ... OK
Configuring AppConfig-1.71 ... OK
==> Found dependencies: Test::Pod
--> Working on Test::Pod
Fetching http://www.cpan.org/authors/id/E/ET/ETHER/Test-Pod-1.51.tar.gz ... OK
Configuring Test-Pod-1.51 ... OK
Building and testing Test-Pod-1.51 ... OK
Successfully installed Test-Pod-1.51
Building and testing AppConfig-1.71 ... OK
Successfully installed AppConfig-1.71
Building and testing Template-Toolkit-2.26 ... OK
Successfully installed Template-Toolkit-2.26
--> Working on MIME::Base64
Fetching http://www.cpan.org/authors/id/G/GA/GAAS/MIME-Base64-3.15.tar.gz ... OK
Configuring MIME-Base64-3.15 ... OK
Building and testing MIME-Base64-3.15 ... OK
Successfully installed MIME-Base64-3.15 (upgraded from 3.08)
--> Working on JSON
Fetching http://www.cpan.org/authors/id/M/MA/MAKAMAKA/JSON-2.90.tar.gz ... OK
Configuring JSON-2.90 ... OK
Building and testing JSON-2.90 ... OK
Successfully installed JSON-2.90
! Installing the dependencies failed: Module 'Plack::Middleware::RemoveRedundantBody' is not installed, Module 'Plack::Middleware::FixMissingBodyInRedirect' is not installed, Module 'Plack' is not installed
! Bailing out the installation for Dancer2-0.163000.
93 distributions installed
[root@dancer-test ~]# echo $?
1
[root@dancer-test ~]# curl -L http://cpanmin.us | perl - --sudo Dancer2 
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  296k  100  296k    0     0  28557      0  0:00:10  0:00:10 --:--:-- 68975
--> Working on Dancer2
Fetching http://www.cpan.org/authors/id/X/XS/XSAWYERX/Dancer2-0.163000.tar.gz ... OK
Configuring Dancer2-0.163000 ... OK
==> Found dependencies: Plack::Middleware::RemoveRedundantBody, Plack::Middleware::FixMissingBodyInRedirect, Plack
--> Working on Plack::Middleware::RemoveRedundantBody
Fetching http://www.cpan.org/authors/id/S/SW/SWEETKID/Plack-Middleware-RemoveRedundantBody-0.05.tar.gz ... OK
Configuring Plack-Middleware-RemoveRedundantBody-0.05 ... OK
==> Found dependencies: Plack::Middleware, Plack::Util, Plack::Test, Plack::Builder
--> Working on Plack::Middleware
Fetching http://www.cpan.org/authors/id/M/MI/MIYAGAWA/Plack-1.0038.tar.gz ... OK
Configuring Plack-1.0038 ... OK
==> Found dependencies: Test::TCP
--> Working on Test::TCP
Fetching http://www.cpan.org/authors/id/T/TO/TOKUHIROM/Test-TCP-2.14.tar.gz ... OK
Configuring Test-TCP-2.14 ... OK
==> Found dependencies: IO::Socket::IP
--> Working on IO::Socket::IP
Fetching http://www.cpan.org/authors/id/P/PE/PEVANS/IO-Socket-IP-0.37.tar.gz ... OK
Configuring IO-Socket-IP-0.37 ... OK
Building and testing IO-Socket-IP-0.37 ... OK
Successfully installed IO-Socket-IP-0.37
Building and testing Test-TCP-2.14 ... OK
Successfully installed Test-TCP-2.14
Building and testing Plack-1.0038 ... OK
Successfully installed Plack-1.0038
Building and testing Plack-Middleware-RemoveRedundantBody-0.05 ... OK
Successfully installed Plack-Middleware-RemoveRedundantBody-0.05
--> Working on Plack::Middleware::FixMissingBodyInRedirect
Fetching http://www.cpan.org/authors/id/S/SW/SWEETKID/Plack-Middleware-FixMissingBodyInRedirect-0.12.tar.gz ... OK
Configuring Plack-Middleware-FixMissingBodyInRedirect-0.12 ... OK
Building and testing Plack-Middleware-FixMissingBodyInRedirect-0.12 ... OK
Successfully installed Plack-Middleware-FixMissingBodyInRedirect-0.12
Building and testing Dancer2-0.163000 ... OK
Successfully installed Dancer2-0.163000
6 distributions installed
[root@dancer-test ~]# echo $?
0
[root@dancer-test ~]# 
~~~

> **Note:** 由于网络原因可能部分包在下载过程中会出错，只要重新在执行一次就可以了




---

## 创建一个应用

首先创建一个用户(最好不要使用root的身份运行web app)

~~~
[root@dancer-test ~]# tail -n 2 /etc/passwd
hunter:x:503:503::/home/hunter:/bin/bash
autotools:x:504:504::/home/autotools:/bin/bash
[root@dancer-test ~]# useradd dancer
[root@dancer-test ~]# tail -n 2 /etc/passwd
autotools:x:504:504::/home/autotools:/bin/bash
dancer:x:505:505::/home/dancer:/bin/bash
[root@dancer-test ~]# su - dancer 
[dancer@dancer-test ~]$ ls
[dancer@dancer-test ~]$ 
~~~


创建一个名字叫 **TEST-APP** 的应用

~~~
[dancer@dancer-test ~]$ dancer2  version
Dancer2 0.163000
[dancer@dancer-test ~]$ dancer2  -a TEST::APP
+ TEST-APP
+ TEST-APP/Makefile.PL
+ TEST-APP/MANIFEST.SKIP
+ TEST-APP/cpanfile
+ TEST-APP/config.yml
+ TEST-APP/views
+ TEST-APP/views/index.tt
+ TEST-APP/views/layouts
+ TEST-APP/views/layouts/main.tt
+ TEST-APP/environments
+ TEST-APP/environments/development.yml
+ TEST-APP/environments/production.yml
+ TEST-APP/t
+ TEST-APP/t/002_index_route.t
+ TEST-APP/t/001_base.t
+ TEST-APP/bin
+ TEST-APP/bin/app.psgi
+ TEST-APP/public
+ TEST-APP/public/404.html
+ TEST-APP/public/dispatch.cgi
+ TEST-APP/public/500.html
+ TEST-APP/public/dispatch.fcgi
+ TEST-APP/public/favicon.ico
+ TEST-APP/public/javascripts
+ TEST-APP/public/javascripts/jquery.js
+ TEST-APP/public/images
+ TEST-APP/public/images/perldancer-bg.jpg
+ TEST-APP/public/images/perldancer.jpg
+ TEST-APP/public/css
+ TEST-APP/public/css/style.css
+ TEST-APP/public/css/error.css
+ TEST-APP/lib/TEST
+ TEST-APP/lib/TEST/APP.pm
[dancer@dancer-test ~]$ ls
TEST-APP
[dancer@dancer-test ~]$ 
~~~


---

## 启动应用

使用 **`plackup -r bin/app.psgi`** 启动应用

~~~
[dancer@dancer-test ~]$ cd TEST-APP/
[dancer@dancer-test TEST-APP]$ ls
bin  config.yml  cpanfile  environments  lib  Makefile.PL  MANIFEST  MANIFEST.SKIP  public  t  views 
[dancer@dancer-test TEST-APP]$ plackup -r bin/app.psgi 
Watching bin/lib bin/app.psgi for file updates.
HTTP::Server::PSGI: Accepting connections at http://0:5000/
...
...
...

~~~

当前的窗口被抢占并且启动了一个监听端口 **5000**

~~~
[root@dancer-test ~]# netstat  -ant | grep 5000
tcp        0      0 0.0.0.0:5000                0.0.0.0:*                   LISTEN      
[root@dancer-test ~]# 
~~~

此时可以访问此服务器的 **5000** 端口

> **Note:** 此时的防火墙对于指定的端口要是放开的

![dancer_1.png](/images/dancer/dancer_1.png)

这是第一次请求中前端产生的访问日志

~~~
[TEST::APP:7832] core @2015-11-27 17:41:17> looking for get / in /usr/local/share/perl5/Dancer2/Core/App.pm l. 1205
[TEST::APP:7832] core @2015-11-27 17:41:17> Entering hook core.app.before_request in (eval 66) l. 1
[TEST::APP:7832] core @2015-11-27 17:41:17> Entering hook core.app.after_request in (eval 66) l. 1
192.168.2.59 - - [27/Nov/2015:17:41:17 +0800] "GET / HTTP/1.1" 200 5237 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/46.0.2490.86 Safari/537.36"
192.168.2.59 - - [27/Nov/2015:17:41:17 +0800] "GET /css/style.css HTTP/1.1" 200 2850 "http://192.168.20.105:5000/" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/46.0.2490.86 Safari/537.36"
192.168.2.59 - - [27/Nov/2015:17:41:17 +0800] "GET /images/perldancer-bg.jpg HTTP/1.1" 200 7125 "http://192.168.20.105:5000/" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/46.0.2490.86 Safari/537.36"
192.168.2.59 - - [27/Nov/2015:17:41:17 +0800] "GET /images/perldancer.jpg HTTP/1.1" 200 2240 "http://192.168.20.105:5000/" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/46.0.2490.86 Safari/537.36"
~~~


---

## 目录结构

~~~
[dancer@dancer-test TEST-APP]$ ls
bin  config.yml  cpanfile  environments  lib  Makefile.PL  MANIFEST  MANIFEST.SKIP  public  t  views
[dancer@dancer-test TEST-APP]$ tree
.
├── bin
│   └── app.psgi
├── config.yml
├── cpanfile
├── environments
│   ├── development.yml
│   └── production.yml
├── lib
│   └── TEST
│       └── APP.pm
├── Makefile.PL
├── MANIFEST
├── MANIFEST.SKIP
├── public
│   ├── 404.html
│   ├── 500.html
│   ├── css
│   │   ├── error.css
│   │   └── style.css
│   ├── dispatch.cgi
│   ├── dispatch.fcgi
│   ├── favicon.ico
│   ├── images
│   │   ├── perldancer-bg.jpg
│   │   └── perldancer.jpg
│   └── javascripts
│       └── jquery.js
├── t
│   ├── 001_base.t
│   └── 002_index_route.t
└── views
    ├── index.tt
    └── layouts
        └── main.tt

11 directories, 23 files
[dancer@dancer-test TEST-APP]$ 
~~~

---

## 安装Expect模块

### 配置cpan

首先要配置cpan

~~~
[root@dancer-test ~]# perl -MCPAN -e shell 
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
Always commit changes to config variables to disk? [no] yes

CPAN.pm can limit the size of the disk area for keeping the build
directories with all the intermediate files.

 <build_cache>
Cache size for build directory (in MB)? [100] 1000

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
Enter another URL or RETURN to quit: [] ftp://mirrors.ustc.edu.cn/CPAN/
Enter another URL or RETURN to quit: [] http://mirrors.ustc.edu.cn/CPAN/
Enter another URL or RETURN to quit: [] rsync://mirrors.ustc.edu.cn/CPAN/
Enter another URL or RETURN to quit: [] ftp://mirrors.xmu.edu.cn/CPAN/
Enter another URL or RETURN to quit: [] http://mirrors.xmu.edu.cn/CPAN/
Enter another URL or RETURN to quit: [] rsync://mirrors.xmu.edu.cn/CPAN/
Enter another URL or RETURN to quit: [] http://mirrors.hust.edu.cn/CPAN/
Enter another URL or RETURN to quit: [] http://mirrors.neusoft.edu.cn/cpan/
Enter another URL or RETURN to quit: [] ftp://ftp.cuhk.edu.hk/pub/packages/perl/CPAN/
Enter another URL or RETURN to quit: [] http://cpan.communilink.net/
Enter another URL or RETURN to quit: [] http://ftp.cuhk.edu.hk/pub/packages/perl/CPAN/
Enter another URL or RETURN to quit: [] http://mirrors.devlib.org/cpan/
Enter another URL or RETURN to quit: [] 
New urllist
  http://mirrors.163.com/cpan/
  http://mirrors.sohu.com/CPAN/
  ftp://mirrors.ustc.edu.cn/CPAN/
  http://mirrors.ustc.edu.cn/CPAN/
  rsync://mirrors.ustc.edu.cn/CPAN/
  ftp://mirrors.xmu.edu.cn/CPAN/
  http://mirrors.xmu.edu.cn/CPAN/
  rsync://mirrors.xmu.edu.cn/CPAN/
  http://mirrors.hust.edu.cn/CPAN/
  http://mirrors.neusoft.edu.cn/cpan/
  ftp://ftp.cuhk.edu.hk/pub/packages/perl/CPAN/
  http://cpan.communilink.net/
  http://ftp.cuhk.edu.hk/pub/packages/perl/CPAN/
  http://mirrors.devlib.org/cpan/


commit: wrote '/usr/share/perl5/CPAN/Config.pm'

cpan shell -- CPAN exploration and modules installation (v1.9402)
Enter 'h' for help.

cpan[1]> h

Display Information                                                (ver 1.9402)
 command  argument          description
 a,b,d,m  WORD or /REGEXP/  about authors, bundles, distributions, modules
 i        WORD or /REGEXP/  about any of the above
 ls       AUTHOR or GLOB    about files in the author's directory
    (with WORD being a module, bundle or author name or a distribution
    name of the form AUTHOR/DISTRIBUTION)

Download, Test, Make, Install...
 get      download                     clean    make clean
 make     make (implies get)           look     open subshell in dist directory
 test     make test (implies make)     readme   display these README files
 install  make install (implies test)  perldoc  display POD documentation

Upgrade
 r        WORDs or /REGEXP/ or NONE    report updates for some/matching/all modules
 upgrade  WORDs or /REGEXP/ or NONE    upgrade some/matching/all modules

Pragmas
 force  CMD    try hard to do command  fforce CMD    try harder
 notest CMD    skip testing

Other
 h,?           display this menu       ! perl-code   eval a perl command
 o conf [opt]  set and query options   q             quit the cpan shell
 reload cpan   load CPAN.pm again      reload index  load newer indices
 autobundle    Snapshot                recent        latest CPAN uploads
cpan[2]> 
~~~

> **Note:**  
>
> * 1.要将配置保存，否则下次还要再配置 
> * 2.主要关心一下 **[cpan mirror][cpan_sites]** 的配置


---

### 安装Expect


可以使用下面方法安装

cpanm

~~~
cpanm Expect
~~~

CPAN shell

~~~
perl -MCPAN -e shell
install Expect
~~~

或直接在cpan中进行安装

~~~
cpan[2]> install Expect
CPAN: Storable loaded ok (v2.20)
CPAN: LWP::UserAgent loaded ok (v5.833)
CPAN: Time::HiRes loaded ok (v1.9721)
Fetching with LWP:
  http://mirrors.163.com/cpan/authors/01mailrc.txt.gz
CPAN: YAML loaded ok (v1.15)
Going to read '/root/.cpan/sources/authors/01mailrc.txt.gz'
............................................................................DONE
Fetching with LWP:
  http://mirrors.163.com/cpan/modules/02packages.details.txt.gz
Going to read '/root/.cpan/sources/modules/02packages.details.txt.gz'
  Database was generated on Thu, 26 Nov 2015 22:41:02 GMT
..............
  New CPAN.pm version (v2.10) available.
  [Currently running version is v1.9402]
  You might want to try
    install CPAN
    reload cpan
  to both upgrade CPAN.pm and run the new version without leaving
  the current session.


..............................................................DONE
Fetching with LWP:
  http://mirrors.163.com/cpan/modules/03modlist.data.gz
Going to read '/root/.cpan/sources/modules/03modlist.data.gz'
DONE
Going to write /root/.cpan/Metadata
Running install for module 'Expect'
Running make for S/SZ/SZABGAB/Expect-1.32.tar.gz
Fetching with LWP:
  http://mirrors.163.com/cpan/authors/id/S/SZ/SZABGAB/Expect-1.32.tar.gz
CPAN: Digest::SHA loaded ok (v5.47)
Fetching with LWP:
  http://mirrors.163.com/cpan/authors/id/S/SZ/SZABGAB/CHECKSUMS
Checksum for /root/.cpan/sources/authors/id/S/SZ/SZABGAB/Expect-1.32.tar.gz ok
Scanning cache /root/.cpan/build for sizes
DONE
CPAN: Archive::Tar loaded ok (v1.58)
Expect-1.32/
Expect-1.32/.perltidyrc
Expect-1.32/.travis.yml
Expect-1.32/Changes
Expect-1.32/examples/
Expect-1.32/lib/
Expect-1.32/Makefile.PL
Expect-1.32/MANIFEST
Expect-1.32/META.json
Expect-1.32/META.yml
Expect-1.32/README.md
Expect-1.32/t/
Expect-1.32/tutorial/
Expect-1.32/tutorial/1.A.Intro
Expect-1.32/tutorial/2.A.ftp
Expect-1.32/tutorial/2.B.rlogin
Expect-1.32/tutorial/3.A.debugging
Expect-1.32/tutorial/4.A.top
Expect-1.32/tutorial/5.A.top
Expect-1.32/tutorial/5.B.top
Expect-1.32/tutorial/6.A.smtp-verify
Expect-1.32/tutorial/6.B.modem-init
Expect-1.32/tutorial/README
Expect-1.32/t/01-test.t
Expect-1.32/t/02-bc.t
Expect-1.32/t/03-log.t
Expect-1.32/t/04-multiline.t
Expect-1.32/t/10-internal.t
Expect-1.32/t/11-calc.t
Expect-1.32/lib/Expect.pm
Expect-1.32/examples/calc.pl
Expect-1.32/examples/expect_calc.pl
Expect-1.32/examples/kibitz/
Expect-1.32/examples/ssh.pl
Expect-1.32/examples/kibitz/Changelog
Expect-1.32/examples/kibitz/kibitz
Expect-1.32/examples/kibitz/kibitz.man
Expect-1.32/examples/kibitz/README
CPAN: File::Temp loaded ok (v0.22)

  CPAN.pm: Going to build S/SZ/SZABGAB/Expect-1.32.tar.gz

Checking if your kit is complete...
Looks good
Warning: prerequisite IO::Pty 1.11 not found.
Warning: prerequisite IO::Tty 1.11 not found.
Generating a Unix-style Makefile
Writing Makefile for Expect
Writing MYMETA.yml and MYMETA.json
---- Unsatisfied dependencies detected during ----
----        SZABGAB/Expect-1.32.tar.gz        ----
    IO::Tty [requires]
    IO::Pty [requires]
Shall I follow them and prepend them to the queue
of modules we are processing right now? [yes] yes
Running make test
  Delayed until after prerequisites
Running make install
  Delayed until after prerequisites
Running install for module 'IO::Tty'
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
+B0 +B110 +B115200 +B1200 +B134 +B150 -B153600 +B1800 +B19200 +B200 +B230400 +B2400 +B300 -B307200 +B38400 +B460800 +B4800 +B50 +B57609600 +BRKINT +BS0 +BS1 +BSDLY +CBAUD -CBAUDEXT +CBRK -CCTS_OFLOW -CDEL +CDSUSP +CEOF +CEOL -CEOL2 +CEOT +CERASE -CESC +CFLUSH +CIBAUD L +CLNEXT +CLOCAL -CNSWTCH -CNUL +CQUIT +CR0 +CR1 +CR2 +CR3 +CRDLY +CREAD +CRPRNT +CRTSCTS -CRTSXOFF -CRTS_IFLOW +CS5 +CS6 +CS7 +CS8 +STOPB +CSUSP -CSWTCH +CWERASE -DEFECHO -DIOC -DIOCGETP -DIOCSETP -DOSMODE +ECHO +ECHOCTL +ECHOE +ECHOK +ECHOKE +ECHONL +ECHOPRT +EXTA -FIORDCHK +FLUSHO +HUPCL +ICANON +ICRNL +IEXTEN +IGNBRK +IGNCR +IGNPAR +IMAXBEL +INLCR +INPCK +ISIG +ISTRIP +IUCLC +IXANY +IXOFF +IXONLOSE -LDDMAP -LDEMAP -LDGETT -LDGMAP -LDIOC -LDNMAP -LDOPEN -LDSETT -LDSMAP -LOBLK +NCCS +NL0 +NL1 +NLDLY +NOFLSH +OCRNL +OFDEL +OFILL+ONOCR +OPOST -PAGEOUT +PARENB -PAREXT +PARMRK +PARODD +PENDIN -RCV1EN -RTS_TOG +TAB0 +TAB1 +TAB2 +TAB3 +TABDLY -TCDSET +TCFLSH +TCGETIOFF +TCIOFLUSH +TCION +TCOFLUSH +TCOOFF +TCOON +TCSADRAIN +TCSAFLUSH +TCSANOW +TCSBRK +TCSETA +TCSETAF +TCSETAW -TCSETCTTY +TCSETS +T -TERM_D40 -TERM_D42 -TERM_H45 -TERM_NONE -TERM_TEC -TERM_TEX -TERM_V10 -TERM_V61 +TIOCCBRK -TIOCCDTR +TIOCCONS +TIOCEXCL -TIOCFLUSH -CGETP -TIOCGLTC +TIOCGPGRP +TIOCGSID +TIOCGSOFTCAR +TIOCGWINSZ -TIOCHPCL -TIOCKBOF -TIOCKBON -TIOCLBIC -TIOCLBIS -TIOCLGET -TIOCLSET +CMGET +TIOCMSET +TIOCM_CAR +TIOCM_CD +TIOCM_CTS +TIOCM_DSR +TIOCM_DTR +TIOCM_LE +TIOCM_RI +TIOCM_RNG +TIOCM_RTS +TIOCM_SR +TIOCM_ST +TCOUTQ -TIOCREMOTE +TIOCSBRK +TIOCSCTTY -TIOCSDTR -TIOCSETC +TIOCSETD -TIOCSETN -TIOCSETP -TIOCSIGNAL -TIOCSLTC +TIOCSPGRP -TIOCSSID +T+TIOCSTI -TIOCSTOP +TIOCSWINSZ -TM_ANL -TM_CECHO -TM_CINVIS -TM_LCF -TM_NONE -TM_SET -TM_SNL +TOSTOP -VCEOF -VCEOL +VDISCARD -VDSUSP +SE +VINTR +VKILL +VLNEXT +VMIN +VQUIT +VREPRINT +VSTART +VSTOP +VSUSP -VSWTCH +VT0 +VT1 +VTDLY +VTIME +VWERASE -WRAP +XCASE -XCLUDE -X

>>> Configuration looks good! <<<

Writing IO::Tty::Constant.pm...
DEFINE = -DHAVE_DEV_PTMX -DHAVE_GETPT -DHAVE_GRANTPT -DHAVE_OPENPTY -DHAVE_POSIX_OPENPT -DHAVE_PTSNAME -DHAVE_PTSNAME_R -DHAVE_PTY_H -TERMIOS_H -DHAVE_TERMIO_H -DHAVE_TTYNAME -DHAVE_UNLOCKPT
Checking if your kit is complete...
Looks good
Generating a Unix-style Makefile
Writing Makefile for IO::Tty
Writing MYMETA.yml and MYMETA.json
cp Tty/Constant.pm blib/lib/IO/Tty/Constant.pm
cp Tty.pm blib/lib/IO/Tty.pm
cp Pty.pm blib/lib/IO/Pty.pm
Running Mkbootstrap for IO::Tty ()
chmod 644 "Tty.bs"
"/usr/bin/perl" "/usr/share/perl5/ExtUtils/xsubpp"  -typemap "/usr/share/perl5/ExtUtils/typemap"  Tty.xs > Tty.xsc && mv Tty.xsc Tty.c
gcc -c   -D_REENTRANT -D_GNU_SOURCE -fno-strict-aliasing -pipe -fstack-protector -I/usr/local/include -D_LARGEFILE_SOURCE -D_FILE_OFFS -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector --param=ssp-buffer-size=4 -m64 -mtune=generic   -DVERSION=\"1.12\" -DXS_"-I/usr/lib64/perl5/CORE"  -DHAVE_DEV_PTMX -DHAVE_GETPT -DHAVE_GRANTPT -DHAVE_OPENPTY -DHAVE_POSIX_OPENPT -DHAVE_PTSNAME -DHAVE_PTSNAM_SIGACTION -DHAVE_TERMIOS_H -DHAVE_TERMIO_H -DHAVE_TTYNAME -DHAVE_UNLOCKPT Tty.c
rm -f blib/arch/auto/IO/Tty/Tty.so
gcc  -shared -O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector --param=ssp-buffer-size=4 -m64 -mtune=generic T/IO/Tty/Tty.so 	\
	   -lutil  	\
	  
chmod 755 blib/arch/auto/IO/Tty/Tty.so
"/usr/bin/perl" -MExtUtils::Command::MM -e 'cp_nonempty' -- Tty.bs blib/arch/auto/IO/Tty/Tty.bs 644
Manifying 3 pod documents
  TODDR/IO-Tty-1.12.tar.gz
  make -- OK
Running make test
Running Mkbootstrap for IO::Tty ()
chmod 644 "Tty.bs"
PERL_DL_NONLAZY=1 "/usr/bin/perl" "-MExtUtils::Command::MM" "-MTest::Harness" "-e" "undef *Test::Harness::Switches; test_harness(0, 'b t/*.t
t/test.t .. # Configuration: -DHAVE_DEV_PTMX -DHAVE_GETPT -DHAVE_GRANTPT -DHAVE_OPENPTY -DHAVE_POSIX_OPENPT -DHAVE_PTSNAME -DHAVE_PTSNVE_SIGACTION -DHAVE_TERMIOS_H -DHAVE_TERMIO_H -DHAVE_TTYNAME -DHAVE_UNLOCKPT
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
# Child PID = 12438
# Good, your raw ptys can handle at least 530 bytes at once.
t/test.t .. 5/5 sysread(): Input/output error at t/test.t line 151.
Slave got EOF at line 530, byte 0.
t/test.t .. ok   
All tests successful.
Files=1, Tests=5,  3 wallclock secs ( 0.04 usr  0.01 sys +  0.08 cusr  0.06 csys =  0.19 CPU)
Result: PASS
  TODDR/IO-Tty-1.12.tar.gz
  make test -- OK
Running make install
Prepending /root/.cpan/build/IO-Tty-1.12-BvmJAD/blib/arch /root/.cpan/build/IO-Tty-1.12-BvmJAD/blib/lib to PERL5LIB for 'install'
Manifying 3 pod documents
Files found in blib/arch: installing files in blib/lib into architecture dependent library tree
Installing /usr/local/lib64/perl5/auto/IO/Tty/Tty.so
Installing /usr/local/lib64/perl5/IO/Tty.pm
Installing /usr/local/lib64/perl5/IO/Pty.pm
Installing /usr/local/lib64/perl5/IO/Tty/Constant.pm
Installing /usr/local/share/man/man3/IO::Tty.3pm
Installing /usr/local/share/man/man3/IO::Tty::Constant.3pm
Installing /usr/local/share/man/man3/IO::Pty.3pm
Appending installation info to /usr/lib64/perl5/perllocal.pod
  TODDR/IO-Tty-1.12.tar.gz
  make install  -- OK
IO::Pty is up to date (1.12).
Running make for S/SZ/SZABGAB/Expect-1.32.tar.gz
  Has already been unwrapped into directory /root/.cpan/build/Expect-1.32-vx0PVI

  CPAN.pm: Going to build S/SZ/SZABGAB/Expect-1.32.tar.gz

cp lib/Expect.pm blib/lib/Expect.pm
Manifying 1 pod document
  SZABGAB/Expect-1.32.tar.gz
  make -- OK
Running make test
PERL_DL_NONLAZY=1 "/usr/bin/perl" "-MExtUtils::Command::MM" "-MTest::Harness" "-e" "undef *Test::Harness::Switches; test_harness(0, 'b t/*.t
t/01-test.t .......     # Basic tests...
t/01-test.t ....... 1/14     # Testing exec failure...
t/01-test.t ....... 2/14     # Testing exp_continue...
    # number of timeout calls in 5 sec: 4
t/01-test.t ....... 4/14     # timeout shouldn't destroy accum contents
t/01-test.t ....... 5/14     # Testing -notransfer...
t/01-test.t ....... 6/14     # Testing raw reversing...
    # isatty($exp): YES
    # Called: 3
    # Elapsed time: 6  delay by expect: 4.5
    # Elapsed time: 5  delay by expect: 3.9
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
t/01-test.t ....... 9/14     # Testing controlling terminal...
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
t/11-calc.t ....... 1/22 # OUTPUT
t/11-calc.t ....... ok     
All tests successful.
Files=6, Tests=97, 66 wallclock secs ( 0.05 usr  0.01 sys +  0.53 cusr  0.16 csys =  0.75 CPU)
Result: PASS
  SZABGAB/Expect-1.32.tar.gz
  make test -- OK
Running make install
Prepending /root/.cpan/build/Expect-1.32-vx0PVI/blib/arch /root/.cpan/build/Expect-1.32-vx0PVI/blib/lib to PERL5LIB for 'install'
Manifying 1 pod document
Installing /usr/local/share/perl5/Expect.pm
Installing /usr/local/share/man/man3/Expect.3pm
Appending installation info to /usr/lib64/perl5/perllocal.pod
  SZABGAB/Expect-1.32.tar.gz
  make install  -- OK

cpan[3]> quit 
No history written (no histfile specified).
Lockfile removed.
[root@dancer-test ~]# 
~~~

> **Note:** cpan会自动发现依赖，并且尝试解决依赖，请求同意时要允许

---

## MVC


MVC(Model-View-Controller) 是一种架构，或者说是设计理念，不同语言有不同的实现，遵循此架构会有很多好处，但详细探讨已经超出了主题，有机会再聊

下面是大体的数据流向图

~~~
MVC
   +------------------------+ 
   |                        V 
+------+    +----+    +----------+     +-----+     +--+
|client|<---|view|<---|controller|<--->|model|<--->|DB|
+------+    +----+    +----------+     +-----+     +--+
~~~

也有如此的

![dancer_4.png](/images/dancer/dancer_4.png)


---


### 添加控制逻辑[C]

在dancer中 **`TEST-APP/lib/TEST/APP.pm`** 是起控制作用的，在 **true** 之前添加以下几行

~~~
get '/check_backup' => sub{
	template 'check_class/check_database_backup';
};
post '/check_backup' => sub{
	my $mail_addr = param("email_addr");
	my @mail_list = split /\n/,$mail_addr;
	my $tmp_resault = '';
	foreach(@mail_list){
		$_ =~ s/(^\s+|\s+$)//g;
		chomp($_);
		next if ($_ eq '');
		unless  (  $_ =~ /\@163.com/ ){
		$tmp_resault .= "error receiver! pleaes retype!!!";
		last;
		}
		$tmp_resault .=`/home/dancer/bin/D_check_backup_for_db.pl  -p /home/dancer/bin/.passfile/abc_pass `;
	}
	return  '<pre>'.$tmp_resault.'</pre>';
	
};
~~~

---

### 添加展示层[V]

在dancer中 **`TEST-APP/views/`** 是控制显示的，创建 **TEST-APP/views/check_class/check_database_backup.tt**

~~~
[dancer@dancer-test ~]$ cat TEST-APP/views/check_class/check_database_backup.tt 
<form class="form-cr" method="POST" action="/check_backup">
  <h2 class="form-cr-heading">Please input the Email address :</h2>
  <% IF errmsg %><p class="alert alert-error"><% errmsg %></p><% END %>
<textarea name="email_addr" cols=40 rows=4>
Type your Email address here...
</textarea>
  <button class="btn btn-large btn-primary" type="submit">submit</button>
</form>

[dancer@dancer-test ~]$ 
~~~

---


### 添加功能逻辑[M]

~~~
[dancer@dancer-test bin]$ cat D_check_backup_for_db.pl 
#!/usr/bin/perl


#require Expect
#require Getopt::Std
#

use Expect;
use Getopt::Std;
use strict;

my (%opts,$host,$user,$pass);

getopts( 'p:h',\%opts );
&help_info() if $opts{h};
&help_info() unless $opts{p};

#bakdir info
my $bakdir="/data/backupdir";
my $baklog="$bakdir/backuplog/backup.log";
my $patt="innobackupex.*completed";
my $chkfile="xtrabackup_checkpoints";
my $num=10;

#comment
my $com1="-"x40;
my $com2="----LSN_Status"."-"x26;
my $com3="----Backup_Resault"."-"x22;

#set expect timeout 
my $timeout=3;


open PASSFILE,"< $opts{p}" or die "Can't open $opts{p}!";
while(<PASSFILE>){
        $_ =~ s/(^\s+|\s+$)//;
        chomp($_);
        ($host,$user,$pass)=split (/\s+/,$_);
}
close PASSFILE;

#autocheck of hostA and hostB
foreach (("hostA","hostB")){

#my $exp = Expect->spawn("ssh $user\@$host  'ssh $_ \"echo $com1; hostname; echo $com2; cat $bakdir/2015-*/$chkfile ; echo $com3;grep $patt  $baklog | tail -n $num \" ' ");
my $exp = Expect->spawn("ssh $user\@$host  'ssh $_ \"
	echo $com1;
	hostname; 
	echo $com2; 
	cat $bakdir/2015-*/$chkfile; 
	echo $com3;
	grep $patt  $baklog | tail -n $num; \" ' 
	");
$exp->expect($timeout,
        [ qr/\(yes\/no\)/i,sub { my $self = shift;$self->send("yes\n");exp_continue;}],
	    [ qr/password:/i,sub { my $self = shift;$self->send("$pass\n");exp_continue;}],
        );
}




sub help_info {
print <<EOF

Usage:
        $0 -p <password_file> [-h]
        -h optical argument
                display this help info
        -p specified the path of password file
Example:
        command:
         $0  -p /path/to/passwordfile
    
EOF
;
exit  0;
}
[dancer@dancer-test bin]$ 
~~~

> **Note:** 一般而言，密码不要直接写到功能逻辑里面，不灵活，不安全

---

## 访问测试

在本地浏览器中输入 **http://ip:5000/check_backup** 进行访问测试

注意服务端的防火墙对于 **5000** 端口要开放

![dancer_2.png](/images/dancer/dancer_2.png)

输入邮箱地址后，点击[submit]

![dancer_3.png](/images/dancer/dancer_3.png)



可以在其中加入更多的逻辑以实现更多的功能


---

[perldancer]:http://www.perldancer.org/
[Dancer2_cpan]:https://metacpan.org/pod/Dancer2
[Dancer2-0.163000]:https://cpan.metacpan.org/authors/id/X/XS/XSAWYERX/Dancer2-0.163000.tar.gz
[Dancer2_git]:https://github.com/PerlDancer/Dancer2
[cpan_sites]:http://www.cpan.org/SITES.html
