
```
/* ADC DMA half transfer complete callback */
void HAL_ADC_ConvHalfCpltCallback(ADC_HandleTypeDef* hadc)
{
  if(hadc->Instance == ADC1)
  {
    ADC_BufferReadyFlag = BUFFER_READY_FLAG_HALF;
  }
}

/* ADC DMA transfer complete callback */
void HAL_ADC_ConvCpltCallback(ADC_HandleTypeDef* hadc)
{
  if(hadc->Instance == ADC1)
  {
    ADC_BufferReadyFlag = BUFFER_READY_FLAG_FULL;
  }
}

```
只需要在HAL库中这样配置就行
```
- **DMA1 channel1 global interrupt**：已勾选（使能）
    
- **ADC1 and ADC2 global interrupts**：未勾选（未使能）
```

