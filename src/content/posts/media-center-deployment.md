---
title: 家庭媒体中心全家桶部署教程：CD2 + Emby + Symedia + Redia
published: 2026-06-28
description: 基于 Docker Compose 的完整媒体库搭建教程，从 CloudDrive2 挂载网盘到 Symedia 刮削、Emby 海报墙再到 Redia 302 直链播放，一步步教你搭建属于你自己的家庭媒体中心。
tags: [Docker, NAS, Emby, 媒体中心, 网盘, 115]
category: 技术教程
draft: false
---

> 基于 Docker Compose 部署，每个服务独立一个 compose 文件  
> 链路：115网盘 → CD2 挂载 → Symedia 生成 strm → Emby 海报墙 → Redia 302 直链

---

## 整体架构

```
115网盘/阿里云盘/百度网盘
       │
       ▼
CloudDrive2 ─── 网盘挂载本地（端口 19798）
       │
       ▼
Symedia ─── 监控网盘 + 生成 .strm + 刮削归档（端口 8095）
       │
       ▼
Emby ─── 媒体库管理 + 海报墙（端口 8096）
       │
       ▼
Redia ─── 302 直链反向代理（端口 9025）
       │
       ▼
用户播放
```

---

## 前置准备

- Docker Engine 24+（含 Compose V2 插件）
- Linux 服务器或 NAS（建议 4G+ 内存）
- 115 网盘会员（或阿里云盘、123 云盘）
- Symedia 激活码（必购）
- Redia 激活码

---

## 目录结构

```bash
mkdir -p /volume1/docker/clouddrive2/config
mkdir -p /volume1/docker/emby/config
mkdir -p /volume1/docker/symedia/config
mkdir -p /volume1/docker/symedia/playwright
mkdir -p /volume1/docker/redia/config
mkdir -p /volume1/CloudNAS
mkdir -p /volume1/media
```

`/volume1` 是 NAS 主存储路径，Linux 服务器可改用 `/opt`

---

## 一、CloudDrive2 — 云盘挂载服务

### docker-compose.yml

创建 `/volume1/docker/clouddrive2/docker-compose.yml`：

```yaml
services:
  clouddrive2:
    container_name: clouddrive2
    image: cloudnas/clouddrive2-unstable:latest
    restart: always
    network_mode: "host"
    pid: "host"
    privileged: true
    devices:
      - /dev/fuse:/dev/fuse
    environment:
      - CLOUDDRIVE_HOME=/Config
      - ENABLE_RUN_AFTER_START=true
    volumes:
      - /volume1/CloudNAS:/CloudNAS:shared
      - /volume1/docker/clouddrive2/config:/Config
```

### 启动和配置

```bash
cd /volume1/docker/clouddrive2
docker compose up -d
```

访问 `http://服务器IP:19798`

1. 设置管理密码
2. 左侧云盘 → 绑定 115 或其他网盘
3. 挂载到本地：挂载点选 `/CloudNAS`

**注意：** 挂载路径前建议加一层目录，如 `/CloudNAS/CloudDrive/115`。这样 CD2 掉盘恢复后不需要重启其他容器。

> 💡 挂载失败则执行：`sudo mount --make-shared /volume1`  
> 飞牛系统需先配置 Docker MountFlags=shared

---

## 二、Emby — 媒体库管理

### docker-compose.yml

创建 `/volume1/docker/emby/docker-compose.yml`：

```yaml
services:
  emby:
    container_name: emby
    image: linuxserver/emby:4.8.11
    restart: always
    network_mode: bridge
    privileged: true
    devices:
      - /dev/dri:/dev/dri
    ports:
      - "8096:8096"
    environment:
      - PUID=0
      - PGID=0
    volumes:
      - /volume1/docker/emby/config:/config
      # 以下路径必须与 Symedia 完全一致
      - /volume1/CloudNAS:/CloudNAS:rslave
      - /volume1/media:/media
```

> 飞牛用户将 `rslave` 改为 `rshared`

### 启动和配置

```bash
cd /volume1/docker/emby
docker compose up -d
```

访问 `http://服务器IP:8096`

1. 语言选简体中文
2. 创建管理员账号
3. 添加媒体库 → 文件夹 `/media` → 元数据语言 Chinese

