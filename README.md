
所有的配置文件都在https://github.com/HalfCoke/prometheusAndGrafana

这里只是对上面安装过程中出现的一些问题提出解决方法

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

