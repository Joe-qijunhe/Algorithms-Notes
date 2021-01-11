[toc]

# 打印字节表示

byte_pointer定义成一个指向类型为unsigned char的指针，这样的一个字节指针应用一个字节序列，每个字节都被认为是一个非负整数。指向x的指针&x，被强制类型转换成unsigned char *，让程序把这个指针看成一个指向一个字节序列，而不是指向一个原始数据类型的对象。然后，这个指针会被看成对象使用的最低字节地址。

>In "0x%2x" the 2 defines the "field width": if the hex representation of the value consists of less than 2 digits, it is prefixed with spaces to end up with a field width of 2.
>
>In "0x%.2x" the 2 defines the precision: there will be at least 2 hex digits in the result, if the representation of the value has less digits, it is prefixed with 0's

```c
#include <stdio.h>

typedef unsigned char * byte_pointer;

void show_bytes(byte_pointer start, size_t len){
    size_t i;
    for (i = 0; i< len; i++){
        //整数必须要用至少两个数字的十六进制格式输出
        printf("%.2x", start[i]);
    }
    printf("\n");
}

void show_int(int x){
    show_bytes((byte_pointer) &x, sizeof(int));
}

void show_float(float x){
    show_bytes((byte_pointer) &x, sizeof(float));
}

void show_pointer(void *x){
    show_bytes((byte_pointer) &x, sizeof(void *));
}

int main(){
    show_int(30);
}

```



