# UhfReaderSdk文档

## 读写器连接
### 连接读写器

| 方法   | synchronized Integer connectReader() |
| ------ | ------------------------------------ |
| 参数   |                                      |
| 返回值 | 是否连接成功                         |
| 说明   |                                      |

### 关闭读写器

| 方法   | synchronized void closeReader() |
| ------ | ------------- |
| 参数   |               |
| 返回值 |               |
| 说明 | 一般不要调用此接口，因为可能有其他app也在用读写器 |

### 是否已打开读写器

| 方法   | Boolean isReaderOpen() |
| ------ | ------------------ |
| 参数   |                    |
| 返回值 | 是否已打开 |
| 说明 | 读写器模块是否已经打开 |

### 获取读写器工作状态

| 方法   | Integer getReaderWorkState() |
| ------ | -------------------- |
| 参数   |                      |
| 返回值 | 读写器工作状态 |
| 说明 |                                                      |

- 读写器工作状态

```kotlin
enum class ReaderWorkState(val state: Int) {
    READER_CLOSE(0),//关闭
    READER_OPEN(1),//开启
    READER_SEARCH(2),//寻卡
    READER_READ(3),//读数据
    READER_WRITE(4),//读数据
    READER_LOCKTAG(5),//锁卡
    READER_DESTROYTAG(6),//销毁卡
}
```


## 寻卡
### 注册寻卡监听

| 方法   | void registerReadListener(@Nullable IReadListener readListener) |
| ------ | -------------------------------------------------- |
| 参数   | 监听器                                             |
| 返回值 |                                                    |
| 说明 | 注册时要确保AIDL服务已经连接 |

- 监听器

```java
private static final IReadListener.Stub readListener = new IReadListener.Stub() {
    @Override
    public void tagRead(List<TagInfo> tags) {
    }
    @Override
    public void tagReadException(int errorCode) {
    }
    
    // 用于注销监听时，AIDL服务区分监听器
    @Override
    public String getTag() throws RemoteException {
        return "" + this.hashCode();
    }
};
```

- TagInfo

```java
// 标签被哪个天线读到
private final byte AntennaID;
// 标签是从哪个频点读到的
private final int Frequency;
// 标签读到的时间戳，单位毫秒（相对于命令发出的时刻）
private final int TimeStamp;
// 附加数据长度
private final short EmbededDatalen;
// 附加数据
@Nullable
private final byte[] EmbededData;
// epc长度，单位为字节
private final short Epclen;
// pc码，十六进制字节数组
@NotNull
private final byte[] PC;
// epc码，十六进制字节数组
@Nullable
private final byte[] EpcId;
// 标签协议
@Nullable
private final String protocol;
// 信号强度
private final int RSSI;
```

- AIDL连接时再注册监听

```java
// 注册观察者，AIDL服务连接上后再连接读写器和注册寻卡监听
com.seuic.androidreader.sdk.Constants.INSTANCE.getConnectState().observeForever(aidlInfoBean -> {
    if (aidlInfoBean.getState() == com.seuic.androidreader.sdk.Constants.CONNECTED) {
        WorkStateUtils.getINSTANCE().tryConnect();

        TagListener.getINSTANCE().registerReadListener();
        Log.e(TAG, "注册寻卡监听");
    }
});
```

- AIDL服务连接状态

```kotlin
const val DISCONNECT = 0 //未连接
const val CONNECTED = 1  //已连接
const val CONNECTING = 2 //正在连接中
```

### 注销寻卡监听

| 方法   | void unregisterReadListener(@Nullable IReadListener readListener) |
| ------ | ---------------------------------------------------- |
| 参数   | 监听器                                               |
| 返回值 |                                                      |
| 说明 |                                                      |

### 清空寻卡监听

| 方法   | void clearReadListeners() |
| ------ | ------------------- |
| 参数   |                     |
| 返回值 |                     |
| 说明 |                                                      |

### 单次寻卡

| 方法   | synchronized Integer inventoryOnce(@NotNull int[] ants, int timeout) |
| ------ | ------------------------------------------------- |
| 参数   | ants:启用的天线数组, timeout:超时时间(毫秒) |
| 返回值 | 0成功，非0失败 |
| 说明 | 寻卡数据通过监听器获取 |

