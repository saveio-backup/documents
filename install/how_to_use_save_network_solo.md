## SAVE Network 本地开发环境配置(SOLO共识)

> 本配置用于说明如何在本地环境搭建SAVE网络，为了方便测试，主链采用solo共识。同时为了简化网络拓扑，不引入NAT代理节点。本文档是在Mac OS系统下运行, 其他操作系统可能在个别命令会有差别。



### 1.代码编译

#### 运行环境

* go version go1.14.1
* 暂不使用go mod, 命令: `$ go env -w GO111MODULE="off"`



#### 各代码版本

| 仓库          | CommitId                                 | 说明           |
| ------------- | ---------------------------------------- | -------------- |
| themis        | 6027567764705309d5f3d085e3c24cdc370af38d | 主链           |
| themis-go-sdk | 3948a9d1fddb3af1443fd09c7ff13d81824c1ca6 | 主链RPC SDK    |
| carrier       | f6c17bc525bcb6fcd521909df5015c16cd3a30b3 | P2P网络        |
| scan          | 90f7246ec1b91949b7e84b1bbafa48afc2448475 | DNS            |
| pylons        | cb128d0a122c85633eb43e7815fcefb512b4c2dd | Layer2支付通道 |
| max           | b61f97e8140f5c4f5ceb3f3d4daa428a7e66c682 | FS存储         |
| edge          | 268d05a5ef5f9f5975b5b7fbafa7465f4ae6d15d | DSP客户端      |
| dsp-go-sdk    | caf2508b922289a91a184a25d4ed8987f8462bc7 | DSP SDK        |
| porter        | 1ffe37af183b8161f93ed83f86163e1eebbe1239 | NAT代理        |



#### 编译

核心代码克隆完成之后，还需要克隆其他依赖代码。由于没有使用`go mod` ，所以需要依次安装各依赖。可以在内网编译机器直接下载。

内网机器 10.0.1.128

依赖所在位置 `~/publish_build/golang/src`

编译参考各项目的说明，基本是`make`方式编译。

### 2.主链配置

> 注意： 这里使用SOLO模式，如果需要测试其他共识，请按照其他共识的方式启动。此说明文档的代码路径均在 `~/go/src` 下。各节点钱包密码均为 `pwd`。为了直接采用开发模式，这里使用`go run`的方式直接启动节点。

#### 创建钱包

```shell
$ cd ~/go/src/github.com/saveio/themis
$ go run main.go account add -d
```

```
输出:

Index:1
Label:
Address:AaK1g8jcH8zRMmN8F8oDaVteMXpit42pjh
Id(For PoC):7163600173146005825
Public key:03977eda4ea3db5757ad1977eaca6172ab3f197c19f06d2361769a6621941c48e1
Signature scheme:SHA256withECDSA
Create account successfully.
```



#### 启动

```shell
$ go run main.go --testmode -p pwd
```



#### 查看余额



另起新的窗口

```shell
$ go run main.go asset balance 1
```

```
输出：

BalanceOf:AaK1g8jcH8zRMmN8F8oDaVteMXpit42pjh
  USDT:100000000
```





### 3.DNS节点配置

> 由于有数据库文件产生，这里需要先编译成可执行文件



#### 创建钱包

```shell
$ cd ~/go/src/github.com/saveio/scan
$ make
$ ./scan accound add -d
```

```
输出:

Index:1
Label:
Address:AYFzNEzNH4KCGw7XS9N9C9oD6VbYobK5f7
Id(For PoC):5654908485478275393
Public key:03ce349d0f6d6913e3a22f5191a7c751cbeedda9ab42ceed217ea8d083cc00dba5
Signature scheme:SHA256withECDSA
Create account successfully.
zhijies-MacBook-Pro:scan zhijie$ pwd
/Users/zhijie/go/src/github.com/saveio/scan/bin/scan
```



#### 获取测试币

到`themis`节点处给 `dns` 节点转账

```shell
$ cd ~/go/src/github.com/saveio/themis
$ go run main.go asset transfer --from=AaK1g8jcH8zRMmN8F8oDaVteMXpit42pjh --to=AYFzNEzNH4KCGw7XS9N9C9oD6VbYobK5f7 --amount=100000
```

```
输出：

Transfer USDT
  From:AaK1g8jcH8zRMmN8F8oDaVteMXpit42pjh
  To:AYFzNEzNH4KCGw7XS9N9C9oD6VbYobK5f7
  Amount:100000
  TxHash:edf053f2855c3890f3ca6f67952e21346e25e6ad66430fbf46fcf4bea7792e9c

Tip:
  Using './themis info status edf053f2855c3890f3ca6f67952e21346e25e6ad66430fbf46fcf4bea7792e9c' to query transaction status.
```



