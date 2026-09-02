# duix-pull-tmp (临时仓库 · 自动清理)

## 用途
通过 GitHub Actions 在云端拉取 Docker 镜像（突破本机外网封堵），上传为 artifact 供下载。

## 当前任务
拉取 `guiji2025/duix.avatar:latest` (灵策AI 数字人 DUIX 镜像)

## 使用步骤

### 1. 触发 workflow
访问 https://github.com/Macxiaxia/duix-pull-tmp/actions/workflows/pull_duix.yml
点击 "Run workflow" → 填入镜像名（默认 guiji2025/duix.avatar:latest）→ 绿色按钮运行

### 2. 等待完成（约 3-10 分钟）
在 Actions 页面查看运行日志。

### 3. 下载产物
完成后页面底部 "Artifacts" 区下载 `duix-image.zip`（含分片 tar.gz）

### 4. 本机合并 + load
```bash
cd ~/Downloads
cat duix-image.tar.gz.part_* > duix-image.tar.gz
gunzip duix-image.tar.gz
docker load -i duix-image.tar
docker images | grep duix
```

### 5. 删除此仓库（用完即焚）
https://github.com/Macxiaxia/duix-pull-tmp/settings → Delete this repository

## 不上传到公开 docker registry 的原因
- guiji2025 是私有 namespace, 公开镜像站不会有副本
- GHCR 要 auth, 多一步配置
- artifact 直接下载最简单