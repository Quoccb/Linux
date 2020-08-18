# Cài đặt và cấu hình Local Mail Server trên CentOS7
## 1. Chuẩn bị
- **Hệ thống:**
  * **OS**: CentOS 7 64bit minimal server
  * **IP Address**: 192.168.49.141/24
  * **Hostname**: mail.test.local
- Đầu tiên chúng ta cần gỡ tất cả các MTA sendmail đã được cài trước đó. Với bản minimal thì mặc định chưa có sendmail nào được cài.
>yum remove sendmail

- Thêm các thông tin vào file **/etc/hosts**:
> vi /etc/hosts

    Thêm tên miền đầy đủ (FQDN):
>127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
<br>::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
<br>
<br>192.168.49.141  mail.test.local mail

- **Disabled SELinux** để giảm độ phức tạp khi cấu hình postfix
> vi /etc/sysconfig/selinux

  Chỉnh SELINUX=enforcing thành SELINUX=disabled
>SELINUX=disabled

- Cài đặt EPEL Repository:
Vì Squirrelmail webmail client không có sẵn trên các repository chính thức của CentOS nên chúng ta cần sử dụng EPEL repository.
>yum install epel-release

- Mở cổng 80 của firewall:
> firewall-cmd --permanent --add-port=80/tcp

  Khởi động lại firewall:
> firewall-cmd --reload

Khởi động lại server để toàn bộ thay đổi có hiệu lực.
## Cài đặt **Postfix**
**Postfix** là một MTA mã nguồn mở miễn phí. Nó có tốc độ nhanh, bảo mật và dễ dàng quản lý. Postfix là MTA mặc định cho RHEL.

Bắt đầu cài đặt Postfix:
>yum install postfix

## Cấu hình **Postfix**
Chỉnh file **/etc/postfix/main.cf**:
> vi /etc/postfix/main.cf

  Tìm và sửa các dòng sau:
  >\## Line no 77 - Bỏ comment và đặt lại FQDN ##
  <br>myhostname = mail.test.local
  <br><br>
  \## Line 85 - Bỏ comment và đặt lại domain name ##<br>
mydomain = test.local
<br><br>
\## Line 101 - Bỏ comment ##<br>
myorigin = $mydomain
<br><br>
\## Line 115 - Bỏ comment và set IPv4 ##<br>
inet_interfaces = all
<br><br>
\## Line 121 - chuyển thành **all** ##<br>
inet_protocols = all
<br><br>
\## Line 166 - Comment ##<br>
\#mydestination = $myhostname, localhost.$mydomain, localhost,
<br><br>
\## Line 167 - Bỏ comment ##<br>
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain
<br><br>
\## Line 266 - Bỏ comment và thêm khoảng địa chỉ IP ##
mynetworks = 192.168.49.0/24, 127.0.0.0/8
<br><br>
\## Line 421 - Bỏ comment ##<br>
home_mailbox = Maildir/

  Lưu và thoát file.
Start/ Restart dịch vụ Postfix :
>systemctl enable postfix

>systemctl restart postfix

Tiến hành Test Mail server Postfix :

  - Đầu tiên, tạo user với tên “user1” :
 >useradd user1

- Set Password cho user :
>passwd user1

- Truy cập Server thông qua Telnet và nhập lệnh thủ công :
>telnet localhost smtp

Lưu ý nếu máy không truy cập được Telnet thì chúng ta cần tiến hành cài thêm Telnet.

