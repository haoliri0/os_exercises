# 死锁与IPC(lec 20) spoc 思考题


- 有"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的ucore_code和os_exercises的git repo上。

## 个人思考题

### 死锁概念 
 - 尝试举一个生活中死锁实例。
 - 可重用资源和消耗资源有什么区别？

### 可重用和不可撤销；
 - 资源分配图中的顶点和有向边代表什么含义？
 - 出现死锁的4个必要条件是什么？

### 死锁处理方法 
 - 死锁处理的方法有哪几种？它们的区别在什么地方？
 - 安全序列的定义是什么？

### 进程的最大资源需要量小于可用资源与前面进程占用资源的总合；
 - 安全、不安全和死锁有什么区别和联系？

### 银行家算法 
 - 什么是银行家算法？
 - 安全状态判断和安全序列是一回事吗？

### 死锁检测 
 - 死锁检测与安全状态判断有什么区别和联系？

> 死锁检测、安全状态判断和安全序列判断的本质就是资源分配图中的循环等待判断。

### 进程通信概念 
 - 直接通信和间接通信的区别是什么？
  本质上来说，间接通信可以理解为两个直接通信，间接通信中假定有一个永远有效的直接通信方。
 - 同步和异步通信有什么区别？
### 信号和管道 
 - 尝试视频中的信号通信例子。
 - 写一个检查本机网络服务工作状态并自动重启相关服务的程序。
 - 什么是管道？

### 消息队列和共享内存 
 - 写测试用例，测试管道、消息队列和共享内存三种通信机制进行不同通信间隔和通信量情况下的通信带宽、通信延时、带宽抖动和延时抖动方面的性能差异。
 
## 小组思考题

 - （spoc） 每人用python实现[银行家算法](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab7/deadlock/bankers-homework.py)。大致输出可参考[参考输出](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab7/deadlock/example-output.txt)。除了`YOUR CODE`部分需要填写代码外，在算法的具体实现上，效率也不高，可进一步改进执行效率。

> 需要修改
```
def ExecuteProcess(self,index):

        #check if less avaliable than Request
        # 2012011332
        ans = True;
        for i in range(len(self.need[index])):
        	if (self.need[index][i] > self.avaliable[i]):
        		ans = False
        		break
        #check END here

        #allocating what they need.
        # 2012011332
        if ans:
        	self.finished[index] = True
        	for i in range(len(self.need[index])):
        		self.avaliable[i] -= self.need[index][i]
        		self.allocated[index][i] += self.need[index][i]
        		self.need[index][i] = 0
        #allocating END here
        return ans

    def TempSafeCheckAfterRelease(self):
        #check if at least one request can be done after previous process done. not check whole sequances.
        #if every element of Requests can't accepted after previous process done, this mean it is not safe state
        # 2012011332
        ans = False
        ishave = False
        for i in range(len(self.need)):
        	if self.finished[i] == False:
        		ishave = True
        		for j in range(len(self.need[i])):
        			if (self.need[i][j] > self.avaliable[j]):
        				break
        			if (j == len(self.need[i])-1):
        				ans = True
        #check END here
        if ishave:
        	return ans
        else:
        	return True
```

 - (spoc) 以小组为单位，请思考在lab1~lab7的基础上，是否能够实现IPC机制，请写出如何实现信号，管道或共享内存（三选一）的设计方案。
 ```
 基本原理： 
 	首先，在内核中维护一个共享内存表，每一个共享内存表项对应一个共享内存空间的入口。
	在进程创建共享内存空间时，会在共享内存表中初始化该内存空间，并且其和其通信进程获得一个key，对应共享内存表可以查询到该共享内存空间的地址。
	利用同步互斥机制完成对共享内存的同步互斥管理。
 ```
 
 - (spoc) 扩展：用C语言实现某daemon程序，可检测某网络服务失效或崩溃，并用信号量机制通知重启网络服务。[信号机制的例子](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab7/ipc/signal-ex1.c)

 - (spoc) 扩展：用C语言写测试用例，测试管道、消息队列和共享内存三种通信机制进行不同通信间隔和通信量情况下的通信带宽、通信延时、带宽抖动和延时抖动方面的性能差异。[管道的例子](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab7/ipc/pipe-ex2.c)
