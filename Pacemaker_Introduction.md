### 1. Giới thiệu chung
Pacemaker là thành phần trong cluster đảm trách việc quản lý tài nguyên (resource management)

![pacemaker_relate_to_otherpart](images/pacemaker_1.jpg)


#### 1.1. Resource Agents
Để thực hiện việc quản lý, nó sử dụng các resource agents. Resource agents là một script cluster dùng để start, stop, giám sát resources. Resource agent cũng định nghĩa các thành phần được quản lý bởi cluster.

#### 1.2. Corosync/cman
Corosync là lớp quản lý node membership, Pacemaker nhận cập nhật về các thay đổi trong trạng thái cluster membership, dựa vào đó để kích hoạt các sự kiện, vd như di dời resource.


#### 1.3. Storage layer
Pacemaker có thể được sử dụng để quản lý các thiết bị lưu trữ chia sẻ (shared storage devices). Để làm được điều này, cần phải sử dụng một thành phần quản lý lock phân tán (distributed lock manager_DLM). DLM đảm trách việc đồng bộ lock giữa các thiết bị lưu trữ, điều này đặc biệt quan trọng nếu lưu trữ chia sẻ được sử dụng, VD như cLVM2 clusterd logical volumes hoặc GSF2 và OCF2 clustered file systems.

### 2. Thành phần

![pacemaker_components](images/pacemaker_2.jpg)