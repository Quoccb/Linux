# Cài đặt và cấu hình Cobbler trên CentOS 7
## 1. Tổng quan về Cobbler
### 1.1. Cobbler là gì?
Cobbler là một máy chủ cung cấp Linux có mã nguồn mở và miễn phí, nó được sử dụng để cài đặt hệ điều hành cho các hệ thống qua mạng một cách tự động và không cần giám sát và có thể cài đặt được cho nhiều hệ thống cùng lúc. Cobbler sử dụng các service như **DHCP**, **TFTP** và **DNS** và dựa vào **Kickstart file** để tiến hành cài đặt tự động cho client hoặc system.
### 1.2. Các thành phần trong Cobbler
* Trong cobbler có các thành phần chính:
  * **Kickstart file**: Là file quy định và định nghĩa nên các bước cho việc cài đặt các distro linux. Do có file này mà toàn bộ quá trình cài đặt sẽ được tự động hóa hoàn toàn (chọn ngôn ngữ, timezone, thiết lập cấu hình mạng, tạo mật khẩu cho root user, ...)
  * **TFTP, FTP**:  Là các giao thức mà cobbler sử dụng để truyển tải các file cài đặt từ cobbler server đến các client để cài linux (hiểu đơn giản là sử dụng giao thức truyền file trong linux để đẩy các bản cài đặt xuống client).
  * **DHCP server**: Đáp ứng cho việc cài đặt qua môi trường mạng client phải kết nối được đến server và được cấp 1 địa chỉ IP. Quá trình cấp địa chỉ này được thực hiện bởi DHCP server trải qua các bước cấp DHCP thông thường.
  * **DNS server**: Giúp thể gán địa chỉ IP với 1 tên miền (là thành phần không bắt buộc).
  * **Web server**: Cobbler cung cấp giao diện web cho phép người quản trị thông qua đó, quản lý các profile cũng như các máy trạm được cài đặt.
### 1.3. Đối tượng trong cobbler
* Các đối tượng chính được sử dụng trong cobbler
  * **Distribution**: Chứa các thông tin về kernel và initrd nào được sử dụng,các dữ liệu dùng để cài đặt, các thông số kernel (đơn giản như file ISO cài đặt).
  * **Profile**: Bao gồm distribution, kickstart file, các package cài đặt.
  * **System**: Gồm profile và MAC address. Đại diện cho các máy client được cung cấp, chỉ tới một profile hoặc một image và chứa thông tin về IP và địa chỉ MAC, quản lý tài nguyên và nhiều loại data chuyên biệt.
  * **Repository**: Giữ thông tin về các mirror repo cho quá trình cài đặt và cập nhật phần mềm của các máy client.
<br>Ta có thể hình dung quá trình khởi tạo OS cho client của cobbler: Tạo distribution -> profile -> repo -> system -> boot client.
## 2. Cài đặt và cấu hình cobbler
## Cài đặt
### Bước 1: Thiết lập Epel Repository.
>[root@quoccb ~]# yum install epel-release
### Bước 2: Cài đặt Cobbler và các package cần thiết.
>[root@quoccb ~]# yum install cobbler cobbler-web dnsmasq syslinux pykickstart xinetd -y
### Bước 3: Khởi động Cobbler và Web Server (httpd).
Thức thi các dòng lệnh systemctl sau để khởi động và enable **Cobbler** và **httpd**.
>[root@quoccb ~]# systemctl start cobblerd ; systemctl enable cobblerd
<br>[root@quoccb ~]# systemctl start httpd ; systemctl enable httpd

      Nếu SElinux được kích hoạt thì cần thiết lập cho nó chế độ **Permissive**.
>[root@quoccb ~]# setenforce 0

      Nếu **Firewall** đang chạy thì cần thiết lập mở các port như sau: </p>
