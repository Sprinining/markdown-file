# UhfReaderSdk文档

## 注册寻卡监听

| 方法   | registerReadListener(readListener: IReadListener?) |
| ------ | -------------------------------------------------- |
| 参数   | 监听器                                             |
| 返回值 |                                                    |
| 说明 | 注册时要确保AIDL服务处于连接中 |

- 监听器

```java
private static final IReadListener.Stub readListener = new IReadListener.Stub() {
    @Override
    public void tagRead(List<TagInfo> tags) {
    }
    @Override
    public void tagReadException(int errorCode) {
    }
};
```

- AIDL连接时再注册监听

```java
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
const val CONNECTED = 1  //已连接
const val DISCONNECT = 0 //未连接
const val CONNECTING = 2 //正在连接中
```

## 注销寻卡监听

| 方法   | unregisterReadListener(readListener: IReadListener?) |
| ------ | ---------------------------------------------------- |
| 参数   | 监听器                                               |
| 返回值 |                                                      |
| 说明 |                                                      |

## 清空寻卡监听

| 方法   | clearReadListener() |
| ------ | ------------------- |
| 参数   |                     |
| 返回值 |                     |
| 说明 |                                                      |

## 连接读写器

| 方法   | connectReader(address: String, rtype: Int): Int?       |
| ------ | ------------------------------------------------------ |
| 参数   | address:串口地址，rtype:天线数                         |
| 返回值 | 是否连接成功                                           |
| 说明   | UhfReaderSdk.INSTANCE.connectReader("/dev/ttyHS0", 8); |

## 是否已打开读写器

| 方法   | isOpen(): Boolean? |
| ------ | ------------------ |
| 参数   |                    |
| 返回值 | 是否已打开 |
| 说明 | 读写器模块是否已经打开 |

## 获取读写器工作状态

| 方法   | getWorkState(): Int? |
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

## 关闭读写器

| 方法   | closeReader() |
| ------ | ------------- |
| 参数   |               |
| 返回值 |               |
| 说明 | 一般不要调用此接口，因为可能有其他app也在用读写器 |

## 单次寻卡

| 方法   | inventoryOnce(ants: IntArray, timeout: Int): Int? |
| ------ | ------------------------------------------------- |
| 参数   |                                                   |
| 返回值 | 0成功，非0失败 |
| 说明 | 寻卡数据都要通过监听器获取 |

- TagInfo

```kotlin
class TagInfo(
    // 天线端口
    val AntennaID: Byte = 0,
    val Frequency: Int = 0,
    // 时间戳
    val TimeStamp: Int = 0,
    // 附加数据长度
    val EmbededDatalen: Short = 0,
    // 附加数据
    val EmbededData: ByteArray? = null,
    val Res: ByteArray = ByteArray(2),
    // epc长度
    val Epclen: Short = 0,
    // pc码，十六进制字节数组
    val PC: ByteArray = ByteArray(2),
    // crc码，十六进制字节数组
    val CRC: ByteArray = ByteArray(2),
    // epc码，十六进制字节数组
    val EpcId: ByteArray? = null,
    val Phase: Int = 0,
    val protocol: String? = null,
    val ReadCnt: Int = 0,
    // 信号强度
    val RSSI: Int = 0,
) 
```

## 连续寻卡

| 方法   | inventoryStart(ants: IntArray, userEmbeded: Boolean): Int? |
| ------ | ---------------------------------------------------------- |
| 参数   |                                                            |
| 返回值 | 0成功，非0失败 |
| 说明 |                                                      |

## 停止寻卡

| 方法   | inventoryStop(): Int? |
| ------ | --------------------- |
| 参数   |                       |
| 返回值 | 0成功，非0失败 |
| 说明 |                                                      |

## 读标签操作

| 方法   | readTagData(<br/>        currentAntenna: Int,<br/>        bank: Int,<br/>        blkcnt: Int,<br/>        address: Int,<br/>        accessWord: String<br/>    ): ParamsBackData<ByteArray>? |
| ------ | ------------------------------------------------------------ |
| 参数   | currentAntenna: Int,天线端口1~8<br/>bank: Int,区(0，保留区；1，EPC 区；2，TID 区；3，用户数据区)<br/>blkcnt: Int,块数（一块一个字，16位）<br/>address: Int,起始地址（字为单位）<br/>accessWord: String访问密码<br/> |
| 返回值 | 十六进制的字节数组 |
| 说明 | 进行单标签操作（读写锁销毁）前都要设置过滤，过滤出要操作的标签，操作完，过滤效果要取消或者设置回上一次的过滤 |

