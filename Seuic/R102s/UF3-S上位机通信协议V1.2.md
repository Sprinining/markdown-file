# UF3-S上位机通信协议

## 一、数据传输格式

- 数据结尾 \$
- 传输的数据格式： json + $
- json格式（编码utf8）：

```json
{
	"code": 1011,
	"data": {},
	"rtCode": 0,
	"rtMsg": "错误信息"
}
```

## 二、连接方式

1、串口连接

 根据串口地址（固定：/dev/ttyHS1）、波特率连接（默认：115200）、打开方式（固定：可读可写）

2、TCP/IP网口连接

根据IP、端口号（默认：8088）连接

3、广播

```java
        Intent intent = new Intent();
        intent.setAction("com.seuic.uhf.service_settings");
        // 输出功率 1~33
        intent.putExtra("power", 1);
        // 0~3(S0~S3)
        intent.putExtra("session", 1);
        // 1~4(P1~P4)
        intent.putExtra("profile", 1);
        // 0:A 1:B
        intent.putExtra("target", 1);
        // FCC ETSI China1 China2
        intent.putExtra("region", "China1");
        // GPIO 3个字节 0:in 1:out 0:down 1:up（gpio号，输入输出，拉低拉高 例如：210）
        intent.putExtra("gpio_mode", new byte[]{1, 2, 1});
        /**
         * 过滤
         * <p>
         * byte[]
         * TagFilter {
         * public int bank;  //一个字节过滤数据区域1,2,3，分别是EPC、TID、User
         * public int startaddr;  //一个字节过滤数据起始地址
         * public int flen;  //一个字节过滤数据长度
         * public int isInvert;  //一个字节过滤数据是否匹配
         * public byte[] fdata;  //过滤数据
         * }
         */
        intent.putExtra("filter_data", new byte[]{1, 32, 4, 1, 48, 49, 50, 51});
        /**
         * 启用附加数据
         * <p>
         * byte[]
         * EmbededData {
         * public int bank;  //一个字节附加数据区域0,2,3，分别是PassWord、TID、User
         * public int startaddr;  //一个字节附加数据起始地址
         * public int bytecnt;  //一个字节附加数据长度
         * public byte[] accesspwd;  //四个字节访问密码，默认无密码00000000
         * }
         */
        intent.putExtra("embeded_data", new byte[]{1, 32, 4, 48, 48, 48, 48});
        sendBroadcast(intent);
```



## 三、通信接口

### 1000.心跳包标识

- 客户端可以主动发送该指令，收到返回的心跳包；或者，客户端10秒未发送指令，服务器每隔10秒发送一次心跳。

- send

```json
{
  "code": 1000
}
```

- receive

```json
{
  "code": 1000,
  "data": {
    "param": "Success"
  },
  "rtCode": 0
}
```

### 1001.标签上报

- LogBaseEpcInfo

```java
public class LogBaseEpcInfo {
    // 天线编号
    private int antId;
    // 读取标签时间
    private Date readTime;
    // 附加数据
    private String additionDataHexStr;
    private byte[] additionDataBytes;
    // PC 值
    private String pc;
    // 16 进制 EPC 字符串
    private String epc;
    // EPC 字节数组
    private byte[] bEpc;
    //信号强度
    private int rssi;
}
```

- 此方法由1018号指令触发，并不是给客户端直接调用的。客户端通过1018号指令配置参数开始寻卡，服务器通过1001号指令返回标签list。

- receive

```json
{
  "code": 1001,
  "data": [
    {
      "additionDataBytes": "",
      "additionDataHexStr": "",
      "antId": 1,
      "bEpc": "MzP+EQUBAAAAAAAA",
      "epc": "3333FE110501000000000000",
      "pc": "3400",
      "readTime": 1654735569614,
      "rssi": -38
    },
    {
      "additionDataBytes": "",
      "additionDataHexStr": "",
      "antId": 1,
      "bEpc": "VVX+EQYCAAAAAAAA",
      "epc": "5555FE110602000000000000",
      "pc": "3400",
      "readTime": 1654735569620,
      "rssi": -53
    }
  ],
  "rtCode": 0
}
```

### 1005.重启读写器标识

- send

```json
{
  "code": 1005
}
```

- receive

```json
{
  "code": 1005,
  "rtCode": 0
}
```

### 1009.查询软件基带版本

```java
public class MsgAppGetBaseVersion {
    // 基带版本
    private String baseVersions;
    // 设备相关信息
    private String deviceInfo;
} 
```

- send

