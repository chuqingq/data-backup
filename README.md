# 数据备份方案



## 问题背景

为了避免硬件损坏导致数据丢失，需要定时对关键业务数据进行备份。

## 详细需求

1. 源备份路径：`192.168.0.200:/home/dilu/rsync_data_src`
2. 目标备份路径：`192.168.0.201:/home/chuqq/rsync_data_dst`
3. 备份时间/间隔：每天凌晨2点
4. 备份保留时间：10天
5. 限制带宽（不能影响现网业务）：1000 KB/S
6. 跨主机备份：需要跨主机

## 方案

整体方案：crontab + rsync + tar

1. 手工添加定时任务

```shell
$ crontab -e
# For example, you can run a backup of all your user accounts
# at 5 a.m every week with:
# 0 5 * * 1 tar -zcf /var/backups/home.tgz /home/
# m h  dom mon dow   command
# 每天凌晨2点执行。周期和脚本自己按需修改
0 2 * * * /home/chuqq/rsync_data_dst/backup.sh 2>&1 >> /home/chuqq/rsync_data_dst/backup.log
```

2. 在目的端的备份文件夹下定时执行同步动作（backup.sh的内容）：

```shell
#!/bin/sh

set -e

echo "===="
echo `date +%F_%T`
echo "backup starting..."

## 下面这几个参数需要修改
BACKUP_SRC=dilu@192.168.0.201:/home/dilu/rsync_data_src
PASSWORD=dilumotors
KEEP_DAYS=10


## 下面的逻辑无需修改
BASE_DIR=`dirname $0`
cd ${BASE_DIR}
mkdir -p backup/

### 同步
sshpass -p${PASSWORD} rsync -avz --delete --bwlimit=1000 ${BACKUP_SRC} backup/

### 备份
tar zcvf backup-`date +%F_%H%M%S`.tar.gz backup/

### 删除过期的备份
find .  -maxdepth 1 -name "backup-*.tar.gz" -ctime +${KEEP_DAYS} -exec rm {} \;

echo "backup success"
```



## 需要备份的服务数据
