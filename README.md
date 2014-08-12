OpenStack-IceHouse
==================
#Hướng dẫn cài đặt OpenStack Icehouse
Đầu tiên là mô hình mạng, chúng ta sẽ cài đặt và cấu hình theo mô hình này:
![Mô hình mạng](https://github.com/trananhkma/image/blob/master/Screenshot_2.png)

Tiếp theo là mô hình cài đặt. Các gói càn cài đặt trên mỗi node sẽ được thể hiện trên hình này:
![Mô hình triển khai OpenStack](https://github.com/trananhkma/image/blob/master/Screenshot_1.png)

Đăng nhập với quyền root để tiến hành cài đặt. Khi đăng nhập lần đầu cần đặt mật khẩu cho root:

	sudo passwd root

Sau khi nhập pass cho root, đăng nhập với lệnh "su"

Để tiện cho việc cấu hình, cài đặt SSH để remote giúp cấu hình dễ dàng và nhanh chóng hơn:

	apt-get install openssh-server

Ở Ubuntu 14.04 mặc định không cho đăng nhập bằng root với SSH, ta phải sửa file /etc/ssh/sshd_config:

	vim /etc/ssh/sshd_config

Tìm và sửa dòng sau:

	PermitRootLogin without-password

Thành:

	PermitRootLogin yes

Sau đó restart SSH:

	service ssh restart

Để sử dụng SSH, dùng lệnh sau cho mỗi node:
	
	ssh root@ip_node

Sau đó hệ thống sẽ yêu cầu bạn nhập password của node mà bạn muốn SSH đến.

##I. Thiết lập cấu hình:
###1. Đổi hostname:
Sau khi dựng 3 máy ảo, bạn cần phải đặt tên (hostname) cho chúng. Vừa để dễ nhìn mà cũng cần thiết cho cấu hình. <br>
Trên mỗi node, sử dụng lệnh sau để đổi hostname:<br>
Ví dụ với Controller node:

	echo controller > /etc/hostname
	hostname -F /etc/hostname

Làm tương tự với hai node còn lại. Thường thì sau khi đổi hostname phải restart máy mới có hiệu lực, nhưng với cách này bạn chỉ cần thoát terminal rồi vào lại là được.

**Cấu hình host name (trên cả 3 node):**

	vim /etc/hosts

Xóa 2 dòng có chữ "localhost" rồi thêm vào như sau:

	10.10.10.81	controller
	
	10.10.10.82	compute
	
	10.10.10.83	network

###2. Cấu hình mạng:
Ở mỗi node, ta sửa file interface để cấu hình ip cho các cổng mạng:

	vim /etc/network/interfaces

**Controller node:**	
	
	10.10.10.81 (vm2 - eth0)
	192.168.1.81 (vm0 - eth1)

Thêm nội dung sau vào file interface:

	auto eth1
	iface eth1 inet static
	address 192.168.1.81
	netmask 255.255.255.0
	gateway 192.168.1.1
	dns-nameservers 8.8.8.8

	auto eth0
	iface eth0 inet static
	address 10.10.10.81
	netmask 255.255.255.0

**Compute node:**
	
	10.10.10.82 (vm2 - eth0)
	192.168.1.82 (vm0 - eth1)
	10.10.20.82 (vm3 - eth2)

Làm tương tự như Controller node:	

	auto eth1
	iface eth1 inet static
	address 192.168.1.82
	netmask 255.255.255.0
	gateway 192.168.1.1
	dns-nameservers 8.8.8.8

	auto eth0
	iface eth0 inet static
	address 10.10.10.82
	netmask 255.255.255.0

	auto eth2
	iface eth2 inet static
	address 10.10.20.82
	netmask 255.255.255.0

**Network node:**

	10.10.10.83 (vm2 - eth0)
	192.168.1.83 (vm0 - eth1)
	10.10.20.83 (vm3 - eth2)

Note cuối cùng:

	auto eth1
	iface eth1 inet static
	address 192.168.1.83
	netmask 255.255.255.0
	gateway 192.168.1.1
	dns-nameservers 8.8.8.8

	auto eth0
	iface eth0 inet static
	address 10.10.10.83
	netmask 255.255.255.0

	auto eth2
	iface eth2 inet static
	address 10.10.20.83
	netmask 255.255.255.0

Sau khi cấu hình xong cần restart mạng:

	/etc/init.d/networking restart

*Để chắc chắc, bạn nên kiểm tra lại xem các cổng đã nhận đúng ip như cấu hình chưa? Kiểm tra với lệnh sau:*

	ip a
	
*Nếu các cổng vẫn chưa nhận đúng IP thì cần restart từng cổng:*

	ifdown eth0 eth1 eth2
	ifup eth0 eth1 eth2

*Nếu vẫn không được bạn phải kiểm tra lại file cấu hình xem đã làm theo đúng như hướng dẫn chưa? Sau đó có thể restart máy cho chắc:*

	init 6

***Note:*** *Trường hợp kiểm tra lại, làm đúng như trên mà các cổng vẫn không nhận đúng IP, thì bạn nên chuẩn bị 1 cái búa loại 10kg và 1 bó hương để cầu nguyện! Chúc may mắn! :)* <br>
Cuối cùng, ta kiểm tra kết nối mạng bằng cách **ping** từ một node đến hai node còn lại. Ví dụ đang ở Controller node thì ping đến Network node và Compute node:
	
	ping compute
	ping network

Khi mạng thông đồng nghĩa với bạn đã hoàn thành cấu hình mạng và chuyển sang bước tiếp theo :)

