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
1. C
