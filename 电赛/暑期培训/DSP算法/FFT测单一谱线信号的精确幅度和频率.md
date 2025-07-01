
## 背景

测量“正弦波”或**以单一频率为主**的信号的**频率与幅度**


## 数学原理

### 符号和定义说明
#### 采样率和采样点数

- **采样率 $f_s$**  
    单位：Hz（每秒采样多少次）  
    决定了**频率范围的上限**（最大能分析的频率就是奈奎斯特频率 $f_s/2$）
    
- **采样点数 $N$**  
    表示你**一共采集了多少个数据点**，即窗口长度  
    决定了**频率分辨率**（最小可分辨的频率间隔）


#### fft和fftshift

- **FFT输出的bin索引是 0 到 $N-1$；**
    
- **fftshift后索引是 $-N/2$ 到 $N/2-1$（偶数点时），中心即零频。**



#### Bin

在**FFT/DFT分析**中，**bin 指的是一个离散的频率区间**，就是你FFT输出的每一个结果点对应的那个频率


- 如果你对长度为 $N$ 的信号做FFT，那么FFT会输出 $N$ 个复数。
    
- 每一个输出点就是一个**bin**，它代表了**一个固定的离散频率**。
    
例如

$$
f_{\text{bin}} = \frac{k}{N}f_s, \quad k=0,1,...,N-1
$$

- 这里$f_s$​是采样率，$k$是bin的编号（通常叫做“bin号”或“频率索引”）。
    
- 比如 $k=5$ 的 bin，对应的频率就是 $\frac{5}{N}f_s$​。


### 频谱泄漏（Spectral Leakage）


假设信号是

$$
x(t) = A \cos (2 \pi f_{0}+\phi)
$$
对它进行如下采样：

$$
x[n] = A\cos\left(2\pi \frac{f_0}{f_s} n + \phi\right)
$$

采样点数为 $N$，采样率 $f_{s}$​ .
所以可以写作：

$$
x[n] = \frac{A}{2} e^{j \phi} e^{j\left(2\pi \frac{f_0}{f_s} n \right)} + \frac{A}{2} e^{-j\phi} e^{-j\left(2\pi \frac{f_0}{f_s} n\right)}
$$

而

$$
1 \xrightarrow{DTFT} 2\pi \sum^{+\infty}_{k = -\infty} \delta (\omega-2\pi k)
$$
$$
e^{j\left( 2\pi \frac{ f_{0}}{f_{s}} n \right)} \xrightarrow{DTFT} 2\pi \sum^{+\infty}_{k = -\infty} \delta \left( \omega-2\pi k - 2\pi\frac{ f_{0}}{f_{s}} \right)
$$
$$
e^{-j\left( 2\pi \frac{ f_{0}}{f_{s}} n \right)} \xrightarrow{DTFT} 2\pi \sum^{+\infty}_{k = -\infty} \delta \left( \omega + 2\pi k + 2\pi\frac{ f_{0}}{f_{s}} \right)
$$

则$x[n]$的傅里叶变换应该为：
$$
x[n] \xrightarrow{DTFT} X(e^{j \omega}) = \frac{A}{2} e^{j \phi}  2\pi \sum^{+\infty}_{k = -\infty} \delta \left( \omega-2\pi k - 2\pi\frac{ f_{0}}{f_{s}} \right) + \frac{A}{2} e^{-j\phi}  2\pi \sum^{+\infty}_{k = -\infty} \delta \left( \omega + 2\pi k + 2\pi\frac{ f_{0}}{f_{s}} \right)
$$

但如果我们只考虑一个周期$[-\pi, \pi)$ 的$X(e^{j \omega})$，且令$\omega_{0}= 2\pi\frac{ f_{0}}{f_{s}}$ ，则
$$
\tilde{X} (e^{j \omega}) = \frac{A}{2} e^{j \phi} 2\pi \delta \left( \omega - \omega_{0} \right) + \frac{A}{2} e^{-j\phi}  2\pi \delta \left( \omega + \omega_{0} \right)
$$

由于我们是对有限长度的信号进行采样，相当于在时域上与一个矩形窗函数相乘，在频域上与一个采样信号卷积。

![[Pasted image 20250629153648.png]]
![[Pasted image 20250629153704.png]]




选取$[0,N-1]$的矩形窗函数

有限窗   $\;w_N[n]=1\;(0≤n≤N-1)$ 的 DTFT 是

$$
W_N(e^{j\omega}) = e^{-j\omega\frac{N-1}{2}}\, \frac{\sin\!\bigl(N\omega/2\bigr)}{\sin\!\bigl(\omega/2\bigr)}
$$

