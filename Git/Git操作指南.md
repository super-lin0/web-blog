---
title: Git团队协作规范
date: 2019-07-17 20:01:16
tags: [Git]
---

<center>
本文介绍了在团队协作中 Git 的使用注意事项以及规范流程
<center>
</br>
</center>
标签：#Git
</center>

<!-- more -->

# Git 使用规范

版本：v1.0

时间：2019-02-12

[TOC]

### 一、创建`Issues`

PM 在 gitlab 创建 Issue，分配给开发人员

![](https://raw.githubusercontent.com/super-lin0/pic/master/img/20190213141359.png)

### 二、获取主干最新代码

开发人员领取到任务之后，每次开发新功能之前，都必须要获取主干最新代码。

```
# 获取主干最新代码
$ git checkout master
$ git pull
```

![](https://raw.githubusercontent.com/super-lin0/pic/master/img/20190213172204.png)

### 三、新建分支

开发人员领取到任务之后，每次开发新功能，都应该新建分支，分支命名以`feature`或`fix`开始。

```
# 新建一个开发分支myfeature
$ git checkout -b myfeature
```

![](https://raw.githubusercontent.com/super-lin0/pic/master/img/20190213141558.png)

**`Notes`分支命名规范**

- 功能性分支

  `feature/{$username}/{$version}/{$date}/{$issue_id}_{$description}`

  其中：

  feature 使用单数；

  变量 \$username 代表开发者。username 统一使用姓名首字母小写，比如 oyboy，hk；

  变量 \$version 代表里程碑版本号，格式为 releasex.x.x（x 为数字），比如 release1.0.0；

  变量`$date`代表分支创建日期，格式为`yyyyMMdd`,比如`20190213`

  变量 \$issue_id 代表 Issue ID，例如#64686；

  变量 \$description 代表分支功能描述。应该尽量用简短的词组描述，建议不要使用中文，多个单词用下划线分割，比如 remove_thread。

  完整的例子：

  `feature/hk/release1.0.0/20190213/#64686_course_conf`

- 修复分支

  `bugfix/{$username}/{$date}/{$issue_id}_{$description}`

  其中：

  bugfix 使用单数；

  变量 \$username 代表开发者。username 统一使用姓名首字母小写，比如 oyboy，hk；

  变量`$date`代表分支创建日期，格式为`yyyyMMdd`,比如`20190213`

  变量 \$issue_id 代表 Issue ID 或者 Aone 缺陷 ID，或者 #64686；

  变量 \$description 代表分支功能描述。应该尽量用简短的词组描述，建议不要使用中文，多个单词用下划线分割，比如 remove_thread。

  完整的例子：

  `bugfix/hk/20190213/#64686_login`

### 四、与主干分支同步

在开发过程当中，要经常与主干分支保持同步，在提交`feature`代码之前，也需要先同步主干分支代码。

```
$git pull origin master
```

![](https://raw.githubusercontent.com/super-lin0/pic/master/img/20190213194004.png)

### 五、提交`commit`

```
git add -a
git commit
```

![](https://raw.githubusercontent.com/super-lin0/pic/master/img/20190213141812.png)

### 六、撰写提交信息

提交`commit`时，需要撰写完整的提交信息

```
feat：新功能（feature）
fix：修补bug
docs：文档（documentation）
style： 格式（不影响代码运行的变动）
refactor：重构（即不是新增功能，也不是修改bug的代码变动）
test：增加测试
chore：构建过程或辅助工具的变动
(详见《开发工具和插件使用指南》``Git``插件部分)
```

![](https://raw.githubusercontent.com/super-lin0/pic/master/img/20190213141900.png)

### 七、推送到远程仓库

```
$ git push origin myfeature
```

![](https://raw.githubusercontent.com/super-lin0/pic/master/img/20190213142034.png)

### 八、发出`Merge Request`

将代码推送到远程仓库后，就可以发出`Merge Request`到`master`分支，请求别人进行代码 review,将代码合并进`master`分支了，可在这一步选择接受之后是否删除原分支。

![](https://raw.githubusercontent.com/super-lin0/pic/master/img/20190213142136.png)

![](https://raw.githubusercontent.com/super-lin0/pic/master/img/20190213142223.png)

### 九、合并到主分支

PM 在 gitlab 上查看提交和代码修改情况，确认无误后，确认将开发人员的分支合并到主分支（master），接受的时候可以选择是否删除原开发分支。

![1550039002736](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1550039002736.png)

### 十、关闭`Issue`

开发人员在 gitlab 上添加评论并确认开发完成，并关闭 issue。

**Notes**

关闭 Issue 两种方式

- 直接关闭

  ![close issue - button](https://docs.gitlab.com/ee/user/project/issues/img/button_close_issue.png)

- 通过`Merge Request`关闭

  通过在`Merge Request`中(title 或描述信息中)添加关键字+Issue 编号，可以直接在`MR`被接受之后自动关闭，例如

  ```
  Closes #333, #444, #555 and #666

  Closes #333, #444, and https://gitlab.com/<username>/<projectname>/issues/<xxx>
  ```

  其他下面的关键字也可以达到相同效果:

  1、Close, Closes, Closed, Closing, close, closes, closed, closing

  2、Fix, Fixes, Fixed, Fixing, fix, fixes, fixed, fixing

  3、Resolve, Resolves, Resolved, Resolving, resolve, resolves, resolved, resolving

---

### 参考资料

1、阿里 Git 使用规范 https://yuque.antfin-inc.com/docs/share/044ce229-b5c2-47f7-803a-544ff683cb26

2、阿里`Git commit`规范 https://yuque.antfin-inc.com/tianchiplatform/ar9o6m/ql2i4g

3、Gitlab Issue https://docs.gitlab.com/ee/user/project/issues/index.html

4、`Closing Issues` https://docs.gitlab.com/ee/user/project/issues/closing_issues.html

5、push Git 自动修复 Aone 中录入的缺陷： https://yuque.antfin-inc.com/aone/platform/bl0h8r

6、生成 Gitlab 机器人（钉钉）
​ https://www.jianshu.com/p/ad6dad6f625f
<br>
​ https://open-doc.dingtalk.com/docs/doc.htm?spm=a219a.7629140.0.0.7M7pzQ&treeId=257&articleId=105736&docType=1#s0

<p style="text-align: center;"><span style="font-size:18px;"><strong><span style="color:#ff00;"><span style="color:#ff0000;">友情提示：</span></span>请尊重作者劳动成果，如需转载本博客文章请注明出处！谢谢合作！</strong></span></p>

<p align="center"><strong><span style="font-size:18px;">【作者：吴林&nbsp;&nbsp;</span></strong><a target="_blank" href="https://super-lin0.github.io/"><strong><span style="font-size:18px;">https://super-lin0.github.io/</span></strong></a><strong>】</span></strong></p>
