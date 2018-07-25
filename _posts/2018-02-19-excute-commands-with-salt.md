---
layout: post
title: "Excute Commands with SaltStack"
author:  wilmosfang
date: 2018-02-19 14:07:50
image: '/assets/img/'
excerpt: '使用 Saltstack 执行命令'
main-class: saltstack
color: '#313233'
tags:
 - saltstack
categories:
 - saltstack
twitter_text: 'Execute Commands with SaltStack'
introduction: 'Execute Commands with SaltStack'
---


## 前言

**[SaltStack][saltstack]** 是一款高性能的自动化运维工具

类似的工具还有 **Puppet、Chef、Ansible**，他们之间可以相互替代，但是哪一个更好，我就不在此引发圣战了

这里分享一下 **[SaltStack][saltstack]** 执行命令的方法

参考 **[EXECUTE COMMANDS][saltstack_remotex]**

> **Tip:** 当前的版本为 **Latest release: 2017.7.3 (February 5, 2018)**

---

# 操作

## 环境

~~~
[root@h209 ~]# hostnamectl
   Static hostname: h209
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 33dc28f7e76c4903ad9b603b77e29a7c
           Boot ID: 6ce363851e6d4a519c97c067a58296ae
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-514.21.1.el7.x86_64
      Architecture: x86-64
