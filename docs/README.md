Tổng quan về High Availability Cluster

### 1. Các thành phần cấu thành HA Cluster

Đây là các thành phần chủ yếu được sử dụng trong hầu hết các Cluster:
 - Shared Storage
 - Different Network
 - Bonded Network Devices
 - Multipathing
 - Fencing/STONISH devices

#### 1.1. Shared Storage

Có 2 hướng để triển khai Shared Storage, sử dụng NFS hoặc SAN.

#### 1.2 Different Network

![Different_network](/images/HACluster_1.jpg)

Một cluster nên có nhiều kết nối mạng.
 - Đường mạng cho user, từ đó user có thể truy xuất tới cluster resource.
 - Đường mạng cho cluster, cần đảm bảo dự phòng cho đường này.
 - Đường mạng cho storage, cấu hình dựa vào loại thiết bị lưu trữ sử dụng.

#### 1.3. Bonded Network Devices

Để kết nối cluster node tới các dải mạng, chỉ dùng 1 card mạng (NIC). Nếu NIC đó lỗi, node sẽ mất kết nối trên dải mạng đó. 
Giải pháp là sử dụng network bonding. 1 network bond là một nhóm nhiều NIC. Thông thường, có 2 NIC trong 1 bond. Mục đích của bonding là dự phòng: đảm bảo nếu 1 NIC lỗi, NIC còn lại sẽ đảm bảo kết nối của node.

#### 1.4. Multipathing

Cluster node kết nối tới SAN qua nhiều đường, do đó 1 node sẽ nhận nhiều thiết bị lưu trữ cho 1 LUN.

Khi cấu hình một node kết nối tới 2 SAN Switch, các switch đó sẽ kết nối tới 2 SAN controller, do đó sẽ 4 đường khác nhau. Do vậy, node sẽ nhận 4 ổ cứng, thay vì 1. Dùng Multipath để có thể sử dụng cả 4 đường này cùng 1 lúc.

Multipath driver sẽ gộp 4 disk đó, và tạo ra 1 disk ảo, có tên là mpatha. Administrator sẽ sử dụng disk mpatha đó để lưu trữ, và khi 1 đường hỏng, multipath sẽ route traffice qua các NIC khác.

#### 1.5. Fencing/STONISH Device và Quorum

Trong Cluster, cần loại bổ hiện tượng "split brain". Split brain là lúc cluster bị chia ra làm 2 hoặc nhiều phần, nhưng cả hai phần đều nghĩ mình là phần duy nhất còn lại của cluster. Điều này dẫn tới một tình huống rất xấu là cả 2 phần đều cố gắng chiếm quyền đối với tài nguyên của cluster. Nếu tài nguyên là file system, và nhiều node cố ghi file đồng thời mà không có coordination, nó có thể làm corrupt và mất dữ liệu.

Có 2 giải pháp để phòng tránh split brain.

- Quorum. Quorum tức là "majority", và ý tưởng đằng sau quorum rất đơn giản: nếu cluster không có quorum, sẽ không có hành động nào xảy ra trong cluster. 

- STONISH (shoot the other node in the head) hoặc Fence: Quorum là giải pháp tố để tránh các vấn đề đã nêu ở trên, nhưng để đảm bảo nó không bao giờ xảy ra khi nhiều node cùng sử dụng chung resource trên cluster, một cơ chế khác được ứng dụng, đó là STONISH hoặc fencing.

 Trong STONISH, phần cứng sẽ được chỉ định tắt khi node không còn trả lời cluster. Ý tưởng của STONISH là trước khi dời resource tới node khác, cluster phải đảm bảo rằng node lỗi đã thực sự down. Để thực hiện điều này, cluster sẽ gửi lệnh `shutdown` tới STONISH device để tắt node tương ứng. Cách này đảm bảo không có data corruption.

 Khi cài đặt cluster, cần quyết định loại STONISH device nào sẽ dùng:

 * Trình quản lý tích hợp trong thiết bị, như HP ILO, Dell IDRAC, IBM RSA.
 * Power Switch có thể điều khiển thông qua IP, như các thiêt bị APC master.
 * Disk-based STONISH, sử dụng shared disk device để thực hiện STONISH operation.
 * Hypervisor-based STONISH, nói chuyện với hypervisor trong môi trường ảo hóa.
 * Các giải pháp STONISH mềm (không nên dùng).

