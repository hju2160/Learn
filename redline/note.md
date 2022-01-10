Redline là một công cụ sử dụng để phân tích memory và kiến trúc file của các máy bị tấn công để tìm ra các dấu hiệu của cuộc tấn công\
Chức năng:
- Thu thập các process đang chạy, file, registry (Windows only), memory images (Windows version < 10)
- View các data quan trọng, giới hạn tìm kiếm với timeframe
- phân tích IoC (Windows only)
- white list lọc giá trị dựa trên mã hash md5

Redline_Revilcorp_TryHackMe
1.**What is the compromised employee's full name?**
Analysis Data > User
![image]()\
flag1:`john coleman`\
2.**What is the operating system of the compromised host?**
Analysis Data > System Information
![image]()\
flag2:`Windows 7 Home Premium 7601 Service Pack 1`\
3.**What is the name of the malicious executable that the user opened?**
Analysis Data > File System > Filter theo User\
Danh sách các file nghi ngờ theo Signature Description `The file is not signed`
| File path | File Hash |
|-----|-----|
|C:\Users\John Coleman\Desktop\d.e.c.r.yp.tor.exe|f617af8c0d276682fdf528bb3e72560b|
|C:\Users\John Coleman\Downloads\WinRAR2021.exe|890a58f200dfff23165df9e1b088e58f|

![image]()\
![image]()\
Kiểm tra mã hash của 2 file này trên Virus Total thì đều thuộc về Redvil Ransomware\
Đọc thêm tại link: https://www.secureworks.com/research/revil-sodinokibi-ransomware \
Có thể thấy Redvil được các kẻ tấn công giả mạo thành file WinRAR và sau đó thay thế bản WinRAR thật trên trang web WinRAR.it của Italian bằng file mã độc\
flag3:`WinRAR2021.exe`\
4.**What is the full URL that the user visited to download the malicious binary? (include the binary as well)**
Analysis Data > File Download History
![image]()\
>| Auto | http://192.168.75.129:4748/Documents/WinRAR2021.exe | C:\Users\John Coleman\Downloads\WinRAR2021.exe | 164 Kilobytes | 2021-08-02 19:42:16Z | Internet Explorer | John Coleman |

flag4:`http://192.168.75.129:4748/Documents/WinRAR2021.exe`\
5.**What is the MD5 hash of the binary?**
flag5:`890a58f200dfff23165df9e1b088e58f`\
6.**What is the size of the binary in kilobytes?**
flag6:`164`\
7.**What is the extension to which the user's files got renamed?**
>Theo Securework, Revil ransomware sẽ kiểm tra registry key trong \SOFTWARE\recfg để kiểm tra giá trị rnd_ext có tồn tại hay k (giá trị này là một extension ngẫu nhiễn đc tạo và gán cho các file mã hóa sinh ra trong quá trình chạy).
Nếu k có registry này, revil sẽ tự tạo một string ngẫu nhiên là tổ hợp của a-z 0-9 có từ 5-10 và sử dụng nó làm extension

Analysis Data > File System\
Có thể thấy trong Desktop có một số file có extension ngẫu nhiên `.t48s39la` và có thời gian được modified cuối giống nhau
![image]()\
flag7:`.t48s39la`\
8.**What is the number of files that got renamed and changed to that extension?**
>Timeline là một tool sử dụng để xác định thời điểm xảy ra các compromise, các file đã bị tác động, các compromise đang tiếp diễn
Timeline hiển thị event đc sắp xếp theo thời gian tăng dần và hỗ trợ áp dụng các filter để chỉ định user và process.
TimeWrinkle và TimeCrunch filter theo list, chỉ hiển thi các sự kiện đã xảy ra hoặc xảy ra gần 1 khoảng thời gian cụ thể
TimeWrinkle hiển thị các sự kiện trong 1 khoảng thời gian chỉ định
TimeCrunch loại bỏ các sự kiện nhiễu (chỉ định)

Analysis Data > Timeline\
Tìm các file được Modified và Change mà có chứa ext `.t48s39la`:
![image]()\
flag8:`48`\
9.**What is the full path to the wallpaper that got changed by an attacker, including the image name? **
>Theo Securework, sau khi mã hóa dữ liệu, REvil sẽ thay đổi background của máy nạn nhân bằng một bitmap image.
Image này sẽ được lưu với filename là bao gồm chữ thường và số từ 3-13 ký tự với ext .bmp. 
REvil sẽ gọi user32.dll SystemParamentersInfoW để thay đổi background.

Analysis Data > Timeline\
Tìm các file có ext `.bmp` được tại bởi user `John Coleman`:
![image]()\
file hk8.bmp là file background được REvil sử dụng\
flag9:`C:\Users\John Coleman\AppData\Local\Temp\hk8.bmp`\
10.**The attacker left a note for the user on the Desktop; provide the name of the note with the extension.**
>Theo Securework, một note sẽ được tạo dưới dạng file txt và có ext trong tên của nó\
REvil generates the ransom note's filename using a similar process. It obtains the value stored within the "nname" key in its configuration and replaces the {EXT} variable placeholder with its corresponding value. In the analyzed sample, the nname key value "{EXT}-HOW-TO-DECRYPT.txt" led to the ransom note filename 9781xsd4-HOW-TO-DECRYPT.txt.

Analysis Data > Timeline\
Tìm các file .txt có chứa ext `t48s39la`:
![image]()\
flag10:`t48s39la-readme.txt`\
11.**The attacker created a folder "Links for United States" under C:\Users\John Coleman\Favorites\ and left a file there. Provide the name of the file.**
Analysis Data > Timeline\
Filter theo tên folder `Links for United States` tìm kiếm với các 
