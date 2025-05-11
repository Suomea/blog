
## 配置
编辑配置文件 `vim ~/.vimrc`。
```
set nu 
syntax on 
set nobackup 
set tabstop=4 
set shiftwidth=4 
set expandtab 
set autoindent 
set noswapfile
```

取消行号：`set nonu`

光标跳到行尾：`shift + $`

光标跳到行首：`shift + ^`

光标跳到文件开头：`gg`/`1g`

光标跳到文件结尾：`shift + g`

## 替换
`:n1,n2s/word1/word2/g` 在 n1 行和 n2 行之间，替换 `word1` 为 `word2`。

`:1,$s/word1/word2/g` 在 第一行与最后一行之间寻找 `word1` 替换为 `word2`，等同于 `:%s/word1/word2/g`。

`:s/word1/word2` 当前行的第一个 `word1` 替换为 `word2`。 

`:s/word1/word2/g`  当前行的所有的 `word1` 替换为 `word2`。

`:%s/word1/word2` 每一行的第一个 `word1` 替换为 `word2`。

## 换行
执行如下命令
```shell
# echo hello > test.txt
# ls -lh
-rw-r--r-- 1 root root    6 Mar  1 14:55 test.txt
```

分析输出结果能够看到，只输出了 `hello` 5 个字符，但是 `test.txt` 文件的大小为 6 个字节。

使用命令 od 查看可以看到 echo 命令默认加了一个换行符，即 `0x0a`。

```
# od -x test.txt 
0000000 6568 6c6c 0a6f
0000006
```

echo 有个 -n 选项能够控制不输出换行符。
```
-n     do not output the trailing newline
```

```shell
# echo -n hello > a.txt
# ls -l
-rw-r--r-- 1 root root        5 Mar  1 14:59 a.txt
-rw-r--r-- 1 root root        6 Mar  1 14:55 test.txt

# od -x a.txt 
0000000 6568 6c6c 006f
0000005
```

同样 VIM 在编辑保存文件的时候会在最后一行默认增加换行符，使用如下命令保存文件可取消最后一行的换行符。
```
:set bin noeol
:wq
```

## 行号展示
```
# 设置展示行号
:set nu

# 取消展示行号
:set nonu
```

## 在行首插入字符
1. 使用 Ctrl + v 进入可是块模式。
2. 使用上下键选择要编辑的行。
3. 使用 Shift + i 进入插入模式。
4. 输出要插入的字符。
5. 按键 `Esc` 完成插入操作。