把理想冲激 $\tilde X(e^{j\omega})$ 与 $W_N$ 卷积后得到

$$
X(e^{j\omega})= \frac{A\pi}{2}\,e^{j\phi}\,W_N\!\bigl(e^{j(\omega-\omega_0)}\bigr)\;+\; \frac{A\pi}{2}\,e^{-j\phi}\,W_N\!\bigl(e^{j(\omega+\omega_0)}\bigr)
$$

FFT 计算的是$e^{j k\omega_{0}n}$之前的系数(这里将n视为变量)，$\omega_{s}=\frac{2\pi}{N}$

$$
x[n] = \frac{1}{N}\sum^{+\infty}_{k=-\infty}a_{k}e^{j \frac{2\pi}{N}kn}
$$
FFT对$a_{k}$的定义是
$$
a_{k} = \sum^{N-1}_{n=0}x[n]e^{-jk \frac{2\pi}{N}n}
$$
而$a_{k}$与FFT的关联是:
$$
a_{k} = X(e^{j k\omega_{s}})
$$
故FFT计算得到的$a_{k}$为：

$$
a_{k} = \frac{A\pi}{2}\,e^{j\phi}\,W_N\!\bigl(e^{j(k\omega_{s}-\omega_0)}\bigr)\;+\; \frac{A\pi}{2}\,e^{-j\phi}\,W_N\!\bigl(e^{j(k\omega_{s}+\omega_0)}\bigr)
$$
将
$$
\omega_{0} = 2\pi \frac{f_{0}}{f_s}
$$带入
$$
a_{k} = \frac{A\pi}{2}\,e^{j\phi}\,W_N\!\bigl(e^{j2\pi(\frac{k}{N}-\frac{f_{0}}{f_{s}})}\bigr)\;+\; \frac{A\pi}{2}\,e^{-j\phi}\,W_N\!\bigl(e^{j 2\pi (\frac{k}{N}+ \frac{f_{0}}{f_{s}})}\bigr)
$$

将
$$
W_N(e^{j\omega}) = e^{-j\omega\frac{N-1}{2}}\, \frac{\sin\!\bigl(N\omega/2\bigr)}{\sin\!\bigl(\omega/2\bigr)}
$$带入


- **抽样点**
    
    $$\omega_k = \frac{2\pi k}{N},\qquad k=0,\dots,N\!-\!1$$
    
    令
    
    $$
    \Delta = \frac{k}{N}-\frac{f_0}{f_s},\qquad \frac{f_{0}}{f_{s}} = \frac{k_{0}}{N};(k_{0}不一定整数)
    $$
- 将 $W_{N}$​ 代入
    
    $$
    a_k=\frac{A}{2}e^{j\phi}\,e^{-j\pi (N-1)\Delta}\, \frac{\sin(\pi N\Delta)}{\sin(\pi\Delta)} +\frac{A}{2}e^{-j\phi}\,e^{-j\pi (N-1)(\tfrac{k}{N}+ \tfrac{f_0}{f_s})}\, \frac{\sin\!\bigl[\pi N\bigl(\tfrac{k}{N}+ \tfrac{f_0}{f_s}\bigr)\bigr]} {\sin\!\bigl[\pi\bigl(\tfrac{k}{N}+ \tfrac{f_0}{f_s}\bigr)\bigr]}
    $$​​
    
把它记成
$$
a_k=\frac{A}{2}e^{j\phi}\,D_N(k-k_0)+ \frac{A}{2}e^{-j\phi}\,D_N(k+k_0)

$$    
其中
    
$$D_N(r)=e^{-j\pi r (N-1)/N}\, \frac{\sin(\pi r)}{\sin(\pi r/N)}​
$$    
这就是 **Dirichlet 核**（FFT “sinc” 泄漏函数）的抽样形式。


#### $k_{0}$是整数

k0​ 为整数且 $k_0\le N/2$。看第一个 Dirichlet：

- 当 $k=k_0$ → $\Delta=0$
    
    $$\frac{\sin(\pi N\Delta)}{\sin(\pi\Delta)} =\frac{\sin 0}{\sin 0}\xrightarrow{\text{l’Hôpital}}N$$
    
    故
    $$
    a_{k_0}=\frac{A}{2}e^{j\phi}\,N$$
- 当 $k\neq k_0$​ → $\sin(\pi r)=0$（$r$ 为整数），分子为 0，  
    因而 $D_N=0$。
    