[root@h209 ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:0b:e9:0b brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 85529sec preferred_lft 85529sec
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:36:8b:0c brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.209/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
[root@h209 ~]# salt -V
Salt Version:
           Salt: 2017.7.3

Dependency Versions:
           cffi: 1.6.0
       cherrypy: Not Installed
       dateutil: 1.5
      docker-py: Not Installed
          gitdb: Not Installed
      gitpython: Not Installed
          ioflo: Not Installed
         Jinja2: 2.7.2
        libgit2: Not Installed
        libnacl: Not Installed
       M2Crypto: Not Installed
           Mako: 0.8.1
   msgpack-pure: Not Installed
 msgpack-python: 0.5.1
   mysql-python: Not Installed
      pycparser: 2.14
       pycrypto: 2.6.1
   pycryptodome: Not Installed
         pygit2: Not Installed
         Python: 2.7.5 (default, Nov  6 2016, 00:28:07)
   python-gnupg: Not Installed
         PyYAML: 3.11
          PyZMQ: 15.3.0
           RAET: Not Installed
          smmap: Not Installed
        timelib: Not Installed
        Tornado: 4.2.1
            ZMQ: 4.1.4

System Versions:
           dist: centos 7.3.1611 Core
         locale: UTF-8
        machine: x86_64
        release: 3.10.0-514.21.1.el7.x86_64
         system: Linux
        version: CentOS Linux 7.3.1611 Core

[root@h209 ~]#
~~~



## 命令语法


~~~
[root@h209 ~]# salt-key -L
Accepted Keys:
much
test-master
test-minion
Denied Keys:
Unaccepted Keys:
Rejected Keys:
[root@h209 ~]# salt "*" cmd.run "echo test"
much:
    test
test-master:
    test
test-minion:
    test
[root@h209 ~]#
~~~

一般而言，语法如下

~~~
salt <target> <module.function> [argument]
~~~

* **target:** 指定哪些系统来执行以下命令
* **command (module.function):** 命令由模块和方法构成，这是 salt 的处理逻辑
* **arguments:** 处理逻辑需要的参数，有些没有参数，就不必添加，有些参数具备默认值，也可以不必加，但是可以通过改变参数值来调整处理逻辑

**salt** 加 **执行对象** 加 **模块.方法** 加 **参数**

## 查看磁盘用量

~~~
[root@h209 ~]# salt "*" disk.usage
test-minion:
    ----------
    /:
        ----------
        1K-blocks:
            40137576
        available:
            33086520
        capacity:
            18%
        filesystem:
            /dev/mapper/cl-root
        used:
            7051056
    /boot:
        ----------
        1K-blocks:
            1038336
        available:
            841580
        capacity:
            19%
        filesystem:
            /dev/sda1
        used:
            196756
    /dev:
        ----------
        1K-blocks:
            2008008
        available:
            2008008
        capacity:
            0%
        filesystem:
            devtmpfs
        used:
            0
    /dev/shm:
        ----------
        1K-blocks:
            2023420
        available:
            2023408
        capacity:
            1%
        filesystem:
            tmpfs
        used:
            12
    /home:
        ----------
        1K-blocks:
            19593216
        available:
            16607368
        capacity:
            16%
        filesystem:
            /dev/mapper/cl-home
        used:
            2985848
    /media/cdrom:
        ----------
        1K-blocks:
            8086368
        available:
            0
        capacity:
            100%
        filesystem:
            /dev/sr0
        used:
            8086368
    /media/sf_script:
        ----------
        1K-blocks:
            269029008
        available:
            39847740
        capacity:
            86%
        filesystem:
            script
        used:
            229181268
    /run:
        ----------
        1K-blocks:
            2023420
        available:
            2014552
        capacity:
            1%
        filesystem:
            tmpfs
        used:
            8868
    /sys/fs/cgroup:
        ----------
        1K-blocks:
            2023420
        available:
            2023420
        capacity:
            0%
        filesystem:
            tmpfs
        used:
            0
test-master:
    ----------
    /:
        ----------
        1K-blocks:
            40137576
        available:
            31538344
        capacity:
            22%
        filesystem:
            /dev/mapper/cl-root
        used:
            8599232
    /boot:
        ----------
        1K-blocks:
            1038336
        available:
            841580
        capacity:
            19%
        filesystem:
            /dev/sda1
        used:
            196756
    /dev:
        ----------
        1K-blocks:
            2008008
        available:
            2008008
        capacity:
            0%
        filesystem:
            devtmpfs
        used:
            0
    /dev/shm:
        ----------
        1K-blocks:
            2023420
        available:
            2023392
        capacity:
            1%
        filesystem:
            tmpfs
        used:
            28
    /home:
        ----------
        1K-blocks:
            19593216
        available:
            16607320
        capacity:
            16%
        filesystem:
            /dev/mapper/cl-home
        used:
            2985896
    /media/cdrom:
        ----------
        1K-blocks:
            8086368
        available:
            0
        capacity:
            100%
        filesystem:
            /dev/sr0
        used:
            8086368
    /media/sf_usb:
        ----------
        1K-blocks:
            269029008
        available:
            39847740
        capacity:
            86%
        filesystem:
            usb
        used:
            229181268
    /run:
        ----------
        1K-blocks:
            2023420
        available:
            2014500
        capacity:
            1%
        filesystem:
            tmpfs
        used:
            8920
    /run/user/0:
        ----------
        1K-blocks:
            404684
        available:
            404684
        capacity:
            0%
        filesystem:
            tmpfs
        used:
            0
    /run/user/1004:
        ----------
        1K-blocks:
            404684
        available:
            404684
        capacity:
            0%
        filesystem:
            tmpfs
        used:
            0
    /sys/fs/cgroup:
        ----------
        1K-blocks:
            2023420
        available:
            2023420
        capacity:
            0%
        filesystem:
            tmpfs
        used:
            0
much:
    ----------
    /:
        ----------
        1K-blocks:
            40137576
        available:
            34442576
        capacity:
            15%
        filesystem:
            /dev/mapper/cl-root
        used:
            5695000
    /boot:
        ----------
        1K-blocks:
            1038336
        available:
            841580
        capacity:
            19%
        filesystem:
            /dev/sda1
        used:
            196756
    /dev:
        ----------
        1K-blocks:
            2008008
        available:
            2008008
        capacity:
            0%
        filesystem:
            devtmpfs
        used:
            0
    /dev/shm:
        ----------
        1K-blocks:
            2023420
        available:
            2023408
        capacity:
            1%
        filesystem:
            tmpfs
        used:
            12
    /home:
        ----------
        1K-blocks:
            19593216
        available:
            16607368
        capacity:
            16%
        filesystem:
            /dev/mapper/cl-home
        used:
            2985848
    /media/cdrom:
        ----------
        1K-blocks:
            8086368
        available:
            0
        capacity:
            100%
        filesystem:
            /dev/sr0
        used:
            8086368
    /run:
        ----------
        1K-blocks:
            2023420
        available:
            2014588
        capacity:
            1%
        filesystem:
            tmpfs
        used:
            8832
    /sys/fs/cgroup:
        ----------
        1K-blocks:
            2023420
        available:
            2023420
        capacity:
            0%
        filesystem:
            tmpfs
        used:
            0
[root@h209 ~]#
~~~

**disk.usage** 就没有参数


## 查看网络配置

~~~
[root@h209 ~]# salt '*' network.interfaces
much:
    ----------
    enp0s3:
        ----------
        hwaddr:
            08:00:27:e3:df:87
        inet:
            |_
              ----------
              address:
                  10.0.2.15
              broadcast:
                  10.0.2.255
              label:
                  enp0s3
              netmask:
                  255.255.255.0
        up:
            True
    enp0s8:
        ----------
        hwaddr:
            08:00:27:d3:ec:e7
        inet:
            |_
              ----------
              address:
                  192.168.56.207
              broadcast:
                  192.168.56.255
              label:
                  enp0s8
              netmask:
                  255.255.255.0
        up:
            True
    lo:
        ----------
        hwaddr:
            00:00:00:00:00:00
        inet:
            |_
              ----------
              address:
                  127.0.0.1
              broadcast:
                  None
              label:
                  lo
              netmask:
                  255.0.0.0
        up:
            True
    virbr0:
        ----------
        hwaddr:
            52:54:00:16:5e:11
        inet:
            |_
              ----------
              address:
                  192.168.122.1
              broadcast:
                  192.168.122.255
              label:
                  virbr0
              netmask:
                  255.255.255.0
        up:
            True
    virbr0-nic:
        ----------
        hwaddr:
            52:54:00:16:5e:11
        up:
            False
test-master:
    ----------
    enp0s3:
        ----------
        hwaddr:
            08:00:27:0b:e9:0b
        inet:
            |_
              ----------
              address:
                  10.0.2.15
              broadcast:
                  10.0.2.255
              label:
                  enp0s3
              netmask:
                  255.255.255.0
        up:
            True
    enp0s8:
        ----------
        hwaddr:
            08:00:27:36:8b:0c
        inet:
            |_
              ----------
              address:
                  192.168.56.209
              broadcast:
                  192.168.56.255
              label:
                  enp0s8
              netmask:
                  255.255.255.0
        up:
            True
    lo:
        ----------
        hwaddr:
            00:00:00:00:00:00
        inet:
            |_
              ----------
              address:
                  127.0.0.1
              broadcast:
                  None
              label:
                  lo
              netmask:
                  255.0.0.0
        up:
            True
    virbr0:
        ----------
        hwaddr:
            52:54:00:16:5e:11
        inet:
            |_
              ----------
              address:
                  192.168.122.1
              broadcast:
                  192.168.122.255
              label:
                  virbr0
              netmask:
                  255.255.255.0
        up:
            True
    virbr0-nic:
        ----------
        hwaddr:
            52:54:00:16:5e:11
        up:
            False
test-minion:
    ----------
    enp0s3:
        ----------
        hwaddr:
            08:00:27:e3:df:87
        inet:
            |_
              ----------
              address:
                  10.0.2.15
              broadcast:
                  10.0.2.255
              label:
                  enp0s3
              netmask:
                  255.255.255.0
        up:
            True
    enp0s8:
        ----------
        hwaddr:
            08:00:27:d3:ec:e7
        inet:
            |_
              ----------
              address:
                  192.168.56.208
              broadcast:
                  192.168.56.255
              label:
                  enp0s8
              netmask:
                  255.255.255.0
        up:
            True
    lo:
        ----------
        hwaddr:
            00:00:00:00:00:00
        inet:
            |_
              ----------
              address:
                  127.0.0.1
              broadcast:
                  None
              label:
                  lo
              netmask:
                  255.0.0.0
        up:
            True
    virbr0:
        ----------
        hwaddr:
            52:54:00:16:5e:11
        inet:
            |_
              ----------
              address:
                  192.168.122.1
              broadcast:
                  192.168.122.255
              label:
                  virbr0
              netmask:
                  255.255.255.0
        up:
            True
    virbr0-nic:
        ----------
        hwaddr:
            52:54:00:16:5e:11
        up:
            False
[root@h209 ~]#
~~~


## 安装软件

~~~
[root@h209 ~]# salt "*" cmd.run "rpm -qa | grep cowsay"
test-minion:
much:
test-master:
    cowsay-3.04-4.el7.noarch
ERROR: Minions returned with non-zero exit code
[root@h209 ~]# salt "*" pkg.install cowsay
test-master:
    ----------
much:
    ----------
    cowsay:
        ----------
        new:
            3.04-4.el7
        old:
test-minion:
    ----------
    cowsay:
        ----------
        new:
            3.04-4.el7
        old:
[root@h209 ~]# salt "*" cmd.run "rpm -qa | grep cowsay"
much:
    cowsay-3.04-4.el7.noarch
test-minion:
    cowsay-3.04-4.el7.noarch
test-master:
    cowsay-3.04-4.el7.noarch
[root@h209 ~]#
~~~

执行新安装的命令进行测试

~~~
[root@h209 ~]# salt "*" cmd.run "cowsay welcome"
much:
     _________
    < welcome >
     ---------
            \   ^__^
             \  (oo)\_______
                (__)\       )\/\
                    ||----w |
                    ||     ||
