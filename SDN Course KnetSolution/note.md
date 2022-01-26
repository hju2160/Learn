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
Có 2 cách để custom topo cho mininet\
Cách 1: Tạo một file custom topo và chạy với lệnh `mn`
Tạo một file `mn.py` như sau:
```python
from mininet.topo import Topo
from mininet.net import Mininet  
class MyTopo( Topo ):  
    "Simple topology example."
    def __init__( self ):
        "Create custom topo."

        # Initialize topology
        Topo.__init__( self )

        # Add hosts and switches
        Host1 = self.addHost( 'h1' )
        Host2 = self.addHost( 'h2' )
        Host3 = self.addHost( 'h3' )
        Host4 = self.addHost( 'h4' )

        sw1 = self.addSwitch('s1')
        sw2 = self.addSwitch('s2')
        sw3 = self.addSwitch('s3')
        # Add links
        self.addLink( sw1, sw2)
        self.addLink( sw1, sw3)

        self.addLink( Host1, sw3 )
        self.addLink( Host2, sw3 )
        self.addLink( Host3, sw2 )
        self.addLink( Host4, sw2 )
topos = { 'tp': ( lambda: MyTopo() ) } 
```
Chạy RYU Controller:
```
sudo mn --custom mn.py --topo tp --controller=remote,ip=127.0.0.1 --mac -i 10.1.1.0/24
```
(Lưu ý: chạy thẳng trong thư mục chứa file mn.py hoặc trích đường dẫn đầy đủ để chạy)\
Kiểm tra flows trong các SW trc khi ping, có thể thấu chỉ tồn tại 1 flow mặc định để gửi packet về Controller:\
![image](https://user-images.githubusercontent.com/95600382/150297307-e2d090ea-e98e-4759-8144-728358bdd57c.png)\
Kiểm tra flows sau khi ping:\
Sw1:\
![image](https://user-images.githubusercontent.com/95600382/150297547-b6efd24e-bc73-4207-b7a4-4ffd5d01c7f7.png)\
Sw2:\
![image](https://user-images.githubusercontent.com/95600382/150297600-24508211-f679-4863-8eb7-3dbe5134a766.png)\
Sw3:\
![image](https://user-images.githubusercontent.com/95600382/150297663-1fde8de2-862a-4f80-8743-a90811cfc4d1.png)\
Kiểm tra thêm trong mininet với `dump` và `net`:\
![image](https://user-images.githubusercontent.com/95600382/150297956-855b1069-e972-4eb5-a4aa-068504fe9bf7.png)\

Cách 2: Tạo một scripts python và chạy không cần lệnh `mn`\
(Tham khảo: https://github.com/mininet/mininet/blob/master/examples/README.md)\
Python Script:
```python
#!/usr/bin/env python
from mininet.net import Mininet
from mininet.node import RemoteController
from mininet.cli import CLI
from mininet.log import setLogLevel, info
import time

def emptyNet():

    "Create an empty network and add nodes to it."

    net = Mininet( controller=RemoteController )

    info( '*** Adding remote controller\n' )
    net.addController( 'c0' , controller=RemoteController,ip="127.0.0.1",port=6653)

    info( '*** Adding hosts\n' )
    h1 = net.addHost( 'h1', ip='10.0.0.1' )
    h2 = net.addHost( 'h2', ip='10.0.0.2' )

    info( '*** Adding switch\n' )
    s3 = net.addSwitch( 's3' )

    info( '*** Creating links\n' )
    net.addLink( h1, s3 )
    net.addLink( h2, s3 )

    info( '*** Starting network\n')
    net.start()

    info( '*** Running CLI\n' )
    CLI( net )

    info( '*** Stopping network' )
    net.stop()


if __name__ == '__main__':
    setLogLevel( 'info' )
    emptyNet()
```
Sau đó chạy bằng lệnh `sudo python3`. Kết quả:
![image](https://user-images.githubusercontent.com/95600382/150300607-0615174b-1ec9-425f-9493-19b04f08b7c6.png)\
![image](https://user-images.githubusercontent.com/95600382/150300681-2d8aad9c-6a4e-47fc-b088-4ab71f553b4e.png)\
Một ví dụ khác:
```python
#!/usr/bin/env python
from mininet.net import Mininet
from mininet.node import RemoteController
from mininet.cli import CLI
from mininet.log import setLogLevel, info
import time

def emptyNet():

    "Create an empty network and add nodes to it."

    net = Mininet( controller=RemoteController )

    info( '*** Adding remote controller\n' )
    net.addController( 'c0' , controller=RemoteController,ip="127.0.0.1",port=6653)

    info( '*** Adding hosts\n' )
    h1 = net.addHost( 'h1' )
    h2 = net.addHost( 'h2' )
    h3 = net.addHost( 'h3' )
	sv1 = net.addHost( 'sv1' )
	sv2 = net.addHost( 'sv2' )
    info( '*** Adding switch\n' )
	s1 = net.addSwitch( 's1' )
	s2 = net.addSwitch( 's2' )
    s3 = net.addSwitch( 's3' )

    info( '*** Creating links\n' )
    net.addLink( h1, s2 )
    net.addLink( h2, s2 )
	net.addLink( h3, s2 )
	net.addLink( sv1, s3 )
    net.addLink( sv2, s3 )
	net.addLink( s1, s2 )
	net.addLink( s1, s3 )
	
    info( '*** Starting network\n')
    net.start()

    info( '*** Running CLI\n' )
    CLI( net )

    info( '*** Stopping network' )
    net.stop()


if __name__ == '__main__':
    setLogLevel( 'info' )
    emptyNet()
```
<h2>Openflow Theory</h2>

<h3>Introduction</h3>

Giao thức OpenFlow sử dụng để quản lý các OpenFlow SW từ remote OpenFlow Controller/
Các version của OpenFlow: 1.1,1.2,1.3,1.4,1.5\
Trong đó sử dụng rộng rãi nhất là 1.3\

<h3>Switch Components</h3>

Các phần logic của một OpenFlow SW bao gồm một hoặc nhiều các flow tables và các table group. Các bảng này được sử dụng trong quá trình lookup và forward packet và một hoặc nhiều các OpenFlow Channel kết nối tới các Controller\
![image](https://user-images.githubusercontent.com/95600382/150316885-e723f892-0707-4471-ad92-968489c7a5b5.png)\
- Các giao tiếp giữa SW với Controller và Controller quản lý SW được thực hiện thông qua giao thức OpenFlow
- Thông qua giao thức OpenFlow, Controller có thể add, update, delete flow trong flow table, theo cả 2 cách chủ động (response packet) và bị động.
- Mỗi flow table sẽ bao gồm 1 tập các flow entry; mỗi flow entry sẽ bao gồm match field, counter và một tập hợp các thông để match gói
- Việc match packet sẽ được so khớp từ các flow table đầu tiên
- Các flow entries sẽ được match với packet theo priority và flow đầu tiên match trong tất cả các bảng sẽ được sử dụng
- Khi tìm thấy flow match, các chỉ dẫn có trong flow đó sẽ được áp dụng cho packet
- Khi packet không match với flow nào, đường đi sẽ phụ thuộc vào cấu hình của table-miss flow entry: VD, packet có thể được forward tới Controller thông qua OpenFlow channel, drop hoặc xét các flow table tiếp theo.
- Các action sẽ được mô tả trong flow entry
- Flow entry có thể forward tới các port trên SW, thường là port vật lý và có thể là các port Logic được định nghĩa trên SW (link aggregation groups, tunnels or loopback interfaces).

<h3>OpenFlow Channel</h3>

- Là một interface kết nỗi các OpenFlow Logic SW tới Openflow Controller. Thông qua interface này, Controller có thể cấu hình và quản lý Switch, nhận event từ SW và điều hướng các packet cho SW
- Control Channel của SW hỗ trợ cả single channel và multiple channel tới Controller
- Các OpenFlow Channel thường được mã hóa với TLS hoặc có thể chạy trực tiếp trên TCP
- Default port: 6653

<h3>OpenFlow Table</h3>

Các Flow table entry được xác định bởi các match field và priority: 2 giá trị này sẽ đi với nhau để quyết định một flow entry duy nhất trong flow table.\
Một flow table bao gồm các flow entry\
![image](https://user-images.githubusercontent.com/95600382/150763851-581f9479-c81c-4f5c-934c-78b8f2b5a2fa.png)\
Trong đó:\
- match fields: so khớp với packet. Bao gồm các cổng và packet header, hoặc các trường được chỉ định trong flow table
- priority: độ ưu tiên của flow entry
- counters: update khi có packet match
- intructions: để sửa đổi tập action hoặc pipeline processing
- timeouts: Khoảng thời gian tối đa hoặc idle time trước khi flow hết hạn
- cookie: giá trị được chọn bởi Controller. Có thể được controller sử dụng để lọc các flow entry bị ảnh hưởng bởi các flow statistics, flow modification và flow deletion requests. k sử dụng để sử lý packet.
- flags: thông báo (VD:  flag OFPFF_SEND_FLOW_REM triggers flow removed messages for that flow entry.)

Một flow entry mà có tất cả match fields là * và priority=0 thì gọi là table-miss flow entry (Flow sử dụng để gửi packet đến Controller)
Ví dụ flow table khi so khớp với MAC:
```
 cookie=0x0, duration=4.742s, table=0, n_packets=2, n_bytes=196, priority=1,in_port="s1-eth2",dl_src=00:00:00:00:11:12,dl_dst=00:00:00:00:11:11 actions=output:"s1-eth1"\
 cookie=0x0, duration=4.738s, table=0, n_packets=1, n_bytes=98, priority=1,in_port="s1-eth1",dl_src=00:00:00:00:11:11,dl_dst=00:00:00:00:11:12 actions=output:"s1-eth2"\
 cookie=0x0, duration=5.781s, table=0, n_packets=29, n_bytes=3102, priority=0 actions=CONTROLLER:65535
```
Ví dụ flow table khi so khớp với IP:
```
 cookie=0x0, duration=12.927s, table=0, n_packets=2, n_bytes=196, priority=1,ip,nw_src=192.168.1.1,nw_dst=192.168.1.2 actions=output:"s1-eth2"
 cookie=0x0, duration=12.918s, table=0, n_packets=2, n_bytes=196, priority=1,ip,nw_src=192.168.1.2,nw_dst=192.168.1.1 actions=output:"s1-eth1"
 cookie=0x0, duration=12.959s, table=0, n_packets=37, n_bytes=3844, priority=0 actions=CONTROLLER:65535
```
Ví dụ flow table khi so khớp với Port:
```
 cookie=0x0, duration=3.933s, table=0, n_packets=238752, n_bytes=11069190464, priority=1,tcp,nw_src=192.168.1.2,nw_dst=192.168.1.1,tp_src=37304,tp_dst=5001 actions=output:"s1-eth1"
 cookie=0x0, duration=3.906s, table=0, n_packets=192421, n_bytes=12699810, priority=1,tcp,nw_src=192.168.1.1,nw_dst=192.168.1.2,tp_src=5001,tp_dst=37304 actions=output:"s1-eth2"
 cookie=0x0, duration=31.495s, table=0, n_packets=43, n_bytes=4309, priority=0 actions=CONTROLLER:65535
suresh@suresh-vm:~$
```

<h3>OpenFlow Matching</h3>

Khi nhận được một gói tin, một OpenFlow Sw sẽ thực hiện các bước sau đây:\
![image](https://user-images.githubusercontent.com/95600382/151138786-90d2e117-e350-44bb-94f7-25d7dfdc5d58.png)
####Match Fields
```
Switch input port. */
Switch physical input port. *
Ethernet destination address. */
Ethernet source address. */
Ethernet frame type. */
VLAN id. */
VLAN priority. */
IP DSCP (6 bits in ToS field). */
IP ECN (2 bits in ToS field). */
IP protocol. */
IPv4 source address. */
IPv4 destination address. */
TCP source port. */
TCP destination port. */
UDP source port. */
UDP destination port. */
SCTP source port. */
SCTP destination port. */
ICMP type. */
ICMP code. */
ARP opcode. */
ARP source IPv4 address. */
ARP target IPv4 address. */
ARP source hardware address. */
ARP target hardware address. */
IPv6 source address. */
IPv6 destination address. */
IPv6 Flow Label */
ICMPv6 type. */
ICMPv6 code. */
Target address for ND.
Source link-layer for ND. */
Target link-layer for ND. */
MPLS label. */
MPLS TC. */
MPLS BoS bit. */
PBB I-SID. */
Logical Port Metadata. */
IPv6 Extension Header pseudo-field */
```
- Match field có 2 loại, header match field hoặc pipeline match field
- Header match field là các giá trị được lấy từ packet header. 
- All header match fields have different size, prerequisites and masking capability
- Pipeline match field là các giá trị đc đính kèm trong packet cho quá trình xử lý pipeline và không liên quan đến packet header như META_DATA, TUNNEL_ID,...

Prerequisties Example:\
- Nếu muốn so khớp theo SRC IP, điều kiện bắt buộc ch ETH_TYPE sẽ là 0X0800. Có nghĩa là sẽ cần trường ETH_TYPE match cùng.

Instructions:
- Mỗi một flow entry sẽ bao gồm một set các chỉ dẫn mà được áp dụng cho các packet match entry. Các chỉ dẫn này là kết quả của sự thay đổi trong packet, action set hoặc pipeline processing

Apply-Actions action(s):
- Apply action ngay lập tức mà không cần thay đổi Action Set
- Instruction này có thể được sử dụng modify the packet between two tables or to execute multiple actions of the same type.
- The actions are specified as a list of actions

.... đọc thêm trong docx\

Action Set và Action:\
VD các action:\
Output port no\
Group group id\
Drop\
Set-Queue queue id\
Push-Tag/Pop-Tag ethertype (VLAN, MPLS, PBP)\
Set-Field field type value\
Change-TTL (IP TTL, MPLS TTL)\

#### Flow Removal\
Flow entries are removed from flow tables in two ways, either at the request of the controller or via the switch flow expiry mechanism.\

### Openflow Messages



