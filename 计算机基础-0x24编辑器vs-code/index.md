# 编辑器 Visual Studio Code

摘要：Visual Studio Code（简称“VS Code”）是Microsoft在2015年4月30日Build开发者大会上正式宣布一个运行于 Mac OS X、Windows和 Linux 之上的，针对于编写现代Web和云应用的跨平台源代码编辑器，可在桌面上运行，并且可用于Windows，macOS和Linux。它具有对JavaScript，TypeScript和Node.js的内置支持，并具有丰富的其他语言（例如C++，C＃，Java，Python，PHP，Go）和运行时（例如.NET和Unity）扩展的生态系统。
<!--more-->

# 编辑器 Visual Studio Code
## 安装
## 配置文件setting.json
**注意：语言环境，需提前安装，如Miniconda, MinGW64等。**
- 设置默认的语言环境, 字体，插件配置等。
- File->Preferences->Settings->settings.json。
- shift+ctrl+p->settings.json。
- **python环境：python(Extension)可以切换python环境，实质是创建在.vscode下创建一个settings.json文件保存路径。**

```json
{
    // python
    "python.pythonPath": "D:/Miniconda3/python.exe",    // python默认环境路径
    "python.venvPath": "D:/Miniconda3/envs",
    // "python.pythonPath": "D:/Miniconda3/envs/cv/python.exe",
    
    "python.dataScience.askForKernelRestart": false,    
    "python.terminal.activateEnvironment": true, // cond环境默认激活
    
    // java 
    // "javahome":"D:/java1.8.0",    // 默认环境路径

    // c/c++ 
    "C_Cpp.default.compilerPath": "D:/mingw64/bin/gcc.exe", // 默认环境路径
    
    // code runner
    "code-runner.runInTerminal": true,
    "code-runner.executorMap":{ // 环境路径设置
            "python": "cd $dir && python $fileName ",
            // "java": "cd $dir && javac -encoding utf-8 $fileName && java $fileNameWithoutExt",
            "c"     : "cd $dir && gcc $fileName -o $fileNameWithoutExt && $dir$fileNameWithoutExt",                
            "cpp" : "cd $dir && g++ $fileName -o $fileNameWithoutExt && $dir$fileNameWithoutExt",       
    },
    "code-runner.saveFileBeforeRun": true, 
    "code-runner.preserveFocus": true, // 若为false，run code后光标会聚焦到终端上。如果需要频繁输入数据可设为false        
    "code-runner.clearPreviousOutput": false, // 每次run code前清空属于code runner的终端消息

    // git路径设置
    "git.path": "D:/Git/bin/git.exe",

    // sync
    // "sync.forceUpload": true,
    // "sync.gist": "3fa28d5fe3c672683e7624d811c39597",

    //vsicons
    "vsicons.dontShowNewVersionMessage": true,

    // terminal设置
    "terminal.integrated.cursorBlinking": true,
    "terminal.integrated.confirmOnExit": true,
    "terminal.integrated.copyOnSelection": false,
    "terminal.integrated.commandsToSkipShell": [

        "workbench.action.terminal.copySelection",
        "workbench.action.terminal.paste",
    ],
    "terminal.integrated.lineHeight": 1.2,
    "terminal.integrated.letterSpacing": 0.1,
    "terminal.integrated.fontSize": 15, //字体大小设置
    "terminal.integrated.fontFamily": "Lucida Console", //字体设置
    "terminal.integrated.cursorStyle": "line",

    //代码格式化 pip install yapf,flake8
    "python.formatting.provider": "yapf",
    "python.formatting.yapfArgs": [
        "--style",
        "{based_on_style: chromium, indent_width: 4}"
    ],
    "python.linting.pylintEnabled": false,
    // "python.linting.flake8Enabled": true,
    // "python.linting.flake8CategorySeverity.F": "Information",
    // "python.linting.flake8Args": [
    //     "--max-line-length=248",
    //     // "--extend-ignore=F401",    //忽略某种错误
    // ],

    //TODO设置
    "todo-tree.highlights.customHighlight": {
        "TODO": {
            "icon": "check",
            "foreground": "#000",
            "background": "#cecb32",
            "iconColour": "#fffc43"
        },
        "FIXME": {
            "icon": "alert",
            "foreground": "#fff",
            "background": "#ca4848",
            "iconColour": "#ff4343"
        }
    },
    "todo-tree.highlights.defaultHighlight": { "type": "text"},
    "todo-tree.tree.showScanModeButton": false,

    // markdown
    "markdown.preview.fontSize": 16,
    "markdown-preview-enhanced.previewTheme": "github-dark.css",

    // 字体, 大小和格式设置
    "editor.minimap.enabled": false,
    "editor.fontSize": 18,
    "explorer.confirmDelete": false,
    "workbench.iconTheme": "vscode-icons",
    "editor.fontWeight": "400",
    "editor.lineHeight": 24,
    "editor.tabSize":4,
    "editor.letterSpacing": 0.5,
       "editor.rulers": [80],
    "editor.wordWrap": "on",
    //文件保存
    "files.autoSave": "onFocusChange",
    // 文件排除
    "files.exclude": {
        "**/__pycache__": true,
        "**/.history": true
    },
    "remote.SSH.showLoginTerminal": true,
    "workbench.colorTheme": "Monokai Dimmed",
}
```

