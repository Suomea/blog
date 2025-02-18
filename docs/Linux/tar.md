用法
```
tar [选项] [归档文件名] [文件或目录列表]
```

tar 用于创建、查看和解包归档文件（archive files），归档文件通常以 .tar 为扩展名，可以将多个文件和目录打包成一个文件，便于传输或备份。

tar 本身并不压缩文件，但可以与其他压缩工具结合使用，生成压缩的归档文件 tar.gz、tar.bz2、tar.xz 等。

-c 创建归档文件。
-x 解包归档文件。

-z 使用 gzip 压缩或解压缩（tar.gz 或 tgz）。
-j 使用 bzip2 压缩或解压缩（tar.bz2）。
-J 使用 xz 压缩或解压缩 （tar.xz）。

-f 指定生成或者处理的归档文件名称。

-v 打印正在处理的文件。
-C 指定解压缩处理后的目录。

示例
```
tar -Jxvf mysql-8.4.4-linux-glibc2.28-x86_64.tar.xz -C /usr/local/
```