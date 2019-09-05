#### EnginPlus 官方使用说明

**EnginPlus** 是面向haoop等大数据集群的管理服务， 帮助用户管理维护集群

集群管理服务主要的功能是:

1. 提供一键创建集群
2. 提供集群机器健康状态检测
3. 提供集群资源的动态伸缩

当然不止是这样， 我们已经为集群性能等的相关参数做了深入优化， 在相同机器配置上性能优于 **AWS EMR** 与 **阿里云 EMR**.

用户只需要简单操作，便可以启动维护一个高性能集群服务。

> 过程 :  
> **启动实例** --> **注册元信息** --> **创建集群** --> **运行作业**

#### 第一步， 订阅我们的服务。

#### 第二步， 注册服务元信息

![UTOOLS1566280011727.png](https://github.com/engine-plus/document/blob/master/jpg/epregister.png?raw=true)

当您订阅我们的服务以后，您就可以使用它并启动一个 **Engineplus** 服务实例。
上面的相关字段都是必填项。

> 字段解释 

字段 | 解释
--- | ---
InstanceType | 服务实例的主机类型
KeyName | 用于登录服务以及集群所有机器的 ssh key name, 登录用户名 `ec2-user`
SubnetId | 服务实例的网络 subnetId
serverSecurityGroupIds | 服务实例的安全组列表
serverUserName | 服务管理的认证用户名
serverUserPassword | 服务管理的认证用户密码
instanceProfileArn | 服务实例以及集群所有机器实例的 IAM 角色名称
clusterSecurityGroupIds | 集群机器的安全组列表
metaHost | 用于存储集群元信息的meta地址
metaUserName | 用于管理元信息库的用户名称
metaPassword | 用于管理元信息库的用户密码

> 说明: 

![](https://github.com/engine-plus/document/blob/master/jpg/epcluster.jpg)

如上所示， **EnginePlus** server 以cluster为单位管理所有的主机资源

1. 对于 EnginePlus 来说， 实例主要分为两类， **server实例** ， **cluster实例**。
2. 所有的实例都统一使用同一个 ssh key 进行认证。
3. server 跟 cluster 建议使用不同的安全组配置， 因为需要访问 github 以及cluster所有机器， 所以需要开通所有的出站规则。 
4. server 安全组开通 22, 443 的相关入栈访问规则
5. cluster 因为集群需要访问所有的实例通信， 出于安全性需要开通所有端口对内网的入栈访问规则
6. cluster 与 server 实例统一使用相同一个 iam role , 需要对这个iam 开启 `ec2:RunInstances`, `ec2:TerminateInstances`, `ssm:*`, `aws-marketplace-management:*` 等的相关权限。
7. 由于集群需要管理元信息， 建议用户提供第三方的mysql数据库用来维护这些元信息， 您只需要在这里提供`库地址`,`用户名`，`密码`.  提供用户必须具有**建库**，**建表**，**建索引**等的权限。 我们内部也维护有一个开源mysql, 用户如果想直接使用 metaHost=`localhost(必须)`, metaUserName=`root`, metaPassword=`root`
8. 暂时只支持 `us-east-1`, 选择subnetId，与安全组需要考虑到


![UTOOLS1566280697025.png](https://github.com/engine-plus/document/blob/master/jpg/75a6216a1cae843d9cc0407e788ce90b.png?raw=true)

当用户启用成功后便可以访问 `https://{host}` 认证登录服务， 用户可以选择`Install Cluster` 来安装集群。


#### 第三步， 安装集群

![](https://github.com/engine-plus/document/blob/master/jpg/d87e8473068e10c836cc912e411353a0.png?raw=true)
 
 当用户选择安装集群时会出现安装表单
 
 字段 | 解释
 --- | ---
 Cluster Name | 集群名称, 数据库中会新建一个以 `{ClusterName}` 命名的库，存放集群信息
 @User/@Password | 集群管理后台`ambari`的登录用户名密码凭证
 SubnetId | 集群实例所在 subnetId
 Company | 云主机厂商， 暂支持aws
 Instance | 实例机型
 
 
 ![UTOOLS1566281259653.png](https://github.com/engine-plus/document/blob/master/jpg/627ed80f16be85aa3bce4986470f9e9a.png?raw=true)
 
 当用户提交后，左侧可以按到安装进度， 安装完成后会在中间表单中显示集群， 安装大概需要20分钟时间

 用户可以选择安装多个集群， 但是安装前必须等前一个安装完才能提交安装。
 
 ![UTOOLS1566282018855.png](https://github.com/engine-plus/document/blob/master/jpg/bec7e6926683884e141254aaedd5a698.png?raw=true)
 
 安装成功后，我们可以通过 `Detail` 按钮查看详情
 
 ![UTOOLS1566286699091.png](https://github.com/engine-plus/document/blob/master/jpg/800515ac420a0fae7e11e75b75fd84ad.png?raw=true)
 
 集群刚建立时候只有一个NodeManager, 我们可以根据用户需求手动或者通过规则设置我们的资源需求
 
 ##### 手动调整
 
 ![UTOOLS1566286883814.png](https://github.com/engine-plus/document/blob/master/jpg/2e6929887c1d5a75f0c1061b1e843121.png?raw=true)
 
 ##### 策略调整
 
![](https://github.com/engine-plus/document/blob/master/jpg/4745150917c78128eec62c11e19390cc.png?raw=true)

#### 第四部， 测试作业

安装成功后， 我们可以点击右上角访问`ambari`， 我们的集群管理安装都是通过ambari来完成的。

安装完成后的组件为

![UTOOLS1566287275715.png](https://github.com/engine-plus/document/blob/master/jpg/828db6e9982c689a56079df19596771a.png?raw=true)


我们可以在 `ambari server` 主机上提交作业: 


```
#!/usr/bin/env bash

INPUT_PATH="s3a://mob-emr-test/dongtao/data/"
OUTPUT_PATH="s3a://mob-emr-test/dongtao/output/MapReduce"

hadoop fs -rm -r "${OUTPUT_PATH}"

hadoop jar \
       hadoop-study-1.0-selfcontained.jar \
       job.HadoopSaligia \
       ${INPUT_PATH} \
       ${OUTPUT_PATH}
```

当然环境服务都位于 `/usr/hdp/` 目录下， 用户可以下载相关的目录文件，放到自己环境中

---

### 注意 

1. 为了安全起见，用户删除集群操作只是删除了集群的监控挂载部分， 机器与元信息库还在，需要手动删除。
2. 安装失败后需要看下数据库有没有添加相关的库名称， 如果已经添加了数据库需要手动删除，防止影响到后续安装过程。


