# Edge 挖矿文档

## 1. 编译与安装

- 从源码编译

    直接使用 `make` 命令即可编译程序

    ```bash
    $ git clone github.com/saveio/edge.git
    $ cd edge
    $ make
    ```

- 下载Release

## 2. 创建钱包

```bash
$ ./edge account add -d
```

输入钱包密码，以及再次确认钱包密码，成功之后会在本地生成一个 `keystore.dat` 文件，同时命令行会有以下类似的输出

```bash
Index:1
Label:
Address:ANN4qyq5TgSHqm1c6WzFWrERgCGzXYrKfH
Id(For PoC):3850992683494297153
Public key:03966795d795ce179e2b464ec7532cf1900e759856b86a4d6b7f29d7c0eaea9043
Signature scheme:SHA256withECDSA
Create account successfully.
```

## 3. 启动

`edge` 是加载本地的 `config.json` 配置文件进行启动。关于程序更多的细节可以参考<测试网开发者文档>。启动命令为

```bash
$ ./edge 
```

输入钱包密码之后即可启动。

## 4. 生成Plot文件

`edge` 启动后，重新打开新的窗口，在新窗口输入一下命令生成 `plot` 文件:

```bash
$ ./edge plot generate --system linux --size 1024 -- num 1
```

`edge` 默认会在 `Chain-1/plots/钱包文件地址/` 目录下生成 `plot` 文件。 也可以使用 `--path` 修改生成文件的路径。参数说明:

`--system` : 本机机器系统，比如 `linux` 或者 `win`

`--size` : 生成的 `plot` 文件大小，单位为KB

`--num` : 生成的 `plot` 文件数， `edge` 使用多线程同时生成文件，这里的配置建议不要超过系统 `CPU` 核心数

`--path` : 生成 `plot` 文件存放位置

`--numericID` : `plot` 文件参数 ID， 与钱包地址一一对应，具体可以用 `./edge account list -v` 查看

`--startNonce` : `plot` 文件参数 开始的Nonce，从0开始，与前一个文件的 `nonce` 一致

`--nonce`: `plot` 文件大小，1 `nonce` = 256 KB

## 5. 添加Plot文件进行挖矿

生成Plot文件之后，可以将文件存与扇区进行挖矿。默认会自动创建扇区，如果需要自定义扇区，可以参考<测试网开发者文档>。添加Plot文件的命令为:

```bash
$ ./edge plot mine --path=./Chain-1/plots/ANN4qyq5TgSHqm1c6WzFWrERgCGzXYrKfH/3850992683494297153_0_1024 --create-sector true
```

或者直接添加整个文件夹

```bash
$ ./edge plot mine --path=./Chain-1/plots/ANN4qyq5TgSHqm1c6WzFWrERgCGzXYrKfH/ --create-sector true
```

添加成功之后可以看到类似的输出

```bash
[
   {
      "TaskId": "4e4c57aa-f33c-11eb-a478-88e9fe5b16bf",
      "UpdateNodeTxHash": "e62108a840e88ed13fcddfbc38710b9088be77660713dec398a7f04c7072943e",
      "CreateSectorTxHash": "579a1be7a5c13376cd3fe9ea98f570718aac50306a12955320a4ce6f2cfa2472"
   },
   {
      "TaskId": "59f4ccb8-f33c-11eb-a478-88e9fe5b16bf",
      "UpdateNodeTxHash": "",
      "CreateSectorTxHash": ""
   }
]
```

说明: `TaskId` 为此次任务的ID， 可以通过查看任务详情的接口查看任务状态。 另，如果  `UpdateNodeTxHash` 或者 `CreateSectorTxHash` 为空，说明这个文件不需要重新开辟新扇区，可以移动到原始扇区进行存储。

## 6. 查看已经提交过PDP证明的Plot文件

对于添加成功后的 `plot` 文件, `edge` 会定期提交 `PDP` 证明来获取挖矿奖励。查看已经提交过证明的 `plot` 文件可以使用以下命令:

 

```bash
$ ./edge plot list-all-proved
```

输出

```bash
{
   "TotalCount": 2,
   "TotalSize": 4096,
   "FileInfos": [
      {
         "FileName": "Chain-1/plots/ANN4qyq5TgSHqm1c6WzFWrERgCGzXYrKfH/3850992683494297153_0_8",
         "FileHash": "QmZnwtAUn73hwmmRV16MH6LWiorahrDvA8t8r3i4kaA5Zx",
         "FileOwner": "ANN4qyq5TgSHqm1c6WzFWrERgCGzXYrKfH",
         "FileSize": 2048,
         "BlockHeight": 101,
         "PlotInfo": {
            "NumericID": 3850992683494297153,
            "StartNonce": 0,
            "Nonces": 8
         }
      },
      {
         "FileName": "Chain-1/plots/ANN4qyq5TgSHqm1c6WzFWrERgCGzXYrKfH/3850992683494297153_8_8",
         "FileHash": "QmdFBBEf2C3T1Gii6NtuFKnyfhNQPnLTHxg1bEJqu546zQ",
         "FileOwner": "ANN4qyq5TgSHqm1c6WzFWrERgCGzXYrKfH",
         "FileSize": 2048,
         "BlockHeight": 103,
         "PlotInfo": {
            "NumericID": 3850992683494297153,
            "StartNonce": 8,
            "Nonces": 8
         }
      }
   ]
}
```