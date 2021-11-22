---
templateKey: blog-post
title: Serial Communicaion
date: 2021-01-20T15:04:10.000Z
featuredpost: false
featuredimage: /img/flavor_wheel.jpg
description: 串口通信
tags:
  - Serial Communication
---

|    日期    |                          进度                           |
| :--------: | :-----------------------------------------------------: |
| 2020-01-20 |                     通讯+USART简介                      |
| 2020-01-21 | STM32中的USART<br>重点在于寄存器及其标志位、中断<br>DMA |

## 通讯

### 基本概念

#### 串行通讯

单车道，同一时刻只能传输一个数据位的数据

#### 并行通讯

多车道，同时传输多个数据位的数据

#### 全双工 半双工 与 单工

| 通讯方式 | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| 全双工   | 同一时刻，两个设备之间可以同时收发数据                       |
| 半双工   | 两个设备之间可收发数据，但不能同时                           |
| 单工     | 任何时刻只能进行一个方向的通讯，一个设备只能发，另一个只能收 |

#### 同步通讯

有一根时钟线，在时钟信号的驱动下两设备同步数据，如下图双方在时钟信号的上升沿或者下降沿对数据线进行采样。

数据中大部分为有效数据，因为双方都依赖时钟信号，同时收发，精度较高。

![](https://gitee.com/cutice/my-blog-images/raw/master/images/20210120173335.png)

<center>同步通讯</center>

#### 异步通讯

没有时钟线，发送设备将数据打包成数据帧的形式发送，需要双方约定传输速率、传输格式、校验格式等。

内容中含有数据帧的标识符、校验位等，收发消息的效率比较低。

#### 比特率与波特率

比特率： 每秒传输的二进制位数 bit/s

波特率： 每秒传输的“码元”个数， Baudrate

码元    ： 通讯中常用的时间间隔相同的符号表示一个二进制数字，

​	如若0V代表0，5V代表1，那么一个码元可以表示两种状态0和1，所以一个码元=一个二进制bit。

​    若0V,2V,4V,6V代表00,01,10,11，那么一个码元代表两个二进制bit，波特率位比特率的一半。

## USART

### 简介

串口通讯（Serial Communication）是设备常用的串行通讯方式，串口通讯协议包括物理层和协议层，限于水平仅作简单介绍。

在物理层上，主要有DB9接口、RS-232标准、TTL标准。DB9接口是指两设备间进行串口通讯所使用的物理接口，两者插接串口信号线。串口信号线是采用RS-232标准来传输信号。又因为RS-232电平标准的信号不能被设备直接识别，所以要经过一个“电平转换芯片”来转换成设备能识别的TTL校准的电平信号。

目前，一般使用时，直接使用RXD（收信息）、TXD（发信息）、GND（接地线）来传输信号，还有RTS,CTS,DSR,DTR,DCD目前用不到也就无需了解。

在协议层上，数据帧由**起始位、主体数据、校验位、停止位**组成，通讯双方约定一直才能正常收发。

起始位有一个逻辑0的数据位表示，停止位信号可由0.5、1、1.5、2个逻辑1的数据位表示。

有效数据（主体数据）通常约定为5、6、7、8位。

校验位，可选选项，常见方法为奇校验（odd）、偶校验（even）、0校验（space）、1校验（mark）以及无校验（noparity）。

| 校验方式 | 介绍                                                         |
| -------- | ------------------------------------------------------------ |
| 奇校验   | 要求有效数据和校验位中“1”的个数为奇数，<br >比如一个 8 位长的有效数据为：01101001，<br>此时总共有 4 个“1”，为达到奇校验效果，校验位为“1”，<br/>最后传输的数据将是 8 位的有效数据加上 1 位的校验位总共 9 位 |
| 偶校验   | 与奇校验同理                                                 |
| 0 校验   | 是不管有效数据中的内容是什么，校验位总为“0”<br/>1 校验是校验位总为“1” |

#### STM32中的USART

##### 功能引脚

| 功能引脚 | 作用                                                         |
| -------- | ------------------------------------------------------------ |
| TX       | 发送数据                                                     |
| RX       | 接收数据                                                     |
| nRTS     | Request To Send [请求发送]  n表示低电平有效 只适用于硬件流控制 |
| nDE      | 驱动器使能                                                   |
| nCTS     | Clear To Send [清除以发送] n表示低电平有效  只适用于硬件流控制 |
| SCLK     | 发送器时钟输出引脚 仅适用于同步模式  [一般不用]              |

各引脚对应的GPIO口查手册并开启。

UART 异步传输 无SLCK,nCTS,nRTS功能引脚。

##### 数据寄存器

USART数据寄存器  USART_DR

USART控制寄存器  USART_CR1

USART_DR只有低9位有效，第9位是否有效取决于USART_CR1的M位设置，当M位为0时，USART_DR只用8位数据字长，当M位为1表示9位数据字长，一般用8位数据字长。

USART_DR包含已发送数据或者接收到数据，它实际上包括两个寄存器，一个是专门发送的可写**TDR**,一个是专门接收的可读**RDR**。发送时写入数据存在TDR中，读取时，自动提取RDR。

发送时把TDR内容转移到发送移位寄存器，然后把移位寄存器每一位发出去；

接收时把接收到的每一位按顺序保存在接收移位寄存器内再转移到RDR。

USART支持DMA传输，重要的数据收发最好通过DMA进行收发。

##### 控制器

发送器、接收器、唤醒单元、中断控制。

###### 发送器

发送数据时有几个重要的标志位：

USART_CR1的TE为1时，启动数据发送，数据在TX引脚输出。

| 名称 |                 描述                 |
| :--: | :----------------------------------: |
|  TE  |               发送使能               |
| TXE  | 发送寄存器为空，发送**单个字节**使用 |
|  TC  |    发送完成，发送**多个字节**使用    |
| TXIE |         发送完成**中断使能**         |

###### 接收器

将USART_CR1寄存器的RE位置1，使能USART接收，接收器再RX线开始搜索起始位。

收到起始位后就根据RX线电平状态把数据存放在**接收移位寄存器**内，接收完成后把接收移位寄存器转移到**RDR**内，同时把**USART_SR寄存器**的**RXNE**位置成1，与之同时若USART_CR2寄存器的RXNEIE置1可以产生中断。

接收数据时重要标志位：

| 名称   | 描述             |
| ------ | ---------------- |
| RE     | 接收使能         |
| RXNE   | 读数据寄存器非空 |
| RXNEIE | 发送完成中断使能 |

过采样：为了得到一个信号，要用比其频率高的采样信号去检测，来从噪声中提取有效数据，称为过采样。采样信号过高，功耗、运算增加，得不偿失，一半为8倍过采样或者16倍过采样。

USART_CR1寄存器的OVER8为1时采用8倍过采样，OVER8为0时采用16倍过采样。

8倍过采样即用8个采样信号采样一位数据，例如8个采样数据中如果6个0，2个1，则该位为0。

###### 校验控制

串口发送数据通常开启USART_CR1的M位=1，传输：8位主体数据+1位校验位，将USART_CR1的PCE=1启动奇偶校验控制，这将由硬件自主完成，如果校验成功正常使用，若校验失败硬件会把USART_SR寄存器的PE=1，并产生奇偶校验中断。

##### 中断控制

中断在串口收发中占据极为重要的位置，在串口传输中需要经常使用中断，而通过操纵中断、判断中断错误类型、解决中断错误更是重中之重。在平时使用中一般是接收中断会出现问题，尤其在串口在接收单帧高数据量、频率高的信号时，容易陷入串口卡断的情况，这种情况之下更应该了解串口中断的错误方式来解决。

UART中断请求种类丰富，需要了解并在必要时刻查询。

| 中断事件                                 | 事件标志  | 使能控制位 |
| ---------------------------------------- | --------- | ---------- |
| 发送寄存器为空                           | TXE       | TXEIE      |
| CTS标志                                  | CTS       | CTSIE      |
| 发送完成                                 | TC        | TCIE       |
| 准备好读取接收到的数据                   | RXNE      | RXNEIE     |
| 检测到上溢错误                           | ORE       | RXNEIE     |
| 检测到空闲线路                           | IDLE      | IDLEIE     |
| 奇偶校验错误                             | PE        | PEIE       |
| 断路标志                                 | LBD       | LBDIE      |
| 多缓冲通信中噪声标志<br>上溢错误和帧错误 | NF/ORE/FE | EIE        |

上面中较为重要的为TC,RENE,ORE,PE,FE,NF。

在使用串口并发现错误时，应该查表并在UART_ErrorHandle()中逐一检验，清除中断标志位，最后再开启串口。

##### 代码配置

UART初始化结构体：

```c
typedef struct {
	uint32_t BaudRate; 			  // 波特率 一般选 115200
	uint32_t WordLength;          // 字长   一般8位 || 9位
    uint32_t StopBits;            // 停止位  一般选1位
	uint32_t Parity;              // 校验位  设定UART_CR1的PCE位和PS位
	uint32_t Mode;                // USART模式 设定CR1的RE和TE位，即使能接收和发送 
	uint32_t HwFlowCtl;           // 硬件流设置 设置为无硬件流
	uint32_t OverSampling;  	  // 过采样设置，8 倍或者 16 倍 
} USART_InitTypeDef;
```

要点：

1) 使能 RX 和 TX 引脚 GPIO 时钟和 USART 时钟； 
2) 初始化 GPIO，并将 GPIO 复用到 USART 上； 
3) 配置 USART 参数； 
4) 配置中断控制器并使能 USART 接收中断； 
5) 使能 USART； 
6) 在 USART 接收中断服务函数实现数据接收和发送。

