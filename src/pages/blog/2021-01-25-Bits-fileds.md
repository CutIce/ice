---
templateKey: blog-post
title: Bits-fileds
date: 2021-01-25T15:04:10.000Z
featuredpost: false
featuredimage: /img/flavor_wheel.jpg
description: 位域
tags:
  - C语言
---

一直在琢磨自瞄相关的代码，在与微型电脑通讯中有这样一个结构体：

```c
typedef struct MCURecvData_t {
    uint8_t start;
    float   yaw;
    float   pitch;
    float   yaw_v;
    float   pitch_v;
    float   period;
    uint8_t shoot_flag:1;
    uint8_t energy_flag:7;
    uint8_t check;
    uint8_t end;
} __packed MCURecvData_t;
```

其中`uint8_t shoot_flag: 1;`和`uint8_t energy_flag: 7;`两句都是冒号后带有一个数字，出于好奇查阅了一下顺道学习位域的使用。

## 位域

有些信息在存储时不需要占据一个完整的字节，只需要一个或者几个二进制`bit`，比如标志位只需要0或者1，为了节省空间、处理方便可以使用“位域”（或“位段”）这种数据结构，把一个`Byte`中的8个`bit`分开几个不同位数的区域，每个区域有一个域名，这样程序可以操作域名来进行对某几个`bit`的判断。

## 位域使用限制

2. 位域可以无域名，此时它不能被程序使用，只能作为填充位来补全字节。

   比如：

   ```c
   struct frame1 {
   	uint8_t a: 4;
       uint8_t :0;
       uint8_t b: 2;
       uint8_t c:6;
   }
   ```

   它会占据2个`Byte`，第一个字节有4`bit`是a，其余是0来补全，第2字节2`bit`是b，6`bit`是c。

2. 位域中变量只能为`int`,`unsigned int`,`signed int`这三类，类型决定了如何解释位域的值。

3. 位域宽度必须小于其类型的宽度。`int`就应当小于32，`uint8_t`的应当小于8。

4. 若相邻位域字段类型相同，其位宽之和小于`sizeof(type)`，则后面的字段将紧邻着前一个字段存储，直到不能容纳。

5. 若相邻位域字段类型相同，其位宽之和大于`sizeof(type)`，后面的字段从新的存储单元开始，偏移量位类型大小的整数倍。

6. 如果位域字段中穿插非位域字段，则不进行压缩。

## 大端与小端

这部分资料见参考资料4。

给一个含有位域的结构体赋值后，各位域分到的`bit`分别是什么呢？

假如现在有这样的结构体

```C
union {
	struct {
        uint8_t a1 : 2;
        uint8_t a2 : 3:
        uint8_t a3 : 3;
    } x;
    uint8_t b;
} data;

int main() {
    d.b = 100;
}
```

那么`a1`，`a2`，`a3`分别是多少呢？

`100`的二进制位为`0110 0100`，但是`a1`到`a3`并非依次取值。

`a1`到`a3`是从低端到高端的分配，是内存从小到大的分配。于是从右向左按位数分配。

多个字节时有大端和小端的问题。

**大端字节序就是数据的高字节存在内存的低地址中，小端字节序就是数据的低字节存在内存的高地址中。**定义数组`uint8_t buffer[12]`，`buffer+1`就比`buffer`在内存中大。

例如`unsigned int value = 0x12345678`

`Big-Endian`: 

```C
// 高地址
buffer[3] (0x78) //低位
buffer[2] (0x56)
buffer[1] (0x34)
buffer[0] (0x12) // 高位
// 低地址
```

`Little Endian`

```C
// 高地址
buffer[3] (0x12) -- 高位
buffer[2] (0x34)
buffer[1] (0x56)
buffer[0] (0x78) -- 低位
// 低地址
```

此处有道有意思的题目，参见参考资料3：[关于牛客网C语言结构体位域(bit-fields)的一道题](https://blog.csdn.net/yuyilahanbao/article/details/97390615?utm_medium=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.control&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.control)。

## 参考资料

1. [C 位域 菜鸟驿站](https://www.runoob.com/cprogramming/c-bit-fields.html)
2. [位域的定义和使用](https://blog.csdn.net/sty124578/article/details/79456405)
3. [关于牛客网C语言结构体位域(bit-fields)的一道题](https://blog.csdn.net/yuyilahanbao/article/details/97390615?utm_medium=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.control&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.control)
4. [C/C++ 字节序/位域(Bit-fields)之我见](https://blog.csdn.net/ztz0223/article/details/3599016)