---
layout: post
title: qshell基础
categories: linux admintools
wc: 670 1741 21048
excerpt: follow me
comments: true
---


---

# 前言

**[qshell][qshell]** 是利用七牛文档上公开的API实现的一个方便开发者测试和使用七牛API服务的命令行工具。

这里介绍一下 **[qshell][qshell]** 的基础操作，详细可以参考 **[官方文档][qshell]**


> **Tip:** 当前版本 **qshell v1.6.0**
 
---

# 概要

* TOC
{:toc}


---

##下载


{% highlight bash %}
[root@h101 qshell]# wget http://devtools.qiniu.com/qshell-v1.6.0.zip 
--2015-11-10 14:42:47--  http://devtools.qiniu.com/qshell-v1.6.0.zip
Resolving devtools.qiniu.com... 218.59.186.254, 58.20.197.62, 110.53.75.44, ...
Connecting to devtools.qiniu.com|218.59.186.254|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 16084564 (15M) [application/zip]
Saving to: “qshell-v1.6.0.zip”

100%[============================================================================================>] 16,084,564   892K/s   in 14s    

2015-11-10 14:43:01 (1.10 MB/s) - “qshell-v1.6.0.zip” saved [16084564/16084564]

[root@h101 qshell]# ls
qshell-v1.6.0.zip
[root@h101 qshell]# unzip qshell-v1.6.0.zip  
Archive:  qshell-v1.6.0.zip
  inflating: qshell_darwin_386       
  inflating: qshell_darwin_amd64     
  inflating: qshell_linux_386        
  inflating: qshell_linux_amd64      
  inflating: qshell_windows_386.exe  
  inflating: qshell_windows_amd64.exe  
[root@h101 qshell]# file * 
qshell_darwin_386:        Mach-O executable i386
qshell_darwin_amd64:      Mach-O 64-bit executable
qshell_linux_386:         ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked (uses shared libs), not stripd
qshell_linux_amd64:       ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked (uses shared libs), not stripped
qshell-v1.6.0.zip:        Zip archive data, at least v2.0 to extract
qshell_windows_386.exe:   PE32 executable for MS Windows (console) Intel 80386 32-bit
qshell_windows_amd64.exe: PE32+ executable for MS Windows (console) Mono/.Net assembly
[root@h101 qshell]#
{% endhighlight %}

| 版本| 支持平台| 链接|更新日志|
| :------- | :----: | :---: |:---: |
| qshell v1.6.0| Linux, Windows, Mac OSX | [下载	][qshell_download]| [查看][CHANGELOG]|


> **Tip:** 以下是解压出来的可执行程序的适用平台


命令    | 平台
--------|--------
`qshell_darwin_386`    |    32位苹果
`qshell_darwin_amd64`  |     64位苹果
`qshell_linux_386`     |   32位linux
`qshell_linux_amd64`   |     64位linux
`qshell_windows_386.exe`    |  32位windows
`qshell_windows_amd64.exe`  |  64位windows


##常用命令


{% highlight bash %}
[root@h101 qshell]# ./qshell_linux_amd64  -h
QShell v1.6.0

Options:
	-d                  Show debug message  
	-v                  Show version        
	-h                  Show help           

Commands:
	account             Get/Set AccessKey and SecretKey and Zone
	zone                Switch the zone, [nb,bc,aws]
	dircache            Cache the directory structure of a file path
	listbucket          List all the file in the bucket by prefix
	alilistbucket       List all the file in the bucket of aliyun oss by prefix
	prefop              Query the fop status
	fput                Form upload a local file
	rput                Resumable upload a local file
	qupload             Batch upload files to the qiniu bucket
	qdownload           Batch download files from the qiniu bucket
	stat                Get the basic info of a remote file
	delete              Delete a remote file in the bucket
	move                Move/Rename a file and save in bucket
	copy                Make a copy of a file and save in bucket
	chgm                Change the mimeType of a file
	sync                Sync big file to qiniu bucket
	fetch               Fetch a remote resource by url and save in bucket
	prefetch            Fetch and update the file in bucket using mirror storage
	batchstat           Batch stat files in bucket
	batchdelete         Batch delete files in bucket
	batchchgm           Batch chgm files in bucket
	batchcopy           Batch copy files from bucket to bucket
	batchmove           Batch move files from bucket to bucket
	batchrename         Batch rename files in the bucket
	batchrefresh        Batch refresh the cdn cache by the url list file
	batchsign           Batch create the private url from the public url list file
	checkqrsync         Check the qrsync result
	b64encode           Base64 Encode       
	b64decode           Base64 Decode       
	urlencode           Url encode          
	urldecode           Url decode          
	ts2d                Convert timestamp in seconds to a date (TZ: Local)
	tms2d               Convert timestamp in milli-seconds to a date (TZ: Local)
	tns2d               Convert timestamp in 100 nano-seconds to a date (TZ: Local)
	d2ts                Create a timestamp in seconds using seconds to now
	ip                  Query the ip information
	qetag               Calculate the hash of local file using the algorithm of qiniu qetag
	unzip               Unzip the archive file created by the qiniu mkzip API
	privateurl          Create private resource access url
	saveas              Create a resource access url with fop and saveas
	reqid               Decode a qiniu reqid
	m3u8delete          Delete m3u8 playlist and the slices it references
	buckets             Get all buckets of the account
	domains             Get all domains of the bucket