test-minion:
     _________
    < welcome >
     ---------
            \   ^__^
             \  (oo)\_______
                (__)\       )\/\
                    ||----w |
                    ||     ||
test-master:
     _________
    < welcome >
     ---------
            \   ^__^
             \  (oo)\_______
                (__)\       )\/\
                    ||----w |
                    ||     ||
[root@h209 ~]#
~~~


## 获取文档

可以通过 **`sys.doc`** 来获取模块方法信息

~~~
[root@h209 ~]# salt '*' sys.doc pkg.install
pkg.install:

    Changed in version 2015.8.12,2016.3.3,2016.11.0
        On minions running systemd>=205, `systemd-run(1)`_ is now used to
        isolate commands which modify installed packages from the
        ``salt-minion`` daemon's control group. This is done to keep systemd
        from killing any yum/dnf commands spawned by Salt when the
        ``salt-minion`` service is restarted. (see ``KillMode`` in the
        `systemd.kill(5)`_ manpage for more information). If desired, usage of
        `systemd-run(1)`_ can be suppressed by setting a :mod:`config option
        <salt.modules.config.get>` called ``systemd.scope``, with a value of
        ``False`` (no quotes).

    .. _`systemd-run(1)`: https://www.freedesktop.org/software/systemd/man/systemd-run.html
    .. _`systemd.kill(5)`: https://www.freedesktop.org/software/systemd/man/systemd.kill.html

    Install the passed package(s), add refresh=True to clean the yum database
    before package is installed.

    name
        The name of the package to be installed. Note that this parameter is
        ignored if either "pkgs" or "sources" is passed. Additionally, please
        note that this option can only be used to install packages from a
        software repository. To install a package file manually, use the
        "sources" option.

        32-bit packages can be installed on 64-bit systems by appending the
        architecture designation (``.i686``, ``.i586``, etc.) to the end of the
        package name.

        CLI Example:

            salt '*' pkg.install <package name>

    refresh
        Whether or not to update the yum database before executing.

    reinstall
        Specifying reinstall=True will use ``yum reinstall`` rather than
        ``yum install`` for requested packages that are already installed.

        If a version is specified with the requested package, then
        ``yum reinstall`` will only be used if the installed version
        matches the requested version.

        Works with ``sources`` when the package header of the source can be
        matched to the name and version of an installed package.

        New in version 2014.7.0

    skip_verify
        Skip the GPG verification check (e.g., ``--nogpgcheck``)

    downloadonly
        Only download the packages, do not install.

    version
        Install a specific version of the package, e.g. 1.2.3-4.el5. Ignored
        if "pkgs" or "sources" is passed.

    update_holds : False
        If ``True``, and this function would update the package version, any
        packages held using the yum/dnf "versionlock" plugin will be unheld so
        that they can be updated. Otherwise, if this function attempts to
        update a held package, the held package(s) will be skipped and an
        error will be raised.

        New in version 2016.11.0


    Repository Options:

    fromrepo
        Specify a package repository (or repositories) from which to install.
        (e.g., ``yum --disablerepo='*' --enablerepo='somerepo'``)

    enablerepo (ignored if ``fromrepo`` is specified)
        Specify a disabled package repository (or repositories) to enable.
        (e.g., ``yum --enablerepo='somerepo'``)

    disablerepo (ignored if ``fromrepo`` is specified)
        Specify an enabled package repository (or repositories) to disable.
        (e.g., ``yum --disablerepo='somerepo'``)

    disableexcludes
        Disable exclude from main, for a repo or for everything.
        (e.g., ``yum --disableexcludes='main'``)

        New in version 2014.7.0


    Multiple Package Installation Options:

    pkgs
        A list of packages to install from a software repository. Must be
        passed as a python list. A specific version number can be specified
        by using a single-element dict representing the package and its
        version.

        CLI Examples:

            salt '*' pkg.install pkgs='["foo", "bar"]'
            salt '*' pkg.install pkgs='["foo", {"bar": "1.2.3-4.el5"}]'

    sources
        A list of RPM packages to install. Must be passed as a list of dicts,
        with the keys being package names, and the values being the source URI
        or local path to the package.

        CLI Example:

            salt '*' pkg.install sources='[{"foo": "salt://foo.rpm"}, {"bar": "salt://bar.rpm"}]'

    normalize : True
        Normalize the package name by removing the architecture. This is useful
        for poorly created packages which might include the architecture as an
        actual part of the name such as kernel modules which match a specific
        kernel version.

            salt -G role:nsd pkg.install gpfs.gplbin-2.6.32-279.31.1.el6.x86_64 normalize=False

        New in version 2014.7.0


    Returns a dict containing the new package names and versions::

        {'<package>': {'old': '<old-version>',
                       'new': '<new-version>'}}


[root@h209 ~]#
[root@h209 ~]# salt '*' sys.doc pkg
pkg.available_version:

This function is an alias of ``latest_version``.

    Return the latest version of the named package available for upgrade or
    installation. If more than one package name is specified, a dict of
    name/version pairs is returned.

    If the latest version of a given package is already installed, an empty
    string will be returned for that package.

    A specific repo can be requested using the ``fromrepo`` keyword argument,
    and the ``disableexcludes`` option is also supported.

    New in version 2014.7.0
        Support for the ``disableexcludes`` option

    CLI Example:

        salt '*' pkg.latest_version <package name>
        salt '*' pkg.latest_version <package name> fromrepo=epel-testing
        salt '*' pkg.latest_version <package name> disableexcludes=main
        salt '*' pkg.latest_version <package1> <package2> <package3> ...


pkg.clean_metadata:

    New in version 2014.1.0

    Cleans local yum metadata. Functionally identical to :mod:`refresh_db()
    <salt.modules.yumpkg.refresh_db>`.

    CLI Example:

        salt '*' pkg.clean_metadata


