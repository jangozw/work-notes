
# lotus 部署

生产环境lotus部署手册


## 基础环境 
1. 运行依赖

Base
```shell
sudo apt install build-essential hwloc libhwloc-dev wget jq  pkg-config -y

sudo apt install mesa-opencl-icd ocl-icd-opencl-dev gcc git bzr jq pkg-config curl clang build-essential hwloc libhwloc-dev wget -y && sudo apt upgrade -y


```

Rust
```shell
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

```

Go
```shell
wget https://go.dev/dl/go1.17.6.linux-amd64.tar.gz

tar -zxvf go1.17.6.linux-amd64.tar.gz

sudo cp -R go /usr/local/

```



2. 参数获取
一般情况下，32G证明参数一共103G, 可以保存起来复制，遇到官方重大更新会提示更新证明参数。
参数默认在： /var/tmp/filecoin-proof-parameters
可以通过环境变量设置其他路径如: FIL_PROOFS_PARAMETER_CACHE=/data/lotus-work/filecoin-proof-parameters/


查看证明参数目录：

```shell
env | grep FIL_PROOFS_PARAMETER_CACHE 
```
重新获取参数
```shell
lotus fetch-params {sectorSize}
```


## 统一配置
尽量按官方默认:

1. /filecoin/ 挂载ssd 
2. ~/.lotus         为lotus数据目录     软链至 /filecoin/.lotus
3. ~/.lotusworker   为worker数据目录    软链至 /filecoin/.lotusworker
4. ~/.lotusminer    为miner数据目录     软链至 /filecoin/.lotusminer
5. /var/tmp/filecoin-proof-parameters  软连至 /filecoin/filecoin-proof-parameters
6. /var/tmp/filecoin-parents           软连至 /filecoin/filecoin-parents
7. /filecoin/logs   为统一日志目录
8. 统一服务器用户名
9. 统一hostname格式,包含ip信息。 如 XG-172-1-1-1


## 环境变量

* RUST_LOG=debug # 开启打印rust日志
* FFI_BUILD_FROM_SOURCE=1 # 从本地filecoini-ffi/rust 编译rust代码， 默认是下载编译的不走本地



配置软链:
```shell
ln -s /filecoin/.lotus /home/user/.lotus
```


## lotus同步
1. 下载最近的数据

非全节点同步只需要从官网说明下载最新的lotus数据导入再开启同步，会在最新基础上更新， 大小为50G。
如果要同步全节点数据，需要下载所有数据

```shell
nohup curl -sI https://fil-chain-snapshots-fallback.s3.amazonaws.com/mainnet/minimal_finality_stateroots_latest.car | perl -ne '/x-amz-website-redirect-location:\s(.+)\.car/ && print "$1.sha256sum\n$1.car"' | xargs wget > download_snap.log  2>&1 &
```

2. 导入下载的数据
```shell
lotus daemon --import-snapshot /home/user/download/minimal_finality_stateroots_1401120_2021-12-24_10-00-00.car --halt-after-import
```
3. 开启同步
```shell
nohup lotus daemon >/filecoin/logs/lotus.log 2>&1 &
```
4. 查看同步状态
```shell
lotus sync status

lotus sync wait
```


## 创建矿工

先创建钱包, 再创建矿工

1. 创建钱包

```shell
lotus wallet new  bls
```
执行完成就得出了owner地址


2. 创建矿工

```shell
lotus-miner init --owner=<owner地址> --sector-size=32GiB
```

创建完以后生成一个矿工号: f0xxxx
生成~/.lotusminer 目录, 这个目录为矿工的初始化目录，需要备份一下。 便于拆分miner后共享

3. 修改miner配置

```vi ~/.lotusminer/config.toml``` 修改miner运行参数，要与机器性能匹配

```
# example
[API]
  # miner 自己的服务, 便于worker与其通信
  ListenAddress = "/ip4/0.0.0.0/tcp/2345/http"
[Sealing]
  FinalizeEarly = true
  BatchPreCommits = false
  AggregateCommits = false
[Storage]
  ParallelFetchLimit = 10
  AllowAddPiece = false
  AllowPreCommit1 = false
  AllowPreCommit2 = false
  AllowCommit = false
  AllowUnseal = false
```

4. 关闭miner存储

cat sectorstore.json
```
{
  "ID": "d0f90006-1b59-47d1-b7e0-d59da908b0b4",
  "Weight": 10,
  "CanSeal": false,
  "CanStore": false,
  "MaxStorage": 0
}
```

5. miner 连接lotus的配置

在家目录下新建.lotus目录，并拷贝lotus的api及token文件，新建datastore文件夹
配置修改/etc/profile,并 source /etc/profile里export RUST_LOG=Trace	

6. 启动miner:

```shell
nohup lotus-miner run > /filecoin/logs/miner.log 2>&1 &
```

## 启动worker
* 将lotus-miner, lotus-worker程序拷贝至 /usr/local/bin
* 在家目录下新建.lotusminer目录，并拷贝miner的api及token文件，新建datastore文件夹
* 配置修改/etc/profile,并 source /etc/profile
```
export RUST_LOG=Trace	
export FIL_PROOFS_MAXIMIZE_CACHING=1
export FIL_PROOFS_USE_MULTICORE_SDR=1
export FIL_PROOFS_USE_GPU_TREE_BUILDER=1
export FIL_PROOFS_USE_GPU_COLUMN_BUILDER=1
```
* 运行P1:
```shell
nohup lotus-worker --worker-repo=/filecoin/.lotusworker run --listen=172.16.1.111:3450 --parallel-fetch-limit=28 --addpiece=true --precommit1=true --precommit2=true --commit=false --unseal=false  > /filecoin/logs/p1_3450-"$(date +%Y%m%d-%H%M%S)".log 2>&1 &
```
* 运行C2:
```shell
nohup lotus-worker --worker-repo=/filecoin/.lotusworker run --listen=172.16.1.105:3450 --parallel-fetch-limit=1 --addpiece=false --precommit1=false --precommit2=false --commit=true --unseal=false  > /filecoin/logs/c2_3450-"$(date+%Y%m%d-%H%M%S)".log 2>&1 &
```


