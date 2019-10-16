# 状态通道方案
> Updated 2019-10-16 

*本文主要列出新状态通道方案的基本设计，以支付通道为基础，后期可扩展为广义状态通道*
## 方案要点
- 节点角色分为客户端和路由节点
- 路由节点负责交易验证，状态路由转发，合约提现等
- 账户状态使用位图模型，路由过程中通过嵌套签名的方式实现链路跟踪及后期分润
- 对账户合约的充值，会注明经过那个路由节点来中转，即第一跳节点，这个节点负责转发交易，并对交易的合法性进行验证
- 结算操作，可以由接收客户端，也可由保留路由交易记录及客户端回执的路由节点来完成
- 用户账户提现，需在完成本合约账本上一笔交易之后，等待一定高度（锁定期）后方可发起提现交易
## 合约设计
1. 路由节点管理合约：路由节点的质押，权限，惩罚，退出等管理
2. 用户账户合约：用户账户体系下，对每个代理节点的位图账户的管理，创建，充值，提现，删除等
3. 分润合约：根据客户端提交的聚合交易，从位图合约中提取对应的锁定单据进行支付转账，同时根据嵌套的路由交易，对路由上的节点进行分润
4. 位图账户合约：管理所有用户的位图账户，包括创建账户，提取，锁定等
## 位图账户
位图账户是将余额账户utxo化，便于提高并行/双花检测能力的特殊账户，所有账户由位图账户管理合约管理
#### 账户格式
位图账户是余额的UTXO集合，假设账户分辨率为小数点后9位，最小单位位图点余额是1000，则生成250*250位图，图上每个点值由16比特构成，一共有1 000 000个 bit,每个bit代表一笔1000的支付，那么这笔价值为1的，包括一百万个utxo的支付集合由125000字节表示，这一百万个utxo可以用来发起并发支付

#### 充值
1. 用户发起一笔特殊交易，从自己的合约账户中提取一部分，同时创建一个或几个位图账户，这些账户可以有同样的单位位图点余额，也可以各不相同，满足业务需求的同时减少合约空间占用
2. 将创建的位图账户绑定在用户账户下，同时每个位图账户必须绑定一个代理节点才能使用，该节点是第一个接收到这个账户位图utxo交易的节点，有义务检查交易的合法性
#### 账户锁定
位图账户由于是特殊用户账户，在某些情况下需要锁定一定时间才能使用
- 结算：在结算交易提交后，位图账户被锁定一定时间，以便支付链路上的节点有时间进行验证，防止节点作恶，或是交易回退等操作，锁定时间过后结算交易执行
- 提现：提现提取所有剩余余额，并删除当前位图账户，默认提现操作必须在最后一次路由支付操作后一定时间没有任何交易情况下才能操作，相关节点在检查到有提现操作后可以提交对应的交易，在锁定期间无异议则解锁后自动提现
- 切换代理节点：同结算操作，必须在最后一次路由支付操作后一定时间没有任何交易情况下才能操作
- 双花：如果发现位图发生双花交易，账户即锁定，除了链路节点提现回退等操作外，直到惩罚提现结束后才能解锁
#### 支付
- 每个位图点位原则上都是一笔可以支付的utxo，只不过状态为0/1，如果该位bit为1，则在链上是可以使用的，但并不代表链下别被使用，所以需要上面的锁定期
- 链下支付时每个节点都会记录本地路由过的位图账户信息，这些信息类似utxo，对应位图账户的每个点位，当某个utxo被提交到链上，则位图账户bit位置0，同时将账户代表的余额对接收方进行转账
- 每一笔位图账户的第一跳交易目的地址都是代理节点，该节点负责这个账户的支付有效性检查等验证
## 路由支付
### 交易结构
- 每个支付交易都会带有签名，当前高度，引用的位图点utxo,支付目的地址，下一跳账户地址等
```
message EnvelopeMessage{
    BitAccount account
    uint64 index_start
    uint64 index_end
    TokenAmount transferred_amount
    TokenNetworkAddress token_network_address
}
 message TransferMessage{
    MessageID message_id
    EnvelopeMessage message[]
    AccountAddress dest_address
    AccountAddress next_address
    Height height
    ExpiredHeight height
    SignedMessage signature
 }
```
dst节点收到交易，验证没问题即返回确认
```
message ComfirmMessage{
    MessageID comfirmed_message_id
    RecevieByte comfirmed_byte
    Height height
    SignedMessage signature
}
```
有问题的返回err
```
message ErrMessage{
    MessageID comfirmed_message_id
    ErrorCode errno
    Height height
    SignedMessage signature
}
```

```
const ErrorCode (
	NoError     = iota 
	DoubleSpendError
	DedeserializationError
	HopCountError        
	ExpiredError         
	RouteError
	...
)
```

路由节点，添加下一跳地址和当前高度，签名转发消息

```
message MediaMessage{
    MessageID message_id
    RecevieByte message_byte
    HopCount current_hop
    AccountAddress dest_address
    AccountAddress next_address
    Height height
    ExpiredHeight height
    SignedMessage signature
 }
```
这样交易就注明了各路由节点转发的高度及路由路径
节点签名的交易都留存在本地存储中，用于超时交易结算，路由失败回退等操作
### 路由
1. 路由算法开始选用互联网常有算法，如ospf或dijkstra，后期根据各节点流量及带宽迭代新的路由算法
2. 路由交易有当前块高及过期块高，每一跳也设置有过期块高，如果收到交易后发现当前块高已经高于过期块高则返回超时错误
3. 收到超时错误的上一跳节点如果此时还没有超过转发的过期高度，则继续寻找其他路由完成本次支付 
4. 路由序列图