等待1个区块高度，查询是否到账

```shell
$ go run main.go asset balance AYFzNEzNH4KCGw7XS9N9C9oD6VbYobK5f7
```

```
输出:

BalanceOf:AYFzNEzNH4KCGw7XS9N9C9oD6VbYobK5f7
  USDT:100000
```



#### 配置文件

```shell
$ cd ~/go/src/github.com/saveio/scan
$ vim scan_config.json
```



```json
{
        "Base": {
                "DumpMemory": false,
                "BaseDir": ".",
                "LogLevel": 0,
                "PublicIP": "127.0.0.1",
          			"IntranetIP": "127.0.0.1",
                "PortBase": 10000,
                "LocalRpcPortOffset": 937,
                "EnableLocalRpc": false,
                "JsonRpcPortOffset": 936,
                "EnableJsonrpc": true,
                "HttpRestPortOffset": 10935,
                "HttpCertPath": "",
                "HttpKeyPath": "",
                "EnableRest": true,
                "ChainRpcAddr": "http://127.0.0.1:20336",
                "ChainRestAddr": "http://127.0.0.1:20334",
                "DisableChain": true,
                "ChannelNetworkId": 1565267317,
                "ChannelPortOffset": 938,
                "ChannelProtocol": "tcp",
                "ChannelClientType": "rpc",
                "ChannelRevealTimeout": "20",
                "ChannelSettleTimeout": "50",
                "ChannelDBPath": "./ChannelDB",
                "DnsNetworkId": 1565511150,
                "DnsProtocol": "tcp",
                "DnsPortOffset": 939,
                "DnsGovernDeposit": 10000000000,
                "DnsChannelDeposit": 1000000000,
                "AutoSetupDNSRegisterEnable": true,
         				"TrackerNetworkId": 1567481543,
                "TrackerProtocol": "tcp",
                "TrackerPortOffset": 6369,
                "TrackerPeerValidDuration": "-24h",
                "NATProxyServerAddr": "",
                "DBPath": "DB",
                "DnsNodeMaxNum": 100,
                "SeedInterval": 10,
                "IgnoreConnectDNSAddrs": [
                ]
        }
}
```



> 配置文件说明

| 配置项                     | 说明                                                         | 示例                   |
| -------------------------- | ------------------------------------------------------------ | ---------------------- |
| DumpMemory                 | 是否开启Profile进行性能记录                                  | false                  |
| BaseDir                    | 数据库文件所在目录(windows系统，请修改此项)                                           | .                      |
| LogLevel                   | 日志等级                                                     | 0                      |
| PublicIP                   | 固定外部IP                                                   | 127.0.0.1              |
| IntranetIP                 | 内网IP                                                       | 127.0.0.1              |
| PortBase                   | 各服务端口基准端口，如下面本地端口为 10000+337即10337        | 10000                  |
| LocalRpcPortOffset         | 本地RPC端口                                                  | 337                    |
| EnableLocalRpc             | 是否开启本地RPC                                              | false                  |
| JsonRpcPortOffset          | JSON-RPC端口                                                 | 336                    |
| EnableJsonrpc              | 是否使用JSON-RPC                                             | true                   |
| HttpRestPortOffset         | REST 端口                                                    | 335                    |
| HttpCertPath               | HTTPS 证书路径                                               | ""                     |
| HttpKeyPath                | HTTPS key 证书路径                                           | ""                     |
| EnableRest                 | 是否开启 REST 服务                                           | true                   |
| ChainRpcAddr               | 主链RPC地址，如果DNS为全节点， RPC地址为本地JSON-RPC地址     | http://127.0.0.1:20336 |
| ChainRestAddr              | 主链REST地址，如果DNS为全节点， REST地址为本地REST地址       | http://127.0.0.1:20334 |
| DisableChain               | 是否启动全节点， 如果关闭全节点功能，上面的主链RPC地址需要配成远端RPC节点的地址 | true                   |
| ChannelNetworkId           | Channel 网络ID                                               | 1565267317             |
| ChannelPortOffset          | Channel网络端口                                              | 338                    |
| ChannelProtocol            | Channel网络协议                                              | tcp                    |
| ChannelClientType          | Channel请求主链方式                                          | rpc                    |
| ChannelRevealTimeout       | Channel Reveal Timeout， 目前建议使用200个区块高度           | 200                    |
| ChannelDBPath              | Channel 数据库路径                                           | ./ChannelDB            |
| DnsNetworkId               | DNS网络ID                                                    | 1565511150             |
| DnsProtocol                | DNS网络协议                                                  | tcp                    |
| DnsPortOffset              | DNS端口                                                      | 339                    |
| DnsGovernDeposit           | DNS治理抵押，如果默认钱包余额充足，会自动抵押；否则，可使用命令手动抵押. | 1000000000             |
| DnsChannelDeposit          | DNS通道抵押                                                  | 1000000000             |
| AutoSetupDNSRegisterEnable | 是否开启DNS自动注册                                          | true                   |
| TrackerNetworkId           | Tracker网络ID                                                | 1567481543             |
| TrackerProtocol            | Tracker协议                                                  | tcp                    |
| TrackerPortOffset          | Tracker端口                                                  | 6369                   |
| TrackerPeerValidDuration   | Tracker文件信息失效时间                                      | -24h                   |
| NATProxyServerAddr         | NAT proxy地址， 目前DNS有固定外部IP, 不使用NAT服务           | ""                     |
| DBPath                     | DNS数据库路径                                                | DB                     |
| DnsNodeMaxNum              | 保留字段                                                     | 100                    |
| SeedInterval               | 保留字段                                                     | 10                     |
| DnsChannelDeposit          | DNS通道抵押的资产数量                                        | 1000000000             |
| AutoSetupDNSRegisterEnable | 是否开启DNS自动注册                                          | true                   |
| AutoSetupDNSChannelsEnable | 是否自动创建通道                                             | false                  |
| InitDeposit                | 保留字段                                                     | 1000000000             |
| ChannelDeposit             | 保留字段                                                     | 1000000                |
| IgnoreConnectDNSAddrs      | 忽略连接其他DNS的钱包地址                                    | []                     |



