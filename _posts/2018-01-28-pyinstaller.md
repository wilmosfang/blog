---
layout: post
title: "pyinstaller"
author:  wilmosfang
date: 2018-01-28 02:24:46
image: '/assets/img/'
excerpt: '使用 PyInstaller 将 python 打包成 exe'
main-class: tools
color:  '#808080'
tags:
 - python
 - tools
 - pip
categories:
 - tools
twitter_text: 'use PyInstaller to package a python script into a exe'
introduction: 'the way to package a python script into a exe'
---



# 前言


**[PyInstaller][pyinstaller]** 可以将 python 程序打包成一个单一的 exe 可执行包

## 可以支持如下平台:

* Windows
* Linux
* Mac OS X
* FreeBSD
* Solaris
* AIX

## 可以支持的 python 版本:

* Python 2.7
* Python 3.3-3.6

>PyInstaller is a program that freezes (packages) Python programs into stand-alone executables, under Windows, Linux, Mac OS X, FreeBSD, Solaris and AIX. Its main advantages over similar tools are that PyInstaller works with Python 2.7 and 3.3—3.6, it builds smaller executables thanks to transparent compression, it is fully multi-platform, and use the OS support to load the dynamic libraries, thus ensuring full compatibility.

## 可以支持的包

**[PyInstaller][pyinstaller]** 的一个主要目标就是独立兼容第三方包(将第三方包的相关依赖都整合进来)

目前可以兼容的包列表

**[Supported Packages][supported_packages]**

这里分享一下 **[PyInstaller][pyinstaller]** 的简单使用方法

参考 **[PyInstaller Manual][pyinstaller_doc]**

> **Tip:** 当前版本 **PyInstaller 3.3.1**

---

# 操作

## 系统环境


![pyinstaller](/assets/img/pyinstaller/pyinstaller01.png)


python 版本

~~~
C:\Python27>python.exe --version
Python 2.7.14

C:\Python27>
~~~


## 安装

~~~
C:\Python27>python.exe -m pip install pyinstaller
Collecting pyinstaller
  Downloading PyInstaller-3.3.1.tar.gz (3.5MB)
    100% |████████████████████████████████| 3.5MB 176kB/s
Requirement already satisfied: setuptools in c:\python27\lib\site-packages (from pyinstaller)
Collecting pefile>=2017.8.1 (from pyinstaller)
  Downloading pefile-2017.11.5.tar.gz (61kB)
    100% |████████████████████████████████| 71kB 393kB/s
Collecting macholib>=1.8 (from pyinstaller)
  Downloading macholib-1.9-py2.py3-none-any.whl (40kB)
    100% |████████████████████████████████| 40kB 460kB/s
Collecting dis3 (from pyinstaller)
  Downloading dis3-0.1.2.tar.gz
Collecting future (from pyinstaller)
  Downloading future-0.16.0.tar.gz (824kB)
    100% |████████████████████████████████| 829kB 254kB/s
Collecting pypiwin32 (from pyinstaller)
  Downloading pypiwin32-222-py2.py3-none-any.whl
Collecting altgraph>=0.15 (from macholib>=1.8->pyinstaller)
  Downloading altgraph-0.15-py2.py3-none-any.whl
Collecting pywin32 (from pypiwin32->pyinstaller)
  Downloading pywin32-222-cp27-cp27m-win_amd64.whl (7.3MB)
    100% |████████████████████████████████| 7.3MB 110kB/s
Installing collected packages: future, pefile, altgraph, macholib, dis3, pywin32, pypiwin32, pyinstaller
  Running setup.py install for future ... done
  Running setup.py install for pefile ... done
  Running setup.py install for dis3 ... done
  Running setup.py install for pyinstaller ... done
Successfully installed altgraph-0.15 dis3-0.1.2 future-0.16.0 macholib-1.9 pefile-2017.11.5 pyinstaller-3.3.1 pypiwin32-222 pywin3
2-222

C:\Python27>
~~~

使用 pip 进行安装


## 路径

在 windows 中 Scripts 目录里存放着 python 的工具，其中 **pyinstaller** 就放在里面

