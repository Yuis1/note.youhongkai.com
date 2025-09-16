---
{"dg-publish":true,"permalink":"/CS计算机科学/运维部署/OS/Linux增加虚拟内存Swapfile/","noteIcon":"","created":"2025-07-31T09:55:12.372+08:00","updated":"2025-06-18T17:33:48.000+08:00"}
---

```
# 禁用现有 Swap（如果有）
sudo swapoff -a

# 创建新的 Swap 文件（示例：16GB）
sudo fallocate -l 16G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile   # 格式化为 Swap
sudo swapon /swapfile   # 启用

# 永久生效（写入 /etc/fstab）
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# 降低 swappiness（减少使用 Swap 的倾向，推荐 10~30）
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf

# 启用内存过量提交（避免 Redis 等应用失败）
echo 'vm.overcommit_memory=1' | sudo tee -a /etc/sysctl.conf

# 应用配置
sudo sysctl -p
```


## ​**​💡 替代方案：直接使用 LVM 或 ZFS​**​


- ​**​使用 LVM 创建 Swap 卷​**​（无文件系统开销）：
    
    ```
    sudo lvcreate -L 16G -n swapvg vg0  # 创建逻辑卷
    sudo mkswap /dev/vg0/swapvg         # 格式化
    sudo swapon /dev/vg0/swapvg         # 启用
    ```
    
- ​**​或者改用 ZFS​**​（支持原生 Swap 卷）：
    
    ```
    zfs create -V 16G -b 4K tank/swap   # 创建 ZVOL
    mkswap /dev/zvol/tank/swap          # 格式化
    swapon /dev/zvol/tank/swap          # 启用
    ```
    

---

## ​**​🚀 总结​**​

|步骤|命令|
|---|---|
|​**​删除旧文件​**​|`sudo swapoff /swapfile && sudo rm /swapfile`|
|​**​创建无空洞文件​**​|`sudo dd if=/dev/zero of=/swapfile bs=1M count=16384 status=progress conv=sparse`|
|​**​强制填充​**​|`sudo fallocate -l 16G /swapfile`|
|​**​设置权限​**​|`sudo chmod 600 /swapfile`|
|​**​格式化 Swap​**​|`sudo mkswap /swapfile`|
|​**​启用 Swap​**​|`sudo swapon /swapfile`|
|​**​永久生效​**​|`echo '/swapfile none swap sw 0 0'|

执行后，`free -h` 必须显示 Swap 已生效！如果仍然失败，建议改用 ​**​LVM/ZFS Swap 卷​**​ 或检查文件系统类型（`ext4` 可能需调整挂载选项）。