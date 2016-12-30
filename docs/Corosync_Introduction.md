### 1. Giới thiệu chung
Corosync là giải pháp để quản lý và giám sát node membership. Ngoài ra, trong các hệ thống Redhat, `cman` được dùng như cluster membership layer thay cho corosync.

#### 1.1. Kiến trúc

Các service được cung cấp bởi Corosync Service Engine Internal API:
 - Totem Single Ring Ordering và giao thức Membership, cung cấp Extended Virtual Synchorony model cho việc trao đổi gói tin và tạo membership.
 - coroipc: hệ thống IPC chia sẻ bộ nhớ hiệu năng cao
 - Object Database, cung cấp một database in-memory
 - Hệ thống để định tuyến IPC và bản tin Totem tởi các engine service tương ứng.

Corosync cũng cung cấp các engine service mặc định dùng C API:
 - cpg: Closed Process Group
 - sam: Simple Availability Manager
 - confdb: Cấu hình DB
 - quorum: Cung cấp thông tin về tình trạng quorum


Tham khảo:

[1] - https://en.wikipedia.org/wiki/Corosync_Cluster_Engine