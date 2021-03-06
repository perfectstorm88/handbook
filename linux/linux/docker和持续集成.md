# docker和持续集成
@(技术笔记)[微服务]

[TOC]
# gitlab-ci

# 附:环境准备升级到Ubuntu 14.04.4 LTS
不管是Docker官方好还是阿里云，都不支持在Ubuntu 12.04.2上部署docker，需要升级到14.04.4 LTS
```
sudo apt-get update 
sudo apt-get dist-upgrade 
sudo apt-get install update-manager-core 
sudo do-release-upgrade -d 
```
主要参考[阿里云服务器从Ubuntu 12.04升级到Ubuntu 14.04](https://www.mobibrw.com/2016/3789) ,介绍挺详细，特别说明两点： 
1、在configuring openssh-server时，惯性地选择了yes，只能升级完毕后通过手工方式修改ssh，参见[ubuntu 14　root 账户 启用与ssh登录](
http://www.cnblogs.com/yudar/p/4475690.html):
ssh的配置(/etc/ssh/sshd_config)
  PermitRootLogin yes #支持root通过密码登录
然后重启服务restart ssh

2、升级过程比较坑爹的是，自动下载最新的jdk版本，由于众所周知的原来，特别慢，等了个把小时。 
3、更坑爹的是，升级后数据盘没有正常挂载（打开后发现/root目录下没有数据，继续坑爹的是，其它几个环境/root目录都是在系统盘上，被惊吓到了，）
``` 
fdisk -l #查看磁盘 
mount /dev/vda /root  #挂载到/root目录下 
umount  /dev/vda  #卸载某个磁盘
```
另外也之前也参考了：[ubuntu 12.04 安装Docker 实战
](http://www.cnblogs.com/babycool/p/5255252.html)

## 安装常用软件

# 附3：基于node:4镜像的定制
## 更新apt源
```
cd /etc/apt
mv sources.list sources.list.bak
echo "deb http://mirrors.aliyun.com/debian jessie main" >> sources.list
echo "deb http://mirrors.aliyun.com/debian jessie-updates main" >> sources.list
apt-get update
```
## 安装vim
apt-get install vim
## 保存https git的登录用户和密码
[.netrc file so you can push/pull to https git repos without entering your creds all the time](https://gist.github.com/technoweenie/1072829)
```
echo "machine gitlab.lingxi.co">>~/.netrc
echo "login deploy">>~/.netrc
echo "password lingxi123">>~/.netrc
```
## 增加npm 的国内镜像
[淘宝 NPM 镜像](https://npm.taobao.org/)

    npm install -g cnpm --registry=https://registry.npm.taobao.org

## 安装程序需要的其它基础模块
安装测试依赖的install global mocha and instanbul
安装excel解析库依赖的python组件


## 强制修改node版本
使用nvm安装node后，在用docker启动，会自动读取/usr/local/bin/node
需要手工，修改node版本
```
mv /usr/local/bin/node /usr/local/bin/node4.6.2
cp /root/.nvm/versions/node/v6.9.4/bin/node /usr/local/bin/node
```
# 附3：基于centos-opencv镜像的定制

# gitlbab持续集成

官方入门：http://docs.gitlab.com/ce/ci/quick_start/README.html
gitlab的架构图，参见：
[Static Code Analysis with Gitlab-CI](https://cds.cern.ch/record/2210418/files/Datko.pdf)，中有个架构图

[Useful tools for your Node.js projects](https://medium.com/london-nodejs/useful-tools-for-your-node-js-projects-20fd1f7c860a#.axzl2zfct):
- Testing and code coverage:Mocha、CucumberJs、Istanbul\Coveralls(online)
- Code consistency and Quality:
  - Jshint\Code Climate(comlexity,duplicate)\Codacy
- Documentation：Raneto\Notes 不感冒
- Keeping dependencies up-to-date and secure:NPM-dview\David\Libraries.io

## 架构
GitLab CI 是gitlab的一部分，除了gitlab的所有特性之外，它管理工程构建project/builds,并提供一个友好的用户接口
GitLab Runner 一个处理构建的应用。可以独立部署，并且通过api与gitlab CI一起工作
为了运行测试，你至少需要一个gitlab实例和一个gitlab Runner 

GitLab Runner 用go编写
## 概念：
- Runners  
  + 运行你的yaml文件，一个runner是一个隔离的虚拟机
  + 专用runner vs  共享runner(服务于所有工程)
  + 理想的，runner不要安装在gitlab的主机上，我们建议每个runner在一个专门主机上
- Pipeline
  + 一个pipeline 按批处理 stages (batches)执行的一组builds(独立运行的job)。
  + 在同一个stage的builds并发执行(依赖于并发runner的个数)
  + 如果都成功了，pipeline移到下一个stage；只要一个失败，下一个stage就不会执行
  
## 介绍
- GitLab提供了CI服务，仅需要增加一个.gitlab-ci.yml文件到仓储根目录，并配置你的project去调用一个runner。这样每次merge或者push就会自动触发CI pipeline；
- 默认的，pipeline会执行三个stage：Build、test、deploy， 如果stage没有job定义会被直接跳过。
- 如果运行正常（无非零值返回），会有个好看的绿色检查标志关联commit，

简而言之,需要两件事

1. Add .gitlab-ci.yml to the root directory of your repository
1. Configure a Runner

## 步骤
### 创建.gitlab-ci.yml 
.gitlab-ci.yml 可以进行版本控制，分支可以有不同的pipeline和jobs
**特别注意：.gitlab-ci.yml是个YAML文件，要使用空格，不要用tab**

  before_script在所有其它job前执行
  每个job有个唯一名称，都包含script关键字；job被用来创建build，在runner中执行

检查语法：
gitlab域名/ci/lint  
### 配置 a Runner


### 选择Executor
https://docs.gitlab.com/runner/executors/README.html#imnotsure


有多种执行器：
选择docker执行
docker执行器的介绍如下：https://docs.gitlab.com/runner/executors/docker.html




## 通过docker创建runner
 参考：[Run gitlab-runner in a container](https://docs.gitlab.com/runner/install/docker.html) 
1、docker镜像安装
有三种安装方式：
- 使用一个配置逻辑卷作为配置和其它资源
- 使用一个配置容器作为定制数据(需要启动两个容器)
- 使用一个配置逻辑卷+使用Docker来生成其它runner，还需要mount socket文件

我采用第三种方式，这样可以支持多个runner
```
docker run -d --name gitlab-runner --restart always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /srv/gitlab-runner/config:/etc/gitlab-runner \
  gitlab/gitlab-runner:latest
```
-  -d (--detach)在后端原型
-  --name 容器的名称
-  -v 绑定一个逻辑卷

- docker ps 显示容器列表
- docker rm  删除一个容器
- docker stop 停止一个容器
- docker images 显示镜像列表

2、Register the runner
 首先看文档：https://docs.gitlab.com/ce/ci/runners/README.html
 配置一个共享runner
```
docker exec -it gitlab-runner gitlab-runner register

Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com )
https://gitlab.com
Please enter the gitlab-ci token for this runner
xxx
Please enter the gitlab-ci description for this runner
my-runner
INFO[0034] fcf5c619 Registering runner... succeeded
Please enter the executor: shell, docker, docker-ssh, ssh?
docker
Please enter the Docker image (eg. ruby:2.1):
ruby:2.1
INFO[0037] Runner registered successfully. Feel free to start it, but if it's
running already the config should be automatically reloaded!
```

注册后的配置信息，可以通过 /srv/gitlab-runner/config目录下的文件查看

## 手动安装Runner

[Manual installation and configuration ](https://docs.gitlab.com/runner/install/linux-manually.html)

1.先在amazon主机上下载安装包(直接国内下载不了)
sudo wget -O /usr/local/bin/gitlab-ci-multi-runner https://gitlab-ci-multi-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-ci-multi-runner-linux-amd64

2.下载完毕后，通过amazon主机传到阿里云主机
```
sudo chmod +x /usr/local/bin/gitlab-ci-multi-runner #执行权限
sudo useradd --comment 'GitLab Runner' --create-home gitlab-runner --shell /bin/bash #创建用户
sudo gitlab-ci-multi-runner register # 注册Runner，操作步骤与通过docker创建runner一致

# Install and run as service (on Linux):
sudo gitlab-ci-multi-runner install --user=gitlab-runner --working-directory=/home/gitlab-runner
sudo gitlab-ci-multi-runner start
```

手动安排的版本与Docker容器中一样，该有的问题都有，最后也都正常

## 使用docker as a service
如下，**在连接mongo时，host的值是mongo
```
services:
  - mongo:3.3
```

辅助文档：
[Configuration of your builds with .gitlab-ci.yml](http://doc.gitlab.com/ce/ci/yaml/README.html)
[Shared Runners](https://about.gitlab.com/2016/04/05/shared-runners/)
### 什么是service
service 定义了另一个docker image,在build时会链接到这个image。
service image可以运行任何应用，但是大多数场景是运行一个数据库容器，例如mysql。比起每次构建时安装mysql,这是非常方便和快速的，使用一个镜像，启动它作为容器。

构建时如何链接service?
[ Linking containers together](https://docs.docker.com/engine/userguide/networking/default_network/dockerlinks/)

The service container for MySQL will be accessible under the hostname mysql. So, in order to access your database service you have to connect to the host named mysql instead of a socket or localhost.

官方用例：
```
docker run -d --name db training/postgres #creates a new container called db 

# create a new web container and link it with your db container
docker run -d -P --name web --link db:db training/webapp python app.py # 
# --link <name or id>:alias or 
# --link <name or id>
docker inspect -f "{{ .HostConfig.Links }}" web
# ==> [/db:/web/db]
```
## 解决问题
### 不能docker pull私有库
[docker executor not working after updating to gitlab ci 1.8.0](https://gitlab.com/gitlab-org/gitlab-ci-multi-runner/issues/1905)

需要在config.toml中增加
```
 [runners.docker]
 pull_policy = "if-not-present"
```
从最佳实践上讲，never更要合适
### 注意mongo的host，注意GitLab CI Services如何使用
[GitLab CI Services](https://docs.gitlab.com/ce/ci/services/README.html) 如何使用？

### 之前还好好的，怎么pull不到工程了？Couldn't resolve host
```
Running on runner-c2af4ede-project-122-concurrent-0 via e9d6c64bebca...
Fetching changes...
HEAD is now at 5732849 clerical error on npm test script
fatal: unable to access 'http://gitlab-ci-token:xxxxxx@gitlab.lingxi.co/jfjun/jfjun-cw-mongoose.git/': Couldn't resolve host 'gitlab.lingxi.co'
ERROR: Build failed: exit code 1
```

参考：
- [Couldn't resolve host](https://gitlab.com/gitlab-org/gitlab-ci-multi-runner/issues/305)
- [Advanced configuration](https://gitlab.com/gitlab-org/gitlab-ci-multi-runner/blob/master/docs/configuration/advanced-configuration.md)

需要配置
```
  [runners.docker]
    extra_hosts = ['gitlab.lingxi.co:121.40.27.6']
```

然后docker重启一下就可以了
```
service docker restart
```
#### 方法二设置dns!!
[各DNS比较opener 阿里 电信 谷歌(怎么设置DNS)](http://jingyan.baidu.com/article/ae97a646b64aaabbfd461dfe.html)
vim  /etc/default/docker
```
# Use DOCKER_OPTS to modify the daemon startup options.
#DOCKER_OPTS="--dns 8.8.8.8 --dns 8.8.4.4" #google dns
# aliyun dns
DOCKER_OPTS="--dns 223.5.5.5 --dns 223.6.6.6"
```
### 关键配置
- `extra_hosts` 解决Couldn't resolve host的问题
- `concurrent` 并发数

### 测试和代码覆盖率

官方文档 ：[CI/CD pipelines settings](https://docs.gitlab.com/ee/user/project/pipelines/settings.html)Test coverage parsing
网友的总结：[Test and Code Coverage with GitLab Docker Runners](http://quaintous.com/2016/08/12/test-and-code-coverage-with-gitlab-ci/)：java/nodejs的代码覆盖率、[代码风格](http://standardjs.com/),java的正则表达式^Coverage+:\s(\d+(?:\.\d+)?%)

- Enable a shared Runner tagged with docker under Project > Runners
- Specify a regular expression under Project > CI/CD Pipelines > Test coverage parsing to filter coverage from console output
- Create a docker image containing everything needed to build and test your project
- Create .gitlab-ci.yml in your project’s root and define how testing/coverage is to be done


正则表达式：^Statements\s+:\s(\d+(?:\.\d+)?%)
```
Below are examples of regex for existing tools:

- Simplecov (Ruby) - \(\d+.\d+\%\) covered
- pytest-cov (Python) - \d+\%\s*$
- phpunit --coverage-text --colors=never (PHP) - ^\s*Lines:\s*\d+.\d+\%
- gcovr (C/C++) - ^TOTAL.*\s+(\d+\%)$
- tap --coverage-report=text-summary (Node.js) - ^Statements\s*:\s*([^%]+)
```

### gitlab-runner-cache不能重复使用【未解决】

# Jenkins+Docker自动化环境配置
## Jenkins安装
dockerhub中的[jenkins官网镜像](https://store.docker.com/images/d55eda09-d7f0-47b0-8780-3407f2f9142c?tab=description)
准备工作
```
cd /root
mkdir jenkins_home
chmod 777 jenkins_home
```
然后就参考官网镜像的推荐启动方式
```
docker run -p 8080:8080 -p 50000:50000 -v /root/jenkins_home:/var/jenkins_home jenkins
```


通过8080端口访问
进入登录页面后
/var/jenkins_home/secrets/initialAdminPassword 对应host的/root/jenkins_home/secrets/initialAdminPassword

然后把密码拷贝过来
## Jenkins节点配置和部署

## 自动测试

gitlabCi有类似功能

## 自动部署，可以分布式多节点部署
## 执行远程脚本
(在Jenkins中配置执行远程shell命令)[http://blog.sina.com.cn/s/blog_1324aaed20102vx4w.html]
- 需要安装件Jenkins SSH plugin.
- 安装了这个插件后，我们进入Job的"配置管理-构建"发现多了1项


## 用户配置权限

[Jenkins进阶系列之——14配置Jenkins用户和权限](http://blog.csdn.net/wangmuming/article/details/22926025)
## 问题
### Could not resolve host: gitlab.lingxi.co
又碰到：
```
stderr: fatal: unable to access 'http://gitlab.lingxi.co/jfjun/jfjun-cw-mongoose.git/': Could not resolve host: gitlab.lingxi.co
```

docker run 时增加启动参数：--add-host gitlab.lingxi.co:121.40.27.6

可以通过docker inspect 看到对应的参数
```
            "ExtraHosts": [
                "gitlab.lingxi.co:121.40.27.6"
            ],
```


### gitlabCI vs jenkins

http://stackoverflow.com/questions/37429453/gitlab-ci-vs-jenkins
 jenkins that allow you to visualize all kinds of reporting, such as test results, coverage and other static analyzers. 

