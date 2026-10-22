# Android App（BLE 控制端）

计划页面：
- `pages/index`：设备扫描/连接、状态显示
- `pages/control`：翻页、暂停、跳转、速度滑条、字号/颜色选择
- `pages/scripts`：脚本列表、预览、编辑、导入（wx.chooseMessageFile）

关键 API：`wx.openBluetoothAdapter` → `wx.startBluetoothDevicesDiscovery` → `wx.createBLEConnection` → `wx.writeBLECharacteristicValue` / `wx.notifyBLECharacteristicValueChange`
BLE 写需注意：单包 ≤20 字节（未协商 MTU 时），脚本传输要分包。
