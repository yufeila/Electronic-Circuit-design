
## 优点：

能够从给定的采样中求出某一特定频率信号的幅值，每次只计算一个频点，算法速度较快，在点数较少的采样序列上也能获得精确的结果。


## 核心思想

**Goertzel算法的核心思想就是**：

把离散信号 $x[n]$与一个专门为目标频率 $k$ 设计的卷积核（冲激响应）$h_k(n)$ 做卷积，   得到的输出在 $n = N-1$ 或 $n = N$ 时的值，就是信号在该频率上的DFT（FFT）分量。

DFT的定义是：

$$
X(k) = \sum^{N-1}_{n = 0} x(n)e^{-j \frac{2\pi}{N}nk}
$$
定义
$$
W_{N}(kn) = e^{-j \frac{2\pi}{N}kn}
$$
$$
X(k) = \sum^{N-1}_{n = 0} x(n) W_{N}(kn)
$$
因为：
$$
W_{N}(-kN) = 1
$$
$$
W_{N}(kn) \times W_{N}(-kN) = W(-k(N-n))
$$
所以
$$
X(k) = \sum^{N-1}_{n = 0} x(n) W_{N}(-k(N-n))
$$
定义卷积核：
$$
h_{k}(n) = W_{N}(-k(N-n))
$$
则$X(k)$可以写做：
$$
X(k) = x(n) * h_{k}(n)|_{n = N}
$$







定义$s(n)$ 为：

$$
s[n] = \sum_{m = 0}^{n} W(-k(n-m))x[m]
$$
$s(n)$满足如下的递推关系

$$
s(n) = x(n) + 2 \cos(\omega_{k})s(n-1) - s(n-2)
$$
下面是$s(n)$递推关系的等价表示
$$
s(n) = x(n) + W_{N}^{k}s(n-1)
$$
$$
s(-1) = 0
$$
$$
s(-2) = 0
$$

最后一步:

$$
s[N] = s(N - 1) - W_{N}^{k}s(N-2)
$$




## 伪代码

```
输入: 
    x[0...N-1]      // 输入信号序列，长度N
    k               // 目标频率索引（0 ≤ k < N）
    N               // 信号长度

参数准备:
    omega = 2 * pi * k / N
    coeff = 2 * cos(omega)

初始化递推变量:
    s_prev = 0      // s[n-1]，前一步状态
    s_prev2 = 0     // s[n-2]，上上步状态

主递推循环:
    for n from 0 to N-1:
        s = x[n] + coeff * s_prev - s_prev2
        s_prev2 = s_prev
        s_prev = s

// 递推结束后，s_prev = s[N-1], s_prev2 = s[N-2]

// 复指数权
    real_part = s_prev - cos(omega) * s_prev2
    imag_part = sin(omega) * s_prev2
    // 或直接： Xk = s_prev - exp(-j*omega) * s_prev2

输出:
    // Xk即为第k个DFT分量的复数结果
    Xk_real = real_part
    Xk_imag = imag_part
    Xk_magnitude = sqrt(real_part^2 + imag_part^2)
    Xk_phase = atan2(imag_part, real_part)

```


## 应用

三角波的傅里叶频率

假设周期为 $T$，基频为 $f_0 = \frac{1}{T}$，幅值为 $A$（峰值），**以 $t=0$ 为奇对称中心，周期性、幅值范围 $[-A, A]$，则：**

$$
x(t) = \frac{8A}{\pi^2} \sum_{n=1,\,\text{奇数}}^\infty \frac{(-1)^{(n-1)/2}}{n^2} \sin\left(2\pi n f_0 t\right)
$$

- $n=1,3,5,\ldots$（仅奇次谐波）
    
- 每一项的幅度按 $1/n^2$ 衰减
    
- $(-1)^{(n-1)/2}$ 控制相位交替（正负交替）



$$
x(t) = \frac{8A}{\pi^2} \left[\sin(\omega_0 t) - \frac{1}{9} \sin(3\omega_0 t) + \frac{1}{25} \sin(5\omega_0 t) - \cdots \right]$$

其中 $\omega_0 = 2\pi f_0$。