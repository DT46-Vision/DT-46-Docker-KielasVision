# DT-46-Docker-KielasVision

# RMVision Docker 使用指南（轻量版 Makefile 适配）

这是针对外部用户的最简 Docker 使用指南，适应新的轻量 Makefile，帮助用户直接运行 ROS2 + GUI + 摄像头环境。

---

## 1️⃣ 前提条件

确保本地已安装：
- Docker
- Docker Compose

如果使用 GPU 或 PyTorch，建议安装 NVIDIA Container Toolkit。

---

## 2️⃣ 获取项目文件

假设你已经有以下文件结构：
```
project/
├─ docker-compose.yml
├─ Makefile        # 轻量版
└─ ros_ws/         # ROS2 工作空间源码，可选
```

---

## 3️⃣ 拉取镜像

```bash
docker pull yourname/kielas_rmvision:latest
```

替换 `yourname/kielas_rmvision:latest` 为实际镜像名。

---

## 4️⃣ 使用 Makefile 启动/管理容器

- 启动容器：
```bash
make up
```
- 进入容器终端：
```bash
make exec
```
- 停止容器：
```bash
make down
```

Makefile 中封装了基础的 Docker 命令，用户无需手动输入复杂命令。

---

## 5️⃣ 直接使用 Docker（无需 Makefile）

如果不使用 Makefile，也可直接运行：

启动容器：
```bash
docker compose up -d
```
进入容器：
```bash
docker exec -it rmvision-core bash
```
停止容器：
```bash
docker compose down
```

---

## 6️⃣ 注意事项

1. `ros_ws` 挂载为卷，可保持源码更新和持久化数据。
2. `/dev/video0` 可根据实际摄像头设备修改。
3. GUI 支持依赖 X11 转发，Linux 下 DISPLAY 环境变量必须正确。
4. Makefile 为轻量管理工具，不包含构建步骤，镜像需提前拉取。
