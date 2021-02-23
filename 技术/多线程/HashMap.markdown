## HashMap的put过程

1.计算key的hash值，定位到数组i

2.查看i位置有没有元素，没有的话直接插入

3.如果i位置又元素的话，且满足相等条件，替换旧值

4.如果是一个链表，则遍历链表，过程中，如果有`key`满足相等条件，替换旧值；否则插入`key-value`到链表最后；插入之后如果当前链表长度大于`TREEIFY_THRESHOLD`，转换成红黑树结构。

5.`size ++`，如果`size > threshold`，进行扩容



插入过程中，判断两个key是否相等的条件就是`hash值相等，并且== 或者equals返回true`.



### resize

### 1.7

可能会形成环，采用的是尾插法。

### 1.8

采用的是头插法。原来的链表会分裂为两个新链表，一个存放索引值不变的键值对，另一个存放索引值变化的键值对；并且键值对在新链表中的相对顺序没有变



## HashMap形成环

### 1.7



## ConcurrentHashMap

### 1.7

采用的是分段锁的概念

![image-20200918184617445](C:\Users\wyh\AppData\Roaming\Typora\typora-user-images\image-20200918184617445.png)

segement继承自ReentranLock

这里会锁住一个segment，一个segment下面是一个table数组

### 1.8

![image-20200918184758510](C:\Users\wyh\AppData\Roaming\Typora\typora-user-images\image-20200918184758510.png)

1.8使用的是Node，插入的时候会锁住一个Node，这里为了防止两个线程修改，采用了CAS的算法就行修改

每一个Node下面是链表或者红黑树