pkg.del_repo:

    Delete a repo from <basedir> (default basedir: all dirs in `reposdir` yum
    option).

    If the .repo file in which the repo exists does not contain any other repo
    configuration, the file itself will be deleted.

    CLI Examples:

        salt '*' pkg.del_repo myrepo
        salt '*' pkg.del_repo myrepo basedir=/path/to/dir
        salt '*' pkg.del_repo myrepo basedir=/path/to/dir,/path/to/another/dir


pkg.diff:

    Return a formatted diff between current files and original in a package.
    NOTE: this function includes all files (configuration and not), but does
    not work on binary content.

    :param path: Full path to the installed file
    :return: Difference string or raises and exception if examined file is binary.

    CLI example:

        salt '*' pkg.diff /etc/apache2/httpd.conf /etc/sudoers


pkg.download:

    New in version 2015.5.0

    Download packages to the local disk. Requires ``yumdownloader`` from
    ``yum-utils`` package.

    Note:

        ``yum-utils`` will already be installed on the minion if the package
        was installed from the Fedora / EPEL repositories.

    CLI example:

        salt '*' pkg.download httpd
        salt '*' pkg.download httpd postfix


pkg.file_dict:

    New in version 2014.1.0

    List the files that belong to a package, grouped by package. Not
    specifying any packages will return a list of *every* file on the system's
    rpm database (not generally recommended).

    CLI Examples:

        salt '*' pkg.file_list httpd
        salt '*' pkg.file_list httpd postfix
        salt '*' pkg.file_list


pkg.file_list:

    New in version 2014.1.0

    List the files that belong to a package. Not specifying any packages will
    return a list of *every* file on the system's rpm database (not generally
    recommended).

    CLI Examples:

        salt '*' pkg.file_list httpd
        salt '*' pkg.file_list httpd postfix
        salt '*' pkg.file_list


pkg.get_locked_packages:

This function is an alias of ``list_holds``.

    Changed in version 2016.3.0,2015.8.4,2015.5.10
        Function renamed from ``pkg.get_locked_pkgs`` to ``pkg.list_holds``.

    List information on locked packages

    Note:
        Requires the appropriate ``versionlock`` plugin package to be installed:

        - On RHEL 5: ``yum-versionlock``
        - On RHEL 6 & 7: ``yum-plugin-versionlock``
        - On Fedora: ``python-dnf-plugins-extras-versionlock``

    pattern : \w+(?:[.-][^-]+)*
        Regular expression used to match the package name

    full : True
        Show the full hold definition including version and epoch. Set to
        ``False`` to return just the name of the package(s) being held.


    CLI Example:

        salt '*' pkg.list_holds
        salt '*' pkg.list_holds full=False


pkg.get_repo:

    Display a repo from <basedir> (default basedir: all dirs in ``reposdir``
    yum option).

    CLI Examples:

        salt '*' pkg.get_repo myrepo
        salt '*' pkg.get_repo myrepo basedir=/path/to/dir
        salt '*' pkg.get_repo myrepo basedir=/path/to/dir,/path/to/another/dir


pkg.group_diff:

    New in version 2014.1.0
    Changed in version 2016.3.0,2015.8.4,2015.5.10
        Environment groups are now supported. The key names have been renamed,
        similar to the changes made in :py:func:`pkg.group_info
        <salt.modules.yumpkg.group_info>`.

    Lists which of a group's packages are installed and which are not
    installed

    CLI Example:

        salt '*' pkg.group_diff 'Perl Support'


pkg.group_info:

    New in version 2014.1.0
    Changed in version 2016.3.0,2015.8.4,2015.5.10
        The return data has changed. A new key ``type`` has been added to
        distinguish environment groups from package groups. Also, keys for the
        group name and group ID have been added. The ``mandatory packages``,
        ``optional packages``, and ``default packages`` keys have been renamed
        to ``mandatory``, ``optional``, and ``default`` for accuracy, as
        environment groups include other groups, and not packages. Finally,
        this function now properly identifies conditional packages.

    Lists packages belonging to a certain group

    name
        Name of the group to query

    expand : False
        If the specified group is an environment group, then the group will be
        expanded and the return data will include package names instead of
        group names.

        New in version 2016.3.0

    CLI Example:

        salt '*' pkg.group_info 'Perl Support'


pkg.group_install:

    New in version 2014.1.0

    Install the passed package group(s). This is basically a wrapper around
    :py:func:`pkg.install <salt.modules.yumpkg.install>`, which performs
    package group resolution for the user. This function is currently
    considered experimental, and should be expected to undergo changes.

    name
        Package group to install. To install more than one group, either use a
        comma-separated list or pass the value as a python list.

        CLI Examples:

            salt '*' pkg.group_install 'Group 1'
            salt '*' pkg.group_install 'Group 1,Group 2'
            salt '*' pkg.group_install '["Group 1", "Group 2"]'

    skip
        Packages that would normally be installed by the package group
        ("default" packages), which should not be installed. Can be passed
        either as a comma-separated list or a python list.

        CLI Examples:

            salt '*' pkg.group_install 'My Group' skip='foo,bar'
            salt '*' pkg.group_install 'My Group' skip='["foo", "bar"]'

    include
        Packages which are included in a group, which would not normally be
        installed by a ``yum groupinstall`` ("optional" packages). Note that
        this will not enforce group membership; if you include packages which
        are not members of the specified groups, they will still be installed.
        Can be passed either as a comma-separated list or a python list.

        CLI Examples:

            salt '*' pkg.group_install 'My Group' include='foo,bar'
            salt '*' pkg.group_install 'My Group' include='["foo", "bar"]'

    Note:

        Because this is essentially a wrapper around pkg.install, any argument
        which can be passed to pkg.install may also be included here, and it
        will be passed along wholesale.


pkg.group_list:

    New in version 2014.1.0

    Lists all groups known by yum on this system

    CLI Example:

        salt '*' pkg.group_list


pkg.groupinstall:

This function is an alias of ``group_install``.

    New in version 2014.1.0

    Install the passed package group(s). This is basically a wrapper around
    :py:func:`pkg.install <salt.modules.yumpkg.install>`, which performs
    package group resolution for the user. This function is currently
    considered experimental, and should be expected to undergo changes.

    name
        Package group to install. To install more than one group, either use a
        comma-separated list or pass the value as a python list.

        CLI Examples:

            salt '*' pkg.group_install 'Group 1'
            salt '*' pkg.group_install 'Group 1,Group 2'
            salt '*' pkg.group_install '["Group 1", "Group 2"]'

    skip
        Packages that would normally be installed by the package group
        ("default" packages), which should not be installed. Can be passed
        either as a comma-separated list or a python list.

        CLI Examples:

            salt '*' pkg.group_install 'My Group' skip='foo,bar'
            salt '*' pkg.group_install 'My Group' skip='["foo", "bar"]'

    include
        Packages which are included in a group, which would not normally be
        installed by a ``yum groupinstall`` ("optional" packages). Note that
        this will not enforce group membership; if you include packages which
        are not members of the specified groups, they will still be installed.
        Can be passed either as a comma-separated list or a python list.

        CLI Examples:

            salt '*' pkg.group_install 'My Group' include='foo,bar'
            salt '*' pkg.group_install 'My Group' include='["foo", "bar"]'

    Note:

        Because this is essentially a wrapper around pkg.install, any argument
        which can be passed to pkg.install may also be included here, and it
        will be passed along wholesale.


