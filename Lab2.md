# Lab2

74 选中Device3，Addr1再选中Port B

U5 PortA的低四位用于控制显示的位置，高四位用于控制D1\-D4的二极管

PortB用于控制七段码的显示

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=NzI1OGUxZjkyNzM3ZThlN2NlZGQwZmIxOWEzYjNmMDhfNDBiMzczY2YxMzA3NjQ1OWFmMDI2NzFhZTFlODU2YjdfSUQ6NzYzMDExODU4OTA5NzM0ODI4Ml8xNzgwODQ0ODEzOjE3ODA5MzEyMTNfVjM)

前两行：输出到哪一位，控制哪几个灯亮

中间两行：控制哪几个晶体管亮

requirement：读入PortC的信息



七段码如何编码

```Plain Text
; 共阴极数码管 0~F 标准段码表 SEGTAB
; 位定义：bit7(DP) bit6(g) bit5(f) bit4(e) bit3(d) bit2(c) bit1(b) bit0(a)
SEGTAB  DB  3FH   ; 0  a,b,c,d,e,f亮  g灭,DP灭
        DB  06H   ; 1  b,c亮
        DB  5BH   ; 2  a,b,g,e,d亮
        DB  4FH   ; 3  a,b,g,c,d亮
        DB  66H   ; 4  f,g,b,c亮
        DB  6DH   ; 5  a,f,g,c,d亮
        DB  7DH   ; 6  a,f,g,c,d,e亮
        DB  07H   ; 7  a,b,c亮
        DB  7FH   ; 8  全亮
        DB  6FH   ; 9  a,b,c,d,f,g亮
        DB  77H   ; A
        DB  7CH   ; b
        DB  39H   ; C
        DB  5EH   ; d
        DB  79H   ; E
        DB  71H   ; F
```

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=OTA0MDljMGRkZjg2NmU2OTY3ZWIxNzkwZWMwM2U0NmRfMzJmNzVhMmUwYWVlZTY4NjNmNWY5OWYxYjMyY2ZmMDBfSUQ6NzYzMDMwNjQ3NDU5NzkxMTc1Nl8xNzgwODQ0ODEzOjE3ODA5MzEyMTNfVjM)

