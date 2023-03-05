




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

<img src="https://typora-images-1307361841.cos.ap-beijing.myqcloud.com/img/image-20230304165059864.png" alt="image-20230304165059864" style="zoom: 25%;" />

`点击下载或更新以后在本地会出现PicGo-Core的配置文件config.json，路径：C:\Users\xyy24\.picgo\config.json`

```json
将文件中内容替换成下面内容就ok了
{
  "picBed": {
    "current": "github",
    "github": {
      "repo": "xingyuyucoder/TyporaImages",
      "branch": "main",
      "token": "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",
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

<img src="https://typora-images-1307361841.cos.ap-beijing.myqcloud.com/img/image-20230304171745656.png" alt="image-20230304171745656" style="zoom:50%;" /> 

## 3.配置腾讯云图床

> 1. 由于GitHub的图床很慢，需要科学上网才能正常查看图片，所以我们下面配置腾讯云图床
>
> 2. 进入[腾讯云](https://console.cloud.tencent.com/)
>
> 3. 找到[云产品]-[对象存储]-[创建存储桶]，之后都是默认下一步
>
>    <img src="https://typora-images-1307361841.cos.ap-beijing.myqcloud.com/img/image-20230305123122789.png" alt="image-20230305123122789" style="zoom:50%;" /> 
>
> 4. 点击[密钥管理]-[访问密钥]-[新建密钥]，如果有了密钥就不要新建了

```bash
{
  "picBed": {
    "uploader": "tcyun", 
    "tcyun": {
      "secretId": "AKIDhIgWOTLyOcdQ2Tkp174pOlvwkKhqIo58",
      "secretKey": "ua2ILn23DD4h6IIArLdidLrBKG9qLcfX",
      "bucket": "typora-images-1307361841",  
      "appId": "1307361841",
      "area": "ap-beijing",
      "path": "img/",
      "customUrl": "",
      "version": "v5"
    }
  },
  "picgoPlugins": {}
}
```