```json
{
  "code": 1009
}
```

- receive

```json
{
  "code": 1009,
  "data": {
    "baseVersions": "19080.4000.00.03.01.01\nSS.V0.05,19080.4000.00.03.01.01\nSS.V0.05, Linux version 4.9.217 (fengxiaodong@Android-05) (clang version 8.0.12 for Android NDK) #2 SMP PREEMPT Tue Nov 23 11:08:02 CST 2021\n",
    "deviceInfo": "RFID_READER_INFORMATION:, IP:, MASK:, GATEWAY:, MAC:64:f6:bb:79:9f:38, PORT:8088, HOST_SERVER_IP:, HOST_SERVER_PORT:, MODE:SERVER, NET_STATE:NORMAL, DHCP_SW:, MODEL:1"
  },
  "rtCode": 0
}
```

### 1010.查询读写器基本信息

```java
public class MsgAppGetReaderInfo {
    // 读写器序列号
    private String readerSerialNumber;
    // 上电时间
    private Date powerOnTime;
    // 基带编译时间
    private Date baseCompileTime;
    // 应用软件版本（如：“0.1.0.0”）
    private String appVersions;
    // 应用编译时间
    private Date appCompileTime;
    // 操作系统版本
    private String systemVersions;
}
```

- send

```json
{
  "code": 1010
}
```

- receive

```json
{
  "code": 1010,
  "data": {
    "appCompileTime": "Nov 29, 2021 2:26:14 PM",
    "appVersions": "1.0",
    "baseCompileTime": "Nov 23, 2021 5:08:02 PM",
    "powerOnTime": "Nov 23, 2021 12:35:37 PM",
    "readerSerialNumber": "",
    "systemVersions": "eng.fengxi.20211123.132305"
  },
  "rtCode": 0
}
```

### 1011.停止寻卡

- send

```json
{
  "code": 1011
}
```

- receive

```json
{
  "code": 1011,
  "rtCode": 0
}
```

### 1012.配置读写器天线功率标识

```java
public class MsgBaseSetPower {
    // 读写器对应天线功率(key：天线索引号，value：天线功率值)
    private Hashtable<Integer, Integer> dicPower;
}
```

- send

```json
{
  "code": 1012,
  "data": {
    "dicPower": {
      "8": 20,
      "7": 20,
      "6": 20,
      "5": 20,
      "4": 20,
      "3": 20,
      "2": 20,
      "1": 20
    }
  }
}
```

- receive

```json
{
  "code": 1012,
  "rtCode": 0
}
```

### 1013.查询读写器天线功率标识

```java
public class MsgBaseGetPower {
    // 读写器对应天线功率(key：天线索引号，value：天线功率值)
    private Hashtable<Integer, Integer> dicPower;
}
```

- send

```json
{
  "code": 1013
}
```

- receive

```json
{
  "code": 1013,
  "data": {
    "dicPower": {
      "8": 20,
      "7": 20,
      "6": 20,
      "5": 20,
      "4": 20,
      "3": 20,
      "2": 20,
      "1": 20
    }
  },
  "rtCode": 0
}
```

### 1018.寻卡

- 过滤参数不传或者enable为false时，清除过滤效果

```java
public class MsgBaseInventoryEpc {
    // 天线端口(使用天线常量,详见快速上手)
    private int antennaEnable;
    // 连续/单次读取 (0: 单次读取模式，读写器尽在各个使能的天线上进行
    // 一轮读卡操作便结束读卡操作并自动进入空闲状态; 1: 连续读取模式，读写器一直进行
    // 读卡操作直到读写器收到停止指令后结束读卡)
    private int inventoryMode;
    // 选择读取参数（可选）
    private ParamEpcFilter filter;

    public static class ParamEpcFilter {
    	// 要匹配的数据区(1，EPC 区；2，TID 区；3，用户数据区)
	    private Integer bank;
    	// 匹配数据起始地址(字节)
	    private Integer byteStart;
    	// 需要匹配的数据内容(16进制 )
	    private String hexData;
    	// 是否匹配
 	   	private Boolean suit;
    	// 是否启用
	    private Boolean enable;
    }
}
```

- send

```json
{
  "code": 1018,
  "data": {
    "antennaEnable": 1,
    "inventoryMode": 1
  }
}
```

- receive

```json
{
  "code": 1018,
  "rtCode": 0
}
```

### 1019.写卡

