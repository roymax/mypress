---
layout: post
title: "Installing GitLab on Ubuntu Server 11.10"
date: 2012-01-20 14:20
comments: true
categories: [guide, lab]
tags: [git, gitlab, ubuntu]

---

打算在团队内推广Git，替换掉当前使用的SVN。所以打算挑选一个类GitHub的管理界面，最终在GitLab和Gitblit中选择了前者。本文为[GitLab](http://gitlabhq.com/)在虚拟机下的安装过程。

###安装Ubuntu 11.10
安装好Ubuntu Server后，我设置源指向163
 
{% codeblock %}
$ sudo cp /etc/apt/sources.list /etc/apt/sources.list.backup
$ sudo echo "" > /etc/apt/sources.list
{% endcodeblock %}

编辑source.list加入

```
deb http://mirrors.163.com/ubuntu/ oneiric main universe restricted multiverse 
deb-src http://mirrors.163.com/ubuntu/ oneiric main universe restricted multiverse 
deb http://mirrors.163.com/ubuntu/ oneiric-security universe main multiverse restricted 
deb-src http://mirrors.163.com/ubuntu/ oneiric-security universe main multiverse restricted 
deb http://mirrors.163.com/ubuntu/ oneiric-updates universe main multiverse restricted 
deb http://mirrors.163.com/ubuntu/ oneiric-proposed universe main multiverse restricted 
deb-src http://mirrors.163.com/ubuntu/ oneiric-proposed universe main multiverse restricted 
deb http://mirrors.163.com/ubuntu/ oneiric-backports universe main multiverse restricted 
deb-src http://mirrors.163.com/ubuntu/ oneiric-backports universe main multiverse restricted 
deb-src http://mirrors.163.com/ubuntu/ oneiric-updates universe main multiverse restricted
```

更新服务器并安装一些必要软件 

```
$ sudo apt-get update
$ sudo apt-get dist-upgrade -y
$ sudo apt-get install git-core openssh-server sendmail curl gcc libxml2-dev libxslt-dev sqlite3 libsqlite3-dev libcurl4-openssl-dev libreadline-dev libc6-dev libssl-dev libmysql++-dev make build-essential zlib1g-dev python-setuptools  
```
如果使用VMWare的虚拟机，最好安装一下VMWare-tools

###安装rvm & ruby
gitlab需要使用ruby 1.9.2-p290，习惯了使用rvm来管理ruby版本，所以把rvm也安装一下

```
$ sudo bash -s stable < <(curl -s https://raw.github.com/wayneeseguin/rvm/master/binscripts/rvm-installer )
$ source /etc/profile.d/rvm.sh
$ type rvm | head -1
rvm is a function


#Rubies installed in system, gemsets separated per user:
$ rvm user gemsets
$ rvmsudo rvm install 1.9.2
$ sudo -H -i   #切换到root
$ rvm --default use 1.9.2
$ exit #退出root
$ rvm reload
$ ruby -v
ruby 1.9.2p290 (2011-07-09 revision 32553) [x86_64-linux]


```

基于速度的原因，把rubygem的源调整成[taobao](http://ruby.taobao.org)的

```
$ gem sources -a http://ruby.taobao.org/
$ gem sources --remove http://rubygems.org/
$ gem sources -l 
*** CURRENT SOURCES ***

http://ruby.taobao.org/
$ 
```

淘宝镜像上第1，2行跟我这里反过来的，按官网写法我无法删除rubygems的源，另外我的source文件`rubygems.org`域名后面是有`/`的，所以也要加入才能删除。

或直接通过手工编辑`vi ~/.gemrc`删除。

由于使用了rvm多用户模式所以部分命令需要加上`rvmsudo` 进行。

###安装rails

```
$ rvmsudo gem update --system 1.8.14 #安装1.8.15会出错
$ rvmsudo gem install rails
```
或安装GitLab指定版本
```
$ rvmsudo gem install rails -v 3.1.1 --no-ri --no-rdoc
```

###安装和设置Gitolite

```
$ sudo adduser \
  --system \
  --shell /bin/sh \
  --gecos 'git version control' \
  --group \
  --disabled-password \
  --home /home/git \
  git

# Add your user to git group
$ sudo usermod -a -G git `eval whoami` 

# Create ssh key
$ ssh-keygen -t rsa

# copy your pub key to git home
sudo cp ~/.ssh/id_rsa.pub /home/git/rails.pub

# clone gitolite
sudo -u git -H git clone git://github.com/gitlabhq/gitolite /home/git/gitolite

# install gitolite
sudo -u git -H /home/git/gitolite/src/gl-system-install

# Setup (Dont forget to set umask as 0007!! 搜索0077修改为0007)
sudo -u git -H sh -c "PATH=/home/git/bin:$PATH; gl-setup ~/rails.pub"

sudo chmod -R g+rwX /home/git/repositories/
sudo chown -R git:git /home/git/repositories/

```

检查gitolite安装是否成功，先重新登录确保已经加入git用户组
```
# clone admin repo to add localhost to known_hosts
# & be sure your user has access to gitolite
git clone git@localhost:gitolite-admin.git /tmp/gitolite-admin 

# if succeed  you can remote it
rm -rf /tmp/gitolite-admin 

```

如果有下面类似提示
```
perl: warning: Setting locale failed.
perl: warning: Please check that your locale settings:
	LANGUAGE = "en_US:",
	LC_ALL = (unset),
	LC_CTYPE = "zh_CN.UTF-8",
	LANG = "en_US.UTF-8"
    are supported and installed on your system.

```

编辑`sudo vi /etc/environment`添加以下内容到最后
```
LC_ALL="en_US.UTF-8"
LANG="en_US.UTF-8"
LANGUAGE="zh:en_US:en"
```
保存后执行`source  /etc/environment`使其生效。


###安装和设置GitLab
这里安装的是GitLab的Stable 2.1版本
#####安装GitLab必须包
```
$ sudo apt-get install python-dev python-pip sendmail redis-server libicu-dev
$ sudo pip install pygments
$ sudo gem install bundler
```
###安装gitlab
按个人习惯将gitlab装在`/usr/local`目录下

```
$ sudo git clone -b stable git://github.com/gitlabhq/gitlabhq.git
$ sudo chown -R `eval whoami`:`eval whoami` /usr/local/gitlabhq
$ cd /usr/local/gitlabhq
$ rvmsudo gem install charlock_holmes -v '0.6.8'
```

修改GemFile的`http://rubygems.org`为`http://ruby.taobao.org/`，然后
```
$ bundle install --without development test 
$ bundle exec rake db:setup RAILS_ENV=production
$ bundle exec rake db:seed_fu RAILS_ENV=production
```
###Start Server
`bundle exec rails s -e production`

###测试安装
打开 `http://<ip>:3000`，使用以下帐号和密码登录

User - `admin@local.host`
Password  - `5iveL!fe`

进入`http://<ip>:3000/admin/users`新增一个用户，完成后退出管理帐号。

使用刚新增的用户登录，并进入`http://<ip>:3000/keys`添加一个ssh key，如果没有则生成一个（可以参考[github设置你的Git](http://help.github.com/mac-set-up-git/)）。

返回首页，新建一个`project`成功后按系统提示操作
```
$ mkdir mylab
$ cd mylab
$ git init
$ touch README
$ git add README
$ git commit -m 'first commit'
$ git remote add origin git@localhost:mylab.git   #将loaclhost换成你的ip
$ git push -u origin master
```
如果push成功，你应该可以通过浏览`http://<ip>:3000/mylab/repository`看到刚刚的第一次提交。


###后续安装
通过安装`Passenger+Nginx`可以使用系统管理更加方便，可以[参考官方文档](https://github.com/gitlabhq/gitlabhq/wiki/V2.0-easy-setup-for-ubuntu)介绍。

Enjoy!

---
###参考资料
1. [如何在Ubuntu Server 11.10上安装GitLab](http://firehare.blog.51cto.com/809276/743509)
2. [Gitolite 构建 Git 服务器](http://www.ossxp.com/doc/git/gitolite.html)
3. [Create git user & install gitolite](https://github.com/gitlabhq/gitlabhq/wiki/Gitolite)
4. [Installing GitLab on Ubuntu Server](http://www.ryanwersal.com/blog/2011/10/18/installing-gitlab-on-ubuntu-server/)
6. [GitLab stable版Ubuntu安装](https://github.com/gitlabhq/gitlabhq/wiki/Ubuntu.stable)