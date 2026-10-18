# ПР №4

##1

```asm
.data
msg: .asciz "hello risc-v\n"

.text
main:
    la a0, msg
    li a7, 4
    ecall

    li a0, 42
    li a7, 1
    ecall

    li a7, 10
    ecall
```

**вивід консолі:**

```
hello risc-v
42
```

##2

```asm
.data
arr: .word 20, -5, 13, 6

.text
main:
    la t0, arr
    li t1, 4
    lw t2, 0(t0)
    li t3, 1

loop:
    bge t3, t1, done
    slli t4, t3, 2
    add t4, t0, t4
    lw t5, 0(t4)
    blt t2, t5, update
    j next
update:
    mv t2, t5
next:
    addi t3, t3, 1
    j loop

done:
    mv a0, t2
    li a7, 1
    ecall
    li a7, 10
    ecall
```

**вивід консолі:**

```
20
```

## 3

```asm
.data
arr: .word 20, -5, 13, 6

.text
main:
    la a0, arr
    li a1, 4
    jal ra, find_max
    li a7, 1
    ecall
    li a7, 10
    ecall

find_max:
    addi sp, sp, -8
    sw s0, 0(sp)
    sw s1, 4(sp)

    lw s0, 0(a0)
    li s1, 1
fm_loop:
    bge s1, a1, fm_done
    slli t0, s1, 2
    add t0, a0, t0
    lw t1, 0(t0)
    blt s0, t1, fm_upd
    j fm_next
fm_upd:
    mv s0, t1
fm_next:
    addi s1, s1, 1
    j fm_loop
fm_done:
    mv a0, s0

    lw s0, 0(sp)
    lw s1, 4(sp)
    addi sp, sp, 8
    ret
```

**вивід консолі:**

```
20
```

##4

```asm
.data
arr: .word 20, -5, 13, 6
txt: .asciz "min: "
nl:  .asciz "\n"

.text
main:
    la a0, arr
    li a1, 4
    jal ra, find_min
    mv s2, a0

    la a0, txt
    li a7, 4
    ecall
    mv a0, s2
    li a7, 1
    ecall
    la a0, nl
    li a7, 4
    ecall

    li a7, 10
    ecall

find_min:
    addi sp, sp, -8
    sw s0, 0(sp)
    sw s1, 4(sp)

    lw s0, 0(a0)
    li s1, 1
fn_loop:
    bge s1, a1, fn_done
    slli t0, s1, 2
    add t0, a0, t0
    lw t1, 0(t0)
    blt t1, s0, fn_upd
    j fn_next
fn_upd:
    mv s0, t1
fn_next:
    addi s1, s1, 1
    j fn_loop
fn_done:
    mv a0, s0

    lw s0, 0(sp)
    lw s1, 4(sp)
    addi sp, sp, 8
    ret
```

**вивід консолі:**

```
min: -5
```

**ручний розрахунок:**

масив: 20, -5, 13, 6

| i | arr[i] | arr[i] < мінімум? | мінімум |
|---|--------|-------------------|---------|
| 0 | 20 | початкове значення | 20 |
| 1 | -5 | так | -5 |
| 2 | 13 | ні | -5 |
| 3 | 6 | ні | -5 |

результат -5 збігається з виводом програми

##5

точка зупинки на першій інструкції підпрограми `addi sp, sp, -8`, далі step
значення розраховані за кодом, адреса масиву в rars 0x10010000,
три послідовні кроки в другій ітерації циклу (i = 1), робочий регістр t1:

| крок | інструкція | a0 | a1 | робочий регістр |
|------|------------|----|----|-----------------|
| 1 | `slli t0, s1, 2` | 0x10010000 | 4 | t0 = 4 |
| 2 | `add t0, a0, t0` | 0x10010000 | 4 | t0 = 0x10010004 |
| 3 | `lw t1, 0(t0)` | 0x10010000 | 4 | t1 = -5 |