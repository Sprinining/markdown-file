## 1.下一步工作内容

- 建行新需求：门禁模式

## 2.注意事项

### 通信方式

#### 蓝牙

- 通过导入UHF_BluetoothClient中的bluetoothSdk-debug.aar与手机端的应用通信

#### 网口

- 开机后由系统拉起，有蓝牙设备连接或者socket连接上服务端后，服务端才会去连接模块

- 使用netty与建行的上位机通信
- MessageUtils.java：所有对外的接口
- 导入AndroidReader-debug.aar（由UHF_AndroidReader项目中的AndroidReader模块封装而来）直接操作读写器，和AIDL服务是在统一层级的，所以同一台机子上最好不要轮流使用这两套应用（一套是建行的，一套是东集的）

## 3.其他文档

- 建行测试代码：一个简单的测试程序
- 建行需求1(已完成)：包含需求文档和实体集
- 建行需求2(未完成)：新需求的说明
