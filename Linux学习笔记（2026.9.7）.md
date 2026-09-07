# Linux 学习笔记（服务管理·网络监控·软件管理）

> 📅 日期：______
> 📚 配套教材：韩顺平 Linux 课程
> 📖 今日内容：进程管理收尾、服务管理、防火墙开放端口、top动态监控、netstat/ping网络监控、RPM与YUM

---

## 一、服务管理（service）

### 1. 什么是服务

**服务 = 开机后一直在后台运行的进程**（守护进程 daemon），服务名常以 `d` 结尾：
- sshd（远程）、crond（定时）、httpd（网站）、mysqld（数据库）、firewalld（防火墙）、network（网络）

### 2. CentOS7 服务管理核心命令（systemctl）⭐

```bash
systemctl start 服务名        # 启动
systemctl stop 服务名         # 停止
systemctl restart 服务名      # 重启（先停再启）
systemctl status 服务名       # 查看状态
systemctl enable 服务名       # 设置开机自启
systemctl disable 服务名      # 取消开机自启
systemctl is-enabled 服务名   # 查看是否自启
systemctl list-unit-files     # 列出所有服务
```

### 3. 查看状态的关键词

```bash
systemctl status sshd
```
- `active (running)` = 正在运行
- `inactive (dead)` = 已停止
- `enabled` = 已开机自启
- `disabled` = 未开机自启

### 4. ⚠️ start vs enable（面试必问）

| 命令 | 管什么 | 特点 |
|---|---|---|
| `start` | **立即启动**（现在开）| 重启后失效 |
| `enable` | **开机自启**（以后自动开）| 现在不启动 |

> 记忆：**start=现在开、enable=以后开机自动开**。两者独立，想都要就都执行。

```bash
systemctl start sshd && systemctl enable sshd   # 现在开 + 以后自启
```

### 5. 运行级别（runlevel）

系统 7 种运行级别（0~6）：

| 级别 | 含义 | 能否设默认 |
|---|---|---|
| 0 | **关机**（停机状态）| ❌ 不能（一开机就关机）|
| 1 | 单用户（维护，禁远程）| — |
| 2 | 多用户无网络 | — |
| 3 | **命令行模式**（有网络）| ✅ 最常用 |
| 4 | 保留未使用 | — |
| 5 | **图形界面**（GUI）| ✅ 常用 |
| 6 | **重启** | ❌ 不能（一开机就重启）|

**两种常用级别**：`3` = 命令行（服务器）、`5` = 图形界面（桌面）

**CentOS7 对应 target：**
```bash
systemctl get-default                                   # 查看默认
systemctl set-default multi-user.target                 # 默认命令行(=级别3)
systemctl set-default graphical.target                  # 默认图形(=级别5)
```

**临时切换：**
```bash
runlevel       # 查看当前级别
init 3         # 切到命令行
init 5         # 切到图形
init 0         # 关机
init 6         # 重启
```

### 6. chkconfig（CentOS6旧命令，了解即可）

```bash
chkconfig --list                    # 查看所有服务自启（旧）
chkconfig 服务名 --list             # 查看某服务各级别
chkconfig --level 5 服务名 on       # 级别5自启
chkconfig --level 5 服务名 off      # 级别5关闭
```
> CentOS7 已用 systemctl 取代，了解即可。改完要 reboot。

---

## 二、防火墙开放端口（firewalld）

### 1. 为什么要开放端口

防火墙开着时**默认拦截外部访问**。服务器上装了服务（网站80、数据库3306），外部要访问，就得在防火墙**放行对应端口**。
> 比喻：防火墙=门卫，默认谁都不放进来，要放行某端口才让外部进。

### 2. 防火墙服务管理

```bash
systemctl status firewalld       # 看防火墙状态
systemctl stop firewalld         # 临时关闭
systemctl start firewalld        # 开启
systemctl disable firewalld      # 永久关闭（开机不自启）
```

### 3. 开放/关闭端口（核心，firewall-cmd）⭐

```bash
# ① 开放端口（永久生效）
firewall-cmd --permanent --add-port=80/tcp

# ② 关闭端口
firewall-cmd --permanent --remove-port=80/tcp

# ③ 重新载入（必须！否则不生效）
firewall-cmd --reload

# ④ 查询端口是否开放
firewall-cmd --query-port=80/tcp
```

**参数说明：**
| 参数 | 含义 |
|---|---|
| `--permanent` | 永久生效（不加=临时，重启失效）|
| `--add-port` | 开放端口 |
| `--remove-port` | 关闭端口 |
| `--reload` | 重新载入，让修改生效（**必做**）|
| `--query-port` | 查询是否开放（yes/no）|
| 协议 | `80/tcp`（tcp 或 udp）|

**开放常用端口：**
```bash
firewall-cmd --permanent --add-port=80/tcp       # 网页
firewall-cmd --permanent --add-port=8080/tcp     # Tomcat
firewall-cmd --permanent --add-port=3306/tcp     # MySQL
firewall-cmd --reload                             # 都要reload
```

**⚠️ 拼写易错（一字不差！）：**
- `firewall-cmd` 是一个词，**中间没空格**
- `--permanent`（不是 premanent）
- `--query-port` 单个横杠（不是 query--port）
- 改完**必须 `--reload`** 才生效

---

## 三、top 动态监控进程

### 1. ps vs top

| | ps | top |
|---|---|---|
| 特点 | **静态**快照 | **动态**实时刷新 |
| 刷新 | 不刷新 | 默认3秒一刷 |
| 比喻 | 拍照片 | 看直播 |

