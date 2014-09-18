---
layout: post
title: 'Linux进程相关'
date: 2014-08-23 10:55
tags: [Linux]
---
最近为了搞judger，开始研究这些东西了。。。（为了能够掐程序）虽然可能会坑掉（擦汗）。

## 1. fork函数
fork()的效果是创建一个新进程，然后：
 1. 创建失败，返回error信息;
 2. 创建成功，且在父进程中，返回子进程的pid;
 3. 创建成功，且在子进程中，返回0

于是可以：

```c
	pid_t fid=fork();
  if(fid==-1) return 1;
  if(fid==0){
  	运行要监控的程序
	}
  else{
  	进行监控
	}
```

