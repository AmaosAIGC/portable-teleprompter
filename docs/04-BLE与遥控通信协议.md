# 通信协议（组内共用，改动需同步组员）

> 🚧 **项目进行中** — 本文件随开发实时更新，预计 **2 个月内**完成作品。

> 本协议是项目卡多个分支共用的接口契约。**第 2 周冻结**，之后任何字段改动必须更新本文件并通知组员。

## 一、BLE（手机端）
设备名：`Teleprompter-XX`（XX 为 MAC 末两字节）

### GATT 结构
| 用途 | UUID | 属性 |
|---|---|---|
| Service | `7a230001-1b2c-4d5e-8f90-a1b2c3d4e5f6` | primary |
| CTRL（下行命令） | `...0002-...` | Write Without Response |
| STATE（上行状态） | `...0003-...` | Notify + Read |
| SCRIPT（脚本传输） | `...0004-...` | Write |
| SCRIPT_RD（脚本读取） | `...0005-...` | Notify |

MTU 建议协商到 247，单包有效载荷 ≤ 244 字节。

### 命令帧（CTRL）
`[opcode][arg...]`，多字节小端。

| Opcode | 功能 | 参数 | 备注 |
|---|---|---|---|
| 0x01 | 播放 | — | 从当前 offset 继续 |
| 0x02 | 暂停 | — | |
| 0x03 | 下一页 | — | 按一屏高度前进 |
| 0x04 | 上一页 | — | |
| 0x05 | 跳转 | u32 offset_px | 0 = 开头 |
| 0x06 | 设速度 | u16 px_s | 10–400，超范围截断 |
| 0x07 | 设显示参数 | u8 font, u8 gap, u16 fg, u16 bg | |
| 0x08 | 镜像开关 | u8 on | |
| 0x09 | 导入开始 | u8 name_len + name + u32 total_bytes | 返回 0x81 |
| 0x0A | 导入分片 | u16 seq + payload | 每片 ≤240B |
| 0x0B | 导入结束 | u32 crc32 | 失败回 0x82 |
| 0x0C | 请求脚本列表 | — | 回 0x83 |
| 0x0D | 请求脚本内容 | u8 script_id + u32 from + u16 len | 走 SCRIPT_RD |
| 0x0E | 删除脚本 | u8 script_id | |
| 0x0F | 保存修改 | u8 script_id + u32 total + crc32 | 覆盖原文件 |

### 状态帧（STATE，2Hz 或变化时推）
`[0x80][state][u32 offset_px][u16 speed][u8 font][u8 gap][u8 script_id][u8 bat]`
`state`：0=IDLE 1=RUNNING 2=PAUSED 3=IMPORTING 4=ERROR

### 应答
`0x81 OK` `0x82 CRC_ERR` `0x83 SCRIPT_LIST(len + [id,name,size]...)` `0x84 BUSY`

## 二、2.4G 遥控（nRF24L01+）
- 频道 76，速率 1Mbps，地址 `0xT3L3P`（5 字节可自定义），CRC 16 位
- 帧 8 字节：`[0xA5][seq][cmd][arg1][arg2][crc8][0][0]`
- 命令集（与 BLE 语义一致，便于双端复用）：
  - `0x01` 播放 / `0x02` 暂停 / `0x03` 下一页 / `0x04` 上一页
  - `0x06` 速度 +/-（arg1=delta）
  - `0x07` 循环切换字号
- 遥控器侧（ESP32-C3）：按键 20ms 消抖，长按连发（300ms 后每 100ms 重发）
- 屏端回 ACK：同一 seq 回 `[0x5A][seq][0]`，遥控器收到可点灯/蜂鸣

## 三、约定
1. 所有多字节字段小端。
2. 未识别 opcode 静默丢弃，不断连。
3. 断线重连后手机端先发 0x0C 拉列表同步。
4. 版本号：协议 v1.0，字段增删必须升 minor 并记入本文件变更记录。

## 变更记录
- v1.0 2026-10-18 初稿
