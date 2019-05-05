# Mac安装intel power gadget后无法开机

现在的mac出现一些奇奇怪怪问题也不稀奇了，早已不是当年神机一样的Mac 15。

今天直接去官网安装的Intel power gadget 3.0，就想试试这个软件的效果来着。结果privacy那里允许后重启，直接卡在进度条，重启几次都是卡进度条。
我的mac系统是 `Mac High Sierra 10.13.6`

无奈直接搜索，[v2ex](https://www.v2ex.com/t/419141)有个人说也是安了某软件重启卡进度条。
看了下，估计我也是驱动问题，因为这个软件就是个看硬件情况的，不管怎么回事，先删了再说。

删除，重启解决一切问题。
我认为所有这一类驱动或者安装某个软件导致的Mac开机问题都可以这个思路解决：

先进入单用户模式，按苹果[help](https://support.apple.com/zh-cn/HT201573)操作。
开机按住`Cmd + R`进入恢复模式，磁盘工具装载本机硬盘，进入终端，
然后回退根目录，`cd /Volumes/Macintosh\ HD/Library/Extensions`
查看有没有`EnergyDriver.kext`, rm -rf删除之。

附mac自带驱动：
```
ACS6X.kext
ArcMSR.kext
ATTOCelerityFC8.kext
ATTOExpressSASHBA2.kext
ATTOExpressSASRAID2.kext
CalDigitHDProDrv.kext
HighPointlOP.kext
HighPointRR.kext
softRAID.kext
```