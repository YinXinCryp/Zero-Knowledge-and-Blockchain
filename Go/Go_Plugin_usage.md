# 1、Go plugin简介

2017年2月，Go 1.8版本中引入了一项插件特性。

# 2、为什么区块链系统适合插件化设计？

一个区块链系统（软件）包含密码算法（或密码协议）模块、共识模块、状态存储引擎模块、网络模块、合约引擎模块等。

**密码算法（或密码协议）**  本模块主要对交易、区块等数据等。常用的密码算法（例如，数字签名算法、密码哈希函数）有很多。我国有商用密码体系，例如sm2、sm3、sm4、sm9。美国NIST也有相应的算法。例如，SHA3、ECDSA、AES。此外，随着量子计算的发展，经典的公钥密码体制安全性受到越来越大的挑战。因此，一个区块链系统在设计时是要考虑到这一点的，这个模块可以随着时间的流淌插拔安全、高效的密码算法。

**共识模块**  本模块主要实现交易排序。根据故障模型、网络假设等，可供选择的算法有很多。例如，宕机故障假设下常用的有Paxos、Raft、EPaxos等；在拜占庭故障+弱同步模型假设下有MirBFT、SCP、PBFT、Tendermint、HotStuff及其变种（例如，DiemBFT）、Casper ffg等；在拜占庭故障模型+异步网络模型假设下有HoneyBadgerBFT、DumboBFT、[SKALE Consensus](https://github.com/skalenetwork/skale-consensus)等。

**状态存储引擎模块**  本模块主要用来存放世界状态。目前多用kv存储，例如leveldb、rocksdb。当然也可以选用关系型数据库，例如postgres等。

**网络模块**  本模块主要收发交易、区块等业务数据。此外，还有节点发现、状态维护等。节点间的传输协议有基于gRPC、TCP和QUIC。

**合约引擎模块**  本模块主要是执行合约。流行的合约引擎有Wasm VM、EVM、Move VM等。Wasm VM是专门解析[WebAssembly](https://webassembly.org/) (Wasm)字节码的栈式虚拟机；Ethereum VM（简称EVM）是专门解析solidity字节码（solidity的编译器solc将.sol文件编译为字节码）的虚拟机；Move VM是专门解析Move字节码的栈式虚拟机；

俗语说，one size does not fit all。区块链系统的各个模块应该是高可扩展的，采用插件化设计来扩展特定的模块接口是工程上的具体技术路线。

# 3、典型案例

据我所知，[EOS](https://github.com/EOSIO/eos/tree/master/plugins)是率先在设计上大量应用插件设计的案例。Go开发的区块链项目也不乏利用go插件特性，例如Hyperledger Fabric、OpenAtom XuperChain。

# 4、GopherCon 2020上关于Go插件的演讲

[Travis Smith](./slides/Go_Plugin_Travis_Smith@GopherCon2020.pdf)在大会上讲述了Go Plugin实践经验。