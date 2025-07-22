
### **A. STM32F407 能力**

- **几乎支持 CMSIS-DSP 所有功能，包括：**
    
    - **浮点 FFT/FIR/IIR**（arm_cfft_f32/f64、arm_rfft_f32、arm_fir_f32等）
	    1. F407硬件FPU仅支持float（单精度）**
	    2. **STM32F407 内置的 FPU（浮点运算单元）** 只支持 **IEEE754 单精度浮点（float, 32位）**。
	    3. **双精度（double, 64位）**运算只能靠软件模拟，非常慢且资源消耗大，实际工程一般不用。
        
    - **定点（Q15/Q31）所有算法**（arm_cfft_q15/q31，arm_fir_q15/q31等）
        
    - **矩阵/统计/滤波/插值/向量操作/卷积/相关等所有数学运算**
        
    - **利用硬件 FPU 和 DSP 指令，速度极快**



### **B. STM32F103 能力**

- **仅能支持 CMSIS-DSP 的“定点（Q15/Q31）”和“部分F32”算法**
    
    - _虽然可以链接浮点库（arm_math_f32），但无 FPU，浮点都是软件模拟，速度极慢，实际不可用_。
        
    - **定点算法支持齐全**（适合音频、低速信号处理）
        
    - **无法高效用浮点FFT/FIR等算法**
        
    - **没有 DSP 指令优化，所有乘累加等都用普通指令，性能较差**


## 调用CMSIS的FFT库

不管你用的是**STM32F407（Cortex-M4）**还是**STM32F103（Cortex-M3）**，**调用的API函数名、参数、用法全部一致**。

### 定点FFT

- **定点FFT**，指的是**输入、运算和输出数据全部采用“定点数”（整数）表示和计算的FFT算法**。
    
    - 例如Q15（16位）、Q31（32位）格式的数据。
        
- 和你在数学/工程里见到的“浮点FFT”**（即float/double类型的数据、支持小数）**区别在于，**定点数只能精确表示有限的小数位**，本质是用整数模拟小数。

| 特性       | 定点FFT（Q15/Q31） | 浮点FFT（float/double） |
| -------- | -------------- | ------------------- |
| **数据类型** | 整型（short/int）  | 浮点型（float/double）   |
| **硬件需求** | 不需要FPU、适合任何MCU | 有FPU则更快             |
| **速度**   | 非FPU平台更快       | FPU平台快（无FPU慢）       |
| **精度**   | 精度低，易溢出/失真     | 精度高，范围广             |
| **内存占用** | 较小             | 较大                  |
| **工程应用** | 低端MCU/低速信号     | 高端MCU/高精度信号         |


- 关键函数接口

```
// Q15定点FFT
#include "arm_math.h"
#define FFT_LEN 1024
q15_t fft_input[2*FFT_LEN];
arm_cfft_instance_q15 fft_inst;
arm_cfft_init_q15(&fft_inst, FFT_LEN);
arm_cfft_q15(&fft_inst, fft_input, 0, 1);

```

### 浮点fft

**STM32F407**有硬件FPU，所以**可以直接用浮点FFT**


### **数据类型/精度区分**

- **f32** ：单精度浮点 (`float`)
    
- **f64** ：双精度浮点 (`double`)
    
- **q15** ：16位定点（Q15格式，int16_t）
    
- **q31** ：32位定点（Q31格式，int32_t）
    
- **f16** ：半精度浮点（float16，较新库支持）

### **FFT算法结构（Radix）**

- **Radix-2** ：基2FFT，最常用的通用算法，点数不限。
    
- **Radix-4** ：基4FFT，点数需为4的整数幂，运算效率高于Radix-2。
    
- **Radix-8** ：基8FFT，部分库支持，针对某些点数做进一步优化。
    
- **arm_cfft_xxx** ：通常是“自适应混合基（mixed radix）”或自动选择最佳算法的主接口，**一般用这个！**




### FFT最大点数

#### **A. 浮点FFT (`arm_cfft_f32`, `arm_rfft_fast_f32` 等)**

- **官方标准库最大点数：4096 点**
    
    - 常见的支持点数：16, 32, 64, 128, 256, 512, 1024, 2048, 4096（全部是2的幂）
        