Kết quả trả về :
>[root@mail user1]# telnet localhost smtp<br>
Trying ::1...<br>
Connected to localhost.<br>
Escape character is '^]'.<br>
220 mail.test.local ESMTP Postfix<br>
ehlo localhost &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;    ## nhập dòng này ## <br>
250-mail.test.local<br>
250-PIPELINING<br>
250-SIZE 10240000<br>
250-VRFY<br>
250-ETRN<br>
250-ENHANCEDSTATUSCODES<br>
250-8BITMIME<br>
250 DSN<br>
mail from:<user1>   &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;  ## nhập vào địa chỉ mail gửi ##  <br>   
250 2.1.0 Ok<br>
rcpt to:<user1>  &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;  ## nhập vào địa chỉ mail nhận##<br>
250 2.1.5 Ok<br>
data  &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;  ## nhập vào để nhập vào nội dung của mail ##   <br>
354 End data with <CR><LF>.<CR><LF><br>
Hello &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;  ## nội dung của mail ## <br>
This mail is for test   &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;  ## nội dung của mail ##  <br>
.  &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; ## đặt dấu (.) để kết thúc nội dung mail ## <br>
250 2.0.0 Ok: queued as 29D4C204FBF0<br>
quit &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; ## quit để thoát ## <br>
221 2.0.0 Bye<br>
Connection closed by foreign host.<br>

  Kiểm tra xem mail đã đến chưa:
>ls /home/user1/Maildir/new/

  Kết quả trả về
> 1597733652.Vfd00I2015c58M404120.mail.test.local

  Như vậy là chúng ta đã gửi nhận mail thành công với user “ user1 ”.

  Để đọc mail, nhập lệnh :
  >cat /home/user1/Maildir/new/1597733652.Vfd00I2015c58M404120.mail.test.local

  Kết quả trả về:
  >Return-Path: <user1@test.local> <br>
X-Original-To: user1 <br>
Delivered-To: user1@test.local <br>
Received: &emsp;from localhost (localhost [IPv6:::1])<br>
      &emsp;&emsp;&emsp;&emsp;&emsp; by mail.test.local (Postfix) with ESMTP id 29D4C204FBF0<br>
    &emsp;&emsp;&emsp;&emsp;&emsp;    for <user1>; Tue, 18 Aug 2020 13:53:14 +0700 (+07)<br>
Message-Id: <20200818065336.29D4C204FBF0@mail.test.local><br>
Date: Tue, 18 Aug 2020 13:53:14 +0700 (+07)<br>
From: user1@test.local<br>
<br>
Hello<br>
This mail is for test

Như vậy là chúng ta đã hoàn thành cài đặt và cấu hình, gửi nhận mail thành công trên Postfix.

## Cài đặt Dovecot :
Dovecot cũng là chương trình Open Source mail server POP3 và IMAP dùng cho hệ thống Linux/Unix System.
>yum install dovecot

## Cấu hình Dovecot :
Edit file **/etc/dovecot/dovecot.conf** :

>vi /etc/dovecot/dovecot.conf

Bỏ comment những dòng sau :

> \## Line 24 – bỏ comment ##<br>
protocols = imap pop3 lmtp

Edit file **/etc/dovecot/conf.d/10-mail.conf** :
>vi /etc/dovecot/conf.d/10-mail.conf

Thay đổi dòng :
>\## Line 24 – bỏ comment ##<br>
mail_location = maildir:~/Maildir

Edit file **/etc/dovecot/conf.d/10-auth.conf** :
>vi /etc/dovecot/conf.d/10-auth.conf

Thay đổi những dòng sau :
>\## line 10 – bỏ comment##<br>
disable_plaintext_auth = yes
<br>
\## Line 100 – thêm từ: "login" ##
<br>auth_mechanisms = plain login

Edit file **/etc/dovecot/conf.d/10-master.conf** :
>vi /etc/dovecot/conf.d/10-master.conf

Tiến hành thay đổi dòng :
>\## Line 91, 92 – bỏ comment và thêm "postfix"
<br>#mode = 0600
<br><br>
&emsp;&emsp;&emsp;   user = postfix
<br><br>
&emsp;&emsp;&emsp;group = postfix
<br><br>
[...]

Enable và Restart dịch vụ Dovecot :
>systemctl enable dovecot

>systemctl start dovecot

### Test Dovecot service :
Nhập lệnh telnet :
>telnet localhost pop3