```java
public class MsgBaseWriteEpc {
    // 天线端口，天线是否启用的byte值，1为启用，0为不启用，int从右到左的八位对应天线从一到八，如0b00000111表示启用第1,2,3根天线
    private int antennaEnable;
    // 待写入的标签数据区(0，保留区；1，EPC 区；2，TID 区；3，用户数据区)
    private int area;
    // 待写入标签数据区的字起始地址
    private int start;
    // 待写入的数据内容(16进制)
    private String hexWriteData;
	// 过滤的EPC
    private String EPC;
    // 访问密码
    private String hexPassword;
}
```

- send

```json
{
  "code": 1019,
  "data": {
    "antennaEnable": 1,
    "area": 1,
    "start": 3,
    "hexWriteData": "BBCC",
    "bwriteData": "",
    "EPC": "777766662F0135065C644CF0",
    "hexPassword": "00000000"
  }
}
```

- receive

```json
{
  "code": 1019,
  "rtCode": 0
}
```

### 1020.锁卡

- 保留区前两个字是销毁密码，后两个字是访问密码
- 锁卡前要先修改访问密码（不能为00000000）

- 锁定保留区时，读写都需要使用修改后的密码
- 锁定其他三个区时，可以用00000000去读，但是如果要写（TID区不可写），必须用修改后的密码

```java
public class MsgBaseLockEpc  {
    // 天线端口
    private int antennaEnable;
    // 待锁定的标签数据区(0，保留区；1，EPC 区；2，TID 区；3， 用户数据区)
    private int area;
    // 锁操作类型(0，解锁；1，锁定；2，永久锁定)
    private int mode;
	// 过滤的EPC
    private String EPC;
    // 访问密码
    private String hexPassword;
}
```

- send

```json
{
  "code": 1020,
  "data": {
    "antennaEnable": 1,
    "area": 1,
    "mode" : 1,
    "EPC": "777766662F0135065C644CF0",
    "hexPassword": "88888888"
  }
}
```

- receive

```json
{
  "code": 1020,
  "rtCode": 0
}
```

### 1021.销卡

```java
public class MsgBaseDestoryEpc {
    // 天线端口
    private int antennaEnable;
    // 销毁密码
    private String hexPassword;
	// 过滤的EPC
    private String EPC;
}
```

- send

```json
{
  "code": 1021,
  "data": {
    "antennaEnable": 1,
    "EPC": "777766662F0135065C644CF0",
    "hexPassword": "88888888"
  }
}
```

- receive

```json
{
  "code": 1021,
  "rtCode": 0
}
```

### 1029.获取ip

- send

```json
{
  "code": 1029
}
```

- receive

```json
{
  "code": 1029,
  "data": {
    "dNS": [
      "218.2.135.1",
      "114.114.114.114"
    ],
    "gATEWAY": "192.168.40.1",
    "iP": [
      "192.168.40.255",
      "fe80::20a:f5ff:fe2d:e758"
    ],
    "mAC": "00:0a:f5:2d:e7:58",
    "nETMASK": "255.255.254.0",
    "networkType": "NETWORK_WIFI",
    "tcpPort": 8088
  },
  "rtCode": 0
}
```

### 1030.设置ipv4

- 设置后设备会自动重启
- 设置DHCP时，只需传mode为“DHCP”，其他都为“”

```java
public class MsgSetIPv4 {
    // STATIC DHCP UNASSIGNED
    String mode;
    String ipAddress;
    String netmask;
    String gateway;
    String dns1;
    String dns2;
}
```

- send

```json
{
  "code": 1030,
  "data": {
    "mode": "STATIC",
    "ipAddress": "192.168.40.219",
    "netmask": "255.255.255.0",
    "gateway": "192.168.40.1",
    "dns1": "",
    "dns2": ""
  }
}
```

- receive

```json
{
  "code": 1030,
  "rtCode": -1,
  "rtMsg": "设置ipv4失败"
}
```

### 1031.设置ipv6

- 设置后设备会自动重启

- send

```json
{
  "code": 1031,
  "data":"fe80::cda0:ef9d:bad9:54ff/64"
}
```

- receive

```json
{
  "code": 1031,
  "rtCode": 0
}
```

### 2000.一键获取

