## 1.下一步工作内容

- 3个三级bug，2个二级bug

## 2.注意事项

- 1.AIDL服务也在这个项目里，如果修改接口，需要重新打成arr，更新到UHF_ReaderServer、UHF_ConfigTools中
  2.有一个二级bug（对TID区过滤失效）是so库的问题，需要芯联更新；一个三级bug（开启蜂鸣器连续寻卡时概率性会出现蜂鸣器长鸣一声然后继续正常寻卡）应该跟应用没啥关系，可能是硬件或者gpio端口的问题，大概率是gpio端口的问题）

#### AndroidReader模块

- 调用so库和jar包，对读写器有直接操作的权限
- UhfReader.kt：封装jar包后的接口，如果修改了接口的定义，需要修改readSdk模块中的UhfReaderSdk.kt
- RFIDService：AIDL服务，开机后会被系统拉起

#### app模块

- 有界面的一个应用，通过readSdk和AIDL服务通信，调用读写器的接口

#### readSdk模块

- 用于和AIDL服务通信，其他应用如果想使用读写器，需要导入此模块，禁止除AIDL服务以外的应用直接操作读写器
- AIDL服务接口改动后，只需要修改这个模块的UhfReaderSdk.kt

#### 机器硬件的一些问题

- micro接口与RJ45接口不可同时使用
- mirco接口插着数据线时，有时候机器会启动不了
- 寻标签天线不要使用自带的黑色塑料天线，天线识别接口无法识别这种

## 3.其他文档

- so库：各个版本的so库
- 芯联：信联提供的一些文档，包括demo源码和接口文档
- app-debug-v220316.apk：芯联的demo
- AUTOID UF3-S 安卓端使用说明书V1.3.docx：app模块的说明文档

- ght-gpiotest.apk：广和通提供的测试gpio的软件，但不太靠谱，有bug，从/sys/kernel/debug/gpio中看到的才是准确的状态
- GPIO.xlsx：gpio对应的端口号
