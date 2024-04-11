## suomea-tool

base64 加密
```shell
curl http://106.15.72.83/suomea-tool/base64/encoder?content=<string>
```

base64 解密
```shell
curl http://106.15.72.83/suomea-tool/base64/decoder?content=<string>
```

base64 文件加密
```shell
curl -X POST -H "Content-Type: multipart/form-data" -F "file=@C:\Users\jacky\Desktop\aaa.txt" http://localhost:9010/suomea-tool/base64/fileEncoder
```