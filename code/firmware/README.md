# 固件（ESP32-S3）

计划结构：
```
src/
  main.cpp              入口与任务创建
  display/st7796.cpp    屏驱动（SPI + DMA）
  font/gb2312.cpp       字库寻址与编码转换
  render/text_view.cpp  文本窗口渲染
  engine/scroll.cpp     滚动引擎（px/s 模型）
  input/encoder.cpp     EC11 + 按键消抖
  storage/scripts.cpp   FATFS 多脚本管理
  ble/gatt.cpp          BLE 服务与命令分发
  radio/nrf24.cpp       2.4G 遥控
  app/state.cpp         状态机
```
构建：PlatformIO（推荐）或 Arduino IDE。目标芯片 esp32-s3，PSRAM 开启。
