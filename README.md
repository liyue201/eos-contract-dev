# eos智能合约开发环境搭建和部署


## docker环境安装
eos官方提供了eos开发环境docker镜像，直接安装即可。下面命令启动了一个只有一个bp节点的eso私有网络。容器内的/work目录是我们的工作目录，为了方便编码我们将它挂载在主机的/Users/stirling/code/eos/work目录。
```
docker run --name eosio -d \
  -p 8888:8888 \
  -p 9876:9876 \
  -v /Users/stirling/code/eos/work:/work  \
  -v /tmp/eosio/data:/mnt/dev/data  \
  -v /tmp/eosio/config:/mnt/dev/config \
  eosio/eos-dev:v1.1.0  \
  /bin/bash -c "nodeos -e -p eosio \
  --plugin eosio::wallet_api_plugin \
  --plugin eosio::wallet_plugin \
  --plugin eosio::producer_plugin \
  --plugin eosio::history_plugin \
  --plugin eosio::chain_api_plugin \
  --plugin eosio::history_api_plugin \
  --plugin eosio::http_plugin \
  -d /mnt/dev/data \
  --config-dir /mnt/dev/config \
  --http-server-address=0.0.0.0:8888 \
  --access-control-allow-origin=* --contracts-console"
```
查看一下容器日志可以，看到每秒钟产生两个区块。
```
 docker logs -f eosio
```

进到容器里面
```
 docker exec  -it eosio /bin/bash
```
查看私链信息
```
root@6af410e7fc7e:/#cleos get info 

{
  "server_version": "75635168",
  "chain_id": "cf057bbfb72640471fd910bcb67639c22df9f92470936cddc1ade0e2f2e7dc4f",
  "head_block_num": 1485,
  "last_irreversible_block_num": 1484,
  "last_irreversible_block_id": "000005cc7132555d31de6bf925fc4e0fab87eb3e9ff7e649811141fbb943991c",
  "head_block_id": "000005cd1747718f33aa1a50f19b606c021b33e3134b0c9ac89cf02a552cc1fc",
  "head_block_time": "2018-08-31T01:35:05.000",
  "head_block_producer": "eosio",
  "virtual_block_cpu_limit": 881052,
  "virtual_block_net_limit": 4626545,
  "block_cpu_limit": 199900,
  "block_net_limit": 1048576
}
```

## 创建钱包，导入eosio私钥

创建默认钱包，记得保存钱包密码
```
root@6af410e7fc7e:/#cleos wallet create 

Creating wallet: default
Save password to use in the future to unlock this wallet.
Without password imported keys will not be retrievable.
"PW5K9UYDNxyBrSHyruWdLywwRghfMmd1u9CUaayqYSjBZL7qLiwBy"
```

若隔一段时间不使用，钱包会自动锁定，可以用这个命令解锁，输入上面保存的钱包密码

```
root@6af410e7fc7e:/# cleos wallet unlock
password: PW5K9UYDNxyBrSHyruWdLywwRghfMmd1u9CUaayqYSjBZL7qLiwBy
```

导入eosio私钥，为了完成后续部分教程，您还需要在钱包解锁状态下导入eosio的私钥
```
root@6af410e7fc7e:/# cleos wallet import 

"/opt/eosio/bin/keosd" launched
private key: 5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3
```

## 部署系统合约
```
root@6af410e7fc7e:/# cleos set contract eosio /contracts/eosio.bios -p eosio

Reading WAST...
Assembling WASM...
Publishing contract...
executed transaction: 414cf0dc7740d22474992779b2416b0eabdbc91522c16521307dd682051af083  4068 bytes  10000 cycles
#         eosio <= eosio::setcode               {"account":"eosio","vmtype":0,"vmversion":0,"code":"0061736d0100000001ab011960037f7e7f0060057f7e7e7e...
#         eosio <= eosio::setabi                {"account":"eosio","abi":{"types":[],"structs":[{"name":"set_account_limits","base":"","fields":[{"n...
```

## 创建合约帐号


创建密钥对
```
root@6af410e7fc7e:/# cleos create key
Private key: 5KJ8EA3TZwrX62wgcSuP9etvFvymyG6Qsz8KzZubKgH1CGt4ssi
Public key: EOS5oR8TdbycNttjL4hYe5KSwGuWVj1YGy8H2gvKHv6J6nTU5yt7G
```
将这个私钥导入钱包：
```
root@6af410e7fc7e:/# cleos wallet import
5KJ8EA3TZwrX62wgcSuP9etvFvymyG6Qsz8KzZubKgH1CGt4ssi
```
创建合约账号（公链上的的账号必须是12位长度，私链可以小于12位）

