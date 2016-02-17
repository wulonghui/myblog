#WAE Proxy
 WAE以Kubernetes为核心，Kubernetes从Docker网络模型(NAT方式网络模型)中独立出来形成一套网络模型。该网络模型的目标是：每一个Pod都拥有一个扁平化共享网络命名空间的IP，称为PodIP，通过PodIP，Pod就能够跨网络与其它物理机和Pod进行通信。另外Service在Pod之间起到服务代理的作用，对外表现为一个单一访问接口，将请求转发给Pod，Service的网络转发是Kubernetes实现服务编排的关键一环。但是Service无法对外暴露（Service的VIP是内部，外部无法寻址到）。为此WAE需要一层代理（WAE Proxy）对外提供访问入口。


WAE Proxy包含2部分：

* Controller: 管理面，监控Kubernetes API Server，控制管理Proxy。
* Proxy：数据转发面，代理服务器，支持Nginx/Tngine


#构建
1.安装Golang

```
wget http://blob.wae.haplat.net/wae/golang/go1.4.1.linux-amd64.tar.gz
tar zxvf go1.4.1.linux-amd64.tar.gz -C /home
```

2.设置环境变量
```
export GOROOT=/home/go
export GOBIN=/home/gopath/bin
export GOPATH=/home/gopath/src/k8s.io/kubernetes/Godeps/_workspace/:/home/gopath
export PATH=$GOBIN:$GOROOT/bin:$PATH
```

3.下载Kubernetes代码
```
mkdir -p /home/gopath/src/k8s.io/
git clone http://<usename>@rd3stash.ourplat.net/scm/wae/kubernetes.git /home/gopath/src/k8s.io/kubernetes

cd /home/gopath/src/k8s.io/kubernetes
git checkout <version>
```

4.编译WAE Proxy
```
mkdir -p /home/gopath/src/wae/
git clone http://<usename>@rd3stash.ourplat.net/scm/wae/wae-proxy.git /home/gopath/src/wae/wae-proxy

cd /home/gopath/src/wae/wae-proxy
git checkout <version>
go build
cp wae-proxy /usr/bin
```

#运行
1.安装运行Proxy

* Nginx

```
yum install nginx
systemctl start nginx
```

	 WAE Proxy不控制Proxy的持续运行，比如由Systemd进行管理。


2.准备模板文件
```
mdkir -p /etc/wae-proxy
cp /home/gopath/src/wae/wae-proxy/template/ /etc/wae-proxy -r

```

3.配置/etc/wae-proxy/wae-proxy.conf
```
sslDir: /etc/wae-proxy/ssl
templateDir: /etc/wae-proxy/template
nginx:
  configFile: /etc/nginx/nginx.conf
  reloadCommand: "nginx -s reload"
```

3.运行WAE Proxy
```
wae-proxy \
--master=<Kubernetes Master URL> \
--config=/etc/wae-proxy/wae-proxy.conf
```