[root@h101 qshell]# ./qshell_linux_amd64  -v
qshell v1.6.0
[root@h101 qshell]# 
{% endhighlight %}


---

###account

[account][account]

设置或显示当前用户的AccessKey和SecretKey

{% highlight bash %}
[root@h101 qshell]# ./qshell_linux_amd64  account 
AccessKey:  SecretKey:  Zone: 
[root@h101 qshell]# 
[root@h101 qshell]# ./qshell_linux_amd64  account ELUs327kxVPJrGCXqWae9yioc0xYZyrIpbM6Wh6x LVzZY2SqOQ_I_kM1n00ygACVBArDvOWtiLkDtKiw
[root@h101 qshell]# ./qshell_linux_amd64  account 
AccessKey: ELUs327kxVPJrGCXqWae9yioc0xYZyrIpbM6Wh6x SecretKey: LVzZY2SqOQ_I_kM1n00ygACVBArDvOWtiLkDtKiw Zone: nb
[root@h101 qshell]# 

{% endhighlight %}

---

###zone

[zone][zone]

切换当前设置帐号所在的机房区域，仅账号拥有该指定区域机房时有效

{% highlight bash %}
[root@h101 qshell]# ./qshell_linux_amd64  zone
Current zone: nb
[root@h101 qshell]#
{% endhighlight %}


###listbucket

[listbucket][listbucket]

列举七牛空间里面的所有文件

{% highlight bash %}
[root@h101 qshell]# ./qshell_linux_amd64  listbucket   qiniucloud-goods    qiniucloud-goods.list.txt
[root@h101 qshell]# cat qiniucloud-goods.list.txt 
ux/20140618/qiniutest-shop_video_001.MP4	24559427	lpb0Xu_Drayb851dgWn7RBPqcu-s	14030768636225751	video/mp4	
[root@h101 qshell]# 
{% endhighlight %}

---

###buckets

[buckets][buckets]


获取当前账号下所有的空间名称

{% highlight bash %}
[root@h101 qshell]# ./qshell_linux_amd64   buckets
qiniucloud-flower
qiniucloud-apple
qiniucloud-people
qiniucloud-site
qiniucloud-upload
qiniucloud-goods
qiniucloudtest
[root@h101 qshell]#
{% endhighlight %}

---

###domains

[domains][domains]

获取指定空间的所有关联域名

{% highlight bash %}
[root@h101 qshell]# ./qshell_linux_amd64   domains qiniucloud-goods 
qiniucloud-goods.qiniudn.com
qiniucloud-goods.u.qiniudn.com
video.qiniutest.cn
7jpnf5.com1.z0.glb.clouddn.com
7jpnf5.com2.z0.glb.clouddn.com
7jpnf5.com2.z0.glb.qiniucdn.com
[root@h101 qshell]# 
{% endhighlight %}

---

###qetag

[qetag][qetag]

根据七牛的qetag算法来计算文件的hash

{% highlight bash %}
[root@h101 qshell]# ./qshell_linux_amd64  qetag   qiniucloud-goods.list.txt 
Flxstje1T4ojLoOe0G4HjI_WJuVl
[root@h101 qshell]#
{% endhighlight %}

---

###ip


[ip][ip]

根据淘宝的公开API查询ip地址的地理位置