>[root@quoccb ~]# firewall-cmd --add-port=80/tcp --permanent
<br>success
<br>[root@quoccb ~]# firewall-cmd --add-port=443/tcp --permanent
<br>success
<br>[root@quoccb ~]# firewall-cmd --add-service=dhcp --permanent
<br>success
<br>[root@quoccb ~]# firewall-cmd --add-port=69/tcp --permanent
<br>success
<br>[root@quoccb ~]# firewall-cmd --add-port=69/udp --permanent
<br>success
<br>[root@quoccb ~]# firewall-cmd --add-port=4011/udp --permanent
<br>success
<br>[root@quoccb ~]# firewall-cmd --reload
<br>success
[root@quoccb ~]#
### Bước 4: Truy cập giao diện Web Cobbler.
Sau khi cài đặt xong ta thử truy cập vào Cobbler từ trình duyệt web.<br>
https://\<IP-Address-or-Hostname-Cobbler-Server\>/cobbler_web/
<br> Địa chỉ Cobbler Server của tôi là "192.168.49.140" User name mặc định là **cobbler** và password cũng mặc định là **cobbler**.
![Cobller](https://i.imgur.com/IN84TW4.png)
Ta đã hoàn thành phần cài đặt Cobbler.
## Cấu hình
### Bước 5: Cấu hình file /etc/cobbler/settings
Đầu tiên ta cần tạo và mã hóa một mật khẩu root (Mật khẩu này sẽ được sử dụng mặc định cho user **root** khi cài đặt hệ điều hành ở Client).
>[root@quoccb ~]# openssl passwd -1
<br>Password:
<br>Verifying - Password:
<br>$1$19TugM1s$MdBdE9EkDcJSehJ4O/J2J/
<br>[root@quoccb ~]#

      Cập nhật chuỗi mã hóa này vào file '/etc/cobbler/settings' cho tham số '**default_password_crypted**' và kích hoạt các tính năng của Cobbler như **DHCP**, **DNS**, **PXE**, và **TFTP** bằng cách đổi giá trị tham số từ 0 thành 1.
      <br> Thiết lập địa chỉ IP của TFTP server ở tham số '**next_server**' và địa chỉ IP của Cobbler Server ở tham số '**server**'.
      > [root@quoccb ~]# vi /etc/cobbler/settings
      <br>-------------------------------------------------------
      <br>default_password_crypted: "$1$19TugM1s$MdBdE9EkDcJSehJ4O/J2J/"
      <br>manage_dhcp: 1
      <br>manage_dns: 1
      <br>pxe_just_once: 1
      <br>next_server: 192.168.49.140
      <br>server: 192.168.49.140
      <br>--------------------------------------------------------

      ### Bước 6: Cập nhật file '/etc/cobbler/dhcp.template' và '/etc/cobbler/dnsmasq.template'.
Chỉnh sửa file 'etc/cobbler/dhcp.template' và cập nhật subnet cho dhcp server theo thiếp lập của mình.
>[root@quoccb ~]# vi /etc/cobbler/dhcp.template
<br>-----------------------------------------------
<br>subnet 192.168.49.0 &emsp; netmask &emsp; 255.255.255.0 {
<br>&emsp;&emsp;&emsp;option routers &emsp;&emsp;&emsp;&ensp;&emsp;&emsp;&emsp;&emsp;&emsp;192.168.49.140;
<br>&emsp;&emsp;&emsp;option domain-name-servers &emsp;&emsp;192.168.49.140;
<br>&emsp;&emsp;&emsp;option subnet-mask &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;255.255.255.0;
<br>&emsp;&emsp;&emsp;range dynamic-bootp &emsp;&emsp;&emsp;&emsp;&emsp; 192.168.49.30 &nbsp; 172.168.10.130;
<br>&emsp;&emsp;&emsp;default-lease-time &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;21700;
<br>&emsp;&emsp;&emsp;max-lease-time &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&nbsp;43100;
<br>&emsp;&emsp;&emsp;next-server &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;$next_server;
<br>
<br>&emsp;&emsp;&emsp;class "pxeclients" {
<br>&emsp;&emsp;&emsp;&emsp;&emsp;match if substring (option vendor-class-identifier, 0, 9) = "PXEClient";
<br>&emsp;&emsp;&emsp;&emsp;&emsp;if option pxe-system-type = 00:02 {
<br>&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;filename "ia64/elilo.efi";
<br>&emsp;&emsp;&emsp;&emsp;&emsp;} else if option pxe-system-type = 00:06 {
<br>&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;filename "grub/grub-x86.efi";
<br>&emsp;&emsp;&emsp;&emsp;&emsp;} else if option pxe-system-type = 00:07 {
<br>&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;filename "grub/grub-x86_64.efi";
<br>&emsp;&emsp;&emsp;&emsp;&emsp;} else {
<br>&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;filename "pxelinux.0";
<br>&emsp;&emsp;&emsp;&emsp;&emsp;}
<br>&emsp;&emsp;&emsp;}
<br>}
<br>--------------------------------------------   

    Cập nhật địa dải địa chỉ IP cho các pxe client trong file '/etc/cobbler/dnsmasq.template'
    >[root@quoccb ~]# vi /etc/cobbler/dnsmasq.template
<br>dhcp-range=192.168.49.30,192.168.49.130

    Khởi động lại **Cobbler** và **xinetd** và đồng bộ những thay đổi cho cobbler.
     >[root@quoccb ~]# systemctl restart cobblerd
<br>[root@quoccb ~]# systemctl restart xinetd ; systemctl enable xinetd
<br>[root@quoccb ~]# cobbler check ; cobbler sync

    **Mount file ISO và import vào trong cobbler**
    <br> Tải và copy file ISO của CentOS 7 và chạy các lệnh sau để import ISO vào Cobbler.<br>
    >[root@quoccb ~]# mkdir  /mnt/iso
<br>[root@quoccb ~]# mount -o loop CentOS-7-x86_64-DVD-1511.iso  /mnt/iso/
<br>[root@quoccb ~]# cobbler import --arch=x86_64 --path=/mnt/iso --name=CentOS7

    Tương tự ta có thể import file ISO của các phiên bản Linux khác. Nếu gặp *signature errors* trong khi import, chạy lệnh sau.
    > [root@quoccb~]# cobbler signature update

    ### Bước 7: Xác minh danh sách Distro bằng command line và giao diện web Cobbler.
    Chạy lệnh sau để xem danh sách các Distro.
    >[root@quoccb ~]# cobbler distro list
  <br> &enps;&enps;CentOS7-x86_64
<br>[root@quoccb ~]#

    Ta cũng có thể xem danh sách các Distro từ giao diện Web của Cobbler.
    <br> Đăng nhập vào giao diện rồi vào Configuration Tab --> click on Distros.
      ![Web interface](https://i.imgur.com/xfy1sv7.png)
    ### Bước 8: Tạo file Kickstart cho CentOS 7.
    Tạo một file kickstart cho CentOS 7 với tên '**CentOS7.ks**', vị trí mặc định của file kickstart là '/var/lib/cobbler/kickstarts'.
```
  TYPE="Ethernet"
#platform=x86, AMD64, or Intel EM64T
#version=DEVEL
# Firewall configuration
firewall --disabled
# Install OS instead of upgrade
install
# Use HTTP installation media
url --url="http://192.168.49.140/cblr/links/CentOS7-x86_64/"
#
# Root password
rootpw --iscrypted $default_password_crypted
#
# Network information
network --bootproto=dhcp --device=eth0 --onboot=on --nameserver=8.8.8.8 --activate
network  --hostname=quoccb.meditech
#
# Reboot after installation
reboot
#
# System authorization information
auth useshadow passalgo=sha512
#
# Use graphical install
graphical
#
firstboot disable
#
# System keyboard
keyboard us
#
# System language
lang en_US
#
# SELinux configuration
selinux disabled
#
# Installation logging level
logging level=info
#
# System timezone
timezone Asia/Ho_Chi_Minh --isUtc
#
# System bootloader configuration
bootloader location=mbr
#
clearpart --all --initlabel
part swap --asprimary --fstype="swap" --size=1024
part /boot --fstype xfs --size=500
part pv.01 --size=1 --grow
volgroup root_vg01 pv.01
logvol / --fstype xfs --name=lv_01 --vgname=root_vg01 --size=1 --grow
#
%packages
@^minimal
@core
%end
#
%post
yum install -y httpd ; systemctl enable httpd
%end
#
%addon com_redhat_kdump --disable --reserve-mb='auto'
%end
```
Bước cuối cùng là đồng bộ profile đã được update lên cobbler server sử dụng lệnh cobbler.

      ```
[root@quoccb ~]# cobbler profile edit --name=CentOS7-x86_64 --kickstart=/var/lib/cobbler/kickstarts/CentOS7.ks
[root@quoccb ~]# cobbler sync
```
  Ta đã cấu hình xong Cobbler Server, bây giờ có thể khởi động hệ thống với pxe hoặc mạng và thử cài đặt.
