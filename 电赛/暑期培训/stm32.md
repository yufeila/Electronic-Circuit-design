
## 外设驱动

在初始化阶段，需要定义两个函数，一个是工作模式配置函数I2C_Mode_Config，一个是HAL_I2C_MspInit函数，用于初始化相关的GPIO引脚和开启时钟，最后，定义一个总的外设初始化函数I2C_EE_Init，（MspInit在底层文件中弱定义，并被HAL_I2C_Init函数调用）

## 显示模块

本来想使用SPI-Flash存储中文字库，但是烧录到开发板上显示乱码，估计是编码问题，或者是字库并未烧录近SPI-Flash(但是官方文档说初始化时已经帮我们干了这一步)

显示模块调用：

- 一切工程都先基于显示模块构建(显示整数和小数)
- 关键函数：
	- **`ILI9341_DispStringLine_EN_CH`**
	- **`LCD_ClearLine`**
	- **`LCD_SetFont`**: 设置显示字体大小，如`&Font8x16`、`&Font24x32`等
	- **`LCD_SetColors`/`LCD_SetTextColor`** - 设置颜色
		调用方式：`LCD_SetTextColor(GREEN);`
	- **`ILI9341_Clear`** - 清屏函数

- 简单的调用方式

```
// 在第N行显示字符串
LCD_SetFont(&Font8x16);                           // 设置字体
LCD_SetColors(RED, BLACK);                        // 设置颜色(前景色,背景色)
LCD_ClearLine(LINE(N));                           // 清除第N行
ILI9341_DispStringLine_EN_CH(LINE(N), "您要显示的字符串");  // 显示在第N行
```


- 定义字符缓冲区`dispBuff`: 

```
char dispBuff[100];
```

	1. **格式化字符串的临时存储区**
	2. **在显示前进行字符串操作**
	3. **存储动态生成的文本**

调用时都封装成函数接口的形式：
```
/**
  * @brief  LCD英文显示测试函数
  * @param  无
  * @retval 无
  */
void LCD_Test2(void)
{
    /* 变量定义 */
    static uint8_t testCounter = 0;
    char dispBuff[100];
    
    /* 计数器递增 */
    testCounter++;
    
    /* 设置字体和颜色 */
    LCD_SetFont(&Font8x16);
    LCD_SetColors(RED, BLACK);
    
    /* 清屏 */
    ILI9341_Clear(0, 0, LCD_X_LENGTH, LCD_Y_LENGTH);
    
    /* 显示标题 */
    ILI9341_DispStringLine_EN_CH(LINE(0), "LCD ASCII Test");
    
    /* 显示分隔线 */
    ILI9341_DispStringLine_EN_CH(LINE(1), "--------------------");
    
    /* 显示计数器值 */
    sprintf(dispBuff, "Counter: %d", testCounter);
    LCD_ClearLine(LINE(3));
    ILI9341_DispStringLine_EN_CH(LINE(3), dispBuff);
    
    /* 显示基本文本 */
    LCD_ClearLine(LINE(5));
    ILI9341_DispStringLine_EN_CH(LINE(5), "Basic Text Display");
    
    /* 测试不同颜色 */
    LCD_SetTextColor(GREEN);
    LCD_ClearLine(LINE(7));
    ILI9341_DispStringLine_EN_CH(LINE(7), "Green Text Example");
    
    LCD_SetTextColor(BLUE);
    LCD_ClearLine(LINE(8));
    ILI9341_DispStringLine_EN_CH(LINE(8), "Blue Text Example");
    
    LCD_SetTextColor(YELLOW);
    LCD_ClearLine(LINE(9));
    ILI9341_DispStringLine_EN_CH(LINE(9), "Yellow Text Example");
    
    /* 测试不同字体 */
    LCD_SetColors(WHITE, BLACK);
    LCD_ClearLine(LINE(11));
    ILI9341_DispStringLine_EN_CH(LINE(11), "Testing Font24x32:");
    
    LCD_SetFont(&Font24x32);
    LCD_ClearLine(LINE(7));
    ILI9341_DispStringLine_EN_CH(LINE(7), "ABC123");
    
    /* 显示对齐文本示例 */
    LCD_SetFont(&Font8x16);
    LCD_SetColors(CYAN, BLACK);
    
    /* 左对齐 */
    LCD_ClearLine(LINE(13));
    ILI9341_DispStringLine_EN_CH(LINE(13), "Left aligned text");
    
    /* 右对齐(通过计算空格) */
    sprintf(dispBuff, "%*s", LCD_X_LENGTH/8, "Right aligned");
    LCD_ClearLine(LINE(14));
    ILI9341_DispStringLine_EN_CH(LINE(14), dispBuff);
    
    /* 居中对齐 */
    char *centerText = "Center Text";
    int spaces = (LCD_X_LENGTH/8 - strlen(centerText))/2;
    sprintf(dispBuff, "%*s%s", spaces, "", centerText);
    LCD_ClearLine(LINE(15));
    ILI9341_DispStringLine_EN_CH(LINE(15), dispBuff);
    
    /* 延时 */
    Delay(0xffffff);
}
```