##II. Cài đặt:
###1. Các gói cần thiết
#####**NTP (cài trên cả 3 node):**
Đây là dịch vụ giúp đồng bộ thời gian trên các máy chủ:

	apt-get install ntp -y

#####**Mysql:**
***Controller node:***

	apt-get install python-mysqldb mysql-server -y

Trong quá trình cài đặt, hệ thống sẽ yêu cầu đặt password cho tài khoản roo (của mysql chứ không phải root hệ thống). Bạn có thể đặt tùy ý, nhưng với bài lap thì nên đặt mọi pass là 1 cho dễ nhớ.<br>
Sau khi cài xong, sửa file cấu hình:
	
	vim  /etc/mysql/my.cnf

Tìm để thẻ [mysqld] và sửa các giá trị (những dòn không có thì thêm vào) như sau:

	[mysqld]
	bind-address = 10.10.10.81
	
	default-storage-engine = innodb
	innodb_file_per_table
	collation-server = utf8_general_ci
	init-connect = 'SET NAMES utf8'
	character-set-server = utf8

Giống như mọi cấu hình khác, để service hoạt động đúng sau khi sửa file cấu hình, ta cần restart service:
	
	service mysql restart

Cấu hình bảo mật cho mysql: (chọn "y" cho mọi câu hỏi)

	mysql_install_db
	mysql_secure_installation

***Compute node và Network node:***

	apt-get install python-mysqldb -y

#####**OpenStack packages:**
Do Ubuntu 14.04 đã tích hợp sẵn gói OpenStack Icehouse nên không cần cài gói này!

#####**RabbitMQ:**
Gói dich vụ này giúp các máy chủ giao tiếp với nhau, chỉ cần cài đặt trên Controller node:
 
	apt-get install rabbitmq-server -y

Đổi pass cho tài khoản guest:

	rabbitmqctl change_password guest 1

***Note:*** *Sau khi cài xong phần này, bạn nên update và tạo snapshot:*

	apt-get update && apt-get dist-upgrade -y

