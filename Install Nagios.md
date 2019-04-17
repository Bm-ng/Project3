# Project3
# Cách Cài Đặt Nagios 4 và Giám Sát Máy Chủ trên CentOS 7
## Yêu cầu: 
- Bài lab thực hiện trên 2 máy Centos, một máy đóng vai trò làm server (`10.10.10.16/24`) , một máy làm client ( máy được giám sát - `10.10.10.16/24`).
- Máy Centos server có cài LAMP.
## Thực hiện:
## 1. Cài Đặt Build Dependencies:
- Bởi vì chúng ta đang xây dựng Nagios Core từ nguồn, chúng ta phải cài đặt một vài thư viện phát triển cho phép hoàn thành việc xây dựng.

Trước tiên, cài đặt các gói cần thiết:
```sh
#yum install gcc glibc glibc-common gd gd-devel make net-snmp openssl-devel xinetd unzip
```
## 2. Tạo người dùng và nhóm Nagios:
Tạo một người dùng và nhóm sẽ chạy quá trình Nagios. Tạo nhóm "nagios" và "nagcmd", sau đó thêm người dùng vào nhóm bằng các lệnh sau:
```sh
#useradd nagios
#groupadd nagcmd
#usermod -a -G nagcmd nagios
```
## 3. Cài Đặt Nagios Core:
- Tải bản mới nhất của Nagios ( bản 4.4.3 )
```sh
#cd~
#curl -L -O https://assets.nagios.com/downloads/nagioscore/releases/nagios-4.4.3.tar.gz
```
- Giải nén kho lưu trữ Nagios bằng lệnh này:
```sh
#tar xvf nagios-*.tar.gz
```
- Sau đó thay đổi thư mục được giải nén:
```sh
#cd nagios-*
```
- Cấu hình:
```sh
#./configure --with-command-group=nagcmd 
```
- Chạy chương trình bằng lệnh:
```sh
#make all
```
- Chạy các lệnh này để cài đặt Nagios, các tập lệnh gốc và các tệp cấu hình mẫu:
```sh
# make install
# make install-commandmode
# make install-init
# make install-config
# make install-webconf
```
 - Để phát hành các lệnh bên ngoài thông qua giao diện web cho Nagios, chúng ta phải thêm người dùng máy chủ web, apache, vào nagcmd nhóm :
```sh 
#usermod -G nagcmd apache
```

## 4. Cài đặt Nagios Plugins
- Phiên bản mới nhất là Nagios Plugins 2.1.1. Tải nó về thư mục chính của bạn với curl:
```sh
#cd ~
#curl -L -O http://nagios-plugins.org/download/nagios-plugins-2.1.1.tar.gz
```
- Giải nén Nagios Plugins lưu trữ với lệnh này:
```sh
#tar xvf nagios-plugins-*.tar.gz
```
- Sau đó thay đổi thư mục được giải nén:
```sh
cd nagios-plugins-*
```
- Cấu hình Nagios-plugins:
```sh
#./configure --with-nagios-user=nagios --with-nagios-group=nagios --with-openssl
```
- Khởi chạy bằng lệnh:
```sh
#make
#make install
```

## 5. Cài đặt NRPE:
- Bản phát hành NRPE mới nhất là 2,15. Tải nó về thư mục chính của bạn với curl:
```sh
#cd ~
#curl -L -O http://downloads.sourceforge.net/project/nagios/nrpe-2.x/nrpe-2.15/nrpe-2.15.tar.gz
```
- Giải nén kho lưu trữ NRPE bằng lệnh này:
```sh
#tar xvf nrpe-*.tar.gz
```
- Sau đó thay đổi thư mục được giải nén:
```sh
#cd nrpe-*
```
- Cấu hình NRPE với các lệnh sau:
```sh
#./configure --enable-command-args --with-nagios-user=nagios --with-nagios-group=nagios --with-ssl=/usr/bin/openssl --with-ssl-lib=/usr/lib/x86_64-linux-gnu
```
- Bây giờ hãy thiết lập và cài đặt NRPE và tập lệnh khởi động xinetd của nó bằng các lệnh sau:
```sh
#make all
#make install
#make install-xinetd
#make install-daemon-config
```
- Mở tập lệnh khởi động xinetd trong trình chỉnh sửa:
```sh
#nano /etc/xinetd.d/nrpe
```
- Sửa đổi dòng only_from bằng cách thêm địa chỉ IP riêng của máy chủ Nagios của bạn vào cuối (thay thế địa chỉ IP thực của máy chủ của bạn):
```sh
only_from = 127.0.0.1 10.10.10.16 
```
- Lưu và thoát. Chỉ máy chủ Nagios mới được phép giao tiếp với NRPE.

