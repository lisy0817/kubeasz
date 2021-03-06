## 升级注意事项

集群更新存在一定风险，请谨慎操作。 

- 项目分支1.8安装的集群目前只能进行小版本1.8.x的升级
- 项目分支1.9和master安装的集群可以任意小版本、大版本的升级，即1.9.x升级至1.10.x也可以

### 备份etcd数据 

- 升级前对 etcd数据做镜像备份  
``` bash
# snapshot备份
$ ETCDCTL_API=3 etcdctl snapshot save backup.db
# 查看备份
$ ETCDCTL_API=3 etcdctl --write-out=table snapshot status backup.db
```
- 从备份恢复可以参考[官方说明](https://github.com/coreos/etcd/blob/master/Documentation/op-guide/recovery.md)

### 升级步骤

- 1.下载最新项目代码 `git pull origin master`
- 2.下载新的二进制解压并覆盖 `/etc/ansible/bin/` 目录下文件
- 3a.如果可以接受短暂业务中断，执行 `ansible-playbook -t upgrade_k8s,restart_dockerd 22.upgrade.yml` 即可
- 3b.如果要求零中断升级集群
  - 首先执行 `ansible-playbook -t upgrade_k8s 22.upgrade.yml` (该步骤不会影响k8s上的业务应用)
  - 然后逐个升级重启每个node节点的dockerd服务
    - 待重启节点，先应用`kubectl cordon`和`kubectl drain`命令
    - 待重启节点执行 `systemctl restart docker`
    - 恢复待重启节点可调度 `kubectl uncordon`
