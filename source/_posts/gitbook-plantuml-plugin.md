title: (记录)配置使用gitbook的plantuml插件
date: 2015-11-25 22:31:31
categories: Git
tags: [gitbook, plantuml]
---

终于可以在gitbook里面直接使用plantuml了, 真心方便!

<!--more-->

# 使用步骤

## 准备工作

1. 安装gitbook-plugin-plantuml

    修改package.json文件,增加一个gitbook-plugin-plantuml插件.

    "gitbook-plugin-plantuml":  "~0.0.15"

        "devDependencies": {
            ......
            "gitbook-plugin-plantuml":  "~0.0.15"
        },

    然后执行命令:

    > npm install .

    也可以一次性执行命令:

    > npm install gitbook-plugin-plantuml --save

2. 设置book.json

	在book.json中增加plugins设置:

        {
            "plugins": ["plantuml"],
        }

3. 下载plantuml.jar

	从[官网download页面](http://www.plantuml.com/download.html)下载到plantuml.jar文件, 放到项目根目录下

4. 创建目录/assets/images/uml

## 使用插件

在需要插入planttuml图片的地方, 插入plantuml内容:

    ```uml
    @startuml
    a -> b
    @enduml
    ```

之后执行gitbook server或者grunt test命令, 生成gitbook的html内容时, 这里的plantuml内容就会被转换为uml图片,然后替换对应的plantuml文本内容.

这样我们就可以在文章内容中直接书写plantuml内容, 然后自动得到对应的图片, 非常方便.

# 参考资料

- [gitbook-plantuml在github上的项目](https://github.com/romanlytkin/gitbook-plantuml)
- [lyhcode贡献的代码和项目](https://github.com/lyhcode/gitbook-plugin-plantuml)
- [GitBook + PlantUML 以 Markdown 快速製作 UML 教材](http://blog.lyhdev.com/2014/12/gitbook-plantuml-markdown-uml.html): lyhcode编写的使用文档