- Khởi động lại dịch vụ xinetd để khởi động NRPE:
```sh
#service xinetd restart
```
Bây giờ Nagios 4 đã được cài đặt, chúng ta cần cấu hình nó.

## 6. Cấu hình Nagios
**Bây giờ chúng ta hãy thực hiện cấu hình Nagios ban đầu. Bạn chỉ cần thực hiện phần này một lần, trên máy chủ Nagios của bạn.**

## 6.1 Tổ chức cấu hình Nagios
 - Sử dụng Nano để chỉnh sửa tệp:
 ```sh
 #nano /usr/local/nagios/etc/nagios.cfg
 ```
 Bây giờ, hãy tìm cách bỏ ghi chú dòng này bằng cách xóa `#`:
 ```sh
 #cfg_dir=/usr/local/nagios/etc/servers
 ```
 - Lưu và thoát.
 -Bây giờ tạo thư mục lưu trữ tệp cấu hình cho mỗi máy chủ mà bạn sẽ giám sát:
```sh
#mkdir /usr/local/nagios/etc/servers
```

## 6.2 Cấu hình Nagios Contacts
- Sử dụng `nano` để chỉnh sửa tệp:
```sh
#nano /usr/local/nagios/etc/objects/contacts.cfg
```
- Tìm chỉ thị email và thay thế giá trị của nó (phần được đánh dấu) bằng địa chỉ email của riêng bạn:
```sh
email nagios@localhost ; <<***** CHANGE THIS TO YOUR EMAIL ADDRESS ******
```
- Lưu và thóat.

## 6.3 cấu hình lệnh check_nrpe
- Hãy thêm một lệnh mới vào cấu hình Nagios của chúng ta:
```sh
#nano /usr/local/nagios/etc/objects/commands.cfg
```
- Thêm phần sau vào cuối tệp:
```sh
define command{
command_name check_nrpe
command_line $USER1$/check_nrpe -H $HOSTADDRESS$ -c $ARG1$
}
```
- Lưu và thoát. Điều này cho phép bạn sử dụng lệnh check_nrpe trong việc xác định dịch vụ Nagios của bạn.

## 6.4 Cấu hình Apache
Sử dụng htpasswd để tạo người dùng quản trị, được gọi là "nagiosadmin", có thể truy cập vào giao diện web của Nagios:
```sh
sudo htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin
```
Nhập mật khẩu tại dấu nhắc. Nhớ thông tin đăng nhập này, vì bạn sẽ cần nó để truy cập vào giao diện web Nagios.

**Nếu bạn tạo một người dùng không được đặt tên là `"nagiosadmin"`, bạn sẽ cần phải chỉnh sửa `/usr/local/nagios/etc/cgi.cfg` và thay đổi tất cả các tham chiếu `"nagiosadmin"` cho người dùng mà bạn đã tạo.**

- Nagios đã sẵn sàng để bắt đầu. Hãy làm điều đó và khởi động lại Apache:
```sh
#systemctl daemon-reload
#systemctl start nagios.service
#systemctl restart httpd.service
```
Hãy kích hoạt Nagios để khởi động máy chủ, hãy chạy lệnh sau:
```sh
#chkconfig nagios on
```
Tùy chọn: Hạn chế quyền truy cập theo địa chỉ IP
Nếu bạn muốn hạn chế địa chỉ IP có thể truy cập đến giao diện web của Nagios,hãy sửa đổi tập tin cấu hình Apache:
```sh
sudo vi /etc/httpd/conf.d/nagios.conf
```
- Tìm và nhận xét hai dòng sau bằng cách thêm # ký tự ở phía trước:
```sh
Order allow,deny
Allow from all
```
Sau đó, bỏ ghi chú các dòng sau, bằng cách xóa `#` và thêm địa chỉ IP hoặc các dãy (dấu tách cách) mà bạn muốn cho phép trong dòng Allow from:
```sh
# Order deny,allow
# Deny from all
# Allow from 127.0.0.1
```
Vì những dòng này sẽ xuất hiện hai lần trong tệp cấu hình nên bạn sẽ cần phải thực hiện lại các bước này một lần nữa.

Lưu và thoát.