> **Tip:** 淘宝的公开API **http://ip.taobao.com/service/getIpInfo.php**


{% highlight bash %}
[root@h101 qshell]# ./qshell_linux_amd64  ip  180.154.236.170  192.168.1.1 
Ip: 180.154.236.170      => 中国	华东	上海市	上海市		电信
Ip: 192.168.1.1          => 未分配或者内网IP					
[root@h101 qshell]# ./qshell_linux_amd64  ip   202.118.1.81 
Ip: 202.118.1.81         => 中国	东北	辽宁省	沈阳市		教育网
[root@h101 qshell]# 
{% endhighlight %}

---

###d2ts


[d2ts][d2ts]

将日期转为timestamp(单位秒)

{% highlight bash %}
[root@h101 qshell]# ./qshell_linux_amd64   d2ts   0
1447159453
[root@h101 qshell]# 
{% endhighlight %}

---

###ts2d

[ts2d][ts2d]

将timestamp(单位秒)转为UTC+8:00中国日期，主要用来检查上传策略的deadline参数

{% highlight bash %}
[root@h101 qshell]# ./qshell_linux_amd64   ts2d  1447159453
2015-11-10 20:44:13 +0800 CST
[root@h101 qshell]# 
{% endhighlight %}


---

###stat

[stat][stat]

查询七牛空间中一个文件的基本信息

{% highlight bash %}
[root@h101 qshell]# ./qshell_linux_amd64 stat qiniucloud-goods   ux/20140618/qiniutest-shop_video_001.MP4 
Bucket:             qiniucloud-goods          
Key:                ux/20140618/qiniutest-shop_video_001.MP4
Hash:               lpb0Xu_Drayb851dgWn7RBPqcu-s
Fsize:              24559427            
PutTime:            14030768636225751   
MimeType:           video/mp4           

[root@h101 qshell]# 
{% endhighlight %}

---

###tns2d


[tns2d][tns2d]

将timestamp(单位100纳秒)转为UTC+8:00中国日期

{% highlight bash %}
[root@h101 qshell]# ./qshell_linux_amd64 tns2d 14030768636225751 
2014-06-18 15:34:23.6225751 +0800 CST
[root@h101 qshell]# 
{% endhighlight %}


---

###qupload


[qupload][qupload]

同步数据到七牛空间， 带同步进度信息，和数据上传完整性检查

{% highlight bash %}
[root@h101 qshell]# tree /root/qshell/abc/
/root/qshell/abc/
├── qiniucloud-goods.list.txt
├── qshell_darwin_386
└── qshell_windows_386.exe

0 directories, 3 files
[root@h101 qshell]# 
[root@h101 qshell]# cat cfg 
{
    "src_dir"   :   "/root/qshell/abc",
    "access_key"    :   "ELUs327kxVPJrGCXqWae9yioc0xYZyrIpbM6Wh6x",
    "secret_key"    :   "LVzZY2SqOQ_I_kM1n00ygACVBArDvOWtiLkDtKiw",
    "bucket"    :   "qiniucloud-goods",
    "ignore_dir"    :   true,
    "key_prefix"    :   "2015/11/10/test/"
}
[root@h101 qshell]# 
[root@h101 qshell]# ./qshell_linux_amd64  qupload 10 cfg
2015/11/10 17:03:31 [INFO][qshell] qupload.go:145: listing local sync dir, this can take a long time, please wait paitently...
2015/11/10 17:03:31 [INFO][qshell] dir_cache.go:18: No cache file `/root/.qshell/qupload/4b1570c14f91b5f7ce90d7b2e8ca50f2/4b1570c14f91b5f7ce90d7b2e8ca50f2.cache' found, will create one
2015/11/10 17:03:31 [INFO][qshell] dir_cache.go:36: Walk `/root/qshell/abc' start from `2015-11-10 17:03:31.682855906 +0800 CST'
2015/11/10 17:03:31 [INFO][qshell] dir_cache.go:64: Walk `/root/qshell/abc' end at `2015-11-10 17:03:31.683089576 +0800 CST'
2015/11/10 17:03:31 [INFO][qshell] dir_cache.go:65: Walk `/root/qshell/abc' last for `264.555us'
2015/11/10 17:03:31 [INFO][qshell] qupload.go:301: Uploading /root/qshell/abc/qiniucloud-goods.list.txt => 2015/11/10/test/qiniucloud-goods.list.txt (1/3, 33.3%) ...
2015/11/10 17:03:31 [INFO][qshell] qupload.go:301: Uploading /root/qshell/abc/qshell_darwin_386 => 2015/11/10/test/qshell_darwin_386 (2/3, 66.7%) ...
2015/11/10 17:03:31 [INFO][qshell] qupload.go:301: Uploading /root/qshell/abc/qshell_windows_386.exe => 2015/11/10/test/qshell_windows_386.exe (3/3, 100.0%) ...
2015/11/10 17:03:42 [INFO][qshell] qupload.go:413: -------Upload Result-------
2015/11/10 17:03:42 [INFO][qshell] qupload.go:414: Total:	 3
2015/11/10 17:03:42 [INFO][qshell] qupload.go:415: Success:	 3
2015/11/10 17:03:42 [INFO][qshell] qupload.go:416: Failure:	 0
2015/11/10 17:03:42 [INFO][qshell] qupload.go:417: Skipped:	 0
2015/11/10 17:03:42 [INFO][qshell] qupload.go:418: Duration:	 10.920875261s
2015/11/10 17:03:42 [INFO][qshell] qupload.go:419: -------------------------
[root@h101 qshell]# 
{% endhighlight %}