###2. Các thành phần core
####**2.1 KEYSTONE**
Keystone là thành phần để chứng thực, token, catalog và policy service cho tất cả các dịch vụ khác của Openstack.
Nó được triển khai thông qua Identity API của Openstack. Kiểm tra ng dùng và quyền của họ.
Cung cấp 1 danh mục các dịch vụ sẵn có cùng với API của Endpoint.<br>
Một số khái niệm cần biết trước khi tiến hành cài đặt:<br>
- **User:** 
Đại diện số hóa của một người, hệ thống, dịch vụ hoặc người sử dụng dịch vụ OpenStack.
Identity service xác nhận rằng các yêu cầu đến được thực hiện bởi user những người mà có quyền được yêu cầu.
User phải đăng nhập và có thể được gán token để truy cập tài nguyên.
- **Credetials:**
Dữ liệu chỉ được biết đến đại diện cho một người và dùng để chứng minh họ là ai.
Trong  Identity Service có 3 loại xác thực: User name và password, user name and API key, hoặc authentication = token được cung cấp bởi Identity Service.
- **Authentication:**
Hành động xác nhận danh tính User.
Identity service xác nhận một yêu cầu gửi đến bằng cách xác nhận một tập hợp các thông tin được cung cấp bởi user.
Identity service cung cấp 1 mã xác thực mà user sử dụng trong yêu cầu tiếp theo của mình.
- **Token:**
Là 1 chuỗi kí tự dài bất kì. 
Token có một phạm vi trong đó mô tả các tài nguyên nó có thể truy cập.
Token có thể bị thu hồi bất cứ lúc nào và có giá trị trong một thời gian hữu hạn.
Keystone cung cấp cho user 1 token trong token đó ngoài việc xác định user là ai còn xác định user có quyền gì.
- **Service:**
Cung cấp một hoặc nhiều Endpoint, thông qua đó người dùng có thể truy cập tài nguyên và thực hiện các hoạt động.
- **Endpoint:**
Một địa chỉ mạng có thể truy cập, thường được mô tả bởi một URL, từ đó bạn truy cập vào một dịch vụ.
Khi user sử dụng 1 endpoint để truy cập vào 1 dịch vụ nào đó, user dùng 1 token đưa cho endpoitn để nó xác nhận xem có được dùng endpoitn này ko và có những quyền gì.
Endpoint này cầm token đi hỏi Keystone xem cái token này có phù hợp ko và có quyền gì.
(Endpoitn giống như 1 cái cổng để user truy cập vào service ). 
Khi token đã dc xác định phù hợp bởi Keystone thì sau đó user sẽ tiếp tục dc truy cập và sử dụng Service.
- **Role:**
Role bao gồm 1 tập hợp các quyền và đặc quyền hay vai trò của 1 đối tượng nào đó.
- **Tenant:**
Một ngăn chứa, khu vực dùng để nhóm hoặc cô lập tài nguyên.
Tùy thuộc vào các nhà cung cấp dịch vụ mà các tenant này được map cho khách hàng hay 1 tài khoản.<br>
Bắt đầu cài đặt trên Controller node:
	
	apt-get install keystone -y

Sau khi hoàn tất cài đặt, sửa file cấu hình:

	vim /etc/keystone/keystone.conf

Tìm đến thẻ [database] và sửa như sau:

	[database]
	connection = mysql://keystone:1@controller/keystone

Xóa db mặc định sử dụng SQLite:

	rm /var/lib/keystone/keystone.db

Tạo database và phân quyền cho user:

	mysql -u root -p1
	CREATE DATABASE keystone;
	GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY '1';
	GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY '1';
	flush privileges;
	exit

Tạo bảng cho keystone:

	su -s /bin/sh -c "keystone-manage db_sync" keystone

Tiếp tục sửa file cấu hình:

	vim  /etc/keystone/keystone.conf
### 
	[DEFAULT]
	admin_token = 123
	log_dir = /var/log/keystone

Reload:

	service keystone restart

Tự động cập nhật keystone:

	(crontab -l -u keystone 2>&1 | grep -q token_flush) || \
	echo '@hourly /usr/bin/keystone-manage token_flush >/var/log/keystone/keystone-tokenflush.log 2>&1' >> /var/spool/cron/crontabs/keystone

Tạo biến môi trường:

	export OS_SERVICE_TOKEN=123
	export OS_SERVICE_ENDPOINT=http://controller:35357/v2.0

Cần phải có bước này để có thể truy cập vào keystone, nó giống như thẻ tạm thời để vào lần đầu. <br>
Tạo Admin user, role, tenant: 

	keystone user-create --name=admin --pass=1 --email=ADMIN_EMAIL
	keystone role-create --name=admin
	keystone tenant-create --name=admin --description="Admin Tenant"
	keystone user-role-add --user=admin --tenant=admin --role=admin
	keystone user-role-add --user=admin --role=_member_ --tenant=admin

	keystone user-create --name=demo --pass=1 --email=DEMO_EMAIL
	keystone tenant-create --name=demo --description="Demo Tenant"
	keystone user-role-add --user=demo --role=_member_ --tenant=demo
	keystone tenant-create --name=service --description="Service Tenant"

Tạo service xác thực:

	keystone service-create --name=keystone --type=identity --description="OpenStack Identity"
	
