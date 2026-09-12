# Linux 学习笔记（2026.9.6）

> 📅 日期：2026.9.6
> 📚 配套教材：韩顺平 Linux
> 📖 今日内容：磁盘分区挂载（完整+自动挂载）→ 统计文件个数技巧 → 网络配置 → 主机名与hosts → 进程管理（开头）

---

## 一、磁盘分区与挂载（第8节 · 完整掌握）

### 1. 核心概念：什么是挂载？

> **挂载 = 把磁盘分区和一个目录绑定。绑定后，访问这个目录 = 访问这块磁盘。**

比喻：新硬盘是"一块空地"，分区=划地皮，格式化=打地基，挂载=开个门，进门=用这块地。

### 2. 查看磁盘的两个命令

| 命令 | 作用 | 场景 |
|---|---|---|
| `lsblk` | 看磁盘**结构**（树状）| 看新硬盘 |
| `df -h` | 看磁盘**空间使用**（容量/已用/挂载点）| 检查磁盘满没满 |

- `-h` = human 人性化显示（K/M/G），跟 `ls -lh` 同理
- 注意：`df -h` 看的是**磁盘空间**，不是内存！内存用 `free -h`

### 3. 设备名规则

- `sda` = 第一块硬盘、`sdb` = 第二块硬盘
- `sda1`、`sda2` = 第一块硬盘上的第1、2个分区
- 分区要先建文件系统（格式化）才能用

### 4. 新硬盘 4 步走（必背）

```
① 分区     fdisk /dev/sdb          → 划出分区 sdb1
② 格式化   mkfs.ext4 /dev/sdb1     → 建文件系统（ext4格式）
③ 建目录   mkdir /mnt/data         → 建个空目录当挂载点
④ 挂载     mount /dev/sdb1 /mnt/data → 绑定
```

- `mkfs` = make file system；`ext4` 是最常用格式（还有 xfs）
- 顺序：**分区 → 格式化 → 建目录 → 挂载**

### 5. 挂载与卸载命令

```bash
mount /dev/sdb1 /mnt     # 挂载：写【设备 + 目录】
umount /mnt              # 卸载：只写【挂载点目录】
```

- ⚠️ 卸载命令是 `umount`（u-mount，**没有 n**！不是 unmount）
- ⚠️ mount 要写设备+目录；umount 只要写目录

### 6. ⚠️ 大坑：手动挂载重启失效

- 手动 `mount` 的挂载，**重启后会失效**
- 要开机自动挂载 → 配置 `/etc/fstab`

---

## 二、自动挂载（/etc/fstab）

### 1. 为什么需要

手动 mount 重启就失效，写进 fstab 让系统**每次开机自动挂载**。

### 2. 配置文件

```bash
/etc/fstab     # fstab = file system table（文件系统表）
```

### 3. fstab 每行 6 个字段（核心）

```
设备    挂载点    文件系统类型    挂载选项    备份    自检
/dev/sdb1  /mnt/data  ext4  defaults  0  0
```

| 列 | 含义 | 常见值 |
|---|---|---|
| ① 设备 | 哪个分区 | /dev/sdb1 |
| ② 挂载点 | 挂到哪个目录 | /mnt/data |
| ③ 类型 | 文件系统 | ext4 / xfs |
| ④ 选项 | 挂载选项 | defaults |
| ⑤ 备份 | 0不备份 | 0 |
| ⑥ 自检 | 0不检查 | 0 |

> 前4列是重点（设备、挂载点、类型、defaults），后2列直接写 0。

### 4. 操作步骤

```bash
vim /etc/fstab        # 1.编辑配置文件
# 末尾追加一行：/dev/sdb1  /mnt/data  ext4  defaults  0 0
mount -a              # 2.不重启让配置生效并验证（-a=all）
reboot                # 3.重启后用 df -h 验证自动挂载成功
```

### 5. ⚠️ 最大风险

- fstab **写错可能导致开不了机**（系统启动必须读它）
- 安全建议：先手动挂载验证没问题再写；写完用 `mount -a` 测试；挂载点目录必须已存在

---

## 三、统计文件/目录个数（ls+grep+wc 综合技巧）

### 1. 核心原理

`ls -l` 每行开头第1个字符 = 类型标记：
- `-` 开头 = 普通文件
- `d` 开头 = 目录
- `l` 开头 = 软链接

### 2. 四个统计命令（必背）

```bash
# 统计普通文件个数（不含子目录）
ls -l /opt | grep "^-" | wc -l

# 统计目录个数（不含子目录）
ls -l /opt | grep "^d" | wc -l

# 统计文件个数（含子目录，递归）
ls -lR /opt | grep "^-" | wc -l

# 统计目录个数（含子目录，递归）
ls -lR /opt | grep "^d" | wc -l
```

