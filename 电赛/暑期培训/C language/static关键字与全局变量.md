

## staic

### 引入

我们知道在函数内部定义的变量，当程序执行到它的定义处时，编译器为它在**栈上**分配空间，函数在栈上分配的空间在此**函数执行结束**时会释放掉，这样就产生了一个问题: 如果想将函数中此变量的值保存至下一次调用时，如何实现？ 最容易想到的方法是定义为全局的变量，但定义一个**全局变量**有许多缺点，最明显的缺点是**破坏了此变量的访问范围**（使得在此函数中定义的变量，不仅仅只受此函数控制）。static 关键字则可以很好的解决这个问题。


### 静态数据的存储

#### 全局(静态)存储区

**Data 段**(全局初始化区)

存放初始化的全局变量和静态变量，程序运行结束时自动释放。

1. 存储在静态数据区的变量会在程序刚开始运行时就完成初始化，也是唯一的一次初始化

**BSS 段**（全局未初始化区）

存放未初始化的全局变量和静态变量，程序运行结束时自动释放。

1. BSS段在程序执行之前会被系统自动清0；
2. 未初始化的全局变量和静态变量在程序执行之前已经为0；


**static的内部实现机制**

静态数据成员要在程序一**开始运行**时就必须存在。因为函数在程序运行中被调用，所以静态数据成员不能在任何函数内分配空间和初始化。静态数据成员要在程序一开始运行时就必须存在。因为函数在程序运行中被调用，所以静态数据成员不能在任何函数内分配空间和初始化。

这样，它的空间分配有三个可能的地方:

1. 一是作为类的**外部接口的头文件**，那里有**类声明**；
2. 二是**类定义的内部实现**，那里有类的成员函数定义；
3. 三是应用程序的 main() 函数前的**全局数据声明和定义处**。


**存储机制**
static 被引入以告知编译器，将变量存储在程序的静态存储区而非栈上空间，静态数据成员按定义出现的先后顺序依次初始化，注意静态成员嵌套时，要保证所嵌套的成员已经初始化了。消除时的顺序是初始化的反顺序。

**优势**
可以**节省内存**，因为它是所有对象所公有的，因此，对多个对象来说，静态数据成员**只存储一处**，供所有对象共用。静态数据成员的值对每个对象都是一样，但它的值是可以更新的。只要对静态数据成员的值更新一次，保证所有对象存取更新后的相同的值，这样可以提高时间效率。

#### 特点

- （1）该变量在**全局数据区**分配内存；
- （2）静态局部变量**在程序执行到该对象的声明处时被首次初始化**，即以后的函数调用不再进行初始化；
- （3）静态局部变量一般在声明处初始化，**如果没有显式初始化，会被程序自动初始化为 0**；(但是在程序开始时就已经存在)
- （4）它始终**驻留在全局数据区**，直到**程序运行结束**。但其**作用域**为**局部作用域**，当定义它的函数或语句块结束时，其作用域随之结束。



## 相关错误记录


### 错误现象:

在程序启动后，无法进入自定义中断标志位引导的条件语句块，并在调试过程中，自定义中断标志位始终不变, 数据处理最终结果存放数组的各位始终为0 。

### 原因

采用了双缓存区机制：

DMA数据搬运一半启动FFT数据处理处理前半部分数据，DMA继续搬运，到搬运结束后再启动FFT数据处理后半部分。

问题：**中断丢失现象**

数据处理函数(时间/空间)复杂度过高，导致DMA搬运全部完成时，前一半数据尚未处理完成。甚至又一次半传输完成中断标志位置位时，数据都未处理完，就会导致一些错误。


### BUG修复

#### 函数调用逻辑的改进

```
Display_Spectrum_Results()      // 负责界面刷新
└─► Process_ADC_Data_F32()      // 负责搬运 DMA 缓冲并决定是否真正分析
    └─► ProcessSampleData_F32() // 负责调用 spectrum_analysis() 得到频率/幅值

```

优势：

**主循环内反复调用 `Display_Spectrum_Results()`，内部通过标志位判断是否有新数据，然后再决定是否做 FFT、显示**

 1. **彻底解耦“中断”和“数据处理”**
	以前的做法（处理+显示都在“if(中断标志)”下）
	```c
if(half_flag) {
    数据处理();    // 可能复杂又慢
    数据显示();
    half_flag = 0;
}
```

