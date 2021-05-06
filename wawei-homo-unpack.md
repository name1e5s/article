# 如何解压鸿蒙操作系统 OTA 包

昨晚我拿到了今年一月份著名的微内核全场景架构国产自主操作系统鸿蒙 OS 的卡刷包。出于好奇便打算拆包看看里面是什么样的，到底是不是套皮安卓，以下是拆包过程。

### 环境需求

Unix 环境；
支持挂载华为 EROFS 的 Linux 发行版一套（Ubuntu 21.04 即可）。

本文中采用的为 macOS Big Sur 11.3 以及 Ubuntu 21.04。

### 下载 OTA 包
我拿到的包是 IT 之家在今年 1 月末发布的 2.0.0.33 Dev Beta 1 版本的卡刷包，[链接](https://115.com/s/swnn31x36g6?password=ITHM&#)。下载完是一组 zip 文件，可以看到有一个近 4G 的 zip 包，想必就是鸿蒙的 OTA 包了吧。

### 解包

首先使用 `unzip -v update_full_base.zip` 看一下里面都有什么。

```
➜  full unzip -v update_full_base.zip
Archive:  update_full_base.zip
signed by SignApk
 Length   Method    Size  Cmpr    Date    Time   CRC-32   Name
--------  ------  ------- ---- ---------- ----- --------  ----
       0  Stored        0   0% 01-01-2009 00:00 00000000  META-INF/blacklist.conf
      23  Stored       23   0% 01-01-2009 00:00 a23ab021  META-INF/com/google/android/verify-script
      20  Stored       20   0% 01-01-2009 00:00 0e30f2a6  OTA_update.tag
      25  Stored       25   0% 01-01-2009 00:00 72916845  SOFTWARE_VER.mbn
      11  Stored       11   0% 01-01-2009 00:00 ef34d0bf  UPT_VER.tag
      30  Stored       30   0% 01-01-2009 00:00 d32ffa0c  VERSION.mbn
      14  Stored       14   0% 01-01-2009 00:00 a87e74e0  full_mainpkg.tag
       8  Stored        8   0% 01-01-2009 00:00 17d7d4ec  rom_hota_for_charger
      24  Stored       24   0% 01-01-2009 00:00 1d146c58  update_feature_list.conf
     440  Defl:N      206  53% 01-01-2009 00:00 05c3794b  META-INF/com/google/android/post-script
 3065040  Defl:N  1420797  54% 01-01-2009 00:00 9869c4eb  META-INF/com/google/android/update-binary
     272  Defl:N       95  65% 01-01-2009 00:00 f65d73cb  META-INF/com/google/android/updater-script_data
  222168  Defl:N     5280  98% 01-01-2009 00:00 96a5b5ab  PTABLE.APP
    2204  Defl:N      268  88% 01-01-2009 00:00 8f06989a  SOFTWARE_VER_LIST.mbn
5046336796  Defl:N 4056952936  20% 01-01-2009 00:00 3e9d82d9  UPDATE.APP
  218112  Defl:N     3529  98% 01-01-2009 00:00 97e00c19  ptable.img
    4096  Defl:N     1866  54% 01-01-2009 00:00 9b8793f3  sec_xloader_header
    1216  Defl:N      914  25% 01-01-2009 00:00 51a821df  META-INF/com/android/otacert
    1756  Defl:N      959  45% 01-01-2009 00:00 5b838775  META-INF/MANIFEST.MF
    1828  Defl:N     1005  45% 01-01-2009 00:00 a662060d  META-INF/CERT.SF
    1330  Defl:N     1085  18% 01-01-2009 00:00 07d60f66  META-INF/CERT.RSA
--------          -------  ---                            -------
5049855413         4058389095  20%                            21 files
```

可以注意到，在开头就有一句注释提及到这个 zip 包 `signed by SignApk`。SignApk 是谷歌推出的对安卓的 OTA 包进行签名的一个软件，rec 可以通过验证签名来确定这个包是否由官方签发，如果不是就会拒绝刷入。产生这个注释的源码在[这里](https://android.googlesource.com/platform/build/+/7e447ed/tools/signapk/SignApk.java#422)。不过啊，因为这个 OTA 包是从安卓升级到高贵的鸿蒙的包，屈尊使用外国人推出的签名软件也是可以理解的，要不手机认不出来这个包嘛。

在这个包中，最大的文件是 `UPDATE.APP` 就是实际的升级映像。这种升级映像是华为一直以来特有的升级格式，非常的先进。其格式为（来自 [JoeyJiao](https://github.com/JoeyJiao/split_updata.pl)，从 commit 日期可以看出这一格式的历史至少有九年了）：

```
1. First 92 bytes are 0x00
2. Each file are started with 55AA 5AA5
3. Then 4 bytes for Header Length
4. Then 4 bytes for Unknown1
5. Then 8 bytes for Hardware ID
6. Then 4 bytes for File Sequence
7. Then 4 bytes for File Size
8. Then 16 byts for File Date
9. Then 16 byts for File Time
10.Then 16 byts for File Type
11.Then 16 byts for Blank1
12.Then 2 bytes for Header Checksum
13.Then 2 bytes for BlockSize
14.Then 2 bytes for Blank2
15.Then ($headerLength-98) bytes for file checksum
16.Then data file length bytes for files.
17.Then padding if have
18.Then repeat 2 to 17
```

采用上述工具，稍作修改即可得到全部升级映像：

```
➜  output l
total 1863864
drwxr-xr-x  57 name1e5s  staff   1.8K  5  6 00:01 .
drwx------@ 33 name1e5s  staff   1.0K  5  6 14:22 ..
-rw-r--r--   1 name1e5s  staff    25B  5  5 22:19 BASE_VER4.img
-rw-r--r--   1 name1e5s  staff   2.2K  5  5 22:19 BASE_VERLIST3.img
-rw-r--r--   1 name1e5s  staff   419K  5  5 22:19 BL29.img
-rw-r--r--   1 name1e5s  staff    25M  5  5 22:19 BOOT10.img
-rw-r--r--   1 name1e5s  staff    72K  5  5 22:19 CACHE37.img
-rw-r--r--   1 name1e5s  staff   301K  5  5 22:19 CRC2.img
-rw-r--r--   1 name1e5s  staff    15M  5  5 22:19 DTBO11.img
-rw-r--r--   1 name1e5s  staff    12M  5  5 22:19 ENG_SYSTEM35.img
-rw-r--r--   1 name1e5s  staff    20M  5  5 22:19 ENG_VENDOR34.img
-rw-r--r--   1 name1e5s  staff    45M  5  5 22:19 ERECOVERY28.img
-rw-r--r--   1 name1e5s  staff    32M  5  5 22:19 ERECOVERY_RAMDIS29.img
-rw-r--r--   1 name1e5s  staff   6.9K  5  5 22:19 ERECOVERY_VBMETA31.img
-rw-r--r--   1 name1e5s  staff    16M  5  5 22:19 ERECOVERY_VENDOR30.img
-rw-r--r--   1 name1e5s  staff   3.3M  5  5 22:19 FASTBOOT8.img
-rw-r--r--   1 name1e5s  staff   8.4M  5  5 22:19 FW_HIFI20.img
-rw-r--r--   1 name1e5s  staff   343K  5  5 22:19 FW_LPM313.img
-rw-r--r--   1 name1e5s  staff   217K  5  5 22:19 HDCP36.img
-rw-r--r--   1 name1e5s  staff    68K  5  5 22:19 HHEE14.img
-rw-r--r--   1 name1e5s  staff   115K  5  5 22:19 HIEPS18.img
-rw-r--r--   1 name1e5s  staff   681K  5  5 22:19 HISEE_IMG51.img
-rw-r--r--   1 name1e5s  staff   213K  5  5 22:19 HISIUFS_GPT6.img
-rw-r--r--   1 name1e5s  staff    10M  5  5 22:19 ISP_FIRMWARE47.img
-rw-r--r--   1 name1e5s  staff   668K  5  5 22:19 IVP49.img
-rw-r--r--   1 name1e5s  staff    66M  5  5 22:19 KPATCH53.img
-rw-r--r--   1 name1e5s  staff    16M  5  5 22:19 METADATA41.img
-rw-r--r--   1 name1e5s  staff   1.3M  5  5 22:19 MODEMNVM_CUST22.img
-rw-r--r--   1 name1e5s  staff   6.4M  5  5 22:19 MODEMNVM_UPDATE21.img
-rw-r--r--   1 name1e5s  staff    20M  5  5 22:19 MODEM_DRIVER23.img
-rw-r--r--   1 name1e5s  staff   133M  5  5 22:19 MODEM_FW48.img
-rw-r--r--   1 name1e5s  staff   804K  5  5 22:19 NPU50.img
-rw-r--r--   1 name1e5s  staff   101B  5  5 22:19 PACKAGE_TYPE5.img
-rw-r--r--   1 name1e5s  staff   584K  5  5 22:19 PATCH52.img
-rw-r--r--   1 name1e5s  staff   102M  5  5 22:19 PREAS32.img
-rw-r--r--   1 name1e5s  staff   536K  5  5 22:19 PREAVS33.img
-rw-r--r--   1 name1e5s  staff   2.0M  5  5 22:19 RAMDISK38.img
-rw-r--r--   1 name1e5s  staff    45M  5  5 22:19 RECOVERY24.img
-rw-r--r--   1 name1e5s  staff    32M  5  5 22:19 RECOVERY_RAMDISK25.img
-rw-r--r--   1 name1e5s  staff   7.6K  5  5 22:19 RECOVERY_VBMETA27.img
-rw-r--r--   1 name1e5s  staff    16M  5  5 22:19 RECOVERY_VENDOR26.img
-rw-r--r--   1 name1e5s  staff   3.6M  5  5 22:19 SENSORHUB19.img
-rw-r--r--   1 name1e5s  staff   900B  5  5 22:19 SHA256RSA1.img
-rw-r--r--   1 name1e5s  staff   271M  5  5 22:19 SUPER40.img
-rw-r--r--   1 name1e5s  staff   4.2M  5  5 22:19 TEEOS16.img
-rw-r--r--   1 name1e5s  staff   250K  5  5 22:19 TRUSTFIRMWARE17.img
-rw-r--r--   1 name1e5s  staff    16K  5  5 22:19 VBMETA15.img
-rw-r--r--   1 name1e5s  staff   1.3K  5  5 22:19 VBMETA_CUST46.img
-rw-r--r--   1 name1e5s  staff   1.3K  5  5 22:19 VBMETA_HW_PRODUC45.img
-rw-r--r--   1 name1e5s  staff   1.4K  5  5 22:19 VBMETA_ODM44.img
-rw-r--r--   1 name1e5s  staff   1.5K  5  5 22:19 VBMETA_SYSTEM42.img
-rw-r--r--   1 name1e5s  staff   1.5K  5  5 22:19 VBMETA_VENDOR43.img
-rw-r--r--   1 name1e5s  staff    24K  5  5 22:19 VECTOR12.img
-rw-r--r--   1 name1e5s  staff   228K  5  5 22:19 XLOADER7.img
```

这次的更新力度还是比较大的，可以看到全部分区都要重新刷写一下。其中最大的 `SUPER.img` 想必就是包含系统分区的镜像了吧，在看这个文件之前，我们先稍微看一下其他几个文件。

#### TEEOS 

有一说一，这个确实是华为自主研发的，没得说。想查看详细信息的可以看这三篇文档：

- [Qihoo 360 的报告](https://www.blackhat.com/docs/us-15/materials/us-15-Shen-Attacking-Your-Trusted-Core-Exploiting-Trustzone-On-Android.pdf)
- [Unearthing the TrustedCore: A Critical Review
on Huawei’s Trusted Execution Environment woot'20](https://www.usenix.org/system/files/woot20-paper-busch.pdf)
- [Downgrade Attack on TrustZone](https://www.researchgate.net/publication/318488121_Downgrade_Attack_on_TrustZone)

#### RECVOERY

在外国人的安卓系统中，RECOVERY 分区用于执行验证并刷入 OTA 包等操作，还可以用于恢复出厂设置等操作。我们先看一下中国人自己的 RECVOERY 里面都有什么吧：

```
➜  output binwalk RECOVERY24.img

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             Android bootimg, kernel size: 20741242 bytes, kernel addr: 0x80000, ramdisk size: 62 bytes, ramdisk addr: 0x66200000, product name: ""
```

。。。，算了直接上 strings 看看有什么好玩的字符串吧：

```
➜  output strings ./RECOVERY24.img | head -2
ANDROID!z|<
loglevel=4 page_tracker=on printktimer=0xfa89b000,0x534,0x538 rcupdate.rcu_expedited=1 androidboot.selinux=enforcing buildvariant=user
```

抛开其中的 ANDROID（安卓） 不谈，第二行是 Linux 的启动参数。这至少证明鸿蒙的 RECVOERY 也是 Linux 作为内核的。虽说和 19 年的微内核宣传不一致，但是和今年以来修订版的鸿蒙 2.0 基于 Linux 的说法一致。还启用了 SELinux，非常的安全。

#### BOOT

这个是手机的启动分区，内核会放在这个分区里面。

```
➜  output binwalk BOOT10.img

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             Android bootimg, kernel size: 20741242 bytes, kernel addr: 0x80000, ramdisk size: 62 bytes, ramdisk addr: 0x66200000, product name: ""
```

和前面一样，也是 Linux 的内核，不再赘述。如果你有全套的安卓 ROM 变异环境，使用 

```bash
mkbootimg --kernel kernel --base 0x0 --cmdline "loglevel=4 page_tracker=on printktimer=0xfa89b000,0x534,0x538 rcupdate.rcu_expedited=1 androidboot.selinux=enforcing buildvariant=user" --tags_offset 0x66000000 --kernel_offset 0x00080000 --ramdisk_offset 0x66200000 --header_version 2 --os_version 10 --os_patch_level 2020-05-01  --output kernel.img
```

就能生成一个这样的 BOOT.img。

#### SUPER

这一分区有两个文件，分别是 `SUPER39.img` 以及 `SUPER40.img`，我们先看比较大的 `SUPER39.img`。

```
➜  i ./imjtool.macOS.x86_64 SUPER39.img
Sparse image v1.0 detected, 1738752 blocks of 4096 bytes
1738752 blocks of 4096 bytes compressed into 528 chunks (43% compressed)
```

显然，这是个 `sparse image` 文件，该格式是谷歌在 Android 6 时期引入的一种新的文件格式，多用于刷机包，使用 Android 提供的 `simg2img` 转换成 img 文件。

查看解压后的 img 文件：

```
➜  extracted ../imjtool.macOS.x86_64 image.img
liblp dynamic partition (super.img) - Blocksize 0x1000, 1 slots
LP MD Header @0x3000, version 10.0, with 5 logical partitions on block device of 3396 GB, at partition super, first sector: 0x1000
	Partitions @0x3080 in 1 groups:
		Group 0: default
			Name: system (Huawei EROFS Filesystem Image, @0x200000 spanning 1 extents and 3 GB)
			Name: hw_product (Huawei EROFS Filesystem Image, @0xcfa00000 spanning 1 extents and 1 GB)
			Name: cust (Huawei EROFS Filesystem Image, @0x144200000 spanning 1 extents and 88 MB)
			Name: vendor (Huawei EROFS Filesystem Image, @0x149a00000 spanning 1 extents and 1 GB)
			Name: odm (Unknown, @0x18e400000 spanning 1 extents and 420 MB)
```

可以看到这 img 文件是由 liblp 生成的动态分区镜像，这一文件是谷歌在最新版安卓中引入的格式，可以采用谷歌提供的[工具](https://android.googlesource.com/platform/system/extras/+/master/partition_tools/)解包。

解包以后终于见到了 `system.img` 文件，查看下这个文件的格式：

```
➜  extracted ../../imjtool.macOS.x86_64 system.img
Huawei EROFS (not handled yet)
```

是华为的 エロ 文件系统呢。macOS 显然没法挂载这个镜像了，改用 Ubuntu 21.04。挂载镜像，拿文件，走人：

```bash
$ sudo mount -o loop system.img ./homo
$ sudo cp -R ./homo ./data
$ sudo chown -R name1e5s:name1e5s ./data
```

至此，拿到了全部文件，可以分析是不是安卓套壳了，当然结果不能说，要不我国籍就没喽。

附世界名画 “此应用专为旧版鸿蒙打造，可能无法正常运行。请尝试检查更新或与开发者联系。” 的多语言源码：

```xml
<string name="deprecated_target_sdk_message">This app was built for an older version of HarmonyOS and may not work properly. Try checking for updates, or contact the developer.</string>
<string name="deprecated_target_sdk_message">이 앱은 이전 버전의 HarmonyOS용으로 제작되어 제대로 작동하지 않을 수 있습니다. 업데이트를 확인하거나 개발자에게 문의하십시오.</string>
<string name="deprecated_target_sdk_message">このアプリは古いバージョンのHarmonyOS用にビルドされているため、正常に機能しない可能性があります。利用可能な更新を確認するか、開発者にお問い合わせください。</string>
```