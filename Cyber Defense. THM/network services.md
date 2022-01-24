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
What comes up as the name of the machine?
![image](https://user-images.githubusercontent.com/95600382/150718902-299b8fc9-7070-41a5-8930-a37a2bc809c3.png)




