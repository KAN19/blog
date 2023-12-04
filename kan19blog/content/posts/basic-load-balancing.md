---
title: "Load Balancing là gì? Có ăn được hong?"
description: "Load Balancing là gì? Có ăn được hong?"
date: 2023-12-04T17:16:25+07:00
tags: ["backend", "microservices"]
---


## 1. Ví dụ mô phỏng

Đây là quán bánh tráng cuộn chấm me Ku Kẹc. Ban đầu, quán cũng chỉ là 1 quầy nhỏ trong ngõ. 

![load-balancer-1.png](/load-balancing/load-balancer-1.png)

Nhưng nhờ khả năng marketing và chất lượng sản phẩm tốt, quán bánh tráng cuộn chấm me ngày càng nhiều khách hơn.

![load-balancer-2.png](/load-balancing/load-balancer-2.png)

Vì có quá nhiều khách order, cửa hàng thường xuyên bị quá tải. Nhiều khách phải chờ lâu và cảm thấy không hài lòng.

Nhìn thấy tình hình như vậy, Ku Kẹc quyết định mở thêm 2 chi nhánh nữa ở các khu lân cận. 
Chủ quán cũng thành lập tổ tiếp nhận và điều phối đơn hàng với 1 số điện thoại duy nhất, chuyên nhận đơn hàng và điều phối đến các chi nhánh có ít order, hoặc là có khoảng cách địa lý gần với người order hơn. 

![loadbalancer-3.png](/load-balancing/loadbalancer-3.png)

Trong bức tranh này, tổ tiếp nhận và điều phối đơn hàng chính là 1 abstract của việc load balancing, tức là cân bằng tải.

## 2. Load balancer là gì?

Load balancing là việc phân phối lưu lượng truy cập đến một nhóm các backend servers. 

Module thực hiện việc load balancing gọi là load balancer. Load balancer sẽ đứng trước server và routing các request của client đến các server có khả năng thực hiện các request đó, đảm bảo không có server nào nhận quá nhiều request. 
![load-balancer-4.png ](/load-balancing/load-balancer-4.png)

Load balancer và công việc balancing cũng tương tự như đổi tiếp nhận đơn hàng của cửa hàng bánh tráng cuộn Ku Kẹc, còn các chi nhánh cửa hàng cũng tương tự như các server hệ thống. 

Load balancer thường chia thành 2 loại: Hardware-based và software based. 
![load-balancer-5.png ](/load-balancing/load-balancer-5.png)

Hardware-based load balancer là các thiết bị vật lý chuyên dụng. Các thiết bị này thường sẽ sử dụng các bộ xử lý đặc thù cho việc load balancing. 

Software-based load balancer thì tương tự như một phần mềm chúng ta dùng hằng ngày. Chúng sẽ được cài đặt trên một server bình thường và chịu trách nhiệm nhận và điều tuyến các request.

## 3. Load balancers hoạt động như thế nào?
Như đã nói ở trên, Load balancer đóng vai trò là người trung gian giữa client và server.  Load balancer sẽ nhận các request từ clients, sau đó điều phối đến servers dựa trên tình trạng của server đó và thuật toán được cài đặt ở load balancers.

![load-balacner-6.png](/load-balancing/load-balancer-6.png)

### Các thuật toán phổ biến cho việc load balancing

#### Round Robin:

Các request được phân bổ một cách luân phiên hoặc xoay vòng

![round-robin.png](/load-balancing/round-robin.png)

Lấy ví dụ từ của hàng bánh tráng cuộn ở trên, đơn đầu tiên sẽ được xử lý bởi chi nhánh 1, đơn thứ 2 sẽ được xử lý bởi chi nhánh 2, đơn thứ 3 sẽ được xử lý bởi chi nhánh 3, và đơn thứ 4 sẽ được xử lý bởi chi nhánh 1,… và cứ tiếp tục như vậy

#### Weighted Round Robin

Thuật toán này khá giống với Round Robin ở trên. Điều khác biệt ở đây là các server sẽ được đánh trọng số. Dựa vào các trọng số này mà các request sẽ được phân phối đến các server. Các trọng số này có thể được đánh dựa trên khả năng xử lý request của server. Chính vì vậy, sẽ có một vài servers phải xử lý nhiều request hơn. 

![weighted-round-robin.png](/load-balancing/weighted-round-robin.png)

Cũng ví dụ từ cửa hàng bánh tráng cuộn trên, giả sử chi nhánh số 2 được đánh trọng số là 0.6, còn 2 chi nhánh còn lại đều là 0.2. Tổ điều phối sẽ dựa trên các trọng số này mà điều phối đơn hàng. Sau 100 đơn hàng, tổng số đơn hàng chi nhánh 2 xử lý sẽ khoảng 60 đơn, trong khi tổng số đơn hàng được xử lý bởi chi nhánh 2 và 3 đều lần lượt khoảng 20. 

#### Least Connection Method

Với phương pháp này, request sẽ được forward đến server đang có số lượng request hoặc connection ít nhất. Để làm điều này, load balancer cần phải thực hiện thêm một số bước tính toán để xác định chính xác đích đến. 

![least-connection.png](/load-balancing/least-connection.png)

#### Least Response Time Method

Với phương pháp này, request sẽ được điều hướng đến server có ít connection nhất và thời gian phản hồi trung bình nhanh nhất. Cũng như phương pháp trên, phương pháp này cần thực hiện thêm một số phép toán để xác định đích đến. 

![least-response.png](/load-balancing/least-response.png)

#### Source IP Hash