ABCD都为质押的路由节
A为代理节点
- 正常路由
```
sequenceDiagram
src->>A:transfer_txn
A->>A:双花检测
A->>B:media_txn
B->>C:media_txn
C->>dst:media_txn
dst->>C:comfirm_msg
C->>B:comfirm_msg
B->>A:comfirm_msg
A->>src:comfirm_msg
```
- 出错返回
```
sequenceDiagram
src->>A:transfer_txn
A->>A:双花检测
A->>B:media_txn
B->>C:media_txn
C->>C:超时
C->>B:error_msg
B->>A:error_msg
A->>src:error_msg
```
- 重新选择路由
```
sequenceDiagram
src->>A:transfer_txn
A->>A:双花检测
A->>B:media_txn
B->>C:media_txn
C->>B: no_routine_error
B->>B: not expired
B->>D:media_txn_2
D->>dst:media_txn
dst->>D:comfirm_msg
D->>B:comfirm_msg
B->>A:comfirm_msg
A->>src:comfirm_msg
```
- 并发
```
sequenceDiagram
src->>A:transfer_txn_dst1
src->>A:transfer_txn_dst2
A->>A:双花检测(dst1/dst2)
A->>B:media_txn_dst1
A->>B:media_txn_dst2
B->>C:media_txn_dst1
B->>dst2:media_txn_dst2
C->>dst1:media_txn_dst1
dst2->>B:comfirm_msg_2
dst1->>C:comfirm_msg_1
B->>A:comfirm_msg_2
C->>B:comfirm_msg_1
A->>src:comfirm_msg_2
B->>A:comfirm_msg_1
A->>src:comfirm_msg_1
```
## 结算
1. 目标节点收到通道交易，提交到链上，即可实现位图账户向自己账户的转移，或是向自己的某个位图账户转移
2. 如果是位图账户转账到普通账户，则将位图账户对应的bit置0，然后按照位图对应的单位换算成金额，扣除手续费和理由费用，转账到目标账户
3. 如果是位图账户转账到位图账户，根据两边位图对应的单位，换算成对应的bit位，添加到目标位图账户中
### 路由费结算
不管是什么结算方式，路由费都在结算时一并转移
参考目标节点收到的路由交易

```
message MediaMessage{
    MessageID message_id
    RecevieByte message_byte
    HopCount current_hop
    AccountAddress dest_address
    AccountAddress next_address
    Height height
    ExpiredHeight height
    SignedMessage signature
 }
```
目标节点将收到的交易包提交上链，根据交易包中的下一跳地址以及签名，就可以找出路由路径，根据合约事先规定的路由费用，将源节点的位图转移到对应的账户中，完成路由费的支付

*代理节点由于涉及到双花支付的检查，得到的路由费会相对高些*

### 代理确认
代理确认通常出现在目标节点不在线无法收到交易的情况下，代理结算的前提是，路由的倒数第二跳是目标节点的代理节点，且目标节点在合约中声明该节点可以代理结算

*代理确认的交易，在目标节点上线后，应向代理节点支付代理费用*
- 代理确认
```
sequenceDiagram
src->>A:transfer_txn
A->>A:双花检测
A->>B:media_txn
B->>C:media_txn
C->>C:dst没有链路
C->>B:agency_comfirm_msg
B->>A:agency_comfirm_msg
A->>src:agency_comfirm_msg
dst->>C: connect
C->>dst:media_txn
dst->C:comfirm_msg
```
- 代理超时

代理节点可以设置一个代理超时时间，如果客户端在某段时间内无法上线，则该代理节点可以将收集的交易上链，手续费从上链交易中扣除。代理节点可以同时上链多个不同的节点的代理交易，手续费由各节点分摊。如果该节点手续费不够或者没有足够多的代理交易，则可以选择存在本地，直到符合条件为止。

*代理节点和目标节点都可能存在某些交易由于手续费的问题一直无法上链的情况，这些交易的总和数额会小于一次上链手续费用*

### 中间路由结算
- 这是一个可选的分润机制
- 中间路由节点可以选择本地存储的且未被链上确认的交易上链，这些交易必须有对应的确认信息
- 共识节点通过对照路由交易及确认信息即可明确该次交易的路径并根据协议进行分润
- 惩罚机制也会使用这个机制

### 零钱整理
位图账户类似于utxo账户，由于经常使用及手续费的问题，是的位图位置会出现许多不连续的状态，导致EnvelopeMessage占用的字节空间大增，所以需要对位图账户进行零钱整理，将一个账户的零钱导入到另一个位图账户

 *零钱整理执行提现加充值操作，需要对当前账户锁定一段时间*
## 双花检测
### 代理节点检测
作为代理节点，是第一个接收支付信息的节点，需要对代理的支付信息进行有效性和双花检测，如果发现源账户引用的index已经被使用，则需要返回DoubleSpendError
```
sequenceDiagram
src->>A:transfer_txn
A->>A:双花检测
A->>src:DoubleSpendError

```
### 路由检测
路由检测作为代理节点双花检测的监督机制。任何路由节点都可以根据本地数据库来判断是否双花，如果双花，则将所有数据打包共识，如果共识记得判断当前交易双花，则对源节点和代理节点进行惩罚，打包上链的动作即中间路由结算，不过结算类型是挑战。
```
sequenceDiagram
src->>A:transfer_txn
A->>A:双花检测
A->>B:media_txn
B->>B:双花检测
B->>CHAIN:challege_txn
CHAIN->>CHAIN: 共识（punish src/A）
```

*如果惩罚时发现源节点的账户不够支付惩罚，则由代理节点支付剩余部分*
## 节点作恶