```SQL
功能：控制7段数码管的显示                                |
; 编写：《嵌入式系统原理与实验》课程组                     |
;-----------------------------------------------------------
                DOSSEG
                .MODEL        SMALL                ; 设定8086汇编程序使用Small model
                .8086                                ; 设定采用8086汇编指令集
;-----------------------------------------------------------
;        符号定义                                               |
;-----------------------------------------------------------
;
; 8255芯片端口地址 （Port number）分配:
PortA        EQU                90H                        ; Port A's port number in I/O space
PortB        EQU         92H                        ; Port B's port number in I/O space
PortC        EQU         94H                        ; Port C's port number in I/O space
CtrlPT        EQU         96H                        ; 8255 Control Register's port number in I/O space
;
Patch_Protues        EQU                IN AL, 0        ;        Simulation Patch for Proteus, please ignore this line


;-----------------------------------------------------------
;        定义数据段                                             |
;-----------------------------------------------------------
                .data                                        ; 定义数据段;

DelayShort        dw        4000                           ; 短延时参量        
DelayLong        dw        40000                        ; 长延时参量



; SEGTAB是显示字符0-F，其中有部分数据的段码有错误，请自行修正
SEGTAB          DB 3FH        ; 7-Segment Tube, 共阴极类型的7段数码管示意图
                DB 06H        ;
                DB 5BH        ;            a a a
                DB 4FH        ;         f         b
                DB 66H        ;         f         b
                DB 6DH        ;         f         b
                DB 7DH        ;            g g g 
                DB 07H        ;         e         c
                DB 7FH        ;         e         c
                DB 6FH        ;         e         c
                DB 77H        ;            d d d     h h h
                DB 7CH        ; ----------------------------------
                DB 39H        ;       b7 b6 b5 b4 b3 b2 b1 b0
                DB 5EH        ;       DP  g  f  e  d  c  b  a
                DB 79H        ;
                DB 71H        ;


;-----------------------------------------------------------
;        定义代码段                                             |
;-----------------------------------------------------------
                .code                                                ; Code segment definition
                .startup                                        ; 定义汇编程序执行入口点
;------------------------------------------------------------------------
                Patch_Protues                                ; Simulation Patch for Proteus,
                                                                        ; Please ignore the above code line.
;------------------------------------------------------------------------


; Init 8255 in Mode 0
; PortA Output, PortB Output
;
Init8255:
                MOV AL, 10001001B
                OUT CtrlPT,AL        ;
MainLoop:
                ; 1. 读取PC口所有开关状态
                IN AL, PortC                        ; AL = PC7-PC0
                
                ; 2. 保存原始值
                MOV BL, AL                                ; BL备份完整开关状态
                
                ; ============================================
                ; 处理LED显示 (PC4-PC7 → PA4-PA7)
                ; ============================================
                ; 提取PC4-PC7（BL的高4位）
                MOV CL, 4
                SHR BL, CL                                ; 将高4位移到低4位
                ; 现在BL低4位 = 原来的PC4-PC7
                
                ; 将LED数据移到高4位位置（PA4-PA7）
                SHL BL, CL                                ; BL高4位 = PC4-PC7，低4位 = 0
                
                ; ============================================
                ; 设置位选信号 (PA0=0, PA1=1, PA2=1, PA3=1)
                ; ============================================
                MOV AL, 0EH                                ; AL = 0000 1110 (PA0=0选通, PA1-PA3=1关闭)
                ; 注意：0EH = 00001110，低4位中PA0=0，PA1=1，PA2=1，PA3=1
                
                ; ============================================
                ; 合并LED数据到位选信号
                ; ============================================
                OR AL, BL                                ; AL高4位=LED状态，低4位=位选信号
                OUT PortA, AL                        ; 输出到PA口
                
                ; ============================================
                ; 处理数码管显示 (PC0-PC3 → 查表 → PB口)
                ; ============================================
                ; 重新读取PC口（因为AL被覆盖了）
                IN AL, PortC
                
                ; 提取低4位(PC0-PC3)
                AND AL, 0FH                                ; AL = 0-15
                
                ; 查段码表
                MOV BX, OFFSET SEGTAB
                XLAT                                        ; AL = SEGTAB[AL]
                
                ; 输出段码到PB口
                OUT PortB, AL
                
                ; 短暂延时，保持显示稳定
                CALL DELAY
                
                ; 无限循环
                JMP MainLoop
;--------------------------------------------
;                                           |
; Delay system running for a while          |
; CX : contains time para.                  |
;                                           |
;--------------------------------------------

DELAY1         PROC
            PUSH CX
            MOV CX,DelayLong        ;
D0:         LOOP D0
            POP CX
            RET
DELAY1         ENDP


;--------------------------------------------
;                                           |
; Delay system running for a while          |
;                                           |
;--------------------------------------------

DELAY         PROC
            PUSH CX
            MOV CX,DelayShort
D1:         LOOP D1
            POP CX
            RET
DELAY         ENDP


;-----------------------------------------------------------
;        定义堆栈段                                             |
;-----------------------------------------------------------
                .stack 100h                                ; 定义256字节容量的堆栈


                END                                                ;指示汇编程序结束编译
```

```SQL
L1:
IN AL, PortC      ; 从 PortC 读入数据
MOV DH, AL        ; 保存到 DH

AND AL, 0FH       ; 保留低4位
MOV BX, OFFSET SEGTAB ; 指向段码表
XLAT              ; AL = SEGTAB[低4位]
MOV DL, AL        ; 保存低4位的段码到 DL 

MOV AL, DH
AND AL, 0F0H      ; 保留高4位
MOV CL, 4
SHR AL, CL        ; 右移4位，变成 0~15
MOV BX, OFFSET SEGTAB
XLAT              ; 高4位的段码
MOV CH, AL        ; 保存高4位的段码到 CH

MOV AL, DH
AND AL, 0F0H       ;低四位变成0
OR AL, 00000101B   ; Port A P0~P3用于控制七段码的显示端口，P4~P7控制led灯
OUT PortA, AL

MOV AL, DL         ; 输出低4位的段码
OUT PortB, AL
CALL DELAY         ; 延时，保持显示


MOV AL,DH
AND AL,0F0H
OR AL,00001010B
OUT PortA,AL
                
MOV AL,CH
OUT PortB,AL
CALL DELAY

JMP L1

RET
```