Với phương pháp này, request sẽ được forward đến server dựa trên địa chỉ IP của clients. Các địa chỉ IP này sẽ được tính toán dựa trên một thuật toán và forward đến server tương ứng.

Ví dụ từ cửa hàng bánh tráng cuộn: Người A đặt bánh tráng cuộn có vị trí là VNG Campus. Dựa trên vị trí của người đặt, các điều phối viên sẽ chỉ forward đơn hàng đến 1 chi nhánh nhất định. 

![ip-hash.png](/load-balancing/ip-hash.png)

## 4. Làm sao để chọn thuật toán phù hợp cho Load Balancer?

Việc chọn các thuật toán cho balancer phụ thuộc vào đặc điểm và yêu cầu cụ thể của hệ thống. Vậy nên trong từng trường hợp cụ thể, chúng ta có thể chọn thuật toán cho phù hợp: 

#### Round robin:

Chúng ta chọn Round robin khi: 

- Các servers có khả năng xử lý tương tự nhau
- Khi chúng ta muốn thứ gì đó đơn giản

#### Weighted Round Robin:

Chúng ta sẽ chọn Weighted Round Robin khi: 

- Các servers có khả năng xử lý khác nhau, và chúng ta muốn các server xử lý request theo tỉ lệ khác nhau.
- Hữu ích trong trường hợp bạn có máy chủ với sức mạnh xử lý hoặc tài nguyên khác nhau.

#### Least Connection Method:

Chúng ta sẽ chọn Least connection method khi: 

- Các servers đang chịu tải khác nhau, và chúng ta muốn hướng traffic vào server có ít active connections hơn.
- Phù hợp cho việc số active connections là chỉ báo tốt về sức khoẻ của server

#### Least Response Time Method:

Chúng ta sẽ chọn Least response time method khi: 

- Khi ta đo được thời gian xử lý, và chúng ta muốn hướng lưu lượng truy cập đến server có phản hồi nhanh nhất
- Hữu ích trong việc tối ưu hoá tốc độ phản hồi của hệ thống.

#### Source IP Hash:

Chúng ta sẽ chọn IP Hash khi:

- Duy trì session, đảm bảo các yêu cầu từ cùng 1 IP được hướng đến cùng 1 server
- Hữu ích trong các ứng dụng khi việc thay đổi server có thể gây gián đoạn hoặc dữ liệu không nhất quán.

## 5. Lợi ích và bất lợi khi triển khai Load Balancer

### Lợi ích
- Improved performance: Vì có nhiều server, và các request được phân bổ đều trên các servers, nên performance được cải thiện, tránh tình trạng overloads
- High availability: Vì các server xử lý các request độc lập, nên trong trường hợp một trong số các server bị lỗi, thì hệ thống vẫn hoạt động nhờ các server còn lại.
- Scalability: Dể dàng mở rộng hệ thống. Trong trường hợp traffic tăng, chúng ta đơn giản chỉ cần thêm server hoặc thêm services để xử lý chúng.

### Bất lợi

- Single point of failure: Vì tất cả các request đều thông qua load balancer trước tiên, nên trong trường hợp load balancer ngủm củ tỏi, thì coi như hệ thống cũng sập.
- Cost: Một số loại load balancer có thể khá đắt đỏ (đối với hardware-based balancers). Ngoài ra việc chạy load balancers cũng tiêu hao tài nguyên trên server (Đối với software-based balancer)
- Performance bottlenecks: Load balancers đôi khi cũng trở thành nút thắt cổ chai trong hệ thống, do không kịp xử lý các incoming request.
    
    ![pros-cons.png](/load-balancing/pros-cons.png)
    
Các lợi ích và bất lợi của load balancer thiệt ra còn rất nhiều, nhưng mình chỉ liệt kê những điều mình thấy rõ nhất :D

## 6. Handling disadvantages
Khi triển khai load balancing, chúng ta cần có những chiến thuật và phương pháp xử lý khi gặp những vấn đề bất lợi mà mình đã nêu trên. Ở đây mình sẽ nói 2 ý là Single point of failure và performance bottlenecks. Cái "Cost" kia thì rõ ràng rùi. Để khắc phục "cost" thì chỉ có nước bơm thêm tiền dô thui

### Single point of failure:

- Redundancy: Tức là chúng ta sẽ triển khai dự phòng các load balancers. Trong trường hợp load balancers chính bị fail, một con load balancer dự phòng sẽ được triển khai ngay lập tức.
- Cloud-based load balancers: Chúng ta có thể cân nhắc sử dụng balancer từ các cloud provider như AWS,..
- Monitoring and Alerting: Triển khai hệ thống giám sát và cảnh báo để có thể xử lý kịp thời trong trường hợp load balancer có dấu hiệu xấu.

### Performance bottlenecks:

- Load balancer scaling: Nếu load balancers trở thành bottlenecks của hệ thống, hãy cân nhắc việc scale thêm load balancers.
- Optimize load balancer configuration
- Cache:
- Content Delivery Networks (CDNs)

## Take away ☕
![take-away.png](/load-balancing/take-away.png)

- Load balancer là trung gian giữa request từ client đến server
- Load balancing rất cần thiết cho việc mở rộng hệ thống
- Có nhiều loại load balancer khác nhau và nhiều kịch bản triển khai khác nhau.
- Lựa chọn thuật toán cho load balancers là rất quan trọng trong việc tối ưu traffic giữa các server với nhau
- Việc triển khai load balancer mang lại nhiều lợi ích. Tuy nhiên, chúng ta cần phải có chiến thuật và kịch bản khi có lỗi phát sinh.
    