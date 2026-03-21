# 使用 Docker Compose 部署 v2et-board（附录）

本文提供容器化参考方案，适合需要可复制部署流程的场景。

## 1. 准备目录

```bash
mkdir -p /opt/v2et-board && cd /opt/v2et-board
git clone https://github.com/pkhosn/v2et-board app
cd app && git checkout main
```

## 2. 创建 compose 文件

在 `/opt/v2et-board/docker-compose.yml` 写入：

```yaml
version: '3.8'

services:
  mysql:
    image: mysql:8.0
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: change_me_root
      MYSQL_DATABASE: v2et_board
      MYSQL_USER: v2et
      MYSQL_PASSWORD: change_me_db
    volumes:
      - mysql_data:/var/lib/mysql

  redis:
    image: redis:7
    restart: always
    volumes:
      - redis_data:/data

  php:
    image: php:8.2-fpm
    restart: always
    working_dir: /var/www/html
    volumes:
      - ./app:/var/www/html
    depends_on:
      - mysql
      - redis

  nginx:
    image: nginx:stable
    restart: always
    ports:
      - '80:80'
    volumes:
      - ./app:/var/www/html
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - php

volumes:
  mysql_data:
  redis_data:
```

## 3. Nginx 配置

在 `/opt/v2et-board/nginx.conf` 写入：

```nginx
server {
    listen 80;
    server_name _;
    root /var/www/html/public;
    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_pass php:9000;
    }
}
```

## 4. 启动与初始化

```bash
cd /opt/v2et-board
docker compose up -d
```

进入 `php` 容器安装依赖并初始化：

```bash
docker compose exec php bash
apt update && apt install -y git unzip curl libzip-dev libonig-dev libxml2-dev
docker-php-ext-install pdo_mysql bcmath
pecl install redis && docker-php-ext-enable redis
curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
cd /var/www/html
composer install -n
cp .env.example .env
php artisan key:generate
exit
```

修改 `.env` 数据库与 Redis 参数后执行迁移（按项目安装脚本流程）。

## 5. 队列服务（必须）

需额外启动 Horizon 容器进程，例如：

```bash
docker compose exec -d php php /var/www/html/artisan horizon
```

建议在生产中将 Horizon 拆成独立 service，确保随容器自动拉起。

## 6. 生产建议

- 前置反向代理（Nginx/Traefik/Caddy）并启用 HTTPS
- 将 MySQL、Redis 数据卷放到可靠磁盘
- 使用 `.env` 管理敏感配置，不写入仓库