---

###qdownload


[qdownload][qdownload]


从七牛空间同步数据到本地，支持只同步某些前缀的文件，支持增量同步

{% highlight bash %}
[root@h101 qshell]# cat ccfg
{
    "dest_dir"  :   "/tmp/x",
    "bucket"    :   "qiniucloud-goods",
    "domain"    :   "http://video.qiniutest.cn",
    "access_key"    :   "ELUs327kxVPJrGCXqWae9yioc0xYZyrIpbM6Wh6x",
    "secret_key"    :   "LVzZY2SqOQ_I_kM1n00ygACVBArDvOWtiLkDtKiw",
    "is_private"    :   false,
    "prefix"    :   "2015/11/10/test/"
}
[root@h101 qshell]# ./qshell_linux_amd64  qdownload 10 ccfg 
2015/11/10 17:09:57 [INFO][qshell] qdownload.go:81: List bucket...
2015/11/10 17:09:58 [INFO][qshell] qdownload.go:172: Downloading 2015/11/10/test/qiniucloud-goods.list.txt => /tmp/x/2015/11/10/test/qiniucloud-goods.list.txt ...
2015/11/10 17:09:58 [INFO][qshell] qdownload.go:172: Downloading 2015/11/10/test/qshell_darwin_386 => /tmp/x/2015/11/10/test/qshell_darwin_386 ...
2015/11/10 17:09:58 [INFO][qshell] qdownload.go:172: Downloading 2015/11/10/test/qshell_windows_386.exe => /tmp/x/2015/11/10/test/qshell_windows_386.exe ...
2015/11/10 17:10:08 [INFO][qshell] qdownload.go:137: -------Download Result-------
2015/11/10 17:10:08 [INFO][qshell] qdownload.go:138: Total:	 3
2015/11/10 17:10:08 [INFO][qshell] qdownload.go:139: Local:	 0
2015/11/10 17:10:08 [INFO][qshell] qdownload.go:140: Success:	 3
2015/11/10 17:10:08 [INFO][qshell] qdownload.go:141: Failure:	 0
2015/11/10 17:10:08 [INFO][qshell] qdownload.go:142: Duration:	 10.805590463s
2015/11/10 17:10:08 [INFO][qshell] qdownload.go:143: -----------------------------
[root@h101 qshell]#
[root@h101 qshell]# tree /tmp/x/
/tmp/x/
└── 2015
    └── 11
        └── 10
            └── test
                ├── qiniucloud-goods.list.txt
                ├── qshell_darwin_386
                └── qshell_windows_386.exe

4 directories, 3 files
[root@h101 qshell]# 
{% endhighlight %}

---

###delete



[delete][delete]

删除七牛空间中的一个文件

{% highlight bash %}
[root@h101 qshell]# ./qshell_linux_amd64  delete   qiniucloud-goods  2015/11/10/test/qshell_windows_386.exe 
Done!
[root@h101 qshell]#
{% endhighlight %}

---

###move


[move][move]

移动或重命名七牛空间中的一个文件

