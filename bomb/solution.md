# Bomb lab

## frequently used functions

string_not_equal: 比较rdi和rsi处的字符串(i.e., rdi和rsi代表的是地址), 结果为:

* 两字符串不相同: 返回1
* 两字符串相同: 返回0

---

read_six_numbers:从标准输入读入6个数字, 第i个数字位于%rsp + 4i

## phase_1

```assembly
  <main>:...
  400e32:       e8 67 06 00 00          call   40149e <read_line>
  400e37:       48 89 c7                mov    %rax,%rdi
  400e3a:       e8 a1 00 00 00          call   400ee0 <phase_1>
  ...
  <phase_1>
  ...
  400ee4:       be 00 24 40 00          mov    $0x402400,%esi
  400ee9:       e8 4a 04 00 00          call   401338 <strings_not_equal>
  400eee:       85 c0                   test   %eax,%eax
  400ef0:       74 05                   je     400ef7 <phase_1+0x17>
  400ef2:       e8 43 05 00 00          call   40143a <explode_bomb>
```

不难看出, main函数先从stdin读入一个字符串, 将其地址放到%rdi中, 随后调用phase_1, 将0x402400作为$rsi的值, 随后调用string_not_euqual对两个字符串进行比较. 如果比较结果为0, 即两字符串相同, 我们就执行跳转, 否则炸弹爆炸. 因此使用gdb查看0x402400处的字符串, 结果如下:
```
(gdb) print (char*) 0x402400
$1 = 0x402400 "Border relations with Canada have never been better."
```

因此phase_1应输入`Border relations with Canada have never been better.`

## phase_2

```assembly
0000000000400efc <phase_2>:
  // %rbp, %rbx: callee saved register
  400efc:       55                      push   %rbp
  400efd:       53                      push   %rbx
  // allocate some space in stack
  400efe:       48 83 ec 28             sub    $0x28,%rsp
  400f02:       48 89 e6                mov    %rsp,%rsi
  // read 6 numbers, the ith number locate at (%rsp + 4 * i)
  400f05:       e8 52 05 00 00          call   40145c <read_six_numbers>
  // compare first number with 1; if not euqual, bomb!
  400f0a:       83 3c 24 01             cmpl   $0x1,(%rsp)
  400f0e:       74 20                   je     400f30 <phase_2+0x34>
  400f10:       e8 25 05 00 00          call   40143a <explode_bomb>
  400f15:       eb 19                   jmp    400f30 <phase_2+0x34>
  
  400f17:       8b 43 fc                mov    -0x4(%rbx),%eax
  400f1a:       01 c0                   add    %eax,%eax
  400f1c:       39 03                   cmp    %eax,(%rbx)
  400f1e:       74 05                   je     400f25 <phase_2+0x29>
  400f20:       e8 15 05 00 00          call   40143a <explode_bomb>
  400f25:       48 83 c3 04             add    $0x4,%rbx
  400f29:       48 39 eb                cmp    %rbp,%rbx
  400f2c:       75 e9                   jne    400f17 <phase_2+0x1b>
  400f2e:       eb 0c                   jmp    400f3c <phase_2+0x40>
  
  400f30:       48 8d 5c 24 04          lea    0x4(%rsp),%rbx
  400f35:       48 8d 6c 24 18          lea    0x18(%rsp),%rbp
  400f3a:       eb db                   jmp    400f17 <phase_2+0x1b>
  
  400f3c:       48 83 c4 28             add    $0x28,%rsp
  400f40:       5b                      pop    %rbx
  400f41:       5d                      pop    %rbp
  400f42:       c3                      ret

```

首先, 调用read_six_numbers读入了6个数; 0x400f0a对第一个数进行检验, 如果不是1, 则炸弹爆炸, 因此我们得知第一个数必须是1; 随后跳到0x400f30, 我们注意到此时将%rbx设为第二个数所在的地址, %rbp为最后一个数的后边一个位置, 猜想%rbp是起一个标志的作用, 从而对6个数进行某种遍历. 随后跳转到 0x400f17, 我们看到将%rbx所在位置的前一个数的两倍与%rbx所在位置的数相比较, 若不相等, 则炸弹爆炸, 随后%rbx移动到下一个数的位置; 当%rbx与%rbp相等时, 释放栈空间. 因此phase_2对于输入的要求是:

* 总共有6个数
* 第一个数为1
* 从第二个数开始, 每个数都是前一个数的两倍

综上所述, phase_2的答案是`1 2 4 8 16 32`

## phase_3

