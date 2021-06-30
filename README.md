# skywalking-kubernetes
>该项目可以迅速将skywalking 8.3.0部署进kubernetes(k8s) ，修改自 [evanxuhe/skywalking-kubernetes](https://github.com/evanxuhe/skywalking-kubernetes)

>项目仅包含ui oap模块 

## 描述
部署Skywalking供公司项目使用，在实践过程中积累的一些产物

>部署方式为集群部署，使用了zookeeper作为注册中心，需要自己部署zookeeper与es6



-------------
# 安装使用
**git地址**
[loveowie/skywalking-kubernetes](https://github.com/loveowie/skywalking-kubernetes)
    cd 8.3.0
    kubectl apply -f namespace.yml
    kubectl apply -f oap
    kubectl apply -f ui 
    
需要修改oap/01-config.yml中的application.yml的配置，必须修改的是cluster.zookeeper.hostPort与storage.elasticsearch.clusterNodes，也可以自行修改注册中心的方式与存储方式；

# 成果展示
```
NAME                            READY   STATUS    RESTARTS   AGE
oap-6b56f8bbf5-7fjvb            1/1     Running   0          118m
oap-6b56f8bbf5-7tldw            1/1     Running   0          118m
oap-6b56f8bbf5-qzdx2            1/1     Running   0          118m
ui-deployment-f4799496c-m5xw6   1/1     Running   0          117m
```

# 模块概述
### 模块概述
|component  | descripiton |
|--|--|
| namespace |创建skywalking命名空间|
| oap |collector 收集agent上传的数据并整合|
| ui | RocketUI展示前端数据 |


### 镜像
|image  | version | descripiton |
|--|--|--|
| apache/skywalking-ui|8.3.0|官方ui镜像|
|apache/skywalking-oap-server|8.3.0-es6|官方oap镜像|


# 使用说明

>应用全部挂在在skywalking namepace下，所以大家使用时不要忘记切换namespace 比如加-n skywalking

kubctl基本使用：
查看pods:  kubectl get pods -n skywalking
查看日志:  kubectl logs -f oap-6b56f8bbf5-7fjvb  -n skywalking


## 配置修改
oap，ui使用deployment部署
因而想要修改副本数，内存，磁盘等请修改对应目录下的deployment.yml

## 服务起停
由于k8s deployment会在pod停止后一直重启
因而修改停止的正确做法是 

    kubectl edit deployment oap -n skywalking
    kubectl edit deployment ui-deployment -n skywalking

修改对应的replicates为0，拓展应用将replicates修改为对应副本数即可，想刷新配置也可以用这种办法

## Java客户端接入
下载skywalking8.3.0文件夹：
```
https://archive.apache.org/dist/skywalking/8.3.0/apache-skywalking-apm-8.3.0.tar.gz
```

获取文件夹中的agent文件夹，放入相关项目，并在启动命令中加入

```
-javaagent:/opt/agent/skywalking-agent.jar 
-Dskywalking.agent.service_name=service-name
-Dskywalking.collector.backend_service=opa-host:11800 
```

将skywalking-agent接入服务中，也可直接修改agent/confg/agent.config文件中的相关配置；



##### 问题排查

若接入后，日志并不显示，此时可以将agent日志打开，便于排查：

```
-Dskywalking.logging.level=INFO 
-Dskywalking.logging.file_name=oap.log  
```

