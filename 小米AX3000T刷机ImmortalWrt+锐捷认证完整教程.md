# 小米路由器 AX3000T 刷机 ImmortalWrt + 锐捷认证 完整教程

---

## 目录

- [一、准备说明](#一准备说明)
- [二、第一步：降级固件](#二第一步降级固件)
- [三、第二步：解锁 SSH 并固化](#三第二步解锁-ssh-并固化)
- [四、第三步：SSH 连接路由器](#四第三步ssh-连接路由器)
- [五、第四步：备份原厂系统 & 刷入 U-Boot](#五第四步备份原厂系统--刷入-u-boot)
- [六、第五步：刷入第三方固件](#六第五步刷入第三方固件)
- [七、第六步：安装锐捷认证（MentoHUST）](#七第六步安装锐捷认证mentohust)
- [八、恢复原厂系统（备用）](#八恢复原厂系统备用)
- [九、常见问题](#九常见问题)

---

## 一、准备说明

### 1.1 硬件版本判断

小米 AX3000T 有 **V1** 和 **V2** 两个硬件版本，不同版本使用的固件和 U-Boot 文件不同：

| 版本 | U-Boot 文件 |
|------|------------|
| V1 | `mt7981_ax3000t-v1multi-layout.bin` |
| V2 | `mt7981-ax3000tv2-an8855-fip-multi.bin` |

> ⚠️ **重要**：务必确认自己的硬件版本，刷错 U-Boot 会导致路由器变砖！可通过路由器底部的标签或小米后台查看。

### 1.2 所需文件清单

本教程所需的所有文件已整理在以下目录中：

```
📁 AX3000T傻瓜式刷机教程/
├── 📁 第一步解锁SSH/         # SSH解锁工具 + 降级固件
│   ├── 📁 v1/                # V1版本降级用固件
│   ├── 📁 v2/                # V2版本降级用固件 (1.0.91)
│   ├── MIWIFIRepairTool.x86.zip   # 小米路由器修复工具
│   └── ssh解锁工具.zip       # SSH解锁脚本
├── 📁 第二步刷入uboot/       # V1/V2 U-Boot 文件
│   ├── 📁 v1/                # V1 U-Boot + 刷入命令
│   └── 📁 v2/                # V2 U-Boot + 刷入命令
├── 📁 第三步刷入openwrt/     # KWRt 固件
│   └── kwrt-11.10.2025-...-sysupgrade.bin
├── 📁 3/                     # ImmortalWrt 固件
│   ├── immortalwrt-24.10.0-...-sysupgrade.bin  (推荐)
│   └── immortalwrt-25.12.1-...-sysupgrade.bin  (新版，用apk管理软件)
├── 📁 ruijie/                # 锐捷认证插件 (ipk格式)
├── 📁 恢复原厂系统/           # 恢复工具 + 原厂固件
├── 📁 原厂备份固件/           # 备份好的原厂分区文件
└── 📁 wrt/                   # 参考固件
```

---

## 二、第一步：降级固件

> 小米官方在新版固件中封堵了 SSH 解锁漏洞，因此需要先将路由器降级到可解锁的旧版本固件。

### 2.1 为什么需要降级？

- 如果你的路由器固件版本较新（特别是出厂版本 > 1.0.47 的 V1，或较新的 V2），可能无法直接解锁 SSH
- 降级到旧版本后，可以利用已知漏洞解锁 SSH
- V1 推荐降级到 1.0.47，V2 推荐降级到 1.0.91

### 2.2 操作步骤

1. 用**网线**将电脑连接到路由器 LAN 口

2. 解压 `第一步解锁SSH/MIWIFIRepairTool.x86.zip`，运行 **小米路由器修复工具**

3. 选择对应的降级固件上传刷入：

| 版本 | 降级固件文件路径 |
|------|-----------------|
| V1 | `第一步解锁SSH/v1/` 目录下的固件 |
| V2 | `第一步解锁SSH/v2/miwifi_rd03_firmware_7df60_1.0.91.bin` |

4. 等待降级完成，路由器自动重启

5. 降级后重新配置路由器（设置上网方式、WiFi 名称密码等），确保能正常联网

> ✅ 降级完成！现在路由器处于可解锁 SSH 的版本，继续下一步。

### 2.3 注意事项

- V1 版本如果当前固件版本 ≤ 1.0.47，可以跳过降级直接解锁
- V2 版本如果解锁工具能直接成功，也可以不降级；解锁失败再回来做这步
- 降级不会清除路由器设置，但建议降级后重置一次路由器再重新配置

---

## 三、第二步：解锁 SSH 并固化

### 3.1 准备工作

1. 用**网线**将电脑连接到路由器 LAN 口
2. 路由器已正常配置并能上网（处于正常使用状态）
3. 知道路由器的管理员密码

### 3.2 操作步骤

1. 打开 `a第一步-解锁固化SSH/xmir-patcher-main/` 目录（在 `小米AX3000T刷机` 文件夹中），双击运行 **`run.bat`**

2. 在程序菜单中选择 **`[1] Set IP-address`** 设置路由器 IP
   - 默认 IP：`192.168.31.1`
   - 如果你的路由器 IP 是默认值，直接回车确认

3. 选择 **`[2] Connect to device`** 连接路由器
   - 输入路由器的**管理员密码**，回车确认
   - 成功后程序会显示 22 端口和 SSH 已开启

4. 选择 **`[8] - {{{ Other functions }}}`**，然后选择 **`[2] 配置 root 密码`**
   - 设置一个你记得住的 root 密码（后续 SSH 连接会用到）
   - 回车确认

5. 选择 **`[7] - Install permanent SSH`** 固化 SSH
   - 这一步确保 SSH 在路由器重启后仍然有效
   - 等待程序提示完成

> ✅ 至此第二步完成！SSH 已解锁并固化，root 密码已配置。

---

## 四、第三步：SSH 连接路由器

### 4.1 使用 MobaXterm 连接

1. 打开 `b第二步-连接SSH/` 目录（在 `小米AX3000T刷机` 文件夹中），运行 **`MobaXterm_Personal.exe`**

2. 点击左上角 **Session（会话）** → 选择 **SSH**

3. 填写连接信息：
   - **Remote host（远程主机）**：`192.168.31.1`（路由器 IP）
   - 其他选项保持默认
   - 点击 **OK**

4. 在黑色终端窗口中：
   - 输入用户名：**`root`**，回车
   - 输入 root 密码：**上一步设置的密码**，回车

5. 连接成功后终端会显示 **OK** 字样

> ✅ 至此第三步完成！已成功通过 SSH 连接到路由器。

---

## 五、第四步：备份原厂系统 & 刷入 U-Boot

### 5.1 备份原厂系统

在 MobaXterm 终端中执行以下命令：

```bash
# 备份原厂系统固件（最重要的分区）
dd if=/dev/mtd8 of=/tmp/8.bin
```

备份完成后：
1. 进入 `/tmp` 目录：`cd /tmp`
2. 在 MobaXterm **左侧文件管理器**中打开 `/tmp` 文件夹
3. 找到 `8.bin` 文件，**拖拽到电脑桌面保存**
4. 建议将此备份文件上传到云盘妥善保管（恢复原厂系统时会用到）

### 5.2 完整备份（可选但推荐）

如果你希望做更完整的备份，可以逐条执行以下命令备份所有分区：

```bash
dd if=/dev/mtd1 of=/tmp/BL2.bin
dd if=/dev/mtd2 of=/tmp/Nvram.bin
dd if=/dev/mtd3 of=/tmp/Bdate.bin
dd if=/dev/mtd4 of=/tmp/Factory.bin
dd if=/dev/mtd5 of=/tmp/FIP.bin
dd if=/dev/mtd6 of=/tmp/crash.bin
dd if=/dev/mtd7 of=/tmp/crash_log.bin
dd if=/dev/mtd8 of=/tmp/ubi.bin
dd if=/dev/mtd9 of=/tmp/ubi1.bin
dd if=/dev/mtd10 of=/tmp/overlay.bin
dd if=/dev/mtd11 of=/tmp/date.bin
dd if=/dev/mtd12 of=/tmp/KF.bin
```

将所有备份文件拖到电脑保存。

### 5.3 刷入 U-Boot

1. 确定你的硬件版本（V1 或 V2），选择对应的 U-Boot 文件：

| 版本 | U-Boot 文件路径 |
|------|----------------|
| V1 | `第二步刷入uboot/v1/mt7981_ax3000t-v1multi-layout.bin` |
| V2 | `第二步刷入uboot/v2/mt7981-ax3000tv2-an8855-fip-multi.bin` |

2. 将对应版本的 U-Boot `.bin` 文件**拖入** MobaXterm 左侧的 `/tmp` 文件夹

3. 在终端中执行刷入命令：

**V1 版本：**
```bash
# 先校验文件完整性（可选）
md5sum /tmp/mt7981_ax3000t-v1multi-layout.bin

# 逐条复制粘贴执行
mtd write /tmp/mt7981_ax3000t-v1multi-layout.bin FIP
mtd verify /tmp/mt7981_ax3000t-v1multi-layout.bin FIP
```

**V2 版本：**
```bash
# 逐条复制粘贴执行
mtd write /tmp/mt7981-ax3000tv2-an8855-fip-multi.bin FIP
mtd verify /tmp/mt7981-ax3000tv2-an8855-fip-multi.bin FIP
```

> ⚠️ **注意**：命令中的文件名必须与你拖入的文件名**完全一致**！
>
> 如果 `mtd verify` 验证通过，说明 U-Boot 刷入成功。

### 5.4 查看分区信息（调试用）

```bash
cat /proc/mtd
```

> ✅ 至此第四步完成！原厂系统已备份，U-Boot 已刷入。

---

## 六、第五步：刷入第三方固件

U-Boot 刷入后，可以选择刷入 **KWRt** 或 **ImmortalWrt** 两种固件。两者都是基于 OpenWrt 的定制版本。

> ⚠️ **重要提示——ImmortalWrt 版本选择**：
>
> ImmortalWrt **较新版本（如 25.x）改用 `apk` 作为包管理器**，不再使用传统的 `opkg`。
> 而本教程的锐捷认证插件全部是 **`.ipk` 格式**，只能通过 `opkg` 安装。
> 因此 **必须刷入仍使用 opkg 的旧版本 ImmortalWrt**，推荐使用 **24.10.0**，否则第 六步的锐捷插件将无法安装！

### 方案一：KWRt（KWRT 固件）

KWRt 是另一个基于 OpenWrt 的第三方固件，自带了一些常用插件。

1. 进入 U-Boot 恢复模式（见下方 6.1 节）
2. 选择固件：`第三步刷入openwrt/kwrt-11.10.2025-mediatek-filogic-xiaomi_mi-router-ax3000t-squashfs-sysupgrade.bin`
3. 上传刷入，等待自动重启

### 方案二：ImmortalWrt（推荐 24.10.0）

ImmortalWrt 是目前社区最活跃的 OpenWrt 分支之一，更新及时，插件生态完善。

1. 进入 U-Boot 恢复模式（见下方 6.1 节）
2. 选择固件（**务必选 24.10.0**）：

| 推荐度 | 固件文件 | 说明 |
|--------|---------|------|
| ⭐ **推荐** | `3/immortalwrt-24.10.0-...-sysupgrade.bin` | 使用 `opkg`，兼容 ipk 插件 |
| ❌ **不推荐** | `3/immortalwrt-25.12.1-...-sysupgrade.bin` | 使用 `apk`，**无法安装 ipk 格式的锐捷插件** |

3. 上传刷入，等待自动重启

### 6.1 进入 U-Boot 恢复模式

1. 断开路由器电源
2. 用网线将电脑连接到路由器 **LAN 口**
3. 按住路由器 **Reset 按钮**不放
4. 插入电源，继续按住 Reset 按钮约 **5-10 秒**
5. 观察指示灯变化，闪烁后松开 Reset 按钮
6. 电脑设置固定 IP：`192.168.1.x`（如 `192.168.1.10`），子网掩码 `255.255.255.0`
7. 浏览器访问 **`http://192.168.1.1`**，进入 U-Boot Web 恢复页面

### 6.2 首次登录

刷入完成后路由器自动重启：

- 默认后台地址：**`http://192.168.1.1`**
- 默认用户名：**`root`**
- 默认密码：**无（首次登录会提示设置密码）**

将电脑 IP 改回自动获取（DHCP）。

> ✅ 至此第五步完成！第三方固件已成功刷入。

---

## 七、第六步：安装锐捷认证（MentoHUST）

锐捷认证使用 MentoHUST 方案，通过 LuCI 图形界面进行配置。

> ⚠️ **前提条件**：必须刷入了使用 `opkg` 的 ImmortalWrt（24.10.0），而不是使用 `apk` 的新版本。

### 7.1 所需文件

所有文件位于 `ruijie/` 目录中：

| 文件名 | 说明 |
|--------|------|
| `libpcap1_1.10.5-r2_aarch64_cortex-a53.ipk` | 依赖库 |
| `mentohust_0.3.1-1_aarch64_cortex-a53.ipk` | MentoHUST 主程序 |
| `luci-app-mentohust_1.0.0_all.ipk` | LuCI 图形界面（英文） |
| `luci-i18n-mentohust-zh-cn_git-23.048.32480-2d20a52_all.ipk` | LuCI 中文语言包 |

> 注意：如果 `_aarch64_cortex-a53` 版本安装失败，`ruijie/` 目录下也提供了 `_aarch64_generic` 通用版本和 `gui/` 子目录下的备用版本。

### 7.2 安装 libpcap1（依赖库）

`libpcap1` 是 MentoHUST 的依赖库，安装方式有两种。

#### 方式一：在线安装（opkg，无需下载 ipk）

`libpcap1` 是常见库，可以直接从 ImmortalWrt 官方源在线安装：

```bash
opkg update
opkg install libpcap1
```

但是，`opkg update` 经常会报错，原因是 opkg 配置文件有个奇怪的 bug。修复方法：

1. 浏览器登录 ImmortalWrt 后台：`http://192.168.1.1`
2. 进入 **系统（System）→ 软件包（Software）**
3. 点击 **配置 opkg** 标签页
4. 找到**最后一个配置框**（通常是以 `src/gz` 开头的那段），**全选剪切**到记事本
5. 点击 **保存**
6. 再从记事本**粘贴回来**，再次点击 **保存**
7. 现在重新执行 `opkg update`，应该不会报错了

> 这个错误很奇怪，内容没变，只是剪切出去再粘贴回来就好。如果还是报错，跳过在线安装，直接用下面的离线方式。

#### 方式二：离线安装（ipk 文件，与下面三个包一起装）

如果不想折腾 opkg 配置，`libpcap1` 的 ipk 文件我们也提供了，直接用 MobaXterm 拖入安装即可，见 [7.3 节](#73-安装-mentohust-主程序--luci-界面推荐) 一起操作。

### 7.3 安装 MentoHUST 主程序 + LuCI 界面（推荐）

前面我们已经用 MobaXterm 连接过路由器 SSH，直接用它上传文件最方便。

1. 打开 **MobaXterm**，SSH 登录路由器（`root@192.168.1.1`）

2. 在左侧文件管理器中进入 **`/tmp`** 目录

3. 将 `ruijie/` 目录下的 4 个 ipk 文件，**直接拖入** MobaXterm 左侧的 `/tmp` 文件夹：
   - `libpcap1_1.10.5-r2_aarch64_cortex-a53.ipk`（如果没在线安装的话）
   - `mentohust_0.3.1-1_aarch64_cortex-a53.ipk`
   - `luci-app-mentohust_1.0.0_all.ipk`
   - `luci-i18n-mentohust-zh-cn_git-23.048.32480-2d20a52_all.ipk`

4. 在终端中依次执行安装命令（顺序不能乱）：

```bash
# 如果没在线安装 libpcap1，先装它
opkg install /tmp/libpcap1_1.10.5-r2_aarch64_cortex-a53.ipk

# 然后装 MentoHUST 主程序
opkg install /tmp/mentohust_0.3.1-1_aarch64_cortex-a53.ipk

# 最后装 LuCI 界面和中文包
opkg install /tmp/luci-app-mentohust_1.0.0_all.ipk
opkg install /tmp/luci-i18n-mentohust-zh-cn_git-23.048.32480-2d20a52_all.ipk
```

5. 安装完成后刷新 LuCI 页面即可看到 MentoHUST 菜单项

> ⚠️ **注意**：`mentohust`、`luci-app-mentohust`、`luci-i18n-mentohust-zh-cn` 这三个包不在官方源里，只能通过 ipk 离线安装。
>
> 如果 `_aarch64_cortex-a53` 版本安装失败，`ruijie/` 目录下也提供了 `_aarch64_generic` 通用版本和 `gui/` 子目录下的备用版本。

### 7.4 备选：通过 LuCI 网页上传

如果不方便用 MobaXterm，也可以通过路由器后台网页上传：

1. 浏览器登录 ImmortalWrt 后台：`http://192.168.1.1`
2. 进入 **系统（System）→ 软件包（Software）**
3. 点击 **上传软件包**，依次上传以上 4 个 ipk 文件
4. **安装顺序不能乱**：
   - ① `libpcap1` → ② `mentohust` → ③ `luci-app-mentohust` → ④ `luci-i18n-mentohust-zh-cn`
5. 安装完成后刷新页面

### 7.4 配置锐捷认证

1. 刷新 LuCI 页面
2. 在菜单中找到 **服务（Services）→ MentoHUST**
3. 配置以下参数：
   - **用户名**：你的校园网 / 锐捷账号
   - **密码**：你的校园网 / 锐捷密码
   - **网卡**：选择 WAN 口对应的网卡（通常为 `eth1` 或 `wan`）
   - **认证方式**：根据学校实际情况选择（通常默认即可）
   - **DHCP 方式**：通常选择"认证后获取"或"二次认证"
4. 勾选 **启用**，点击 **保存 & 应用**
5. 查看日志确认认证是否成功

### 7.5 命令行方式（备选）

如果图形界面不工作，可以在 SSH 终端中直接使用命令行：

```bash
# 测试认证（前台运行，可看到输出）
mentohust -u 你的用户名 -p 你的密码 -n eth1

# 后台运行
mentohust -u 你的用户名 -p 你的密码 -n eth1 -b3

# 查看帮助
mentohust -h
```

> ✅ 至此第六步完成！锐捷认证已配置完毕。

---

## 八、恢复原厂系统（备用）

如果刷机后想恢复到小米原厂系统：

### 8.1 方法一：通过 U-Boot 恢复

1. 进入 U-Boot 恢复模式（方法同第五步）
2. 在 U-Boot 页面上传原厂固件：

| 版本 | 原厂固件 |
|------|---------|
| V1 | `恢复原厂系统/v1/miwifi_rd03_firmware_ef0ee_1.0.47.ubi` |
| V2 | `恢复原厂系统/v2/官方组件1.0.92刷过uboot用这个.bin` |

3. 等待刷入完成，路由器重启

### 8.2 方法二：使用小米路由器修复工具

1. 解压 `MIWIFIRepairTool.x86.zip`
2. 运行小米路由器修复工具
3. 按照工具提示刷入原厂固件

---

## 九、常见问题

### Q1：刷入 U-Boot 后无法进入 U-Boot Web 页面？

- 确认电脑 IP 已设置为固定 IP（如 192.168.1.10）
- 检查网线是否插在 LAN 口
- 重新进入 U-Boot 恢复模式（按住 Reset 加电）
- 尝试更换浏览器或清除缓存

### Q2：刷错 U-Boot 版本变砖了怎么办？

- 使用小米官方修复工具尝试恢复
- 如果完全无法启动，需要使用 TTL 串口线救砖

### Q3：ImmortalWrt 刷入后无法联网？

- 检查 WAN 口设置，确保网络配置正确
- 如果校园网需要锐捷认证，必须先完成第六步配置

### Q4：MentoHUST 认证失败？

- 确认用户名和密码正确
- 确认选择的网卡正确（通过 `ip addr` 或 `ifconfig` 查看）
- 尝试不同的 DHCP 方式和认证方式组合
- 某些学校可能需要抓包分析认证参数

### Q5：安装 ipk 时提示不兼容 / 找不到 opkg 命令？

- **ImmortalWrt 版本太新了**，25.x 版本已将包管理器从 `opkg` 换成了 `apk`
- 解决方法：重新刷入 **ImmortalWrt 24.10.0**（`3/` 目录下的 24.10.0 版本）
- `apk` 无法直接安装 `.ipk` 格式的软件包，需要等社区提供对应的 `.apk` 版本插件

### Q6：如何升级 ImmortalWrt？

- 下载新版 `sysupgrade.bin` 固件
- 在 LuCI 后台 → 系统 → 备份/升级 → 刷写新的固件
- 注意：不需要再进入 U-Boot 模式
- ⚠️ 如果升级到 25.x 版本，锐捷插件将失效（原因见 Q5）

### Q7：为什么很多 U-Boot 要收费？刷 Wrt 固件去官网下载就行，为什么 U-Boot 到处找还要花钱？

这是一个很常见的问题，原因是 **U-Boot 和 Wrt 固件的工作量不在一个量级上**：

- **Wrt 固件（ImmortalWrt/OpenWrt）**：由庞大的开源社区维护，有自动化的构建系统。官方 CI/CD 一键编译，你的路由器芯片（MediaTek MT7981）已被主线 Linux 内核良好支持，所以官方直接就能出 `sysupgrade.bin`。说白了，厂商把芯片驱动提交到了 Linux 主线，大家都能直接编译用上。

- **U-Boot（引导程序）**：情况完全不同：
  1. **芯片厂商不把这部分开源**。MediaTek 提供的 U-Boot 源码包含大量闭源二进制 blob（如 DDR 初始化、射频校准数据加载等），不能直接放到开源 U-Boot 主线。
  2. **每款路由器都需要单独适配**。即使是同一颗 MT7981 芯片，不同品牌、不同型号的电路板布局、分区表、网口交换机都不一样。AX3000T V1 和 V2 都用了不同版本的 U-Boot。而 Wrt 固件可以用同一套内核，因为内核在运行时能自动探测硬件。
  3. **Web 恢复界面是个人开发者加的**。你在 `192.168.1.1` 看到的浏览器刷机页面（上传固件、进度条），不是 U-Boot 自带的功能，是有能力的开发者自己写出来集成进去的。这个功能让刷机从"TTL 串口 + TFTP 命令行"变成了"浏览器上传文件"，但背后需要大量逆向和调试工作。
  4. **作者付出了劳动，且未开源**。做适配的人通常是个人开发者，他们需要自己购买路由器反复调试，花费大量时间。有些作者选择收费是可以理解的。本教程用的是社区流通版本，如果你认可作者的付出，建议去支持正版。

**简单类比**：Wrt 固件像给电脑装 Windows/Linux——一个安装镜像适用多种配置；U-Boot 像给每款主板单独写 BIOS——不同主板的 BIOS 不能互换，写错了主板变砖。

### Q8：U-Boot 有开源免费替代方案吗？

- 一些热心开发者维护着开源 U-Boot 分支（如 `hanwckf` 的 `bl-mt798x`），某些型号是免费的
- 但免费开源版本通常功能更基础，不一定有方便的 Web 恢复界面
- 对于我们普通用户来说，找到一个可靠的、适配好的 U-Boot 就够了（教程已提供对应版本）

---

## 参考资料

- [小米 AX3000T 刷机教程视频](https://www.bilibili.com/video/BV1dj411h7EP/)
- [ImmortalWrt 官网](https://immortalwrt.org/)
- [XMiR-Patcher 项目](https://github.com/nik-nel/xmir-patcher)

---

> 📝 **免责声明**：刷机有风险，操作需谨慎。请务必备份原厂系统数据。本教程仅供学习交流使用。
