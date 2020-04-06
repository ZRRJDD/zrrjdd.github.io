# Nginx_Mac安装


Nginx(音“engin x")是一款开源的web和反向代理服务器器。具有高性能、高并发、低内存等特点，另外还有一些特色的Web服务器功能，如负载均衡、缓存、访问和带宽控制。


## 1.安装Nginx，终端下执行

```bash
brew install nginx
```

安装过程中会自己安装依赖：

```shell
$ brew install nginx
Warning: You have Xcode 8 installed without the CLT;
this causes certain builds to fail on OS X El Capitan (10.11).
Please install the CLT via:
  sudo xcode-select --install
==> Installing dependencies for nginx: openssl, pcre
==> Installing nginx dependency: openssl
```

## 2.启动Nginx服务

```shell
brew services start nginx
```

```shell
(base) zrrjdd@zrrjdddembp nginx % brew services start nginx
==> Successfully started `nginx` (label: homebrew.mxcl.nginx)
```

启动成功后，使用浏览器 打开 [http://localhost:8080](http://localhost:8080)

如果打开页面如下，证明安装完成，可以使用了。

![](https://gitee.com/ZRRJDD/ImgRepo/raw/master/1586079994_20200405173758926_795114577.png =800x)

## 3.Nginx文件目录

- 1.nginx安装文件目录 `/usr/local/Cellar/nginx`
- 2.nginx配置文件目录 `/usr/local/etc/nginx`
- 3.config文件目录 `/usr/local/etc/nginx/nginx.conf`
- 4.系统hosts位置 `/private/etc/hosts`

## 4.Nginx 常用命令

```bash
nginx #启动nginx
nginx -s quit #快速停止nginx
nginx -V #查看版本，以及配置文件地址
nginx -v #查看版本
nginx -s reload|reopen|stop|quit  #重新加载配置|重启|快速启动|安全关闭nginx
```

```bash
(base) zrrjdd@zrrjdddembp nginx % nginx -h #帮助
nginx version: nginx/1.17.9
Usage: nginx [-?hvVtTq] [-s signal] [-c filename] [-p prefix] [-g directives]

Options:
  -?,-h         : this help
  -v            : show version and exit
  -V            : show version and configure options then exit
  -t            : test configuration and exit
  -T            : test configuration, dump it and exit
  -q            : suppress non-error messages during configuration testing
  -s signal     : send signal to a master process: stop, quit, reopen, reload
  -p prefix     : set prefix path (default: /usr/local/Cellar/nginx/1.17.9/)
  -c filename   : set configuration file (default: /usr/local/etc/nginx/nginx.conf)
  -g directives : set global directives out of configuration file

```

## 5.卸载Nginx

```bash
brew uninstall nginx
```