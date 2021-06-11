# git 迁移仓库

假如我们原有的仓库为**[git@codehub.devcloud.huaweicloud.com](mailto:git@codehub.devcloud.huaweicloud.com):project.git**



###### **1. 从原地址克隆一份裸版本库**

```git
$ git clone --bare git@codehub.devcloud.huaweicloud.com:project.git
......
```

###### **2. 在新目录创建git___空___项目

这一步是为了让旧项目有**镜像**

假如新仓库地址为**[git@codehub.devcloud.huaweicloud.com](mailto:git@codehub.devcloud.huaweicloud.com):leaderProject.git**

###### **3. 镜像推送代码到新仓库**

进入旧git目录，推送即可

```
$ cd project （进入 .git 文件目录）
$ git push --mirror git@codehub.devcloud.huaweicloud.com:leaderProject.git
```