## 写标签操作

| 方法   | writeTagData(<br/>        currentAntenna: Int,<br/>        bank: Int,<br/>        data: String,<br/>        address: Int,<br/>        accessWord: String<br/>    ): Int? |
| ------ | ------------------------------------------------------------ |
| 参数   | currentAntenna: Int,天线端口1~8<br/>bank: Int,区(0，保留区；1，EPC 区；2，TID 区；3，用户数据区)<br/>data: String,十六进制字符串<br/>address: Int,起始地址（字为单位）<br/>accessWord: String访问密码<br/> |
| 返回值 | 0成功，非0失败 |
| 说明 |                                                      |

## 锁标签操作

| 方法   | lockTag(<br/>        currentAntenna: Int,<br/>        lbank: Int,<br/>        ltype: Int,<br/>        accessWord: String<br/>    ): ParamsBackData<Int>? |
| ------ | ------------------------------------------------------------ |
| 参数   | currentAntenna: Int,天线端口1~8<br/>lbank: Int,区(0，保留区；1，EPC 区；2，TID 区；3，用户数据区)<br/>ltype: Int,锁操作类型(0，解锁；1，锁定；2，永久锁定)<br/>accessWord: String访问密码<br/> |
| 返回值 | 0成功，非0失败 |
| 说明 | 保留区前两个字是销毁密码，后两个字是访问密码；锁卡前要先修改访问密码（不能为00000000）；锁定保留区时，读写都需要使用修改后的密码； 锁定其他三个区时，可以用00000000去读，但是如果要写（TID区不可写），必须用修改后的密码 |

## 销毁标签操作

| 方法   | destoryTag(currentAntenna: Int, accessWord: String): Int? |
| ------ | --------------------------------------------------------- |
| 参数   | currentAntenna: Int, Int,天线端口1~8<br /> accessWord: String销毁密码 |
| 返回值 | 0成功，非0失败 |
| 说明 |                                                      |

## 获取输出功率

| 方法   | getPower(): ParamsBackData<Array<AntPowerData?>>? |
| ------ | ------------------------------------------------- |
| 参数   |                                                   |
| 返回值 | 各个天线端口的输入和输出功率 |
| 说明 | writePower3000对应功率30（要除100） |

- AntPowerData

```kotlin
class AntPowerData(
    // 天线端口
    var antid: Int = 0,
    var readPower: Short = 0,
    var writePower: Short = 0
)
```

## 设置功率

| 方法   | setPower(antPowers: Array<AntPowerData>): Int? |
| ------ | ---------------------------------------------- |
| 参数   | 功率数组 |
| 返回值 | 0成功，非0失败 |
| 说明 |                                                      |

## 获取频段

| 方法   | getRegion(): ParamsBackData<String>? |
| ------ | ------------------------------------ |
| 参数   | 频段字符串 |
| 返回值 | 频段 |
| 说明 |                                                      |

- 频段

```java
public static enum Region_Conf {
    RG_NONE(0),
    RG_NA(1),
    RG_EU(2),
    RG_EU2(7),
    RG_EU3(8),
    RG_KR(3),
    RG_PRC(6),
    RG_PRC2(10),
    RG_OPEN(255);

    int p_v;

    private Region_Conf(int v) {
        this.p_v = v;
    }

    public int value() {
        return this.p_v;
    }

    public static Reader.Region_Conf valueOf(int value) {
        switch(value) {
        case 0:
            return RG_NONE;
        case 1:
            return RG_NA;
        case 2:
            return RG_EU;
        case 3:
            return RG_KR;
        case 6:
            return RG_PRC;
        case 7:
            return RG_EU2;
        case 8:
            return RG_EU3;
        case 10:
            return RG_PRC2;
        case 255:
            return RG_OPEN;
        default:
            return null;
        }
    }
}
```

## 设置频段

| 方法   | setRegion(@RegionConfAnnotation rcfValue: Int): Int? |
| ------ | ---------------------------------------------------- |
| 参数   | 频段对应的int值 |
| 返回值 | 0成功，非0失败 |
| 说明 |                                                      |

## 获取附加数据配置

