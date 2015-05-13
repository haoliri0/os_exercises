# 文件系统(lec 21) spoc 思考题

## 个人思考题
### 文件系统和文件 
 1. 文件系统的功能是什么？
>  负责数据持久保存，功能是数据存储和访问
>  具体功能：文件分配、文件管理、数据可靠和安全
 1. 什么是文件？
>  文件系统中具有符号名的基本数据单位。

### 文件描述符
 1. 打开文件时，文件系统要维护哪些信息？
>  文件指针、打开文件计数、访问权限、文件位置和数据缓存
 1. 文件系统的基本数据访问单位是什么？这对文件系统有什么影响？
 1. 文件的索引访问有什么特点？如何优化索引访问的速度？

### 目录、文件别名和文件系统种类
 1. 什么是目录？
>  由文件索引项组成的特殊文件。
 1. 目录的组织结构是什么样的？
>  树结构、有向图
 1. 目录操作有哪些种类？
 1. 什么是文件别名？软链接和硬链接有什么区别？
 1. 路径遍历的流程是什么样的？如何优化路径遍历？
 1. 什么是文件挂载？
 1. 为什么会存在大量的文件类型？

### 虚拟文件系统 
 1. 虚拟文件系统的功能是什么？
>  对上对下的接口、高效访问实现
 1. 文件卷控制块、文件控制块和目录项的相互关系是什么？
 1. 可以把文件控制块放到目录项中吗？这样做有什么优缺点？


### 文件缓存和打开文件
 1. 文件缓存和页缓存有什么区别和联系？
 1. 为什么要同时维护进程的打开文件表和操作系统的打开文件表？这两个打开文件表有什么区别和联系？为什么没有线程的打开文件表？
 
### 文件分配
 1. 文件分配的三种方式是如何组织文件数据块的？各有什么特征？
 1. UFS多级索引分配是如何组织文件数据块的位置信息的？

### 空闲空间管理和冗余磁盘阵列RAID
 1. 硬盘空闲空间组织和文件分配有什么异同？
 1. RAID-0、1、4和5分别是如何组织磁盘数据块的？各有什么特征？

## 小组思考题
 1. (spoc)完成Simple File System的功能，支持应用程序的一般文件操作。具体帮助和要求信息请看[sfs-homework](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab8/sfs-homework.md)
> 完成的代码如下：
```
    def deleteFile(self, tfile):
        if printOps:
            print 'unlink("%s");' % tfile

        inum = self.nameToInum[tfile]

    # YOUR CODE, YOUR ID
        # IF inode.refcnt ==1, THEN free data blocks first, then free inode, ELSE dec indoe.refcnt
        # remove from parent directory: delete from parent inum, delete from parent addr
    # DONE

        # finally, remove from files list
        refcnt = self.inodes[inum].getRefCnt()
        if (refcnt == 1):
            if (self.inodes[inum].getAddr() != -1):
                self.dbitmap.free(self.inodes[inum].getAddr())
                self.data[self.inodes[inum].getAddr()].free()
            self.inodes[inum].free()
            self.ibitmap.free(inum)
        else:
            self.inodes[inum].decRefCnt()

        parent = self.getParent(tfile)
        inum = self.nameToInum[parent]
        self.inodes[inum].decRefCnt()
        addr = self.inodes[inum].getAddr()
        self.data[addr].delDirEntry(tfile)
        self.files.remove(tfile)
        return 0

   
    def createLink(self, target, newfile, parent):
    # YOUR CODE, YOUR ID
        # find info about parent
        # is there room in the parent directory?
        # if the newfile was already in parent dir?
        # now, find inumber of target
        # inc parent ref count
        # now add to directory
    # DONE
        inum = self.nameToInum[parent]
        addr = self.inodes[inum].getAddr()
        assert(addr != -1)
        if self.data[addr].getFreeEntries() <= 0:
            return -1
        if self.data[addr].dirEntryExists(newfile):
            return -1
        tinum = self.nameToInum[target]
        self.inodes[tinum].incRefCnt()
        self.inodes[inum].incRefCnt()
        self.data[addr].addDirEntry(newfile, tinum)
        return tinum

   
    def createFile(self, parent, newfile, ftype):
    # YOUR CODE, YOUR ID
        # find info about parent
        # is there room in the parent directory?
        # have to make sure file name is unique
        # find free inode
        # if a directory, have to allocate directory block for basic (., ..) info
        # now ok to init inode properly
        # inc parent ref count
        # and add to directory of parent
    # DONE
        inum = self.nameToInum[parent]
        addr = self.inodes[inum].getAddr()
        assert (addr != -1)
        if self.data[addr].getFreeEntries() <= 0:
            return -1
        if self.data[addr].dirEntryExists(newfile):
            return -1
        new_addr = -1
        if ftype == 'd':
            new_addr = self.dbitmap.alloc()
            if new_addr == -1:
                return -1
        new_inum = self.ibitmap.alloc()
        if new_inum == -1:
            return -1
        if ftype == 'd':
            self.data[new_addr].setType('d')
            self.data[new_addr].addDirEntry('.', new_inum)
            self.data[new_addr].addDirEntry('..', inum)
        self.inodes[inum].incRefCnt()
        self.inodes[new_inum].setAll(ftype, new_addr, 1)
        self.data[addr].addDirEntry(newfile, new_inum)
        return new_inum

   
    def writeFile(self, tfile, data):
        inum = self.nameToInum[tfile]
        curSize = self.inodes[inum].getSize()
        dprint('writeFile: inum:%d cursize:%d refcnt:%d' % (inum, curSize, self.inodes[inum].getRefCnt()))

    # YOUR CODE, 2012011332
        # file is full?
        # no data blocks left
        # write file data
    # DONE
        if (curSize != 0):
            dprint('Error,file is full')
            return -1
        else:
            curr_num = self.dbitmap.alloc()
            if curr_num != -1:
                self.data[curr_num].setType('f')
                self.data[curr_num].addData(data)
                self.inodes[inum].setAddr(curr_num)
            else:
                return -1

        if printOps:
            print 'fd=open("%s", O_WRONLY|O_APPEND); write(fd, buf, BLOCKSIZE); close(fd);' % tfile
        return 0
            
```

 1. (spoc)FAT、UFS、YAFFS、NTFS这几种文件系统中选一种，分析它的文件卷结构、目录结构、文件分配方式，以及它的变种。
  wikipedia上的文件系统列表参考
  - http://en.wikipedia.org/wiki/List_of_file_systems
  - http://en.wikipedia.org/wiki/File_system
  - http://en.wikipedia.org/wiki/List_of_file_systems

  请同学们依据自己的选择，在下面链接处回复分析结果。
  - [https://piazza.com/class/i5j09fnsl7k5x0?cid=416 FAT文件系统分析]
  - [https://piazza.com/class/i5j09fnsl7k5x0?cid=417 NTFS文件系统分析]
  - [https://piazza.com/class/i5j09fnsl7k5x0?cid=418 UFS文件系统分析]
  - [https://piazza.com/class/i5j09fnsl7k5x0?cid=419 ZFS文件系统分析]
  - [https://piazza.com/class/i5j09fnsl7k5x0?cid=420 YAFFS文件系统分析]
