
## 一、镜像准备
如果你是内网环境，需要先将镜像导入到自定义镜像仓库

```bash
docker pull mysql:5.7.44
docker pull prom/mysqld-exporter:v0.15.1

HARBOR=test.harbor.io/public
docker tag mysql:5.7.44 ${HARBOR}/mysql:5.7.44
docker tag prom/mysqld-exporter:v0.15.1 ${HARBOR}/mysqld-exporter:v0.15.1

docker push ${HARBOR}/mysql:5.7.44
docker push ${HARBOR}/mysqld-exporter:v0.15.1
```

## 二、存储选择
修改 values.yaml 中 persistence，根据你的存储类型，填写对应类型的参数

MySQL 在启动初始化时会判断目录是否为空，所以选择 NFS 时一般需要设置 subpath

## 三、使用方法
根据需要修改 values.yaml 中变量的值

```bash
helm template mysql-01 . -n app > aio.yaml
kubectl apply -f aio.yaml
```

或者

```bash
helm install mysql-01 . -n app
```



## 四、mysql 监控
### 1. 指标采集
指标采集由 [mysqld_exporter](https://github.com/prometheus/mysqld_exporter) 完成，在 values.yaml 文件中配置

```yaml
exporter:
  enabled: true
```
即可启动，容器启动完成后，可以通过下面命令验证是否可以正常采集指标

```bash
curl http://{{ .Release.Name }}-metrics.{{ .Release.Namespace }}:9104/metrics
```

### 2. 指标上报
这里通过创建 Prometheus 的 ServiceMonitor 资源对象来实现的，需要先安装 [Prometheus Operator](https://github.com/prometheus-operator/prometheus-operator)，
Prometheus 会根据 ServiceMonitor 抓取对应的指标内容。

可以打开 Prometheus 的管理页面，在 `Service Discovery` 页面中检查是否存在我们添加的 serviceMonitor，比如

```
serviceMonitor/app/sm-mysql-nfs-metrics/0 (1 / 5 active targets)
```

### 3. 指标查看
在 grafana 中新建一个 Dashboard，我这里使用的是 https://grafana.com/grafana/dashboards/7362-mysql-overview/
也可以使用文件夹 grafana 目录的文件内容来创建。

