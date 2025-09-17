## 一、Gitee PR拉起Jenkins构建

PR拉起Jenkins构建有多个Jenkins插件可以选择，这里选择Gitee官网插件进行配合，在Jenkins插件里下载Gitee-Jenkins-Plugin（现在官方版本似乎有bug，自己拉取代码做了修改）

具体配置可以参考：https://gitee.com/oschina/Gitee-Jenkins-Plugin#%E6%8F%92%E4%BB%B6%E5%AE%89%E8%A3%85

项目中配置了Gitee 触发构建策略、Gitee WebHook 密码、**将构建状态评论到 Gitee Pull Request 中**等

暂时官方版本不支持PR，需要进行适配。

gitee的webhook设置：

![image-20241209173614622](../../picbed/store/picbed/img/image-20241209173614622.png)

## 二、Jenkins构建时进行代码质量检测

安装sonarqube软件，在Jenkins软件中配置sonarqube，下载sonarqube scanner、quality gate插件，并配置对应内容。后续继续集成checkstyle、pmd、findbugs，配置对应报告地址统一交由sonarqube展示。

## 三、Jenkins构建完成邮件通知

构建完成后，将对应状态回评至gitee的PR中。涉及插件：Email Extension Plugin

配置参考：https://blog.csdn.net/GLYX5717/article/details/119747860

系统邮箱：cqait_notice@163.com/changme_123

配置发送内容与trigger，注意配置trigger时，系统配置页与任务配置页都需要添加trigger

## 四、仓库保护分支配置

由于Gitee-Jenkins-Plugin不支持跨仓库PR拉起构建任务，所以只能在一个仓库的分支中进行。为了保护重要分支不被误操作，需要设置保护分支：release3.0  / master等，不被强推，只能通过PR合入代码。

![image-20241210110747817](../../picbed/store/picbed/img/image-20241210110747817.png)

![image-20241210110924963](../../picbed/store/picbed/img/image-20241210110924963.png)

![image-20241210111041992](../../picbed/store/picbed/img/image-20241210111041992.png)

## 五、代码评审设置

![image-20241210110543701](../../picbed/store/picbed/img/image-20241210110543701.png)

## 六、master分支自动备份

- 安装插件：Environment Injector Plugin，EnvInject API
- 使用Use secret text(s) or file(s)绑定用户名与密码
- shell中使用

```
git config --global credential.helper store
echo "https://${MY_USERNAME}:${MY_PASSWORD}@gitee.com" >> ~/.git-credentials
# 列出所有包含 'master_auto_banch_' 的远程分支
branches=$(git branch -r | grep 'master_auto_banch_' | sort)

# 检查 branches 是否为空
if [ -z "$branches" ]; then
    echo "No branches found matching 'master_auto_banch_'."
else
    # 保留最近5个分支
    recent_branches=$(echo "$branches" | tail -n 4)

    # 删除其他分支
    for branch in $branches; do
        if ! echo "$recent_branches" | grep -q "$branch"; then
            echo "Deleting branch: $branch"
            new_branch=$(echo "$branch" | sed 's/^origin\///')
            git push origin --delete "$new_branch"
        fi
    done
fi
```

- 检出新分支，然后push——**Git Publisher**