#### 启动

```shell
$ ./scan -p pwd
```



#### 查询节点是否成功注册（可跳过)

> 如果查询为空，则重启scan节点，重试几次

```shell
$ ./scan dns getRegInfo --all
```

```json
输出

Get dns register info success. Default limit 50
Index 0:
{
   "PeerPubkey": "03ce349d0f6d6913e3a22f5191a7c751cbeedda9ab42ceed217ea8d083cc00dba5",
   "WalletAddress": "AYFzNEzNH4KCGw7XS9N9C9oD6VbYobK5f7",
   "Status": 0,
   "TotalInitPos": 10000000000
}
```



```shell
$ ./scan dns getHostInfo --all
```



```json
输出

Get dns host info success. Default limit 50
Index 0:
{
   "WalletAddr": "AYFzNEzNH4KCGw7XS9N9C9oD6VbYobK5f7",
   "IP": "127.0.0.1",
   "Port": "10938",
   "InitDeposit": 10000000000,
   "PeerPubKey": "03ce349d0f6d6913e3a22f5191a7c751cbeedda9ab42ceed217ea8d083cc00dba5"
}
```





### 4.存储节点配置



> 为了方便测试，创建一个文件夹 `node1`

#### 创建存储节点文件夹

```shell
$ cd ~/go/src/github.com/saveio/edge
$ mkdir node1
```



#### 编译

```shell
$ cd ~/go/src/github.com/saveio/edge
$ make
$ cd node1
$ ln -s ../edge .
```



#### 创建钱包并获取测试币

```shell
$ ./edge account add -d
```

```
输出

Index:1
Label:
Address:AHgGsdkCyxFpy3uV5kxQ9ChHZLy41FmSw2
Public key:024617e072438af2e19929ab3079d474499e9991cda3a52daea4b657ac788a98ee
Signature scheme:SHA256withECDSA
Create account successfully.
```



```shell
$ cd ~/go/src/github.com/saveio/themis
$ go run main.go asset transfer --from=AaK1g8jcH8zRMmN8F8oDaVteMXpit42pjh --to=AHgGsdkCyxFpy3uV5kxQ9ChHZLy41FmSw2 --amount=100000
```



```
输出

Transfer USDT
  From:AaK1g8jcH8zRMmN8F8oDaVteMXpit42pjh
  To:AHgGsdkCyxFpy3uV5kxQ9ChHZLy41FmSw2
  Amount:100000
  TxHash:1acd3137a1ed9bf16fe9b4aa7766d921e3b7e9422a57542c453e02798a6b0b14

Tip:
  Using './themis info status 1acd3137a1ed9bf16fe9b4aa7766d921e3b7e9422a57542c453e02798a6b0b14' to query transaction status.
```



等一个区块高度后，查询是否到账

```shell
$ go run main.go asset balance AHgGsdkCyxFpy3uV5kxQ9ChHZLy41FmSw2
```

