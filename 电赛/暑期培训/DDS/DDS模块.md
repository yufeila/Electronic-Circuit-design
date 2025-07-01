

## DDS原理

![[Pasted image 20250630102738.png]]

![[Pasted image 20250630103053.png]]

低通滤波器相当于一个积分器。

如何按照时间依次输出ROM的波形？

查找表（通过matlab计算）将查找表存储在ROM里。

![[Pasted image 20250630103530.png]]

改进频率:(频率减小) 16 ms -> 8ms

![[Pasted image 20250630103949.png]]


改进频率:(频率增大)
采用移位（右移/2）或截位的方式

```
time_r = time >> 1
```

![[Pasted image 20250630104316.png]]

发现
```
time  = time + r
```
这个r是可调的，r就是频率控制字

![[Pasted image 20250630103139.png]]

### ROM的结构

几种不同的ROM:

- ROM(Read Only Memory)： 出厂时写好存储数据
- RROM(Programmable Read Only Memory)：可编程
- EPROM(Erasable Programmable Read-Only Memory)：可擦除+可编程（十几次）
- EEPROM(Electrically Erasable Programmable Read-Only Memory)：电可擦除+可编程（几千次及以上）

字线：地址译码器的输出端个数（行数）
位线：一个字的位数（列数）
存储容量: 字线×位线,（存储容量 = 存储单元的个数 = 能存储的2进制数的个数）

位宽：与位线对应(列数)，位宽 = 位线数
地址数(有时也称深度)：与字线对应(行数)，地址数 = 位线数 = 地址译码器输入线最大值
容量：存储容量（位宽 × 地址数 = 字线×位线）

### 查找表

![[Pasted image 20250630154452.png]]


### 相关计算

![[Pasted image 20250630154532.png]]

由

$$
F_{o}=\frac{M}{2^N}F_{i}
$$
可知：
$$
M = F_{o}\times \frac{2^N}{F_{i}}
$$

频率控制字 = (输出频率 × 2^32) / 参考时钟频率
## AD9851引脚图

![[Pasted image 20250630131014.png]]

- **LSB（最低有效位）**：数值最小，通常是第0位
    
- **MSB（最高有效位）**：数值最大，通常是最高位（比如8位字节的第7位，或32位字的第31位）

各引脚功能：

![[Pasted image 20250630154605.png]]


### 并行写入

控制字和时序

![[Pasted image 20250630154639.png]]

### 串行写入

从引脚图可以看出，串行数据输入端口是D7


时序和控制字
![[Pasted image 20250630154742.png]]