# 同步互斥(lec 18) spoc 思考题


- 有"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的ucore_code和os_exercises的git repo上。

## 个人思考题

### 基本理解
 - 什么是信号量？它与软件同步方法的区别在什么地方？
 - 什么是自旋锁？它为什么无法按先来先服务方式使用资源？
 - 下面是一种P操作的实现伪码。它能按FIFO顺序进行信号量申请吗？
```
 while (s.count == 0) {  //没有可用资源时，进入挂起状态；
        调用进程进入等待队列s.queue;
        阻塞调用进程;
}
s.count--;              //有可用资源，占用该资源； 
```

> 参考回答： 它的问题是，不能按FIFO进行信号量申请。
> 它的一种出错的情况
```
一个线程A调用P原语时，由于线程B正在使用该信号量而进入阻塞状态；注意，这时value的值为0。
线程B放弃信号量的使用，线程A被唤醒而进入就绪状态，但没有立即进入运行状态；注意，这里value为1。
在线程A处于就绪状态时，处理机正在执行线程C的代码；线程C这时也正好调用P原语访问同一个信号量，并得到使用权。注意，这时value又变回0。
线程A进入运行状态后，重新检查value的值，条件不成立，又一次进入阻塞状态。
至此，线程C比线程A后调用P原语，但线程C比线程A先得到信号量。
```

### 信号量使用

 - 什么是条件同步？如何使用信号量来实现条件同步？
 - 什么是生产者-消费者问题？
 - 为什么在生产者-消费者问题中先申请互斥信息量会导致死锁？

### 管程

 - 管程的组成包括哪几部分？入口队列和条件变量等待队列的作用是什么？
 - 为什么用管程实现的生产者-消费者问题中，可以在进入管程后才判断缓冲区的状态？
 - 请描述管程条件变量的两种释放处理方式的区别是什么？条件判断中while和if是如何影响释放处理中的顺序的？

### 哲学家就餐问题

 - 哲学家就餐问题的方案2和方案3的性能有什么区别？可以进一步提高效率吗？

### 读者-写者问题

 - 在读者-写者问题的读者优先和写者优先在行为上有什么不同？
 - 在读者-写者问题的读者优先实现中优先于读者到达的写者在什么地方等待？
 
## 小组思考题

