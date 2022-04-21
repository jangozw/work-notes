

##  查看内核错误
```shell
$ dmesg
```
结果: 
```txt
[23515.828684] [  11162]  1000 11162     2102      972    61440        0             0 bash
[23515.828686] [  11198]  1000 11198 546277824 521627605 4185128960        0             0 lotus-worker
[23515.828690] oom-kill:constraint=CONSTRAINT_NONE,nodemask=(null),cpuset=/,mems_allowed=0-1,global_oom,task_memcg=/user.slice/user-1000.slice/session-8.scope,task=lotus-worker,pid=11198,uid=1000
[23515.829731] Out of memory: Killed process 11198 (lotus-worker) total-vm:2185111296kB, anon-rss:2086510420kB, file-rss:0kB, shmem-rss:0kB, UID:1000 pgtables:4087040kB oom_score_adj:0
[23661.659484] oom_reaper: reaped process 11198 (lotus-worker), now anon-rss:0kB, file-rss:0kB, shmem-rss:0kB
```
显示出时间:
```shell
# 将23515.829731 替换为 dmesg的
date -d "1970-01-01 UTC `echo "$(date +%s)-$(cat /proc/uptime|cut -f 1 -d' ')+23515.829731"|bc `seconds"
```




sudo chown  shbw:shbw reseal.json


## 软链

ln -s /data/tmp /tmp （将/tmp 软链至 /data/tmp, /data/tmp 要存在， /tmp要不存在）

ln -s /filecoin/.lotusminer /home/ipfs/.lotusminer
ln -s /filecoin/.lotusworker /home/ipfs/.lotusworker

# 创建/tank1/.c2in/的软链 /root/.c2in
ln -s /tank1/.c2in/ /root/.c2in


## 查看文件夹大小

```shell
du -sh *
```


1. argumenterror: could not find a temporary directory
/tmp 目录的权限应该为: 
chmod +t /tmp

```txt
drwxrwxrwt   14 root root       4096 Jan  7 15:55 tmp/
```





lotus-miner sectors update-state --really-do-it  1 PreCommitting
lotus-miner sectors update-state --really-do-it  1 xxx

lotus-miner sectors list

lotus-miner sectors status --log 1



 手动发出batch 消息
 vi config.toml OR LOTUS_SEALING_BATCHPRECOMMITABOVEBASEFEE="1000000000000000000 FIL"
 lotus-miner sectors batching precommit  --publish-now


 ## 修改和编译rust部分代码



vi lotus/.gitmodules 

修改extern/filecoin-ffi的url: https://github.com/jangozw/filecoin-ffi.git

添加其他rust代码库到submodule:
git submodule add  https://github.com/jangozw/rust-filecoin-proofs-api.git ./extern/rust-filecoin-proofs-api

git submodule add  https://github.com/jangozw/rust-fil-proofs.git ./extern/rust-fil-proofs



extern/filecoin-ffi/rust/Cargo.toml修改filecoin-proofs-api的依赖路径:
path = "../../rust-filecoin-proofs-api"


extern/ust-filecoin-proofs-api/Cargo.toml 中替换

filecoin-proofs-v1 = { path = "../rust-fil-proofs/filecoin-proofs", version = "~11.0", default-features = false }
filecoin-hashers = { path = "../rust-fil-proofs/filecoin-hashers", version = "~6.0", default-features = false, features = ["poseidon", "sha256"] }
fr32 = { path = "../rust-fil-proofs/fr32", version = "~4.0", default-features = false }
storage-proofs-core = { path = "../rust-fil-proofs/storage-proofs-core", version = "~11.0", default-features = false }
storage-proofs-porep = { path = "../rust-fil-proofs/storage-proofs-porep", version = "~11.0", default-features = false }