{% highlight bash %}
[root@h101 qshell]# ./qshell_linux_amd64  move    qiniucloud-goods  2015/11/10/test/qiniucloud-goods.list.txt  qiniucloud-goods  2015/11/10/test/qshell_windows_386.exe 
Done!
[root@h101 qshell]# 
{% endhighlight %}

---

###copy


[copy][copy]

移动或重命名七牛空间中的一个文件

{% highlight bash %}
[root@h101 qshell]# ./qshell_linux_amd64  copy     qiniucloud-goods  2015/11/10/test/qshell_windows_386.exe   qiniucloud-goods  2015/11/10/test/qshell_windows_386_test_cccccopy.exe 
Done!
[root@h101 qshell]# 
{% endhighlight %}

---

###chgm


[chgm][chgm]


修改七牛空间中的一个文件的MimeType

{% highlight bash %}
[root@h101 qshell]# ./qshell_linux_amd64  chgm   qiniucloud-goods  2015/11/10/test/qshell_windows_386_test_cccccopy.exe  image/jpeg
Done!
[root@h101 qshell]# ./qshell_linux_amd64  stat   qiniucloud-goods  2015/11/10/test/qshell_windows_386_test_cccccopy.exe 
Bucket:             qiniucloud-goods          
Key:                2015/11/10/test/qshell_windows_386_test_cccccopy.exe
Hash:               Flxstje1T4ojLoOe0G4HjI_WJuVl
Fsize:              106                 
PutTime:            14471470979836277   
MimeType:           image/jpeg          

[root@h101 qshell]# 
{% endhighlight %}

---

###fetch


[fetch][fetch]

从Internet上抓取一个资源并存储到七牛空间中

{% highlight bash %}
[root@h101 qshell]# ./qshell_linux_amd64  fetch 'https://www.baidu.com/img/bdlogo.png' qiniucloud-goods baidu_logo_1.png 
Key: baidu_logo_1.png
Hash: FrUHIqhkDDd77-AtiDcOwi94YIeM
[root@h101 qshell]# 
[root@h101 abc]# wget  https://www.baidu.com/img/bdlogo.png
--2015-11-10 17:23:04--  https://www.baidu.com/img/bdlogo.png
Resolving www.baidu.com... 61.135.169.121, 61.135.169.125
Connecting to www.baidu.com|61.135.169.121|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 5331 (5.2K) [image/png]
Saving to: “bdlogo.png”

100%[============================================================================================>] 5,331       --.-K/s   in 0s      

2015-11-10 17:23:04 (10.4 MB/s) - “bdlogo.png” saved [5331/5331]

[root@h101 abc]#
[root@h101 abc]# ../qshell_linux_amd64   qetag  bdlogo.png  
FrUHIqhkDDd77-AtiDcOwi94YIeM
[root@h101 abc]# 
{% endhighlight %}

---

###batchdelete

[batchdelete][batchdelete]

批量删除七牛空间中的文件，可以直接根据listbucket的结果来删除

{% highlight bash %}
[root@h101 abc]# cat abc.txt 
2015/11/10/test/qshell_darwin_386	7885316	lvDom3i6Wp4mvC7EStSgXZ__2Z7a	14471462224350860	application/octet-stream	
2015/11/10/test/qshell_windows_386.exe	106	Flxstje1T4ojLoOe0G4HjI_WJuVl	14471470171609874	text/plain	
baidu_logo_1.png	5331	FrUHIqhkDDd77-AtiDcOwi94YIeM	14471472708081319	image/png	
[root@h101 abc]# ../qshell_linux_amd64   batchdelete    qiniucloud-goods  abc.txt 
<DANGER> Input giddfi to confirm operation: giddfi
All deleted!
[root@h101 abc]# 
{% endhighlight %}

完全删除某空间下的文件 

{% highlight bash %}
[root@h101 copy]# ../qshell_linux_amd64  listbucket   qiniucloud-goods  list.video 
[root@h101 copy]# ../qshell_linux_amd64  batchdelete  qiniucloud-goods  list.video 
<DANGER> Input hdbiha to confirm operation: hdbiha
All deleted!
[root@h101 copy]# 
{% endhighlight %}



---

###batchcopy

[batchcopy][batchcopy]

批量复制七牛空间中的文件到另一个空间

准备列表