- **风险**：如果“数据处理”很慢，
    
    - ① 在此期间 **新的 DMA 半/全传输中断** 可能已经来了，**但 main 循环还没跑完“数据处理”**，就会错过/丢失下一批数据（标志位没来得及清除/置位）。
        
    - ② 甚至**DMA 下次写数据时会覆盖当前处理中的缓冲区**，导致数据不一致，显示乱码或完全失真。
        
    - ③ 一旦占用太高，**中断优先级再高也进不去**，严重时直接“假死”。
        

 新方式:数据搬运/采样和数据分析完全解耦

```c
// DMA中断仅置标志
void HAL_ADC_ConvHalfCpltCallback(...) { ADC_BufferReadyFlag = BUFFER_READY_FLAG_HALF; }
void HAL_ADC_ConvCpltCallback(...)     { ADC_BufferReadyFlag = BUFFER_READY_FLAG_FULL; }

// 主循环里反复调用
while(1) {
    Display_Spectrum_Results();
    // 其他任务
}
```

- **好处**：
    
    - **中断响应几乎不耗时**，永远不会“卡住”；
        
    - **主循环不必跟 DMA 速度强行同步**，即使数据分析极其复杂（FFT、SINC插值等），下一帧采样也不会丢；新模式下，**每次只分析一帧，不会因为主循环慢而让 DMA 写乱分析区**，数据安全，分析慢最多就是帧率下降，不会错乱。
        
    - 永远不会出现“标志没及时清除/丢失”的情况。
    - `process_interval` 参数让你轻松限制“FFT 刷新频率”，即使采样 1000 次/sec，也只做 10 次 FFT；

    
 这种结构是绝大多数**音频/数据采集+DSP+实时显示系统**的经典架构。



#### 数据处理函数的BUG修复

(1)为什么 **`FFTResult` 总是 0**？

在 F1-系列（Cortex-M3）默认的 **main stack 只有 2 KB~4 KB**。  
而你在 `spectrum_analysis()` 里按下面方式临时开了两块 **VLA（可变长局部数组）**：

|数组|大小|说明|
|---|---|---|
|`float X_fft[N]`|1024 × 4 B = **4096 B**||
|`float mag[N/2+1]`|513 × 4 B ≈ **2052 B**||
|**合计**|**≈ 6148 B**||

一次调用就要 6 KB 多的栈，再加上 CMSIS-DSP、`printf` 等函数自己的栈开销，**立刻把 2 KB~4 KB 的 MSP 挤爆** → 结果是**静默栈溢出**：

- 程序并不会马上 HardFault，而是把栈底下方的全局变量（刚好是 `FFTResult`）覆盖成 0。
    
- 于是你在调试器里看到 `FFTResult.amplitude/frequency/bin_index` 全为 0，LCD 当然也不会刷新有意义的值。
    

---

(2) 一步到位的修复

1. 把“大数组”挪到 **静态区 / 堆**，绝不要放在栈上

```c
/* ========== spectrum_analysis.c 顶部 ========== */
#include "./spectrum_analysis/spectrum_analysis.h"
#include "arm_math.h"
#include "arm_const_structs.h"
#include <math.h>

#define MAX_FFT_N   1024                 // 目前只支持 1024 点
/* 静态工作缓冲，放在 .bss，不占用栈 */
static float   g_fft_workbuf[MAX_FFT_N];          // rfft 输出/工作区
static float   g_mag_buf[MAX_FFT_N / 2 + 1];      // 幅度谱
/* ------------------------------------------------ */
```

然后把原本的 VLA 改成对这两个全局缓冲的引用：

```c
void spectrum_analysis(const float *x, uint16_t N, float fs, SpectrumResult_t *result)
{
    float *X_fft = g_fft_workbuf;   // 替代原来的 float X_fft[N]
    float *mag   = g_mag_buf;       // 替代原来的 float mag[N/2+1]

    calc_magnitude(x, N, mag);      // 其内部别再声明 VLA！
    ...
}
```

