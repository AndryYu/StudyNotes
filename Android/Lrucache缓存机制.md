# Lrucache缓存机制

### LinkedHashMap
LinkedHashMap的特点
* 有序性：
LinkedHashMap维护了一个双向链表来保存元素的插入顺序。
    插入顺序、访问顺序等都可以通过构造函数来指定。
* 性能：
    LinkedHashMap的性能与HashMap类似，平均情况下，大多数操作的时间复杂度为O(1)。
    但是由于额外的链表维护，性能略低于HashMap。
* 构造函数：
    LinkedHashMap提供了几个构造函数，可以根据需要来创建不同的实例。
    构造函数
    LinkedHashMap提供了几个构造函数，其中最常用的是以下两种：

    * 无参构造函数：
        默认使用HashMap的默认初始容量（16）和加载因子（0.75），并保持插入顺序。

        LinkedHashMap<K, V> map = new LinkedHashMap<>();
    
    * 带参数的构造函数：
        可以指定初始容量、加载因子和访问顺序或插入顺序。
        
        LinkedHashMap<K, V> map = new LinkedHashMap<>(initialCapacity, loadFactor, accessOrder);

        initialCapacity：初始容量。

        loadFactor：加载因子。

        accessOrder：如果为true，则按访问顺序排序；如果为false，则按插入顺序排序。

**注意**
* 如果你使用accessOrder=true，那么每次访问一个键都会将该键移到链表的末尾，从而改变元素的顺序。
* 如果你使用accessOrder=false，那么元素将保持插入时的顺序。
* LinkedHashMap可以用来实现LRU缓存，因为你可以利用它的访问顺序特性来实现缓存的淘汰策略。