demo:

```shell
lotus-worker --worker-repo=/filecoin/.lotusworker run --listen=172.16.1.110:3450 --parallel-fetch-limit=20 --addpiece=true --precommit1=true --precommit2=true --commit=false --unseal=false
```

1. 查看miner worker

```
lotus-miner sealing workers
```

## 查看硬件配置


## 错误类型 errors
1. OpenCL 错误 not such file found

(root) cd /usr/lib && ln -s /usr/lib/x86_64-linux-gnu/libOpenCL.so.1.0.0 libOpenCL.so


查看外网地址
curl ifconfig.me 


2. keystore 的权限必须是600或者700
ERROR: initializing node: starting node: could not build arguments for function "github.com/filecoin-project/lotus/node/modules/lp2p".PstoreAddSelfKeys (/home/ipfs/work/lotus/node/modules/lp2p/libp2p.go:97): failed to build peer.ID: could not build arguments for function "reflect".makeFuncStub (/usr/local/go/src/reflect/asm_amd64.s:30): failed to build crypto.PubKey: could not build arguments for function "reflect".makeFuncStub (/usr/local/go/src/reflect/asm_amd64.s:30): failed to build crypto.PrivKey: received non-nil error from function "reflect".makeFuncStub (/usr/local/go/src/reflect/asm_amd64.s:30): permissions of key: 'libp2p-host' are too relaxed, required: 0600, got: 0777

.lotus/keystore 的权限必须是600或者700

```
user@yz-p-lotus-server01:/filecoin/.lotus$ ll
total 9953004
drwxrwxrwx 10 user user         270 Jan  5 17:22 ./
drwxr-xr-x  5 user user          54 Jan  5 17:21 ../
-rw-r--r--  1 user user          28 Jan  5 17:22 api
-rwxrwxrwx  1 user user        5058 Dec 24 14:41 config.toml*
drwxrwxrwx  6 user user          84 Dec 24 14:41 datastore/
drwxrwxrwx  2 user user           6 Dec 24 14:41 data-transfer/
drwxrwxrwx  2 user user           6 Dec 24 14:41 heapprof/
drwxrwxrwx  2 user user           6 Dec 24 14:41 imports/
drwxrwxrwx  2 user user        4096 Jan  5 17:22 journal/
drwx------  2 user user         327 Jan  4 11:43 keystore/
drwxrwxrwx  3 user user          30 Dec 24 14:41 kvlog/
-rwxrwxrwx  1 user user 10191855010 Jan  5 15:50 lotus.log*
-rw-rw-r--  1 user user           0 Jan  5 17:22 repo.lock
drwxrwxrwx  2 user user           6 Dec 24 14:41 retrievals/
-rwxrwxrwx  1 user user         136 Dec 24 14:41 token*

```

## miner 配置不做任务

任务让woker做， miner 不做

```shell
cat > ~/.lotusminer/config.toml << EOF
[API]
ListenAddress = "/ip4/0.0.0.0/tcp/2345/http"
[Storage]
AllowAddPiece = true
AllowPreCommit1 = true
AllowPreCommit2 = true
AllowCommit = true
[Sealing]
BatchPreCommits = false
AggregateCommits = false
EOF

```

## 修改rust代码本地编译设置

环境变量设置FFI_BUILD_FROM_SOURCE=1， rust部分将从本地filecoin-ffi/rust编译(默认是下载官方编译) 参考编译脚本filecoin-ffi/install-filcrypto

mac系统：

rust self uninstall

curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

cargo install cargo-lipo

rustup target add aarch64-apple-darwin

rustup install nightly

rustup +nightly target add aarch64-apple-darwin


rustup default nightly

cargo install xargo

rustup target add thumbv7m-none-eabi --toolchain nightly

rustup component add rust-src

// -Z build-std
/Users/django/.rustup/toolchains/nightly-2021-07-29-x86_64-apple-darwin/bin/cargo --color auto build -Z build-std -p filcrypto --target aarch64-apple-darwin --release --lib --no-default-features --features multicore-sdr,opencl 



cargo +nightly-2021-07-29 lipo --release --targets x86_64-apple-darwin,aarch64-apple-darwin --no-default-features --features multicore-sdr,opencl

git remote add tmp git@tmp.gitlab:filecoin-project-improve/lotus.git

git remote add tmp git@tmp.gitlab:filecoin-project-improve/filecoin-ffi.git

rust 编译分cgo, rust 两个部分 rust部分可以用cargo build 单独测试。 如果rust没问题， 那就是cgo有问题, 跟xcode, gcc,g++有关

遇到 can't find core, std 参考: https://juejin.cn/post/6844903760158785544


### 操作步骤

1. git clone 官方lotus
2. 编译官方lotus
3. 在extern/filecoin-ffi/rust 里修改cargo.toml 将依赖包改为本地
4. 克隆这个https://github.com/filecoin-project/rust-fil-proofs.git， 修改对应的路径, 用git log查看切换到里面5个库通用的tag



