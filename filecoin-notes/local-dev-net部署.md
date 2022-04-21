# Build Lotus dev net

build 2k OR 512m local dev net of lotus


## 1. Reset env
```shell
rm -rf ~/.lotus* ~/.genesis-sectors ~/dev.gen dev.gen localnet.json ~/localnet.json 
```

## 2. Build source

cd ./lotus

```shell
# (512M扇区也用make 2k的编译)
make 2k && make install
```
## 3. Download params
```shell
# 2k
lotus fetch-params 2048
```
OR
```shell
# 512m
lotus fetch-params 512m
```

## 4. Pre-seal

预密封2个扇区, 将生成 .genesis-sectors/pre-seal-t01000.json 

```shell
lotus-seed pre-seal --sector-size 2KiB --num-sectors 2
```
OR
```shell
lotus-seed pre-seal --sector-size 512m --num-sectors 2
```

## 5. Gen localnet
```shell    
lotus-seed genesis new localnet.json
```

## 6. Gen genesis
```shell
lotus-seed genesis add-miner localnet.json ~/.genesis-sectors/pre-seal-t01000.json
```
## 7. Run lotus
```shell
# first time
lotus daemon --lotus-make-genesis=dev.gen --genesis-template=localnet.json --bootstrap=false
```
```shell
# second time
nohup lotus daemon --genesis=dev.gen --bootstrap=false > lotus.log 2>&1 &
```
## 8. Import wallet
```shell
lotus wallet import ~/.genesis-sectors/pre-seal-t01000.key
```
## 9. Init miner
```shell

lotus-miner init --genesis-miner --actor=t01000 --sector-size=2048 --pre-sealed-sectors=~/.genesis-sectors --pre-sealed-metadata=~/.genesis-sectors/pre-seal-t01000.json --nosync

```

## 10. Run miner
```shell
nohup lotus-miner run --enable-gpu-proving=false  --nosync > miner.log 2>&1 &
```

## 11. Run worker
```shell
nohup lotus-worker run --listen=127.0.0.1:3456 > worker.log  2>&1 &
```
## 12. sealing
执行一次派发一个任务密封
```shell
lotus-miner sectors pledge






```
 