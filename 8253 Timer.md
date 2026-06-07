作用：计数降频。有三个独立的计数器；可以被 divided by162536 （⼆进制）或 者 110000 （ BCD ）产⽣的 output 有方波 /one-shot 。
## 1.内部结构

| ![Pasted image 20260501204753.png](assets/Pasted%20image%2020260501204753.png) | ![Pasted image 20260501204736.png](assets/Pasted%20image%2020260501204736.png) |
| ----------------------------------------- | ----------------------------------------- |
三个独立的计数器。最大计数初值是0 （2^16） 
CPU写入CR放初值control register —— 搬到 CE开始倒数control element——OL输出当前倒数值，但是需要先 latch 住output latch

**8253 内部有一个 Control Word Register（控制字寄存器）**，但它不是独立可寻址的寄存器，而是一个**只写、不可读**的寄存器，用于接收 CPU 发来的控制字，决定三个独立计数器的工作方式、计数格式和读写操作。
- **Write（写）**：**CPU → 8253**
- **Read（读）**：**8253 → CPU**
## 2. PINs

| 图片                                        | 功能                                                                                                                                                                                                    |
| ----------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ![Pasted image 20260502114432.png](assets/Pasted%20image%2020260502114432.png) | - 三态的 data bus buffer<br>- ~CS 用于选片<br>- ~RD,~WR 用来确定读/写 (isolated/memory-mapped IO Read/Write) <br>- A1A0 ：00用于counter0，01用于counter1，10 用于 counter2，11用于control <br>- 8254 最多 10MHz ， 8253 最多 2.6MHz |

## 3.Control Register
![Pasted image 20260502111750.png](assets/Pasted%20image%2020260502111750.png)
![Pasted image 20260502111902.png](assets/Pasted%20image%2020260502111902.png)

![Pasted image 20260502113758.png](assets/Pasted%20image%2020260502113758.png)
## 4.Setting Up a computer
![Pasted image 20260501211848.png](assets/Pasted%20image%2020260501211848.png)
![Pasted image 20260501212258.png](assets/Pasted%20image%2020260501212258.png)
(a)00110111
D0=1说明用的是BCD模式，后面的4282H就是在BCD模式下的
RW1RW2=11 说明先读低字节再读高字节
**PS.A1A0在这个期间也在变化**
(b)10110110
(c)做除法，M表示10⁶ 注意进制的统一（decimal十进制，hex16进制）

## 5.Working Mode
CPU-->CR即刻 ==CR-->CE需要一个时钟周期==
Gate: 时钟上升沿检查
CE(倒数): Gate=1&WR后的一个时钟下降沿
==如果已经从CR-->CE，在时钟上升沿检测到Gate=1，在当前这个时钟周期的下降沿就可以开始计数==

几个关键点：
是否重复；什么时候触发；高低电平的变化

| Mode 0![Pasted image 20260501224134.png](assets/Pasted%20image%2020260501224134.png)       | Mode 1![Pasted image 20260501224109.png](assets/Pasted%20image%2020260501224109.png) |
| ----------------------------------------------------- | ----------------------------------------------- |
| ==不会重复==，software trigger，Interrupt on Terminal Count | ==不会重复==，hardware retriggerable one-shot        |
| 在set mode(CW)之后输出变==低==                               | 在set mode(CW)之后输出变==高==                         |
| ==下一个时钟周期的下降沿==且Gate=1，开始计数，==保持低电平==                 | ==Gate上升沿后==的时钟下降沿变成==低电平==，开始计数                |
| Gate=0暂停计数                                            | 未到0，Gate又上升沿，从原始值开始再次计数                         |
| ===0后变成高电平==                                          | ===0后变成高电平==                                    |
| 新的计数值会在下一个时钟周期的下降沿覆盖                                  | 新的计数值会在Gate上升沿后的时钟下降沿覆盖                         |


| Mode 2![Pasted image 20260501231257.png](assets/Pasted%20image%2020260501231257.png)                                             | Mode 3![Pasted image 20260501231503.png](assets/Pasted%20image%2020260501231503.png)                     |
| -------------------------------------------------------------------------------------- | -------------------------------------------------------------- |
| 重复，software trigger，rate generator                                                     | 重复，software trigger，Square Wave rate generator                 |
| 一开始处于高电平                                                                               | 一开始处于高电平                                                       |
| ==从下一个时钟周期的下降沿==开始计数，==n-1个高电平+1个低电平==                                                 | ==从下一个时钟周期的下降沿==开始计数，==大约50%的占空比==                             |
|                                                                                        | N=偶数，每个时钟周期-2，0后变成低电平，翻转成N，-2-2……；N=奇数，-1-2-2……变成低电平-3-2-2……   |
| GATE 低电平时，计数器停在当前值<br>GATE 恢复高电平后，计数从停下的值继续<br>--如果在 OUT 为低（负脉冲期间）GATE 被拉低，OUT 立即返回高电平 | 同左 如果在 OUT 为低时，GATE 被拉低，OUT 立即返回高电平，Gate上升沿后一个时钟下降沿，计数器回到初始计数值 |
| 写入新值后，无 GATE 上升沿触发--当前计数周期结束后用新值                                                       | 没有收到触发（或半周期已经结束）--- 新计数值在当前半周期结束时加载                            |
| 写入新值后，且在当前周期结束前收到 GATE 上升沿触发--下一个 CLK 下降沿，从新值计数，覆盖                                     | 在半个周期结束前收到了“触发”--- 在下一个 CLK 下降沿，计数器加载新计数值                      |
| N=1 illegal                                                                            | ==两者都不需要Gate上升沿触发，除非在周期中写入新值。但是Gate变成低电平后会暂停计数==               |

| Mode 4![Pasted image 20260502110712.png](assets/Pasted%20image%2020260502110712.png) | Mode 5![Pasted image 20260502110744.png](assets/Pasted%20image%2020260502110744.png) |
| ----------------------------------------------- | ----------------------------------------------- |
| 不重复，Software Triggered Strobe                   | 不重复，Hardware Triggered Strobe                   |
| CW后处于高电平                                        | CW后处于高电平                                        |
| N个高电平，1个低电平                                     | N个高电平，1个低电平                                     |
| Gate=0计数暂停                                      | 需要Gate上升沿做trigger                               |
| 新的计数值会隔一个CLK周期直接在后面计数                           | 没到0，又遇到一个Gate上升沿，从原始值开始再次计数                     |
| 如果写入两个Byte（CR中D5D4=11）                          | 新的计数值在Gate上升沿后的CLK下降沿载入                         |
| 第一个Byte不动，写入第二个Byte后，下一个CLK cycle载入             |                                                 |
如何写控制字 mode counter
改变频率
assembly program
波形图