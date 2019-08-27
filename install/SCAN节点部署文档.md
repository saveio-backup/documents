### SCAN节点部署



#### 一、依赖环境

* 支持ubuntu、mac OS、 windows操作系统

- Go 1.12.2 及以上版本
- 均衡性+高带宽型服务器 （500G SSD + 8G RAM + 100M带宽）



#### 二、安装

从代码库Release处可以下载最新的SCAN, 直接使用命令行启动



#### 三、使用源代码进行编译

SCAN节点可使用Makefile进行编译。执行如下命令

```shell
$ make
```

编译完成之后会在当前目录生成`scan` 可执行程序



#### 四、配置文件

SCAN节点目前有两个配置，一个配置是作为全节点的配置，另一个配置是自身DNS服务的配置

全节点配置文件名字目前为 `chain_config.json` 和主链配置保持一致

```json
{
    "SeedList": [
	"221.179.156.57:10338",
	"221.179.156.57:11338",
	"221.179.156.57:12338",
	"221.179.156.57:13338",
	"221.179.156.57:14338",
	"221.179.156.57:15338",
	"221.179.156.57:16338"
   ],
    "ConsensusType": "vbft",
    "VBFT": {
        "n": 7,
        "c": 2,
        "k": 7,
        "l": 112,
        "block_msg_delay": 1000,
        "hash_msg_delay": 1000,
        "peer_handshake_timeout": 10,
        "max_block_change_view": 3000,
        "admin_ont_id": "did:ont:AMAx993nE6NEqZjwBssUfopxnnvTdob9ij",
        "min_init_stake": 10000000000000,
        "vrf_value": "1c9810aa9822e511d5804a9c4db9dd08497c31087b0daafa34d768a3253441fa20515e2f30f81741102af0ca3cefc4818fef16adb825fbaa8cad78647f3afb590e",
        "vrf_proof": "c57741f934042cb8d8b087b44b161db56fc3ffd4ffb675d36cd09f83935be853d8729f3f5298d12d6fd28d45dde515a4b9d7f67682d182ba5118abf451ff1988",
        "peers": [
            {
                "index": 1,
                "peerPubkey": "020140c925735fbc2a115251402864f13bb9d5c9280f1b4d09f1bd122ba74b539f",
                "address": "AV8e7ZMThRRwtD34SbTmZcs2MQacSMmer7",
                "initPos": 10000
            },
            {
                "index": 2,
                "peerPubkey": "029b6437374a7283085595a8638389807260da4b35703172523457d23ef75e325b",
                "address": "AGBA9KRXZh36PQdxpPkqXb8sE4V3bPHTp5",
                "initPos": 20000
            },
            {
                "index": 3,
                "peerPubkey": "03c6467e8d39d32ca66ff29131f62d0ea33a2a6624f7d364ee4e7a600ac30c327d",
                "address": "APheEbeVmVrWHBrFvE4GBMdic4x8HjEsa9",
                "initPos": 30000
            },
            {
                "index": 4,
                "peerPubkey": "028c714c0972f5db988c6d6bf902c5d62be798d9ef99a20d18585b305589c0bbd5",
                "address": "AX9yabJdAsPsCQrQ7spCt63sSMZ3UW1XfC",
                "initPos": 40000
            },
            {
                "index": 5,
                "peerPubkey": "0334a897a32233ce66f94e0a27032f99c37e88d78dd5ed178b370f6acad43abe23",
                "address": "AWHuXCEW3oH47YGiV5cJV5DLJo46zwnbwN",
                "initPos": 30000
            },
            {
                "index": 6,
                "peerPubkey": "0324ead47383d4949df908b1361c3bdcf8970780b3b982961f86d828a1a08e7b74",
                "address": "AYunUVX91WQwmJChA54epSuh4KKu62ibWq",
                "initPos": 20000
            },
            {
                "index": 7,
                "peerPubkey": "02dfc324f683cd22ccb74c56527174064db22f7e13d274b8523bbf2ebf5f72d63c",
                "address": "AJ76sGePV8LohnF81gsAxL5W6M43p1npT5",
                "initPos": 10000
            }
        ]
    }
}
```



DNS服务的配置文件名称为`scan_config.json`

```json
{
    "Base": {
				"DumpMemory": false,        		// 是否开启Profile进行性能记录
        "BaseDir": ".",									// 数据库文件所在目录
        "LogLevel": 0,									// 日志等级
        "PublicIP": "40.73.102.177",		// 固定外部IP
        "PortBase": 10000,							// 各服务端口基准端口，如下面本地端口为 10000+337即10337
        "LocalRpcPortOffset": 337,			// 本地RPC端口
        "EnableLocalRpc": false,				// 是否开启本地RPC
        "JsonRpcPortOffset": 336,				// JSON-RPC端口
        "EnableJsonrpc": true,					// 是否使用JSON-RPC
        "HttpRestPortOffset": 335,		  // REST 端口
        "HttpCertPath": "",							// HTTPS 证书路径
        "HttpKeyPath": "",							// HTTPS key 证书路径
        "EnableRest": true,							// 是否开启 REST 服务
        "ChainRpcAddr": "http://127.0.0.1:20336",	// 主链RPC地址，如果DNS为全节点， RPC地址为本地JSON-RPC地址
        "ChainRestAddr": "http://127.0.0.1:20334", // 主链REST地址，如果DNS为全节点， REST地址为本地REST地址
        "DisableChain": false, 					// 是否启动全节点， 如果关闭全节点功能，上面的主链RPC地址需要配成远端RPC节点的地址
        "ChannelNetworkId": 1565267317,	// Channel 网络ID
        "ChannelPortOffset": 338,				// Channel网络端口
        "ChannelProtocol": "tcp",				// Channel网络协议
        "ChannelClientType": "rpc",			// Channel请求主链方式
        "ChannelRevealTimeout": "200",	// Channel Reveal Timeout， 目前建议使用200个区块高度
        "ChannelDBPath": "./ChannelDB",	// Channel 数据库路径
        "DnsNetworkId": 1565511150,			// DNS网络ID	
        "DnsProtocol": "tcp",						// DNS网络协议
        "DnsPortOffset": 339,						// DNS端口
        "DnsGovernDeposit": 1000000000,	// DNS治理抵押
        "DnsChannelDeposit": 1000000000,// DNS通道抵押
        "AutoSetupDNSRegisterEnable": true,		// 是否开启DNS自动注册
        "TrackerPortOffset": 6369,			// Tracker端口
        "TrackerPeerValidDuration": "-24h",	// Tracker文件信息失效时间
        "NATProxyServerAddr": "",				// NAT proxy地址， 目前DNS有固定外部IP, 不使用NAT服务
        "DBPath": "DB",									// DNS数据库路径
        "DnsNodeMaxNum": 100,						// 保留字段
        "SeedInterval": 10,							// 保留字段
        "IgnoreConnectDNSAddrs": [		// 忽略连接其他DNS的钱包地址
        ]
    }
}
```



#### 五、启动SCAN



```shell
$ nohup ./scan --scanconfig . --config chain_config.json --networkid 1557388198 --loglevel=1 -p pwd >/dev/null 2>nohup.log &
```


命令解释：

--scanconfig:   指定DNS配置文件所在的目录为当前目录（或配置文件名，默认是scan_config.json）

--config: 	    指定全节点服务的配置文件

--networkid:    主链P2P网络ID

--loglevel:     日志文件等级

--wallet, -w:   钱包文件目录，默认为 ./wallet.dat

--password, -p: 钱包密码