### 连续寻卡

| 方法   | synchronized Integer inventoryStart(@NotNull int[] ants) |
| ------ | ---------------------------------------------------------- |
| 参数   | ants:启用的天线数组 |
| 返回值 | 0成功，非0失败 |
| 说明 | 寻卡数据通过监听器获取 |

### 停止寻卡

| 方法   | synchronized Integer inventoryStop() |
| ------ | --------------------- |
| 参数   |                       |
| 返回值 | 0成功，非0失败 |
| 说明 |                                                      |

## 标签操作
### 读标签

| 方法   | synchronized ParamsBackData<byte[]> readTagData(int antenna, int bank, int blockCount, int startAddress, @NotNull String accessPassword, @NotNull String epc) |
| ------ | ------------------------------------------------------------ |
| 参数   | antenna: 天线端口1~8<br/>bank: 区(0，保留区；1，EPC 区；2，TID 区；3，用户数据区)<br/>blockCount: 块数（一块一个字，字为单位）<br/>startAddress: 起始地址（字为单位）<br/>accessPassword: 访问密码<br/>epc: 要过滤的标签EPC值 |
| 返回值 | 十六进制的字节数组 |
| 说明 |  |

### 写标签

| 方法   | synchronized Integer writeTagData(int antenna, int bank, @NotNull String data, int startAddress, @NotNull String accessPassword, @NotNull String epc) |
| ------ | ------------------------------------------------------------ |
| 参数   | antenna: 天线端口1~8<br/>bank: 区(0，保留区；1，EPC 区；2，TID 区；3，用户数据区)<br/>data: 十六进制字符串<br/>startAddress: 起始地址（字为单位）<br/>accessPassword: 访问密码<br/>epc: 要过滤的标签EPC值 |
| 返回值 | 0成功，非0失败 |
| 说明 |                                                      |

### 锁标签

| 方法   | synchronized ParamsBackData\<Integer> lockTag(int antenna, int lockBank, int lockType, @NotNull String accessPassword, @NotNull String epc) |
| ------ | ------------------------------------------------------------ |
| 参数   | antenna: 天线端口1~8<br/>lockBank: 区(0，保留区；1，EPC 区；2，TID 区；3，用户数据区)<br/>lockType: 锁操作类型(0，解锁；1，锁定；2，永久锁定)<br/>accessPassword: 访问密码<br/>epc: 要过滤的标签EPC值 |
| 返回值 | 0成功，非0失败 |
| 说明 | 保留区前两个字是销毁密码，后两个字是访问密码；锁卡前要先修改访问密码（不能为00000000）；锁定保留区时，读写都需要使用修改后的密码； 锁定其他三个区时，可以用00000000去读，但是如果要写（TID区不可写），必须用修改后的密码 |

### 销毁标签

| 方法   | synchronized Integer killTag(int antenna, @NotNull String killPassword, @NotNull String epc) |
| ------ | --------------------------------------------------------- |
| 参数   | antenna:  天线端口1~8<br />killPassword: 销毁密码<br />epc: 要过滤的标签EPC值 |
| 返回值 | 0成功，非0失败 |
| 说明 |                                                      |


## 读写器参数

### 获取功率

| 方法   | ParamsBackData<AntPowerData[]> getPower() |
| ------ | ------------------------------------------------- |
| 参数   |                                                   |
| 返回值 | 功率1~33 |
| 说明 |  |

### 设置功率

| 方法   | Integer setPower(@NotNull AntPowerData[] antPowers) |
| ------ | ---------------------------------------------- |
| 参数   | 功率1~33 |
| 返回值 | 0成功，非0失败 |
| 说明 |  |

### 获取频段

| 方法   | ParamsBackData\<String> getRegion() |
| ------ | ------------------------------------ |
| 参数   | 频段字符串(FCC, ETSI, China1, China2) |
| 返回值 | 频段 |
| 说明 | FCC: 北美, ETSI: 欧洲, China1: 中国1, China2: 中国2 |

### 设置频段

| 方法   | Integer setRegion(String region) |
| ------ | ---------------------------------------------------- |
| 参数   | 频段字符串 |
| 返回值 | 0成功，非0失败 |
| 说明 |                                                      |

