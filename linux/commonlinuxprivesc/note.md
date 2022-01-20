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
2 cách. Cách 1 host 1 server với python và down trực tiếp về máy mục tiêu bằng cách sử dụng `wget` sau đó phân quyền với `chmod +x` và thực thi script.\
VD: đăng nhập ssh vào máy mục tiêu với user3:password\
Target hostname:`polobox`\
![image](https://user-images.githubusercontent.com/95600382/150293666-32f50534-6865-40f6-b112-4aecf6c3e9ad.png)\

