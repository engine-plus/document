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

#### 第一步， 您需要通过镜像来启动一个实例。

![UTOOLS1566274360147.png](https://github.com/engine-plus/document/blob/master/jpg/4600e56c0e3342cb27d03ffbe9996d0c.png?raw=true)

启动实例使用订阅ami

启动过程需要注意的是： 启动过程中选择 IAM 角色

> 注意的是 : IAM 需要具有 Marketplace相关权限。


当实例启动以后我们可以通过 : `https://{host}` 来访问， 端口 `443`

我们建议您开启 `443` 的外网访问端口， 或者是通过内网代理来访问。

#### 第二步， 注册服务元信息

![UTOOLS1566280011727.png](https://github.com/engine-plus/document/blob/master/jpg/953d5264192dd8cd3add7c7b0ee5ac44.png?raw=true)

用户初次登录时， 会跳转到注册页面， 注册集群元信息

字段 | 解释
--- | ---
UserAdmin | 用户登录管理平台用户名
Password | 用户登录管理平台用户密码
MysqlHost | 用于存储集群元信息的数据库地址
MysqlUser | 数据库认证名称， 需具有建库，建表，添加索引等相关权限
MysqlPassword | 数据库认证密码
Region | 集群所在区
AWS ID | 启动机器的aws 凭证ID, 需要具有**启动实例**， **注销实例**等权限。
AWS KEY | aws security key
SSH Key Name | 集群机器登录公钥， 使用root认证
AWS IAM Profile | 集群机器IAM， IAM 需要具有 **SSM 所有授权**。
Security Group Id | 机器安全组， 建议开放内网所有端口访问， 多个ID间使用 `,`分割

> 说明: 

我们再集群主机上维护了一个Mysql, 用户名密码为`root`, 用户没有自己的meta库可以使用这个， host 地址为内网ip.

![UTOOLS1566280697025.png](https://github.com/engine-plus/document/blob/master/jpg/75a6216a1cae843d9cc0407e788ce90b.png?raw=true)

当用户注册成功再登录后会调到主页面， 用户可以选择`Install Cluster` 来安装集群。


#### 第三步， 安装集群

![](https://github.com/engine-plus/document/blob/master/jpg/d87e8473068e10c836cc912e411353a0.png?raw=true)
 
 当用户选择安装集群时会出现安装表单
 
 字段 | 解释
 --- | ---
 Cluster Name | 集群名称, 数据库中会新建一个以 `{ClusterName}` 命名的库，存放集群信息
 @User/@Password | 集群管理后台`ambari`的登录用户名密码凭证
 SubnetId | 集群安装vpc区
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