```java
public class MsgAllParameters {
    // 只读
    // 读写器版本号
    private ReadVersion readVersion;
    // 连接中的天线端口
    private ConnectedAntennas connectedAntennas;
    // 获取gpi
    private Integer[] gpi;
    // 获取温度
    private Integer temperature;
    // 网络
    private NetworkStateBean networkStateBean;

    // 只写
    // 设置gpo
    private ArrayList<Gpo> gpoArrayList;

    // 可读可写
    // 频段
    private String region;
    // 功率
    private ArrayList<AntennaPower> antennaPowers;
    // session
    private Integer session;
    // profile
    private Integer profile;
    // target
    private Integer target;
    // Q值
    private Integer qValue;
    // 附加数据
    private AdditionalData additionalData;
    // 设置过滤，设置完后会一直生效
    private ParamEpcFilter filter;
    // 蜂鸣器使能
    private Boolean buzzerEnable;

    public static class ReadVersion {
        // boot版本
        private String bootVersion;
        // 固件信息
        private String firmwareData;
        // 固件版本
        private String firmwareVersion;
        // 硬件版本
        private String hardwareVersion;
    }

    public static class Gpo {
        // 1~4
        private Integer gpo;
        // 0低电平 1高电平
        private Integer value;
    }

    public static class AdditionalData {
	    // TID2 USER3
        private Integer bank;
        // 起始地址(字节)
        private Integer startAddress;
        // 字节数(偶数个字节)
        private Integer byteCount;
        // 访问密码
        private String password;
        // 是否启用
        private Boolean enable;
    }

    public static class ParamEpcFilter {
    	// 要匹配的数据区(1，EPC 区；2，TID 区；3，用户数据区)
	    private Integer bank;
    	// 匹配数据起始地址(字节)
	    private Integer byteStart;
    	// 需要匹配的数据内容(16进制 )
	    private String hexData;
    	// 是否匹配
 	   	private Boolean suit;
    	// 是否启用
	    private Boolean enable;
    }

    public static class ConnectedAntennas {
		// 连接中的天线数量
        private Integer antennaCount;
        // 连接中的天线数组
        private int[] antennas;
    }

    public static class AntennaPower {
        // 1~8
        private Integer antennaId;
        // 1~33
        private Short readPower;
        // 1~33
        private Short writePower;
    }
}
```

- send

```json
{
  "code": 2000
}
```

- reveive

```json
{
  "code": 2000,
  "data": {
    "additionalData": {
      "bank": 0,
      "byteCount": 0,
      "enable": false,
      "password": "00000000",
      "startAddress": 0
    },
    "antennaPowers": [
      {
        "antennaId": 1,
        "readPower": 33,
        "writePower": 33
      },
      {
        "antennaId": 2,
        "readPower": 33,
        "writePower": 33
      },
      {
        "antennaId": 3,
        "readPower": 33,
        "writePower": 33
      },
      {
        "antennaId": 4,
        "readPower": 33,
        "writePower": 33
      },
      {
        "antennaId": 5,
        "readPower": 33,
        "writePower": 33
      },
      {
        "antennaId": 6,
        "readPower": 33,
        "writePower": 33
      },
      {
        "antennaId": 7,
        "readPower": 33,
        "writePower": 33
      },
      {
        "antennaId": 8,
        "readPower": 33,
        "writePower": 33
      }
    ],
    "buzzerEnable": false,
    "connectedAntennas": {
      "antennaCount": 1,
      "antennas": [
        1
      ]
    },
    "filter": {
      "bank": 0,
      "byteStart": 0,
      "enable": false,
      "hexData": "",
      "suit": false
    },
    "gpi": [
      1,
      1,
      1,
      1
    ],
    "profile": 1,
    "qValue": 4,
    "readVersion": {
      "bootVersion": "15.01.04.00",
      "firmwareData": "20.22.03.15",
      "firmwareVersion": "22.03.15.00",
      "hardwareVersion": "A1.00.02.01"
    },
    "region": "FCC",
    "session": 0,
    "target": 0,
    "temperature": 37
  },
  "rtCode": 0,
  "rtMsg": ""
}
```

### 2001.一键设置

- 所有参数都为可选参数
- send

