查看规则，默认查看 filter 表：
```
iptables -L -n -v --line-number
```

删除规则：
```
iptables -D INPUT [num]
```

插入规则，插入到第五行，只接受建立连接的 tcp 包且端口号为 18094：
```
iptables -I INPUT 5 -m state --state NEW  -p tcp  --dport 18094  -j ACCEPT
iptables -I INPUT 5 -m conntrack --ctstate NEW -p tcp --dport 18094 -j ACCEPT
```

使用 -A 为追加到最后一行，两个语句等效，下面一个更推荐：
```
iptables -A INPUT -m state --state NEW  -p tcp  --dport 18094  -j ACCEPT
iptables -A INPUT -m conntrack --ctstate NEW -p tcp --dport 18094 -j ACCEPT
```

实例分析
```
# 1. 允许已建立的连接和相关的流量（放行返回包，比如你访问网站后的响应）
iptables -A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT

# 或者旧版写法（如果 conntrack 不可用）：
iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT

# 2. 允许 ICMP (Ping) 请求
iptables -A INPUT -p icmp --icmp-type echo-request -j ACCEPT

# 或者允许所有 ICMP（更宽松）：
iptables -A INPUT -p icmp -j ACCEPT

# 3. 允许本地回环接口 (lo) 的所有流量
iptables -A INPUT -i lo -j ACCEPT

# 4. 允许新的 TCP 连接访问 SSH（端口 22）
iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW -j ACCEPT

# 5. 添加兜底策略
iptables -A INPUT -j REJECT --reject-with icmp-host-prohibited

# iptables -L -n -v --line-numbers
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination
1      31M 5337M ACCEPT     all  --  *      *       0.0.0.0/0            0.0.0.0/0           state RELATED,ESTABLISHED
2        7   484 ACCEPT     icmp --  *      *       0.0.0.0/0            0.0.0.0/0
3    1339K   80M ACCEPT     all  --  lo     *       0.0.0.0/0            0.0.0.0/0
4       73  4072 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:22
· · ·
13   2864K 1098M REJECT     all  --  *      *       0.0.0.0/0            0.0.0.0/0           reject-with icmp-host-prohibited

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination
1        0     0 REJECT     all  --  *      *       0.0.0.0/0            0.0.0.0/0           reject-with icmp-host-prohibited

Chain OUTPUT (policy ACCEPT 383K packets, 89M bytes)
num   pkts bytes target     prot opt in     out     source               destination
```
分析一下上述配置的信息

1. 默认展示的是 filter 表的链，包含三条链：INPUT、OUTPUT、FORWARD
2. INPUT 下包含 13 条规则（省略了 5 到 12 条的数据）
3. 第一条为接收全部已经建立连接的包或者和当前主机发出的包有联系的包。
4. 第二条为接收 icmp 协议的包。
5. 第三条为发送到本地回环网卡 lo 的包全部接受。
6. 第四条为建立 tcp 连接且端口号为 22 的包全部接受。
7. 第十三条为拒绝全部的包。

具体情况

1. ssh 登录

第一条不满足；第二条不满足；第三条不满足；第四条建立 tcp 22 端口连接满足放行（结束）。建立连接之后后续的包都满足第一条全部放行。

1. 比如访问 23 端口

第一条不满足；------ 满足第十三条，第十三条为丢弃（结束）

设置默认的策略

上述有 13 条规则，如果进来的包都不匹配全部的十三条规则的话（上述的案例中不会发生这种情况，因为第13条规则匹配所有的包进行拒绝），那么我们可以设置默认的策略进行兜底。

```
iptables -P INPUT DROP

iptables -P OUTPUT ACCEPT

iptables -P FORWARD ACCEPT
```


安装服务
```
apt install iptables-persistent
```

手动保存、恢复规则
```
netfilter-persistent save
iptables-restore < /etc/iptables/rules.v4
```

 `iptables-persistent` 它会在每次系统启动时 ​**​自动恢复你之前保存的规则​**​，无需手动操作。但是保存需要手动保存。

参考连接：[鸟哥的 Linux 私房菜 -- Linux 防火墙与 NAT 服务器 (vbird.org)](http://cn.linux.vbird.org/linux_server/0250simple_firewall_3.php)