```
输出

BalanceOf:AHgGsdkCyxFpy3uV5kxQ9ChHZLy41FmSw2
  USDT:100000
```





#### 配置文件

```shell
$ cd ~/go/src/github.com/saveio/edge/node1
$ vim config.json
```



```json
{
	"Base": {
		"BaseDir": ".",
		"LogPath": "./EdgeFsLog/",
		"NetworkId": 1565267317,
		"ChainId": "0",
		"BlockTime": 6,
		"PublicIP": "127.0.0.1",
		"IntranetIP": "127.0.0.1",
		"PortBase": 10000,
		"LogLevel": 1,
		"LocalRpcPortOffset": 138,
		"EnableLocalRpc": true,
		"JsonRpcPortOffset": 136,
		"EnableJsonRpc": true,
		"HttpRestPortOffset": 135,
		"HttpCertPath": "",
		"HttpKeyPath": "",
		"RestEnable": true,
		"WsPortOffset": 139,
		"WsCertPath": "",
		"WsKeyPath": "",
		"ChannelProtocol": "tcp",
		"ChannelPortOffset": 132,
		"DBPath": "./DB",
		"PpListenAddr": "",
		"PpBootstrap": null,
		"ChainRestAddrs": [
			"http://127.0.0.1:20334"
		],
		"ChainRpcAddrs": [
			"http://127.0.0.1:20336"
		],
		"NATProxyServerAddrs": "",
		"DspProtocol": "tcp",
		"DspPortOffset": 131,
		"TrackerPortOffset": 137,
		"TrackerProtocol":"tcp",
		"WalletDir": "./keystore.dat",
		"ProfilePortOffset":133
	},
	"Dsp": {
		"ChannelClientType": "rpc",
		"ChannelRevealTimeout": "20",
		"ChannelSettleTimeout": "50",
		"BlockDelay": "3",
		"DnsNodeMaxNum": 100,
		"SeedInterval": 600,
		"DnsChannelDeposit": 1000000000,
		"Trackers": [],
		"DNSWalletAddrs": [
		],
		"AutoSetupDNSEnable": false,
	  "MaxUploadTask": 1000,
	  "MaxDownloadTask": 100,
	  "MaxShareTask": 200
	},
	"Fs": {
		"FsRepoRoot": "./FS",
		"FsFileRoot": "./Downloads",
		"FsType": 1,
		"EnableBackup": true,
		"FsGCPeriod": "24h"
	},
	"Bootstraps": []
}
```



#### 启动存储节点

```
$ ./edge --config=config.json -p pwd
```



#### 注册存储节点信息(1TB)

> 另起窗口

```shell
$ ./edge node register --nodeAddr=tcp://127.0.0.1:10131 --volume=1073741824 --serviceTime=8760
```

```json
输出：

{
   "Tx": "c4d8b3c0c9173ce5801872b3ba417d17b98eade90c2cdb2a10c94189b1429dc7"
}
```



#### 查询是否注册成功

```shell
$ ./edge node query --walletAddr=AHgGsdkCyxFpy3uV5kxQ9ChHZLy41FmSw2
```

```json
输出:

{
   "Info": {
      "NodeAddr": "tcp://127.0.0.1:10131",
      "Pledge": 1.073741824,
      "Profit": 0,
      "RestVol": 1048576,
      "ServiceTime": 8760,
      "Volume": 1048576,
      "WalletAddr": "AHgGsdkCyxFpy3uV5kxQ9ChHZLy41FmSw2"
   }
}
```



#### 创建存储节点的扇区(1TB)

```shell
$ ./edge sector create --sectorId=1 --proveLevel=1 --sectorSize=1073741824
```

```json
输出：

{
   "Tx": "bf2ae6d08bb190b13c028f819e190868752fc686707ffd715440056dfee8d665"
}
```



#### 查询是否注册成功

```shell
$ ./edge sector getsector --sectorId=1
```

```json
输出:

{
   "NodeAddr": "AHgGsdkCyxFpy3uV5kxQ9ChHZLy41FmSw2",
   "SectorID": 1,
   "Size": 1073741824,
   "ProveLevel": 1,
   "FirstProveHeight": 0,
   "NextProveHeight": 0,
   "TotalBlockNum": 0,
   "FileNum": 0,
   "FileList": null
}
```





#### 与DNS节点建立支付通道

```shell
$ ./edge channel open --partnerAddr=AYFzNEzNH4KCGw7XS9N9C9oD6VbYobK5f7 --amount=10000
```

等待几分钟

```json
输出

{
   "Id": "101"
}
```

再查询一次

```shell
$ ./edge channel list
```



