#Loop Unrolling

```
Loop: lw   $t0, 0($s1)
      addu $t0, $t0, $s2
      sw   $t0, 0($s1)
      addi $s1, $s1, -4
      bne  $s1, $zero, Loop
```
The above code, if executes in multiple-issue MIPS pipeline processor.
We will find that the first 3 instructions have data dependences and the last 2 also.

the best scheduled code may look like this:
|  -   |  ALU or branch instruction | Data transfer instruction | Clock cycle |
|----|---------------------------------|----------------------------|---------------|
|Loop:|             |`lw   $t0, 0($s1)`| 1 |
|     | `addi $s1, $s1, -4` |            | 2 |
|     | `addu $t0, $t0, $s2`  |            | 3 |
|     | `bne  $s1, $zero, Loop` | `sw   $t0, 0($s1)` | 4|

we can calculate the CPI is 0.8, far away from the 0.5(the best result)

---

Now let consider the compiler how to do the improvement.

we can find that if we unroll the loop, there will be serval instruction set that are independent, (or just name dependence).  we can rename the register `$t1`, `$t2`,  `$t3` instead of `$t0` all the way.

we consider that loop unrolling 4 times as the origin code.
we will get 4 tables like the above. However, we can compact 4 tables to 1 with less blank, which results in a better performance.
|  -   |  ALU or branch instruction | Data transfer instruction | Clock cycle |
|----|---------------------------------|----------------------------|---------------|
|Loop:| `addi $s1, $s1, -16`            |`lw   $t0, 0($s1)`| 1 |
| | | `lw $t1, 12($s1)`| 2|
|     | `addu $t0, $t0, $s2`  |   `lw $t2, 8($s1)`         | 3 |
|     | `addu $t1, $t1, $s2`  |   `lw $t3, 4($s1)`         | 4 |
|     | `addu $t2, $t2, $s2`  |   `sw $t0, 16($s1)`         | 5 |
|     | `addu $t3, $t3, $s2`  |   `sw $t1, 12($s1)`         | 6 |
| |              | `sw $t1, *($s1)`| 7|
|     | `bne  $s1, $zero, Loop` | `sw   $t3, 4($s1)` | 8|


Through loop unrolling, the CPI become 8 / 14 = 0.57.  Loop unrolling and scheduling with dual issue gave us an improvement factor of almost 2. 
Points need to notice is that, we use 4 register instead of 1, and also the loop unrolling will increase the size of the source code.

##conclusion

I think it is a interesting point to explain the improvement achieved by the loop unrolling. 