~~~
C:\Python27>dir Scripts
 驱动器 C 中的卷没有标签。
 卷的序列号是 2C0E-49DD

 C:\Python27\Scripts 的目录

2018/01/28  00:22    <DIR>          .
2018/01/28  00:22    <DIR>          ..
2018/01/24  23:43            98,153 easy_install-2.7.exe
2018/01/24  23:43            98,153 easy_install.exe
2018/01/28  00:22               402 futurize-script.py
2018/01/28  00:22            74,752 futurize.exe
2018/01/28  00:22            98,141 macho_dump.exe
2018/01/28  00:22            98,141 macho_find.exe
2018/01/28  00:22            98,147 macho_standalone.exe
2018/01/28  00:22               406 pasteurize-script.py
2018/01/28  00:22            74,752 pasteurize.exe
2018/01/24  23:43            98,125 pip.exe
2018/01/24  23:43            98,125 pip2.7.exe
2018/01/24  23:43            98,125 pip2.exe
2018/01/28  00:22               434 pyi-archive_viewer-script.py
2018/01/28  00:22            74,752 pyi-archive_viewer.exe
2018/01/28  00:22               424 pyi-bindepend-script.py
2018/01/28  00:22            74,752 pyi-bindepend.exe
2018/01/28  00:22               430 pyi-grab_version-script.py
2018/01/28  00:22            74,752 pyi-grab_version.exe
2018/01/28  00:22               422 pyi-makespec-script.py
2018/01/28  00:22            74,752 pyi-makespec.exe
2018/01/28  00:22               428 pyi-set_version-script.py
2018/01/28  00:22            74,752 pyi-set_version.exe
2018/01/28  00:22               420 pyinstaller-script.py
2018/01/28  00:22            74,752 pyinstaller.exe
2018/01/28  00:22            24,184 pywin32_postinstall.py
2018/01/28  00:22            18,837 pywin32_postinstall.pyc
2018/01/28  00:22             3,390 pywin32_testall.py
2018/01/28  00:22             3,044 pywin32_testall.pyc
              28 个文件      1,435,947 字节
               2 个目录 42,461,179,904 可用字节

C:\Python27>
~~~

## 查看信息

~~~

C:\Python27>Scripts\pyinstaller.exe -v
3.3.1

C:\Python27>Scripts\pyinstaller.exe -h
usage: pyinstaller [-h] [-v] [-D] [-F] [--specpath DIR] [-n NAME]
                   [--add-data <SRC;DEST or SRC:DEST>]
                   [--add-binary <SRC;DEST or SRC:DEST>] [-p DIR]
                   [--hidden-import MODULENAME]
                   [--additional-hooks-dir HOOKSPATH]
                   [--runtime-hook RUNTIME_HOOKS] [--exclude-module EXCLUDES]
                   [--key KEY] [-d] [-s] [--noupx] [-c] [-w]
                   [-i <FILE.ico or FILE.exe,ID or FILE.icns>]
                   [--version-file FILE] [-m <FILE or XML>] [-r RESOURCE]
                   [--uac-admin] [--uac-uiaccess] [--win-private-assemblies]
                   [--win-no-prefer-redirects]
                   [--osx-bundle-identifier BUNDLE_IDENTIFIER]
                   [--runtime-tmpdir PATH] [--distpath DIR]
                   [--workpath WORKPATH] [-y] [--upx-dir UPX_DIR] [-a]
                   [--clean] [--log-level LEVEL]
                   scriptname [scriptname ...]

positional arguments:
  scriptname            name of scriptfiles to be processed or exactly one
                        .spec-file. If a .spec-file is specified, most options
                        are unnecessary and are ignored.

optional arguments:
  -h, --help            show this help message and exit
  -v, --version         Show program version info and exit.
  --distpath DIR        Where to put the bundled app (default: .\dist)
  --workpath WORKPATH   Where to put all the temporary work files, .log, .pyz
                        and etc. (default: .\build)
  -y, --noconfirm       Replace output directory (default:
                        SPECPATH\dist\SPECNAME) without asking for
                        confirmation
  --upx-dir UPX_DIR     Path to UPX utility (default: search the execution
                        path)
  -a, --ascii           Do not include unicode encoding support (default:
                        included if available)
  --clean               Clean PyInstaller cache and remove temporary files
                        before building.
  --log-level LEVEL     Amount of detail in build-time console messages. LEVEL
                        may be one of TRACE, DEBUG, INFO, WARN, ERROR,
                        CRITICAL (default: INFO).

