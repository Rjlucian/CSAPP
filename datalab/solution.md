# Solution

## bitXor

```c
/* 
 * bitXor - x^y using only ~ and & 
 * Example: bitXor(4, 5) = 1
 * Legal ops: ~ &
 * Max ops: 14
 * Rating: 1
 */
int bitXor(int x, int y) {
    return ~((~(~x & y)) & (~(x & ~y)));
}
```
x ^ y == (~x & y) | (x & ~ y), 再利用De Morgan's law将其中的或用(~, &)表示.
De Morgan's law: ~(x | y) = (~x & ~y)

## tmin

```c
/* 
 * tmin - return minimum two's complement integer 
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 4
 *   Rating: 1
 */
int tmin(void) {
    return 1 << 31;
}
```

TMin: 0X70000000 = 0100 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000

## isTmax

```c
/*
 * isTmax - returns 1 if x is the maximum, two's complement number,
 *     and 0 otherwise 
 *   Legal ops: ! ~ & ^ | +
 *   Max ops: 10
 *   Rating: 1
 */
int isTmax(int x) {
    return !(((x + x) | 0x1) + 1) & !!(x + 1);
}
```

以8位补码为例子, tmax = 01111111, tmax << 1 = 11111110, (tmax  << 1) | 0x1 = 11111111 = -1 = tmin, 而tmin + 1 = 0. 因此这一系列等式给出了验证tmax的方法. 
同时需要注意(tmin << 1) | 0x1 = tmin, 因此我们还需要排除tmin的干扰.
最后, 由于不允许使用移位, 将 x << 1操作变为x + x即可

## allOddBits

```c
/* 
 * allOddBits - return 1 if all odd-numbered bits in word set to 1
 *   where bits are numbered from 0 (least  significant) to 31 (most  significant)
 *   Examples allOddBits(0xFFFFFFFD) = 0, allOddBits(0xAAAAAAAA) = 1
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 2
 */
177 int allOddBits(int x) {
178   int mask = 0xAA + (0xAA << 8) + (0xAA << 16) + (0xAA << 24);
179   return !((x & mask) ^ mask);
180 }

```

以8位补码为例子, 符合要求的的数其位表示为1_1_1_1_(`_`表示可以1也可以为0).
`1_1_1_1_ & 101010 = 10101010` => `(1_1_1_1_ & 10101010) & 10101010 = 0`
再将8位扩展到32位即可.

## negate

```c
/* 
 * negate - return -x 
 *   Example: negate(1) = -1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 5
 *   Rating: 2
 */
int negate(int x) {
    return ~x + 1;
}
```

以8位补码系统为例, x为一个补码数的二进制位表示, 则有x + (~x) = 1111111~2~  = -1, 因此x + ((~x) + 1) = 0 => -x = ~x + 1

## isAsciiDigit

```c
/* 
 * isAsciiDigit - return 1 if 0x30 <= x <= 0x39 (ASCII codes for charact    '0' to '9')
 *   Example: isAsciiDigit(0x35) = 1.
 *            isAsciiDigit(0x3a) = 0.
 *            isAsciiDigit(0x05) = 0.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 15
 *   Rating: 3
 */
int isAsciiDigit(int x) {
  int v0 = 0x30;
  int v9 = 0x39;
  int dif1 = x + (~v0) + 1;
  int dif2 = v9 + (~x) + 1;
  return !((dif1 >> 31) | (dif2 >> 31));
}
```

通过man ascii命令查看ascii表, 发现从0x30-0x39分别是0-9的编码值. 我们将数轴分为三个部分. 

1. x < 0x30
2. 0x30 <= x <= 0x39
3. 0x39 < x

不难发现, 仅仅当x属于第二种情况时, x - 0x30 >= 0 且 0x39 - x >= 0, 提取符号位即可.
注意: 减法通过取反加一做加法实现, 返回值通过De morgan's law来减少了一个运算符的使用

## conditional

```c
/* 
 * conditional - same as x ? y : z 
 *   Example: conditional(2,4,5) = 4
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 16
 *   Rating: 3
 */
int conditional(int x, int y, int z) {
  int bit2 = !x;
  int bit1 = !bit2;
  int mask1 = (~bit1) + 1;
  int mask2 = (~bit2) + 1;
  return (y & mask1) | (z & mask2);
}
```

以八位补码系统为例子.
前提: x为八位补码系统下的任意数, x | 00000000~2~ = x, x & 11111111~2~ = x, x & 00000000~2~ = 0.
首先, 获得两个掩码, 分两种情况: 

