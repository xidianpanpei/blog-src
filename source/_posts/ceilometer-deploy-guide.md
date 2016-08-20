title: OpenStack Havana Ceilometer安装指导
date: 2014-03-08 15:44:07
categories: [OpenStack]
tags: [Ceilometer,OpenStack]
---
![](/img/2014/03/08/openstack.jpg)

## 题记
由于我们的系统在很早之前就部署完成了。当时Ceilomter刚刚在Havana版本中发布，我们主要专注于网络和虚拟化部分，所以当时就没有在我们的Havana版本环境中安装Ceilomter，但是现在由于需要，开始研究OpenStack的监控功能， 所以就需要在我们现有的系统上补上Ceilomter了。利用了网上的各种教程，但是发现都有些问题，最终不得不依靠官方的Havana部署教程，利用其Ceilometer安装那一节的教程，完成了Ceilometer的完美部署。

<!--more-->

## 教程
我们的系统是依据OpenStack Havana版本部署的，整个系统采用网络集中式部署，也就是说我们的系统分为三个部分：控制节点，网络节点和计算节点。具体的系统相关信息可以参照该篇[教程][4]。

### 所有节点
我们需要在所有的节点安装以下的安装包，这个是Ceilometer的实现基础：

	sudo apt-get install python-ceilometer  ceilometer-common

### 控制节点
1.在控制节点安装以下的包：

	sudo apt-get install ceilometer-api ceilometer-collector ceilometer-agent-central python-ceilometerclient

2.如此之后，控制节点上就已经完成了Ceilometer大部分安装了，接下来就是安装mogodb数据库，这个是Ceilometer默认的数据存储的仓库：

	sudo apt-get install mongodb

3.接着我们开始配置mongodb监听所有的网络接口请求：

	sudo sed -i 's/127.0.0.1/0.0.0.0/g' /etc/mongodb.conf

4.重启mongodb服务，让配置生效：

	service mongodb restart

5.创建数据库ceilometer和对应的用户：

首先进入mongodb：

	mongo

接着执行创建用户命令：

	>use ceilometer	
	>db.addUser( { user: "ceilometer", pwd: "CEILOMETER_DBPASS", roles: [ "readWrite", "dbAdmin" ]} )

6.编辑/etc/ceilometer/ceilometer.conf文件，配置数据库参数：
```
[database]
# The SQLAlchemy connection string used to connect to the
# database (string value)
connection = mongodb://ceilometer:CEILOMETER_DBPASS@controller:27017/ceilometer
```

7.利用openssl生成一个随机token密钥，该密钥用于Ceilometer各个组件之间通信加密使用：

	openssl rand -hex 10	
	cefafd2288d0e4e43005 （注：这是命令生成的随机token）

编辑/etc/ceilometer/ceilometer.conf文件，修改中间的 `[publisher_rpc]` 选项，配置`token`：
```
[publisher_rpc]
# Secret value for signing metering messages (string value)
metering_secret = cefafd2288d0e4e43005
...
```

8.编辑/etc/ceilometer/ceilometer.conf，修改`RabbitMQ`配置选项：
```
rabbit_host = controller_ip_address（自行修改）
```

9.编辑/etc/ceilometer/ceilometer.conf，修改日志打印目录：
```
[DEFAULT]
log_dir = /var/log/ceilometer
```

10.Keystone相关信息创建：

	keystone user-create --name=ceilometer --pass=CEILOMETER_PASS --email=ceilometer@example.com	
	keystone user-role-add --user=ceilometer --tenant=service --role=admin		
	keystone service-create --name=ceilometer --type=metering --description="Ceilometer Telemetry Service"		
	keystone endpoint-create --service-id=the_service_id_above --publicurl=http://controller_ip_address:8777 --internalurl=http://controller_ip_address:8777 --adminurl=http://controller_ip_address:8777

11.编辑/etc/ceilometer/ceilometer.conf，修改相关配置：
```
[keystone_authtoken]
auth_host = controller_ip_address
auth_port = 35357
auth_protocol = http
auth_uri = http://controller_ip_address:5000
admin_tenant_name = service
admin_user = ceilometer
admin_password = CEILOMETER_PASS

[service_credentials]
os_username = ceilometer
os_tenant_name = service
os_password = CEILOMETER_PASS
```

12.重启Ceilometer相关服务，使其生效：

	service ceilometer-agent-central restart	
	service ceilometer-api restart	
	service ceilometer-collector restart

13.编辑/etc/glance/glance-api.conf，修改`Glance`配置：
```
notifier_strategy = rabbit
rabbit_host = controller
```

重启相关服务：

	service glance-registry restart		
	service glance-api restart

14.编辑/etc/cinder/cinder.conf，修改`Cinder`配置：
```
control_exchange = cinder
notification_driver = cinder.openstack.common.notifier.rpc_notifier
```

重启相关服务：

	service cinder-volume restart	
	service cinder-api restart

注：由于我们的环境中没有安装Swift，所以有关Swift配置的部分就省略了。需要的话，请查看本教程中的参考资料。

### 计算节点
1.安装计算节点所需服务：

	sudo apt-get install ceilometer-agent-compute

2.编辑 /etc/nova/nova.conf文件，配置Nova相关选项：
```
[DEFAULT]
...
instance_usage_audit = True
instance_usage_audit_period = hour
notify_on_state_change = vm_and_task_state
notification_driver = nova.openstack.common.notifier.rpc_notifier
notification_driver = ceilometer.compute.nova_notifier
```

3.编辑/etc/ceilometer/ceilometer.conf，配置计算节点上Ceilometer的选项：
```
[publisher_rpc]
# Secret value for signing metering messages (string value)
metering_secret = cefafd2288d0e4e43005

[DEFAULT]
rabbit_host = controller

[keystone_authtoken]
auth_host = controller_ip_address
auth_port = 35357
auth_protocol = http
admin_tenant_name = service
admin_user = ceilometer
admin_password = CEILOMETER_PASS

[service_credentials]
os_auth_url = http://controller_ip_address:5000/v2.0
os_username = ceilometer
os_tenant_name = service
os_password = CEILOMETER_PASS

[DEFAULT]
log_dir = /var/log/ceilometer
```

4.重启服务使得配置生效：

	service ceilometer-agent-compute restart

## 参考资料
1.[Install the Telemetry module - OpenStack Installation Guide for Ubuntu 12.04 (LTS)  - havana][1]		
2.[Ubuntu 12.04 Server OpenStack Havana多节点(OVS+GRE)安装][2]		
3.[部署Ceilometer到已有环境中][3]


[1]:http://docs.openstack.org/havana/install-guide/install/apt/content/ceilometer-install.html
[2]:http://www.cnblogs.com/awy-blog/p/3447176.html
[3]:http://yansu.org/2013/10/01/deploy-ceilometer-of-openstack.html
[4]:https://github.com/xidianpanpei/OpenStack-Havana-Install-Guide-CN-OVS_MutliNode