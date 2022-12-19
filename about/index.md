# 关于


{{< style "min-height: 230px;" >}}
{{< typeit code=javascript >}}

#include <cstring>
#include <iostream>

class Blog{
public:
    Blog();
    Blog(std::string author, std::string url);
    virtual ~Blog();
    void PrintInfo();
private:
    std::string m_author;
    std::string m_url;
};

Blog::Blog():m_author(""), m_url(""){}
Blog::Blog(std::string author, std::string url):m_author(author), m_url(url){}
Blog::~Blog(){}

void Blog::PrintInfo(){
    printf("Blog info: %s, %s", m_author.c_str(), m_url.c_str());
}

int main(int argc, char const *argv[])
{
    Blog* blog = new Blog("Yangliul1", "https://yangliuly1.github.io/");
    if(blog == nullptr){  // 初始化失败
        printf("%s, New Blog failed!", __FUNCTION__);
        return -1;
    }
    blog->PrintInfo();
    delete blog;
    return 0;
}

{{< /typeit >}}
{{< /style >}}

## 资讯

[<img src="https://img.shields.io/github/followers/Yangliuly1.svg?style=social&label=Followers">](https://github.com/Yangliuly1)

{{< admonition info "关于我" false >}}

<div style="display:flex;justify-content:space-around;">
<span>

**角色**：


  - ~~在校大学生~~
  - ~~实习生~~
  - 社畜 ...  

</span>
<span>

**职业**：

  - ~~AI炼丹师~~
  - AI模型部署：量化，算子开发

</span>
</div>

{{< /admonition >}}

> 用我所学，学我所用。不盲目堆叠技术栈，保持谦虚，保持探索欲，砥砺前行。

{{< link href="https://github.com/Yangliuly1" content="@Yangliuly1's Recent activity" card=true >}}

## 初衷

建站的初衷不是為了炫耀所知，而是记录无知。  
人知道的越多，就会发现无知的越多。有更广袤的世界可以探索，真是莫大的快乐！
{{< style "display: block;text-align: right;" small >}}
  —— 创建于 2022-12-19 23:07:01
{{< /style >}}

## 期许

{{% center-quote %}}
不卑不亢，不矜不伐，戒骄戒躁  
不嗔不怒，不争不弃，独善其身
{{% /center-quote %}}

## 订阅

- [RSS](https://yangliuly1.github.io//index.xml)
- [GitHub](https://github.com/Yangliuly1/hugo-blog)



<!-- 音乐
{{< music url="https://cdn.lruihao.cn/files/nanjing.mp3" name="李志" artist="你离开了南京，从此没人和我说话" cover="https://p2.music.126.net/UuSe-Vc6rS7JtRJSQgDU2g==/2323268069553116.jpg?param=300x300" fixed=true >}}
-->


---

> 作者: [Yangliuly1](https://github.com/Yangliuly1)  
> URL: https://yangliuly1.github.io/about/  

