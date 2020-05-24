






绑定用户
----------
因为Git是分布式版本控制系统，所以需要填写用户名和邮箱作为一个标识，用户和邮箱为你github注册的账号和邮箱

提示（配置的帐号名和邮箱一定要与GitHub相同，不然会提交失败）
```
git config --global user.name "xxx"     (GitHub相对应的帐号名称)

git config --global user.email "xxx@163.com"  （GitHbu相对应的邮箱帐号）
```



为Github账户设置SSH key
---------
1、 生成ssh key

首先检查是否已生成密钥，如果有3个文件，则密钥已经生成，id_rsa.pub就是公钥
```
cd ~/.ssh
ll
```

如果没有，输入: ssh-keygen -t rsa -C "你的邮箱"，然后一路回车即可
```
ssh-keygen -t rsa -C "你的github注册邮箱"

# 查看生成的文件
ll ~/.ssh/

# 查看公钥内容
cat ~/.ssh/id_rsa.pub
```


2、连接github，打开GitHub 进入setting 侧边栏点击ssh key进入，然后新建ssh key
把id_rsa.pub的内容，粘贴进去



然后就开始你的github之旅吧
----------
```
git clone xxx
git add xxx
git commit -m ""
git push origin master
```

已add后，又想撤回
```
#撤回整个add
git reset HEAD

#撤回某个文件，带不带引号均可
git reset HEAD "xxx文件"
或者
git rm --cached "xxx文件"
```






问题
=========

fatal: pathspec XXXX.... did not match any files
---------
一般 git rm 时报上面的错误

一般是你的文件名是中文引起的，执行以下命令解决
> git config --global core.quotepath false



如果git add 时出上面的错  
git add ./ -u 就可以啦






