```assembly
0000000000400f43 <phase_3>:
  400f43:       48 83 ec 18             sub    $0x18,%rsp
  // read two integers; if less, bomb! we refer them as a(first) and b(second).
  400f47:       48 8d 4c 24 0c          lea    0xc(%rsp),%rcx
  400f4c:       48 8d 54 24 08          lea    0x8(%rsp),%rdx
  400f51:       be cf 25 40 00          mov    $0x4025cf,%esi
  400f56:       b8 00 00 00 00          mov    $0x0,%eax
  400f5b:       e8 90 fc ff ff          call   400bf0 <__isoc99_sscanf@plt>
  400f60:       83 f8 01                cmp    $0x1,%eax
  400f63:       7f 05                   jg     400f6a <phase_3+0x27>
  400f65:       e8 d0 04 00 00          call   40143a <explode_bomb>

  // compare a with 0x7, if a > 7, jump, them bomb!
  400f6a:       83 7c 24 08 07          cmpl   $0x7,0x8(%rsp)
  400f6f:       77 3c                   ja     400fad <phase_3+0x6a>

  400f71:       8b 44 24 08             mov    0x8(%rsp),%eax
  400f75:       ff 24 c5 70 24 40 00    jmp    *0x402470(,%rax,8)

  400f7c:       b8 cf 00 00 00          mov    $0xcf,%eax
  400f81:       eb 3b                   jmp    400fbe <phase_3+0x7b>

  400f83:       b8 c3 02 00 00          mov    $0x2c3,%eax
  400f88:       eb 34                   jmp    400fbe <phase_3+0x7b>

  400f8a:       b8 00 01 00 00          mov    $0x100,%eax
  400f8f:       eb 2d                   jmp    400fbe <phase_3+0x7b>

  400f91:       b8 85 01 00 00          mov    $0x185,%eax
  400f96:       eb 26                   jmp    400fbe <phase_3+0x7b>

  400f98:       b8 ce 00 00 00          mov    $0xce,%eax
  400f9d:       eb 1f                   jmp    400fbe <phase_3+0x7b>

  400f9f:       b8 aa 02 00 00          mov    $0x2aa,%eax
  400fa4:       eb 18                   jmp    400fbe <phase_3+0x7b>

  400fa6:       b8 47 01 00 00          mov    $0x147,%eax
  400fab:       eb 11                   jmp    400fbe <phase_3+0x7b>

  400fad:       e8 88 04 00 00          call   40143a <explode_bomb>

  400fb2:       b8 00 00 00 00          mov    $0x0,%eax
  400fb7:       eb 05                   jmp    400fbe <phase_3+0x7b>

  400fb9:       b8 37 01 00 00          mov    $0x137,%eax

  400fbe:       3b 44 24 0c             cmp    0xc(%rsp),%eax
  400fc2:       74 05                   je     400fc9 <phase_3+0x86>
  400fc4:       e8 71 04 00 00          call   40143a <explode_bomb>
  400fc9:       48 83 c4 18             add    $0x18,%rsp
  400fcd:       c3                      ret
```

首先查看0x4025cf处的内容:
```
(gdb) print (char*) 0x4025cf
$1 = 0x4025cf "%d %d"
```

得知,我们需要输入两个整数, 再根据%rdx和%rcx的内容, 我们知道第一个整数存储在%(rsp + 0x8), 第二个整数存储在(%rsp + 0xc), 记第一个为a, 第二个为b
查看400f6a处, 如果a > 7, 则炸弹爆炸, 因此a < 7.
随后我们看到将a读入%eax, 这表明高位被置0, 也就是说高位不重要, 因此可以进一步缩小a的范围为[0, 7].
来到400f71, 这里做了一个简介跳转, 跳转到内存上地址为0x402470 + 8 * a处所表示的地址, 因此先查看0x402470处的内容:

``` 
(gdb) x/12gx 0x402470
0x402470:	0x0000000000400f7c	0x0000000000400fb9
0x402480:	0x0000000000400f83	0x0000000000400f8a
0x402490:	0x0000000000400f91	0x0000000000400f98
0x4024a0:	0x0000000000400f9f	0x0000000000400fa6
0x4024b0 <array.3449>:	0x737265697564616d	0x6c796276746f666e
0x4024c0:	0x7420756f79206f53	0x756f79206b6e6968
```

跳转目的不应该超过phase_3代码的地址范围, 因此我们主要观察0x402470-0x4024a0间的内容.
我们看到, 这个范围内的8个目的地址都是phase_3代码中mov指令的地址, 作用是将一个直接数送入%eax, 且下一行代码都是跳转到同一个位置, 将b与%eax中的数相比较. 因此有八种答案:

* 若a = 0, 则跳转地址为*(402470 + 8 * 0), 即0x400f7c, 则b = 0xcf
* 若a = 1, 则跳转地址位*(402470 + 8 * 1), 即0x400fb9, 则b = 0x2c3
* 以此类推

## phase_4



