# IO设备(lec 23) spoc 思考题

## 个人思考题
### IO特点 
 1. 字符设备的特点是什么？
 1. 块设备的特点是什么？
 1. 网络设备的特点是什么？
 1. 阻塞I/O、非阻塞I/O和异步I/O这三种I/O方式有什么区别？

### I/O结构
 1. 请描述I/O请求到完成的整个执行过程
 1. CPU与设备通信的手段有哪些？

> 显式的IO指令，如x86的in, out； 或者是memory读写方式，即把device的寄存器，内存等映射到物理内存中 

### IO数据传输
 1. IO数据传输有哪几种？
 1. 轮询方式的特点是什么？
 1. 中断方式的特点是什么？
 1. DMA方式的特点是什么？

### 磁盘调度
 1. 请简要阐述磁盘的工作过程
 1. 请用一表达式（包括寻道时间，旋转延迟，传输时间）描述磁盘I/O传输时间
 1. 请说明磁盘调度算法的评价指标
 1. FIFO磁盘调度算法的特点是什么?
 1. 最短服务时间优先(SSTF)磁盘调度算法的特点是什么?
 1. 扫描(SCAN)磁盘调度算法的特点是什么?
 1. 循环扫描(C-SCAN)磁盘调度算法的特点是什么?
 1. C-LOOK磁盘调度算法的特点是什么?
 1. N步扫描(N-step-SCAN)磁盘调度算法的特点是什么?
 1. 双队列扫描(FSCAN)磁盘调度算法的特点是什么?

### 磁盘缓存
 1. 磁盘缓存的作用是什么？
 1. 请描述单缓存(Single Buffer Cache)的工作原理
 1. 请描述双缓存(Double Buffer Cache)的工作原理
 1. 请描述访问频率置换算法(Frequency-based Replacement)的基本原理

## 小组思考题
 - (spoc)完成磁盘访问与磁盘寻道算法的作业，具体帮助和要求信息请看[disksim指导信息](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab8/disksim-homework.md)和[disksim参考代码](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab8/disksim-homework.py)

```
扇区访问序列：10,11,12,13,24,1

# SSTF
# sector 10 seek 0  rotate 105 transer 30
# sector 11 seek 0  rotate 0   transer 30
# sector 1  seek 0  rotate 60  transer 30
# sector 12 seek 40 rotate 290 transer 30
# sector 13 seek 0  rotate 0   transer 30
# sector 24 seek 40 rotate 260 transer 30
# total          80        715         150

1) 按照我们小组的考虑，SCAN和C-SCAN算法在访问完sector 11后会跳下另一道访问sector 12，而不是访问sector 1，这是因为sector 11是外道的最后一个sector。当然，造成这种原因主要怪磁头初始位置并没有在sector 0前面......如果访问序列最后是sector 7，而不是sector1。那么这个sector 7在第一遍scan的时候会被读到访问
2) 其次关于第二道里为什么会读sector 12和13，按(1)中的说法，转一圈回过来再访问sector12 和 13与(1)的说法矛盾。因为磁头读完11跳到第2道这段时间内磁盘仍在转，寻道完成后已经是sector 13的1/3处，因而无法完整访问sector12 和 13。但是这是演示程序与测试用例本身的原因导致的。实际情况中，访问磁盘并不以扇区为基本单位。同时对于连续数据的访问，如果跨越了不同磁道，并且下一磁道的不留空白数据段作缓冲的话，磁头是不能高效、充分地访问数据的，就会出现转一圈再回来读写的情况。在本例中说的就是sector 12、13。按照本例来说，sector 12以及sector13的前1/3部分，都应该作为空白区间，这样磁头在从最外道换到第2道时，磁头就能直接开始访问连续数据了，而不需要等磁盘转一圈再开始。
当然，第(2)问的说法和本题无关。

# SCAN
# sector 10 seek 0  rotate 105 transer 30
# sector 11 seek 0  rotate 0   transer 30
# sector 12 seek 40 rotate 320 transer 30
# sector 13 seek 0  rotate 0   transer 30
# sector 24 seek 40 rotate 260 transer 30
# sector 1  seek 80 rotate 280 transer 30
# total          160       965         150

# C-SCAN
# sector 10 seek 0  rotate 105 transer 30
# sector 11 seek 0  rotate 0   transer 30
# sector 12 seek 40 rotate 320 transer 30
# sector 13 seek 0  rotate 0   transer 30
# sector 24 seek 40 rotate 260 transer 30
# sector 1  seek 40 rotate 320 transer 30
# total          120       1005        150
```