# 一些编译openwrt的经验

毕设要给两个AR9331设备开发ipk，两个设备上都是古老的openwrt，一个是`barrier_breaker`,一个是`chaos_calmer`。而我本地下载的openwrt源码是最新的稳定版，好不容易编译成功，编译的ipk部署时却总是报错`incompatible architecture`. 搜集了很多资料，终于勉强解决，记录于下。

## 正常编译

在OpenWrt可以查找你的设备支持的最新版本，只要版本能对上，就按官网的helloworld教程[1]走即可。注意源码下载要完整，可以用`git checkout version`检查。第一次编译可以加 -j1 单线程编译，靠谱。
编译ipk其实没什么稀奇，Makefile要写规范，然后往feeds.conf里写好你的包在哪即可。

## 上古版本的编译
99%的问题都出自现有版本编译的ipk和硬件上老板的openwrt不兼容的情况。
---

### 坑1 - 安装ipk提示incompatible architecture

架构不匹配，先检查硬件具体是什么架构，可以用`cat /proc/cpuinfo`和`opkg print-architecture`确定。但是有时候因为版本更新问题，相同架构具体名字可能不一样，参考链接[2]，我在chaos_calmer以后编译出来的ipk，解压后control文件的信息就是mips_24kc，其实就是ar71xx的架构，但是老版本的openwrt就是不认。这时候要么换老版本编译，要么可以修改`/etc/opkg.conf`，手动添加架构名字。

### 坑2 - 程序安装后，运行提示xxx cant found

问题原因不明，但是基本是因为编译的c库文件和部署的硬件上的c库文件不一致，chaos_calmer后的openwrt从uClibc换了muls的c链接库。所以没办法，只能换老版本的openwrt了。

### 坑3 - automake-1.15;...Unescaped left brace in regex is illegal here in regex; marked by <-- HERE in m/\${ <-- HERE ([^ \t=:+{}]+)}/ at ./bin/automake.tmp line 3938.

这是openwrt 15.05的一个bug，主要跟automake有关，详情可以看链接[3], openwrt 15.05.1已修复。

### 坑4 - relocation R_X86_64_32 against `.rodata' can not be used when making a shared object;

编译 `make toolchain/install` 时的迷之错误，参考链接[4],使用`make -C scripts/config/ clean`解决

### 坑5 - Build dependency: Please install Git (git-core) >= 1.6.5 以及 Please install openssl问题

openwrt旧版本用古老命令检查依赖，与最新的库不一致导致的，通过搜索prereq.mk的commit记录，发现了端倪：

Upgrade host tools bison, m4 and fix version check for openssl, git-core #21

https://github.com/openwrt/chaos_calmer/pull/21/commits/5a82e198de4570ce6c123af0d0c727cabaed3afb

当然，首先你得确保确实安装了这些依赖。

### 坑6 - 其他若干迷之错误

一般都是缺文件，缺依赖的问题，`apt-get install` 即可解决

## 总结

对付编译项目的疑难杂症，一要注意版本的变动，二要注意下载失败，文件缺失的问题。三要注意硬件架构选择正确，基本能解决99%的问题。这种能力只能通过多动手才能增长经验，

## Refferences

[1]https://openwrt.org/docs/guide-developer/helloworld/

[2]https://github.com/dafeiyoung/sguclient/wiki/%E4%BF%AE%E6%94%B9opkg.conf%E8%A7%A3%E5%86%B3incompatible-with-the-architectures-configured%E9%97%AE%E9%A2%98

[3]https://bbs.archlinux.org/viewtopic.php?id=229303

[4]https://forum.openwrt.org/t/relocation-r-x86-64-32s-against-symbol-symbol-yes-error-when-using-make/10681/2