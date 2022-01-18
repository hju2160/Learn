<h2>Một ví dụ đơn giản dễ hiểu về SDN</h2>

Trong các Sw L2 truyền thống, chúng sẽ sử dụng giao thức ARP để quyết định đường đi cho các flow.\
![image](https://user-images.githubusercontent.com/95600382/149897343-b9155f28-c60c-4275-a2f6-22b10b391ed6.png)
Trong đó:\
**Control Plane**\
Học các địa chỉ MAC từ các packet và lưu chúng vào trong MAC table\
**Data Plane**
- Nhận các packet
- Đọc các địa chỉ MAC từ packet
- Lookup tới MAC table để quyết định out port
- Forward packet ra out port
Openvswitch là một softswitch và có thể hoạt động như một switch truyền thống hoặc Openflow switch sử dụng trong SDN\
Các câu lệnh thường được sử dụng cho OVS switch
```console
sudo ovs-vsctl   --version
sudo ovs-vsctl show
sudo ovs-appctl fdb/show s1
sudo ovs-dpctl dump-flows
```
Cơ chế hoạt động:
1. Một switch truyền thống được xây dựng kết hợp cả Control Plane và Data Plane
2. Mac table (Forwarding table) trống khi switch khởi động
3. Control Plane sẽ update MAC Table với địa chỉ MAC và PORT Number. Các thông tin này sẽ được thu thập từ các packet gửi đến sw
4. Control PLane sẽ duy trì build/update Mac table.
5. Khi một packet đến, Data plane sẽ tham chiếu với Mac Table, nếu MAC đích khớp trong MAC table thì packet sẽ được forward ra port tương ứng
<h2>Openflow Switch</h2>

1. Khởi động Mininet
```console
sudo mn --controller=remote,ip=127.0.0.1 --mac -i 10.1.1.0/24 --switch=ovsk,protocols=OpenFlow13 --topo=linear,4
```
Kết quả:
![image](https://user-images.githubusercontent.com/95600382/149899458-20d4aabb-e473-46ff-a7f6-7ccd5c5b0d71.png)\
Thử sử dụng lệnh pingall để test kết nối:\
![image](https://user-images.githubusercontent.com/95600382/149899791-bb4331b0-2c3b-4dfe-a9db-e9e0852e6de0.png)\
Có thể thấy khi ryu app không được khởi động, mininet sẽ kết nối tới Controller mặc định của nó. Lý do không ping được lẫn nhau
là do Controller mặc định của Miniet không có flow nào:
![image](https://user-images.githubusercontent.com/95600382/149900503-a0d46ad5-c2ad-4dc3-a38b-16d45e7a9f01.png)\
2. Khởi động Wireshark bắt gói tin ở card l0 hoặc any
```console
sudo wireshark
```
Giao diện sau khi khởi động:
![image](https://user-images.githubusercontent.com/95600382/149900770-ce5cd580-1dff-44d1-bf02-fa5cafee23c5.png)\
Thực hiện bắt ở card any\
3. Chạy Ryu app
```console
ryu-manager ryu.app.simple_switch_13
```
![image](https://user-images.githubusercontent.com/95600382/149901000-7e3f42fc-795d-43ff-bbc9-4bc3ae088a71.png)\
4. Ở giao diện cli mininet, thực hiện h1 ping h2
```console
mininet>h1 ping h2
```
Kết quả: Có thể thấy đã ping thành công
![image](https://user-images.githubusercontent.com/95600382/149901210-9e4dbfed-46c1-41e6-9bfc-0bd5618bae9f.png)\
5. Kiểm tra OpenFlow flow
```console
sudo ovs-ofctl -O OpenFlow13 dump-flows s1
```
Kết quả:
![image](https://user-images.githubusercontent.com/95600382/149901517-2cb3f560-656a-45b0-8f0e-87f3b2aea1be.png)\
**Cơ chế hoạt động**
1. Các SW sẽ được config IP và Openflow version của SDN Controller
2. SW xác lập kết nối với SDN Controller
3. SDN Controller sẽ đặt Openflow rule mặc định (TABLE MISS ENTRY) cho SW Flow table
4. TABLE MISS ENTRY Openflow sẽ match với tất cả packet và gửi chúng đến CONTROLLER. Priority là thấp nhất (0)
![image](https://user-images.githubusercontent.com/95600382/149902280-d3eeb8b6-7be9-4d90-8e01-12eab2f9cf84.png)\
5. Khi một packet được gửi từ một host đến Switch, nó sẽ match với TME và được gửi tới Controller (PACKET IN Message)
6. Controller nhận packet và xây dựng SW Logic với các thông tin từ packets
7. Controller thêm các OPENFLOW flows tới SW
8. SW data path giờ đã được build với các flows. Các gói tin kế tiếp sẽ dựa vào Flow table này để quyết định đường đi của mình.
![image](https://user-images.githubusercontent.com/95600382/149903125-d1886d5c-b621-4787-bd5b-dc52494e0ae0.png)\

<h2>Mininet</h2>

<h3>Basic Operations</h3>

1. Kiểm tra version
```console
mn --verison
```
![image](https://user-images.githubusercontent.com/95600382/149904970-abfe992f-7aa5-49bb-a693-d0eb14209233.png)\
2. Clean mininet\
Note: Đôi khi Mininet có thể bị tắt sai hoặc crash, và topo vẫn còn tồn tại. Lệnh này để làm sạch mininet cho các lần chạy mới.
```console
mn -c
```
![image](https://user-images.githubusercontent.com/95600382/149905120-2d29d2c9-d2ff-4c00-b385-a148cb98a596.png)\
3. Single Topo\
Topo này có 1 SW và 4 Host:
![image](https://user-images.githubusercontent.com/95600382/149905394-9f27c161-aa2d-405a-86df-6a26e71db50a.png)\
RYU SDN Controller
```console
ryu-manager ryu.app.simple_switch_13
```
![image](https://user-images.githubusercontent.com/95600382/149906060-fc5e528d-beda-4916-a73b-4b5e15764c1a.png)\
Mininet Topo
```console
sudo mn --controller=remote,ip=127.0.0.1 --mac -i 10.1.1.0/24 --switch=ovsk,protocols=OpenFlow13 --topo=single,4
```
![image](https://user-images.githubusercontent.com/95600382/149906337-aa7e87fd-1025-421f-8aa9-34d571245e28.png)\
|Options|Des|
|---|---|
|--controller|Loại controller local/remote và remote controller ip|
|--mac|Địa chỉ MAC sẽ được đặt bắt đầu từ 00:00:00:00:00:01|
|-i|IP Subnet của Topo|
|--switch|Loại SW (ovsk - openvswitch kernel module) và openflow version|
|--topo|Loại topo và các thông số|

Khi sử dụng xong, sử dụng lệnh `exit` để thoát khỏi mininet shell\
![image](https://user-images.githubusercontent.com/95600382/149907198-a3488846-0a81-4898-82f7-bb85bb852437.png)\
4. Mininet Basic Shell Commands
Các lệnh thường dùng
```
help
dump
net
links
pingall
```
Để thực hiện các lệnh với các host được chỉ định:\
C1: Sử dụng `xterm`
```console
mininet>xterm h1
```
C2: Sử dụng thông qua mininet shell
```console
mininet><hostname> command
```
VD:
```console
mininet>h1 ifconfig
mininet>h1 ping h2
mininet>h1 ip route
```
![image](https://user-images.githubusercontent.com/95600382/149908169-8b6b81d6-005c-4670-adf7-838963847b0d.png)\
5. Line Topo
Topo:
![image](https://user-images.githubusercontent.com/95600382/149908376-70b3d57f-e20f-4ee6-b935-1ff53086e5a1.png)\
```console
sudo mn --controller=remote,ip=127.0.0.1 --mac -i 10.1.1.0/24 --switch=ovsk,protocols=OpenFlow13 --topo=linear,4
```
Trong đó 4 là số SW trong line
![image](https://user-images.githubusercontent.com/95600382/149908847-2b8e4b71-eb2c-4c08-8a0f-341f5e739f7e.png)\
6. Tree Topo
Topo:
![image](https://user-images.githubusercontent.com/95600382/149908939-48ee3e2d-5361-4797-b6c0-d60194615fb6.png)
```console
sudo mn --controller=remote,ip=127.0.0.1 --mac -i 10.1.1.0/24 --topo=tree,depth=2,fanout=3
```
Trong đó, `fanout` là số SW được connect vào gốc và `depth` là số host mỗi SW
![image](https://user-images.githubusercontent.com/95600382/149909321-a542ebe2-6241-4aee-b926-7290e278027f.png)

<h3>Running Traffic Tests</h3>

1. TCP/UDP Traffic Tests\
a. Setup Topo: Sử dụng topo tree ở trên\
b. TCP Traffic test\
Setup IPERF TCP Server ở h4
```
mininet> xterm h4
ipref -s
```
![image](https://user-images.githubusercontent.com/95600382/149911172-a8698673-ea80-4b48-8834-ad6640f90de2.png)\
Chạy IPERF TCP Client trên h1:
```
iperf -c 10.1.1.4 -i 10 -b 10m -t 30
iperf -c 10.1.1.4 -i 10 -P 10 -t 30
```
-c: Client mode\
-i: Khoảng thời gian báo cáo (10s/1 lần)\
-t: Test trong khoảng 30s\
-b: Bandwith 10m ~ 10 Mbps\
-P: Số kết nối song song\
Kết quả:
![image](https://user-images.githubusercontent.com/95600382/149911870-ed01246b-ac56-423d-b8c1-9541c68536a1.png)\
![image](https://user-images.githubusercontent.com/95600382/149912134-5a12e5a3-19a2-4672-bc79-0cbf8c27986f.png)\
c. UDP Traffic\
Setup IPERF UCP Server ở h4:\
```
iperf -u -s
```
-u: udp
![image](https://user-images.githubusercontent.com/95600382/149913659-4ea6063c-0c9e-497d-9931-235868f8f4ac.png)\

Chạy IPERF UCP Client trên h1:
```
iperf -u -c 10.1.1.4 -i 10 -b 10m -t 30
iperf -u -c 10.1.1.4 -i 10 -b 10m -P 10 -t 30
```
Kết quả:
![image](https://user-images.githubusercontent.com/95600382/149913629-198bc358-1637-4b92-8c05-20ac86ea98be.png)\
![image](https://user-images.githubusercontent.com/95600382/149913793-f8aa47b6-9244-486a-9763-386ad88a9c9c.png)\

2. HTTP Traffic Tests\
Chạy một Web server đơn giản với python trên h4
```
python3 -m http.server 80
```
![image](https://user-images.githubusercontent.com/95600382/149914647-710b841c-3e0e-4480-8a8e-1beb4a631e79.png)\

Từ h1 truy cập đến Webserver
```
curl http://10.1.1.4/
```
![image](https://user-images.githubusercontent.com/95600382/149914709-3bc9227d-cb11-4981-9d54-8eca110ded85.png)
Có thể giả lập truy cập của 1000 user bới ab(apache bench):
```
ab -n 500 -c 50 http://10.1.1.4/
```
-c: Số request mỗi giây\
-n: Số request\
Kết quả:\
Bên Client:\
![image](https://user-images.githubusercontent.com/95600382/149915253-343c5ee8-43ee-4d52-8740-ae126eb02350.png)\
Bên Server:\
![image](https://user-images.githubusercontent.com/95600382/149915372-9584e94f-9f25-4a55-91e0-b0e3a64dfb63.png)\

<h3>Writing Custom Topo in Mininet (Dang hoan thien)</h3> 

Mininet có API python, chúng ta có thể tạo các topo custom với API này
1. Import libary
```python
from mininet.topo import Topo
from mininet.net import Mininet
```
2. Định nghĩa các class trong topo
```python
class SingleSwitchTopo(Topo):
   def build(self):
       s1 = self.addSwitch('s1')

       h1 = self.addHost('h1')
       h2 = self.addHost('h2')
       h3 = self.addHost('h3')
       h4 = self.addHost('h4')  

       self.addLink(h1, s1)
       self.addLink(h2, s1)
       self.addLink(h3, s1)
       self.addLink(h4, s1)
```
Các APIs định nghĩa topo quan trọng
```
addSwitch
addHost
addLink
```
3. Định nghĩa Controller 
```python
if __name__ == '__main__':
    topo = SingleSwitchTopo()
    c1 = RemoteController('c1', ip='127.0.0.1')
    net = Mininet(topo=topo, controller=c1)
    net.start()
```
4. Chạy Test
- Start RYU SDN Controller
```
ryu-manager ryu.app.simple_switch_13
```
- Chạy Mininet topo
```

```
- Test
