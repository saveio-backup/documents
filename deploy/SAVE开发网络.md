### SAVE 开发网络环境



> 更新于 2021-06-07 14:27



#### 1. 登录



```
$ ssh save-dev@IP
$ 输入密码
```



##### 注意：

**以下服务器因安全组策略，需要改用32000端口进行`ssh`登录:**

* **107.155.56.98**
* **152.32.217.181**



```shell
$ ssh -p 32000 save-dev@IP
```



#### 2. 服务器硬件配置



| 角色     | 云主机配置 | 磁盘  | 公网IP         | 内网IP          | 设备名                           | 备注 |
| -------- | ---------- | ----- | -------------- | --------------- | -------------------------------- | ---- |
| 共识节点 | 4C/8GB     | 150GB | 107.155.56.98  | 10.35.147.106   | U-SP-SAVE-DEV-CONSENSUSRPC-NODE1 |      |
|          |            |       |                |                 |                                  |      |
| RPC      | 4C/16GB    | 20GB  | 117.50.99.70   | 10.9.141.111    | U-BJ-SAVE-DEV-RPC-NODE2          |      |
| RPC      | 4C/16GB    | 20GB  | 117.50.109.175 | 10.9.71.17      | U-BJ-SAVE-DEV-RPC-NODE1          |      |
|          |            |       |                |                 |                                  |      |
| 存储节点 | 4C/8GB     | 500GB | 152.32.217.181 | 10.35.44.195    | U-SP-SAVE-DEV-SAVE-NODE1         |      |
| 存储节点 | 4C/16GB    | 20GB  | 117.50.109.179 | 10.9.170.232    | U-BJ-SAVE-DEV-STORAGE-NODE1      |      |
| 存储节点 | 4C/16GB    | 20GB  | 117.50.99.70   | 10.9.141.111    | U-BJ-SAVE-DEV-RPC-NODE2          |      |
|          |            |       |                |                 |                                  |      |
| DNS      | 4C/16GB    | 100GB | 117.50.17.39   | 192.168.109.141 | U-BJ-SAVE-DEV-DNS-NODE2          |      |
| DNS      | 2C/8GB     | 20GB  | 106.75.19.10   | 10.9.77.117     | U-BJ-SAVE-TEST-STORAGE-NODE1     |      |
| DNS      | 2C/8GB     | 20GB  | 106.75.22.115  | 10.9.182.203    | U-BJ-SAVE-TEST-STORAGE-NODE2     |      |
|          |            |       |                |                 |                                  |      |
| Proxy    | 4C/16GB    | 100GB | 106.75.48.16   | 192.168.69.222  | U-BJ-SAVE-DEV-PROXY-NODE2        |      |
| Proxy    | 2C/8GB     | 20GB  | 106.75.76.46   | 10.9.70.45      | U-BJ-SAVE-DEV-NAT-NODE1          |      |



#### 3. 各程序路径



所有程序路径位于 `~/saveio/` 目录下, 分别为:

`~/saveio/themis`

`~/saveio/edge`

`~/saveio/scan`

`~/saveio/porter`







#### 4. 防火墙

开放端口范围为:

6000-7000

10000-11000

20000-21000

30000-40000



#### 5. 编译与运行

使用jenkins直接编译并部署, jenkins网址为: 

http://10.0.1.138:8080/



对应的开发网项目为:

`deploy-themis`: 编译themis, 同时编译 scan、porter、edge

`deploy-scan` : 编译scan, 同时编译 porter、edge

`deploy-porter`：编译porter, 同时编译 edge

`deploy-edge` : 编译edge

`init-start-dev-net`： 重启所有程序









