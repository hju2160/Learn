Redline là một công cụ sử dụng để phân tích memory và kiến trúc file của các máy bị tấn công để tìm ra các dấu hiệu của cuộc tấn công\
Chức năng:
- Thu thập các process đang chạy, file, registry (Windows only), memory images (Windows version < 10)
- View các data quan trọng, giới hạn tìm kiếm với timeframe
- phân tích IoC (Windows only)
- white list lọc giá trị dựa trên mã hash md5

<h1>Redline_Revilcorp_TryHackMe</h1>

1.**What is the compromised employee's full name?**
Analysis Data > User
![image](https://github.com/s1mHATs/Learn/blob/main/redline/image/1.png?raw=true)\
flag1:`john coleman`\
2.**What is the operating system of the compromised host?**
Analysis Data > System Information
![image](https://github.com/s1mHATs/Learn/blob/main/redline/image/2.png?raw=true)\
flag2:`Windows 7 Home Premium 7601 Service Pack 1`\
3.**What is the name of the malicious executable that the user opened?**
Analysis Data > File System > Filter theo User\
Danh sách các file nghi ngờ theo Signature Description `The file is not signed`
| File path | File Hash |
|-----|-----|
|C:\Users\John Coleman\Desktop\d.e.c.r.yp.tor.exe|f617af8c0d276682fdf528bb3e72560b|
|C:\Users\John Coleman\Downloads\WinRAR2021.exe|890a58f200dfff23165df9e1b088e58f|

![image](https://github.com/s1mHATs/Learn/blob/main/redline/image/3.1.png?raw=true)\
![image](https://github.com/s1mHATs/Learn/blob/main/redline/image/3.2.png?raw=true)\
Kiểm tra mã hash của 2 file này trên Virus Total thì đều thuộc về Redvil Ransomware\
Đọc thêm tại link: https://www.secureworks.com/research/revil-sodinokibi-ransomware \
Có thể thấy Redvil được các kẻ tấn công giả mạo thành file WinRAR và sau đó thay thế bản WinRAR thật trên trang web WinRAR.it của Italian bằng file mã độc\
flag3:`WinRAR2021.exe`\
4.**What is the full URL that the user visited to download the malicious binary? (include the binary as well)**
Analysis Data > File Download History
![image](https://github.com/s1mHATs/Learn/blob/main/redline/image/4.1.png?raw=true)
>| Auto | http://192.168.75.129:4748/Documents/WinRAR2021.exe | C:\Users\John Coleman\Downloads\WinRAR2021.exe | 164 Kilobytes | 2021-08-02 19:42:16Z | Internet Explorer | John Coleman |

flag4:`http://192.168.75.129:4748/Documents/WinRAR2021.exe`\
5.**What is the MD5 hash of the binary?**\
flag5:`890a58f200dfff23165df9e1b088e58f`\
6.**What is the size of the binary in kilobytes?**\
flag6:`164`\
7.**What is the extension to which the user's files got renamed?**
>Theo Securework, Revil ransomware sẽ kiểm tra registry key trong \SOFTWARE\recfg để kiểm tra giá trị rnd_ext có tồn tại hay k (giá trị này là một extension ngẫu nhiễn đc tạo và gán cho các file mã hóa sinh ra trong quá trình chạy).
Nếu k có registry này, revil sẽ tự tạo một string ngẫu nhiên là tổ hợp của a-z 0-9 có từ 5-10 và sử dụng nó làm extension

Analysis Data > File System\
Có thể thấy trong Desktop có một số file có extension ngẫu nhiên `.t48s39la` và có thời gian được modified cuối giống nhau
![image](https://github.com/s1mHATs/Learn/blob/main/redline/image/7.png?raw=true)\
flag7:`.t48s39la`\
8.**What is the number of files that got renamed and changed to that extension?**
>Timeline là một tool sử dụng để xác định thời điểm xảy ra các compromise, các file đã bị tác động, các compromise đang tiếp diễn
Timeline hiển thị event đc sắp xếp theo thời gian tăng dần và hỗ trợ áp dụng các filter để chỉ định user và process.
TimeWrinkle và TimeCrunch filter theo list, chỉ hiển thi các sự kiện đã xảy ra hoặc xảy ra gần 1 khoảng thời gian cụ thể
TimeWrinkle hiển thị các sự kiện trong 1 khoảng thời gian chỉ định
TimeCrunch loại bỏ các sự kiện nhiễu (chỉ định)

Analysis Data > Timeline\
Tìm các file được Modified và Change mà có chứa ext `.t48s39la`:
![image](https://github.com/s1mHATs/Learn/blob/main/redline/image/8.png?raw=true)\
flag8:`48`\
9.**What is the full path to the wallpaper that got changed by an attacker, including the image name? **
>Theo Securework, sau khi mã hóa dữ liệu, REvil sẽ thay đổi background của máy nạn nhân bằng một bitmap image.
Image này sẽ được lưu với filename là bao gồm chữ thường và số từ 3-13 ký tự với ext .bmp. 
REvil sẽ gọi user32.dll SystemParamentersInfoW để thay đổi background.

Analysis Data > Timeline\
Tìm các file có ext `.bmp` được tại bởi user `John Coleman`:
![image](https://github.com/s1mHATs/Learn/blob/main/redline/image/9.png?raw=true)\
file hk8.bmp là file background được REvil sử dụng\
flag9:`C:\Users\John Coleman\AppData\Local\Temp\hk8.bmp`\
10.**The attacker left a note for the user on the Desktop; provide the name of the note with the extension.**
>Theo Securework, một note sẽ được tạo dưới dạng file txt và có ext trong tên của nó\
REvil generates the ransom note's filename using a similar process. It obtains the value stored within the "nname" key in its configuration and replaces the {EXT} variable placeholder with its corresponding value. In the analyzed sample, the nname key value "{EXT}-HOW-TO-DECRYPT.txt" led to the ransom note filename 9781xsd4-HOW-TO-DECRYPT.txt.

Analysis Data > Timeline\
Tìm các file .txt có chứa ext `t48s39la`:
![image](https://github.com/s1mHATs/Learn/blob/main/redline/image/10.png?raw=true)\
flag10:`t48s39la-readme.txt`\
11.**The attacker created a folder "Links for United States" under C:\Users\John Coleman\Favorites\ and left a file there. Provide the name of the file.**
Analysis Data > Timeline\
Filter theo tên folder `Links for United States`, có thể thấy có 2 file `USA.gov.url.t48s39la` và `GobiernoUSA.gov.url.t48s39la` được tạo cùng lúc với folder:
![image](https://github.com/s1mHATs/Learn/blob/main/redline/image/11.png?raw=true)\
flag11:`GobiernoUSA.gov.url.t48s39la`\
12.**There is a hidden file that was created on the user's Desktop that has 0 bytes. Provide the name of the hidden file.**
Analysis Data > File System > User > John Coleman > Desktop
![image](https://github.com/s1mHATs/Learn/blob/main/redline/image/12.png?raw=true)\
flag12:`d60dff40.lock`\
13.**The user downloaded a decryptor hoping to decrypt all the files, but he failed. Provide the MD5 hash of the decryptor file.**
Thử mã md5 của file `d.e.c.r.yp.tor.exe` tìm ở trên:
![image](https://github.com/s1mHATs/Learn/blob/main/redline/image/13.png?raw=true)\
flag13:`f617af8c0d276682fdf528bb3e72560b`\
14.**In the ransomware note, the attacker provided a URL that is accessible through the normal browser in order to decrypt one of the encrypted files for free. The user attempted to visit it. Provide the full URL path.**
> Theo Secureworks, REvil sẽ cung cấp một url duy nhất để giải mã các file đã bị mã hóa.

Kiểm tra trong Note mẫu có thể thấy url là: http://decryptor.top/{UID}\
Trong đó UID là số serial của ổ đĩa và CPUID của máy chủ.
![image](https://github.com/s1mHATs/Learn/blob/main/redline/image/14.1.png?raw=true)\
Analysis Data > Browser URL History\
Tìm kiếm với filter `decryptor.top`
![image](https://github.com/s1mHATs/Learn/blob/main/redline/image/14.2.png?raw=true)\
flag14:`http://decryptor.top/644E7C8EFA02FBB7`\
15.**What are some three names associated with the malware which infected this host? (enter the names in alphabetical order)**
OSINT: https://attack.mitre.org/software/S0496/ 
![image](https://github.com/s1mHATs/Learn/blob/main/redline/image/15.png?raw=true)\
flag15:`REvil,Sodin,Sodinokibi`
