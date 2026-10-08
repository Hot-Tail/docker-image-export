# docker-image-export

## 这是什么

这是一个基于 GitHub Actions 的镜像导出工作流，用于将项目所需的全部依赖镜像**预拉取并打包为离线 tar 文件**。工作流采用手动触发方式，运行后会生成三个独立的离线镜像包（Redis、Python、SearXNG），并作为 Artifacts 提供下载。

## 解决了什么问题

在离线环境、内网环境或网络受限的场景中，直接拉取 Docker 镜像往往非常困难甚至不可能。本仓库通过 GitHub Actions 的免费算力，提前将所需镜像拉取并导出为 tar 包，你只需下载这些 tar 包，通过 `docker load` 即可在任意机器上快速导入镜像，**彻底摆脱对 Docker Hub / 网络环境的依赖**。

同时，镜像版本固定（如 `redis:7-alpine`、`python:3.12-slim`），可保证部署环境的一致性，避免因镜像更新导致的意外不兼容。

## 如何使用

### 1. 手动触发工作流

在 GitHub 仓库页面，进入 **Actions** 选项卡，选择左侧的 **导出全部依赖镜像**，点击 **Run workflow** 按钮，选择分支后确认执行。

### 2. 等待工作流运行完成

工作流会自动在 `ubuntu-latest` 环境中执行以下操作：
- 拉取三个官方/第三方镜像
- 分别保存为 tar 文件
- 上传为三个独立的 Artifacts

### 3. 下载离线镜像包

运行成功后，在 **Actions → 对应运行记录 → Artifacts** 区域，可下载以下三个文件：
- `redis-offline`（内含 `redis-7-alpine.tar`）
- `python-offline`（内含 `python-3.12-slim.tar`）
- `searxng-offline`（内含 `searxng-latest.tar`）

### 4. 在目标机器导入镜像

将下载的 tar 文件传输到目标机器，执行：

```bash
docker load -i redis-7-alpine.tar
docker load -i python-3.12-slim.tar
docker load -i searxng-latest.tar
```

导入完成后，即可通过 `docker images` 查看到对应镜像，随时启动容器。

## 包含的镜像列表

| 镜像 | 标签 | 用途说明 | 许可 |
|------|------|----------|------|
| `redis` | `7-alpine` | 轻量级内存数据库，常用于缓存和消息队列 | BSD-3-Clause（Alpine 发行版） |
| `python` | `3.12-slim` | 精简版 Python 运行环境，适合构建应用镜像 | PSF License（Python-2.0） |
| `searxng/searxng` | `latest` | 开源元搜索引擎，提供隐私友好的搜索服务 | AGPL-3.0-or-later |

> **关于 Redis 许可的补充说明**：Redis 官方自 7.4 版本起改为 RSALv2 + SSPLv1 双许可。本仓库导出的 `redis:7-alpine` 为 **Alpine 发行版**，Alpine 官方仓库中的 Redis 仍采用 **BSD-3-Clause** 许可。如果你使用的 Redis 版本高于 7.4，请自行确认其许可条款是否满足你的使用场景。
>
> 如果需要增减镜像，直接修改 `.github/workflows/export.yml` 中的 `docker pull` 和 `docker save` 命令即可。

## 注意事项

- 本工作流使用 `workflow_dispatch` 触发，不会自动运行，完全由你控制。
- 导出的 tar 文件通常较大（尤其 Python 镜像），下载时请注意网络和磁盘空间。
- 所有 Artifacts 默认保留 90 天，请及时下载保存到本地。
- 本仓库仅提供镜像导出功能，不包含镜像本身的任何修改或定制。

## 许可

### 本仓库代码

本仓库的 GitHub Actions 工作流文件及文档采用 **MIT License** 发布。

### 导出的镜像

本仓库仅提供导出功能，不改变镜像本身的许可。各镜像的许可如下：

- **redis:7-alpine**：BSD-3-Clause（Alpine 发行版）
- **python:3.12-slim**：PSF License（Python-2.0）
- **searxng/searxng:latest**：AGPL-3.0-or-later

使用上述镜像时，请遵守各自的许可条款。如需用于商业场景，请确认 AGPL-3.0 等 copyleft 许可的合规要求。

## 许可

本项目采用 Apache License 2.0 发布。

Copyright (c) 2026 Author

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
