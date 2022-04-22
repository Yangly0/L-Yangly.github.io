# 多视窗 Screen

摘要：screen是linux下的一种视窗多重复用管理程序。
<!--more-->

# 多视窗 Screen
## 创建
```bash
$ screen -s [windowName]
```

## 挂起
```bash
[键盘命令] ctrl+a+d
```

## 查看
```
$ screen -ls
# detach（离线）或者attach（在线）
# 状态为dead，那么用screen -wipe id清理
```

## 返回
```bash
$ screen -r id
```

## 离线
```bash
$ screen -d id
```

## 退出
```basg
$ screen -r id
$ exit
```


