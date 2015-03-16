# lec5 SPOC思考题


NOTICE
- 有"w3l1"标记的题是助教要提交到学堂在线上的。
- 有"w3l1"和"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的git repo上。
- 有"hard"标记的题有一定难度，鼓励实现。
- 有"easy"标记的题很容易实现，鼓励实现。
- 有"midd"标记的题是一般水平，鼓励实现。


## 个人思考题
---

请简要分析最优匹配，最差匹配，最先匹配，buddy systemm分配算法的优势和劣势，并尝试提出一种更有效的连续内存分配算法 (w3l1)
```
  + 采分点：说明四种算法的优点和缺点
  - 答案没有涉及如下3点；（0分）
  - 正确描述了二种分配算法的优势和劣势（1分）
  - 正确描述了四种分配算法的优势和劣势（2分）
  - 除上述两点外，进一步描述了一种更有效的分配算法（3分）
 ```
- [x]  
优点：
	最先匹配：实现简单，高地址空间有大块空分区，释放分区很快
	最差匹配：适合中等大小分配较多的情况，不会出现太多的小碎片
	最佳匹配：可避免过多的空闲大分区被拆分，外部碎片很小
	buddy system：不会产生太多的小碎片，对大、中、小空间分配都很均衡，效率也较高（O(logN)），空间释放也比较快（O(logN)）

缺点：
	最先匹配：会产生大量外部碎片，分配大块空间时效率低（分区的list太长）
	最差匹配：释放分区较慢，容易破坏大的空闲分区，同样会有外部碎片
	最佳匹配：容易产生许多无用的小碎片，分配或释放后重新维护顺序需要一定的开销，并且释放分区的合并与最差匹配一样慢
	buddy system：有时候会产生较大的内部碎片，对于恰好比2的幂次方大一点点的空间需求，空间浪费比较严重 (比如65KB的需求会浪费63KB的内存空间)

我的想法：
	在buddy system的基础上作一个改进，以便更高效地利用空闲内存。
	举例说明如下：
		当系统接收到一个65KB的内存空间分配需求时，首先同样是找到128KB的一个空闲块。然后，将空闲块分成两份，前一份64KB被全部占用，后面一份64KB则往左儿子递归并且不断二分，直至二分到1KB的叶子节点。如果是67KB的话，则会最终占用64+2+1KB三个节点。 
	设空间需求的大小为N字节，由于一次需求最多会占用logN个节点，此分配过程依然是O(logN)的，不过期望操作数比原Buddy System慢一倍左右。
	释放的话，一次至多释放logN个节点，并且这些节点在地址空间上是连续的，所以合并仍然只需要合并logN次，还是O(logN)的。
	与原Buddy System相比，这个改进最大的优点是将内部碎片变成了外部碎片（如65KB的需求，分配完128KB后，依然可以利用剩下的63KB），使得内存空间的利用率增加。不过需要维护的信息和处理开销都有常数倍增长。
小组成员的想法：
	应用局部性原理，Buddy System的合并不会一路执行到底（只往上合并若干次），这一方法应该会提升分配速度，但对空间的利用率没有提升。

>  

## 小组思考题

请参考ucore lab2代码，采用`struct pmm_manager` 根据你的`学号 mod 4`的结果值，选择四种（0:最优匹配，1:最差匹配，2:最先匹配，3:buddy systemm）分配算法中的一种或多种，在应用程序层面(可以 用python,ruby,C++，C，LISP等高语言)来实现，给出你的设思路，并给出测试用例。 (spoc)

我的学号是2012011332，因此写了一个最佳匹配的算法模型
//最佳匹配
#include <iosteam>
#include <string>
using namespace std;
int *p;
string st;
int h;
vector<int> a,b;  //维护从小到大的内存块,a表示内存块的大小,b表示对应内存块的起始地址

int main()
{
    malloc(p,1024);
    a.pushback(1024);
    b.pushback(p);
    while (true) {
        cin >> st >> h;
        if (st == "arrange") {  //分配大小为h的内存
            int l=0,r=a.size();
            while (l<r) {       //二分查找满足要求(size>=h)的最小空闲块
                int mid = (l+r)/2;
                if (a[mid]<h) l=mid+1;
                else r=mid;
            }
            if (a[l]>=h) {  //存在满足要求的空闲块
                int remain = a[l]-h;
                int new_addr = b[l]+h;
                a.erase(l);
                b.erase(l);
                for (int i=0;i<a.size();i++)
                    if (a[i]>remain) {
                        a.insert(i,remain);
                        b.insert(i,new_addr);
                }
            }
            else cout <<"没有足够大小的内存块\n";
        }
        else if (st == "free") {    //释放起始地址为h的一段内存空间
            int l=-1,r=1025,ll=-1,rr=1025;
            int head=h,len;         //当内存块释放并且合并后，head表示这一空闲块的起始地址，len存储该块的大小
            for (int i=0;i<b.size();i++) {
                if (b[i]<h && b[i]>l) l=b[i],ll=i,head=l;
                if (b[i]>h && b[i]<r) r=b[i],rr=i;
            }
            if (rr<1025) {  //查看释放的块是否是整段内存空间的最后一部分
                len += a[rr]+b[rr]-h;
                a.erase(rr);b.erase(rr);
            }
            else len = b[rr]-h;

            if (ll>=0) {    //查看释放的块是否是整段内存空间的首部分
                len += a[ll];
                a.erase(ll);b.erase(ll);
            }
            l=0;r=a.size()-1;
            while (l<r) {
                mid = (l+r)/2;
                if (a[mid]>=len) r=mid;
                else l=mid+1;
            }
            //添加合并后的新块到list中
            if (a[l]>=len) {
                a.insert(l,len);b.insert(l,head);
            }
            else {
                a.pushback(len);b.pushback(head);
            }
        }
    }
}


--- 

## 扩展思考题

阅读[slab分配算法](http://en.wikipedia.org/wiki/Slab_allocation)，尝试在应用程序中实现slab分配算法，给出设计方案和测试用例。

## “连续内存分配”与视频相关的课堂练习

### 5.1 计算机体系结构和内存层次
MMU的工作机理？

- [x]  

>  http://en.wikipedia.org/wiki/Memory_management_unit

L1和L2高速缓存有什么区别？

- [x]  

>  http://superuser.com/questions/196143/where-exactly-l1-l2-and-l3-caches-located-in-computer
>  Where exactly L1, L2 and L3 Caches located in computer?

>  http://en.wikipedia.org/wiki/CPU_cache
>  CPU cache

### 5.2 地址空间和地址生成
编译、链接和加载的过程了解？

- [x]  

>  

动态链接如何使用？

- [x]  

>  


### 5.3 连续内存分配
什么是内碎片、外碎片？

- [x]  

>  

为什么最先匹配会越用越慢？

- [x]  

>  

为什么最差匹配会的外碎片少？

- [x]  

>  

在几种算法中分区释放后的合并处理如何做？

- [x]  

>  

### 5.4 碎片整理
一个处于等待状态的进程被对换到外存（对换等待状态）后，等待事件出现了。操作系统需要如何响应？

- [x]  

>  

### 5.5 伙伴系统
伙伴系统的空闲块如何组织？

- [x]  

>  

伙伴系统的内存分配流程？

- [x]  

>  

伙伴系统的内存回收流程？

- [x]  

>  

struct list_entry是如何把数据元素组织成链表的？

- [x]  

>  



