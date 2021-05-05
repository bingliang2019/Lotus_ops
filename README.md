# Lotus 运维

## 常用命令

### 环境变量

```
export LOTUS_PATH="/md0/daemon"
export WORKER_PATH="/md0/worker"
export FIL_PROOFS_PARAMETER_CACHE="/proof"
export IPFS_GATEWAY="https://proof-parameters.s3.cn-south-1.jdcloud-oss.com/ipfs/"

export BELLMAN_CUSTOM_GPU="GeForce RTX 3090:10496, GeForce RTX 3080:8704"

export FIL_PROOFS_MAXIMIZE_CACHING=1
unset USE_EXP_CACHE
export FIL_PROOFS_USE_GPU_COLUMN_BUILDER=1
export FIL_PROOFS_USE_GPU_TREE_BUILDER=1
```

### 查询类

```
查询 Miner 信息 ：lotus-miner info
查询 Worker 状态 ：lotus-miner sealing workers
查询 任务状态 ： lotus-miner sealing jobs

UUID查询: ll -l /dev/disk/by-uuid/ | awk -F ' ' '{print $9}'
```

### 操作类

#### 设置 软阵列 Raid0
```
fdisk /dev/sda     // g n 回车 回车 wq
fdisk /dev/sdb     // g n 回车 回车 wq
mkfs.ext4 /dev/sda1
mkfs.ext4 /dev/sdb1
mdadm -C -a yes /dev/md001 -l 0 -n 2 /dev/sd{a,b}1
```
#### 磁盘开机启动
```
/dev/disk/by-uuid/c4e44938-6c3d-43c9-b8e3-86cf95659a93 /worker ext4 noatime,nofail 0 0
```

### 显卡驱动
```
# 卸载显卡驱动
sudo apt-get --purge remove nvidia*
sudo apt autoremove
sudo apt-get --purge remove "*nvidia*"
```

3080:
```
https://cn.download.nvidia.com/XFree86/Linux-x86_64/460.39/NVIDIA-Linux-x86_64-460.39.run
```
#### 网卡设置
vim /etc/netpanl
     ```
                   network:
              ethernets:
                eno1np0(物理网卡地址):
                  addresses:
                  - 192.168.60.112/24  
                  gateway4: 192.168.60.1
                  nameservers:
                    addresses:
                    - 223.5.5.5
              version: 2
     ```
     ip a  查询网卡地址
     
####worker 换miner机
停止 worker进程  supervisorctl stop worker
查看进程状态supervisorctl status
修改 挂载脚本 mount.sh  注意路径和ip
修改 run_worker (接口、环境变量等)和在目标miner组对应的worker对照脚本修改注意下面的 ip设置成固定 端口号注意不能冲突
supervisorctl reread  & supervisorctl update 

最后查看进程状态  supervisorctl status


## 常见问题处理

```
报错 : lotus-miner: error while loading shared libraries: libhwloc.so.5: cannot open shared object file: No such file or directory
sudo ln -s /usr/lib/x86_64-linux-gnu/libhwloc.so /usr/lib/libhwloc.so.5
```

```
删除ret-wait任务
lotus-miner sealing abort $id
```
