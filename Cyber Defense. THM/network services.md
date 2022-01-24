<h2>Understanding SMB</h2>

SMB- Server Message Block Protocol, client- server protocol để chia sẻ các truy cập đến file, printer, serial port, và các tài nguyên khác trong mạng.\
Request-Response protocol. Client kết nối với server sử dụng NetBIOS over TCP/IP, NETBEUI hoặc IPX/SPX. Chạy với port mặc định là 445 và 139.\
![image](https://user-images.githubusercontent.com/95600382/150717151-e24b5ee8-44f1-42fa-807a-e28c286cabef.png)\

<h2>Enumerating SMB</h2>

Thu thập thông tin từ SMB để chuẩn bị tấn công\
Công cụ có thể sử dụng `nmap`, `Enum4Linux`,...\
Trong đó `Enum4Linux` là tool sử dụng để thu thập thông tin từ SMB cho cả Windows và Linux từ các samba packet\
https://github.com/CiscoCXSecurity/enum4linux\
Syntax:
```
enum4linux [options] ip
```
|tag|function|
|---|---|
|-U|get userlist|
|-M|get machine list|
|-N|get namelist dump (different from -U and-M)|
|-S|get sharelist|
|-P|get password policy information|
|-G|get group and member list|
|-a|all of the above (full basic enumeration)|

![image](https://user-images.githubusercontent.com/95600382/150718456-e3c8293b-7113-49e2-a088-d9efbc87a261.png)\
Conduct an nmap scan of your choosing, How many ports are open? `3`\
What ports is SMB running on? `139/445`\
Let's get started with Enum4Linux, conduct a full basic enumeration. For starters, what is the workgroup name?\
![image](https://user-images.githubusercontent.com/95600382/150718706-506f3e71-e25a-48bc-b155-2a68eda8c032.png)\
`WORKGROUP`\
What comes up as the name of the machine?\
![image](https://user-images.githubusercontent.com/95600382/150718902-299b8fc9-7070-41a5-8930-a37a2bc809c3.png)\
`polosmb`\
What operating system version is running?\    
`6.1`\
What share sticks out as something we might want to investigate?\
![image](https://user-images.githubusercontent.com/95600382/150719226-46fdcc8a-3248-474b-9260-e5b9d7fda36d.png)\
`profiles`\

<h2>Exploiting SMB</h2>

**Type of SMB Exploit**\
Có một số lỗ hổng của SMB như CVE-2017-7494 cho phép RCE. Trong bài này sẽ khai thác anonomous SMB share access.\
Trong bc điều tra ở trên, ta đã biết được:\
SMB Share location\
Các SMB share thú vị ?\

Sử dụng SMBClient để truy cập vào tài nguyên được share\
Syntax:
```
smbclient //[IP]/[SHARE]
```
-U user\
-p port\
What would be the correct syntax to access an SMB share called "secret" as user "suit" on a machine with the IP 10.10.10.2 on the default port?
```
smbclient //10.10.10.2/secret -U suit -p 149
```
Thử đăng nhập bằng accpunt `guest`:
```
smbclient //10.10.180.253/profiles -u guest
```
Đã access thành công:\
![image](https://user-images.githubusercontent.com/95600382/150723030-31a5187a-2768-4d5e-b603-70339a1d5d89.png)\
Great! Have a look around for any interesting documents that could contain valuable information. Who can we assume this profile folder belongs to?\
Thực hiện GET file `Working From Home Information.txt` và đọc vs `cat`:\
![image](https://user-images.githubusercontent.com/95600382/150738004-ea8ea459-2f3b-41a7-8bd7-efce50f27113.png)\
`John Cactus`\
What service has been configured to allow him to work from home?\
`ssh`\
Okay! Now we know this, what directory on the share should we look in?\
`.ssh`\
This directory contains authentication keys that allow a user to authenticate themselves on, and then access, a server. Which of these keys is most useful to us?\
`id_rsa`\
Thực hiện download thư mục .ssh về máy:\
![image](https://user-images.githubusercontent.com/95600382/150738797-eb12e800-022e-4140-bca9-e9004f8acce8.png)\
![image](https://user-images.githubusercontent.com/95600382/150740482-57a32fa1-9121-4292-9a17-9aeab01642f2.png)\
https://serverpilot.io/docs/how-to-use-ssh-public-key-authentication/\

<h2>Telnet</h2>

Telnet là dạng Client-server. Client sẽ kết nối đến server và trở thành một virtual terminal để tương tác với server\
Telnet gửi tất cả message ở dạng clear text, port 23
✔\

<h2>Exploiting Telnet</h2>

![image](https://user-images.githubusercontent.com/95600382/150752120-daae686b-ea98-4b9e-9fce-f9dee964aaa0.png)\

<h2>FTP</h2>

FTP server có thể hỗ trợ cả kết nối Active và Passive\
ở Active, cliet mở port và lắng nghe, server sẽ chủ động mở kết nối tới client\
ở passive thì ngược lại\

✔\

<h2>Exploit FTP</h2>

![image](https://user-images.githubusercontent.com/95600382/150757764-08ce8321-5f50-461e-8aa2-e5c029de5a46.png)\
Brutce force FTP password:\
![image](https://user-images.githubusercontent.com/95600382/150758084-7d48ff35-9431-4d0c-a40d-b2a8af521b86.png)\
![image](https://user-images.githubusercontent.com/95600382/150758538-f3f519d0-9f08-43ef-b5ee-eb175044b605.png)\












