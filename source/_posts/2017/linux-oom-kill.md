---
layout:     post
title:      "关闭Linux强行杀掉进程服务"
subtitle:   "在Centos上强行关闭OOM_Killer(程序内存过高强行杀掉的服务)"
date:       2017-06-08
author:     "BoomXu"
header-img: "https://ooo.0o0.ooo/2017/06/08/5939609956425.jpg"
tags:
    - Linux
    - 游戏
---


> 本人搭建了个Minecraft的服务器运行在了Centos上，本身机器内存为1G，我设置的Java **Xms700m -Xmx700m **.

### 那么问题来了，我的世界玩着的时候经常断开，而且服务器也连接不上了？

![Minecraft](https://ooo.0o0.ooo/2017/06/08/5939657f9516b.png)

仔细查看系统日志发现：

![kill java](https://ooo.0o0.ooo/2017/06/08/5939665e84bb4.png)

竟然因为内存占用过多被系统直接Kill掉了。我们都知道，Linux系统为了保护系统运行，会把内存占用过高的程序杀掉，那么这个进程就是 **OOM_killer**

# OOM_killer

**OOM_killer**是Linux自我保护的方式，当内存不足时不至于出现太严重问题，有点壮士断腕的意味
在kernel 2.6，内存不足将唤醒oom_killer，挑出/proc/<pid>/oom_score最大者并将之kill掉
 
为了保护重要进程不被oom-killer掉，我们可以：
```
echo -17 > /proc/<pid>/oom_adj,-17表示禁用OOM
```

我们也可以对把整个系统的OOM给禁用掉：
```
sysctl -w vm.panic_on_oom=1 （默认为0，表示开启）
sysctl -p
```

值得注意的是，有些时候 free -m 时还有剩余内存，但还是会触发OOM-killer，可能是因为进程占用了特殊内存地址
 
平时我们应该留意下新进来的进程内存使用量，免得系统重要的业务进程被无辜牵连
可用 top M 查看最消耗内存的进程，但也不是进程一超过就会触发oom_killer
参数/proc/sys/vm/overcommit_memory可以控制进程对内存过量使用的应对策略

```
当overcommit_memory=0 允许进程轻微过量使用内存，但对于大量过载请求则不允许(默认）
当overcommit_memory=1 永远允许进程overcommit
当overcommit_memory=2 永远禁止overcommit
```

OK 了，果然没在出现掉线的情况了，不过这个方式还是会存在问题，就是当内存实在是过大的时候，会破坏系统的整体稳定性，**极大可能会宕机！**

22：54更新

## 宕机了！！！

重启后发现：

![error](https://ooo.0o0.ooo/2017/06/08/593966b07671c.png)

这就悲剧了！！！

看来还是老老实实的 Java **Xms256m -Xmx256m **

这样确实不出现连接问题，但是这样会损失一点游戏的流畅性(比如在铁路上做很长距离的矿车，会卡成狗。。)

什么？？？想完美解决？ 还是 ： **给服务器加大内存把@！！@！**