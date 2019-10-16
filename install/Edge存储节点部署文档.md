### Edge存储节点部署



#### 一、依赖环境

- 支持ubuntu、mac OS、 windows操作系统

- Go 1.12.2 及以上版本
- 大硬盘+高带宽型服务器 （1T SSD + 16G RAM + 100M带宽）



#### 二、安装

从代码库Release处可以下载最新的Edge, 直接使用命令行启动



#### 三、使用源代码进行编译

Edge节点可使用Makefile进行编译。执行如下命令

```shell
$ make
```

编译完成之后会在当前目录生成`edge` 可执行程序



#### 四、配置文件

edge节点的配置文件为`config.json`， 具体含义

```json
{
  "Base": {
    "BaseDir": ".",  // 工作目录
    "NetworkId": 1565267317,	 // 网络ID， 保留字段
    "PortBase": 10000,	// 端口起点
    "LogLevel": 1,			// 日志等级
    "LogPath": "./EdgeLog/",	// 日志文件目录
    "LocalRpcPortOffset": 138,	// 本地RPC端口offset, 实际端口为 base+offset  即 10138
    "EnableLocalRpc": true,			// 是否开启本地RPC服务
    "HttpRestPortOffset": 135,	// 本地Rest端口offset
    "HttpCertPath": "",					// https证书路径	
    "HttpKeyPath": "",					// https 私钥路径
    "RestEnable": true,					// 是否开启本地Rest服务
    "ChannelProtocol": "tcp",		// channel网络协议
    "ChannelPortOffset": 3001,	// channel端口offset
    "ChannelClientType": "rpc",	// channel调用全节点接口方式
		"ChannelRevealTimeout": "200", // Lock的超时时间
		"ChannelSettleTimeout": "500", // 双方非合作关闭通道时，从关闭通道到结算的结算时间
    "BlockDelay": "",  				// LockExpire允许落后的块数，目前有默认配置
    "DBPath": "./DB",					// 业务逻辑层数据库目录
    "PpListenAddr": "",				// 保留字段	
    "PpBootstrap": {},				// 保留字段
    "ChainRestAddrs": ["http://221.179.156.57:10334"],  // 主链restful api地址
    "ChainRpcAddrs": ["http://221.179.156.57:10336"],		// 主链rpc api 地址
    "NATProxyServerAddrs": "tcp://40.73.103.72:6007",
    "DspProtocol": "tcp",			// dsp网络协议
    "DspPortOffset": 4001,		// dsp 端口offset
    "TrackerPortOffset": 0,		// 保留字段
    "DnsNodeMaxNum": 100,			// DNS节点最大数目
    "AutoSetupDNSEnable": false,	// 是否允许自动连接DNS节点并创建通道
    "SeedInterval": 10,		// 自身做种时间间隔
    "DnsChannelDeposit": 1000000000,		// 自动连接DNS节点时，自动充值的通道金额
    "DNSWalletAddrs": [],  		// 指定DNS的钱包地址
    "WalletPwd": "pwd",				// 钱包密码
    "WalletDir": "./wallet.dat"	// 钱包文件
  },
  "Fs": {
    "FsRepoRoot": "./FS", // fs目录路径
    "FsFileRoot": "./Downloads",  // 下载的文件路径
    "FsType": 1,		// 1: 存储端
    "FsGCPeriod": "1h", // 存储节点GC时间
		"EnableBackup": true  // 存储节点启动Backup服务	
  },
  "Bootstraps": [] // 保留字段
}
```



| 配置项     |                                                              |
| ---------- | ------------------------------------------------------------ |
| 可自定义   | PublicIP (公网IP)<br>LogLevel (日志等级)<br>BlockConfirm (等待区块确认的区块数)<br>BlockDelay (等待区块确认的区块数)<br>ChainRestAddrs (全节点Rest地址)<br>ChainRpcAddrs (全节点RPC地址)<br>NATProxyServerAddrs (NAT代理节点地址)<br>DnsNodeMaxNum (连接的最多的DNS数量)<br>SeedInterval (提交下载的文件信息时间间隔)<br>Trackers (连接的Tracker服务器地址)<br>FsFileRoot (下载文件所在路径)<br>FsGCPeriod (GC回收时间间隔)<br>EnableBackup (是否开启备份任务线程，客户端无此配置) |
| 建议不改动 | BaseDir (文档目录)<br>LogPath (日志文件夹名称) <br>PortBase (P2P基端口) <br>***PortOffset (相关服务端口偏移)<br>DBPath (数据库路径)<br>ChannelClientType (通道调用主链接口方式) <br>WalletDir (钱包所在路径) |
| 不能修改   | BlockTime (出块时间) <br>NetworkId (网络ID) <br>ChannelProtocol (通道网络协议) <br>ChannelRevealTimeout(开关通道设置的超时) <br>ChannelSettleTimeout (通道结算的超时) <br>DspProtocol (dsp网络协议) <br/>FsType (FS底层存储方式) |



#### 五、启动Edge



```shell
$ nohup ./edge --config=. >/dev/null 2>nohup.log &
```



命令解释：

--config:  指定Edge配置文件所在的目录为当前目录

