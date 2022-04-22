# 多终端 Tmux

摘要：tmux是指通过一个终端登录远程主机并运行后，在其中可以开启多个控制台的终端复用软件
<!--more-->

# 多终端 Tmux
## tmux-session
### 1、查询
```bash
# 列出所有绑定的键
$ tmux list-key
# 列出所有命令
$ tmux list-command
```

### 2、创建
```bash
# 默认创建
$ tmux new
# 自定义创建,名为mysession的会话
$ tmux new -s [mysession]
```

### 3、挂起
```
[键盘命令1] ctrl + b
[键盘命令2] d
```

### 4、查看
```bash
# 显示会话列表
$ tmux ls
```

### 5、恢复
```bash
# 连接上一个会话
$ tmux a
# 连接指定会话
$ tmux a -t mysession
### [键盘命令] 
[键盘命令] ctrl+b s
```

### 6、重命名
```bash
# 重命名会话s1为s2
$ tmux rename -t s1 s2
```

### 7、关闭
```bash
# 关闭上次打开的会话
$ tmux kill-session
# 关闭会话s1
$ tmux kill-session -t s1
# 关闭除s1外的所有会话
$ tmux kill-session -a -t s1
# 关闭所有会话
$ tmux kill-server
```

### 8、翻页
```
# 启动
[键盘命令1]  ctrl+b [ 
# 操作
[键盘命令2] 上下  翻页
# 退出
[键盘命令3] q  退出
```

## tmux-session-window
### 1、创建window
```
[键盘命令] Ctrl+b c
```

### 2、删除window
```
[键盘命令] Ctrl+b &
```

### 3、上下window切换
```
[键盘命令] Ctrl+b n
[键盘命令] Ctrl+b p
```

### 4、重命名window
```
[键盘命令] Ctrl+b ,
```

### 5、搜索关键字
```
[键盘命令] Ctrl+b f
```

### 6、相邻window切换
```
[键盘命令] Ctrl+b l
```

##  tmux-session-window-pane

### 1、创建pane
```
# 横切split pane horizontal
[键盘命令] Ctrl+b ” (问号的上面，shift+’)
# 竖切split pane vertical
[键盘命令] Ctrl+b % （shift+5）
```

### 2、顺序pane切换
```
[键盘命令] Ctrl+b o
```

### 3、上下左右选择pane
```
[键盘命令]Ctrl+b 方向键上下左右
```

### 4、调整pane的大小
```
[键盘命令] Ctrl+b :resize-pane -U #向上
[键盘命令] Ctrl+b :resize-pane -D #向下
[键盘命令] Ctrl+b :resize-pane -L #向左
[键盘命令] Ctrl+b :resize-pane -R #向右

# 在上下左右的调整里，最后的参数可以加数字 用以控制移动的大小
[键盘命令] Ctrl+b :resize-pane -D 50
```

### 5、同窗口移动pane
```
[键盘命令] Ctrl+b { （往左边，往上面）
[键盘命令] Ctrl+b } （往右边，往下面）
```

### 6、删除pane
```
[键盘命令] Ctrl+b x
```

### 7、更换pane排版
```
[键盘命令] Ctrl+b “空格”
```

### 8、移动pane至window
```
[键盘命令] Ctrl+b !
```

### 9、移动pane合并至某个window
```
[键盘命令] Ctrl+b :join-pane -t $window_name
```

### 10、显示pane编号
```
[键盘命令] Ctrl+b q
```

### 11、按顺序移动pane位置
```
[键盘命令] Ctrl+b Ctrl+o
```

## 其他命令
### 1、复制模式
```
[键盘命令] Ctrl+b [
空格标记复制开始，回车结束复制。
```

### 2、粘贴最后一个缓冲区内容
```
[键盘命令] Ctrl+b ]
```

### 3、选择性粘贴缓冲区
```
[键盘命令] Ctrl+b =
```

### 4、列出缓冲区目标
```
[键盘命令]Ctrl+b :list-buffer
```

### 5、查看缓冲区内容
```
[键盘命令] Ctrl+b :show-buffer
```

### 6、vi模式
```
[键盘命令] Ctrl+b :set mode-keys vi
```

### 7、显示时间
```
[键盘命令] Ctrl+b t
```

### 8、快捷键帮助
```
[键盘命令] Ctrl+b ? (Ctrl+b :list-keys)
```

### 9、tmux内置命令帮助
```
[键盘命令] Ctrl+b :list-commands
```