### 2. top 选项

```bash
top -d 秒数     # 指定刷新间隔（默认3秒）
top -i          # 不显示空闲(idle)和僵尸(zombie)进程
top -p PID      # 只监控指定进程
```

### 3. top 交互按键（进入top后可按键）⭐

| 按键 | 作用 |
|---|---|
| `q` | 退出 top |
| `P` | 按 **CPU** 排序 |
| `M` | 按 **内存** 排序 |
| `k` | **杀掉**指定进程（输入PID）|
| `u` | 只显示**指定用户**的进程 |
| `1` | 查看每个CPU核心 |

### 4. top 上半部分怎么读

- **load average**: 3个数字 = 1分钟/5分钟/15分钟平均负载，越大越忙
- **Tasks**: 总进程数，看有没有 `zombie`（僵尸）
- **%Cpu(s)**: us用户、sy系统、**id空闲**（id高=CPU闲）
- **KiB Mem**: 内存总量/空闲/已用

> ⚠️ zombie（僵尸进程）= 已经结束但没被回收的残留进程，占资源不干活，多了要处理

---

## 四、网络监控 netstat & ping

### 1. 查看网络连接 netstat

```bash
netstat -anp                # 看所有网络连接+对应进程（最常用）
netstat -anp | grep sshd    # 只看某服务（如sshd）
netstat -tlnp               # 只看TCP监听
```

**选项：** `-a`所有、`-n`数字显示、`-p`显示进程(PID)

**输出解读**（例：`netstat -anp | grep sshd`）：
```
tcp  0  0  0.0.0.0:22  0.0.0.0:*  LISTEN  1234/sshd
```
- `0.0.0.0:22` = 监听22端口
- `LISTEN` = 正在监听（等着别人连）
- `1234/sshd` = PID 1234 的 sshd 进程

**常见状态：**
| 状态 | 含义 |
|---|---|
| LISTEN | **正在监听**（服务在等连接）|
| ESTABLISHED | **已建立连接**（正在通信）|
| TIME_WAIT | 连接关闭等待 |

> 看 netstat 输出觉得"乱"，是因为混着 Unix 本地socket（STREAM/CONNECTED），想只看网络连接加 `grep tcp` 或 `-t`。

### 2. 检测连通 ping

```bash
ping 192.168.1.1          # 测连通（记得跟地址）
ping www.baidu.com        # 测外网
ping -c 4 192.168.1.1     # 只ping 4次自动停（-c=count次数）
```
- 有响应 = 网络通；无响应/超时 = 不通
- **Ctrl + C** 停止（会显示汇总：发几个、收几个、**丢包率**）

### 3. netstat vs ping

| | netstat | ping |
|---|---|---|
| 查什么 | 本机**连接/端口**状态 | 测**到远程通不通** |
| 对象 | 自己机器 | 远程主机 |

> 记忆：netstat 查本机端口连接，ping 测去对方通不通

### 4. 网络排查思路（实战）

```bash
ping www.baidu.com          # 1.外网通不通
ping 自己IP                 # 2.本机通不通
netstat -anp | grep 80      # 3.80端口有没有服务监听
systemctl status firewalld  # 4.防火墙挡没挡
firewall-cmd --query-port=80/tcp  # 5.防火墙放行没
```

---

## 五、软件包管理 RPM & YUM

### 1. rpm vs yum 区别（面试常考）

| | RPM | YUM |
|---|---|---|
| 方式 | **本地**装安装包 | **在线**下载安装 |
| 依赖处理 | ❌ 不管 | ✅ 自动解决 |
| 场景 | 手里有安装包文件 | 日常首选 |

> yum = 应用商店（自动+依赖）；rpm = 手拿U盘自己装（缺依赖自己找）

### 2. YUM

```bash
yum install -y 软件名       # 安装（-y自动确认yes）
yum remove -y 软件名        # 卸载
yum list | grep 软件名       # 搜索软件
```

### 3. RPM

```bash
rpm -ivh 包名.rpm         # 安装（i安装 v过程 h进度条）
rpm -qa | grep 软件名      # 查询是否已装（qa=query all）
rpm -e 软件名             # 卸载（e=erase）
```

---

## 📌 今日重点速查表

```bash
# 服务管理（CentOS7核心）
systemctl start/stop/restart/status/enable 服务名

# 防火墙开放端口
firewall-cmd --permanent --add-port=80/tcp && firewall-cmd --reload

# top 动态监控
top -d 5 / top -p PID / top -i
# 按键：q退出、P按CPU、M按内存、k杀进程、u按用户

# 网络监控
netstat -anp | grep 服务名
ping -c 4 IP

# 软件管理
yum install -y 软件名
rpm -ivh 包名.rpm
```

---

## ⚠️ 今日易错点

| # | 易错 | 正确 |
|---|---|---|
| 1 | start / enable | start=立即启动、enable=开机自启 |
| 2 | 运行级别0和6 | 0关机、6重启，都不能设默认 |
| 3 | 级别3/5 | 3命令行、5图形（最常用）|
| 4 | firewall-cmd拼写 | 中间没空格、--permanent、单横杠query-port |
| 5 | 防火墙忘了reload | 开放端口必须 `firewall-cmd --reload` |
| 6 | top -4 | 只发4个包是 `ping -c 4`（-4是IPv4）|
| 7 | LISTEN/ESTABLISHED | LISTEN=正在监听、ESTABLISHED=已连接 |
| 8 | top按键 | q退出、M内存、P CPU |
| 9 | rpm/yum | rpm本地不解决依赖、yum在线自动解决 |