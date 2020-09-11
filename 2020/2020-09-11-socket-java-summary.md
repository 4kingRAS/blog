# 开发遇坑小结

## socket closed exception

问题：socket莫名关闭


原因：socket的东西都是成对的，注意对方socket有没有关闭，获得socket的input ， outputstream的reader/writer有没有哪个关闭了，任何一个关闭都会关闭socket。


## socket closed exception

问题：socket莫名关闭


原因：socket的东西都是成对的，注意对方socket有没有关闭，获得socket的input ， outputstream的reader/writer有没有哪个关闭了，任何一个关闭都会关闭socket。