中断服务函数 USART_IRQHandler(void);存放在stm32f4xx_it.c中。样例如下：

```c
void DEBUG_USART_IRQHandler(void) {
	uint8_t ch = a;
    if (__HAL_UART_GET_FLAG(&UartHanle, UART_FLAG_RXNE) != RESET) {
        ch = (uint16_t) READ_REG(UartHandle.Instance->DR);
        WRITE_REG(UartHandle.Instance->DR, ch);
    }
}
```

使能接收中断后，当USART接收到数据就会执行USART_IRQHandler函数

`__HAL_UART_GET_FLAG`函数用来获取中断标志，即前面各表的标志位。判断是否为RESET（是否为真）。

`READ_REG`函数读取数据赋值ch，读取过程中会清除`UART_FLAG_RXNE`标志位。

最后`WRITE_REG`函数把数据发给源设备。

在读取前，`UART_FLAG_RXNE`为`RESET`即为真，说明读取寄存器有东西，上面这个函数相当于读取、转发了数据，并且将`UART_FLAG_RXNE`标志清除，使得外设重新回到了读取寄存器为空的状态。

## DMA  直接存储区访问

### DMA 简介

不通过CPU操作，快速移动内存数据，实现高效的数据传输。

DMA的外设通道选择、仲裁器、FIFIO模式存储器端口、外设端口与编程端口太多不看。DMA的传输模式、源地址与目标地址、流控制器、循环模式、直接模式、双缓冲模式也太长不看。~~我就是懒狗~~

