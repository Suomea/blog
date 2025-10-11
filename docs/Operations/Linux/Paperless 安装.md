准备 Redis

准备 PG
```sql
create user paperless with password 'xxxx';
create database paperless owner paperless;

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