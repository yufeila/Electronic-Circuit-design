
使用USART将printf重定向到串口时，一定要启用MicroLIB, 否则，程序会卡住进入不了中断。


## 关于不定长中断

```
void HAL_UARTEx_RxEventCallback(UART_HandleTypeDef *huart, uint16_t Size)
```

是 **STM32 HAL 库**中，**接收完成（带 IDLE 分帧）中断的回调函数**，**专门为**

```
HAL_UARTEx_ReceiveToIdle_DMA()
```


- **不是**普通的 DMA half/full transfer 回调；
    
- **不是**普通的 `HAL_UART_RxCpltCallback`（接收一整个固定长度才触发）；
    
- **是**`HAL_UARTEx_ReceiveToIdle_DMA()` + IDLE 检测下，HAL 自动帮你触发的数据包边界回调。