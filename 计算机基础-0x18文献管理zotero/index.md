# 文献管理 Zotero

摘要：zotero是开源的文献管理工具，可以方便的收集，组织，引用，和共享文献的工具。由安德鲁·w·梅隆基金会，斯隆基金会以及美国博物馆和图书馆服务协会资助开发。
<!--more-->

# 文献管理 Zotero
## 安装
- [Zotero](https://www.zotero.org/)
- [Zotero Connector](https://www.zotero.org/download/connectors) 网络剪切插件

## 设置
### 1、字体
Zotero->工具栏->Edit->Preferences->Advanced->Config Editor->Search。
```
# extensions.zotero.note.css  格式设置
p {line-height: 1.4; margin-bottom: 1en;} 
li {line-height: 1.2; margin-bottom: 1en;} 
blockquote {margin-left: 1en;}
# extensions.zotero.note.fontSize 字号设置
15
# extensions.spellcheck.inline.max-misspellings 拼写检查
0
```

### 2、sci-hub源
Zotero->工具栏->Edit->Preferences->Advanced->Config Editor->Search (`extensions.zotero.findPDFs.resolvers`)->修改。

```json
[{
	"name":"Sci-Hub",
    "method":"GET",
    "url":"https://sci-hub.tw/{doi}",
    "mode":"html",
    "selector":"#pdf",
    "attribute":"src",
    "automatic":true
}]
```

1. `url`: `https://sci-hub.tw/{doi}`中，建议使用`.tw`，因为相比`.se`、`.si`、`.shop`等，`.tw`的访问速度更好。
2. `url`:`https://sci-hub.tw/{doi}`中。由于Sci-Hub是通过doi下载文献的，因此该PDF解析器也需要doi。也就说文献必须要有doi，如果doi是空缺的，便无法通过PDF解析器免费下载文献。对于缺失doi的文献，可以通过插件zotero-shortdoi插件一键抓取doi。
3. `"automatic":true`，如果设置为true，Zotero会自动下载保存到Zotero中的文献的PDF。比如用Zotero Connector保存了一些文献到Zotero，它便会自动从Sci-Hub下载文献，并附在相应文献条目下。如果不需要自动下载，可以设置为`"automatic":false`。

### 3、自定义数据存储位置
Zotero->Edit->Preferences->Advanced->Files and Folderes
- Base directory 源数据存储位置，与ZotFile设置中`Source Folder for Attaching new Files`对应，`F:`，F可以根据自己选择不同盘。
- Data Directory Location->Custom->设置 自定义保存路径，与ZotFile设置中Custom Location ZotFile一致, `F:\Zotero`。

## 文献
### 1、文献导入
浏览器插件Zotero Connector。（注意，保证Zotero客户端在运行状态。）
- Save to Zotero。
  1. Embedded MetaData    元数据。
  2. Web Page with Snapshot 离线保存。
  3. Web Page without Snapshot 。

文件手动导入：
- PDF
  1. 会自动识别PDF中元数据，失败可以Retrieve，或者创建父条目。
  2. 文献->选中右键-> Retrieve Metadata for PDF。
  3. 文献->选中右键-> Create Parent Item。
- 参考文献
  1. 文献->选中右键->Add attachment。
     - Attach Stored Copy of File 复制进文献库。
     - Attach Link to File   超链接，附件指向文件。
  2. Zotero->工具栏->魔棒按钮 Add Item(s) by identifier->ISBN、DOI、PMID或arXiv ID。
- 条目: Zotero->工具栏->加号按钮 New Item->Journal Article。

### 2、文献管理
#### 文件夹（collection）管理
- Zotero->工具栏->New Collection。
- 层级树状结构。
- 数据来源层级的区分。

#### 标签（tag）管理
- Item->Tag->Add (View->Layout->Item Pane)。
- 细粒分类。
- 界面左下角的tag面板，选中targ，Assign Color，9种不同的颜色。

#### 便捷搜索（saved search）
- My Library->New Saved Search。
- Zotero->工具栏->->Advanced Search。
- 限定条件并搜索。

### 3、文献导出
格式设置：Zotero->工具栏->Edit->Preferences->Cite->Get additional styles->Style Search->china (GB/T 7714)。

文献导出：
1. Export Items 将文献导出成不同的格式，包括BibTex、BookmarksL等，也同时可以导出附件、文章分享或者数据迁移。
2. Create Bibliography from Items 生成参考文献，选择不同的参考文献格式和导出的形式，默认是复制到剪贴板。
3. Generate Report from Items 生成一个固定格式的HTML report，里面有每篇文献的元数据。

Word->Zotero->Add/Edit Ctation->添加文献。

Word->Zotero->Insert Bibliography。  

### 4、文献删除
`Remove Item from Collection` ：从文献集中删除,只是把文献从当前所在文献集中删除，并不会将其从我的文库和它所在的其他文献集中删除，更不会将其附件从本地删除，也就无法将其从云端删除。

`Move Item to Trash`：移至回收站，该文献在所有它存在的文献集中的条目将全部删除，并且PDF附件也会移至Zotero回收站。

如果想让文献的附件彻底地从本地删除，只需要右键Zotero的Trash，并选择Empty Trash。

云端删除，Zotero->工具栏->同步sync with zotero.org。

## 插件
插件集合: https://www.zotero.org/support/plugins

### 1、安装
Zotero->工具栏->Tools->Add-ons->齿轮按钮->Install Add-on From File->选取插件。

### 2、常见插件
#### Quicklook 快速预览
Quicklook 文件快速预览插件： [zoteroquicklook.zoteroplugin](https://github.com/mronkko/ZoteroQuickLook/releases)。

#### shortdoi DOI
zotero-shortdoi 文件下载插件： [zotero-shortdoi-1.3.9.xpi](https://github.com/bwiernik/zotero-shortdoi/releases)
插件设置：文献->Item pane->info->DOI->设置。
插件操作：

- 文献->选中右键->Manage DOIs
  - get shortDOIs
  - get longDOIs
  - verify and clean DOIs

#### Markdown Here
markdown 文档编辑插件：[markdown-here](https://github.com/adam-p/markdown-here)。

插件安装：
1. 创建mdh文件夹。
2. 复制common，firefox，chrome.manifest，install.rdf到mdh文件夹。
3. 修改mdh文件夹为mdh.xpi的后缀文件。
4. 导入mdh.xpi文件到Zotero。

插件操作：
1. 文献->Item pane->Notes->Add
2. ctrl+alt+M渲染

#### ZotFile
ZotFile 文献管理插件 [zotfile.xpi](https://github.com/jlegewie/zotfile)
插件配置： Zotero->Tools->Zotfile Preferences。

- 同步设置(**关键**)：
  1. General Settings->Source Folder for Attaching new Files 附加新文件的源文件夹，与Zotero本身数据存储位置一致（Advanced->Files and Folders->Custom）+storage，`F:\Zotero\storage`。
  2. General Settings->Location of Files->Custom Location ZotFile自定义保存目录，与Zotero本身链接附件的根目录对应（Advanced->Files and Folders->Base directory）,`F:\Zotero\resources`。
- 格式设置：
  1. General Settings->Location of Files->Use subfolder defined by (\%c\%t)
  2. Renaming Rules->Format for all Item Types except Patents ({%y\_}{%a\_}{%t})
  3. Advanced Settings->Other Advanced Settings->Automatically rename new attachments (Always rename)

插件操作：
- 文献->选中右键->Manage Attachments->Rename Attachments

移动设备同步：Tablet
- Zotero->Tools->Zotfile Preferences->Tablet Settings
  - Use ZotFile to send and get files from tablet
  - Base Folder
  - Custom subfolders
  - Use additional subfolders edfined by (/%w/%y)

## 同步
### 1、默认同步（仅同步条目）
Zotero->Edit->Preferences->Sync->Data Syncing
注意：不勾选Syn full-text content，300M内存不够。

### 2、WebDAV（可选）
Zotero->Edit->Preferences->Sync->File Syncing
   - [坚果云](https://dav.jianguoyun.com/dav)->账户信息->安全选项->第三方应用管理->添加应用->生成密码。
   - 参数"URL": https://dav.jianguoyun.com/dav。

### 3、ZotFile+同步盘（推荐）
1. 需要设置自定义数据存储位置 和ZotFile 插件。
2. 同步Zotero文件夹，其他设置设置Zotero文件夹的位置，即设置Base directory和Data Directory Location。

## 参考文献

- [01-Hsin-我的 Zotero 实践汇总-知乎](https://zhuanlan.zhihu.com/p/108366072)
- [02-你好啊-zotero、zotfile、坚果云之间的爱恨情长-知乎](https://zhuanlan.zhihu.com/p/93757626)
- [03-青柠学术-Zotero-知乎](https://www.zhihu.com/column/zotero)