Thêm lần lượt những dòng lệnh khu vực được tô đậm :
>Trying ::1...<br>
Connected to localhost.<br>
Escape character is '^]'.<br>
+OK Dovecot ready.<br>
**user user1** &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; ## Nhập vào tên user ## <br>
+OK<br>
**pass user1** &emsp;&emsp;&emsp;&emsp;&emsp; ## Nhập mật khẩu cho user. Ở đây ***user1*** là mật khẩu của **user1** ##<br>
+OK Logged in.<br>
retr 1&emsp;&emsp;&emsp;&emsp;&emsp; ## nhập lệnh để xem mail ##<br>
+OK 407 octets<br>
Return-Path: <user1@test.local><br>
X-Original-To: user1<br>
Delivered-To: user1@test.local<br>
Received: &emsp;from localhost (localhost [IPv6:::1])<br>
  &emsp;&emsp;&emsp;&emsp;&emsp;      by mail.test.local (Postfix) with ESMTP id 29D4C204FBF0<br>
&emsp;&emsp;&emsp;&emsp;&emsp;        for <user1>; Tue, 18 Aug 2020 13:53:14 +0700 (+07)<br>
Message-Id: <20200818065336.29D4C204FBF0@mail.test.local><br>
Date: Tue, 18 Aug 2020 13:53:14 +0700 (+07)<br>
From: user1@test.local<br>
<br>
Hello<br>
This mail is for test<br>
.<br>
quit&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; ## Nhập quit để thoát ## <br>
+OK Logging out.<br>
Connection closed by foreign host.<br>

Ta đã cài đặt và cấu hình xong **Dovecot**
## Cài đặt Squirrelmail :
Đối với người dùng bình thường, thì việc gửi và nhận mail bằng command line xem ra rất khó khăn. Sẽ dễ dàng hơn cho họ nếu có giao diện đồ họa. **Squirrelmail** được xem là giải pháp hoàn hảo khắc phục những nhược điểm của việc gửi nhận mail bằng command line.

Trước tiên, đảm bảo rằng bạn đã cài đặt và enable EPEL repository.

Sau đó, cài đặt Squirrelmail bằng dòng lệnh :

>yum install squirrelmail

## Cấu hình Squirrelmail :
Di chuyển đến thư mục **/usr/share/squirrelmail/config/**

>cd /usr/share/squirrelmail/config/

và chạy dòng lệnh để mở và cấu hình Squirrelmail :

> ./conf.pl

Bảng lựa chọn sẽ hiển thị, từ bảng lựa chọn này ta sẽ cấu hình những thông tin cần thiết như **Organization Name, Title, Logo,** **nhà cung cấp tên miền**.<br>
Sau đó tiếp tục setup cho mail server các thông số như domain name, mail agent…
<br> chuyển từ **Sendmail** sang **Postfix MTA**

Xong chúng ta nhấn “ **S** “ tiếp theo là “ **Q** “ để *save* và *exit* bảng cấu hình Squirrelmail.

Tạo một squirrelmail vhost trong file **httpd.conf**
>vi /etc/httpd/conf/httpd.conf

tại cuối file, thêm vào những dòng này :

>Alias /webmail /usr/share/squirrelmail
<br><br>
<Directory /usr/share/squirrelmail>
<br><br>
Options Indexes FollowSymLinks
<br><br>
RewriteEngine On
<br><br>
AllowOverride All
<br><br>
DirectoryIndex index.php
<br><br>
Order allow,deny
<br><br>
Allow from all
<br><br>
</Directory>

Restart lại Apache Server :

>systemctl restart httpd

Ta tạo các mail user bằng lệnh **useradd** và đặt password bằng lệnh **passwd**
### Truy cập Webmail :
Vào trình duyệt gõ http://ip-address/webmail hoặc http://domain-name/webmail :

![Webmail](https://i.imgur.com/gObZ5Z9.png)
<br><br>
Đăng nhập vào. <br><br>
![Webmail](https://i.imgur.com/XoeSug5.png)

Vào **Compose** để gửi mail. Địa chỉ mail sẽ là ***user1@test.local***.

Như vậy là ta đã cài đặt xong một **Local Mail Server** trên **CentOS 7**