### 获取附加数据

| 方法   | ParamsBackData\<EmbededBean> getEmbededData() |
| ------ | ---------------------------------------------- |
| 参数   |                                                |
| 返回值 | 附加数据 |
| 说明 |                                                      |

- EmbededBean

```java
// 保留区0，TID区2，USER区3
private int bank; 
// 起始地址（字节为单位）
private int startaddr; 
// 字节数
private int bytecnt; 
// 访问密码
@NotNull
private byte[] accesspwd;
```

### 设置附加数据

| 方法   | Integer setEmbededData(int startAddress, int byteCount, int bank, String accessPassword, boolean enable) |
| ------ | ------------------------------------------------------------ |
| 参数   | startAddress: 起始地址（字节为单位）<br />byteCount: 字节数<br />bank: 保留区0，TID区2，USER区3<br />accessPassword: 访问密码<br />enable: 是否启用附加数据 |
| 返回值 | 0成功，非0失败 |
| 说明 | EPC区不能设置 |

### 获取过滤

| 方法   | ParamsBackData\<FilterBean> getFilter() |
| ------ | --------------------------------------- |
| 参数   |                                         |
| 返回值 | 过滤参数                                |
| 说明   |                                         |

- FilterBean

```java
// 过滤数据区域1,2,3，分别是EPC、TID、User
private int bank;
// 过滤数据起始地址（字节为单位）
private int startaddr;
// 过滤数据长度（比特位数）
private int flen;
// 过滤数据
@NotNull
private byte[] fdata;
// 是否匹配
private int isInvert;
```

### 设置过滤

| 方法   | Integer setFilter(int startAddress, int bank, String data, boolean isInvert, boolean enable) |
| ------ | ------------------------------------------------------------ |
| 参数   | startAddress: 起始地址（字节为单位）<br />bank: 过滤数据区域1,2,3，分别是EPC、TID、User<br />data: 过滤的数据<br />isInvert: 是否匹配<br />enable: 是否启用过滤 |
| 返回值 | 0成功，非0失败                                               |
| 说明   | enable为true设置后会一直生效，除非设置enable为false清空过滤  |

### 获取session

| 方法   | ParamsBackData\<Integer> getSession() |
| ------ | ---------------------------------- |
| 参数   |                                    |
| 返回值 | session |
| 说明 |                                                      |

### 设置session

| 方法   | Integer setSession(@IntRange(from = 0L,to = 3L) int session) |
| ------ | ---------------------------------------------------------- |
| 参数   | session 0~3 |
| 返回值 | 0成功，非0失败 |
| 说明 |                                                      |

### 获取profile

| 方法   | ParamsBackData\<Integer> getProfile() |
| ------ | ---------------------------------- |
| 参数   |                                    |
| 返回值 | profile |
| 说明 |                                                      |

### 设置profile

| 方法   | Integer setProfile(@IntRange(from = 1L,to = 4L)int profile) |
| ------ | ------------------------------ |
| 参数   | profile 1~4 |
| 返回值 | 0成功，非0失败 |
| 说明 |                                                      |

### 获取target

| 方法   | ParamsBackData\<Integer> getTarget() |
| ------ | --------------------------------- |
| 参数   |                                   |
| 返回值 | target |
| 说明 |                                                      |

### 设置target

| 方法   | Integer setTarget(@IntRange(from = 0L,to = 3L) int flag) |
| ------ | ------------------------------------------------------ |
| 参数   | target 0~3 |
| 返回值 | 0成功，非0失败 |
| 说明 |                                                      |

### 获取qValue

| 方法   | ParamsBackData\<Integer> getQValue() |
| ------ | --------------------------------- |
| 参数   |                                   |
| 返回值 | qValue |
| 说明 |                                                      |

### 设置qValue

| 方法   | Integer setQValue(@IntRange(from = -1L,to = 15L) int qValue) |
| ------ | ---------------------------------------------------------- |
| 参数   | q -1~15 |
| 返回值 | 0成功，非0失败 |
| 说明 | q=-1表示自动 |


### 获取识别到的天线

