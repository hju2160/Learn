<h2>NFS</h2>

NFS as Network File System cho phép hệ thống chia sẻ thư mục và file với các thành phần khác trong mạng. Bằng cách sử dụng NFS, user hoặc program có thể truy cập file trên các hệ thống từ xa. Chức năng này được thực hiện bằng cách mount toàn bộ hoặc 1 phần file system trên server. Các phần file system mà được mount có thể được truy cập bởi Client bất chấp các quyền được gán cho file đó.\
NFS work\
detail: https://docs.oracle.com/cd/E19683-01/816-4882/6mb2ipq7l/index.html\
Đầu tiên, Client sẽ request để mount thư mục trên remote host tương tự với cách mount các thiết bị vật lý.\
Sau đó, mount service sẽ kết nối tới các mount deamon liên quan bằn RPC\
Server sẽ thực hiện check quyền mount của user đối với các thư mục đc request. Server sẽ trả lại môt file handle xác định từng file và thư mục riêng biệt tồn tại trên server\
Nếu có người thực hiện access file sử dụng NFS, RPC call sẽ được đặt vào NFSD (NFS daemon) trên server. RPC call có các tham số như:\
- File handle
- Tên file có thể access
- User ID
- Group ID
Các tham số trên được sử dụng để xác định quyền truy cập vào tệp đc chỉ định và kiểm soát quyền của người dùng.\
NFS được sử dụng để chuyển file giứa Windows OS và non-Windows OS\
Windows Server có thể hoạt động như 1 NFS File Server hoặc sử dụng NFS để truy cập các file trên các non-Windows OS\
**Answer**\
What process allows an NFS client to interact with a remote directory as though it was a physical device? `mounting`\
What does NFS use to represent files and directories on the server? `file handler`\
What protocol does NFS use to communicate between the server and client? `rpc`\
What two pieces of user data does the NFS server take as parameters for controlling user permissions? `user id / group id`\
Can a Windows NFS server share files with a Linux client? `Y`\
Can a Linux NFS server share files with a MacOS client? `Y`\
What is the latest version of NFS? [released in 2016, but is still up to date as of 2020] `4.2`\

<h3>Khai thác NFS</h3>

Để thực hiện dò quét NFS ta cần 1 số tool như sau:\
NFS-Common(!!!) là một packet quan trọng được cài trên tất cả các máy sử dụng NFS, cả client lẫn server. bao gồm các program như: lockd, statd, showmount, nfsstat, gssdm, idmap, mount.nfs. Đặc biệt là là 2 program "showmount" và "mount.nfs" là hữu ích nhất khi khai thách thông tin từ NFS\
port scanning có thể sử dụng nmap với option -A và -p-\
Mounting NFS Shares: Client sẽ cần một thư mục mà chứa tất cả các nội dụng mà được share bởi server. B có thể tạo folder và sử dụng lệnh `mount` để kết nối tới NFS server
```
sudo mount -t nfs IP:share /tmp/mount/ -nolock
```
-t nfs type of device to mount\
IP:share IP của NFS server, trên của phần muốn mount\
-nolock chỉnh định không dùng NLM locking

Sử dụng nmap để khai thác thông tin.
```
nmap -p- -A -sC -Pn [IP add]
```
![image](https://user-images.githubusercontent.com/95600382/155302225-bbccb88b-f5da-48d0-8680-7860da114abc.png)\
![image](https://user-images.githubusercontent.com/95600382/155302248-8f488bb4-6b5b-4a41-ae01-6b4efa928ab2.png)\

**Answer**\
Conduct a thorough port scan scan of your choosing, how many ports are open? `7`\
Which port contains the service we're looking to enumerate? `2049`\
Sử dụng lệnh để xem các file đc share
```
showmount -e [IP]
```
![image](https://user-images.githubusercontent.com/95600382/155302968-e0ca18be-8bfe-40b7-bedc-d806e505f0a5.png)\
Now, use /usr/sbin/showmount -e [IP] to list the NFS shares, what is the name of the visible share? `/home`\
Tạo thư mục /tmp/mount và mount file share về máy local vào thư mục vừa tạo
```
mkdir /tmp/mount
sudo mount -t nfs IP:file_share /tmp/mount/ -nolock
```
Then, use the mount command we broke down earlier to mount the NFS share to your local machine. Change directory to where you mounted the share- what is the name of the folder inside? `cappucino`\
![image](https://user-images.githubusercontent.com/95600382/155303923-df4a0e1a-ad17-4cbc-96f7-13dd14eea3f1.png)\
Nhìn vào file vừa lấy đc, có vẻ đây là thư mục home của uesr cappucino.\
Interesting! Let's do a bit of research now, have a look through the folders. Which of these folders could contain keys that would give us remote access to the server? `.ssh`\
Which of these keys is most useful to us? `id_rsa`\
Copy this file to a different location your local machine, and change the permissions to "600" using "chmod 600 file".\
Assuming we were right about what type of directory this is, we can pretty easily work out the name of the user this key corresponds to.\
Can we log into the machine using ssh -i <key-file> <username>@<ip> ? `Y`
```
ssh -i file_key user@IP
```  
![image](https://user-images.githubusercontent.com/95600382/155305146-99b81688-e6b5-45d4-920d-6a930ed2bd38.png)\
Có thể sử dụng NFS share để thực hiện leo thang đặc quyền, phụ thuộc vào cách NFS đc config. Mặc định NFS Shares- Root Squashing được enable
