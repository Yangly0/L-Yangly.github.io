# MD编辑器 Typora

摘要：_Typora_ 是一款由 Abner Lee 开发的轻量级 Markdown 编辑器，与其他 Markdown 编辑器不同的是，_Typora_ 没有采用源代码和预览双栏显示的方式，而是采用所见即所得的编辑方式，实现了即时预览的功能，但也可切换至源代码编辑模式。
<!--more-->

# MD编辑器 Typora
## 图床
### 1. picgo-core
1、软件默认安装；
2、配置文件`xxx/user/.picgo/config.json`;
```json
{
	"picBed":{
		"uploader": "aliyun",
		"aliyun":{
			"accessKeyId":"你的accessKeyId",
			"accessKeySecret": "你的accessKeySecret",
			"bucket": "存储空间名",  
			"area": "存储区域代号oss-cn-beigin",
			"path": "自定义存储路径，可自行修改 img/", 
			"customUrl": "需删除，自定义域名，注意要加http://或者https// 非必填项目",
			"options": "需删除，图片一些后缀处理参数PicGo 2.2.0 + PicGo-Core 1.4.0+ 非必填项", 
		}

	},
	"picgoPlugins":{
	}
}
```

### 2. iPic Mover
图床迁移功能。

## 问题
### 1. 中文与英文符号输入问题。
> `ctrl + .` 即可改回中文输入状态输出中文符号
