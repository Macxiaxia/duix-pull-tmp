# duix-pull-tmp — 临时仓库（DUIX 镜像拉取 → ACR 中转）

## 一次性流程（已自动化）

1. ✅ GHA 在云端 `docker pull guiji2025/duix.avatar:latest` (~4.94 GB)
2. ✅ 重命名 + 推到 **阿里云 ACR 个人版**（你提供凭据）
3. ✅ 本机 `docker pull` ACR 地址（国内仓库，国内带宽秒级）

## 你需要做的（一次性，约 5 分钟）

### 第 1 步：建阿里云 ACR 个人版

浏览器打开 https://cr.console.aliyun.com/

- 开通容器镜像服务（个人版，**完全免费**）
- 创建个人版实例 → 设置 Registry 登录密码（**记牢**，只设一次）
- 创建命名空间（Namespace）：例如 `lingce`
- 4 个值需要给 Agent：

| 字段 | 例子 | 怎么查 |
|------|------|--------|
| Registry 地址 | `registry.cn-hangzhou.aliyuncs.com` | ACR 控制台左上角 |
| Namespace | `lingce` | ACR → 个人版 → 命名空间 |
| 用户名 | 阿里云账号 ID 或 RAM 子账号 | ACR 控制台 → 访问凭证 |
| 密码 | 上一步设的固定密码 | （只设一次，要记牢） |

### 第 2 步：填入 GitHub Secrets

打开 https://github.com/Macxiaxia/duix-pull-tmp/settings/secrets/actions

点 "New repository secret"，依次添加 4 个：

- `ACR_REGISTRY` = `registry.cn-hangzhou.aliyuncs.com`（或你的实际地址）
- `ACR_NAMESPACE` = `lingce`（或你的命名空间）
- `ACR_USERNAME` = 你的阿里云账号
- `ACR_PASSWORD` = 你设置的固定密码

### 第 3 步：触发 workflow

打开 https://github.com/Macxiaxia/duix-pull-tmp/actions/workflows/pull_duix.yml
点 "Run workflow" → 绿色按钮 → 等 5-10 分钟

完成后 README 第一栏会显示推送目标。

### 第 4 步：本机拉取

```bash
docker login registry.cn-hangzhou.aliyuncs.com
docker pull registry.cn-hangzhou.aliyuncs.com/lingce/guiji2025_duix_avatar_latest
docker run -d -p 8080:8080 registry.cn-hangzhou.aliyuncs.com/lingce/guiji2025_duix_avatar_latest
```

### 第 5 步：删除此仓库（用完即焚）

https://github.com/Macxiaxia/duix-pull-tmp/settings → "Delete this repository"

## 为什么用 ACR 中转而不是 GHA artifact

| 方式 | 速度 | 限制 |
|------|------|------|
| GHA artifact 下载（**失败**） | 73 KB/s × 19h | 单文件 5GB 上限 |
| GHCR | 国内被封（本机拉不到） | - |
| **阿里云 ACR 个人版** | 国内 1Gbps + 免费 | 完美 |

## 不直接用 GHCR 的原因

ghcr.io 走 GitHub IP，国内访问被封。本机 docker pull 不通。
ACR 国内有 CDN，免费个人版支持完整 docker registry 协议。