{% highlight bash %}
[root@h101 copy]# ../qshell_linux_amd64  listbucket  qiniucloud-people  coffee/video  t.video
[root@h101 copy]# ../qshell_linux_amd64  listbucket  qiniucloud-people  coffee/audio  t.audio
[root@h101 copy]# awk '{print $1}' t.video   >> all.list
[root@h101 copy]# awk '{print $1}' t.audio   >> all.list
[root@h101 copy]# less all.list 
{% endhighlight %}

进行复制

{% highlight bash %}
[root@h101 copy]# ../qshell_linux_amd64  batchcopy  qiniucloud-people qiniucloud-goods  all.list 
<DANGER> Input aijbdc to confirm operation: aijbdc
All Copyed!
[root@h101 copy]#
{% endhighlight %}



---

###dircache

[dircache][dircache]

输出本地指定路径下所有的文件列表

{% highlight bash %}
[root@h101 abc]# ../qshell_linux_amd64 dircache  /root/qshell/abc   /tmp/iii
2015/11/10 21:28:02 [INFO][qshell] dir_cache.go:18: No cache file `/tmp/iii' found, will create one
2015/11/10 21:28:02 [INFO][qshell] dir_cache.go:36: Walk `/root/qshell/abc' start from `2015-11-10 21:28:02.039784717 +0800 CST'
2015/11/10 21:28:02 [INFO][qshell] dir_cache.go:64: Walk `/root/qshell/abc' end at `2015-11-10 21:28:02.040013828 +0800 CST'
2015/11/10 21:28:02 [INFO][qshell] dir_cache.go:65: Walk `/root/qshell/abc' last for `254.615us'
[root@h101 abc]# cat /tmp/iii 
abc.txt	300	14471476202016185
bdlogo.png	5331	14068942770000000
qiniucloud-goods.list.txt	106	14471460520916193
i	221	14471620636996111
qshell_darwin_386	7885316	14471460520296193
qshell_windows_386.exe	8027136	14471460520916193
[root@h101 abc]# 
{% endhighlight %}


---


###其它命令

可以直接在qshell后面加一个命令，然后不指定参数，来查看其用法

{% highlight bash %}
[root@h101 abc]# ../qshell_linux_amd64 sync
Usage: qshell [-d] sync <SrcResUrl> <Bucket> <Key> [<UpHostIp>]
  Sync big file to qiniu bucket

[root@h101 abc]# ../qshell_linux_amd64 prefop
Usage: qshell [-d] prefop <PersistentId>
  Query the fop status

[root@h101 abc]# ../qshell_linux_amd64 b64encode
Usage: qshell [-d] b64encode [<UrlSafe>] <DataToEncode>
  Base64 Encode

[root@h101 abc]# 
{% endhighlight %}


---

[qshell]:https://github.com/qiniu/qshell
[qshell_download]:http://devtools.qiniu.com/qshell-v1.6.0.zip
[CHANGELOG]:https://github.com/qiniu/qshell/blob/master/CHANGELOG.md
[account]:https://github.com/qiniu/qshell/wiki/account
[zone]:https://github.com/qiniu/qshell/wiki/zone
[listbucket]:http://github.com/jemygraw/qshell/wiki/listbucket
[buckets]:https://github.com/qiniu/qshell/wiki/buckets
[domains]:https://github.com/qiniu/qshell/wiki/domains
[qetag]:https://github.com/qiniu/qshell/wiki/qetag
[ip]:https://github.com/qiniu/qshell/wiki/ip
[d2ts]:https://github.com/qiniu/qshell/wiki/d2ts
[ts2d]:https://github.com/qiniu/qshell/wiki/ts2d
[stat]:https://github.com/qiniu/qshell/wiki/stat
[tns2d]:https://github.com/qiniu/qshell/wiki/tns2d
[qupload]:https://github.com/qiniu/qshell/wiki/qupload
[qdownload]:https://github.com/qiniu/qshell/wiki/qdownload
[delete]:https://github.com/qiniu/qshell/wiki/delete
[move]:https://github.com/qiniu/qshell/wiki/move
[copy]:https://github.com/qiniu/qshell/wiki/move
[chgm]:https://github.com/qiniu/qshell/wiki/chgm
[fetch]:https://github.com/qiniu/qshell/wiki/fetch
[batchdelete]:https://github.com/qiniu/qshell/wiki/batchdelete
[batchcopy]:https://github.com/qiniu/qshell/wiki/batchcopy
[dircache]:https://github.com/qiniu/qshell/wiki/dircache

