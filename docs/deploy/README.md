# v2et-board 部署总览

本文提供部署方式快速选择，详细步骤见对应文档。

## 推荐顺序

1. aaPanel 部署（主推荐）
2. Docker Compose（附录，适合容器化）

## 方式对比

| 方式 | 适合人群 | 复杂度 | 维护方式 |
|---|---|---|---|
| aaPanel | 偏面板化运维、习惯宝塔生态 | 低 | 面板 + 少量命令 |
| Docker Compose | 需要可复制环境、偏 DevOps | 中 | compose 文件 + 容器生命周期 |

## 文档入口

- [aaPanel 部署](./aapanel.md)
- [Docker Compose 部署（附录）](./docker-compose.md)

## 部署后必须检查

- 后台可正常登录
- `/monitor/api/stats` 返回 200
- Horizon 状态为 running
- 节点拉取配置和用户接口返回 200
