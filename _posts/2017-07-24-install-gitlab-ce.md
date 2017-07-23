---
layout:  post
title:  安装 GitLab CE
author:  wilmosfang
tags: gitlab  git
categories:  gitlab 
wc: 6954 36473 347764
excerpt:  Centos7 中安装 gitlab ce
comments: true
---


# 前言


DevOps 理念落实得最为彻底的一类案例就是 CI/CD(持续集成/持续交付) 系统

CI/CD(持续集成/持续交付) 系统的一个关键环节就是版本控制，因为它是多是工作流的起点

版本控制软件有很多种，比较熟知的开源版本控制软件有 **[CVS][cvs]** ，**[SVN][svn]** 和 **[Git][git]** ，从目前使用情况来看最受欢迎的开源版本控制系统还是 **[Git][git]**

单单看 **[Git][git]** 所专注的版本控制功能，其强大与高效鲜有软件可以与其比拟，但是 **[Git][git]** 没有友好的管理界面和配备服务，大型项目管理的过程中也缺少权限管理的功能

于是世面上有各种基于 **[Git][git]** 的集成软件，**[GitLab][gitlab]** 就是其中优秀的一款

> **Tip:**  当前最新版本为 **9.4.0**

**[GitLab][gitlab]** 除了具备基本的版本控制能力外，还有内建的 CI/CD 功能，GitLab Pages(类似于 github pages，可以用于写 wiki，或其它帮助文档)，管理 issue，基本的 review 功能，时间追踪等功能

这些功能对于一个自动化的运维环境来讲，可以非常明显地提升工作效率

相对于基础的社区版，企业版和企业增强版还提供很多附加的功能，详细可以参考 **[版本对比][products]**　

这里就如何快速搭建 gitlab-ce 给出一个过程参考

其它环境下的详细安装过程可以参考 **[GitLab 的安装][gitlab_installation]**

---

# 概要

* TOC
{:toc}



---


## 系统环境


~~~
[root@much ~]# hostnamectl 
   Static hostname: much
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 33dc28f7e76c4903ad9b603b77e29a7c
           Boot ID: 16c8f52b10f2442f85308cce86bf08f7
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-514.21.1.el7.x86_64
      Architecture: x86-64
[root@much ~]# ip  a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:c2:66:f7 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 86055sec preferred_lft 86055sec
    inet6 fe80::2bb7:5b3:9584:d8eb/64 scope link 
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:03:d0:2d brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.203/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe03:d02d/64 scope link 
       valid_lft forever preferred_lft forever
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
[root@much ~]# uname  -a 
Linux much 3.10.0-514.21.1.el7.x86_64 #1 SMP Thu May 25 17:04:51 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
[root@much ~]# 
~~~

---

## 安装并且配置必要的依赖包

~~~
[root@much ~]# yum install curl policycoreutils openssh-server openssh-clients
Loaded plugins: fastestmirror, langpacks
Repodata is over 2 weeks old. Install yum-cron? Or run: yum makecache fast
base                                                                                                                                                                 | 3.6 kB  00:00:00     
c7-media                                                                                                                                                             | 3.6 kB  00:00:00     
extras                                                                                                                                                               | 3.4 kB  00:00:00     
updates                                                                                                                                                              | 3.4 kB  00:00:00     
(1/2): extras/7/x86_64/primary_db                                                                                                                                    | 191 kB  00:00:00     
(2/2): updates/7/x86_64/primary_db                                                                                                                                   | 7.8 MB  00:00:00     
Determining fastest mirrors
 * base: mirrors.tuna.tsinghua.edu.cn
 * c7-media: 
 * extras: centos.ustc.edu.cn
 * updates: centos.ustc.edu.cn
Package curl-7.29.0-35.el7.centos.x86_64 already installed and latest version
Package policycoreutils-2.5-11.el7_3.x86_64 already installed and latest version
Package openssh-server-6.6.1p1-35.el7_3.x86_64 already installed and latest version
Package openssh-clients-6.6.1p1-35.el7_3.x86_64 already installed and latest version
Nothing to do
[root@much ~]# systemctl enable sshd
[root@much ~]# systemctl start sshd
[root@much ~]# yum install postfix
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirrors.tuna.tsinghua.edu.cn
 * c7-media: 
 * extras: centos.ustc.edu.cn
 * updates: centos.ustc.edu.cn
Package 2:postfix-2.10.1-6.el7.x86_64 already installed and latest version
Nothing to do
[root@much ~]# systemctl enable postfix
[root@much ~]# systemctl start postfix
[root@much ~]# firewall-cmd --permanent --add-service=http
success
[root@much ~]# systemctl reload firewalld
[root@much ~]# 
~~~

gitlab-ce 对 **`curl policycoreutils openssh-server openssh-clients postfix`** 这些服务有依赖，需要提前安装和开启

防火墙要打开 http 的访问，否则无法对外提供服务



---

## 安装 Gitlab 服务包

~~~
[root@much ~]# yum list all | grep gitlab
[root@much ~]# curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash
Detected operating system as centos/7.
Checking for curl...
Detected curl...
Downloading repository file: https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/config_file.repo?os=centos&dist=7&source=script
done.
Installing pygpgme to verify GPG signatures...
Loaded plugins: fastestmirror, langpacks
gitlab_gitlab-ce-source/signature                                                                                                                                    |  836 B  00:00:00     
Retrieving key from https://packages.gitlab.com/gitlab/gitlab-ce/gpgkey
Importing GPG key 0xE15E78F4:
 Userid     : "GitLab B.V. (package repository signing key) <packages@gitlab.com>"
 Fingerprint: 1a4c 919d b987 d435 9396 38b9 1421 9a96 e15e 78f4
 From       : https://packages.gitlab.com/gitlab/gitlab-ce/gpgkey
gitlab_gitlab-ce-source/signature                                                                                                                                    |  951 B  00:00:00 !!! 
gitlab_gitlab-ce-source/primary                                                                                                                                      |  175 B  00:00:03     
Loading mirror speeds from cached hostfile
 * base: mirrors.tuna.tsinghua.edu.cn
 * c7-media: 
 * extras: centos.ustc.edu.cn
 * updates: centos.ustc.edu.cn
Package pygpgme-0.3-9.el7.x86_64 already installed and latest version
Nothing to do
Installing yum-utils...
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirrors.tuna.tsinghua.edu.cn
 * c7-media: 
 * extras: centos.ustc.edu.cn
 * updates: centos.ustc.edu.cn
Package yum-utils-1.1.31-40.el7.noarch already installed and latest version
Nothing to do
Generating yum cache for gitlab_gitlab-ce...
Importing GPG key 0xE15E78F4:
 Userid     : "GitLab B.V. (package repository signing key) <packages@gitlab.com>"
 Fingerprint: 1a4c 919d b987 d435 9396 38b9 1421 9a96 e15e 78f4
 From       : https://packages.gitlab.com/gitlab/gitlab-ce/gpgkey

The repository is setup! You can now install packages.
[root@much ~]# echo $?
0
[root@much ~]# yum list all | grep gitlab
gitlab-ce.x86_64                           9.4.0-ce.0.el7              gitlab_gitlab-ce
[root@much ~]#
[root@much ~]# yum install gitlab-ce
Loaded plugins: fastestmirror, langpacks
gitlab_gitlab-ce/x86_64/signature                                                                                                                                    |  836 B  00:00:00     
gitlab_gitlab-ce/x86_64/signature                                                                                                                                    | 1.0 kB  00:00:00 !!! 
gitlab_gitlab-ce-source/signature                                                                                                                                    |  836 B  00:00:00     
gitlab_gitlab-ce-source/signature                                                                                                                                    |  951 B  00:00:00 !!! 
Loading mirror speeds from cached hostfile
 * base: mirrors.tuna.tsinghua.edu.cn
 * c7-media: 
 * extras: centos.ustc.edu.cn
 * updates: centos.ustc.edu.cn
Resolving Dependencies
--> Running transaction check
---> Package gitlab-ce.x86_64 0:9.4.0-ce.0.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

============================================================================================================================================================================================
 Package                                    Arch                                    Version                                         Repository                                         Size
============================================================================================================================================================================================
Installing:
 gitlab-ce                                  x86_64                                  9.4.0-ce.0.el7                                  gitlab_gitlab-ce                                  340 M

Transaction Summary
============================================================================================================================================================================================
Install  1 Package

Total download size: 340 M
Installed size: 1.0 G
Is this ok [y/d/N]: y
Downloading packages:
No Presto metadata available for gitlab_gitlab-ce
gitlab-ce-9.4.0-ce.0.el7.x86_64.rpm                                                                                                                                  | 340 MB  00:44:02     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : gitlab-ce-9.4.0-ce.0.el7.x86_64                                                                                                                                          1/1 


       *.                  *.
      ***                 ***
     *****               *****
    .******             *******
    ********            ********
   ,,,,,,,,,***********,,,,,,,,,
  ,,,,,,,,,,,*********,,,,,,,,,,,
  .,,,,,,,,,,,*******,,,,,,,,,,,,
      ,,,,,,,,,*****,,,,,,,,,.
         ,,,,,,,****,,,,,,
            .,,,***,,,,
                ,*,.

     _______ __  __          __
    / ____(_) /_/ /   ____ _/ /_
   / / __/ / __/ /   / __ `/ __ \
  / /_/ / / /_/ /___/ /_/ / /_/ /
  \____/_/\__/_____/\__,_/_.___/


gitlab: Thank you for installing GitLab!
gitlab: To configure and start GitLab, RUN THE FOLLOWING COMMAND:

sudo gitlab-ctl reconfigure

gitlab: GitLab should be reachable at http://much
gitlab: Otherwise configure GitLab for your system by editing /etc/gitlab/gitlab.rb file
gitlab: And running reconfigure again.
gitlab: 
gitlab: For a comprehensive list of configuration options please see the Omnibus GitLab readme
gitlab: https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/README.md
gitlab: 
It looks like GitLab has not been configured yet; skipping the upgrade script.
  Verifying  : gitlab-ce-9.4.0-ce.0.el7.x86_64                                                                                                                                          1/1 

Installed:
  gitlab-ce.x86_64 0:9.4.0-ce.0.el7                                                                                                                                                         

Complete!
[root@much ~]# echo $?
0
[root@much ~]#
~~~


---

## 启动 GitLab ce

~~~
[root@much ~]# gitlab-ctl status
[root@much ~]# gitlab-ctl reconfigure
Starting Chef Client, version 12.12.15
resolving cookbooks for run list: ["gitlab"]
Synchronizing Cookbooks:
  - package (0.0.0)
  - gitlab (0.0.1)
  - runit (0.14.2)
  - registry (0.1.0)
Installing Cookbook Gems:
Compiling Cookbooks...
Recipe: gitlab::default
  * directory[/etc/gitlab] action create
    - change mode from '0755' to '0775'
    - restore selinux security context
/sbin/init: unrecognized option '--version'
  -.mount                                                                                  loaded active mounted   /
  Converging 465 resources
  * directory[/etc/gitlab] action create (up to date)
  * directory[Create /var/opt/gitlab] action create
    - create new directory /var/opt/gitlab
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * directory[/opt/gitlab/embedded/etc] action create
    - create new directory /opt/gitlab/embedded/etc
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * template[/opt/gitlab/embedded/etc/gitconfig] action create
    - create new file /opt/gitlab/embedded/etc/gitconfig
    - update content in file /opt/gitlab/embedded/etc/gitconfig from none to 5fe039
    --- /opt/gitlab/embedded/etc/gitconfig	2017-07-24 00:12:56.993000000 +0800
    +++ /opt/gitlab/embedded/etc/.chef-gitconfig20170724-7201-pdd52n	2017-07-24 00:12:56.993000000 +0800
    @@ -1 +1,10 @@
    +[pack]
    +  threads = 1
    +[receive]
    +  fsckObjects = true
    +[repack]
    +  writeBitmaps = true
    +[transfer]
    +  hideRefs=^refs/tmp/
    +hideRefs=^refs/keep-around/
    - change mode from '' to '0755'
    - restore selinux security context
Recipe: gitlab::web-server
  * group[Webserver user and group] action create
    - create group gitlab-www
  * user[Webserver user and group] action create
    - create user gitlab-www
Recipe: gitlab::users
  * directory[/var/opt/gitlab] action create (up to date)
  * group[GitLab user and group] action create
    - create group git
  * user[GitLab user and group] action create
    - create user git
  * template[/var/opt/gitlab/.gitconfig] action create
    - create new file /var/opt/gitlab/.gitconfig
    - update content in file /var/opt/gitlab/.gitconfig from none to 973154
    --- /var/opt/gitlab/.gitconfig	2017-07-24 00:12:57.192000000 +0800
    +++ /var/opt/gitlab/.chef-.gitconfig20170724-7201-161mgmq	2017-07-24 00:12:57.192000000 +0800
    @@ -1 +1,12 @@
    +# This file is managed by gitlab-ctl. Manual changes will be
    +# erased! To change the contents below, edit /etc/gitlab/gitlab.rb
    +# and run `sudo gitlab-ctl reconfigure`.
    +
    +[user]
    +        name = GitLab
    +        email = gitlab@much
    +[core]
    +        autocrlf = input
    +[gc]
    +        auto = 0
    - change mode from '' to '0644'
    - change owner from '' to 'git'
    - change group from '' to 'git'
    - restore selinux security context
Recipe: gitlab::gitlab-shell
  * ruby_block[directory resource: /var/opt/gitlab/git-data] action run
    - execute the ruby block directory resource: /var/opt/gitlab/git-data
  * ruby_block[directory resource: /var/opt/gitlab/git-data/repositories] action run
    - execute the ruby block directory resource: /var/opt/gitlab/git-data/repositories
  * ruby_block[directory resource: /var/opt/gitlab/.ssh] action run
    - execute the ruby block directory resource: /var/opt/gitlab/.ssh
  * directory[/var/log/gitlab/gitlab-shell/] action create
    - create new directory /var/log/gitlab/gitlab-shell/
    - change mode from '' to '0700'
    - change owner from '' to 'git'
    - restore selinux security context
  * directory[/var/opt/gitlab/gitlab-shell] action create
    - create new directory /var/opt/gitlab/gitlab-shell
    - change mode from '' to '0700'
    - change owner from '' to 'git'
    - restore selinux security context
  * templatesymlink[Create a config.yml and create a symlink to Rails root] action create
    * template[/var/opt/gitlab/gitlab-shell/config.yml] action create
      - create new file /var/opt/gitlab/gitlab-shell/config.yml
      - update content in file /var/opt/gitlab/gitlab-shell/config.yml from none to cce2bf
      --- /var/opt/gitlab/gitlab-shell/config.yml	2017-07-24 00:12:57.952000000 +0800
      +++ /var/opt/gitlab/gitlab-shell/.chef-config.yml20170724-7201-18rakrc	2017-07-24 00:12:57.952000000 +0800
      @@ -1 +1,44 @@
      +# This file is managed by gitlab-ctl. Manual changes will be
      +# erased! To change the contents below, edit /etc/gitlab/gitlab.rb
      +# and run `sudo gitlab-ctl reconfigure`.
      +
      +# GitLab user. git by default
      +user: git
      +
      +# Url to gitlab instance. Used for api calls. Should end with a slash.
      +gitlab_url: "http://127.0.0.1:8080"
      +
      +http_settings:
      +  
      +#  user: someone
      +#  password: somepass
      +#  ca_file: /etc/ssl/cert.pem
      +#  ca_path: /etc/pki/tls/certs
      +#  self_signed_cert: false
      +
      +# File used as authorized_keys for gitlab user
      +auth_file: "/var/opt/gitlab/.ssh/authorized_keys"
      +
      +# Redis settings used for pushing commit notices to gitlab
      +redis:
      +  bin: /opt/gitlab/embedded/bin/redis-cli
      +  host: 127.0.0.1
      +  port: 
      +  socket: /var/opt/gitlab/redis/redis.socket
      +  database: 
      +  namespace: resque:gitlab
      +
      +# Log file.
      +# Default is gitlab-shell.log in the root directory.
      +log_file: "/var/log/gitlab/gitlab-shell/gitlab-shell.log"
      +
      +# Log level. INFO by default
      +log_level: 
      +
      +# Audit usernames.
      +# Set to true to see real usernames in the logs instead of key ids, which is easier to follow, but
      +# incurs an extra API call on every gitlab-shell command.
      +audit_usernames: 
      +
      +
      - restore selinux security context
    * link[Link /opt/gitlab/embedded/service/gitlab-shell/config.yml to /var/opt/gitlab/gitlab-shell/config.yml] action create
      - create symlink at /opt/gitlab/embedded/service/gitlab-shell/config.yml to /var/opt/gitlab/gitlab-shell/config.yml
  
  * link[/opt/gitlab/embedded/service/gitlab-shell/.gitlab_shell_secret] action create
    - create symlink at /opt/gitlab/embedded/service/gitlab-shell/.gitlab_shell_secret to /opt/gitlab/embedded/service/gitlab-rails/.gitlab_shell_secret
  * execute[/opt/gitlab/embedded/service/gitlab-shell/bin/gitlab-keys check-permissions] action run
    - execute /opt/gitlab/embedded/service/gitlab-shell/bin/gitlab-keys check-permissions
  * bash[Set proper security context on ssh files for selinux] action run
    - execute "bash"  "/tmp/chef-script20170724-7201-1q92aa0"
Recipe: gitlab::gitlab-rails
  * directory[/var/log/gitlab] action create
    - change owner from 'root' to 'git'
    - restore selinux security context
  * ruby_block[directory resource: /var/opt/gitlab/gitlab-rails/shared] action run
    - execute the ruby block directory resource: /var/opt/gitlab/gitlab-rails/shared
  * ruby_block[directory resource: /var/opt/gitlab/gitlab-rails/shared/artifacts] action run
    - execute the ruby block directory resource: /var/opt/gitlab/gitlab-rails/shared/artifacts
  * ruby_block[directory resource: /var/opt/gitlab/gitlab-rails/shared/lfs-objects] action run
    - execute the ruby block directory resource: /var/opt/gitlab/gitlab-rails/shared/lfs-objects
  * ruby_block[directory resource: /var/opt/gitlab/gitlab-rails/uploads] action run
    - execute the ruby block directory resource: /var/opt/gitlab/gitlab-rails/uploads
  * ruby_block[directory resource: /var/opt/gitlab/gitlab-ci/builds] action run
    - execute the ruby block directory resource: /var/opt/gitlab/gitlab-ci/builds
  * ruby_block[directory resource: /var/opt/gitlab/gitlab-rails/shared/pages] action run
    - execute the ruby block directory resource: /var/opt/gitlab/gitlab-rails/shared/pages
  * directory[create /var/opt/gitlab/gitlab-rails/etc] action create
    - create new directory /var/opt/gitlab/gitlab-rails/etc
    - change mode from '' to '0700'
    - change owner from '' to 'git'
    - restore selinux security context
  * directory[create /opt/gitlab/etc/gitlab-rails] action create
    - create new directory /opt/gitlab/etc/gitlab-rails
    - change mode from '' to '0700'
    - change owner from '' to 'git'
    - restore selinux security context
  * directory[create /var/opt/gitlab/gitlab-rails/working] action create
    - create new directory /var/opt/gitlab/gitlab-rails/working
    - change mode from '' to '0700'
    - change owner from '' to 'git'
    - restore selinux security context
  * directory[create /var/opt/gitlab/gitlab-rails/tmp] action create
    - create new directory /var/opt/gitlab/gitlab-rails/tmp
    - change mode from '' to '0700'
    - change owner from '' to 'git'
    - restore selinux security context
  * directory[create /var/opt/gitlab/gitlab-rails/upgrade-status] action create
    - create new directory /var/opt/gitlab/gitlab-rails/upgrade-status
    - change mode from '' to '0700'
    - change owner from '' to 'git'
    - restore selinux security context
  * directory[create /var/log/gitlab/gitlab-rails] action create
    - create new directory /var/log/gitlab/gitlab-rails
    - change mode from '' to '0700'
    - change owner from '' to 'git'
    - restore selinux security context
  * directory[/var/opt/gitlab/backups] action create
    - create new directory /var/opt/gitlab/backups
    - change mode from '' to '0700'
    - change owner from '' to 'git'
    - restore selinux security context
  * directory[/var/opt/gitlab/gitlab-rails] action create
    - change owner from 'root' to 'git'
    - restore selinux security context
  * directory[/var/opt/gitlab/gitlab-ci] action create
    - change owner from 'root' to 'git'
    - restore selinux security context
  * file[/var/opt/gitlab/gitlab-rails/etc/gitlab-registry.key] action create (skipped due to only_if)
  * template[/opt/gitlab/etc/gitlab-rails/gitlab-rails-rc] action create
    - create new file /opt/gitlab/etc/gitlab-rails/gitlab-rails-rc
    - update content in file /opt/gitlab/etc/gitlab-rails/gitlab-rails-rc from none to 15c7d9
    --- /opt/gitlab/etc/gitlab-rails/gitlab-rails-rc	2017-07-24 00:12:59.564000000 +0800
    +++ /opt/gitlab/etc/gitlab-rails/.chef-gitlab-rails-rc20170724-7201-cyydm3	2017-07-24 00:12:59.564000000 +0800
    @@ -1 +1,2 @@
    +gitlab_user='git'
    - restore selinux security context
  * file[/opt/gitlab/embedded/service/gitlab-rails/.secret] action delete (up to date)
  * file[/var/opt/gitlab/gitlab-rails/etc/secret] action delete (up to date)
  * templatesymlink[Create a database.yml and create a symlink to Rails root] action create
    * template[/var/opt/gitlab/gitlab-rails/etc/database.yml] action create
      - create new file /var/opt/gitlab/gitlab-rails/etc/database.yml
      - update content in file /var/opt/gitlab/gitlab-rails/etc/database.yml from none to f96ae4
      --- /var/opt/gitlab/gitlab-rails/etc/database.yml	2017-07-24 00:12:59.594000000 +0800
      +++ /var/opt/gitlab/gitlab-rails/etc/.chef-database.yml20170724-7201-1imtzor	2017-07-24 00:12:59.594000000 +0800
      @@ -1 +1,22 @@
      +# This file is managed by gitlab-ctl. Manual changes will be
      +# erased! To change the contents below, edit /etc/gitlab/gitlab.rb
      +# and run `sudo gitlab-ctl reconfigure`.
      +
      +production:
      +  adapter: postgresql
      +  encoding: unicode
      +  collation: 
      +  database: gitlabhq_production
      +  pool: 10
      +  username: 'gitlab'
      +  password: 
      +  host: '/var/opt/gitlab/postgresql'
      +  port: 5432
      +  socket: 
      +  sslmode: 
      +  sslrootcert: 
      +  sslca: 
      +  load_balancing: {"hosts":[]}
      +  prepared_statements: true
      +  statements_limit: 1000
      - change mode from '' to '0644'
      - change owner from '' to 'root'
      - change group from '' to 'root'
      - restore selinux security context
    * link[Link /opt/gitlab/embedded/service/gitlab-rails/config/database.yml to /var/opt/gitlab/gitlab-rails/etc/database.yml] action create
      - create symlink at /opt/gitlab/embedded/service/gitlab-rails/config/database.yml to /var/opt/gitlab/gitlab-rails/etc/database.yml
  
  * templatesymlink[Create a secrets.yml and create a symlink to Rails root] action create
    * template[/var/opt/gitlab/gitlab-rails/etc/secrets.yml] action create
      - create new file /var/opt/gitlab/gitlab-rails/etc/secrets.yml
      - update content in file /var/opt/gitlab/gitlab-rails/etc/secrets.yml from none to b7ccd5
      - suppressed sensitive resource
      - change mode from '' to '0644'
      - change owner from '' to 'root'
      - change group from '' to 'root'
      - restore selinux security context
    * link[Link /opt/gitlab/embedded/service/gitlab-rails/config/secrets.yml to /var/opt/gitlab/gitlab-rails/etc/secrets.yml] action create
      - create symlink at /opt/gitlab/embedded/service/gitlab-rails/config/secrets.yml to /var/opt/gitlab/gitlab-rails/etc/secrets.yml
  
  * templatesymlink[Create a resque.yml and create a symlink to Rails root] action create
    * template[/var/opt/gitlab/gitlab-rails/etc/resque.yml] action create
      - create new file /var/opt/gitlab/gitlab-rails/etc/resque.yml
      - update content in file /var/opt/gitlab/gitlab-rails/etc/resque.yml from none to ec4232
      --- /var/opt/gitlab/gitlab-rails/etc/resque.yml	2017-07-24 00:12:59.663000000 +0800
      +++ /var/opt/gitlab/gitlab-rails/etc/.chef-resque.yml20170724-7201-6xk1z2	2017-07-24 00:12:59.663000000 +0800
      @@ -1 +1,3 @@
      +production:
      +  url: unix:/var/opt/gitlab/redis/redis.socket
      - change mode from '' to '0644'
      - change owner from '' to 'root'
      - change group from '' to 'root'
      - restore selinux security context
    * link[Link /opt/gitlab/embedded/service/gitlab-rails/config/resque.yml to /var/opt/gitlab/gitlab-rails/etc/resque.yml] action create
      - create symlink at /opt/gitlab/embedded/service/gitlab-rails/config/resque.yml to /var/opt/gitlab/gitlab-rails/etc/resque.yml
  
  * templatesymlink[Create a aws.yml and create a symlink to Rails root] action delete
    * template[/var/opt/gitlab/gitlab-rails/etc/aws.yml] action delete (up to date)
    * link[Link /opt/gitlab/embedded/service/gitlab-rails/config/aws.yml to /var/opt/gitlab/gitlab-rails/etc/aws.yml] action delete (up to date)
     (up to date)
  * templatesymlink[Create a smtp_settings.rb and create a symlink to Rails root] action delete
    * template[/var/opt/gitlab/gitlab-rails/etc/smtp_settings.rb] action delete (up to date)
    * link[Link /opt/gitlab/embedded/service/gitlab-rails/config/initializers/smtp_settings.rb to /var/opt/gitlab/gitlab-rails/etc/smtp_settings.rb] action delete (up to date)
     (up to date)
  * templatesymlink[Create a gitlab.yml and create a symlink to Rails root] action create
    * template[/var/opt/gitlab/gitlab-rails/etc/gitlab.yml] action create
      - create new file /var/opt/gitlab/gitlab-rails/etc/gitlab.yml
      - update content in file /var/opt/gitlab/gitlab-rails/etc/gitlab.yml from none to 1666f1
      --- /var/opt/gitlab/gitlab-rails/etc/gitlab.yml	2017-07-24 00:12:59.706000000 +0800
      +++ /var/opt/gitlab/gitlab-rails/etc/.chef-gitlab.yml20170724-7201-1a7q6sx	2017-07-24 00:12:59.704000000 +0800
      @@ -1 +1,466 @@
      +# This file is managed by gitlab-ctl. Manual changes will be
      +# erased! To change the contents below, edit /etc/gitlab/gitlab.rb
      +# and run `sudo gitlab-ctl reconfigure`.
      +
      +production: &base
      +  #
      +  # 1. GitLab app settings
      +  # ==========================
      +
      +  ## GitLab settings
      +  gitlab:
      +    ## Web server settings (note: host is the FQDN, do not include http://)
      +    host: much
      +    port: 80
      +    https: false
      +
      +    # Uncommment this line below if your ssh host is different from HTTP/HTTPS one
      +    # (you'd obviously need to replace ssh.host_example.com with your own host).
      +    # Otherwise, ssh host will be set to the `host:` value above
      +    ssh_host: 
      +
      +    # WARNING: See config/application.rb under "Relative url support" for the list of
      +    # other files that need to be changed for relative url support
      +    relative_url_root: 
      +
      +    # Trusted Proxies
      +    # Customize if you have GitLab behind a reverse proxy which is running on a different machine.
      +    # Add the IP address for your reverse proxy to the list, otherwise users will appear signed in from that address.
      +    trusted_proxies:
      +
      +    # Uncomment and customize if you can't use the default user to run GitLab (default: 'git')
      +    user: git
      +
      +    ## Date & Time settings
      +    time_zone: 
      +
      +    ## Email settings
      +    # Uncomment and set to false if you need to disable email sending from GitLab (default: true)
      +    email_enabled: 
      +    # Email address used in the "From" field in mails sent by GitLab
      +    email_from: gitlab@much
      +    email_display_name: 
      +    email_reply_to: 
      +    email_subject_suffix: 
      +
      +    # Email server smtp settings are in [a separate file](initializers/smtp_settings.rb.sample).
      +
      +    ## User settings
      +    default_can_create_group:   # default: true
      +    username_changing_enabled:  # default: true - User can change her username/namespace
      +    ## Default theme
      +    ##   1 - Graphite
      +    ##   2 - Charcoal
      +    ##   3 - Green
      +    ##   4 - Gray
      +    ##   5 - Violet
      +    ##   6 - Blue
      +    default_theme:  # default: 2
      +
      +    ## Automatic issue closing
      +    # If a commit message matches this regular expression, all issues referenced from the matched text will be closed.
      +    # This happens when the commit is pushed or merged into the default branch of a project.
      +    # When not specified the default issue_closing_pattern as specified below will be used.
      +    # Tip: you can test your closing pattern at http://rubular.com
      +    issue_closing_pattern: 
      +
      +    ## Default project features settings
      +    default_projects_features:
      +      issues: 
      +      merge_requests: 
      +      wiki: 
      +      snippets: 
      +      builds: 
      +      container_registry: 
      +
      +    ## Webhook settings
      +    # Number of seconds to wait for HTTP response after sending webhook HTTP POST request (default: 10)
      +    webhook_timeout: 
      +
      +    ## Repository downloads directory
      +    # When a user clicks e.g. 'Download zip' on a project, a temporary zip file is created in the following directory.
      +    # The default is 'tmp/repositories' relative to the root of the Rails app.
      +    repository_downloads_path: 
      +
      +    usage_ping_enabled: 
      +
      +  ## Reply by email
      +  # Allow users to comment on issues and merge requests by replying to notification emails.
      +  # For documentation on how to set this up, see https://docs.gitlab.com/ce/administration/reply_by_email.html
      +  incoming_email:
      +    enabled: false
      +
      +    # The email address including the `%{key}` placeholder that will be replaced to reference the item being replied to.
      +    # The placeholder can be omitted but if present, it must appear in the "user" part of the address (before the `@`).
      +    address: 
      +
      +    # Email account username
      +    # With third party providers, this is usually the full email address.
      +    # With self-hosted email servers, this is usually the user part of the email address.
      +    user: 
      +    # Email account password
      +    password: 
      +
      +    # IMAP server host
      +    host: 
      +    # IMAP server port
      +    port: 
      +    # Whether the IMAP server uses SSL
      +    ssl: 
      +    # Whether the IMAP server uses StartTLS
      +    start_tls: 
      +
      +    # The mailbox where incoming mail will end up. Usually "inbox".
      +    mailbox: 'inbox'
      +    # The IDLE command timeout.
      +    idle_timeout: 
      +
      +  ## Build Artifacts
      +  artifacts:
      +    enabled: true
      +    # The location where Build Artifacts are stored (default: shared/artifacts).
      +    path: /var/opt/gitlab/gitlab-rails/shared/artifacts
      +    object_store:
      +      enabled: false
      +      remote_directory: 'artifacts'
      +      connection: {}
      +
      +  ## Git LFS
      +  lfs:
      +    enabled: 
      +    # The location where LFS objects are stored (default: shared/lfs-objects).
      +    storage_path: /var/opt/gitlab/gitlab-rails/shared/lfs-objects
      +
      +  ## Container Registry
      +  registry:
      +    enabled: false
      +    host: 
      +    port: 
      +    api_url:  # internal address to the registry, will be used by GitLab to directly communicate with API
      +    path: 
      +    key: /var/opt/gitlab/gitlab-rails/etc/gitlab-registry.key
      +    issuer: omnibus-gitlab-issuer
      +
      +  mattermost:
      +    enabled: false
      +    host: 
      +
      +  ## GitLab Pages
      +  pages:
      +    enabled: false
      +    path: /var/opt/gitlab/gitlab-rails/shared/pages
      +    host: 
      +    port: 
      +    https: false
      +    external_http: nil
      +    external_https: nil
      +
      +  ## Gravatar
      +  ## For Libravatar see: https://docs.gitlab.com/ce/customization/libravatar.html
      +  gravatar:
      +    # gravatar urls: possible placeholders: %{hash} %{size} %{email}
      +    plain_url:      # default: http://www.gravatar.com/avatar/%{hash}?s=%{size}&d=identicon
      +    ssl_url:       # default: https://secure.gravatar.com/avatar/%{hash}?s=%{size}&d=identicon
      +
      +  ## Auxiliary jobs
      +  # Periodically executed jobs, to self-heal GitLab, do external synchronizations, etc.
      +  # Please read here for more information: https://github.com/ondrejbartas/sidekiq-cron#adding-cron-job
      +  cron_jobs:
      +    # Flag stuck CI builds as failed
      +    stuck_ci_jobs_worker:
      +      cron:
      +    # Remove expired build artifacts
      +    expire_build_artifacts_worker:
      +      cron:
      +    # Schedule pipelines in the near future
      +    pipeline_schedule_worker:
      +      cron:
      +    # Periodically run 'git fsck' on all repositories. If started more than
      +    # once per hour you will have concurrent 'git fsck' jobs.
      +    repository_check_worker:
      +      cron:
      +    # Send admin emails once a week
      +    admin_email_worker:
      +      cron:
      +
      +    # Remove outdated repository archives
      +    repository_archive_cache_worker:
      +      cron:
      +
      +    ##
      +    # GitLab EE only jobs:
      +
      +    # Snapshot active users statistics
      +
      +    # In addition to refreshing users when they log in,
      +    # periodically refresh LDAP users membership.
      +    # NOTE: This will only take effect if LDAP is enabled
      +
      +    # GitLab LDAP group sync worker
      +    # NOTE: This will only take effect if LDAP is enabled
      +
      +    # Gitlab Geo nodes notification worker
      +    # NOTE: This will only take effect if Geo is enabled
      +
      +    # GitLab Geo repository sync worker
      +    # NOTE: This will only take effect if Geo is enabled
      +
      +    # GitLab Geo file download dispatch worker
      +    # NOTE: This will only take effect if Geo is enabled
      +
      +  #
      +  # 2. GitLab CI settings
      +  # ==========================
      +
      +  gitlab_ci:
      +    # Default project notifications settings:
      +    #
      +    # Send emails only on broken builds (default: true)
      +    all_broken_builds: 
      +    #
      +    # Add pusher to recipients list (default: false)
      +    add_pusher: 
      +
      +    # The location where build traces are stored (default: builds/). Relative paths are relative to Rails.root
      +    builds_path: /var/opt/gitlab/gitlab-ci/builds
      +
      +  #
      +  # 3. Auth settings
      +  # ==========================
      +
      +  ## LDAP settings
      +  # You can inspect a sample of the LDAP users with login access by running:
      +  #   bundle exec rake gitlab:ldap:check RAILS_ENV=production
      +  ldap:
      +    enabled: false
      +    sync_time: 
      +    host: 
      +    port: 
      +    uid: 
      +    method:  # "tls" or "ssl" or "plain"
      +    bind_dn: 
      +    password: 
      +    active_directory: 
      +    allow_username_or_email_login: 
      +    base: 
      +    user_filter: 
      +
      +    ## EE only
      +    group_base: 
      +    admin_group: 
      +    sync_ssh_keys: 
      +    sync_time: 
      +
      +  ## Kerberos settings
      +  kerberos:
      +    # Allow the HTTP Negotiate authentication method for Git clients
      +    enabled: 
      +
      +    # Kerberos 5 keytab file. The keytab file must be readable by the GitLab user,
      +    # and should be different from other keytabs in the system.
      +    # (default: use default keytab from Krb5 config)
      +    keytab: 
      +
      +    # The Kerberos service name to be used by GitLab.
      +    # (default: accept any service name in keytab file)
      +    service_principal_name: 
      +
      +    # Dedicated port: Git before 2.4 does not fall back to Basic authentication if Negotiate fails.
      +    # To support both Basic and Negotiate methods with older versions of Git, configure
      +    # nginx to proxy GitLab on an extra port (e.g. 8443) and uncomment the following lines
      +    # to dedicate this port to Kerberos authentication. (default: false)
      +    use_dedicated_port: 
      +    port: 
      +    https: 
      +
      +
      +  ## OmniAuth settings
      +  omniauth:
      +    # Allow login via Twitter, Google, etc. using OmniAuth providers
      +    enabled: false
      +
      +    # Uncomment this to automatically sign in with a specific omniauth provider's without
      +    # showing GitLab's sign-in page (default: show the GitLab sign-in page)
      +    auto_sign_in_with_provider: 
      +
      +    # Sync user's email address from the specified Omniauth provider every time the user logs
      +    # in (default: nil). And consequently make this field read-only.
      +
      +    # CAUTION!
      +    # This allows users to login without having a user account first. Define the allowed
      +    # providers using an array, e.g. ["saml", "twitter"]
      +    # User accounts will be created automatically when authentication was successful.
      +    allow_single_sign_on: ["saml"]
      +
      +    # Locks down those users until they have been cleared by the admin (default: true).
      +    block_auto_created_users: 
      +    # Look up new users in LDAP servers. If a match is found (same uid), automatically
      +    # link the omniauth identity with the LDAP account. (default: false)
      +    auto_link_ldap_user: 
      +
      +    # Allow users with existing accounts to login and auto link their account via SAML
      +    # login, without having to do a manual login first and manually add SAML
      +    # (default: false)
      +    auto_link_saml_user: null
      +
      +    # Set different Omniauth providers as external so that all users creating accounts
      +    # via these providers will not be able to have access to internal projects. You
      +    # will need to use the full name of the provider, like `google_oauth2` for Google.
      +    # Refer to the examples below for the full names of the supported providers.
      +    # (default: [])
      +    external_providers: null
      +
      +    ## Auth providers
      +    # Uncomment the following lines and fill in the data of the auth provider you want to use
      +    # If your favorite auth provider is not listed you can use others:
      +    # see https://github.com/gitlabhq/gitlab-public-wiki/wiki/Custom-omniauth-provider-configurations
      +    # The 'app_id' and 'app_secret' parameters are always passed as the first two
      +    # arguments, followed by optional 'args' which can be either a hash or an array.
      +    # Documentation for this is available at https://docs.gitlab.com/ce/integration/omniauth.html
      +    providers:
      +      # - { name: 'google_oauth2', app_id: 'YOUR APP ID',
      +      #     app_secret: 'YOUR APP SECRET',
      +      #     args: { access_type: 'offline', approval_prompt: '' } }
      +      # - { name: 'twitter', app_id: 'YOUR APP ID',
      +      #     app_secret: 'YOUR APP SECRET'}
      +      # - { name: 'github', app_id: 'YOUR APP ID',
      +      #     app_secret: 'YOUR APP SECRET',
      +      #     args: { scope: 'user:email' } }
      +
      +  # Shared file storage settings
      +  shared:
      +    path: /var/opt/gitlab/gitlab-rails/shared
      +
      +  # Gitaly settings
      +  # This setting controls whether GitLab uses Gitaly
      +  # Eventually Gitaly use will become mandatory and
      +  # this option will disappear.
      +  gitaly:
      +    token: ""
      +
      +
      +  #
      +  # 4. Advanced settings
      +  # ==========================
      +
      +  ## Repositories settings
      +  repositories:
      +    # Paths where repositories can be stored. Give the canonicalized absolute pathname.
      +    # NOTE: REPOS PATHS MUST NOT CONTAIN ANY SYMLINK!!!
      +    storages: {"default":{"path":"/var/opt/gitlab/git-data/repositories","gitaly_address":"unix:/var/opt/gitlab/gitaly/gitaly.socket"}}
      +
      +  ## Backup settings
      +  backup:
      +    path: "/var/opt/gitlab/backups"   # Relative paths are relative to Rails.root (default: tmp/backups/)
      +    archive_permissions:  # Permissions for the resulting backup.tar file (default: 0600)
      +    keep_time:    # default: 0 (forever) (in seconds)
      +    pg_schema:    # default: nil, it means that all schemas will be backed up
      +    upload:
      +      # Fog storage connection settings, see http://fog.io/storage/ .
      +      connection: 
      +      # The remote 'directory' to store your backups. For S3, this would be the bucket name.
      +      remote_directory: 
      +      multipart_chunk_size: 
      +      encryption: 
      +      storage_class: 
      +
      +  ## GitLab Shell settings
      +  gitlab_shell:
      +    path: /opt/gitlab/embedded/service/gitlab-shell/
      +    hooks_path: /opt/gitlab/embedded/service/gitlab-shell/hooks/
      +
      +    # Git over HTTP
      +    upload_pack: 
      +    receive_pack: 
      +
      +    # If you use non-standard ssh port you need to specify it
      +    ssh_port: 
      +
      +    # Git import/fetch timeout
      +    git_timeout: 800
      +
      +  ## Git settings
      +  # CAUTION!
      +  # Use the default values unless you really know what you are doing
      +  git:
      +    bin_path: /opt/gitlab/embedded/bin/git
      +    # The next value is the maximum memory size grit can use
      +    # Given in number of bytes per git object (e.g. a commit)
      +    # This value can be increased if you have very large commits
      +    max_size: 
      +    # Git timeout to read a commit, in seconds
      +    timeout: 
      +
      +  ## GitLab Geo settings (EE-only)
      +  geo_primary_role:
      +    enabled: false
      +  geo_secondary_role:
      +    enabled: false
      +
      +  monitoring:
      +    # Time between sampling of unicorn socket metrics, in seconds
      +    unicorn_sampler_interval: 10
      +    # IP whitelist controlling access to monitoring endpoints
      +    ip_whitelist:
      +      - 127.0.0.0/8
      +
      +  #
      +  # 5. Extra customization
      +  # ==========================
      +
      +  extra:
      +
      +
      +  rack_attack:
      +    git_basic_auth: 
      +
      +
      +development:
      +  <<: *base
      +
      +test:
      +  <<: *base
      +  gravatar:
      +    enabled: true
      +  gitlab:
      +    host: localhost
      +    port: 80
      +
      +    # When you run tests we clone and setup gitlab-shell
      +    # In order to setup it correctly you need to specify
      +    # your system username you use to run GitLab
      +    # user: YOUR_USERNAME
      +  repositories:
      +    storages:
      +      default: { "path": "tmp/tests/repositories/" }
      +  gitlab_shell:
      +    path: tmp/tests/gitlab-shell/
      +    hooks_path: tmp/tests/gitlab-shell/hooks/
      +  issues_tracker:
      +    redmine:
      +      title: "Redmine"
      +      project_url: "http://redmine/projects/:issues_tracker_id"
      +      issues_url: "http://redmine/:project_id/:issues_tracker_id/:id"
      +      new_issue_url: "http://redmine/projects/:issues_tracker_id/issues/new"
      +    jira:
      +      title: "JIRA"
      +      url: https://samplecompany.example.net
      +      project_key: PROJECT
      +  ldap:
      +    enabled: false
      +    servers:
      +      main:
      +        label: ldap
      +        host: 127.0.0.1
      +        port: 3890
      +        uid: 'uid'
      +        method: 'plain' # "tls" or "ssl" or "plain"
      +        base: 'dc=example,dc=com'
      +        user_filter: ''
      +        group_base: 'ou=groups,dc=example,dc=com'
      +        admin_group: ''
      +        sync_ssh_keys: false
      +
      +staging:
      +  <<: *base
      - change mode from '' to '0644'
      - change owner from '' to 'root'
      - change group from '' to 'root'
      - restore selinux security context
    * link[Link /opt/gitlab/embedded/service/gitlab-rails/config/gitlab.yml to /var/opt/gitlab/gitlab-rails/etc/gitlab.yml] action create
      - create symlink at /opt/gitlab/embedded/service/gitlab-rails/config/gitlab.yml to /var/opt/gitlab/gitlab-rails/etc/gitlab.yml
  
  * templatesymlink[Create a rack_attack.rb and create a symlink to Rails root] action create
    * template[/var/opt/gitlab/gitlab-rails/etc/rack_attack.rb] action create
      - create new file /var/opt/gitlab/gitlab-rails/etc/rack_attack.rb
      - update content in file /var/opt/gitlab/gitlab-rails/etc/rack_attack.rb from none to a61b95
      --- /var/opt/gitlab/gitlab-rails/etc/rack_attack.rb	2017-07-24 00:12:59.774000000 +0800
      +++ /var/opt/gitlab/gitlab-rails/etc/.chef-rack_attack.rb20170724-7201-pgsx81	2017-07-24 00:12:59.774000000 +0800
      @@ -1 +1,32 @@
      +# This file is managed by gitlab-ctl. Manual changes will be
      +# erased! To change the contents below, edit /etc/gitlab/gitlab.rb
      +# and run `sudo gitlab-ctl reconfigure`.
      +
      +# 1. Rename this file to rack_attack.rb
      +# 2. Review the paths_to_be_protected and add any other path you need protecting
      +#
      +
      +paths_to_be_protected = [
      +  "#{Rails.application.config.relative_url_root}/users/password",
      +  "#{Rails.application.config.relative_url_root}/users/sign_in",
      +  "#{Rails.application.config.relative_url_root}/api/#{API::API.version}/session.json",
      +  "#{Rails.application.config.relative_url_root}/api/#{API::API.version}/session",
      +  "#{Rails.application.config.relative_url_root}/users",
      +  "#{Rails.application.config.relative_url_root}/users/confirmation",
      +  "#{Rails.application.config.relative_url_root}/unsubscribes/",
      +  "#{Rails.application.config.relative_url_root}/import/github/personal_access_token",
      +]
      +
      +# Create one big regular expression that matches strings starting with any of
      +# the paths_to_be_protected.
      +paths_regex = Regexp.union(paths_to_be_protected.map { |path| /\A#{Regexp.escape(path)}/ })
      +rack_attack_enabled = Gitlab.config.rack_attack.git_basic_auth['enabled']
      +
      +unless Rails.env.test? || !rack_attack_enabled
      +  Rack::Attack.throttle('protected paths', limit: 10, period: 60.seconds) do |req|
      +    if req.post? && req.path =~ paths_regex
      +      req.ip
      +    end
      +  end
      +end
      - change mode from '' to '0644'
      - change owner from '' to 'root'
      - change group from '' to 'root'
      - restore selinux security context
    * link[Link /opt/gitlab/embedded/service/gitlab-rails/config/initializers/rack_attack.rb to /var/opt/gitlab/gitlab-rails/etc/rack_attack.rb] action create
      - create symlink at /opt/gitlab/embedded/service/gitlab-rails/config/initializers/rack_attack.rb to /var/opt/gitlab/gitlab-rails/etc/rack_attack.rb
  
  * templatesymlink[Create a gitlab_workhorse_secret and create a symlink to Rails root] action create
    * template[/var/opt/gitlab/gitlab-rails/etc/gitlab_workhorse_secret] action create
      - create new file /var/opt/gitlab/gitlab-rails/etc/gitlab_workhorse_secret
      - update content in file /var/opt/gitlab/gitlab-rails/etc/gitlab_workhorse_secret from none to a0a86d
      - suppressed sensitive resource
      - change mode from '' to '0644'
      - change owner from '' to 'root'
      - change group from '' to 'root'
      - restore selinux security context
    * link[Link /opt/gitlab/embedded/service/gitlab-rails/.gitlab_workhorse_secret to /var/opt/gitlab/gitlab-rails/etc/gitlab_workhorse_secret] action create
      - create symlink at /opt/gitlab/embedded/service/gitlab-rails/.gitlab_workhorse_secret to /var/opt/gitlab/gitlab-rails/etc/gitlab_workhorse_secret
  
  * templatesymlink[Create a gitlab_shell_secret and create a symlink to Rails root] action create
    * template[/var/opt/gitlab/gitlab-rails/etc/gitlab_shell_secret] action create
      - create new file /var/opt/gitlab/gitlab-rails/etc/gitlab_shell_secret
      - update content in file /var/opt/gitlab/gitlab-rails/etc/gitlab_shell_secret from none to 18cbe1
      - suppressed sensitive resource
      - change mode from '' to '0644'
      - change owner from '' to 'root'
      - change group from '' to 'root'
      - restore selinux security context
    * link[Link /opt/gitlab/embedded/service/gitlab-rails/.gitlab_shell_secret to /var/opt/gitlab/gitlab-rails/etc/gitlab_shell_secret] action create
      - create symlink at /opt/gitlab/embedded/service/gitlab-rails/.gitlab_shell_secret to /var/opt/gitlab/gitlab-rails/etc/gitlab_shell_secret
  
  * directory[/opt/gitlab/etc/gitlab-rails/env] action create
    - create new directory /opt/gitlab/etc/gitlab-rails/env
    - restore selinux security context
  * file[/opt/gitlab/etc/gitlab-rails/env/HOME] action create
    - create new file /opt/gitlab/etc/gitlab-rails/env/HOME
    - update content in file /opt/gitlab/etc/gitlab-rails/env/HOME from none to 205bb9
    --- /opt/gitlab/etc/gitlab-rails/env/HOME	2017-07-24 00:12:59.909000000 +0800
    +++ /opt/gitlab/etc/gitlab-rails/env/.chef-HOME20170724-7201-yjudn5	2017-07-24 00:12:59.909000000 +0800
    @@ -1 +1,2 @@
    +/var/opt/gitlab
    - restore selinux security context
  * file[/opt/gitlab/etc/gitlab-rails/env/RAILS_ENV] action create
    - create new file /opt/gitlab/etc/gitlab-rails/env/RAILS_ENV
    - update content in file /opt/gitlab/etc/gitlab-rails/env/RAILS_ENV from none to ab8e18
    --- /opt/gitlab/etc/gitlab-rails/env/RAILS_ENV	2017-07-24 00:12:59.934000000 +0800
    +++ /opt/gitlab/etc/gitlab-rails/env/.chef-RAILS_ENV20170724-7201-1i8dfqh	2017-07-24 00:12:59.934000000 +0800
    @@ -1 +1,2 @@
    +production
    - restore selinux security context
  * file[/opt/gitlab/etc/gitlab-rails/env/LD_PRELOAD] action create
    - create new file /opt/gitlab/etc/gitlab-rails/env/LD_PRELOAD
    - update content in file /opt/gitlab/etc/gitlab-rails/env/LD_PRELOAD from none to f79114
    --- /opt/gitlab/etc/gitlab-rails/env/LD_PRELOAD	2017-07-24 00:12:59.959000000 +0800
    +++ /opt/gitlab/etc/gitlab-rails/env/.chef-LD_PRELOAD20170724-7201-1ogxdeo	2017-07-24 00:12:59.959000000 +0800
    @@ -1 +1,2 @@
    +/opt/gitlab/embedded/lib/libjemalloc.so
    - restore selinux security context
  * file[/opt/gitlab/etc/gitlab-rails/env/SIDEKIQ_MEMORY_KILLER_MAX_RSS] action create
    - create new file /opt/gitlab/etc/gitlab-rails/env/SIDEKIQ_MEMORY_KILLER_MAX_RSS
    - update content in file /opt/gitlab/etc/gitlab-rails/env/SIDEKIQ_MEMORY_KILLER_MAX_RSS from none to 6cce36
    --- /opt/gitlab/etc/gitlab-rails/env/SIDEKIQ_MEMORY_KILLER_MAX_RSS	2017-07-24 00:12:59.984000000 +0800
    +++ /opt/gitlab/etc/gitlab-rails/env/.chef-SIDEKIQ_MEMORY_KILLER_MAX_RSS20170724-7201-ju6xgk	2017-07-24 00:12:59.984000000 +0800
    @@ -1 +1,2 @@
    +1000000
    - restore selinux security context
  * file[/opt/gitlab/etc/gitlab-rails/env/BUNDLE_GEMFILE] action create
    - create new file /opt/gitlab/etc/gitlab-rails/env/BUNDLE_GEMFILE
    - update content in file /opt/gitlab/etc/gitlab-rails/env/BUNDLE_GEMFILE from none to 28d586
    --- /opt/gitlab/etc/gitlab-rails/env/BUNDLE_GEMFILE	2017-07-24 00:13:00.005000000 +0800
    +++ /opt/gitlab/etc/gitlab-rails/env/.chef-BUNDLE_GEMFILE20170724-7201-129x48n	2017-07-24 00:13:00.005000000 +0800
    @@ -1 +1,2 @@
    +/opt/gitlab/embedded/service/gitlab-rails/Gemfile
    - restore selinux security context
  * file[/opt/gitlab/etc/gitlab-rails/env/PATH] action create
    - create new file /opt/gitlab/etc/gitlab-rails/env/PATH
    - update content in file /opt/gitlab/etc/gitlab-rails/env/PATH from none to d5dc07
    --- /opt/gitlab/etc/gitlab-rails/env/PATH	2017-07-24 00:13:00.031000000 +0800
    +++ /opt/gitlab/etc/gitlab-rails/env/.chef-PATH20170724-7201-1iejh5t	2017-07-24 00:13:00.031000000 +0800
    @@ -1 +1,2 @@
    +/opt/gitlab/bin:/opt/gitlab/embedded/bin:/bin:/usr/bin
    - restore selinux security context
  * file[/opt/gitlab/etc/gitlab-rails/env/ICU_DATA] action create
    - create new file /opt/gitlab/etc/gitlab-rails/env/ICU_DATA
    - update content in file /opt/gitlab/etc/gitlab-rails/env/ICU_DATA from none to a04260
    --- /opt/gitlab/etc/gitlab-rails/env/ICU_DATA	2017-07-24 00:13:00.055000000 +0800
    +++ /opt/gitlab/etc/gitlab-rails/env/.chef-ICU_DATA20170724-7201-jio9ij	2017-07-24 00:13:00.055000000 +0800
    @@ -1 +1,2 @@
    +/opt/gitlab/embedded/share/icu/current
    - restore selinux security context
  * file[/opt/gitlab/etc/gitlab-rails/env/PYTHONPATH] action create
    - create new file /opt/gitlab/etc/gitlab-rails/env/PYTHONPATH
    - update content in file /opt/gitlab/etc/gitlab-rails/env/PYTHONPATH from none to 990cc2
    --- /opt/gitlab/etc/gitlab-rails/env/PYTHONPATH	2017-07-24 00:13:00.081000000 +0800
    +++ /opt/gitlab/etc/gitlab-rails/env/.chef-PYTHONPATH20170724-7201-1wp48fw	2017-07-24 00:13:00.081000000 +0800
    @@ -1 +1,2 @@
    +/opt/gitlab/embedded/lib/python3.4/site-packages
    - restore selinux security context
  * file[/opt/gitlab/etc/gitlab-rails/env/EXECJS_RUNTIME] action create
    - create new file /opt/gitlab/etc/gitlab-rails/env/EXECJS_RUNTIME
    - update content in file /opt/gitlab/etc/gitlab-rails/env/EXECJS_RUNTIME from none to 75081b
    --- /opt/gitlab/etc/gitlab-rails/env/EXECJS_RUNTIME	2017-07-24 00:13:00.104000000 +0800
    +++ /opt/gitlab/etc/gitlab-rails/env/.chef-EXECJS_RUNTIME20170724-7201-q3ytb3	2017-07-24 00:13:00.104000000 +0800
    @@ -1 +1,2 @@
    +Disabled
    - restore selinux security context
  * link[/opt/gitlab/embedded/service/gitlab-rails/tmp] action create
    - create symlink at /opt/gitlab/embedded/service/gitlab-rails/tmp to /var/opt/gitlab/gitlab-rails/tmp
  * link[/opt/gitlab/embedded/service/gitlab-rails/public/uploads] action create
    - create symlink at /opt/gitlab/embedded/service/gitlab-rails/public/uploads to /var/opt/gitlab/gitlab-rails/uploads
  * link[/opt/gitlab/embedded/service/gitlab-rails/log] action create
    - create symlink at /opt/gitlab/embedded/service/gitlab-rails/log to /var/log/gitlab/gitlab-rails
  * link[/var/log/gitlab/gitlab-rails/sidekiq.log] action create
    - create symlink at /var/log/gitlab/gitlab-rails/sidekiq.log to /var/log/gitlab/sidekiq/current
  * file[/opt/gitlab/embedded/service/gitlab-rails/db/schema.rb] action create
    - change owner from 'root' to 'git'
    - restore selinux security context
  * remote_file[/var/opt/gitlab/gitlab-rails/VERSION] action create
    - create new file /var/opt/gitlab/gitlab-rails/VERSION
    - update content in file /var/opt/gitlab/gitlab-rails/VERSION from none to b138de
    --- /var/opt/gitlab/gitlab-rails/VERSION	2017-07-24 00:13:00.156000000 +0800
    +++ /var/opt/gitlab/gitlab-rails/.chef-VERSION20170724-7201-1qlqusm	2017-07-24 00:13:00.156000000 +0800
    @@ -1 +1,2 @@
    +9.4.0
    - restore selinux security context
  * remote_file[/var/opt/gitlab/gitlab-rails/REVISION] action create
    - create new file /var/opt/gitlab/gitlab-rails/REVISION
    - update content in file /var/opt/gitlab/gitlab-rails/REVISION from none to 6271d4
    --- /var/opt/gitlab/gitlab-rails/REVISION	2017-07-24 00:13:00.181000000 +0800
    +++ /var/opt/gitlab/gitlab-rails/.chef-REVISION20170724-7201-1e628xr	2017-07-24 00:13:00.181000000 +0800
    @@ -1 +1,2 @@
    +9bbe2ac
    - restore selinux security context
  * file[/var/opt/gitlab/gitlab-rails/RUBY_VERSION] action create
    - create new file /var/opt/gitlab/gitlab-rails/RUBY_VERSION
    - update content in file /var/opt/gitlab/gitlab-rails/RUBY_VERSION from none to ed5a25
    --- /var/opt/gitlab/gitlab-rails/RUBY_VERSION	2017-07-24 00:13:00.206000000 +0800
    +++ /var/opt/gitlab/gitlab-rails/.chef-RUBY_VERSION20170724-7201-kr370z	2017-07-24 00:13:00.206000000 +0800
    @@ -1 +1,2 @@
    +ruby 2.3.3p222 (2016-11-21 revision 56859) [x86_64-linux]
    - restore selinux security context
  * execute[chown -R root:root /opt/gitlab/embedded/service/gitlab-rails/public] action run
    - execute chown -R root:root /opt/gitlab/embedded/service/gitlab-rails/public
  * execute[clear the gitlab-rails cache] action nothing (skipped due to action :nothing)
  * file[/var/opt/gitlab/gitlab-rails/config.ru] action delete (up to date)
Recipe: gitlab::selinux
  * execute[semodule -i /opt/gitlab/embedded/selinux/rhel/7/gitlab-7.2.0-ssh-keygen.pp] action run
    - execute semodule -i /opt/gitlab/embedded/selinux/rhel/7/gitlab-7.2.0-ssh-keygen.pp
Recipe: gitlab::add_trusted_certs
  * directory[/etc/gitlab/trusted-certs] action create
    - create new directory /etc/gitlab/trusted-certs
    - change mode from '' to '0755'
    - restore selinux security context
  * directory[/opt/gitlab/embedded/ssl/certs] action create (up to date)
  * file[/opt/gitlab/embedded/ssl/certs/README] action create
    - create new file /opt/gitlab/embedded/ssl/certs/README
    - update content in file /opt/gitlab/embedded/ssl/certs/README from none to 623059
    --- /opt/gitlab/embedded/ssl/certs/README	2017-07-24 00:13:16.754000000 +0800
    +++ /opt/gitlab/embedded/ssl/certs/.chef-README20170724-7201-1ues06e	2017-07-24 00:13:16.754000000 +0800
    @@ -1 +1,4 @@
    +This directory is managed by omnibus-gitlab.
    + Any file placed in this directory will be ignored
    +. Place certificates in /etc/gitlab/trusted-certs.
    - change mode from '' to '0644'
    - restore selinux security context
  * ruby_block[Move existing certs and link to /opt/gitlab/embedded/ssl/certs] action run

  * Moving existing certificates found in /opt/gitlab/embedded/ssl/certs

  * Symlinking existing certificates found in /etc/gitlab/trusted-certs

    - execute the ruby block Move existing certs and link to /opt/gitlab/embedded/ssl/certs
Recipe: gitlab::default
  * service[create a temporary unicorn service] action nothing (skipped due to action :nothing)
  * service[create a temporary sidekiq service] action nothing (skipped due to action :nothing)
  * service[create a temporary mailroom service] action nothing (skipped due to action :nothing)
Recipe: runit::systemd
  * directory[/usr/lib/systemd/system] action create (up to date)
  * cookbook_file[/usr/lib/systemd/system/gitlab-runsvdir.service] action create
    - create new file /usr/lib/systemd/system/gitlab-runsvdir.service
    - update content in file /usr/lib/systemd/system/gitlab-runsvdir.service from none to bf758a
    --- /usr/lib/systemd/system/gitlab-runsvdir.service	2017-07-24 00:13:16.845000000 +0800
    +++ /usr/lib/systemd/system/.chef-gitlab-runsvdir.service20170724-7201-1p6932	2017-07-24 00:13:16.844000000 +0800
    @@ -1 +1,11 @@
    +[Unit]
    +Description=GitLab Runit supervision process
    +After=basic.target
    +
    +[Service]
    +ExecStart=/opt/gitlab/embedded/bin/runsvdir-start
    +Restart=always
    +
    +[Install]
    +WantedBy=basic.target
    - change mode from '' to '0644'
    - restore selinux security context
  * execute[systemctl daemon-reload] action run
    - execute systemctl daemon-reload
  * execute[systemctl enable gitlab-runsvdir] action run
    [execute] Created symlink from /etc/systemd/system/basic.target.wants/gitlab-runsvdir.service to /usr/lib/systemd/system/gitlab-runsvdir.service.
    - execute systemctl enable gitlab-runsvdir
  * execute[systemctl start gitlab-runsvdir] action run
    - execute systemctl start gitlab-runsvdir
  * file[/etc/systemd/system/default.target.wants/gitlab-runsvdir.service] action delete (up to date)
  * execute[systemctl daemon-reload] action nothing (skipped due to action :nothing)
  * execute[systemctl enable gitlab-runsvdir] action nothing (skipped due to action :nothing)
  * execute[systemctl start gitlab-runsvdir] action nothing (skipped due to action :nothing)
Recipe: gitlab::redis
  * group[user and group for redis] action create
    - create group gitlab-redis
  * user[user and group for redis] action create
    - create user gitlab-redis
  * group[Socket group] action create (up to date)
  * directory[/var/opt/gitlab/redis] action create
    - create new directory /var/opt/gitlab/redis
    - change mode from '' to '0750'
    - change owner from '' to 'gitlab-redis'
    - change group from '' to 'git'
    - restore selinux security context
  * directory[/var/log/gitlab/redis] action create
    - create new directory /var/log/gitlab/redis
    - change mode from '' to '0700'
    - change owner from '' to 'gitlab-redis'
    - restore selinux security context
  * template[/var/opt/gitlab/redis/redis.conf] action create
    - create new file /var/opt/gitlab/redis/redis.conf
    - update content in file /var/opt/gitlab/redis/redis.conf from none to c1ed62
    --- /var/opt/gitlab/redis/redis.conf	2017-07-24 00:13:17.374000000 +0800
    +++ /var/opt/gitlab/redis/.chef-redis.conf20170724-7201-2frh6c	2017-07-24 00:13:17.373000000 +0800
    @@ -1 +1,1059 @@
    +# This file is managed by gitlab-ctl. Manual changes will be
    +# erased! To change the contents below, edit /etc/gitlab/gitlab.rb
    +# and run `sudo gitlab-ctl reconfigure`.
    +
    +# Redis configuration file example.
    +#
    +# Note that in order to read the configuration file, Redis must be
    +# started with the file path as first argument:
    +#
    +# ./redis-server /path/to/redis.conf
    +
    +# Note on units: when memory size is needed, it is possible to specify
    +# it in the usual form of 1k 5GB 4M and so forth:
    +#
    +# 1k => 1000 bytes
    +# 1kb => 1024 bytes
    +# 1m => 1000000 bytes
    +# 1mb => 1024*1024 bytes
    +# 1g => 1000000000 bytes
    +# 1gb => 1024*1024*1024 bytes
    +#
    +# units are case insensitive so 1GB 1Gb 1gB are all the same.
    +
    +################################## INCLUDES ###################################
    +
    +# Include one or more other config files here.  This is useful if you
    +# have a standard template that goes to all Redis servers but also need
    +# to customize a few per-server settings.  Include files can include
    +# other files, so use this wisely.
    +#
    +# Notice option "include" won't be rewritten by command "CONFIG REWRITE"
    +# from admin or Redis Sentinel. Since Redis always uses the last processed
    +# line as value of a configuration directive, you'd better put includes
    +# at the beginning of this file to avoid overwriting config change at runtime.
    +#
    +# If instead you are interested in using includes to override configuration
    +# options, it is better to use include as the last line.
    +#
    +# include /path/to/local.conf
    +# include /path/to/other.conf
    +
    +################################## NETWORK #####################################
    +
    +# By default, if no "bind" configuration directive is specified, Redis listens
    +# for connections from all the network interfaces available on the server.
    +# It is possible to listen to just one or multiple selected interfaces using
    +# the "bind" configuration directive, followed by one or more IP addresses.
    +#
    +# Examples:
    +#
    +# bind 192.168.1.100 10.0.0.1
    +# bind 127.0.0.1 ::1
    +#
    +# ~~~ WARNING ~~~ If the computer running Redis is directly exposed to the
    +# internet, binding to all the interfaces is dangerous and will expose the
    +# instance to everybody on the internet. So by default we uncomment the
    +# following bind directive, that will force Redis to listen only into
    +# the IPv4 lookback interface address (this means Redis will be able to
    +# accept connections only from clients running into the same computer it
    +# is running).
    +#
    +# IF YOU ARE SURE YOU WANT YOUR INSTANCE TO LISTEN TO ALL THE INTERFACES
    +# JUST COMMENT THE FOLLOWING LINE.
    +# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    +bind 127.0.0.1
    +
    +# Protected mode is a layer of security protection, in order to avoid that
    +# Redis instances left open on the internet are accessed and exploited.
    +#
    +# When protected mode is on and if:
    +#
    +# 1) The server is not binding explicitly to a set of addresses using the
    +#    "bind" directive.
    +# 2) No password is configured.
    +#
    +# The server only accepts connections from clients connecting from the
    +# IPv4 and IPv6 loopback addresses 127.0.0.1 and ::1, and from Unix domain
    +# sockets.
    +#
    +# By default protected mode is enabled. You should disable it only if
    +# you are sure you want clients from other hosts to connect to Redis
    +# even if no authentication is configured, nor a specific set of interfaces
    +# are explicitly listed using the "bind" directive.
    +# protected-mode yes
    +
    +# Accept connections on the specified port, default is 6379 (IANA #815344).
    +# If port 0 is specified Redis will not listen on a TCP socket.
    +port 0
    +
    +# TCP listen() backlog.
    +#
    +# In high requests-per-second environments you need an high backlog in order
    +# to avoid slow clients connections issues. Note that the Linux kernel
    +# will silently truncate it to the value of /proc/sys/net/core/somaxconn so
    +# make sure to raise both the value of somaxconn and tcp_max_syn_backlog
    +# in order to get the desired effect.
    +# tcp-backlog 511
    +
    +# Unix socket.
    +#
    +# Specify the path for the Unix socket that will be used to listen for
    +# incoming connections. There is no default, so Redis will not listen
    +# on a unix socket when not specified.
    +#
    +unixsocket /var/opt/gitlab/redis/redis.socket
    +unixsocketperm 777
    +
    +# Close the connection after a client is idle for N seconds (0 to disable)
    +timeout 60
    +
    +# TCP keepalive.
    +#
    +# If non-zero, use SO_KEEPALIVE to send TCP ACKs to clients in absence
    +# of communication. This is useful for two reasons:
    +#
    +# 1) Detect dead peers.
    +# 2) Take the connection alive from the point of view of network
    +#    equipment in the middle.
    +#
    +# On Linux, the specified value (in seconds) is the period used to send ACKs.
    +# Note that to close the connection the double of the time is needed.
    +# On other kernels the period depends on the kernel configuration.
    +#
    +# A reasonable value for this option is 300 seconds, which is the new
    +# Redis default starting with Redis 3.2.1.
    +tcp-keepalive 300
    +
    +################################# GENERAL #####################################
    +
    +# By default Redis does not run as a daemon. Use 'yes' if you need it.
    +# Note that Redis will write a pid file in /var/run/redis.pid when daemonized.
    +daemonize no
    +
    +# If you run Redis from upstart or systemd, Redis can interact with your
    +# supervision tree. Options:
    +#   supervised no      - no supervision interaction
    +#   supervised upstart - signal upstart by putting Redis into SIGSTOP mode
    +#   supervised systemd - signal systemd by writing READY=1 to $NOTIFY_SOCKET
    +#   supervised auto    - detect upstart or systemd method based on
    +#                        UPSTART_JOB or NOTIFY_SOCKET environment variables
    +# Note: these supervision methods only signal "process is ready."
    +#       They do not enable continuous liveness pings back to your supervisor.
    +# supervised no
    +
    +# If a pid file is specified, Redis writes it where specified at startup
    +# and removes it at exit.
    +#
    +# When the server runs non daemonized, no pid file is created if none is
    +# specified in the configuration. When the server is daemonized, the pid file
    +# is used even if not specified, defaulting to "/var/run/redis.pid".
    +#
    +# Creating a pid file is best effort: if Redis is not able to create it
    +# nothing bad happens, the server will start and run normally.
    +pidfile "/var/run/redis_0.pid"
    +
    +# Specify the server verbosity level.
    +# This can be one of:
    +# debug (a lot of information, useful for development/testing)
    +# verbose (many rarely useful info, but not a mess like the debug level)
    +# notice (moderately verbose, what you want in production probably)
    +# warning (only very important / critical messages are logged)
    +loglevel notice
    +
    +# Specify the log file name. Also the empty string can be used to force
    +# Redis to log on the standard output. Note that if you use standard
    +# output for logging but daemonize, logs will be sent to /dev/null
    +logfile ""
    +
    +# To enable logging to the system logger, just set 'syslog-enabled' to yes,
    +# and optionally update the other syslog parameters to suit your needs.
    +# syslog-enabled no
    +
    +# Specify the syslog identity.
    +# syslog-ident redis
    +
    +# Specify the syslog facility. Must be USER or between LOCAL0-LOCAL7.
    +# syslog-facility local0
    +
    +# Set the number of databases. The default database is DB 0, you can select
    +# a different one on a per-connection basis using SELECT <dbid> where
    +# dbid is a number between 0 and 'databases'-1
    +databases 16
    +
    +################################ SNAPSHOTTING  ################################
    +#
    +# Save the DB on disk:
    +#
    +#   save <seconds> <changes>
    +#
    +#   Will save the DB if both the given number of seconds and the given
    +#   number of write operations against the DB occurred.
    +#
    +#   In the example below the behaviour will be to save:
    +#   after 900 sec (15 min) if at least 1 key changed
    +#   after 300 sec (5 min) if at least 10 keys changed
    +#   after 60 sec if at least 10000 keys changed
    +#
    +#   Note: you can disable saving completely by commenting out all "save" lines.
    +#
    +#   It is also possible to remove all the previously configured save
    +#   points by adding a save directive with a single empty string argument
    +#   like in the following example:
    +#
    +#   save ""
    +
    +save 900 1
    +save 300 10
    +save 60 10000
    +
    +# By default Redis will stop accepting writes if RDB snapshots are enabled
    +# (at least one save point) and the latest background save failed.
    +# This will make the user aware (in a hard way) that data is not persisting
    +# on disk properly, otherwise chances are that no one will notice and some
    +# disaster will happen.
    +#
    +# If the background saving process will start working again Redis will
    +# automatically allow writes again.
    +#
    +# However if you have setup your proper monitoring of the Redis server
    +# and persistence, you may want to disable this feature so that Redis will
    +# continue to work as usual even if there are problems with disk,
    +# permissions, and so forth.
    +stop-writes-on-bgsave-error yes
    +
    +# Compress string objects using LZF when dump .rdb databases?
    +# For default that's set to 'yes' as it's almost always a win.
    +# If you want to save some CPU in the saving child set it to 'no' but
    +# the dataset will likely be bigger if you have compressible values or keys.
    +rdbcompression yes
    +
    +# Since version 5 of RDB a CRC64 checksum is placed at the end of the file.
    +# This makes the format more resistant to corruption but there is a performance
    +# hit to pay (around 10%) when saving and loading RDB files, so you can disable it
    +# for maximum performances.
    +#
    +# RDB files created with checksum disabled have a checksum of zero that will
    +# tell the loading code to skip the check.
    +rdbchecksum yes
    +
    +# The filename where to dump the DB
    +dbfilename "dump.rdb"
    +
    +# The working directory.
    +#
    +# The DB will be written inside this directory, with the filename specified
    +# above using the 'dbfilename' configuration directive.
    +#
    +# The Append Only File will also be created inside this directory.
    +#
    +# Note that you must specify a directory here, not a file name.
    +dir "/var/opt/gitlab/redis"
    +
    +################################# REPLICATION #################################
    +
    +# Master-Slave replication. Use slaveof to make a Redis instance a copy of
    +# another Redis server. A few things to understand ASAP about Redis replication.
    +#
    +# 1) Redis replication is asynchronous, but you can configure a master to
    +#    stop accepting writes if it appears to be not connected with at least
    +#    a given number of slaves.
    +# 2) Redis slaves are able to perform a partial resynchronization with the
    +#    master if the replication link is lost for a relatively small amount of
    +#    time. You may want to configure the replication backlog size (see the next
    +#    sections of this file) with a sensible value depending on your needs.
    +# 3) Replication is automatic and does not need user intervention. After a
    +#    network partition slaves automatically try to reconnect to masters
    +#    and resynchronize with them.
    +#
    +# slaveof <masterip> <masterport>
    +
    +
    +# If the master is password protected (using the "requirepass" configuration
    +# directive below) it is possible to tell the slave to authenticate before
    +# starting the replication synchronization process, otherwise the master will
    +# refuse the slave request.
    +#
    +# masterauth <master-password>
    +
    +
    +# When a slave loses its connection with the master, or when the replication
    +# is still in progress, the slave can act in two different ways:
    +#
    +# 1) if slave-serve-stale-data is set to 'yes' (the default) the slave will
    +#    still reply to client requests, possibly with out of date data, or the
    +#    data set may just be empty if this is the first synchronization.
    +#
    +# 2) if slave-serve-stale-data is set to 'no' the slave will reply with
    +#    an error "SYNC with master in progress" to all the kind of commands
    +#    but to INFO and SLAVEOF.
    +#
    +slave-serve-stale-data yes
    +
    +# You can configure a slave instance to accept writes or not. Writing against
    +# a slave instance may be useful to store some ephemeral data (because data
    +# written on a slave will be easily deleted after resync with the master) but
    +# may also cause problems if clients are writing to it because of a
    +# misconfiguration.
    +#
    +# Since Redis 2.6 by default slaves are read-only.
    +#
    +# Note: read only slaves are not designed to be exposed to untrusted clients
    +# on the internet. It's just a protection layer against misuse of the instance.
    +# Still a read only slave exports by default all the administrative commands
    +# such as CONFIG, DEBUG, and so forth. To a limited extent you can improve
    +# security of read only slaves using 'rename-command' to shadow all the
    +# administrative / dangerous commands.
    +slave-read-only yes
    +
    +# Replication SYNC strategy: disk or socket.
    +#
    +# -------------------------------------------------------
    +# WARNING: DISKLESS REPLICATION IS EXPERIMENTAL CURRENTLY
    +# -------------------------------------------------------
    +#
    +# New slaves and reconnecting slaves that are not able to continue the replication
    +# process just receiving differences, need to do what is called a "full
    +# synchronization". An RDB file is transmitted from the master to the slaves.
    +# The transmission can happen in two different ways:
    +#
    +# 1) Disk-backed: The Redis master creates a new process that writes the RDB
    +#                 file on disk. Later the file is transferred by the parent
    +#                 process to the slaves incrementally.
    +# 2) Diskless: The Redis master creates a new process that directly writes the
    +#              RDB file to slave sockets, without touching the disk at all.
    +#
    +# With disk-backed replication, while the RDB file is generated, more slaves
    +# can be queued and served with the RDB file as soon as the current child producing
    +# the RDB file finishes its work. With diskless replication instead once
    +# the transfer starts, new slaves arriving will be queued and a new transfer
    +# will start when the current one terminates.
    +#
    +# When diskless replication is used, the master waits a configurable amount of
    +# time (in seconds) before starting the transfer in the hope that multiple slaves
    +# will arrive and the transfer can be parallelized.
    +#
    +# With slow disks and fast (large bandwidth) networks, diskless replication
    +# works better.
    +# repl-diskless-sync no
    +
    +# When diskless replication is enabled, it is possible to configure the delay
    +# the server waits in order to spawn the child that transfers the RDB via socket
    +# to the slaves.
    +#
    +# This is important since once the transfer starts, it is not possible to serve
    +# new slaves arriving, that will be queued for the next RDB transfer, so the server
    +# waits a delay in order to let more slaves arrive.
    +#
    +# The delay is specified in seconds, and by default is 5 seconds. To disable
    +# it entirely just set it to 0 seconds and the transfer will start ASAP.
    +# repl-diskless-sync-delay 5
    +
    +# Slaves send PINGs to server in a predefined interval. It's possible to change
    +# this interval with the repl_ping_slave_period option. The default value is 10
    +# seconds.
    +#
    +# repl-ping-slave-period 10
    +
    +# The following option sets the replication timeout for:
    +#
    +# 1) Bulk transfer I/O during SYNC, from the point of view of slave.
    +# 2) Master timeout from the point of view of slaves (data, pings).
    +# 3) Slave timeout from the point of view of masters (REPLCONF ACK pings).
    +#
    +# It is important to make sure that this value is greater than the value
    +# specified for repl-ping-slave-period otherwise a timeout will be detected
    +# every time there is low traffic between the master and the slave.
    +#
    +# repl-timeout 60
    +
    +# Disable TCP_NODELAY on the slave socket after SYNC?
    +#
    +# If you select "yes" Redis will use a smaller number of TCP packets and
    +# less bandwidth to send data to slaves. But this can add a delay for
    +# the data to appear on the slave side, up to 40 milliseconds with
    +# Linux kernels using a default configuration.
    +#
    +# If you select "no" the delay for data to appear on the slave side will
    +# be reduced but more bandwidth will be used for replication.
    +#
    +# By default we optimize for low latency, but in very high traffic conditions
    +# or when the master and slaves are many hops away, turning this to "yes" may
    +# be a good idea.
    +repl-disable-tcp-nodelay no
    +
    +# Set the replication backlog size. The backlog is a buffer that accumulates
    +# slave data when slaves are disconnected for some time, so that when a slave
    +# wants to reconnect again, often a full resync is not needed, but a partial
    +# resync is enough, just passing the portion of data the slave missed while
    +# disconnected.
    +#
    +# The bigger the replication backlog, the longer the time the slave can be
    +# disconnected and later be able to perform a partial resynchronization.
    +#
    +# The backlog is only allocated once there is at least a slave connected.
    +#
    +# repl-backlog-size 1mb
    +
    +# After a master has no longer connected slaves for some time, the backlog
    +# will be freed. The following option configures the amount of seconds that
    +# need to elapse, starting from the time the last slave disconnected, for
    +# the backlog buffer to be freed.
    +#
    +# A value of 0 means to never release the backlog.
    +#
    +# repl-backlog-ttl 3600
    +
    +# The slave priority is an integer number published by Redis in the INFO output.
    +# It is used by Redis Sentinel in order to select a slave to promote into a
    +# master if the master is no longer working correctly.
    +#
    +# A slave with a low priority number is considered better for promotion, so
    +# for instance if there are three slaves with priority 10, 100, 25 Sentinel will
    +# pick the one with priority 10, that is the lowest.
    +#
    +# However a special priority of 0 marks the slave as not able to perform the
    +# role of master, so a slave with priority of 0 will never be selected by
    +# Redis Sentinel for promotion.
    +#
    +# By default the priority is 100.
    +slave-priority 100
    +
    +# It is possible for a master to stop accepting writes if there are less than
    +# N slaves connected, having a lag less or equal than M seconds.
    +#
    +# The N slaves need to be in "online" state.
    +#
    +# The lag in seconds, that must be <= the specified value, is calculated from
    +# the last ping received from the slave, that is usually sent every second.
    +#
    +# This option does not GUARANTEE that N replicas will accept the write, but
    +# will limit the window of exposure for lost writes in case not enough slaves
    +# are available, to the specified number of seconds.
    +#
    +# For example to require at least 3 slaves with a lag <= 10 seconds use:
    +#
    +# min-slaves-to-write 3
    +# min-slaves-max-lag 10
    +#
    +# Setting one or the other to 0 disables the feature.
    +#
    +# By default min-slaves-to-write is set to 0 (feature disabled) and
    +# min-slaves-max-lag is set to 10.
    +
    +# A Redis master is able to list the address and port of the attached
    +# slaves in different ways. For example the "INFO replication" section
    +# offers this information, which is used, among other tools, by
    +# Redis Sentinel in order to discover slave instances.
    +# Another place where this info is available is in the output of the
    +# "ROLE" command of a masteer.
    +#
    +# The listed IP and address normally reported by a slave is obtained
    +# in the following way:
    +#
    +#   IP: The address is auto detected by checking the peer address
    +#   of the socket used by the slave to connect with the master.
    +#
    +#   Port: The port is communicated by the slave during the replication
    +#   handshake, and is normally the port that the slave is using to
    +#   list for connections.
    +#
    +# However when port forwarding or Network Address Translation (NAT) is
    +# used, the slave may be actually reachable via different IP and port
    +# pairs. The following two options can be used by a slave in order to
    +# report to its master a specific set of IP and port, so that both INFO
    +# and ROLE will report those values.
    +#
    +# There is no need to use both the options if you need to override just
    +# the port or the IP address.
    +#
    +# slave-announce-ip 5.5.5.5
    +# slave-announce-port 1234
    +
    +################################## SECURITY ###################################
    +
    +# Require clients to issue AUTH <PASSWORD> before processing any other
    +# commands.  This might be useful in environments in which you do not trust
    +# others with access to the host running redis-server.
    +#
    +# This should stay commented out for backward compatibility and because most
    +# people do not need auth (e.g. they run their own servers).
    +#
    +# Warning: since Redis is pretty fast an outside user can try up to
    +# 150k passwords per second against a good box. This means that you should
    +# use a very strong password otherwise it will be very easy to break.
    +#
    +
    +
    +# Command renaming.
    +#
    +# It is possible to change the name of dangerous commands in a shared
    +# environment. For instance the CONFIG command may be renamed into something
    +# hard to guess so that it will still be available for internal-use tools
    +# but not available for general clients.
    +#
    +# Example:
    +#
    +# rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52
    +#
    +# It is also possible to completely kill a command by renaming it into
    +# an empty string:
    +#
    +# rename-command CONFIG ""
    +#
    +# Please note that changing the name of commands that are logged into the
    +# AOF file or transmitted to slaves may cause problems.
    +
    +################################### LIMITS ####################################
    +
    +# Set the max number of connected clients at the same time. By default
    +# this limit is set to 10000 clients, however if the Redis server is not
    +# able to configure the process file limit to allow for the specified limit
    +# the max number of allowed clients is set to the current file limit
    +# minus 32 (as Redis reserves a few file descriptors for internal uses).
    +#
    +# Once the limit is reached Redis will close all the new connections sending
    +# an error 'max number of clients reached'.
    +#
    +maxclients 10000
    +
    +# Don't use more memory than the specified amount of bytes.
    +# When the memory limit is reached Redis will try to remove keys
    +# according to the eviction policy selected (see maxmemory-policy).
    +#
    +# If Redis can't remove keys according to the policy, or if the policy is
    +# set to 'noeviction', Redis will start to reply with errors to commands
    +# that would use more memory, like SET, LPUSH, and so on, and will continue
    +# to reply to read-only commands like GET.
    +#
    +# This option is usually useful when using Redis as an LRU cache, or to set
    +# a hard memory limit for an instance (using the 'noeviction' policy).
    +#
    +# WARNING: If you have slaves attached to an instance with maxmemory on,
    +# the size of the output buffers needed to feed the slaves are subtracted
    +# from the used memory count, so that network problems / resyncs will
    +# not trigger a loop where keys are evicted, and in turn the output
    +# buffer of slaves is full with DELs of keys evicted triggering the deletion
    +# of more keys, and so forth until the database is completely emptied.
    +#
    +# In short... if you have slaves attached it is suggested that you set a lower
    +# limit for maxmemory so that there is some free RAM on the system for slave
    +# output buffers (but this is not needed if the policy is 'noeviction').
    +#
    +# maxmemory <bytes>
    +
    +# MAXMEMORY POLICY: how Redis will select what to remove when maxmemory
    +# is reached. You can select among five behaviors:
    +#
    +# volatile-lru -> remove the key with an expire set using an LRU algorithm
    +# allkeys-lru -> remove any key according to the LRU algorithm
    +# volatile-random -> remove a random key with an expire set
    +# allkeys-random -> remove a random key, any key
    +# volatile-ttl -> remove the key with the nearest expire time (minor TTL)
    +# noeviction -> don't expire at all, just return an error on write operations
    +#
    +# Note: with any of the above policies, Redis will return an error on write
    +#       operations, when there are no suitable keys for eviction.
    +#
    +#       At the date of writing these commands are: set setnx setex append
    +#       incr decr rpush lpush rpushx lpushx linsert lset rpoplpush sadd
    +#       sinter sinterstore sunion sunionstore sdiff sdiffstore zadd zincrby
    +#       zunionstore zinterstore hset hsetnx hmset hincrby incrby decrby
    +#       getset mset msetnx exec sort
    +#
    +# The default is:
    +#
    +# maxmemory-policy noeviction
    +
    +# LRU and minimal TTL algorithms are not precise algorithms but approximated
    +# algorithms (in order to save memory), so you can tune it for speed or
    +# accuracy. For default Redis will check five keys and pick the one that was
    +# used less recently, you can change the sample size using the following
    +# configuration directive.
    +#
    +# The default of 5 produces good enough results. 10 Approximates very closely
    +# true LRU but costs a bit more CPU. 3 is very fast but not very accurate.
    +#
    +# maxmemory-samples 5
    +
    +############################## APPEND ONLY MODE ###############################
    +
    +# By default Redis asynchronously dumps the dataset on disk. This mode is
    +# good enough in many applications, but an issue with the Redis process or
    +# a power outage may result into a few minutes of writes lost (depending on
    +# the configured save points).
    +#
    +# The Append Only File is an alternative persistence mode that provides
    +# much better durability. For instance using the default data fsync policy
    +# (see later in the config file) Redis can lose just one second of writes in a
    +# dramatic event like a server power outage, or a single write if something
    +# wrong with the Redis process itself happens, but the operating system is
    +# still running correctly.
    +#
    +# AOF and RDB persistence can be enabled at the same time without problems.
    +# If the AOF is enabled on startup Redis will load the AOF, that is the file
    +# with the better durability guarantees.
    +#
    +# Please check http://redis.io/topics/persistence for more information.
    +
    +appendonly no
    +
    +# The name of the append only file (default: "appendonly.aof")
    +
    +# appendfilename "appendonly.aof"
    +
    +# The fsync() call tells the Operating System to actually write data on disk
    +# instead of waiting for more data in the output buffer. Some OS will really flush
    +# data on disk, some other OS will just try to do it ASAP.
    +#
    +# Redis supports three different modes:
    +#
    +# no: don't fsync, just let the OS flush the data when it wants. Faster.
    +# always: fsync after every write to the append only log. Slow, Safest.
    +# everysec: fsync only one time every second. Compromise.
    +#
    +# The default is "everysec", as that's usually the right compromise between
    +# speed and data safety. It's up to you to understand if you can relax this to
    +# "no" that will let the operating system flush the output buffer when
    +# it wants, for better performances (but if you can live with the idea of
    +# some data loss consider the default persistence mode that's snapshotting),
    +# or on the contrary, use "always" that's very slow but a bit safer than
    +# everysec.
    +#
    +# More details please check the following article:
    +# http://antirez.com/post/redis-persistence-demystified.html
    +#
    +# If unsure, use "everysec".
    +
    +# appendfsync always
    +appendfsync everysec
    +# appendfsync no
    +
    +# When the AOF fsync policy is set to always or everysec, and a background
    +# saving process (a background save or AOF log background rewriting) is
    +# performing a lot of I/O against the disk, in some Linux configurations
    +# Redis may block too long on the fsync() call. Note that there is no fix for
    +# this currently, as even performing fsync in a different thread will block
    +# our synchronous write(2) call.
    +#
    +# In order to mitigate this problem it's possible to use the following option
    +# that will prevent fsync() from being called in the main process while a
    +# BGSAVE or BGREWRITEAOF is in progress.
    +#
    +# This means that while another child is saving, the durability of Redis is
    +# the same as "appendfsync none". In practical terms, this means that it is
    +# possible to lose up to 30 seconds of log in the worst scenario (with the
    +# default Linux settings).
    +#
    +# If you have latency problems turn this to "yes". Otherwise leave it as
    +# "no" that is the safest pick from the point of view of durability.
    +
    +no-appendfsync-on-rewrite no
    +
    +# Automatic rewrite of the append only file.
    +# Redis is able to automatically rewrite the log file implicitly calling
    +# BGREWRITEAOF when the AOF log size grows by the specified percentage.
    +#
    +# This is how it works: Redis remembers the size of the AOF file after the
    +# latest rewrite (if no rewrite has happened since the restart, the size of
    +# the AOF at startup is used).
    +#
    +# This base size is compared to the current size. If the current size is
    +# bigger than the specified percentage, the rewrite is triggered. Also
    +# you need to specify a minimal size for the AOF file to be rewritten, this
    +# is useful to avoid rewriting the AOF file even if the percentage increase
    +# is reached but it is still pretty small.
    +#
    +# Specify a percentage of zero in order to disable the automatic AOF
    +# rewrite feature.
    +
    +auto-aof-rewrite-percentage 100
    +auto-aof-rewrite-min-size 64mb
    +
    +# An AOF file may be found to be truncated at the end during the Redis
    +# startup process, when the AOF data gets loaded back into memory.
    +# This may happen when the system where Redis is running
    +# crashes, especially when an ext4 filesystem is mounted without the
    +# data=ordered option (however this can't happen when Redis itself
    +# crashes or aborts but the operating system still works correctly).
    +#
    +# Redis can either exit with an error when this happens, or load as much
    +# data as possible (the default now) and start if the AOF file is found
    +# to be truncated at the end. The following option controls this behavior.
    +#
    +# If aof-load-truncated is set to yes, a truncated AOF file is loaded and
    +# the Redis server starts emitting a log to inform the user of the event.
    +# Otherwise if the option is set to no, the server aborts with an error
    +# and refuses to start. When the option is set to no, the user requires
    +# to fix the AOF file using the "redis-check-aof" utility before to restart
    +# the server.
    +#
    +# Note that if the AOF file will be found to be corrupted in the middle
    +# the server will still exit with an error. This option only applies when
    +# Redis will try to read more data from the AOF file but not enough bytes
    +# will be found.
    +# aof-load-truncated yes
    +
    +################################ LUA SCRIPTING  ###############################
    +
    +# Max execution time of a Lua script in milliseconds.
    +#
    +# If the maximum execution time is reached Redis will log that a script is
    +# still in execution after the maximum allowed time and will start to
    +# reply to queries with an error.
    +#
    +# When a long running script exceeds the maximum execution time only the
    +# SCRIPT KILL and SHUTDOWN NOSAVE commands are available. The first can be
    +# used to stop a script that did not yet called write commands. The second
    +# is the only way to shut down the server in the case a write command was
    +# already issued by the script but the user doesn't want to wait for the natural
    +# termination of the script.
    +#
    +# Set it to 0 or a negative value for unlimited execution without warnings.
    +lua-time-limit 5000
    +
    +################################ REDIS CLUSTER  ###############################
    +#
    +# ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
    +# WARNING EXPERIMENTAL: Redis Cluster is considered to be stable code, however
    +# in order to mark it as "mature" we need to wait for a non trivial percentage
    +# of users to deploy it in production.
    +# ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
    +#
    +# Normal Redis instances can't be part of a Redis Cluster; only nodes that are
    +# started as cluster nodes can. In order to start a Redis instance as a
    +# cluster node enable the cluster support uncommenting the following:
    +#
    +# cluster-enabled yes
    +
    +# Every cluster node has a cluster configuration file. This file is not
    +# intended to be edited by hand. It is created and updated by Redis nodes.
    +# Every Redis Cluster node requires a different cluster configuration file.
    +# Make sure that instances running in the same system do not have
    +# overlapping cluster configuration file names.
    +#
    +# cluster-config-file nodes-6379.conf
    +
    +# Cluster node timeout is the amount of milliseconds a node must be unreachable
    +# for it to be considered in failure state.
    +# Most other internal time limits are multiple of the node timeout.
    +#
    +# cluster-node-timeout 15000
    +
    +# A slave of a failing master will avoid to start a failover if its data
    +# looks too old.
    +#
    +# There is no simple way for a slave to actually have a exact measure of
    +# its "data age", so the following two checks are performed:
    +#
    +# 1) If there are multiple slaves able to failover, they exchange messages
    +#    in order to try to give an advantage to the slave with the best
    +#    replication offset (more data from the master processed).
    +#    Slaves will try to get their rank by offset, and apply to the start
    +#    of the failover a delay proportional to their rank.
    +#
    +# 2) Every single slave computes the time of the last interaction with
    +#    its master. This can be the last ping or command received (if the master
    +#    is still in the "connected" state), or the time that elapsed since the
    +#    disconnection with the master (if the replication link is currently down).
    +#    If the last interaction is too old, the slave will not try to failover
    +#    at all.
    +#
    +# The point "2" can be tuned by user. Specifically a slave will not perform
    +# the failover if, since the last interaction with the master, the time
    +# elapsed is greater than:
    +#
    +#   (node-timeout * slave-validity-factor) + repl-ping-slave-period
    +#
    +# So for example if node-timeout is 30 seconds, and the slave-validity-factor
    +# is 10, and assuming a default repl-ping-slave-period of 10 seconds, the
    +# slave will not try to failover if it was not able to talk with the master
    +# for longer than 310 seconds.
    +#
    +# A large slave-validity-factor may allow slaves with too old data to failover
    +# a master, while a too small value may prevent the cluster from being able to
    +# elect a slave at all.
    +#
    +# For maximum availability, it is possible to set the slave-validity-factor
    +# to a value of 0, which means, that slaves will always try to failover the
    +# master regardless of the last time they interacted with the master.
    +# (However they'll always try to apply a delay proportional to their
    +# offset rank).
    +#
    +# Zero is the only value able to guarantee that when all the partitions heal
    +# the cluster will always be able to continue.
    +#
    +# cluster-slave-validity-factor 10
    +
    +# Cluster slaves are able to migrate to orphaned masters, that are masters
    +# that are left without working slaves. This improves the cluster ability
    +# to resist to failures as otherwise an orphaned master can't be failed over
    +# in case of failure if it has no working slaves.
    +#
    +# Slaves migrate to orphaned masters only if there are still at least a
    +# given number of other working slaves for their old master. This number
    +# is the "migration barrier". A migration barrier of 1 means that a slave
    +# will migrate only if there is at least 1 other working slave for its master
    +# and so forth. It usually reflects the number of slaves you want for every
    +# master in your cluster.
    +#
    +# Default is 1 (slaves migrate only if their masters remain with at least
    +# one slave). To disable migration just set it to a very large value.
    +# A value of 0 can be set but is useful only for debugging and dangerous
    +# in production.
    +#
    +# cluster-migration-barrier 1
    +
    +# By default Redis Cluster nodes stop accepting queries if they detect there
    +# is at least an hash slot uncovered (no available node is serving it).
    +# This way if the cluster is partially down (for example a range of hash slots
    +# are no longer covered) all the cluster becomes, eventually, unavailable.
    +# It automatically returns available as soon as all the slots are covered again.
    +#
    +# However sometimes you want the subset of the cluster which is working,
    +# to continue to accept queries for the part of the key space that is still
    +# covered. In order to do so, just set the cluster-require-full-coverage
    +# option to no.
    +#
    +# cluster-require-full-coverage yes
    +
    +# In order to setup your cluster make sure to read the documentation
    +# available at http://redis.io web site.
    +
    +################################## SLOW LOG ###################################
    +
    +# The Redis Slow Log is a system to log queries that exceeded a specified
    +# execution time. The execution time does not include the I/O operations
    +# like talking with the client, sending the reply and so forth,
    +# but just the time needed to actually execute the command (this is the only
    +# stage of command execution where the thread is blocked and can not serve
    +# other requests in the meantime).
    +#
    +# You can configure the slow log with two parameters: one tells Redis
    +# what is the execution time, in microseconds, to exceed in order for the
    +# command to get logged, and the other parameter is the length of the
    +# slow log. When a new command is logged the oldest one is removed from the
    +# queue of logged commands.
    +
    +# The following time is expressed in microseconds, so 1000000 is equivalent
    +# to one second. Note that a negative number disables the slow log, while
    +# a value of zero forces the logging of every command.
    +slowlog-log-slower-than 10000
    +
    +# There is no limit to this length. Just be aware that it will consume memory.
    +# You can reclaim memory used by the slow log with SLOWLOG RESET.
    +slowlog-max-len 128
    +
    +################################ LATENCY MONITOR ##############################
    +
    +# The Redis latency monitoring subsystem samples different operations
    +# at runtime in order to collect data related to possible sources of
    +# latency of a Redis instance.
    +#
    +# Via the LATENCY command this information is available to the user that can
    +# print graphs and obtain reports.
    +#
    +# The system only logs operations that were performed in a time equal or
    +# greater than the amount of milliseconds specified via the
    +# latency-monitor-threshold configuration directive. When its value is set
    +# to zero, the latency monitor is turned off.
    +#
    +# By default latency monitoring is disabled since it is mostly not needed
    +# if you don't have latency issues, and collecting data has a performance
    +# impact, that while very small, can be measured under big load. Latency
    +# monitoring can easily be enabled at runtime using the command
    +# "CONFIG SET latency-monitor-threshold <milliseconds>" if needed.
    +# latency-monitor-threshold 0
    +
    +############################# EVENT NOTIFICATION ##############################
    +
    +# Redis can notify Pub/Sub clients about events happening in the key space.
    +# This feature is documented at http://redis.io/topics/notifications
    +#
    +# For instance if keyspace events notification is enabled, and a client
    +# performs a DEL operation on key "foo" stored in the Database 0, two
    +# messages will be published via Pub/Sub:
    +#
    +# PUBLISH __keyspace@0__:foo del
    +# PUBLISH __keyevent@0__:del foo
    +#
    +# It is possible to select the events that Redis will notify among a set
    +# of classes. Every class is identified by a single character:
    +#
    +#  K     Keyspace events, published with __keyspace@<db>__ prefix.
    +#  E     Keyevent events, published with __keyevent@<db>__ prefix.
    +#  g     Generic commands (non-type specific) like DEL, EXPIRE, RENAME, ...
    +#  $     String commands
    +#  l     List commands
    +#  s     Set commands
    +#  h     Hash commands
    +#  z     Sorted set commands
    +#  x     Expired events (events generated every time a key expires)
    +#  e     Evicted events (events generated when a key is evicted for maxmemory)
    +#  A     Alias for g$lshzxe, so that the "AKE" string means all the events.
    +#
    +#  The "notify-keyspace-events" takes as argument a string that is composed
    +#  of zero or multiple characters. The empty string means that notifications
    +#  are disabled.
    +#
    +#  Example: to enable list and generic events, from the point of view of the
    +#           event name, use:
    +#
    +#  notify-keyspace-events Elg
    +#
    +#  Example 2: to get the stream of the expired keys subscribing to channel
    +#             name __keyevent@0__:expired use:
    +#
    +#  notify-keyspace-events Ex
    +#
    +#  By default all notifications are disabled because most users don't need
    +#  this feature and the feature has some overhead. Note that if you don't
    +#  specify at least one of K or E, no events will be delivered.
    +notify-keyspace-events ""
    +
    +############################### ADVANCED CONFIG ###############################
    +
    +# Hashes are encoded using a memory efficient data structure when they have a
    +# small number of entries, and the biggest entry does not exceed a given
    +# threshold. These thresholds can be configured using the following directives.
    +hash-max-ziplist-entries 512
    +hash-max-ziplist-value 64
    +
    +# Lists are also encoded in a special way to save a lot of space.
    +# The number of entries allowed per internal list node can be specified
    +# as a fixed maximum size or a maximum number of elements.
    +# For a fixed maximum size, use -5 through -1, meaning:
    +# -5: max size: 64 Kb  <-- not recommended for normal workloads
    +# -4: max size: 32 Kb  <-- not recommended
    +# -3: max size: 16 Kb  <-- probably not recommended
    +# -2: max size: 8 Kb   <-- good
    +# -1: max size: 4 Kb   <-- good
    +# Positive numbers mean store up to _exactly_ that number of elements
    +# per list node.
    +# The highest performing option is usually -2 (8 Kb size) or -1 (4 Kb size),
    +# but if your use case is unique, adjust the settings as necessary.
    +# list-max-ziplist-size -2
    +
    +# Lists may also be compressed.
    +# Compress depth is the number of quicklist ziplist nodes from *each* side of
    +# the list to *exclude* from compression.  The head and tail of the list
    +# are always uncompressed for fast push/pop operations.  Settings are:
    +# 0: disable all list compression
    +# 1: depth 1 means "don't start compressing until after 1 node into the list,
    +#    going from either the head or tail"
    +#    So: [head]->node->node->...->node->[tail]
    +#    [head], [tail] will always be uncompressed; inner nodes will compress.
    +# 2: [head]->[next]->node->node->...->node->[prev]->[tail]
    +#    2 here means: don't compress head or head->next or tail->prev or tail,
    +#    but compress all nodes between them.
    +# 3: [head]->[next]->[next]->node->node->...->node->[prev]->[prev]->[tail]
    +# etc.
    +# list-compress-depth 0
    +
    +# Sets have a special encoding in just one case: when a set is composed
    +# of just strings that happen to be integers in radix 10 in the range
    +# of 64 bit signed integers.
    +# The following configuration setting sets the limit in the size of the
    +# set in order to use this special memory saving encoding.
    +set-max-intset-entries 512
    +
    +# Similarly to hashes and lists, sorted sets are also specially encoded in
    +# order to save a lot of space. This encoding is only used when the length and
    +# elements of a sorted set are below the following limits:
    +zset-max-ziplist-entries 128
    +zset-max-ziplist-value 64
    +
    +# HyperLogLog sparse representation bytes limit. The limit includes the
    +# 16 bytes header. When an HyperLogLog using the sparse representation crosses
    +# this limit, it is converted into the dense representation.
    +#
    +# A value greater than 16000 is totally useless, since at that point the
    +# dense representation is more memory efficient.
    +#
    +# The suggested value is ~ 3000 in order to have the benefits of
    +# the space efficient encoding without slowing down too much PFADD,
    +# which is O(N) with the sparse encoding. The value can be raised to
    +# ~ 10000 when CPU is not a concern, but space is, and the data set is
    +# composed of many HyperLogLogs with cardinality in the 0 - 15000 range.
    +# hll-sparse-max-bytes 3000
    +
    +# Active rehashing uses 1 millisecond every 100 milliseconds of CPU time in
    +# order to help rehashing the main Redis hash table (the one mapping top-level
    +# keys to values). The hash table implementation Redis uses (see dict.c)
    +# performs a lazy rehashing: the more operation you run into a hash table
    +# that is rehashing, the more rehashing "steps" are performed, so if the
    +# server is idle the rehashing is never complete and some more memory is used
    +# by the hash table.
    +#
    +# The default is to use this millisecond 10 times every second in order to
    +# actively rehash the main dictionaries, freeing memory when possible.
    +#
    +# If unsure:
    +# use "activerehashing no" if you have hard latency requirements and it is
    +# not a good thing in your environment that Redis can reply from time to time
    +# to queries with 2 milliseconds delay.
    +#
    +# use "activerehashing yes" if you don't have such hard requirements but
    +# want to free memory asap when possible.
    +activerehashing yes
    +
    +# The client output buffer limits can be used to force disconnection of clients
    +# that are not reading data from the server fast enough for some reason (a
    +# common reason is that a Pub/Sub client can't consume messages as fast as the
    +# publisher can produce them).
    +#
    +# The limit can be set differently for the three different classes of clients:
    +#
    +# normal -> normal clients including MONITOR clients
    +# slave  -> slave clients
    +# pubsub -> clients subscribed to at least one pubsub channel or pattern
    +#
    +# The syntax of every client-output-buffer-limit directive is the following:
    +#
    +# client-output-buffer-limit <class> <hard limit> <soft limit> <soft seconds>
    +#
    +# A client is immediately disconnected once the hard limit is reached, or if
    +# the soft limit is reached and remains reached for the specified number of
    +# seconds (continuously).
    +# So for instance if the hard limit is 32 megabytes and the soft limit is
    +# 16 megabytes / 10 seconds, the client will get disconnected immediately
    +# if the size of the output buffers reach 32 megabytes, but will also get
    +# disconnected if the client reaches 16 megabytes and continuously overcomes
    +# the limit for 10 seconds.
    +#
    +# By default normal clients are not limited because they don't receive data
    +# without asking (in a push way), but just after a request, so only
    +# asynchronous clients may create a scenario where data is requested faster
    +# than it can read.
    +#
    +# Instead there is a default limit for pubsub and slave clients, since
    +# subscribers and slaves receive data in a push fashion.
    +#
    +# Both the hard or the soft limit can be disabled by setting them to zero.
    +client-output-buffer-limit normal 0 0 0
    +client-output-buffer-limit slave 256mb 64mb 60
    +client-output-buffer-limit pubsub 32mb 8mb 60
    +
    +# Redis calls an internal function to perform many background tasks, like
    +# closing connections of clients in timeout, purging expired keys that are
    +# never requested, and so forth.
    +#
    +# Not all tasks are performed with the same frequency, but Redis checks for
    +# tasks to perform according to the specified "hz" value.
    +#
    +# By default "hz" is set to 10. Raising the value will use more CPU when
    +# Redis is idle, but at the same time will make Redis more responsive when
    +# there are many keys expiring at the same time, and timeouts may be
    +# handled with more precision.
    +#
    +# The range is between 1 and 500, however a value over 100 is usually not
    +# a good idea. Most users should use the default of 10 and raise this up to
    +# 100 only in environments where very low latency is required.
    +hz 10
    +
    +# When a child rewrites the AOF file, if the following option is enabled
    +# the file will be fsync-ed every 32 MB of data generated. This is useful
    +# in order to commit the file to the disk more incrementally and avoid
    +# big latency spikes.
    +aof-rewrite-incremental-fsync yes
    - change mode from '' to '0644'
    - change owner from '' to 'gitlab-redis'
    - restore selinux security context
  * directory[/opt/gitlab/sv/redis] action create
    - create new directory /opt/gitlab/sv/redis
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * directory[/opt/gitlab/sv/redis/log] action create
    - create new directory /opt/gitlab/sv/redis/log
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * directory[/opt/gitlab/sv/redis/log/main] action create
    - create new directory /opt/gitlab/sv/redis/log/main
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * template[/opt/gitlab/sv/redis/run] action create
    - create new file /opt/gitlab/sv/redis/run
    - update content in file /opt/gitlab/sv/redis/run from none to 535f80
    --- /opt/gitlab/sv/redis/run	2017-07-24 00:13:17.516000000 +0800
    +++ /opt/gitlab/sv/redis/.chef-run20170724-7201-bz93kb	2017-07-24 00:13:17.514000000 +0800
    @@ -1 +1,6 @@
    +#!/bin/sh
    +exec 2>&1
    +
    +umask 077
    +exec chpst -P -U gitlab-redis -u gitlab-redis /opt/gitlab/embedded/bin/redis-server /var/opt/gitlab/redis/redis.conf
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * template[/opt/gitlab/sv/redis/log/run] action create
    - create new file /opt/gitlab/sv/redis/log/run
    - update content in file /opt/gitlab/sv/redis/log/run from none to af1017
    --- /opt/gitlab/sv/redis/log/run	2017-07-24 00:13:17.554000000 +0800
    +++ /opt/gitlab/sv/redis/log/.chef-run20170724-7201-edmrm5	2017-07-24 00:13:17.554000000 +0800
    @@ -1 +1,3 @@
    +#!/bin/sh
    +exec svlogd -tt /var/log/gitlab/redis
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * template[/var/log/gitlab/redis/config] action create
    - create new file /var/log/gitlab/redis/config
    - update content in file /var/log/gitlab/redis/config from none to 623c00
    --- /var/log/gitlab/redis/config	2017-07-24 00:13:17.588000000 +0800
    +++ /var/log/gitlab/redis/.chef-config20170724-7201-zptbkh	2017-07-24 00:13:17.588000000 +0800
    @@ -1 +1,7 @@
    +s209715200
    +n30
    +t86400
    +!gzip
    +
    +
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * ruby_block[reload redis svlogd configuration] action nothing (skipped due to action :nothing)
  * file[/opt/gitlab/sv/redis/down] action delete (up to date)
  * link[/opt/gitlab/init/redis] action create
    - create symlink at /opt/gitlab/init/redis to /opt/gitlab/embedded/bin/sv
  * link[/opt/gitlab/service/redis] action create
    - create symlink at /opt/gitlab/service/redis to /opt/gitlab/sv/redis
  * ruby_block[supervise_redis_sleep] action run
    - execute the ruby block supervise_redis_sleep
  * directory[/opt/gitlab/sv/redis/supervise] action create
    - change mode from '0700' to '0755'
    - restore selinux security context
  * directory[/opt/gitlab/sv/redis/log/supervise] action create
    - change mode from '0700' to '0755'
    - restore selinux security context
  * file[/opt/gitlab/sv/redis/supervise/ok] action touch (skipped due to not_if)
  * file[/opt/gitlab/sv/redis/log/supervise/ok] action touch (skipped due to not_if)
  * file[/opt/gitlab/sv/redis/supervise/control] action touch (skipped due to not_if)
  * file[/opt/gitlab/sv/redis/log/supervise/control] action touch (skipped due to not_if)
  * service[redis] action nothing (skipped due to action :nothing)
  * execute[/opt/gitlab/bin/gitlab-ctl start redis] action run
    [execute] ok: run: redis: (pid 8075) 1s
    - execute /opt/gitlab/bin/gitlab-ctl start redis
Recipe: gitlab::postgresql_user
  * group[Postgresql user and group] action create
    - create group gitlab-psql
  * user[Postgresql user and group] action create
    - create user gitlab-psql
Recipe: gitlab::postgresql
  * directory[/var/opt/gitlab/postgresql] action create
    - create new directory /var/opt/gitlab/postgresql
    - change mode from '' to '0755'
    - change owner from '' to 'gitlab-psql'
    - restore selinux security context
  * directory[/var/opt/gitlab/postgresql/data] action create
    - create new directory /var/opt/gitlab/postgresql/data
    - change mode from '' to '0700'
    - change owner from '' to 'gitlab-psql'
    - restore selinux security context
  * directory[/var/log/gitlab/postgresql] action create
    - create new directory /var/log/gitlab/postgresql
    - change mode from '' to '0700'
    - change owner from '' to 'gitlab-psql'
    - restore selinux security context
  * link[/var/opt/gitlab/postgresql/data] action create (skipped due to not_if)
  * file[/var/opt/gitlab/postgresql/.profile] action create
    - create new file /var/opt/gitlab/postgresql/.profile
    - update content in file /var/opt/gitlab/postgresql/.profile from none to 3b0387
    --- /var/opt/gitlab/postgresql/.profile	2017-07-24 00:13:23.202000000 +0800
    +++ /var/opt/gitlab/postgresql/.chef-.profile20170724-7201-cbnobx	2017-07-24 00:13:23.202000000 +0800
    @@ -1 +1,2 @@
    +PATH=/opt/gitlab/embedded/bin:/opt/gitlab/bin:$PATH
    - change mode from '' to '0600'
    - change owner from '' to 'gitlab-psql'
    - restore selinux security context
  * directory[create /etc/sysctl.d for kernel.shmmax] action create (up to date)
  * file[create /opt/gitlab/embedded/etc/90-omnibus-gitlab-kernel.shmmax.conf kernel.shmmax] action create
    - create new file /opt/gitlab/embedded/etc/90-omnibus-gitlab-kernel.shmmax.conf
    - update content in file /opt/gitlab/embedded/etc/90-omnibus-gitlab-kernel.shmmax.conf from none to 75a195
    --- /opt/gitlab/embedded/etc/90-omnibus-gitlab-kernel.shmmax.conf	2017-07-24 00:13:23.226000000 +0800
    +++ /opt/gitlab/embedded/etc/.chef-90-omnibus-gitlab-kernel.shmmax.conf kernel.shmmax20170724-7201-4ublls	2017-07-24 00:13:23.226000000 +0800
    @@ -1 +1,2 @@
    +kernel.shmmax = 17179869184
    - restore selinux security context
  * execute[load sysctl conf kernel.shmmax] action run
    - execute cat /etc/sysctl.conf /etc/sysctl.d/*.conf  | sysctl -e -p -
  * link[/etc/sysctl.d/90-omnibus-gitlab-kernel.shmmax.conf] action create
    - create symlink at /etc/sysctl.d/90-omnibus-gitlab-kernel.shmmax.conf to /opt/gitlab/embedded/etc/90-omnibus-gitlab-kernel.shmmax.conf
  * file[delete /etc/sysctl.d/90-postgresql.conf kernel.shmmax] action delete (skipped due to only_if)
  * file[delete /etc/sysctl.d/90-unicorn.conf kernel.shmmax] action delete (skipped due to only_if)
  * file[delete /opt/gitlab/embedded/etc/90-omnibus-gitlab.conf kernel.shmmax] action delete (skipped due to only_if)
  * file[delete /etc/sysctl.d/90-omnibus-gitlab.conf kernel.shmmax] action delete (skipped due to only_if)
  * execute[load sysctl conf kernel.shmmax] action nothing (skipped due to action :nothing)
  * directory[create /etc/sysctl.d for kernel.shmall] action create (up to date)
  * file[create /opt/gitlab/embedded/etc/90-omnibus-gitlab-kernel.shmall.conf kernel.shmall] action create
    - create new file /opt/gitlab/embedded/etc/90-omnibus-gitlab-kernel.shmall.conf
    - update content in file /opt/gitlab/embedded/etc/90-omnibus-gitlab-kernel.shmall.conf from none to 6d765d
    --- /opt/gitlab/embedded/etc/90-omnibus-gitlab-kernel.shmall.conf	2017-07-24 00:13:23.284000000 +0800
    +++ /opt/gitlab/embedded/etc/.chef-90-omnibus-gitlab-kernel.shmall.conf kernel.shmall20170724-7201-1oscpd8	2017-07-24 00:13:23.284000000 +0800
    @@ -1 +1,2 @@
    +kernel.shmall = 4194304
    - restore selinux security context
  * execute[load sysctl conf kernel.shmall] action run
    [execute] kernel.shmmax = 17179869184
    - execute cat /etc/sysctl.conf /etc/sysctl.d/*.conf  | sysctl -e -p -
  * link[/etc/sysctl.d/90-omnibus-gitlab-kernel.shmall.conf] action create
    - create symlink at /etc/sysctl.d/90-omnibus-gitlab-kernel.shmall.conf to /opt/gitlab/embedded/etc/90-omnibus-gitlab-kernel.shmall.conf
  * file[delete /etc/sysctl.d/90-postgresql.conf kernel.shmall] action delete (skipped due to only_if)
  * file[delete /etc/sysctl.d/90-unicorn.conf kernel.shmall] action delete (skipped due to only_if)
  * file[delete /opt/gitlab/embedded/etc/90-omnibus-gitlab.conf kernel.shmall] action delete (skipped due to only_if)
  * file[delete /etc/sysctl.d/90-omnibus-gitlab.conf kernel.shmall] action delete (skipped due to only_if)
  * execute[load sysctl conf kernel.shmall] action nothing (skipped due to action :nothing)
  * directory[create /etc/sysctl.d for kernel.sem] action create (up to date)
  * file[create /opt/gitlab/embedded/etc/90-omnibus-gitlab-kernel.sem.conf kernel.sem] action create
    - create new file /opt/gitlab/embedded/etc/90-omnibus-gitlab-kernel.sem.conf
    - update content in file /opt/gitlab/embedded/etc/90-omnibus-gitlab-kernel.sem.conf from none to 09a346
    --- /opt/gitlab/embedded/etc/90-omnibus-gitlab-kernel.sem.conf	2017-07-24 00:13:23.335000000 +0800
    +++ /opt/gitlab/embedded/etc/.chef-90-omnibus-gitlab-kernel.sem.conf kernel.sem20170724-7201-28h8m5	2017-07-24 00:13:23.335000000 +0800
    @@ -1 +1,2 @@
    +kernel.sem = 250 32000 32 262
    - restore selinux security context
  * execute[load sysctl conf kernel.sem] action run
    [execute] kernel.shmall = 4194304
              kernel.shmmax = 17179869184
    - execute cat /etc/sysctl.conf /etc/sysctl.d/*.conf  | sysctl -e -p -
  * link[/etc/sysctl.d/90-omnibus-gitlab-kernel.sem.conf] action create
    - create symlink at /etc/sysctl.d/90-omnibus-gitlab-kernel.sem.conf to /opt/gitlab/embedded/etc/90-omnibus-gitlab-kernel.sem.conf
  * file[delete /etc/sysctl.d/90-postgresql.conf kernel.sem] action delete (skipped due to only_if)
  * file[delete /etc/sysctl.d/90-unicorn.conf kernel.sem] action delete (skipped due to only_if)
  * file[delete /opt/gitlab/embedded/etc/90-omnibus-gitlab.conf kernel.sem] action delete (skipped due to only_if)
  * file[delete /etc/sysctl.d/90-omnibus-gitlab.conf kernel.sem] action delete (skipped due to only_if)
  * execute[load sysctl conf kernel.sem] action nothing (skipped due to action :nothing)
  * execute[/opt/gitlab/embedded/bin/initdb -D /var/opt/gitlab/postgresql/data -E UTF8] action run
    [execute] The files belonging to this database system will be owned by user "gitlab-psql".
              This user must also own the server process.
              
              The database cluster will be initialized with locale "en_US.UTF-8".
              The default text search configuration will be set to "english".
              
              Data page checksums are disabled.
              
              fixing permissions on existing directory /var/opt/gitlab/postgresql/data ... ok
              creating subdirectories ... ok
              selecting default max_connections ... 100
              selecting default shared_buffers ... 128MB
              selecting dynamic shared memory implementation ... posix
              creating configuration files ... ok
              running bootstrap script ... ok
              performing post-bootstrap initialization ... ok
              syncing data to disk ... ok
              
              Success. You can now start the database server using:
              
                  /opt/gitlab/embedded/bin/pg_ctl -D /var/opt/gitlab/postgresql/data -l logfile start
              
              
              WARNING: enabling "trust" authentication for local connections
              You can change this by editing pg_hba.conf or using the option -A, or
              --auth-local and --auth-host, the next time you run initdb.
    - execute /opt/gitlab/embedded/bin/initdb -D /var/opt/gitlab/postgresql/data -E UTF8
  * template[/var/opt/gitlab/postgresql/data/postgresql.conf] action create
    - update content in file /var/opt/gitlab/postgresql/data/postgresql.conf from 6a23e5 to 4afa78
    --- /var/opt/gitlab/postgresql/data/postgresql.conf	2017-07-24 00:13:23.494000000 +0800
    +++ /var/opt/gitlab/postgresql/data/.chef-postgresql.conf20170724-7201-l39zp4	2017-07-24 00:13:24.894000000 +0800
    @@ -1,3 +1,7 @@
    +# This file is managed by gitlab-ctl. Manual changes will be
    +# erased! To change the contents below, edit /etc/gitlab/gitlab.rb
    +# and run `sudo gitlab-ctl reconfigure`.
    +
     # -----------------------------
     # PostgreSQL configuration file
     # -----------------------------
    @@ -27,7 +31,7 @@
     # Memory units:  kB = kilobytes        Time units:  ms  = milliseconds
     #                MB = megabytes                     s   = seconds
     #                GB = gigabytes                     min = minutes
    -#                TB = terabytes                     h   = hours
    +#                                                   h   = hours
     #                                                   d   = days
     
     
    @@ -38,16 +42,16 @@
     # The default values of these variables are driven from the -D command-line
     # option or PGDATA environment variable, represented here as ConfigDir.
     
    -#data_directory = 'ConfigDir'		# use data in another directory
    -					# (change requires restart)
    -#hba_file = 'ConfigDir/pg_hba.conf'	# host-based authentication file
    -					# (change requires restart)
    -#ident_file = 'ConfigDir/pg_ident.conf'	# ident configuration file
    -					# (change requires restart)
    +#data_directory = 'ConfigDir'   # use data in another directory
    +          # (change requires restart)
    +#hba_file = 'ConfigDir/pg_hba.conf' # host-based authentication file
    +          # (change requires restart)
    +#ident_file = 'ConfigDir/pg_ident.conf' # ident configuration file
    +          # (change requires restart)
     
     # If external_pid_file is not explicitly set, no extra PID file is written.
    -#external_pid_file = ''			# write an extra PID file
    -					# (change requires restart)
    +#external_pid_file = '(none)'   # write an extra PID file
    +          # (change requires restart)
     
     
     #------------------------------------------------------------------------------
    @@ -56,52 +60,48 @@
     
     # - Connection Settings -
     
    -#listen_addresses = 'localhost'		# what IP address(es) to listen on;
    -					# comma-separated list of addresses;
    -					# defaults to 'localhost'; use '*' for all
    -					# (change requires restart)
    -#port = 5432				# (change requires restart)
    -max_connections = 100			# (change requires restart)
    -#superuser_reserved_connections = 3	# (change requires restart)
    -#unix_socket_directories = '/tmp'	# comma-separated list of directories
    -					# (change requires restart)
    -#unix_socket_group = ''			# (change requires restart)
    -#unix_socket_permissions = 0777		# begin with 0 to use octal notation
    -					# (change requires restart)
    -#bonjour = off				# advertise server via Bonjour
    -					# (change requires restart)
    -#bonjour_name = ''			# defaults to the computer name
    -					# (change requires restart)
    +listen_addresses = ''    # what IP address(es) to listen on;
    +          # comma-separated list of addresses;
    +          # defaults to 'localhost', '*' = all
    +          # (change requires restart)
    +port = 5432        # (change requires restart)
    +max_connections = 200      # (change requires restart)
    +# Note:  Increasing max_connections costs ~400 bytes of shared memory per
    +# connection slot, plus lock space (see max_locks_per_transaction).
    +#superuser_reserved_connections = 3 # (change requires restart)
    +unix_socket_directories = '/var/opt/gitlab/postgresql'   # (change requires restart)
    +#unix_socket_group = ''     # (change requires restart)
    +#unix_socket_permissions = 0777   # begin with 0 to use octal notation
    +          # (change requires restart)
    +#bonjour = off        # advertise server via Bonjour
    +          # (change requires restart)
    +#bonjour_name = ''      # defaults to the computer name
    +          # (change requires restart)
     
     # - Security and Authentication -
     
    -#authentication_timeout = 1min		# 1s-600s
    -#ssl = off				# (change requires restart)
    -#ssl_ciphers = 'HIGH:MEDIUM:+3DES:!aNULL' # allowed SSL ciphers
    -					# (change requires restart)
    -#ssl_prefer_server_ciphers = on		# (change requires restart)
    -#ssl_ecdh_curve = 'prime256v1'		# (change requires restart)
    -#ssl_cert_file = 'server.crt'		# (change requires restart)
    -#ssl_key_file = 'server.key'		# (change requires restart)
    -#ssl_ca_file = ''			# (change requires restart)
    -#ssl_crl_file = ''			# (change requires restart)
    +#authentication_timeout = 1min    # 1s-600s
    +#ssl = off        # (change requires restart)
    +#ssl_ciphers = 'ALL:!ADH:!LOW:!EXP:!MD5:@STRENGTH'  # allowed SSL ciphers
    +          # (change requires restart)
    +#ssl_renegotiation_limit = 512MB  # amount of data between renegotiations
     #password_encryption = on
     #db_user_namespace = off
    -#row_security = on
     
    -# GSSAPI using Kerberos
    +# Kerberos and GSSAPI
     #krb_server_keyfile = ''
    +#krb_srvname = 'postgres'   # (Kerberos only)
     #krb_caseins_users = off
     
     # - TCP Keepalives -
     # see "man 7 tcp" for details
     
    -#tcp_keepalives_idle = 0		# TCP_KEEPIDLE, in seconds;
    -					# 0 selects the system default
    -#tcp_keepalives_interval = 0		# TCP_KEEPINTVL, in seconds;
    -					# 0 selects the system default
    -#tcp_keepalives_count = 0		# TCP_KEEPCNT;
    -					# 0 selects the system default
    +#tcp_keepalives_idle = 0    # TCP_KEEPIDLE, in seconds;
    +          # 0 selects the system default
    +#tcp_keepalives_interval = 0    # TCP_KEEPINTVL, in seconds;
    +          # 0 selects the system default
    +#tcp_keepalives_count = 0   # TCP_KEEPCNT;
    +          # 0 selects the system default
     
     
     #------------------------------------------------------------------------------
    @@ -110,62 +110,40 @@
     
     # - Memory -
     
    -shared_buffers = 128MB			# min 128kB
    -					# (change requires restart)
    -#huge_pages = try			# on, off, or try
    -					# (change requires restart)
    -#temp_buffers = 8MB			# min 800kB
    -#max_prepared_transactions = 0		# zero disables the feature
    -					# (change requires restart)
    -# Caution: it is not advisable to set max_prepared_transactions nonzero unless
    -# you actively intend to use prepared transactions.
    -#work_mem = 4MB				# min 64kB
    -#maintenance_work_mem = 64MB		# min 1MB
    -#replacement_sort_tuples = 150000	# limits use of replacement selection sort
    -#autovacuum_work_mem = -1		# min 1MB, or -1 to use maintenance_work_mem
    -#max_stack_depth = 2MB			# min 100kB
    -dynamic_shared_memory_type = posix	# the default is the first option
    -					# supported by the operating system:
    -					#   posix
    -					#   sysv
    -					#   windows
    -					#   mmap
    -					# use none to disable dynamic shared memory
    +shared_buffers = 500MB # min 128kB
    +          # (change requires restart)
    +#temp_buffers = 8MB     # min 800kB
    +#max_prepared_transactions = 0    # zero disables the feature
    +          # (change requires restart)
    +# Note:  Increasing max_prepared_transactions costs ~600 bytes of shared memory
    +# per transaction slot, plus lock space (see max_locks_per_transaction).
    +# It is not advisable to set max_prepared_transactions nonzero unless you
    +# actively intend to use prepared transactions.
    +#max_stack_depth = 2MB      # min 100kB
     
    -# - Disk -
    -
    -#temp_file_limit = -1			# limits per-process temp file space
    -					# in kB, or -1 for no limit
    -
     # - Kernel Resource Usage -
     
    -#max_files_per_process = 1000		# min 25
    -					# (change requires restart)
    -#shared_preload_libraries = ''		# (change requires restart)
    +#max_files_per_process = 1000   # min 25
    +          # (change requires restart)
    +shared_preload_libraries = ''    # (change requires restart)
     
     # - Cost-Based Vacuum Delay -
     
    -#vacuum_cost_delay = 0			# 0-100 milliseconds
    -#vacuum_cost_page_hit = 1		# 0-10000 credits
    -#vacuum_cost_page_miss = 10		# 0-10000 credits
    -#vacuum_cost_page_dirty = 20		# 0-10000 credits
    -#vacuum_cost_limit = 200		# 1-10000 credits
    +#vacuum_cost_delay = 0ms    # 0-100 milliseconds
    +#vacuum_cost_page_hit = 1   # 0-10000 credits
    +#vacuum_cost_page_miss = 10   # 0-10000 credits
    +#vacuum_cost_page_dirty = 20    # 0-10000 credits
    +#vacuum_cost_limit = 200    # 1-10000 credits
     
     # - Background Writer -
     
    -#bgwriter_delay = 200ms			# 10-10000ms between rounds
    -#bgwriter_lru_maxpages = 100		# 0-1000 max buffers written/round
    -#bgwriter_lru_multiplier = 2.0		# 0-10.0 multiplier on buffers scanned/round
    -#bgwriter_flush_after = 512kB		# measured in pages, 0 disables
    +#bgwriter_delay = 200ms     # 10-10000ms between rounds
    +#bgwriter_lru_maxpages = 100    # 0-1000 max buffers written/round
    +#bgwriter_lru_multiplier = 2.0    # 0-10.0 multipler on buffers scanned/round
     
     # - Asynchronous Behavior -
     
    -#effective_io_concurrency = 1		# 1-1000; 0 disables prefetching
    -#max_worker_processes = 8		# (change requires restart)
    -#max_parallel_workers_per_gather = 0	# taken from max_worker_processes
    -#old_snapshot_threshold = -1		# 1min-60d; -1 disables; 0 is immediate
    -					# (change requires restart)
    -#backend_flush_after = 0		# measured in pages, 0 disables
    +#effective_io_concurrency = 1   # 1-1000. 0 disables prefetching
     
     
     #------------------------------------------------------------------------------
    @@ -174,103 +152,62 @@
     
     # - Settings -
     
    -#wal_level = minimal			# minimal, replica, or logical
    -					# (change requires restart)
    -#fsync = on				# flush data to disk for crash safety
    -						# (turning this off can cause
    -						# unrecoverable data corruption)
    -#synchronous_commit = on		# synchronization level;
    -					# off, local, remote_write, remote_apply, or on
    -#wal_sync_method = fsync		# the default is the first option
    -					# supported by the operating system:
    -					#   open_datasync
    -					#   fdatasync (default on Linux)
    -					#   fsync
    -					#   fsync_writethrough
    -					#   open_sync
    -#full_page_writes = on			# recover from partial page writes
    -#wal_compression = off			# enable compression of full-page writes
    -#wal_log_hints = off			# also do full page writes of non-critical updates
    -					# (change requires restart)
    -#wal_buffers = -1			# min 32kB, -1 sets based on shared_buffers
    -					# (change requires restart)
    -#wal_writer_delay = 200ms		# 1-10000 milliseconds
    -#wal_writer_flush_after = 1MB		# measured in pages, 0 disables
    +wal_level = minimal
    +          # (change requires restart)
    +#fsync = on       # turns forced synchronization on or off
    +#wal_sync_method = fsync    # the default is the first option
    +          # supported by the operating system:
    +          #   open_datasync
    +          #   fdatasync (default on Linux)
    +          #   fsync
    +          #   fsync_writethrough
    +          #   open_sync
    +#full_page_writes = on      # recover from partial page writes
    +wal_buffers = -1 # -1     # min 32kB, -1 sets based on shared_buffers
    +          # (change requires restart)
    +#wal_writer_delay = 200ms   # 1-10000 milliseconds
     
    -#commit_delay = 0			# range 0-100000, in microseconds
    -#commit_siblings = 5			# range 1-1000
    +#commit_delay = 0     # range 0-100000, in microseconds
    +#commit_siblings = 5      # range 1-1000
     
    -# - Checkpoints -
    +min_wal_size = 80MB
    +max_wal_size = 1GB
     
    -#checkpoint_timeout = 5min		# range 30s-1d
    -#max_wal_size = 1GB
    -#min_wal_size = 80MB
    -#checkpoint_completion_target = 0.5	# checkpoint target duration, 0.0 - 1.0
    -#checkpoint_flush_after = 256kB		# measured in pages, 0 disables
    -#checkpoint_warning = 30s		# 0 disables
    +# The number of replication slots to reserve.
    +max_replication_slots = 0
     
    +
     # - Archiving -
     
    -#archive_mode = off		# enables archiving; off, on, or always
    -				# (change requires restart)
    -#archive_command = ''		# command to use to archive a logfile segment
    -				# placeholders: %p = path of file to archive
    -				#               %f = file name only
    -				# e.g. 'test ! -f /mnt/server/archivedir/%f && cp %p /mnt/server/archivedir/%f'
    -#archive_timeout = 0		# force a logfile segment switch after this
    -				# number of seconds; 0 disables
    +archive_mode = off   # allows archiving to be done
    +        # (change requires restart, also requires 'wal_level' of 'hot_standby' OR 'replica')
     
    -
     #------------------------------------------------------------------------------
     # REPLICATION
     #------------------------------------------------------------------------------
     
    -# - Sending Server(s) -
    -
    -# Set these on the master and on any standby that will send replication data.
    -
    -#max_wal_senders = 0		# max number of walsender processes
    -				# (change requires restart)
    -#wal_keep_segments = 0		# in logfile segments, 16MB each; 0 disables
    -#wal_sender_timeout = 60s	# in milliseconds; 0 disables
    -
    -#max_replication_slots = 0	# max number of replication slots
    -				# (change requires restart)
    -#track_commit_timestamp = off	# collect timestamp of transaction commit
    -				# (change requires restart)
    -
     # - Master Server -
     
    -# These settings are ignored on a standby server.
    +# These settings are ignored on a standby server
     
    -#synchronous_standby_names = ''	# standby servers that provide sync rep
    -				# number of sync standbys and comma-separated list of application_name
    -				# from standby(s); '*' = all
    -#vacuum_defer_cleanup_age = 0	# number of xacts by which cleanup is delayed
    +max_wal_senders = 0
    +        # (change requires restart)
    +#wal_sender_delay = 1s    # walsender cycle time, 1-10000 milliseconds
    +#vacuum_defer_cleanup_age = 0 # number of xacts by which cleanup is delayed
    +#replication_timeout = 60s  # in milliseconds; 0 disables
    +#synchronous_standby_names = '' # standby servers that provide sync rep
    +        # comma-separated list of application_name
    +        # from standby(s); '*' = all
     
     # - Standby Servers -
     
    -# These settings are ignored on a master server.
    +# These settings are ignored on a master server
     
    -#hot_standby = off			# "on" allows queries during recovery
    -					# (change requires restart)
    -#max_standby_archive_delay = 30s	# max delay before canceling queries
    -					# when reading WAL from archive;
    -					# -1 allows indefinite delay
    -#max_standby_streaming_delay = 30s	# max delay before canceling queries
    -					# when reading streaming WAL;
    -					# -1 allows indefinite delay
    -#wal_receiver_status_interval = 10s	# send replies at least this often
    -					# 0 disables
    -#hot_standby_feedback = off		# send info from standby to prevent
    -					# query conflicts
    -#wal_receiver_timeout = 60s		# time that receiver waits for
    -					# communication from master
    -					# in milliseconds; 0 disables
    -#wal_retrieve_retry_interval = 5s	# time to wait before retrying to
    -					# retrieve WAL after a failed attempt
    +hot_standby = off
    +          # (change requires restart)
    +#wal_receiver_status_interval = 10s # send replies at least this often
    +          # 0 disables
     
    -
     #------------------------------------------------------------------------------
     # QUERY TUNING
     #------------------------------------------------------------------------------
    @@ -281,7 +218,6 @@
     #enable_hashagg = on
     #enable_hashjoin = on
     #enable_indexscan = on
    -#enable_indexonlyscan = on
     #enable_material = on
     #enable_mergejoin = on
     #enable_nestloop = on
    @@ -291,35 +227,28 @@
     
     # - Planner Cost Constants -
     
    -#seq_page_cost = 1.0			# measured on an arbitrary scale
    -#random_page_cost = 4.0			# same scale as above
    -#cpu_tuple_cost = 0.01			# same scale as above
    -#cpu_index_tuple_cost = 0.005		# same scale as above
    -#cpu_operator_cost = 0.0025		# same scale as above
    -#parallel_tuple_cost = 0.1		# same scale as above
    -#parallel_setup_cost = 1000.0	# same scale as above
    -#min_parallel_relation_size = 8MB
    -#effective_cache_size = 4GB
    +#cpu_tuple_cost = 0.01      # same scale as above
    +#cpu_index_tuple_cost = 0.005   # same scale as above
    +#cpu_operator_cost = 0.0025   # same scale as above
     
     # - Genetic Query Optimizer -
     
     #geqo = on
     #geqo_threshold = 12
    -#geqo_effort = 5			# range 1-10
    -#geqo_pool_size = 0			# selects default based on effort
    -#geqo_generations = 0			# selects default based on effort
    -#geqo_selection_bias = 2.0		# range 1.5-2.0
    -#geqo_seed = 0.0			# range 0.0-1.0
    +#geqo_effort = 5      # range 1-10
    +#geqo_pool_size = 0     # selects default based on effort
    +#geqo_generations = 0     # selects default based on effort
    +#geqo_selection_bias = 2.0    # range 1.5-2.0
    +#geqo_seed = 0.0      # range 0.0-1.0
     
     # - Other Planner Options -
     
    -#default_statistics_target = 100	# range 1-10000
    -#constraint_exclusion = partition	# on, off, or partition
    -#cursor_tuple_fraction = 0.1		# range 0.0-1.0
    +#default_statistics_target = 100  # range 1-10000
    +#constraint_exclusion = partition # on, off, or partition
    +#cursor_tuple_fraction = 0.1    # range 0.0-1.0
     #from_collapse_limit = 8
    -#join_collapse_limit = 8		# 1 disables collapsing of explicit
    -					# JOIN clauses
    -#force_parallel_mode = off
    +#join_collapse_limit = 8    # 1 disables collapsing of explicit
    +          # JOIN clauses
     
     
     #------------------------------------------------------------------------------
    @@ -328,143 +257,105 @@
     
     # - Where to Log -
     
    -#log_destination = 'stderr'		# Valid values are combinations of
    -					# stderr, csvlog, syslog, and eventlog,
    -					# depending on platform.  csvlog
    -					# requires logging_collector to be on.
    +#log_destination = 'stderr'   # Valid values are combinations of
    +          # stderr, csvlog, syslog, and eventlog,
    +          # depending on platform.  csvlog
    +          # requires logging_collector to be on.
     
     # This is used when logging to stderr:
    -#logging_collector = off		# Enable capturing of stderr and csvlog
    -					# into log files. Required to be on for
    -					# csvlogs.
    -					# (change requires restart)
    +#logging_collector = off    # Enable capturing of stderr and csvlog
    +          # into log files. Required to be on for
    +          # csvlogs.
    +          # (change requires restart)
     
     # These are only used if logging_collector is on:
    -#log_directory = 'pg_log'		# directory where log files are written,
    -					# can be absolute or relative to PGDATA
    -#log_filename = 'postgresql-%Y-%m-%d_%H%M%S.log'	# log file name pattern,
    -					# can include strftime() escapes
    -#log_file_mode = 0600			# creation mode for log files,
    -					# begin with 0 to use octal notation
    -#log_truncate_on_rotation = off		# If on, an existing log file with the
    -					# same name as the new log file will be
    -					# truncated rather than appended to.
    -					# But such truncation only occurs on
    -					# time-driven rotation, not on restarts
    -					# or size-driven rotation.  Default is
    -					# off, meaning append to existing files
    -					# in all cases.
    -#log_rotation_age = 1d			# Automatic rotation of logfiles will
    -					# happen after that time.  0 disables.
    -#log_rotation_size = 10MB		# Automatic rotation of logfiles will
    -					# happen after that much log output.
    -					# 0 disables.
    +#log_directory = 'pg_log'   # directory where log files are written,
    +          # can be absolute or relative to PGDATA
    +#log_filename = 'postgresql-%Y-%m-%d_%H%M%S.log'  # log file name pattern,
    +          # can include strftime() escapes
    +#log_file_mode = 0600     # creation mode for log files,
    +          # begin with 0 to use octal notation
    +#log_truncate_on_rotation = off   # If on, an existing log file with the
    +          # same name as the new log file will be
    +          # truncated rather than appended to.
    +          # But such truncation only occurs on
    +          # time-driven rotation, not on restarts
    +          # or size-driven rotation.  Default is
    +          # off, meaning append to existing files
    +          # in all cases.
    +#log_rotation_age = 1d      # Automatic rotation of logfiles will
    +          # happen after that time.  0 disables.
    +#log_rotation_size = 10MB   # Automatic rotation of logfiles will
    +          # happen after that much log output.
    +          # 0 disables.
     
     # These are relevant when logging to syslog:
     #syslog_facility = 'LOCAL0'
     #syslog_ident = 'postgres'
    -#syslog_sequence_numbers = on
    -#syslog_split_messages = on
     
    -# This is only relevant when logging to eventlog (win32):
    -#event_source = 'PostgreSQL'
    +#silent_mode = off      # Run server silently.
    +          # DO NOT USE without syslog or
    +          # logging_collector
    +          # (change requires restart)
     
    +
     # - When to Log -
     
    -#client_min_messages = notice		# values in order of decreasing detail:
    -					#   debug5
    -					#   debug4
    -					#   debug3
    -					#   debug2
    -					#   debug1
    -					#   log
    -					#   notice
    -					#   warning
    -					#   error
    +#client_min_messages = notice   # values in order of decreasing detail:
    +          #   debug5
    +          #   debug4
    +          #   debug3
    +          #   debug2
    +          #   debug1
    +          #   log
    +          #   notice
    +          #   warning
    +          #   error
     
    -#log_min_messages = warning		# values in order of decreasing detail:
    -					#   debug5
    -					#   debug4
    -					#   debug3
    -					#   debug2
    -					#   debug1
    -					#   info
    -					#   notice
    -					#   warning
    -					#   error
    -					#   log
    -					#   fatal
    -					#   panic
    +#log_min_messages = warning   # values in order of decreasing detail:
    +          #   debug5
    +          #   debug4
    +          #   debug3
    +          #   debug2
    +          #   debug1
    +          #   info
    +          #   notice
    +          #   warning
    +          #   error
    +          #   log
    +          #   fatal
    +          #   panic
     
    -#log_min_error_statement = error	# values in order of decreasing detail:
    -					#   debug5
    -					#   debug4
    -					#   debug3
    -					#   debug2
    -					#   debug1
    -					#   info
    -					#   notice
    -					#   warning
    -					#   error
    -					#   log
    -					#   fatal
    -					#   panic (effectively off)
    +#log_min_error_statement = error  # values in order of decreasing detail:
    +          #   debug5
    +          #   debug4
    +          #   debug3
    +          #   debug2
    +          #   debug1
    +          #   info
    +          #   notice
    +          #   warning
    +          #   error
    +          #   log
    +          #   fatal
    +          #   panic (effectively off)
     
    -#log_min_duration_statement = -1	# -1 is disabled, 0 logs all statements
    -					# and their durations, > 0 logs only
    -					# statements running at least this number
    -					# of milliseconds
    -
    -
     # - What to Log -
     
     #debug_print_parse = off
     #debug_print_rewritten = off
     #debug_print_plan = off
     #debug_pretty_print = on
    -#log_checkpoints = off
     #log_connections = off
     #log_disconnections = off
     #log_duration = off
    -#log_error_verbosity = default		# terse, default, or verbose messages
    +#log_error_verbosity = default    # terse, default, or verbose messages
     #log_hostname = off
    -#log_line_prefix = ''			# special values:
    -					#   %a = application name
    -					#   %u = user name
    -					#   %d = database name
    -					#   %r = remote host and port
    -					#   %h = remote host
    -					#   %p = process ID
    -					#   %t = timestamp without milliseconds
    -					#   %m = timestamp with milliseconds
    -					#   %n = timestamp with milliseconds (as a Unix epoch)
    -					#   %i = command tag
    -					#   %e = SQL state
    -					#   %c = session ID
    -					#   %l = session line number
    -					#   %s = session start timestamp
    -					#   %v = virtual transaction ID
    -					#   %x = transaction ID (0 if none)
    -					#   %q = stop here in non-session
    -					#        processes
    -					#   %% = '%'
    -					# e.g. '<%u%%%d> '
    -#log_lock_waits = off			# log lock waits >= deadlock_timeout
    -#log_statement = 'none'			# none, ddl, mod, all
    -#log_replication_commands = off
    -#log_temp_files = -1			# log temporary files equal or larger
    -					# than the specified size in kilobytes;
    -					# -1 disables, 0 logs all temp files
    -log_timezone = 'PRC'
    +#log_lock_waits = off     # log lock waits >= deadlock_timeout
    +#log_statement = 'none'     # none, ddl, mod, all
    +#log_timezone = '(defaults to server environment setting)'
     
     
    -# - Process Title -
    -
    -#cluster_name = ''			# added to process titles if nonempty
    -					# (change requires restart)
    -#update_process_title = on
    -
    -
     #------------------------------------------------------------------------------
     # RUNTIME STATISTICS
     #------------------------------------------------------------------------------
    @@ -473,9 +364,9 @@
     
     #track_activities = on
     #track_counts = on
    -#track_io_timing = off
    -#track_functions = none			# none, pl, all
    -#track_activity_query_size = 1024	# (change requires restart)
    +#track_functions = none     # none, pl, all
    +track_activity_query_size = 1024 # (change requires restart)
    +#update_process_title = on
     #stats_temp_directory = 'pg_stat_tmp'
     
     
    @@ -491,93 +382,50 @@
     # AUTOVACUUM PARAMETERS
     #------------------------------------------------------------------------------
     
    -#autovacuum = on			# Enable autovacuum subprocess?  'on'
    -					# requires track_counts to also be on.
    -#log_autovacuum_min_duration = -1	# -1 disables, 0 logs all actions and
    -					# their durations, > 0 logs only
    -					# actions running at least this number
    -					# of milliseconds.
    -#autovacuum_max_workers = 3		# max number of autovacuum subprocesses
    -					# (change requires restart)
    -#autovacuum_naptime = 1min		# time between autovacuum runs
    -#autovacuum_vacuum_threshold = 50	# min number of row updates before
    -					# vacuum
    -#autovacuum_analyze_threshold = 50	# min number of row updates before
    -					# analyze
    -#autovacuum_vacuum_scale_factor = 0.2	# fraction of table size before vacuum
    -#autovacuum_analyze_scale_factor = 0.1	# fraction of table size before analyze
    -#autovacuum_freeze_max_age = 200000000	# maximum XID age before forced vacuum
    -					# (change requires restart)
    -#autovacuum_multixact_freeze_max_age = 400000000	# maximum multixact age
    -					# before forced vacuum
    -					# (change requires restart)
    -#autovacuum_vacuum_cost_delay = 20ms	# default vacuum cost delay for
    -					# autovacuum, in milliseconds;
    -					# -1 means use vacuum_cost_delay
    -#autovacuum_vacuum_cost_limit = -1	# default vacuum cost limit for
    -					# autovacuum, -1 means use
    -					# vacuum_cost_limit
    -
    -
    +autovacuum_max_workers = 3 # max number of autovacuum subprocesses
    +          # (change requires restart)
    +autovacuum_freeze_max_age = 200000000  # maximum XID age before forced vacuum
    +          # (change requires restart)
     #------------------------------------------------------------------------------
     # CLIENT CONNECTION DEFAULTS
     #------------------------------------------------------------------------------
     
     # - Statement Behavior -
     
    -#search_path = '"$user", public'	# schema names
    -#default_tablespace = ''		# a tablespace name, '' uses the default
    -#temp_tablespaces = ''			# a list of tablespace names, '' uses
    -					# only default tablespace
    +#search_path = '"$user",public'   # schema names
    +#default_tablespace = ''    # a tablespace name, '' uses the default
    +#temp_tablespaces = ''      # a list of tablespace names, '' uses
    +          # only default tablespace
     #check_function_bodies = on
     #default_transaction_isolation = 'read committed'
     #default_transaction_read_only = off
     #default_transaction_deferrable = off
     #session_replication_role = 'origin'
    -#statement_timeout = 0			# in milliseconds, 0 is disabled
    -#lock_timeout = 0			# in milliseconds, 0 is disabled
    -#idle_in_transaction_session_timeout = 0		# in milliseconds, 0 is disabled
     #vacuum_freeze_min_age = 50000000
     #vacuum_freeze_table_age = 150000000
    -#vacuum_multixact_freeze_min_age = 5000000
    -#vacuum_multixact_freeze_table_age = 150000000
    -#bytea_output = 'hex'			# hex, escape
    +#bytea_output = 'hex'     # hex, escape
     #xmlbinary = 'base64'
     #xmloption = 'content'
    -#gin_fuzzy_search_limit = 0
    -#gin_pending_list_limit = 4MB
     
     # - Locale and Formatting -
     
    -datestyle = 'iso, mdy'
     #intervalstyle = 'postgres'
    -timezone = 'PRC'
    +#timezone = '(defaults to server environment setting)'
     #timezone_abbreviations = 'Default'     # Select the set of available time zone
    -					# abbreviations.  Currently, there are
    -					#   Default
    -					#   Australia (historical usage)
    -					#   India
    -					# You can create your own file in
    -					# share/timezonesets/.
    -#extra_float_digits = 0			# min -15, max 3
    -#client_encoding = sql_ascii		# actually, defaults to database
    -					# encoding
    +          # abbreviations.  Currently, there are
    +          #   Default
    +          #   Australia
    +          #   India
    +          # You can create your own file in
    +          # share/timezonesets/.
    +#extra_float_digits = 0     # min -15, max 3
    +#client_encoding = sql_ascii    # actually, defaults to database
    +          # encoding
     
    -# These settings are initialized by initdb, but they can be changed.
    -lc_messages = 'en_US.UTF-8'			# locale for system error message
    -					# strings
    -lc_monetary = 'en_US.UTF-8'			# locale for monetary formatting
    -lc_numeric = 'en_US.UTF-8'			# locale for number formatting
    -lc_time = 'en_US.UTF-8'				# locale for time formatting
    -
    -# default configuration for text search
    -default_text_search_config = 'pg_catalog.english'
    -
     # - Other Defaults -
     
     #dynamic_library_path = '$libdir'
     #local_preload_libraries = ''
    -#session_preload_libraries = ''
     
     
     #------------------------------------------------------------------------------
    @@ -585,12 +433,14 @@
     #------------------------------------------------------------------------------
     
     #deadlock_timeout = 1s
    -#max_locks_per_transaction = 64		# min 10
    -					# (change requires restart)
    -#max_pred_locks_per_transaction = 64	# min 10
    -					# (change requires restart)
    +max_locks_per_transaction = 128   # min 10
    +          # (change requires restart)
    +# Note:  Each lock table slot uses ~270 bytes of shared memory, and there are
    +# max_locks_per_transaction * (max_connections + max_prepared_transactions)
    +# lock table slots.
    +#max_pred_locks_per_transaction = 64  # min 10
    +          # (change requires restart)
     
    -
     #------------------------------------------------------------------------------
     # VERSION/PLATFORM COMPATIBILITY
     #------------------------------------------------------------------------------
    @@ -598,11 +448,10 @@
     # - Previous PostgreSQL Versions -
     
     #array_nulls = on
    -#backslash_quote = safe_encoding	# on, off, or safe_encoding
    +#backslash_quote = safe_encoding  # on, off, or safe_encoding
     #default_with_oids = off
     #escape_string_warning = on
     #lo_compat_privileges = off
    -#operator_precedence_warning = off
     #quote_all_identifiers = off
     #sql_inheritance = on
     #standard_conforming_strings = on
    @@ -617,26 +466,15 @@
     # ERROR HANDLING
     #------------------------------------------------------------------------------
     
    -#exit_on_error = off			# terminate session on any error?
    -#restart_after_crash = on		# reinitialize after backend crash?
    +#exit_on_error = off        # terminate session on any error?
    +#restart_after_crash = on     # reinitialize after backend crash?
     
     
     #------------------------------------------------------------------------------
    -# CONFIG FILE INCLUDES
    -#------------------------------------------------------------------------------
    -
    -# These options allow settings to be loaded from files other than the
    -# default postgresql.conf.
    -
    -#include_dir = 'conf.d'			# include files ending in '.conf' from
    -					# directory 'conf.d'
    -#include_if_exists = 'exists.conf'	# include file only if it exists
    -#include = 'special.conf'		# include file
    -
    -
    -#------------------------------------------------------------------------------
     # CUSTOMIZED OPTIONS
     #------------------------------------------------------------------------------
     
    -# Add settings for extensions here
    +#custom_variable_classes = ''   # list of custom variable class names
    +
    +include 'runtime.conf'
    - change mode from '0600' to '0644'
    - restore selinux security context
  * template[/var/opt/gitlab/postgresql/data/runtime.conf] action create
    - create new file /var/opt/gitlab/postgresql/data/runtime.conf
    - update content in file /var/opt/gitlab/postgresql/data/runtime.conf from none to 68690f
    --- /var/opt/gitlab/postgresql/data/runtime.conf	2017-07-24 00:13:24.954000000 +0800
    +++ /var/opt/gitlab/postgresql/data/.chef-runtime.conf20170724-7201-1tnsifg	2017-07-24 00:13:24.954000000 +0800
    @@ -1 +1,114 @@
    +# This file is managed by gitlab-ctl. Manual changes will be
    +# erased! To change the contents below, edit /etc/gitlab/gitlab.rb
    +# and run `sudo gitlab-ctl reconfigure`.
    +
    +# Changing variables in this file should only require a reload of PostgreSQL
    +# As the gitlab-psql user, run:
    +# /opt/gitlab/embedded/bin/pg_ctl reload -D /var/opt/gitlab/postgresql/data
    +work_mem = 16MB				# min 64kB
    +maintenance_work_mem = 16MB # 16MB    # min 1MB
    +synchronous_commit = on # synchronization level; on, off, or local
    +synchronous_standby_names = ''
    +
    +# - Checkpoints -
    +min_wal_size = 80MB
    +max_wal_size = 1GB
    +
    +checkpoint_timeout = 5min		# range 30s-1h, default 5min
    +checkpoint_completion_target = 0.9	# checkpoint target duration, 0.0 - 1.0, default 0.5
    +checkpoint_warning = 30s		# 0 disables, default 30s
    +
    +# - Archiving -
    +archive_command = ''   # command to use to archive a logfile segment
    +archive_timeout = 60    # force a logfile segment switch after this
    +        # number of seconds; 0 disables
    +
    +# - Replication
    +wal_keep_segments = 10
    +
    +max_standby_archive_delay = 30s # max delay before canceling queries
    +          # when reading WAL from archive;
    +          # -1 allows indefinite delay
    +max_standby_streaming_delay = 30s # max delay before canceling queries
    +          # when reading streaming WAL;
    +          # -1 allows indefinite delay
    +
    +hot_standby_feedback = off   # send info from standby to prevent
    +          # query conflicts
    +
    +# - Planner Cost Constants -
    +#seq_page_cost = 1.0      # measured on an arbitrary scale
    +random_page_cost = 2.0     # same scale as above
    +
    +effective_cache_size = 1000MB # Default 128MB
    +
    +log_min_duration_statement = -1  # -1 is disabled, 0 logs all statements
    +          # and their durations, > 0 logs only
    +          # statements running at least this number
    +          # of milliseconds
    +
    +log_checkpoints = off
    +
    +log_line_prefix = '' # default '', special values:
    +          #   %a = application name
    +          #   %u = user name
    +          #   %d = database name
    +          #   %r = remote host and port
    +          #   %h = remote host
    +          #   %p = process ID
    +          #   %t = timestamp without milliseconds
    +          #   %m = timestamp with milliseconds
    +          #   %i = command tag
    +          #   %e = SQL state
    +          #   %c = session ID
    +          #   %l = session line number
    +          #   %s = session start timestamp
    +          #   %v = virtual transaction ID
    +          #   %x = transaction ID (0 if none)
    +          #   %q = stop here in non-session
    +          #        processes
    +          #   %% = '%'
    +
    +log_temp_files = -1      # log temporary files equal or larger
    +          # than the specified size in kilobytes;
    +          # -1 disables, 0 logs all temp files
    +
    +# - Autovacuum parameters -
    +autovacuum = on # Enable autovacuum subprocess?  'on'
    +          # requires track_counts to also be on.
    +
    +log_autovacuum_min_duration = -1 # -1 disables, 0 logs all actions and
    +          # their durations, > 0 logs only
    +          # actions running at least this number
    +          # of milliseconds.
    +
    +autovacuum_naptime = 1min # time between autovacuum runs
    +autovacuum_vacuum_threshold = 50 # min number of row updates before
    +          # vacuum
    +autovacuum_analyze_threshold = 50 # min number of row updates before
    +          # analyze
    +autovacuum_vacuum_scale_factor = 0.02 # fraction of table size before vacuum
    +autovacuum_analyze_scale_factor = 0.01 # fraction of table size before analyze
    +autovacuum_vacuum_cost_delay = 20ms # default vacuum cost delay for
    +          # autovacuum, in milliseconds;
    +          # -1 means use vacuum_cost_delay
    +autovacuum_vacuum_cost_limit = -1 # default vacuum cost limit for
    +          # autovacuum, -1 means use
    +          # vacuum_cost_limit
    +
    +# - Client connection timeouts
    +statement_timeout = 60000
    +
    +# - Locale and Formatting -
    +datestyle = 'iso, mdy'
    +
    +# These settings are initialized by initdb, but they can be changed.
    +lc_messages = 'C'     # locale for system error message
    +          # strings
    +lc_monetary = 'C'     # locale for monetary formatting
    +lc_numeric = 'C'      # locale for number formatting
    +lc_time = 'C'       # locale for time formatting
    +
    +# default configuration for text search
    +default_text_search_config = 'pg_catalog.english'
    - change mode from '' to '0644'
    - change owner from '' to 'gitlab-psql'
    - restore selinux security context
  * template[/var/opt/gitlab/postgresql/data/pg_hba.conf] action create
    - update content in file /var/opt/gitlab/postgresql/data/pg_hba.conf from d7d331 to 40e348
    --- /var/opt/gitlab/postgresql/data/pg_hba.conf	2017-07-24 00:13:23.494000000 +0800
    +++ /var/opt/gitlab/postgresql/data/.chef-pg_hba.conf20170724-7201-ygfu4n	2017-07-24 00:13:24.985000000 +0800
    @@ -1,94 +1,74 @@
    +# This file is managed by gitlab-ctl. Manual changes will be
    +# erased! To change the contents below, edit /etc/gitlab/gitlab.rb
    +# and run `sudo gitlab-ctl reconfigure`.
    +
     # PostgreSQL Client Authentication Configuration File
     # ===================================================
     #
    -# Refer to the "Client Authentication" section in the PostgreSQL
    -# documentation for a complete description of this file.  A short
    -# synopsis follows.
    +# Refer to the "Client Authentication" section in the
    +# PostgreSQL documentation for a complete description
    +# of this file.  A short synopsis follows.
     #
     # This file controls: which hosts are allowed to connect, how clients
     # are authenticated, which PostgreSQL user names they can use, which
     # databases they can access.  Records take one of these forms:
     #
    -# local      DATABASE  USER  METHOD  [OPTIONS]
    -# host       DATABASE  USER  ADDRESS  METHOD  [OPTIONS]
    -# hostssl    DATABASE  USER  ADDRESS  METHOD  [OPTIONS]
    -# hostnossl  DATABASE  USER  ADDRESS  METHOD  [OPTIONS]
    +# local      DATABASE  USER  METHOD  [OPTION]
    +# host       DATABASE  USER  CIDR-ADDRESS  METHOD  [OPTION]
    +# hostssl    DATABASE  USER  CIDR-ADDRESS  METHOD  [OPTION]
    +# hostnossl  DATABASE  USER  CIDR-ADDRESS  METHOD  [OPTION]
     #
     # (The uppercase items must be replaced by actual values.)
     #
    -# The first field is the connection type: "local" is a Unix-domain
    -# socket, "host" is either a plain or SSL-encrypted TCP/IP socket,
    -# "hostssl" is an SSL-encrypted TCP/IP socket, and "hostnossl" is a
    -# plain TCP/IP socket.
    +# The first field is the connection type: "local" is a Unix-domain socket,
    +# "host" is either a plain or SSL-encrypted TCP/IP socket, "hostssl" is an
    +# SSL-encrypted TCP/IP socket, and "hostnossl" is a plain TCP/IP socket.
     #
    -# DATABASE can be "all", "sameuser", "samerole", "replication", a
    -# database name, or a comma-separated list thereof. The "all"
    -# keyword does not match "replication". Access to replication
    -# must be enabled in a separate record (see example below).
    +# DATABASE can be "all", "sameuser", "samerole", a database name, or
    +# a comma-separated list thereof.
     #
    -# USER can be "all", a user name, a group name prefixed with "+", or a
    -# comma-separated list thereof.  In both the DATABASE and USER fields
    -# you can also write a file name prefixed with "@" to include names
    -# from a separate file.
    +# USER can be "all", a user name, a group name prefixed with "+", or
    +# a comma-separated list thereof.  In both the DATABASE and USER fields
    +# you can also write a file name prefixed with "@" to include names from
    +# a separate file.
     #
    -# ADDRESS specifies the set of hosts the record matches.  It can be a
    -# host name, or it is made up of an IP address and a CIDR mask that is
    -# an integer (between 0 and 32 (IPv4) or 128 (IPv6) inclusive) that
    -# specifies the number of significant bits in the mask.  A host name
    -# that starts with a dot (.) matches a suffix of the actual host name.
    -# Alternatively, you can write an IP address and netmask in separate
    -# columns to specify the set of hosts.  Instead of a CIDR-address, you
    -# can write "samehost" to match any of the server's own IP addresses,
    -# or "samenet" to match any address in any subnet that the server is
    -# directly connected to.
    +# CIDR-ADDRESS specifies the set of hosts the record matches.
    +# It is made up of an IP address and a CIDR mask that is an integer
    +# (between 0 and 32 (IPv4) or 128 (IPv6) inclusive) that specifies
    +# the number of significant bits in the mask.  Alternatively, you can write
    +# an IP address and netmask in separate columns to specify the set of hosts.
     #
    -# METHOD can be "trust", "reject", "md5", "password", "gss", "sspi",
    -# "ident", "peer", "pam", "ldap", "radius" or "cert".  Note that
    -# "password" sends passwords in clear text; "md5" is preferred since
    -# it sends encrypted passwords.
    +# METHOD can be "trust", "reject", "md5", "crypt", "password", "gss", "sspi",
    +# "krb5", "ident", "pam" or "ldap".  Note that "password" sends passwords
    +# in clear text; "md5" is preferred since it sends encrypted passwords.
     #
    -# OPTIONS are a set of options for the authentication in the format
    -# NAME=VALUE.  The available options depend on the different
    -# authentication methods -- refer to the "Client Authentication"
    -# section in the documentation for a list of which options are
    -# available for which authentication methods.
    +# OPTION is the ident map or the name of the PAM service, depending on METHOD.
     #
    -# Database and user names containing spaces, commas, quotes and other
    -# special characters must be quoted.  Quoting one of the keywords
    -# "all", "sameuser", "samerole" or "replication" makes the name lose
    -# its special character, and just match a database or username with
    -# that name.
    +# Database and user names containing spaces, commas, quotes and other special
    +# characters must be quoted. Quoting one of the keywords "all", "sameuser" or
    +# "samerole" makes the name lose its special character, and just match a
    +# database or username with that name.
     #
     # This file is read on server startup and when the postmaster receives
     # a SIGHUP signal.  If you edit the file on a running system, you have
    -# to SIGHUP the postmaster for the changes to take effect.  You can
    -# use "pg_ctl reload" to do that.
    +# to SIGHUP the postmaster for the changes to take effect.  You can use
    +# "pg_ctl reload" to do that.
     
     # Put your actual configuration here
     # ----------------------------------
     #
     # If you want to allow non-local connections, you need to add more
    -# "host" records.  In that case you will also need to make PostgreSQL
    -# listen on a non-local interface via the listen_addresses
    -# configuration parameter, or via the -i or -h command line switches.
    +# "host" records. In that case you will also need to make PostgreSQL listen
    +# on a non-local interface via the listen_addresses configuration parameter,
    +# or via the -i or -h command line switches.
    +#
     
    -# CAUTION: Configuring the system for local "trust" authentication
    -# allows any local user to connect as any PostgreSQL user, including
    -# the database superuser.  If you do not trust all your local users,
    -# use another authentication method.
     
    +# TYPE  DATABASE    USER        CIDR-ADDRESS          METHOD
     
    -# TYPE  DATABASE        USER            ADDRESS                 METHOD
    -
     # "local" is for Unix domain socket connections only
    -local   all             all                                     trust
    -# IPv4 local connections:
    -host    all             all             127.0.0.1/32            trust
    -# IPv6 local connections:
    -host    all             all             ::1/128                 trust
    -# Allow replication connections from localhost, by a user with the
    -# replication privilege.
    -#local   replication     gitlab-psql                                trust
    -#host    replication     gitlab-psql        127.0.0.1/32            trust
    -#host    replication     gitlab-psql        ::1/128                 trust
    +local   all         all                               peer map=gitlab
    +
    +
    +
    - change mode from '0600' to '0644'
    - restore selinux security context
  * template[/var/opt/gitlab/postgresql/data/pg_ident.conf] action create
    - update content in file /var/opt/gitlab/postgresql/data/pg_ident.conf from 297f46 to 5399a1
    --- /var/opt/gitlab/postgresql/data/pg_ident.conf	2017-07-24 00:13:23.494000000 +0800
    +++ /var/opt/gitlab/postgresql/data/.chef-pg_ident.conf20170724-7201-1v3erle	2017-07-24 00:13:25.019000000 +0800
    @@ -40,4 +40,8 @@
     # ----------------------------------
     
     # MAPNAME       SYSTEM-USERNAME         PG-USERNAME
    +gitlab  git  gitlab
    +gitlab  mattermost  gitlab_mattermost
    +# Default to a 1-1 mapping between system usernames and Postgres usernames
    +gitlab  /^(.*)$  \1
    - change mode from '0600' to '0644'
    - restore selinux security context
  * directory[/opt/gitlab/sv/postgresql] action create
    - create new directory /opt/gitlab/sv/postgresql
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * directory[/opt/gitlab/sv/postgresql/log] action create
    - create new directory /opt/gitlab/sv/postgresql/log
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * directory[/opt/gitlab/sv/postgresql/log/main] action create
    - create new directory /opt/gitlab/sv/postgresql/log/main
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * template[/opt/gitlab/sv/postgresql/run] action create
    - create new file /opt/gitlab/sv/postgresql/run
    - update content in file /opt/gitlab/sv/postgresql/run from none to 870bb6
    --- /opt/gitlab/sv/postgresql/run	2017-07-24 00:13:25.128000000 +0800
    +++ /opt/gitlab/sv/postgresql/.chef-run20170724-7201-cax1ky	2017-07-24 00:13:25.128000000 +0800
    @@ -1 +1,5 @@
    +#!/bin/sh
    +exec 2>&1
    +
    +exec chpst -P -U gitlab-psql -u gitlab-psql /opt/gitlab/embedded/bin/postgres -D /var/opt/gitlab/postgresql/data
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * template[/opt/gitlab/sv/postgresql/log/run] action create
    - create new file /opt/gitlab/sv/postgresql/log/run
    - update content in file /opt/gitlab/sv/postgresql/log/run from none to ce742a
    --- /opt/gitlab/sv/postgresql/log/run	2017-07-24 00:13:25.154000000 +0800
    +++ /opt/gitlab/sv/postgresql/log/.chef-run20170724-7201-1s2s2sb	2017-07-24 00:13:25.154000000 +0800
    @@ -1 +1,3 @@
    +#!/bin/sh
    +exec svlogd -tt /var/log/gitlab/postgresql
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * template[/var/log/gitlab/postgresql/config] action create
    - create new file /var/log/gitlab/postgresql/config
    - update content in file /var/log/gitlab/postgresql/config from none to 623c00
    --- /var/log/gitlab/postgresql/config	2017-07-24 00:13:25.186000000 +0800
    +++ /var/log/gitlab/postgresql/.chef-config20170724-7201-yeuq4	2017-07-24 00:13:25.184000000 +0800
    @@ -1 +1,7 @@
    +s209715200
    +n30
    +t86400
    +!gzip
    +
    +
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * ruby_block[reload postgresql svlogd configuration] action nothing (skipped due to action :nothing)
  * file[/opt/gitlab/sv/postgresql/down] action delete (up to date)
  * directory[/opt/gitlab/sv/postgresql/control] action create
    - create new directory /opt/gitlab/sv/postgresql/control
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * template[/opt/gitlab/sv/postgresql/control/t] action create
    - create new file /opt/gitlab/sv/postgresql/control/t
    - update content in file /opt/gitlab/sv/postgresql/control/t from none to 05ae12
    --- /opt/gitlab/sv/postgresql/control/t	2017-07-24 00:13:25.239000000 +0800
    +++ /opt/gitlab/sv/postgresql/control/.chef-t20170724-7201-16idnqv	2017-07-24 00:13:25.239000000 +0800
    @@ -1 +1,4 @@
    +#!/bin/sh
    +echo "received TERM from runit, sending INT instead to force quit connections"
    +/opt/gitlab/embedded/bin/sv interrupt postgresql
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * link[/opt/gitlab/init/postgresql] action create
    - create symlink at /opt/gitlab/init/postgresql to /opt/gitlab/embedded/bin/sv
  * link[/opt/gitlab/service/postgresql] action create
    - create symlink at /opt/gitlab/service/postgresql to /opt/gitlab/sv/postgresql
  * ruby_block[supervise_postgresql_sleep] action run
    - execute the ruby block supervise_postgresql_sleep
  * directory[/opt/gitlab/sv/postgresql/supervise] action create
    - change mode from '0700' to '0755'
    - restore selinux security context
  * directory[/opt/gitlab/sv/postgresql/log/supervise] action create
    - change mode from '0700' to '0755'
    - restore selinux security context
  * file[/opt/gitlab/sv/postgresql/supervise/ok] action touch
    - create new file /opt/gitlab/sv/postgresql/supervise/ok
    - change owner from '' to 'gitlab-psql'
    - change group from '' to 'gitlab-psql'
    - restore selinux security context
    - update utime on file /opt/gitlab/sv/postgresql/supervise/ok
  * file[/opt/gitlab/sv/postgresql/log/supervise/ok] action touch
    - create new file /opt/gitlab/sv/postgresql/log/supervise/ok
    - change owner from '' to 'gitlab-psql'
    - change group from '' to 'gitlab-psql'
    - restore selinux security context
    - update utime on file /opt/gitlab/sv/postgresql/log/supervise/ok
  * file[/opt/gitlab/sv/postgresql/supervise/control] action touch
    - create new file /opt/gitlab/sv/postgresql/supervise/control
    - change owner from '' to 'gitlab-psql'
    - change group from '' to 'gitlab-psql'
    - restore selinux security context
    - update utime on file /opt/gitlab/sv/postgresql/supervise/control
  * file[/opt/gitlab/sv/postgresql/log/supervise/control] action touch
    - create new file /opt/gitlab/sv/postgresql/log/supervise/control
    - change owner from '' to 'gitlab-psql'
    - change group from '' to 'gitlab-psql'
    - restore selinux security context
    - update utime on file /opt/gitlab/sv/postgresql/log/supervise/control
  * service[postgresql] action nothing (skipped due to action :nothing)
Recipe: gitlab::postgresql-bin
  * ruby_block[Link postgresql bin files to the correct version] action run (skipped due to only_if)
Recipe: gitlab::postgresql
  * execute[/opt/gitlab/bin/gitlab-ctl start postgresql] action run
    [execute] ok: run: postgresql: (pid 8198) 0s
    - execute /opt/gitlab/bin/gitlab-ctl start postgresql
  * template[/opt/gitlab/etc/gitlab-psql-rc] action create
    - create new file /opt/gitlab/etc/gitlab-psql-rc
    - update content in file /opt/gitlab/etc/gitlab-psql-rc from none to eaeb56
    --- /opt/gitlab/etc/gitlab-psql-rc	2017-07-24 00:13:28.807000000 +0800
    +++ /opt/gitlab/etc/.chef-gitlab-psql-rc20170724-7201-rnid5u	2017-07-24 00:13:28.807000000 +0800
    @@ -1 +1,4 @@
    +psql_user='gitlab-psql'
    +psql_host='/var/opt/gitlab/postgresql'
    +psql_port='5432'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * postgresql_user[gitlab] action create
    * execute[create gitlab postgresql user] action run
      [execute] CREATE ROLE
      - execute /opt/gitlab/bin/gitlab-psql -d template1 -c "CREATE USER gitlab "
  
  * execute[create gitlabhq_production database] action run
    - execute /opt/gitlab/embedded/bin/createdb --port 5432 -h /var/opt/gitlab/postgresql -O gitlab gitlabhq_production
  * postgresql_user[gitlab_replicator] action create
    * execute[create gitlab_replicator postgresql user] action run
      [execute] CREATE ROLE
      - execute /opt/gitlab/bin/gitlab-psql -d template1 -c "CREATE USER gitlab_replicator replication"
  
  * execute[enable pg_trgm extension] action nothing (skipped due to action :nothing)
  * execute[reload postgresql] action nothing (skipped due to action :nothing)
  * execute[start postgresql] action nothing (skipped due to action :nothing)
Recipe: gitlab::database_migrations
  * bash[migrate gitlab-rails database] action run
    - Would execute "bash"  "/tmp/chef-script20170724-7201-12ymo6s"
Recipe: gitlab::postgresql
  * execute[enable pg_trgm extension] action run
    [execute] CREATE EXTENSION
    - execute /opt/gitlab/bin/gitlab-psql -d gitlabhq_production -c "CREATE EXTENSION IF NOT EXISTS pg_trgm;"
Recipe: gitlab::database_migrations
  * bash[migrate gitlab-rails database] action run
    [execute] -- enable_extension("plpgsql")
                 -> 0.0087s
              -- enable_extension("pg_trgm")
                 -> 0.0020s
              -- create_table("abuse_reports", {:force=>:cascade})
                 -> 0.0166s
              -- create_table("appearances", {:force=>:cascade})
                 -> 0.0061s
              -- create_table("application_settings", {:force=>:cascade})
                 -> 0.0183s
              -- create_table("audit_events", {:force=>:cascade})
                 -> 0.0060s
              -- add_index("audit_events", ["entity_id", "entity_type"], {:name=>"index_audit_events_on_entity_id_and_entity_type", :using=>:btree})
                 -> 0.0044s
              -- create_table("award_emoji", {:force=>:cascade})
                 -> 0.0077s
              -- add_index("award_emoji", ["awardable_type", "awardable_id"], {:name=>"index_award_emoji_on_awardable_type_and_awardable_id", :using=>:btree})
                 -> 0.0038s
              -- add_index("award_emoji", ["user_id", "name"], {:name=>"index_award_emoji_on_user_id_and_name", :using=>:btree})
                 -> 0.0036s
              -- create_table("boards", {:force=>:cascade})
                 -> 0.0038s
              -- add_index("boards", ["project_id"], {:name=>"index_boards_on_project_id", :using=>:btree})
                 -> 0.0038s
              -- create_table("broadcast_messages", {:force=>:cascade})
                 -> 0.0058s
              -- create_table("chat_names", {:force=>:cascade})
                 -> 0.0056s
              -- add_index("chat_names", ["service_id", "team_id", "chat_id"], {:name=>"index_chat_names_on_service_id_and_team_id_and_chat_id", :unique=>true, :using=>:btree})
                 -> 0.0037s
              -- add_index("chat_names", ["user_id", "service_id"], {:name=>"index_chat_names_on_user_id_and_service_id", :unique=>true, :using=>:btree})
                 -> 0.0035s
              -- create_table("chat_teams", {:force=>:cascade})
                 -> 0.0056s
              -- add_index("chat_teams", ["namespace_id"], {:name=>"index_chat_teams_on_namespace_id", :unique=>true, :using=>:btree})
                 -> 0.0038s
              -- create_table("ci_builds", {:force=>:cascade})
                 -> 0.0061s
              -- add_index("ci_builds", ["auto_canceled_by_id"], {:name=>"index_ci_builds_on_auto_canceled_by_id", :using=>:btree})
                 -> 0.0039s
              -- add_index("ci_builds", ["commit_id", "stage_idx", "created_at"], {:name=>"index_ci_builds_on_commit_id_and_stage_idx_and_created_at", :using=>:btree})
                 -> 0.0038s
              -- add_index("ci_builds", ["commit_id", "status", "type"], {:name=>"index_ci_builds_on_commit_id_and_status_and_type", :using=>:btree})
                 -> 0.0034s
              -- add_index("ci_builds", ["commit_id", "type", "name", "ref"], {:name=>"index_ci_builds_on_commit_id_and_type_and_name_and_ref", :using=>:btree})
                 -> 0.0038s
              -- add_index("ci_builds", ["commit_id", "type", "ref"], {:name=>"index_ci_builds_on_commit_id_and_type_and_ref", :using=>:btree})
                 -> 0.0036s
              -- add_index("ci_builds", ["project_id"], {:name=>"index_ci_builds_on_project_id", :using=>:btree})
                 -> 0.0036s
              -- add_index("ci_builds", ["runner_id"], {:name=>"index_ci_builds_on_runner_id", :using=>:btree})
                 -> 0.0040s
              -- add_index("ci_builds", ["stage_id"], {:name=>"index_ci_builds_on_stage_id", :using=>:btree})
                 -> 0.0035s
              -- add_index("ci_builds", ["status", "type", "runner_id"], {:name=>"index_ci_builds_on_status_and_type_and_runner_id", :using=>:btree})
                 -> 0.0035s
              -- add_index("ci_builds", ["status"], {:name=>"index_ci_builds_on_status", :using=>:btree})
                 -> 0.0038s
              -- add_index("ci_builds", ["token"], {:name=>"index_ci_builds_on_token", :unique=>true, :using=>:btree})
                 -> 0.0038s
              -- add_index("ci_builds", ["updated_at"], {:name=>"index_ci_builds_on_updated_at", :using=>:btree})
                 -> 0.0035s
              -- add_index("ci_builds", ["user_id"], {:name=>"index_ci_builds_on_user_id", :using=>:btree})
                 -> 0.0033s
              -- create_table("ci_pipeline_schedule_variables", {:force=>:cascade})
                 -> 0.0066s
              -- add_index("ci_pipeline_schedule_variables", ["pipeline_schedule_id", "key"], {:name=>"index_ci_pipeline_schedule_variables_on_schedule_id_and_key", :unique=>true, :using=>:btree})
                 -> 0.0036s
              -- create_table("ci_group_variables", {:force=>:cascade})
                 -> 0.0057s
              -- add_index("ci_group_variables", ["group_id", "key"], {:name=>"index_ci_group_variables_on_group_id_and_key", :unique=>true, :using=>:btree})
                 -> 0.0038s
              -- create_table("ci_pipeline_schedules", {:force=>:cascade})
                 -> 0.0055s
              -- add_index("ci_pipeline_schedules", ["next_run_at", "active"], {:name=>"index_ci_pipeline_schedules_on_next_run_at_and_active", :using=>:btree})
                 -> 0.0034s
              -- add_index("ci_pipeline_schedules", ["project_id"], {:name=>"index_ci_pipeline_schedules_on_project_id", :using=>:btree})
                 -> 0.0041s
              -- create_table("ci_pipelines", {:force=>:cascade})
                 -> 0.0055s
              -- add_index("ci_pipelines", ["auto_canceled_by_id"], {:name=>"index_ci_pipelines_on_auto_canceled_by_id", :using=>:btree})
                 -> 0.0044s
              -- add_index("ci_pipelines", ["pipeline_schedule_id"], {:name=>"index_ci_pipelines_on_pipeline_schedule_id", :using=>:btree})
                 -> 0.0037s
              -- add_index("ci_pipelines", ["project_id", "ref", "status"], {:name=>"index_ci_pipelines_on_project_id_and_ref_and_status", :using=>:btree})
                 -> 0.0062s
              -- add_index("ci_pipelines", ["project_id", "sha"], {:name=>"index_ci_pipelines_on_project_id_and_sha", :using=>:btree})
                 -> 0.0044s
              -- add_index("ci_pipelines", ["project_id"], {:name=>"index_ci_pipelines_on_project_id", :using=>:btree})
                 -> 0.0043s
              -- add_index("ci_pipelines", ["status"], {:name=>"index_ci_pipelines_on_status", :using=>:btree})
                 -> 0.0038s
              -- add_index("ci_pipelines", ["user_id"], {:name=>"index_ci_pipelines_on_user_id", :using=>:btree})
                 -> 0.0037s
              -- create_table("ci_runner_projects", {:force=>:cascade})
                 -> 0.0041s
              -- add_index("ci_runner_projects", ["project_id"], {:name=>"index_ci_runner_projects_on_project_id", :using=>:btree})
                 -> 0.0037s
              -- add_index("ci_runner_projects", ["runner_id"], {:name=>"index_ci_runner_projects_on_runner_id", :using=>:btree})
                 -> 0.0039s
              -- create_table("ci_runners", {:force=>:cascade})
                 -> 0.0063s
              -- add_index("ci_runners", ["contacted_at"], {:name=>"index_ci_runners_on_contacted_at", :using=>:btree})
                 -> 0.0037s
              -- add_index("ci_runners", ["is_shared"], {:name=>"index_ci_runners_on_is_shared", :using=>:btree})
                 -> 0.0039s
              -- add_index("ci_runners", ["locked"], {:name=>"index_ci_runners_on_locked", :using=>:btree})
                 -> 0.0039s
              -- add_index("ci_runners", ["token"], {:name=>"index_ci_runners_on_token", :using=>:btree})
                 -> 0.0037s
              -- create_table("ci_stages", {:force=>:cascade})
                 -> 0.0053s
              -- add_index("ci_stages", ["pipeline_id", "name"], {:name=>"index_ci_stages_on_pipeline_id_and_name", :using=>:btree})
                 -> 0.0033s
              -- add_index("ci_stages", ["pipeline_id"], {:name=>"index_ci_stages_on_pipeline_id", :using=>:btree})
                 -> 0.0037s
              -- add_index("ci_stages", ["project_id"], {:name=>"index_ci_stages_on_project_id", :using=>:btree})
                 -> 0.0036s
              -- create_table("ci_trigger_requests", {:force=>:cascade})
                 -> 0.0054s
              -- add_index("ci_trigger_requests", ["commit_id"], {:name=>"index_ci_trigger_requests_on_commit_id", :using=>:btree})
                 -> 0.0035s
              -- create_table("ci_triggers", {:force=>:cascade})
                 -> 0.0055s
              -- add_index("ci_triggers", ["project_id"], {:name=>"index_ci_triggers_on_project_id", :using=>:btree})
                 -> 0.0034s
              -- create_table("ci_variables", {:force=>:cascade})
                 -> 0.0054s
              -- add_index("ci_variables", ["project_id", "key", "environment_scope"], {:name=>"index_ci_variables_on_project_id_and_key_and_environment_scope", :unique=>true, :using=>:btree})
                 -> 0.0034s
              -- create_table("container_repositories", {:force=>:cascade})
                 -> 0.0050s
              -- add_index("container_repositories", ["project_id", "name"], {:name=>"index_container_repositories_on_project_id_and_name", :unique=>true, :using=>:btree})
                 -> 0.0036s
              -- add_index("container_repositories", ["project_id"], {:name=>"index_container_repositories_on_project_id", :using=>:btree})
                 -> 0.0035s
              -- create_table("conversational_development_index_metrics", {:force=>:cascade})
                 -> 0.0048s
              -- create_table("deploy_keys_projects", {:force=>:cascade})
                 -> 0.0037s
              -- add_index("deploy_keys_projects", ["project_id"], {:name=>"index_deploy_keys_projects_on_project_id", :using=>:btree})
                 -> 0.0034s
              -- create_table("deployments", {:force=>:cascade})
                 -> 0.0057s
              -- add_index("deployments", ["created_at"], {:name=>"index_deployments_on_created_at", :using=>:btree})
                 -> 0.0035s
              -- add_index("deployments", ["project_id", "environment_id", "iid"], {:name=>"index_deployments_on_project_id_and_environment_id_and_iid", :using=>:btree})
                 -> 0.0038s
              -- add_index("deployments", ["project_id", "iid"], {:name=>"index_deployments_on_project_id_and_iid", :unique=>true, :using=>:btree})
                 -> 0.0033s
              -- create_table("emails", {:force=>:cascade})
                 -> 0.0052s
              -- add_index("emails", ["email"], {:name=>"index_emails_on_email", :unique=>true, :using=>:btree})
                 -> 0.0039s
              -- add_index("emails", ["user_id"], {:name=>"index_emails_on_user_id", :using=>:btree})
                 -> 0.0034s
              -- create_table("environments", {:force=>:cascade})
                 -> 0.0055s
              -- add_index("environments", ["project_id", "name"], {:name=>"index_environments_on_project_id_and_name", :unique=>true, :using=>:btree})
                 -> 0.0041s
              -- add_index("environments", ["project_id", "slug"], {:name=>"index_environments_on_project_id_and_slug", :unique=>true, :using=>:btree})
                 -> 0.0048s
              -- create_table("events", {:force=>:cascade})
                 -> 0.0056s
              -- add_index("events", ["action"], {:name=>"index_events_on_action", :using=>:btree})
                 -> 0.0040s
              -- add_index("events", ["author_id"], {:name=>"index_events_on_author_id", :using=>:btree})
                 -> 0.0034s
              -- add_index("events", ["created_at"], {:name=>"index_events_on_created_at", :using=>:btree})
                 -> 0.0037s
              -- add_index("events", ["project_id"], {:name=>"index_events_on_project_id", :using=>:btree})
                 -> 0.0038s
              -- add_index("events", ["target_id"], {:name=>"index_events_on_target_id", :using=>:btree})
                 -> 0.0037s
              -- add_index("events", ["target_type"], {:name=>"index_events_on_target_type", :using=>:btree})
                 -> 0.0039s
              -- create_table("feature_gates", {:force=>:cascade})
                 -> 0.0055s
              -- add_index("feature_gates", ["feature_key", "key", "value"], {:name=>"index_feature_gates_on_feature_key_and_key_and_value", :unique=>true, :using=>:btree})
                 -> 0.0036s
              -- create_table("features", {:force=>:cascade})
                 -> 0.0051s
              -- add_index("features", ["key"], {:name=>"index_features_on_key", :unique=>true, :using=>:btree})
                 -> 0.0037s
              -- create_table("forked_project_links", {:force=>:cascade})
                 -> 0.0043s
              -- add_index("forked_project_links", ["forked_to_project_id"], {:name=>"index_forked_project_links_on_forked_to_project_id", :unique=>true, :using=>:btree})
                 -> 0.0037s
              -- create_table("identities", {:force=>:cascade})
                 -> 0.0057s
              -- add_index("identities", ["user_id"], {:name=>"index_identities_on_user_id", :using=>:btree})
                 -> 0.0038s
              -- create_table("issue_assignees", {:id=>false, :force=>:cascade})
                 -> 0.0020s
              -- add_index("issue_assignees", ["issue_id", "user_id"], {:name=>"index_issue_assignees_on_issue_id_and_user_id", :unique=>true, :using=>:btree})
                 -> 0.0038s
              -- add_index("issue_assignees", ["user_id"], {:name=>"index_issue_assignees_on_user_id", :using=>:btree})
                 -> 0.0038s
              -- create_table("issue_metrics", {:force=>:cascade})
                 -> 0.0052s
              -- add_index("issue_metrics", ["issue_id"], {:name=>"index_issue_metrics", :using=>:btree})
                 -> 0.0038s
              -- create_table("issues", {:force=>:cascade})
                 -> 0.0069s
              -- add_index("issues", ["assignee_id"], {:name=>"index_issues_on_assignee_id", :using=>:btree})
                 -> 0.0043s
              -- add_index("issues", ["author_id"], {:name=>"index_issues_on_author_id", :using=>:btree})
                 -> 0.0036s
              -- add_index("issues", ["confidential"], {:name=>"index_issues_on_confidential", :using=>:btree})
                 -> 0.0038s
              -- add_index("issues", ["created_at"], {:name=>"index_issues_on_created_at", :using=>:btree})
                 -> 0.0035s
              -- add_index("issues", ["deleted_at"], {:name=>"index_issues_on_deleted_at", :using=>:btree})
                 -> 0.0034s
              -- add_index("issues", ["description"], {:name=>"index_issues_on_description_trigram", :using=>:gin, :opclasses=>{"description"=>"gin_trgm_ops"}})
                 -> 0.0028s
              -- add_index("issues", ["due_date"], {:name=>"index_issues_on_due_date", :using=>:btree})
                 -> 0.0038s
              -- add_index("issues", ["milestone_id"], {:name=>"index_issues_on_milestone_id", :using=>:btree})
                 -> 0.0038s
              -- add_index("issues", ["project_id", "iid"], {:name=>"index_issues_on_project_id_and_iid", :unique=>true, :using=>:btree})
                 -> 0.0034s
              -- add_index("issues", ["relative_position"], {:name=>"index_issues_on_relative_position", :using=>:btree})
                 -> 0.0041s
              -- add_index("issues", ["state"], {:name=>"index_issues_on_state", :using=>:btree})
                 -> 0.0038s
              -- add_index("issues", ["title"], {:name=>"index_issues_on_title_trigram", :using=>:gin, :opclasses=>{"title"=>"gin_trgm_ops"}})
                 -> 0.0025s
              -- create_table("keys", {:force=>:cascade})
                 -> 0.0058s
              -- add_index("keys", ["fingerprint"], {:name=>"index_keys_on_fingerprint", :unique=>true, :using=>:btree})
                 -> 0.0037s
              -- add_index("keys", ["user_id"], {:name=>"index_keys_on_user_id", :using=>:btree})
                 -> 0.0039s
              -- create_table("label_links", {:force=>:cascade})
                 -> 0.0056s
              -- add_index("label_links", ["label_id"], {:name=>"index_label_links_on_label_id", :using=>:btree})
                 -> 0.0038s
              -- add_index("label_links", ["target_id", "target_type"], {:name=>"index_label_links_on_target_id_and_target_type", :using=>:btree})
                 -> 0.0037s
              -- create_table("label_priorities", {:force=>:cascade})
                 -> 0.0040s
              -- add_index("label_priorities", ["priority"], {:name=>"index_label_priorities_on_priority", :using=>:btree})
                 -> 0.0037s
              -- add_index("label_priorities", ["project_id", "label_id"], {:name=>"index_label_priorities_on_project_id_and_label_id", :unique=>true, :using=>:btree})
                 -> 0.0034s
              -- create_table("labels", {:force=>:cascade})
                 -> 0.0060s
              -- add_index("labels", ["group_id", "project_id", "title"], {:name=>"index_labels_on_group_id_and_project_id_and_title", :unique=>true, :using=>:btree})
                 -> 0.0037s
              -- add_index("labels", ["project_id"], {:name=>"index_labels_on_project_id", :using=>:btree})
                 -> 0.0041s
              -- add_index("labels", ["title"], {:name=>"index_labels_on_title", :using=>:btree})
                 -> 0.0066s
              -- add_index("labels", ["type", "project_id"], {:name=>"index_labels_on_type_and_project_id", :using=>:btree})
                 -> 0.0050s
              -- create_table("lfs_objects", {:force=>:cascade})
                 -> 0.0052s
              -- add_index("lfs_objects", ["oid"], {:name=>"index_lfs_objects_on_oid", :unique=>true, :using=>:btree})
                 -> 0.0034s
              -- create_table("lfs_objects_projects", {:force=>:cascade})
                 -> 0.0035s
              -- add_index("lfs_objects_projects", ["project_id"], {:name=>"index_lfs_objects_projects_on_project_id", :using=>:btree})
                 -> 0.0034s
              -- create_table("lists", {:force=>:cascade})
                 -> 0.0037s
              -- add_index("lists", ["board_id", "label_id"], {:name=>"index_lists_on_board_id_and_label_id", :unique=>true, :using=>:btree})
                 -> 0.0038s
              -- add_index("lists", ["label_id"], {:name=>"index_lists_on_label_id", :using=>:btree})
                 -> 0.0038s
              -- create_table("members", {:force=>:cascade})
                 -> 0.0050s
              -- add_index("members", ["access_level"], {:name=>"index_members_on_access_level", :using=>:btree})
                 -> 0.0037s
              -- add_index("members", ["invite_token"], {:name=>"index_members_on_invite_token", :unique=>true, :using=>:btree})
                 -> 0.0036s
              -- add_index("members", ["requested_at"], {:name=>"index_members_on_requested_at", :using=>:btree})
                 -> 0.0037s
              -- add_index("members", ["source_id", "source_type"], {:name=>"index_members_on_source_id_and_source_type", :using=>:btree})
                 -> 0.0041s
              -- add_index("members", ["user_id"], {:name=>"index_members_on_user_id", :using=>:btree})
                 -> 0.0032s
              -- create_table("merge_request_diff_commits", {:id=>false, :force=>:cascade})
                 -> 0.0038s
              -- add_index("merge_request_diff_commits", ["merge_request_diff_id", "relative_order"], {:name=>"index_merge_request_diff_commits_on_mr_diff_id_and_order", :unique=>true, :using=>:btree})
                 -> 0.0069s
              -- create_table("merge_request_diff_files", {:id=>false, :force=>:cascade})
                 -> 0.0045s
              -- add_index("merge_request_diff_files", ["merge_request_diff_id", "relative_order"], {:name=>"index_merge_request_diff_files_on_mr_diff_id_and_order", :unique=>true, :using=>:btree})
                 -> 0.0042s
              -- create_table("merge_request_diffs", {:force=>:cascade})
                 -> 0.0049s
              -- add_index("merge_request_diffs", ["merge_request_id"], {:name=>"index_merge_request_diffs_on_merge_request_id", :using=>:btree})
                 -> 0.0034s
              -- create_table("merge_request_metrics", {:force=>:cascade})
                 -> 0.0038s
              -- add_index("merge_request_metrics", ["first_deployed_to_production_at"], {:name=>"index_merge_request_metrics_on_first_deployed_to_production_at", :using=>:btree})
                 -> 0.0035s
              -- add_index("merge_request_metrics", ["merge_request_id"], {:name=>"index_merge_request_metrics", :using=>:btree})
                 -> 0.0035s
              -- add_index("merge_request_metrics", ["pipeline_id"], {:name=>"index_merge_request_metrics_on_pipeline_id", :using=>:btree})
                 -> 0.0033s
              -- create_table("merge_requests", {:force=>:cascade})
                 -> 0.0066s
              -- add_index("merge_requests", ["assignee_id"], {:name=>"index_merge_requests_on_assignee_id", :using=>:btree})
                 -> 0.0057s
              -- add_index("merge_requests", ["author_id"], {:name=>"index_merge_requests_on_author_id", :using=>:btree})
                 -> 0.0039s
              -- add_index("merge_requests", ["created_at"], {:name=>"index_merge_requests_on_created_at", :using=>:btree})
                 -> 0.0038s
              -- add_index("merge_requests", ["deleted_at"], {:name=>"index_merge_requests_on_deleted_at", :using=>:btree})
                 -> 0.0048s
              -- add_index("merge_requests", ["description"], {:name=>"index_merge_requests_on_description_trigram", :using=>:gin, :opclasses=>{"description"=>"gin_trgm_ops"}})
                 -> 0.0026s
              -- add_index("merge_requests", ["head_pipeline_id"], {:name=>"index_merge_requests_on_head_pipeline_id", :using=>:btree})
                 -> 0.0034s
              -- add_index("merge_requests", ["milestone_id"], {:name=>"index_merge_requests_on_milestone_id", :using=>:btree})
                 -> 0.0037s
              -- add_index("merge_requests", ["source_branch"], {:name=>"index_merge_requests_on_source_branch", :using=>:btree})
                 -> 0.0035s
              -- add_index("merge_requests", ["source_project_id"], {:name=>"index_merge_requests_on_source_project_id", :using=>:btree})
                 -> 0.0034s
              -- add_index("merge_requests", ["target_branch"], {:name=>"index_merge_requests_on_target_branch", :using=>:btree})
                 -> 0.0036s
              -- add_index("merge_requests", ["target_project_id", "iid"], {:name=>"index_merge_requests_on_target_project_id_and_iid", :unique=>true, :using=>:btree})
                 -> 0.0035s
              -- add_index("merge_requests", ["title"], {:name=>"index_merge_requests_on_title", :using=>:btree})
                 -> 0.0043s
              -- add_index("merge_requests", ["title"], {:name=>"index_merge_requests_on_title_trigram", :using=>:gin, :opclasses=>{"title"=>"gin_trgm_ops"}})
                 -> 0.0028s
              -- create_table("merge_requests_closing_issues", {:force=>:cascade})
                 -> 0.0038s
              -- add_index("merge_requests_closing_issues", ["issue_id"], {:name=>"index_merge_requests_closing_issues_on_issue_id", :using=>:btree})
                 -> 0.0041s
              -- add_index("merge_requests_closing_issues", ["merge_request_id"], {:name=>"index_merge_requests_closing_issues_on_merge_request_id", :using=>:btree})
                 -> 0.0035s
              -- create_table("milestones", {:force=>:cascade})
                 -> 0.0052s
              -- add_index("milestones", ["description"], {:name=>"index_milestones_on_description_trigram", :using=>:gin, :opclasses=>{"description"=>"gin_trgm_ops"}})
                 -> 0.0024s
              -- add_index("milestones", ["due_date"], {:name=>"index_milestones_on_due_date", :using=>:btree})
                 -> 0.0037s
              -- add_index("milestones", ["group_id"], {:name=>"index_milestones_on_group_id", :using=>:btree})
                 -> 0.0038s
              -- add_index("milestones", ["project_id", "iid"], {:name=>"index_milestones_on_project_id_and_iid", :unique=>true, :using=>:btree})
                 -> 0.0038s
              -- add_index("milestones", ["title"], {:name=>"index_milestones_on_title", :using=>:btree})
                 -> 0.0038s
              -- add_index("milestones", ["title"], {:name=>"index_milestones_on_title_trigram", :using=>:gin, :opclasses=>{"title"=>"gin_trgm_ops"}})
                 -> 0.0026s
              -- create_table("namespaces", {:force=>:cascade})
                 -> 0.0068s
              -- add_index("namespaces", ["created_at"], {:name=>"index_namespaces_on_created_at", :using=>:btree})
                 -> 0.0039s
              -- add_index("namespaces", ["deleted_at"], {:name=>"index_namespaces_on_deleted_at", :using=>:btree})
                 -> 0.0040s
              -- add_index("namespaces", ["name", "parent_id"], {:name=>"index_namespaces_on_name_and_parent_id", :unique=>true, :using=>:btree})
                 -> 0.0040s
              -- add_index("namespaces", ["name"], {:name=>"index_namespaces_on_name_trigram", :using=>:gin, :opclasses=>{"name"=>"gin_trgm_ops"}})
                 -> 0.0025s
              -- add_index("namespaces", ["owner_id"], {:name=>"index_namespaces_on_owner_id", :using=>:btree})
                 -> 0.0033s
              -- add_index("namespaces", ["parent_id", "id"], {:name=>"index_namespaces_on_parent_id_and_id", :unique=>true, :using=>:btree})
                 -> 0.0034s
              -- add_index("namespaces", ["path"], {:name=>"index_namespaces_on_path", :using=>:btree})
                 -> 0.0033s
              -- add_index("namespaces", ["path"], {:name=>"index_namespaces_on_path_trigram", :using=>:gin, :opclasses=>{"path"=>"gin_trgm_ops"}})
                 -> 0.0027s
              -- add_index("namespaces", ["require_two_factor_authentication"], {:name=>"index_namespaces_on_require_two_factor_authentication", :using=>:btree})
                 -> 0.0032s
              -- add_index("namespaces", ["type"], {:name=>"index_namespaces_on_type", :using=>:btree})
                 -> 0.0037s
              -- create_table("notes", {:force=>:cascade})
                 -> 0.0057s
              -- add_index("notes", ["author_id"], {:name=>"index_notes_on_author_id", :using=>:btree})
                 -> 0.0040s
              -- add_index("notes", ["commit_id"], {:name=>"index_notes_on_commit_id", :using=>:btree})
                 -> 0.0036s
              -- add_index("notes", ["created_at"], {:name=>"index_notes_on_created_at", :using=>:btree})
                 -> 0.0045s
              -- add_index("notes", ["discussion_id"], {:name=>"index_notes_on_discussion_id", :using=>:btree})
                 -> 0.0040s
              -- add_index("notes", ["line_code"], {:name=>"index_notes_on_line_code", :using=>:btree})
                 -> 0.0041s
              -- add_index("notes", ["note"], {:name=>"index_notes_on_note_trigram", :using=>:gin, :opclasses=>{"note"=>"gin_trgm_ops"}})
                 -> 0.0024s
              -- add_index("notes", ["noteable_id", "noteable_type"], {:name=>"index_notes_on_noteable_id_and_noteable_type", :using=>:btree})
                 -> 0.0032s
              -- add_index("notes", ["noteable_type"], {:name=>"index_notes_on_noteable_type", :using=>:btree})
                 -> 0.0034s
              -- add_index("notes", ["project_id", "noteable_type"], {:name=>"index_notes_on_project_id_and_noteable_type", :using=>:btree})
                 -> 0.0038s
              -- add_index("notes", ["updated_at"], {:name=>"index_notes_on_updated_at", :using=>:btree})
                 -> 0.0039s
              -- create_table("notification_settings", {:force=>:cascade})
                 -> 0.0055s
              -- add_index("notification_settings", ["source_id", "source_type"], {:name=>"index_notification_settings_on_source_id_and_source_type", :using=>:btree})
                 -> 0.0043s
              -- add_index("notification_settings", ["user_id", "source_id", "source_type"], {:name=>"index_notifications_on_user_id_and_source_id_and_source_type", :unique=>true, :using=>:btree})
                 -> 0.0030s
              -- add_index("notification_settings", ["user_id"], {:name=>"index_notification_settings_on_user_id", :using=>:btree})
                 -> 0.0033s
              -- create_table("oauth_access_grants", {:force=>:cascade})
                 -> 0.0042s
              -- add_index("oauth_access_grants", ["token"], {:name=>"index_oauth_access_grants_on_token", :unique=>true, :using=>:btree})
                 -> 0.0034s
              -- create_table("oauth_access_tokens", {:force=>:cascade})
                 -> 0.0053s
              -- add_index("oauth_access_tokens", ["refresh_token"], {:name=>"index_oauth_access_tokens_on_refresh_token", :unique=>true, :using=>:btree})
                 -> 0.0032s
              -- add_index("oauth_access_tokens", ["resource_owner_id"], {:name=>"index_oauth_access_tokens_on_resource_owner_id", :using=>:btree})
                 -> 0.0036s
              -- add_index("oauth_access_tokens", ["token"], {:name=>"index_oauth_access_tokens_on_token", :unique=>true, :using=>:btree})
                 -> 0.0045s
              -- create_table("oauth_applications", {:force=>:cascade})
                 -> 0.0068s
              -- add_index("oauth_applications", ["owner_id", "owner_type"], {:name=>"index_oauth_applications_on_owner_id_and_owner_type", :using=>:btree})
                 -> 0.0042s
              -- add_index("oauth_applications", ["uid"], {:name=>"index_oauth_applications_on_uid", :unique=>true, :using=>:btree})
                 -> 0.0040s
              -- create_table("oauth_openid_requests", {:force=>:cascade})
                 -> 0.0059s
              -- create_table("pages_domains", {:force=>:cascade})
                 -> 0.0057s
              -- add_index("pages_domains", ["domain"], {:name=>"index_pages_domains_on_domain", :unique=>true, :using=>:btree})
                 -> 0.0033s
              -- add_index("pages_domains", ["project_id"], {:name=>"index_pages_domains_on_project_id", :using=>:btree})
                 -> 0.0034s
              -- create_table("personal_access_tokens", {:force=>:cascade})
                 -> 0.0070s
              -- add_index("personal_access_tokens", ["token"], {:name=>"index_personal_access_tokens_on_token", :unique=>true, :using=>:btree})
                 -> 0.0049s
              -- add_index("personal_access_tokens", ["user_id"], {:name=>"index_personal_access_tokens_on_user_id", :using=>:btree})
                 -> 0.0030s
              -- create_table("project_authorizations", {:id=>false, :force=>:cascade})
                 -> 0.0017s
              -- add_index("project_authorizations", ["project_id"], {:name=>"index_project_authorizations_on_project_id", :using=>:btree})
                 -> 0.0032s
              -- add_index("project_authorizations", ["user_id", "project_id", "access_level"], {:name=>"index_project_authorizations_on_user_id_project_id_access_level", :unique=>true, :using=>:btree})
                 -> 0.0037s
              -- create_table("project_features", {:force=>:cascade})
                 -> 0.0040s
              -- add_index("project_features", ["project_id"], {:name=>"index_project_features_on_project_id", :using=>:btree})
                 -> 0.0035s
              -- create_table("project_group_links", {:force=>:cascade})
                 -> 0.0037s
              -- add_index("project_group_links", ["group_id"], {:name=>"index_project_group_links_on_group_id", :using=>:btree})
                 -> 0.0042s
              -- add_index("project_group_links", ["project_id"], {:name=>"index_project_group_links_on_project_id", :using=>:btree})
                 -> 0.0033s
              -- create_table("project_import_data", {:force=>:cascade})
                 -> 0.0048s
              -- add_index("project_import_data", ["project_id"], {:name=>"index_project_import_data_on_project_id", :using=>:btree})
                 -> 0.0030s
              -- create_table("project_statistics", {:force=>:cascade})
                 -> 0.0048s
              -- add_index("project_statistics", ["namespace_id"], {:name=>"index_project_statistics_on_namespace_id", :using=>:btree})
                 -> 0.0032s
              -- add_index("project_statistics", ["project_id"], {:name=>"index_project_statistics_on_project_id", :unique=>true, :using=>:btree})
                 -> 0.0028s
              -- create_table("projects", {:force=>:cascade})
                 -> 0.0087s
              -- add_index("projects", ["ci_id"], {:name=>"index_projects_on_ci_id", :using=>:btree})
                 -> 0.0032s
              -- add_index("projects", ["created_at"], {:name=>"index_projects_on_created_at", :using=>:btree})
                 -> 0.0045s
              -- add_index("projects", ["creator_id"], {:name=>"index_projects_on_creator_id", :using=>:btree})
                 -> 0.0038s
              -- add_index("projects", ["description"], {:name=>"index_projects_on_description_trigram", :using=>:gin, :opclasses=>{"description"=>"gin_trgm_ops"}})
                 -> 0.0027s
              -- add_index("projects", ["last_activity_at"], {:name=>"index_projects_on_last_activity_at", :using=>:btree})
                 -> 0.0031s
              -- add_index("projects", ["last_repository_check_failed"], {:name=>"index_projects_on_last_repository_check_failed", :using=>:btree})
                 -> 0.0032s
              -- add_index("projects", ["last_repository_updated_at"], {:name=>"index_projects_on_last_repository_updated_at", :using=>:btree})
                 -> 0.0030s
              -- add_index("projects", ["name"], {:name=>"index_projects_on_name_trigram", :using=>:gin, :opclasses=>{"name"=>"gin_trgm_ops"}})
                 -> 0.0026s
              -- add_index("projects", ["namespace_id"], {:name=>"index_projects_on_namespace_id", :using=>:btree})
                 -> 0.0029s
              -- add_index("projects", ["path"], {:name=>"index_projects_on_path", :using=>:btree})
                 -> 0.0034s
              -- add_index("projects", ["path"], {:name=>"index_projects_on_path_trigram", :using=>:gin, :opclasses=>{"path"=>"gin_trgm_ops"}})
                 -> 0.0026s
              -- add_index("projects", ["pending_delete"], {:name=>"index_projects_on_pending_delete", :using=>:btree})
                 -> 0.0033s
              -- add_index("projects", ["runners_token"], {:name=>"index_projects_on_runners_token", :using=>:btree})
                 -> 0.0038s
              -- add_index("projects", ["star_count"], {:name=>"index_projects_on_star_count", :using=>:btree})
                 -> 0.0031s
              -- add_index("projects", ["visibility_level"], {:name=>"index_projects_on_visibility_level", :using=>:btree})
                 -> 0.0035s
              -- create_table("protected_branch_merge_access_levels", {:force=>:cascade})
                 -> 0.0041s
              -- add_index("protected_branch_merge_access_levels", ["protected_branch_id"], {:name=>"index_protected_branch_merge_access", :using=>:btree})
                 -> 0.0033s
              -- create_table("protected_branch_push_access_levels", {:force=>:cascade})
                 -> 0.0043s
              -- add_index("protected_branch_push_access_levels", ["protected_branch_id"], {:name=>"index_protected_branch_push_access", :using=>:btree})
                 -> 0.0036s
              -- create_table("protected_branches", {:force=>:cascade})
                 -> 0.0045s
              -- add_index("protected_branches", ["project_id"], {:name=>"index_protected_branches_on_project_id", :using=>:btree})
                 -> 0.0039s
              -- create_table("protected_tag_create_access_levels", {:force=>:cascade})
                 -> 0.0036s
              -- add_index("protected_tag_create_access_levels", ["protected_tag_id"], {:name=>"index_protected_tag_create_access", :using=>:btree})
                 -> 0.0036s
              -- add_index("protected_tag_create_access_levels", ["user_id"], {:name=>"index_protected_tag_create_access_levels_on_user_id", :using=>:btree})
                 -> 0.0070s
              -- create_table("protected_tags", {:force=>:cascade})
                 -> 0.0069s
              -- add_index("protected_tags", ["project_id"], {:name=>"index_protected_tags_on_project_id", :using=>:btree})
                 -> 0.0037s
              -- create_table("redirect_routes", {:force=>:cascade})
                 -> 0.0061s
              -- add_index("redirect_routes", ["path"], {:name=>"index_redirect_routes_on_path", :unique=>true, :using=>:btree})
                 -> 0.0067s
              -- add_index("redirect_routes", ["path"], {:name=>"index_redirect_routes_on_path_text_pattern_ops", :using=>:btree, :opclasses=>{"path"=>"varchar_pattern_ops"}})
                 -> 0.0042s
              -- add_index("redirect_routes", ["source_type", "source_id"], {:name=>"index_redirect_routes_on_source_type_and_source_id", :using=>:btree})
                 -> 0.0073s
              -- create_table("releases", {:force=>:cascade})
                 -> 0.0085s
              -- add_index("releases", ["project_id", "tag"], {:name=>"index_releases_on_project_id_and_tag", :using=>:btree})
                 -> 0.0037s
              -- add_index("releases", ["project_id"], {:name=>"index_releases_on_project_id", :using=>:btree})
                 -> 0.0042s
              -- create_table("routes", {:force=>:cascade})
                 -> 0.0051s
              -- add_index("routes", ["path"], {:name=>"index_routes_on_path", :unique=>true, :using=>:btree})
                 -> 0.0035s
              -- add_index("routes", ["path"], {:name=>"index_routes_on_path_text_pattern_ops", :using=>:btree, :opclasses=>{"path"=>"varchar_pattern_ops"}})
                 -> 0.0034s
              -- add_index("routes", ["source_type", "source_id"], {:name=>"index_routes_on_source_type_and_source_id", :unique=>true, :using=>:btree})
                 -> 0.0045s
              -- create_table("sent_notifications", {:force=>:cascade})
                 -> 0.0055s
              -- add_index("sent_notifications", ["reply_key"], {:name=>"index_sent_notifications_on_reply_key", :unique=>true, :using=>:btree})
                 -> 0.0034s
              -- create_table("services", {:force=>:cascade})
                 -> 0.0119s
              -- add_index("services", ["project_id"], {:name=>"index_services_on_project_id", :using=>:btree})
                 -> 0.0066s
              -- add_index("services", ["template"], {:name=>"index_services_on_template", :using=>:btree})
                 -> 0.0052s
              -- create_table("snippets", {:force=>:cascade})
                 -> 0.0062s
              -- add_index("snippets", ["author_id"], {:name=>"index_snippets_on_author_id", :using=>:btree})
                 -> 0.0040s
              -- add_index("snippets", ["file_name"], {:name=>"index_snippets_on_file_name_trigram", :using=>:gin, :opclasses=>{"file_name"=>"gin_trgm_ops"}})
                 -> 0.0026s
              -- add_index("snippets", ["project_id"], {:name=>"index_snippets_on_project_id", :using=>:btree})
                 -> 0.0034s
              -- add_index("snippets", ["title"], {:name=>"index_snippets_on_title_trigram", :using=>:gin, :opclasses=>{"title"=>"gin_trgm_ops"}})
                 -> 0.0027s
              -- add_index("snippets", ["updated_at"], {:name=>"index_snippets_on_updated_at", :using=>:btree})
                 -> 0.0035s
              -- add_index("snippets", ["visibility_level"], {:name=>"index_snippets_on_visibility_level", :using=>:btree})
                 -> 0.0031s
              -- create_table("spam_logs", {:force=>:cascade})
                 -> 0.0057s
              -- create_table("subscriptions", {:force=>:cascade})
                 -> 0.0055s
              -- add_index("subscriptions", ["subscribable_id", "subscribable_type", "user_id", "project_id"], {:name=>"index_subscriptions_on_subscribable_and_user_id_and_project_id", :unique=>true, :using=>:btree})
                 -> 0.0040s
              -- create_table("system_note_metadata", {:force=>:cascade})
                 -> 0.0055s
              -- add_index("system_note_metadata", ["note_id"], {:name=>"index_system_note_metadata_on_note_id", :unique=>true, :using=>:btree})
                 -> 0.0034s
              -- create_table("taggings", {:force=>:cascade})
                 -> 0.0049s
              -- add_index("taggings", ["tag_id", "taggable_id", "taggable_type", "context", "tagger_id", "tagger_type"], {:name=>"taggings_idx", :unique=>true, :using=>:btree})
                 -> 0.0035s
              -- add_index("taggings", ["taggable_id", "taggable_type", "context"], {:name=>"index_taggings_on_taggable_id_and_taggable_type_and_context", :using=>:btree})
                 -> 0.0033s
              -- create_table("tags", {:force=>:cascade})
                 -> 0.0054s
              -- add_index("tags", ["name"], {:name=>"index_tags_on_name", :unique=>true, :using=>:btree})
                 -> 0.0034s
              -- create_table("timelogs", {:force=>:cascade})
                 -> 0.0038s
              -- add_index("timelogs", ["issue_id"], {:name=>"index_timelogs_on_issue_id", :using=>:btree})
                 -> 0.0038s
              -- add_index("timelogs", ["merge_request_id"], {:name=>"index_timelogs_on_merge_request_id", :using=>:btree})
                 -> 0.0039s
              -- add_index("timelogs", ["user_id"], {:name=>"index_timelogs_on_user_id", :using=>:btree})
                 -> 0.0043s
              -- create_table("todos", {:force=>:cascade})
                 -> 0.0051s
              -- add_index("todos", ["author_id"], {:name=>"index_todos_on_author_id", :using=>:btree})
                 -> 0.0028s
              -- add_index("todos", ["commit_id"], {:name=>"index_todos_on_commit_id", :using=>:btree})
                 -> 0.0033s
              -- add_index("todos", ["note_id"], {:name=>"index_todos_on_note_id", :using=>:btree})
                 -> 0.0032s
              -- add_index("todos", ["project_id"], {:name=>"index_todos_on_project_id", :using=>:btree})
                 -> 0.0032s
              -- add_index("todos", ["target_type", "target_id"], {:name=>"index_todos_on_target_type_and_target_id", :using=>:btree})
                 -> 0.0028s
              -- add_index("todos", ["user_id"], {:name=>"index_todos_on_user_id", :using=>:btree})
                 -> 0.0032s
              -- create_table("trending_projects", {:force=>:cascade})
                 -> 0.0036s
              -- add_index("trending_projects", ["project_id"], {:name=>"index_trending_projects_on_project_id", :using=>:btree})
                 -> 0.0041s
              -- create_table("u2f_registrations", {:force=>:cascade})
                 -> 0.0051s
              -- add_index("u2f_registrations", ["key_handle"], {:name=>"index_u2f_registrations_on_key_handle", :using=>:btree})
                 -> 0.0036s
              -- add_index("u2f_registrations", ["user_id"], {:name=>"index_u2f_registrations_on_user_id", :using=>:btree})
                 -> 0.0028s
              -- create_table("uploads", {:force=>:cascade})
                 -> 0.0046s
              -- add_index("uploads", ["checksum"], {:name=>"index_uploads_on_checksum", :using=>:btree})
                 -> 0.0032s
              -- add_index("uploads", ["model_id", "model_type"], {:name=>"index_uploads_on_model_id_and_model_type", :using=>:btree})
                 -> 0.0040s
              -- add_index("uploads", ["path"], {:name=>"index_uploads_on_path", :using=>:btree})
                 -> 0.0034s
              -- create_table("user_agent_details", {:force=>:cascade})
                 -> 0.0052s
              -- add_index("user_agent_details", ["subject_id", "subject_type"], {:name=>"index_user_agent_details_on_subject_id_and_subject_type", :using=>:btree})
                 -> 0.0038s
              -- create_table("users", {:force=>:cascade})
                 -> 0.0127s
              -- add_index("users", ["admin"], {:name=>"index_users_on_admin", :using=>:btree})
                 -> 0.0037s
              -- add_index("users", ["authentication_token"], {:name=>"index_users_on_authentication_token", :unique=>true, :using=>:btree})
                 -> 0.0040s
              -- add_index("users", ["confirmation_token"], {:name=>"index_users_on_confirmation_token", :unique=>true, :using=>:btree})
                 -> 0.0037s
              -- add_index("users", ["created_at"], {:name=>"index_users_on_created_at", :using=>:btree})
                 -> 0.0039s
              -- add_index("users", ["email"], {:name=>"index_users_on_email", :unique=>true, :using=>:btree})
                 -> 0.0037s
              -- add_index("users", ["email"], {:name=>"index_users_on_email_trigram", :using=>:gin, :opclasses=>{"email"=>"gin_trgm_ops"}})
                 -> 0.0029s
              -- add_index("users", ["ghost"], {:name=>"index_users_on_ghost", :using=>:btree})
                 -> 0.0036s
              -- add_index("users", ["incoming_email_token"], {:name=>"index_users_on_incoming_email_token", :using=>:btree})
                 -> 0.0035s
              -- add_index("users", ["name"], {:name=>"index_users_on_name", :using=>:btree})
                 -> 0.0039s
              -- add_index("users", ["name"], {:name=>"index_users_on_name_trigram", :using=>:gin, :opclasses=>{"name"=>"gin_trgm_ops"}})
                 -> 0.0026s
              -- add_index("users", ["reset_password_token"], {:name=>"index_users_on_reset_password_token", :unique=>true, :using=>:btree})
                 -> 0.0034s
              -- add_index("users", ["rss_token"], {:name=>"index_users_on_rss_token", :using=>:btree})
                 -> 0.0032s
              -- add_index("users", ["state"], {:name=>"index_users_on_state", :using=>:btree})
                 -> 0.0037s
              -- add_index("users", ["username"], {:name=>"index_users_on_username", :using=>:btree})
                 -> 0.0037s
              -- add_index("users", ["username"], {:name=>"index_users_on_username_trigram", :using=>:gin, :opclasses=>{"username"=>"gin_trgm_ops"}})
                 -> 0.0025s
              -- create_table("users_star_projects", {:force=>:cascade})
                 -> 0.0038s
              -- add_index("users_star_projects", ["project_id"], {:name=>"index_users_star_projects_on_project_id", :using=>:btree})
                 -> 0.0035s
              -- add_index("users_star_projects", ["user_id", "project_id"], {:name=>"index_users_star_projects_on_user_id_and_project_id", :unique=>true, :using=>:btree})
                 -> 0.0041s
              -- create_table("web_hook_logs", {:force=>:cascade})
                 -> 0.0054s
              -- add_index("web_hook_logs", ["web_hook_id"], {:name=>"index_web_hook_logs_on_web_hook_id", :using=>:btree})
                 -> 0.0037s
              -- create_table("web_hooks", {:force=>:cascade})
                 -> 0.0076s
              -- add_index("web_hooks", ["project_id"], {:name=>"index_web_hooks_on_project_id", :using=>:btree})
                 -> 0.0036s
              -- add_index("web_hooks", ["type"], {:name=>"index_web_hooks_on_type", :using=>:btree})
                 -> 0.0037s
              -- add_foreign_key("boards", "projects", {:name=>"fk_f15266b5f9", :on_delete=>:cascade})
                 -> 0.0049s
              -- add_foreign_key("chat_teams", "namespaces", {:on_delete=>:cascade})
                 -> 0.0018s
              -- add_foreign_key("ci_builds", "ci_pipelines", {:column=>"auto_canceled_by_id", :name=>"fk_a2141b1522", :on_delete=>:nullify})
                 -> 0.0026s
              -- add_foreign_key("ci_builds", "ci_stages", {:column=>"stage_id", :name=>"fk_3a9eaa254d", :on_delete=>:cascade})
                 -> 0.0020s
              -- add_foreign_key("ci_builds", "projects", {:name=>"fk_befce0568a", :on_delete=>:cascade})
                 -> 0.0022s
              -- add_foreign_key("ci_pipeline_schedule_variables", "ci_pipeline_schedules", {:column=>"pipeline_schedule_id", :name=>"fk_41c35fda51", :on_delete=>:cascade})
                 -> 0.0020s
              -- add_foreign_key("ci_group_variables", "namespaces", {:column=>"group_id", :name=>"fk_33ae4d58d8", :on_delete=>:cascade})
                 -> 0.0019s
              -- add_foreign_key("ci_pipeline_schedules", "projects", {:name=>"fk_8ead60fcc4", :on_delete=>:cascade})
                 -> 0.0019s
              -- add_foreign_key("ci_pipeline_schedules", "users", {:column=>"owner_id", :name=>"fk_9ea99f58d2", :on_delete=>:nullify})
                 -> 0.0024s
              -- add_foreign_key("ci_pipelines", "ci_pipeline_schedules", {:column=>"pipeline_schedule_id", :name=>"fk_3d34ab2e06", :on_delete=>:nullify})
                 -> 0.0019s
              -- add_foreign_key("ci_pipelines", "ci_pipelines", {:column=>"auto_canceled_by_id", :name=>"fk_262d4c2d19", :on_delete=>:nullify})
                 -> 0.0013s
              -- add_foreign_key("ci_pipelines", "projects", {:name=>"fk_86635dbd80", :on_delete=>:cascade})
                 -> 0.0021s
              -- add_foreign_key("ci_runner_projects", "projects", {:name=>"fk_4478a6f1e4", :on_delete=>:cascade})
                 -> 0.0020s
              -- add_foreign_key("ci_stages", "ci_pipelines", {:column=>"pipeline_id", :name=>"fk_fb57e6cc56", :on_delete=>:cascade})
                 -> 0.0020s
              -- add_foreign_key("ci_stages", "projects", {:name=>"fk_2360681d1d", :on_delete=>:cascade})
                 -> 0.0020s
              -- add_foreign_key("ci_trigger_requests", "ci_triggers", {:column=>"trigger_id", :name=>"fk_b8ec8b7245", :on_delete=>:cascade})
                 -> 0.0021s
              -- add_foreign_key("ci_triggers", "projects", {:name=>"fk_e3e63f966e", :on_delete=>:cascade})
                 -> 0.0021s
              -- add_foreign_key("ci_triggers", "users", {:column=>"owner_id", :name=>"fk_e8e10d1964", :on_delete=>:cascade})
                 -> 0.0021s
              -- add_foreign_key("ci_variables", "projects", {:name=>"fk_ada5eb64b3", :on_delete=>:cascade})
                 -> 0.0021s
              -- add_foreign_key("container_repositories", "projects")
                 -> 0.0019s
              -- add_foreign_key("deploy_keys_projects", "projects", {:name=>"fk_58a901ca7e", :on_delete=>:cascade})
                 -> 0.0020s
              -- add_foreign_key("deployments", "projects", {:name=>"fk_b9a3851b82", :on_delete=>:cascade})
                 -> 0.0020s
              -- add_foreign_key("environments", "projects", {:name=>"fk_d1c8c1da6a", :on_delete=>:cascade})
                 -> 0.0023s
              -- add_foreign_key("events", "projects", {:name=>"fk_0434b48643", :on_delete=>:cascade})
                 -> 0.0027s
              -- add_foreign_key("forked_project_links", "projects", {:column=>"forked_to_project_id", :name=>"fk_434510edb0", :on_delete=>:cascade})
                 -> 0.0029s
              -- add_foreign_key("issue_assignees", "issues", {:name=>"fk_b7d881734a", :on_delete=>:cascade})
                 -> 0.0021s
              -- add_foreign_key("issue_assignees", "users", {:name=>"fk_5e0c8d9154", :on_delete=>:cascade})
                 -> 0.0021s
              -- add_foreign_key("issue_metrics", "issues", {:on_delete=>:cascade})
                 -> 0.0021s
              -- add_foreign_key("issues", "projects", {:name=>"fk_899c8f3231", :on_delete=>:cascade})
                 -> 0.0019s
              -- add_foreign_key("label_priorities", "labels", {:on_delete=>:cascade})
                 -> 0.0020s
              -- add_foreign_key("label_priorities", "projects", {:on_delete=>:cascade})
                 -> 0.0020s
              -- add_foreign_key("labels", "namespaces", {:column=>"group_id", :on_delete=>:cascade})
                 -> 0.0022s
              -- add_foreign_key("labels", "projects", {:name=>"fk_7de4989a69", :on_delete=>:cascade})
                 -> 0.0019s
              -- add_foreign_key("lists", "boards", {:name=>"fk_0d3f677137", :on_delete=>:cascade})
                 -> 0.0020s
              -- add_foreign_key("lists", "labels", {:name=>"fk_7a5553d60f", :on_delete=>:cascade})
                 -> 0.0020s
              -- add_foreign_key("merge_request_diff_commits", "merge_request_diffs", {:on_delete=>:cascade})
                 -> 0.0025s
              -- add_foreign_key("merge_request_diff_files", "merge_request_diffs", {:on_delete=>:cascade})
                 -> 0.0020s
              -- add_foreign_key("merge_request_diffs", "merge_requests", {:name=>"fk_8483f3258f", :on_delete=>:cascade})
                 -> 0.0030s
              -- add_foreign_key("merge_request_metrics", "ci_pipelines", {:column=>"pipeline_id", :on_delete=>:cascade})
                 -> 0.0024s
              -- add_foreign_key("merge_request_metrics", "merge_requests", {:on_delete=>:cascade})
                 -> 0.0026s
              -- add_foreign_key("merge_requests", "projects", {:column=>"target_project_id", :name=>"fk_a6963e8447", :on_delete=>:cascade})
                 -> 0.0021s
              -- add_foreign_key("merge_requests_closing_issues", "issues", {:on_delete=>:cascade})
                 -> 0.0023s
              -- add_foreign_key("merge_requests_closing_issues", "merge_requests", {:on_delete=>:cascade})
                 -> 0.0019s
              -- add_foreign_key("milestones", "namespaces", {:column=>"group_id", :name=>"fk_95650a40d4", :on_delete=>:cascade})
                 -> 0.0021s
              -- add_foreign_key("milestones", "projects", {:name=>"fk_9bd0a0c791", :on_delete=>:cascade})
                 -> 0.0023s
              -- add_foreign_key("notes", "projects", {:name=>"fk_99e097b079", :on_delete=>:cascade})
                 -> 0.0024s
              -- add_foreign_key("oauth_openid_requests", "oauth_access_grants", {:column=>"access_grant_id", :name=>"fk_oauth_openid_requests_oauth_access_grants_access_grant_id"})
                 -> 0.0021s
              -- add_foreign_key("pages_domains", "projects", {:name=>"fk_ea2f6dfc6f", :on_delete=>:cascade})
                 -> 0.0021s
              -- add_foreign_key("personal_access_tokens", "users")
                 -> 0.0023s
              -- add_foreign_key("project_authorizations", "projects", {:on_delete=>:cascade})
                 -> 0.0019s
              -- add_foreign_key("project_authorizations", "users", {:on_delete=>:cascade})
                 -> 0.0021s
              -- add_foreign_key("project_features", "projects", {:name=>"fk_18513d9b92", :on_delete=>:cascade})
                 -> 0.0026s
              -- add_foreign_key("project_group_links", "projects", {:name=>"fk_daa8cee94c", :on_delete=>:cascade})
                 -> 0.0020s
              -- add_foreign_key("project_import_data", "projects", {:name=>"fk_ffb9ee3a10", :on_delete=>:cascade})
                 -> 0.0020s
              -- add_foreign_key("project_statistics", "projects", {:on_delete=>:cascade})
                 -> 0.0021s
              -- add_foreign_key("protected_branch_merge_access_levels", "protected_branches", {:name=>"fk_8a3072ccb3", :on_delete=>:cascade})
                 -> 0.0020s
              -- add_foreign_key("protected_branch_push_access_levels", "protected_branches", {:name=>"fk_9ffc86a3d9", :on_delete=>:cascade})
                 -> 0.0021s
              -- add_foreign_key("protected_branches", "projects", {:name=>"fk_7a9c6d93e7", :on_delete=>:cascade})
                 -> 0.0020s
              -- add_foreign_key("protected_tag_create_access_levels", "namespaces", {:column=>"group_id"})
                 -> 0.0022s
              -- add_foreign_key("protected_tag_create_access_levels", "protected_tags")
                 -> 0.0022s
              -- add_foreign_key("protected_tag_create_access_levels", "users")
                 -> 0.0022s
              -- add_foreign_key("protected_tags", "projects", {:name=>"fk_8e4af87648", :on_delete=>:cascade})
                 -> 0.0020s
              -- add_foreign_key("releases", "projects", {:name=>"fk_47fe2a0596", :on_delete=>:cascade})
                 -> 0.0021s
              -- add_foreign_key("services", "projects", {:name=>"fk_71cce407f9", :on_delete=>:cascade})
                 -> 0.0022s
              -- add_foreign_key("snippets", "projects", {:name=>"fk_be41fd4bb7", :on_delete=>:cascade})
                 -> 0.0022s
              -- add_foreign_key("subscriptions", "projects", {:on_delete=>:cascade})
                 -> 0.0020s
              -- add_foreign_key("system_note_metadata", "notes", {:name=>"fk_d83a918cb1", :on_delete=>:cascade})
                 -> 0.0020s
              -- add_foreign_key("timelogs", "issues", {:name=>"fk_timelogs_issues_issue_id", :on_delete=>:cascade})
                 -> 0.0019s
              -- add_foreign_key("timelogs", "merge_requests", {:name=>"fk_timelogs_merge_requests_merge_request_id", :on_delete=>:cascade})
                 -> 0.0021s
              -- add_foreign_key("todos", "projects", {:name=>"fk_45054f9c45", :on_delete=>:cascade})
                 -> 0.0022s
              -- add_foreign_key("trending_projects", "projects", {:on_delete=>:cascade})
                 -> 0.0020s
              -- add_foreign_key("u2f_registrations", "users")
                 -> 0.0021s
              -- add_foreign_key("users_star_projects", "projects", {:name=>"fk_22cd27ddfc", :on_delete=>:cascade})
                 -> 0.0020s
              -- add_foreign_key("web_hook_logs", "web_hooks", {:on_delete=>:cascade})
                 -> 0.0020s
              -- add_foreign_key("web_hooks", "projects", {:name=>"fk_0c8ca6d9d1", :on_delete=>:cascade})
                 -> 0.0021s
              -- initialize_schema_migrations_table()
                 -> 0.0114s
              
              == Seed from /opt/gitlab/embedded/service/gitlab-rails/db/fixtures/production/001_admin.rb
              Administrator account created:
              
              login:    root
              password: You'll be prompted to create one on your first visit.
              
              
              == Seed from /opt/gitlab/embedded/service/gitlab-rails/db/fixtures/production/010_settings.rb
    - execute "bash"  "/tmp/chef-script20170724-7201-1apnk1e"
Recipe: gitlab::gitlab-rails
  * execute[clear the gitlab-rails cache] action run
    - execute /opt/gitlab/bin/gitlab-rake cache:clear
Recipe: gitlab::logrotate_folders_and_configs
  * directory[/var/opt/gitlab/logrotate] action create
    - create new directory /var/opt/gitlab/logrotate
    - change mode from '' to '0700'
    - restore selinux security context
  * directory[/var/opt/gitlab/logrotate/logrotate.d] action create
    - create new directory /var/opt/gitlab/logrotate/logrotate.d
    - change mode from '' to '0700'
    - restore selinux security context
  * directory[/var/log/gitlab/logrotate] action create
    - create new directory /var/log/gitlab/logrotate
    - change mode from '' to '0700'
    - restore selinux security context
  * template[/var/opt/gitlab/logrotate/logrotate.conf] action create
    - create new file /var/opt/gitlab/logrotate/logrotate.conf
    - update content in file /var/opt/gitlab/logrotate/logrotate.conf from none to 378c95
    --- /var/opt/gitlab/logrotate/logrotate.conf	2017-07-24 00:13:54.854000000 +0800
    +++ /var/opt/gitlab/logrotate/.chef-logrotate.conf20170724-7201-byi53k	2017-07-24 00:13:54.854000000 +0800
    @@ -1 +1,10 @@
    +# Generated by 'gitlab-ctl reconfigure'.
    +# Modifications will be overwritten!
    +
    +include /var/opt/gitlab/logrotate/logrotate.d/nginx
    +include /var/opt/gitlab/logrotate/logrotate.d/unicorn
    +include /var/opt/gitlab/logrotate/logrotate.d/gitlab-rails
    +include /var/opt/gitlab/logrotate/logrotate.d/gitlab-shell
    +include /var/opt/gitlab/logrotate/logrotate.d/gitlab-workhorse
    +include /var/opt/gitlab/logrotate/logrotate.d/gitlab-pages
    - change mode from '' to '0644'
    - restore selinux security context
  * template[/var/opt/gitlab/logrotate/logrotate.d/nginx] action create
    - create new file /var/opt/gitlab/logrotate/logrotate.d/nginx
    - update content in file /var/opt/gitlab/logrotate/logrotate.d/nginx from none to b0c555
    --- /var/opt/gitlab/logrotate/logrotate.d/nginx	2017-07-24 00:13:54.887000000 +0800
    +++ /var/opt/gitlab/logrotate/logrotate.d/.chef-nginx20170724-7201-1f5loj9	2017-07-24 00:13:54.884000000 +0800
    @@ -1 +1,15 @@
    +# Generated by gitlab-ctl reconfigure
    +# Modifications will be overwritten!
    +
    +/var/log/gitlab/nginx/*.log {
    +  daily
    +  
    +  rotate 30
    +  compress
    +  copytruncate
    +  missingok
    +  postrotate
    +    
    +  endscript
    +}
    - restore selinux security context
  * template[/var/opt/gitlab/logrotate/logrotate.d/unicorn] action create
    - create new file /var/opt/gitlab/logrotate/logrotate.d/unicorn
    - update content in file /var/opt/gitlab/logrotate/logrotate.d/unicorn from none to 1520c7
    --- /var/opt/gitlab/logrotate/logrotate.d/unicorn	2017-07-24 00:13:54.938000000 +0800
    +++ /var/opt/gitlab/logrotate/logrotate.d/.chef-unicorn20170724-7201-wh3maz	2017-07-24 00:13:54.937000000 +0800
    @@ -1 +1,15 @@
    +# Generated by gitlab-ctl reconfigure
    +# Modifications will be overwritten!
    +
    +/var/log/gitlab/unicorn/*.log {
    +  daily
    +  
    +  rotate 30
    +  compress
    +  copytruncate
    +  missingok
    +  postrotate
    +    
    +  endscript
    +}
    - restore selinux security context
  * template[/var/opt/gitlab/logrotate/logrotate.d/gitlab-rails] action create
    - create new file /var/opt/gitlab/logrotate/logrotate.d/gitlab-rails
    - update content in file /var/opt/gitlab/logrotate/logrotate.d/gitlab-rails from none to f44586
    --- /var/opt/gitlab/logrotate/logrotate.d/gitlab-rails	2017-07-24 00:13:54.972000000 +0800
    +++ /var/opt/gitlab/logrotate/logrotate.d/.chef-gitlab-rails20170724-7201-tb588	2017-07-24 00:13:54.972000000 +0800
    @@ -1 +1,15 @@
    +# Generated by gitlab-ctl reconfigure
    +# Modifications will be overwritten!
    +
    +/var/log/gitlab/gitlab-rails/*.log {
    +  daily
    +  
    +  rotate 30
    +  compress
    +  copytruncate
    +  missingok
    +  postrotate
    +    
    +  endscript
    +}
    - restore selinux security context
  * template[/var/opt/gitlab/logrotate/logrotate.d/gitlab-shell] action create
    - create new file /var/opt/gitlab/logrotate/logrotate.d/gitlab-shell
    - update content in file /var/opt/gitlab/logrotate/logrotate.d/gitlab-shell from none to d12268
    --- /var/opt/gitlab/logrotate/logrotate.d/gitlab-shell	2017-07-24 00:13:54.995000000 +0800
    +++ /var/opt/gitlab/logrotate/logrotate.d/.chef-gitlab-shell20170724-7201-dz2o55	2017-07-24 00:13:54.995000000 +0800
    @@ -1 +1,15 @@
    +# Generated by gitlab-ctl reconfigure
    +# Modifications will be overwritten!
    +
    +/var/log/gitlab/gitlab-shell//*.log {
    +  daily
    +  
    +  rotate 30
    +  compress
    +  copytruncate
    +  missingok
    +  postrotate
    +    
    +  endscript
    +}
    - restore selinux security context
  * template[/var/opt/gitlab/logrotate/logrotate.d/gitlab-workhorse] action create
    - create new file /var/opt/gitlab/logrotate/logrotate.d/gitlab-workhorse
    - update content in file /var/opt/gitlab/logrotate/logrotate.d/gitlab-workhorse from none to 359431
    --- /var/opt/gitlab/logrotate/logrotate.d/gitlab-workhorse	2017-07-24 00:13:55.023000000 +0800
    +++ /var/opt/gitlab/logrotate/logrotate.d/.chef-gitlab-workhorse20170724-7201-8sunof	2017-07-24 00:13:55.023000000 +0800
    @@ -1 +1,15 @@
    +# Generated by gitlab-ctl reconfigure
    +# Modifications will be overwritten!
    +
    +/var/log/gitlab/gitlab-workhorse/*.log {
    +  daily
    +  
    +  rotate 30
    +  compress
    +  copytruncate
    +  missingok
    +  postrotate
    +    
    +  endscript
    +}
    - restore selinux security context
  * template[/var/opt/gitlab/logrotate/logrotate.d/gitlab-pages] action create
    - create new file /var/opt/gitlab/logrotate/logrotate.d/gitlab-pages
    - update content in file /var/opt/gitlab/logrotate/logrotate.d/gitlab-pages from none to 3a241b
    --- /var/opt/gitlab/logrotate/logrotate.d/gitlab-pages	2017-07-24 00:13:55.048000000 +0800
    +++ /var/opt/gitlab/logrotate/logrotate.d/.chef-gitlab-pages20170724-7201-1jeccsk	2017-07-24 00:13:55.048000000 +0800
    @@ -1 +1,15 @@
    +# Generated by gitlab-ctl reconfigure
    +# Modifications will be overwritten!
    +
    +/var/log/gitlab/gitlab-pages/*.log {
    +  daily
    +  
    +  rotate 30
    +  compress
    +  copytruncate
    +  missingok
    +  postrotate
    +    
    +  endscript
    +}
    - restore selinux security context
Recipe: gitlab::unicorn
  * directory[/var/log/gitlab/unicorn] action create
    - create new directory /var/log/gitlab/unicorn
    - change mode from '' to '0700'
    - change owner from '' to 'git'
    - restore selinux security context
  * directory[/opt/gitlab/var/unicorn] action create
    - create new directory /opt/gitlab/var/unicorn
    - change mode from '' to '0700'
    - change owner from '' to 'git'
    - restore selinux security context
  * directory[/var/opt/gitlab/gitlab-rails/sockets] action create
    - create new directory /var/opt/gitlab/gitlab-rails/sockets
    - change mode from '' to '0750'
    - change owner from '' to 'git'
    - change group from '' to 'gitlab-www'
    - restore selinux security context
  * directory[/var/opt/gitlab/gitlab-rails/etc] action create (up to date)
  * template[/var/opt/gitlab/gitlab-rails/etc/unicorn.rb] action create
    - create new file /var/opt/gitlab/gitlab-rails/etc/unicorn.rb
    - update content in file /var/opt/gitlab/gitlab-rails/etc/unicorn.rb from none to 850ad1
    --- /var/opt/gitlab/gitlab-rails/etc/unicorn.rb	2017-07-24 00:13:55.144000000 +0800
    +++ /var/opt/gitlab/gitlab-rails/etc/.chef-unicorn.rb20170724-7201-17xprcw	2017-07-24 00:13:55.144000000 +0800
    @@ -1 +1,57 @@
    +# This file is managed by gitlab-ctl. Manual changes will be
    +# erased! To change the contents below, edit /etc/gitlab/gitlab.rb
    +# and run `sudo gitlab-ctl reconfigure`.
    +
    +
    +# What ports/sockets to listen on, and what options for them.
    +listen "127.0.0.1:8080", :tcp_nopush => true
    +listen "/var/opt/gitlab/gitlab-rails/sockets/gitlab.socket", :backlog => 1024
    +
    +working_directory '/var/opt/gitlab/gitlab-rails/working'
    +
    +# What the timeout for killing busy workers is, in seconds
    +timeout 60
    +
    +# Whether the app should be pre-loaded
    +preload_app true
    +
    +# How many worker processes
    +worker_processes 3
    +
    +# What to do before we fork a worker
    +before_fork do |server, worker|
    +        old_pid = "#{server.config[:pid]}.oldbin"
    +      if old_pid != server.pid
    +        begin
    +          sig = (worker.nr + 1) >= server.worker_processes ? :QUIT : :TTOU
    +          Process.kill(sig, File.read(old_pid).to_i)
    +        rescue Errno::ENOENT, Errno::ESRCH
    +        end
    +      end
    +
    +      ActiveRecord::Base.connection.disconnect! if defined?(ActiveRecord::Base)
    +
    +end
    +
    +# What to do after we fork a worker
    +after_fork do |server, worker|
    +        ActiveRecord::Base.establish_connection if defined?(ActiveRecord::Base)
    +
    +end
    +
    +# Where to drop a pidfile
    +pid '/opt/gitlab/var/unicorn/unicorn.pid'
    +
    +# Where stderr gets logged
    +stderr_path '/var/log/gitlab/unicorn/unicorn_stderr.log'
    +
    +# Where stdout gets logged
    +stdout_path '/var/log/gitlab/unicorn/unicorn_stdout.log'
    +
    +# Min memory size (RSS) per worker
    +ENV['GITLAB_UNICORN_MEMORY_MIN'] = (400 * 1 << 20).to_s
    +
    +# Max memory size (RSS) per worker
    +ENV['GITLAB_UNICORN_MEMORY_MAX'] = (650 * 1 << 20).to_s
    +
    - change mode from '' to '0644'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * directory[/opt/gitlab/sv/unicorn] action create
    - create new directory /opt/gitlab/sv/unicorn
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * directory[/opt/gitlab/sv/unicorn/log] action create
    - create new directory /opt/gitlab/sv/unicorn/log
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * directory[/opt/gitlab/sv/unicorn/log/main] action create
    - create new directory /opt/gitlab/sv/unicorn/log/main
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * template[/opt/gitlab/sv/unicorn/run] action create
    - create new file /opt/gitlab/sv/unicorn/run
    - update content in file /opt/gitlab/sv/unicorn/run from none to dab388
    --- /opt/gitlab/sv/unicorn/run	2017-07-24 00:13:55.254000000 +0800
    +++ /opt/gitlab/sv/unicorn/.chef-run20170724-7201-mm0sn5	2017-07-24 00:13:55.254000000 +0800
    @@ -1 +1,25 @@
    +#!/bin/bash
    +
    +# Let runit capture all script error messages
    +exec 2>&1
    +
    +# Setup run directory.
    +mkdir -p /run/gitlab/unicorn
    +rm /run/gitlab/unicorn/*.db 2> /dev/null
    +chmod 0700 /run/gitlab/unicorn
    +chown git /run/gitlab/unicorn
    +export prometheus_run_dir='/run/gitlab/unicorn'
    +
    +
    +
    +
    +exec chpst -P -u git \
    +  /usr/bin/env \
    +    current_pidfile=/opt/gitlab/var/unicorn/unicorn.pid \
    +    rails_app=gitlab-rails \
    +    user=git \
    +    environment=production \
    +    unicorn_rb=/var/opt/gitlab/gitlab-rails/etc/unicorn.rb \
    +    prometheus_multiproc_dir="${prometheus_run_dir}" \
    +    /opt/gitlab/embedded/bin/gitlab-unicorn-wrapper
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * template[/opt/gitlab/sv/unicorn/log/run] action create
    - create new file /opt/gitlab/sv/unicorn/log/run
    - update content in file /opt/gitlab/sv/unicorn/log/run from none to d50262
    --- /opt/gitlab/sv/unicorn/log/run	2017-07-24 00:13:55.283000000 +0800
    +++ /opt/gitlab/sv/unicorn/log/.chef-run20170724-7201-1mktjkt	2017-07-24 00:13:55.283000000 +0800
    @@ -1 +1,3 @@
    +#!/bin/sh
    +exec svlogd -tt /var/log/gitlab/unicorn
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * template[/var/log/gitlab/unicorn/config] action create
    - create new file /var/log/gitlab/unicorn/config
    - update content in file /var/log/gitlab/unicorn/config from none to 623c00
    --- /var/log/gitlab/unicorn/config	2017-07-24 00:13:55.308000000 +0800
    +++ /var/log/gitlab/unicorn/.chef-config20170724-7201-ci43s5	2017-07-24 00:13:55.308000000 +0800
    @@ -1 +1,7 @@
    +s209715200
    +n30
    +t86400
    +!gzip
    +
    +
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * ruby_block[reload unicorn svlogd configuration] action nothing (skipped due to action :nothing)
  * file[/opt/gitlab/sv/unicorn/down] action delete (up to date)
  * directory[/opt/gitlab/sv/unicorn/control] action create
    - create new directory /opt/gitlab/sv/unicorn/control
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * template[/opt/gitlab/sv/unicorn/control/t] action create
    - create new file /opt/gitlab/sv/unicorn/control/t
    - update content in file /opt/gitlab/sv/unicorn/control/t from none to 84b233
    --- /opt/gitlab/sv/unicorn/control/t	2017-07-24 00:13:55.367000000 +0800
    +++ /opt/gitlab/sv/unicorn/control/.chef-t20170724-7201-1d9t7rx	2017-07-24 00:13:55.367000000 +0800
    @@ -1 +1,4 @@
    +#!/bin/sh
    +echo "Received TERM from runit, sending to process group (-PID)"
    +kill -- -$(cat /opt/gitlab/service/unicorn/supervise/pid)
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * link[/opt/gitlab/init/unicorn] action create
    - create symlink at /opt/gitlab/init/unicorn to /opt/gitlab/embedded/bin/sv
  * link[/opt/gitlab/service/unicorn] action create
    - create symlink at /opt/gitlab/service/unicorn to /opt/gitlab/sv/unicorn
  * ruby_block[supervise_unicorn_sleep] action run
    - execute the ruby block supervise_unicorn_sleep
  * directory[/opt/gitlab/sv/unicorn/supervise] action create
    - change mode from '0700' to '0755'
    - restore selinux security context
  * directory[/opt/gitlab/sv/unicorn/log/supervise] action create
    - change mode from '0700' to '0755'
    - restore selinux security context
  * file[/opt/gitlab/sv/unicorn/supervise/ok] action touch (skipped due to not_if)
  * file[/opt/gitlab/sv/unicorn/log/supervise/ok] action touch (skipped due to not_if)
  * file[/opt/gitlab/sv/unicorn/supervise/control] action touch (skipped due to not_if)
  * file[/opt/gitlab/sv/unicorn/log/supervise/control] action touch (skipped due to not_if)
  * service[unicorn] action nothing (skipped due to action :nothing)
  * execute[/opt/gitlab/bin/gitlab-ctl start unicorn] action run
    [execute] ok: run: unicorn: (pid 8411) 1s
    - execute /opt/gitlab/bin/gitlab-ctl start unicorn
  * directory[create /etc/sysctl.d for net.core.somaxconn] action create (up to date)
  * file[create /opt/gitlab/embedded/etc/90-omnibus-gitlab-net.core.somaxconn.conf net.core.somaxconn] action create
    - create new file /opt/gitlab/embedded/etc/90-omnibus-gitlab-net.core.somaxconn.conf
    - update content in file /opt/gitlab/embedded/etc/90-omnibus-gitlab-net.core.somaxconn.conf from none to 353a75
    --- /opt/gitlab/embedded/etc/90-omnibus-gitlab-net.core.somaxconn.conf	2017-07-24 00:14:00.804000000 +0800
    +++ /opt/gitlab/embedded/etc/.chef-90-omnibus-gitlab-net.core.somaxconn.conf net.core.somaxconn20170724-7201-1y9xrph	2017-07-24 00:14:00.804000000 +0800
    @@ -1 +1,2 @@
    +net.core.somaxconn = 1024
    - restore selinux security context
  * execute[load sysctl conf net.core.somaxconn] action run
    [execute] kernel.sem = 250 32000 32 262
              kernel.shmall = 4194304
              kernel.shmmax = 17179869184
    - execute cat /etc/sysctl.conf /etc/sysctl.d/*.conf  | sysctl -e -p -
  * link[/etc/sysctl.d/90-omnibus-gitlab-net.core.somaxconn.conf] action create
    - create symlink at /etc/sysctl.d/90-omnibus-gitlab-net.core.somaxconn.conf to /opt/gitlab/embedded/etc/90-omnibus-gitlab-net.core.somaxconn.conf
  * file[delete /etc/sysctl.d/90-postgresql.conf net.core.somaxconn] action delete (skipped due to only_if)
  * file[delete /etc/sysctl.d/90-unicorn.conf net.core.somaxconn] action delete (skipped due to only_if)
  * file[delete /opt/gitlab/embedded/etc/90-omnibus-gitlab.conf net.core.somaxconn] action delete (skipped due to only_if)
  * file[delete /etc/sysctl.d/90-omnibus-gitlab.conf net.core.somaxconn] action delete (skipped due to only_if)
  * execute[load sysctl conf net.core.somaxconn] action nothing (skipped due to action :nothing)
Recipe: gitlab::sidekiq
  * directory[/var/log/gitlab/sidekiq] action create
    - create new directory /var/log/gitlab/sidekiq
    - change mode from '' to '0700'
    - change owner from '' to 'git'
    - restore selinux security context
  * directory[/opt/gitlab/sv/sidekiq] action create
    - create new directory /opt/gitlab/sv/sidekiq
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * directory[/opt/gitlab/sv/sidekiq/log] action create
    - create new directory /opt/gitlab/sv/sidekiq/log
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * directory[/opt/gitlab/sv/sidekiq/log/main] action create
    - create new directory /opt/gitlab/sv/sidekiq/log/main
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * template[/opt/gitlab/sv/sidekiq/run] action create
    - create new file /opt/gitlab/sv/sidekiq/run
    - update content in file /opt/gitlab/sv/sidekiq/run from none to f49a74
    --- /opt/gitlab/sv/sidekiq/run	2017-07-24 00:14:00.972000000 +0800
    +++ /opt/gitlab/sv/sidekiq/.chef-run20170724-7201-2029fa	2017-07-24 00:14:00.971000000 +0800
    @@ -1 +1,25 @@
    +#!/bin/sh
    +
    +cd /var/opt/gitlab/gitlab-rails/working
    +
    +exec 2>&1
    +# Setup run directory.
    +mkdir -p /run/gitlab/sidekiq
    +rm /run/gitlab/sidekiq/*.db 2> /dev/null
    +chmod 0700 /run/gitlab/sidekiq
    +chown git /run/gitlab/sidekiq
    +export prometheus_run_dir='/run/gitlab/sidekiq'
    +
    +
    +
    +exec chpst -e /opt/gitlab/etc/gitlab-rails/env -P \
    +  -U git -u git \
    +  /usr/bin/env \
    +    prometheus_multiproc_dir="${prometheus_run_dir}" \
    +    /opt/gitlab/embedded/bin/bundle exec sidekiq \
    +      -C /opt/gitlab/embedded/service/gitlab-rails/config/sidekiq_queues.yml \
    +      -e production \
    +      -r /opt/gitlab/embedded/service/gitlab-rails \
    +      -t 4 \
    +      -c 25
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * template[/opt/gitlab/sv/sidekiq/log/run] action create
    - create new file /opt/gitlab/sv/sidekiq/log/run
    - update content in file /opt/gitlab/sv/sidekiq/log/run from none to 051d8f
    --- /opt/gitlab/sv/sidekiq/log/run	2017-07-24 00:14:01.002000000 +0800
    +++ /opt/gitlab/sv/sidekiq/log/.chef-run20170724-7201-tpp74t	2017-07-24 00:14:01.000000000 +0800
    @@ -1 +1,3 @@
    +#!/bin/sh
    +exec svlogd -tt /var/log/gitlab/sidekiq
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * template[/var/log/gitlab/sidekiq/config] action create
    - create new file /var/log/gitlab/sidekiq/config
    - update content in file /var/log/gitlab/sidekiq/config from none to 623c00
    --- /var/log/gitlab/sidekiq/config	2017-07-24 00:14:01.029000000 +0800
    +++ /var/log/gitlab/sidekiq/.chef-config20170724-7201-1q31eo5	2017-07-24 00:14:01.029000000 +0800
    @@ -1 +1,7 @@
    +s209715200
    +n30
    +t86400
    +!gzip
    +
    +
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * ruby_block[reload sidekiq svlogd configuration] action nothing (skipped due to action :nothing)
  * file[/opt/gitlab/sv/sidekiq/down] action delete (up to date)
  * link[/opt/gitlab/init/sidekiq] action create
    - create symlink at /opt/gitlab/init/sidekiq to /opt/gitlab/embedded/bin/sv
  * link[/opt/gitlab/service/sidekiq] action create
    - create symlink at /opt/gitlab/service/sidekiq to /opt/gitlab/sv/sidekiq
  * ruby_block[supervise_sidekiq_sleep] action run
    - execute the ruby block supervise_sidekiq_sleep
  * directory[/opt/gitlab/sv/sidekiq/supervise] action create
    - change mode from '0700' to '0755'
    - restore selinux security context
  * directory[/opt/gitlab/sv/sidekiq/log/supervise] action create
    - change mode from '0700' to '0755'
    - restore selinux security context
  * file[/opt/gitlab/sv/sidekiq/supervise/ok] action touch (skipped due to not_if)
  * file[/opt/gitlab/sv/sidekiq/log/supervise/ok] action touch (skipped due to not_if)
  * file[/opt/gitlab/sv/sidekiq/supervise/control] action touch (skipped due to not_if)
  * file[/opt/gitlab/sv/sidekiq/log/supervise/control] action touch (skipped due to not_if)
  * service[sidekiq] action nothing (skipped due to action :nothing)
  * execute[/opt/gitlab/bin/gitlab-ctl start sidekiq] action run
    [execute] ok: run: sidekiq: (pid 8480) 1s
    - execute /opt/gitlab/bin/gitlab-ctl start sidekiq
Recipe: gitlab::gitaly
  * directory[/var/opt/gitlab/gitaly] action create
    - create new directory /var/opt/gitlab/gitaly
    - change mode from '' to '0700'
    - change owner from '' to 'git'
    - restore selinux security context
  * directory[/var/log/gitlab/gitaly] action create
    - create new directory /var/log/gitlab/gitaly
    - change mode from '' to '0700'
    - change owner from '' to 'git'
    - restore selinux security context
  * directory[/opt/gitlab/etc/gitaly] action create
    - create new directory /opt/gitlab/etc/gitaly
    - restore selinux security context
  * file[/opt/gitlab/etc/gitaly/PATH] action create
    - create new file /opt/gitlab/etc/gitaly/PATH
    - update content in file /opt/gitlab/etc/gitaly/PATH from none to d5dc07
    --- /opt/gitlab/etc/gitaly/PATH	2017-07-24 00:14:07.154000000 +0800
    +++ /opt/gitlab/etc/gitaly/.chef-PATH20170724-7201-m7bak5	2017-07-24 00:14:07.154000000 +0800
    @@ -1 +1,2 @@
    +/opt/gitlab/bin:/opt/gitlab/embedded/bin:/bin:/usr/bin
    - restore selinux security context
  * file[/opt/gitlab/etc/gitaly/HOME] action create
    - create new file /opt/gitlab/etc/gitaly/HOME
    - update content in file /opt/gitlab/etc/gitaly/HOME from none to 205bb9
    --- /opt/gitlab/etc/gitaly/HOME	2017-07-24 00:14:07.196000000 +0800
    +++ /opt/gitlab/etc/gitaly/.chef-HOME20170724-7201-1y8kq2j	2017-07-24 00:14:07.196000000 +0800
    @@ -1 +1,2 @@
    +/var/opt/gitlab
    - restore selinux security context
  * file[/opt/gitlab/etc/gitaly/TZ] action create
    - create new file /opt/gitlab/etc/gitaly/TZ
    - update content in file /opt/gitlab/etc/gitaly/TZ from none to 983a95
    --- /opt/gitlab/etc/gitaly/TZ	2017-07-24 00:14:07.264000000 +0800
    +++ /opt/gitlab/etc/gitaly/.chef-TZ20170724-7201-wql65q	2017-07-24 00:14:07.264000000 +0800
    @@ -1 +1,2 @@
    +:/etc/localtime
    - restore selinux security context
  * template[Create Gitaly config.toml] action create
    - create new file /var/opt/gitlab/gitaly/config.toml
    - update content in file /var/opt/gitlab/gitaly/config.toml from none to c435a3
    --- /var/opt/gitlab/gitaly/config.toml	2017-07-24 00:14:07.345000000 +0800
    +++ /var/opt/gitlab/gitaly/.chef-Create Gitaly config.toml20170724-7201-jwtmr8	2017-07-24 00:14:07.345000000 +0800
    @@ -1 +1,19 @@
    +# Gitaly configuration file
    +# This file is managed by gitlab-ctl. Manual changes will be
    +# erased! To change the contents below, edit /etc/gitlab/gitlab.rb
    +# and run:
    +# sudo gitlab-ctl reconfigure
    +
    +socket_path = '/var/opt/gitlab/gitaly/gitaly.socket'
    +
    +
    +
    +[[storage]]
    +name = 'default'
    +path = '/var/opt/gitlab/git-data/repositories'
    +
    +[logging]
    +
    +
    +[auth]
    - change mode from '' to '0644'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * directory[/opt/gitlab/sv/gitaly] action create
    - create new directory /opt/gitlab/sv/gitaly
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * directory[/opt/gitlab/sv/gitaly/log] action create
    - create new directory /opt/gitlab/sv/gitaly/log
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * directory[/opt/gitlab/sv/gitaly/log/main] action create
    - create new directory /opt/gitlab/sv/gitaly/log/main
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * template[/opt/gitlab/sv/gitaly/run] action create
    - create new file /opt/gitlab/sv/gitaly/run
    - update content in file /opt/gitlab/sv/gitaly/run from none to ca190e
    --- /opt/gitlab/sv/gitaly/run	2017-07-24 00:14:07.614000000 +0800
    +++ /opt/gitlab/sv/gitaly/.chef-run20170724-7201-pcrflv	2017-07-24 00:14:07.614000000 +0800
    @@ -1 +1,15 @@
    +#!/bin/sh
    +set -e # fail on errors
    +
    +# Redirect stderr -> stdout
    +exec 2>&1
    +
    +
    +
    +cd /var/opt/gitlab/gitaly
    +
    +exec chpst -e /opt/gitlab/etc/gitaly -P \
    +  -U git \
    +  -u git \
    +  /opt/gitlab/embedded/bin/gitaly /var/opt/gitlab/gitaly/config.toml
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * template[/opt/gitlab/sv/gitaly/log/run] action create
    - create new file /opt/gitlab/sv/gitaly/log/run
    - update content in file /opt/gitlab/sv/gitaly/log/run from none to 0627d2
    --- /opt/gitlab/sv/gitaly/log/run	2017-07-24 00:14:07.676000000 +0800
    +++ /opt/gitlab/sv/gitaly/log/.chef-run20170724-7201-brfdge	2017-07-24 00:14:07.676000000 +0800
    @@ -1 +1,3 @@
    +#!/bin/sh
    +exec svlogd -tt /var/log/gitlab/gitaly
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * template[/var/log/gitlab/gitaly/config] action create
    - create new file /var/log/gitlab/gitaly/config
    - update content in file /var/log/gitlab/gitaly/config from none to 623c00
    --- /var/log/gitlab/gitaly/config	2017-07-24 00:14:07.767000000 +0800
    +++ /var/log/gitlab/gitaly/.chef-config20170724-7201-f7hdwm	2017-07-24 00:14:07.767000000 +0800
    @@ -1 +1,7 @@
    +s209715200
    +n30
    +t86400
    +!gzip
    +
    +
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * ruby_block[reload gitaly svlogd configuration] action nothing (skipped due to action :nothing)
  * file[/opt/gitlab/sv/gitaly/down] action delete (up to date)
  * link[/opt/gitlab/init/gitaly] action create
    - create symlink at /opt/gitlab/init/gitaly to /opt/gitlab/embedded/bin/sv
  * link[/opt/gitlab/service/gitaly] action create
    - create symlink at /opt/gitlab/service/gitaly to /opt/gitlab/sv/gitaly
  * ruby_block[supervise_gitaly_sleep] action run
    - execute the ruby block supervise_gitaly_sleep
  * directory[/opt/gitlab/sv/gitaly/supervise] action create
    - change mode from '0700' to '0755'
    - restore selinux security context
  * directory[/opt/gitlab/sv/gitaly/log/supervise] action create
    - change mode from '0700' to '0755'
    - restore selinux security context
  * file[/opt/gitlab/sv/gitaly/supervise/ok] action touch (skipped due to not_if)
  * file[/opt/gitlab/sv/gitaly/log/supervise/ok] action touch (skipped due to not_if)
  * file[/opt/gitlab/sv/gitaly/supervise/control] action touch (skipped due to not_if)
  * file[/opt/gitlab/sv/gitaly/log/supervise/control] action touch (skipped due to not_if)
  * service[gitaly] action nothing (skipped due to action :nothing)
  * execute[/opt/gitlab/bin/gitlab-ctl start gitaly] action run
    [execute] ok: run: gitaly: (pid 8545) 1s
    - execute /opt/gitlab/bin/gitlab-ctl start gitaly
Recipe: gitlab::gitlab-workhorse
  * directory[/var/opt/gitlab/gitlab-workhorse] action create
    - create new directory /var/opt/gitlab/gitlab-workhorse
    - change mode from '' to '0750'
    - change owner from '' to 'git'
    - change group from '' to 'gitlab-www'
    - restore selinux security context
  * directory[/var/log/gitlab/gitlab-workhorse] action create
    - create new directory /var/log/gitlab/gitlab-workhorse
    - change mode from '' to '0700'
    - change owner from '' to 'git'
    - restore selinux security context
  * directory[/opt/gitlab/etc/gitlab-workhorse] action create
    - create new directory /opt/gitlab/etc/gitlab-workhorse
    - change mode from '' to '0700'
    - change owner from '' to 'git'
    - restore selinux security context
  * directory[/opt/gitlab/etc/gitlab-workhorse/env] action create
    - create new directory /opt/gitlab/etc/gitlab-workhorse/env
    - restore selinux security context
  * file[/opt/gitlab/etc/gitlab-workhorse/env/PATH] action create
    - create new file /opt/gitlab/etc/gitlab-workhorse/env/PATH
    - update content in file /opt/gitlab/etc/gitlab-workhorse/env/PATH from none to d5dc07
    --- /opt/gitlab/etc/gitlab-workhorse/env/PATH	2017-07-24 00:14:12.548000000 +0800
    +++ /opt/gitlab/etc/gitlab-workhorse/env/.chef-PATH20170724-7201-67myzb	2017-07-24 00:14:12.547000000 +0800
    @@ -1 +1,2 @@
    +/opt/gitlab/bin:/opt/gitlab/embedded/bin:/bin:/usr/bin
    - restore selinux security context
  * file[/opt/gitlab/etc/gitlab-workhorse/env/HOME] action create
    - create new file /opt/gitlab/etc/gitlab-workhorse/env/HOME
    - update content in file /opt/gitlab/etc/gitlab-workhorse/env/HOME from none to 205bb9
    --- /opt/gitlab/etc/gitlab-workhorse/env/HOME	2017-07-24 00:14:12.585000000 +0800
    +++ /opt/gitlab/etc/gitlab-workhorse/env/.chef-HOME20170724-7201-h4ojpg	2017-07-24 00:14:12.585000000 +0800
    @@ -1 +1,2 @@
    +/var/opt/gitlab
    - restore selinux security context
  * directory[/opt/gitlab/sv/gitlab-workhorse] action create
    - create new directory /opt/gitlab/sv/gitlab-workhorse
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * directory[/opt/gitlab/sv/gitlab-workhorse/log] action create
    - create new directory /opt/gitlab/sv/gitlab-workhorse/log
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * directory[/opt/gitlab/sv/gitlab-workhorse/log/main] action create
    - create new directory /opt/gitlab/sv/gitlab-workhorse/log/main
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * template[/opt/gitlab/sv/gitlab-workhorse/run] action create
    - create new file /opt/gitlab/sv/gitlab-workhorse/run
    - update content in file /opt/gitlab/sv/gitlab-workhorse/run from none to 7cf0fc
    --- /opt/gitlab/sv/gitlab-workhorse/run	2017-07-24 00:14:12.724000000 +0800
    +++ /opt/gitlab/sv/gitlab-workhorse/.chef-run20170724-7201-1l3sq21	2017-07-24 00:14:12.724000000 +0800
    @@ -1 +1,26 @@
    +#!/bin/sh
    +set -e # fail on errors
    +
    +# Redirect stderr -> stdout
    +exec 2>&1
    +
    +
    +
    +cd /var/opt/gitlab/gitlab-workhorse
    +
    +exec chpst -e /opt/gitlab/etc/gitlab-workhorse/env -P \
    +  -U git \
    +  -u git \
    +  /opt/gitlab/embedded/bin/gitlab-workhorse \
    +    -listenNetwork unix \
    +    -listenUmask 0 \
    +    -listenAddr /var/opt/gitlab/gitlab-workhorse/socket \
    +    -authBackend http://localhost:8080 \
    +    -authSocket /var/opt/gitlab/gitlab-rails/sockets/gitlab.socket \
    +    -documentRoot /opt/gitlab/embedded/service/gitlab-rails/public \
    +    -pprofListenAddr ''\
    +    -secretPath /opt/gitlab/embedded/service/gitlab-rails/.gitlab_workhorse_secret \
    +    -config config.toml \
    +
    +# Do not remove this line; it prevents trouble with the trailing backslashes above.
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * template[/opt/gitlab/sv/gitlab-workhorse/log/run] action create
    - create new file /opt/gitlab/sv/gitlab-workhorse/log/run
    - update content in file /opt/gitlab/sv/gitlab-workhorse/log/run from none to 6ed0e1
    --- /opt/gitlab/sv/gitlab-workhorse/log/run	2017-07-24 00:14:12.756000000 +0800
    +++ /opt/gitlab/sv/gitlab-workhorse/log/.chef-run20170724-7201-1y2pbj1	2017-07-24 00:14:12.756000000 +0800
    @@ -1 +1,3 @@
    +#!/bin/sh
    +exec svlogd -tt /var/log/gitlab/gitlab-workhorse
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * template[/var/log/gitlab/gitlab-workhorse/config] action create
    - create new file /var/log/gitlab/gitlab-workhorse/config
    - update content in file /var/log/gitlab/gitlab-workhorse/config from none to 623c00
    --- /var/log/gitlab/gitlab-workhorse/config	2017-07-24 00:14:12.785000000 +0800
    +++ /var/log/gitlab/gitlab-workhorse/.chef-config20170724-7201-15sa5c0	2017-07-24 00:14:12.785000000 +0800
    @@ -1 +1,7 @@
    +s209715200
    +n30
    +t86400
    +!gzip
    +
    +
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * ruby_block[reload gitlab-workhorse svlogd configuration] action nothing (skipped due to action :nothing)
  * file[/opt/gitlab/sv/gitlab-workhorse/down] action delete (up to date)
  * link[/opt/gitlab/init/gitlab-workhorse] action create
    - create symlink at /opt/gitlab/init/gitlab-workhorse to /opt/gitlab/embedded/bin/sv
  * link[/opt/gitlab/service/gitlab-workhorse] action create
    - create symlink at /opt/gitlab/service/gitlab-workhorse to /opt/gitlab/sv/gitlab-workhorse
  * ruby_block[supervise_gitlab-workhorse_sleep] action run
    - execute the ruby block supervise_gitlab-workhorse_sleep
  * directory[/opt/gitlab/sv/gitlab-workhorse/supervise] action create
    - change mode from '0700' to '0755'
    - restore selinux security context
  * directory[/opt/gitlab/sv/gitlab-workhorse/log/supervise] action create
    - change mode from '0700' to '0755'
    - restore selinux security context
  * file[/opt/gitlab/sv/gitlab-workhorse/supervise/ok] action touch (skipped due to not_if)
  * file[/opt/gitlab/sv/gitlab-workhorse/log/supervise/ok] action touch (skipped due to not_if)
  * file[/opt/gitlab/sv/gitlab-workhorse/supervise/control] action touch (skipped due to not_if)
  * file[/opt/gitlab/sv/gitlab-workhorse/log/supervise/control] action touch (skipped due to not_if)
  * service[gitlab-workhorse] action nothing (skipped due to action :nothing)
  * file[/var/opt/gitlab/gitlab-workhorse/VERSION] action create
    - create new file /var/opt/gitlab/gitlab-workhorse/VERSION
    - update content in file /var/opt/gitlab/gitlab-workhorse/VERSION from none to e110bf
    --- /var/opt/gitlab/gitlab-workhorse/VERSION	2017-07-24 00:14:17.895000000 +0800
    +++ /var/opt/gitlab/gitlab-workhorse/.chef-VERSION20170724-7201-ll3pgq	2017-07-24 00:14:17.895000000 +0800
    @@ -1 +1,2 @@
    +gitlab-workhorse v2.3.0-20170722.062507
    - restore selinux security context
  * template[/var/opt/gitlab/gitlab-workhorse/config.toml] action create
    - create new file /var/opt/gitlab/gitlab-workhorse/config.toml
    - update content in file /var/opt/gitlab/gitlab-workhorse/config.toml from none to cb62fe
    --- /var/opt/gitlab/gitlab-workhorse/config.toml	2017-07-24 00:14:17.924000000 +0800
    +++ /var/opt/gitlab/gitlab-workhorse/.chef-config.toml20170724-7201-dki6n1	2017-07-24 00:14:17.924000000 +0800
    @@ -1 +1,4 @@
    +[redis]
    +URL = "unix:/var/opt/gitlab/redis/redis.socket"
    +Password = ""
    - change owner from '' to 'git'
    - restore selinux security context
Recipe: gitlab::mailroom_disable
  * link[/opt/gitlab/service/mailroom] action delete (up to date)
  * directory[/opt/gitlab/sv/mailroom] action delete (up to date)
Recipe: gitlab::nginx
  * directory[/var/opt/gitlab/nginx] action create
    - create new directory /var/opt/gitlab/nginx
    - change mode from '' to '0750'
    - change owner from '' to 'root'
    - change group from '' to 'gitlab-www'
    - restore selinux security context
  * directory[/var/opt/gitlab/nginx/conf] action create
    - create new directory /var/opt/gitlab/nginx/conf
    - change mode from '' to '0750'
    - change owner from '' to 'root'
    - change group from '' to 'gitlab-www'
    - restore selinux security context
  * directory[/var/log/gitlab/nginx] action create
    - create new directory /var/log/gitlab/nginx
    - change mode from '' to '0750'
    - change owner from '' to 'root'
    - change group from '' to 'gitlab-www'
    - restore selinux security context
  * link[/var/opt/gitlab/nginx/logs] action create
    - create symlink at /var/opt/gitlab/nginx/logs to /var/log/gitlab/nginx
  * template[/var/opt/gitlab/nginx/conf/gitlab-http.conf] action create
    - create new file /var/opt/gitlab/nginx/conf/gitlab-http.conf
    - update content in file /var/opt/gitlab/nginx/conf/gitlab-http.conf from none to 821be1
    --- /var/opt/gitlab/nginx/conf/gitlab-http.conf	2017-07-24 00:14:18.044000000 +0800
    +++ /var/opt/gitlab/nginx/conf/.chef-gitlab-http.conf20170724-7201-weaf94	2017-07-24 00:14:18.044000000 +0800
    @@ -1 +1,114 @@
    +# This file is managed by gitlab-ctl. Manual changes will be
    +# erased! To change the contents below, edit /etc/gitlab/gitlab.rb
    +# and run `sudo gitlab-ctl reconfigure`.
    +
    +## GitLab
    +## Modified from https://gitlab.com/gitlab-org/gitlab-ce/blob/master/lib/support/nginx/gitlab-ssl & https://gitlab.com/gitlab-org/gitlab-ce/blob/master/lib/support/nginx/gitlab
    +##
    +## Lines starting with two hashes (##) are comments with information.
    +## Lines starting with one hash (#) are configuration parameters that can be uncommented.
    +##
    +##################################
    +##        CHUNKED TRANSFER      ##
    +##################################
    +##
    +## It is a known issue that Git-over-HTTP requires chunked transfer encoding [0]
    +## which is not supported by Nginx < 1.3.9 [1]. As a result, pushing a large object
    +## with Git (i.e. a single large file) can lead to a 411 error. In theory you can get
    +## around this by tweaking this configuration file and either:
    +## - installing an old version of Nginx with the chunkin module [2] compiled in, or
    +## - using a newer version of Nginx.
    +##
    +## At the time of writing we do not know if either of these theoretical solutions works.
    +## As a workaround users can use Git over SSH to push large files.
    +##
    +## [0] https://git.kernel.org/cgit/git/git.git/tree/Documentation/technical/http-protocol.txt#n99
    +## [1] https://github.com/agentzh/chunkin-nginx-module#status
    +## [2] https://github.com/agentzh/chunkin-nginx-module
    +##
    +###################################
    +##         configuration         ##
    +###################################
    +
    +upstream gitlab-workhorse {
    +  server unix:/var/opt/gitlab/gitlab-workhorse/socket;
    +}
    +
    +
    +server {
    +  listen *:80;
    +
    +
    +  server_name much;
    +  server_tokens off; ## Don't show the nginx version number, a security best practice
    +
    +  ## Increase this if you want to upload large attachments
    +  ## Or if you want to accept large git objects over http
    +  client_max_body_size 0;
    +
    +
    +  ## Real IP Module Config
    +  ## http://nginx.org/en/docs/http/ngx_http_realip_module.html
    +
    +  ## HSTS Config
    +  ## https://www.nginx.com/blog/http-strict-transport-security-hsts-and-nginx/
    +  add_header Strict-Transport-Security "max-age=31536000";
    +
    +  ## Individual nginx logs for this GitLab vhost
    +  access_log  /var/log/gitlab/nginx/gitlab_access.log gitlab_access;
    +  error_log   /var/log/gitlab/nginx/gitlab_error.log;
    +
    +  if ($http_host = "") {
    +    set $http_host_with_default "much";
    +  }
    +
    +  if ($http_host != "") {
    +    set $http_host_with_default $http_host;
    +  }
    +
    +  ## If you use HTTPS make sure you disable gzip compression
    +  ## to be safe against BREACH attack.
    +  
    +
    +  ## https://github.com/gitlabhq/gitlabhq/issues/694
    +  ## Some requests take more than 30 seconds.
    +  proxy_read_timeout      3600;
    +  proxy_connect_timeout   300;
    +  proxy_redirect          off;
    +  proxy_http_version 1.1;
    +
    +  proxy_set_header Host $http_host_with_default;
    +  proxy_set_header X-Real-IP $remote_addr;
    +  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    +  proxy_set_header Upgrade $http_upgrade;
    +  proxy_set_header Connection $connection_upgrade;
    +  proxy_set_header X-Forwarded-Proto http;
    +
    +  location ~ (\.git/gitlab-lfs/objects|\.git/info/lfs/objects/batch$) {
    +    proxy_cache off;
    +    proxy_pass http://gitlab-workhorse;
    +    proxy_request_buffering off;
    +  }
    +
    +  location / {
    +    proxy_cache off;
    +    proxy_pass  http://gitlab-workhorse;
    +  }
    +
    +  location /assets {
    +    proxy_cache gitlab;
    +    proxy_pass  http://gitlab-workhorse;
    +  }
    +
    +  error_page 404 /404.html;
    +  error_page 422 /422.html;
    +  error_page 500 /500.html;
    +  error_page 502 /502.html;
    +  location ~ ^/(404|422|500|502)(-custom)?\.html$ {
    +    root /opt/gitlab/embedded/service/gitlab-rails/public;
    +    internal;
    +  }
    +
    +  
    +}
    - change mode from '' to '0644'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * template[/var/opt/gitlab/nginx/conf/gitlab-pages.conf] action delete (up to date)
  * template[/var/opt/gitlab/nginx/conf/gitlab-registry.conf] action delete (up to date)
  * template[/var/opt/gitlab/nginx/conf/gitlab-mattermost-http.conf] action delete (up to date)
  * template[/var/opt/gitlab/nginx/conf/nginx-status.conf] action create
    - create new file /var/opt/gitlab/nginx/conf/nginx-status.conf
    - update content in file /var/opt/gitlab/nginx/conf/nginx-status.conf from none to bee808
    --- /var/opt/gitlab/nginx/conf/nginx-status.conf	2017-07-24 00:14:18.085000000 +0800
    +++ /var/opt/gitlab/nginx/conf/.chef-nginx-status.conf20170724-7201-1oqgb1s	2017-07-24 00:14:18.085000000 +0800
    @@ -1 +1,12 @@
    +server  {
    +    listen *:8060;
    +    server_name localhost;
    +    location /nginx_status {
    +      stub_status on;
    +      server_tokens off;
    +      access_log off;
    +      allow 127.0.0.1;
    +      deny all;
    +    }
    +}
    - change mode from '' to '0644'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * template[/var/opt/gitlab/nginx/conf/nginx.conf] action create
    - create new file /var/opt/gitlab/nginx/conf/nginx.conf
    - update content in file /var/opt/gitlab/nginx/conf/nginx.conf from none to 3e2b68
    --- /var/opt/gitlab/nginx/conf/nginx.conf	2017-07-24 00:14:18.124000000 +0800
    +++ /var/opt/gitlab/nginx/conf/.chef-nginx.conf20170724-7201-17atqes	2017-07-24 00:14:18.124000000 +0800
    @@ -1 +1,53 @@
    +# This file is managed by gitlab-ctl. Manual changes will be
    +# erased! To change the contents below, edit /etc/gitlab/gitlab.rb
    +# and run `sudo gitlab-ctl reconfigure`.
    +
    +user gitlab-www gitlab-www;
    +worker_processes 2;
    +error_log stderr;
    +pid nginx.pid;
    +
    +daemon off;
    +
    +events {
    +  worker_connections 10240;
    +}
    +
    +http {
    +  log_format gitlab_access '$remote_addr - $remote_user [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent"';
    +  log_format gitlab_mattermost_access '$remote_addr - $remote_user [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent"';
    +
    +  server_names_hash_bucket_size 64;
    +
    +  sendfile on;
    +  tcp_nopush on;
    +  tcp_nodelay on;
    +
    +  keepalive_timeout 65;
    +
    +  gzip on;
    +  gzip_http_version 1.0;
    +  gzip_comp_level 2;
    +  gzip_proxied any;
    +  gzip_types text/plain text/css application/x-javascript text/xml application/xml application/xml+rss text/javascript application/json;
    +
    +  include /opt/gitlab/embedded/conf/mime.types;
    +
    +  proxy_cache_path proxy_cache keys_zone=gitlab:10m max_size=1g levels=1:2;
    +  proxy_cache gitlab;
    +
    +  map $http_upgrade $connection_upgrade {
    +      default upgrade;
    +      ''      close;
    +  }
    +
    +  include /var/opt/gitlab/nginx/conf/gitlab-http.conf;
    +
    +
    +
    +
    +  include /var/opt/gitlab/nginx/conf/nginx-status.conf;
    +
    +  
    +}
    - change mode from '' to '0644'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * directory[/opt/gitlab/sv/nginx] action create
    - create new directory /opt/gitlab/sv/nginx
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * directory[/opt/gitlab/sv/nginx/log] action create
    - create new directory /opt/gitlab/sv/nginx/log
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * directory[/opt/gitlab/sv/nginx/log/main] action create
    - create new directory /opt/gitlab/sv/nginx/log/main
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * template[/opt/gitlab/sv/nginx/run] action create
    - create new file /opt/gitlab/sv/nginx/run
    - update content in file /opt/gitlab/sv/nginx/run from none to d75aea
    --- /opt/gitlab/sv/nginx/run	2017-07-24 00:14:18.235000000 +0800
    +++ /opt/gitlab/sv/nginx/.chef-run20170724-7201-jfn1c1	2017-07-24 00:14:18.235000000 +0800
    @@ -1 +1,6 @@
    +#!/bin/sh
    +exec 2>&1
    +
    +cd /var/opt/gitlab/nginx
    +exec chpst -P /opt/gitlab/embedded/sbin/nginx -p /var/opt/gitlab/nginx
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * template[/opt/gitlab/sv/nginx/log/run] action create
    - create new file /opt/gitlab/sv/nginx/log/run
    - update content in file /opt/gitlab/sv/nginx/log/run from none to c70025
    --- /opt/gitlab/sv/nginx/log/run	2017-07-24 00:14:18.266000000 +0800
    +++ /opt/gitlab/sv/nginx/log/.chef-run20170724-7201-1cz7otx	2017-07-24 00:14:18.266000000 +0800
    @@ -1 +1,3 @@
    +#!/bin/sh
    +exec svlogd -tt /var/log/gitlab/nginx
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * template[/var/log/gitlab/nginx/config] action create
    - create new file /var/log/gitlab/nginx/config
    - update content in file /var/log/gitlab/nginx/config from none to 623c00
    --- /var/log/gitlab/nginx/config	2017-07-24 00:14:18.320000000 +0800
    +++ /var/log/gitlab/nginx/.chef-config20170724-7201-129ex69	2017-07-24 00:14:18.320000000 +0800
    @@ -1 +1,7 @@
    +s209715200
    +n30
    +t86400
    +!gzip
    +
    +
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * ruby_block[reload nginx svlogd configuration] action nothing (skipped due to action :nothing)
  * file[/opt/gitlab/sv/nginx/down] action delete (up to date)
  * link[/opt/gitlab/init/nginx] action create
    - create symlink at /opt/gitlab/init/nginx to /opt/gitlab/embedded/bin/sv
  * link[/opt/gitlab/service/nginx] action create
    - create symlink at /opt/gitlab/service/nginx to /opt/gitlab/sv/nginx
  * ruby_block[supervise_nginx_sleep] action run
    - execute the ruby block supervise_nginx_sleep
  * directory[/opt/gitlab/sv/nginx/supervise] action create
    - change mode from '0700' to '0755'
    - restore selinux security context
  * directory[/opt/gitlab/sv/nginx/log/supervise] action create
    - change mode from '0700' to '0755'
    - restore selinux security context
  * file[/opt/gitlab/sv/nginx/supervise/ok] action touch (skipped due to not_if)
  * file[/opt/gitlab/sv/nginx/log/supervise/ok] action touch (skipped due to not_if)
  * file[/opt/gitlab/sv/nginx/supervise/control] action touch (skipped due to not_if)
  * file[/opt/gitlab/sv/nginx/log/supervise/control] action touch (skipped due to not_if)
  * service[nginx] action nothing (skipped due to action :nothing)
  * execute[/opt/gitlab/bin/gitlab-ctl start nginx] action run
    [execute] ok: run: nginx: (pid 8692) 1s
    - execute /opt/gitlab/bin/gitlab-ctl start nginx
Recipe: gitlab::remote-syslog_disable
  * link[/opt/gitlab/service/remote-syslog] action delete (up to date)
  * directory[/opt/gitlab/sv/remote-syslog] action delete (up to date)
Recipe: gitlab::logrotate
  * directory[/opt/gitlab/sv/logrotate] action create
    - create new directory /opt/gitlab/sv/logrotate
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * directory[/opt/gitlab/sv/logrotate/log] action create
    - create new directory /opt/gitlab/sv/logrotate/log
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * directory[/opt/gitlab/sv/logrotate/log/main] action create
    - create new directory /opt/gitlab/sv/logrotate/log/main
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * template[/opt/gitlab/sv/logrotate/run] action create
    - create new file /opt/gitlab/sv/logrotate/run
    - update content in file /opt/gitlab/sv/logrotate/run from none to 07f1b6
    --- /opt/gitlab/sv/logrotate/run	2017-07-24 00:14:20.874000000 +0800
    +++ /opt/gitlab/sv/logrotate/.chef-run20170724-7201-10ceer8	2017-07-24 00:14:20.874000000 +0800
    @@ -1 +1,11 @@
    +#!/bin/sh
    +exec 2>&1
    +
    +cd /var/opt/gitlab/logrotate
    +
    +exec /opt/gitlab/embedded/bin/chpst -P /usr/bin/env \
    +  dir=/var/opt/gitlab/logrotate \
    +  pre_sleep=600 \
    +  post_sleep=3000 \
    +  /opt/gitlab/embedded/bin/gitlab-logrotate-wrapper
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * template[/opt/gitlab/sv/logrotate/log/run] action create
    - create new file /opt/gitlab/sv/logrotate/log/run
    - update content in file /opt/gitlab/sv/logrotate/log/run from none to 94afe6
    --- /opt/gitlab/sv/logrotate/log/run	2017-07-24 00:14:20.906000000 +0800
    +++ /opt/gitlab/sv/logrotate/log/.chef-run20170724-7201-1j1rvb7	2017-07-24 00:14:20.906000000 +0800
    @@ -1 +1,3 @@
    +#!/bin/sh
    +exec svlogd -tt /var/log/gitlab/logrotate
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * template[/var/log/gitlab/logrotate/config] action create
    - create new file /var/log/gitlab/logrotate/config
    - update content in file /var/log/gitlab/logrotate/config from none to 623c00
    --- /var/log/gitlab/logrotate/config	2017-07-24 00:14:20.936000000 +0800
    +++ /var/log/gitlab/logrotate/.chef-config20170724-7201-8pxiq1	2017-07-24 00:14:20.935000000 +0800
    @@ -1 +1,7 @@
    +s209715200
    +n30
    +t86400
    +!gzip
    +
    +
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * ruby_block[reload logrotate svlogd configuration] action nothing (skipped due to action :nothing)
  * file[/opt/gitlab/sv/logrotate/down] action delete (up to date)
  * directory[/opt/gitlab/sv/logrotate/control] action create
    - create new directory /opt/gitlab/sv/logrotate/control
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * template[/opt/gitlab/sv/logrotate/control/t] action create
    - create new file /opt/gitlab/sv/logrotate/control/t
    - update content in file /opt/gitlab/sv/logrotate/control/t from none to 8fa3fa
    --- /opt/gitlab/sv/logrotate/control/t	2017-07-24 00:14:20.994000000 +0800
    +++ /opt/gitlab/sv/logrotate/control/.chef-t20170724-7201-1efcns7	2017-07-24 00:14:20.994000000 +0800
    @@ -1 +1,4 @@
    +#!/bin/sh
    +echo "Received TERM from runit, sending to process group (-PID)"
    +kill -- -$(cat /opt/gitlab/service/logrotate/supervise/pid)
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * link[/opt/gitlab/init/logrotate] action create
    - create symlink at /opt/gitlab/init/logrotate to /opt/gitlab/embedded/bin/sv
  * link[/opt/gitlab/service/logrotate] action create
    - create symlink at /opt/gitlab/service/logrotate to /opt/gitlab/sv/logrotate
  * ruby_block[supervise_logrotate_sleep] action run
    - execute the ruby block supervise_logrotate_sleep
  * directory[/opt/gitlab/sv/logrotate/supervise] action create
    - change mode from '0700' to '0755'
    - restore selinux security context
  * directory[/opt/gitlab/sv/logrotate/log/supervise] action create
    - change mode from '0700' to '0755'
    - restore selinux security context
  * file[/opt/gitlab/sv/logrotate/supervise/ok] action touch (skipped due to not_if)
  * file[/opt/gitlab/sv/logrotate/log/supervise/ok] action touch (skipped due to not_if)
  * file[/opt/gitlab/sv/logrotate/supervise/control] action touch (skipped due to not_if)
  * file[/opt/gitlab/sv/logrotate/log/supervise/control] action touch (skipped due to not_if)
  * service[logrotate] action nothing (skipped due to action :nothing)
  * execute[/opt/gitlab/bin/gitlab-ctl start logrotate] action run
    [execute] ok: run: logrotate: (pid 8774) 1s
    - execute /opt/gitlab/bin/gitlab-ctl start logrotate
Recipe: gitlab::bootstrap
  * file[/var/opt/gitlab/bootstrapped] action create
    - create new file /var/opt/gitlab/bootstrapped
    - update content in file /var/opt/gitlab/bootstrapped from none to 4ae00c
    --- /var/opt/gitlab/bootstrapped	2017-07-24 00:14:26.449000000 +0800
    +++ /var/opt/gitlab/.chef-bootstrapped20170724-7201-tcgw8q	2017-07-24 00:14:26.449000000 +0800
    @@ -1 +1,2 @@
    +All your bootstraps are belong to Chef
    - change mode from '' to '0600'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
Recipe: gitlab::mattermost_disable
  * link[/opt/gitlab/service/mattermost] action delete (up to date)
  * directory[/opt/gitlab/sv/mattermost] action delete (up to date)
Recipe: gitlab::gitlab-pages_disable
  * link[/opt/gitlab/service/gitlab-pages] action delete (up to date)
  * directory[/opt/gitlab/sv/gitlab-pages] action delete (up to date)
Recipe: registry::disable
  * link[/opt/gitlab/service/registry] action delete (up to date)
  * directory[/opt/gitlab/sv/registry] action delete (up to date)
Recipe: gitlab::gitlab-healthcheck
  * template[/opt/gitlab/etc/gitlab-healthcheck-rc] action create
    - create new file /opt/gitlab/etc/gitlab-healthcheck-rc
    - update content in file /opt/gitlab/etc/gitlab-healthcheck-rc from none to 6da55f
    --- /opt/gitlab/etc/gitlab-healthcheck-rc	2017-07-24 00:14:26.483000000 +0800
    +++ /opt/gitlab/etc/.chef-gitlab-healthcheck-rc20170724-7201-14iirf5	2017-07-24 00:14:26.483000000 +0800
    @@ -1 +1,3 @@
    +url='http://localhost:80/help'
    +flags='--insecure'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
Recipe: gitlab::prometheus_user
  * group[Prometheus user and group] action create
    - create group gitlab-prometheus
  * user[Prometheus user and group] action create
    - create user gitlab-prometheus
Recipe: gitlab::prometheus
  * directory[/var/opt/gitlab/prometheus] action create
    - create new directory /var/opt/gitlab/prometheus
    - change mode from '' to '0750'
    - change owner from '' to 'gitlab-prometheus'
    - restore selinux security context
  * directory[/var/log/gitlab/prometheus] action create
    - create new directory /var/log/gitlab/prometheus
    - change mode from '' to '0700'
    - change owner from '' to 'gitlab-prometheus'
    - restore selinux security context
  * file[Prometheus config] action create
    - create new file /var/opt/gitlab/prometheus/prometheus.yml
    - update content in file /var/opt/gitlab/prometheus/prometheus.yml from none to 4fada3
    --- /var/opt/gitlab/prometheus/prometheus.yml	2017-07-24 00:14:26.664000000 +0800
    +++ /var/opt/gitlab/prometheus/.chef-Prometheus config20170724-7201-1eu9fz5	2017-07-24 00:14:26.664000000 +0800
    @@ -1 +1,107 @@
    +---
    +global:
    +  scrape_interval: 15s
    +  scrape_timeout: 15s
    +scrape_configs:
    +- job_name: prometheus
    +  static_configs:
    +  - targets:
    +    - localhost:9090
    +- job_name: redis
    +  static_configs:
    +  - targets:
    +    - localhost:9121
    +- job_name: postgres
    +  static_configs:
    +  - targets:
    +    - localhost:9187
    +- job_name: node
    +  static_configs:
    +  - targets:
    +    - localhost:9100
    +- job_name: gitlab-unicorn
    +  metrics_path: "/-/metrics"
    +  static_configs:
    +  - targets:
    +    - 127.0.0.1:8080
    +- job_name: gitlab_monitor_database
    +  metrics_path: "/database"
    +  static_configs:
    +  - targets:
    +    - localhost:9168
    +- job_name: gitlab_monitor_sidekiq
    +  metrics_path: "/sidekiq"
    +  static_configs:
    +  - targets:
    +    - localhost:9168
    +- job_name: gitlab_monitor_process
    +  metrics_path: "/process"
    +  static_configs:
    +  - targets:
    +    - localhost:9168
    +- job_name: kubernetes-nodes
    +  scheme: https
    +  tls_config:
    +    ca_file: "/var/run/secrets/kubernetes.io/serviceaccount/ca.crt"
    +    insecure_skip_verify: true
    +  bearer_token_file: "/var/run/secrets/kubernetes.io/serviceaccount/token"
    +  kubernetes_sd_configs:
    +  - role: node
    +    api_server: https://kubernetes.default.svc:443
    +    tls_config:
    +      ca_file: "/var/run/secrets/kubernetes.io/serviceaccount/ca.crt"
    +    bearer_token_file: "/var/run/secrets/kubernetes.io/serviceaccount/token"
    +  relabel_configs:
    +  - action: labelmap
    +    regex: __meta_kubernetes_node_label_(.+)
    +  - target_label: __address__
    +    replacement: kubernetes.default.svc:443
    +  - source_labels:
    +    - __meta_kubernetes_node_name
    +    regex: "(.+)"
    +    target_label: __metrics_path__
    +    replacement: "/api/v1/nodes/${1}/proxy/metrics"
    +  metric_relabel_configs:
    +  - source_labels:
    +    - pod_name
    +    target_label: environment
    +    regex: "(.+)-.+-.+"
    +- job_name: kubernetes-pods
    +  tls_config:
    +    ca_file: "/var/run/secrets/kubernetes.io/serviceaccount/ca.crt"
    +    insecure_skip_verify: true
    +  bearer_token_file: "/var/run/secrets/kubernetes.io/serviceaccount/token"
    +  kubernetes_sd_configs:
    +  - role: pod
    +    api_server: https://kubernetes.default.svc:443
    +    tls_config:
    +      ca_file: "/var/run/secrets/kubernetes.io/serviceaccount/ca.crt"
    +    bearer_token_file: "/var/run/secrets/kubernetes.io/serviceaccount/token"
    +  relabel_configs:
    +  - source_labels:
    +    - __meta_kubernetes_pod_annotation_prometheus_io_scrape
    +    action: keep
    +    regex: 'true'
    +  - source_labels:
    +    - __meta_kubernetes_pod_annotation_prometheus_io_path
    +    action: replace
    +    target_label: __metrics_path__
    +    regex: "(.+)"
    +  - source_labels:
    +    - __address__
    +    - __meta_kubernetes_pod_annotation_prometheus_io_port
    +    action: replace
    +    regex: "([^:]+)(?::[0-9]+)?;([0-9]+)"
    +    replacement: "$1:$2"
    +    target_label: __address__
    +  - action: labelmap
    +    regex: __meta_kubernetes_pod_label_(.+)
    +  - source_labels:
    +    - __meta_kubernetes_namespace
    +    action: replace
    +    target_label: kubernetes_namespace
    +  - source_labels:
    +    - __meta_kubernetes_pod_name
    +    action: replace
    +    target_label: kubernetes_pod_name
    - change mode from '' to '0644'
    - change owner from '' to 'gitlab-prometheus'
    - restore selinux security context
  * directory[/opt/gitlab/sv/prometheus] action create
    - create new directory /opt/gitlab/sv/prometheus
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * directory[/opt/gitlab/sv/prometheus/log] action create
    - create new directory /opt/gitlab/sv/prometheus/log
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * directory[/opt/gitlab/sv/prometheus/log/main] action create
    - create new directory /opt/gitlab/sv/prometheus/log/main
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * template[/opt/gitlab/sv/prometheus/run] action create
    - create new file /opt/gitlab/sv/prometheus/run
    - update content in file /opt/gitlab/sv/prometheus/run from none to 04c51e
    --- /opt/gitlab/sv/prometheus/run	2017-07-24 00:14:26.770000000 +0800
    +++ /opt/gitlab/sv/prometheus/.chef-run20170724-7201-17pbqs9	2017-07-24 00:14:26.770000000 +0800
    @@ -1 +1,7 @@
    +#!/bin/sh
    +exec 2>&1
    +
    +umask 077
    +exec chpst -P -U gitlab-prometheus -u gitlab-prometheus \
    +  /opt/gitlab/embedded/bin/prometheus -web.listen-address=localhost:9090 -storage.local.path=/var/opt/gitlab/prometheus/data -storage.local.chunk-encoding-version=2 -storage.local.target-heap-size=68168089 -config.file=/var/opt/gitlab/prometheus/prometheus.yml
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * template[/opt/gitlab/sv/prometheus/log/run] action create
    - create new file /opt/gitlab/sv/prometheus/log/run
    - update content in file /opt/gitlab/sv/prometheus/log/run from none to 072b20
    --- /opt/gitlab/sv/prometheus/log/run	2017-07-24 00:14:26.798000000 +0800
    +++ /opt/gitlab/sv/prometheus/log/.chef-run20170724-7201-1m8v4sg	2017-07-24 00:14:26.798000000 +0800
    @@ -1 +1,3 @@
    +#!/bin/sh
    +exec svlogd -tt /var/log/gitlab/prometheus
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * template[/var/log/gitlab/prometheus/config] action create
    - create new file /var/log/gitlab/prometheus/config
    - update content in file /var/log/gitlab/prometheus/config from none to 623c00
    --- /var/log/gitlab/prometheus/config	2017-07-24 00:14:26.825000000 +0800
    +++ /var/log/gitlab/prometheus/.chef-config20170724-7201-lrdhvh	2017-07-24 00:14:26.825000000 +0800
    @@ -1 +1,7 @@
    +s209715200
    +n30
    +t86400
    +!gzip
    +
    +
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * ruby_block[reload prometheus svlogd configuration] action nothing (skipped due to action :nothing)
  * file[/opt/gitlab/sv/prometheus/down] action delete (up to date)
  * link[/opt/gitlab/init/prometheus] action create
    - create symlink at /opt/gitlab/init/prometheus to /opt/gitlab/embedded/bin/sv
  * link[/opt/gitlab/service/prometheus] action create
    - create symlink at /opt/gitlab/service/prometheus to /opt/gitlab/sv/prometheus
  * ruby_block[supervise_prometheus_sleep] action run
    - execute the ruby block supervise_prometheus_sleep
  * directory[/opt/gitlab/sv/prometheus/supervise] action create
    - change mode from '0700' to '0755'
    - restore selinux security context
  * directory[/opt/gitlab/sv/prometheus/log/supervise] action create
    - change mode from '0700' to '0755'
    - restore selinux security context
  * file[/opt/gitlab/sv/prometheus/supervise/ok] action touch (skipped due to not_if)
  * file[/opt/gitlab/sv/prometheus/log/supervise/ok] action touch (skipped due to not_if)
  * file[/opt/gitlab/sv/prometheus/supervise/control] action touch (skipped due to not_if)
  * file[/opt/gitlab/sv/prometheus/log/supervise/control] action touch (skipped due to not_if)
  * service[prometheus] action nothing (skipped due to action :nothing)
  * execute[/opt/gitlab/bin/gitlab-ctl start prometheus] action run
    [execute] ok: run: prometheus: (pid 8854) 1s
    - execute /opt/gitlab/bin/gitlab-ctl start prometheus
Recipe: gitlab::node-exporter
  * directory[/var/log/gitlab/node-exporter] action create
    - create new directory /var/log/gitlab/node-exporter
    - change mode from '' to '0700'
    - change owner from '' to 'gitlab-prometheus'
    - restore selinux security context
  * directory[/var/opt/gitlab/node-exporter/textfile_collector] action create
    - create new directory /var/opt/gitlab/node-exporter/textfile_collector
    - change mode from '' to '0755'
    - change owner from '' to 'gitlab-prometheus'
    - restore selinux security context
  * directory[/opt/gitlab/sv/node-exporter] action create
    - create new directory /opt/gitlab/sv/node-exporter
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * directory[/opt/gitlab/sv/node-exporter/log] action create
    - create new directory /opt/gitlab/sv/node-exporter/log
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * directory[/opt/gitlab/sv/node-exporter/log/main] action create
    - create new directory /opt/gitlab/sv/node-exporter/log/main
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * template[/opt/gitlab/sv/node-exporter/run] action create
    - create new file /opt/gitlab/sv/node-exporter/run
    - update content in file /opt/gitlab/sv/node-exporter/run from none to 09c3e6
    --- /opt/gitlab/sv/node-exporter/run	2017-07-24 00:14:32.365000000 +0800
    +++ /opt/gitlab/sv/node-exporter/.chef-run20170724-7201-1heaaqx	2017-07-24 00:14:32.365000000 +0800
    @@ -1 +1,6 @@
    +#!/bin/sh
    +exec 2>&1
    +
    +umask 077
    +exec chpst -P -U gitlab-prometheus -u gitlab-prometheus /opt/gitlab/embedded/bin/node_exporter -web.listen-address=localhost:9100 -collector.textfile.directory=/var/opt/gitlab/node-exporter/textfile_collector
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * template[/opt/gitlab/sv/node-exporter/log/run] action create
    - create new file /opt/gitlab/sv/node-exporter/log/run
    - update content in file /opt/gitlab/sv/node-exporter/log/run from none to ae1796
    --- /opt/gitlab/sv/node-exporter/log/run	2017-07-24 00:14:32.396000000 +0800
    +++ /opt/gitlab/sv/node-exporter/log/.chef-run20170724-7201-1nuh44d	2017-07-24 00:14:32.396000000 +0800
    @@ -1 +1,3 @@
    +#!/bin/sh
    +exec svlogd -tt /var/log/gitlab/node-exporter
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * template[/var/log/gitlab/node-exporter/config] action create
    - create new file /var/log/gitlab/node-exporter/config
    - update content in file /var/log/gitlab/node-exporter/config from none to 623c00
    --- /var/log/gitlab/node-exporter/config	2017-07-24 00:14:32.424000000 +0800
    +++ /var/log/gitlab/node-exporter/.chef-config20170724-7201-6cff4a	2017-07-24 00:14:32.424000000 +0800
    @@ -1 +1,7 @@
    +s209715200
    +n30
    +t86400
    +!gzip
    +
    +
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * ruby_block[reload node-exporter svlogd configuration] action nothing (skipped due to action :nothing)
  * file[/opt/gitlab/sv/node-exporter/down] action delete (up to date)
  * link[/opt/gitlab/init/node-exporter] action create
    - create symlink at /opt/gitlab/init/node-exporter to /opt/gitlab/embedded/bin/sv
  * link[/opt/gitlab/service/node-exporter] action create
    - create symlink at /opt/gitlab/service/node-exporter to /opt/gitlab/sv/node-exporter
  * ruby_block[supervise_node-exporter_sleep] action run
    - execute the ruby block supervise_node-exporter_sleep
  * directory[/opt/gitlab/sv/node-exporter/supervise] action create
    - change mode from '0700' to '0755'
    - restore selinux security context
  * directory[/opt/gitlab/sv/node-exporter/log/supervise] action create
    - change mode from '0700' to '0755'
    - restore selinux security context
  * file[/opt/gitlab/sv/node-exporter/supervise/ok] action touch (skipped due to not_if)
  * file[/opt/gitlab/sv/node-exporter/log/supervise/ok] action touch (skipped due to not_if)
  * file[/opt/gitlab/sv/node-exporter/supervise/control] action touch (skipped due to not_if)
  * file[/opt/gitlab/sv/node-exporter/log/supervise/control] action touch (skipped due to not_if)
  * service[node-exporter] action nothing (skipped due to action :nothing)
  * execute[/opt/gitlab/bin/gitlab-ctl start node-exporter] action run
    [execute] ok: run: node-exporter: (pid 8904) 1s
    - execute /opt/gitlab/bin/gitlab-ctl start node-exporter
Recipe: gitlab::redis-exporter
  * directory[/var/log/gitlab/redis-exporter] action create
    - create new directory /var/log/gitlab/redis-exporter
    - change mode from '' to '0700'
    - change owner from '' to 'gitlab-redis'
    - restore selinux security context
  * directory[/opt/gitlab/sv/redis-exporter] action create
    - create new directory /opt/gitlab/sv/redis-exporter
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * directory[/opt/gitlab/sv/redis-exporter/log] action create
    - create new directory /opt/gitlab/sv/redis-exporter/log
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * directory[/opt/gitlab/sv/redis-exporter/log/main] action create
    - create new directory /opt/gitlab/sv/redis-exporter/log/main
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * template[/opt/gitlab/sv/redis-exporter/run] action create
    - create new file /opt/gitlab/sv/redis-exporter/run
    - update content in file /opt/gitlab/sv/redis-exporter/run from none to fa3e94
    --- /opt/gitlab/sv/redis-exporter/run	2017-07-24 00:14:34.987000000 +0800
    +++ /opt/gitlab/sv/redis-exporter/.chef-run20170724-7201-uzlsn	2017-07-24 00:14:34.987000000 +0800
    @@ -1 +1,6 @@
    +#!/bin/sh
    +exec 2>&1
    +
    +umask 077
    +exec chpst -P -U gitlab-redis:git -u gitlab-redis:git /opt/gitlab/embedded/bin/redis_exporter -web.listen-address=localhost:9121 -redis.addr=unix:///var/opt/gitlab/redis/redis.socket
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * template[/opt/gitlab/sv/redis-exporter/log/run] action create
    - create new file /opt/gitlab/sv/redis-exporter/log/run
    - update content in file /opt/gitlab/sv/redis-exporter/log/run from none to 082dea
    --- /opt/gitlab/sv/redis-exporter/log/run	2017-07-24 00:14:35.023000000 +0800
    +++ /opt/gitlab/sv/redis-exporter/log/.chef-run20170724-7201-x9z5r9	2017-07-24 00:14:35.023000000 +0800
    @@ -1 +1,3 @@
    +#!/bin/sh
    +exec svlogd -tt /var/log/gitlab/redis-exporter
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * template[/var/log/gitlab/redis-exporter/config] action create
    - create new file /var/log/gitlab/redis-exporter/config
    - update content in file /var/log/gitlab/redis-exporter/config from none to 623c00
    --- /var/log/gitlab/redis-exporter/config	2017-07-24 00:14:35.061000000 +0800
    +++ /var/log/gitlab/redis-exporter/.chef-config20170724-7201-1wld103	2017-07-24 00:14:35.061000000 +0800
    @@ -1 +1,7 @@
    +s209715200
    +n30
    +t86400
    +!gzip
    +
    +
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * ruby_block[reload redis-exporter svlogd configuration] action nothing (skipped due to action :nothing)
  * file[/opt/gitlab/sv/redis-exporter/down] action delete (up to date)
  * link[/opt/gitlab/init/redis-exporter] action create
    - create symlink at /opt/gitlab/init/redis-exporter to /opt/gitlab/embedded/bin/sv
  * link[/opt/gitlab/service/redis-exporter] action create
    - create symlink at /opt/gitlab/service/redis-exporter to /opt/gitlab/sv/redis-exporter
  * ruby_block[supervise_redis-exporter_sleep] action run
    - execute the ruby block supervise_redis-exporter_sleep
  * directory[/opt/gitlab/sv/redis-exporter/supervise] action create
    - change mode from '0700' to '0755'
    - restore selinux security context
  * directory[/opt/gitlab/sv/redis-exporter/log/supervise] action create
    - change mode from '0700' to '0755'
    - restore selinux security context
  * file[/opt/gitlab/sv/redis-exporter/supervise/ok] action touch (skipped due to not_if)
  * file[/opt/gitlab/sv/redis-exporter/log/supervise/ok] action touch (skipped due to not_if)
  * file[/opt/gitlab/sv/redis-exporter/supervise/control] action touch (skipped due to not_if)
  * file[/opt/gitlab/sv/redis-exporter/log/supervise/control] action touch (skipped due to not_if)
  * service[redis-exporter] action nothing (skipped due to action :nothing)
  * execute[/opt/gitlab/bin/gitlab-ctl start redis-exporter] action run
    [execute] ok: run: redis-exporter: (pid 8953) 1s
    - execute /opt/gitlab/bin/gitlab-ctl start redis-exporter
Recipe: gitlab::postgres-exporter
  * directory[/var/log/gitlab/postgres-exporter] action create
    - create new directory /var/log/gitlab/postgres-exporter
    - change mode from '' to '0700'
    - change owner from '' to 'gitlab-psql'
    - restore selinux security context
  * directory[/var/opt/gitlab/postgres-exporter] action create
    - create new directory /var/opt/gitlab/postgres-exporter
    - change mode from '' to '0700'
    - change owner from '' to 'gitlab-psql'
    - restore selinux security context
  * directory[/opt/gitlab/etc/postgres-exporter/env] action create
    - create new directory /opt/gitlab/etc/postgres-exporter/env
    - restore selinux security context
  * file[/opt/gitlab/etc/postgres-exporter/env/DATA_SOURCE_NAME] action create
    - create new file /opt/gitlab/etc/postgres-exporter/env/DATA_SOURCE_NAME
    - update content in file /opt/gitlab/etc/postgres-exporter/env/DATA_SOURCE_NAME from none to cd58e5
    --- /opt/gitlab/etc/postgres-exporter/env/DATA_SOURCE_NAME	2017-07-24 00:14:40.575000000 +0800
    +++ /opt/gitlab/etc/postgres-exporter/env/.chef-DATA_SOURCE_NAME20170724-7201-1babw1w	2017-07-24 00:14:40.575000000 +0800
    @@ -1 +1,2 @@
    +user=gitlab-psql host=/var/opt/gitlab/postgresql database=postgres sslmode=allow
    - restore selinux security context
  * directory[/opt/gitlab/sv/postgres-exporter] action create
    - create new directory /opt/gitlab/sv/postgres-exporter
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * directory[/opt/gitlab/sv/postgres-exporter/log] action create
    - create new directory /opt/gitlab/sv/postgres-exporter/log
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * directory[/opt/gitlab/sv/postgres-exporter/log/main] action create
    - create new directory /opt/gitlab/sv/postgres-exporter/log/main
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * template[/opt/gitlab/sv/postgres-exporter/run] action create
    - create new file /opt/gitlab/sv/postgres-exporter/run
    - update content in file /opt/gitlab/sv/postgres-exporter/run from none to ee42a2
    --- /opt/gitlab/sv/postgres-exporter/run	2017-07-24 00:14:40.686000000 +0800
    +++ /opt/gitlab/sv/postgres-exporter/.chef-run20170724-7201-1tg3ap3	2017-07-24 00:14:40.686000000 +0800
    @@ -1 +1,7 @@
    +#!/bin/sh
    +exec 2>&1
    +
    +umask 077
    +exec chpst -e /opt/gitlab/etc/postgres-exporter/env -P -U gitlab-psql:git -u gitlab-psql:git /opt/gitlab/embedded/bin/postgres_exporter -web.listen-address=localhost:9187 -extend.query-path=/var/opt/gitlab/postgres-exporter/queries.yaml
    +
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * template[/opt/gitlab/sv/postgres-exporter/log/run] action create
    - create new file /opt/gitlab/sv/postgres-exporter/log/run
    - update content in file /opt/gitlab/sv/postgres-exporter/log/run from none to b971c9
    --- /opt/gitlab/sv/postgres-exporter/log/run	2017-07-24 00:14:40.715000000 +0800
    +++ /opt/gitlab/sv/postgres-exporter/log/.chef-run20170724-7201-np8ecy	2017-07-24 00:14:40.715000000 +0800
    @@ -1 +1,3 @@
    +#!/bin/sh
    +exec svlogd -tt /var/log/gitlab/postgres-exporter
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * template[/var/log/gitlab/postgres-exporter/config] action create
    - create new file /var/log/gitlab/postgres-exporter/config
    - update content in file /var/log/gitlab/postgres-exporter/config from none to 623c00
    --- /var/log/gitlab/postgres-exporter/config	2017-07-24 00:14:40.747000000 +0800
    +++ /var/log/gitlab/postgres-exporter/.chef-config20170724-7201-17o1m74	2017-07-24 00:14:40.747000000 +0800
    @@ -1 +1,7 @@
    +s209715200
    +n30
    +t86400
    +!gzip
    +
    +
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * ruby_block[reload postgres-exporter svlogd configuration] action nothing (skipped due to action :nothing)
  * file[/opt/gitlab/sv/postgres-exporter/down] action delete (up to date)
  * link[/opt/gitlab/init/postgres-exporter] action create
    - create symlink at /opt/gitlab/init/postgres-exporter to /opt/gitlab/embedded/bin/sv
  * link[/opt/gitlab/service/postgres-exporter] action create
    - create symlink at /opt/gitlab/service/postgres-exporter to /opt/gitlab/sv/postgres-exporter
  * ruby_block[supervise_postgres-exporter_sleep] action run
    - execute the ruby block supervise_postgres-exporter_sleep
  * directory[/opt/gitlab/sv/postgres-exporter/supervise] action create
    - change mode from '0700' to '0755'
    - restore selinux security context
  * directory[/opt/gitlab/sv/postgres-exporter/log/supervise] action create
    - change mode from '0700' to '0755'
    - restore selinux security context
  * file[/opt/gitlab/sv/postgres-exporter/supervise/ok] action touch (skipped due to not_if)
  * file[/opt/gitlab/sv/postgres-exporter/log/supervise/ok] action touch (skipped due to not_if)
  * file[/opt/gitlab/sv/postgres-exporter/supervise/control] action touch (skipped due to not_if)
  * file[/opt/gitlab/sv/postgres-exporter/log/supervise/control] action touch (skipped due to not_if)
  * service[postgres-exporter] action nothing (skipped due to action :nothing)
  * template[/var/opt/gitlab/postgres-exporter/queries.yaml] action create
    - create new file /var/opt/gitlab/postgres-exporter/queries.yaml
    - update content in file /var/opt/gitlab/postgres-exporter/queries.yaml from none to f7537d
    --- /var/opt/gitlab/postgres-exporter/queries.yaml	2017-07-24 00:14:42.847000000 +0800
    +++ /var/opt/gitlab/postgres-exporter/.chef-queries.yaml20170724-7201-14qvjoc	2017-07-24 00:14:42.847000000 +0800
    @@ -1 +1,174 @@
    +pg_replication:
    +  query: "SELECT EXTRACT(EPOCH FROM (now() - pg_last_xact_replay_timestamp()))::INT as lag"
    +  metrics:
    +    - lag:
    +        usage: "GAUGE"
    +        description: "Replication lag behind master in seconds"
    +
    +pg_postmaster:
    +  query: "SELECT pg_postmaster_start_time as start_time_seconds from pg_postmaster_start_time()"
    +  metrics:
    +    - start_time_seconds:
    +        usage: "GAUGE"
    +        description: "Time at which postmaster started"
    +
    +pg_settings_shared_buffers:
    +  query: "SELECT 8192*setting::bigint as bytes from pg_settings where name = 'shared_buffers'"
    +  metrics:
    +    - bytes:
    +        usage: "GAUGE"
    +        description: "Size of shared_buffers"
    +
    +pg_settings_checkpoint:
    +  query: "select (select setting::int from pg_settings where name = 'checkpoint_segments') as segments, (select setting::int from pg_settings where name = 'checkpoint_timeout') as timeout_seconds, (select setting::float from pg_settings where name = 'checkpoint_completion_target') as completion_target"
    +  metrics:
    +    - segments:
    +        usage: "GAUGE"
    +        description: "Number of checkpoint segments"
    +    - timeout_seconds:
    +        usage: "GAUGE"
    +        description: "Checkpoint timeout in seconds"
    +    - completion_target:
    +        usage: "GAUGE"
    +        description: "Checkpoint completion target, ranging from 0 to 1"
    +
    +pg_stat_user_tables:
    +  query: "SELECT schemaname, relname, seq_scan, seq_tup_read, idx_scan, idx_tup_fetch, n_tup_ins, n_tup_upd, n_tup_del, n_tup_hot_upd, n_live_tup, n_dead_tup, n_mod_since_analyze, last_vacuum, last_autovacuum, last_analyze, last_autoanalyze, vacuum_count, autovacuum_count, analyze_count, autoanalyze_count FROM pg_stat_user_tables"
    +  metrics:
    +    - schemaname:
    +        usage: "LABEL"
    +        description: "Name of the schema that this table is in"
    +    - relname:
    +        usage: "LABEL"
    +        description: "Name of this table"
    +    - seq_scan:
    +        usage: "COUNTER"
    +        description: "Number of sequential scans initiated on this table"
    +    - seq_tup_read:
    +        usage: "COUNTER"
    +        description: "Number of live rows fetched by sequential scans"
    +    - idx_scan:
    +        usage: "COUNTER"
    +        description: "Number of index scans initiated on this table"
    +    - idx_tup_fetch:
    +        usage: "COUNTER"
    +        description: "Number of live rows fetched by index scans"
    +    - n_tup_ins:
    +        usage: "COUNTER"
    +        description: "Number of rows inserted"
    +    - n_tup_upd:
    +        usage: "COUNTER"
    +        description: "Number of rows updated"
    +    - n_tup_del:
    +        usage: "COUNTER"
    +        description: "Number of rows deleted"
    +    - n_tup_hot_upd:
    +        usage: "COUNTER"
    +        description: "Number of rows HOT updated (i.e., with no separate index update required)"
    +    - n_live_tup:
    +        usage: "GAUGE"
    +        description: "Estimated number of live rows"
    +    - n_dead_tup:
    +        usage: "GAUGE"
    +        description: "Estimated number of dead rows"
    +    - n_mod_since_analyze:
    +        usage: "GAUGE"
    +        description: "Estimated number of rows changed since last analyze"
    +    - last_vacuum:
    +        usage: "GAUGE"
    +        description: "Last time at which this table was manually vacuumed (not counting VACUUM FULL)"
    +    - last_autovacuum:
    +        usage: "GAUGE"
    +        description: "Last time at which this table was vacuumed by the autovacuum daemon"
    +    - last_analyze:
    +        usage: "GAUGE"
    +        description: "Last time at which this table was manually analyzed"
    +    - last_autoanalyze:
    +        usage: "GAUGE"
    +        description: "Last time at which this table was analyzed by the autovacuum daemon"
    +    - vacuum_count:
    +        usage: "COUNTER"
    +        description: "Number of times this table has been manually vacuumed (not counting VACUUM FULL)"
    +    - autovacuum_count:
    +        usage: "COUNTER"
    +        description: "Number of times this table has been vacuumed by the autovacuum daemon"
    +    - analyze_count:
    +        usage: "COUNTER"
    +        description: "Number of times this table has been manually analyzed"
    +    - autoanalyze_count:
    +        usage: "COUNTER"
    +        description: "Number of times this table has been analyzed by the autovacuum daemon"
    +
    +pg_blocked:
    +  query: |
    +    SELECT
    +      count(blocked.transactionid) AS queries,
    +      '__transaction__' AS table
    +    FROM pg_catalog.pg_locks blocked
    +    WHERE NOT blocked.granted AND locktype = 'transactionid'
    +    GROUP BY locktype
    +    UNION
    +    SELECT
    +      count(blocked.relation) AS queries,
    +      blocked.relation::regclass::text AS table
    +    FROM pg_catalog.pg_locks blocked
    +    WHERE NOT blocked.granted AND locktype != 'transactionid'
    +    GROUP BY relation
    +  metrics:
    +    - queries:
    +        usage: "GAUGE"
    +        description: "The current number of blocked queries"
    +    - table:
    +        usage: "LABEL"
    +        description: "The table on which a query is blocked"
    +
    +pg_slow:
    +  query: |
    +    SELECT COUNT(*) AS queries
    +    FROM pg_stat_activity
    +    WHERE state = 'active' AND (now() - query_start) > '1 seconds'::interval
    +  metrics:
    +    - queries:
    +        usage: "GAUGE"
    +        description: "Current number of slow queries"
    +
    +pg_vacuum:
    +  query: |
    +    SELECT
    +      COUNT(*) AS queries,
    +      MAX(EXTRACT(EPOCH FROM (clock_timestamp() - query_start))) AS age_in_seconds
    +    FROM pg_catalog.pg_stat_activity
    +    WHERE state = 'active' AND trim(query) ~* '\AVACUUM (?!ANALYZE)'
    +  metrics:
    +    - queries:
    +        usage: "GAUGE"
    +        description: "The current number of VACUUM queries"
    +    - age_in_seconds:
    +        usage: "GAUGE"
    +        description: "The current maximum VACUUM query age in seconds"
    +
    +pg_vacuum_analyze:
    +  query: |
    +    SELECT
    +      COUNT(*) AS queries,
    +      MAX(EXTRACT(EPOCH FROM (clock_timestamp() - query_start))) AS age_in_seconds
    +    FROM pg_catalog.pg_stat_activity
    +    WHERE state = 'active' AND trim(query) ~* '\AVACUUM ANALYZE'
    +  metrics:
    +    - queries:
    +        usage: "GAUGE"
    +        description: "The current number of VACUUM ANALYZE queries"
    +    - age_in_seconds:
    +        usage: "GAUGE"
    +        description: "The current maximum VACUUM ANALYZE query age in seconds"
    +
    +pg_stuck_idle_in_transaction:
    +  query: |
    +    SELECT COUNT(*) AS queries
    +    FROM pg_stat_activity
    +    WHERE state = 'idle in transaction' AND (now() - query_start) > '10 minutes'::interval
    +  metrics:
    +    - queries:
    +        usage: "GAUGE"
    +        description: "Current number of queries that are stuck being idle in transactions"
    - change mode from '' to '0644'
    - change owner from '' to 'gitlab-psql'
    - restore selinux security context
  * execute[/opt/gitlab/bin/gitlab-ctl start postgres-exporter] action run
    [execute] ok: run: postgres-exporter: (pid 9016) 2s
    - execute /opt/gitlab/bin/gitlab-ctl start postgres-exporter
Recipe: gitlab::gitlab-monitor
  * directory[/var/opt/gitlab/gitlab-monitor] action create
    - create new directory /var/opt/gitlab/gitlab-monitor
    - change mode from '' to '0755'
    - change owner from '' to 'git'
    - restore selinux security context
  * directory[/var/log/gitlab/gitlab-monitor] action create
    - create new directory /var/log/gitlab/gitlab-monitor
    - change mode from '' to '0700'
    - change owner from '' to 'git'
    - restore selinux security context
  * template[/var/opt/gitlab/gitlab-monitor/gitlab-monitor.yml] action create
    - create new file /var/opt/gitlab/gitlab-monitor/gitlab-monitor.yml
    - update content in file /var/opt/gitlab/gitlab-monitor/gitlab-monitor.yml from none to d96040
    --- /var/opt/gitlab/gitlab-monitor/gitlab-monitor.yml	2017-07-24 00:14:43.266000000 +0800
    +++ /var/opt/gitlab/gitlab-monitor/.chef-gitlab-monitor.yml20170724-7201-un513k	2017-07-24 00:14:43.266000000 +0800
    @@ -1 +1,73 @@
    +db_common: &db_common
    +  methods:
    +    - probe_db
    +  opts:
    +    connection_string: dbname=gitlabhq_production user=gitlab host=/var/opt/gitlab/postgresql port=5432 password=
    +
    +common_git: &common_git
    +  class_name: GitProber
    +  methods:
    +    - probe_pull
    +    - probe_push
    +
    +# Web server config
    +server:
    +  listen_address: localhost
    +  listen_port: 9168
    +
    +# Probes config
    +probes:
    +  git:
    +    multiple: true
    +    default:
    +      <<: *common_git
    +      opts:
    +        source: /var/opt/gitlab/git-data
    +        labels:
    +          name: default
    +
    +  git_process:
    +    class_name: GitProcessProber # `class_name` is redundant here
    +    methods:
    +    - probe_git
    +    opts:
    +      quantiles: true
    +
    +  # We can group multiple probes under a single endpoint by setting the `multiple` key to `true`, followed
    +  # by probe definitions as usual.
    +  database:
    +    multiple: true
    +    ci_builds:
    +      class_name: Database::CiBuildsProber
    +      <<: *db_common
    +    tuple_stats:
    +      class_name: Database::TuplesProber
    +      <<: *db_common
    +    rows_count:
    +      class_name: Database::RowCountProber
    +      <<: *db_common
    +
    +  process:
    +    methods:
    +      - probe_memory
    +      - probe_age
    +      - probe_count
    +    opts:
    +      - pid_or_pattern: "sidekiq .* \\[.*?\\]"
    +        name: sidekiq
    +      - pid_or_pattern: "unicorn worker\\[.*?\\]"
    +        name: unicorn
    +      - pid_or_pattern: "git-upload-pack --stateless-rpc"
    +        name: git_upload_pack
    +        quantiles: true
    +
    +  sidekiq:
    +    methods:
    +      - probe_queues
    +      - probe_jobs
    +      - probe_workers
    +      - probe_retries
    +      - probe_dead
    +    opts:
    +      redis_url: "unix:/var/opt/gitlab/redis/redis.socket"
    - change mode from '' to '0644'
    - change owner from '' to 'git'
    - restore selinux security context
  * directory[/opt/gitlab/sv/gitlab-monitor] action create
    - create new directory /opt/gitlab/sv/gitlab-monitor
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * directory[/opt/gitlab/sv/gitlab-monitor/log] action create
    - create new directory /opt/gitlab/sv/gitlab-monitor/log
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * directory[/opt/gitlab/sv/gitlab-monitor/log/main] action create
    - create new directory /opt/gitlab/sv/gitlab-monitor/log/main
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * template[/opt/gitlab/sv/gitlab-monitor/run] action create
    - create new file /opt/gitlab/sv/gitlab-monitor/run
    - update content in file /opt/gitlab/sv/gitlab-monitor/run from none to 2f480f
    --- /opt/gitlab/sv/gitlab-monitor/run	2017-07-24 00:14:43.367000000 +0800
    +++ /opt/gitlab/sv/gitlab-monitor/.chef-run20170724-7201-2r59vp	2017-07-24 00:14:43.367000000 +0800
    @@ -1 +1,6 @@
    +#!/bin/sh
    +exec 2>&1
    +
    +umask 077
    +exec chpst -P -U git -u git /opt/gitlab/embedded/bin/gitlab-mon web -c /var/opt/gitlab/gitlab-monitor/gitlab-monitor.yml
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * template[/opt/gitlab/sv/gitlab-monitor/log/run] action create
    - create new file /opt/gitlab/sv/gitlab-monitor/log/run
    - update content in file /opt/gitlab/sv/gitlab-monitor/log/run from none to be403a
    --- /opt/gitlab/sv/gitlab-monitor/log/run	2017-07-24 00:14:43.393000000 +0800
    +++ /opt/gitlab/sv/gitlab-monitor/log/.chef-run20170724-7201-2fv0yr	2017-07-24 00:14:43.393000000 +0800
    @@ -1 +1,3 @@
    +#!/bin/sh
    +exec svlogd -tt /var/log/gitlab/gitlab-monitor
    - change mode from '' to '0755'
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * template[/var/log/gitlab/gitlab-monitor/config] action create
    - create new file /var/log/gitlab/gitlab-monitor/config
    - update content in file /var/log/gitlab/gitlab-monitor/config from none to 623c00
    --- /var/log/gitlab/gitlab-monitor/config	2017-07-24 00:14:43.420000000 +0800
    +++ /var/log/gitlab/gitlab-monitor/.chef-config20170724-7201-1nji8o7	2017-07-24 00:14:43.420000000 +0800
    @@ -1 +1,7 @@
    +s209715200
    +n30
    +t86400
    +!gzip
    +
    +
    - change owner from '' to 'root'
    - change group from '' to 'root'
    - restore selinux security context
  * ruby_block[reload gitlab-monitor svlogd configuration] action nothing (skipped due to action :nothing)
  * file[/opt/gitlab/sv/gitlab-monitor/down] action delete (up to date)
  * link[/opt/gitlab/init/gitlab-monitor] action create
    - create symlink at /opt/gitlab/init/gitlab-monitor to /opt/gitlab/embedded/bin/sv
  * link[/opt/gitlab/service/gitlab-monitor] action create
    - create symlink at /opt/gitlab/service/gitlab-monitor to /opt/gitlab/sv/gitlab-monitor
  * ruby_block[supervise_gitlab-monitor_sleep] action run
    - execute the ruby block supervise_gitlab-monitor_sleep
  * directory[/opt/gitlab/sv/gitlab-monitor/supervise] action create
    - change mode from '0700' to '0755'
    - restore selinux security context
  * directory[/opt/gitlab/sv/gitlab-monitor/log/supervise] action create
    - change mode from '0700' to '0755'
    - restore selinux security context
  * file[/opt/gitlab/sv/gitlab-monitor/supervise/ok] action touch (skipped due to not_if)
  * file[/opt/gitlab/sv/gitlab-monitor/log/supervise/ok] action touch (skipped due to not_if)
  * file[/opt/gitlab/sv/gitlab-monitor/supervise/control] action touch (skipped due to not_if)
  * file[/opt/gitlab/sv/gitlab-monitor/log/supervise/control] action touch (skipped due to not_if)
  * service[gitlab-monitor] action nothing (skipped due to action :nothing)
  * execute[/opt/gitlab/bin/gitlab-ctl start gitlab-monitor] action run
    [execute] ok: run: gitlab-monitor: (pid 9087) 1s
    - execute /opt/gitlab/bin/gitlab-ctl start gitlab-monitor
Recipe: gitlab::default
  * link[/opt/gitlab/service/gitlab-git-http-server] action delete (up to date)
  * directory[/opt/gitlab/sv/gitlab-git-http-server] action delete (up to date)
Recipe: gitlab::gitlab-rails
  * execute[clear the gitlab-rails cache] action run
    - execute /opt/gitlab/bin/gitlab-rake cache:clear
Recipe: gitlab::redis
  * ruby_block[reload redis svlogd configuration] action create
    - execute the ruby block reload redis svlogd configuration
Recipe: gitlab::postgresql
  * ruby_block[reload postgresql svlogd configuration] action create
    - execute the ruby block reload postgresql svlogd configuration
Recipe: gitlab::unicorn
  * ruby_block[reload unicorn svlogd configuration] action create
    - execute the ruby block reload unicorn svlogd configuration
Recipe: gitlab::sidekiq
  * ruby_block[reload sidekiq svlogd configuration] action create
    - execute the ruby block reload sidekiq svlogd configuration
Recipe: gitlab::gitaly
  * service[gitaly] action restart
    - restart service service[gitaly]
  * ruby_block[reload gitaly svlogd configuration] action create
    - execute the ruby block reload gitaly svlogd configuration
Recipe: gitlab::gitlab-workhorse
  * service[gitlab-workhorse] action restart
    - restart service service[gitlab-workhorse]
  * ruby_block[reload gitlab-workhorse svlogd configuration] action create
    - execute the ruby block reload gitlab-workhorse svlogd configuration
Recipe: gitlab::nginx
  * ruby_block[reload nginx svlogd configuration] action create
    - execute the ruby block reload nginx svlogd configuration
Recipe: gitlab::logrotate
  * ruby_block[reload logrotate svlogd configuration] action create
    - execute the ruby block reload logrotate svlogd configuration
Recipe: gitlab::prometheus
  * service[prometheus] action restart
    - restart service service[prometheus]
  * ruby_block[reload prometheus svlogd configuration] action create
    - execute the ruby block reload prometheus svlogd configuration
Recipe: gitlab::node-exporter
  * ruby_block[reload node-exporter svlogd configuration] action create
    - execute the ruby block reload node-exporter svlogd configuration
Recipe: gitlab::redis-exporter
  * ruby_block[reload redis-exporter svlogd configuration] action create
    - execute the ruby block reload redis-exporter svlogd configuration
Recipe: gitlab::postgres-exporter
  * service[postgres-exporter] action restart
    - restart service service[postgres-exporter]
  * ruby_block[reload postgres-exporter svlogd configuration] action create
    - execute the ruby block reload postgres-exporter svlogd configuration
Recipe: gitlab::gitlab-monitor
  * service[gitlab-monitor] action restart
    - restart service service[gitlab-monitor]
  * ruby_block[reload gitlab-monitor svlogd configuration] action create
    - execute the ruby block reload gitlab-monitor svlogd configuration

Running handlers:
Running handlers complete
Chef Client finished, 362/517 resources updated in 02 minutes 10 seconds
gitlab Reconfigured!
[root@much ~]# echo $?
0
[root@much ~]# gitlab-ctl status
run: gitaly: (pid 9151) 10s; run: log: (pid 8544) 59s
run: gitlab-monitor: (pid 9215) 8s; run: log: (pid 9086) 23s
run: gitlab-workhorse: (pid 9174) 9s; run: log: (pid 8629) 53s
run: logrotate: (pid 8774) 45s; run: log: (pid 8773) 45s
run: nginx: (pid 8692) 51s; run: log: (pid 8691) 51s
run: node-exporter: (pid 8904) 37s; run: log: (pid 8903) 37s
run: postgres-exporter: (pid 9202) 8s; run: log: (pid 9015) 29s
run: postgresql: (pid 8198) 102s; run: log: (pid 8197) 102s
run: prometheus: (pid 9189) 9s; run: log: (pid 8853) 39s
run: redis: (pid 8075) 108s; run: log: (pid 8074) 108s
run: redis-exporter: (pid 8953) 31s; run: log: (pid 8952) 31s
run: sidekiq: (pid 8480) 65s; run: log: (pid 8479) 65s
run: unicorn: (pid 8411) 71s; run: log: (pid 8410) 71s
[root@much ~]# 
~~~

从 **`gitlab-ctl status`** 的结果来看

**`gitaly gitlab-monitor gitlab-workhorse logrotate nginx node-exporter postgres-exporter postgresql prometheus redis redis-exporter sidekiq unicorn`** 

这些服务已经在后台运行了

而且已经打开了80端口

~~~
[root@much ~]# netstat  -ant  | grep 80
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:8080          0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:8060            0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:8080          127.0.0.1:52232         TIME_WAIT  
tcp        0      0 127.0.0.1:8080          127.0.0.1:52256         TIME_WAIT  
tcp        0      0 127.0.0.1:8080          127.0.0.1:52208         TIME_WAIT  
tcp6       0      0 ::1:9168                ::1:32980               TIME_WAIT  
[root@much ~]#
~~~

如果防火墙已经开启，需要另外开放 80/tcp 端口

---

## 访问 GitLab ce

登录界面，第一次登录要求输入管理员的密码

![gitlab_ce1.png](/images/gitlab/gitlab_ce1.png)

使用管理员的密码登录

![gitlab_ce2.png](/images/gitlab/gitlab_ce2.png)

进入管理界面

![gitlab_ce3.png](/images/gitlab/gitlab_ce3.png)

右上角有设置菜单

![gitlab_ce4.png](/images/gitlab/gitlab_ce4.png)

查看版本与帮助信息

![gitlab_ce5.png](/images/gitlab/gitlab_ce5.png)


---

# 命令汇总

* **`hostnamectl`**
* **`uname  -a`**
* **`yum install curl policycoreutils openssh-server openssh-clients`**
* **`systemctl enable sshd`**
* **`systemctl start sshd`**
* **`yum install postfix`**
* **`systemctl enable postfix`**
* **`systemctl start postfix`**
* **`firewall-cmd --permanent --add-service=http`**
* **`systemctl reload firewalld`**
* **`curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash`**
* **`yum list all | grep gitlab`**
* **`yum install gitlab-ce`**
* **`gitlab-ctl reconfigure`**
* **`gitlab-ctl status`**
* **`netstat  -ant  | grep 80`**



---

[cvs]:http://www.nongnu.org/cvs/
[svn]:http://subversion.apache.org/
[git]:https://git-scm.com/
[gitlab]:https://about.gitlab.com/
[products]:https://about.gitlab.com/products/
[gitlab_installation]:https://about.gitlab.com/installation/#centos-7

