# for_builder — 本地构建与打包说明

本文件面向 **开发者 / 打包者**，用于在本地将 `ros_ws` 源码构建为 Docker 镜像 `kielas_rmvision:latest`，并导出为 `kielas_rmvision.tar` 以便分发给最终用户。

> 假设仓库结构（工作目录为仓库根）：
>
> ```
> .
> ├── for_builder
> │   ├── docker-compose.yml
> │   ├── Dockerfile
> │   ├── Makefile
> │   └── ros_ws
> └── for_user
>     ├── docker-compose.yml
>     └── Makefile
> ```

## 快速前提检查（宿主机）
- 操作系统：Ubuntu 22.04（推荐）
- 已安装 Docker（参考 README_DOCKER_FULL 中安装步骤）
- 若需显示 GUI（rqt/rviz2）：确保宿主机为 Xorg 会话，并运行 `xhost +local:root`
- 若需访问相机：确保 `/dev/video0` 可用

---

## 1. 准备源码
把项目源码放到 `for_builder/ros_ws`。例如：
```bash
cd for_builder
git clone https://github.com/DT46-Vision/DT-46-KielasVision.git ros_ws
```
或者把本地现有 `ros_ws` 目录复制到 `for_builder/ros_ws`。

如果项目需要厂商 SDK（MindVision / Hik），请把 SDK 文件（如 `MindVisionSDK/` 或 `MVS.deb`）放到 `for_builder` 根目录，Dockerfile 将根据存在性安装。

---

## 2. 构建镜像（只编译）
在 `for_builder` 根目录运行：
```bash
make build
# 或
docker compose build
```
若你修改了 Dockerfile 或遇到缓存问题，使用无缓存构建：
```bash
docker compose build --no-cache
```

构建完成后本地会有镜像 `kielas_rmvision:latest`（或你在 Dockerfile/compose 指定的镜像名）。

---

## 3. 启动并调试容器
启动（前台查看输出）：
```bash
make up
# 或
docker compose up
```
后台启动：
```bash
docker compose up -d
```

打开另一个终端进入容器进行交互（多终端调试）：
```bash
make exec
# 或
docker exec -it rmvision-core bash
```

在容器内你可以直接运行：
```bash
source /opt/ros/humble/setup.bash
source /ros_ws/install/setup.bash || true
rqt
rviz2
ros2 topic list
```

---

## 4. 开发与重新编译（容器内）
若你修改了源码，进入容器并重新构建：
```bash
docker exec -it rmvision-core bash
rm -rf build install log
colcon build --symlink-install
source install/setup.bash
```

---

## 5. 将镜像导出为 tar（打包）
验证运行无误后导出镜像：
```bash
# 在仓库根或 for_builder 中运行
docker save -o ../kielas_rmvision.tar kielas_rmvision:latest
```
这会在父目录生成 `kielas_rmvision.tar`，把该文件发给最终用户或放进 `for_user`。

---

## 6. 常见注意点（简要）
- X11：宿主机需允许 X11（`xhost +local:root`），Compose 必须挂载 `/tmp/.X11-unix` 并传 `DISPLAY`。  
- 相机：确保 `devices: - "/dev/video0:/dev/video0"` 被正确设置并具备权限。  
- Hik SDK：镜像内的 Hik `.so` 很可能是 x86_64。若目标平台为 ARM，请在目标平台安装厂商 SDK 并替换库文件（详见主文档/README_DOCKER_FULL）。  
- 若 `colcon build` 报缺少 Python 包（如 `lark`），请在 Dockerfile 中添加 `pip install lark-parser`。

---

## 7. 导出示例（放到 for_user）
建议把导出的 tar 放到仓库根或直接复制到 `for_user`，便于最终用户直接导入：
```bash
# 假设当前在 for_builder
docker save -o ../kielas_rmvision.tar kielas_rmvision:latest
cp ../kielas_rmvision.tar ../for_user/
```

---

保存完后，把 `kielas_rmvision.tar` 交给用户，或推送到私有 Registry（推荐长期维护）。