Tạo endpoint:

	keystone endpoint-create \
  	--service-id=$(keystone service-list | awk '/ identity / {print $2}') \
  	--publicurl=http://controller:5000/v2.0 \
  	--internalurl=http://controller:5000/v2.0 \
	--adminurl=http://controller:35357/v2.0

Kiểm tra dịch vu:
Sau khi có được endpoint, ta hủy "thẻ tạm thời" đã tạo lúc đầu để vào tạo "thẻ chính thức":

	unset OS_SERVICE_TOKEN OS_SERVICE_ENDPOINT

	keystone --os-username=admin --os-password=1 --os-auth-url=http://controller:35357/v2.0 token-get

	keystone --os-username=admin --os-password=1 \
	--os-tenant-name=admin --os-auth-url=http://controller:35357/v2.0 \
	token-get

Vì "thẻ chính thức" đó chỉ có tác dụng trong một phiên, nghĩa lần sau muốn truy cập ta lại phải "show" cái thẻ đó ra. <br>
Để cho tiện, không phải mất công tạo biến thì ta sẽ tạo sẵn một file chứa biến:

	vim  admin-openrc.sh
	
Nội dung file:

	export OS_USERNAME=admin
	export OS_PASSWORD=1
	export OS_TENANT_NAME=admin
	export OS_AUTH_URL=http://controller:35357/v2.0

Khi muốn sử dụng ra dùng lệnh sau:

	source admin-openrc.sh
	
Cuối cùng, kiểm tra lại xem Keystone đã hoạt động chưa, đồng nghĩa với bạn đã cài đặt đúng hay chưa. Sử dụng các lệnh sau, nếu hệ thống trả về các list yêu cầu thì bạn đã cài đúng.
Nếu không thì bạn phải xem lại từng bước cấu hình! (Hoặc búa + bó hương như đã nói ở trên):
	
	keystone token-get
	keystone user-list
	keystone user-role-list --user admin --tenant admin
	
####**2.2 GLANCE**
Đây là thành phần cài trên Controller node, giúp tạo và quản lý các file image, cần thiết cho việc tạo máy ảo.
Các thành phần: <br>
- Glance API server - nhận các hàm gọi API, tương tự như nova-api, nó chờ các API request sau đó giao tiếp với các thành phần khác (glance-registry và image store) sau đó thực hiện các công việc được yêu cầu: truy vấn, upload, delete image...
- Glance Registry server - lưu và cung cấp các thông tin (metadata) về image (định dạng, ID, dung lượng...) Mặc định sử dụng Sqlite để lưu các metadata. Ngoài ra glance-registry luôn nghe cổng 9191.
- Image Storage - lưu trữ các file image.<br>
Cài đặt các gói cần thiết:

	apt-get install glance python-glanceclient -y

Sửa file cấu hình để Glance có thể kết nối tới cơ sở dữ liệu và sử dụng rabbit:

	vim /etc/glance/glance-api.conf
	
	[database]
	connection = mysql://glance:1@controller/glance

	[DEFAULT]
	rpc_backend = rabbit
	rabbit_host = controller
	rabbit_password = 1

File registry:

	vim /etc/glance/glance-registry.conf

Cũng trong thẻ [database]:

	connection = mysql://glance:1@controller/glance

Tiếp theo, tạo cơ sở dữ liệu cho Glance:

	mysql -u root -p1
	CREATE DATABASE glance;
	GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' IDENTIFIED BY '1';
	GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' IDENTIFIED BY '1';
	flush privileges;
	exit

Tạo bảng cần thiết cho database:

	su -s /bin/sh -c "glance-manage db_sync" glance

Tạo user và add role:

	keystone user-create --name=glance --pass=1 --email=glance@example.com
	keystone user-role-add --user=glance --tenant=service --role=admin

Tiếp tục sửa file cấu hình:
/etc/glance/glance-api.conf    và     /etc/glance/glance-registry.conf

	[keystone_authtoken]
	auth_uri = http://controller:5000
	auth_host = controller
	auth_port = 35357
	auth_protocol = http
	admin_tenant_name = service
	admin_user = glance
	admin_password = 1

	[paste_deploy]
	flavor = keystone

Tạo service và endpoint:

	keystone service-create --name=glance --type=image --description="OpenStack Image Service"
	keystone endpoint-create --service-id=$(keystone service-list | awk '/ image / {print $2}') --publicurl=http://controller:9292 --internalurl=http://controller:9292 --adminurl=http://controller:9292