| 方法   | ParamsBackData\<InvantAntData> getInvantAnt() |
| ------ | ---------------------------------------------- |
| 参数   |                                                |
| 返回值 | 识别到的天线 |
| 说明 |                                                      |

- InvantAntData

```java
// 识别到的天线总数
private int antcnt;
// 识别到的天线数组
@NotNull
private int[] connectedants;
```

### 获取温度

| 方法   | ParamsBackData\<Integer> getTemperature() |
| ------ | ----------------------------------- |
| 参数   |                                     |
| 返回值 | 温度（摄氏度） |
| 说明 |                                                      |

### 获取读写器版本

| 方法   | ParamsBackData\<VersionData> getVersion() |
| ------ | ------------------------------------------ |
| 参数   |                                            |
| 返回值 | 读写器版本 |
| 说明 |                                                      |

- VersionData

```java
// 固件信息
@NotNull
private String firmdata;
// boot 版本
@NotNull
private String bootver;
// 硬件版本
@NotNull
private String hardver;
// 固件版本
@NotNull
private String firmver;
```


## GPIO

### 获取gpi

| 方法   | ParamsBackData\<Integer> getGpi(int gpi) |
| ------ | ------------------------------------------------------------ |
| 参数   | gpi: 端口号1~4 |
| 返回值 | 0低电平 |
| 说明 |  |

### 设置gpo

| 方法   | Integer setGpo(int gpo, int value) |
| ------ | ---------------------------------- |
| 参数   | gpo: 端口1~4<br />value: 电平高低 |
| 返回值 | 0成功，非0失败 |
| 说明 |                                                      |

## LED灯
### 寻卡时绿灯开始闪

| 方法   | void startBling() |
| ------ | ------------ |
| 参数   |              |
| 返回值 |              |
| 说明 | 在连续寻卡时调用此接口，绿灯闪烁 |

### 寻卡时绿灯停止闪

| 方法   | void stopBling() |
| ------ | ----------- |
| 参数   |             |
| 返回值 |             |
| 说明 | 停止连续寻卡时调用此接口，绿灯常亮 |

### 设置LED灯是否亮

| 方法   | void setLed(@IntRange(from = 0L,to = 3L) int port, boolean enable) |
| ------ | ------------------------------------------------------------ |
| 参数   | port:0红灯，1绿灯，2蓝灯，3粉灯<br />enable:true亮 |
| 返回值 |                                                              |
| 说明 | 一般不用调用此接口 |


## 蜂鸣器
### 设置蜂鸣器使能

| 方法   | void setBZEnable(boolean enable) |
| ------ | ---------------------------- |
| 参数   | 是否启用蜂鸣器 |
| 返回值 |                              |
| 说明 | 不是指让蜂鸣器响，而是指蜂鸣器被调用响的接口后能不能响 |

### 获取蜂鸣器使能

| 方法   | Boolean getBZEnable() |
| ------ | ----------------------- |
| 参数   |                         |
| 返回值 | 蜂鸣器是否处于可以使用状态 |
| 说明 | 处于不可使用状态时，仍可以调用蜂鸣器响的接口，但实际不会响 |

### 蜂鸣器响

| 方法   | void startBZ() |
| ------ | --------- |
| 参数   |           |
| 返回值 |           |
| 说明 | 蜂鸣器长鸣，超过15秒就会自动停 |

### 蜂鸣器不响

| 方法   | void stopBZ() |
| ------ | -------- |
| 参数   |          |
| 返回值 |          |
| 说明 | 停止蜂鸣器长鸣 |

### 寻卡时蜂鸣器开始响

| 方法   | void startBZForSearching() |
| ------ | --------------------- |
| 参数   |                       |
| 返回值 |                       |
| 说明 | 在连续寻卡时调用此接口，每秒响一次 |

### 寻卡时蜂鸣器停止响

| 方法   | void stopBZForSearching() |
| ------ | -------------------- |
| 参数   |                      |
| 返回值 |                      |
| 说明 | 停止连续寻卡时调用此接口，停止蜂鸣器响 |
## 封装类

- ParamsBackData<T>

```java
// 操作是否成功0表示成功，成功再去取data
private int err;
// 获取的结果
@Nullable
private Object data;
// data的类型
@Nullable
private Class classType;
```

