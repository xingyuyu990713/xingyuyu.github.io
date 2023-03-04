





> 在typora中，插入的图片本不是在文件当中，而是该文件当中只留下了这个图片所在的路径。那么就造成了如果这个文件给到了别人查看，那么里面的图片是无法看到的。下面就是来解决这个问题。Typora [PicGO](https://so.csdn.net/so/search?q=PicGO&spm=1001.2101.3001.7020)-Core (command line) 配置Github 图床，实现图片自动上传功能，让别人打开你分享的Markdown文档时也能正常看到图片
>
> 思路：在GitHub上，创建一个仓库用于存放图片，这样无论在哪都可以访问图片了。

## 1.配置GitHub

```http
1.创建一个公开的仓库,[Your repositories]-[New]-输入Repository name,选择public,Create repository
2.在刚才创建好的仓库中新建一个文件夹，img
3.创建token，点击[Settings]-[Developer settings]-[Personal access tokens]-[Tokens(classic)]-[New GitHub App],输入Token name,Expiration选择No expiration，勾选repo,提交。然后网页出现一串token，复制保存，这个token只出现一次。
```

## 2.配置Typora

按照如下图配置：

![image-20230304165059864](https://github.com/xingyuyucoder/TyporaImages/raw/main/img/image-20230304165059864.png)

`点击下载或更新以后在本地会出现PicGo-Core的配置文件config.json，路径：C:\Users\xyy24\.picgo\config.json`
将文件中内容替换成下面内容就ok了
```json
{
  "picBed": {
    "current": "github",
    "github": {
      "repo": "xingyuyucoder/TyporaImages",
      "branch": "main",
      "token": "github的token",
      "path": "img/",
      "customUrl": "https://github.com/xingyuyucoder/TyporaImages/raw/main"
    },
    "uploader": "github",
    "transformer": "path"
  },
  "picgoPlugins": {
    "picgo-plugin-github-plus": true
  }
}
customUrl：后面一定是/raw/main
```

![image-20230304171745656](https://github.com/xingyuyucoder/TyporaImages/raw/main/img/image-20230304171745656.png) 