Cuối cùng là reload dịch vụ:

	service glance-registry restart
	service glance-api restart

Sau khi cài đặt, ta kiểm tra xem đã cài đặt đúng hay chưa, bằng cách tạo file image rồi đẩy lên máy chủ. Tạo thư mục chứa image:

	mkdir /tmp/images
	cd /tmp/images/

Tải một file img có sẵn về:

	wget http://cdn.download.cirros-cloud.net/0.3.2/cirros-0.3.2-x86_64-disk.img

Đổi định dạng và thiết lập cho file đó:

 	glance image-create --name "cirros-0.3.2-x86_64" --disk-format qcow2 \
  	--container-format bare --is-public True --progress < cirros-0.3.2-x86_64-disk.img

Kiểm tra xem đã có chưa:

	glance image-list
	
Nếu có rồi thì nghĩa là bạn đã cài đặt thành công, xóa bỏ thư mục đã tạo lúc nãy:

	rm -r /tmp/images

Còn nếu không thì bạn biết phải làm gì rồi đấy! (Búa + hương)
####**2.3 NOVA**
Đây là thành phần giúp quản lý tài nguyên ảo hóa bao gồm CPU, memory, disks, network interfaces.
Tất cả các tài nguyên được hợp nhất vào trong 1 “bể” – “pool of computing”.
Các thành phần bao gồm: <br>
- Cloud Controller - quản lý và tương tác với tất cả các thành phần của Nova.
- API Server - giống như một Web service đầu cuối của Cloud Controller.
- Compute Controller - cung cấp, quản lý tài nguyên từ các instance. Object Store - cung cấp khả năng lưu trữ, thành phần này đi cùng với Compute Controller.
- Auth Manager - dịch vụ authentication và authorization.
- Volume Controller - lưu trữ theo block-level - giống như Amazon EBS.
- Network Controller - tạo quản lý các kết nối trong virtual network để các server có thể tương tác với nhau và với public network.
- Scheduler - chọn ra compute controller thích hợp nhất để lưu instance.

#####**Trên Controller node:**

	apt-get install nova-api nova-cert nova-conductor nova-consoleauth nova-novncproxy nova-scheduler python-novaclient -y

Quá trình cài đặt sẽ mất một lúc tùy thuộc vào tốc độ mạng.
Sau khi cài đặt hoàn tất, sửa file cấu hình như sau:

	vim /etc/nova/nova.conf

Tìm đến các thẻ tương ứng rồi thêm các nội dung vào, cũng giống như khi cấu hình keystone và glance:

	[database]
	connection = mysql://nova:1@controller/nova

	[DEFAULT]
	rpc_backend = rabbit
	rabbit_host = controller
	rabbit_password = 1

	my_ip = 10.10.10.81
	vncserver_listen = 10.10.10.81
	vncserver_proxyclient_address = 10.10.10.81

Sau đó xóa db mặc định của nova (vì chúng ta dùng mysql chứ không dùng cái này):

	rm /var/lib/nova/nova.sqlite

Tiến hành tạo cơ sở dữ liệu và phân quyền cho user:

	mysql -u root -p1
	CREATE DATABASE nova;
	GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' IDENTIFIED BY '1';
	GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' IDENTIFIED BY '1';
	flush privileges;
	exit

Tạo bảng cần thiết cho database nova:

	su -s /bin/sh -c "nova-manage db sync" nova

Tạo user và add role:

	keystone user-create --name=nova --pass=1 --email=nova@example.com
	keystone user-role-add --user=nova --tenant=service --role=admin

Tiếp tục sửa file cấu hình để móc nova với keystone:

	vim /etc/nova/nova.conf

Sửa nội dung như sau:

	[DEFAULT]
	auth_strategy = keystone

	[keystone_authtoken]
	auth_uri = http://controller:5000
	auth_host = controller
	auth_port = 35357
	auth_protocol = http
	admin_tenant_name = service
	admin_user = nova
	admin_password = 1

Sau đó tạo service và endpoint:

	keystone service-create --name=nova --type=compute \
  	--description="OpenStack Compute"
  	
	keystone endpoint-create \
  	--service-id=$(keystone service-list | awk '/ compute / {print $2}') \
  	--publicurl=http://controller:8774/v2/%\(tenant_id\)s \
  	--internalurl=http://controller:8774/v2/%\(tenant_id\)s \
  	--adminurl=http://controller:8774/v2/%\(tenant_id\)s