## 关于HAL 库

## 🧠 STM32外设底层硬件初始化（MspInit）的内容

**通常包括如下3大项：**

1. **开启时钟**
    
    - 例如`__HAL_RCC_ADC1_CLK_ENABLE()`、`__HAL_RCC_GPIOC_CLK_ENABLE()`
        
    - 目的是让外设和相关GPIO有电，能正常工作。
        
2. **配置相应GPIO端口**
    
    - 例如设置为模拟输入、推挽输出等（如ADC引脚用`GPIO_MODE_ANALOG`）。
        
    - 可能有多个端口（比如ADC多通道、USART需要TX/RX）。
        
3. **配置中断**
    
    - 如果外设用到了中断功能，如USART、定时器、DMA等，会在MspInit里配置优先级、使能中断线。
        
    - 例如：`HAL_NVIC_SetPriority()`、`HAL_NVIC_EnableIRQ()`。
        

**DMA配置也算底层硬件配置的一部分。**

---

## 🔬 HAL_XXX_MspInit()的调用与文件位置

### 1. **在哪里实现的？**

- **文件位置：**  
    一般在你的工程目录下的 `stm32xxxx_hal_msp.c`（如 `stm32f4xx_hal_msp.c`）文件内。
    
- **CubeMX自动生成时，会在这个文件里为每个外设生成独立的 MspInit 函数。**
    
    - 例如：`void HAL_ADC_MspInit(ADC_HandleTypeDef* hadc)`
        
    - 例如：`void HAL_UART_MspInit(UART_HandleTypeDef* huart)`
        

### 2. **被谁调用？怎么调用？**

- **这些MspInit函数由 HAL_xxx_Init()（比如HAL_ADC_Init()）在初始化时自动调用**。
    
- **流程图如下：**
    

```
main.c
  └── MX_ADC1_Init()          // 你的外设初始化入口（CubeMX自动生成）
        └── HAL_ADC_Init()    // STM32官方HAL库提供的初始化函数
              └── HAL_ADC_MspInit() // 你的工程里的“底层硬件初始化”，自动被HAL库弱定义调用
```

### 3. **为什么说是“弱定义”？**

- HAL库内部用`__weak void HAL_ADC_MspInit()`等进行弱定义。
    
- 如果你的项目有同名的强定义（在 `stm32xxxx_hal_msp.c` 文件），就会**自动用你的版本**，不会用库里的空实现。
    

---

## 🏷️ 总结

> **底层硬件初始化函数`HAL_XXX_MspInit()`实现于`stm32xxxx_hal_msp.c`文件，由`HAL_XXX_Init()`自动调用。内容通常包括：时钟使能、GPIO配置、DMA和中断配置。`MX_XXX_Init()`是CubeMX生成的外设初始化入口，会调用`HAL_XXX_Init()`，从而间接调用你的底层MspInit。**
