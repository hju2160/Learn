<h2>ARP Traffic</h2>

Về cơ bản thì ARP là một protocol layer2 sử dụng để ánh xạ MAC add với IP add bao gồm 2 gói tin là Request và Response. Có thể xác định packet với code tương ứng:\
request (1)\
reply (2)\
Với gói ARP Request:\
![image](https://user-images.githubusercontent.com/95600382/154234439-cf6c5956-b030-447c-bc90-1dfba4f67473.png)\
Với gói ARP Reply:\
![image](https://user-images.githubusercontent.com/95600382/154234602-e16bb03a-fa95-4391-b79b-db81c1bdccf4.png)\
Answer:\
What is the Opcode for Packet 6?\
![image](https://user-images.githubusercontent.com/95600382/154235690-79ecc940-e80c-429b-8213-2e54d93dac92.png)\
`request (1)`\
What is the source MAC Address of Packet 19?\
![image](https://user-images.githubusercontent.com/95600382/154235811-85e4acd9-0c42-439a-b1eb-33241d24b99a.png)\
`80:fb:06:f0:45:d7`\
What 4 packets are Reply packets?\
filter:`arp.opcode == 2`\
![image](https://user-images.githubusercontent.com/95600382/154235968-1d8d6c2d-943b-4c35-840a-b315ee645c2a.png)\
`76,400,459,520`\
What IP Address is at 80:fb:06:f0:45:d7?\
![image](https://user-images.githubusercontent.com/95600382/154235811-85e4acd9-0c42-439a-b1eb-33241d24b99a.png)\
`10.251.23.1`\

<h2>ICMP Traffic</h2>

ICMP là giao thức thường được sử dụng bởi các tính năng như ping, traceroute.\
ICMP của ping:\
![image](https://user-images.githubusercontent.com/95600382/154240058-59e89b10-acaa-4c8c-9055-1dd31a641d49.png)\
**ICMP request:**\ 
Thông tin quan trọng trong gói tin này sẽ là type và code của packet. type=8 -> đây là một request packet, type=0 là reply packet. Có thể dựa vào tham số type để phát hiện các hoạt động đáng ngờ.\
2 thông tin hữu ích trong packet mà có thể hỗ trợ trong quá trình điều tra: timestamp và data. Timestamp có thể sử dụng để xác định mốc thời gian của sự kiện. Data thường là một chuối ngẫu nhiên và cta có thể khai thác thông tin từ nó trong một số trường hợp.\
![image](https://user-images.githubusercontent.com/95600382/154240910-9c85594b-0e09-425e-9d8f-986c4bdc0365.png)\
**ICMP reply:**\ 
Tương tự như ICMP request packet.\ 
![image](https://user-images.githubusercontent.com/95600382/154243588-c84118fe-d635-4830-a312-a37d837bf56b.png)\
Answer:\
What is the type for packet 4?\
`8`\
What is the type for packet 5?\
`0`\
What is the timestamp for packet 12, only including month day and year?\
note: Wireshark bases it’s time off of your devices time zone, if your answer is wrong try one day more or less.\
`May 30, 2013`\
What is the full data string for packet 18?\
`08090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f202122232425262728292a2b2c2d2e2f3031323334353637`\

<h2>TCP Traffic</h2>

TCP là giao thức chịu trách nhiệm vận chuyển các packet bao gồm cả các sequence và errors.\
Sau đây là ví dụ về việc sử dụng nmap scan port 80 và port 443. Có thể thấy port đang đóng với cờ RST và gói tin ACK là màu đỏ\
![image](https://user-images.githubusercontent.com/95600382/154245975-d1f7607b-2cba-45c3-9932-bdaa3d9790be.png)\
Khi phân tích các gói TCP, color code của wireshark sẽ giúp ích rất nhiều trong việc phân loại mức độ nguy hiểm của packet.\
TCP có thể giúp nhà phân tích có cái nhìn sâu vào luồng network nhưng lại khó phân tích vì sô lượng quá lớn. Lúc này cta cần các tools như RSA NetWithness và NetworkMiner để hỗ trợ phân tích.\
**TCP Traffic Overview**\
Thứ b sẽ thường xuyên thấy trong quá trình phân tích TCP và TCP handshake, bao gồm 1 series các gói: syn, syn-ack, ack để etablish kết nối\
![image](https://user-images.githubusercontent.com/95600382/154247156-6fd7ba22-b61c-407d-98cc-b93f26051b76.png)\
Quá trình bắt tay TCP sẽ là k đáng tin khi nó bao gồm các cờ khác như RST như nmap là một ví dụ\
**TCP Packet Analysis**\
Để phân tích TCP, ta sẽ k đi vào chi tiết của từng gói mà sẽ nhìn vào hành vi và cấu trúc mà chúng có.\
VD sau là chi tiết về một SYN packet, thứ cần lưu ý ở đây là seq number và ack number:\
![image](https://user-images.githubusercontent.com/95600382/154247870-c6b297df-2163-42c2-89c8-b235686ad633.png)\
Ở trường hợp này, port đang đóng và ack number = 0\
Within Wireshark, we can also see the original sequence number by navigating to edit > preferences > protocols > TCP > relative sequence numbers (uncheck boxes).\
![image](https://user-images.githubusercontent.com/95600382/154248034-f01ea4f6-0d5e-4eb4-b653-2080d7e63b7a.png)\
![image](https://user-images.githubusercontent.com/95600382/154248140-477e5921-d3e5-4c36-a5f1-4e425d7020a6.png)\

<h2>DNS Traffic</h2>

DNS là giao thức sử dụng để phân giải tên miền thành IP add.\
https://www.ietf.org/rfc/rfc1035.txt\
Một số lưu ý khi thực hiện phân tích các DNS packet:\
Query-Response\
DNS-Servers Only\
UDP\
Nếu bất kỳ yếu tố nào ở trên có chỗ không đúng thì có khả năng hoạt đông đáng ngờ đang diễn ra.\
vd:\
![image](https://user-images.githubusercontent.com/95600382/154424789-a6435b8b-5f9f-48ff-b7ae-56d916a79bf9.png)\
**DNS Traffic**\
DNS Query: VD sau cho biết một số các thông tin có thể sử dụng để phân tích packet. Đầu tiên là UDP 53 (Mặc định của DNS) nếu là TCP 53 thì đây có thể là lưu lượng độc hại và cần phân tích sâu hơn. \
![image](https://user-images.githubusercontent.com/95600382/154425362-d257f9fa-6291-4648-b21a-a7871ed72c78.png)\
Khi phân tích DNS cần thật sự hiểu môi trường để có thể phát hiện ra điểm bất thường trong lưu lượng DNS\
DNS Response:\
Tương tự như DNS Query:\
![image](https://user-images.githubusercontent.com/95600382/154425562-fc7ace6e-8001-4a4e-ad87-8061704d8708.png)\
**Answer**\
What is being queried in packet 1?\
`8.8.8.8.in-addr.arpa`\
What site is being queried in packet 26?\
`www.wireshark.org`\
What is the Transaction ID for packet 26?\
`0x2c58`\

<h2>HTTP Traffic</h2>

https://www.ietf.org/rfc/rfc2616.txt\
**Answer**\
What percent of packets originate from Domain Name System?\
`4.7`\
What endpoint ends in .237?\
`145.254.160.237`\
What is the user-agent listed in packet 4?\
`Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.6) Gecko/20040113`\
Looking at the data stream what is the full request URI from packet 18?\
`http://pagead2.googlesyndication.com/pagead/ads?client=ca-pub-2309191948673629&random=1084443430285&lmt=1082467020&format=468x60_as&output=html&url=http%3A%2F%2Fwww.ethereal.com%2Fdownload.html&color_bg=FFFFFF&color_text=333333&color_link=000000&color_url=666633&color_border=666633`\
What domain name was requested from packet 38?\
`www.ethereal.com`\
Looking at the data stream what is the full request URI from packet 38?\
`http://www.ethereal.com/download.html`\

<h2>HTTPS Traffic</h2>

**HTTPS Traffic**\
Các bước mà client và server cần thực hiện để tạo một tunnel được mã hóa:\
Client và Server thống nhất protocol version\
Client và Server lựa chọn thuật toán mã hóa\
Client và Server có thể xác thực lẫn nhau (optional)\
Tạo một secure tunnel với pub key\
Chúng ta có thể bắt phân tích HTTPS bằng gói handshake giữa client và server. Vd gói Client Hello cho biết SSLv2 Record Layer, Handshake Type, SSL Version\
![image](https://user-images.githubusercontent.com/95600382/154428872-156c6dc6-e7ca-4dce-b53d-05446c5f7ff8.png)\
VD dưới đây cũng tương tự nhưng bao gồm thêm thông tin về SSL cert\
![image](https://user-images.githubusercontent.com/95600382/154428985-a426a48e-12b5-49d0-8454-dd4c020c0f73.png)\
Tiếp đó là Client Key Exchange packet, ở đây sẽ quyest định public key sẽ được sử dụng để mã hóa các packet về sau.\
![image](https://user-images.githubusercontent.com/95600382/154429148-e0e015fc-f9c4-42e5-ac7f-0e6e4b81e5f5.png)\
Packet tiếp theo, server sẽ xác nhận pub key và thiết lập secure tunnel.\
![image](https://user-images.githubusercontent.com/95600382/154429262-4c15e765-b177-4b92-8bb3-8d9850aa91f7.png)\
Data trao đổi giữa Client và Server sau đó sẽ đc mã hóa và chúng ra cần secret key để giải mã.\
![image](https://user-images.githubusercontent.com/95600382/154429387-fb4fa3c3-c121-4455-87c0-6de18fc96b8c.png)\
**Answer**\
Edit > Preferences > Protocols > TLS >  [+] để add key giải mã dữ liệu\
Looking at the data stream what is the full request URI for packet 31?\
`https://localhost/icons/apache_pb.png`\
Looking at the data stream what is the full request URI for packet 50?\
`https://localhost/icons/back.gif`\
What is the User-Agent listed in packet 50?\
`Mozilla/5.0 (X11; U; Linux i686; fr; rv:1.8.0.2) Gecko/20060308 Firefox/1.5.0.2`\

<h2>Expoit PCAPs</h2>

Zerologon PCAP Overview
Phân tích PCAP file về tấn công khai thác CVE-2020-1472 vào Windows AD Exploit gọi là Zerologon. server có ip 192.168.100.6 và attacker 192.168.100.128. Các step sau có thể đc ứng dụng để phân tích pcap và đưa ra giả thuyết về sự kiện đã xảy ra\
![image](https://user-images.githubusercontent.com/95600382/154432351-295fffc4-7528-4e90-8b2c-ff773f8e0604.png)\
1. Xác định Attacker
Nhìn vào gói tin PCAP, ta trc tiên có thể xác định được các traffic thông thường như OpenVPN, ARP, ... bên cạnh đó cũng có một số protocol lạ như DCERPC, EPM,...
![image](https://user-images.githubusercontent.com/95600382/154434955-e466dc96-9708-4b0b-b42e-fbb5958bf849.png)\
Có thể thấy 2 ip giao tiếp nhiều nhất trong mạng là 192.168.100.6 và 192.168.100.128. Trong đó các request đều được gửi từ 192.168.100.128.\
![image](https://user-images.githubusercontent.com/95600382/154435402-1818a7d0-ce74-439e-9747-253cdde8e857.png)\
![image](https://user-images.githubusercontent.com/95600382/154435515-bdd7456a-c065-46a4-be9b-93a0cf917496.png)\
2. Zerologon POC Connection Analysis
Tiếp đó sử dụng filter có `ip.src=192.168.100.128` để lọc và bắt đầu phân tích gói tin. Nên kết hợp với IOCs đi kèm với lỗ hổng để phân tích. Ở zerologon này, có thể thấy kẻ tấn công sử dụng nhiều kêt nối RPC và sử dụng DCERPC request để thay đổi mật khẩu\
![image](https://user-images.githubusercontent.com/95600382/154439877-39a6d100-8042-49ed-b01a-92ba3a586a75.png)\
3. Secretsdump SMB Analysis
![image](https://user-images.githubusercontent.com/95600382/154439911-3f1cc152-8b60-4e49-b73c-04b852e8085b.png)\
Looking further at the PCAP we can see SMB2/3 traffic and DRSUAPI traffic, again with prior knowledge of the attack we know that it uses secretsdump to dump hashes. Secretsdump abuses SMB2/3 and DRSUAPI to do this, so we can assume that this traffic is secretsdump.\
Each exploit and attack will come with its unique artifacts, in this case, it is clear what happened and the order of events that occurred. Once we have identified the attacker we would need to move on to other steps to identify and isolate as well as report the incident if we were on a Threat Hunting or DFIR team.\











