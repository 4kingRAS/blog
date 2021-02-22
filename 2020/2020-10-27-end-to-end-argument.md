当应用程序调用Send之后怎么判断对方是否成功接收?
---

今天为了实现项目的一个ACK机制需求，看到知乎有这么一个问题：[当应用程序调用Send之后怎么判断对方是否成功接收? - 陈硕的回答 - 知乎](https://www.zhihu.com/question/25016042/answer/29798924) 答案不稀奇，稀奇的是评论里有个杠精一直觉得使用tcp判断返回值就知道传输成不成功，不需要应用层再做一个ack机制。

这个人明显是愚蠢的，他的愚蠢来自于读书太少，实践又不多。TCP所做的可靠通信其实只是保证网络没问题的情况下，把你的数据从你的socket缓冲区发送到对方的socket缓冲区而已，如果没收到ACK则重传，如果连接断了则通知ERROR，nothing else. TCP不能保证的是你和对方之间无数路由的故障，超时，错误。以及socket缓冲区到程序显示界面的错误。

完美的ACK是不存在的，基本上系统的每一层设计都需要ACK机制。这就是**End to End argument**.

今天的主题就是解读这篇论文：[END-TO-END ARGUMENTS IN SYSTEM DESIGN](https://link.zhihu.com/?target=http%3A//web.mit.edu/Saltzer/www/publications/endtoend/endtoend.pdf)

再进入解读前，可以看AWS大佬的一篇文章 https://zhuanlan.zhihu.com/p/55311553




