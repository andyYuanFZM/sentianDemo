# 森田区块链EVM合约使用教程

## 文档目录
	- [文档修改记录](#文档修改记录)
	- [文档阅读说明](#文档阅读说明)
	- [术语介绍](#术语介绍)
	- [森田链环境部署](#森田链环境部署)
	- [NFT合约概述](#NFT合约概述 )
	- [通过SDK实现合约部署调用](#通过SDK实现合约部署调用)
	- [应用和森田平行链对接注意事项](#应用和森田平行链对接注意事项)

### 文档修改记
| 版本号 | 版本描述                              | 修改日期   | 备注 |
| ------ | ------------------------------------- | ---------- | ---- |
| V1.0   | 1. 主链+平行链部署<br>2. NFT合约简单概述<br>3.通过 JAVA-SDK 在平行链上发行 NFT<br>4. 对接注意事项| 2022/06/03 |

### 文档阅读说明
- 术语介绍：简单了解  
- 森田链环境部署：由森田运维负责部署，略。  
- NFT合约概述： 了解下概念，文档中有NFT合约的样例，如果样例太简单满足不了业务需求，需要自己开发新合约的，可以重点了解  
- 通过SDK实现合约部署调用：***重点了解***
- 应用和森田平行链对接注意事项: ***重点了解，是项目中踩过的坑，了解过后可少走弯路。***

### 术语介绍 
| 序号 | 术语 缩写                              | 解释   |
| ------- | -------------------------------------- | --------------------- |
| 1   | 主链| 森田主链是一条具备多共识节点的区块链，采用安全的POS共识机制|
| 2   | 平行链| 平行链依附于主链，平行链之间通过名称来区分，平行链与平行链之间数据相互隔离， 平行链与主链之间通过grpc通信。|
| 3   | EVM| 以太坊虚拟机的缩写，目前EVM算是区块链中最大的生态，很多链都支持EVM的能力，森田平行链也完全兼容EVM,通过EVM可以动态的部署智能合约进行计算|
| 4   | ERC721| 运行在EVM中，服务于非同质化代币（NFT）, 每个Token都是不一样的，都有自己的唯一性和独特价值,不可分割，可追踪。|
| 5   | ERC1155| 运行在EVM中，也是服务于非同质化代币(NFT),相比于ERC721它同时还支持在一个合约中存储多个数字资产，支持一次性批量发行多个不同类型的的数字资产，支持在一次转账过程中转多个不同类型的数字资产。|
| 6   | 交易组| 把两笔及以上的交易放在一个组里一次性发送。|
| 7   | 代扣手续费| 将代扣交易和正常用户的交易打包进一个交易组中，代扣交易使用代扣地址签名，用于主链上手续费扣除。|
| 8   | SDK| 封装了同区块链交互的接口和区块链通用方法（包括：公私钥生成，签名，交易构造等）, 支持java-sdk, go-sdk, web3.js等 |

### 森田链环境部署
**环境已经具备，此章略过。**  

### NFT合约概述
NFT合约运行在平行链的EVM虚拟机中, EVM虚拟机运行solidity语言编写和编译的智能合约。   
Solidity语言更多信息, 请参阅  [[Solidity中文官方文档]](https://learnblockchain.cn/docs/solidity/)  
下文[NFT合约开发编译]链接介绍ERC1155和ERC721两类合约最简单的使用。  
两种合约的基本介绍， 合约的编写和编译，以及和链的交互流程等 [[NFT合约开发编译]](https://github.com/andyYuanFZM/sentianDemo/blob/master/NFT合约开发编译.md)   

### 通过SDK实现合约部署调用     
#### JAVA-SDK
适用于应用平台使用JAVA开发的情况,提供SDK对应的jar包，SDK里包含了公私钥生成,合约部署方法,合约调用方法,交易签名,交易查询,区块链信息查询等方法。   
JAVA-SDK环境开发环境部署参考链接： [[JAVA-SDK]](https://github.com/andyYuanFZM/sentianDemo/blob/master/JAVA-SDK开发环境.md)  

1. 子目录说明
mintByManager目录下的NFT合约只支持管理员来发行NFT， 适用于平台对于NFT发行有严格限制的业务场景。（**最常用，一般情况下都是用这种方式**）  
mintByUser目录下的NFT合约不限制只有管理员才能发行，任何用户都可以调用mint方法发行NFT， 适用于平台任意作者都可以发行NFT的业务场景。   
deployByUser此目录下的用例支持任意用户都可以部署NFT合约，适用于平台支持每一个艺术家都可以部署自己的智能合约（这种场景较少）。  

**目前使用最多的是mintByManager目录下的ERC1155合约, 建议只关注这个用例**  

2. 运行JAVA Demo程序  
- 调用 [[BlockChainTest.java]](https://github.com/andyYuanFZM/sentianDemo/blob/master/BlockChain.java)  中的createAccount方法，生成地址和私钥  
- 修改对应子目录下的ERC1155Test或ERC721Test文件，将上一步生成的内容，分别填充到以下几个参数中，注意私钥即资产，要隐私存放，而地址是可以公开的  
```  
// 管理员地址和私钥
String managerAddress = "";
String managerPrivateKey = "";
    
// 代扣地址和私钥,用于所有用户的手续费代扣,避免用户接触燃料费这一底层概念
String withholdAddress = "";
String withholdPrivateKey = "";
```  
-给上述两个地址下充值测试用的燃料  
-修改ERC1155Test或ERC721Test两个文件中以下两个参数  
```  
// 改成自己平行链所在服务器IP地址
String ip = "";
// 改成自己平行链服务端口，对应的是配置文件里的jrpcBindAddr配置项，默认的是8901。 注意：如果远程访问，防火墙要放行此端口
int port = 8901;
```   
-修改平行链名称  
```  
// 改成和部署的森田平行链一致，目前叫：user.p.sentianPara.
String paraName = "user.p.sentianPara.";
```   
- 运行测试程序  

#### GO-SDK  
略

#### web3.js
略

## 应用和森田平行链对接注意事项   
由于森田主链涉及燃料费,同时森田主链平均每3秒一个确认的特性,可能会存在交易的失败,主要有以下两大类情况：  
1. 交易上链了，但交易执行失败（有返回交易hash）：   这类交易通过了mempool（交易缓存池）的合法性检查，但是在合约执行过程中失败了（ 比如转移了错误数量的NFT）。
2. 交易没有上链（没有返回交易hash，rpc接口直接返回出错信息）： 这类交易在mempool的合法性检查中没有通过，包括以下以类错误：  
	- 签名错误（ErrSign）： 签名校验不通过，一般不会遇到，除非人为去改交易内容。  -- 不常见   
	- 交易重复（ErrDupTx）：mempool中发现重复交易，一般不会遇到，除非人为发送重复交易（所谓重复是hash完全一模一样的两笔交易，而不是指业务上数据相同）。 -- 不常见   
	- 手续费不足： 代扣地址下手续费不足会导致交易无法上链，需要保证代扣地址下GAS费充足。  -- 有可能会遇到  
	- 手续费太低（ErrTxFeeTooLow）： 交易设置的手续费比链上要求的手续费低，常见于部署EVM合约或批量发行大量NFT的场合， 需要通过queryEVMGas预估计出一个GAS费，然后在这个基础上再加上0.001作为手续费，这样能保证交易不会上链失败。 -- 有可能遇到，参考用例中手续费设置方式 
	- 交易账户在mempool中存在过多交易（ErrManyTx）： BTY为防止来自于同一个地址的频繁交易，限制每个账户在mempool中的最大交易数量不能超过100， 所以当交易频率很高时，mempool中代扣手续费的交易（都是来自同一个代扣地址）可能会超过100的， 而100笔以后的交易会被丢弃（rpc返回errmanytx的错）从而导致关联的交易也被丢弃。   -- 有可能会遇到   

举例说明应用层和区块链整合后的一般处理方案：  
为保证数据的一致性，需要判断交易在区块链上确实成功（拿到交易hash，且实际交易的执行结果是：ExecOk）, 业务上才能判定为成功。    且BTY支持以代扣的方式来扣除用户交易的燃料费， 代扣包含两笔交易：第一笔是代扣的交易（最简单的存证）， 第二笔是实际用户的交易。  交易上链后，区块链返回的是第一笔交易的hash值， 而这笔交易hash不能用于判断交易是否执行成功，需要拿这个hash值查到对应的第二笔交易才能判断。  具体见下面处理流程以及测试用例：  

场景：NFT平台上新品，大量用户抢购。  
处理流程：  
1. 用户完成支付，应用层触发NFT转账动作，并拿到区块链返回的交易hash(在采用代扣的情况下，这个hash对应的是代扣手续费交易，而非实际转NFT交易)  
2. 由于区块链是异步处理且有一定时延，这时NFT资产还没有真正转到用户地址下，但应用层可以预先将该笔订单标记为【待完成】。 这个状态不用给用户显示，只由应用层记录，对于用户而言，他只要一完成支付，就视为成功，不需要让他等待区块链的处理结果。  
3. 应用层把第1步中拿到的hash放到处理队列中，定时拿这个hash去查结果，如果查询结果返回空，hash继续留在队列中等待下一次查询， 如果结果不为空，从返回结果中拿到第二笔交易的hash,再根据这第二笔的交易hash去查询（这一步查询不用等待，拿到hash后可以马上查），从这次查询的返回结果拿到执行结果，如果是ExecOk代表执行成功，应用层再【待完成】的订单标记为【完成】，并从处理队列中删除第一笔交易的hash。 如果结果是ExecPack，代表执行失败，这个一般可能是业务上的bug(比如转了超过地址下数量的NFT资产，或是权限不够等等)，这种情况后续怎么处理由应用层根据实际情况来判断。    
4. 其它异常  
4.1 根据hash一直查不到有结果返回的可能性况： 1. 部署的平行链名称和上链交易中带的平行链名称不一致。  2.  平行链连接的主链节点高度落后于主网最大高度，或是平行连连接的那个主链节点离线。  
4.2 数据上链后，没有拿到交易hash，且rpc返回ErrTxFeeTooLow， 代表设置的交易手续费低于区块链所需的最低手续费。 需要在发交易前调用查询GAS费接口来估算出手续费（参考用例），再设置到交易中。  
4.3 数据上链后，没有拿到交易hash，且rpc返回ErrManyTx。  有大量交易上来时，应用层用队列缓存发送，比如每间隔10秒发送100笔交易，这样基本能解决ErrManyTx问题，如果再有个别交易还发生ErrManyTx，也需要把失败的再放回队列，后面重新发送。  
4.4 数据上链后，没有拿到交易hash，且rpc返回ErrTxMsgSizeTooBig， 代表交易体太大。 一般出现于ERC1155批量mint一批NFT时， 建议一次mint的token数量不要超过1000个， 数量比较大的，可以分多次来mint。  
4.5 定期检查代扣地址下的余额，保证手续费充足。  

