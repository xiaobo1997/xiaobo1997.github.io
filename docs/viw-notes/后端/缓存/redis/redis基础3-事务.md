[TOC]



# 事务



## 介绍

1.redis的事务：会把命令放到一个queue中，提交时会把queue中的命令都提交 `exec`，取消就都取消  `discard` ，如果提交时中间有错误的，那这一次提交就有的成功，有的失败。

2.redis的事务命令行开启   ，`multi`

3.增加监听机制功能： 命令行 `match`

> 在开启事务之前，可以通过 `watch` 命令去监听多个命令，在开启事务后，有其他客户端修改了就直接取消再一次事务，watch监听器也自动消除， 就不需要 手动执行 `unwatch`