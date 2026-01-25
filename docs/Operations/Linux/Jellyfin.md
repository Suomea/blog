```

```

```
docker run -d \
--name jellyfin \
-p 8096:8096/tcp \
-p 7359:7359/udp \
--device /dev/dri:/dev/dri \
--volume ./config:/config \
--volume ./cache:/cache \
--mount type=bind,source=/data/disk1/qBittorent/,target=/media \
--restart=unless-stopped \
jellyfin/jellyfin
```