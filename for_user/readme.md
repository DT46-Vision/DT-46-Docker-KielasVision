# for_user — 最终用户快速启动说明

本文件面向 **最终用户**，假设你拿到的是已经由开发者打包好的镜像 `kielas_rmvision.tar` 或已推到 registry（如 `yourname/kielas_rmvision:latest`）。`for_user` 的 Makefile 仅包含轻量命令（up / exec / down），无需构建。

## 假设准备
- 已安装 Docker，并能运行 `docker` 命令
- 已拿到镜像文件 `kielas_rmvision.tar` 或有访问 registry 权限

---

## 1. 导入镜像（如果是 tar）
把 `kielas_rmvision.tar` 放到仓库根或 `for_user` 目录，然后执行：
```bash
# 在仓库根或 for_user 中
docker load -i kielas_rmvision.tar
```

如果镜像在 registry（示例）：
```bash
docker pull yourname/kielas_rmvision:latest
```

---

## 2. 启动容器（轻量 Makefile / Compose）
切换到 `for_user` 目录并启动：
```bash
cd for_user
make up
# 或
docker compose up
```

容器默认使用 host 网络并会运行项目的 launch 文件（由 compose 的 command 指定）。

---

## 3. 进入容器（调试 / 可视化）
打开新终端，进入容器：
```bash
make exec
# 或
docker exec -it rmvision-core bash
```

进入容器后可直接运行：
```bash
# 切换到 ROS 环境（若未自动加载）
source /opt/ros/humble/setup.bash
source /ros_ws/install/setup.bash || true

# 查看话题
ros2 topic list
ros2 topic hz /detector/armors_info

# 开 GUI（宿主需已允许 X11）
rqt
rviz2
```

---

## 4. 停止与清理
停止容器：
```bash
make down
# 或
docker compose down
```

如需完全删除容器与本地镜像：
```bash
docker compose down --rmi all --volumes --remove-orphans
```

---

## 5. 常见问题（给最终用户的快速排查）
- **看不到 GUI**：先在宿主执行 `xhost +local:root`，并确保 `DISPLAY` 已传入容器。若使用 Wayland，请切换到 Xorg 或使用 VNC。  
- **相机打不开**：确认宿主具有 `/dev/video0`，且容器 compose 中配置了 `devices: - "/dev/video0:/dev/video0"`。  
- **foxglove 无法连接**：检查是否启动了 foxglove bridge，并监听 `0.0.0.0:8765`，在宿主用 `ws://localhost:8765` 连接。  
- **镜像架构不匹配**：如果容器崩溃且日志显示 `.so` 链接错误，可能是 Hik SDK 为 x86_64，而你在 ARM 平台运行——联系开发者或使用相应架构的镜像。

---

## 6. 快速示例（一键流程）
```bash
# 导入镜像（仅第一次）
docker load -i kielas_rmvision.tar

# 启动
cd for_user
make up

# 打开另一个终端进入容器
make exec

# 测试话题与可视化
ros2 topic list
make rqt   # 如果 for_user 的 Makefile 有 rqt 任务（可选）
```

---

如遇无法解决的问题，请把容器日志贴出来（`docker logs -f rmvision-core`）并联系开发者提供 `kielas_rmvision.tar` 的构建日志与 Dockerfile 版本信息。