```
root@6af410e7fc7e:/# cleos create account eosio user EOS5oR8TdbycNttjL4hYe5KSwGuWVj1YGy8H2gvKHv6J6nTU5yt7G EOS5oR8TdbycNttjL4hYe5KSwGuWVj1YGy8H2gvKHv6J6nTU5yt7G
executed transaction: 29ad947d7813094afd84dc6fe49fe983f5f245cac82dcaba5dc7b665e991c86c  200 bytes  268 us
#         eosio <= eosio::newaccount            {"creator":"eosio","name":"user","owner":{"threshold":1,"keys":[{"key":"EOS5oR8TdbycNttjL4hYe5KSwGuW...
warning: transaction executed locally, but may not be confirmed by the network yet    ]
```

查看账号信息
```
root@6af410e7fc7e:/# cleos get account  user
permissions: 
     owner     1:    1 EOS5oR8TdbycNttjL4hYe5KSwGuWVj1YGy8H2gvKHv6J6nTU5yt7G
        active     1:    1 EOS5oR8TdbycNttjL4hYe5KSwGuWVj1YGy8H2gvKHv6J6nTU5yt7G
memory: 
     quota:       unlimited  used:      2.66 KiB  

net bandwidth: 
     used:               unlimited
     available:          unlimited
     limit:              unlimited

cpu bandwidth:
     used:               unlimited
     available:          unlimited
     limit:              unlimited
```

## 开发和部署合约

进入/work目录,使用命令创建合约hello合约
```
root@6af410e7fc7e:/# cd /work
```
```
root@6af410e7fc7e:/work# eosiocpp -n hello
created hello from skeleton
```

进入hello目录，编译合约
```
root@6af410e7fc7e:/work# cd hello
```
```
root@6af410e7fc7e:/work/hello# eosiocpp -o hello.wast hello.cpp
```

```
root@6af410e7fc7e:/work/hello# eosiocpp -g hello.abi hello.cpp
```
部署合约

```
root@6af410e7fc7e:/work/hello#  cleos  set contract user ../hello  -p user@active
Reading WAST/WASM from ../hello/hello.wasm...
Using already assembled WASM...
Publishing contract...
executed transaction: a2c9c91db47b510f72f43fc4e5150f22dd04549d4e781244749b1701b63e9a59  1792 bytes  4555 us
#         eosio <= eosio::setcode               {"account":"user","vmtype":0,"vmversion":0,"code":"0061736d01000000013b0c60027f7e006000017e60027e7e0...
#         eosio <= eosio::setabi                {"account":"user","abi":"0e656f73696f3a3a6162692f312e30000102686900010475736572046e616d6501000000000...
warning: transaction executed locally, but may not be confirmed by the network yet    ] 
```

调用合约
```
root@6af410e7fc7e:/work/hello# cleos push action user hi '["user"]' -p user@active
executed transaction: ff3d2f2c7d03a0a01778c25b6fa7453f4eaf87d3097c6934d27b79c3f6132b89  104 bytes  3368 us
#          user <= user::hi                     {"user":"user"}
>> Hello, user
warning: transaction executed locally, but may not be confirmed by the network yet    ] 
```

### 部署合约到测试网络
我们使用CryptoKylin Testnet

这里可以注册免费的账号和领取eos
https://tools.cryptokylin.io/#/tools/create
我们这注册了账号为aaaaaaaannnn并领取了一定数量的eos

我们定义一个新的命令，将cleos指像Testnet某个bp节点的地址
```
root@6af410e7fc7e:/work/hello# alias cltest='cleos -u http://kylin.fn.eosbixin.com'
```
查看网络信息，chain_id跟之前不一样了，指向了新的链
```
root@6af410e7fc7e:/work/hello# cltest get info
{
  "server_version": "e87d245d",
  "chain_id": "5fff1dae8dc8e2fc4d5b23b2c7665c97f9e9d8edf2b6485a86ba311c25639191",
  "head_block_num": 7863045,
  "last_irreversible_block_num": 7862709,
  "last_irreversible_block_id": "0077f9b5b96a276d10d0a6c8f08d9a431b78fdee833ec77945d6ef21b7d276a1",
  "head_block_id": "0077fb05e65dd124c212e1ef748a594f8a07106d51e4bcac9477bcd95db6184b",
  "head_block_time": "2018-08-31T03:33:11.500",
  "head_block_producer": "eosthueosthu",
  "virtual_block_cpu_limit": 200000000,
  "virtual_block_net_limit": 1048576000,
  "block_cpu_limit": 167533,
  "block_net_limit": 1044800
}
```