### 3. 命令拆解

- `^` = 行开头（Shift + 6 打出来），`^-` = 以 `-` 开头
- `grep "^-"` 挑普通文件，`grep "^d"` 挑目录
- `wc -l` = 数行数 = 个数（l 是字母 L，不是竖线 |）
- 加 `-R` = 递归（连子文件夹里的也算）

### 4. 树状显示目录结构

```bash
tree /home/aaa    # 树状显示（没有就先 yum install -y tree）
```

---

## 四、网络配置（第9节）

### 1. 查看网络命令（CentOS7）

```bash
ip addr                          # 查看IP地址和网卡（CentOS7用这个！）
ifconfig                         # 旧命令，CentOS7默认没装，别用
ping www.baidu.com               # 测试外网连通，Ctrl+C 停止
systemctl restart network        # 重启网络（改完配置生效）
```

> ⚠️ 网络配置的易错：CentOS7 查看 IP 用 `ip addr`，不是 `ifconfig`！

### 2. 网卡配置文件路径（必背）

```bash
/etc/sysconfig/network-scripts/ifcfg-ens33
```

> 规律：`/etc/sysconfig/network-scripts/` 目录 + `ifcfg-` + 网卡名（ens33）

### 3. 配置文件关键项

```
BOOTPROTO=dhcp      # dhcp自动获取 / static静态IP
ONBOOT=yes          # 开机启用网卡（必须yes）
IPADDR=192.168.1.100    # 静态IP地址
NETMASK=255.255.255.0   # 子网掩码
GATEWAY=192.168.1.1     # 网关
DNS1=8.8.8.8            # DNS
```

### 4. 动态IP vs 静态IP

| | 动态 dhcp | 静态 static |
|---|---|---|
| IP来源 | 路由器自动分配 | 手动写死 |
| 特点 | 重启可能变 | 永远不变 |
| 用途 | 普通上网 | 服务器（IP要固定）|

---

## 五、主机名和 hosts 映射

### 1. 查看/改主机名

```bash
hostname                              # 查看当前主机名（如localhost）
hostnamectl set-hostname 新名字       # CentOS7永久改主机名
hostnamectl                           # 查看详细信息
```

### 2. hosts 映射（通讯录）

> hosts = "IP地址 ↔ 名字"对照表，让系统能按名字找机器，相当于手机的通讯录。

```bash
cat /etc/hosts        # 查看映射表
vim /etc/hosts        # 编辑，格式：IP 名字
```

示例（给 IP 起名叫 mydb）：
```
192.168.1.100   mydb
```
之后就能用 `ping mydb` 或 `ssh mydb` 访问那台机器。

---

## 六、进程管理（第10节 · 开头）

### 1. 什么是进程

- 程序 = 硬盘上的文件；进程 = 程序运行起来的样子
- 每个进程有唯一编号 **PID**

### 2. 查看进程：ps

```bash
ps -aux    # 看所有进程+资源占用（%CPU %MEM）
ps -ef     # 看进程父子关系（PPID父进程号）
ps -ef | grep sshd    # 过滤只看sshd（配合管道）
```

- `ps -aux` 适合找"谁吃资源多"
- `ps -ef` 适合查"谁是谁的儿子"

### 3. 动态监控：top

```bash
top    # 实时动态监控（像任务管理器），退出按 q
```

- `ps` 看瞬间快照，`top` 看实时直播

### 4. 杀进程：kill

```bash
kill PID        # 温柔退出
kill -9 PID     # 强制杀死（-9=强制信号，杀不掉时用）

# 完整三步：
ps -aux | grep 程序名    # 1.查PID
kill -9 PID              # 2.强杀
ps -aux | grep 程序名    # 3.确认没了
```

### 5. 进程状态 STAT

- `S` = 睡眠（正常等待）
- `R` = 正在运行
- `Z` = 僵尸进程（坏东西，要处理）

---

## 📌 今日易错点总结

| # | 易错点 | 正确 |
|---|---|---|
| 1 | 卸载命令拼错 | `umount`（没有n，不是unmount）|
| 2 | CentOS7看IP用ifconfig | 用 `ip addr` |
| 3 | wc -l 打成竖线 | `wc -l`（字母L）|
| 4 | grep 筛选漏 `^` | `grep "^-"`（^=行开头，Shift+6）|
| 5 | df -h 看成内存 | df看磁盘空间，内存用free |
| 6 | 忘记自动挂载文件 | `/etc/fstab` |
| 7 | 手动mount重启还在？ | 会失效！要配fstab |
| 8 | 进程查询不看父子关系 | 看PPID用 `ps -ef` |