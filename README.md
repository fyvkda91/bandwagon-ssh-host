# bandwagonhost ssh：新手完全指南，从购买 VPS 到成功登录 SSH

买了搬瓦工 VPS，卡在 SSH 登录这一步的朋友，举个手。

这种感觉很典型：后台开通了，IP 地址有了，打开终端，一行 `ssh root@你的IP` 敲下去，然后……转圈、超时、`Connection refused`，或者更让人崩溃的 `Permission denied`。

明明花了钱，服务器在那里，就是进不去。

这篇文章就是写给这种情况的。从 KiwiVM 面板里找到 SSH 所需的三样东西，到 Windows / macOS / Linux 三端实际连接，再到连不上时的排查思路，一步步来，跟着做就行。

顺带讲一下套餐选择，因为很多朋友 SSH 连不上，根本原因是买了入门线路，结果碰上 IP 被墙——选对套餐，能省很多麻烦。

---

## 一、准备阶段：你需要提前搞定这三样东西

SSH 登录一台 VPS，本质上只需要四个信息：**IP 地址、SSH 端口、用户名、密码**。

用户名不用费心找，搬瓦工的默认用户名统一是 `root`（全小写，不要大写）。

另外三样，全在 KiwiVM 控制面板里。

### 1.1 进入 KiwiVM 面板

购买后，登录搬瓦工后台，进入「My Services」，点击你的 VPS 旁边的「KiwiVM Control Panel」按钮，就能进入 KiwiVM 面板。

面板首页，你能直接看到：

- **Public IP address**：这就是你的服务器 IP 地址
- **SSH Port**：搬瓦工的 SSH 端口**不是默认的 22**，是系统随机分配的一个五位数端口，类似 `29875` 这种，每台机器不一样，必须记好

### 1.2 获取 root 密码

密码需要主动生成一次。KiwiVM 面板左侧菜单找到「Root password modification」，点进去。

**重要：目前搬瓦工要求 VPS 处于开机状态才能重置密码**（和以前相反，以前是要关机）。确认 VPS 是运行状态，点「Generate and set new root password」。

密码只会显示这一次，务必立刻复制保存到某个地方。忘记了怎么办？只能再重置一次。

---

## 二、SSH 客户端选择

有了 IP、端口、密码，接下来需要一个 SSH 客户端。

**Linux / macOS 用户**：直接用系统自带终端，不需要安装任何软件。

**Windows 用户**：推荐以下两款免费工具之一：

- **Termius**：跨平台、界面现代、可以同步配置，首选
- **Xshell**（免费个人版）：功能成熟，国内用户用得多

手机用户也可以下载 Termius 的手机版，配置和桌面端可以同步。

---

## 三、实际连接操作（分系统）

### 3.1 Linux / macOS 终端连接

打开终端，输入：

bash
ssh root@你的IP地址 -p 你的SSH端口号


比如：

bash
ssh root@192.168.1.1 -p 29875


回车，提示输入密码时把刚才保存的 root 密码粘贴进去（Linux/macOS 终端输入密码不显示字符，这是正常的，盲输然后回车即可）。

首次连接会弹出一个确认 fingerprint 的提示，输入 `yes` 回车，后续连接就不再提示了。

### 3.2 Windows 使用 Termius

1. 下载安装 Termius，注册一个免费账号
2. 点「New Host」
3. 填入：Hostname（IP 地址）、Port（SSH 端口）、Username（`root`）、Password（你的密码）
4. 保存，双击连接

### 3.3 Windows 使用 Xshell

1. 新建会话，协议选 SSH
2. 主机填 IP 地址，端口号填你的 SSH 端口（不是 22）
3. 用户身份验证，方法选「Password」，用户名 `root`，密码填入
4. 连接，首次会弹确认窗口，选「接受并保存」

连接成功后，你会看到 Linux 的命令行欢迎界面，类似 `root@xxx:~#`，恭喜你，进去了。

---

## 四、SSH 连不上？按这个顺序排查

SSH 连不上是新手最常见的卡点，原因其实就那几个，对号入座就行。

### 4.1 先确认 VPS 是否在运行

进 KiwiVM 面板，首页看状态是不是「Running」。如果显示 Stopped，点 Start 先开机。

### 4.2 测试 IP 和端口连通性

