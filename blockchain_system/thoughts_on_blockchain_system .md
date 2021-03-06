# 思考

## 1、确定性代码 vs 非确定性代码

Fabric有fabric peer和fabric orderer两种节点。Fabric的EOV（endorse-order-validation）架构使得“确定性代码”有一些不寻常的地方。Fabric在validation阶段，针对状态数据只是做了版本号冲突的检查，并未重新执行合约。但是这并不意味着选用Fabric，chaincode里面就可以引入“不确定性代码”。虽然客户端在收到提案背书应答后只取了第一个应答，但Peer交易背书阶段会针对该应答检查背书。若取的第一个应答不是此peer返回的那个应答（若存在随机代码），那么就会存在该peer背书校验不通过。因此，Fabric的设计也是不允许非确定性代码的。

**合约语言设计**  在Solidity中内置uint和int类型是uint256和int256的别名，这是为了避免不同CPU型号（大小端？）下，变量分配的空间（字节码在EVM中不一样？）不一致而带来的不一致性。

**密码操作**   在公钥密码学中，加密算法、签名算法通常是随机算法，哈希算法、解密算法、验签算法则是确定性算法。因此在类Ethereum的架构中设计业务合约，应该避免在合约中使用加密算法、签名算法操作。但合约中能做哈希算法、解密算法、验签算法、零知识证明验证算法、密文加法算法（在链外做加密，随机性放在链外，即客户端，完成）、密文乘法算法等操作。这也是[FISCO BCOS](https://github.com/FISCO-BCOS/FISCO-BCOS/issues/1865)在paillier预编译合约中只支持密文加法的原因。Ethereum 1.x支持的[预编译合约](https://github.com/ethereum/go-ethereum/blob/master/core/vm/contracts.go)清一色的支持确定性密码操作。例如，哈希函数（例如，SHA256、RIPEMD160、BLAKE2b）、椭圆曲线群上的加法、乘法等。

## 2、Fabric的设计

Fabric采用背书、排序、提交三阶段的设计，虽然截至Fabric 2.2 LTS，fabric orderer集群采用kafka、raft作为共识算法，虽然故障模型是宕机容错，但这并不意味着orderer就可以胡作非为（例如，篡改交易中的状态数据），因为提交阶段会对交易的背书进行校验，除非该组织和背书策略中指定的（例如，大多数）peer组织进行合谋，否则该交易在提交阶段会被标记为无效。因此，区块链系统的安全性是可以得到保证的，区块链系统的活性也是可以得到保障的（若故障数量不超过算法要求阈值）。