# lab5 spoc 思考题

- 有"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的ucore_code和os_exercises的git repo上。


## 个人思考题

### 总体介绍

 - ucore中的调度点在哪里，完成了啥事？


### 调度算法支撑框架

 - 调度算法支撑框架中的各个函数指针的功能是啥？会被谁在何种情况下调用？

### 时间片轮转调度算法

 - 时间片轮转调度算法是如何基于调度算法支撑框架实现的？

### stride调度算法

 - stride调度算法是如何避免stride溢出问题的？

## 小组练习与思考题

### (1)(spoc) 理解调度算法支撑框架的执行过程

即在ucore运行过程中通过`cprintf`函数来完整地展现出来多个进程在调度算法和框架的支撑下，在相关调度点如何动态调度和执行的细节。(约全面细致约好)

请完成如下练习，完成代码填写，并形成spoc练习报告
> 需写练习报告和简单编码，完成后放到git server 对应的git repo中

### 练习用的[lab6 spoc exercise project source code](https://github.com/chyyuu/ucore_lab/tree/master/labcodes_answer/lab6_result)

```
the process 1 delete from the queue
the process 1 will run
the process 2 will wake up
the process 2 add to the queue
the process 1 will sleep
the process 2 delete from the queue
the process 2 will run
kernel_execve: pid = 2, name = "priority".
the process 3 will wake up
the process 3 add to the queue
the process 4 will wake up
the process 4 add to the queue
the process 5 will wake up
the process 5 add to the queue
the process 6 will wake up
the process 6 add to the queue
the process 7 will wake up
the process 7 add to the queue
main: fork ok,now need to wait pids.
the process 2 will sleep
the process 7 delete from the queue
the process 7 will run
child pid 7, acc 968000, time 1003
the process 7 will be Zombie
the process 2 will wake up
the process 2 add to the queue
the process 6 delete from the queue
the process 6 will run
child pid 6, acc 4000, time 1008
the process 6 will be Zombie
the process 5 delete from the queue
the process 5 will run
child pid 5, acc 4000, time 1012
the process 5 will be Zombie
the process 4 delete from the queue
the process 4 will run
child pid 4, acc 4000, time 1018
the process 4 will be Zombie
the process 3 delete from the queue
the process 3 will run
child pid 3, acc 4000, time 1023
the process 3 will be Zombie
the process 2 delete from the queue
the process 2 will run
main: pid 3, acc 4000, time 1024
main: pid 4, acc 4000, time 1025
main: pid 5, acc 4000, time 1025
main: pid 6, acc 4000, time 1026
main: pid 7, acc 968000, time 1026
main: wait pids over
stride sched correct result: 1 1 1 1 242
the process 2 will be Zombie
the process 1 will wake up
the process 1 add to the queue
the process 1 delete from the queue
the process 1 will run
the process 1 add to the queue
the process 1 delete from the queue
all user-mode processes have quit.
```