pkg.hold:

    New in version 2014.7.0

    Version-lock packages

    Note:
        Requires the appropriate ``versionlock`` plugin package to be installed:

        - On RHEL 5: ``yum-versionlock``
        - On RHEL 6 & 7: ``yum-plugin-versionlock``
        - On Fedora: ``python-dnf-plugins-extras-versionlock``


    name
        The name of the package to be held.

    Multiple Package Options:

    pkgs
        A list of packages to hold. Must be passed as a python list. The
        ``name`` parameter will be ignored if this option is passed.

    Returns a dict containing the changes.

    CLI Example:

        salt '*' pkg.hold <package name>
        salt '*' pkg.hold pkgs='["foo", "bar"]'


pkg.info_installed:

    New in version 2015.8.1

    Return the information of the named package(s), installed on the system.

    CLI example:

        salt '*' pkg.info_installed <package1>
        salt '*' pkg.info_installed <package1> <package2> <package3> ...


pkg.install:

    Changed in version 2015.8.12,2016.3.3,2016.11.0
        On minions running systemd>=205, `systemd-run(1)`_ is now used to
        isolate commands which modify installed packages from the
        ``salt-minion`` daemon's control group. This is done to keep systemd
        from killing any yum/dnf commands spawned by Salt when the
        ``salt-minion`` service is restarted. (see ``KillMode`` in the
        `systemd.kill(5)`_ manpage for more information). If desired, usage of
        `systemd-run(1)`_ can be suppressed by setting a :mod:`config option
        <salt.modules.config.get>` called ``systemd.scope``, with a value of
        ``False`` (no quotes).

    .. _`systemd-run(1)`: https://www.freedesktop.org/software/systemd/man/systemd-run.html
    .. _`systemd.kill(5)`: https://www.freedesktop.org/software/systemd/man/systemd.kill.html

    Install the passed package(s), add refresh=True to clean the yum database
    before package is installed.

    name
        The name of the package to be installed. Note that this parameter is
        ignored if either "pkgs" or "sources" is passed. Additionally, please
        note that this option can only be used to install packages from a
        software repository. To install a package file manually, use the
        "sources" option.

        32-bit packages can be installed on 64-bit systems by appending the
        architecture designation (``.i686``, ``.i586``, etc.) to the end of the
        package name.

        CLI Example:

            salt '*' pkg.install <package name>

    refresh
        Whether or not to update the yum database before executing.

    reinstall
        Specifying reinstall=True will use ``yum reinstall`` rather than
        ``yum install`` for requested packages that are already installed.

        If a version is specified with the requested package, then
        ``yum reinstall`` will only be used if the installed version
        matches the requested version.

        Works with ``sources`` when the package header of the source can be
        matched to the name and version of an installed package.

        New in version 2014.7.0

    skip_verify
        Skip the GPG verification check (e.g., ``--nogpgcheck``)

    downloadonly
        Only download the packages, do not install.

    version
        Install a specific version of the package, e.g. 1.2.3-4.el5. Ignored
        if "pkgs" or "sources" is passed.

    update_holds : False
        If ``True``, and this function would update the package version, any
        packages held using the yum/dnf "versionlock" plugin will be unheld so
        that they can be updated. Otherwise, if this function attempts to
        update a held package, the held package(s) will be skipped and an
        error will be raised.

        New in version 2016.11.0


    Repository Options:

    fromrepo
        Specify a package repository (or repositories) from which to install.
        (e.g., ``yum --disablerepo='*' --enablerepo='somerepo'``)

    enablerepo (ignored if ``fromrepo`` is specified)
        Specify a disabled package repository (or repositories) to enable.
        (e.g., ``yum --enablerepo='somerepo'``)

    disablerepo (ignored if ``fromrepo`` is specified)
        Specify an enabled package repository (or repositories) to disable.
        (e.g., ``yum --disablerepo='somerepo'``)

    disableexcludes
        Disable exclude from main, for a repo or for everything.
        (e.g., ``yum --disableexcludes='main'``)

        New in version 2014.7.0


    Multiple Package Installation Options:

    pkgs
        A list of packages to install from a software repository. Must be
        passed as a python list. A specific version number can be specified
        by using a single-element dict representing the package and its
        version.

        CLI Examples:

            salt '*' pkg.install pkgs='["foo", "bar"]'
            salt '*' pkg.install pkgs='["foo", {"bar": "1.2.3-4.el5"}]'

    sources
        A list of RPM packages to install. Must be passed as a list of dicts,
        with the keys being package names, and the values being the source URI
        or local path to the package.

        CLI Example:

            salt '*' pkg.install sources='[{"foo": "salt://foo.rpm"}, {"bar": "salt://bar.rpm"}]'

    normalize : True
        Normalize the package name by removing the architecture. This is useful
        for poorly created packages which might include the architecture as an
        actual part of the name such as kernel modules which match a specific
        kernel version.

            salt -G role:nsd pkg.install gpfs.gplbin-2.6.32-279.31.1.el6.x86_64 normalize=False

        New in version 2014.7.0


    Returns a dict containing the new package names and versions::

        {'<package>': {'old': '<old-version>',
                       'new': '<new-version>'}}


pkg.latest_version:

    Return the latest version of the named package available for upgrade or
    installation. If more than one package name is specified, a dict of
    name/version pairs is returned.

    If the latest version of a given package is already installed, an empty
    string will be returned for that package.

    A specific repo can be requested using the ``fromrepo`` keyword argument,
    and the ``disableexcludes`` option is also supported.

    New in version 2014.7.0
        Support for the ``disableexcludes`` option

    CLI Example:

        salt '*' pkg.latest_version <package name>
        salt '*' pkg.latest_version <package name> fromrepo=epel-testing
        salt '*' pkg.latest_version <package name> disableexcludes=main
        salt '*' pkg.latest_version <package1> <package2> <package3> ...