| 方法   | getEmbededData(): ParamsBackData<EmbededBean>? |
| ------ | ---------------------------------------------- |
| 参数   |                                                |
| 返回值 | 附加数据配置 |
| 说明 |                                                      |

- 附加数据

```kotlin
class EmbededBean(
    // 保留区0，TID区2，USER区3
    var bank: Int = 0,
    // 起始地址（字为单位）
    var startaddr: Int = 0,
    // 字节数
    var bytecnt: Int = 0,
    // 访问密码
    var accesspwd: ByteArray = ByteArray(4)
)
```

## 设置并启用附加数据配置

| 方法   | setEmbededData(st: String, ct: String, bank: Int, pwd: String): Int? |
| ------ | ------------------------------------------------------------ |
| 参数   | st: String,起始地址（字为单位）<br />ct: String,字节数<br />bank: Int,保留区0，TID区2，USER区3<br />pwd: String访问密码 |
| 返回值 | 0成功，非0失败 |
| 说明 | EPC区不能设置 |

## 设置不启用附加数据

| 方法   | setEmbededDataNull(): Int? |
| ------ | -------------------------- |
| 参数   |                            |
| 返回值 | 0成功，非0失败 |
| 说明 |                                                      |

## 获取session

| 方法   | getSession(): ParamsBackData<Int>? |
| ------ | ---------------------------------- |
| 参数   |                                    |
| 返回值 | session |
| 说明 |                                                      |

## 设置session

| 方法   | setSession(@IntRange(from = 0, to = 3) session: Int): Int? |
| ------ | ---------------------------------------------------------- |
| 参数   | session 0~3 |
| 返回值 | 0成功，非0失败 |
| 说明 |                                                      |

## 获取profile

| 方法   | getProfile(): ParamsBackData<Int>? |
| ------ | ---------------------------------- |
| 参数   |                                    |
| 返回值 | profile |
| 说明 |                                                      |

## 设置profile

| 方法   | setProfile(profile: Int): Int? |
| ------ | ------------------------------ |
| 参数   | profile 1~4 |
| 返回值 | 0成功，非0失败 |
| 说明 |                                                      |

## 获取target

| 方法   | getTarget(): ParamsBackData<Int>? |
| ------ | --------------------------------- |
| 参数   |                                   |
| 返回值 | target |
| 说明 |                                                      |

## 设置target

| 方法   | setTarget(@IntRange(from = 0, to = 3) flag: Int): Int? |
| ------ | ------------------------------------------------------ |
| 参数   | target 0~3 |
| 返回值 | 0成功，非0失败 |
| 说明 |                                                      |

## 获取qValue

| 方法   | getQValue(): ParamsBackData<Int>? |
| ------ | --------------------------------- |
| 参数   |                                   |
| 返回值 | qValue |
| 说明 |                                                      |

## 设置qValue

| 方法   | setQValue(@IntRange(from = -1, to = 15) qValue: Int): Int? |
| ------ | ---------------------------------------------------------- |
| 参数   | q -1~15 |
| 返回值 | 0成功，非0失败 |
| 说明 | q=-1表示自动 |

## 获取gpi

| 方法   | getGpi(gpoId: Int, intArray: IntArray): ParamsBackData<IntArray>? |
| ------ | ------------------------------------------------------------ |
| 参数   | gpoId: Int, gpin的端口号1~4<br />intArray: IntArray已废弃 |
| 返回值 | gpi电平数组，实际只要用首元素，0低电平 |
| 说明 | 取返回的数组首元素 |

## 设置gpo

| 方法   | setGpo(gpoId: Int, gpo: Int): Int? |
| ------ | ---------------------------------- |
| 参数   | gpoId: Int,gpout端口1~4<br />gpo: Int电平高低 |
| 返回值 | 0成功，非0失败 |
| 说明 |                                                      |

## 获取识别到的天线

| 方法   | getInvantAnt(): ParamsBackData<InvantAntData>? |
| ------ | ---------------------------------------------- |
| 参数   |                                                |
| 返回值 | 识别到的天线 |
| 说明 |                                                      |

- 识别到的天线

```kotlin
class InvantAntData(
    // 识别到的天线总数
    var antcnt: Int = 0,
    // 识别到的天线数组
    var connectedants: IntArray = IntArray(16)
) 
```

## 获取温度

| 方法   | getTempture(): ParamsBackData<Int>? |
| ------ | ----------------------------------- |
| 参数   |                                     |
| 返回值 | 温度（摄氏度） |
| 说明 |                                                      |