`calc_magnitude()` 里同样删掉 `float X_fft[N];`，直接使用传进来的 `X_fft` 缓冲，或者再开一个静态缓冲（见示例补丁）。

> **好处**：
> 
> - 栈占用从 6 KB 降到几十字节，绝不会再溢出。
>     
> - 静态缓冲只有一份，可重复用，不会频繁 memset/alloc。
>     

---

（3）若仍想用 **局部数组**，就必须 **增大堆栈**

- **Keil / IAR**：在项目 Option → Linker → Stack Size 改到 8 KB 以上。
    
- **GCC-Make / CubeIDE**：在 `startup_stm32f103xx.s` 里把
    
    ```
    Stack_Size      EQU     0x00000800    ; 2 KB
    ```
    
    改成 `0x00002000`（8 KB），并确保 `.ld` 脚本里留足空间。
    

> 但在资源紧张的 MCU 上，推荐方案 1 把大缓冲做成静态全局，更可控。

---

（4） 已测试补丁（关键片段）

```diff
/* ---------- spectrum_analysis.c ---------- */
- static void calc_magnitude(const float *x, uint16_t N, float *mag) {
-     float X_fft[N];
+ static void calc_magnitude(const float *x, uint16_t N,
+                            float *workbuf, float *mag)   /* 新增参数 workbuf */
+ {
-     arm_rfft_fast_instance_f32 S;
-     arm_rfft_fast_init_f32(&S, N);
-     arm_rfft_fast_f32(&S, (float *)x, X_fft, 0);
+     arm_rfft_fast_instance_f32 S;
+     arm_rfft_fast_init_f32(&S, N);
+     arm_rfft_fast_f32(&S, (float *)x, workbuf, 0);
...
-     float re = X_fft[2*k];
-     float im = X_fft[2*k+1];
+     float re = workbuf[2*k];
+     float im = workbuf[2*k+1];
...
  }

void spectrum_analysis(const float *x, uint16_t N, float fs, SpectrumResult_t *result)
{
-     float mag[N/2 + 1];
-     calc_magnitude(x, N, mag);
+     float *work = g_fft_workbuf;
+     float *mag  = g_mag_buf;
+     calc_magnitude(x, N, work, mag);   /* 传入共用缓冲 */
...
}
```

把以上补丁应用后重新编译，**FFTResult 会立刻出现正常数值**（例如 `Freq: 997.56 Hz`, `Amp: 1834.2 AU`），LCD 也随 DMA 采样实时刷新。


(4) 其他好用工具

| 项目                    | 说明                                                                                                                          |
| --------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| **HAL_ADC_Start_DMA** | 请确保在 `main()` 的 `MX_ADC1_Init()` 之后调用一次 `HAL_ADC_Start_DMA(&hadc1, (uint32_t*)adc_buffer, ADC_BUFFER_SIZE);`，否则 DMA 不会真正启动。 |
| **零点偏置**              | 12-bit ADC 理论中点是 2048，但如果你的模拟前端有偏置，可在调试时把 `-2048.0f` 改成实测均值。                                                                |
| **打印格式**              | `sprintf(dispBuff,"Amp:%8.3f",val)` 在 `val<0.001` 时会打印 `0.000`，如需更直观，可改成科学计数 `%8.3e`。                                       |
| **栈溢出检测**             | 开启 MPU 或在 Keil 设 `--C99 --check_stack`，能第一时间捕获溢出。                                                                           |

修完 **“栈溢出”** 这一核心问题后，`FFTResult` 就能正常更新，LCD 也会持续显示实时的频率与幅度。祝调试顺利！



### 改进后代码

