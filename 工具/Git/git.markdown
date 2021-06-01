# git

## 本地项目上传到github

### 第一步：git init 

将本地文件变成git可管理的仓库

### 第二步：git add&&git commit 

将本地文件上传到本地库

### 补充：生成ssh命令

 $ ssh-keygen -t rsa -C "XXXX"

然后一路回车。就生成了ssh文件，这个文件不要去改变，一旦改变，本地文件再次上传到github上就会出现上传不上去的问题

### 第四步：将公钥设置到github自己账号上

![image-20200904182955796](C:\Users\wyh\AppData\Roaming\Typora\typora-user-images\image-20200904182955796.png)

### 第五步：创建一个项目（github上）

![image-20200904183028646](C:\Users\wyh\AppData\Roaming\Typora\typora-user-images\image-20200904183028646.png)

创建完成之后，复制一下ssh的连接

### 第六步：将本地库和远端的git库建立联系

git romote add origin XXXXX

### 第七步：建立联系之后，就可以上传你自己的代码了

 git push -u origin master

​    由于新建的远程仓库是空的，所以要加上-u这个参数。然后进去GitHub test2这个仓库刷新下就会有已经上传的文件夹了。

#### 补充：

1.如果在新建远程库的时候，新建了一个readme文件，那么上面的命令会报错，因为你本地没有这个文件。所以我们可以先将内容合并一下

git pull --rebase origin master

再次上传就可以了

2.提交代码的时候出现的问题

warning: LF will be replaced by CRLF in one.txt.

The file will have its original line endings in your working directory.

```shell
1.Git设置
git config --global core.autocrlf false
git config --global core.safecrlf true
```





## git常用命令

### 查看本地库所关联的远程库信息

```shell
git remote -v 
```

Git 要求对本地仓库关联的每个远程主机都必须指定一个主机名（默认为 origin），用于本地仓库识别自己关联的主机，git remote 命令就用于管理本地仓库所关联的主机，一个本地仓库可以关联任意多个主机（即远程仓库）。

### 查看提交的记录

```shell
git log 
```

git rebase用于把一个分支的修改合并到当前分支。

```shell
git bash
```



## git分支

```
ssh-keygen
```

### git fetch 

拉取远程仓库的信息，不是更新代码

### git pull

将远程的更新拉取到本地

## git 远程信息切换

### 从ssh切换至https 

```shell
git remote set-url origin(远程仓库名称) https://email/username/ProjectName.git 
```

