准备 Redis

准备 PG
```sql
-- 创建数据库
CREATE DATABASE paperless;

\c paperless

-- 创建用户并设置密码
CREATE USER paperless WITH PASSWORD 'Asu@950820';

-- 授予用户对数据库的全部权限
GRANT ALL PRIVILEGES ON DATABASE paperless TO paperless;
GRANT ALL PRIVILEGES ON SCHEMA public TO paperless;

-- 为 Paperless 的搜索功能启用扩展（重要！） 
CREATE EXTENSION IF NOT EXISTS pg_trgm; 
CREATE EXTENSION IF NOT EXISTS unaccent;
```

准备目录
```
/data/paperless
/data/paperless/export
/data/paperless/consume
/data/paperless/media
/data/paperless/data
```

docker-compose.yml
```yml
services:
  webserver:
    image: ghcr.io/paperless-ngx/paperless-ngx:latest
    container_name: paperless
    network_mode: host
    restart: unless-stopped
    volumes:
      - ./data:/usr/src/paperless/data
      - ./media:/usr/src/paperless/media
      - ./export:/usr/src/paperless/export
      - ./consume:/usr/src/paperless/consume
    environment:
      PAPERLESS_REDIS: redis://:xxxx@localhost:6379
      PAPERLESS_REDIS_PREFIX: paperless
      PAPERLESS_DBENGINE: postgresql
      PAPERLESS_DBHOST: localhost
      PAPERLESS_DBPORT: 5432
      PAPERLESS_DBNAME: paperless
      PAPERLESS_DBUSER: paperless
      PAPERLESS_DBPASS: xxxx
```

启动：
```
docker compose up -d
```

停止并删除容器：
```
docker compose down 
```