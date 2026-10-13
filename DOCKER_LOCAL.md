# MaxKB 本地版本运行指南

本仓库为 [MaxKB](https://github.com/1Panel-dev/MaxKB) 的增强/修复版本，包含了以下核心缺陷修复：

1. **`base_reset_problem_step.py`**: 修复 `run_time` KeyError 与 `step_args` NoneType AttributeError。
2. **`base_generate_human_message_step.py`**: 修复引用分段字符数超限时的负切片 Bug（彻底解决超长笔记检索内容被整体置空问题）。

---

## 方式一：使用修复后的极速构建镜像（推荐）

只需 1 秒即可构建：

```bash
docker build -t maxkb:local -f Dockerfile.local .
```

### 替换现有正在运行的容器（无损切换，保留全部数据）

现有的容器名称为 `maxkb`，挂载卷与外部端口映射（3902）已与 DBngin 本地 PostgreSQL 关联：

```bash
# 1. 停止并移除旧容器（数据保存在 Docker Volume 与宿主机 DBngin 中，不丢失）
docker stop maxkb
docker rm maxkb

# 2. 用新构建的本地镜像启动（保持原有端口与数据卷）
docker run -d --name maxkb \
  -p 3902:8080 \
  --add-host host.docker.internal:host-gateway \
  -v maxkb_maxkb-data:/opt/maxkb \
  -v maxkb_maxkb-python-packages:/opt/maxkb/python-packages \
  -e MAXKB_DB_HOST=host.docker.internal \
  -e MAXKB_DB_PORT=5432 \
  -e MAXKB_DB_USER=postgres \
  -e MAXKB_DB_NAME=maxkb \
  -e MAXKB_DB_PASSWORD=Passw0rd \
  -e MAXKB_REDIS_HOST=host.docker.internal \
  -e MAXKB_REDIS_PORT=6379 \
  -e MAXKB_REDIS_DB=2 \
  --restart always \
  maxkb:local
```

---

## 方式二：开发挂载模式（修改 Python 代码无需重构镜像，即刻生效）

如果需要在本地直接修改代码并热调试，可以将本仓库的 `apps/` 目录直接挂载进容器：

```bash
-v /Users/ethereal3x/project/python/MaxKB/apps:/opt/maxkb-app/apps
```

只要保存代码，Gunicorn / Celery 即加载最新代码。

---

## 方式三：从零编译构建全量镜像（含前端）

如需修改前端 Vue 代码并完整从源码编译：

```bash
docker build -t maxkb:full -f installer/Dockerfile .
```