Sau khi hoàn tất quá trình cấu hình, cần reload các dịch vụ để có thể chạy được:

	service nova-api restart
	service nova-cert restart
	service nova-consoleauth restart
	service nova-scheduler restart
	service nova-conductor restart
	service nova-novncproxy restart

Cuối cùng là kiểm tra dịch vụ:

	nova-manage service list

Nếu hiện ra các service với các state có hình mặt cười nghĩa là đã cài đặt đúng.
Nếu cài đặt sai, các state này sẽ có hình "XXXX".

#####**Trên Compute node:**

	apt-get install nova-compute-kvm python-guestfs -y

Vì lí do bảo mật, nhân linux không cho phép người dùng bình thường sẽ bị hạn chế các dịch vụ ảo như QEMU và libguestfs.
Để cho phép không bị hạn chế, chạy lệnh sau:
 
	dpkg-statoverride  --update --add root root 0644 /boot/vmlinuz-$(uname -r)

Để sau này các bản nâng cấp kernel cũng không bị hạn chế, ta tạo săn 1 file như sau:

	vim /etc/kernel/postinst.d/statoverride

Với nội dung:

	#!/bin/sh
	version="$1"
	# passing the kernel version is required
	[ -z "${version}" ] && exit 0
	dpkg-statoverride --update --add root root 0644 /boot/vmlinuz-${version}

Cấp quyền thực thi cho file này:

	chmod +x /etc/kernel/postinst.d/statoverride

Sửa file cấu hình:

	vim /etc/nova/nova.conf

Nội dung cần sửa trong các thẻ để móc nova với các dịch vụ:

	[DEFAULT]
	auth_strategy = keystone

	rpc_backend = rabbit
	rabbit_host = controller
	rabbit_password = 1

	my_ip = 10.10.10.82
	vnc_enabled = True
	vncserver_listen = 0.0.0.0
	vncserver_proxyclient_address = 10.10.10.82
	novncproxy_base_url = http://controller:6080/vnc_auto.html

	glance_host = controller

	[database]
	connection = mysql://nova:1@controller/nova

	[keystone_authtoken]
	auth_uri = http://controller:5000
	auth_host = controller
	auth_port = 35357
	auth_protocol = http
	admin_tenant_name = service
	admin_user = nova
	admin_password = 1

Xóa bỏ db mặc định và reload dịch vụ:

	rm /var/lib/nova/nova.sqlite
	service nova-compute restart

Cuối cùng là kiểm tra dịch vụ. Bạn hãy cầu trời khấn phật cho các state sẽ xuất hiện 5 hình mặt cười.
Nếu không quá trình kiểm tra lại sẽ rất khổ sở! Có thể cần dùng đến búa!

	nova-manage service list

Chúc may mắn! :-)

####**2.4 NEUTRON**
Nó cho phép cung cấp kết nối mạng như một dịch vụ cho dịch vụ OpenStack khác như compute.
Phần này cần cài đặt trên cả 3 node.
#####**Trên Controller node:**
Tạo database và cấp quyền cho user:

	mysql -u root -p1
	CREATE DATABASE neutron;
	GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' IDENTIFIED BY '1';
	GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' IDENTIFIED BY '1';
	flush privileges;
	exit

Tạo user, add role, service, endpoint:

	keystone user-create --name neutron --pass 1 --email neutron@example.com
	keystone user-role-add --user neutron --tenant service --role admin
	keystone service-create --name neutron --type network --description "OpenStack Networking"
	keystone endpoint-create \
  	--service-id $(keystone service-list | awk '/ network / {print $2}') \
  	--publicurl http://controller:9696 \
  	--adminurl http://controller:9696 \
  	--internalurl http://controller:9696

Cài đặt các gói cần thiết:

	apt-get install neutron-server neutron-plugin-ml2 -y

Dùng lệnh sau để xem service-tenant-id để ghi vào file config:

	keystone tenant-get service

Sửa file config:

	vim /etc/neutron/neutron.conf

