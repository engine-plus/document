# EnginePlus Introduction

>  **EnginePlus** is a big data cluster management service that helps users to manage and operate clusters such as Hadoop.

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
 - `SSH key` user is ec2-user.
 - `IAM role`: `ssm:*`,`aws-marketplace:*`,`rds:CreateDBInstance`.
 - Security Group Ids:one for EnginePlus server instance, another for cluster instances.
 - MySQL RDS.(Currently,we will create MySQL RDS automatically in EnginePlus-Enterprise by Cloudformation,the RDS could be connect by your vpc network).
 - VPC subnetIds(at least 2 availability zones for DBSubnetGroup when create MySQL RDS,but EnginePlus server only need one subnetId).
 
  By the words,we hope you sign in AWS Marketplace using IAM role that includes `ssm:*`,`aws-marketplace:*`,`rds:CreateDBInstance`, if your AWS account for marketplace is not the IAM role you will use in EnginePlus,please allow the permission`rds:CreateDBInstance` for your AWS account.
 
You can generate those in AWS.
> **Notice**: 

![](https://github.com/engine-plus/document/blob/master/jpg/epcluster.jpg)
As shown above, the EnginePlus server manages all host resources in cluster units.
1. For EnginePlus, the examples are mainly divided into two categories, `server instance` and `cluster instance`.
2. All instances use the same ssh key for authentication.
3. Server and cluster are recommended to use different security group configurations. Because you need to access github and all machines of cluster, you need to open all outbound rules.
4. The port of 22, 443 related inbound access should be allowed in server security group.
5. Because the cluster needs to access all instance communication, for security reasons, all ports must be opened to the intranet.
6. Cluster and the server instance use the same IAM role. You need to enable  `ssm:*`,`aws-marketplace:*` for this IAM role.

Please add Service Policy `System Manager`(`ssm:*`) and import managed policy `AWSMarketplaceFullAccess`(`aws-marketplace:*`) like belows:
![](https://github.com/engine-plus/document/blob/master/jpg/engineplus_attach_iam_policy.gif)

 - `ssm:*` (used to send commands to the cluster nodes);
 - `aws-marketplace:*`(used to start,stop,terminate cluster instance,if you subscribe EnginePlus-Enterprise,the policy of `aws-marketplace:MeterUsage` will be used to send metering records).
 
**If the policies are not correct,the Engineplus will stop,and will restart automatically after 30s util you set correct IAM polices.**

**Also,if the metering service of Engineplus-Enterprise has exceptions,the server will switch the restart process utill the metering service turns normal,but the cluster you have created will be alive.**

7. Since the meta-information of cluster needs to be managed, it is recommended that users should provide a third-party mysql database(you can use AWS RDS,the EnginePlus-Enterprise will create the AWS RDS `db.t3.medium` automatically) to maintain these meta-information. You only need to provide the database address, username, and password here. The user must have the permissions of creating `database`, `table`, `index`, etc. But please don't use root credentials.
   
8. Temporarily only supports us-east-1, when you select subnetId, we should take the security group into consideration.
 

## Steps :  
> **AWS CloudFormation** --> **Start Service** --> **Create cluster** --> **Run Application**

![UTOOLS1566274360149.png](https://github.com/engine-plus/document/blob/master/jpg/3137472106854333557l.png?raw=true)

### Step 1: 
Subscribe EnginePlus.

### Step 2:
Use AWS CloudFormation to deploy the service automatically (The AMI will be provided by subscription in AWS Marketplace).
Please fill out the CloudFormation as belows,all of fields is required:

![UTOOLS1566280011727.png](https://github.com/engine-plus/document/blob/master/jpg/epregister.png?raw=true)

When the instance is launched,we can access it via:  `https://{host}` , port is `443`.
We recommend that you open the external network access port of 443, or access it through proxy.

**Form Description**

Field |  Description
--- | ---
InstanceType | EnginePlus Sever instance type.
KeyName |  SSH key, by which you can access instances of EnginePlus and clusters.
SubnetIds | the subnetIds of instances vpc network,please provide subnetIds covered at least 2 AZs.
serverSecurityGroupIds | the list of security group ids in EnginePlus instance.
serverUserName | the authentication user of EnginePlus.
serverUserPassword | the authentication password of EnginePlus.
iamRoleName | the name of IAM role for all instances.
clusterSecurityGroupIds | the list of security group ids in Cluster instances.
metaHost | address of RDS,where stored the cluster meta info, but the field will not appear in the EnginePlus-Enterprise.
metaUserName | meta database authentication user.
metaPassword | meta database authentication password.


### Step 3:
When EnginePlus is deployed successfully, you can access `https://{host}` to sigin in to the management page. Users can choose `Install Cluster` to install new cluster.
![UTOOLS1566280697025.png](https://github.com/engine-plus/document/blob/master/jpg/75a6216a1cae843d9cc0407e788ce90b.png?raw=true)


- When click `Install Cluster`, you need to fill out the installation form:

![](https://github.com/engine-plus/document/blob/master/jpg/d87e8473068e10c836cc912e411353a0.png?raw=true)

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