pkg.list_downloaded:

    New in version 2017.7.0

    List prefetched packages downloaded by Yum in the local disk.

    CLI example:

        salt '*' pkg.list_downloaded


pkg.list_holds:

    Changed in version 2016.3.0,2015.8.4,2015.5.10
        Function renamed from ``pkg.get_locked_pkgs`` to ``pkg.list_holds``.

    List information on locked packages

    Note:
        Requires the appropriate ``versionlock`` plugin package to be installed:

        - On RHEL 5: ``yum-versionlock``
        - On RHEL 6 & 7: ``yum-plugin-versionlock``
        - On Fedora: ``python-dnf-plugins-extras-versionlock``

    pattern : \w+(?:[.-][^-]+)*
        Regular expression used to match the package name

    full : True
        Show the full hold definition including version and epoch. Set to
        ``False`` to return just the name of the package(s) being held.


    CLI Example:

        salt '*' pkg.list_holds
        salt '*' pkg.list_holds full=False


pkg.list_installed_patches:

    New in version 2017.7.0

    List installed advisory patches on the system.

    CLI Examples:

        salt '*' pkg.list_installed_patches


pkg.list_patches:

    New in version 2017.7.0

    List all known advisory patches from available repos.

    refresh
        force a refresh if set to True.
        If set to False (default) it depends on yum if a refresh is
        executed.

    CLI Examples:

        salt '*' pkg.list_patches


pkg.list_pkgs:

    List the packages currently installed in a dict::

        {'<package_name>': '<version>'}

    CLI Example:

        salt '*' pkg.list_pkgs


pkg.list_repo_pkgs:

    New in version 2014.1.0
    Changed in version 2014.7.0
        All available versions of each package are now returned. This required
        a slight modification to the structure of the return dict. The return
        data shown below reflects the updated return dict structure. Note that
        packages which are version-locked using :py:mod:`pkg.hold
        <salt.modules.yumpkg.hold>` will only show the currently-installed
        version, as locking a package will make other versions appear
        unavailable to yum/dnf.
    Changed in version 2017.7.0
        By default, the versions for each package are no longer organized by
        repository. To get results organized by repository, use
        ``byrepo=True``.

    Returns all available packages. Optionally, package names (and name globs)
    can be passed and the results will be filtered to packages matching those
    names. This is recommended as it speeds up the function considerably.

    Warning:
        Running this function on RHEL/CentOS 6 and earlier will be more
        resource-intensive, as the version of yum that ships with older
        RHEL/CentOS has no yum subcommand for listing packages from a
        repository. Thus, a ``yum list installed`` and ``yum list available``
        are run, which generates a lot of output, which must then be analyzed
        to determine which package information to include in the return data.

    This function can be helpful in discovering the version or repo to specify
    in a :mod:`pkg.installed <salt.states.pkg.installed>` state.

    The return data will be a dictionary mapping package names to a list of
    version numbers, ordered from newest to oldest. If ``byrepo`` is set to
    ``True``, then the return dictionary will contain repository names at the
    top level, and each repository will map packages to lists of version
    numbers. For example:

        # With byrepo=False (default)
        {
            'bash': ['4.1.2-15.el6_5.2',
                     '4.1.2-15.el6_5.1',
                     '4.1.2-15.el6_4'],
            'kernel': ['2.6.32-431.29.2.el6',
                       '2.6.32-431.23.3.el6',
                       '2.6.32-431.20.5.el6',
                       '2.6.32-431.20.3.el6',
                       '2.6.32-431.17.1.el6',
                       '2.6.32-431.11.2.el6',
                       '2.6.32-431.5.1.el6',
                       '2.6.32-431.3.1.el6',
                       '2.6.32-431.1.2.0.1.el6',
                       '2.6.32-431.el6']
        }
        # With byrepo=True
        {
            'base': {
                'bash': ['4.1.2-15.el6_4'],
                'kernel': ['2.6.32-431.el6']
            },
            'updates': {
                'bash': ['4.1.2-15.el6_5.2', '4.1.2-15.el6_5.1'],
                'kernel': ['2.6.32-431.29.2.el6',
                           '2.6.32-431.23.3.el6',
                           '2.6.32-431.20.5.el6',
                           '2.6.32-431.20.3.el6',
                           '2.6.32-431.17.1.el6',
                           '2.6.32-431.11.2.el6',
                           '2.6.32-431.5.1.el6',
                           '2.6.32-431.3.1.el6',
                           '2.6.32-431.1.2.0.1.el6']
            }
        }

    fromrepo : None
        Only include results from the specified repo(s). Multiple repos can be
        specified, comma-separated.

    enablerepo (ignored if ``fromrepo`` is specified)
        Specify a disabled package repository (or repositories) to enable.
        (e.g., ``yum --enablerepo='somerepo'``)

        New in version 2017.7.0

    disablerepo (ignored if ``fromrepo`` is specified)
        Specify an enabled package repository (or repositories) to disable.
        (e.g., ``yum --disablerepo='somerepo'``)

        New in version 2017.7.0

    byrepo : False
        When ``True``, the return data for each package will be organized by
        repository.

        New in version 2017.7.0

    cacheonly : False
        When ``True``, the repo information will be retrieved from the cached
        repo metadata. This is equivalent to passing the ``-C`` option to
        yum/dnf.

        New in version 2017.7.0

    CLI Examples:

        salt '*' pkg.list_repo_pkgs
        salt '*' pkg.list_repo_pkgs foo bar baz
        salt '*' pkg.list_repo_pkgs 'samba4*' fromrepo=base,updates
        salt '*' pkg.list_repo_pkgs 'python2-*' byrepo=True


pkg.list_repos:

    Lists all repos in <basedir> (default: all dirs in `reposdir` yum option).

    CLI Example:

        salt '*' pkg.list_repos
        salt '*' pkg.list_repos basedir=/path/to/dir
        salt '*' pkg.list_repos basedir=/path/to/dir,/path/to/another/dir


pkg.list_updates:

This function is an alias of ``list_upgrades``.

    Check whether or not an upgrade is available for all packages

    The ``fromrepo``, ``enablerepo``, and ``disablerepo`` arguments are
    supported, as used in pkg states, and the ``disableexcludes`` option is
    also supported.

    New in version 2014.7.0
        Support for the ``disableexcludes`` option

    CLI Example:

        salt '*' pkg.list_upgrades


