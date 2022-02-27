涉及命令：

- git init： 初始化仓库
- git clone：拷贝一份远程仓库，也就是下载一个项目
- git remote:在远程仓库的操作
- git status：查看仓库当前的状态，显示有变更的文件。
- git add： 添加文件到暂存区
- git commit： 将暂存区内容添加到仓库中
- git checkout：切换分支或恢复工作树文件
- git push：上传远程代码并合并
- git pull：下载远程代码并合并
- git fetch：从远程获取代码库
- git rm：删除工作区文件。
- git mv：移动或重命名工作区文件。
- git diff：比较文件的不同，即暂存区和工作区的差异。
- git log：查看历史提交记录
- git blame xxxfile：以列表形式查看指定文件的历史修改记录



## 一、基本流程图

![image-20210110160957814](https://cdn.jsdelivr.net/gh/JarvisTH/picbed/img/20210110161000.png)

说明：

- workspace：工作区

- staging area：暂存区/缓存区

- local repository：或本地仓库

- remote repository：远程仓库

  

  一般来说，现在使用git进行项目管理的一般开发流程是：

- 从主仓库中fork代码到自己的个人仓库
- 在本地新建存放代码目录，然后用git下载项目代码到目录中
- 使用tortoisegit工具配置主仓库路径、个人仓库路径
- 在本地修改代码或者新增文件后，使用git add跟踪新增文件。如果想删除某个文件，则使用git rm。然后使用git commit命令，将变更提交到仓库。
- 本地提交完成后，可以使用git push提交到个人仓库中。如果提示本地head落后，可以先pull代码，然后再进行push。
- 在git pull时，如果存在修改还为提交的代码，可以先git stash将修改暂存，然后git pull拉取代码，然后git stash pop弹出修改。
- 如果git pull后存在冲突文件，需要解决冲突，可以使用tortoisegit工具的git resolve conflict工具，界面形式选择使用哪个修改，并标记冲突解决。