1. （spoc） 每人用python threading机制用信号量和条件变量两种手段分别实现[47个同步问题](07-2-spoc-pv-problems.md)中的一题。向勇老师的班级从前往后，陈渝老师的班级从后往前。请先理解[]python threading 机制的介绍和实例](https://github.com/chyyuu/ucore_lab/tree/master/related_info/lab7/semaphore_condition)

> 李日灵 2012011332
我选择的是第7题：
设有一个可以装A、B两种物品的仓库,其容量有限(分别为N),但要求仓库中A、B两种物品的数量满足下述不等式: -M≤A物品数量-B物品数量≤N 其中M和N为正整数。另外,还有一个进程消费A,B,一次取一个A,B组装成C。 试用信号量和PV操作描述A、B两种物品的入库过程。
解：因为0<=A,B<=N,所以0<=A-B<=N恒成立，因而可以去掉一个条件。另，原题所附的伪代码有误，修改后semaphore和lock两种方式的伪代码如下：
1、semaphore
semaphore mutex=1,diff=m,empty1=N,empty2=N,full1,full2=0;
cobegin
	process(A);
	process(B);
	process(C)
coend
// A物品入库
process A
begin
	while(TRUE)
	begin
	p(empty1);
	p(mutex);
	A物品入库;
	v(mutex);
	v(full1);
	v(diff);
	end
end
// B物品入库：
process B
begin
	while(TRUE)
	begin
	p(empty2);
	p(diff);
	
	p(mutex);
	B物品入库;
	v(mutex);
	v(full2);
	end
end
// process C
begin
	while(TRUE)
	begin
	p(full1);
	p(full2);
	p(mutex);
	组装;
	v(mutex);
	v(empty1);
	v(empty2);
	end
end
 
2、lock伪代码

procedure producerA() {
	lock->Acquire();
	while (countA == n)
	admit_A.Wait(&lock);
	
	Add A to the buffer;
	countA++;
	
	admit_B.Signal();
	admit_C.signal();
	lock->Release();
}

procedure producerB() {
lock->Acquire();
while (countB == n || countA-countB<=-M)
admit_B.Wait(&lock);

Add B to the buffer;
countB++;
admit_A.Signal();
admit_C.signal();
lock->Release();
}
procedure consumer() {
lock->Acquire();
while (countA==0 || countB == 0)
admit_C.Signal();
countA--;
countB--;
admit_A.Signal();
admit_B.signal();
lock->Release(); 
}
 
对应的python代码
semaphore方法：
```
import multiprocessing
import time
def producerA(mutex,empty1,full1,diff,i):
    print("ready to produce A")
    empty1.acquire()
    mutex.acquire()
    # print(multiprocessing.current_process().name + " acquire")
    print("producing A succeed")
    time.sleep(i)
    # print(multiprocessing.current_process().name + " release")
    mutex.release()
    full1.release()

    diff.release()

def producerB(mutex,empty2,full2,diff,i):
    print("ready to produce B")
    empty2.acquire()
    diff.acquire()
    mutex.acquire()

    print("producing B succeed")
    time.sleep(i)
    
    mutex.release()
    full2.release()

def consumer(mutex,empty1,empty2,full1,full2,i):
    print("ready to cosume")
    full1.acquire()
    full2.acquire()
    mutex.acquire()
    
    print("cosuming suceed")
    time.sleep(i)

    mutex.release()
    empty1.release()
    empty2.release()
    
if __name__ == "__main__":
    N = 10
    m = 5
    full1 = multiprocessing.Semaphore(0)
    full2 = multiprocessing.Semaphore(0)
    empty1 = multiprocessing.Semaphore(N)
    empty2 = multiprocessing.Semaphore(N)
    
    diff = multiprocessing.Semaphore(m)
    mutex = multiprocessing.Semaphore(1)

    for i in range(5):
        pa = multiprocessing.Process(target=producerA, args=(mutex,empty1,full1,diff,i*2))
        pa.start()
        pb = multiprocessing.Process(target=producerB, args=(mutex,empty2,full2,diff,i*2))
        pb.start()
        c = multiprocessing.Process(target=consumer, args=(mutex,empty1,empty2,full1,full2,i*2))
```

condition方法：
```
import threading  
import time  

#countA means the amounts of A in storage. samely,countB means the amount of B.
condition = threading.Condition()
countA = 0
countB = 0
N = 10
M = 5

#threadA: add one A to the storage
class ProducerA(threading.Thread):  
    def __init__(self):  
        threading.Thread.__init__(self)  
          
    def run(self):  
        global condition, countA, N, M
        while True:  
            if condition.acquire():  
                if countA < N:  
                    countA += 1;  
                    print "ProducerA():deliver one, now countA:%s" %countA  
                    condition.notify()  
                else:  
                    print "ProducerA():already %s, stop deliver, now countA:%s" %(N, countA)  
                    condition.wait();  
                condition.release()  
                time.sleep(2)  

#threadB: add one B to the storage
class ProducerB(threading.Thread):  
    def __init__(self):  
        threading.Thread.__init__(self)  
          
    def run(self):  
        global condition,countA,countB,N,M
        while True:  
            if condition.acquire():  
                if (countB < N and countA-countB > -M):  
                    countB += 1;  
                    print "ProducerB():deliver one, now countB:%s" %countB  
                    condition.notify()  
                else:  
                    print "ProducerB():already %s, stop deliver, now countA:%s" %(N, countB)  
                    condition.wait();  
                condition.release()  
                time.sleep(2)  
          
      
#threadC: consume one A and one B in storage,then create one C   
class Consumer(threading.Thread):  
    def __init__(self):  
        threading.Thread.__init__(self)  
          
    def run(self):  
        global condition,countA,countB
        while True:  
            if condition.acquire():  
                if (countA>0 and countB>0):  
                	# consume a pair of A&B
                    countA -= 1;
                    countB -= 1;
                    print "Consumer():consume one A and one B, now countA:%s countB:%s" %(countA, countB)  
                    condition.notify()  
                else:  
                	# lack of A or B in storage
                    print "Consumer():lack of resource,not runnable" %(self.name, products)  
                    condition.wait();  
                condition.release()  
                time.sleep(2)  
                  
if __name__ == "__main__":
    for i in range(0,10):
        pa = ProducerA()
        pa.start()
        pb = ProducerB()  
        pb.start()  
          
    for c in range(0, 8):  
        c = Consumer()  
        c.start() 

```