Bây giờ bắt đầu Nagios và khởi động lại Apache để đặt thay đổi có hiệu lực:
```sh
#systemctl restart nagios.service
#systemctl restart httpd.service
```
Nagios hiện đang chạy, vì vậy hãy thử và đăng nhập. `http://nagios_server_public_ip/nagios`

## 7. Giám sát một máy chủ CentOS 7 với NRPE
- Trên máy chủ mà bạn muốn theo dõi, cài đặt kho EPEL:
```sh 
#yum install epel-release
```
Bây giờ cài đặt Nagios Plugins và NRPE:
```sh
#yum install nrpe nagios-plugins-all
```
Bây giờ, hãy cập nhật tệp cấu hình NRPE:
```sh
#nano /etc/nagios/nrpe.cfg
```
- Tìm chỉ thị allowed_hosts và thêm địa chỉ IP riêng của máy chủ Nagios của bạn vào danh sách được phân cách bằng dấu phẩy (thay thế nó thay cho ví dụ được đánh dấu):

`allowed_hosts=127.0.0.1,10.10.10.16`
- Lưu và thoát. Điều này cấu hình NRPE để chấp nhận yêu cầu từ máy chủ Nagios của bạn, thông qua địa chỉ IP riêng của nó.

- Khởi động lại NRPE để thay đổi có hiệu lực:
```sh
#systemctl start nrpe.service
#systemctl enable nrpe.service
```
- Khi bạn đã hoàn tất việc cài đặt và cấu hình NRPE trên máy chủ mà bạn muốn theo dõi, bạn sẽ phải thêm các máy chủ này vào cấu hình máy chủ Nagios của bạn trước khi nó bắt đầu theo dõi chúng.

## 8. Thêm máy chủ lưu trữ vào cấu hình Nagios
- Trên máy chủ Nagios của bạn, tạo một tệp cấu hình mới cho mỗi máy chủ từ xa mà bạn muốn theo dõi trong `/usr/local/nagios/etc/servers/` Thay thế từ được đánh dấu, `"yourhost"`, với tên của máy chủ lưu trữ của bạn:

```sh
#nano /usr/local/nagios/etc/servers/yourhost.cfg
```
- Thêm vào định nghĩa máy chủ sau, thay thế giá trị `host_name` bằng tên máy chủ từ xa của bạn ,giá trị alias với mô tả của máy chủ và giá trị address với địa chỉ IP riêng của máy chủ từ xa:

```sh
define host {
use linux-server
host_name yourhost; alias My first Apache server address 10.10.10.15 max_check_attempts 5
check_period 24x7
notification_interval 30
notification_period 24x7
}
```
- Với tập tin cấu hình ở trên, Nagios sẽ chỉ giám sát nếu máy chủ lưu trữ lên hoặc xuống. Nếu điều này là đủ cho bạn, hãy lưu và thoát rồi khởi động lại Nagios

- Thêm bất kỳ khối dịch vụ nào cho các dịch vụ bạn muốn theo dõi. Lưu ý rằng giá trị của check_command xác định những gì sẽ được theo dõi, bao gồm các giá trị ngưỡng trạng thái. Dưới đây là một số ví dụ mà bạn có thể thêm vào tệp cấu hình của máy chủ lưu trữ của mình:
**PING**
```sh
define service {
use generic-service
host_name yourhost service_description PING
check_command check_ping!100.0,20%!500.0,60%
}
```
**SSH (thông báo_enabled được đặt thành 0 sẽ tắt thông báo cho dịch vụ):**
```sh
define service {
use generic-service
host_name yourhost service_description SSH
check_command check_ssh
notifications_enabled 0
}
```
- Nếu bạn không chắc chắn về việc sử dụng các dịch vụ chung, nó đơn giản là kế thừa các giá trị của một mẫu dịch vụ được gọi là `"generic-service"` được xác định use `generic-service` theo mặc định  

Bây giờ hãy lưu và thoát. Tải lại cấu hình Nagios của bạn để đặt bất kỳ thay đổi nào có hiệu lực:
```sh
#systemctl reload nagios.service
```
- Khi bạn đã hoàn tất việc cấu hình Nagios để giám sát tất cả các máy chủ từ xa của mình, bạn nên thiết lập. Hãy chắc chắn truy cập vào giao diện web **Nagios** của bạn và xem trang **Services** để xem tất cả các máy chủ và dịch vụ được giám sát của bạn.









