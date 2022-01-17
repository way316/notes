Java Collection

# 列表和队列

## ArrayList

默认情况下是10，每次扩容原来大小的1/2。

- 可以随机访问，按照索引位置进行访问效率很高，用算法描述中的术语，效率是O(1)，简单说就是可以一步到位。
- 除非数组已排序，否则按照内容查找元素效率比较低，具体是O(N)，N为数组内容长度，也就是说，性能与数组长度成正比。
- 添加元素的效率还可以，重新分配和拷贝数组的开销被平摊了，具体来说，添加N个元素的效率为O(N)。
- 插入和删除元素的效率比较低，因为需要移动元素，具体为O(N)。 



## LinkedList

双向列表

- 按需分配空间，不需要预先分配很多空间
- 不可以随机访问，按照索引位置访问效率比较低，必须从头或尾顺着链接找，效率为O(N/2)。
- 不管列表是否已排序，只要是按照内容查找元素，效率都比较低，必须逐个比较，效率为O(N)。
- 在两端添加、删除元素的效率很高，为O(1)。
- 在中间插入、删除元素，要先定位，效率比较低，为O(N)，但修改本身的效率很高，效率为O(1)。 

在内存里分散排列

## ArrayDeque

它实现了双端队列接口，，相比LinkedList效率要更高一些，实现原理上，它采用动态扩展的循环数组，使用高效率的位操作。

![image-20211017104238211](C:\Users\Wang\OneDrive\Java全家桶\集合框架.assets\image-20211017104238211.png)

ArrayDeque的高效来源于head和tail这两个变量，它们使得物理上简单的从头到尾的数组变为了一个逻辑上循环的数组，避免了在头尾操作时的移动。

![image-20211017104138941](C:\Users\Wang\OneDrive\Java全家桶\集合框架.assets\image-20211017104138941.png)

扩容：默认情况下是8，如果创建的时候大于8，则分配2的整数次幂

进行与操作是要保证索引在正确范围，(tail + 1)与(elements.length - 1)相与就可以得到下一个正确位置，是因为elements.length是2的幂次方，(elements.length - 1)的后几位全是1，无论是正数还是负数，与(elements.length - 1)相与都能得到期望的下一个正确位置。

比如说，如果elements.length为8，则(elements.length - 1)为7，二进制为0111，对于负数-1，与7相与，结果为7，对于正数8，与7相与，结果为0，都能达到循环数组中找下一个正确位置的目的。



- 在两端添加、删除元素的效率很高，动态扩展需要的内存分配以及数组拷贝开销可以被平摊，具体来说，添加N个元素的效率为O(N)。  
- 根据元素内容查找和删除的效率比较低，为O(N)。
- 与ArrayList和LinkedList不同，没有索引位置的概念，不能根据索引位置进行操作。



# Map和Set

## HashMap

内部使用Entry<K,V>[]来实现存储数据

添加第一个元素的时候默认分配的table是16，threshold等于table.length乘以loadFactory. LoadFactory默认是0.75.

HashMap的基本实现原理:

内部有一个数组table，每个元素table[i]指向一个单向链表，根据键存取值，用键算出hash，取模得到数组中的索引位置buketIndex，然后操作table[buketIndex]指向的单向链表。

![image-20211016134919483](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20211016134919483.png)

添加元素操作：

![image-20211016133418959](C:\Users\Wang\AppData\Roaming\Typora\typora-user-images\image-20211016133418959.png)

1. 调用inflateTable方法，给table分配实际的空间
2. 计算key的hash值
3. 按照key的hash值找到index     return h & (length-1)  等价于 求模运算h%length。
4. <u>**找到了保存位置index[i]指向一个单向链表，遍历链表查找是否有这个键。**</u>
5. 先比较hash值，再用equals方法比较key值，因为hash是int，比较更快。
6. 如果找到相同的就修改值，如果没有就直接添加。

1.8的put

![image-20211027180857282](C:\Users\Wang\OneDrive\Java全家桶\集合框架.assets\image-20211027180857282.png)



- 根据键保存和获取值的效率都很高，为O(1)，每个单向链表往往只有一个或少数几个节点，根据hash值就可以直接快速定位。
- HashMap中的键值对没有顺序，因为hash值是随机的。

HashMap计算hash方法：

![image-20211027181243455](C:\Users\Wang\OneDrive\Java全家桶\集合框架.assets\image-20211027181243455.png)

 如果经常需要根据键存取值，而且不要求顺序，那HashMap就是理想的选择。



HashMap扩容问题：Hashmap扩容比较耗费时间，所有数值都需要rehash，而且多线程下调整大小存在条件竞争，容易造成死锁。



## ConcurrentHashMap

​	CAS+synchronized使锁更细化。

![image-20211027184546925](C:\Users\Wang\OneDrive\Java全家桶\集合框架.assets\image-20211027184546925.png)

## HashSet

与hashmap一样，对于自己创建的class必须要重写hashCode和equals方法。

HashSet内部使用了HashMap，只保存键不保存值，从而实现了HashMap这种方法，

- 没有重复元素
- 可以高效的添加、删除元素、判断元素是否存在，效率都为O(1)。
- 没有顺序

![image-20211027184808657](C:\Users\Wang\OneDrive\Java全家桶\集合框架.assets\image-20211027184808657.png)

## TreeMap

**TreeMap特点分析**

与HashMap相比，TreeMap同样实现了Map接口，但内部使用红黑树实现，红黑树是统计效率比较高的大致平衡的排序二叉树，这决定了它有如下特点：

- 按键有序，TreeMap同样实现了SortedMap和NavigableMap接口，可以方便的根据键的顺序进行查找，如第一个、最后一个、某一范围的键、邻近键等。
- 为了按键有序，TreeMap要求键实现Comparable接口或通过构造方法提供一个Comparator对象。
- 根据键保存、查找、删除的效率比较高，为O(h)，h为树的高度，在树平衡的情况下，h为log2(N)，N为节点数。
- 不要求排序，优先考虑HashMap，要求排序，考虑TreeMap。



## TreeSet

- 没有重复元素
- 添加、删除元素、判断元素是否存在，效率比较高，为O(log2(N))，N为元素个数。
- 有序，TreeSet同样实现了SortedSet和NavigatableSet接口，可以方便的根据顺序进行查找和操作，如第一个、最后一个、某一取值范围、某一值的邻近元素等。
- 为了有序，TreeSet要求元素实现Comparable接口或通过构造方法提供一个Comparator对象。