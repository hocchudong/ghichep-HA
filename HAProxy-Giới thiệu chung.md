###I. Giới thiệu chung

HAProxy - High Availability Proxy
	- Phần mềm mã nguồn mở Load Balancer và proxying TCP/HTTP chạy trên Linux, Solaris và FreeBSD
	- Được dùng nhằm tăng hiệu năng và đáp ứng sự phân tán khối lượng công việc trên các server (web, ứng dụng, database...)
	- Được sử dụng trong rất nhiều môi trường cấu hình cao, như : Github, Imgur, Instagram, Twitter...
	
###II. Một số thuật ngữ trong HAProxy

####1. Access Control List (ACL)

	ACLs được dùng để test 1 vài điều kiện và thực hiện hành động (chọn server, block 1 request...) dựa vào kết quả kiểm 
	tra. Sử dụng ACLs cho phép forward linh hoạt các network traffic dựa vào cac nhân tố như pattern-matching và một số 
	kết nối tới backend.
	
VD : **acl url_blog path_beg /blog**

	ACL sẽ match nếu như phần request của user bắt đầu với /blog, nó sẽ match với request của http://yourdomain.com/blog/blog-entry-1

####2. Backend

	Backend là tập hợp của các server mà nhận được request. Các backed được định nghĩa trong section "backend" của file cấu
	hình HAProxy. Trong phần lớn form cơ bản, 1 backend được định nghĩa bởi :
	
		- thuật toán cân bằng tải nào được dùng
		- danh sách các server và port
		
	Một backend có thể chưa một hoặc nhiều server trong nó, thêm nhiều server vào backend của bạn sẽ tiềm năng công suốt 
	tải bằng việc phân bố tải tới các server. Trong trường hợp các backend server có thể không khả dụng, việc này sẽ làm 
	tăng độ tin cậy.
	
VD : Cấu hình 2 backend, ưeb-backend và blog-backend với 2 web server :

		backend web-backend <ul>
		   balance roundrobin
		   server web1 web1.yourdomain.com:80 check
		   server web2 web2.yourdomain.com:80 check
</ul>
		backend blog-backend
		   balance roundrobin
		   mode http
		   server blog1 blog1.yourdomain.com:80 check
		   server blog1 blog1.yourdomain.com:80 check
	Thuật toán sử dụng là roundrobin, "mode http" chỉ ra rằng layer 7 proxying dợc dùng. option "check" chỉ ra rằng các 
	check sẽ được thực hiện ở các server này.
	
3. Frontend
	Frontend định nghĩa việc các request nên được chỏ tới cách backend như thế nào. Frontend được định nghĩa trong section 
	"frontend" của file cấu hình HAProxy :
		- một set các IP và port (vd 10.1.1.7:80, *:443...)
		- ACLs
		- "use_backend" rules, định nghĩa ra các backend nào sẽ được dùng dựa vào điều kiện ACL được match, and/or một 
		"default_backend" rule kiểm soát các case khác.
	Một frontend có thể được cấu hình theo nhiều kiểu của network traffic.
	
Các dạng của Load Balancing
1. Không Load Balancing
	User sẽ truy cập trực tiếp đến webserver. Nếu web server bị sập, hoặc số lượng người truy cập quá đông sẽ dẫn đến quá 
	tải, việc kết nối sẽ không thể thực hiện được.
2. Layer 4 Load Balancing
	Cân bằng tải kiểu này sẽ chỏ user traffic dựa vào IP range và port (VD : 1 request tới từ http://yourdomain.com/anything, 
	traffic sẽ được chỏ tới backend mà được kiểm soát bởi yourdomain.com trên port 80)
	Hình 
	User truy cập tới load balancer, request của user sẽ được chỏ tới "web-backend" group của các backend server. Bất kỳ 
	backend server nào được chọn sẽ phản hồi trực tiếp tới request của user. Thông thường, tất cả các user trong web-backend
	nên có nội dung thống nhất, trong khi user nhận được nội dung không đồng nhất. Chú ý rằng các server kết nối tới cùng 
	một database.
3. Layer 7 Load Balancing
	Hình
	Nếu người dùng request yourdomain.com/blog, chúng được chỏ tới "blog" backend - một set các server chạy 1 khối ứng 
	dụng. Các request khác được chỏ tới "web-backend", chạy ứng dụng khác. Cả 2 backend đều được chỏ tới cùng 1 database..
VD : 		
		frontend http
		  bind *:80
		  mode http

		  acl url_blog path_beg /blog
		  use_backend blog-backend if url_blog

		  default_backend web-backend
Giải thích : 
		mode là "http", kiểm soát tất cả incoming traffic trên port 80. 
		acl url_blog path_beg /blog : match với một request nếu như phần đầu của user bắt đầu với /blog
		use_backend blog-backend if url_blog : sử dụng ACLs để proxy traffic tới blog-backend
		default_backend web-backend : chỉ ra tất cả các traffic khác sẽ chỏ tới web-backend
Các thuật toán Cân bằng tải
	Các thuật toán cân bằng tải được dùng để chỉ ra rằng server nào, trong 1 backend. sẽ được chọn khi cân bằng tải. 
	HAProxy đưa ra 1 vài lựa chọn cho các thuật toán. Các server có thể được gán parameter "weight" để đánh giá server 
	có được chọn thường xuyên không, so sánh với các server khác.
  Một vài thuật toán thông dụng
1. rounrobin
	Round Robin chọ server nào được quay. Đây là thuật toán mặc định.
2. leastconn
	Chọn server với số các kết nối ít nhất - được đề xuất với các session dài hạn. Các server trong cùng 1 backend được 
	quay vòng với round-robin
3. source
	Chọn server dể dùng dựa vào source IP. Phương thức này đảm bảo user sẽ kết nối tới cùng 1 server.
	
Sticky Session
	Một vài ứng dụng yêu cầu user tiếp tục kết nối tới cùng backend server. Điều này được thực hiện bởi sticky session, 
	yêu cầu sử dụng "appsession" parameter ở trên backend.

Health Check	
	HAProxy sử dụng health checks để xác định nếu 1 backend server khả dụng cho quá trình request, nó sẽ tránh được việc 
	nếu backend không còn khả dụng, thì việc remove server ra khỏi backend đó phải làm bằng tay. Mặc định thì health
	check sẽ cố khởi tạo 1 kết nối TCP tới server, nó kiểm tra nếu backend server được lắng nghe trên IP và port được cấu 
	hình.
	Nếu 1 server fail việc health check, vậy nên không thể tiếp nhận request, nó sẽ tự động tắt trong backend traffic sẽ 
	chỏ đến nó đến khi nó khả dụng trở lại. NẾu tất cả các server tron backend fail, service sẽ không khả dụng đến khi có 
	ít nhất 1 tron các backend server khả dụng trở lại.
	Cho một số loại nhất định của backend, như là database server trong 1 số trường hợp nhất định, health check mặc định 
	là không đủ để chỉ ra server là vẫn trong tình trạng khỏe mạnh.