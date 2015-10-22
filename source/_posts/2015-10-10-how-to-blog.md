title: HEXO+KOA+ECS搭建博客
date: 2015-10-10 19:40:37
tags: 现实
---
## 搭建
[hexo](https://hexo.io/) 是`Node.js`写的，轻量，简单的博客系统，麻雀虽小，也能吃饱。

主题是用的 [jackman](http://wuchong.me/jacman/)，官方推荐主题，作者博客 [jark's blog](http://wuchong.me/)，是阿里新进的小朋友，不得不感叹后浪一浪接一浪，前浪已经拍成翔。

按照它的步骤搭建即可，想改主题，可以在`./_config.yml`里修改`theme`配置，`hexo`默认是`landscape`.

想添加多说评论，或者微搜索也很方便，在`themes/jackman/_config.yml`里开启设置

```
duoshuo_shortname:  ## e.g. wuchong   your duoshuo short name.

tinysou_search:     ## http://tinysou.com/
  enable: true
  id:   ## e.g. "4ac092ad8d749fdc6293" for your tiny search id
```

想配置RSS，装个插件

```
npm install hexo-generator-feed
```

`_config.yml`里配置一下

```
plugins:
- hexo-generator-feed
```

不能再简单。

博客搭建完了之后需要部署到服务器上，当然你也可以利用`gitpage`，网上有很多教程，部署起来超方便，这里就不在累述。

但是我的目的是要折腾，所以我买了个`ECS`，准备在上面折腾。

## 配置
`ECS`默认的登陆名是`root`，密码需要进到[ECS控制台](https://ecs.console.aliyun.com)里重置密码,没错第一次也是重置，我找了好半天，在 **实例详情 -->基本信息** 里。

然后你就可以`ssh`上去了，但是不要高兴太早，里面空空荡荡，啥都没有，你需要自己各种安装。

我选择的是最低的配置，**CPU: 1核，内存：1024 MB，带宽：1Mbps，系统: Ubuntu** ，因为比较贵，一个月六十多，但你可以选择按流量计费。

### 配置运行环境
然后你就要开始安装各种环境，`Git`，`Node.js`，统统`apt-get install XXX`

`Ubuntu`比较爽的是如果你敲的命令不存在，它会提示你怎么去安装。

### 配置FTP

本节参考： [Ubuntu下搭建FTP服务器](http://dhq.me/ubuntu-install-vsftpd)

然后你需要搭建一个`FTP`服务，因为你要上传博客文件，也就是`hexo`系统里`public`文件夹里的内容，你总不能直接连到服务器上去写，当然你喜欢也可以，不过通常我们是本地写好，然后`update`到`ecs`实例上。

推荐`vsftpd`， `apt-get install vsftpd`安装好后，开始配置

首先你要创建一个文件夹作为FTP的根目录

```
mkdir -p /home/test
```
然后创建一个专门访问FTP的用户，例如叫`test`

```
useradd test -g ftp -d /home/test -s /sbin/nologin
```

设置密码:

```
passwd test
```

修改vsftpd的配置文件“vi /etc/vsftpd.conf”：

```
#禁止匿名访问
anonymous_enable=NO
#接受本地用户
local_enable=YES
#可以上传
write_enable=YES
#启用在chroot_list_file的用户只能访问根目录
chroot_list_enable=YES
chroot_list_file=/etc/vsftpd.chroot_list
```

在/etc/vsftpd.chroot_list添加受访问目录限制的用户：

```
echo "test" >> /etc/vsftpd.chroot_list
```

然后你就可以登陆了，然后用`get`和`put`命令上传下载文件

我在`put`的时候碰到了一个很操蛋的错误：

```
227 Entering Passive Mode 
553 Could not create file.
```
没办法上传，最后发现是Ubuntu的权限问题

登陆用户不允许访问根目录，只能访问子目录，所以

```
mkdir test/blog

chmod 755 blog
chmod 777 blog
```

然后上传文件到 `/blog` 中，ok了

## 部署

部署是很简单的，看[官方文档](https://hexo.io/zh-cn/docs/deployment.html)，提供了好几种部署方式，本博就用的是`FTPSync`。

配置`./_config.yml`

```
plugins:
- hexo-deployer-ftpsync

deploy:
  type: ftpsync
  host: <host>
  user: <user>
  pass: <password>
  remote: [remote]
  port: [port]
  ignore: [ignore]
  connections: [connections]
  verbose: [true|false]
```

注意 `remote` 的配置项，因为是在子目录，所以要写成`./blog`

然后运行

```
hexo g
hexo d
```

就 Ok 了

## 搭建Web服务器
当然了，你只是把静态文件上传到了ftp服务器中，想让它能被访问到，还需要Web服务，一般来说要在`KOA`前面再加一层`nginx`,但是我也懒得配置了,只是单应用，直接启Node。

`app.js`非常简单，加上注释只有51行，不能不说`Node.js`的强大。

主要代码：

```
app.use(logger());
app.use(function *(next){
  var page = this.path.indexOf('blog') > -1 ? this.path : '/blog/' + this.path;
  var path = __dirname + page;

  var fstat = yield stat(path);
  if(fstat.isFile()){
    this.type = extname(path);
    this.body = fs.createReadStream(path);
  }
  else if(fstat.isDirectory()){
    this.type = extname(path + '/index.html');
    this.body = fs.createReadStream(path + '/index.html');
  }

});


function stat(path){
  return function(done){
    fs.stat(path, done);
  }
}
```
然后使用[sreen](http://www.ahlinux.com/start/cmd/705.html)守护进程，一切就都OK了。

## 域名

最后一步，你需要一个域名，本博是在万网买的，然后你需要在[控制台](http://netcn.console.aliyun.com/core/domain/list)，进行维护，解析域名什么的。

这里我不得不吐槽一下，控制台太TM难找了，这个地方，万网首页找了半天，最后在中间的 **续费** 那里点进去了。

最后的最后，经过万恶的备案之后，终于可以访问了。

感谢老妹帮忙设计的icon，棒棒哒~
