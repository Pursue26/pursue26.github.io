---
title: C语言之大小端知识
date: 2023-09-18 11:48:41
categories:
    - 学习笔记
    - C语言
tags:
    - 大小端
    - 字节序
    - 网络字节序
abbrlink: 230918114841
---

## 字节序

### 主机字节序

在计算机中，数据在内存中的存储是以 byte 为单位的。字节序是由于 CPU 对大于一个字节的变量，在内存中的存放顺序不同而产生的。

不同的计算机体系结构使用不同的字节序，对于大于一个字节的变量表示，有如下两种方式：

- 小端模式（Listen Endian, LE）：高位字节存储在高位地址，而低位字节存储在低位地址，与大端模式相反。

- 大端模式（Big Endian, BE）：高位字节存储在低位地址，而低位字节存储在高位地址。

<!--more-->

![大小端模式](/images/le-be-endian.png)

在小端字节序主机系统中进行字节序转换时，需要将低地址的字节和高地址的字节进行交换即可得到大端字节序。

### 网络字节序

不同的机器主机字节序不相同，与 CPU 设计有关，数据的顺序是由 CPU 决定的，而与操作系统无关。我们把某个给定系统所用的字节序称为主机字节序（host byte order）。比如 x86 系列 CPU 都是 little-endian 的字节序。

网络字节序是 TCP/IP 中规定好的一种数据表示格式，它与具体的CPU类型、操作系统等无关，从而可以保证数据在不同主机之间传输时能够被正确解释。网络字节序采用 big-endian 排序方式。

由于这个原因，为了确保数据在不同主机之间传输时能够被正确解释，需要在不同字节序之间进行转换，所以在网络通信中，一般需要将数据转换为网络字节序进行传输。

## 主机字节序类型判断

可以使用共用体（union）来判断当前 CPU 平台是大端字节序还是小端字节序。这是因为共用体的特点是：使用长度最大的数据类型作为共用体的大小。

- 建立一个联合类型 `BYTE_ORDER_UN`，用于测试字节序，可以通过成员 `byte` 来访问 `value` 变量的高字节和低字节。

- 声明一个 `BYTE_ORDER_UN` 类型的变量 `unByteOrder`，将值 `0xabcd` 赋给成员变量 `value`。由于在类型 `BYTE_ORDER_UN` 中，`value` 和 `byte` 成员**共享一块内存**，所以可以通过 `byte` 的不同成员来访问 `value` 的高字节和低字节。

```c
#include <stdio.h>

typedef union byte_order {
    unsigned short int value;
    unsigned char byte[sizeof(unsigned short int)];
} BYTE_ORDER_UN;

int main(int argc, char *argv[]) {
    BYTE_ORDER_UN unByteOrder;
    unByteOrder.value = 0xabcd;

    if ((unByteOrder.byte[0] ==0xcd) && (unByteOrder.byte[1] ==0xab)) {
        printf("Listen endian byte order, byte[0]: 0x%x, byte[1]: 0x%x\n", \
        unByteOrder.byte[0], unByteOrder.byte[1]);
    } else if((unByteOrder.byte[0] ==0xab) && (unByteOrder.byte[1] ==0xcd)) {
        printf("Big endian byte order, byte[0]: 0x%x, byte[1]: 0x%x\n", \
        unByteOrder.byte[0], unByteOrder.byte[1]);
    }
    return 0;
}
```

主机系统为小端字节序的测试结果：

```text
$ gcc byteOrder.c -o byteOrder
$ ./byteOrder
Listen endian byte order, byte[0]: 0xcd, byte[1]: 0xab
```

## 主机字节序到网络字节序转换

字节交换的作用是生成一个网络字节序的变量，**其字节的顺序与主机类型和操作系统无关**。进行网络字节序转换的时候，只要转换一次就可以了，不要进行多次的转换。如果进行多次字节序的转换，最后生成的网络字节序的值可能是错误的。例如：

- 对于主机为小端字节序的系统，进行两次字节序转换的过程如下图所示，经过两次转换，最终的值与最初的主机字节序相同。

- 对于主机为大端字节序的系统，无论进行多少次字节序的转换，最终的值与最初的主机字节序相同。

![小端系统中的变量多次字节序转换](/images/htos-conversion.png)

