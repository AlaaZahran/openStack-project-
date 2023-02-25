# Install Single Node OpenStack on CentOS 8
Controller  and network Node Details :
Hostname = controller.alaa.local
IP Address = 192.168.1.30
OS = CentOS 8
DNS = 192.168.1.10
Compute Node Details :
Hostname = compute.alaa.local
IP Address = 192.168.1.20
OS = CentOS 8
DNS = 192.168.1.10
I. Hosts Preparation
A. Hardware
1. Create VM with 12 GB RAM ,2 cpu ,2 Nics and 25G disk
• Image: centos8
2. Enable virtualization
B. Host Configuration
1. Stop firewall
systemctl disable firewalld 
systemctl stop firewalld
2. Stop Network Manager
systemctl disable NetworkManager
systemctl stop NetworkManager
3. Install network-scripts
dnf install -y network-scripts
4. Enable and start network service
systemctl enable network 
systemctl start network
5. Name the host
hostnamectl set-hostname controller.alaa.local => for controller
hostnamectl set-hostname compute.alaa.local => for compute
6. Update /etc/hosts with the new hostname
vim /etc/hosts 
192.168.1.20 compute.alaa.local
192.168.1.30 controller.alaa.local
7. Disable selinux
setenforce 0
C. Set Passwordless authentication from Controller node to Compute
[root@controller ~]# ssh-keygen
[root@controller ~]# ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.1.20
Now check :
[root@controller ~]# ssh compute 
Last login: Sun jan 3 00:03:44 2023 from controller.lab.local
[root@compute ~]# hostname
compute.lab.local
[root@compute ~]#
II. Software repositories
1. Install the release of openstack package on two server
dnf config-manager --enable powertools
dnf install -y centos-release-openstack-yoga
2. Update the system
dnf -y update
III. install packstack installer
dnf install -y openstack-packstack
IV. Run the packstack to install openstack on controller server 
packstack --gen-answer-file=/root/answers.txt --os-neutron-l2-agent=openvswitch --os-neutron-ml2-mechanism-drivers=openvswitch --os-neutron-ml2-tenant-network-types=vxlan --os-neutron-ml2-type-drivers=vxlan,flat --provision-demo=n --os-neutron-ovs-bridge-mappings=extnet:br-ex --os-neutron-ovs-bridge-interfaces=br-ex:ens192 --keystone-admin-passwd=redhat --os-heat-install=n
[root@controller ~]# vi /root/answer.txt
CONFIG_CONTROLLER_HOST=192.168.1.30
CONFIG_COMPUTE_HOSTS=192.168.1.20
CONFIG_NETWORK_HOSTS=192.168.1.30
CONFIG_PROVISION_DEMO=n
CONFIG_CEILOMETER_INSTALL=n
CONFIG_HORIZON_SSL=y
CONFIG_NTP_SERVERS=<Specify NTP Server IP >
CONFIG_KEYSTONE_ADMIN_PW=<Specify New_Password>
2. packstack --answer-file=/root/answers.txt -t 3600
Validation
1. Validate through Dashboard
![op1](https://user-images.githubusercontent.com/46306526/221382248-1397836f-cf26-4b77-b6b8-d31acb0ae7a8.png)
2. Validate through cli
![op2](https://user-images.githubusercontent.com/46306526/221382321-74c11e5a-7dcc-4a54-8262-af55c2c99b47.png)
Manage openstack using cli
  source keystonerc_admin
  Networking
  1. Create external network for instances so they can communicate with the outside world
  neutron net-create external --provider:network_type flat --provider:physical_network extnet --router:external
  2. Create a subnet attached to the external network
  neutron subnet-create --name external_subnet --enable_dhcp=False --allocation-pool start=192.168.1.50,end=192.168.1.100 --gateway=192.168.1.30 external  192.168.1.0/24
  3. Create private network
  neutron net-create private
  4. Create a subnet attached to the private network
  neutron subnet-create --name private_subnet private 192.168.2.0/24
  5. Create router to connect external network and internal network to each other
  neutron router-create router
  6. Set its gateway using the external network
  neutron router-gateway-set router external
  7. Connect your new private network to the public network through the router
  neutron router-interface-add router private_subnet
  ![op3](https://user-images.githubusercontent.com/46306526/221382843-7defcc73-f674-46e9-b27f-71cbd549416f.png)
  ![op4](https://user-images.githubusercontent.com/46306526/221382873-96825826-14fb-4f67-ad2a-a958d998463d.png)
  Prerequisites for Creation instance
  1. Image
  o Install image file and Create image using glance component
  curl -L http://download.cirros-cloud.net/0.3.4/cirros-0.3.4-x86_64-disk.img | glance image-create --name='cirros image' --visibility=public --container-format=bare --disk-format=qcow2
  2. Flavor
  openstack flavor list
  3. Private Network
  openstack network list
  Launch an instance
  openstack server create -- flavor i  --image b90b2bea-cb3f-a64f-bc63993a01ef --nic net-id=private  ec2-test  
  openstcak server list 
  ![op5](https://user-images.githubusercontent.com/46306526/221383173-e30904f2-d8e2-43f8-be83-9217895511ae.png)
  Floating ip and assign it to an instance
  1. Create floating ip >> it will be created from the range of the external network
  openstack floating ip create external
  from gui asssin that floating ip to instance
  4. Configure security group of the instance
  
![image](https://user-images.githubusercontent.com/46306526/221383415-5ee61064-efd7-475c-9e2a-36581d61d727.png)
  ![op5](https://user-images.githubusercontent.com/46306526/221383279-6271a8e9-804f-4136-80f4-6b7eb1509303.png)
  ![op7](https://user-images.githubusercontent.com/46306526/221383325-3fc2b831-2d8e-49f9-b658-3b5d0e174f5a.png)
  
 
  
