## git问题

## 1、提交之前先拷贝一份！或者在编译器打开

## 2、远程有数据，而拉去到本地没有数据的话，本地的分支可能在master上！

查看下本地的分支，然后切换到下自己的分支！



## 3、合并dev分支

1）git commit

2）git  branch //feature/h5_page_config

3）git pull

4）git push

5）git checkout dev

6）git pull//现拉区一下

7）git merge feature/h5_page_config//合并在dev环境上合并

8）git push



## 4、在不小心在dev分支开发了！

1）、git log  复制上一次的提交记录

2）、git reset --hard  id

3）、git push --force



## 5、git的eslint报错，可以不去静态检查

git commit --no-verify -m "修改文件"





## 6、每次新的开发版本或者迭代

要取切换到master分支

然后一定要git  pull拉取下最新代码

然后创建分支git checkout -b   。。。

git  branch





## 7、把本地创建好的分支切换到远程

git push  origin feature/distribution_v3.2:feature/distribution_v3.2





## 8、项目已经上传到正式环境， 要删除本地分支，和远程分支！

git branch -d feature/h5_page_config

git push origin -d feature/h5_page_config







## 9、将本地分支推送到远程，并把代码提交到远程

git push --set-upsteam origin DBfront_polarRedemptionCode



## 10、本地项目于远程分支创建关联

git init 

git add .

git commit -m "feat: 新增..."

git    remote   add   <name>  <url>//name:是自己仓库的名字、url是远端的地址

git push --set-upstream microApp master

ok



## 11、删除远程分支，然后也删除本地分支

git push origin -d DBfront_polarRedemptionCode

git branch -d  DBfront_polarRedemptionCode

```
error: The branch 'DBfront_polarRedemptionCode' is not fully merged.
If you are sure you want to delete it, run 'git branch -D DBfront_polarRedemptionCode'.
```

git branch -D DBfront_polarRedemptionCode

12、配置密钥

git config --global user.email "3081440674@qq.com"

 git config --global user.name "dangbo"

ssh-keygen -t rsa -C '3081440674@qq.com'一路回车，就会生成
