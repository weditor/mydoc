# 撰写文章

所有文章都存放在_posts文件夹，需要新建文章的时候直接在里面新建一个文件即可，可以使markdown或者textile.

### 注意事项
* 文章后缀名决定了文件如何解析，markdown后缀是md，textile后缀是textile.
* 文章严格遵循一下明明格式,否则不会被build。
    ```
    年-月-日-标题.MARKUP
    ```
    其中MARKUP为后缀，即md或者textile。

# 构建文章

文章写完之后，需要运行build命令构建生成静态网页：
```shell
jekyll build
或者 jekyll b
```

# 运行服务

构建文章后，启动服务：
```
jekyll serve
```
可以看到打印server address。默认是http://localhost:4000，打开浏览器直接访问，就能看到结果。


# 关于设置主题

# MathJax支持

