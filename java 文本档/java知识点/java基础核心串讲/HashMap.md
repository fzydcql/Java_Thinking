### HashMap

类主要成员：

- transient int size；
  - 记录了Map的KV对的个数
- LoadFactor
  - 装载印子，用来衡量HashMap满的程度。LoadFactor的默认值为0.75f
- int  threshold
  - 临界值，当实际kv个数超过threshold时，HashMap会将容量扩容，threshold = 容量*加载因子
- 除了以上这些重要成员变量之外，HashMap中还有一个和他们紧密相关的概念：capacity
  - 容量，如果不指定，默认容量是16

注意：常用的哈希函数的冲突解决办法中有一种方法叫做链地址发，其实就是将数组合链表组合在一起，发挥两者的优势，我们可以将其理解为链表数组。

源码解析：

- hash：该方法主要是将Object转换成一个整型
- indexFor：该方法主要是将hash生成的整型转换成链表数组中的下标。