- 有些新版本和特殊移植版本，`arm_cfft_f32`支持到8192或更大（**但STM32CubeF4/官方常用包，通常最大4096**）。
    

#### **B. 定点FFT (`arm_cfft_q15`, `arm_cfft_q31`)**

- **同样最大 4096 点**（有的官方包支持到8192点，但很罕见）
    
- 注意大点数下定点FFT更容易溢出。
    

#### **C. 实数FFT（`arm_rfft_fast_f32`）**

- 一般最大也是 4096 点（新版有8192）


## CMSIS DSP频谱分析函数


## 1. **FFT快速傅里叶变换函数**

### **复数FFT（Complex FFT）**

- **单精度浮点**
    
    - `arm_cfft_f32`    → 复数float FFT主函数
        
- **定点16位**
    
    - `arm_cfft_q15` → 复数Q15定点FFT
        
- **定点32位**
    
    - `arm_cfft_q31` → 复数Q31定点FFT
        
- **双精度浮点**
    
    - `arm_cfft_f64` → 复数double FFT（新库，部分芯片支持）
        

### **实数FFT（Real FFT）**

- **快速RFFT（推荐！）**
    
    - `arm_rfft_fast_f32` → 实数float FFT（ADC采样直接用）
        
    - `arm_rfft_fast_init_f32` → 实数FFT初始化
        
- **传统RFFT**
    
    - `arm_rfft_f32`、`arm_rfft_q15`、`arm_rfft_q31` → 兼容老接口

## 2. **DCT离散余弦变换函数（部分应用做频谱/压缩）**

- `arm_dct4_f32`、`arm_dct4_q15`、`arm_dct4_q31`
    
    - DCT-IV型，常用于音频、图像压缩，但在信号处理场合也可用于谱估计


## 3. **辅助分析与谱操作函数**

### **功率谱、幅度谱相关**

- **没有专门“自动算功率谱/幅度谱”的一键API**，但**可用下列函数辅助：**
    
    - `arm_cmplx_mag_f32` / `arm_cmplx_mag_q15` / `arm_cmplx_mag_q31`
        
        - 复数转模长（幅值谱）
            
        - 用于 FFT 结果后，快速计算每个bin的幅值
            
    - `arm_cmplx_mag_squared_f32` 等
        
        - 复数功率谱
            

### **最大/最小/峰值搜索**

- `arm_max_f32` / `arm_max_q15` / `arm_max_q31`
    
    - 最大值及其索引，用于找主频点
        
- `arm_min_f32` / `arm_min_q15` / `arm_min_q31`





## DSP_Lib库功能简介

 3、DSP_Lib中包含了如下所示的多个文件夹，它们中的文件功能如下：1）BasicMathFunctions

基本数学函数：提供浮点数的各种基本运算函数，如向量加减乘除等运算。

2）CommonTables

数字信号处理常用参数表。

3）ComplexMathFunctions

复数计算数学函数。

4）ControllerFunctions

控制算法函数。包括正弦余弦，PID电机控制，矢量Clarke变换，矢量Clarke逆变换等。

5）FastMathFunctions

常见快速算法的数学函数。

6）FilteringFunctions

滤波函数功能，主要为FIR和LMS（最小均方根）等滤波函数。

7）MatrixFunctions

矩阵处理函数。包括矩阵加法、矩阵初始化、矩阵反、矩阵乘法、矩阵规模、矩阵减法、矩阵转置等函数。

8）StatisticsFunctions

统计功能函数。如求平均值、最大值、最小值、计算均方根RMS、计算方差/标准差等。

9）SupportFunctions

支持功能函数，如数据拷贝，Q格式和浮点格式相互转换，Q任意格式相互转换。

10）TransformFunctions

变换功能。包括复数FFT（CFFT）/复数FFT逆运算（CIFFT）、实数FFT（RFFT）/实数FFT逆运算（RIFFT）、和DCT（离散余弦变换）和配套的初始化函数。

例如，这里我打算通过浮点FFT运算进行频谱分析，就使用了TransformFunctions文件夹下的arm_cfft_f32.c中的函数。