pkg.list_upgrades:

    Check whether or not an upgrade is available for all packages

    The ``fromrepo``, ``enablerepo``, and ``disablerepo`` arguments are
    supported, as used in pkg states, and the ``disableexcludes`` option is
    also supported.

    New in version 2014.7.0
        Support for the ``disableexcludes`` option

    CLI Example:

        salt '*' pkg.list_upgrades


pkg.mod_repo:

    Modify one or more values for a repo. If the repo does not exist, it will
    be created, so long as the following values are specified:

    repo
        name by which the yum refers to the repo
    name
        a human-readable name for the repo
    baseurl
        the URL for yum to reference
    mirrorlist
        the URL for yum to reference

    Key/Value pairs may also be removed from a repo's configuration by setting
    a key to a blank value. Bear in mind that a name cannot be deleted, and a
    baseurl can only be deleted if a mirrorlist is specified (or vice versa).

    CLI Examples:

        salt '*' pkg.mod_repo reponame enabled=1 gpgcheck=1
        salt '*' pkg.mod_repo reponame basedir=/path/to/dir enabled=1
        salt '*' pkg.mod_repo reponame baseurl= mirrorlist=http://host.com/


pkg.modified:

    List the modified files that belong to a package. Not specifying any packages
    will return a list of _all_ modified files on the system's RPM database.

    New in version 2015.5.0

    Filtering by flags (True or False):

    size
        Include only files where size changed.

    mode
        Include only files which file's mode has been changed.

    checksum
        Include only files which MD5 checksum has been changed.

    device
        Include only files which major and minor numbers has been changed.

    symlink
        Include only files which are symbolic link contents.

    owner
        Include only files where owner has been changed.

    group
        Include only files where group has been changed.

    time
        Include only files where modification time of the file has been
        changed.

    capabilities
        Include only files where capabilities differ or not. Note: supported
        only on newer RPM versions.

    CLI Examples:

        salt '*' pkg.modified
        salt '*' pkg.modified httpd
        salt '*' pkg.modified httpd postfix
        salt '*' pkg.modified httpd owner=True group=False


pkg.normalize_name:

    Strips the architecture from the specified package name, if necessary.
    Circumstances where this would be done include:

    * If the arch is 32 bit and the package name ends in a 32-bit arch.
    * If the arch matches the OS arch, or is ``noarch``.

    CLI Example:

        salt '*' pkg.normalize_name zsh.x86_64


pkg.owner:

    New in version 2014.7.0

    Return the name of the package that owns the file. Multiple file paths can
    be passed. Like :mod:`pkg.version <salt.modules.yumpkg.version`, if a
    single path is passed, a string will be returned, and if multiple paths are
    passed, a dictionary of file/package name pairs will be returned.

    If the file is not owned by a package, or is not present on the minion,
    then an empty string will be returned for that path.

    CLI Examples:

        salt '*' pkg.owner /usr/bin/apachectl
        salt '*' pkg.owner /usr/bin/apachectl /etc/httpd/conf/httpd.conf


pkg.purge:

    Changed in version 2015.8.12,2016.3.3,2016.11.0
        On minions running systemd>=205, `systemd-run(1)`_ is now used to
        isolate commands which modify installed packages from the
        ``salt-minion`` daemon's control group. This is done to keep systemd
        from killing any yum/dnf commands spawned by Salt when the
        ``salt-minion`` service is restarted. (see ``KillMode`` in the
        `systemd.kill(5)`_ manpage for more information). If desired, usage of
        `systemd-run(1)`_ can be suppressed by setting a :mod:`config option
        <salt.modules.config.get>` called ``systemd.scope``, with a value of
        ``False`` (no quotes).

    .. _`systemd-run(1)`: https://www.freedesktop.org/software/systemd/man/systemd-run.html
    .. _`systemd.kill(5)`: https://www.freedesktop.org/software/systemd/man/systemd.kill.html

    Package purges are not supported by yum, this function is identical to
    :mod:`pkg.remove <salt.modules.yumpkg.remove>`.

    name
        The name of the package to be purged


    Multiple Package Options:

    pkgs
        A list of packages to delete. Must be passed as a python list. The
        ``name`` parameter will be ignored if this option is passed.

    New in version 0.16.0


    Returns a dict containing the changes.

    CLI Example:

        salt '*' pkg.purge <package name>
        salt '*' pkg.purge <package1>,<package2>,<package3>
        salt '*' pkg.purge pkgs='["foo", "bar"]'


pkg.refresh_db:

    Check the yum repos for updated packages

    Returns:

    - ``True``: Updates are available
    - ``False``: An error occurred
    - ``None``: No updates are available

    repo
        Refresh just the specified repo

    disablerepo
        Do not refresh the specified repo

    enablerepo
        Refresh a disabled repo using this option

    branch
        Add the specified branch when refreshing

    disableexcludes
        Disable the excludes defined in your config files. Takes one of three
        options:
        - ``all`` - disable all excludes
        - ``main`` - disable excludes defined in [main] in yum.conf
        - ``repoid`` - disable excludes defined for that repo


    CLI Example:

        salt '*' pkg.refresh_db


pkg.remove:

    Changed in version 2015.8.12,2016.3.3,2016.11.0
        On minions running systemd>=205, `systemd-run(1)`_ is now used to
        isolate commands which modify installed packages from the
        ``salt-minion`` daemon's control group. This is done to keep systemd
        from killing any yum/dnf commands spawned by Salt when the
        ``salt-minion`` service is restarted. (see ``KillMode`` in the
        `systemd.kill(5)`_ manpage for more information). If desired, usage of
        `systemd-run(1)`_ can be suppressed by setting a :mod:`config option
        <salt.modules.config.get>` called ``systemd.scope``, with a value of
        ``False`` (no quotes).

    .. _`systemd-run(1)`: https://www.freedesktop.org/software/systemd/man/systemd-run.html
    .. _`systemd.kill(5)`: https://www.freedesktop.org/software/systemd/man/systemd.kill.html

    Remove packages

    name
        The name of the package to be removed


    Multiple Package Options:

    pkgs
        A list of packages to delete. Must be passed as a python list. The
        ``name`` parameter will be ignored if this option is passed.

    New in version 0.16.0


    Returns a dict containing the changes.

    CLI Example:

        salt '*' pkg.remove <package name>
        salt '*' pkg.remove <package1>,<package2>,<package3>
        salt '*' pkg.remove pkgs='["foo", "bar"]'


