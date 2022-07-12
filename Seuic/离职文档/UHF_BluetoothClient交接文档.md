## 1.下一步工作内容

- 建行新需求：日志过滤导出

## 2.注意事项

### 模块

#### app

- 与建行服务（UHF_AndroidReaderServer)通过蓝牙方式通信，控制读写器
- 搜索设备对UF3设备进行了过滤（蓝牙MAC名必须以UF3C开头）

#### bluetoothSdk

- 蓝牙sdk的源码在这，基本不需要改动。
- 使用的是传统蓝牙协议，不是低功耗的
- 建行服务（UHF_AndroidReaderServer)中导入了这个模块打成的arr包
