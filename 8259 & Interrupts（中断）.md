8086/8088中共256种中断 00-0FFh
中断向量表，每个中断占4个Byte（2个字节存CS，2个字节存IP）
Interrupt Vector指向Interrupt Service Routine中的入口地址
![Pasted image 20260502141854.png](assets/Pasted%20image%2020260502141854.png)
## 分类
### 硬件中断INTR：
#### maskable：
IF=1时才使能 STI/CLI设置/清除IF
中断流程：
- *外设*发送INTR给CPU，CPU在每条指令的最后一个时钟周期check，如果IF=1并且INTR是高电平，CPU就发送两个低电平pulse给外设，收到INTRA（interrupt acknowledge）之后IO发送中断向量号。
- 若前发生的是 **外部中断**，CPU从数据总线读取向量号N
- push FR到堆栈，自动clear IF和TF（clear IF--禁止同时响应其他中断 clear TF--关闭单步调试）
- push CS:IP（记录被中断程序的下一条指令）
- load ISR的CS:IP（根据中断类型号，在中断向量表中找到对应的入口地址，跳转执行）
- 最后IRET，pop CS:IP，FR
#### non-maskable：
不受IF影响
检查NMI pin：输入信号，上升沿且持续两个周期的高电平有效。产生INT02中断

| `CALL FAR`（远调用）              | `INT n`/外部中断                  |
| ---------------------------- | ----------------------------- |
| 可以跳转到 1MB 内的任何位置             | 只能跳转到固定的位置，通过查找中断向量表找到对应的 ISR |
| 出现在指令的执行序列中                  | 可以在任何时候发生（由硬件触发）              |
| 不能被mask                      | 可以被mask                       |
| 保存的是下一条指令的 `CS:IP`           | 保存的是 `FR`+ 下一条指令的 `CS:IP      |
| 最后一条指令：`CALL FAR` 对应  `RETF` | `IRET`                        |

### 软件中断INT：
不受IF影响，遇到‘INT XX’就直接执行ISR程序
预定义的条件中断:
- **“INT 00”（除法错误）**
    除以零，或商太大（超出寄存器所能表示的范围）
- **“INT 01”（单步）**
    `TF = 1`，CPU 会在每条指令执行后产生一个 INT 1 中断，用于调试
- **“INT 03”（断点）**
    当 CPU 执行到程序中设置的断点时，CPU 产生 INT 3 中断，用于调试
- **“INT 04”（有符号数溢出）**
    由 **INTO** 指令触发，在算术指令之后检查 OF（溢出标志位）

## 优先级：
INT指令>NMI>INTR
- 当多个硬件中断（INTR）同时请求时，CPU 如何决定先处理哪一个。
1.软件查询
中断优先级完全由程序检查外设的顺序决定，而不是由硬件固定死的。
![Pasted image 20260502142829.png](assets/Pasted%20image%2020260502142829.png)
2.硬件查询
中断优先级由设备在链中的物理位置决定
- ==INTR 线：向 CPU 发中断请求==
- ==INTA 线：CPU 发出的中断响应信号，逐级往下传递==
离 CPU 越近，优先级越高
![Pasted image 20260502143237.png](assets/Pasted%20image%2020260502143237.png)
## 8259(Programmable Interrupt Controller)
- by default **IR0 优先级最高，IR7 最低**
- 但 IR0 对应的**中断类型号** 可以是任意值（通过初始化编程设置） 例如可以让 IR0 对应 INT 8，IR1 对应 INT 9，等等
- 可处理64 interrupt inputs（级联），可以mask
![Pasted image 20260502144829.png](assets/Pasted%20image%2020260502144829.png)
## Interrupt Nesting
优先级高的interrupt可以插入优先级低的interrupt
优先级低的interrupt要在前一个interrupt执行完了之后才能执行
![Pasted image 20260502144335.png](assets/Pasted%20image%2020260502144335.png)