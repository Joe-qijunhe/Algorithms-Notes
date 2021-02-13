[toc]

# 打印字节表示

byte_pointer定义成一个指向类型为unsigned char的指针，这样的一个字节指针应用一个字节序列，每个字节都被认为是一个非负整数。指向x的指针&x，被强制类型转换成unsigned char *，让程序把这个指针看成一个指向一个字节序列，而不是指向一个原始数据类型的对象。然后，这个指针会被看成对象使用的最低字节地址。

>In "0x%2x" the 2 defines the "field width": if the hex representation of the value consists of less than 2 digits, it is prefixed with spaces to end up with a field width of 2.
>
>In "0x%.2x" the 2 defines the precision: there will be at least 2 hex digits in the result, if the representation of the value has less digits, it is prefixed with 0's

```c
#include <stdio.h>

typedef unsigned char * byte_pointer;

void show_bytes(byte_pointer start, size_t len)
{
    size_t i;
    for (i = 0; i< len; i++)
    {
        //整数必须要用至少两个数字的十六进制格式输出
        printf("%.2x", start[i]);
    }
    printf("\n");
}

void show_int(int x)
{
    show_bytes((byte_pointer) &x, sizeof(int));
}

void show_float(float x)
{
    show_bytes((byte_pointer) &x, sizeof(float));
}

void show_pointer(void *x)
{
    show_bytes((byte_pointer) &x, sizeof(void *));
}

int main()
{
    show_int(30);
}

```

# 判断溢出

判断无符号相加会不会产生溢出：

```c
int uadd_ok(unsigned x, unsigned y)
{
	unsigned sum = x + y;
	return sum >= x;
}
```

判断整数相加会不会产生溢出：

```c
int tadd_ok(int x, int y)
{
	int sum = x + y;
	int neg_over = x < 0 && y < 0 && sum >= 0;
	int pos_over = x >= 0 && y >= 0 && sum < 0;
	return !neg_over && !pos_over;
}
```

判断整数相乘会不会溢出：

对于数据类型int为32位的情况，使用64位精度的数据类型int64_t，而不使用除法

```c
int tmult_ok(int x, int y)
{
	int64_t pll = (int64_t)x * y;
	return pll == (int)pll;
}
```

注：如果写成int64_t pll = x * y;就会用32位值来计算乘积（可能溢出），然后再符号扩展到64位。而int64_t pll = (int64_t)x * y;会先强转成int64_t再计算。

# 除法

对于整数参数 x 返回 x/16 的值。不能用除法，模运算，乘法，条件语句，比较运算符和循环。假设数据类型int是32位长，使用补码表示，右移是算术右移。

```c
int div16(int x)
{
    int bias = (x >> 31) & 0xF;
    return (x + bias) >> 4;
}
```

x >> 31产生一个字，如果x是负数，这个字为全1，否则为全0。通过掩码屏蔽调适当的位，我们就能得到期望的偏置值