### DMA 中断

发送：每个DMA数据流可以产生以下中断：

1. 半传输： DMA数据发送一半， **HTIF**标志位被置为1，如果使能了**HTIE**中断控制会产生半传输中断。

2. 传输完成： DMA数据传输完成时**TCIF**标志位置为1 ，使能了**TCIE**中断控制将有传输完成中断。
3. 传输错误： 访问总线错误或者双缓冲模式下试图访问“受限”的存储器时**TEIE**标志位置为1，会有传输错误中断。
4. **FIFO**错误： 发送**FIFO**下溢或者上溢时，**FEIF标志位**置为1，若使能了**FEIE中断控制**将产生FIFO错误中断。
5. 直接模式错误：在外设到存储器的直接模式下，因为存储器总线没得到授权，使得先前数据没有完成被传输到存储器空间上，此时 DMEIF 标志位被置 1，如果使能 DMEIE 中断控制位将产生直接模式错误中断。

`__HAL_DMA_GET_FLAG` 函数获取DMA数据流事件标志位的当前状态。

ErrorCode：DMA错误码，包含：

|                                  |                             |
| :------------------------------: | :-------------------------: |
|              无错误              |     HAL_DMA_ERROR_NONE      |
|             传输错误             |      HAL_DMA_ERROR_TE       |
|            FIFO 错误             |      HAL_DMA_ERROR_FE       |
|           直接模式错误           |      HAL_DMA_ERROR_DME      |
|             超时错误             |    HAL_DMA_ERROR_TIMEOUT    |
|             参数错误             |     HAL_DMA_ERROR_PARAM     |
| 没有回调函数正在执行退出请求错误 |    HAL_DMA_ERROR_NO_XFER    |
|          不支持模式错误          | HAL_DMA_ERROR_NOT_SUPPORTED |