```json
输出

{
   "Balance": 10000000000000,
   "BalanceFormat": "10000",
   "Channels": [
      {
         "ChannelId": 101,
         "Balance": 10000000000000,
         "BalanceFormat": "10000",
         "Address": "AYFzNEzNH4KCGw7XS9N9C9oD6VbYobK5f7",
         "HostAddr": "tcp://127.0.0.1:10938",
         "TokenAddr": "AFmseVrdL9f9oyCzZefL9tG6UbvhUMqNMV",
         "IsParticipant1Closer": false,
         "IsParticipant2Closer": false,
         "IsDNS": true,
         "IsOnline": true,
         "Selected": true,
         "CreatedAt": 1606366142
      }
   ]
}
```





### 5.客户端



#### 启动Seeker

```shell
$ open seeker
```



#### 设置为本地开发模式

```shell
$ cd ~/Library/Application Support/seek 
$ vim front-config.json
```

设置`devEdgeEnable` 为 `true`

```json
{"console":false,"devEdgeEnable":true,"maxNumUpload":5,"lang":"en"}
```



#### 创建客户端文件夹

```shell
$ cd ~/go/src/github.com/saveio/edge
$ mkdir client1
```



#### 链接`edge`

```shell
$ cd client1
$ ln -s ../edge .
```



#### 配置文件

```shell
$ cd ~/go/src/github.com/saveio/edge/client1
$ vim config.json
```



```json
{
        "Base": {
                "BaseDir": ".",
                "LogPath": "./EdgeClientLog/",
                "NetworkId": 1565267317,
                "ChainId": "0",
                "BlockTime": 6,
                "PublicIP": "127.0.0.1",
                "IntranetIP": "127.0.0.1",
                "PortBase": 10000,
                "LogLevel": 1,
                "LocalRpcPortOffset": 338,
                "EnableLocalRpc": true,
                "JsonRpcPortOffset": 336,
                "EnableJsonRpc": true,
                "HttpRestPortOffset": 335,
                "HttpCertPath": "",
                "HttpKeyPath": "",
                "RestEnable": true,
                "WsPortOffset": 339,
                "WsCertPath": "",
                "WsKeyPath": "",
                "ChannelProtocol": "tcp",
                "ChannelPortOffset": 332,
                "DBPath": "./DB",
                "PpListenAddr": "",
                "PpBootstrap": null,
                "ChainRestAddrs": [
                        "http://127.0.0.1:20334"
                ],
                "ChainRpcAddrs": [
                        "http://127.0.0.1:20336"
                ],
                "NATProxyServerAddrs": "",
                "DspProtocol": "tcp",
                "DspPortOffset": 331,
                "TrackerPortOffset": 337,
                "TrackerProtocol":"tcp",
                "WalletDir": "./keystore.dat",
                "ProfilePortOffset":333
        },
        "Dsp": {
                "ChannelClientType": "rpc",
                "ChannelRevealTimeout": "20",
                "ChannelSettleTimeout": "50",
                "BlockDelay": "3",
                "DnsNodeMaxNum": 100,
                "SeedInterval": 600,
                "DnsChannelDeposit": 1000000000,
                "Trackers": [],
                "DNSWalletAddrs": [
                ],
                "AutoSetupDNSEnable": false,
          "MaxUploadTask": 1000,
          "MaxDownloadTask": 100,
          "MaxShareTask": 200
        },
        "Fs": {
                "FsRepoRoot": "./FS",
                "FsFileRoot": "./Downloads",
                "FsType": 0,
                "EnableBackup": true,
                "FsGCPeriod": "24h"
        },
        "Bootstraps": []
}
```



#### 创建钱包并获取测试币

> 此操作为命令行操作，也可以使用Seeker操作



```shell
$ ./edge account add -d
```

```json
输出:

Index:1
Label:
Address:APDvqeWQsPh4m9gFZ7ipmet42pRphLpgBJ
Public key:03806b62e8773fe403aec0d93730b30c15276b8a4e7a0cd858861ec09a383b565c
Signature scheme:SHA256withECDSA
Create account successfully.
```



#### 获取测试币(参考之前的步骤)



#### 启动

```shell
$ ./edge --config=config.json -p pwd
```



#### 注册DNS Header 

> 说明：注册DNS Header是为了上传文件时，可以生成URL。任意Edge节点都可以注册，仅需注册一次。 



```shell
$ ./edge dns registerheader --header="oni" --desc="oni" --ttl=100000
```

```json
输出：

{
   "Tx": "a4efd633a4342dd0c1796d52e75a8c1a716a41b845f67fbf348d9ab4e11e8cac"
}
```





#### 启动Seeker

