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



