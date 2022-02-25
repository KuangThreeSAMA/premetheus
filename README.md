
所有的配置文件都在https://github.com/HalfCoke/prometheusAndGrafana

这里只是对上面安装过程中出现的一些问题提出解决方法

安装过程都是只需要执行对应的setup文件

# prometheus
prometheus部署的时候需要更改--web.listen-address参数

> ```
> sudo vi /etc/rc.d/init.d/prometheus
> ```
>
> 将其中的`--web.listen-address=localhost:9090`改为`--web.listen-address=0.0.0.0:9090`
>
> 也可以用sed替换
>
> ```
> sudo sed -i s/localhost:9090/0.0.0.0:9090/ /etc/prometheus/prometheus.yml 
> ```
>
> 具体原因是因为监听的是本地地址（127.0.0.1）外网访问不了，只能本机内网访问

同时需要更改监听目标，将整个集群都放入进去，具体操作为

>``` 
>sudo vi /etc/prometheus/prometheus.yml
>```
>
> 将其中的targets字段复制添加，并改为想监听的机器ip和端口号
>
> ```
> - targets: ['localhost:9090']
> - targets: ['目标ip:端口号']   #node_exporter默认是9100
>   labels:
>     instance:节点号或机器名称   #注意yml文件里面不要有制表符即tab
> ```

prometheus操作

```
#开启
sudo service prometheus start
#停止
sudo service prometheus stop
#重启
sudo service prometheus restart
#如果配置文件更改，则需要执行以下命令
sudo systemctl daemon-reload
sudo service prometheus restart
```

# node_exporter
node_exporter执行setupNodeExporter会报错是因为文件权限的问题

```
sudo chmod 755 /etc/init.d/grafana
#或者在执行setupNodeExporter先在对应文件夹执行
sudo chmod 755 grafana
```

# grafana
由于有总的监控系统，所以这里只是在验证性安装

grafana出现的问题与上面node_exporter基本相同

```
sudo chmod 755 /etc/init.d/
#或者在执行setupGrafana先在对应文件夹执行
sudo chmod 755 node_exporter
```

还需要更改监听端口，否则不能正常访问

>```
>sudo vi /etc/grafana/conf/custom.ini
>```
>
>将其中`;http_port = 3000`的分号去掉
>
>也可以用sed命令
>
>```
>sudo sed -i s/;http_port = 3000/http_port = 3000/ /etc/grafana/conf/custom.ini
>```

grafana最后需要一个模板来查看信息，这是使用的是`8919`


# 其他辅助命令
查看端口配占用情况

```
sudo netstat -tlnp | grep 端口号
```

查看服务进程状态（如premetheus或者防火墙，需要加入服务才能使用）

```
systemctl status prometheus
systemctl status firewalld
```

关闭防火墙
```
#暂时关闭防火墙
systemctl stop prometheus
#永久关闭防火墙
systemctl disable prometheus
```

# ubuntu解决方法

主要是setup和对应开机自启动的文件有问题

## setup文件

setup中`chkconfig`命令在ubuntu中被更换为`sysv-rc-conf`

sysv-rc-conf没有--add直接删除该行

sysv-rc-conf若无法apt安装

则

```shell
sudo vi /etc/apt/sources.list
#并加入下面这一行
deb http://archive.ubuntu.com/ubuntu/ trusty main universe restricted multiverse
#并执行
sudo apt-get update
```

setup执行之后会出现无service文件的情况去`/etc/init.d`下找到对应的文件输入`./文件名 start`，这时sudo netstat -tlnp能看到已启动，但是该启动的不受systemctl控制，需要将进程kill掉然后通过systemctl重新启动

## 开机自启动文件

为相应的名字文件如node_exporter、prometheus(应为可执行文件)

`. /etc/init.d/functions` 应该为`. /lib/lsb/init-functions `或删除

start函数中`action $"Starting $DESC..." su -s /bin/sh -c`应被删掉 并把后面的引号也删除

若想添加认证则可以

```shell
  if ! [ $(id -u) = 0 ]; then
    echo "The script need to be run as root." >&2
    exit 1
  fi
  nohup $prog_path $options >> $prog_logs 2>&1 & 2> /dev/null
```