**获取 API Key：** Emby → 设置 → 高级 → API 密钥 → 生成（Symedia 联动需要）

---

## 三、Symedia（Dockter）— 智能媒体库管家

### docker-compose.yml

创建 `/volume1/docker/symedia/docker-compose.yml`：

```yaml
services:
  symedia:
    container_name: symedia
    image: shenxianmq/symedia:latest
    # ARM 架构: shenxianmq/symedia_arm64:latest
    restart: always
    network_mode: "host"
    ports:
      - "8095:8095"
    environment:
      - TZ=Asia/Shanghai
      - LICENSE_KEY=你的激活码
      - SA_2FA=true
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - /volume1/docker/symedia/config:/app/config
      - /volume1/docker/symedia/playwright:/symedia/.cache/ms-playwright
      - /volume1/CloudNAS:/CloudNAS:rslave
      - /volume1/media:/media
```

### 启动和登录

```bash
cd /volume1/docker/symedia
docker compose up -d
```

访问 `http://服务器IP:8095` | 账号 `admin` | 密码 `password`

### 配置步骤

#### 步骤 1：全局设置

设置 → 全局设置

![全局设置](https://images.symedia.top/2025/04/07/20250407072643343.png)

- 云盘根目录：`/CloudNAS/CloudDrive`
- 链接同步间隔：按需设置
- TMDB API 域名：`api.themoviedb.org`
- TMDB 图片域名：`image.tmdb.org`
- 建议开启调试模式（方便排错）

#### 步骤 2：配置 CD2 助手

插件 → CloudDrive2 助手

![CD2助手基本配置](https://images.symedia.top/2025/04/07/20250407073433144.png)

- CD2 IP：CD2 所在地址（不要加 http://）
- 端口：`19798`
- 用户名 / 密码：填入 CD2 登录信息
- 根目录：`/CloudNAS/CloudDrive`
- 监听目录：如 `/115open/emby`
- 开启"监听文件变化事件"

点击 **刷新令牌** → **保存配置**

#### 步骤 3：添加链接同步（生成 strm 文件）

链接同步 → 添加同步

- 源路径：`/CloudNAS/CloudDrive/115/你的影视文件夹`
- 目标路径：`/media`
- 文件格式：选择 `strm`

开始同步后，Symedia 会扫描网盘中的视频文件，在 `/media` 下生成对应的 `.strm` 文件。

#### 步骤 4：配置 EmbyServer 联动

插件 → EmbyServer

- Emby 地址：`http://服务器IP:8096`
- API Key：填入上一步生成的密钥

保存后，Symedia 会自动通知 Emby 扫描新生成的 strm 文件。

#### 步骤 5：配置 Webhook（CD2 会员功能）

设置 → Webhook

填入 CD2 地址后，网盘新增文件会自动触发 strm 生成，无需手动同步。

#### 步骤 6：配置通知

设置 → 通知

![通知配置](https://images.symedia.top/2025/04/07/20250407072643343.png)

配置 Telegram Bot：
1. Telegram 搜索 @BotFather → `/newbot` → 创建机器人 → 保存 API Token
2. 搜索 @getidsbot → `/about` → 获取你的 User ID
3. 在 Symedia 通知设置填入 API Token 和 User ID
4. 开启需要的通知开关 → 保存并测试

#### 步骤 7：安装神医插件（StrmAssistant）

前往 [StrmAssistant Releases](https://github.com/sjtuross/StrmAssistant/releases) 下载插件，放入 Emby 的 `plugins` 文件夹，重启 Emby。

![神医插件安装](https://images.symedia.top/2025/04/14/20250414210241_6e6613f0.png)

重启后在 Emby 设置中出现"神医助手"即成功。

![神医插件设置](https://images.symedia.top/2025/04/14/20250414210444_f02152ed.png)

推荐开启"提取视频信息"功能，这能显著加快 Strm 文件的起播速度。

#### 步骤 8：配置刮削归档

归档 → 归档刮削

- 设置刮削源（TMDB）
- 开启实时监控
- 配置归档目录结构

#### 步骤 9：进阶配置

**聚合搜索：** 搜索 → 设置 → 填入 Telegram 频道 ID

![聚合搜索配置](https://images.symedia.top/2025/04/07/20250407073459087.png)

**网盘转存：** 插件 → 对应网盘助手 → 填入 cid（文件夹 ID）

![网盘转存配置](https://images.symedia.top/2025/04/07/20250407074646569.png)

配置后，发送网盘分享链接给 Telegram Bot 即可自动转存。

---

## 四、Redia — 302 直链反向代理

### docker-compose.yml

创建 `/volume1/docker/redia/docker-compose.yml`：

```yaml
services:
  redia:
    container_name: redia
    image: shenxianmq/redia:latest
    # ARM: shenxianmq/redia_arm64:latest
    restart: always
    network_mode: "host"
    environment:
      - LICENSE_KEY=你的Redia密钥
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - /volume1/docker/redia/config:/app/config
```

### 启动和配置

```bash
cd /volume1/docker/redia
docker compose up -d
```

访问 `http://服务器IP:9025` | 账号 `admin` | 密码 `password`

#### 步骤 1：配置云盘助手

反向代理 → 云盘助手 → 添加 115，配置名称填 `115`

![配置云盘助手](https://images.symedia.top/2025/09/21/20250921224635_0f7f5c39.png)

#### 步骤 2：填入 115 配置

![115配置示例](https://images.symedia.top/2025/09/21/20250921224728_831d3529.png)

#### 步骤 3：配置 Emby 服务器路径替换

反向代理 → 媒体服务器 → Emby 服务器

路径替换是 Redia 的核心功能，将 strm 中的网盘路径映射到云盘助手名称：

```bash
/CloudNAS/CloudDrive/115 => 115
/CloudNAS/CloudDrive/123云盘 => 123
/CloudNAS/CloudDrive/天翼云盘 => 天翼云盘
```

![Emby服务器配置](https://images.symedia.top/2025/09/21/20250921224506_0dceb6a4.png)

![路径替换示例](https://images.symedia.top/2025/09/21/20250921225106_53733573.png)

#### 步骤 4：保存并启用

![Emby配置完成](https://images.symedia.top/2025/09/21/20250921224928_0d4edfa7.png)

启用服务器 → 保存配置

---

## 部署顺序总结

```
1. 安装 Symedia 容器
2. 全局设置（TMDB、云盘根目录）
3. CD2 助手（刷新令牌）
4. 链接同步（生成 strm）
5. EmbyServer 联动
6. Webhook（实时监控）
7. 通知渠道（Telegram）
8. 神医插件（加速播放）
9. 刮削归档
10. 进阶：聚合搜索 + 网盘转存
```

---

## 运维命令

```bash
# 各服务独立管理
cd /volume1/docker/clouddrive2 && docker compose logs -f
cd /volume1/docker/emby && docker compose restart
cd /volume1/docker/symedia && docker compose pull && docker compose up -d
cd /volume1/docker/redia && docker compose down

# 所有容器状态
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"

# 更新全部
for dir in clouddrive2 emby symedia redia; do
  cd /volume1/docker/$dir && docker compose pull && docker compose up -d
done

# 备份配置
tar -czf /backup/media-stack-$(date +%Y%m%d).tar.gz /volume1/docker/*/config
```

---

## FAQ

**CD2 挂载失败？**  
`sudo mount --make-shared /volume1`

**Emby 扫不到媒体？**  
确认 Symedia 和 Emby 的 `/CloudNAS`、`/media` 路径映射完全一致

**CD2 掉盘后恢复不了？**  
挂载路径前加一层目录，如 `/CloudNAS/CloudDrive/115`

**115 429 限流？**  
CD2 → 115 设置 → MaxQueriesPerSecond 改为 1

**Strm 起播慢？**  
安装神医插件，开启"提取视频信息"

**端口表：**

| 服务 | 端口 |
|------|------|
| CloudDrive2 | 19798 |
| Emby | 8096 |
| Symedia | 8095 |
| Redia | 9025 |

---

**参考链接：**  
- [Symedia Wiki](https://wiki.viplee.cc)  
- [Redia 部署教程](https://www.symedia.top/archive/redia/Redia%E9%83%A8%E7%BD%B2%E6%95%99%E7%A8%8B.html)  
- [StrmAssistant 神医插件](https://github.com/sjtuross/StrmAssistant)