pkg.unhold:

    New in version 2014.7.0

    Remove version locks

    Note:
        Requires the appropriate ``versionlock`` plugin package to be installed:

        - On RHEL 5: ``yum-versionlock``
        - On RHEL 6 & 7: ``yum-plugin-versionlock``
        - On Fedora: ``python-dnf-plugins-extras-versionlock``


    name
        The name of the package to be unheld

    Multiple Package Options:

    pkgs
        A list of packages to unhold. Must be passed as a python list. The
        ``name`` parameter will be ignored if this option is passed.

    Returns a dict containing the changes.

    CLI Example:

        salt '*' pkg.unhold <package name>
        salt '*' pkg.unhold pkgs='["foo", "bar"]'


pkg.upgrade:

    Run a full system upgrade (a ``yum upgrade`` or ``dnf upgrade``), or
    upgrade specified packages. If the packages aren't installed, they will
    not be installed.

    Changed in version 2014.7.0
    Changed in version 2015.8.12,2016.3.3,2016.11.0
        On minions running systemd>=205, `systemd-run(1)`_ is now used to
        isolate commands which modify installed packages from the
        ``salt-minion`` daemon's control group. This is done to keep systemd
        from killing any yum/dnf commands spawned by Salt when the
        ``salt-minion`` service is restarted. (see ``KillMode`` in the
        `systemd.kill(5)`_ manpage for more information). If desired, usage of
        `systemd-run(1)`_ can be suppressed by setting a :mod:`config option
        <salt.modules.config.get>` called ``systemd.scope``, with a value of
        ``False`` (no quotes).

    .. _`systemd-run(1)`: https://www.freedesktop.org/software/systemd/man/systemd-run.html
    .. _`systemd.kill(5)`: https://www.freedesktop.org/software/systemd/man/systemd.kill.html

    Run a full system upgrade, a yum upgrade

    Returns a dictionary containing the changes:

        {'<package>':  {'old': '<old-version>',
                        'new': '<new-version>'}}


    CLI Example:

        salt '*' pkg.upgrade
        salt '*' pkg.upgrade name=openssl

    Repository Options:

    fromrepo
        Specify a package repository (or repositories) from which to install.
        (e.g., ``yum --disablerepo='*' --enablerepo='somerepo'``)

    enablerepo (ignored if ``fromrepo`` is specified)
        Specify a disabled package repository (or repositories) to enable.
        (e.g., ``yum --enablerepo='somerepo'``)

    disablerepo (ignored if ``fromrepo`` is specified)
        Specify an enabled package repository (or repositories) to disable.
        (e.g., ``yum --disablerepo='somerepo'``)

    disableexcludes
        Disable exclude from main, for a repo or for everything.
        (e.g., ``yum --disableexcludes='main'``)

        New in version 2014.7

    name
        The name of the package to be upgraded. Note that this parameter is
        ignored if "pkgs" is passed.

        32-bit packages can be upgraded on 64-bit systems by appending the
        architecture designation (``.i686``, ``.i586``, etc.) to the end of the
        package name.

        Warning: if you forget 'name=' and run pkg.upgrade openssl, ALL packages
        are upgraded. This will be addressed in next releases.

        CLI Example:

            salt '*' pkg.upgrade name=openssl

        New in version 2016.3.0

    pkgs
        A list of packages to upgrade from a software repository. Must be
        passed as a python list. A specific version number can be specified
        by using a single-element dict representing the package and its
        version. If the package was not already installed on the system,
        it will not be installed.

        CLI Examples:

            salt '*' pkg.upgrade pkgs='["foo", "bar"]'
            salt '*' pkg.upgrade pkgs='["foo", {"bar": "1.2.3-4.el5"}]'

        New in version 2016.3.0

    normalize : True
        Normalize the package name by removing the architecture. This is useful
        for poorly created packages which might include the architecture as an
        actual part of the name such as kernel modules which match a specific
        kernel version.

            salt -G role:nsd pkg.upgrade gpfs.gplbin-2.6.32-279.31.1.el6.x86_64 normalize=False

        New in version 2016.3.0

    Note:

        To add extra arguments to the ``yum upgrade`` command, pass them as key
        word arguments.  For arguments without assignments, pass ``True``

        salt '*' pkg.upgrade security=True exclude='kernel*'



pkg.upgrade_available:

    Check whether or not an upgrade is available for a given package

    CLI Example:

        salt '*' pkg.upgrade_available <package name>


pkg.verify:

    New in version 2014.1.0

    Runs an rpm -Va on a system, and returns the results in a dict

    Pass options to modify rpm verify behavior using the ``verify_options``
    keyword argument

    Files with an attribute of config, doc, ghost, license or readme in the
    package header can be ignored using the ``ignore_types`` keyword argument

    CLI Example:

        salt '*' pkg.verify
        salt '*' pkg.verify httpd
        salt '*' pkg.verify 'httpd postfix'
        salt '*' pkg.verify 'httpd postfix' ignore_types=['config','doc']
        salt '*' pkg.verify 'httpd postfix' verify_options=['nodeps','nosize']


pkg.version:

    Returns a string representing the package version or an empty string if not
    installed. If more than one package name is specified, a dict of
    name/version pairs is returned.

    CLI Example:

        salt '*' pkg.version <package name>
        salt '*' pkg.version <package1> <package2> <package3> ...


pkg.version_cmp:

    New in version 2015.5.4

    Do a cmp-style comparison on two packages. Return -1 if pkg1 < pkg2, 0 if
    pkg1 == pkg2, and 1 if pkg1 > pkg2. Return None if there was a problem
    making the comparison.

    ignore_epoch : False
        Set to ``True`` to ignore the epoch when comparing versions

        New in version 2015.8.10,2016.3.2

    CLI Example:

        salt '*' pkg.version_cmp '0.2-001' '0.2.0.1-002'


[root@h209 ~]#
[root@h209 ~]# salt '*' sys.doc network.interfaces
network.interfaces:

    Return a dictionary of information about all the interfaces on the minion

    CLI Example:

        salt '*' network.interfaces


[root@h209 ~]# salt '*' sys.doc disk.usage
disk.usage:

    Return usage information for volumes mounted on this minion

    CLI Example:

        salt '*' disk.usage


[root@h209 ~]#
~~~

其后可以加模块，也可以加模块方法

从文档内容来看，质量还是相当高的，用法和实例都有

在离线的情况下对于了解一个陌生的模块方法，回顾一个模块的使用都能启到很好的提示作用


---

# 总结

通过 salt 来执行远程命令，可以逐步感受到 salt 的强大与易用

* TOC
{:toc}


---


[saltstack]:https://saltstack.com/
[saltstack_remotex]:https://docs.saltstack.com/en/getstarted/fundamentals/remotex.html