## 获取读写器版本

| 方法   | getVersion(): ParamsBackData<VersionData>? |
| ------ | ------------------------------------------ |
| 参数   |                                            |
| 返回值 | 读写器版本 |
| 说明 |                                                      |

- 读写器版本

```kotlin
class VersionData(
    // 固件信息
    var firmdata: String = "",
    // boot 版本
    var bootver: String = "",
    // 硬件版本
    var hardver: String = "",
    // 固件版本
    var firmver: String = ""
) 
```

## 获取过滤

| 方法   | getFilter(): ParamsBackData<FilterBean>? |
| ------ | ---------------------------------------- |
| 参数   |                                          |
| 返回值 | 过滤参数 |
| 说明 |                                                      |

## 设置过滤

| 方法   | setFilter(sdt: String, bank: Int, data: String, suit: Boolean): Int? |
| ------ | ------------------------------------------------------------ |
| 参数   |                                                              |
| 返回值 | 0成功，非0失败 |
| 说明 | 设置后会一直生效，除非清空过滤 |

- 过滤参数

```kotlin
class FilterBean(
    // 过滤数据区域1,2,3，分别是EPC、TID、User
    var bank: Int = 0,
    // 过滤数据起始地址
    var startaddr: Int = 0,
    // 过滤数据长度
    var flen: Int = 0,
    // 过滤数据
    var fdata: ByteArray = ByteArray(255),
    // 是否匹配
    var isInvert: Int = 0
) 
```

## 清空过滤

| 方法   | setFilterNull(): Int? |
| ------ | --------------------- |
| 参数   |                       |
| 返回值 | 0成功，非0失败 |
| 说明 |                                                      |

## 设置蜂鸣器使能

| 方法   | setBZEnable(enable: Boolean) |
| ------ | ---------------------------- |
| 参数   | 是否启用蜂鸣器 |
| 返回值 |                              |
| 说明 | 不是指让蜂鸣器响，而是指蜂鸣器被调用响的接口后能不能响 |

## 获取蜂鸣器使能

| 方法   | getBZEnable(): Boolean? |
| ------ | ----------------------- |
| 参数   |                         |
| 返回值 | 蜂鸣器是否处于可以使用状态 |
| 说明 | 处于不可使用状态时，仍可以调用蜂鸣器响的接口，但实际不会响 |

## 蜂鸣器响

| 方法   | startBZ() |
| ------ | --------- |
| 参数   |           |
| 返回值 |           |
| 说明 | 蜂鸣器长鸣，超过15秒就会自动停 |

## 蜂鸣器不响

| 方法   | stopBZ() |
| ------ | -------- |
| 参数   |          |
| 返回值 |          |
| 说明 | 停止蜂鸣器长鸣 |

## 寻卡时蜂鸣器开始响

| 方法   | startBZForSearching() |
| ------ | --------------------- |
| 参数   |                       |
| 返回值 |                       |
| 说明 | 在连续寻卡时调用此接口，每秒响一次 |

## 寻卡时蜂鸣器停止响

| 方法   | stopBZForSearching() |
| ------ | -------------------- |
| 参数   |                      |
| 返回值 |                      |
| 说明 | 停止连续寻卡时调用此接口，停止蜂鸣器响 |

## 寻卡时绿灯开始闪

| 方法   | startBling() |
| ------ | ------------ |
| 参数   |              |
| 返回值 |              |
| 说明 | 在连续寻卡时调用此接口，绿灯闪烁 |

## 寻卡时绿灯停止闪

| 方法   | stopBling() |
| ------ | ----------- |
| 参数   |             |
| 返回值 |             |
| 说明 | 停止连续寻卡时调用此接口，绿灯常亮 |

## 设置LED灯是否亮

| 方法   | setLed(@IntRange(from = 0, to = 3) port: Int, enable: Boolean) |
| ------ | ------------------------------------------------------------ |
| 参数   | port:0红灯，1绿灯，2蓝灯，3粉灯<br />enable:true亮 |
| 返回值 |                                                              |
| 说明 | 一般不用调用此接口 |

## 封装类

- ParamsBackData<T>

```kotlin
class ParamsBackData<T>(
    // 操作是否成功0表示成功，成功再去取data
    var err: Int = 0,
    // 获取的结果
    var data: T? = null,
    private var classType: Class<out T?>? = data?.run { this::class.java }
)
```

