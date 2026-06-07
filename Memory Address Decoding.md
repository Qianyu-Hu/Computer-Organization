//这版有点问题，少了一张图，另有一些隐式的代码捣蛋
## Memory address decoding circuitry内存地址译码电路
CPU通过基址×10H+偏移量计算操作数的物理地址（20位）对于32K×8bit的RAM芯片只有15根线，故用最高的5位做chip selection
通过译码找到比 2^数据线 多的地址

| ![Pasted image 20260503180807.png](assets/Pasted%20image%2020260503180807.png) | ![Pasted image 20260503180839.png](assets/Pasted%20image%2020260503180839.png) |
| -------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| 80000H-8FFFFH                                                              | C0000-CFFFFH                                                               |
注意只有画出来的这几条线，需要固定它们的值，没有画出来的线都随意
![Pasted image 20260503182242.png](assets/Pasted%20image%2020260503182242.png)

## Data Integrity数据完整性
### Checksum Byte for ROM
将所有byte加起来，丢掉进位再取补码。将所有data和checksum加起来，和为0(低8位)则正确
### Parity Bit for DRAM
even parity：加上parity bit后1的总数是偶数
odd parity：加上parity bit后1的总数是奇数
==8086中的PF用的是odd parity==
<div style="break-after: page;"></div>

## Memory Organization in 8086
![Pasted image 20260503225347.png](assets/Pasted%20image%2020260503225347.png)

- **数据总线宽度**：8086 CPU 有 16 根数据线（D0-D15），理论上一次可以传输 16 位（2 个字节）的数据。
- **内存芯片宽度**：当时的主流内存芯片每个只能提供 8 位（1 个字节）的数据。
- **解决方案**：**同时使用两个 8 位的内存芯片**，将它们“拼”成一个 16 位的内存系统。
    - **偶库**：连接 D0-D7（低 8 位数据线），负责存储**偶数地址**（如 0x00, 0x02, 0x04...）的字节。
    - **奇库**：连接 D8-D15（高 8 位数据线），负责存储**奇数地址**（如 0x01, 0x03, 0x05...）的字节。

这样，当 CPU 需要读取地址 0x00 和 0x01 的两个字节组成的一个字（16 位）时，就可以**同时**从偶库（读 0x00）和奇库（读 0x01）中获取数据，一次完成。

**Byte-Memory Operations** 根据奇/偶地址，到对应的库中找
**Aligned Word-Memory Operations--even address**
BHE=0在一个bus cycle中可以同时读取偶地址X和奇地址X+1
**Misaligned Word-Memory Operations**--odd address
第一个bus cycle读奇地址，第二个bus cycle读偶地址


<div style="break-after: page;"></div>
## I/O in 86
![Pasted image 20260503233507.png](assets/Pasted%20image%2020260503233507.png)
port为地址，但实际上是向AL写入port中的数字，或将AL中的数字写入port