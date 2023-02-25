# Install Single Node OpenStack on CentOS 8
Controller  and network Node Details :
Hostname = controller.lab.local
IP Address = 192.168.1.30
OS = CentOS 8
DNS = 192.168.1.10
Compute Node Details :
Hostname = compute.lab.local
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
hostnamectl set-hostname controller.lab.local => for controller
hostnamectl set-hostname compute.lab.local => for compute
6. Update /etc/hosts with the new hostname
vim /etc/hosts 
192.168.1.20 compute.lab.local
192.168.1.30 controller.lab.local
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