Nội dung cần sửa:

	[database]
	connection = mysql://neutron:1@controller/neutron

	[DEFAULT]
	auth_strategy = keystone

	rpc_backend = neutron.openstack.common.rpc.impl_kombu
	rabbit_host = controller
	rabbit_password = 1

	notify_nova_on_port_status_changes = True
	notify_nova_on_port_data_changes = True
	nova_url = http://controller:8774/v2
	nova_admin_username = nova
	nova_admin_tenant_id = copy_id_đã_xem_ở_bước_trên
	nova_admin_password = 1
	nova_admin_auth_url = http://controller:35357/v2.0

	core_plugin = ml2
	service_plugins = router
	allow_overlapping_ips = True
	verbose = True

	[keystone_authtoken]
	auth_uri = http://controller:5000
	auth_host = controller
	auth_protocol = http
	auth_port = 35357
	admin_tenant_name = service
	admin_user = neutron
	admin_password = 1

Sửa file modul layer 2:

	vim /etc/neutron/plugins/ml2/ml2_conf.ini

Nội dung cần sửa:

	[ml2]
	type_drivers = gre
	tenant_network_types = gre
	mechanism_drivers = openvswitch

	[ml2_type_gre]
	tunnel_id_ranges = 1:1000

	[securitygroup]
	firewall_driver = neutron.agent.linux.iptables_firewall.OVSHybridIptablesFirewallDriver
	enable_security_group = True

Tiếp tục sửa file cấu hình:

	vim /etc/nova/nova.conf 

Thêm các nội dung sau:

	[DEFAULT]
	network_api_class = nova.network.neutronv2.api.API
	neutron_url = http://controller:9696
	neutron_auth_strategy = keystone
	neutron_admin_tenant_name = service
	neutron_admin_username = neutron
	neutron_admin_password = 1
	neutron_admin_auth_url = http://controller:35357/v2.0
	linuxnet_interface_driver = nova.network.linux_net.LinuxOVSInterfaceDriver
	firewall_driver = nova.virt.firewall.NoopFirewallDriver
	security_group_api = neutron

Reload các dịch vụ:

	service nova-api restart
	service nova-scheduler restart
	service nova-conductor restart
	service neutron-server restart

#####**Trên Network node:**
Trước khi cấu hình Network, cần phải enable chức năng kernel networking:

	vim  /etc/sysctl.conf

Sửa các giá trị sau:

	net.ipv4.ip_forward=1
	net.ipv4.conf.all.rp_filter=0
	net.ipv4.conf.default.rp_filter=0

Sau đó cài đặt các gói cần thiết:

	apt-get install neutron-plugin-ml2 neutron-plugin-openvswitch-agent neutron-l3-agent neutron-dhcp-agent -y

Nếu kernel version thấp hơn 3.11 thì phải cài thêm gói: *openvswitch-datapath-dkms*. <br>
Sửa file cấu hình:

	vim /etc/neutron/neutron.conf 

Nội dung quen thuộc:

	[DEFAULT]
	auth_strategy = keystone

	rpc_backend = neutron.openstack.common.rpc.impl_kombu
	rabbit_host = controller
	rabbit_password = 1

	core_plugin = ml2
	service_plugins = router
	allow_overlapping_ips = True

	verbose = True

	[keystone_authtoken]
	auth_uri = http://controller:5000
	auth_host = controller
	auth_protocol = http
	auth_port = 35357
	admin_tenant_name = service
	admin_user = neutron
	admin_password = 1

Sửa tiếp file Layer 3:

	vim /etc/neutron/l3_agent.ini
### 

	[DEFAULT]
	interface_driver = neutron.agent.linux.interface.OVSInterfaceDriver
	use_namespaces = True

	verbose = True

Sửa cả file này nữa:

	vim /etc/neutron/dhcp_agent.ini

#### 

	[DEFAULT]
	interface_driver = neutron.agent.linux.interface.OVSInterfaceDriver
	dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
	use_namespaces = True

	verbose = True

Sửa file metadata để cấu hình thông tin để điểu khiển truy cập từ xa đến các máy ảo.
	
	vim /etc/neutron/metadata_agent.ini

Thêm nội dung sau:

	[DEFAULT]
	auth_url = http://controller:5000/v2.0
	auth_region = regionOne
	admin_tenant_name = service
	admin_user = neutron
	admin_password = 1
	nova_metadata_ip = controller
	metadata_proxy_shared_secret = 123456

	verbose = True