What to generate:
  -D, --onedir          Create a one-folder bundle containing an executable
                        (default)
  -F, --onefile         Create a one-file bundled executable.
  --specpath DIR        Folder to store the generated spec file (default:
                        current directory)
  -n NAME, --name NAME  Name to assign to the bundled app and spec file
                        (default: first script's basename)

What to bundle, where to search:
  --add-data <SRC;DEST or SRC:DEST>
                        Additional non-binary files or folders to be added to
                        the executable. The path separator is platform
                        specific, ``os.pathsep`` (which is ``;`` on Windows
                        and ``:`` on most unix systems) is used. This option
                        can be used multiple times.
  --add-binary <SRC;DEST or SRC:DEST>
                        Additional binary files to be added to the executable.
                        See the ``--add-data`` option for more details. This
                        option can be used multiple times.
  -p DIR, --paths DIR   A path to search for imports (like using PYTHONPATH).
                        Multiple paths are allowed, separated by ';', or use
                        this option multiple times
  --hidden-import MODULENAME, --hiddenimport MODULENAME
                        Name an import not visible in the code of the
                        script(s). This option can be used multiple times.
  --additional-hooks-dir HOOKSPATH
                        An additional path to search for hooks. This option
                        can be used multiple times.
  --runtime-hook RUNTIME_HOOKS
                        Path to a custom runtime hook file. A runtime hook is
                        code that is bundled with the executable and is
                        executed before any other code or module to set up
                        special features of the runtime environment. This
                        option can be used multiple times.
  --exclude-module EXCLUDES
                        Optional module or package (the Python name, not the
                        path name) that will be ignored (as though it was not
                        found). This option can be used multiple times.
  --key KEY             The key used to encrypt Python bytecode.

How to generate:
  -d, --debug           Tell the bootloader to issue progress messages while
                        initializing and starting the bundled app. Used to
                        diagnose problems with missing imports.
  -s, --strip           Apply a symbol-table strip to the executable and
                        shared libs (not recommended for Windows)
  --noupx               Do not use UPX even if it is available (works
                        differently between Windows and *nix)

Windows and Mac OS X specific options:
  -c, --console, --nowindowed
                        Open a console window for standard i/o (default)
  -w, --windowed, --noconsole
                        Windows and Mac OS X: do not provide a console window
                        for standard i/o. On Mac OS X this also triggers
                        building an OS X .app bundle. This option is ignored
                        in *NIX systems.
  -i <FILE.ico or FILE.exe,ID or FILE.icns>, --icon <FILE.ico or FILE.exe,ID or FILE.icns>
                        FILE.ico: apply that icon to a Windows executable.
                        FILE.exe,ID, extract the icon with ID from an exe.
                        FILE.icns: apply the icon to the .app bundle on Mac OS
                        X

Windows specific options:
  --version-file FILE   add a version resource from FILE to the exe
  -m <FILE or XML>, --manifest <FILE or XML>
                        add manifest FILE or XML to the exe
  -r RESOURCE, --resource RESOURCE
                        Add or update a resource to a Windows executable. The
                        RESOURCE is one to four items,
                        FILE[,TYPE[,NAME[,LANGUAGE]]]. FILE can be a data file
                        or an exe/dll. For data files, at least TYPE and NAME
                        must be specified. LANGUAGE defaults to 0 or may be
                        specified as wildcard * to update all resources of the
                        given TYPE and NAME. For exe/dll files, all resources
                        from FILE will be added/updated to the final
                        executable if TYPE, NAME and LANGUAGE are omitted or
                        specified as wildcard *.This option can be used
                        multiple times.
  --uac-admin           Using this option creates a Manifest which will
                        request elevation upon application restart.
  --uac-uiaccess        Using this option allows an elevated application to
                        work with Remote Desktop.

Windows Side-by-side Assembly searching options (advanced):
  --win-private-assemblies
                        Any Shared Assemblies bundled into the application
                        will be changed into Private Assemblies. This means
                        the exact versions of these assemblies will always be
                        used, and any newer versions installed on user
                        machines at the system level will be ignored.
  --win-no-prefer-redirects
                        While searching for Shared or Private Assemblies to
                        bundle into the application, PyInstaller will prefer
                        not to follow policies that redirect to newer
                        versions, and will try to bundle the exact versions of
                        the assembly.

Mac OS X specific options:
  --osx-bundle-identifier BUNDLE_IDENTIFIER
                        Mac OS X .app bundle identifier is used as the default
                        unique program name for code signing purposes. The
                        usual form is a hierarchical name in reverse DNS
                        notation. For example:
                        com.mycompany.department.appname (default: first
                        script's basename)

Rarely used special options:
  --runtime-tmpdir PATH
                        Where to extract libraries and support files in
                        `onefile`-mode. If this option is given, the
                        bootloader will ignore any temp-folder location
                        defined by the run-time OS. The ``_MEIxxxxxx``-folder
                        will be created here. Please use this option only if
                        you know what you are doing.

C:\Python27>
~~~


## 准备 python 脚本

~~~
C:\Python27\test>type command.py
import os
import re

command ='netstat -ant|find "TCP"| find "LISTENING"'
res=os.popen(command)
port_set=set()

for line in res:
    port_set.add(re.split(':',re.split('\s+',line)[2])[-1])

print '{"data":[',
for port in port_set:
    print "{\"{#OPENPORT}\":\""+ port + "\"},",
print  '{"{#OPENPORT}":"END"}]}'
C:\Python27\test>C:\Python27\python.exe command.py
{"data":[ {"{#OPENPORT}":"10000"}, {"{#OPENPORT}":"49757"}, {"{#OPENPORT}":"35311"}, {"{#OPENPORT}":"554"}, {"{#OPENPORT}":"49759"
}, {"{#OPENPORT}":"135"}, {"{#OPENPORT}":"139"}, {"{#OPENPORT}":"80"}, {"{#OPENPORT}":"5357"}, {"{#OPENPORT}":"12292"}, {"{#OPENPO
RT}":"12291"}, {"{#OPENPORT}":"49159"}, {"{#OPENPORT}":"2869"}, {"{#OPENPORT}":"49152"}, {"{#OPENPORT}":"49153"}, {"{#OPENPORT}":"
49154"}, {"{#OPENPORT}":"8733"}, {"{#OPENPORT}":"3587"}, {"{#OPENPORT}":"443"}, {"{#OPENPORT}":"445"}, {"{#OPENPORT}":"9007"}, {"{
#OPENPORT}":"49761"}, {"{#OPENPORT}":"10243"}, {"{#OPENPORT}":"28317"}, {"{#OPENPORT}":"49160"}, {"{#OPENPORT}":"49162"}, {"{#OPEN
PORT}":"5939"}, {"{#OPENPORT}":"END"}]}

C:\Python27\test>
~~~

这个脚本的作用就是查看本地哪些 TCP 的端口有打开并且处于监听状态


## 打包脚本

~~~
C:\Python27\test>C:\Python27\Scripts\pyinstaller.exe -F command.py
171 INFO: PyInstaller: 3.3.1
174 INFO: Python: 2.7.14
175 INFO: Platform: Windows-7-6.1.7600-SP0
177 INFO: wrote C:\Python27\test\command.spec
179 INFO: UPX is not available.
182 INFO: Extending PYTHONPATH with paths
['C:\\Python27\\test', 'C:\\Python27\\test']
183 INFO: checking Analysis
184 INFO: Building Analysis because out00-Analysis.toc is non existent
185 INFO: Initializing module dependency graph...
187 INFO: Initializing module graph hooks...
263 INFO: running Analysis out00-Analysis.toc
273 INFO: Adding Microsoft.VC90.CRT to dependent assemblies of final executable
  required by C:\Python27\python.exe
907 INFO: Found C:\Windows\WinSxS\Manifests\amd64_policy.9.0.microsoft.vc90.crt_1fc8b3b9a1e18e3b_9.0.30729.1_none_3da38fdebd0e6822
.manifest
911 INFO: Found C:\Windows\WinSxS\Manifests\amd64_policy.9.0.microsoft.vc90.crt_1fc8b3b9a1e18e3b_9.0.30729.4926_none_accf10dbe1dc8
ba2.manifest
955 INFO: Searching for assembly amd64_Microsoft.VC90.CRT_1fc8b3b9a1e18e3b_9.0.30729.4926_none ...
958 INFO: Found manifest C:\Windows\WinSxS\Manifests\amd64_microsoft.vc90.crt_1fc8b3b9a1e18e3b_9.0.30729.4926_none_08e1a05ba83fe55
4.manifest
960 INFO: Searching for file msvcr90.dll
961 INFO: Found file C:\Windows\WinSxS\amd64_microsoft.vc90.crt_1fc8b3b9a1e18e3b_9.0.30729.4926_none_08e1a05ba83fe554\msvcr90.dll
962 INFO: Searching for file msvcp90.dll
963 INFO: Found file C:\Windows\WinSxS\amd64_microsoft.vc90.crt_1fc8b3b9a1e18e3b_9.0.30729.4926_none_08e1a05ba83fe554\msvcp90.dll
963 INFO: Searching for file msvcm90.dll
966 INFO: Found file C:\Windows\WinSxS\amd64_microsoft.vc90.crt_1fc8b3b9a1e18e3b_9.0.30729.4926_none_08e1a05ba83fe554\msvcm90.dll
1005 INFO: Found C:\Windows\WinSxS\Manifests\amd64_policy.9.0.microsoft.vc90.crt_1fc8b3b9a1e18e3b_9.0.30729.1_none_3da38fdebd0e682
2.manifest
1008 INFO: Found C:\Windows\WinSxS\Manifests\amd64_policy.9.0.microsoft.vc90.crt_1fc8b3b9a1e18e3b_9.0.30729.4926_none_accf10dbe1dc
8ba2.manifest
1011 INFO: Adding redirect Microsoft.VC90.CRT version (9, 0, 21022, 8) -> (9, 0, 30729, 4926)
1124 INFO: Caching module hooks...
1140 INFO: Analyzing C:\Python27\test\command.py
2549 INFO: Loading module hooks...
2551 INFO: Loading module hook "hook-encodings.py"...
3259 INFO: Looking for ctypes DLLs
3261 INFO: Analyzing run-time hooks ...
3266 INFO: Looking for dynamic libraries
3325 INFO: Looking for eggs
3327 INFO: Using Python library C:\Windows\system32\python27.dll
3328 INFO: Found binding redirects:
[BindingRedirect(name=u'Microsoft.VC90.CRT', language=None, arch=u'amd64', oldVersion=(9, 0, 21022, 8), newVersion=(9, 0, 30729, 4
926), publicKeyToken=u'1fc8b3b9a1e18e3b')]
3331 INFO: Warnings written to C:\Python27\test\build\command\warncommand.txt
3351 INFO: Graph cross-reference written to C:\Python27\test\build\command\xref-command.html
3384 INFO: checking PYZ
3384 INFO: Building PYZ because out00-PYZ.toc is non existent
3387 INFO: Building PYZ (ZlibArchive) C:\Python27\test\build\command\out00-PYZ.pyz
3607 INFO: Building PYZ (ZlibArchive) C:\Python27\test\build\command\out00-PYZ.pyz completed successfully.
3637 INFO: checking PKG
3639 INFO: Building PKG because out00-PKG.toc is non existent
3640 INFO: Building PKG (CArchive) out00-PKG.pkg
3697 INFO: Redirecting Microsoft.VC90.CRT version (9, 0, 21022, 8) -> (9, 0, 30729, 4926)
4930 INFO: Building PKG (CArchive) out00-PKG.pkg completed successfully.
4935 INFO: Bootloader C:\Python27\lib\site-packages\PyInstaller\bootloader\Windows-64bit\run.exe
4936 INFO: checking EXE
4937 INFO: Building EXE because out00-EXE.toc is non existent
4937 INFO: Building EXE from out00-EXE.toc
4938 INFO: Appending archive to EXE C:\Python27\test\dist\command.exe
4947 INFO: Building EXE from out00-EXE.toc completed successfully.

C:\Python27\test>dir
 驱动器 C 中的卷没有标签。
 卷的序列号是 2C0E-49DD

 C:\Python27\test 的目录

2018/01/28  11:24    <DIR>          .
2018/01/28  11:24    <DIR>          ..
2018/01/28  11:24    <DIR>          build
2018/01/25  01:36               324 command.py
2018/01/28  11:24               747 command.spec
2018/01/28  11:24    <DIR>          dist
               2 个文件          1,071 字节
               4 个目录 42,452,365,312 可用字节

C:\Python27\test>
~~~


## 目录结构

~~~
C:\Python27\test>dir
 驱动器 C 中的卷没有标签。
 卷的序列号是 2C0E-49DD

 C:\Python27\test 的目录

2018/01/28  11:24    <DIR>          .
2018/01/28  11:24    <DIR>          ..
2018/01/28  11:24    <DIR>          build
2018/01/25  01:36               324 command.py
2018/01/28  11:24               747 command.spec
2018/01/28  11:24    <DIR>          dist
               2 个文件          1,071 字节
               4 个目录 42,452,295,680 可用字节

C:\Python27\test>dir build
 驱动器 C 中的卷没有标签。
 卷的序列号是 2C0E-49DD

 C:\Python27\test\build 的目录

2018/01/28  11:24    <DIR>          .
2018/01/28  11:24    <DIR>          ..
2018/01/28  11:24    <DIR>          command
               0 个文件              0 字节
               3 个目录 42,452,033,536 可用字节

C:\Python27\test>dir dist
 驱动器 C 中的卷没有标签。
 卷的序列号是 2C0E-49DD

 C:\Python27\test\dist 的目录

2018/01/28  11:24    <DIR>          .
2018/01/28  11:24    <DIR>          ..
2018/01/28  11:24         3,895,500 command.exe
               1 个文件      3,895,500 字节
               2 个目录 42,451,783,680 可用字节

C:\Python27\test>
~~~

我们的最终结果为 **dist/command.exe**

其它都是打包过程中生成的中间文件，在打包完成后，可以删除

## 运行命令

~~~
C:\Python27\test>dist\command.exe
{"data":[ {"{#OPENPORT}":"10000"}, {"{#OPENPORT}":"49757"}, {"{#OPENPORT}":"35311"}, {"{#OPENPORT}":"554"}, {"{#OPENPORT}":"49759"
}, {"{#OPENPORT}":"135"}, {"{#OPENPORT}":"139"}, {"{#OPENPORT}":"80"}, {"{#OPENPORT}":"5357"}, {"{#OPENPORT}":"12292"}, {"{#OPENPO
RT}":"12291"}, {"{#OPENPORT}":"49159"}, {"{#OPENPORT}":"2869"}, {"{#OPENPORT}":"49152"}, {"{#OPENPORT}":"49153"}, {"{#OPENPORT}":"
49154"}, {"{#OPENPORT}":"8733"}, {"{#OPENPORT}":"3587"}, {"{#OPENPORT}":"443"}, {"{#OPENPORT}":"445"}, {"{#OPENPORT}":"9007"}, {"{
#OPENPORT}":"49761"}, {"{#OPENPORT}":"10243"}, {"{#OPENPORT}":"28317"}, {"{#OPENPORT}":"49160"}, {"{#OPENPORT}":"49162"}, {"{#OPEN
PORT}":"5939"}, {"{#OPENPORT}":"END"}]}

C:\Python27\test>
~~~

运行结果符合预期

把这个 exe 文件拷贝到其它相同版本的系统中也是可以正常运行的


---

# 总结

window 系统自带的 bat 在实现复杂处理的时候非常不给力

这时使用 python 就是一个好的选择，但是给所有目标系统安装一个 python 运行环境，又是一件很有挑战的事儿

使用 pyinstaller 就很好的解决了这个问题

这是一个最简单实用的例子

* TOC
{:toc}


---

[pyinstaller]:http://www.pyinstaller.org/
[supported_packages]:https://github.com/pyinstaller/pyinstaller/wiki/Supported-Packages
[pyinstaller_doc]:https://pyinstaller.readthedocs.io/en/stable/