---

在串口通讯中经常会遇到神奇的bug，比如串口接收消息过一会不能接收了，比如大量数据蜂拥而至（100HZ，每帧20Byte）将串口卡死等等情况，这些往往是串口中断标志位没有清除导致。

下面代码为往年电控做的UART错误处理函数

```c
uint16_t ErrorTest;
void HAL_UART_ErrorCallback(UART_HandleTypeDef *UartHandle) {
	ErrorTest = UartHandle->ErrorCode;
    tx_free	  = 1;
    uint32_t isrflags = READ_REG(UartHandle->Instance->SR); // 手册上有讲，清错误要先读SR 
    // SR 应该是状态寄存器 
    // 读 + 清 PE[奇偶校验]， 
    if ( (__HAL_UART_GET_FLAG(UartHandle, UART_FLAG_PE)) != RESET ) {
        READ_REG(UartHandle->Instance->DR);  // PE清标志，第二步读DR
        __HAL_UART_CLEAR_FLAG(UartHandle, UART_FLAG_PE); //  清标志
        UartHandle->gState  = HAL_UART_STATE_READY;
        UartHandle->RxState = HAL_UART_STATE_READY; 
    }
    // 读FE[frame error],读DR,清FE
    if  ( (__HAL_UART_GET_FLAG(UartHandle, UART_FLAG_FE)) != RESET) {
        READ_REG(UartHandle->Instance->DR);
        __HAL_UART_CLEAR_FLAG(UartHandle, UART_FLAG_FE);
        UartHandle->gState  = HAL_UART_STATE_READY;
        UartHandle->RxState = HAL_UART_STATE_READY;
    }
    // 读NE[noise error]，读DR,清NE
    if  ( (__HAL_UART_GET_FLAG(UartHandle, UART_FLAG_NE)) != RESET) {
        READ_REG(UartHandle->Instance->DR);
        __HAL_UART_CLEAR_FLAG(UartHandle, UART_FLAG_NE);
        UartHandle->gState  = HAL_UART_STATE_READY;
        UartHandle->RxState = HAL_UART_STATE_READY;
    }
    
    // 读ORE[overrun error], 读DR, 清ORE
        if  ( (__HAL_UART_GET_FLAG(UartHandle, UART_FLAG_ORE)) != RESET) {
        READ_REG(UartHandle->Instance->DR);
        __HAL_UART_CLEAR_FLAG(UartHandle, UART_FLAG_ORE);
        UartHandle->gState  = HAL_UART_STATE_READY;
        UartHandle->RxState = HAL_UART_STATE_READY;
    }
	if (UartHandle == & RC_UART) {
        InitRemoteControl();
    } else if (UartHandle == &JUDGE_UART) {
        InitJudgeUart();
    } else if (UartHandle == &AUTOAIM_UART) {
        AutoAimReceiveStart();
    } else if (UartHandle == &AUTOAVOID_UART) {
        AutoAvoidRevciveStart();
    }
}
```

---

查看源码的一点发现：

```
gState:  与全局句柄管理相关 也与发送操作相关的UART状态
```

几个比较好的网页：

**1. 一种基于HAL+UART+DMA环形队列接收数据实现方法**

https://www.stmcu.org.cn/module/forum/thread-620881-1-1.html

**2.慎用HAL_UART_RxCpltCallback中调用HAL_UART_Receive_IT** 

https://www.stm32cube.com/article/139

