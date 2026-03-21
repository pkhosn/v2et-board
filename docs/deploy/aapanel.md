# 使用 aaPanel 部署 v2et-board

本文基于原版 V2Board aaPanel 文档，按 `v2et-board` 的实际运行方式整理。

## 1. 环境准备

- Linux 服务器（推荐 Ubuntu 20.04+/Debian 11+）
- 域名已解析到服务器
- aaPanel 已安装

在 aaPanel 中安装 LNMP 组件：

- Nginx
- MySQL/MariaDB
- PHP（建议 8.2）

## 2. 安装 PHP 扩展

在 aaPanel -> App Store -> PHP -> Install Extensions 安装：

- redis
- fileinfo
- mbstring
- bcmath
- curl
- xml
- zip
- pdo_mysql

## 3. 站点与数据库

1. aaPanel -> Website -> Add Site
2. 绑定域名
3. 创建 MySQL 数据库
4. PHP 版本选择 8.2

创建后将运行目录设置为 `public`。

## 4. 拉取代码并初始化

进入站点目录后执行：

```bash
git clone https://github.com/pkhosn/v2et-board.git ./
git checkout main
chmod +x init.sh update.sh
./init.sh
```

根据安装脚本提示完成 `.env` 配置。

## 5. 伪静态（Nginx Rewrite）

```nginx
location /downloads {
}

location / {
    try_files $uri $uri/ /index.php$is_args$query_string;
}

location ~ .*\.(js|css)?$ {
    expires 1h;
    error_log off;
    access_log /dev/null;
}
```

## 6. 定时任务

aaPanel -> Cron -> Shell Script，每分钟执行：

```bash
php /www/wwwroot/<你的站点目录>/artisan schedule:run
```

## 7. 启动队列（必须）

`v2et-board` 依赖 Horizon 队列服务，未启动会出现“当前队列服务运行异常”。

### 方式 A：aaPanel Supervisor（推荐）

- Name: `v2et-horizon`
- Run User: `www`
- Run Dir: 站点目录
- Start Command: `php artisan horizon`
- Processes: `1`

### 方式 B：systemd（可选）

创建服务后执行：

```bash
systemctl daemon-reload
systemctl enable --now v2et-horizon.service
```

## 8. 常用维护命令

```bash
php artisan config:clear
php artisan config:cache
php artisan horizon:terminate
php artisan horizon:status
```

## 9. 常见问题

- 500 错误：优先检查 `storage/logs`、PHP 扩展、目录权限
- 后台登录闪退：检查 `/monitor/api/stats` 是否 200
- 后台提示队列异常：确认 Horizon 已运行，且 `config/horizon.php` 包含 `production` 配置
