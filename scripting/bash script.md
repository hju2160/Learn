1 bash script sẽ luôn bắt đầu vs `#!/bin/bash` ở đầu script\
Điều này để giúp máy biết đc đây là một shell và chạy file đó với bash trong terminal\
VD:
```bash
#!/bin/bash\
echo "Hello World!"
```
```bash
#!/bin/bash\
echo "Hello World!"
whoami
id
```
**Variables**\
Các biến trong bash script có thể tạo đơn giản như sau:
```bash
name="Hieu"
```
Sử dụng ký tự `$` để gọi biến\
```bash
name="Hieu"
echo $name
```
Debug script, có thể chạy với lệnh bash -x ./<scipt_file> để biết được dòng nào đang lỗi\
Hoặc có thể debug từng đoạn trong code với set +/-x\
```
echo "hieu"
set -x
#đoạn debug
set +x
```
Test với đoạn script:\
```bash
#!/bin/bash\
echo "Hello World!"
whoami
id
```
![image](https://user-images.githubusercontent.com/95600382/149728352-2f9a5cf7-3b4e-4c24-9d2c-6eb9660e9c57.png)
Dấu `+` sẽ thể hiện cho các command đã được thực thi, `-` thì ngược lại\
Có thể sử dụng nhiều biến trong cùng 1 đoạn script. VD:\
```bash
#!/bin/bash\
name="Hieu"
city="HN"
age=22
echo "$name $age $city"
```
![image](https://user-images.githubusercontent.com/95600382/149728905-20e0aac2-b51e-4c7d-a0fe-b1e732f0fbce.png)
**Parameters**
Các tham số vẫn thường có ký hiệu `$` đứng trc vì chúng cũng được coi là một biến\
Thử chạy script sau với input là Hieu123:\
```
name=$1
echo $name
```
KQ:\
![image](https://user-images.githubusercontent.com/95600382/149729884-3631fd89-e90f-4b6e-bf28-7c92fdacd3e6.png)\
Có thể thấy nó đã in ra tham số `Hieu123` vừa được sử dụng cho input\
Test tiếp với tham số bằng `$2` và 2 input là `Hieu1` và `Hieu2`. Kết quả:
![image](https://user-images.githubusercontent.com/95600382/149730099-569ec17c-11ea-4720-afc1-0b3850c32a8c.png)\
Để gán một biến động ta có thể sử dụng `read`. VD:\
```bash
echo "Name:"
read name
echo "Name is: $name"
```
KQ:\
![image](https://user-images.githubusercontent.com/95600382/149730438-d35242f3-9075-4bad-bbf0-74e38316df68.png)\
**Array**
