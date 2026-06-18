# 安装与部署

网页侦探提供 **桌面客户端** 与 **Web 部署** 两种使用方式，功能完全一致。请根据你的操作系统与使用场景选择其一。

::: tip 下载入口
所有安装包与镜像均可在官网 [下载页面](https://webdiff.perk-net.com/download) 获取。当前最新版本：**v1.2.3**。
:::

## 如何选择？

| 对比项 | 桌面客户端 | Web 部署 |
| --- | --- | --- |
| 功能 | 完整 | 完整（与桌面版一致） |
| 性能 | **更优，推荐优先使用** | 良好 |
| 推荐场景 | Windows · macOS · Linux 桌面 | Linux 服务器 · NAS · 无图形界面环境 |
| 访问方式 | 原生桌面应用 | 浏览器访问 `http://<主机>:9876` |
| 安装方式 | 下载安装包，一键安装 | Linux 安装包或 Docker 镜像 |

::: info 选型建议
- **Windows / macOS 个人电脑**：优先安装桌面客户端，性能更好、体验更流畅。
- **Linux 服务器 / NAS / 无桌面环境**：推荐使用 Web 部署（安装包或 Docker）。
- **Linux 桌面用户**：两种方式均可；日常个人使用可选桌面客户端，长期后台运行可选 Web 部署。
:::

## 环境要求

| 类别 | 要求 |
| --- | --- |
| 操作系统 | Windows 10 / 11 (x64) · macOS 11+ · Linux (x64) |
| 硬件 | 4 核 CPU · 4 GB 内存 · 1 GB 可用磁盘 |
| 网络 | 监控任务执行时需要网络连接 |
| 安全 | 本地数据加密存储，账号端到端同步 |

Web 部署额外要求：

- **Node.js 22+**（安装包方式，镜像内已内置）

---

## 桌面客户端

适用于 Windows、macOS 及 Linux 桌面环境。下载对应平台安装包，按向导完成安装即可。

### 下载

| 平台 | 说明 |
| --- | --- |
| [Windows 版](https://1822104859.share.123865.com/123pan/3UMBjv-sVhlh) | 64 位安装包 |
| [macOS 版](https://1822104859.share.123865.com/123pan/3UMBjv-WVhlh) | macOS 11 及以上 |
| [Linux 版](https://1822104859.share.123865.com/123pan/3UMBjv-uFH7h) | x64 桌面安装包 |

也可前往官网 [下载页面](https://webdiff.perk-net.com/download) 一键下载。

### 安装步骤

1. 下载对应平台的安装包
2. 双击运行安装程序，按向导完成安装
3. 启动「网页侦探」，使用 **微信扫码** 登录
4. 登录成功后即可创建监控任务

::: tip 开机自启
Windows / macOS 桌面版支持 **开机自启动**，可在「我的 → 设置」中开启，确保监控任务 7×24 持续运行。
:::

---

## Web 部署

适合 Linux 服务器、NAS 或无图形界面的环境。部署完成后通过浏览器访问管理界面，功能与桌面版完全一致。

默认服务端口：**9876**  
默认访问地址：`http://<主机 IP>:9876`

### 方式一：Linux 安装包

#### 1. 下载并解压

从 [下载页面](https://webdiff.perk-net.com/download) 获取 **Linux 安装包**（`webdiff-1.2.3-linux-x64.tar.gz`），解压到目标目录：

```bash
tar -xzf webdiff-1.2.3-linux-x64.tar.gz -C /opt/webdiff
cd /opt/webdiff
```

#### 2. 启动服务

```bash
node server/index.js --host 0.0.0.0 --port 9876 --data-dir /var/lib/webdiff
```

或使用包内 CLI：

```bash
node bin/webdiff.js start --host 0.0.0.0 --port 9876 --data-dir /var/lib/webdiff
```

#### 3. 访问与验证

浏览器打开 `http://<服务器 IP>:9876`，微信扫码登录后即可使用。

#### 环境变量（可选）

| 变量 | 说明 | 默认值 |
| --- | --- | --- |
| `WEBDIFF_HOST` | 监听地址 | `0.0.0.0` |
| `WEBDIFF_PORT` | 监听端口 | `9876` |
| `WEBDIFF_DATA_DIR` | 数据目录 | `~/.webdiff` |

#### 后台运行（systemd 示例）

```ini
# /etc/systemd/system/webdiff.service
[Unit]
Description=WebDiff Monitor Service
After=network.target

[Service]
Type=simple
WorkingDirectory=/opt/webdiff
ExecStart=/usr/bin/node server/index.js --host 0.0.0.0 --port 9876 --data-dir /var/lib/webdiff
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now webdiff
```

---

### 方式二：Docker 部署（推荐）

Docker 方式依赖更少、升级方便，适合服务器与 NAS 长期运行。

#### 1. 获取镜像

**在线拉取**（服务器可访问 Docker Hub 时）：

```bash
docker pull pcstx/perk-webdiff:1.2.3
```

**离线部署**（无公网或内网环境）：

1. 从 [下载页面](https://webdiff.perk-net.com/download) 下载 **Docker 离线镜像**
2. 导入镜像：

```bash
docker load -i perk-webdiff-1.2.3.tar
```

#### 2. 启动容器

```bash
docker run -d --name webdiff \
  --restart unless-stopped \
  -p 9876:9876 \
  -v /data/webdiff:/data \
  pcstx/perk-webdiff:1.2.3
```

#### 3. 访问与验证

浏览器打开 `http://<主机 IP>:9876`，微信扫码登录。

- **数据持久化**：宿主机 `/data/webdiff` 映射到容器内 `/data`
- **自动重启**：`--restart unless-stopped` 确保宿主机重启后服务自动恢复

#### 可选：访问令牌

若服务暴露在公网或不可信网络，建议设置本地访问令牌：

```bash
docker run -d --name webdiff \
  --restart unless-stopped \
  -p 9876:9876 \
  -v /data/webdiff:/data \
  -e WEBDIFF_LOCAL_TOKEN=your-secret-token \
  pcstx/perk-webdiff:1.2.3
```

并在反向代理或防火墙层限制访问来源。

---

## 安全提示

- Web 部署默认监听 `0.0.0.0`，局域网内其他设备可访问管理界面
- 生产环境请配置 **防火墙**，仅允许可信 IP 访问 9876 端口
- 公网暴露时建议设置 `WEBDIFF_LOCAL_TOKEN`，并在 Nginx 等反向代理后增加 HTTPS 与访问控制
- 任务数据、Cookie、执行快照默认保存在本机数据目录，请定期备份

---

## 常见问题

### 端口被占用怎么办？

修改启动参数中的 `--port`，或设置环境变量 `WEBDIFF_PORT`，例如改为 `8080`：

```bash
node server/index.js --host 0.0.0.0 --port 8080 --data-dir /var/lib/webdiff
```

### Web 版能否替代桌面版？

可以。Web 版功能与桌面版完全一致，适合服务器长期后台运行。个人 Windows / macOS 用户仍推荐桌面版以获得更好性能与系统集成（托盘、本地通知、自动更新）。

### 如何升级？

- **桌面版**：客户端内自动检测更新，或重新下载最新安装包覆盖安装
- **Web 安装包**：停止服务 → 备份数据目录 → 解压新版本 → 重启服务
- **Docker**：备份 volume → `docker pull` 新镜像 → 停止旧容器 → 用新镜像启动

版本变更记录见 [更新日志](/guide/changelog)。