```json
{
  "code": 2001,
  "data": {
    "antennaPowers": [
      {
        "antennaId": 1,
        "readPower": 31,
        "writePower": 31
      },
      {
        "antennaId": 2,
        "readPower": 31,
        "writePower": 31
      },
      {
        "antennaId": 3,
        "readPower": 31,
        "writePower": 31
      },
      {
        "antennaId": 4,
        "readPower": 31,
        "writePower": 31
      },
      {
        "antennaId": 5,
        "readPower": 31,
        "writePower": 31
      },
      {
        "antennaId": 6,
        "readPower": 31,
        "writePower": 31
      },
      {
        "antennaId": 7,
        "readPower": 31,
        "writePower": 31
      },
      {
        "antennaId": 8,
        "readPower": 31,
        "writePower": 31
      }
    ],
    "profile": 2,
    "qValue": -1,
    "region": "FCC",
    "session": 0,
    "target": 0,
    "additionalData": {
      "bank": 3,
      "startAddress": "0",
      "byteCount": "2",
      "password": "00000000",
      "enable": true
    },
    "gpoArrayList": [
      {
        "gpo": 1,
        "value": 1
      },
      {
        "gpo": 2,
        "value": 1
      },
      {
        "gpo": 3,
        "value": 1
      },
      {
        "gpo": 4,
        "value": 1
      }
    ],
    "filter": {
      "bank": 1,
      "byteStart": 4,
      "hexData": "44",
      "suit": true,
      "enable": true
    },
    "buzzerEnable": true
  }
}
```

- receive

```json
{
  "code": 2001,
  "rtCode": 0,
  "rtMsg": ""
}
```

### 2002.恢复出厂

- send

```json
{
  "code": 2002
}
```

- reveive

```json
{
  "code": 2002,
  "rtCode": 0,
  "rtMsg": "恢复出厂参数: region=FCC power=33 session=0 profile=1 target=0 q=4, 过滤和附加数据不启用, 蜂鸣器启用, gpo全为低电平"
}
```

### 2003.获取自动模式参数

- send

```json
{
  "code": 2003
}
```

- reveive

```json
{
  "code": 2003,
  "data": {
    "autoMode": 0,
    "ip": "192.168.1.1",
    "output": 0,
    "port": "8080",
    "trigger": 0,
    "gpiId": 1
  }
}
```

### 2004.设置自动模式参数

- 设置完后，设备会尝试连接设置的ip和端口

```java
public class AutoModeBean {
    // 自动模式 0连续 1触发
    private int autoMode;
    // PC服务IP地址
    private String ip;
    // 输出模式  0串口 1socket
    private int output;
    // PC服务端口
    private String port;
    // 触发方式 0低电平 1高电平
    private int trigger;
    // 触发的电平id
    private int gpiId;
}
```

- send

```json
{
  "code": 2004,
  "data": {
    "autoMode": 0,
    "ip": "192.168.1.1",
    "output": 0,
    "port": "9999",
    "trigger": 0,
    "gpiId": 1
  }
}
```

- reveive

```json
{
  "code": 2004,
  "rtCode": 0
}
```

### 2005.自动模式开始

```java
// 天线端口，天线是否启用的byte值，1为启用，0为不启用，int从右到左的八位对应天线从一到八，如0b00000111表示启用第1,2,3根天线
private int antennaEnable;
```

- send

```json
{
  "code": 2005,
  "data": {
    "antennaEnable": 1
  }
}
```

- reveive

```json
{
  "code": 2005,
  "rtCode": 0
}
```

### 2006.自动模式结束

- send

```json
{
  "code": 2006
}
```

- reveive

```json
{
  "code": 2006,
  "rtCode": 0
}
```

### 2008.读卡

```java
public class MsgReadTag {
    // 天线端口
    private int antennaEnable;
    // 区(0，保留区；1，EPC 区；2，TID 区；3，用户数据区)
    private int bank;
    // 块数（一块一个字，16位）
    private int blockCount;
    // 起始地址（字为单位）
    private int startAddress;
    // 访问密码
    private String hexPassword;
    // 过滤的EPC
    private String EPC;
}
```

- send：

```json
{
  "code": 2008,
  "data": {
    "antennaEnable": 1,
    "bank": 1,
    "blockCount": 6,
    "startAddress": 2,
    "hexPassword": "00000000",
    "EPC": "3333FE110501000000000000"
  }
}
```

- receive：

```json
{
  "code": 2008,
  "data": "3333FE110501000000000000",
  "rtCode": 0
}
```

### 2009.设置ip

```java
public class MsgSetIP {
    private String mode;
    private String ipv4Address;
    private String netmask;
    private String gateway;
    private String dns1;
    private String dns2;
    private String ipv6Address;
}
```

- send

```json
{
  "code": 2009,
  "data": {
    "mode": "STATIC",
    "ipv4Address": "192.168.40.219",
    "netmask": "255.255.255.0",
    "gateway": "192.168.40.1",
    "dns1": "",
    "dns2": "",
    "ipv6Address": "fe80::cda0:ef9d:bad9:54ff/64"
  }
}
```

- receive

```JSON
{
  "code": 2009,
  "rtCode": 0
}
```

