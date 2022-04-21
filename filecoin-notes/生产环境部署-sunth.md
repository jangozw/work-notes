1. lotus
1.1 运行lotus节点，并同步正常


2. miner 
2.0 将lotus, lotus-miner程序拷贝至 /usr/local/bin
2.1 在lotus节点新建owner钱包地址：
	lotus wallet new  bls
2.2 使用owner钱包创建miner:
	lotus-miner init --owner=<owner地址> --sector-size=32GiB
2.3 将要新生成 .lotusminer 文件夹拷贝之miner机器的家目录下
2.4 修改.lotusminer下的config配置文件：

[API]
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

2.5 在家目录下新建.lotus目录，并拷贝lotus的api及token文件，新建datastore文件夹
2.7 配置修改/etc/profile,并 source /etc/profile
	export RUST_LOG=Trace	
2.6 启动miner:
	nohup lotus-miner run > /filecoin/logs/miner.log 2>&1 &



3. worker 
3.0 将lotus-miner, lotus-worker程序拷贝至 /usr/local/bin
3.1 在家目录下新建.lotusminer目录，并拷贝miner的api及token文件，新建datastore文件夹
3.2 配置修改/etc/profile,并 source /etc/profile
	export RUST_LOG=Trace	
	export FIL_PROOFS_MAXIMIZE_CACHING=1
	export FIL_PROOFS_USE_MULTICORE_SDR=1
	export FIL_PROOFS_USE_GPU_TREE_BUILDER=1
	export FIL_PROOFS_USE_GPU_COLUMN_BUILDER=1
3.3 运行P1:
	nohup lotus-worker --worker-repo=/filecoin/.lotusworker run --listen=172.16.1.111:3450 --parallel-fetch-limit=28 --addpiece=true --precommit1=true --precommit2=true --commit=false --unseal=false  > /filecoin/logs/p1_3450-"$(date +%Y%m%d-%H%M%S)".log 2>&1 &

3.4 运行C2:
	nohup lotus-worker --worker-repo=/filecoin/.lotusworker run --listen=172.16.1.105:3450 --parallel-fetch-limit=1 --addpiece=false --precommit1=false --precommit2=false --commit=true --unseal=false  > /filecoin/logs/c2_3450-"$(date+%Y%m%d-%H%M%S)".log 2>&1 &
