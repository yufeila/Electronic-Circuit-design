数字PI（比例-积分）控制器常用公式为：

$$
u[n] = K_p \cdot e[n] + K_i \cdot \sum_{k=0}^{n} e[k] \cdot T_s$$

或者递推形式（便于实时实现）：

$$
\text{integrator}_n = \text{integrator}_{n-1} + K_i \cdot e[n] \cdot T_s
$$$$
u[n] = K_p \cdot e[n] + \text{integrator}_n
​$$

其中

- $e[n]$：本次频率/相位误差
    
- $K_p$​：比例增益
    
- $K_i$​：积分增益
    
- $T_s$​：采样周期
    
- $\text{integrator}_{n}$​：积分器状态（累计误差）



## 可调参数

### 环路带宽

- 在二阶系统标准传递函数里，带宽 $(f_{BW})$ 通常被定义为使幅频响应下降到最大值的 $-3\,\mathrm{dB}$ 点对应的频率。
    
- 对应自然频率 $\omega_n$​（单位为弧度/秒），常用换算：
    
$$f_{BW} = \frac{\omega_n}{2\pi}​​$$


- **带宽越大**，系统对输入频率/相位的突变响应越快，锁定时间短；但抗噪能力降低，易受高频噪声影响。
    
- **带宽越小**，系统响应慢，锁定时间变长，但能很好抑制高频噪声（锁定更稳）。



### KP 和 KI


你设计一个锁相环/频率环，最重要的是**带宽**和**阻尼**能按你期望设定，而不是蒙着头调参数。  
理论上二阶系统的标准特性为：

$$\text{特征方程:} \quad s^2 + 2\zeta\omega_n s + \omega_n^2 = 0$$

希望调节参数后环路传递函数直接落在理想位置。

标准PI结构，离散化前（连续域）设计公式如下：

$$K_p = 2\zeta \omega_n$$
$$Ki=ωn2K_i = \omega_n^2$$

- $K_p$​：比例系数决定阻尼（动态稳定与超调）
    
- $K_i$​：积分系数决定静态误差（锁定精度）

通过上文所述的环路带宽计算得出

```
// 从连续域参数计算离散域KP和KI

const float omega_n = 2.0f * 3.1415926f * FLL_BANDWIDTH_HZ;

const float kp = 2.0f * FLL_DAMPING_RATIO * omega_n; // Kp在离散化中不变

const float ki = omega_n * omega_n * Ts; // Ki需要乘以采样周期
```
