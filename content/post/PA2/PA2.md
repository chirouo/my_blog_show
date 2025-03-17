---

title: PA2

description: 一生一芯 nemu PA2。

date: 2025-02-20

# slug:

image: https://raw.githubusercontent.com/chirouo/my_blog_img/main/xdc1.jpg

categories:
  - ysyx
tags:
  - ysyx
---

## YEMU

### 端序和位域

位域

- [C语言结构体定义位域，从bit0开始，依次到最高bit位 - Risun_Lee - 博客园](https://www.cnblogs.com/risunlee/p/11507103.html)

- 样例

  ```c
  typedef union {
    struct { uint8_t rs : 2, rt : 2, op : 4; } rtype;
    struct { uint8_t addr : 4      , op : 4; } mtype;
    uint8_t inst;
  } inst_t;
  uint8_t M[NMEM] = {   // 内存, 其中包含一个计算z = x + y的程序
    0b11100110,  // load  6#     | R[0] <- M[y]
    0b00000100,  // mov   r1, r0 | R[1] <- R[0]
    0b11100101,  // load  5#     | R[0] <- M[x]
    0b00010001,  // add   r0, r1 | R[0] <- R[0] + R[1]
    0b11110111,  // store 7#     | M[z] <- R[0]
    0b00010000,  // x = 16
    0b00100001,  // y = 33
    0b00000000,  // z = 0
  };
  inst_t this = M[0];
  //C语言中位域从低到高排列this.rtype.op != 0110, == 1110
  ```

端序（Endianness）：

- 端序是指**多字节**数据在内存中的存储顺序

- 小端：低字节存储在低地址（x86常用）

- 大端：高字节存储在低地址（网络字节序）

- 样例

  ```c
  #include <stdio.h>
  #include <stdint.h>
  
  // 位域结构体
  typedef union {
      struct {
          uint8_t a : 4;
          uint8_t b : 4;
      } bits;
      uint8_t byte;
  } BitField;
  
  // 检查端序的联合体
  typedef union {
      uint16_t value;
      uint8_t bytes[2];
  } EndianTest;
  
  int main() {
      // 测试位域排列
      BitField bf;
      bf.byte = 0x12;  // 0001 0010
      printf("位域测试:\n");
      printf("完整字节: 0x%02x\n", bf.byte);
      printf("低4位(a): 0x%x\n", bf.bits.a);
      printf("高4位(b): 0x%x\n", bf.bits.b);
  
      // 测试端序
      EndianTest et;
      et.value = 0x1234;
      printf("\n端序测试:\n");
      printf("完整数值: 0x%04x\n", et.value);
      printf("第一个字节: 0x%02x\n", et.bytes[0]);
      printf("第二个字节: 0x%02x\n", et.bytes[1]);
  
      return 0;
  }
  /*运行结果
  位域测试:
  完整字节: 0x12
  低4位(a): 0x2
  高4位(b): 0x1
  
  端序测试:
  完整数值: 0x1234
  第一个字节: 0x34
  第二个字节: 0x12
  */
  
  /*题外话
  结构体 EndianTest 中的bytes[2]在内存中的排列方式是：
  bytes[0] = 0x34（低字节）在低地址
  bytes[1] = 0x12（高字节）在高地址
  即	bytes[0]	bytes[1]
  		0x34		0x12	x86的小端存储
  */
  ```

总结：x86上端序是从低到高 小端序；c语言编译器决定位域，位域一般也是从低到高（先声明变量的在低位，后声明的在高位）

## 指令周期

```
取指 --> 译码（操作码译码和操作数译码）--> 执行
cmd_c() 
--> cpu_exec(uint64_t n)  
--> execute(uint64_t n) 
--> exec_once(Decode *s, vaddr_t pc) 
--> isa_exec_once(Decode *s)//进入 riscv_32
--> inst_fetch(vaddr_t *pc, int len)
    decode_exec(Decode *s)
--> #define INSTPAT_START(name) { const void * __instpat_end = &&concat(__instpat_end_, name);
--> INSTPAT("??????? ????? ????? ??? ????? 00101 11", auipc  , U, R(rd) = s->pc + imm);
--> pattern_decode(pattern, STRLEN(pattern), &key, &mask, &shift);
    INSTPAT_MATCH(s, ##__VA_ARGS__);
    	--> decode_operand(s, &rd, &src1, &src2, &imm, concat(TYPE_, type));
    		__VA_ARGS__ ;
--> #define INSTPAT_END(name)   concat(__instpat_end_, name): ; }
--> R(0) = 0;
```

