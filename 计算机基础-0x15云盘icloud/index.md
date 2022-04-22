# 云盘 icloud

摘要：云盘是一种专业的互联网存储工具，是互联网云技术的产物，它通过互联网为企业和个人提供信息的储存，读取，下载等服务。具有安全稳定、海量存储的特点。
<!--more-->

# 云盘 icloud
## 常见云盘
- Onedrive
- 百度云盘
- 阿里云盘
- Google云盘

## Onedrive 云盘
### 1、安装
安装office时，选择onedrive安装选项。

### 2、注册（校园账号）
路径：[link](https://products.office.com/en-us/student?tab=students)
登录学校邮箱账号(.edu.cn结尾)，填写信息，会收到验证码，点击提交。
- **e7vyirau@mail.hrka.net**

### 3、登陆、注销与删除
登陆：onedrive->右键设置->账户-> 添加账户。  
注销：onedrive->右键设置->账户->取消链接此电脑。  
删除：`win+R`->regedit->输入`[HKEY\_CLASSES\_ROOT\CLSID]和[HKEY\_CLASSES\_ROOT\WOW6432Node]`下ctrl+F搜索OneDrive账号栏所显示的内容，删除该内容。  

### 4、同步 mklink
```bash
$ mklink /d ..\OneDrive\work E:\work
```
注意：`..\OneDrive\work` 不能提前存在.

## Google 云盘
### 1. 团队云盘
- 团队共享网盘[github](https://github.com/yyuueexxiinngg/some-scripts/blob/master/workers/google/drive/create-share-teamdrive.js)
- https://gd.zxd.workers.dev/