下面的例子是对16位数值和32位数值进行字节序转换，每种类型的数值进行两次转换，最后打印结果。

```c
#include <stdio.h>
#include <arpa/inet.h>

#define BITS16 16
#define BITS32 32

typedef union two_bytes_order {
    unsigned short int value;
    unsigned char byte[sizeof(unsigned short)];
} TWO_BYTES_ORDER_UN;

typedef union four_bytes_order {
    unsigned long int value;
    unsigned char byte[sizeof(unsigned long int)];
} FOUR_BYTES_ORDER_UN;

void showValue(void *begin, int flag) {
    for (int i = 0; i < ((flag) / 8); i++) {
        printf("%x (%p) ", *((unsigned char *)begin + i), (unsigned char *)begin + i);
    }
    printf("\n");
}

int main(int argc, char *argv[]) {
    TWO_BYTES_ORDER_UN v16_orig, v16_turn1, v16_turn2;
    FOUR_BYTES_ORDER_UN v32_orig, v32_turn1, v32_turn2;

    v16_orig.value = 0xabcd;
    v16_turn1.value = htons(v16_orig.value);
    v16_turn2.value = htons(v16_turn1.value);

    v32_orig.value = 0x12345678;
    v32_turn1.value = htonl(v32_orig.value);
    v32_turn2.value = htonl(v32_turn1.value);

    printf("16 host to network byte order change:\n");
    printf("\torig:    ");
    showValue(v16_orig.byte, BITS16);
    printf("\t1 times: ");
    showValue(v16_turn1.byte, BITS16);
    printf("\t2 times: ");
    showValue(v16_turn2.byte, BITS16);

    printf("32 host to network byte order change:\n");
    printf("\torig:    ");
    showValue(v32_orig.byte, BITS32);
    printf("\t1 times: ");
    showValue(v32_turn1.byte, BITS32);
    printf("\t2 times: ");
    showValue(v32_turn2.byte, BITS32);
    return 0;
}
```

小端模式到网络字节序转换的测试结果：

```text
$ gcc byteOrder2.c -o byteOrder2
$ ./byteOrder2
16 host to network byte order change:
        orig:    cd (0x7ffcd6ad66de) ab (0x7ffcd6ad66df)
        1 times: ab (0x7ffcd6ad66dc) cd (0x7ffcd6ad66dd)
        2 times: cd (0x7ffcd6ad66da) ab (0x7ffcd6ad66db)
32 host to network byte order change:
        orig:    78 (0x7ffcd6ad66d0) 56 (0x7ffcd6ad66d1) 34 (0x7ffcd6ad66d2) 12 (0x7ffcd6ad66d3)
        1 times: 12 (0x7ffcd6ad66c8) 34 (0x7ffcd6ad66c9) 56 (0x7ffcd6ad66ca) 78 (0x7ffcd6ad66cb)
        2 times: 78 (0x7ffcd6ad66c0) 56 (0x7ffcd6ad66c1) 34 (0x7ffcd6ad66c2) 12 (0x7ffcd6ad66c3)
```

16位变量 0xabcd 在内存中的表示方式为 cd 在前，ab 在后；进行一次字节序转换后变为 ab 在前，cd 在后。在进行第一次转换后字节序发生了变化，而进行第二次字节序转换后与原始的排列方式一致。

## 大小端的转换

上面的代码中 `htons` 和 `htonl`，分别给出了主机字节序到网络字节序的 `short` 和 `long` 类型的转换接口，那么这个接口是如何实现大小端转换的呢？

**通过位运算的方式来实现大小端的转换**：

```c
#include <stdio.h>

unsigned int little_endian_to_big_endian_4bytes(unsigned int value) {
    return ((value & 0x000000ff) << 24) | 
           ((value & 0x0000ff00) << 8)  | 
           ((value & 0x00ff0000) >> 8)  | 
           ((value & 0xff000000) >> 24);
}

unsigned short little_endian_to_big_endian_2bytes(unsigned short value) {
    return ((value & 0x00ff) << 8) | ((value & 0xff00) >> 8);
}

int main(int argc, char *argv[]) {
    unsigned int little_value4 = 0x12345678;
    unsigned short little_value2 = 0xabcd;
    unsigned int big_value4 = little_endian_to_big_endian_4bytes(little_value4);
    unsigned short big_value2 = little_endian_to_big_endian_2bytes(little_value2);
    printf("LE2 value: 0x%04x\n", little_value2);  // LE2 value: 0xabcd
    printf("BE2 value: 0x%04x\n", big_value2);  // BE2 value: 0xcdab
    printf("LE4 value: 0x%08x\n", little_value4);  // LE4 value: 0x12345678
    printf("BE4 value: 0x%08x\n", big_value4);  // BE4 value: 0x78563412
    return 0;
}
```

