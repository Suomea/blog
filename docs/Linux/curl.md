用法
```
curl [options...] <url>
```

## 指定 HTTP 请求方法

默认为 GET 请求。
```
 -X <method>   Specify request method to use
```

示例
```shell
curl -X DELETE www.example.com
```

## 携带请求头

```
 -H, --header <header/@file> Pass custom header(s) to server
```

示例
```shell
curl -H 'aaa: bbbb' -H 'cccc: ddd' www.example.com
```

## 携带 JSON 请求体

```
 -d, --data <data>        HTTP POST data
```

示例
```shell
curl -X POST -H 'Content-Type: application/json' -d '{"hello": "world"}' www.example.com
```

## 表单上传文件

```
 -F, --form <name=content> Specify multipart MIME data
```

```shell
curl -X POST -OJ -H "Content-Type: multipart/form-data" \
	-F "file=@C:\Users\jacky\Desktop\aaa.txt" \
	http://localhost:9010/suomea-tool/base64/fileEncoder
```

## 打印响应头

```
 -i, --include            Include protocol response headers in the output
 
 -I 只打印响应头，忽略响应内容
```

## 文件下载选项

```
 -J, --remote-header-name Use the header-provided filename
 
 -O, --remote-name        Write output to a file named as the remote file
 
 -o, --output <file>      Write to file instead of stdout
```

按照响应头 `Content-Disposition` 指定的文件名下载文件。
```

```