1. 当x == 0, mask1 = 00000000, mask2 = 11111111
2. 当 x != 0, mask1 = 11111111, mask2 = 00000000

9到12行完成了此过程, 利用了x = ~x + 1, -1 = 11111111~2~等技巧
最后, 若x为0, y不受影响, z被置为0, y | z = y, x != 0时同理

## isLessOrEqual

```c
/* 
 * isLessOrEqual - if x <= y  then return 1, else return 0 
 *   Example: isLessOrEqual(4,5) = 1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 24
 *   Rating: 3
 */
int isLessOrEqual(int x, int y) {
  int xSign = !!(x >> 31);
  int ySign = !!(y >> 31);
  int differ = y + (~x) + 1;
  return !(!xSign & ySign) & ((xSign & !ySign) | ((differ >> 31) + 1));
}
```

共有四种情况: 

1. x < 0,  y >= 0
2. x < 0, y < 0
3. x >= 0, y >= 0
4. x >= 0, y < 0

* 对于情况4, x <= y肯定不成立, 因此我们需要判断x,y不符合这种情况.
  即答案中的!(!xSign & ySign)

* 对于情况1, x <= y肯定成立

* 对于情况2, 3, 我们我们需要可以确定除了x和y都等于Tmin时, 不会发生溢出.
  考虑y - x, 

  * 如果y >= x, 则y - x >= 0, 记differ = y - x, 则(differ >> 31) + 1 = 1
  * 如果y < x, 则y - x < 0, 记differ = y - x, 则(differ >> 31) + 1 = 0

  因此我们得到了检验x <= y的方法

## logicalNeg

```c
/* 
 * logicalNeg - implement the ! operator, using all of 
 *              the legal operators except !
 *   Examples: logicalNeg(3) = 0, logicalNeg(0) = 1
 *   Legal ops: ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 4 
 */
254 int logicalNeg(int x) {
255   int negX = ~x + 1;
256   int sign = (negX | x) >> 31;
257   return sign + 1;
258 }
```

以8位补码系统为例子, x位任意8位补码数字.

1. x = 0: x = 0, 则 -x = 0, 首位都为0
2. x = Tmin, 则 -x = Tmin, 首位都为1
3. 其他: x首位为0且-x首位为1 或者 x首位为1且-x首位为0

记x首位为a, -x首位为b, 对于情况1, a | b = 0, 对于情况2,3 a | b = 1.
作x | -x, 将此值右移7位得到 00000000~2~ = 0 或 11111111~2~ = -1, 此值加一, 得到0(情况2, 情况3)或1(情况1)

## howManyBits

```c
/* howManyBits - return the minimum number of bits required to represent x in
 *             two's complement
 *  Examples: howManyBits(12) = 5
 *            howManyBits(298) = 10
 *            howManyBits(-5) = 4
 *            howManyBits(0)  = 1
 *            howManyBits(-1) = 1
 *            howManyBits(0x80000000) = 32
 *  Legal ops: ! ~ & ^ | + << >>
 *  Max ops: 90
 *  Rating: 4
 */
int howManyBits(int x) {
  int sign = x >> 31;
  int bit_16, bit_8, bit_4, bit_2, bit_1, bit_0;
  int ret;
  x = (x & (~sign)) | ((~x) & sign);

  bit_16 = (!!(x >> 16)) << 4;
  x = x >> bit_16;

  bit_8 = (!!(x >> 8)) << 3;
  x = x >> bit_8;

  bit_4 = (!!(x >> 4)) << 2;
  x = x >> bit_4;

  bit_2 = (!!(x >> 2)) << 1;
  x = x >> bit_2;

  bit_1 = !!(x >> 1);
  x = x >> bit_1;

  bit_0 = x;
  ret = bit_16 + bit_8 + bit_4 + bit_2 + bit_1 + bit_0 + 1;
  return ret;
}
```

前提: x为补码系统的一个负数, 则需要表示x的位数等于需要表示~x的位数
```
0000 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000 | 0000
0000 | 0000 | 0000 | 0000 | 0010 | 0000 | 0000 | 0000
```

采用二分法的思想,对于2w位二进制向量, 若前w位含1, 则舍去后W位部分, 累加器加上w; 前W位不含1, 则舍去前半部分, 处理后半部分, 累加器不变.循环此过程直到w = 0; 最后加上一位符号位即可

* 15行与17行完成了当x为负数时将其转变为~x