## 多变量存储在连续内存中的字节序转换

示例1：一个2字节变量存储在连续的内存中，与一个4字节变量存储在连续的内容中，从小端主机字节序转换为大端网络字节序后的结果一样吗？一样的！

```c
#include <stdio.h>
#include <arpa/inet.h>

typedef struct four_bytes_order {
    unsigned short int val1;
    unsigned short int val2;
    unsigned long int val3;
} FOUR_BYTES_ORDER_S;

#define BITS16 16
#define BITS32 32

void showValue(void *begin, int flag) {
    for (int i = 0; i < ((flag) / 8); i++) {
        printf("%x (%p) ", *((unsigned char *)begin + i), (unsigned char *)begin + i);
    }
    printf("\n");
}

int main(int argc, char *argv[]) {
    FOUR_BYTES_ORDER_S sOrig, sTurn;
    
    sOrig.val1 = 0x1234;
    sOrig.val2 = 0x5678;
    sTurn.val1 = htons(sOrig.val1);
    sTurn.val2 = htons(sOrig.val2);

    printf("16 + 16 host to network byte order change:\n");
    printf("\thost:    ");
    showValue(&sOrig.val1, BITS32);
    printf("\tnetwork: ");
    showValue(&sTurn.val1, BITS32);

    sOrig.val3 = 0x12345678;
    sTurn.val3 = htonl(sOrig.val3);
    printf("32 host to network byte order change:\n");
    printf("\thost:    ");
    showValue(&sOrig.val3, BITS32);
    printf("\tnetwork: ");
    showValue(&sTurn.val3, BITS32);

    return 0;
}
```

对于上述连续内存的两个2字节变量和一个4字节变量，转换成的网络字节序都是 `0x12345678`：

```text
$ gcc byteOrder2.c -o byteOrder3
$ ./byteOrder3
16 + 16 host to network byte order change:
        host:    34 (0x7fffd69b9a10) 12 (0x7fffd69b9a11) 78 (0x7fffd69b9a12) 56 (0x7fffd69b9a13)
        network: 12 (0x7fffd69b9a00) 34 (0x7fffd69b9a01) 56 (0x7fffd69b9a02) 78 (0x7fffd69b9a03)
32 host to network byte order change:
        host:    78 (0x7fffd69b9a18) 56 (0x7fffd69b9a19) 34 (0x7fffd69b9a1a) 12 (0x7fffd69b9a1b)
        network: 12 (0x7fffd69b9a08) 34 (0x7fffd69b9a09) 56 (0x7fffd69b9a0a) 78 (0x7fffd69b9a0b)
```

示例2：一个变量 `0x12345678abcd9876`，分别以 `short + long + short` 和 `long + long` 变量存储在一个连续内容中，那么两种存储的小端主机字节序和大端网络字节序在内存中的存储的值顺序一致吗？一致的！

```text
val_le = 0x12345678abcd9876

short + long + short
    a = 0x1234
    b = 0x5678abcd
    c = 0x9876

分别给出其主机序和网络序：
    内存低地址 -----------------------> 内存高地址
    小端主机序：34 12 | cd ab 78 56 | 76 98
    大端网络序：12 34 | 56 78 ab cd | 98 76

long + long
    d = 0x12345678
    e = 0xabcd9876

分别给出其主机序和网络序：
    内存低地址 -----------------------> 内存高地址
    主机序：78 56 34 12 | 76 98 cd ab
    网络序：12 34 56 78 | ab cd 98 76
```

可以看出, 两种组合变量的主机序到网络序转换后的结果是一致的！

> 参考：
> 
> 1. https://blog.51cto.com/u_15249901/4893764
> 2. https://blog.csdn.net/Jmilk/article/details/106898871
