# самостійна робота №7

варіант 3 - обчислення факторіала ітеративно

## 1 функція і асемблер трьох цілей

код на C

```c
int factorial(int n) {
    int result = 1;
    for (int i = 2; i <= n; i++) {
        result *= i;
    }
    return result;
}
```

компіляція з прапорцем -O0 компіляторами GCC 13 для x86-64, ARM64 і RISC-V RV64GC, службові директиви прибрані

x86-64

```
factorial:
    push   rbp
    mov    rbp, rsp
    mov    DWORD PTR [rbp-20], edi
    mov    DWORD PTR [rbp-8], 1
    mov    DWORD PTR [rbp-4], 2
    jmp    .L2
.L3:
    mov    eax, DWORD PTR [rbp-8]
    imul   eax, DWORD PTR [rbp-4]
    mov    DWORD PTR [rbp-8], eax
    add    DWORD PTR [rbp-4], 1
.L2:
    mov    eax, DWORD PTR [rbp-4]
    cmp    eax, DWORD PTR [rbp-20]
    jle    .L3
    mov    eax, DWORD PTR [rbp-8]
    pop    rbp
    ret
```

ARM64

```
factorial:
    sub    sp, sp, #32
    str    w0, [sp, 12]
    mov    w0, 1
    str    w0, [sp, 24]
    mov    w0, 2
    str    w0, [sp, 28]
    b      .L2
.L3:
    ldr    w1, [sp, 24]
    ldr    w0, [sp, 28]
    mul    w0, w1, w0
    str    w0, [sp, 24]
    ldr    w0, [sp, 28]
    add    w0, w0, 1
    str    w0, [sp, 28]
.L2:
    ldr    w1, [sp, 28]
    ldr    w0, [sp, 12]
    cmp    w1, w0
    ble    .L3
    ldr    w0, [sp, 24]
    add    sp, sp, 32
    ret
```

RISC-V RV64GC

```
factorial:
    addi   sp,sp,-48
    sd     s0,40(sp)
    addi   s0,sp,48
    mv     a5,a0
    sw     a5,-36(s0)
    li     a5,1
    sw     a5,-24(s0)
    li     a5,2
    sw     a5,-20(s0)
    j      .L2
.L3:
    lw     a5,-24(s0)
    mv     a4,a5
    lw     a5,-20(s0)
    mulw   a5,a4,a5
    sw     a5,-24(s0)
    lw     a5,-20(s0)
    addiw  a5,a5,1
    sw     a5,-20(s0)
.L2:
    lw     a5,-20(s0)
    mv     a4,a5
    lw     a5,-36(s0)
    sext.w a4,a4
    sext.w a5,a5
    ble    a4,a5,.L3
    lw     a5,-24(s0)
    mv     a0,a5
    ld     s0,40(sp)
    addi   sp,sp,48
    ret
```

в RISC-V компілятор виводить повернення як jr ra, це те саме що ret

## 2 кількість інструкцій у тілі функції

| архітектура | кількість інструкцій |
|---|---|
| X86-64 | 16 |
| ARM64 | 21 |
| RISC-V | 29 |

рахувалися всі інструкції від мітки factorial до повернення, мітки не рахувались

## 3 регістри аргументів і результату

| архітектура | регістр аргументів | регістр результату |
|---|---|---|
| X86-64 | EDI, це молодші 32 біти RDI | EAX |
| ARM64 | W0, це молодші 32 біти X0 | W0 |
| RISC-V | A0 | A0 |

функція має один аргумент N, тому використовується лише перший регістр аргументів

## 4 висновок

для факторіала x86-64 виконав 16 інструкцій, ARM64 21, а RISC-V 29

x86-64 звертається до пам’яті прямо в арифметиці, бо imul і add беруть операнд із пам’яті і це по одній команді

ARM64 і RISC-V за принципом load/store мусять спершу завантажити значення в регістр командами ldr або lw, а потім записати результат назад командами str або sw, тому команд більше

у RISC-V додаються ще mv і sext.w для розширення знакових значень