这是确诊关键。用 [tcp.ping.pe](https://tcp.ping.pe) 这个工具，输入你的 IP 和端口进行测试：

- **国内节点全红，国外节点全绿** → IP 被墙了，这是最麻烦的情况
- **IP 能 ping 通，但 SSH 端口全红** → 端口被封，或者 SSH 服务挂了

**IP 被墙的处理**：搬瓦工不提供免费换 IP，需要去后台付费申请更换，通常 24 小时内生效。这也是为什么推荐选 CN2 GIA-E 以上线路套餐——IP 被墙的概率低很多，因为 GIA 线路有专门的 DDoS 防护机制。

👉 [点击查看搬瓦工套餐，选择 CN2 GIA-E 等稳定线路方案](https://bwh81.net/aff.php?aff=77528)

**端口被封的处理**：进 KiwiVM 面板，找到「Root shell – interactive」，启动网页版 Shell，登录后修改 `/etc/ssh/sshd_config` 里的 Port 值，换一个 1 万以上的大端口，保存重启 sshd 服务。修改后要同时在 KiwiVM 面板记下新端口。

**SSH 服务挂了的处理**：在 KiwiVM 的网页 Root shell 里执行：

bash
systemctl status sshd   # 查看状态
systemctl start sshd    # 强制启动
sshd -t                 # 检查配置文件语法错误


### 4.3 密码错误 / Permission denied

如果能看到输入密码的界面，但提示 `Permission denied`，99% 是密码输错了，或者用户名大小写不对（必须是小写 `root`）。去 KiwiVM 面板重置一次密码，重新试。

---

## 五、SSH 连接成功后的基础安全设置

进去了，先别急着搞业务，做两件事能省掉很多后续麻烦：

### 5.1 修改 SSH 端口（如果还没改）

SSH 默认端口 22 每天会被大量自动扫描工具轮询，改成一个大端口（比如 45678）能显著减少无效登录尝试。修改方法参考上面 4.2 的端口修改流程。

### 5.2 配置 SSH 密钥登录（进阶）

密码登录相对不安全，配置 SSH 公钥认证后可以禁用密码登录，大幅提升安全性。KiwiVM 面板本身也支持管理 SSH 公钥（Public SSH Keys 功能）。

---

## 六、搬瓦工 VPS 套餐一览（含购买链接）

选对套餐，SSH 问题能少一半。CN2 GIA-E 以上线路 IP 稳定性明显好于入门线路，建站或有长期稳定需求的朋友建议直接上 CN2 GIA-E。

| 套餐系列 | 内存 | SSD | 月流量 | 带宽 | 线路 | 价格 | 购买 |
|---|---|---|---|---|---|---|---|
| KVM 入门（20G） | 1 GB | 20 GB | 1 TB | 1 Gbps | CN2 GT | $49.99/年 |  [立即购买](https://bwh81.net/aff.php?aff=77528&pid=57) |
| KVM 入门（40G） | 2 GB | 40 GB | 2 TB | 1 Gbps | CN2 GT | $52.99/半年 |  [立即购买](https://bwh81.net/aff.php?aff=77528&pid=58) |
| CN2 GIA-E 20G | 1 GB | 20 GB | 1 TB | 2.5 Gbps | CN2 GIA-E 三网 | $49.99/季 / $169.99/年 |  [立即购买](https://bwh81.net/aff.php?aff=77528&pid=87) |
| CN2 GIA-E 40G | 2 GB | 40 GB | 2 TB | 2.5 Gbps | CN2 GIA-E 三网 | $89.99/季 / $299.99/年 |  [立即购买](https://bwh81.net/aff.php?aff=77528&pid=88) |
| CN2 GIA-E 80G | 4 GB | 80 GB | 3 TB | 2.5 Gbps | CN2 GIA-E 三网 | $17.99/月 / $559.99/年 |  [立即购买](https://bwh81.net/aff.php?aff=77528&pid=89) |
| 香港 CN2 GIA（入门） | 2 GB | 40 GB | 500 GB | 1 Gbps | CN2 GIA 直连 | $89.99/月 / $899.99/年 |  [立即购买](https://bwh81.net/aff.php?aff=77528&pid=95) |
| 香港 CN2 GIA（高配） | 4 GB | 80 GB | 1 TB | 1 Gbps | CN2 GIA 直连 | $155.99/月 / $1559.99/年 |  [立即购买](https://bwh81.net/aff.php?aff=77528&pid=96) |

**说明：**
- CN2 GIA-E 系列可切换 DC6、DC9、日本软银、荷兰联通等多个机房，性价比最高
- 香港 CN2 GIA 延迟最低，适合对实时性要求极高的场景
- 优惠码 **NODESEEK2026** 可在结算时使用，享受全场常规套餐折扣

---

## 七、常见错误汇总

**问：SSH 端口为什么不是 22？**

搬瓦工所有 VPS 的 SSH 端口都是系统随机分配的，不是标准的 22。必须在 KiwiVM 面板首页查看实际端口，连接时用 `-p 端口号` 参数指定。

**问：密码重置失败，提示 739102 错误？**

这是因为 VPS 处于关机状态。先在 KiwiVM 面板把 VPS 开机，再去重置密码。

**问：连接时提示 Host key verification failed？**

这通常发生在换 IP 或重装系统之后。删除本地 `~/.ssh/known_hosts` 文件里对应 IP 的记录，或者整个文件删掉重新连接即可。

**问：搬瓦工 SSH 连接后自动断开？**

服务器端 SSH 空闲超时导致。在本地 SSH 配置文件（`~/.ssh/config`）里加上 `ServerAliveInterval 60` 参数，让客户端每 60 秒发一个心跳包保持连接。

---

## 八、小结

搬瓦工 SSH 登录的核心流程就三步：KiwiVM 拿信息（IP + 端口 + 密码）→ SSH 客户端填信息 → 连接。

坑基本就几个：端口号忘了用随机端口、密码在 VPS 开机状态下才能重置、IP 被墙需要换 IP。搞清楚这几点，SSH 登录基本不会卡太久。

想从一开始就省去 IP 被墙的烦恼，直接选 CN2 GIA-E 系列套餐是更省心的选择，线路质量和 IP 稳定性都好一个档次。

👉 [前往搬瓦工官方购买页，查看当前在售套餐](https://bwh81.net/aff.php?aff=77528)