## 编译置
### 1、tasks.json
- [link](https://code.visualstudio.com/docs/cpp/config-msvc#_run-vs-code-outside-the-developer-command-prompt)
- 设置指令编译代码。
- `shift+ctrl+p`->`tasks.json`。
- C/C++ 编译路径。

```json
{
    // See https://go.microsoft.com/fwlink/?LinkId=733558
    // for the documentation about the tasks.json format
    "version": "2.0.0",
    "tasks": [
        {
            "taskName": "Python",
            "type": "shell",
            "command": "${workspaceRoot}/env/bin/python3.5",
            "args": [
                "${file}"
            ],
            "presentation": {
                "reveal": "always",
                "panel": "shared"
            },
            "group": {
                "kind": "build",
                "isDefault": true
            }
        },
        {
            "type": "shell",
            "label": "compile",
            "command": "g++",  // 编译路径 gcc c语言
            // "command": "D:/mingw64/bin/g++.exe", 
            "args": [
                "-g",
                "-std=c++11",
                "${file}",  // # "${workspaceFolder}/*.cpp" 多文件编译
                "-o",
                "${fileDirname}\\${fileBasenameNoExtension}.exe"
            ],
            "problemMatcher": [
                "$gcc"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            }
        }
    ]
}
```

### 2、c_cpp_properties.json
- 头文件导入和自动补全。
- `ctrl+shift+p` -> `C/Cpp: Edit configurations `。

```json
{
    "configurations": [
        {
            "name": "Win32",
            "includePath": [
                "${workspaceFolder}/**",
                "D:/mingw64//include",
                "D:/mingw64//x86_64-w64-mingw32/include"
            ],
            "defines": [
                "_DEBUG",
                "UNICODE",
                "_UNICODE"
            ],

            "cStandard": "c11",
            "cppStandard": "gnu++14",
            "intelliSenseMode": "clang-x64", 
            "browse": {
                "path": [
                    "D:/mingw64/include",
                    "D:/mingw64/x86_64-w64-mingw32/include",
                    "${workspaceRoot}"
                ],
                "limitSymbolsToIncludedHeaders": true,
                "databaseFilename": ""
            },
            "compilerPath":  "D:/mingw64/bin/g++.exe" 
        },
        {
            "name": "Mac",
            "includePath": [
                "/usr/include",
                "/usr/local/include",
                "${workspaceRoot}"
            ],
            "defines": [],
            "intelliSenseMode": "clang-x64",
            "browse": {
                "path": [
                    "/usr/include",
                    "/usr/local/include",
                    "${workspaceRoot}"
                ],
                "limitSymbolsToIncludedHeaders": true,
                "databaseFilename": ""
            },
            "macFrameworkPath": [
                "/System/Library/Frameworks",
                "/Library/Frameworks"
            ]
        },
        {
            "name": "Linux",
            "includePath": [
                "/usr/include",
                "/usr/local/include",
                "${workspaceRoot}"
            ],
            "defines": [],
            "intelliSenseMode": "clang-x64",
            "browse": {
                "path": [
                    "/usr/include",
                    "/usr/local/include",
                    "${workspaceRoot}"
                ],
                "limitSymbolsToIncludedHeaders": true,
                "databaseFilename": ""
            }
        }
    ],
    "version": 4
}
```

## 调试
### 1、调试配置launch.json
- launch.json设置执行环境来执行代码。
- Run(ctrl+shift+d)->Add Configuration->python 或者cC++(GDB/LLDB)->生成launch.json。
- 设置python路径。

```json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [

        {
            "name": "Python",            // 调试名称
            "type": "python",            // 语言
            "request": "launch",         
            "program": "${file}",     
            "console": "integratedTerminal", // vscode终端
            "stopOnEntry": false
        },
        {
            "name": "C/C++ (gdb) Launch", // 调试名称
            "type": "cppdbg",             // 语言
            "request": "launch",          //launch(直接调试)和attach（附加）。
            "program": "${fileDirname}/${fileBasenameNoExtension}.exe", //程序所在路径和程序名。
            "args": [],                   // 填命令行参数（main函数的形参）。
            "stopAtEntry": false,         // 开始运行程序时，直接暂停。
            "cwd": "${workspaceFolder}",  // 目标工作目录。
            "environment": [],            // 临时手动添加环境变量。
            "externalConsole": false,     // 是否在运行时额外打开终端。
            "MIMode": "gdb",              // 指定调试器gdb或lldb。
            "miDebuggerPath": "gdb",      // 调试环境路径。
            // "miDebuggerPath": "D:/mingw64/bin/gdb.exe",   
            "preLaunchTask": "compile",
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
        },

    ]
}
```

### 2、调试命令
`Step Into`：单步执行，遇到子函数就进入并且继续单步执行（简而言之，进入子函数）；

`Step Over`：在单步执行时，在函数内遇到子函数时不会进入子函数内单步执行，而是将子函数整个执行完再停止，也就是把子函数整个作为一步。

`Step Out`：当单步执行到子函数内时，用step out就可以执行完子函数余下部分，并返回到上一层函数。

`Continue`: 执行到下一个断点。

`Restart`：重头开始。

`stop`：退出。

### 3、Debug console
实现交互功能，命令查看.

## 用户脚本 User Snippet
```json
{
	// Place your snippets for python here. Each snippet is defined under a snippet name and has a prefix, body and 
	// description. The prefix is what is used to trigger the snippet and the body will be expanded and inserted. Possible variables are:
	// $1, $2 for tab stops, $0 for the final cursor position, and ${1:label}, ${2:another} for placeholders. Placeholders with the 
	// same ids are connected.
	// Example:
	// "Print to console": {
	// 	"prefix": "log",
	// 	"body": [
	// 		"console.log('$1');",
	// 		"$2"
	// 	],
	// 	"description": "Log output to console"
	// }
	"HEADER": {        
		"prefix": "header",        
		"body": [            
				"#!/usr/bin/env/python",
				"# -*- encoding: utf-8 -*-",
				"\"\"\"",
				"################################################################################", 
				"@File              :   $TM_FILENAME",
				"@Time              :   $CURRENT_YEAR/$CURRENT_MONTH/$CURRENT_DATE $CURRENT_HOUR:$CURRENT_MINUTE:$CURRENT_SECOND",
				"@Author            :   Lyangly ",
				"@Version           :   1.0",
				"@Desc              :   None",
				"################################################################################", 
				"\"\"\"",       
				"",  
				"",  
				"# standard packages", 
				"", 
				"# third-party packages", 
				"", 
				"# sub-toolkit packages", 
				"", 
				"",
				"",
		],	
	},
	"PARSE_ARGS": {
		"prefix": "parse_args",
		"body": [
				"def parse_args():",
				"    parser = argparse.ArgumentParser(description=\"test\")",
				"    parser.add_argument(\"--path\", type=str, default=\".\", help=\"Project Path\")",
				"    return parser.parse_args()",
				"",
				"def main():",
				"    function(**vars(parse_args()))",
				"",
				"",
				"if __name__ == '__main__':",
				"    main()",
		],
	},
	"COMMENT": {        
		"prefix": "comment",        
		"body": [            
				"r\"\"\"Overview",
				"",             
				"Description",                      
				"", 
				"Args:", 
				"    variable (type): introduction", 
				"", 
				"Returns:",
				"    variable (type): introduction",
				".. warning::",       
				"    context",
				"",
				".. note::",
				"    context",
				"",
				"Example::",
				"    >>> model = class()",
				"\"\"\"",                
		],	
	},
	"TODO": {        
		"prefix": "todo",        
		"body": [                     
				"# TODO:(Lyangly,522927317@qq.com) ",                           
			],	
    }
}
```

## 插件
### 1、Git
命令：
```bash
ctrl+shift+G
```

**需要忽略.history文件夹**:
```
touch .gitignore
###################
.history
.vscode
################
# settings -> files.exclude 
**/.history
**/.vscode
**/__pycache\__
```

### 2、Local History

### 3、git history

### 4、python
```
"python.pythonPath": "D:/Miniconda3/envs/speech/python.exe",
```

### 5、c/c++

### 6、TODO highlight 和 Todo Tree
```json
// ctrl+shift+p->settings.son
//TODO设置
"todo-tree.highlights.customHighlight": {
"TODO": {
  "icon": "check",
  "foreground": "#000",
  "background": "#cecb32",
  "iconColour": "#fffc43"
},
"FIXME": {
  "icon": "alert",
  "foreground": "#fff",
  "background": "#ca4848",
  "iconColour": "#ff4343"
}
},
"todo-tree.highlights.defaultHighlight": {
"type": "text"
},
"todo-tree.tree.showScanModeButton": false,
```

### 7、vscode-icons
```
// ctrl+shift+p->settings.son
"vsicons.dontShowNewVersionMessage": true,
```

### 8、Coder Runner
```
// ctrl+shift+p->settings.son
// code runner
 "code-runner.runInTerminal": true,
 "code-runner.executorMap":{
    "java": "cd $dir && javac -encoding utf-8 $fileName && java $fileNameWithoutExt"
},
```

### 9、Markdown plugh
- `Markdown Preview Enhanced`, `markdownlint`和`Markdown All in One`。

```
// ctrl+shift+p->settings.son
// markdown
"markdown.preview.fontSize": 16,
"markdown-preview-enhanced.previewTheme": "github-dark.css",
```

问题：markdown 图片不能显示
1. 在VScode编辑页面，按快捷键Ctrl+Shift+P，搜索markdown更改预览安全设置，选择允许不安全内容，即可显示预览网页图片。
2. 本地图片显示预览，要注意.md文件的路径和图片存放的路径，使用图片的相对路径即可显示预览。（注意路径不能有引号" "）。

```markdown
![abs](./RE32bIh_1920x1080.jpg)
```

### 10、picgo
1. github配置，github -> Settings -> Developer settings -> Personal access tokens -> vscode sync-gist -> tokens->repo。

2. picgo 配置，vsocde->extensions->picgo->extension setting。

3. 无法加载github图片：复制这一段代码到`C:\Windows\System32\drivers\etc`下的hosts文件。

```
# GitHub Start 
192.30.253.112 github.com 
192.30.253.119 gist.github.com
151.101.184.133 assets-cdn.github.com
151.101.184.133 raw.githubusercontent.com
151.101.184.133 gist.githubusercontent.com
151.101.184.133 cloud.githubusercontent.com
151.101.184.133 camo.githubusercontent.com
151.101.184.133 avatars0.githubusercontent.com
151.101.184.133 avatars1.githubusercontent.com
151.101.184.133 avatars2.githubusercontent.com
151.101.184.133 avatars3.githubusercontent.com
151.101.184.133 avatars4.githubusercontent.com
151.101.184.133 avatars5.githubusercontent.com
151.101.184.133 avatars6.githubusercontent.com
151.101.184.133 avatars7.githubusercontent.com
151.101.184.133 avatars8.githubusercontent.com
```

注意：需要修改权限：右键hosts文件属性-安全-编辑-添加-高级-立即查找，选择你的账户，确定-确定，勾选“完全控制”，确定-确定即可。

### 11、Java Extension Pack
```
// setting.json
"java.home" = D:/jdk-1.8
```

java调试：`ctrl+shift+p` ->`java Open java language server log file`打开日志查看问题原因。

问题：`BUILD FAILED, DO YOU WANT TO CONTINUE?`

- 原因：.histoy文件夹缓存java文件问题
- 解决：local histoy忽略java文件

```json
"local-history.exclude": [
"**/.history/**",
"**/*.java",
]
```

### 12、Settings Sync
1. 插件栏搜索Settings Sync并安装。安装完之后会弹出一个登陆界面,这里点击login with github，直到`Success! You may now close this tab`。
2. 进入github -> Settings -> Developer settings -> Personal access tokens -> vscode sync-gist -> tokens。
3. 回到VSCode配置将token配置到本地，ctrl+shift+p->`sync:advanced option`->syns:打开设置，配置vscode sync-gist的tokens。
4. 设置上同步上传设置。(Sync: Update / Uplaod Settings) Shift + Alt + U 在弹窗里输入你的token， 回车后会生成syncSummary.txt文件。
5. 设置上同步下载设置。(Sync: Download  Settings) Shift + Alt + D 在弹窗里输入你的gist值，稍后片刻便可同步成功。
6. 如果要重置同步设置，变更其它token。Ctrl+P / F1 弹出输入>sync，即可重新配置你的其它token来同步。

```
vscode://vscode.github-authentication/did-authenticate?windowid=1&code=15f3e2a834ec565b4783&state=2d01d65e-7c1a-4f55-b67e-dcde2994cf71
```

### 13、Remote-WSL
WSL linux开发。

### 14、Remote-SSH
1. 生成密钥id\_rsa1.pub并上传到远程服务器上.ssh目录下。

```bash
$ cd ~/.ssh
$ ssh-keygen
$ ls 
-a----         2020/9/12     20:21            128 config
-a----         2020/9/12     20:24           1679 id_rsa
-a----         2020/9/12     20:24            396 id_rsa.pub
```

2. 用ssh命令（ssh username@ip -p port）连接远程主机，并将`id\_rsa1.pub`加入到`authorized_keys`中。

```bash
$ ssh username@ip -p port
$ cd .ssh
$ ssh-add ~/.ssh/id_ras
# Could not open a connection to your authentication agent
$ ssh-agent bash
$ cat id_rsa.pub >> authorized_keys
$ exit
```

3. 用私钥登录，此次登录无需输入密码。

```bash
$ ssh username@ip -p port –i id_rsa
```

4. 配置文件Remote SSH。

```bash
# Remote explorer -> configure -> ~/.ssh/config
# .vscode\extensions\ms-vscode-remote.remote-ssh-0.54.0
Host 连接的主机的名称，可自定
    Hostname 远程主机的IP地址
    User 用于登录远程主机的用户名
    Port 22
    IdentityFile ~/.ssh/id_rsa
    PreferredAuthentications publickey
```

```
问题：
exec request failed, try username/server/account as login name.
```

5. 注意：修改ssh免密登录

```bash
$ sudo vim /etc/ssh/sshd_config 
# PermitRootLogin prohibit-password 
PermitRootLogin yes
PubkeyAuthentication no
PasswordAuthentication no
```

6. 过程试图写入的管道不存在。

```bash
# Remote explorer -> configure -> ~/.ssh/config
# 127.0.0.1
Host Linux_WSL
    HostName 127.0.0.1
    User ly
    IdentityFile ~/.ssh/id_rsa 
```

### 15、docker
- Atom Material Theme
- Beteer Align
- Bracker Pair Colorizer
- change-case: 代码格式 驼峰
- koroFileHeader

### 16、Jupyter
```
Enter : 转入编辑模式
Shift-Enter : 运行本单元，选中或插入（最后一个Cell的时候）下个单元
Ctrl-Enter : 运行本单元
Alt-Enter : 运行本单元，在其下插入新单元
Y : 单元转入代码状态
M :单元转入markdown状态 （目前尚不支持R 原生状态）
Up : 选中上方单元
K : 选中上方单元
Down : 选中下方单元
J : 选中下方单元
A : 在上方插入新单元
B : 在下方插入新单元
D,D : 删除选中的单元
L : 转换行号
Shift-Space : 向上滚动
Space : 向下滚动    
```

## 字体
1. 打开vscode的配置页面，并搜索font。
2. 修改editor.fontFamily配置项的内容为：'Fira Code Retina', 'Sarasa Term SC Regular'。Fira Code为首选字体，更纱黑体为备选字体。
3. 勾选editor.fontLigatures配置，启用连字符。
4. 配置字体过程中若提示字体无法识别，尝试配置后重启vscode。
5. 返回vscode编辑器，输入@，r，&字符验证字体是否生效。

## 参考
[1] https://www.cnblogs.com/moshuying/p/11348634.html  
[2] https://www.cnblogs.com/kenz520/p/7416836.html  
