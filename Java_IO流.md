# Java I/O流

文章来自https://www.cnblogs.com/wugongzi/p/12092326.html

![img](https://img2018.cnblogs.com/common/1058428/201912/1058428-20191224142510049-508348747.png)

- 对于字节流读取数据时使用的是byte数组存储，将byte数组转成字符串可以用String的一个构造方法直接转，指明转换的起始位置和长度。

- 对于字符流读取数据时使用的是char数组存储，也是应该String的一个构造方法直接转为字符串，指明转换的起始位置和长度。
- PrintWriter是字符流，但比较特殊，它即可以封装字符流又可以封装字节流。

![img](https://img2018.cnblogs.com/common/1058428/201912/1058428-20191224142538937-2092088348.jpg)