- 第二项 $D_N(k+k_0)$ 落在高于 $π$ 的区间，如果原信号带宽 $<f_s/2$，  
    通常计算正频半谱时直接舍掉，所以不影响结果。



#### $k_{0}$非整数(不对齐)

若 $k_0$ **不是整数**，则 $\Delta\neq 0$ 对所有 k。  

第一个 Dirichlet 的幅值

$$
\bigl|D_N(k-k_0)\bigr| =\left|\frac{\sin(\pi N\Delta)}{\sin(\pi\Delta)}\right|​​
​$$

就是熟悉的

$$
\underbrace{\frac{\sin(Nx)}{\sin x}}_{\text{sinc 形}} ,\qquad x=\pi\Delta
$$

- **主瓣宽 $\approx 2$ 个 bin**，高度 < NNN；
    
- 旁瓣 –13 dB 起，再 –6 dB/瓣衰减。
    
- 第二个 Dirichlet $D_N(k+k_0)$ 为负频镜像，对实信号同样存在但通常不画。
    
**于是你看到两条 sinc 包络而非冲激。**


#### 频谱泄漏现象

当信号频率**不落在 DFT/FFT 的整数 bin 上**，  
本应出现在某一 bin 的能量会“泼洒”到邻近频点，主瓣呈 sinc 形，旁瓣衰减，这就是**频谱泄漏**。


![[Pasted image 20250701102033.png]]

![[Pasted image 20250701102052.png]]


根本原因：

在采样定理中，我们无法在全时域上对信号进行冲激串采样，只能在有限的窗内对信号进行采样。







## 插值（interpolation）


### **问题设定**

设有 FFT 频谱中主峰点（最大幅值 bin）索引为 $m$，幅值为 $M_m$​；  
其相邻点（次大幅值 bin）索引为 $n$ ，幅值为 $M_n$​。  
假设真实峰值在 $m+\delta$ 之间（$0 < \delta < 1$）。

---

### **1. 两个等式（sinc插值关系）**

$$
\begin{cases} A \cdot \mathrm{sinc}(\delta) = M_m \\ A \cdot \mathrm{sinc}(1 - \delta) = M_n \\ \end{cases}$$

其中

- $A$ 为真实幅度
    
- $\mathrm{sinc}(x) = \dfrac{\sin(\pi x)}{\pi x}$​
    

---

### **2. 解出 $\delta$（即 $\text{dindex}$）**

将上面两式相除得：

$$
\Rightarrow \frac{M_n}{M_m} = \frac{\sin(\pi (1-\delta))}{(1-\delta)\pi} \cdot \frac{\delta\pi}{\sin(\pi \delta)} = \frac{\sin(\pi (1-\delta)) \cdot \delta}{\sin(\pi \delta) \cdot (1-\delta)}​
$$

数值上一般直接用**牛顿法**等迭代求 $\delta$。

---

### **3. 校正后的频率索引和幅度**

$\text{频率索引：}$ $$k_{\text{true}} = m + \delta \cdot (n - m)$$

- 若 $n = m+1$：即右侧点大，$\delta > 0$
    
- 若 $n = m-1$：即左侧点大，$\delta < 0$
    

$\text{幅度：}$ $A = \frac{M_m}{\mathrm{sinc}(\delta)}$​

---

### **4. 简洁归纳写法**

$$
\boxed{ \begin{aligned} \frac{M_n}{M_m} &= \frac{\mathrm{sinc}(1 - \delta)}{\mathrm{sinc}(\delta)} \\ A &= \frac{M_m}{\mathrm{sinc}(\delta)} \\ k_{\text{true}} &= m + (n - m) \cdot \delta \end{aligned} }​$$


## 代码实现

1. 定义结构体（或者是class）
	1. 需要幅度  and  相位
2. 定义全局查找最大值函数
	1. fft之后需要取靠近0点的那一个峰值（第一个最大值）
3. 定义查找特定bin两侧的赋值最大点(返回是一个dot类型的结构体实例)
4. 定义针对本情形的牛顿法(输入M_n/M_m和精度阈值)
5. 定义频谱分析函数（顶层函数），返回A和$k_{true}$

需要注意的是，FFT计算得到的$a_{k}$是这样的：
$$
a_{k} = \sum^{+\infty}_{n=-\infty}x[n]e^{-jk \frac{2\pi}{N}n}
$$
因此，得到实际的幅度，需要

```
X = fft(x, N)./N;
```

同时，由于一个频率的基波的频谱是偶函数，它的能量分散在$\pm \omega_{0}$两个频率上，因此需要最终幅值✖️2

