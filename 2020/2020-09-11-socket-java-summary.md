# 开发遇坑小结

## socket closed exception

问题：socket莫名关闭


原因：socket的东西都是成对的，注意对方socket有没有关闭，获得socket的input ， outputstream的reader/writer有没有哪个关闭了，任何一个关闭都会关闭socket。


## Android 8.0后权限问题

Android定位权限现在要弹个对话框让用户自己选定，光在manifest文件里写没用了。