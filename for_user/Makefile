# 轻量 RMVision Makefile

IMAGE_NAME = yourname/kielas_rmvision:latest
CONTAINER_NAME = rmvision-core

.PHONY: up exec down

up: ## 启动容器
	docker compose up -d

exec: ## 进入容器交互终端
	docker exec -it $(CONTAINER_NAME) bash

down: ## 停止容器
	docker compose down
