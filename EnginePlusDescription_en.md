# EnginePlus Intruduction

>  **EngiePlus** is a big data cluster management service that helps users to manage and operate clusters such as Hadoop.

## Features
 * `One-click` create cluster.(Hadoop)
 * `Healthy-Status` dection in cluster.
 * `Auto-Scaling` for cluster nodes.
 
  Of course, it is not only that, we have done in-depth optimization for cluster performance and other related parameters,
the performance compared with similar instances is better than **AWS EMR** and **Aliyun EMR** .

With some simple operations, users can start a service to maintain a high performance cluster.

## Quick-Start Guide 
Assumption: You are familiar with AWS.
Prepare: 
 - an instance for EnginePlus Service
 - `SSH key`
 - `IAM role`
 - Security Group Ids
 - VPC subnetId
 
You can generate those in AWS.

> Steps :  
> **Start service instance** --> **Register service meta** --> **Create cluster** --> **Run Application**

![UTOOLS1566274360149.png](https://github.com/engine-plus/document/blob/master/jpg/3137472106854333557l.png?raw=true)

### Step 1: 
Please start an instance by AMI. (The AMI will be provided by subscription in AWS Marketplace).

![UTOOLS1566274360149.png](https://github.com/engine-plus/document/blob/master/jpg/lALPDgQ9q757G5HNA4HNB2o_1898_899.png?raw=true)

And you must chose IAM role for your instance. When the instance is launched,we can access it via:  `https://{host}` , port is `443`.
We recommend that you open the external network access port of 443, or access it through proxy.

### Step 2:
Register Service Meta, when you firstly access the page `https://{host}`, you will jump to the registration page, then you need to fill out the meta form as belows:

![UTOOLS1566280011727.png](https://github.com/engine-plus/document/blob/master/jpg/953d5264192dd8cd3add7c7b0ee5ac44.png?raw=true)

**Form Description**

Field |  Description
--- | ---
UserAdmin | user name when login EnginePlus
Password |  user password when login EnginePlus
MysqlHost | Database address used to store cluster meta information
MysqlUser | Database authentication name, need to have the relevant permissions for building a database, table, index, etc.
MysqlPassword | Database authentication password
Region | the region of cluster instances
AWS ID | To start the machine's aws credential ID, you need to have the relevant permissions for the launch instance, logout instance, SSM, and so on.
AWS KEY | aws security key
SSH Key Name | cluster instances ssh private key file name,like `*.pem`, please use root authentication
AWS IAM Profile | cluster instances IAM, IAM requires all SSM authorizations.
Security Group Id | instance security group, it's recommended to open all ports on the intranet，multi-group-Id splitted by `,`

> **Note:**

EnginePlus AMI contains a local mysql service,you can use local database to fill in the form if you don't have own meta Database.
You can fill in `Ambari MySQL Meta`:
```
  MysqlHost: your intranet ip
  MysqlUser: root
  MysqlPassword: root
```
### Step 3:
When the user registers successfully and then logs in, it will be transferred to the main page. The user can select `Install Cluster` to install the cluster.

![UTOOLS1566280697025.png](https://github.com/engine-plus/document/blob/master/jpg/75a6216a1cae843d9cc0407e788ce90b.png?raw=true)

- When click `Install Cluster`, you need to fill out the installation form:

Field | Description
 --- | ---
 Cluster Name | EnginePlus will create a  database named by`{ClusterName}`, store the cluster meta.
 @User/@Password | login username/password credentials used by Cluster management based on 'ambari' 
 SubnetId | the cluster vpc subnetId 
 Company | Cloud Provider, only support aws currently
 Instance | Instance type
 
 ![UTOOLS1566281259653.png](https://github.com/engine-plus/document/blob/master/jpg/627ed80f16be85aa3bce4986470f9e9a.png?raw=true)
 
- After the user submits, the installation progress can be observed on the left side. Once the installation is completed, the cluster will be displayed in the middle form. The installation takes about 20 minutes.

**Users can install multiple clusters, but must wait for the previous installation before submitting the installation.**
 
  ![UTOOLS1566282018855.png](https://github.com/engine-plus/document/blob/master/jpg/bec7e6926683884e141254aaedd5a698.png?raw=true)

- Once the installation is successful, we can check the details through the `Detail` button.

![UTOOLS1566286699091.png](https://github.com/engine-plus/document/blob/master/jpg/800515ac420a0fae7e11e75b75fd84ad.png?raw=true)

#### Resource Scaling
When the cluster is installed, there is only one NodeManager in Hadoop Cluster. We can set our resource requirements manually or auto-scaling rules according to  requirements.

##### Manual Operation

![UTOOLS1566286883814.png](https://github.com/engine-plus/document/blob/master/jpg/2e6929887c1d5a75f0c1061b1e843121.png?raw=true)

##### Auto-scaling Strategy

![](https://github.com/engine-plus/document/blob/master/jpg/4745150917c78128eec62c11e19390cc.png?raw=true)

### Step 4: Run test Application
When the installation is successful, we can click on the access to `ambari` in the upper right corner. We use Ambari to complete the job of cluster management and installation.

After the installation is completed, the components are:

![UTOOLS1566287275715.png](https://github.com/engine-plus/document/blob/master/jpg/828db6e9982c689a56079df19596771a.png?raw=true)

Now, We can submit the application on the `ambari server` instance:

```bash
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
Of course, the cluster environment files are all located in the '/usr/HDP/' directory, where users can download and put into their own instance's environment if they want to submit their applications in other instances.
### Remove Cluster
> **Note**:
1. For security reasons, the operation of remove cluster only deletes the part of the monitor mounting, and the instance and the meta-information database are still in use. You need to manually release the instances and delete the database corresponding to mysql, if you want to remove completely.
2. If the installation is failed, you need to check whether the database has been added. 
If the database has been automatically added, you need to manually delete it to prevent 
the subsequent installation process.
