---
title: "skynet源码梳理"
date: 2024-01-17T11:12:41+08:00
draft: true
tags: 
    - 游戏服务器
    - c
series:
    - skynet 学习笔记
---

# skynet_main.c
skynet 的程序入口`skynet_main.c` 中.

## skynet_globalinit()
`skynet_globalinit()` 方法进行了当前节点的初始化, 节点的内存结构如下:
``` c
struct skynet_node {
	ATOM_INT total; //Todo: 不知道干啥的
	int init; //初始化是否已完成(猜测不用bool是因为部分平台不支持?)
	uint32_t monitor_exit; //Todo: 不知道干啥的
	pthread_key_t handle_key; //threadLocal 记录本线程类型的 key
	bool profile;	// default is on
};
```
其中 `pthread_key_t` 来自 linux 的 `pthread` 库, 该库提供了常见的线程操作. 这里用到了类似 ThreadLocal 的线程内变量的功能(在不同线程中名称相同, 但内容各自独立的变量):
- `pthread_key_create()` 创建一个绑定于当前线程的全局变量
    - @param `pthread_key_t *` 该变量的 key
    - @param `void (* _Nullable)(void *)` 变量销毁时的析构函数
- `pthread_setspecific()` 设置变量的值
    - @param `pthread_key_t`: 该变量的 key
    - @param `const void * _Nullable`: 该变量的 value
- `pthread_getspecific()`
    - @param `pthread_key_t`: 该变量的 key
    - @return `* _Nullable`: 该变量的 value
- `pthread_key_delete()` 删除该变量, 并触发其析构函数
    - @param `pthread_key_t`: 该变量的 key
