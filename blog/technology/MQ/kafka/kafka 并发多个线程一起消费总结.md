1，一个topic同时并发多少个线程一起消费是由：topic的分区数量决定的。

2，如果发消息指定key：同时只有一个线程可以消费到