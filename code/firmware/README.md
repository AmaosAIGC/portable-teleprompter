# 固件（STM32F401CCU6）

推荐 PlatformIO（framework = stm32cube，board = blackpill_f401cc）或 Keil MDK + HAL 库。

计划结构：
```
src/
  main.c              初始化 + 主循环状态机
  bsp/spi.c           SPI1（42MHz，DMA）
  bsp/st7789.c        屏驱动：初始化、窗口、填充、MADCTL、VSCRDEF/VSCRSADD
  bsp/w25q64.c        Flash 读写擦、JEDEC 校验
  bsp/usart.c         USART1 + IDLE 中断 + DMA
  bsp/encoder.c       EC11 正交解码 + 按键消抖
  font/gb2312.c       编码转换 + 区位码寻址
  ui/render.c         行条带渲染
  engine/scroll.c     像素级滚动引擎
  app/script.c        多脚本存储与切换
  app/proto.c         帧解析 / CRC8
```
构建注意：`-O2`；HSE 25MHz（Black Pill 板载晶振）配 PLL 到 84MHz；SysTick 1ms；栈 ≥2KB。