创建Testnet钱包

```
root@6af410e7fc7e:/work/hello# cltest wallet create -n testnet
Creating wallet: testnet
Save password to use in the future to unlock this wallet.
Without password imported keys will not be retrievable.
"PW5JGEZzfMX85xCi36UFrvbSCPDKWRNGhgZv4XsnbmyoB9omeAbKs"
```

导入前面申请的账号aaaaaaaannnn私钥到钱包
```
root@6af410e7fc7e:/work/helll# cltest wallet import -n testnet
private key: 5JRxGG2SxH5QmUJBXpuhLYs41o4XKVrhJxzq7wBUErozZnkTLGj
imported private key for: EOS823rdQMisWJoWMMJEjqi2Wm548BWSojVL63M7HRV2ec2WKinpJ
```

给账号买点ram
```
root@6af410e7fc7e:/work/hello# cltest system buyram aaaaaaaannnn aaaaaaaannnn "10 EOS"
2018-08-31T03:44:10.187 thread-0   main.cpp:438                  create_action        ] result: {"binargs":"30e79cc618638c3130e79cc618638c31a08601000000000004454f5300000000"} arg: {"code":"eosio","action":"buyram","args":{"payer":"aaaaaaaannnn","receiver":"aaaaaaaannnn","quant":"10.0000 EOS"}} 
executed transaction: b04917bdc0e9d61830f446ea96417d2050e107531a218b6cc9918bb7168c1c17  128 bytes  12500 us
#         eosio <= eosio::buyram                {"payer":"aaaaaaaannnn","receiver":"aaaaaaaannnn","quant":"10.0000 EOS"}
#   eosio.token <= eosio.token::transfer        {"from":"aaaaaaaannnn","to":"eosio.ram","quantity":"9.9500 EOS","memo":"buy ram"}
#  aaaaaaaannnn <= eosio.token::transfer        {"from":"aaaaaaaannnn","to":"eosio.ram","quantity":"9.9500 EOS","memo":"buy ram"}
#     eosio.ram <= eosio.token::transfer        {"from":"aaaaaaaannnn","to":"eosio.ram","quantity":"9.9500 EOS","memo":"buy ram"}
#   eosio.token <= eosio.token::transfer        {"from":"aaaaaaaannnn","to":"eosio.ramfee","quantity":"0.0500 EOS","memo":"ram fee"}
#  aaaaaaaannnn <= eosio.token::transfer        {"from":"aaaaaaaannnn","to":"eosio.ramfee","quantity":"0.0500 EOS","memo":"ram fee"}
#  eosio.ramfee <= eosio.token::transfer        {"from":"aaaaaaaannnn","to":"eosio.ramfee","quantity":"0.0500 EOS","memo":"ram fee"}
warning: transaction executed locally, but may not be confirmed by the network yet    ] 
```

部署合约

```
root@6af410e7fc7e:/work/hello# cltest  set contract aaaaaaaannnn ../hello   -p aaaaaaaannnn@active
Reading WAST/WASM from ../hello/hello.wasm...
Using already assembled WASM...
Publishing contract...
executed transaction: f38fe45f1c4148fa51833e4d1147bb803efa031f49b30c33712ea8d065a571fa  1800 bytes  2415 us
#         eosio <= eosio::setcode               {"account":"aaaaaaaannnn","vmtype":0,"vmversion":0,"code":"0061736d01000000013b0c60027f7e006000017e6...
#         eosio <= eosio::setabi                {"account":"aaaaaaaannnn","abi":"0e656f73696f3a3a6162692f312e30000102686900010475736572046e616d65010...
warning: transaction executed locally, but may not be confirmed by the network yet    ] 
```
调用合约
```
root@6af410e7fc7e:/work/hello# cltest push action aaaaaaaannnn hi '["aaaaaaaannnn"]' -p aaaaaaaannnn@active
executed transaction: 873e2f3ad9e7dca43d2ec1f68aadcd9ff84dfc22825494b711f5840f92c323c4  104 bytes  857 us
#  aaaaaaaannnn <= aaaaaaaannnn::hi             {"user":"aaaaaaaannnn"}
>> Hello, aaaaaaaannnn
warning: transaction executed locally, but may not be confirmed by the network yet    ] 

```

### 部署合约到主链

方法跟测试链一致，这里就不重复了






