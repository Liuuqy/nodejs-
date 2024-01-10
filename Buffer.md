## Buffer（缓冲器）

### 1.概念

Buffer是一个类似于数组的对象，用于表示固定长度的字节序列；

Buffer本质是一段内存空间，专门用来处理二进制数据。

### 2.特点

1. Buffer大小固定且无法调整
2. Buffer性能较好，可以直接对计算机内存进行操作；
3. 每个元素的大小为1字节（byte）

### 3.使用Buffer

#### 3-1创建

#### 3-2操作

1.buffer与字符串的转换

buf.toString()

2.buffer里元素的操作

3.溢出

一字节八比特，最多表示255，如果让某个字节的值超过255，内部则会舍弃高位数据，如：361转二进制位 0001 0110 1001，写入的数据则为0110 1001，为105

4.中文

一个utf-8的中文一般占3个字节：


