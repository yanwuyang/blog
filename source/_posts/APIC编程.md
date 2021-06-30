---
title: APIC编程
date: 2021-03-10 9:30:21
comments: true 
categories: 操作系统
toc: false
---

APIC（高级可编程中断控制器）
<!--more-->

### smp_boot.asm 
```asm

[org 0x7c00]
ICR_LOW         equ     0x0FEE00300
SVR     	equ     0x0FEE000F0
LVT1		equ	0x0FEE00350
LVT2		equ	0x0FEE00360
APIC_ENABLED	equ	0x000000100
APIC_ID	        equ     0x0FEE00020
COUNT           equ     0x0

start:
   mov al,0x80
   out 0x70,al

   in al,0x21
   push ax
   mov al,0xff
   out 0x21,al
   in al,0xa1
   push ax
   mov al,0xff
   out 0xa1,al           ;屏蔽PIC

   call setup_gdt

   mov eax,cr0
   or eax,0x1
   mov cr0,eax
  
   jmp code_cs:pm

setup_gdt:
  lgdt [gdt_descr]
  ret

gdt:
gdt_null:
  dd 0x0
  dd 0x0

gdt_code:
    dw 0xffff                   ;limit(bits 0-15)
    dw 0x0                      ;基址(bits 0-15)
    db 0x0                      ;基址(bits 16-23)
    db 10011010b                ;1st flags,type flags
    db 11001111b                ;2nd flags,Limit(bits 16-19)
    db 0x0                      ;基址(bits 24-31)

gdt_data:
    dw 0xffff
    dw 0x0
    db 0x0
    db 10010010b
    db 11001111b
    db 0x0

gdt_end:
gdt_descr:
    dw gdt_end-gdt-1
    dd gdt

code_cs equ gdt_code-gdt
data_ds equ gdt_data-gdt

ap_init:
   jmp $


[bits 32]
pm:  
   mov ax,data_ds
   mov ds,ax
   mov ss,ax
   mov es,ax
   mov fs,ax
   mov gs,ax

   call initLocalAPIC
   jmp $


initLocalAPIC:
   mov esi,APIC_ID
   mov eax,[esi]
   and eax,0xff000000
   mov [BOOT_ID],eax

   ;通过设置一个8bit的APIC虚拟向量寄存器开启本地APIC
   mov esi,SVR
   mov eax,[esi]                   ;read SVR(spurious vector register)
   and eax,0xFFFFFF0F              ;清空虚拟向量寄存器  
   or eax,APIC_ENABLED             ;bit 8=1
   mov [esi],eax                   ;write SVR

   ;将LVT1编程为ExtInt ,他将信号传递给所有处理的INTR信号
   mov esi,LVT1
   mov eax,[esi]
   and eax,0xFFFE00FF
   or  eax,0x00005700          ;ExtInt
   mov [esi],eax

   ;将LVT2编程为NMI
   mov esi,LVT2
   mov eax,[esi]
   and eax,0xFFFE00FF
   or  eax,0x00005400           ;NMI
   mov [esi],eax

   ;广播一个INIT-SIPI-SIPI IPI序列，来唤醒AP进行初始化
   ;加载中断控制寄存器地址到esi    
   mov esi,ICR_LOW
   mov eax,0xc4500               ;广播 init ipi到所有AP
   mov [esi],eax
   dw 0x00eb,0x00eb              ;等待10毫秒
   
   ;mov edx,ap_init   
   mov eax,0x00c4600               ;广播SIPI API到所有AP  00 4K字节页中的AP启动代码基地址转换为8bit的向量
   ;add eax,edx
   mov [esi],eax
   mov [esi],eax

 
   ret

BOOT_ID dd 0x0
times 510-($-$$) db 0
dw 0xaa55


```

### 构建脚本

```
nasm smp_boot.asm  -f bin -o smp_boot.bin
```