```
#include "./spectrum_analysis/spectrum_analysis.h"
#include "arm_math.h"
//#include "arm_const_structs.h"
#include <math.h>

/* ==============================================================
 *  用户可调常量
 * ============================================================*/
#define MAX_FFT_N   1024u        /* 允许的最大 FFT 点数 (必须 ≥ N) */

/* ==============================================================
 *  静态工作缓冲区 —— 放在 .bss，绝不占用栈
 * ============================================================*/
static float g_fft_workbuf[MAX_FFT_N];          /* RFFT 输出/临时区   N  个 float */
static float g_mag_buf [MAX_FFT_N/2 + 1];       /* 单边幅度谱         N/2+1 个 */


// ========== 工具函数 ==========

/* 归一化 sinc(x) = sin(pi x)/(pi x) */
static __inline float sinc_norm(float x)
{
    if (fabsf(x) < 1e-6f) return 1.0f;
    return sinf(M_PI * x) / (M_PI * x);
}

/* 由最大幅值与邻点幅值求 |δ| = r/(1+r)，0≤|δ|≤1(这里采用了递归调用的思路) */
static __inline float interp_delta(float mag_max, float mag_neigh)
{
    if (mag_neigh <= 0.0f || mag_max <= 0.0f) return 0.0f;
    float r = mag_neigh / mag_max;
    return r / (1.0f + r);
}

/**
 * @brief  计算单边幅度谱 |X[k]|，结果存入 mag[]
 * @param  x        实数输入序列，长度 N
 * @param  N        FFT 点数 (2 的幂)
 * @param  workbuf  长度 N 的工作区 (RFFT 输出)
 * @param  mag      输出幅度谱，长度 N/2+1
 */
static void calc_magnitude(const float *x, uint16_t N,
                           float *workbuf,
                           float *mag)
{
    /* 1) 实数 FFT */
    arm_rfft_fast_instance_f32 S;
    arm_rfft_fast_init_f32(&S, N);
    arm_rfft_fast_f32(&S, (float *)x, workbuf, 0);      /* 正向变换 */

    /* 2) 单边幅度谱 |X[k]| */
    mag[0] = fabsf(workbuf[0]);                         /* DC */
    for (uint16_t k = 1; k < N/2; ++k)
    {
        float re = workbuf[2u * k];
        float im = workbuf[2u * k + 1u];
        mag[k] = sqrtf(re * re + im * im);
    }
    mag[N/2] = fabsf(workbuf[1]);                       /* Nyquist */
}

/* ==============================================================
 *  主接口：高精度幅频测量（3 点 sinc 插值）
 * ============================================================*/
void spectrum_analysis(const float *x, uint16_t N, float fs, SpectrumResult_t *result)
{
    /* -------- 参数检查 -------- */
    if (!x || !result || N == 0 || (N & (N - 1))) return; /* N 必须是 2 的幂 */
    if (N > MAX_FFT_N) return;                            /* 越界保护          */

    float *work = g_fft_workbuf;         // 全局静态变量
    float *mag  = g_mag_buf;             // 全局静态变量

    /* 1) 幅度谱 */
    calc_magnitude(x, N, work, mag);

    /* 2) 找主峰(跳过 DC) */
    uint16_t k_max   = 1;
    float    max_val = mag[1];
    for (uint16_t k = 2; k < N / 2; ++k)
    {
        if (mag[k] > max_val) { max_val = mag[k]; k_max = k; }
    }

    /* 3) 选左 / 右邻幅值大的做插值 */
    uint16_t k_left  = (k_max == 1)       ? k_max     : k_max - 1;
    uint16_t k_right = (k_max >= N/2 - 1) ? k_max     : k_max + 1;
    uint16_t k_neigh = (mag[k_right] > mag[k_left]) ? k_right : k_left;

    /* 4) 三点 sinc-插值求 δ (-1..1) */
    float delta = interp_delta(mag[k_max], mag[k_neigh]);
    delta *= (k_neigh > k_max) ? 1.0f : -1.0f;

    /* 5) 频率 & 幅值校正 */
    float k_true = (float)k_max + delta;                /* 真正 bin 位置 */
    float freq   = k_true * fs / (float)N;              /* 频率估计 (Hz)  */

    /* ---------- 幅度校正 ---------- */
    /* CMSIS RFFT 的幅度定义: |X[k]| = A_pk * N / 2
    * 因此要恢复到幅度峰值 A_pk ： Amp_pk = mag * 2 / N
    */
    float amp_pk = (mag[k_max] / fabsf(sinc_norm(delta))) * (2.0f / (float)N);

    result->amplitude = amp_pk;     /* 单位 = Volt_peak (Vpk) */
    result->frequency = freq;
    result->bin_index = k_max;
    result->delta     = delta;
   
}

```