#####**Trở lại Controller node:**

	vim /etc/nova/nova.conf

Thêm nội dung về metadata để tích hợp với bên Compute node:

	[DEFAULT]
	service_neutron_metadata_proxy = true
	neutron_metadata_proxy_shared_secret = 123456

Reload:

	service nova-api restart

#####**Tiếp tục trên Network node:**
Sửa các file cấu hình:

	vim /etc/neutron/plugins/ml2/ml2_conf.ini

##### 

	[ml2]
	type_drivers = gre
	tenant_network_types = gre
	mechanism_drivers = openvswitch

	[ml2_type_gre]
	tunnel_id_ranges = 1:1000

	[ovs]
	local_ip = 10.10.20.83
	tunnel_type = gre
	enable_tunneling = True

	[securitygroup]
	firewall_driver = neutron.agent.linux.iptables_firewall.OVSHybridIptablesFirewallDriver
	enable_security_group = True

Reload dịch vụ:

	service openvswitch-switch restart

Tạo mạng vành đai:

	ovs-vsctl add-br br-int
	ovs-vsctl add-br br-ex
	
Add mạng ngoài vào cổng eth1:

	ovs-vsctl add-port br-ex eth1

Reload các dịch vụ:

	service neutron-plugin-openvswitch-agent restart
	service neutron-l3-agent restart
	service neutron-dhcp-agent restart
	service neutron-metadata-agent restart

#####*Trên Compute node:
Enable dịch vụ giống như ở Network node:

	vim /etc/sysctl.conf

##### 

	net.ipv4.conf.all.rp_filter=0
	net.ipv4.conf.default.rp_filter=0

Cài đặt các gói cần thiết:

	apt-get install neutron-common neutron-plugin-ml2 neutron-plugin-openvswitch-agent

Nếu kernel version thấp hơn 3.11 thì cài thêm gói: *openvswitch-datapath-dkms*

	vim /etc/neutron/neutron.conf

Nội dung quen thuộc:

	[DEFAULT]
	auth_strategy = keystone

	rpc_backend = neutron.openstack.common.rpc.impl_kombu
	rabbit_host = controller
	rabbit_password = 1

	core_plugin = ml2
	service_plugins = router
	allow_overlapping_ips = True

	verbose = True

	[keystone_authtoken]
	auth_uri = http://controller:5000
	auth_host = controller
	auth_protocol = http
	auth_port = 35357
	admin_tenant_name = service
	admin_user = neutron
	admin_password = 1

Sửa tiếp file module layer 2:

	vim /etc/neutron/plugins/ml2/ml2_conf.ini
	
##### 

	[ml2]
	type_drivers = gre
	tenant_network_types = gre
	mechanism_drivers = openvswitch

	[ml2_type_gre]
	tunnel_id_ranges = 1:1000

	[ovs]
	local_ip = 10.10.20.82
	tunnel_type = gre
	enable_tunneling = True

	[securitygroup]
	firewall_driver = neutron.agent.linux.iptables_firewall.OVSHybridIptablesFirewallDriver
	enable_security_group = True

	service openvswitch-switch restart
	ovs-vsctl add-br br-int

Tiếp tục sửa file cấu hình:

	vim /etc/nova/nova.conf

##### 

	[DEFAULT]
	network_api_class = nova.network.neutronv2.api.API
	neutron_url = http://controller:9696
	neutron_auth_strategy = keystone
	neutron_admin_tenant_name = service
	neutron_admin_username = neutron
	neutron_admin_password = 1
	neutron_admin_auth_url = http://controller:35357/v2.0
	linuxnet_interface_driver = nova.network.linux_net.LinuxOVSInterfaceDriver
	firewall_driver = nova.virt.firewall.NoopFirewallDriver
	security_group_api = neutron

Reload các dịch vụ:

	service nova-compute restart
	service neutron-plugin-openvswitch-agent restart

####**2.5 HORIZON**
Dashboard cung cấp một giao diện web nhằm tương tác quản lý các thành phần còn lại của Openstack( tạo máy ảo, đặt ip, điều khiển kết nối...), nó kết hợp với Keystone để chứng thực user.
	
	apt-get install apache2 memcached libapache2-mod-wsgi openstack-dashboard -y
	apt-get remove --purge openstack-dashboard-ubuntu-theme







